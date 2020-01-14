# Hive starter

## 创建不同格式表

测试参考：http://183.6.50.10:4999/web/#/7?page_id=191

### ORC

```
create table hdfswriter2(name string,price long,date_time date,is_kill boolean,price2 double) STORED AS ORC;
```

### Parquet

```
create table storage_format_parquet_hive
STORED AS Parquet
as
select id,indic_serial_number,report_date_serial,fee_code
from tb_cis_inhos_dis_reg
```





## 内置函数

共291个。

![](http://image-picgo.test.upcdn.net/img/20200110185655.png)

```
--查看所有内置函数
SHOW FUNCTIONS ;
--查看某个函数的描述
DESCRIBE FUNCTION !;
 --查看某个函数的具体使用方法
DESCRIBE FUNCTION EXTENDED !;

```



## 表分区

分区字段也是可以作为where条件使用的。

### 创建分区表

```
create table logs(ts bigint,line string)partitioned by (dt String,country string)
```

创建表后不会生成分区目录

![](http://image-picgo.test.upcdn.net/img/20191224104106.png)

插入数据后才会有分区目录，可以看到数据只包含非分区列的值。

```
insert into logs values(1,'/root/hive/partitions/file1','2018','gz');

insert into logs partition (dt='2018',country='china') values(2,'/root/hive/partitions/file2') ;
```

![](http://image-picgo.test.upcdn.net/img/20191224134756.png)



### 查看分区

```
show partitions table_name;
```



### 导入数据并指定分区

（没有这个分区则会自动创建分区）

```
LOAD DATA LOCAL INPATH '/Users/huzekang/tmp/data' OVERWRITE  INTO TABLE logs PARTITION (dt='2018',country='UK')
```

有LOCAL表示从本地文件系统加载（文件会被拷贝到HDFS中）
无LOCAL表示从HDFS中加载数据（注意：文件直接被移动！！！而不是拷贝！！！ 并且。。文件名都不带改的。。）
OVERWRITE  表示是否覆盖表中数据（或指定分区的数据）（没有OVERWRITE  会直接APPEND，而不会滤重!）



### 添加分区

```sql
ALTER TABLE table_name ADD PARTITION (partCol = 'value1') location 'loc1'; //示例
ALTER TABLE table_name ADD IF NOT EXISTS PARTITION (dt='20130101') LOCATION '/user/hadoop/warehouse/table_name/dt=20130101'; //一次添加一个分区

ALTER TABLE page_view ADD PARTITION (dt='2008-08-08', country='us') location '/path/to/us/part080808' PARTITION (dt='2008-08-09', country='us') location '/path/to/us/part080809';  //一次添加多个分区
```

 

### 删除分区

```sql
ALTER TABLE login DROP IF EXISTS PARTITION (dt='2008-08-08');

ALTER TABLE page_view DROP IF EXISTS PARTITION (dt='2008-08-08', country='us');
```

 

### 修改分区

```
ALTER TABLE table_name PARTITION (dt='2008-08-08') SET LOCATION "new location";
ALTER TABLE table_name PARTITION (dt='2008-08-08') RENAME TO PARTITION (dt='20080808');
```

 

## 常用命令

### 查看表信息（最全）

其中`transient_lastDdlTime`是最后修改时间（秒），`CreateTime`是创建时间。

```shell
hive> desc formatted user;
OK
# col_name            	data_type           	comment

id                  	bigint
name                	string              	姓名
age                 	bigint              	年龄

# Partition Information
# col_name            	data_type           	comment

year                	string

# Detailed Table Information
Database:           	default
Owner:              	anonymous
CreateTime:         	Fri Oct 18 16:45:10 CST 2019
LastAccessTime:     	UNKNOWN
Protect Mode:       	None
Retention:          	0
Location:           	hdfs://localhost:8020/hive/warehouse/user
Table Type:         	MANAGED_TABLE
Table Parameters:
	comment             	用户表
	numPartitions       	1
	transactional       	true
	transient_lastDdlTime	1571388310

# Storage Information
SerDe Library:      	org.apache.hadoop.hive.ql.io.orc.OrcSerde
InputFormat:        	org.apache.hadoop.hive.ql.io.orc.OrcInputFormat
OutputFormat:       	org.apache.hadoop.hive.ql.io.orc.OrcOutputFormat
Compressed:         	No
Num Buckets:        	2
Bucket Columns:     	[id]
Sort Columns:       	[]
Storage Desc Params:
	field.delim         	\t
	serialization.format	\t

```







### 查看建表语句

可以看**分区字段**和**普通列字段**还有**存储格式**等。

```
show create table logs;
```



### 添加列

```
ALTER TABLE table_name ADD COLUMNS (col_name STRING);  //在所有存在的列后面，但是在分区列之前添加一列
```

 

### 修改列

```
CREATE TABLE test_change (a int, b int, c int);

// will change column a's name to a1
ALTER TABLE test_change CHANGE a a1 INT; 

// will change column a's name to a1, a's data type to string, and put it after column b. The new table's structure is: b int, a1 string, c int
ALTER TABLE test_change CHANGE a a1 STRING AFTER b; 

// will change column b's name to b1, and put it as the first column. The new table's structure is: b1 int, a string, c int
ALTER TABLE test_change CHANGE b b1 INT FIRST; 
```



### 修改表属性

```
alter table table_name set TBLPROPERTIES ('EXTERNAL'='TRUE');  //内部表转外部表 
alter table table_name set TBLPROPERTIES ('EXTERNAL'='FALSE');  //外部表转内部表
```

 

### 表的重命名

```
ALTER TABLE table_name RENAME TO new_table_name
```



## 移除表

表结构，表数据都被移除。

```
drop table logs;
```



## 删除表数据

### hive清空表中数据

```javascript
truncate table table_name;
```

表结构仍保留，分区仍保留，但里面的数据文件全没了。

![](http://image-picgo.test.upcdn.net/img/20200106143030.png)



### hive按分区删除数据

```javascript
alter table table_name drop partition (partition_name='分区名')
```

整个分区文件夹都没了。

![](http://image-picgo.test.upcdn.net/img/20200106143212.png)



### 删除部分满足条件的数据

#### 一、有partition表

删除具体partition

```
alter table table_name drop partition(partiton_name='value')
```

删除partition内的部分信息(INSERT OVERWRITE TABLE）

```
INSERT OVERWRITE TABLE table_name PARTITION(dt='v3') SELECT column1,column2 FROM alpha_sales_staff_info WHERE dt='v3' AND category is not null;
```


重新把对应的partition信息写一遍，通过WHERE 来限定需要留下的信息，没有留下的信息就被删除了。

#### 二、无partiton表

```
INSERT OVERWRITE TABLE dpc_test SELECT * FROM dpc_test WHERE age is not null;
```





## 基准测试

Hive基准测试工具工具，可用来造数测试Hive基本性能

Github：https://github.com/hortonworks/hive-testbench/

**TPC-DS：提供一个公平和诚实的业务和数据模型，99个案例**
**TPC-H：面向商品零售业的决策支持系统测试基准，定义了8张表，22个查询**

```
wget https://github.com/hortonworks/hive-testbench/archive/hive14.zip
unzip hive14.zip
cd hive-testbench-hive14/
./tpcds-build.sh
./tpcds-setup.sh 1000 //生成1000G的hive表数据集
FORMAT=parquet ./tpcds-setup.sh 10 //生成10G的parquet格式的hive表
```





