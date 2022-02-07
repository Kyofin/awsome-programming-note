## USDP2.0.0.0内使用JuiceFS1.0



## 服务器清单

10.81.16.199（minio）

10.81.16.172（minio）

10.81.16.168（minio）

10.81.16.25（usdp）

10.81.16.79（usdp）

10.81.16.22（usdp、juicefs）

## minio部署

首先下载minio的执行文件，然后上传服务器，在编写一个启动脚本如下：

```shell
#!/bin/bash

export MINIO_ACCESS_KEY=minio
export MINIO_SECRET_KEY=minio123
nohup /opt/minio_home/minio server --address :9000 --console-address :9001 http://10.81.16.168/data/minio/data http://10.81.16.199/data/minio/data  \
http://10.81.16.172/data/minio/data1 \
http://10.81.16.172/data/minio/data2   > /opt/minio_home/minio.log 2>&1 &
```

启动成后，可以访问下面地址访问。

http://10.81.16.168:9001/
http://10.81.16.199:9001/
http://10.81.16.172:9001/

账号密码为：minio  / minio123





## 初始化juicefs

登录10.81.16.22服务器，解压juicefs。并进行格式：

```shell
/opt/juicefs_home/juicefs format --storage minio \
    --bucket http://10.81.16.168:9000/myjfs \
    --access-key minio \
    --secret-key minio123 \
    "mysql://root:730xd7.2@(10.81.16.22:3306)/juicefs"  myjfs
```

为了方便本地查看juicefs文件数据，也可以挂载到本地盘。

```shell
/opt/juicefs_home/juicefs mount  "mysql://root:730xd7.2@(10.81.16.22:3306)/juicefs"   /opt/juicefs_home/mnt -d 

取消挂载
/opt/juicefs_home/juicefs  umount /opt/juicefs_home/mnt
```



## 修改usdp hadoop组件客户端配置

先在服务器`10.81.16.22`上配置好core-site.xml文件。

注意这里是要修改yarn的配置。因为使用命令`hadoop classpath`时，可以看到只会加载该目录的配置，因此如果不改它，会导致后面`hadoop fs -ls jfs://myjfs/`无法使用。

vi /opt/usdp-srv/srv/udp/2.0.0.0/yarn/etc/hadoop/core-site.xml

```
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
        <value>mysql://root:730xd7.2@(10.81.16.22:3306)/juicefs</value>
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

将core-site.xml拷贝到spark和hive的配置目录。

```
cp /opt/usdp-srv/srv/udp/2.0.0.0/yarn/etc/hadoop/core-site.xml /opt/usdp-srv/srv/udp/2.0.0.0/hive/conf/  && cp /opt/usdp-srv/srv/udp/2.0.0.0/yarn/etc/hadoop/core-site.xml /opt/usdp-srv/srv/udp/2.0.0.0/spark/conf/
```



拷贝到其他服务器

```
scp /opt/usdp-srv/srv/udp/2.0.0.0/yarn/etc/hadoop/core-site.xml udp03:/opt/usdp-srv/srv/udp/2.0.0.0/yarn/etc/hadoop && scp /opt/usdp-srv/srv/udp/2.0.0.0/yarn/etc/hadoop/core-site.xml udp02:/opt/usdp-srv/srv/udp/2.0.0.0/yarn/etc/hadoop

scp /opt/usdp-srv/srv/udp/2.0.0.0/hive/conf/core-site.xml udp03:/opt/usdp-srv/srv/udp/2.0.0.0/hive/conf && scp /opt/usdp-srv/srv/udp/2.0.0.0/hive/conf/core-site.xml udp02:/opt/usdp-srv/srv/udp/2.0.0.0/hive/conf

scp /opt/usdp-srv/srv/udp/2.0.0.0/spark/conf/core-site.xml udp03:/opt/usdp-srv/srv/udp/2.0.0.0/spark/conf && scp /opt/usdp-srv/srv/udp/2.0.0.0/spark/conf/core-site.xml udp02:/opt/usdp-srv/srv/udp/2.0.0.0/spark/conf


```







## 拷贝juicefs客户端依赖包到usdp集群各客户端

先在服务器`10.81.16.22`上将juicefs的包放到各个hadoop组件目录里。

```

cp /opt/juicefs_home/juicefs-hadoop-1.0.0-beta1-linux-amd64.jar /opt/usdp-srv/srv/udp/2.0.0.0/yarn/share/hadoop/common/

cp /opt/juicefs_home/juicefs-hadoop-1.0.0-beta1-linux-amd64.jar /opt/usdp-srv/srv/udp/2.0.0.0/spark/jars

cp /opt/juicefs_home/juicefs-hadoop-1.0.0-beta1-linux-amd64.jar /opt/usdp-srv/srv/udp/2.0.0.0/hive/lib


```

再拷贝到其他服务器。

```
scp /opt/juicefs_home/juicefs-hadoop-1.0.0-beta1-linux-amd64.jar udp03:/opt/usdp-srv/srv/udp/2.0.0.0/yarn/share/hadoop/common/ && scp /opt/juicefs_home/juicefs-hadoop-1.0.0-beta1-linux-amd64.jar udp02:/opt/usdp-srv/srv/udp/2.0.0.0/yarn/share/hadoop/common/

scp /opt/juicefs_home/juicefs-hadoop-1.0.0-beta1-linux-amd64.jar udp03:/opt/usdp-srv/srv/udp/2.0.0.0/spark/jars &&  scp /opt/juicefs_home/juicefs-hadoop-1.0.0-beta1-linux-amd64.jar udp02:/opt/usdp-srv/srv/udp/2.0.0.0/spark/jars

scp /opt/juicefs_home/juicefs-hadoop-1.0.0-beta1-linux-amd64.jar udp03:/opt/usdp-srv/srv/udp/2.0.0.0/hive/lib && scp /opt/juicefs_home/juicefs-hadoop-1.0.0-beta1-linux-amd64.jar udp02:/opt/usdp-srv/srv/udp/2.0.0.0/hive/lib





```





## 使用ssb数据集进行造数30GB

具体使用方法参考：https://github.com/Kyligence/ssb-kylin

```
nohup bin/run.sh --scale 50    2>&1 & 
```









## hadoop cli测试

hadoop fs -ls jfs://myjfs/





## spark测试

scala> List("hello,pp,kkk").toDF.write.mode("overwrite").text("jfs://myjfs/demotxt")

scala> spark.read.text("jfs://myjfs/demotxt").show



## hive测试

hive >

```
CREATE TABLE IF NOT EXISTS person3
(
  name STRING,
  age INT
) LOCATION 'jfs://myjfs/tmp/person3';


```



## 使用ssb数据集测试hive和spark性能

用hive在juicefs中建好库表。

hive >

```
create database ssb_juicefs location 'jfs://myjfs/ssb';
use ssb_juicefs;
CREATE TABLE `lineorder`(
  `lo_orderkey` bigint, 
  `lo_linenumber` bigint, 
  `lo_custkey` int, 
  `lo_partkey` int, 
  `lo_suppkey` int, 
  `lo_orderdate` int, 
  `lo_orderpriotity` string, 
  `lo_shippriotity` int, 
  `lo_quantity` bigint, 
  `lo_extendedprice` bigint, 
  `lo_ordtotalprice` bigint, 
  `lo_discount` bigint, 
  `lo_revenue` bigint, 
  `lo_supplycost` bigint, 
  `lo_tax` bigint, 
  `lo_commitdate` int, 
  `lo_shipmode` string)
ROW FORMAT SERDE 
  'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe' 
WITH SERDEPROPERTIES ( 
  'field.delim'='|', 
  'serialization.format'='|') 
STORED AS INPUTFORMAT 
  'org.apache.hadoop.mapred.TextInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat';

CREATE TABLE `customer`(
  `c_custkey` int, 
  `c_name` string, 
  `c_address` string, 
  `c_city` string, 
  `c_nation` string, 
  `c_region` string, 
  `c_phone` string, 
  `c_mktsegment` string)
ROW FORMAT SERDE 
  'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe' 
WITH SERDEPROPERTIES ( 
  'field.delim'='|', 
  'serialization.format'='|') 
STORED AS INPUTFORMAT 
  'org.apache.hadoop.mapred.TextInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat';
  
  CREATE TABLE `dates`(
  `d_datekey` int, 
  `d_date` string, 
  `d_dayofweek` string, 
  `d_month` string, 
  `d_year` int, 
  `d_yearmonthnum` int, 
  `d_yearmonth` string, 
  `d_daynuminweek` int, 
  `d_daynuminmonth` int, 
  `d_daynuminyear` int, 
  `d_monthnuminyear` int, 
  `d_weeknuminyear` int, 
  `d_sellingseason` string, 
  `d_lastdayinweekfl` int, 
  `d_lastdayinmonthfl` int, 
  `d_holidayfl` int, 
  `d_weekdayfl` int)
ROW FORMAT SERDE 
  'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe' 
WITH SERDEPROPERTIES ( 
  'field.delim'='|', 
  'serialization.format'='|') 
STORED AS INPUTFORMAT 
  'org.apache.hadoop.mapred.TextInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat';
  
  CREATE TABLE `part`(
  `p_partkey` int, 
  `p_name` string, 
  `p_mfgr` string, 
  `p_category` string, 
  `p_brand` string, 
  `p_color` string, 
  `p_type` string, 
  `p_size` int, 
  `p_container` string)
ROW FORMAT SERDE 
  'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe' 
WITH SERDEPROPERTIES ( 
  'field.delim'='|', 
  'serialization.format'='|') 
STORED AS INPUTFORMAT 
  'org.apache.hadoop.mapred.TextInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat';
  
  CREATE TABLE `supplier`(
  `s_suppkey` int, 
  `s_name` string, 
  `s_address` string, 
  `s_city` string, 
  `s_nation` string, 
  `s_region` string, 
  `s_phone` string)
ROW FORMAT SERDE 
  'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe' 
WITH SERDEPROPERTIES ( 
  'field.delim'='|', 
  'serialization.format'='|') 
STORED AS INPUTFORMAT 
  'org.apache.hadoop.mapred.TextInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat';
  
  CREATE VIEW `p_lineorder` AS SELECT `lineorder`.`lo_orderkey`,     
`lineorder`.`lo_linenumber`,
`lineorder`.`lo_custkey`,  
`lineorder`.`lo_partkey`,     
`lineorder`.`lo_suppkey`,
`lineorder`.`lo_orderdate`,
`lineorder`.`lo_orderpriotity`,
`lineorder`.`lo_shippriotity`, 
`lineorder`.`lo_quantity`,     
`lineorder`.`lo_extendedprice`,
`lineorder`.`lo_ordtotalprice`,
`lineorder`.`lo_discount`,     
`lineorder`.`lo_revenue`,      
`lineorder`.`lo_supplycost`,   
`lineorder`.`lo_tax`,          
`lineorder`.`lo_commitdate`,   
`lineorder`.`lo_shipmode`,
`lineorder`.`lo_extendedprice`*`lineorder`.`lo_discount` AS `V_REVENUE`
FROM `LINEORDER`;
```



使用sparkshell将ssb数据集写到juicefs的库里

```
scala> spark.read.table("ssb.lineorder").write.insertInto("ssb_juicefs.lineorder")

scala> spark.read.table("ssb.supplier").write.insertInto("ssb_juicefs.supplier")

scala> spark.read.table("ssb.part").write.insertInto("ssb_juicefs.part")

scala> spark.read.table("ssb.dates").write.insertInto("ssb_juicefs.dates")

scala> spark.read.table("ssb.customer").write.insertInto("ssb_juicefs.customer")


```



使用sparksql对比跑分。

```
spark-sql \
  --master yarn \
  --num-executors 5 \
  --executor-memory 2G \
  --executor-cores 4 \
  --driver-memory 2G 


```



ssb数据集各表数据量统计。

```
customer  1500000
dates  2556
lineorder 200025884
part 1200000
supplier 100000
p_lineorder 200025884
```





**Q4.1**

```
select d_year, c_nation, sum(lo_revenue) - sum(lo_supplycost) as profit
from p_lineorder
left join dates on lo_orderdate = d_datekey
left join customer on lo_custkey = c_custkey
left join supplier on lo_suppkey = s_suppkey
left join part on lo_partkey = p_partkey
where c_region = 'AMERICA' and s_region = 'AMERICA' and (p_mfgr = 'MFGR#1' or p_mfgr = 'MFGR#2')
group by d_year, c_nation
order by d_year, c_nation;
```

HDFS => Time taken: 36.951 seconds, Fetched 35 row(s)

JuiceFS => Time taken: 49.893 seconds, Fetched 35 row(s)

##### Q4.2

```
select d_year, s_nation, p_category, sum(lo_revenue) - sum(lo_supplycost) as profit
from p_lineorder
left join dates on lo_orderdate = d_datekey
left join customer on lo_custkey = c_custkey
left join supplier on lo_suppkey = s_suppkey
left join part on lo_partkey = p_partkey
where c_region = 'AMERICA'and s_region = 'AMERICA'
and (d_year = 1997 or d_year = 1998)
and (p_mfgr = 'MFGR#1' or p_mfgr = 'MFGR#2')
group by d_year, s_nation, p_category
order by d_year, s_nation, p_category;
```

HDFS => Time taken: 32.196 seconds, Fetched 100 row(s)

JuiceFS => Time taken: 35.728 seconds, Fetched 100 row(s)

**Q4.3**

```
select d_year, s_city, p_brand, sum(lo_revenue) - sum(lo_supplycost) as profit
from p_lineorder
left join dates on lo_orderdate = d_datekey
left join customer on lo_custkey = c_custkey
left join supplier on lo_suppkey = s_suppkey
left join part on lo_partkey = p_partkey
where c_region = 'AMERICA'and s_nation = 'UNITED STATES'
and (d_year = 1997 or d_year = 1998)
and p_category = 'MFGR#14'
group by d_year, s_city, p_brand
order by d_year, s_city, p_brand;
```

HDFS => Time taken: 32.726 seconds, Fetched 800 row(s)

JuiceFS =>Time taken: 31.455 seconds, Fetched 800 row(s)

##### Q2.2

```
select sum(lo_revenue) as lo_revenue, d_year, p_brand
from p_lineorder
left join dates on lo_orderdate = d_datekey
left join part on lo_partkey = p_partkey
left join supplier on lo_suppkey = s_suppkey
where p_brand between 'MFGR#2221' and 'MFGR#2228' and s_region = 'ASIA'
group by d_year, p_brand
order by d_year, p_brand;
```

HDFS => Time taken: 31.59 seconds, Fetched 56 row(s)

JuiceFS => Time taken: 35.32 seconds, Fetched 56 row(s)