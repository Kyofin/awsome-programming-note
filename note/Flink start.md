# Flink starter



## 参考文档

**[ververica flink forward](https://ververica.cn/developers-resources/)**

**[Flink on Yarn原理](https://www.infoq.cn/article/UFeFrdqSlqI3HyRHbPNm)**

##### [flink客户端操作的 5 种模式](https://mp.weixin.qq.com/s/KfuAZv2G0682NNzHv0iFfQ)

###### [Apache Flink 零基础入门](https://segmentfault.com/a/1190000020300020)

##### [Apache Flink 1.10.0 最新发布，年度最大规模版本升级！](https://blog.csdn.net/qq_18769269/article/details/104290984)

##### [5 个 TableEnvironment 我该用哪个？](https://mp.weixin.qq.com/s?__biz=MzU3Mzg4OTMyNQ==&mid=2247484827&idx=1&sn=0b9da36050532b8adb4e6ad11a0f4a23&chksm=fd3b8bd9ca4c02cf310a84c464e7a882742fbcaa1ed62520c2d045eb2e0bcd28278e85433c38&scene=0&xtrack=1&clicktime=1569728441&enterid=1569728441&ascene=7&)

**[Flink Table API&SQL的概念和通用API  (重点)](https://my.oschina.net/u/2935389/blog/3023112)**

**[Flink SQL-Client探索](https://blog.csdn.net/qq_36076256/article/details/89927268)**

[**flink SQL Client Beta官方文档**](https://ci.apache.org/projects/flink/flink-docs-stable/dev/table/sqlClient.html)

[flink cli官方文档](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/cli.html)



## Flink sql client

#### 下载sql client测试docker compose

https://github.com/huzekang/sql-training



#### Docker启动测试环境

```
docker-compose up
```



#### Docker启动容器中的sqlclient

```
docker-compose exec sql-client ./sql-client.sh
```

![](http://image-picgo.test.upcdn.net/img/20200325120435.png)

测试sql。

```
Flink SQL> select count(1) from DriverChanges;
```

访问http://localhost:8081/#/overview可以看到每个sql提交的作业。

![](http://image-picgo.test.upcdn.net/img/20200325120743.png)

终端中也可以看到数量在不断增加。

![](http://image-picgo.test.upcdn.net/img/20200325120816.png)



## flink shell

```
~/opt/flink-1.9.1 » bin/start-scala-shell.sh local
```

成功启动如下图：

![](https://i.loli.net/2019/11/20/mdFv5GAn3gQ8EkU.png)





## Flink sql client查询hive

1. 配置sql-client-defaults.yaml

   配置execution。

   ![](http://image-picgo.test.upcdn.net/img/20200409161116.png)

   配置catalogs。

   ![](http://image-picgo.test.upcdn.net/img/20200409161230.png)

   

2. 下载hive所需要依赖放入lib目录（这些依赖是根据官方文档hive Integration在maven仓库下载的）

   ![](http://image-picgo.test.upcdn.net/img/20200409161148.png)

   

3. 启动sql client。

   ```
   ~/opt/flink-1.10.0 » bin/sql-client.sh embedded                                       
   ```

   此时已经可以看到hive上的表了。

   ![](http://image-picgo.test.upcdn.net/img/20200409161418.png)
   
   **但是好像不能查询orc的数据表。（待解决）**



## Flink sql benchmark

参考：https://github.com/ververica/flink-sql-benchmark

该项目包含了**生成hive数据库**和**flink sql benchmark程序**。

1. git下载源码`https://github.com/ververica/flink-sql-benchmark`

2. MAVEN编译。

   ```
   mvn clean install
   ```

3. 启动standalone或者yarn session。

4. 提交自动基准测试程序。

   ```
   bin/flink run -c com.ververica.flink.benchmark.Benchmark /opt/flink-sql-benchmark/flink-tpcds/target/flink-tpcds-0.1-SNAPSHOT-jar-with-dependencies.jar --database tpcds_text_100 --hive_conf /etc/hive/conf --parallelism 30 --queries q1.sql
   ```

5. 可以看到flink ui上执行的sql

   ![](http://image-picgo.test.upcdn.net/img/20200429113025.png)





## 提交flink作业

####  YARN PER JOB (推荐)

在本地提交作业到远程yarn，不用在本地启动flink的任何服务。

```shell
 [root@cdh05 flink-1.10.0]#  ./bin/flink run -m yarn-cluster  -p 8 -ys 8 -ynm happy  -yjm 1024m -ytm 4096m ./examples/batch/WordCount.jar
```

通过环境变量的指定可以在本地以client模式提交到远程的yarn集群中执行。**注意的是提交到yarn需要hadoop的一些jar包，所以本地环境要装hadoop，并在环境变量中指定。**而且也要注意`conf/flink-conf.yaml`配置的core和内存是否大于yarn的容器资源，超过就申请不了容器，就启动不了作业了。

![](http://image-picgo.test.upcdn.net/img/20200121093101.png)

由于提交到yarn，输出返回都直接打印在控制台，虽然执行过程中可以在yarn上跳转到该作业的flink web查看每个步骤的耗时等参数，但是作业结束后就不能查看了，因为作业结束flink web也会随之关闭。

![](http://image-picgo.test.upcdn.net/img/20200121093930.png)

这时候就需要在**本地启动history sever来记录作业**了。

在flink yml中配置如下：

```yml
jobmanager.archive.fs.dir: hdfs://cdh04:8020/flink/v1.0copy/completed-jobs/
 
# The address under which the web-based HistoryServer listens.
historyserver.web.address: 0.0.0.0
 
# The port under which the web-based HistoryServer listens.
historyserver.web.port: 8088
 
# Comma separated list of directories to monitor for completed jobs.
historyserver.archive.fs.dir: hdfs://cdh04:8020/flink/v1.0copy/completed-jobs/,hdfs://cdh04:8020/flink/v1.0copy/h-completed-jobs/,hdfs://cdh04:8020/flink/completed-jobs/
 
# Interval in milliseconds for refreshing the monitored directories.
historyserver.archive.fs.refresh-interval: 1000

```

![](http://image-picgo.test.upcdn.net/img/20200121094159.png)

然后单独启动flink history sever即可，其他服务不用在本地启动。

```
~/opt/flink-1.9.1 » bin/historyserver.sh start                                                                
```

这时候跑的作业就会记录下来了。

![](http://image-picgo.test.upcdn.net/img/20200121094414.png)

实际的记录则根据配置保存在远程的hdfs上。

![](http://image-picgo.test.upcdn.net/img/20200121094550.png)



也可以在**yarn上查看flink的执行日志**。

首先根据控制台输出获取yarn上的appid。

![](http://image-picgo.test.upcdn.net/img/20200322170724.png)

打开浏览器，访问容器的地址。注意替换`1584856682455_0011`为控制台输出的appid。

各类日志：

http://huzekangdembp:8042/node/containerlogs/container_1584856682455_0011_01_000001/huzekang/

log日志：

http://huzekangdembp:8042/node/containerlogs/container_1584856682455_0011_01_000001/huzekang/jobmanager.log/?start=0







## 运行long time flink集群的方式



### 1. 使用yarn模式运行flink集群

##### 启动yarn

```
~/opt/hadoop-cdh/hadoop-2.6.0-cdh5.14.2/logs » ../sbin/start-all.sh              
```

##### 启动flink集群（用yarn模式）

```
~/Downloads/flink-1.9.1 » bin/yarn-session.sh
```

![](https://i.loli.net/2019/11/19/F3ia5ZXrv8cbLlk.png)

注意：在启动完后，只有一个1G的jobmanager，没有taskmanager的，只有在提交作业时才会像yarn申请资源启动。

上面的YARN session是在Hadoop YARN环境下启动一个Flink cluster集群，里面的资源是可以共享给其他的Flink作业

此时打开flink jobmanager web interface地址可以看到如下画面。

![](https://i.loli.net/2019/11/19/ymRBopgDqevCVsE.png)

查看resourcemanager也可以资源被分配出去了。

![](https://i.loli.net/2019/11/19/dClNqknAaSIvguO.png)





##### 运行example

上传文件到hdfs

```
~/opt/hadoop-cdh/hadoop-2.6.0-cdh5.14.2 » bin/hadoop fs -copyFromLocal LICENSE.txt hdfs:///tmp/
```



使用flink 集群计算wordcount

```
~/Downloads/flink-1.9.1 » ./bin/flink run ./examples/batch/WordCount.jar --input hdfs:///tmp/LICENSE.txt
```

![](https://i.loli.net/2019/11/19/ULWmFqvOilxHIMV.png)

可以看到作业正在执行。

一切顺利的话，可以看到在终端会显示出计算的结果：

```
(0,9)
(1,6)
(10,3)
(12,1)
(15,1)
(17,1)
(2,9)
(2004,1)
(2010,2)
(2011,2)
(2012,5)
(2013,4)
(2014,6)
(2015,7)
(2016,2)
(3,6)
(4,4)
(5,3)
(50,1)
(6,3)
(7,3)
(8,2)
(9,2)
(a,25)
(above,4)
(acceptance,1)
(accepting,3)
(act,1)
```



可以在运行作业的时候指定特定配置。如下，指定了并行度为6，就会向yarn申请更多资源（container）。但是注意，并行度越高其实不一定越快。而且如果申请不了更多资源，flink作业运行过程中也会出错。

```
bin/flink run -p 4 -c com.alibaba.alink.ALSExample  /Volumes/Samsung_T5/huzekang/opensource/Alink/examples/target/alink_examples_flink-1.10_2.11-1.1-SNAPSHOT.jar
```

![](http://image-picgo.test.upcdn.net/img/20200315124324.png)





### 2. Start a local  flink cluster

##### 启动flink

```
~/opt/flink-1.9.1 » bin/start-cluster.sh                                                                               
Starting cluster.
Starting standalonesession daemon on host huzekangdembp.
Starting taskexecutor daemon on host huzekangdembp.
```

打开`http://localhost:8081/#/job/running`即可看到flink 集群。

![](http://image-picgo.test.upcdn.net/img/20191204222452.png)

##### 跑example

使用flink 集群计算wordcount

```
 flink run ~/opt/flink-1.9.1/examples/batch/WordCount.jar --input ~/opt/flink-1.9.1/README.txt
```

可以使用`-m`参数将作业提交到远程flink 集群中执行。例如：

```shell
bin/flink run -m 192.168.5.8:8081 -c com.alibaba.alink.ALSExample  /Volumes/Samsung_T5/huzekang/opensource/Alink/examples/target/alink_examples_flink-1.10_2.11-1.1-SNAPSHOT.jar
```





## 并行度对比

这里执行了一个数据质量分析的作业，读的phoenix的表，数据量两百多w。

在作业中使用了row key对phoenix表做了切分读入（即分段读）。

### 并行度为8

两分半跑完。

![](http://image-picgo.test.upcdn.net/img/20200319144750.png)

### 并行度为6

三分钟跑完。

![](http://image-picgo.test.upcdn.net/img/20200319144343.png)

### 并行度为3

程序6分钟就执行完了。时间多数花在读。

```
flink-1.10.0/bin/flink run -p 3 -c com.yibo.flink.DataQualityJobStart flink-jobs-1.1-SNAPSHOT.jar -p 4 -dburl http://cdh05:8765 -readTableNames ODS_3_EMPLOYEES_SALARIES_20200313091118   -userId 1
```

![](http://image-picgo.test.upcdn.net/img/20200319143433.png)

### 并行度为1

同样的程序，时间却要18分钟。

![](http://image-picgo.test.upcdn.net/img/20200319143535.png)



## 集成prometheus监控

将flink目录下opt里的jar放入lib目录中。

```
~/opt/flink-1.10.0/opt » cp flink-metrics-prometheus-1.10.0.jar ../lib                     
```

配置flink，添加下面内容。

这里指定了pushgateway的地址，flink会将自己的指标推给pushgateway。

```properties
# 监控
metrics.reporter.promgateway.class: org.apache.flink.metrics.prometheus.PrometheusPushGatewayReporter
metrics.reporter.promgateway.host: localhost
metrics.reporter.promgateway.port: 9091
metrics.reporter.promgateway.jobName: flink_metrics
metrics.reporter.promgateway.randomJobNameSuffix: true
metrics.reporter.promgateway.deleteOnShutdown: false
```

启动pushgateway。

配置prometheus.yml

```
# scape配置
scrape_configs:

- job_name: 'flink'
  static_configs:
  - targets: ['192.168.1.36:9091']
```

启动prometheus。

启动grafana。在grafana中引入社区看板`11049`

![](http://image-picgo.test.upcdn.net/img/20200330155811.png)



## Flink1.10 java读写hive

编写测试代码

```java
public class HiveStart {
	public static void main(String[] args) throws Exception {
		final EnvironmentSettings settings = EnvironmentSettings.newInstance().useBlinkPlanner().inBatchMode().build();
		final TableEnvironment btableEnv = TableEnvironment.create(settings);

		final HiveCatalog hiveCatalog = new HiveCatalog(
				"myhive",
				"default",
				"/Users/huzekang/Downloads/yibo(CLUSTER)-configs/HIVE_CLIENT",
				"2.1.0");
		btableEnv.registerCatalog("myhive",hiveCatalog);
		btableEnv.useCatalog("myhive");
		// 查询
		final Table table = btableEnv.sqlQuery("select * from csv_salaries where emp_no ='1'");
		table.printSchema();
		final List<Row> rows = TableUtils.collectToList(table);
		System.out.println(rows);

		// 插入
		btableEnv.sqlUpdate("insert into csv_salaries select '2','2','2','2' ");

		btableEnv.execute("");
	}
}

```

引入依赖

```xml
     <dependency>
                    <groupId>org.apache.flink</groupId>
                    <artifactId>flink-connector-hive_2.11</artifactId>
                    <version>${flink.version}</version>
                </dependency>

                <!-- https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-mapreduce-client-core -->
                <dependency>
                    <groupId>org.apache.hadoop</groupId>
                    <artifactId>hadoop-mapreduce-client-core</artifactId>
                    <version>2.7.4</version>
                </dependency>

                <!-- Hive Dependency -->
                <!-- https://mvnrepository.com/artifact/org.apache.hive/hive-exec -->
                <dependency>
                    <groupId>org.apache.hive</groupId>
                    <artifactId>hive-exec</artifactId>
                    <version>2.1.0</version>
                </dependency>
   <dependency>
                    <groupId>org.apache.flink</groupId>
                    <artifactId>flink-table-planner-blink_${scala.binary.version}</artifactId>
                    <version>${flink.version}</version>
                </dependency>
```



## flink 配置优化

**配置flink-conf.yml文件。**

### 增加自定义jvm参数

开启远程debug。一些jvm参数也可以这样传递。

```java
env.java.opts.taskmanager: "-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005 "
env.java.opts.jobmanager: "-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5006"

```

### 调整taskmanager内存

```properties
# 调大整个taskmanager的应用内存
taskmanager.memory.process.size: 9024m
# 相对增大metaspace大小，防止加载过多class 时 ，metaspace OOM。
taskmanager.memory.jvm-metaspace.size : 1024m
```

