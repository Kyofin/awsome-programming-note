# Spark Starter

## spark必须知道的概念

一个**Job**被拆分成若干个**Stage**，每个Stage执行一些计算，产生一些中间结果。它们的目的是最终生成这个Job的计算结果。而每个**Stage**是一个task set，包含若干个**task**。Task是Spark中最小的工作单元，在一个executor上完成一个特定的事情。



## spark资料

```java
知识类:
https://github.com/JerryLead/SparkInternals 
https://github.com/JerryLead/SparkLearning 
https://github.com/databricks/spark-knowledgebase 
https://github.com/knoldus/Play-Spark-Scala
https://www.zybuluo.com/changedi/note/1413747 // Spark SQL玩起来
https://databricks.com/blog/2015/07/15/introducing-window-functions-in-spark-sql.html //窗口函数
https://github.com/marsishandsome/SparkSQL-Internal


接口类:
https://github.com/plaa/mongo-spark 
https://github.com/datastax/spark-cassandra-connector https://github.com/deanwampler/spark-workshop 
https://github.com/databricks/spark-avro 
https://github.com/massie/spark-parquet-example 
https://github.com/tmalaska/SparkOnHBase 
https://github.com/skrusche63/spark-elastic

功能类:
https://github.com/databricks/spark-perf 
https://github.com/sequenceiq/docker-spark 
https://github.com/ezhaar/spark-installer

算法类:
https://github.com/BaiGang/spark_multiboost 
https://github.com/alitouka/spark_dbscan 
https://github.com/OndraFiedler/spark-recommender
https://github.com/endymecy/spark-ml-source-analysis

应用类:
https://github.com/yaooqinn/kyuubi   //spark thrift jdbc增强版
https://github.com/spark-jobserver/spark-jobserver 
https://github.com/ooyala/spark-jobserver 
https://github.com/ibm-et/spark-kernel
https://github.com/sigmoidanalytics/spork
https://github.com/adobe-research/spindle
https://github.com/adobe-research/spindle 
https://github.com/hohonuuli/sparknotebook
https://github.com/edp963/moonbox
https://github.com/qiniu/QStreaming
https://github.com/awslabs/deequ
https://github.com/YotpoLtd/metorikku
https://github.com/yaooqinn/spark-authorizer
https://github.com/MrPowers/spark-daria // Spark helper methods to maximize developer productivity.
https://github.com/bebee4java/ides
https://github.com/harbby/sylph // Streaming Job Manager.
https://github.com/teeyog/IQL // adhoc query service based on the spark sql engine.
https://github.com/intel-analytics/analytics-zoo //A unified Data Analytics and AI platform for distributed TensorFlow, Keras and PyTorch on Apache Spark
https://github.com/cas-bigdatalab/piflow
https://github.com/apache/griffin //Data Quality
https://github.com/allwefantasy/mlsql //The Programming Language Designed For Big Data and AI
https://github.com/Qihoo360/Quicksql
https://github.com/InterestingLab/waterdrop


```





## Spark APP常用配置

可以在命令行指定，也可以在spark作业初始化sparksession时指定。

```java
			// hive metastore指定,可以读hive上的表。只有当session使用了enableHiveSupport()才生效
			.config("spark.hadoop.hive.metastore.uris","thrift://cdh04:9083")
        // 指定spark sql中创建的表的数据存到hdfs的哪个目录
        .config("spark.sql.warehouse.dir","/user/hive/warehouse")
			// yarn 资源管理指定,yarn模式才需要
			.config("spark.hadoop.yarn.resourcemanager.address","cdh04:8050")
        // 指定fs.defaultFS
        .config("spark.hadoop.fs.defaultFS","hdfs://cdh04:8020")
			//动态资源调整
      .config("spark.shuffle.service.enabled", "true")
      .config("spark.dynamicAllocation.enabled", "true")
      .config("spark.dynamicAllocation.executorIdleTimeout", "30s")
       .config("spark.dynamicAllocation.initialExecutors", "3")
      .config("spark.dynamicAllocation.maxExecutors", "100")
      .config("spark.dynamicAllocation.minExecutors", "0")
      //动态分区
      .config("hive.exec.dynamic.partition", "true")
      .config("hive.exec.dynamic.partition.mode", "nonstrict")
      .config("hive.exec.max.dynamic.partitions", 20000)
      //调度模式FAIR
      .config("spark.scheduler.mode", "FAIR")
       //调度fair模式配置文件
      .config("spark.scheduler.allocation.file", "/usr/hdp/current/spark2-thriftserver/conf/spark-thrift-fairscheduler.xml")
      .config("spark.executor.memoryOverhead", "512")
      .config("spark.serializer","org.apache.spark.serializer.KryoSerializer")
        // 允许sql中使用corss join
        .config("spark.sql.crossJoin.enabled","true")
        // 不允许开web ui
       .config("spark.ui.enabled", "false")
        // 设置rdd和shuffle数据的本地存放目录
       .config("spark.local.dir","/Users/huzekang/tmp/spark_local_dir")
         // 设置sparkUI端口占用重试次数
       .config("spark.port.maxRetries","100")



```

如果hdfs集群开启了HA，则需添加如下配置：

```JAVA

sc.hadoopConfiguration.set("fs.defaultFS", "hdfs://cdhservice")
sc.hadoopConfiguration.set("dfs.nameservices", "cdhservice")
sc.hadoopConfiguration.set("dfs.ha.namenodes.cdhservice", "namenode36,namenode105")
sc.hadoopConfiguration.set("dfs.namenode.rpc-address.cdhservice.namenode36", "cdh1:8020")
sc.hadoopConfiguration.set("dfs.namenode.rpc-address.cdhservice.namenode105", "cdh3:8020")
sc.hadoopConfiguration.set("dfs.client.failover.proxy.provider.cdhservice", "org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider")

```



## spark shell指定fair调度模式

启动命令增加下面配置。

```shell
 --conf spark.scheduler.allocation.file="/Users/huzekang/tmp/fairscheduler.xml" 
 --conf spark.scheduler.mode="FAIR"           
```

其中**fairscheduler.xml** 文件

```XML
<?xml version="1.0"?>

<allocations>
  <pool name="fair_pool">
    <schedulingMode>FAIR</schedulingMode>
    <weight>2</weight>
    <minShare>4</minShare>
  </pool>
  <pool name="a_different_pool">
    <schedulingMode>FIFO</schedulingMode>
    <weight>1</weight>
    <minShare>2</minShare>
  </pool>
</allocations>
```

在spark shell中执行代码，**运行时**指定调度池。

```scala
scala> sc.setLocalProperty("spark.scheduler.pool","fair_pool")

scala> val someRdd = sc.parallelize(1 to 100,2)
someRdd: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[0] at parallelize at <console>:24

scala> someRdd.collect
res1: Array[Int] = Array(1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 98, 99, 100)
```

其中`sc.setLocalProperty("spark.scheduler.pool","fair_pool")`指定了pool，所以后面的代码会用这个pool。

![image-20200701102335428](http://image-picgo.test.upcdn.net/img/20200701102335.png)



## Spark sql自适应调优参数

```
--conf spark.sql.adaptive.enabled=true
--conf  spark.sql.adaptive.join.enabled=true 
--conf  spark.sql.adaptive.skewedJoin.enabled=true

```

参考[Adaptive Execution 让 Spark SQL 更高效更智能](http://www.jasongj.com/spark/adaptive_execution/)

### 自动分配资源

  spark.dynamicAllocation.enabled  =true

### 自动设置 Shuffle Partition 

- 可通过 `spark.sql.adaptive.enabled=true` 启用 Adaptive Execution 从而启用自动设置 Shuffle Reducer 这一特性。

- 通过 `spark.sql.adaptive.shuffle.targetPostShuffleInputSize` 可设置每个 Reducer 读取的目标数据量，其单位是字节，默认值为 64 MB。

### 自动调整join优化

- 当 `spark.sql.adaptive.enabled` 与 `spark.sql.adaptive.join.enabled` 都设置为 `true` 时，开启 Adaptive Execution 的动态调整 Join 功能。

- `spark.sql.adaptiveBroadcastJoinThreshold` 设置了 SortMergeJoin 转 BroadcastJoin 的阈值。如果不设置该参数，该阈值与 `spark.sql.autoBroadcastJoinThreshold` 的值相等

### 自动数据倾斜优化

- 将 `spark.sql.adaptive.skewedJoin.enabled` 设置为 true 即可自动处理 Join 时数据倾斜
- `spark.sql.adaptive.skewedPartitionMaxSplits` 控制处理一个倾斜 Partition 的 Task 个数上限，默认值为 5
- `spark.sql.adaptive.skewedPartitionRowCountThreshold` 设置了一个 Partition 被视为倾斜 Partition 的行数下限，也即行数低于该值的 Partition 不会被当作倾斜 Partition 处理。其默认值为 10L * 1000 * 1000 即一千万
- `spark.sql.adaptive.skewedPartitionSizeThreshold` 设置了一个 Partition 被视为倾斜 Partition 的大小下限，也即大小小于该值的 Partition 不会被视作倾斜 Partition。其默认值为 64 * 1024 * 1024 也即 64MB
- `spark.sql.adaptive.skewedPartitionFactor` 该参数设置了倾斜因子。如果一个 Partition 的大小大于 `spark.sql.adaptive.skewedPartitionSizeThreshold` 的同时大于各 Partition 大小中位数与该因子的乘积，或者行数大于 `spark.sql.adaptive.skewedPartitionRowCountThreshold` 的同时大于各 Partition 行数中位数与该因子的乘积，则它会被视为倾斜的 Partition





## spark streaming日志过多解决

需要先建log4j.properties文件。

```
log4j.rootCategory=ERROR, console
log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.target=System.err
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=%d{yy/MM/dd HH:mm:ss} %p %c{1}: %m%n
```

然后在spark-sumbit提交作业时设置日志级别

```
spark-submit 
--conf "spark.driver.extraJavaOptions=-Dlog4j.configuration=file:log4j.properties"
```



## spark-shell显示INFO日志

修改`conf/log4j.properties`文件。将repl的默认日志级别WARN修改成INFO。

```
log4j.logger.org.apache.spark.repl.Main=INFO
```






## spark命令启动参数作用对比

这里以spark-sql命令为例。

```
spark-sql --driver-cores    5   \
        --driver-memory   5G  \
        --executor-memory 5g \
        --num-executors 8 \
        --total-executor-cores 8\
        --conf spark.serializer=org.apache.spark.serializer.KryoSerializer \
        --conf spark.kryoserializer.buffer.max.mb=1024 \
        --conf spark.storage.memoryFraction=0.2
```

资源使用的是yarn的资源。这里的大数据环境是在hdp环境。

![](http://image-picgo.test.upcdn.net/img/20200429141527.png)

测试的数据集是` tpcds_bin_partitioned_orc_200`。200g的数据库。

使用的是hive benchmark中的一条语句。

```sql
select  
  cd_gender,
  cd_marital_status,
  cd_education_status,
  count(*) cnt1,
  cd_purchase_estimate,
  count(*) cnt2,
  cd_credit_rating,
  count(*) cnt3,
  cd_dep_count,
  count(*) cnt4,
  cd_dep_employed_count,
  count(*) cnt5,
  cd_dep_college_count,
  count(*) cnt6
 from
  customer c,customer_address ca,customer_demographics
 where
  c.c_current_addr_sk = ca.ca_address_sk and
  ca_county in ('Walker County','Richland County','Gaines County','Douglas County','Dona Ana County') and
  cd_demo_sk = c.c_current_cdemo_sk and 
  exists (select *
          from store_sales,date_dim
          where c.c_customer_sk = ss_customer_sk and
                ss_sold_date_sk = d_date_sk and
                d_year = 2002 and
                d_moy between 4 and 4+3) and
   (exists (select *
            from web_sales,date_dim
            where c.c_customer_sk = ws_bill_customer_sk and
                  ws_sold_date_sk = d_date_sk and
                  d_year = 2002 and
                  d_moy between 4 ANd 4+3) or 
    exists (select * 
            from catalog_sales,date_dim
            where c.c_customer_sk = cs_ship_customer_sk and
                  cs_sold_date_sk = d_date_sk and
                  d_year = 2002 and
                  d_moy between 4 and 4+3))
 group by cd_gender,
          cd_marital_status,
          cd_education_status,
          cd_purchase_estimate,
          cd_credit_rating,
          cd_dep_count,
          cd_dep_employed_count,
          cd_dep_college_count
 order by cd_gender,
          cd_marital_status,
          cd_education_status,
          cd_purchase_estimate,
          cd_credit_rating,
          cd_dep_count,
          cd_dep_employed_count,
          cd_dep_college_count
limit 100;
```

128s计算出结果

![](http://image-picgo.test.upcdn.net/img/20200429141444.png)

当调小了启动参数

```
spark-sql --driver-cores    2   \
        --driver-memory   4G  \
        --executor-memory 2g \
        --num-executors 2 \
        --total-executor-cores 6\
        --conf spark.serializer=org.apache.spark.serializer.KryoSerializer \
        --conf spark.kryoserializer.buffer.max.mb=1024 \
        --conf spark.storage.memoryFraction=0.2
```

yarn分配的资源就少了。

![](http://image-picgo.test.upcdn.net/img/20200429142607.png)

执行相同sql，计算的时间也长了，需要382s。

![](http://image-picgo.test.upcdn.net/img/20200429142628.png)



## 本地idea运行spark app

A master URL must be set in your configuration

Run->Edit Configurations
![](http://image-picgo.test.upcdn.net/img/20200430092835.png)

```
-Dspark.master=local
```



## idea中运行SparkSQLCLIDriver

1. 成功完成spark编译压缩部署包（按照印象笔记中笔记方式）

2. 修改spark的pom文件

   ![image-20200922093516261](http://image-picgo.test.upcdn.net/img/20200922093516.png)

3. 修改profile

   ![image-20200922093645306](http://image-picgo.test.upcdn.net/img/20200922093645.png)

4. 刷新下pom文件，然后rebuild项目

5. 配置启动类

   ![image-20200922093741352](http://image-picgo.test.upcdn.net/img/20200922093741.png)

6. 放入hive-site和core-site到resource目录

   ![image-20200922093835652](http://image-picgo.test.upcdn.net/img/20200922093835.png)

7. 测试

   ![image-20200922093913858](http://image-picgo.test.upcdn.net/img/20200922093913.png)



## spark on yarn的动态资源调度

1. 将spark目录下的yarn文件中的`spark-<version>-yarn-shuffle.jar`复制到`hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/lib`目录下，让nodeManager可以加载到。

2. 配置hadoop目录下的yarn-site.xml文件,加入以下配置

   ```xml
   	
     <property>
           <name>yarn.nodemanager.aux-services</name>
           <value>mapreduce_shuffle,spark_shuffle</value>
       </property>
   
        <property>
           <name>yarn.nodemanager.aux-services.spark_shuffle.class</name>
           <value>org.apache.spark.network.yarn.YarnShuffleService</value>
       </property>
   ```

   **注意**：aux-services中追加spark_shuffle即可，不要把前面的mapreduce_shuffle删了，删了会导致mr程序无法执行shuffle操作。

3. Increase `NodeManager's` heap size by setting `YARN_HEAPSIZE` (1000 by default) in `etc/hadoop/yarn-env.sh` to avoid garbage collection issues during shuffle.(spark官方推荐的)

4. 启动spark-sql时加上动态资源的配置

   ```
    spark-sql --master yarn --conf spark.dynamicAllocation.enabled=true --conf spark.shuffle.service.enabled=true
   ```

   




## spark app远程debug

这里使用spark自带的example `org.apache.spark.examples.JavaSparkPi`断点测试。

该方法好像只能断点driver的代码，**不能断点rdd**的代码。

即图中**红色部分**才可以。

![image-20200526163627365](http://image-picgo.test.upcdn.net/img/20200526163627.png)



使用方法如下。

1. 配置idea

   ![](http://image-picgo.test.upcdn.net/img/20200526163004.png)

2. spark启动命令增加参数

   `--conf spark.driver.extraJavaOptions=-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=5005 `

   完整命令如下：

   本质是使用sparksubmit时带上上面的java参数。

   ```shell
   [root@cdh06 spark2]#  bin/run-example --conf spark.driver.extraJavaOptions=-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=5005 org.apache.spark.examples.JavaSparkPi 100
   ```

3. 断点测试

   在看到命令行输出`Listening for transport dt_socket at address: 5005`后，在idea中启动远程断点。

   可以看到断点进入循环体内了。

   ![image-20200526163333716](http://image-picgo.test.upcdn.net/img/20200526163333.png)





## spark shell远程debug

1. sparkshell启动前输入环境变量

   ```shell
   [root@cdh06 spark2]# export SPARK_SUBMIT_OPTS=-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=5005
   ```

2. 参考app断点设置idea

3. 测试

   ![image-20200526165337439](http://image-picgo.test.upcdn.net/img/20200526165337.png)

4. 演示debug本地spark-shell在启动时怎么根据master去创建TaskScheduler的。

   启动命令。这里使用代码打包完成的二进制包中的spark-shell。

   ```shell
   /Volumes/Samsung_T5/huzekang/opensource/hdp-3.1.0.0-78/spark2-release-HDP-3.1.0.0-78/dist(master*) » spark-shell --master local[2]                       
   
   ```

   启动后在idea中打开断点监听可以看到如下：

   ![image-20200529123747475](http://image-picgo.test.upcdn.net/img/20200529123747.png)

   可以看到spark会根据`--master local[2]         `启动2个线程来执行任务。

   

## spark编译二进制部署包（HDP版本）

zinc 可以增量编译scala，因为编译scala最慢的就是他自己的library了。

### mac中zinc启动、关闭。

```shell
/usr/local/Cellar/maven/3.6.2 » zinc -shutdown                                                                                                           
/usr/local/Cellar/maven/3.6.2 » zinc -start     
```

### 执行编译

```shell
./dev/make-distribution.sh --name hzk-spark   --tgz  -Phadoop-2.7 -Phive -Phive-thriftserver -Pyarn -DskipTests
```

成功后，可以看到如下文件夹。

![image-20200527155438930](http://image-picgo.test.upcdn.net/img/20200527155439.png)

可以看到压缩包。

![image-20200527170015880](/Users/huzekang/Library/Application Support/typora-user-images/image-20200527170015880.png)



### 配置好环境变量指向该目录

![image-20200527155926459](http://image-picgo.test.upcdn.net/img/20200527155926.png)

### 测试编译好的spark-shell

会报如下错误。

![image-20200527155555462](http://image-picgo.test.upcdn.net/img/20200527155555.png)

此时修改该文件。

![image-20200527155635702](http://image-picgo.test.upcdn.net/img/20200527155635.png)

再启动则可以使用local模式了。虽然会报错找不到hadoop的lib。

![image-20200527155751809](http://image-picgo.test.upcdn.net/img/20200527155752.png)



## spark源码编译

命令行设置代理翻墙

```
export http_proxy=10.0.0.17:1087
```

使用spark项目内置脚本执行编译

```
~/openSource/spark(02b510728c) » ./build/mvn -DskipTests clean package      
```

编译成功后使用idea打开，并将pom.xml移入maven栏目，项目会自动加载。

但如果直接启动example模块的例子还是会报错。

![](http://image-picgo.test.upcdn.net/img/20200115140027.png)

是因为pom.xml的spark依赖都是provided范围。需要修改idea的启动选项。

![](http://image-picgo.test.upcdn.net/img/20200115140042.png)

并指定master，就可以正常运行了。

![](http://image-picgo.test.upcdn.net/img/20200115140258.png)





## Spark submit 不同的模式运行

### 1. 运行在 yarn集群上

系统环境变量需要有yarn的配置。

![](https://i.loli.net/2019/11/20/bLxrwyP3DJ6QglY.png)

因为spark提交作业时会找yarn的配置去获取资源。

- Spark on YARN 集群上 yarn-cluster 模式运行

  ```SHELL
  ./bin/spark-submit --class org.apache.spark.examples.SparkPi \
      --master yarn \
      --deploy-mode cluster \
      --driver-memory 2g \
      --executor-memory 2g \
      --executor-cores 1 \
      examples/jars/spark-examples*.jar \
      10
  ```
  

  
- spark on yarn 集群上 yarn-client模式运行

  ```SHELL
  ./bin/spark-submit --class org.apache.spark.examples.SparkPi \
      --master yarn \
      --deploy-mode client \
      --driver-memory 2g \
      --executor-memory 2g \
      --executor-cores 1 \
      examples/jars/spark-examples*.jar \
      10
  ```



注意 Spark on YARN 支持两种运行模式，分别为`yarn-cluster`和`yarn-client`，具体的区别可以看[这篇博文](http://www.iteblog.com/archives/1223)，从广义上讲，yarn-cluster适用于生产环境；而yarn-client适用于交互和调试，也就是希望快速地看到application的输出。



## spark sql的元数据使用pg替代derby数据库

在spark目录的conf文件夹中，创建hive-site.xml

```XML
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
<property>
  <name>javax.jdo.option.ConnectionURL</name>
  <value>jdbc:postgresql://192.168.5.18:5432/demo_hive_spark</value>
  <description>the URL of the MySQL database</description>
</property>

<property>
  <name>javax.jdo.option.ConnectionDriverName</name>
  <value>org.postgresql.Driver</value>
</property>

<property>
  <name>javax.jdo.option.ConnectionUserName</name>
  <value>postgres</value>
</property>

<property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>izA8v8gFIXqsXXwa4BZWahrW6dtQ21T8</value>
    </property>
<!-- 数据存储目录 -->

    <property>
  <name>hive.metastore.warehouse.dir</name>
  <value>/user/spark/warehouse</value>
</property>
</configuration>
```

 启动spark-sql后，创建的表的元信息都会存放到pg中



## spark sql 语法文档

参考：[SQL reference](https://docs.databricks.com/spark/latest/spark-sql/language-manual/index.html)

```SQL
-- 分析表信息
ANALYZE TABLE HIVE_ODS_22_TPCDS_200_TIME_DIM_20200611101934 COMPUTE STATISTICS

-- 查询表信息分析结果
DESCRIBE EXTENDED HIVE_ODS_22_TPCDS_200_TIME_DIM_20200611101934
DESCRIBE  HIVE_ODS_22_TPCDS_200_TIME_DIM_20200611101934
DESCRIBE Formatted  HIVE_ODS_22_TPCDS_200_TIME_DIM_20200611101934



-- 分析表字段
ANALYZE TABLE HIVE_ODS_22_TPCDS_200_TIME_DIM_20200611101934 COMPUTE STATISTICS FOR COLUMNS t_time_sk ,  t_time_id, t_time

-- 查询表字段分析结果
DESCRIBE  EXTENDED HIVE_ODS_22_TPCDS_200_TIME_DIM_20200611101934  t_time_sk 
DESCRIBE  EXTENDED  HIVE_ODS_22_TPCDS_200_TIME_DIM_20200611101934  t_time_id
DESCRIBE  EXTENDED  HIVE_ODS_22_TPCDS_200_TIME_DIM_20200611101934  t_time

-- 缓存表
cache TABLE HIVE_ODS_22_TPCDS_200_TIME_DIM_20200611101934

-- 清除缓存
UNCACHE TABLE HIVE_ODS_22_TPCDS_200_TIME_DIM_20200611101934

-- 查询建表语句
SHOW CREATE TABLE HIVE_ODS_22_TPCDS_200_TIME_DIM_20200611101934
```



## Spark history  server

用于记录启动过的spark app。

需要配置`spark-defaults.conf`文件

不能指定hdfs地址，只能使用本地文件夹。

```properties
spark.eventLog.enabled=true
spark.eventLog.dir=file:///Users/huzekang/tmp/spark_history
spark.history.fs.logDirectory=file:///Users/huzekang/tmp/spark_history
```

然后启动`sbin/start-history-server.sh`

![image-20200711105324764](http://image-picgo.test.upcdn.net/img/20200711105324.png)

作业日志会存放到文件中。

![image-20200711105332628](http://image-picgo.test.upcdn.net/img/20200711105332.png)

## Spark sql 连接es

es官网下载hadoop依赖

![image-20200605142538866](http://image-picgo.test.upcdn.net/img/20200605142539.png)

```shell
 spark-sql --jars ~/Downloads/elasticsearch-hadoop-7.5.0/dist/elasticsearch-spark-20_2.11-7.5.0.jar  --driver-class-path ~/Downloads/elasticsearch-hadoop-7.5.0/dist/elasticsearch-spark-20_2.11-7.5.0.jar
```

启动spark-sql终端后，创建视图。

```SQL
create temporary table metadata
using org.elasticsearch.spark.sql
options('resource'='metadata/jdbc', 
  'nodes'= '192.168.1.130',
  'es.nodes.wan.only'='true',
  'es.port'='9200');
```

![image-20200605142621726](http://image-picgo.test.upcdn.net/img/20200605142621.png)



## sparkSql中连接自定义sql外部源

使用sparksql通常都是建立连接时指定特定表，如下

```scala
sqlContext.read.format("jdbc").options(Map("url"->"jdbc:mysql://10.10.190.102:3306/db",
            "driver" -> "com.mysql.jdbc.Driver",
            "dbtable"->"temp",
            "user"->"root","password"->"root")).load()

```

但是我们也可以在建立该连接时指定自定义的sql作为一个表。如：

```scala
sqlContext.read.format("jdbc").option("driver","com.cloudera.impala.jdbc4.Driver").option("url","jdbc:impala://cdh02:21050/honghui_sqlserver").option("dbtable","(select  a.patid as patient_id,    b.xxdm as abo_code,    c.ysdm2 as assistant_ii_staff_name,    c.ysdm1 as assistant_i_staff_name,    b.bmy as coder_signature,    d.bah as medical_record_number,    b.bazl as med_record_code,    b.blbh as pathology_number,    b.blzd as pathologic_diagnosis_name,    d.csd_s as birth_place_province,    d.csd_x as birth_place_county,    d.birth as birth_date,    b.cych as discharge_room,    b.cyks as discharge_departmen,    a.cyrq as discharge_date,    e.zddm as dis_main_diagnosis_code,    e.zdmc as dis_main_diagnosis_name,    d.lxdh as telephone_number,    d.dwyb as work_unit_addr_postal_code,    d.dwdh as work_company_tel,    d.dwmc as work_company_name,    d.gjbm as nationality_code,    b.hkdz as registered_house_number,    b.hkyb as registered_postal_code,    a.sfzh as id_number,    d.hyzk as marital_status_code,    b.jgdm as native_place_province,    a.ysdm as training_doctor_signature,    f.cardno as health_card_number,    a.cyfs as leave_way_code,    b.lxdz as linkman_addr_house_number,    b.lxdh as linkman_tel,    b.lxrm as linkman_name,    b.lxgx as linkman_patient_relationship_code,    g.mzdm as anesth_type_code,    h.zddm as diagnostic_disease_code,    h.zdmc as diagnostic_name,    d.mzbm as nation,    b.brnl as age,    b.rych as admiss_room,    b.ryks as admission_department,    b.ryrq as into_date,    b.rytj as adm_way_code,    i.zyts as actual_hospital_stay,    g.ssdm as surgery_code,    g.ssmc as surgery_operation_name,    g.kssj as surgery_operation_target_name,    g.qkdj as op_incision_grade_code,    g.ysdm as op_doctor_name,    b.xyf as western_medicine_fee,    d.lxdz as residence_door,    d.lxyb as residence_post,    j.tz as newborn_weight,    k.tz as newborn_admiss_weight,    a.sex as gender_code,    a.hzxm as name,    b.xxdm as blood_type_code,    b.sxf as blood_fee,    d.zybm as occupational_category_code,    b.zkhs as qc_nurse_signs,    b.barq as qc_date,    b.zkys as quality_control_doctor_signature,    b.qtf4 + b.ssf as surgical_treatment_costs,    b.qtf4 as surgical_treatment_narcotize_costs,    b.ssf as surgical_treatment_operating_costs,    b.zyf as chinese_herbal_medicine_fee,    b.zyf as chinese_patent_medicine_fee,    b.zrys as chief_doctor_signature,    b.zzys as attending_doctor_signature,    a.zycs as hospitalization_times,    a.syxh as admisson_no,    m.ysdm1 as hospitalization_doctor_signature,    l.zje as total_hospitalization_fee,    l.zfje as total_hospitalization_self_fee,    b.zkks as transfer_branch,    b.hlf as self_care_code,    a.lrrq as createdate    from zy_brsyk as a     left join bq_ys_syjb as b on a.syxh = b.syxh    left join bq_ys_bassk as c on a.syxh = c.syxh    left join zy_brxxk as d on a.patid = d.patid    left join zy_brzdqk as e on a.syxh = e.syxh and e.zdlb = 2    left join sf_brcard as f on a.patid = f.patid    left join ss_ssdjk as g on a.syxh = g.syxh    left join zy_brzdqk as h on a.syxh = h.syxh and h.zdlb = 1    left join zy_brjsk as i on a.syxh = i.syxh    left join zy_babysyk as j on a.syxh = j.syxh    left join bq_yetzjlk as k on a.syxh = k.syxh    left join zy_brfymxk2 as l on a.syxh = l.syxh    left join bq_ys_cyxj as m on a.syxh = m.syxh) as aa").load
```



## spark-shell连接phoenix

- 需要用到的jar到服务器上
  下载地址：https://pan.baidu.com/s/1bpGtcficrxJVd_RxDbfTsw
  ![](http://183.6.50.10:4999/server/../Public/Uploads/2019-11-20/5dd49b3d1a745.png)

```shell
scp spark-extra-lib/phoenix-*.jar root@agent1:/root/spark-extra-libs
```

- 启动spark shell

  在spark-shell 启动时指定加载的 lib，解决在 spark-shell中使用 phoenix 的问题

```shell
spark-shell --driver-library-path=/root/spark-extra-libs/* --driver-class-path=/root/spark-extra-libs/*
```

- 测试连接 phoenix

首先输入 `:paste`

```
val df = spark.read 
      .format("jdbc") 
      .option("driver", "org.apache.phoenix.jdbc.PhoenixDriver") 
      .option("url", "jdbc:phoenix:cdh01:2181")
      .option("dbtable", "gzhonghui_new.patient_basic_information") 
      .option("phoenix.schema.isNamespaceMappingEnabled", "true") 
      .load() 
```

Ctrl +D 执行

```
df.show(1,20,true)
```

![](https://i.loli.net/2019/11/20/GYyEVTj8I7b4gS6.png)



## Spark-sql工具操作hive

在conf目录下放入hive-stie.xml后使用`bin/spark-sql`即可。

可以在启动时加入master即可指定spark standalone资源(非必须)。

![](http://image-picgo.test.upcdn.net/img/20191223092548.png)



### **创建表**

#### **1，以文本方式存储**

```
create external table mytest1(id bigint, name string)  row format delimited fields terminated by ','  location 'hdfs://bigserver1:9000/test/spark/tank3';  
```

**这种方式创建的表，是以文本的形式存储的**

#### **2，以parquet存储**

```
CREATE TABLE mytest3 (id bigint, name string)  USING HIVE OPTIONS(fileFormat 'PARQUET')  location 'hdfs://cdh04:8020/test/spark/tank4';  
```

**这种创建表的方式，指定了文件存储方式，在用scala读取时会方便一些。**但要注意有写入该文件夹**权限**。

**在这里要注意一点，如果没有指定location的话，默认会装到SPARK thrift的warehouse**。

写入数据（这样写很慢，建议用hdfs写入再查）

```
INSERT INTO mytest3 VALUES (1,"zhang"), (2,"tank")  
```



## 内置函数

共267个

![](http://image-picgo.test.upcdn.net/img/20200110185755.png)

```
--查看所有内置函数
SHOW FUNCTIONS ;
--查看某个函数的描述
DESCRIBE FUNCTION !;
 --查看某个函数的具体使用方法
DESCRIBE FUNCTION EXTENDED !;
```





## 开启thriftserver启动spark-sql远程服务

如果端口指定下面的没用，可以使用环境变量

```
export HIVE_SERVER2_THRIFT_PORT=10014
```



### 启动

- 使用yarn启动

  设置环境变量`export HADOOP_CONF_DIR=/etc/hadoop/conf`。

  ```shell
  ./sbin/start-thriftserver.sh \
  				--hiveconf hive.server2.thrift.port=14000 \
          --hiveconf hive.exec.mode.local.auto=true  \
          --hiveconf hive.auto.convert.join=true     \
          --hiveconf hive.mapjoin.smalltable.filesize=50000000 \
          --name thriftserver    \
          --master yarn-client \
          --driver-cores    5   \
          --driver-memory   5G  \
          --executor-memory 5g \
          --total-executor-cores 10\
          --conf spark.serializer=org.apache.spark.serializer.KryoSerializer \
          --conf spark.kryoserializer.buffer.max.mb=1024 \
          --conf spark.storage.memoryFraction=0.2
  ```

  启动端口是14000，即可用jdbc连接。

  ![](http://image-picgo.test.upcdn.net/img/20191223100247.png)

- 使用spark-alone模式启动

  ```
  ./sbin/start-thriftserver.sh \
  				--hiveconf hive.server2.thrift.port=14000 \
          --hiveconf hive.exec.mode.local.auto=true  \
          --hiveconf hive.auto.convert.join=true     \
          --hiveconf hive.mapjoin.smalltable.filesize=50000000 \
          --name thriftserver    \
          --master spark://cdh01:7077 \
          --driver-cores    5   \
          --driver-memory   5G  \
          --executor-memory 5g \
          --total-executor-cores 10\
          --conf spark.serializer=org.apache.spark.serializer.KryoSerializer \
          --conf spark.kryoserializer.buffer.max.mb=1024 \
          --conf spark.storage.memoryFraction=0.2
  ```

  ![](http://image-picgo.test.upcdn.net/img/20191223194248.png)

- 使用local模式启动

  ```
  ./sbin/start-thriftserver.sh --hiveconf hive.server2.thrift.port=14000
  ```

### 连接使用

启动后可以看打印的日志查看thrift server的jdbc连接端口。

![](http://image-picgo.test.upcdn.net/img/20200412162130.png)

```
./bin/beeline
Beeline version 1.2.1.spark2 by Apache Hive
beeline> !connect jdbc:hive2://localhost:14000
```

![](http://image-picgo.test.upcdn.net/img/20191223094140.png)

### 关闭服务

```
[root@cdh01 spark-2.4.4-bin-hadoop2.6]# sbin/stop-thriftserver.sh 
stopping org.apache.spark.sql.hive.thriftserver.HiveThriftServer2
```



### troubleshooting

使用beeline插入数据时报错。

![](http://image-picgo.test.upcdn.net/img/20191223132409.png)

#### 解决方法：

在使用beeline连接时输入用户名和密码。

此处的root用户是linux的用户，需先同步到hdfs才可以使用，即`hdfs dfsadmin -refreshUserToGroupsMappings`

![](http://image-picgo.test.upcdn.net/img/20191223132546.png)



## 使用spark shell读写sqlserver数据

启动shell带上sqlserver驱动。

```shell
 spark-shell --master yarn  --jars sqljdbc4-4.0.jar --driver-class-path sqljdbc4-4.0.jar
```

读sqlserver数据，写出到sqlserver表中，不设置`mode`默认是建一张新表，所以已存在同名表写出时会报错。

其中`List("FPatientInfoId >1  ").toArray`就是读取数据的分区条件。

```scala
scala> import java.util.Properties
import java.util.Properties

scala> val pros = new Properties
pros: java.util.Properties = {}

scala> pros.put("user","sa")
res1: Object = null

scala> pros.put("password","yibosa@142")
res2: Object = null

scala> pros.put("databaseName","medicare_ShiFuErYiLiaoZhongXin")
res3: Object = null

scala> val predicateDs = spark.read.jdbc("jdbc:sqlserver://192.168.1.142:1433","TPatientInfo",List("FPatientInfoId >1  ").toArray,pros)
predicateDs: org.apache.spark.sql.DataFrame = [FPatientInfoId: bigint, FAccountDate: timestamp ... 51 more fields]

scala> predicateDs.show

scala> predicateDs.createOrReplaceTempView("TPatientInfo")

scala> 
```



## 使用sparkshell读mysql写clickhouse

启动命令：

```shell
spark-shell --name "to_ck_scene_model"   --packages ru.yandex.clickhouse:clickhouse-jdbc:0.2 --jars mysql-connector-java-8.0.11.jar
```

执行脚本：

```scala
import java.sql.DriverManager

val ckTableN = "salaries"
val partitionField ="from_date"
val orderFieldAndDefauV = "emp_no"
val tableName="salaries"
val mysqlUser="root"
val mysqlPwd="root"
val mysqlDriver="com.mysql.jdbc.Driver"
val ckUrl ="jdbc:clickhouse://192.168.1.238:8123"
val ckDriver="ru.yandex.clickhouse.ClickHouseDriver"

// 读mysql数据作为df
val pros = new java.util.Properties
pros.setProperty("driver",mysqlDriver)
pros.setProperty("user", mysqlUser)
pros.setProperty("password", mysqlPwd)
pros.setProperty("driver",mysqlDriver)
val df = spark.read.jdbc("jdbc:mysql://192.168.1.130:3306/employees","salaries",pros)
// 预览并缓存mysql数据
df.cache.show

//通过mysql桥接 ，通过查询语句在clickhouse中创建表 操作后两边数据结构会一致。也可以手动在ck侧建好表。
val connection = DriverManager.getConnection(ckUrl,"default","123")
var pst=connection.createStatement()
pst.execute("drop table if exists "+ckTableN)
pst.execute("create table "+ckTableN+" ENGINE = MergeTree order by "+orderFieldAndDefauV+" as  SELECT * FROM mysql('192.168.1.130:3306', 'employees', "+tableName+", '"+mysqlUser+"', '"+mysqlPwd+"')")

// spark写出数据到ck
var pro = new java.util.Properties
pro.put("driver",ckDriver)
pro.setProperty("user", "default")
pro.setProperty("password", "123")
// 默认写入批次是2w，可以调大至5w
df.write.mode("append").option("socket_timeout","300000").option("rewriteBatchedStatements", "true").option("batchsize", "50000").option("isolationLevel", "NONE").option("numPartitions", "1").jdbc(ckUrl,ckTableN,pro)


```

`"socket_timeout": "300000"`是为了解决`read time out`异常的问题，当某个分区的数据量较大时会出现这个问题，单位为毫秒。

`rewriteBatchedStatements`，`batchsize`, `numPartitions`解决`DB::Exception: Merges are processing signiﬁcantly slower than inserts`问题，原因是批次写入量少，并发多。



## thriftserver启动http连接模式

配置spark配置目录下的hive-site.xml文件

![image-20200605155238901](http://image-picgo.test.upcdn.net/img/20200605155239.png)

正常启动thriftserver后使用beeline去连接。注意端口不再是10000而是10001。

```shell
/Volumes/Samsung_T5/huzekang/opensource/hdp-3.1.0.0-78/spark2-release-HDP-3.1.0.0-78/dist(master*) » bin/beeline                                         
Beeline version 1.21.2.3.1.0.0-78 by Apache Hive
beeline> !connect jdbc:hive2://localhost:10001/default;transportMode=http;httpPath=cliservice

```







## 使用thrifserver查询mysql数据

### 参考文档

https://ieevee.com/tech/2016/07/22/spark-mysql.html

### 示例

启动时使用`—-jars`让executor带上mysql依赖。

使用`--driver-class-path`让driver带上mysql依赖。

只使用–driver-class-path还不够，这个参数只会给driver指定jar包，但是Spark JDBC访问Mysql是发生在各个Excutor上的，还需要–jars为各个Excutor指定jar包，否则会报“no suitable driver found”。

```shell
~/opt/spark-2.4.4-bin-hadoop2.6 » ./sbin/start-thriftserver.sh --jars mysql-connector-java-8.0.11.jar --driver-class-path mysql-connector-java-8.0.11.jar

starting org.apache.spark.sql.hive.thriftserver.HiveThriftServer2, logging to /Users/huzekang/opt/spark-2.4.4-bin-hadoop2.6/logs/spark-huzekang-org.apache.spark.sql.hive.thriftserver.HiveThriftServer2-1-huzekangdembp.out
```

使用beeline连接。

![](http://image-picgo.test.upcdn.net/img/20200412162922.png)

#### **单分区表**

创建spark侧表关联mysql侧**单分区表**表。

```sql
0: jdbc:hive2://localhost:10003> CREATE TEMPORARY TABLE dept USING org.apache.spark.sql.jdbc OPTIONS (driver "com.mysql.jdbc.Driver" ,url "jdbc:mysql://localhost/test?user=root&password=eWJmP7yvpccHCtmVb61Gxl2XLzIrRgmT", dbtable "dept");
```

注册完即可使用spark sql查询了。

```
0: jdbc:hive2://localhost:10003> select * from dept;
```

![](http://image-picgo.test.upcdn.net/img/20200412163055.png)

由于指定了表只有**1个分区**，所以`select count`的时候，只会起1个task。

插入数据。

```scala
0: jdbc:hive2://localhost:10003> insert into dept select t.* from (select 5,5) t;
+---------+--+
| Result  |
+---------+--+
+---------+--+
No rows selected (0.552 seconds)
0: jdbc:hive2://localhost:10003> select * from dept;
+-----+------------+--+
| id  | dept_name  |
+-----+------------+--+
| 22  | ddit调度     |
| 3   | new_bin2   |
| 5   | 5          |
+-----+------------+--+
3 rows selected (0.136 seconds)
```

将MySQL的数据导入到本地hive表`xxx`中

```
0: jdbc:hive2://localhost:10003> create table xxx as select * from dept;
+---------+--+
| Result  |
+---------+--+
+---------+--+
No rows selected (3.783 seconds)
0: jdbc:hive2://localhost:10003> select * from xxx;
+-----+------------+--+
| id  | dept_name  |
+-----+------------+--+
| 22  | ddit调度     |
| 3   | new_bin2   |
| 5   | 5          |
+-----+------------+--+
3 rows selected (0.28 seconds)
```

#### **指定Partition**

```sql
CREATE TEMPORARY TABLE jdbc_copy USING org.apache.spark.sql.jdbc OPTIONS (url "jdbc:mysql://localhost/test?user=root&password=eWJmP7yvpccHCtmVb61Gxl2XLzIrRgmT", dbtable "dept", partitionColumn "id", lowerBound "3", upperBound "22", numPartitions "6");
```

需要指定partitionColumn, lowerBound, upperBound, numPartitions，缺一不可。指定多个partition后，select count查询会起partition个TASK并发处理。从代码上来看，指定partition会在组织select语句的时候，带上whereClause，具体可以看JDBCRelation.columnPartition，但从实际体验上来看，貌似还是每个container全量抽取到内存了。



## spark ui界面分析

部署spark alone

```
cdh01 (主节点)
cdh02 (slave)
cdh03 (slave)
cdh04 (slave)
cdh05 (slave)
```

在cdh01启动spark alone模式集群。

启动集群后，通过启动thriftserver的应用来观察UI。

```
./sbin/start-thriftserver.sh \
				--hiveconf hive.server2.thrift.port=14000 \
        --hiveconf hive.exec.mode.local.auto=true  \
        --hiveconf hive.auto.convert.join=true     \
        --hiveconf hive.mapjoin.smalltable.filesize=50000000 \
        --name thriftserver    \
        --master spark://cdh01:7077 \
        --driver-cores    5   \
        --driver-memory   5G  \
        --executor-memory 5g \
        --total-executor-cores 10\
        --conf spark.serializer=org.apache.spark.serializer.KryoSerializer \
        --conf spark.kryoserializer.buffer.max.mb=1024 \
        --conf spark.storage.memoryFraction=0.2
```

上面指定了每个executor内存为5g，总共核数10。

![](http://image-picgo.test.upcdn.net/img/20191223202259.png)

打开spark ui。可以发现drvier指定的内存和核数在spark首页都是看不到的。它统计的都是worker工作节点的占用资源，即非cdh01的节点。



测试一个很耗时的操作，读hive上的一张大数目为1000000010的表并写入到新表中。

```
create table hdfs2_id as select *,id as id2 from hdfswriter2;
```

![](http://image-picgo.test.upcdn.net/img/20191223203035.png)



再进入该sql 的作业页面。

![](http://image-picgo.test.upcdn.net/img/20191223203123.png)



可以看到该作业用了38分钟。只有一个stage4，而且stage4耗时最长。

![](http://image-picgo.test.upcdn.net/img/20191223203515.png)



点击进入stage4的监控页面。

可以看到每个工作节点都读了不同数目的数据。

并且task任务的总数和`--total-executor-cores 10`是相等的。即使这里有四个节点，但是只有0和1工作节点分到3个任务，而2和3都只分到2个。因此，我们能调整这个参数来挺高并发数。

![](http://image-picgo.test.upcdn.net/img/20191223204009.png)



## spark编程常用例子

### 初始测试数据

#### scala

这里定义了一个中包含多个元组的list作为df

```scala
scala> val  df =  List((1,"Peter"),(2,"kkk")).toDF("ID","NAME")
df: org.apache.spark.sql.DataFrame = [ID: int, NAME: string]

scala> df.show
+---+-----+
| ID| NAME|
+---+-----+
|  1|Peter|
|  2|  kkk|
+---+-----+

```





### 添加列

#### java

重点是要引入`functions`类。

```java
import org.apache.spark.sql.functions;

Dataset<Row> dataset = spark.sql(inputParameter.getEtlSql());
Dataset<Row> addColDs = dataset.withColumn("newColName", functions.md5(functions.concat_ws(",", columnsArray)));
				addColDs.show(false);
```

#### scala

$跟的是字段名。

函数直接使用即可。

```scala
scala>  val addCol = df.withColumn("foo", md5(concat_ws(",",$"ID",$"NAME")))
addCol: org.apache.spark.sql.DataFrame = [ID: int, NAME: string ... 1 more field]
 
 scala> addCol.show(false)
+---+-----+--------------------------------+
|ID |NAME |foo                             |
+---+-----+--------------------------------+
|1  |Peter|90483ca7ae25254b49695c9ae419670c|
|2  |kkk  |9a55c34f22b91ec8bdb4ab23140aa545|
+---+-----+--------------------------------+


```



### 获取dataset结构

#### java

```java
Dataset<Row> dataset = spark.sql(inputParameter.getEtlSql());
StructType schema = dataset.schema()
```

spark sql中的schema是一个`org.apache.spark.sql.types.StructType`对象，包含了多个`org.apache.spark.sql.types.DataType`对象





## delta lake体验

### 参考文档

https://docs.delta.io/latest/quick-start.html



### spark shell快速体验

```shell
spark-shell --name "deltalake_test"   --packages io.delta:delta-core_2.11:0.5.0
```

建表

```scala
scala> val data = spark.range(5,10)
scala> data.write.format("delta").mode("overwrite").save("/tmp/delta-table")
```

条件删除

```scala
scala> import io.delta.tables._
scala> val deltaTable = DeltaTable.forPath("/tmp/delta-table")
scala> deltaTable.delete(expr("id % 2 == 0"))
scala> deltaTable.toDF.show
+---+
| id|
+---+
|  7|
|  5|
|  9|
+---+
```



## 常见错误

1. ### 本地mac的spark-shell中使用sql时报无法访问8020端口

   ![image-20200527160459040](http://image-picgo.test.upcdn.net/img/20200527160459.png)

   解决：

   将远程hadoop集群的`core-site.xml`和`hive-site.xml`放入**spark的conf**目录中即可。

   ![image-20200527162807100](http://image-picgo.test.upcdn.net/img/20200527162807.png)

   此时就可以在本地的spark-shell中访问远程集群中的hive表了。

   ![image-20200527162922564](http://image-picgo.test.upcdn.net/img/20200527162922.png)





2. ### 本地mac的spark-shell无法连接内网的yarn资源

   ![image-20200528182312570](http://image-picgo.test.upcdn.net/img/20200528182312.png)

   启动后一直访问本地的yarn获取资源，由于本地的没开yarn，就一直无法获取资源了。

   解决办法：

   将远程hadoop集群的`yarn-site.xml`放入**spark的conf**目录中即可。

   ![image-20200528182450444](http://image-picgo.test.upcdn.net/img/20200528182450.png)

   重启spark-shell后，发现新的问题。

   ![image-20200528182742878](http://image-picgo.test.upcdn.net/img/20200528182743.png)

   解决办法：

   将集群安装spark客户端的服务器上的`topology_script.py`复制到本地mac电脑的`/etc/hadoop/conf`目录

   ![image-20200528182913980](http://image-picgo.test.upcdn.net/img/20200528182914.png)

   此时重启spark-shell即可。



## deequ体验

### 介绍

Deequ is a library built on top of Apache Spark for defining "unit tests for data", which measure data quality in large datasets.

**可以用于观测数据，和评估数据质量。**

[文档](https://github.com/awslabs/deequ)



### 快速体验

启动命令。

```
spark-shell --name "deequ_test"   --packages com.amazon.deequ:deequ:1.0.2
```

输入代码：

```scala
scala> :paste
// Entering paste mode (ctrl-D to finish)

case class Item(
  id: Long,
  productName: String,
  description: String,
  priority: String,
  numViews: Long
)
val rdd = spark.sparkContext.parallelize(Seq(
  Item(1, "Thingy A", "awesome thing.", "high", 0),
  Item(2, "Thingy B", "available at http://thingb.com", null, 0),
  Item(3, null, null, "low", 5),
  Item(4, "Thingy D", "checkout https://thingd.ca", "low", 10),
  Item(5, "Thingy E", null, "high", 12)))

val data = spark.createDataFrame(rdd)

// Exiting paste mode, now interpreting.

defined class Item
rdd: org.apache.spark.rdd.RDD[Item] = ParallelCollectionRDD[0] at parallelize at <console>:20
data: org.apache.spark.sql.DataFrame = [id: bigint, productName: string ... 3 more fields]

scala> :paste
// Entering paste mode (ctrl-D to finish)

import com.amazon.deequ.VerificationSuite
import com.amazon.deequ.checks.{Check, CheckLevel, CheckStatus}


val verificationResult = VerificationSuite()
  .onData(data)
  .addCheck(
    Check(CheckLevel.Error, "unit testing my data")
      .hasSize(_ == 5) // we expect 5 rows
      .isComplete("id") // should never be NULL
      .isUnique("id") // should not contain duplicates
      .isComplete("productName") // should never be NULL
      // should only contain the values "high" and "low"
      .isContainedIn("priority", Array("high", "low"))
      .isNonNegative("numViews") // should not contain negative values
      // at least half of the descriptions should contain a url
      .containsURL("description", _ >= 0.5)
      // half of the items should have less than 10 views
      .hasApproxQuantile("numViews", 0.5, _ <= 10))
    .run()

// Exiting paste mode, now interpreting.

Hive Session ID = 7e17f6f9-eab9-4e08-90f4-f9709cec30f1
import com.amazon.deequ.VerificationSuite                                       
import com.amazon.deequ.checks.{Check, CheckLevel, CheckStatus}
verificationResult: com.amazon.deequ.VerificationResult = VerificationResult(Error,Map(Check(Error,unit testing my data,List(SizeConstraint(Size(None)), CompletenessConstraint(Completeness(id,None)), UniquenessConstraint(Uniqueness(List(id))), CompletenessConstraint(Completeness(productName,None)), ComplianceConstraint(Compliance(priority contained in high,low,`priority` IS NULL OR `priority` IN ('high','low'),None)), ComplianceConstraint(Compliance(numViews is non-negative,COALESCE(numViews, 0.0) >= 0,None)), containsURL(description), ApproxQuantileConstraint(ApproxQuantile(numViews,0.5,0.01)))) -> CheckResult(Check(Error,unit testing my data,List(SizeConstraint(Size(None)), Comple...
scala> :paste
// Entering paste mode (ctrl-D to finish)

import com.amazon.deequ.constraints.ConstraintStatus


if (verificationResult.status == CheckStatus.Success) {
  println("The data passed the test, everything is fine!")
} else {
  println("We found errors in the data:\n")

  val resultsForAllConstraints = verificationResult.checkResults
    .flatMap { case (_, checkResult) => checkResult.constraintResults }

  resultsForAllConstraints
    .filter { _.status != ConstraintStatus.Success }
    .foreach { result => println(s"${result.constraint}: ${result.message.get}") }
}


// Exiting paste mode, now interpreting.

We found errors in the data:

CompletenessConstraint(Completeness(productName,None)): Value: 0.8 does not meet the constraint requirement!
containsURL(description): Value: 0.4 does not meet the constraint requirement!
import com.amazon.deequ.constraints.ConstraintStatus

scala> 
```



## oap

https://github.com/Intel-bigdata/OAP

提升交互式查询性能，增加索引和缓存

## waterdrop



## griffin



## Piflow



## carbondata



## icberg



## hudi



## [spark-alchemy](https://github.com/swoop-inc/spark-alchemy)



## [mahout](https://github.com/apache/mahout)

机器学习



## [spark-df-profiling](https://github.com/julioasotodv/spark-df-profiling)





## Sparklens

它是一个内置 Spark Scheduler 模拟器的 Spark 分析和性能预测工具





## [spark-ranger](https://github.com/NetEase/spark-ranger)

ACL Management for Apache Spark SQL with Apache Ranger



## Optimus

在Spark（Pyspark）的支持下，Optimus允许用户使用自己的或一组预先创建的数据转换功能来清理数据，对其进行概要分析并应用与数据分析和机器学习等场景，可以轻松地利用python语言进行所有这些操作。Optimus主要关注与以下几个方面：

创建一个可靠的API来访问和处理数据。

让用户轻松地从Pandas迁移。

使数据探索更加容易。
