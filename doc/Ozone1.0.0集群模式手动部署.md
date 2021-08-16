# Ozone1.0.0集群模式手动部署

### 网络拓扑

ozone001部署om、scm、dn

ozone002部署dn

ozone003部署dn



### 环境依赖

jdk1.8



### 设置每台etc/hosts

```
172.22.66.220 ozone003
172.22.66.218 ozone002
172.22.66.219 ozone001
```



### 创建linuex用户

由于ozone的datanode启动不能用root用户。

```
useradd -m huzekang
passwd huzekang
```

设置ssh密码。



### 主节点打通每台服务器ssh免密

主节点上生成自己的公钥和秘钥

```
ssh-keygen -b 4096 -t rsa -C gp002
```

然后复制到所有服务器

```
ssh-copy-id -i ~/.ssh/id_rsa.pub -p 22 -f huzekang@ozone003
ssh-copy-id -i ~/.ssh/id_rsa.pub -p 22 -f huzekang@ozone002
ssh-copy-id -i ~/.ssh/id_rsa.pub -p 22 -f huzekang@ozone001
```





### 上传ozone并解压

```
cd /opt/
tar zxvf hadoop-ozone-1.0.0.tar.gz 
```



### 在主节点设置ozone-site.xml的配置文件

```XML
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<configuration>
    <property>
        <name>ozone.om.address</name>
        <value>ozone001</value>
        <tag>OM, REQUIRED</tag>
        <description>
      The address of the Ozone OM service. This allows clients to discover
      the address of the OM.
    </description>
    </property>
    <property>
        <name>ozone.metadata.dirs</name>
        <value>/opt/ozone-1.0.0/metadata</value>
        <tag>OZONE, OM, SCM, CONTAINER, STORAGE, REQUIRED</tag>
        <description>
      This setting is the fallback location for SCM, OM, Recon and DataNodes
      to store their metadata. This setting may be used only in test/PoC
      clusters to simplify configuration.

      For production clusters or any time you care about performance, it is
      recommended that ozone.om.db.dirs, ozone.scm.db.dirs and
      dfs.container.ratis.datanode.storage.dir be configured separately.
    </description>
    </property>

<property>
    <name>dfs.datanode.data.dir</name>
    <value>file:///opt/ozone-1.0.0/data</value>
</property>

    <property>
        <name>ozone.scm.client.address</name>
        <value>ozone001</value>
        <tag>OZONE, SCM, REQUIRED</tag>
        <description>
      The address of the Ozone SCM client service. This is a required setting.

      It is a string in the host:port format. The port number is optional
      and defaults to 9860.
    </description>
    </property>
    <property>
        <name>ozone.scm.names</name>
        <value>ozone001</value>
        <tag>OZONE, REQUIRED</tag>
        <description>
      The value of this property is a set of DNS | DNS:PORT | IP
      Address | IP:PORT. Written as a comma separated string. e.g. scm1,
      scm2:8020, 7.7.7.7:7777.
      This property allows datanodes to discover where SCM is, so that
      datanodes can send heartbeat to SCM.
    </description>
    </property>
</configuration>

```



### 在主节点设置workers配置文件

```
ozone001
ozone002
ozone003

```



### 在主节点设置hadoop-env.sh配置文件

加入以下配置。

```
export JAVA_HOME=/opt/jdk1.8.0_181
```





### 在主节点拷贝ozone安装目录到两个子节点

```
scp -r /opt/ozone-1.0.0/ ozone003:/opt/
scp -r /opt/ozone-1.0.0/ ozone001:/opt/
```





### 主节点启动ozone

scm初始化

```
ozone scm --init
```

启动scm

```
ozone --daemon start scm

```

om初始化

```
ozone om --init
```

启动om

```
ozone --daemon start om
```

启动ozone全部服务。该命令会ssh到各节点后启动dn，然后在本节点上启动scm和om。

```
sbin/start-ozone.sh
```



### 验证启动是否成功

使用jps观察ozone001

![image-20210812163938492](http://image-picgo.test.upcdn.net/img/20210812163938.png)

使用jps观察ozone002

![image-20210812163952500](http://image-picgo.test.upcdn.net/img/20210812163952.png)

使用jps观察ozone003

![image-20210812164011414](http://image-picgo.test.upcdn.net/img/20210812164011.png)

可以看到各组件都启动成功。

打开浏览器观察scm也已经成功。

![image-20210812164051920](/Users/huzekang/Library/Application Support/typora-user-images/image-20210812164051920.png)

使用ozone自带的工具生成测试数据

```
ozone freon rk --numOfVolumes=10 --numOfBuckets=10 --numOfKeys=100    --factor=THREE      --replicationType=RATIS
```

可以观察到文件都写入完毕。

![image-20210812164452116](http://image-picgo.test.upcdn.net/img/20210812164452.png)







### 集群启动依赖配置项的设置

------

下面是集群启动依赖的配置项的设置，配置项如果设的不对，集群相关服务启动会出现一定的问题。在这里会涉及到3个服务角色，它们之间还有一定的依赖关系：

- SCM服务，不需要依赖其它服务，直接自身启动起来即可，为最底层服务。
- OM服务，需要依赖SCM服务，要配置SCM的通信地址。
- Datanode节点服务，需要依赖SCM，OM服务，二者通信地址都得配上。



### 常见QA

#### 本地spark程序读写远程服务器上的ozone报错

spark读取ozone数据时，会先访问om，然后再根据pipeline里的datanode通讯获取数据。

而pipeline中的datanode ip是datanode启动时取自hosts文件的，这里`172.22.66.219`明显是内网地址。因此我本地的mac是无法访问的。

```
21/08/12 17:24:40 ERROR XceiverClientGrpc: Failed to execute command cmdType: GetBlock
traceID: ""
containerID: 3
datanodeUuid: "8a2fdfce-0457-401d-8020-d5bb2249105b"
getBlock {
  blockID {
    containerID: 3
    localID: 106742401654194283
    blockCommitSequenceId: 418
  }
}
 on the pipeline Pipeline[ Id: 0c3251e2-1fed-482a-af01-0bf4fd655f6c, Nodes: 8a2fdfce-0457-401d-8020-d5bb2249105b{ip: 172.22.66.219, host: ozone001, networkLocation: /default-rack, certSerialId: null}a0b21347-b142-4e00-871e-492d40c368b5{ip: 172.22.66.220, host: ozone003, networkLocation: /default-rack, certSerialId: null}a816ae9f-d6b3-4690-bc98-a0577155f1f2{ip: 172.22.66.218, host: ozone002, networkLocation: /default-rack, certSerialId: null}, Type:STAND_ALONE, Factor:THREE, State:OPEN, leaderId:, CreationTimestamp2021-08-12T09:23:09.615Z].
21/08/12 17:24:40 ERROR Executor: Exception in task 0.0 in stage 0.0 (TID 0)
java.io.IOException: java.util.concurrent.ExecutionException: org.apache.ratis.thirdparty.io.grpc.StatusRuntimeException: DEADLINE_EXCEEDED: deadline exceeded after 29.999743099s. [buffered_nanos=30003804661, waiting_for_connection]
	at org.apache.hadoop.hdds.scm.XceiverClientGrpc.sendCommandWithRetry(XceiverClientGrpc.java:364)
	at org.apache.hadoop.hdds.scm.XceiverClientGrpc.lambda$sendCommandWithTraceIDAndRetry$0(XceiverClientGrpc.java:281)
	at org.apache.hadoop.hdds.tracing.TracingUtil.executeInSpan(TracingUtil.java:174)
	at org.apache.hadoop.hdds.tracing.TracingUtil.executeInNewSpan(TracingUtil.java:148)
	at org.apache.hadoop.hdds.scm.XceiverClientGrpc.sendCommandWithTraceIDAndRetry(XceiverClientGrpc.java:275)
	at org.apache.hadoop.hdds.scm.XceiverClientGrpc.sendCommand(XceiverClientGrpc.java:256)
	at org.apache.hadoop.hdds.scm.storage.ContainerProtocolCalls.getBlock(ContainerProtocolCalls.java:119)
	at org.apache.hadoop.hdds.scm.storage.BlockInputStream.getChunkInfos(BlockInputStream.java:199)
	at org.apache.hadoop.hdds.scm.storage.BlockInputStream.initialize(BlockInputStream.java:133)
	at org.apache.hadoop.hdds.scm.storage.BlockInputStream.read(BlockInputStream.java:254)
	at org.apache.hadoop.ozone.client.io.KeyInputStream.read(KeyInputStream.java:199)
	at org.apache.hadoop.fs.ozone.OzoneFSInputStream.read(OzoneFSInputStream.java:63)
	at java.io.DataInputStream.read(DataInputStream.java:149)
	at org.apache.hadoop.mapreduce.lib.input.UncompressedSplitLineReader.fillBuffer(UncompressedSplitLineReader.java:62)
	at org.apache.hadoop.util.LineReader.readDefaultLine(LineReader.java:216)
	at org.apache.hadoop.util.LineReader.readLine(LineReader.java:174)
	at org.apache.hadoop.mapreduce.lib.input.UncompressedSplitLineReader.readLine(UncompressedSplitLineReader.java:94)
	at org.apache.hadoop.mapreduce.lib.input.LineRecordReader.skipUtfByteOrderMark(LineRecordReader.java:144)
	at org.apache.hadoop.mapreduce.lib.input.LineRecordReader.nextKeyValue(LineRecordReader.java:184)
	at org.apache.spark.sql.execution.datasources.RecordReaderIterator.hasNext(RecordReaderIterator.scala:39)
	at org.apache.spark.sql.execution.datasources.HadoopFileLinesReader.hasNext(HadoopFileLinesReader.scala:69)
	at scala.collection.Iterator$$anon$11.hasNext(Iterator.scala:408)
```

**解决办法**：

修改hosts文件。用公网地址，然后重启ozone集群。

```
116.62.227.93 ozone003
47.98.214.48 ozone002
47.98.194.178 ozone001
```

重新用put命令上传一个文件到ozone。然后可以用admin命令检查pipeline

```
ozone admin pipeline list
```

可以观察到红框内的pipeline里的datanode的ip都是公网的，后面spark程序取数时就可以连接到了

![image-20210812174625837](http://image-picgo.test.upcdn.net/img/20210812174625.png)