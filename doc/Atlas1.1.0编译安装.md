# Atlas1.1.0编译安装

## 环境

本地mac是基于cdh5.16.2版本的hadoop环境

## 下载

http://archive.apache.org/dist/atlas/1.1.0/



## 编译

### 为避免出现编译文档错误

```
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-site-plugin:3.7:site (default) on project atlas-docs: SiteToolException: The site descriptor cannot be resolved from the repository: ArtifactResolutionException: Unable to locate site descriptor: Could not transfer artifact org.apache:apache:xml:site_en:17 from/to typesafe (http://repo.typesafe.com/typesafe/releases/): Transfer failed for http://repo.typesafe.com/typesafe/releases/org/apache/apache/17/apache-17-site_en.xml
[ERROR]   org.apache:apache:xml:17
[ERROR]
[ERROR] from the specified remote repositories:
[ERROR]   central (https://maven.aliyun.com/nexus/content/groups/public, releases=true, snapshots=false),
[ERROR]   snapshots (https://maven.aliyun.com/nexus/content/groups/public, releases=false, snapshots=true),
[ERROR]   rdc-releases (https://repo.rdc.aliyun.com/repository/129217-release-BDOmuI/, releases=true, snapshots=false),
[ERROR]   rdc-snapshots (https://repo.rdc.aliyun.com/repository/129217-snapshot-dLwFxP/, releases=false, snapshots=true),
[ERROR]   oracleReleases (http://download.oracle.com/maven, releases=true, snapshots=true),
[ERROR]   hortonworks.repo (http://repo.hortonworks.com/content/repositories/releases, releases=true, snapshots=false),
[ERROR]   apache.snapshots.repo (https://repository.apache.org/content/groups/snapshots, releases=true, snapshots=true),
[ERROR]   apache-staging (https://repository.apache.org/content/groups/staging/, releases=true, snapshots=true),
[ERROR]   default (https://repository.apache.org/content/groups/public/, releases=true, snapshots=true),
[ERROR]   java.net-Public (https://maven.java.net/content/groups/public/, releases=true, snapshots=true),
[ERROR]   repository.jboss.org-public (https://repository.jboss.org/nexus/content/groups/public, releases=true, snapshots=true),
[ERROR]   typesafe (http://repo.typesafe.com/typesafe/releases/, releases=true, snapshots=true),
[ERROR]   apache.snapshots (http://repository.apache.org/snapshots, releases=false, snapshots=true)
```

可以注释掉pom中的docs模块

![image-20201205221333361](http://image-picgo.test.upcdn.net/img/20201205221333.png)





### 为了避免编译Apache Atlas JanusGraph DB Impl失败

```
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-remote-resources-plugin:1.5:process (default) on project atlas-graphdb-janus: Error resolving project artifact: Could not transfer artifact com.sleepycat:je:pom:7.4.5 from/to central (http://repo1.maven.org/maven2): Failed to transfer file: http://repo1.maven.org/maven2/com/sleepycat/je/7.4.5/je-7.4.5.pom. Return code is: 501, ReasonPhrase: HTTPS Required. for project com.sleepycat:je:jar:7.4.5 -> [Help 1]
[ERROR] 
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/MojoExecutionException
[ERROR] 
[ERROR] After correcting the problems, you can resume the build with the command
[ERROR]   mvn <goals> -rf :atlas-graphdb-janus
```

需要修改pom.xml增加

```
<repository>
    <id>oracleReleases</id>
    <name>Oracle Released Java Packages</name>
    <url>http://download.oracle.com/maven</url>
    <layout>default</layout>
</repository>
```

执行编译:

1. 该方式编译不会内嵌HBase和Solr

   ```
   mvn clean -DskipTests package -Pdist 
   ```

2. 内嵌HBase和Solr，测试用这种方式

   ```
   mvn clean -DskipTests package -Pdist,embedded-hbase-solr  
   ```



## 安装运行

1.  进入target目录找到编译好的文件

   ```
   cd ~/Downloads/apache-atlas-1.1.0/distro/target
   ```

2. 找到二进制部署压缩包`apache-atlas-1.1.0-bin.tar.gz   `

3. 默认编译好就有上面二进制包解压好的目录，进入该目录

   ```
   cd ~/Downloads/apache-atlas-1.1.0/distro/target/apache-atlas-1.1.0-bin/apache-atlas-1.1.0
   ```

4. 启动atlas，会把内嵌的solr和hbase都启动。

   ```
   ~/Downloads/apache-atlas-1.1.0/distro/target/apache-atlas-1.1.0-bin/apache-atlas-1.1.0 » bin/atlas_start.py   
   ```

   ![image-20201206133627262](http://image-picgo.test.upcdn.net/img/20201206133627.png)

   第一次启动会比较慢。会在solr中初始化三个core。

   ![image-20201206133814087](http://image-picgo.test.upcdn.net/img/20201206133814.png)

   ![image-20201206133929897](http://image-picgo.test.upcdn.net/img/20201206133930.png)

   使用hbase shell可以访问atlas内嵌的hbase里的数据。

   ![image-20201206134204278](http://image-picgo.test.upcdn.net/img/20201206134204.png)

5. 跑起来后登陆浏览器查看http://localhost:21000/，默认账号密码都是admin

6. 跑起来后跑官方例子导入数据

   ```shell
   ~/Downloads/apache-atlas-1.1.0/distro/target/apache-atlas-1.1.0-bin/apache-atlas-1.1.0 » bin/quick_start.py   
   log4j:WARN No such property [maxFileSize] in org.apache.log4j.PatternLayout.
   log4j:WARN No such property [maxBackupIndex] in org.apache.log4j.PatternLayout.
   log4j:WARN No such property [maxFileSize] in org.apache.log4j.PatternLayout.
   log4j:WARN No such property [maxBackupIndex] in org.apache.log4j.PatternLayout.
   log4j:WARN No such property [maxFileSize] in org.apache.log4j.PatternLayout.
   Enter username for atlas :- admin
   Enter password for atlas :-
   
   Creating sample types:
   Created type [DB]
   Created type [Table]
   Created type [StorageDesc]
   Created type [Column]
   Created type [LoadProcess]
   Created type [View]
   Created type [JdbcAccess]
   Created type [ETL]
   Created type [Metric]
   Created type [PII]
   Created type [Fact]
   Created type [Dimension]
   Created type [Log Data]
   
   Creating sample entities:
   Created entity of type [DB], guid: 11b8d158-b3d8-4f47-a6bf-0b627edae901
   Created entity of type [DB], guid: d407c8d4-f03a-44cc-bd36-904c4e7c4c53
   Created entity of type [DB], guid: 914a3abd-af99-4857-9f14-cc2115ce7f22
   Created entity of type [StorageDesc], guid: 334d2e53-3865-4433-b489-c2a386403ab6
   Created entity of type [Column], guid: 2118436c-7315-4ae4-870a-602ab0708eb5
   Created entity of type [Column], guid: c983d7fd-cf3a-48bf-8d54-286d086ea5c5
   Created entity of type [Column], guid: 3a0e9a33-466a-4416-9a19-51675c6b153e
   Created entity of type [Column], guid: 32ae5e65-d461-4dcf-bef3-11ef307454f5
   Created entity of type [Column], guid: 763e6d60-f66a-4d99-b21c-bb489ddb0448
   Created entity of type [Column], guid: 6eb237d8-f446-406a-8ffe-d9b2f801d448
   Created entity of type [Column], guid: 8eb59808-83eb-4213-9a53-48a40d9f679f
   Created entity of type [Column], guid: f43aba5e-db9b-45ca-aeb8-8396db2e461d
   Created entity of type [Column], guid: 7ed9fab6-e923-42b1-90ab-9fc27e8876d7
   Created entity of type [Column], guid: d0ac5370-8c71-4156-845d-1ecf903d35ac
   Created entity of type [Column], guid: d3203eb0-5f54-47ff-8aed-311768f60606
   Created entity of type [Column], guid: 2cd8623b-4fae-47bb-ad11-3b21bbf453a7
   Created entity of type [Column], guid: 77b50dc2-db18-4a28-b113-c66b386f5e6e
   Created entity of type [Column], guid: a7d3f082-920e-41f9-9926-bce01b6561ef
   Created entity of type [Column], guid: 87d2d086-0437-4de6-afc4-a250b8453023
   Created entity of type [Column], guid: b53b7af9-db2b-4329-9a5d-7f624b2fd7d1
   Created entity of type [Column], guid: 30f7d576-baaf-4c10-8e58-7d3e83718329
   Created entity of type [Table], guid: febcfa8a-39c1-4217-ad17-10a4ffeba7bd
   Created entity of type [Table], guid: d45866c3-f604-4bac-a395-e93a4efacab6
   Created entity of type [Table], guid: e902287a-8942-4a35-834d-cbf4a2a7455e
   Created entity of type [Table], guid: 4ee45ecd-13e7-42d2-a8c7-f2f59fb92715
   Created entity of type [Table], guid: 5b5590a7-926a-4a75-a2a5-0ce50201544a
   Created entity of type [Table], guid: 44b7c85e-9ea3-4a07-b17c-53d545a9dff2
   Created entity of type [Table], guid: bdf01a6f-5b73-4763-a13a-cbf744e627b4
   Created entity of type [Table], guid: 59a0bdfc-8fbe-46c7-8256-c88b5f3d2b30
   Created entity of type [View], guid: 00c9a3c0-f04d-4d9d-b3f8-0dcab24c69e1
   Created entity of type [View], guid: 487f7424-dfd7-4fba-96ea-91cdf64fc561
   Created entity of type [LoadProcess], guid: fa3dc4f8-dc43-4935-8435-8a011149d79f
   Created entity of type [LoadProcess], guid: cad882bb-bdf5-4beb-8043-af0adc1f26a2
   Created entity of type [LoadProcess], guid: 60fdd448-bba0-4d25-89d2-2f96950ec0e2
   
   Sample DSL Queries:
   query [from DB] returned [3] rows.
   query [DB] returned [3] rows.
   query [DB where name=%22Reporting%22] returned [1] rows.
   query [DB where name=%22encode_db_name%22] returned [ 0 ] rows.
   query [Table where name=%2522sales_fact%2522] returned [1] rows.
   query [DB where name="Reporting"] returned [1] rows.
   query [DB where DB.name="Reporting"] returned [1] rows.
   query [DB name = "Reporting"] returned [1] rows.
   query [DB DB.name = "Reporting"] returned [1] rows.
   query [DB where name="Reporting" select name, owner] returned [1] rows.
   query [DB where DB.name="Reporting" select name, owner] returned [1] rows.
   aquery [DB has name] returned [3] rows.
   query [DB where DB has name] returned [3] rows.
   query [DB is JdbcAccess] returned [ 0 ] rows.
   query [from Table] returned [8] rows.
   query [Table] returned [8] rows.
   query [Table is Dimension] returned [5] rows.
   query [Column where Column isa PII] returned [4] rows.
   query [View is Dimension] returned [2] rows.
   query [Column select Column.name] returned [7] rows.
   query [Column select name] returned [6] rows.
   query [Column where Column.name="customer_id"] returned [2] rows.
   query [from Table select Table.name] returned [8] rows.
   query [DB where (name = "Reporting")] returned [1] rows.
   query [DB where DB is JdbcAccess] returned [ 0 ] rows.
   query [DB where DB has name] returned [3] rows.
   query [DB as db1 Table where (db1.name = "Reporting")] returned [ 0 ] rows.
   query [Dimension] returned [9] rows.
   query [JdbcAccess] returned [2] rows.
   query [ETL] returned [6] rows.
   query [Metric] returned [4] rows.
   query [PII] returned [4] rows.
   query [`Log Data`] returned [4] rows.
   query [Table where name="sales_fact", columns] returned [4] rows.
   query [Table where name="sales_fact", columns as column select column.name, column.dataType, column.comment] returned [4] rows.
   query [from DataSet] returned [10] rows.
   query [from Process] returned [3] rows.
   
   Sample Lineage Info:
   sales_fact_daily_mv(Table) -> loadSalesMonthly(LoadProcess)
   loadSalesDaily(LoadProcess) -> sales_fact_daily_mv(Table)
   loadSalesMonthly(LoadProcess) -> sales_fact_monthly_mv(Table)
   sales_fact(Table) -> loadSalesDaily(LoadProcess)
   time_dim(Table) -> loadSalesDaily(LoadProcess)
   Sample data added to Apache Atlas Server.
   ```

   可以看到例子创建的数据。

   ![image-20201119152734637](http://image-picgo.test.upcdn.net/img/20201119152734.png)

   

## 导入hive数据

本地是cdh5.16.2的hive。

导入hive数据到atlas是不需要启动hive或者hadoop的。

### 确保已经配置hive到环境变量

```
~/Downloads/apache-atlas-1.1.0/distro/target/apache-atlas-1.1.0-bin/apache-atlas-1.1.0 » which hive           
/Users/huzekang/cdh5.16/hive-1.1.0-cdh5.16.2/bin/hive
```



### 复制atlas配置到hive配置目录

```
~/Downloads/apache-atlas-1.1.0/distro/target/apache-atlas-1.1.0-bin/apache-atlas-1.1.0 » cp conf/atlas-application.properties ~/cdh5.16/hive-1.1.0-cdh5.16.2/conf
```

否则会报如下错误：

```
2020-12-06 13:44:46,263 ERROR - [main:] ~ Import failed (HiveMetaStoreBridge:176)
org.apache.atlas.AtlasException: Failed to load application properties
	at org.apache.atlas.ApplicationProperties.get(ApplicationProperties.java:109)
	at org.apache.atlas.ApplicationProperties.get(ApplicationProperties.java:71)
	at org.apache.atlas.hive.bridge.HiveMetaStoreBridge.main(HiveMetaStoreBridge.java:119)
Caused by: org.apache.commons.configuration.ConfigurationException: Cannot locate configuration source null
	at org.apache.commons.configuration.AbstractFileConfiguration.load(AbstractFileConfiguration.java:259)
	at org.apache.commons.configuration.AbstractFileConfiguration.load(AbstractFileConfiguration.java:238)
	at org.apache.commons.configuration.AbstractFileConfiguration.<init>(AbstractFileConfiguration.java:197)
	at org.apache.commons.configuration.PropertiesConfiguration.<init>(PropertiesConfiguration.java:284)
	at org.apache.atlas.ApplicationProperties.<init>(ApplicationProperties.java:54)
	at org.apache.atlas.ApplicationProperties.get(ApplicationProperties.java:100)
	... 2 more
```



### 复制atlas执行import-hive.sh时缺失的依赖到atlas的hive hook中

如果不做这步，会报错：

```
Enter username for atlas :- admin
Enter password for atlas :-
Exception in thread "main" java.lang.NoClassDefFoundError: com/fasterxml/jackson/jaxrs/json/JacksonJaxbJsonProvider
	at org.apache.atlas.AtlasBaseClient.getClient(AtlasBaseClient.java:253)
	at org.apache.atlas.AtlasBaseClient.initializeState(AtlasBaseClient.java:425)
	at org.apache.atlas.AtlasBaseClient.initializeState(AtlasBaseClient.java:420)
	at org.apache.atlas.AtlasBaseClient.<init>(AtlasBaseClient.java:115)
	at org.apache.atlas.AtlasClientV2.<init>(AtlasClientV2.java:77)
	at org.apache.atlas.hive.bridge.HiveMetaStoreBridge.main(HiveMetaStoreBridge.java:131)
Caused by: java.lang.ClassNotFoundException: com.fasterxml.jackson.jaxrs.json.JacksonJaxbJsonProvider
	at java.net.URLClassLoader.findClass(URLClassLoader.java:382)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
	at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:349)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
	... 6 more
Failed to import Hive Meta Data!!!
```

解决的方法是，进入目录`~/Downloads/apache-atlas-1.1.0/distro/target/apache-atlas-1.1.0-bin/apache-atlas-1.1.0`中执行。

```
cp server/webapp/atlas/WEB-INF/lib/jackson-jaxrs-base-2.9.6.jar hook/hive/atlas-hive-plugin-impl

cp server/webapp/atlas/WEB-INF/lib/jackson-jaxrs-json-provider-2.9.6.jar hook/hive/atlas-hive-plugin-impl

cp server/webapp/atlas/WEB-INF/lib/jackson-module-jaxb-annotations-2.9.6.jar hook/hive/atlas-hive-plugin-impl
```

补充好依赖jar包后，就可以执行导入hive数据到 atlas。

```
~/Downloads/apache-atlas-1.1.0/distro/target/apache-atlas-1.1.0-bin/apache-atlas-1.1.0 » bin/import-hive.sh   
Using Hive configuration directory [/Users/huzekang/cdh5.16/hive-1.1.0-cdh5.16.2/conf]
Log file for import is /Users/huzekang/Downloads/apache-atlas-1.1.0/distro/target/apache-atlas-1.1.0-bin/apache-atlas-1.1.0/logs/import-hive.log
log4j:WARN No such property [maxFileSize] in org.apache.log4j.PatternLayout.
log4j:WARN No such property [maxBackupIndex] in org.apache.log4j.PatternLayout.
Enter username for atlas :- admin
Enter password for atlas :-
Hive Meta Data imported successfully!!!
```

查看import-hive.log日志，可以看到查找到5个hive的数据库，并通过post的方式提交给atlas。

![image-20201206134747094](http://image-picgo.test.upcdn.net/img/20201206134747.png)

而且通过观察日志，可以发现atlas是通过`AtlasBaseClient`提交数据的。

```java
2020-12-06 13:47:00,905 INFO  - [main:] ~ method=POST path=api/atlas/v2/entity/ contentType=application/json; charset=UTF-8 accept=application/json status=200 (AtlasBaseClient:361)
2020-12-06 13:47:00,920 INFO  - [main:] ~ method=GET path=api/atlas/v2/entity/guid/ contentType=application/json; charset=UTF-8 accept=application/json status=200 (AtlasBaseClient:361)
2020-12-06 13:47:00,920 INFO  - [main:] ~ Created hive_table entity: name=hzk.member@primary, guid=db8deb41-f4b0-45c2-998f-c2d6c9ad9369 (HiveMetaStoreBridge:454)
```

此时，hive的全量数据已经导入。

![image-20201119164517081](http://image-picgo.test.upcdn.net/img/20201119164517.png)

### hive中配置post hook

实现创建hive表后能立刻更新到atlas中。

配置hive-env.sh。

```
export HIVE_AUX_JARS_PATH=/Users/huzekang/Downloads/apache-atlas-1.1.0/distro/target/apache-atlas-1.1.0-bin/apache-atlas-1.1.0/hook/hive/atlas-plugin-classloader-1.1.0.jar,/Users/huzekang/Downloads/apache-atlas-1.1.0/distro/target/apache-atlas-1.1.0-bin/apache-atlas-1.1.0/hook/hive/hive-bridge-shim-1.1.0.jar

```

配置hive-site.xml。

```
<property>
   <name>hive.exec.post.hooks</name>
   <value>org.apache.atlas.hive.hook.HiveHook</value>
</property>
```

