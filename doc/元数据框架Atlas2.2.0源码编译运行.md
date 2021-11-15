## 元数据框架Atlas2.2.0源码编译运行

## 组件版本

atlas2.2.0

hbase2.3.3【试过用hbase1.x的版本，会报错，没仔细定位问题】

zookeeper3.6.3【应该不一定这个版本也可以】

Elasticserach7.5.1【没试过其他版本】

## mac上编译

```
 mvn clean -DskipTests package -Pdist
```



如果遇到错误:`Too many files with unapproved license`，可以加入参数`-Drat.skip=true`

## centos编译源码

```
 mvn clean -DskipTests package -Pdist
```

注意这里不要选内置的hbase和solr，因为它会下载hbase和solr，都比较慢。

注意要在centos系统中用root用户编译。

### 启动依赖组件顺序

### 外部启动zk

要在zk的conf目录下创建zoo.cfg文件

```properties
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=/tmp/zookeeper
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1
admin.serverPort=8888

```



### 外部启动hbase

hbase-site.xml如下配置

```xml
 <property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
  </property>
  <property>
    <name>hbase.tmp.dir</name>
    <value>./tmp</value>
  </property>
  <property>
    <name>hbase.unsafe.stream.capability.enforce</name>
    <value>false</value>
  </property>
<property>
   <name>hbase.zookeeper.quorum</name>
   <value>localhost</value>
</property>

<property>
   <name>hbase.zookeeper.property.clientPort</name>
   <value>2181</value>
</property>

```

启动后可以看到这些hbase master和hbase regionserver都在同一台服务器中。

![image-20211111124448290](http://image-picgo.test.upcdn.net/img/20211111124448.png)

### 外部启动es

由于我的es和atlas不在同一台服务器上，所以atlas需要能远程访问es，因此需要ES配置如下。

config/elasticsearch.yml

```
http.host=0.0.0.0
```



### 更改atlas配置

atlas-application.properties。下面指贴需要调整的重要配置

```properties
atlas.graph.storage.backend=hbase2
atlas.graph.storage.hbase.table=apache_atlas_janus2

atlas.graph.storage.hostname=localhost:2181


atlas.graph.index.search.backend=elasticsearch
atlas.graph.index.search.hostname=10.211.55.2

atlas.audit.hbase.tablename=apache_atlas_entity_audit
atlas.audit.hbase.zookeeper.quorum=localhost:2181
```

atlas-env.sh

```properties
# indicates whether or not a local instance of HBase should be started for Atlas
export MANAGE_LOCAL_HBASE=false

# indicates whether or not a local instance of Solr should be started for Atlas
export MANAGE_LOCAL_SOLR=false

# indicates whether or not cassandra is the embedded backend for Atlas
export MANAGE_EMBEDDED_CASSANDRA=false

# indicates whether or not a local instance of Elasticsearch should be started for Atlas
export MANAGE_LOCAL_ELASTICSEARCH=false
```





### 启动atlas

```
export HBASE_CONF_DIR=/home/huzekang/software/hbase-2.3.3/conf
export MANAGE_LOCAL_HBASE=false
export MANAGE_LOCAL_SOLR=false
bin/atlas_start.py
```

启动成功后，访问21000端口。

![image-20211111125157411](http://image-picgo.test.upcdn.net/img/20211111125157.png)

启动成功后，hbase会自动建两个表

```
atlas.graph.storage.hbase.table=apache_atlas_janus2
atlas.audit.hbase.tablename=apache_atlas_entity_audit
```

es会会自动建三个索引

```
janusgraph_vertex_index
janusgraph_fulltext_index
janusgraph_edge_index
```

