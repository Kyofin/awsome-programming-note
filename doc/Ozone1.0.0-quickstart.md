# Ozone1.0.0-quickstart

## 资源文件准备

ozone1.0.0二进制部署包

https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/ozone/ozone-1.0.0/hadoop-ozone-1.0.0.tar.gz

hadoop2兼容ozone的jar包

https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-ozone-filesystem-hadoop2/1.0.0/hadoop-ozone-filesystem-hadoop2-1.0.0.jar





## 单节点手动部署Ozone

**注意**：

```
当系统环境变量里有HADOOP_HOME时，需要先注释掉，否则在安装ozone过程中，执行的ozone的脚本都会寻找HADOOP_HOME目录下的文件，就会报找不到的错误。
```

### 修改配置

生成默认ozone配置到当前目录

```
bin/ozone genconf .
```

修改ozone-site.xml

- ozone metadata 存储目录:

```
<property>
	<name>ozone.metadata.dirs</name>
	<value>/Users/huzekang/opt/ozone-1.0.0/metadata</value>
</property>
```

- datanode 存储目录

```

<property>
    <name>dfs.datanode.data.dir</name>
    <value>file:///Users/huzekang/opt/ozone-1.0.0/data</value>
</property>
```

- 单副本

```
     <property>
 <name>ozone.replication</name>
 <value>1</value>
 </property>
```

- om address

```
 <property>
        <name>ozone.om.address</name>
        <value>localhost:6789</value>
    </property>
```

- 其他

```
<property>
        <name>ozone.scm.client.address</name>
        <value>localhost</value>
       
    </property>
    <property>
        <name>ozone.scm.names</name>
        <value>localhost</value>
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

如果要关闭ozone，则执行下面命令

```
bin/ozone  --daemon stop datanode
sbin/stop-ozone.sh
```





## ozone默认ui

### Ozone manager UI

![image-20210803171428027](http://image-picgo.test.upcdn.net/img/20210803171428.png)



### Storage Container Manager UI

![image-20210803171506944](http://image-picgo.test.upcdn.net/img/20210803171507.png)

### S3 gateway

![image-20210803172819041](http://image-picgo.test.upcdn.net/img/20210803172819.png)

### datanode

![image-20210803173906201](http://image-picgo.test.upcdn.net/img/20210803173906.png)





## aws cli

aws s3api --endpoint http://localhost:9878/ create-bucket --bucket=wordcount

aws s3api --endpoint http://localhost:9878/  list-buckets

aws s3api --endpoint http://localhost:9878/  put-object --bucket=wordcount  --storage-class REDUCED_REDUNDANCY --key Doc1 --body  ~/table1_data

aws s3api --endpoint http://localhost:9878/  list-objects --bucket=wordcount

aws s3api --endpoint http://localhost:9878/  get-object --bucket=wordcount --key Doc1 ~/testfile2 

aws s3 --endpoint http://localhost:9878  ls  s3://wordcount/Doc1

aws s3 --endpoint http://localhost:9878  rm  s3://wordcount/Doc1

 aws s3api --endpoint http://localhost:9878/  delete-object --key Doc1 --bucket=test



## ozone cli

- 创建 volume

```
ozone sh volume create  --quota=1TB  /hive
ozone sh volume create /volume
ozone sh volume list
```

- 创建 bucket

```
ozone sh bucket create /hive/dc2
ozone sh bucket create /volume/bucket
ozone sh bucket list /hive
```

- 写入并读取文件，比对

```
ozone sh key put --replication=ONE /hive/dc2/README.md README.md
ozone sh key list /hive/dc2
ozone sh key get /hive/dc2/README.md README2.txt
diff README2.txt README.md

```



## 使用cdh5.16.2的hdfs cli访问Ozone

配置cdh-hadoop的core-site-xml，增加下面配置

```XML
<property>
  <name>fs.AbstractFileSystem.o3fs.impl</name>
  <value>org.apache.hadoop.fs.ozone.OzFs</value>
</property>
<property>
  <name>fs.defaultFS</name>
  <value>o3fs://bucket.volume.localhost:6789</value>
</property>
```

注意：上面的`bucket`是ozone的bucket名，`volume`是ozone的volume名，`localhos:6789`是ozone的om address。

执行hdfs cli前，需要执行下面命令将ozone lib加载到hadoop classpath中。

```bash
export HADOOP_CLASSPATH=/Users/huzekang/Downloads/hadoop-ozone-filesystem-hadoop2-1.0.0.jar:$HADOOP_CLASSPATH
```

使用hdfs cli测试。

### 查看数据：

```
hadoop fs -ls /                                                                                           
hadoop fs -ls o3fs://bucket.volume.localhost:6789/
```

### 上传数据时：

```
hadoop fs -put table1_data /                                                                              
```

但是这里会报错，`put: Allocated 0 blocks. Requested 1 blocks`。估计这个错误是由于我只起了一个datanode，但客户端请求要上传3副本导致的。

解决办法：

复制ozone-site.xml到hdfs cli的配置文件目录，并且该文件中要指定`ozone.replication`为1。

```
cp ~/opt/ozone-1.0.0/etc/hadoop/ozone-site.xml ~/cdh5.16/hadoop-2.6.0-cdh5.16.2/etc/hadoop/
```

### 删除数据：

```
hadoop fs -rm o3fs://bucket.volume.localhost:6789/README.md
```



## 使用Spark2.4.6访问Ozone

### 修改spark配置文件

修改SPAKR_HOME里的conf目录下的core-site.xml

```
<property>
  <name>fs.AbstractFileSystem.o3fs.impl</name>
  <value>org.apache.hadoop.fs.ozone.OzFs</value>
</property>
<property>
  <name>fs.defaultFS</name>
  <value>o3fs://bucket.volume.localhost:6789</value>
</property>
```

### 启动spark-shell测试

```
spark-shell --jars /Users/huzekang/Downloads/hadoop-ozone-filesystem-hadoop2-1.0.0.jar --driver-class-path /Users/huzekang/Downloads/hadoop-ozone-filesystem-hadoop2-1.0.0.jar
```

#### 读ozone数据测试

![image-20210803180651317](http://image-picgo.test.upcdn.net/img/20210803180651.png)

#### 写数据到ozone测试

![image-20210803181122817](http://image-picgo.test.upcdn.net/img/20210803181122.png)

#### 使用hdfs cli检验产出的数据

![image-20210803181233159](http://image-picgo.test.upcdn.net/img/20210803181233.png)