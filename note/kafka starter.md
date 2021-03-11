# kafka starter

## 常用命令

### **1.kafka server启动:**  

```shell
./kafka-server-start.sh ../config/server.properties &
```



### **2.创建topic:** 

```shell
 ./kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic tes
```



### **3.查看kafka的所有topic：**

```shell
./kafka-topics.sh --zookeeper localhost:2181 --list
```



### **4.查看kafka某个topic下partition信息:** 

```shell
./kafka-topics.sh --describe --zookeeper localhost:2181 --topic test-topic
```



### **5.查看kafka的指定topic的描述:**  

```shell
./kafka-topics.sh --zookeeper localhost:2181 --describe --topic yq20171220
```



### **6.控制台向kafka生产数据:**  

```shell
./kafka-console-producer.sh --broker-list localhost:9092 --topic source --property "parse.key=true" --property "key.separator=@@@"
```

进入终端后，输入数据

```shell
> data_increment_data.kafka.edp.source.ums_extension.*.*.*@@@{"id": 1, "name": "test", "phone":"18074546423", "city": "Beijing", "time": "2017-12-22 10:00:00"}
>data_increment_data.kafka.edp.source.ums_extension.*.*.*@@@{"id": 1, "name": "test", "phone":"18074546423", "city": "Beijing", "time": "2017-12-22 10:00:00"}
```





### 7.控制台消费kafka的数据（也可以读全部数据）:

```shell
./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic source --from-beginning
```

加入参数`--from-beginning`会把数据全部都读出来，不加只读启动脚本后采集到的。

![](http://image-picgo.test.upcdn.net/img/20200519093745.png)





### 8.查看topic下某分区偏移量的最小值: 

```shell
./kafka-run-class.sh kafka.tools.GetOffsetShell --topic test-topic  --time -1 --broker-list localhost:9092 --partitions 0
```



### 9.增加topic的partition:

```
./kafka-topics.sh --alter --topic jason_20180519 --zookeeper 10.200.10.24:2181,10.200.10.26:2181,10.200.10.29:2181 --partitions 5  
```



### 10.删除topic，慎用，只会删除zookeeper中的元数据，消息文件须手动删除:  

```
./kafka-run-class.sh kafka.admin.DeleteTopicCommand --zookeeper localhost:2181 --topic yq20171220
```



### 11.彻底删除topic:

```
 rmr /brokers/topics/【topic name】即可
```



## Kafka connect

### 介绍

Kafka connect 只是Apache Kafka提出的一种kafka数据流处理的框架，目前有很多开源的、优秀的实现，比较著名的是Confluent平台，支持很多Kafka connect的实现，例如Elasticsearch(Sink)、HDFS(Sink)、JDBC等等

![kafka_connect](http://www.itrensheng.com/upload/2019/7/kafka_connect-669224bae2e44ae6a149ef820c1b70c1.png)

### 预期目标

将指定topic的数据通过kafka开源组件connect输出到本地文件中。

### 编辑配置

- connect-standalone.properties

  ![](http://image-picgo.test.upcdn.net/img/20191207223224.png)

- Connect-file-sink.properties

  ![](http://image-picgo.test.upcdn.net/img/20191207223304.png)



### 运行standalone模式

```
~/opt/hadoop-cdh/kafka_2.12-2.1.1 » bin/connect-standalone.sh config/connect-standalone.properties   config/connect-file-sink.properties
```

使用kafka console provider输入json。

![](http://image-picgo.test.upcdn.net/img/20191207223612.png)

查看文件已经被修改了。

![](http://image-picgo.test.upcdn.net/img/20191207223558.png)

### 收集connect日志到文件

设置kafka/config目录下的该文件。

![](http://image-picgo.test.upcdn.net/img/20191207232102.png)

```
log4j.appender.stdfile=org.apache.log4j.DailyRollingFileAppender
log4j.appender.stdfile.DatePattern='.'yyyy-MM-dd-HH
log4j.appender.stdfile.File=${kafka.logs.dir}/stdout.log
log4j.appender.stdfile.layout=org.apache.log4j.PatternLayout
log4j.appender.stdfile.layout.ConversionPattern=[%d] %p %m (%c)%n
```



### 收集mysql cdc数据

#### 下载cdc插件

https://repo1.maven.org/maven2/io/debezium/debezium-connector-mysql/1.4.0.Final/debezium-connector-mysql-1.4.0.Final-plugin.tar.gz



#### 将插件放入自行创建的connect目录

![image-20210125174417619](http://image-picgo.test.upcdn.net/img/20210125174417.png)



#### 配置config/connect-standalone.properties

使其加载下载的binlog插件。

![image-20210125174508988](http://image-picgo.test.upcdn.net/img/20210125174509.png)



#### 编写binlog-source配置

conncet-mysqlbinlog-source.properties。

具体参考：https://debezium.io/documentation/reference/1.4/connectors/mysql.html#mysql-property-database-include-list

```
name=inventory-connector
connector.class=io.debezium.connector.mysql.MySqlConnector
database.hostname=hadoop001
database.port=3307
database.user=root
database.password=debezium
database.server.id=1840541
database.server.name=fullfillment4
table.include.list=inventory.t2
database.history.kafka.bootstrap.servers=hadoop001:9092
database.history.kafka.topic=dbhistory.fullfillment
database.history.skip.unparseable.ddl=true
include.schema.changes=true 


```



#### 启动

```
 bin/connect-standalone.sh config/connect-standalone.properties config/conncet-mysqlbinlog-source.properties
```





### 收集sqlserver CDC数据

1. sqlserver开启SQL server agent服务

   通过图形界面操作：

   开始--> [win + R] --> services.msc -- 启动SQL server 代理服务。

2. 配置sqlserver开启CDC

   参考文档：https://blog.csdn.net/vkingnew/article/details/89508885?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-5.nonecase&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-5.nonecase

   

   CDC适用的环境：

   1.SQL server 2008版本以上的企业版、开发版和评估版中可用；

   2.需要开启代理服务（作业）。

   3.CDC需要业务库之外的额外的磁盘空间。

   4.CDC的表需要主键或者唯一主键。

    

```sql
use cdc_test;

--库开启
EXEC sys.sp_cdc_enable_db  
--库检查是否开启
select * from sys.databases where is_cdc_enabled = 1  

--创建测试表
 create table NewTable3(id varchar(36) not null primary key,city_name varchar(20),userid bigint,useramount decimal(18,6),ismaster bit,createtime datetime default getdate());


--表级别开启
  
EXEC sys.sp_cdc_enable_table
        @source_schema = 'dbo', -- source_schema
        @source_name = 'NewTable3', -- table_name
        @capture_instance = NULL, -- capture_instance
        @supports_net_changes = 1, -- supports_net_changes
        @role_name = NULL, -- role_name
        @index_name = NULL, -- index_name
        @captured_column_list = NULL -- captured_column_list

--表检查是否开启
select name,type,create_date,modify_date,is_tracked_by_cdc from sys.tables where is_tracked_by_cdc = 1;
 
SELECT sys.fn_cdc_get_max_lsn()
 
 -- 查询某个库的物理文件：
SELECT name, physical_name FROM sys.master_files WHERE database_id = DB_ID('cdc_test');

-- 检查作业配置
EXEC sp_cdc_help_jobs

--对要捕获的表进行DML操作和DDL操作测试
-- INSERT:
insert into NewTable3(id,city_name,userid,useramount,ismaster)values('1','wuhan',     10,1000.25,1);
insert into NewTable3(id,city_name,userid,useramount,ismaster)values('1A','xiangyang',11,11000.35,0);
insert into NewTable3(id,city_name,userid,useramount,ismaster)values('1B','yichang',  12,12000.45,0);
insert into NewTable3(id,city_name,userid,useramount,ismaster)values('1c','yichddang',  13,123000.45,0);

-- update
update NewTable3 set ismaster=0 WHERE id = '1'

-- delete
delete from NewTable3  WHERE id = '1'

alter  table NewTable3 add   product_count decimal(18,2);
alter  table NewTable3 add   product_count2 decimal(18,2);


--查询：
select *  from cdc_test.dbo.NewTable3;

--查询捕获的数据：
SELECT * from  cdc_test.cdc.dbo_NewTable3_CT
```



3. 下载debezium sqlserver插件

   ```shell
   wget https://repo1.maven.org/maven2/io/debezium/debezium-connector-sqlserver/1.1.1.Final/debezium-connector-sqlserver-1.1.1.Final-plugin.tar.gz
   ```

4. 解压到kafka目录下

   ![image-20200527135538759](http://image-picgo.test.upcdn.net/img/20200527135538.png)

5. 编辑配置文件`connect-standalone.properties`指向connect插件路径

   ![image-20200527135625641](http://image-picgo.test.upcdn.net/img/20200527135625.png)

6. 启动kafka connect

   ```shell
   [root@cdh05 kafka]# bin/connect-standalone.sh conf/connect-standalone.properties  conf/connect-console-source.properties 
   ```

7. 提交请求让kafka connect监听sqlserver

   ```shell
   [root@cdh05 connect]# curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" localhost:8083/connectors/ -d '{"name":"yibo-connector","config":{"connector.class":"io.debezium.connector.sqlserver.SqlServerConnector","database.hostname":"192.168.1.86","database.port":"1433","database.user":"sa","database.password":"Sa123","database.dbname":"cdc_test","database.server.name":"cdc_test","table.whitelist":"dbo.NewTable3","database.history.kafka.bootstrap.servers":"cdh05:6667","database.history.kafka.topic":"dbhistory.cdc_test"}}'
   ```

8. 查看kafka的topic可以看到两个

   其中`cdc_test.dbo.NewTable3`是表数据变化的监听。

   即该表数据增加删除更新都会收到消息。

   ![](http://image-picgo.test.upcdn.net/img/20200527140034.png)

9. 使用kafka console consumer消费

   ```shell
   [root@cdh05 kafka]# bin/kafka-console-consumer.sh --bootstrap-server cdh05:6667 --topic cdc_test.dbo.NewTable3 --from-beginning
   ```

   当更改了表`NewTable3`的一条数据后。

   ![image-20200527140647117](http://image-picgo.test.upcdn.net/img/20200527140647.png)

   可以看到控制台输出。

   ![image-20200527140625698](http://image-picgo.test.upcdn.net/img/20200527140625.png)



**备注：目前发现，如果开启cdc的表增加字段后没法监听到多出的字段变化。**

