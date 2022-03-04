# Griffin0.7流式质检源码研究

## 源码准备

### 下载源码

```
https://github.com/apache/griffin.git
```

切换分支`griffin-0.7.0-rc0`

![image-20220304084824156](http://image-picgo.test.upcdn.net/img/20220304084824.png)



## 造数据

### 创建topic

```
bin/kafka-topics.sh \
--create \
--zookeeper localhost:12181/kafka \
--partitions 2 \
--replication-factor 1 \
--topic sss



bin/kafka-topics.sh \
--create \
--zookeeper localhost:12181/kafka \
--partitions 2 \
--replication-factor 1 \
--topic ttt
```





### 生成样本数据

```


cat > 1.txt <<EOF
{"name": "kevin", "age": 24}
{"name": "jason", "age": 25}
{"name": "jhon", "age": 28}
{"name": "steve", "age": 31}
EOF

cat > 2.txt <<EOF
{"name": "kevin", "age": 24}
{"name": "jason", "age": 25}
{"name": "steve", "age": 20}
EOF
```

### 将样本数据通过控制台发送kafka

```
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic sss  < /Volumes/Samsung_T5/opensource/griffin/hzktest/1.txt



bin/kafka-console-producer.sh --broker-list localhost:9092 --topic ttt  < /Volumes/Samsung_T5/opensource/griffin/hzktest/2.txt
```



## profiling类型质检本地idea调试



### 创建任务文件

### env.json

```
{
  "spark": {
    "log.level": "WARN",
    "checkpoint.dir": "file:///Volumes/Samsung_T5/opensource/griffin/chk/streaming_cp",
    "batch.interval": "1m",
    "process.interval": "3m",
    "config": {
      "spark.master": "local[*]"
    }
  },
  "sinks": [
    {
      "name": "MyConsoleSink",
      "type": "CONSOLE",
      "config": {
        "max.log.lines": 10
      }
    },
    {
      "name": "MyHDFSSink",
      "type": "HDFS",
      "config": {
        "path": "file:///Volumes/Samsung_T5/opensource/griffin/hzktest_result",
        "max.persist.lines": 10000,
        "max.lines.per.file": 10000
      }
    }
  ],
  "griffin.checkpoint": [
    {
      "type": "zk",
      "config": {
        "hosts": "localhost:12181",
        "namespace": "griffin/infocache",
        "lock.path": "lock",
        "mode": "persist",
        "init.clear": true,
        "close.clear": false
      }
    }
  ]
}

```

声明了两个输出，一个控制台一个hdfs，方便检查质检结果。

`batch.interval`是sparkstreaming每隔多久产生一个rdd。

`process.interval`是质检任务对对rdd持久化下来的数据，多久触发一下检查，并输出结果。

### dq.json

```
{
  "name": "流式profile检查",
  "process.type": "STREAMING",
  "data.sources": [
    {
      "name": "source",
      "connector": {
        "type": "KAFKA",
        "version": "0.8",
        "config": {
          "kafka.config": {
            "bootstrap.servers": "localhost:9092",
            "group.id": "group1",
            "auto.offset.reset": "smallest",
            "auto.commit.enable": "false"
          },
          "topics": "sss",
          "key.type": "java.lang.String",
          "value.type": "java.lang.String"
        },
        "pre.proc": [
          {
            "dsl.type": "df-ops",
            "in.dataframe.name": "this",
            "out.dataframe.name": "s1",
            "rule": "from_json"
          },
          {
            "dsl.type": "spark-sql",
            "out.dataframe.name": "this",
            "rule": "select upper(name) as name, age from s1"
          }
        ]
      },
      "checkpoint": {
        "type":"json",
        "file.path": "file:///Volumes/Samsung_T5/opensource/griffin/chk/source_cache/sss",
        "info.path": "source",
        "ready.time.interval": "10s",
        "ready.time.delay": "0",
        "time.range": [
          "0",
          "0"
        ]
      }
    }
  ],
  "evaluate.rule": {
    "rules": [
      {
        "dsl.type": "griffin-dsl",
        "dq.type": "PROFILING",
        "out.dataframe.name": "AgeProfile",
        "rule": "select count(name) as `cnt`, max(age) as `max`, min(age) as `min` from source",
        "out": [
          {
            "type": "metric",
            "name": "年龄字段统计profile"
          }
        ]
      },
      {
        "dsl.type": "griffin-dsl",
        "dq.type": "PROFILING",
        "out.dataframe.name": "NameGroup",
        "rule": "select name, count(*) as `cnt` from source group by name",
        "out": [
          {
            "type": "metric",
            "name": "按名字分组计数",
            "flatten": "array"
          }
        ]
      }
    ]
  },
  "sinks": [
    "MyConsoleSink",
    "MyHDFSSink"
  ]
}

```

声明了两个质控规则，一个是对年龄字段的profile，算最大最小值。一个是对名字进行分组计数。因为整个任务是流式的，不会停的，程序会根据前面声明的`process.interval`进行两个质控规则的计算，每次计算只处理之前产生的数据。

`checkpoint`会将每个rdd用json格式持久化下来写成文件。等待后面质检线程像离线时一样计算。

`pre.proc`应该是对rdd的数据用transform转成df后，简单的预处理数据。

`dq.type`是质检类型：

- profiling（对应实现类ProfilingExpr2DQSteps）
- ACCURACY（对应实现类AccuracyExpr2DQSteps）
- uniqueness（对应实现类UniquenessExpr2DQSteps）
- timeliness（对应实现类TimelinessExpr2DQSteps）
- distinct（对应实现类DistinctnessExpr2DQSteps）
- completeness(对应实现类CompletenessExpr2DQSteps)



## idea启动类

![image-20220304085558182](http://image-picgo.test.upcdn.net/img/20220304085558.png)

启动任务时，需要指定好前面准备好的两个文件。



## 本地启动调试效果

将准备好的数据写入kafka后，通过启动类启动质检任务。

可以看到rdd持久化下来的临时数据。

![image-20220304091110896](http://image-picgo.test.upcdn.net/img/20220304091110.png)

![image-20220304091025300](http://image-picgo.test.upcdn.net/img/20220304091025.png)

注意：这个目录的数据，在质检线程执行完计算后会清理

![image-20220304091316308](http://image-picgo.test.upcdn.net/img/20220304091316.png)

质检的结果会输出到文件目录下。

![image-20220304091340511](http://image-picgo.test.upcdn.net/img/20220304091340.png)

```
{"name":"流式profile检查","tmst":1646356200000,"value":{"cnt":52,"max":31,"min":24,"按名字分组计数":[{"name":"JHON","cnt":13},{"name":"JASON","cnt":13},{"name":"STEVE","cnt":13},{"name":"KEVIN","cnt":13}]},"metadata":{"applicationId":"local-1646356156840"}}

```



## completeness类型质检本地idea调试

dq完整性.json

```
{
  "name": "完整性检查",
  "process.type": "STREAMING",
  "data.sources": [
    {
      "name": "source",
      "connector": {
        "type": "KAFKA",
        "version": "0.8",
        "config": {
          "kafka.config": {
            "bootstrap.servers": "localhost:9092",
            "group.id": "group1",
            "auto.offset.reset": "smallest",
            "auto.commit.enable": "false"
          },
          "topics": "sss",
          "key.type": "java.lang.String",
          "value.type": "java.lang.String"
        },
        "pre.proc": [
          {
            "dsl.type": "df-ops",
            "in.dataframe.name": "this",
            "out.dataframe.name": "s1",
            "rule": "from_json"
          },
          {
            "dsl.type": "spark-sql",
            "out.dataframe.name": "this",
            "rule": "select upper(name) as name, age from s1"
          }
        ]
      },
      "checkpoint": {
        "type":"json",
        "file.path": "file:///Volumes/Samsung_T5/opensource/griffin/chk/source_cache/sss",
        "info.path": "source",
        "ready.time.interval": "10s",
        "ready.time.delay": "0",
        "time.range": [
          "0",
          "0"
        ]
      }
    }
  ],
  "evaluate.rule": {
    "rules": [
      {
        "dsl.type": "griffin-dsl",
        "dq.type": "completeness",
        "out.dataframe.name": "NameAgeComplete",
        "rule": "name, age",
        "out": [
          {
            "type": "metric",
            "name": "名字年龄完整性校验"
          }
        ]
      }
    ]
  },
  "sinks": [
    "MyConsoleSink",
    "MyHDFSSink"
  ]
}

```

分析源码可以知道实现类是CompletenessExpr2DQSteps，该step内部会执行几个SparkSqlTransformStep(即用sparksql生成df)。

1. 算出不符合的数据df
2. 算出总的数据df
3. leftjoin两个df得出结果，并将结果输出。



结果如下：

```
{"name":"完整性检查","tmst":1646361960000,"value":{"total":52,"incomplete":0,"complete":52},"metadata":{"applicationId":"local-1646361923992"}}

```

