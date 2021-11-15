# IDEA中启动Atlas2.2.0调试

## 编译atlas源码

mac中使用命令进行编译

```
mvn clean -DskipTests package -Pdist -Drat.skip=true  
```

![image-20211115172329406](http://image-picgo.test.upcdn.net/img/20211115172329.png)



## idea中启动atlas

atlas的启动类是`org.apache.atlas.Atlas`。由于启动的时候需要加载`atlas-application.properties`文件，所以需要先准备好。

创建deploy目录，然后将负责的deploy目录下。

![image-20211115172917635](http://image-picgo.test.upcdn.net/img/20211115172917.png)

然后解压`apache-atlas-2.2.0-bin.tar.gz`文件。

![image-20211115173001576](http://image-picgo.test.upcdn.net/img/20211115173001.png)

解压后，需要修改deploy目录下的`apache-atlas-2.2.0/conf/atlas-application.properties`。

主要修改的内容如下：

```properties
# 指定berkeleyje作为后端存储
atlas.graph.storage.backend=berkeleyje
atlas.graph.storage.directory=/Users/huzekang/openSource/atlas/deploy/db

# 指定审计用内存
atlas.EntityAuditRepository.impl=org.apache.atlas.repository.audit.InMemoryEntityAuditRepository

# 指定索引后端用es
atlas.graph.index.search.backend=elasticsearch
atlas.graph.index.search.hostname=10.119.85.55

# 指定atlas启动时会在同一jvm中把zk和kafka启动（默认的，不用修改）
atlas.notification.embedded=true
atlas.kafka.data=${sys:atlas.home}/data/kafka
atlas.kafka.zookeeper.connect=localhost:9026
atlas.kafka.bootstrap.servers=localhost:9027
```



最后，改idea启动类的运行时参数。

![image-20211115173455824](http://image-picgo.test.upcdn.net/img/20211115173455.png)

```
-Datlas.conf=/Users/huzekang/openSource/atlas/deploy/apache-atlas-2.2.0/conf
-Datlas.home=/Users/huzekang/openSource/atlas/deploy/apache-atlas-2.2.0
```



## 启动成功后

可以看到es中的数据

![image-20211115175031036](http://image-picgo.test.upcdn.net/img/20211115175031.png)

可以看到mac的磁盘中的berkeleyje的数据

![image-20211115175113789](http://image-picgo.test.upcdn.net/img/20211115175113.png)

可以看到内嵌kafka和zk的数据

![image-20211115175103995](http://image-picgo.test.upcdn.net/img/20211115175104.png)