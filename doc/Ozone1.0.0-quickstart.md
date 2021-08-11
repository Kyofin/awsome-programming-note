# Ozone1.0.0-quickstart

## 架构图

Ozone由三部分组成，分别为`Ozone Manager（OM）`、`Storage Container Manager（SCM）`和`Datanodes（DN）`

![img](http://image-picgo.test.upcdn.net/img/20210804171904.png)

## 资源文件准备

ozone1.0.0二进制部署包

https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/ozone/ozone-1.0.0/hadoop-ozone-1.0.0.tar.gz

hadoop2兼容ozone的jar包

https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-ozone-filesystem-hadoop2/1.0.0/hadoop-ozone-filesystem-hadoop2-1.0.0.jar





## 单节点手动部署Ozone

**注意**：当系统环境变量里有HADOOP_HOME时，需要先注释掉。因为在很多ozone自带的shell脚本过程中都在`HADOOP_HOME`目录下寻找ozone的文件，就会报找不到的错误。
错误如下：

![image-20210805105954808](http://image-picgo.test.upcdn.net/img/20210805105954.png)

解决的办法：

执行ozone的脚本前，输入以下命令。

```
export HADOOP_HOME=
```





### 修改配置

生成默认ozone配置到当前目录

```
bin/ozone genconf .
```

修改ozone-site.xml

- ozone metadata 存储目录:

```XML
<property>
	<name>ozone.metadata.dirs</name>
	<value>/Users/huzekang/opt/ozone-1.0.0/metadata</value>
</property>
```

- datanode 存储目录

```XML

<property>
    <name>dfs.datanode.data.dir</name>
    <value>file:///Users/huzekang/opt/ozone-1.0.0/data</value>
</property>
```

- 单副本

```XML
     <property>
 <name>ozone.replication</name>
 <value>1</value>
 </property>
```

- om address

```XML
 <property>
        <name>ozone.om.address</name>
        <value>ozone1:9862</value>
    </property>
```

- 配置datanode的hostname

该配置会影响dn启动后注册到scm中的DatanodeDetail信息。默认是localhost，会导致spark程序远程连接ozone的datanode取数时报无法连接`localhost:9859`的错误。

```XML
<property>
    <name>dfs.datanode.hostname</name>
    <value>ozone1</value>
</property>
<property>
    <name>dfs.datanode.use.datanode.hostname</name>
    <value>true</value>
</property>
```



- 开启prometheus监控

开启后，访问下面url即可看到监控数据。

http://scm:9874/prom

http://ozoneManager:9876/prom

```XML
<property>
   <name>hdds.prometheus.endpoint.enabled</name>
   <value>true</value>
 </property>
```

- 客户端使用访问 SCM 服务地址和端口

```XML
<property>
        <name>ozone.scm.client.address</name>
        <value>ozone1</value>
       
    </property>
```

- 为datanode指定scm的地址

可以用逗号分隔。Ozone 目前尚未支持 SCM 的 HA，ozone.scm.names 只需配置单个 SCM 地址即可。

例如：scm2:8020

```xml
 <property>
        <name>ozone.scm.names</name>
        <value>ozone1</value>
    </property>
```



拷贝配置文件到ozone根目录下的 `etc/hadoop/`，使配置生效：

```
~/opt/ozone-1.0.0 » cp ozone-site.xml etc/hadoop/
```

### 创建元数据和数据的存储目录

创建存储数据的目录：

```
~/opt/ozone-1.0.0 » mkdir metadata data
```

### 启动ozone

初始化 scm,该命令用于初始化 scm metadata磁盘数据目录

```
bin/ozone scm --init
```

启动scm

```
bin/ozone --daemon start scm
```

初始化om,该命令用于初始化 om metadata磁盘数据目录

```
bin/ozone  om --init
```

启动om

```
bin/ozone --daemon start om
```

启动ozone datanode

```
bin/ozone --daemon start datanode
```

都去启动成功后，可以看到进程如下。

![image-20210803162701837](http://image-picgo.test.upcdn.net/img/20210803162701.png)

如果想启动s3 gateway 服务，需要手动启动s3g。启动成功后，可以访问`http://localhost:9878/`

```
 bin/ozone --daemon start s3g
```

如果要快速关闭ozone所有组件，则执行下面命令

```
sbin/stop-ozone.sh
```

如果要快速启动ozone所有组件，则执行下面命令

```
sbin/start-ozone.sh
```





## ozone端口说明

| 服务     | 端口 | 参数                        | 备注                                                         |
| -------- | ---- | --------------------------- | ------------------------------------------------------------ |
| OM       | 9862 | ozone.om.address            |
| OM       | 9874 | ozone.om.http-address       | OM web页面                                                   |
| Datanode | 9858 | dfs.container.ratis.ipc     | Datanode节点                                                 |
| Datanode | 9859 | dfs.container.ipc           | Datanode节点                                                 |
| Datanode | 9882 | hdds.datanode.http-address  | Datanode节点                                                 |
| SCM      | 9860 | ozone.scm.client.address    |                                                              |
| SCM      | 9861 | ozone.scm.datanode.address  | scm和datanode通信端口                                        |
| SCM      | 9863 | ozone.scm.block.client.port |                                                              |
| SCM      | 9876 | ozone.scm.http-address      | SCM web页面                                                  |
| S3G      | 9878 | ozone.s3g.http-address      | S3Gateway                                                    |
| Recon    | 9888 | ozone.recon.http-address    | Recon web页面                                                |
| Recon    | 9891 | ozone.recon.address         | 默认无，Datanode节点必配参数，以便Recon可以监测到Datanode服务 |



## ozone默认ui

### Ozone manager UI

默认端口：9874

![image-20210803171428027](http://image-picgo.test.upcdn.net/img/20210803171428.png)



### Storage Container Manager UI

默认端口：9876

![image-20210803171506944](http://image-picgo.test.upcdn.net/img/20210803171507.png)

### Recon UI

![image-20210805143655223](http://image-picgo.test.upcdn.net/img/20210805143655.png)



### S3 gateway UI

默认端口：9878

![image-20210803172819041](http://image-picgo.test.upcdn.net/img/20210803172819.png)

### HDDS Datanode UI

默认端口：9882

![image-20210803173906201](http://image-picgo.test.upcdn.net/img/20210803173906.png)



## 常用aws cli命令

aws s3api --endpoint http://localhost:9878/ create-bucket --bucket=wordcount

aws s3api --endpoint http://localhost:9878/  list-buckets

aws s3api --endpoint http://localhost:9878/  put-object --bucket=wordcount  --storage-class REDUCED_REDUNDANCY --key Doc1 --body  ~/table1_data

aws s3api --endpoint http://localhost:9878/  list-objects --bucket=wordcount

aws s3api --endpoint http://localhost:9878/  get-object --bucket=wordcount --key Doc1 ~/testfile2 

aws s3 --endpoint http://localhost:9878  ls  s3://wordcount/Doc1

aws s3 --endpoint http://localhost:9878  rm  s3://wordcount/Doc1

 aws s3api --endpoint http://localhost:9878/  delete-object --key Doc1 --bucket=test



## 常用ozone cli命令

### 创建 volume

```
ozone sh volume create  --quota=1TB  /hive
ozone sh volume create /volume
ozone sh volume list
```

### 创建 bucket

```
ozone sh bucket create /hive/dc2
ozone sh bucket create /volume/bucket
ozone sh bucket list /hive
```

### 写入并读取文件，比对

```
ozone sh key put --replication=ONE /hive/dc2/README.md README.md
ozone sh key list /hive/dc2
ozone sh key get /hive/dc2/README.md README2.txt
diff README2.txt README.md

```

### 查看数据

```
ozone sh key cat /volume/bucket/README.md
```

### 使用测试工具

```
ozone freon rk --numOfVolumes=10 --numOfBuckets=10 --numOfKeys=100    --factor=THREE      --replicationType=RATIS
```

### 查看scm网络拓扑

```
bin/ozone admin  printTopology
```

![image-20210811164614887](http://image-picgo.test.upcdn.net/img/20210811164615.png)

### 查看pipeline

```
bin/ozone admin  pipeline list
```

![image-20210811164649901](http://image-picgo.test.upcdn.net/img/20210811164649.png)



## 常见QA

### volume的配额不足，datanode报错。

```
2021-08-11 11:36:07,183 [ChunkWriter-9-0] INFO org.apache.hadoop.ozone.container.keyvalue.KeyValueHandler: Operation: CreateContainer , Trace ID:  , Message: Container creation failed, due to disk out of space , Result: DISK_OUT_OF_SPACE , StorageContainerException Occurred.
org.apache.hadoop.hdds.scm.container.common.helpers.StorageContainerException: Container creation failed, due to disk out of space
	at org.apache.hadoop.ozone.container.keyvalue.KeyValueContainer.create(KeyValueContainer.java:147)
	at org.apache.hadoop.ozone.container.keyvalue.KeyValueHandler.handleCreateContainer(KeyValueHandler.java:250)
	at org.apache.hadoop.ozone.container.keyvalue.KeyValueHandler.dispatchRequest(KeyValueHandler.java:167)
	at org.apache.hadoop.ozone.container.keyvalue.KeyValueHandler.handle(KeyValueHandler.java:155)
	at org.apache.hadoop.ozone.container.common.impl.HddsDispatcher.createContainer(HddsDispatcher.java:419)
	at org.apache.hadoop.ozone.container.common.impl.HddsDispatcher.dispatchRequest(HddsDispatcher.java:255)
	at org.apache.hadoop.ozone.container.common.impl.HddsDispatcher.dispatch(HddsDispatcher.java:166)
	at org.apache.hadoop.ozone.container.common.transport.server.ratis.ContainerStateMachine.dispatchCommand(ContainerStateMachine.java:400)
	at org.apache.hadoop.ozone.container.common.transport.server.ratis.ContainerStateMachine.runCommand(ContainerStateMachine.java:410)
	at org.apache.hadoop.ozone.container.common.transport.server.ratis.ContainerStateMachine.lambda$handleWriteChunk$2(ContainerStateMachine.java:448)
	at java.util.concurrent.CompletableFuture$AsyncSupply.run(CompletableFuture.java:1590)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
Caused by: org.apache.hadoop.util.DiskChecker$DiskOutOfSpaceException: Out of space: The volume with the most available space (=5049147137 B) is less than the container size (=5368709120 B).
	at org.apache.hadoop.ozone.container.common.volume.RoundRobinVolumeChoosingPolicy.chooseVolume(RoundRobinVolumeChoosingPolicy.java:77)
	at org.apache.hadoop.ozone.container.keyvalue.KeyValueContainer.create(KeyValueContainer.java:110)
	... 13 more
2021-08-11 11:36:07,201 [ChunkWriter-9-0] INFO org.apache.hadoop.ozone.container.common.impl.HddsDispatcher: Operation: WriteChunk , Trace ID:  , Message: ContainerID 2 creation failed , Result: DISK_OUT_OF_SPACE , StorageContainerException Occurred.
```

解决办法：

更新volume的配额。

```
bin/ozone sh volume update  --quota=1TB  /hive
```

