# hive外部表关联phoenix表

### 分析：

由于在大数据的生态中hive是最通用的，基本每个组件都会优先考虑和hive集成。

所以一张表注册到hive的catalog后，像spark，flink，kylin等都能直接和hive连接就可以查到这张表。

实现了读取源的统一。



### 实现思路：

1. phoenix上的表用hive的外部表管理起来（本文的思路）

2. 将数据存储到hbase表 （参考https://www.cnblogs.com/wuning/p/11568719.html）

   1. 使用phoenix表与hbase表关联起来
   2. 使用hive表与hbase表关联起来

   



###  准备phoenix hive jar

在phoenix目录下找到该依赖。

![](http://image-picgo.test.upcdn.net/img/20200411215909.png)

### 

### 在phoenix中建测试表初始化数据

```
create table car (ID varchar,COORID varchar,CX varchar, DATE varchar,HPHM varchar,YS varchar);
upsert into car values('1','2','3','4','ddfd','dd');
```

![](http://image-picgo.test.upcdn.net/img/20200411220726.png)



### hive shell中集成示例

将上述依赖放入`hive/lib`中

![](http://image-picgo.test.upcdn.net/img/20200411220128.png)

修改hive-site.xml。增加下面配置。

```properties
 <property>
    <name>hive.aux.jars.path</name>
    <value>/Users/huzekang/opt/hadoop-cdh/hive-1.1.0-cdh5.14.2/lib/phoenix-4.14.0-cdh5.14.2-hive.jar</value>
</property>
```

打开hive shell进行建外部表关联phoenix。

```sql
create external table car_zw (
id string,
coorid string,
cx string,
date1 string,
hphm string,
ys string
)
STORED BY 'org.apache.phoenix.hive.PhoenixStorageHandler'
TBLPROPERTIES (
"phoenix.table.name" = "car",
"phoenix.zookeeper.quorum" = "localhost",
"phoenix.zookeeper.znode.parent" = "/hbase",
"phoenix.zookeeper.client.port" = "2181",
"phoenix.rowkeys" = "id",
"phoenix.column.mapping" = "id:ID,coorid:COORID,cx:CX,date1:DATE,hphm:HPHM,ys:YS"
);

```

![](http://image-picgo.test.upcdn.net/img/20200411220416.png)



### spark shell中集成示例

启动spark shell时的把phoenix hive jar当成外部依赖带上即可。

```shell
~/opt/hadoop-cdh/phoenix-4.14.0-cdh5.14.2-bin » spark-shell --master yarn  --jars phoenix-4.14.0-cdh5.14.2-hive.jar
```

![](http://image-picgo.test.upcdn.net/img/20200411220634.png)



直接在spark shell中对该hive外部表插入数据。

![](http://image-picgo.test.upcdn.net/img/20200411223158.png)