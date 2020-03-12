# HDP后续调优

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




### spark使用的用户无法访问hive建的表数据

```
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



## hbase

### hdp默认的hbase在zookeeper的目录是hbase-unsecure

                           ```
 "zookeeper.znode.parent": "/hbase-unsecure"
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



