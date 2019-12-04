# Awsome Confluent  

## 参考文档

https://docs.confluent.io/current/quickstart/ce-quickstart.html#step-6-stop-cp



## 安装说明	

### 下载压缩包

https://packages.confluent.io/archive/5.3/confluent-5.3.1-2.12.zip

解压完成如下图

![](https://i.loli.net/2019/11/22/1PYgTwGcDXN5ma9.png)

进入解压后的文件夹。创建一个目录叫`confluentCli`。

### 安装confluent Cli工具

然后执行命令安装confluent Cli工具。它会设置一些环境变量，让系统知道启动的参数。

```SHELL
~/Downloads/confluent-5.3.1 » curl -L https://cnfl.io/cli | sh -s -- -b /Users/huzekang/Downloads/confluent-5.3.1/confluentCli/bin
```

等待几分钟，安装完成后可以看到。

![](https://i.loli.net/2019/11/22/2za1knwf5mYSRuE.png)



### 设置环境变量

![](https://i.loli.net/2019/11/22/Ciwgz65v4Xth2nJ.png)

### 启动confluent

```
~/Downloads/confluent-5.3.1 » ./confluentCli/bin/confluent local start
```

#### 浏览器查看

看到控制台输出，等待一两分钟后打开http://localhost:9021/clusters即可。

```
Starting zookeeper
zookeeper is [UP]
Starting kafka
kafka is [UP]
Starting schema-registry
schema-registry is [UP]
Starting kafka-rest
kafka-rest is [UP]
Starting connect
connect is [UP]
Starting ksql-server
ksql-server is [UP]
Starting control-center
control-center is [UP]

```

可以看到页面打开如下

![](https://i.loli.net/2019/11/22/1kh9SePm4dFtXRA.png)

#### 工作目录

默认情况下confluent启动后会在在`/tmp`目录建工做目录，各个组件的数据和日志都能在这里找到。但其实我们不用自己找日志看，可以用**confluent cli**的**log**命令查看，下面有说到。

![](https://i.loli.net/2019/12/02/MI3LlwpYBg7WKet.png)



### confluent cli常用命令

| Command                                                      | Description                                                  |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [confluent local acl](https://docs.confluent.io/current/cli/command-reference/confluent-local/confluent_local_acl.html#confluent-local-acl) | Specify ACL for a service.                                   |
| [confluent local config](https://docs.confluent.io/current/cli/command-reference/confluent-local/confluent_local_config.html#confluent-local-config) | Configure a connector.                                       |
| [confluent local consume](https://docs.confluent.io/current/cli/command-reference/confluent-local/confluent_local_consume.html#confluent-local-consume) | Consume data from topics.                                    |
| [confluent local current](https://docs.confluent.io/current/cli/command-reference/confluent-local/confluent_local_current.html#confluent-local-current) | Print the filesystem path of the data and logs of the services managed by the current `confluent local` command. |
| [confluent local demo](https://docs.confluent.io/current/cli/command-reference/confluent-local/confluent_local_demo.html#confluent-local-demo) | Run demos provided in GitHub repo https://github.com/confluentinc/examples. |
| [confluent local destroy](https://docs.confluent.io/current/cli/command-reference/confluent-local/confluent_local_destroy.html#confluent-local-destroy) | Delete the data and logs of the current Confluent Platform run. |
| [confluent local list](https://docs.confluent.io/current/cli/command-reference/confluent-local/confluent_local_list.html#confluent-local-list) | List all available services or plugins.                      |
| [confluent local load](https://docs.confluent.io/current/cli/command-reference/confluent-local/confluent_local_load.html#confluent-local-load) | Load a connector.                                            |
| [confluent local log](https://docs.confluent.io/current/cli/command-reference/confluent-local/confluent_local_log.html#confluent-local-log) | Read or tail the log of a service.                           |
| [confluent local produce](https://docs.confluent.io/current/cli/command-reference/confluent-local/confluent_local_produce.html#confluent-local-produce) | Produce data to topics.                                      |
| [confluent local start](https://docs.confluent.io/current/cli/command-reference/confluent-local/confluent_local_start.html#confluent-local-start) | Start all services or a specific service along with its dependencies. |
| [confluent local status](https://docs.confluent.io/current/cli/command-reference/confluent-local/confluent_local_status.html#confluent-local-status) | Get the status of all services or the status of a specific service and its dependencies. |
| [confluent local stop](https://docs.confluent.io/current/cli/command-reference/confluent-local/confluent_local_stop.html#confluent-local-stop) | Stop all services or a specific service its dependent services. |
| [confluent local top](https://docs.confluent.io/current/cli/command-reference/confluent-local/confluent_local_top.html#confluent-local-top) | View service resource usage.                                 |
| [confluent local unload](https://docs.confluent.io/current/cli/command-reference/confluent-local/confluent_local_unload.html#confluent-local-unload) | Unload a connector.                                          |
| [confluent local version](https://docs.confluent.io/current/cli/command-reference/confluent-local/confluent_local_version.html#confluent-local-version) | Print the Confluent CLI version.                             |



### 查看confluent 各组件日志

#### Kafka

```
confluent local log kafka -- -f
```

#### Zookeeper

```
confluent local log zookeeper -- -f
```

#### Kafka connect

```
confluent local log connect -- -f
```

#### ksql-server

```
confluent local log ksql-server  -- -f
```







### 关闭confluent

Stop Confluent Platform using the [Confluent CLI](https://docs.confluent.io/current/cli/index.html#cli) [confluent local stop](https://docs.confluent.io/current/cli/command-reference/confluent-local/confluent_local_stop.html#confluent-local-stop) command.

```
<path-to-confluent-cli>/bin/confluent local stop
```

![](https://i.loli.net/2019/11/22/Ft7lyCTxdLkz8qO.png)

### 销毁confluent实例数据

Destroy the data in the Confluent Platform instance with the [confluent local destroy](https://docs.confluent.io/current/cli/command-reference/confluent-local/confluent_local_destroy.html#confluent-local-destroy) command.

```
<path-to-confluent-cli>/bin/confluent local destroy
```



## Conflunet组件介绍



### Confluent Kafka Rest

#### 介绍

Kafka REST代理为Kafka集群提供了RESTful接口。无需使用本机Kafka协议或客户端，即可轻松生成和使用消息，查看集群状态以及执行管理操作。用例示例包括从使用任何语言构建的任何前端应用向Kafka报告数据，将消息吸收到尚不支持Kafka的流处理框架中以及编写管理操作脚本。

#### 部署

脚本 `bin/kafka-rest-start`和`bin/kafka-rest-stop`是推荐的启动和停止服务的方法。

#### 使用例子

```bash
 # Get a list of topics
    $ curl "http://localhost:8082/topics"
      
      ["__consumer_offsets","jsontest"]

    # Get info about one topic
    $ curl "http://localhost:8082/topics/jsontest"
    
      {"name":"jsontest","configs":{},"partitions":[{"partition":0,"leader":0,"replicas":[{"broker":0,"leader":true,"in_sync":true}]}]}

    # Produce a message with JSON data
    $ curl -X POST -H "Content-Type: application/vnd.kafka.json.v2+json" \
          --data '{"records":[{"value":{"name": "testUser"}}]}' \
          "http://localhost:8082/topics/jsontest"
          
      {"offsets":[{"partition":0,"offset":3,"error_code":null,"error":null}],"key_schema_id":null,"value_schema_id":null}

    # Create a consumer for JSON data, starting at the beginning of the topic's
    # log. The consumer group is called "my_json_consumer" and the instance is "my_consumer_instance".
    
    $ curl -X POST -H "Content-Type: application/vnd.kafka.v2+json" -H "Accept: application/vnd.kafka.v2+json" \
    --data '{"name": "my_consumer_instance", "format": "json", "auto.offset.reset": "earliest"}' \
    http://localhost:8082/consumers/my_json_consumer
          
      {"instance_id":"my_consumer_instance","base_uri":"http://localhost:8082/consumers/my_json_consumer/instances/my_consumer_instance"}
      
    # Subscribe the consumer to a topic
    
    $ curl -X POST -H "Content-Type: application/vnd.kafka.v2+json" --data '{"topics":["jsontest"]}' \
    http://localhost:8082/consumers/my_json_consumer/instances/my_consumer_instance/subscription
    # No content in response
      
    # Then consume some data from a topic using the base URL in the first response.

    $ curl -X GET -H "Accept: application/vnd.kafka.json.v2+json" \
    http://localhost:8082/consumers/my_json_consumer/instances/my_consumer_instance/records
      
      [{"key":null,"value":{"name":"testUser"},"partition":0,"offset":3,"topic":"jsontest"}]
   
    # Finally, close the consumer with a DELETE to make it leave the group and clean up
    # its resources.  
    
    $ curl -X DELETE -H "Accept: application/vnd.kafka.v2+json" \
          http://localhost:8082/consumers/my_json_consumer/instances/my_consumer_instance
      # No content in response
```



### Conflunet Schema Registry

#### 介绍

它提供了一个RESTful接口，用于存储和检索Avro模式。它存储所有模式的版本历史记录，提供多个兼容性设置，并允许根据配置的兼容性设置来发展模式。它提供了插入到Kafka客户端的序列化程序，这些序列化程序处理模式存储和以Avro格式发送的Kafka消息的检索。

#### Rest Api

The following assumes you have Kafka and an instance of the Schema Registry running using the default settings.

```
# Register a new version of a schema under the subject "Kafka-key"
$ curl -X POST -H "Content-Type: application/vnd.schemaregistry.v1+json" \
    --data '{"schema": "{\"type\": \"string\"}"}' \
    http://localhost:8081/subjects/Kafka-key/versions
  {"id":1}

# Register a new version of a schema under the subject "Kafka-value"
$ curl -X POST -H "Content-Type: application/vnd.schemaregistry.v1+json" \
    --data '{"schema": "{\"type\": \"string\"}"}' \
     http://localhost:8081/subjects/Kafka-value/versions
  {"id":1}

# List all subjects
$ curl -X GET http://localhost:8081/subjects
  ["Kafka-value","Kafka-key"]

# List all schema versions registered under the subject "Kafka-value"
$ curl -X GET http://localhost:8081/subjects/Kafka-value/versions
  [1]

# Fetch a schema by globally unique id 1
$ curl -X GET http://localhost:8081/schemas/ids/1
  {"schema":"\"string\""}

# Fetch version 1 of the schema registered under subject "Kafka-value"
$ curl -X GET http://localhost:8081/subjects/Kafka-value/versions/1
  {"subject":"Kafka-value","version":1,"id":1,"schema":"\"string\""}

# Fetch the most recently registered schema under subject "Kafka-value"
$ curl -X GET http://localhost:8081/subjects/Kafka-value/versions/latest
  {"subject":"Kafka-value","version":1,"id":1,"schema":"\"string\""}

# Delete version 3 of the schema registered under subject "Kafka-value"
$ curl -X DELETE http://localhost:8081/subjects/Kafka-value/versions/3
  3

# Delete all versions of the schema registered under subject "Kafka-value"
$ curl -X DELETE http://localhost:8081/subjects/Kafka-value
  [1, 2, 3, 4, 5]

# Check whether a schema has been registered under subject "Kafka-key"
$ curl -X POST -H "Content-Type: application/vnd.schemaregistry.v1+json" \
    --data '{"schema": "{\"type\": \"string\"}"}' \
    http://localhost:8081/subjects/Kafka-key
  {"subject":"Kafka-key","version":1,"id":1,"schema":"\"string\""}

# Test compatibility of a schema with the latest schema under subject "Kafka-value"
$ curl -X POST -H "Content-Type: application/vnd.schemaregistry.v1+json" \
    --data '{"schema": "{\"type\": \"string\"}"}' \
    http://localhost:8081/compatibility/subjects/Kafka-value/versions/latest
  {"is_compatible":true}

# Get top level config
$ curl -X GET http://localhost:8081/config
  {"compatibilityLevel":"BACKWARD"}

# Update compatibility requirements globally
$ curl -X PUT -H "Content-Type: application/vnd.schemaregistry.v1+json" \
    --data '{"compatibility": "NONE"}' \
    http://localhost:8081/config
  {"compatibility":"NONE"}

# Update compatibility requirements under the subject "Kafka-value"
$ curl -X PUT -H "Content-Type: application/vnd.schemaregistry.v1+json" \
    --data '{"compatibility": "BACKWARD"}' \
    http://localhost:8081/config/Kafka-value
  {"compatibility":"BACKWARD"}
```



### Confluent Kafka Connect

#### 支持插件

https://www.confluent.io/hub/#download

#### 常用命令

我们可以通过`  confluent local list connectors`查看已经安装的connect。

![](https://i.loli.net/2019/12/02/nVe5Zc3Eklmor68.png)

#### Rest Api

列出此工作程序上可用的连接器插件

```
  curl localhost:8083/connector-plugins | jq
[
  {
    "class": "io.confluent.connect.replicator.ReplicatorSourceConnector"
  },
  {
    "class": "org.apache.kafka.connect.file.FileStreamSourceConnector"
  },
  {
    "class": "io.confluent.connect.jdbc.JdbcSinkConnector"
  },
  {
    "class": "io.confluent.connect.jdbc.JdbcSourceConnector"
  },
  {
    "class": "io.confluent.connect.hdfs.HdfsSinkConnector"
  },
  {
    "class": "org.apache.kafka.connect.file.FileStreamSinkConnector"
  }
]
```



**列出工作程序上的活动连接器**

```
  curl localhost:8083/connectors
["local-file-sink"]
```



重新启动连接器

```
  curl -X POST localhost:8083/connectors/local-file-sink/restart
(no response printed if success)
```



**获取某个连接器的任务**

```
  curl localhost:8083/connectors/local-file-sink/tasks | jq
[
  {
    "id": {
      "connector": "local-file-sink",
      "task": 0
    },
    "config": {
      "task.class": "org.apache.kafka.connect.file.FileStreamSinkTask",
      "topics": "connect-test",
      "file": "test.sink.txt"
    }
  }
]
```



重新启动任务（成功则不打印响应）

```
curl -X POST localhost:8083/connectors/local-file-sink/tasks/0/restart
```



暂停连接器（如果成功，则不打印响应）（如果连接器与之交互的系统需要停机，则很有用）

```
curl -X PUT localhost:8083/connectors/local-file-sink/pause
```



恢复连接器（如果成功，将不打印响应）

```
curl -X PUT localhost:8083/connectors/local-file-sink/resume
```



更新连接器配置

```
  curl -X PUT -H "Content-Type: application/json" --data '{"connector.class":"FileStreamSinkConnector","file":"test.sink.txt","tasks.max":"2","topics":"connect-test","name":"local-file-sink"}' localhost:8083/connectors/local-file-sink/config
{
  "name": "local-file-sink",
  "config": {
    "connector.class": "FileStreamSinkConnector",
    "file": "test.sink.txt",
    "tasks.max": "2",
    "topics": "connect-test",
    "name": "local-file-sink"
  },
  "tasks": [
    {
      "connector": "local-file-sink",
      "task": 0
    },
    {
      "connector": "local-file-sink",
      "task": 1
    }
  ]
}
```



获取连接器状态

```
  curl localhost:8083/connectors/local-file-sink/status | jq
{
  "name": "local-file-sink",
  "connector": {
    "state": "RUNNING",
    "worker_id": "192.168.86.101:8083"
  },
  "tasks": [
    {
      "state": "RUNNING",
      "id": 0,
      "worker_id": "192.168.86.101:8083"
    },
    {
      "state": "RUNNING",
      "id": 1,
      "worker_id": "192.168.86.101:8083"
    }
  ]
}
```



获取连接器配置

```
  curl localhost:8083/connectors/local-file-sink | jq
{
  "name": "local-file-sink",
  "config": {
    "connector.class": "FileStreamSinkConnector",
    "file": "test.sink.txt",
    "tasks.max": "2",
    "topics": "connect-test",
    "name": "local-file-sink"
  },
  "tasks": [
    {
      "connector": "local-file-sink",
      "task": 0
    },
    {
      "connector": "local-file-sink",
      "task": 1
    }
  ]
}
```



删除连接器（如果成功，将不打印响应）

```
curl -X DELETE localhost:8083/connecto
```





### Confluent Ksql

#### 官方文档

https://docs.confluent.io/current/ksql/docs/developer-guide/syntax-reference.html

#### 官方cookbook

https://www.confluent.io/stream-processing-cookbook/

包含了

- stream ETL例子

- data warning例子

- data serialization例子

  

#### 使用例子

##### 生成数据

使用confluent ksql-datagen生成一些自带的模拟数据。

```
ksql-datagen quickstart=clickstream format=json topic=clickstream maxInterval=500 iterations=5000
```

这里使用的是点击事件的quickstart。可以看到不断在写入点击日志到topic clickstream中。

![](https://i.loli.net/2019/12/02/iIqNn8Q5VCr3kRK.png)

使用的是json格式。在web界面看到的数据是这样的，是没有schema的。

![](https://i.loli.net/2019/12/02/kYWUZeFS1KwyN6u.png)

如果上面的format用avro的话，看到的数据是有schema的。

**注意：只有avro的才可以在topic schema中看到自动定义好的格式。其他格式只有在建立stream和table时手动创建schema。**

![](https://i.loli.net/2019/12/02/8qie9tQlaYLS7yz.png)



而如果使用的format是delimited，则看到每条message是用逗号分隔的一条字符串。

![](https://i.loli.net/2019/12/02/etcpQmD7Rl5n6IX.png)





##### 使用ksql建stream

![](https://i.loli.net/2019/12/02/lndIwJgpxM6bVQ9.png)

这里使用ksql定义了topic clickstream 的stream schema。

```SQL
create stream clickstream (_time BIGINT, time VARCHAR,ip VARCHAR,request VARCHAR,status INT,userid INT ,bytes BIGINT ,agent VARCHAR) WITH(KAFKA_TOPIC ='clickstream',VALUE_FORMAT='JSON');
```

##### 使用ksql读出数据

当上面的json格式的datagen还在运行时，我们用ksql客户端使用下面语句可以看到不断能读出数据。

```
select * from clickstream
```

![](https://i.loli.net/2019/12/02/hLqiDIZ9nsfmrER.png)

当datagen关闭，ksql也不会退出而是继续监听，如果还有新数据就能继续读，否则只有收到关闭才会结束监听。

##### 基于stream clickstream做过滤query

```
ksql> create stream test1 as select * from clickstream where userid >30;
```

该命令会创建一个topic叫TEST1，也会创建一个stream叫TEST1。

数据都是`userid>30`的。

![](https://i.loli.net/2019/12/02/x72po3IzLXABlCK.png)



#### ksql常用命令

##### 使用ksql列出stream和table

![](https://i.loli.net/2019/12/02/GufSWIyAQbl2Y1q.png)







## Kafka conncet convert

用于新增connect时指定message的key和value的转换器。

- org.apache.kafka.connect.json.JsonConverter
- org.apache.kafka.connect.storage.StringConverter
- io.confluent.connect.avro.AvroConverter（不写默认就是）

![](https://i.loli.net/2019/12/02/73ehSlp9BUk1x4i.png)



## 使用Debezium监听mysql CDC

### 说明

参考文档https://debezium.io/documentation/reference/0.10/tutorial.html

Debezium是一个分布式平台，可将您现有的数据库转换为事件流，因此应用程序可以查看并立即响应数据库中的每个行级更改。Debezium建立在[Apache Kafka](http://kafka.apache.org/)之上，并提供[Kafka Connect](http://kafka.apache.org/documentation.html#connect)兼容连接器，用于监视特定的数据库管理系统。Debezium在Kafka日志中记录数据更改的历史记录，您的应用程序将在此位置使用它们。这使您的应用程序可以轻松，正确，完整地使用所有事件。即使您的应用程序停止（或崩溃），在重新启动时，它也会开始使用中断处的事件，因此不会丢失任何内容。

Debezium 0.10.0.Final支持使用其[MySQL连接器](https://debezium.io/documentation/reference/0.10/connectors/mysql.html)监视MySQL数据库服务器。

### 安装

```
~/opt/confluent-5.3.1 » confluent-hub install debezium/debezium-connector-mysql:0.10.0
```

其中有一些选项，选择y即可

![](https://i.loli.net/2019/11/22/tA7uQpiYRaeZT5M.png)

安装完成可以看到目录下多了组件

![](https://i.loli.net/2019/11/22/XYcTMUfjx1Or6Ke.png)

重启confluent后即可使用该connector



### 使用docker启动一个开启binlog的mysql

```
~/opt/confluent-5.3.1 » docker run -it --rm --name mysql_binlog -p 3307:3306 -e MYSQL_ROOT_PASSWORD=debezium -e MYSQL_USER=mysqluser -e MYSQL_PASSWORD=mysqlpw debezium/example-mysql:0.10
```





### 配置使用mysql connector

可以通过web页面添加

![](https://i.loli.net/2019/11/22/qb1H6C5sAdQZziw.png)

也可以用curl添加。

```shell
curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" localhost:8083/connectors/ -d '{ "name": "inventory-connector", "config": { "connector.class": "io.debezium.connector.mysql.MySqlConnector", "tasks.max": "1", "database.hostname": "localhost", "database.port": "3307", "database.user": "debezium", "database.password": "dbz", "database.server.id": "184054", "database.server.name": "dbserver1", "database.whitelist": "inventory", "database.history.kafka.bootstrap.servers": "localhost:9092", "database.history.kafka.topic": "dbhistory.inventory" } }'
```

其中`"database.server.id"`和`database.server.name`都是可以随便写的。

`database.whitelist`则是要监听的数据库名。

`database.history.kafka.bootstrap.servers`是kafka server的地址。这里用的是confluent自带的，所以写本地地址即可。

`database.history.kafka.topic`是根据数据库对应的topic名，后续connector会把表名都读出来建成一个个topic。

**用curl命令添加完后，可以看到mysql控制台多了监听信息。**

![](https://i.loli.net/2019/11/22/EwD95R7vYth2Qkr.png)

此时web页面也多了一个运行中的connector。

![](https://i.loli.net/2019/11/22/fQgBHKyMSiAJ3OL.png)

打开topic页面，也能看到connector帮我们把所有表都建成了一个个topic。

![](https://i.loli.net/2019/11/22/qBDzwPvZnftE83X.png)

### 测试使用

1. 打开address topic页面。

![](https://i.loli.net/2019/11/22/2xmYc6iHbI1OL8C.png)

2. 使用navicat网address表中添加一条数据。

   ![](https://i.loli.net/2019/11/22/XLzr4oHmnsYcV2R.png)

3. 可以看到web 页面中多了一条message。

   ![](https://i.loli.net/2019/11/22/MZog8pieESXBsbc.png)





## jdbc connector将数据从oracle到mysql

### 配置source读取oracle

配置jdbc connect。

这里定义了一个导入schema为`YIBO_TEST`的数据库，导入`table.whitelist`中的表到kafka中。

默认不设置value和key的convert就是**avro**，因此对应的topic是有schema的。可以参考上面的`kafka connect kafka`。

默认情况下，`poll interval`是5000ms，因此会每隔5s就执行以bulk操作。**所以会导致kafka中数据重复**。所以要应该设置长一点。

![](https://i.loli.net/2019/12/02/s5RghVT6MFpL4uO.png)

使用的**bulk模式**，即对指定表全量导入。

```
{
  "name": "jdbc-oracle",
  "connector.class": "io.confluent.connect.jdbc.JdbcSourceConnector",
  "tasks.max": "10",
  "connection.url": "jdbc:oracle:thin:@192.168.1.130:1521:ORCL",
  "connection.user": "yibo_test",
  "connection.password": "*******",
  "connection.attempts": "10",
  "table.whitelist": "TB_CIS_PATIENT_INFO, TB_LIS_INDICATORS, TB_CIS_INHOS_FEE_DETAIL",
  "schema.pattern": "YIBO_TEST",
  "mode": "bulk",
  "topic.prefix": "GZFY."
}
```



![](https://i.loli.net/2019/12/02/oMnwBipOPYNmSHZ.png)

此时可以看到confluent新建的topics和正在导入的数据。

![](https://i.loli.net/2019/12/02/K4YcvJxRkOBgpeQ.png)

### 配置sink从kafka到mysql

这里会自动建表。

只能选择单一topic数据。

默认情况下，建表的表名和数据库名是根据topic名的。

```
{
  "name": "sink-mysql",
  "connector.class": "io.confluent.connect.jdbc.JdbcSinkConnector",
  "tasks.max": "6",
  "topics": [
    "GZFY.TB_CIS_PATIENT_INFO"
  ],
  "connection.url": "jdbc:mysql://192.168.1.150:3306/GZFY?serverTimezone=UTC",
  "connection.user": "root",
  "connection.password": "****",
  "auto.create": "true"
}
```

可以看到confluent会根据从oracle读到topic中的数据➕schema registry的信息在mysql自动建表了。

![](https://i.loli.net/2019/12/02/u3vZnMTmiVUbYwQ.png)