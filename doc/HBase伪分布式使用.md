# HBase伪分布式使用

[TOC]

## 下载

```
wget https://archive.apache.org/dist/hbase/1.2.4/hbase-1.2.4-bin.tar.gz
```



## 修改hbase相关文件

### conf/hbase-env.sh

 

- 对于HBase 0.98.5和更高版本，需要在启动HBase之前设置JAVA_HOME环境变量  

```
export JAVA_HOME=/usr/java/jdk1.7.0_80
```

假定集群的每个节点使用相同的配置。如果不是这样，您需要为每个节点单独设置JAVA_HOME。

**这个环境变量设置不是必须的**

 

- 告诉HBase是否应该管理自己的Zookeeper实例，设置为true让它自己管理

```
export HBASE_MANAGES_ZK=true
```

 

- 设置PID文件存储路径，缺省是在/tmp 下

```
export HBASE_PID_DIR=/data/hbase/tmp/pids
```

 

- 使用JDK8 ，需要在HBase的配置文件中hbase-env.sh，注释掉两行

```
# Configure PermSize. Only needed in JDK7. You can safely remove it for JDK8+
export HBASE_MASTER_OPTS="$HBASE_MASTER_OPTS -XX:PermSize=128m -XX:MaxPermSize=128m"
export HBASE_REGIONSERVER_OPTS="$HBASE_REGIONSERVER_OPTS -XX:PermSize=128m -XX:MaxPermSize=128m"
```

 

### conf/hbase-site.xml 

这是HBase的主配置文件

缺省配置：<http://hbase.apache.org/book.html#hbase_default_configurations>

 

- 配置本地临时目录，缺省值是'/tmp'

```
<property>
  <name>hbase.tmp.dir</name>
  <value>/opt/data/hbase/tmp</value>
</property>
```

- 指定zookeeper数据目录

  ```
  <property>
    <name>hbase.zookeeper.property.dataDir</name>
    <value>/opt/data/zookeeper</value>
  </property>
  ```

  

- 指定分布式运行

```
<property>
  <name>hbase.cluster.distributed</name>
  <value>true</value>
</property>
```

 

- 配置hbase目录，让HDFS生成该目录给hbase使用

```
<property>
  <name>hbase.rootdir</name>
  <value>hdfs://localhost:8020/hbase</value>
</property>
```

 

> 注意：

> $HBASE_HOME/conf/hbase-site.xml 的 hbase.rootdir的主机和端口号
>
> $HADOOP_HOME/conf/core-site.xml 的 fs.default.name的主机和端口号一致



## 配置环境变量方便使用

```
export HBASE_HOME=/opt/hadoop/hbase-1.2.4
export PATH=$HBASE_HOME/bin:$PATH
```



## 访问地址

HBase的Web界面 <http://134.175.29.194:16010/>





------

## shell操作hbase

## 常用命令

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190418215047.png)

### 使用hbase shell

```
$ hbase shell
```

