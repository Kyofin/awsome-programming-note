[TOC]

# 改变hbase使用外部zookeeper

## 下载文件

https://archive.apache.org/dist/zookeeper/zookeeper-3.4.9/zookeeper-3.4.9.tar.gz

```
wget https://archive.apache.org/dist/zookeeper/zookeeper-3.4.9/zookeeper-3.4.9.tar.gz
```



## 停止相关服务

停止hbase

```
stop-hbase.sh
```

停止haddop

```
stop-dfs.sh
```





## 配置zookeeper

复制一份配置

```
/opt/zookeeper-3.4.9/conf # cp zoo_sample.cfg zoo.cfg                                                                     
```

添加该配置到zoo.cfg文件

```
server.1=localhost:2888:3888
```

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190509220445.png)



创建数据文件夹

```
mkdir -p /opt/data/zookeeper
```

根据上述填写的`server.1`写入进程号`1`到文件my.pid，如果上述写`server.2`则需要写入`2`

```
vim /opt/data/zookeeper/my.pid
```

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190509211650.png)





## 启动zookeeper

```shell
/opt/zookeeper-3.4.9/bin # ./zkServer.sh start               

ZooKeeper JMX enabled by default
Using config: /opt/zookeeper-3.4.9/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```

检查启动情况

```shell
/opt/zookeeper-3.4.9/bin # ./zkServer.sh status              

ZooKeeper JMX enabled by default
Using config: /opt/zookeeper-3.4.9/bin/../conf/zoo.cfg
Mode: standalone
```





## 删除原本hbase的数据

查看hdfs.site文件

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190509220836.png)

清空这两个目录

```
rm -rf /opt/hadoop/data/datanode
rm -rf /opt/hadoop/data/namenode
```

格式化hadoop

```
hadoop namenode -format
```

如果看到此处为0则格式化成功

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190509214509.png)



## 启动hadoop

```
start-dfs.sh
```

此时可以用`jps`命令看到hadoop和zookeeper都启动了。

其中QuorumPeerMain就zookeeper应用

```
/opt/hadoop/hadoop-2.7.3/etc/hadoop # jps                    
6112 QuorumPeerMain
9296 DataNode
9186 NameNode
9490 SecondaryNameNode
5080 ProxyServerContainer
9629 Jps
```





## 配置hbase

设置hbase-env.sh不用hbase自带的zookeeper

```
/opt/hadoop/hbase-1.2.4/conf # vim hbase-env.sh              
```

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190509214928.png)



设置hbase.site，指定外部zookeeper地址和端口

```
/opt/hadoop/hbase-1.2.4/conf # vim hbase-site.xml            
```

添加下面👇配置

```
<property>
  <name>hbase.zookeeper.quorum</name>
  <value>localhost</value>
</property>

<property>
  <name>hbase.zookeeper.property.clientPort</name>
  <value>2181</value>
</property>
```

再添加下面配置优化hbase。

设置menstore大小，设置region大小，关闭major compact。

```
<property>
  <name>hbase.hregion.menstore.flush.size</name>
  <value>268435456</value>
</property>
<property>
  <name>hbase.hregion.max.filesize</name>
  <value>107374182400</value>
</property>
<property>
  <name>hbase.hregion.majorcompaction</name>
  <value>0</value>
</property>
```



## 启动hbase

可以看到只启动了master和region没有zookeeper，说明没使用自带的zookeeper。

```
/opt/hadoop/hbase-1.2.4/conf # start-hbase.sh                
starting master, logging to /opt/hadoop/hbase-1.2.4/logs/hbase-root-master-VM_0_7_centos.out
starting regionserver, logging to /opt/hadoop/hbase-1.2.4/logs/hbase-root-1-regionserver-VM_0_7_centos.out
```

