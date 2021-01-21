# presto-server-0.233.1安装体验

server:https://repo1.maven.org/maven2/com/facebook/presto/presto-server/0.233.1/presto-server-0.233.1.tar.gz

client:https://repo1.maven.org/maven2/com/facebook/presto/presto-cli/0.233.1/presto-cli-0.233.1-executable.jar

## 1 安装

```shell
#server
tar -zxvf /opt/software/presto/presto-server-0.233.1.tar.gz -C /opt/module/presto-0.233.1/

#1.把 presto-cli-0.223.1.jar 复制到 /opt/module/presto-0.233.1/presto-server-0.233.1/bin 目录下
cp /opt/software/presto/presto-cli-0.223.1.jar  /opt/module/presto-0.233.1/presto-server-0.233.1/bin

#2.presto-cli-0.223.1.jar 重命名 presto 
mv /opt/module/presto-0.233.1/presto-server-0.233.1/bin/presto-cli-0.223.1.jar /opt/module/presto-0.233.1/presto-server-0.233.1/bin/presto

#3.增加 presto 的执行权限
chmod +x /opt/module/presto-0.233.1/presto-server-0.233.1/bin/presto
```

## 2 配置 Presto

###### 1.配置数据目录

```shell
#最好安装在 presto server 安装目录外
mkdir /opt/module/presto-0.233.1/data
```

###### 2.创建配置文件

在 presto server 安装目录 /opt/module/presto-0.233.1/presto-server-0.233.1 创建 etc 文件夹

```shell
mkdir /opt/module/presto-0.233.1/presto-server-0.233.1/etc
```

2.在 /opt/module/presto-0.233.1/presto-server-0.233.1/etc 下创建 config.properties，jvm.properties，node.properties，log.properties 文件

**config.properties**（coordinator）

如果想一台服务器同时启动coordinator和worker，只需要`node-scheduler.include-coordinator=true`

```
coordinator=true
node-scheduler.include-coordinator=false
http-server.http.port=8089
query.max-memory=10GB
query.max-memory-per-node=1GB
query.max-total-memory-per-node=2GB
discovery-server.enabled=true
discovery.uri=http://10.93.6.247:8089

```

**config.properties**（worker）

```
coordinator=false
http-server.http.port=8089
query.max-memory=10GB
query.max-memory-per-node=1GB
query.max-total-memory-per-node=2GB
discovery.uri=http://10.93.6.247:8089

```



**jvm.config**  (Presto集群coordinator和worker的JVM配置是一致的)

```
-server
-Xmx16G
-XX:+UseG1GC
-XX:G1HeapRegionSize=32M
-XX:+UseGCOverheadLimit
-XX:+ExplicitGCInvokesConcurrent
-XX:+HeapDumpOnOutOfMemoryError
-XX:+ExitOnOutOfMemoryError
```

**node.properties **(worker和coordinator的node.id注意要不一样)

```
node.environment=production
node.id=ffffffff-ffff-ffff-ffff-ffffffffffff01
node.data-dir=/opt/module/presto-0.233.1/data
```

**log.properties**

```
com.facebook.presto=DEBUG
```





## 3 配置 connector

1.在 /opt/module/presto-0.233.1/presto-server-0.233.1/etc 创建 catalog 目录

```
mkdir /opt/module/presto-0.233.1/presto-server-0.233.1/etc/catalog
```

2.在 catalog 目录下 创建 hive connector

**hive.properties**

```
connector.name=hive-hadoop2
hive.metastore.uri=thrift://hadoop-slave1:9083
hive.config.resources=/etc/hadoop/core-site.xml,/etc/hadoop/hdfs-site.xml
```

3.在 catalog 目录下 创建 mysql connector

**mysql.properties**

```
connector.name=mysql
connection-url=jdbc:mysql://hadoop101:3306
connection-user=root
connection-password=123456
```





## 4 启动 presto

注意：Presto requires Java 8u151+，需要jdk 1.8.151 以上，否则 PrestoServer 进程会自动死亡

```
#后台启动 （日志在 数据目录 /opt/module/presto-0.233.1/data/var/log）
/opt/module/presto-0.233.1/presto-server-0.233.1/bin/launcher start

#调试启动
/opt/module/presto-0.233.1/presto-server-0.233.1/bin/launcher --verbose run
```

访问 presto webui [http://hadoop101:8085](http://hadoop101:8085/)