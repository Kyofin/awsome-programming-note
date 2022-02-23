# HDP2.6.5集成Spark3.1.2



## 部署包安装

链接: https://pan.baidu.com/s/1FwlaReWO9F9sAnovzUgXTA  密码: wic1
从上述网盘下载需要的依赖和spark3.1.2。

其中spark3.1.2是直接从spark官网中下载的`spark-3.1.2-bin-hadoop2.7.tgz`。





## 配置spark3

在spark3客户端的conf目录中，增加配置文件`hive-site.xml`。

```
<configuration>

    <property>
      <name>hive.metastore.client.connect.retry.delay</name>
      <value>5</value>
    </property>

    <property>
      <name>hive.metastore.client.socket.timeout</name>
      <value>1800</value>
    </property>

    <property>
      <name>hive.metastore.uris</name>
      <value>thrift://10.93.6.167:9083</value>
    </property>

    <property>
      <name>hive.server2.enable.doAs</name>
      <value>false</value>
    </property>

    <property>
      <name>hive.server2.thrift.port</name>
      <value>10016</value>
    </property>

    <property>
      <name>hive.server2.transport.mode</name>
      <value>binary</value>
    </property>

 <property>
      <name>hive.metastore.warehouse.dir</name>
      <value>/apps/hive/warehouse</value>
    </property>

  </configuration>
```



在spark3客户端的conf目录中，增加配置文件`spark-defaults.conf`。

```

# Default system properties included when running spark-submit.
# This is useful for setting default environmental settings.

# Example:
# spark.master                     spark://master:7077
# spark.eventLog.enabled           true
# spark.eventLog.dir               hdfs://namenode:8021/directory
# spark.serializer                 org.apache.spark.serializer.KryoSerializer
# spark.driver.memory              5g
# spark.executor.extraJavaOptions  -XX:+PrintGCDetails -Dkey=value -Dnumbers="one two three"
spark.driver.extraLibraryPath /usr/hdp/current/hadoop-client/lib/native:/usr/hdp/current/hadoop-client/lib/native/Linux-amd64-64
spark.eventLog.dir hdfs:///spark2-history/
spark.eventLog.enabled true
spark.executor.extraLibraryPath /usr/hdp/current/hadoop-client/lib/native:/usr/hdp/current/hadoop-client/lib/native/Linux-amd64-64
spark.history.fs.logDirectory hdfs:///spark2-history/
spark.history.kerberos.keytab none
spark.history.kerberos.principal none
spark.history.provider org.apache.spark.deploy.history.FsHistoryProvider
spark.history.ui.port 18081
spark.yarn.historyServer.address 10.93.6.167:18081
spark.yarn.queue default


spark.driver.extraJavaOptions -Dhdp.version=2.6.5.0-292
spark.yarn.am.extraJavaOptions -Dhdp.version=2.6.5.0-292
```



复制`spark-env.sh.template`文件为`spark-env.sh`，并在文件内加入以下配置。

```
export HADOOP_CONF_DIR=/etc/hadoop/conf
```



## FAQ

### 使用yarn client模式启动时报错。

```
[hdfs@hadoop-slave1 spark-3.1.2-bin-hadoop2.7]$ bin/spark-shell --master yarn
Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
java.lang.NoClassDefFoundError: com/sun/jersey/api/client/config/ClientConfig
  at org.apache.hadoop.yarn.client.api.TimelineClient.createTimelineClient(TimelineClient.java:55)
  at org.apache.hadoop.yarn.client.api.impl.YarnClientImpl.createTimelineClient(YarnClientImpl.java:181)
  at org.apache.hadoop.yarn.client.api.impl.YarnClientImpl.serviceInit(YarnClientImpl.java:168)
  at org.apache.hadoop.service.AbstractService.init(AbstractService.java:163)
  at org.apache.spark.deploy.yarn.Client.submitApplication(Client.scala:175)
  at org.apache.spark.scheduler.cluster.YarnClientSchedulerBackend.start(YarnClientSchedulerBackend.scala:62)
  at org.apache.spark.scheduler.TaskSchedulerImpl.start(TaskSchedulerImpl.scala:220)
  at org.apache.spark.SparkContext.<init>(SparkContext.scala:579)
  at org.apache.spark.SparkContext$.getOrCreate(SparkContext.scala:2672)
  at org.apache.spark.sql.SparkSession$Builder.$anonfun$getOrCreate$2(SparkSession.scala:945)
  at scala.Option.getOrElse(Option.scala:189)
  at org.apache.spark.sql.SparkSession$Builder.getOrCreate(SparkSession.scala:939)
  at org.apache.spark.repl.Main$.createSparkSession(Main.scala:106)
  ... 55 elided
Caused by: java.lang.ClassNotFoundException: com.sun.jersey.api.client.config.ClientConfig
  at java.net.URLClassLoader.findClass(URLClassLoader.java:381)
  at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
  at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:349)
  at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
  ... 68 more
<console>:14: error: not found: value spark
       import spark.implicits._
              ^
<console>:14: error: not found: value spark
       import spark.sql
              ^
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 3.1.2
      /_/
         
Using Scala version 2.12.10 (Java HotSpot(TM) 64-Bit Server VM, Java 1.8.0_181)
Type in expressions to have them evaluated.
Type :help for more information.

scala> 
```

关闭yarn的timeline service。

![image-20210709093224357](http://image-picgo.test.upcdn.net/img/20210709093224.png)



也可以不关闭这个yarn的配置，直接在spark3的客户端里的`spark-default.conf`里配置以下内容：

```
spark.hadoop.yarn.timeline-service.enabled false
```





### 读写hive表时报错。

```
scala> spark.table("text_t_dept_doctor_patient_doris").show
org.apache.spark.sql.AnalysisException: org.apache.hadoop.hive.ql.metadata.HiveException: Unable to fetch table text_t_dept_doctor_patient_doris. Invalid method name: 'get_table_req'
  at org.apache.spark.sql.hive.HiveExternalCatalog.withClient(HiveExternalCatalog.scala:112)
  at org.apache.spark.sql.hive.HiveExternalCatalog.tableExists(HiveExternalCatalog.scala:854)
  at org.apache.spark.sql.catalyst.catalog.ExternalCatalogWithListener.tableExists(ExternalCatalogWithListener.scala:146)
  at org.apache.spark.sql.catalyst.catalog.SessionCatalog.tableExists(SessionCatalog.scala:462)
  at org.apache.spark.sql.catalyst.catalog.SessionCatalog.requireTableExists(SessionCatalog.scala:197)
  at org.apache.spark.sql.catalyst.catalog.SessionCatalog.getTableRawMetadata(SessionCatalog.scala:488)
  at org.apache.spark.sql.catalyst.catalog.SessionCatalog.getTableMetadata(SessionCatalog.scala:474)
  at org.apache.spark.sql.execution.datasources.v2.V2SessionCatalog.loadTable(V2SessionCatalog.scala:65)
  at org.apache.spark.sql.connector.catalog.CatalogV2Util$.loadTable(CatalogV2Util.scala:282)
  at org.apache.spark.sql.catalyst.analysis.Analyzer$ResolveRelations$.loaded$lzycompute$1(Analyzer.scala:1183)
  at org.apache.spark.sql.catalyst.analysis.Analyzer$ResolveRelations$.loaded$1(Analyzer.scala:1183)
  at org.apache.spark.sql.catalyst.analysis.Analyzer$ResolveRelations$.$anonfun$lookupRelation$3(Analyzer.scala:1221)
  at scala.Option.orElse(Option.scala:447)
  at org.apache.spark.sql.catalyst.analysis.Analyzer$ResolveRelations$.org$apache$spark$sql$catalyst$analysis$Analyzer$ResolveRelations$$lookupRelation(Analyzer.scala:1220)
  at org.apache.spark.sql.catalyst.analysis.Analyzer$ResolveRelations$$anonfun$apply$10.applyOrElse(Analyzer.scala:1145)
  at org.apache.spark.sql.catalyst.analysis.Analyzer$ResolveRelations$$anonfun$apply$10.applyOrElse(Analyzer.scala:1112)
  at org.apache.spark.sql.catalyst.plans.logical.AnalysisHelper.$anonfun$resolveOperatorsUp$3(AnalysisHelper.scala:90)
  at org.apache.spark.sql.catalyst.trees.CurrentOrigin$.withOrigin(TreeNode.scala:74)
  at org.apache.spark.sql.catalyst.plans.logical.AnalysisHelper.$anonfun$resolveOperatorsUp$1(AnalysisHelper.scala:90)
  at org.apache.spark.sql.catalyst.plans.logical.AnalysisHelper$.allowInvokingTransformsInAnalyzer(AnalysisHelper.scala:221)
  at org.apache.spark.sql.catalyst.plans.logical.AnalysisHelper.resolveOperatorsUp(AnalysisHelper.scala:86)
  at org.apache.spark.sql.catalyst.plans.logical.AnalysisHelper.resolveOperatorsUp$(AnalysisHelper.scala:84)
  at org.apache.spark.sql.catalyst.plans.logical.LogicalPlan.resolveOperatorsUp(LogicalPlan.scala:29)
  at org.apache.spark.sql.catalyst.analysis.Analyzer$ResolveRelations$.apply(Analyzer.scala:1112)
  at org.apache.spark.sql.catalyst.analysis.Analyzer$ResolveRelations$.apply(Analyzer.scala:1077)
  at org.apache.spark.sql.catalyst.rules.RuleExecutor.$anonfun$execute$2(RuleExecutor.scala:216)
  at scala.collection.LinearSeqOptimized.foldLeft(LinearSeqOptimized.scala:126)
  at scala.collection.LinearSeqOptimized.foldLeft$(LinearSeqOptimized.scala:122)
  at scala.collection.immutable.List.foldLeft(List.scala:89)
  at org.apache.spark.sql.catalyst.rules.RuleExecutor.$anonfun$execute$1(RuleExecutor.scala:213)
  at org.apache.spark.sql.catalyst.rules.RuleExecutor.$anonfun$execute$1$adapted(RuleExecutor.scala:205)
  at scala.collection.immutable.List.foreach(List.scala:392)
  at org.apache.spark.sql.catalyst.rules.RuleExecutor.execute(RuleExecutor.scala:205)
  at org.apache.spark.sql.catalyst.analysis.Analyzer.org$apache$spark$sql$catalyst$analysis$Analyzer$$executeSameContext(Analyzer.scala:196)
  at org.apache.spark.sql.catalyst.analysis.Analyzer.execute(Analyzer.scala:190)
  at org.apache.spark.sql.catalyst.analysis.Analyzer.execute(Analyzer.scala:155)
  at org.apache.spark.sql.catalyst.rules.RuleExecutor.$anonfun$executeAndTrack$1(RuleExecutor.scala:183)
  at org.apache.spark.sql.catalyst.QueryPlanningTracker$.withTracker(QueryPlanningTracker.scala:88)
  at org.apache.spark.sql.catalyst.rules.RuleExecutor.executeAndTrack(RuleExecutor.scala:183)
  at org.apache.spark.sql.catalyst.analysis.Analyzer.$anonfun$executeAndCheck$1(Analyzer.scala:174)
  at org.apache.spark.sql.catalyst.plans.logical.AnalysisHelper$.markInAnalyzer(AnalysisHelper.scala:228)
  at org.apache.spark.sql.catalyst.analysis.Analyzer.executeAndCheck(Analyzer.scala:173)
  at org.apache.spark.sql.execution.QueryExecution.$anonfun$analyzed$1(QueryExecution.scala:73)
  at org.apache.spark.sql.catalyst.QueryPlanningTracker.measurePhase(QueryPlanningTracker.scala:111)
  at org.apache.spark.sql.execution.QueryExecution.$anonfun$executePhase$1(QueryExecution.scala:143)
  at org.apache.spark.sql.SparkSession.withActive(SparkSession.scala:775)
  at org.apache.spark.sql.execution.QueryExecution.executePhase(QueryExecution.scala:143)
  at org.apache.spark.sql.execution.QueryExecution.analyzed$lzycompute(QueryExecution.scala:73)
  at org.apache.spark.sql.execution.QueryExecution.analyzed(QueryExecution.scala:71)
  at org.apache.spark.sql.execution.QueryExecution.assertAnalyzed(QueryExecution.scala:63)
  at org.apache.spark.sql.Dataset$.$anonfun$ofRows$1(Dataset.scala:90)
  at org.apache.spark.sql.SparkSession.withActive(SparkSession.scala:775)
  at org.apache.spark.sql.Dataset$.ofRows(Dataset.scala:88)
  at org.apache.spark.sql.DataFrameReader.table(DataFrameReader.scala:891)
  at org.apache.spark.sql.SparkSession.table(SparkSession.scala:596)
  ... 47 elided
Caused by: org.apache.hadoop.hive.ql.metadata.HiveException: Unable to fetch table text_t_dept_doctor_patient_doris. Invalid method name: 'get_table_req'
  at org.apache.hadoop.hive.ql.metadata.Hive.getTable(Hive.java:1282)
  at org.apache.spark.sql.hive.client.HiveClientImpl.getRawTableOption(HiveClientImpl.scala:392)
  at org.apache.spark.sql.hive.client.HiveClientImpl.$anonfun$tableExists$1(HiveClientImpl.scala:406)
  at scala.runtime.java8.JFunction0$mcZ$sp.apply(JFunction0$mcZ$sp.java:23)
  at org.apache.spark.sql.hive.client.HiveClientImpl.$anonfun$withHiveState$1(HiveClientImpl.scala:291)
  at org.apache.spark.sql.hive.client.HiveClientImpl.liftedTree1$1(HiveClientImpl.scala:224)
  at org.apache.spark.sql.hive.client.HiveClientImpl.retryLocked(HiveClientImpl.scala:223)
  at org.apache.spark.sql.hive.client.HiveClientImpl.withHiveState(HiveClientImpl.scala:273)
  at org.apache.spark.sql.hive.client.HiveClientImpl.tableExists(HiveClientImpl.scala:406)
  at org.apache.spark.sql.hive.HiveExternalCatalog.$anonfun$tableExists$1(HiveExternalCatalog.scala:854)
  at scala.runtime.java8.JFunction0$mcZ$sp.apply(JFunction0$mcZ$sp.java:23)
  at org.apache.spark.sql.hive.HiveExternalCatalog.withClient(HiveExternalCatalog.scala:102)
  ... 101 more
Caused by: org.apache.thrift.TApplicationException: Invalid method name: 'get_table_req'
  at org.apache.thrift.TServiceClient.receiveBase(TServiceClient.java:79)
  at org.apache.hadoop.hive.metastore.api.ThriftHiveMetastore$Client.recv_get_table_req(ThriftHiveMetastore.java:1567)
  at org.apache.hadoop.hive.metastore.api.ThriftHiveMetastore$Client.get_table_req(ThriftHiveMetastore.java:1554)
  at org.apache.hadoop.hive.metastore.HiveMetaStoreClient.getTable(HiveMetaStoreClient.java:1350)
  at org.apache.hadoop.hive.ql.metadata.SessionHiveMetaStoreClient.getTable(SessionHiveMetaStoreClient.java:127)
  at sun.reflect.GeneratedMethodAccessor29.invoke(Unknown Source)
  at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
  at java.lang.reflect.Method.invoke(Method.java:498)
  at org.apache.hadoop.hive.metastore.RetryingMetaStoreClient.invoke(RetryingMetaStoreClient.java:173)
  at com.sun.proxy.$Proxy42.getTable(Unknown Source)
  at sun.reflect.GeneratedMethodAccessor29.invoke(Unknown Source)
  at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
  at java.lang.reflect.Method.invoke(Method.java:498)
  at org.apache.hadoop.hive.metastore.HiveMetaStoreClient$SynchronizedHandler.invoke(HiveMetaStoreClient.java:2336)
  at com.sun.proxy.$Proxy42.getTable(Unknown Source)
  at org.apache.hadoop.hive.ql.metadata.Hive.getTable(Hive.java:1274)
  ... 112 more
```

补充hive依赖，如下：

![image-20210708181941320](http://image-picgo.test.upcdn.net/img/20210708181941.png)

在spark-3客户端中建目录`dependency_libs/hive`，将这些依赖放入spark-3的客户端目录中。

![image-20210708182025017](http://image-picgo.test.upcdn.net/img/20210708182025.png)

启动spark作业时，增加命令行参数。

```
bin/spark-shell --master local[*] --conf spark.sql.hive.metastore.version=1.2.1  --conf spark.sql.hive.metastore.jars=/opt/spark-3.1.2-bin-hadoop2.7/dependency_libs/hive/*
```

此时就可以正常读写hdp集群中的hive表了。

如果不想每次提交作业都要手动加上这两个配置参数，可以把它们配置到`spark-default.conf`文件中。

```
spark.sql.hive.metastore.version 1.2.1  
spark.sql.hive.metastore.jars /opt/spark-3.1.2-bin-hadoop2.7/dependency_libs/hive/*
```





# yarn开启HA后报错

报错信息显示下面类不存在。

```
org.apache.hadoop.yarn.client.RequestHedgingRMFailoverProxyProvider
```

**解决办法**：

在ambari中设置yarn的配置。

```
yarn.client.failover-proxy-provider=org.apache.hadoop.yarn.client.ConfiguredRMFailoverProxyProvider
```

