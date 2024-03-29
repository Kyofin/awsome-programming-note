# Ozone1.0.0对接Spark、Hive、HDFS

## 使用cdh5.16.2的hdfs cli访问Ozone

配置cdh-hadoop的`core-site-xml`，增加下面配置

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

复制`ozone-site.xml`到hdfs cli的配置文件目录，并且该文件中要指定`ozone.replication`为1。

![image-20210804172619941](http://image-picgo.test.upcdn.net/img/20210804172620.png)

```
cp ~/opt/ozone-1.0.0/etc/hadoop/ozone-site.xml ~/cdh5.16/hadoop-2.6.0-cdh5.16.2/etc/hadoop/
```

### 删除数据：

```
hadoop fs -rm o3fs://bucket.volume.localhost:6789/README.md
```



## 使用Spark2.4.6访问Ozone

### 修改spark配置文件

修改SPAKR_HOME里的conf目录下的`core-site.xml`

```xml
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

```shell
spark-shell --jars /Users/huzekang/Downloads/hadoop-ozone-filesystem-hadoop2-1.0.0.jar --driver-class-path /Users/huzekang/Downloads/hadoop-ozone-filesystem-hadoop2-1.0.0.jar
```

#### 读ozone数据测试

![image-20210803180651317](http://image-picgo.test.upcdn.net/img/20210803180651.png)

#### 写数据到ozone测试

![image-20210803181122817](http://image-picgo.test.upcdn.net/img/20210803181122.png)

#### 使用hdfs cli检验产出的数据

![image-20210803181233159](http://image-picgo.test.upcdn.net/img/20210803181233.png)





#### 编写spark代码测试

引入依赖：

```xml
  <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-ozone-filesystem-hadoop2</artifactId>
            <version>1.0.0</version>
        </dependency>
```

需要配置sparkSession：

```scala
  .config("fs.AbstractFileSystem.o3fs.impl","org.apache.hadoop.fs.ozone.OzFs")
      .config("fs.defaultFS","o3fs://bucket.volume.localhost:6789")
      .config("ozone.replication",1)
```

![image-20210805095119406](http://image-picgo.test.upcdn.net/img/20210805095119.png)

可以看到成功运行结果。

![image-20210805095206370](http://image-picgo.test.upcdn.net/img/20210805095206.png)

## 使用hive cli访问Ozone

### 修改hive配置文件

主要改动`hive/conf`下的几个文件。



**core-site.xml**

增加ozone为defaultFS。

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!--Autogenerated by Cloudera Manager-->
<configuration>
 
<property>
  <name>fs.AbstractFileSystem.o3fs.impl</name>
  <value>org.apache.hadoop.fs.ozone.OzFs</value>
</property>
<property>
  <name>fs.defaultFS</name>
  <value>o3fs://bucket.volume.localhost:6789</value>
</property>
</configuration>
```



**hive-env.sh**

增加引入ozone依赖。

```shell
export HIVE_AUX_JARS_PATH=/Users/huzekang/Downloads/hadoop-ozone-filesystem-hadoop2-1.0.0.jar
```



**hive-site.xml**

如果希望内部表可以使用ozone的bucket进行存储，所以需要增加下面配置

```xml
<property>
    <name>hive.metastore.warehouse.dir</name>
    <value>o3fs://bucket.volume.localhost:6789/hive/warehouse</value>
</property>
```





### 测试

#### 建表

外部表

```sql
create external  table transaction1(id int,sex string,age int,date string,role string,region string) row format delimited fields terminated by ' ' stored as textfile location '/transaction1' ;
```



内部表

```sql
create   table transaction3(id int,sex string,age int,date string,role string,region string) row format delimited fields terminated by ' ' stored as textfile  ;
```

可以看到即使没有指定location，默认的存储都是ozone的bucket。

![image-20210804170812008](http://image-picgo.test.upcdn.net/img/20210804170812.png)

#### 插入数据

```sql
insert into transaction1 values(1,'女','22','19950822','管理员','广州');
insert into transaction3 values(1,'女','22','19950822','管理员','广州');
```



#### 复杂查询

```sql
select count(1) from transaction1 group by date;
select count(1) from transaction3 group by date;
```



#### 删除表

```sql
drop table transaction3;
```

