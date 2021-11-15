# Atlas2.2.0用berkeleyje加Es做存储

## 准备

需要提前编译好atlas二进制包，然后进行解压。

## 组件版本

Atlas2.2.0

berkeleyje【je-18.3.12】

Elasticserach7.5.1

## 放入berkeleyje依赖

需要手动放入依赖包【je-18.3.12.jar】到lib中

![image-20211115170344345](http://image-picgo.test.upcdn.net/img/20211115170344.png)

如果不放的话，启动时会报下面错误

```
java.lang.ClassNotFoundException: com.sleepycat.je.DatabaseException

```

![image-20211115171146272](http://image-picgo.test.upcdn.net/img/20211115171146.png)

## 调整配置

主要修改文件`atlas-application.properties`下面配置

```properties
atlas.graph.storage.backend=berkeleyje
atlas.graph.storage.directory=/tmp/janusgraph_db

atlas.EntityAuditRepository.impl=org.apache.atlas.repository.audit.InMemoryEntityAuditRepository

atlas.graph.index.search.backend=elasticsearch
atlas.graph.index.search.hostname=10.119.85.55
```

## 启动atlas

```
bin/atlas_start.py
```

启动后，可以看到es中的数据如下：

![image-20211115171107884](http://image-picgo.test.upcdn.net/img/20211115171107.png)

启动后，可以看到服务器磁盘中生成了数据：

![image-20211115170827235](http://image-picgo.test.upcdn.net/img/20211115170827.png)