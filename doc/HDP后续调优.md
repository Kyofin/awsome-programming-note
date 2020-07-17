# HDP后续调优

## 常用目录

各个组件安装目录：/usr/hdp/current/ 

各组件日志存放：/var/log 

各组件配置：/etc/hadoop/conf 

 phoenix query server日志存放地址 ：/var/log/hbase

hbase region server日志存放地址 ：/var/log/hbase

```
tail -f hbase-hbase-regionserver-cdh05.log
```



## 大目录文件调整软连接

### ambari-infra-solr

ambari内置的日志是用solr建立索引的，而它的数据文件在`/var/lib/ambari-infra-solr`,固会占用过多空间，需要建立软连接。

停止ambari所有服务和agent

复制原来存在的数据文件到/home目录

```
cp -r ambari-infra-solr/ /home/
cd /home
mkdir ambari_data
mv ambari-infra-solr/ ambari_data/
```

删除原来文件

```
rm -fr /var/lib/ambari-infra-solr/
```

建立软连接

```
ln -s /home/ambari_data/ambari-infra-solr/ /var/lib/ambari-infra-solr
```





### /usr/hdp

```
cp -r /usr/hdp/ /home/
cd /home/
mkdir hdp_data
mv hdp/ hdp_data/
```

删除原来文件

```
rm -rf /usr/hdp/
```

建立软连接

```
ln -s /home/hdp_data/hdp/ /usr/hdp
```





### /var/log

```
cp -r /var/log /home/
cd /home/
mkdir log_data
mv log/ log_data/
```

删除原来文件

```
rm -rf  /var/log
```

建立软连接

```
ln -s /home/log_data/log/ /var/log
```



## Hive

-  orc大表在查询`select count(1) from table`时查询结果为0，但是是有数据的。

  ![](http://image-picgo.test.upcdn.net/img/20200427155825.png)





## Yarn

yarn配置调优，可以参考[Determine HDP Memory Configuration Settings](https://docs.cloudera.com/HDPDocuments/HDP2/HDP-2.1.1/bk_installing_manually_book/content/rpm-chap1-11.html)

1. 下载测试脚本

   ```shell
   wget http://public-repo-1.hortonworks.com/HDP/tools/2.0.6.0/hdp_manual_install_rpm_helper_files-2.0.6.101.tar.gz 
   ```

2. 解压压缩包，进入目录`script`

3. 使用命令

   ```
   python yarn-utils.py -c 8 -m 32 -d 4 -k False
   ```

   With the following options:

   | **Option** | **Description**                               |
   | ---------- | --------------------------------------------- |
   | -c CORES   | The number of cores on each host.             |
   | -m MEMORY  | The amount of memory on each host in GB.      |
   | -d DISKS   | The number of disks on each host.             |
   | -k HBASE   | "True" if HBase is installed, "False" if not. |

4. 可以看到计算出的最佳设置

   ![image-20200611153413131](http://image-picgo.test.upcdn.net/img/20200611153413.png)



## spark

### spark无法查询hive命令行建的表

通过Ambari2.7安装好HDP3.1后，发现在spark-sql中无法读到hive命令行创建的数据库和表。

后来查了网上资料，发现hive 3.0之后默认开启ACID功能，而且新建的表默认是ACID表。而spark目前还不支持hive的ACID功能，因此无法读取ACID表的数据。

1. 在ambari页面中对hive服务修改配置

   ```
   hive.strict.managed.tables=false
   hive.create.as.insert.only=false
   metastore.create.as.acid=false
   ```

   重启hive。

2. 在ambari页面中对spark2服务修改配置

   `metastore.catalog.default`配置为：Hive

   这个选项默认为Spark， 即读取SparkSQL自己的metastore_db，修改完后重启spark服务。

   Spark Shell会去读取Hive的metastore，这样就可以实现以Spark Shell方式访问Hive SQL方式创建的databases/tables。

   

3. 在hive中建表时使用下面格式

   ```
   create table xxx.***(....) stored as orc TBLPROPERTIES('transactional'='false')
   ```



### spark应用查不了hive中的表

![img](https://img2018.cnblogs.com/blog/1018938/201904/1018938-20190420171931251-1256469962.png)我在 Spark-shell 里创建了一张 hive 表，发现其创建的位置是 spark.sql.warehouse.dir 指向的目录。

解决问题参考：

https://www.cnblogs.com/langfanyun/p/10741775.html



### spark使用的用户无法访问hive建的表数据

```ruby
Caused by: org.apache.hadoop.ipc.RemoteException(org.apache.hadoop.security.AccessControlException): Permission denied: user=root, access=WRITE, inode="/user":hdfs:supergroup:drwxr-xr-x
	at org.apache.hadoop.hdfs.server.namenode.DefaultAuthorizationProvider.checkFsPermission(DefaultAuthorizationProvider.java:279)
	at org.apache.hadoop.hdfs.server.namenode.DefaultAuthorizationProvider.check(DefaultAuthorizationProvider.java:260)
	at org.apache.hadoop.hdfs.server.namenode.DefaultAuthorizationProvider.check(DefaultAuthorizationProvider.java:240)
	at org.apache.hadoop.hdfs.server.namenode.DefaultAuthorizationProvider.checkPermission(DefaultAuthorizationProvider.java:162)
	at org.apache.hadoop.hdfs.server.namenode.FSPermissionChecker.checkPermission(FSPermissionChecker.java:152)
	at org.apache.hadoop.hdfs.server.namenode.FSDirectory.checkPermission(FSDirectory.java:3529)
	at org.apache.hadoop.hdfs.server.namenode.FSDirectory.checkPermission(FSDirectory.java:3512)
	at org.apache.hadoop.hdfs.server.namenode.FSDirectory.checkAncestorAccess(FSDirectory.java:3494)
	at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.checkAncestorAccess(FSNamesystem.java:6599)

```

关闭hdfs权限就可以了。

![](http://image-picgo.test.upcdn.net/img/20200116140436.png)





### Spark thrift支持查询mysql表

ambari加配置声明启动spark thrfit时的命令。

如下：

![](http://image-picgo.test.upcdn.net/img/20200515102117.png)

```shell
 --jars hdfs:///jdbc/mysql-connector-java-8.0.11.jar  --driver-class-path hdfs:///jdbc/mysql-connector-java-8.0.11.jar
```

然后就可以在客户端工具查询mysql表了。

```SQL
CREATE TEMPORARY TABLE dept 
USING org.apache.spark.sql.jdbc 
OPTIONS(
url 'jdbc:mysql://192.168.1.130:3306/yiboard', 
driver 'com.mysql.jdbc.Driver', 
dbtable 'chart', 
user 'root', 
password 'root',
fetchSize '1000');
SELECT * from dept;
```

![](http://image-picgo.test.upcdn.net/img/20200515102311.png)

如果想支持ck，sqlserver，pg，oracle的话。

可以参考以下：

先上传需要的驱动到hdfs。

![image-20200601184523625](http://image-picgo.test.upcdn.net/img/20200601184523.png)

测试

```SQL
 CREATE TEMPORARY TABLE catalog_page_mysql USING org.apache.spark.sql.jdbc OPTIONS (driver "com.mysql.jdbc.Driver" ,url "jdbc:mysql://192.168.1.130:3306/tpcds_200?user=root&password=root&rewriteBatchedStatements=true", dbtable "catalog_page");


CREATE TEMPORARY TABLE titles_pg USING org.apache.spark.sql.jdbc OPTIONS (driver "org.postgresql.Driver", url "jdbc:postgresql://192.168.1.150:5432/employees?user=postgres&password=postgres&rewriteBatchedStatements=true", dbtable "titles");

CREATE TEMPORARY TABLE catalog_page_ck USING org.apache.spark.sql.jdbc OPTIONS (driver "ru.yandex.clickhouse.ClickHouseDriver" ,url "jdbc:clickhouse://cdh06:8123/tpcds_200?user=default&password=123&rewriteBatchedStatements=true", dbtable "catalog_page");


select * from catalog_page_mysql;
select * from titles_pg;
select * from catalog_page_ck;

DROP view catalog_page_mysql ;

```



### 部署apache版本spark2.4.4客户端到hdp

1. 将hdp集群中原有spark中的conf目录下的hive-stie.xml复制到新的spark的conf目录下

   ![image-20200717104903690](http://image-picgo.test.upcdn.net/img/20200717104903.png)

   这样就可以访问hdp集群中hive的数据了。

2. 将hdp集群中原有spark中的conf目录下的spark-env.sh复制到新的spark的conf目录下

   修改部分配置。

   下面为参考

   ```shell
   
   #!/usr/bin/env bash
   
   export SPARK_HOME=/opt/spark-2.4.4-bin-hadoop2.6
   
   # Generic options for the daemons used in the standalone deploy mode
   
   # Alternate conf dir. (Default: ${SPARK_HOME}/conf)
   export SPARK_CONF_DIR=${SPARK_HOME}/conf
   
   # Where log files are stored.(Default:${SPARK_HOME}/logs)
   #export SPARK_LOG_DIR=${SPARK_HOME:-/usr/hdp/current/spark2-thriftserver}/logs
   export SPARK_LOG_DIR=${SPARK_HOME}/logs
   
   # Where the pid file is stored. (Default: /tmp)
   export SPARK_PID_DIR=${SPARK_HOME}/pid
   
   #Memory for Master, Worker and history server (default: 1024MB)
   export SPARK_DAEMON_MEMORY=2048m
   
   # A string representing this instance of spark.(Default: $USER)
   SPARK_IDENT_STRING=$USER
   
   # The scheduling priority for daemons. (Default: 0)
   SPARK_NICENESS=0
   
   export HADOOP_HOME=${HADOOP_HOME:-/usr/hdp/3.1.0.0-78/hadoop}
   export HADOOP_CONF_DIR=${HADOOP_CONF_DIR:-/usr/hdp/3.1.0.0-78/hadoop/conf}
   
   # The java implementation to use.
   export JAVA_HOME=/usr/java/default
   ```

   关键是SPARK_HOME、HADOOP_HOME、HADOOP_CONF_DIR、JAVA_HOME

3. 在新spark的conf目录下创建文件`spark-defaults.conf`文件

   ```properties
   #
   # Licensed to the Apache Software Foundation (ASF) under one or more
   # contributor license agreements.  See the NOTICE file distributed with
   # this work for additional information regarding copyright ownership.
   # The ASF licenses this file to You under the Apache License, Version 2.0
   # (the "License"); you may not use this file except in compliance with
   # the License.  You may obtain a copy of the License at
   #
   #    http://www.apache.org/licenses/LICENSE-2.0
   #
   # Unless required by applicable law or agreed to in writing, software
   # distributed under the License is distributed on an "AS IS" BASIS,
   # WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   # See the License for the specific language governing permissions and
   # limitations under the License.
   #
   
   # Default system properties included when running spark-submit.
   # This is useful for setting default environmental settings.
   
   # Example:
   # spark.master                     spark://master:7077
   # spark.eventLog.enabled           true
   # spark.eventLog.dir               hdfs://namenode:8021/directory
   # spark.serializer                 org.apache.spark.serializer.KryoSerializer
   # spark.driver.memory              5g
   # spark.executor.extraJavaOptions  -XX:+PrintGCDetails -Dkey=value -Dnumbers="one two three"
   
   
   spark.driver.extraJavaOptions -Dhdp.version=3.1.0.0-78
   spark.yarn.am.extraJavaOptions -Dhdp.version=3.1.0.0-78
   
   spark.master yarn-client
   
   ```

   关键是-Dhdp.version=3.1.0.0-78。如果不配，则无法使用yarn client模式。

4. 在ambari中设置yarn

   设置`spark.hadoop.yarn.timeline-service.enabled=false`

   ![image-20200717105900362](http://image-picgo.test.upcdn.net/img/20200717105900.png)

   然后在ambari中重启yarn





## hbase

### hbase结构

![img](http://www.raincent.com/uploadfile/2019/0227/20190227014751823.jpg)

HBase**每张表**在底层存储上是由至少一个Region组成。

HBase新建一张表时默认Region即分区的数量为1。



### 默认要修改的配置

##### 1 ）hdp默认的hbase在zookeeper的目录是hbase-unsecure，这个不用调，客户端连接的时候注意一下就好。

```
 "zookeeper.znode.parent": "/hbase-unsecure"
```

##### 2 ）hbase默认只启动一个master，容易单点故障，所以应该在hbase master的其他节点上开多一个master 进行standby。

##### 3 ）调大hbase  regionserver的最大内存。

##### 4 ）调大zk的连接地址。

##### 5 ）开启phoenix query server。调整phoenix超时时间为3分钟。



### Region per regionserver过多问题

参考案例：[20000个分区导致HBase集群宕机事故处理](https://mp.weixin.qq.com/s?__biz=MzUxOTU5Mjk2OA==&mid=2247484038&idx=1&sn=8f20834ced315c0c0911ed63693f7a6c&chksm=f9f60fe1ce8186f703b8c6bdeefaf2b66888c59830b207e2e9f2914f501b1941173007d91024&scene=21#wechat_redirect)

通常情况下，生产环境的每个regionserver节点上会有很多Region存在，我们一般比较关心每个节点上的Region数量，主要为了防止HBase分区过多影响到集群的稳定性。

#### 频繁刷写

我们知道Region的一个列族对应一个写**缓存MemStore**，假设HBase表都有统一的1个列族配置，则每个Region只包含一个**缓存MemStore**。通常HBase的一个**缓存MemStore**默认大小为128 MB，见参数hbase.hregion.memstore.flush.size。当可用内存足够时，每个MemStore可以分配128 MB空间。当可用内存紧张时，假设每个Region写入压力相同，则理论上每个**缓存MemStore**会平均分配可用内存空间。

因此，当节点Region过多时，每个**缓存MemStore**分到的内存空间就会很小。这个时候，写入很小的数据量就会被强制Flush到磁盘，将会导致频繁刷写。频繁刷写磁盘，会对集群HBase与HDFS造成很大的压力，可能会导致不可预期的严重后果。

#### 压缩风暴

因Region过多导致的频繁刷写，将在磁盘上产生非常多的HFile小文件，当小文件过多的时候HBase为了优化查询性能就会做Compaction操作，合并HFile减少文件数量。当小文件一直很多的时候，就会出现 "压缩风暴"。Compaction非常消耗系统io资源，还会降低数据写入的速度，严重的会影响正常业务的进行。

#### MSLAB内存消耗较大

MSLAB（MemStore-local allocation buffer）存在于每个MemStore中，主要是为了解决HBase内存碎片问题，默认会分配 2 MB 的空间用于缓存最新数据。如果Region数量过多，MSLAB总的空间占用就会比较大。比如当前节点有1000个包含1个列族的Region，MSLAB就会使用1.95GB的堆内存，即使没有数据写入也会消耗这么多内存。

#### Master assign region时间较长

HBase Region过多时Master分配Region的时间将会很长。特别体现在重启HBase时Region上线时间较长，严重的会达到小时级，造成业务长时间等待的后果。

#### 影响MapReduce并发数

当使用MapReduce操作HBase时，通常Region数量就是MapReduce的任务数，Region数量过多会导致并发数过多，产生过多的任务。任务太多将会占用大量资源，当操作包含很多Region的大表时，占用过多资源会影响其他任务的执行。

#### 计算集群region数量的公式：

((RS Xmx) * hbase.regionserver.global.memstore.size) / (hbase.hregion.memstore.flush.size * (# column families))

其中：

- RS memory：表示regionserver堆内存大小，即HBASE_HEAPSIZE。
- total memstore fraction：表示所有MemStore占HBASE_HEAPSIZE的比例，HBase0.98版本以后由hbase.regionserver.global.memstore.size参数控制，老版本由hbase.regionserver.global.memstore.upperLimit参数控制，默认值0.4。
- memstore size：即每个MemStore的大小，原生HBase中默认128M。
- column families：即表的列族数量，通常情况下只设置1个，最多不超过3个。

**假设一个RS有16GB内存，那么16384*0.4/128m 等于51个活跃的region。**

如果写很重的场景下，可以适当调高hbase.regionserver.global.memstore.size，这样可以容纳更多的region数量。

**总结，建议分配合理的region数量，根据写请求量的情况，一般20-200个之间，可以提高集群稳定性，排除很多不确定的因素，提升读写性能。监控Region Server中所有Memstore的大小总和是否达到了上限（hbase.regionserver.global.memstore.upperLimit ＊ hbase_heapsize，默认 40%的JVM内存使用量），超过可能会导致不良后果，如服务器反应迟钝或compact风暴。**





### rit一直都是24问题



### region server 的cup达到200+问题

检查是不是rit(REGIONS IN TRANSITION)的数量很高。看看是不是在压缩。

检查regionserver日志文件，看看是不是连不上zk。如果一直显示如下错误，重启zk即可。

![](http://image-picgo.test.upcdn.net/img/20200314185916.png)



### regionserver percent file local 过多问题

这个好像是datanode和regionserver放一起就会有，值越高代表本地化越高，好像没什么问题。