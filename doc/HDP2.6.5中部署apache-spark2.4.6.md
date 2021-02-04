# HDP2.6.5中部署apache-spark2.4.6

## 下载spark2.4.6

```
wget http://archive.apache.org/dist/spark/spark-2.4.6/spark-2.4.6.tgz
```



## 使用hdp2.6.5版hadoop编译spark2.4.6

maven要求3.6.3和Java 8

```
export MAVEN_OPTS=-Xmx2g -XX:ReservedCodeCacheSize=512m
```

进入源码目录，改变scala版本。执行：

```
./dev/change-scala-version.sh 2.12
```

修改`./dev/make-distribution.sh`，加进去

```
VERSION=2.4.6
SCALA_VERSION=2.12
SPARK_HADOOP_VERSION=2.7.3.2.6.5.0-292
SPARK_HIVE=1

```

make-distribute注释掉检查环境变量的脚本。

![DA6A44BB-A7F9-48E2-A40A-0948262DB161](http://image-picgo.test.upcdn.net/img/20210204141817.png)

pom默认里没有hdp的仓库，需要pom里加进hdp的仓库。

```
<repository>
    <id>hortonworks</id>
    <url>https://repo.hortonworks.com/content/repositories/releases/</url>
</repository>

```

执行编译部署包

```
./dev/make-distribution.sh --name 2.7.3-hdp2.6.5 \
--tgz -Phive -Phive-thriftserver -Pyarn \
-Phadoop-2.7 -Dhadoop.version=2.7.3.2.6.5.0-292 -Dscala.version=2.12.10

```





## 上传部署包到hdp集群中



## 配置spark-env.sh

```
export HADOOP_CONF_DIR=/etc/hadoop/conf

```



## 配置spark-defaults.conf

```
spark.driver.extraLibraryPath /usr/hdp/current/hadoop-client/lib/native:/usr/hdp/current/hadoop-client/lib/native/Linux-amd64-64
spark.eventLog.dir hdfs:///spark2-history/
spark.eventLog.enabled true
spark.executor.extraLibraryPath /usr/hdp/current/hadoop-client/lib/native:/usr/hdp/current/hadoop-client/lib/native/Linux-amd64-64
spark.history.fs.logDirectory hdfs:///spark2-history/
spark.history.kerberos.keytab none
spark.history.kerberos.principal none
spark.history.provider org.apache.spark.deploy.history.FsHistoryProvider
spark.history.ui.port 18081
spark.yarn.historyServer.address hadoop-master:18081
spark.yarn.queue default


spark.driver.extraJavaOptions -Dhdp.version=2.6.5.0-292
spark.yarn.am.extraJavaOptions -Dhdp.version=2.6.5.0-292

```

重点是最后的hdp.version。不加提交到yarn上会报错。其他可以参考hdp自带的spark2.3客户端中的配置。



## 配置hive-site

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
      <value>thrift://hadoop-slave1:9083</value>
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

  </configuration>

```



## 解决jersy依赖缺失问题

### 

spark 提交任务到yarn上时报如下错误：

```
2020-02-17 09:53:01 INFO  SparkUI:54 - Bound SparkUI to 0.0.0.0, and started at http://master:4040
2020-02-17 09:53:01 INFO  SparkContext:54 - Added JAR file:/root/software/spark-2.3.0/import.jar at spark://master:45607/jars/import.jar with timestamp 1581951181988
Exception in thread "main" java.lang.NoClassDefFoundError: com/sun/jersey/api/client/config/ClientConfig
        at org.apache.hadoop.yarn.client.api.TimelineClient.createTimelineClient(TimelineClient.java:45)
        at org.apache.hadoop.yarn.client.api.impl.YarnClientImpl.serviceInit(YarnClientImpl.java:163)
        at org.apache.hadoop.service.AbstractService.init(AbstractService.java:163)
        at org.apache.spark.deploy.yarn.Client.submitApplication(Client.scala:151)
        at org.apache.spark.scheduler.cluster.YarnClientSchedulerBackend.start(YarnClientSchedulerBackend.scala:57)
        at org.apache.spark.scheduler.TaskSchedulerImpl.start(TaskSchedulerImpl.scala:164)
        at org.apache.spark.SparkContext.<init>(SparkContext.scala:500)
        at org.apache.spark.SparkContext$.getOrCreate(SparkContext.scala:2486)
        at org.apache.spark.sql.SparkSession$Builder$$anonfun$7.apply(SparkSession.scala:930)
        at org.apache.spark.sql.SparkSession$Builder$$anonfun$7.apply(SparkSession.scala:921)
        at scala.Option.getOrElse(Option.scala:121)
        at org.apache.spark.sql.SparkSession$Builder.getOrCreate(SparkSession.scala:921)
        at cn.aiaudit.KafkaToHive.main(KafkaToHive.java:71)
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:498)
        at org.apache.spark.deploy.JavaMainApplication.start(SparkApplication.scala:52)
        at org.apache.spark.deploy.SparkSubmit$.org$apache$spark$deploy$SparkSubmit$$runMain(SparkSubmit.scala:879)
        at org.apache.spark.deploy.SparkSubmit$.doRunMain$1(SparkSubmit.scala:197)
        at org.apache.spark.deploy.SparkSubmit$.submit(SparkSubmit.scala:227)
        at org.apache.spark.deploy.SparkSubmit$.main(SparkSubmit.scala:136)
        at org.apache.spark.deploy.SparkSubmit.main(SparkSubmit.scala)
Caused by: java.lang.ClassNotFoundException: com.sun.jersey.api.client.config.ClientConfig
        at java.net.URLClassLoader.findClass(URLClassLoader.java:381)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
        at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:335)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
        ... 23 more
2020-02-17 09:53:02 INFO  DiskBlockManager:54 - Shutdown hook called
2020-02-17 09:53:02 INFO  ShutdownHookManager:54 - Shutdown hook called
2020-02-17 09:53:02 INFO  ShutdownHookManager:54 - Deleting directory /tmp/spark-25555cbf-3fe2-4706-86bf-5bce80f8f0ca

1234567891011121314151617181920212223242526272829303132333435
```

错误原因

```
spark版本：2.3.0
hadoop版本：2.6.5
12
```

spark依赖的jersey版本为2.22.2，hadoop yarn依赖的jersey版本为1.9。所以spark提交不到yarn上去。

### 方法1

查看jersey jar包版本

```
#hadoop yarn
/root/software/hadoop-2.6.5/share/hadoop/yarn/lib/jersey-core-1.9.jar
/root/software/hadoop-2.6.5/share/hadoop/yarn/lib/jersey-client-1.9.jar

#spark
/root/software/spark-2.3.0/jars/jersey-client-2.22.2.jar
123456
```

拷贝jar包到spark jar目录下

```
cp /root/software/hadoop-2.6.5/share/hadoop/yarn/lib/jersey-core-1.9.jar /root/software/spark-2.3.0/jars
cp /root/software/hadoop-2.6.5/share/hadoop/yarn/lib/jersey-client-1.9.jar /root/software/spark-2.3.0/jars
12
```

拷贝完成后就可以正常提交了。



### 方法2

取消yarn的timeline