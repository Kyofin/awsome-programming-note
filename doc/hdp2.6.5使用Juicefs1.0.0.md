# hdp2.6.5使用Juicefs1.0.0

## 部署服务器清单

10.93.6.6（hdp/minio/juicefs）

10.93.6.7（hdp/minio/juicefs）

10.93.6.8（hdp/minio/juicefs）

10.93.11.50（hdp/minio/juicefs）



## 部署minio

首先下载minio的执行文件，然后上传服务器，在编写一个启动脚本如下：

```bash
#!/bin/bash

export MINIO_ACCESS_KEY=minio
export MINIO_SECRET_KEY=minio123
nohup /opt/minio_home/minio server --address :9003 --console-address :9005 http://10.93.6.6/data/minio/data http://10.93.6.7/data/minio/data  \
http://10.93.6.8/data/minio/data \
http://10.93.11.50/data/minio/data   > /opt/minio_home/minio.log 2>&1 &
```

启动成功后，可以访问http://10.93.6.6:9005。账号密码为minio/minio123



## 初始化juicefs元数据

官方建议元数据存储在redis，但是hdp集群目前还没装redis，所以就用mysql来代替。首先下载minio的执行文件，然后上传服务器，然后在准备好mysql和minio后，就可以执行juicefs的元数据初始化命令。

```bash
/opt/juicefs_home/juicefs format --storage minio \
    --bucket http://10.93.6.6:9003/myjfs \
    --access-key minio \
    --secret-key minio123 \
    "mysql://root:Root@123@(10.93.6.6:33066)/juicefs"  myjfs
```



## 本地挂载juicefs

下面步骤非必须，只是可以方便在本地磁盘挂载juicefs，来验证juicefs是否正常。

```bash
/opt/juicefs_home/juicefs mount  "mysql://root:730xd7.2@(10.81.16.22:3306)/juicefs"   /opt/juicefs_home/mnt -d 
/opt/juicefs_home/juicefs  umount /opt/juicefs_home/mnt


```



## hdp各节点部署juicefs-hadoop jar包

```bash
mkdir /usr/hdp/current/hive-client/auxlib

cp /opt/juicefs_home/juicefs-hadoop-1.0.0-beta1-linux-amd64.jar   /usr/hdp/current/hadoop-client/lib && cp /opt/juicefs_home/juicefs-hadoop-1.0.0-beta1-linux-amd64.jar   /usr/hdp/current/hive-client/auxlib  && cp /opt/juicefs_home/juicefs-hadoop-1.0.0-beta1-linux-amd64.jar   /usr/hdp/current/spark2-client/jars
```







## ambari上改HDFS的配置core-site.xml



```xml
  <property>
        <name>fs.jfs.impl</name>
        <value>io.juicefs.JuiceFileSystem</value>
    </property>
    <property>
        <name>fs.AbstractFileSystem.jfs.impl</name>
        <value>io.juicefs.JuiceFS</value>
    </property>
    <property>
        <name>juicefs.meta</name>
        <value>mysql://root:Root@123@(10.93.6.6:33066)/juicefs</value>
    </property>
    <property>
        <name>juicefs.cache-size</name>
        <value>1024</value>
    </property>
    <property>
        <name>juicefs.access-log</name>
        <value>/tmp/juicefs.access.log</value>
    </property>
    <property>
        <name>juicefs.cache-dir</name>
        <value>/data/juicefs</value>
    </property>
```

![image-20220130092757678](http://image-picgo.test.upcdn.net/img/20220130092757.png)

![image-20220130092809590](http://image-picgo.test.upcdn.net/img/20220130092809.png)





## datax支持读写juicefs

在datax部署目录的插件中放入juicefs-hadoop客户端包

```bash
cp /opt/juicefs_home/juicefs-hadoop-1.0.0-beta1-linux-amd64.jar /data/usync/lastest/plugin/writer/hdfswriter/libs && cp /opt/juicefs_home/juicefs-hadoop-1.0.0-beta1-linux-amd64.jar /data/usync/lastest/plugin/reader/hdfsreader/libs
```



构建job json的配置文件时，读写hdfs都需要加入下面配置

```json
{
  "defaultFS": "jfs://myjfs",
"hadoopConfig": {
    "fs.jfs.impl": "io.juicefs.JuiceFileSystem",
    "fs.AbstractFileSystem.jfs.impl": "io.juicefs.JuiceFS",
    "juicefs.access-log": "/tmp/juicefs.access.log",
    "juicefs.meta": "mysql://root:Root@123@(10.93.6.6:33066)/juicefs",
    "juicefs.cache-dir": "/data/juicefs",
    "juicefs.cache-size": "1024"
}
```



## 检验是否安装成功

### HDFS cli测试

```bash
hadoop fs -ls jfs://myjfs/
hadoop fs -mkdir jfs://myjfs/jfs-test
hadoop fs -rm -r jfs://myjfs/jfs-test
```



### hive cli测试

```sql
create database juicefs location  'jfs://myjfs/hive_warehouse/';

use juicefs;

create table if not exists person(
    name string,
    age int
);

insert into table person values('tom',25);
insert overwrite table person select name, age from person;
select name, age from person;
drop table person;
```

