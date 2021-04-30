# Doris0.9编译部署

## 参考文档

[使用docker开发镜像编译](http://doris.apache.org/master/zh-CN/installing/compilation.html#%E4%BD%BF%E7%94%A8-docker-%E5%BC%80%E5%8F%91%E9%95%9C%E5%83%8F%E7%BC%96%E8%AF%91%EF%BC%88%E6%8E%A8%E8%8D%90%EF%BC%89)



## 编译fe和be

下载官方docker编译镜像

```
docker pull apachedoris/doris-dev:build-env
```

使用下载好的doris镜像启动容器

![image-20210429113612638](http://image-picgo.test.upcdn.net/img/20210429113612.png)

```
docker run -it   fd75ea7306bf
```

进入容器后，下载doris0.9的源码

```
wget https://dist.apache.org/repos/dist/dev/incubator/doris/0.9/0.9.0-incubating-rc01/apache-doris-0.9.0.rc01-incubating-src.tar.gz
```

解压源码

```
tar zxvf apache-doris-0.9.0.rc01-incubating-src.tar.gz
```

执行编译命令

```
cd apache-doris-0.9.0.rc01-incubating-src
sh build.sh
```

编译过程中会过如下错误![image-20210429141205656](http://image-picgo.test.upcdn.net/img/20210429141205.png)

需要修改fe/pom.xml文件，pluginRepositories增加如下内容

```
 <!-- for cup-maven-plugin -->
                <pluginRepository>
                    <id>cloudera-public</id>
                    <url>https://repository.cloudera.com/artifactory/public/</url>
                </pluginRepository>
```

repositories增加如下内容

```
<repository>
     <id>cloudera-public</id>
      <url>https://repository.cloudera.com/artifactory/public/</url>
 </repository>
```

编译成功后，打开output目录即有编译好的fe和be目录。

## 编译hdfs_broker

进入`fs_brokers/apache_hdfs_broker`目录，执行

```
sh build.sh
```

编译成功后，进入`fs_brokers/apache_hdfs_broker/output/`，即可看见编译好的hdfs_broker



## 启动hdfs_broker

将编译好的hdfs_broker复制到服务器上

![image-20210430090529107](http://image-picgo.test.upcdn.net/img/20210430090529.png)

使用命令启动

```
sh bin/start_broker.sh --daemon
```

要让 Doris 的 FE 和 BE 知道 Broker 在哪些节点上，通过 sql 命令添加 Broker 节点列表。

使用 mysql-client 连接启动的 FE，执行以下命令：

```
ALTER SYSTEM ADD BROKER broker_name "10.93.11.52:8000";
```

其中 host 为 Broker 所在节点 ip；port 为 Broker 配置文件中的 broker_ipc_port。默认端口是8000。

添加成功后，可以查看是否注册成功。

```
SHOW PROC "/brokers";
```

![image-20210430090903714](http://image-picgo.test.upcdn.net/img/20210430090903.png)



## 使用broker导入HDFS数据

```
LOAD LABEL example_db.label6
(
    DATA INFILE("hdfs://hacluster/apps/hive/warehouse/t1")
    INTO TABLE `t1`
)
WITH BROKER "broker_name"
(
    "username"="hdfs",
    "password"="",
    "dfs.nameservices" = "hacluster",
    "dfs.ha.namenodes.hacluster" = "nn1,nn2",
    "dfs.namenode.rpc-address.hacluster.nn1" = "hadoop001:8020",
    "dfs.namenode.rpc-address.hacluster.nn2" = "hadoop002:8020",
    "dfs.client.failover.proxy.provider" = "org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider"
)
```

执行后，可以看是否成功

```
show load order by createtime desc limit 
```

