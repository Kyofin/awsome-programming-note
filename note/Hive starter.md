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



## idea导入hive源码

这里下载的是cdh版本的hive源码。

在根目录下执行源码编译。

```shell
/cdh5.16.2/hive-1.1.0-cdh5.16.2 » mvn clean package -DskipTests -Phadoop-2
```

![image-20200729092203625](http://image-picgo.test.upcdn.net/img/20200729092203.png)

注意：

1. 编译成功后用idea导入时需要将右边maven插件中需要选中hadoop-2的profile

2. 最好在idea中rebuild整个project

3. 将环境中的hive-site.xml放入resource目录

   ![image-20200730172414379](http://image-picgo.test.upcdn.net/img/20200730172414.png)

4. 启动CliDriver测试



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



### 级联删除数据库

```
drop database tpcds_bin_partitioned_orc_5  cascade;
```



### 添加列

```sql
ALTER TABLE table_name ADD COLUMNS (col_name STRING);  //在所有存在的列后面，但是在分区列之前添加一列
```

 

### 修改列

```sql
CREATE TABLE test_change (a int, b int, c int);

// will change column a's name to a1
ALTER TABLE test_change CHANGE a a1 INT; 

// will change column a's name to a1, a's data type to string, and put it after column b. The new table's structure is: b int, a1 string, c int
ALTER TABLE test_change CHANGE a a1 STRING AFTER b; 

// will change column b's name to b1, and put it as the first column. The new table's structure is: b1 int, a string, c int
ALTER TABLE test_change CHANGE b b1 INT FIRST; 
```



### 修改表属性（内外表转换）

```sql
alter table table_name set TBLPROPERTIES ('EXTERNAL'='TRUE');  //内部表转外部表 
alter table table_name set TBLPROPERTIES ('EXTERNAL'='FALSE');  //外部表转内部表
```

 

### 表的重命名

```sql
ALTER TABLE table_name RENAME TO new_table_name
```



### 数据插入

​        hive是基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张数据库表，并提供简单的sql查询功能，可以将sql语句转换为MapReduce任务进行运行。通常hive包括以下四种数据导入方式：

- 从本地文件系统中导入数据到Hive表；

- 从HDFS上导入数据到Hive表；

- 在创建表的时候通过从别的表中查询出相应的记录并插入到所创建的表中；

- 从别的表中查询出相应的数据并导入到Hive表中。

#### INSERT INTO

使用样例

 ```SQL
   insert into tablename1 select a, b, c from tablename2;
 ```

#### INSERT OVERWRITE

使用样例

```sql
  insert overwrite table tablename1 select a, b, c from tablename2
```

两者的异同
        insert into 与 insert overwrite 都可以向hive表中插入数据，**但是insert into直接追加到表中数据的尾部，而insert overwrite会重写数据，既先进行删除，再写入。**如果存在分区的情况，insert overwrite会只重写当前分区数据。

### 表分区

分区字段也是可以作为where条件使用的。

#### 创建分区表

```sql
create table logs(ts bigint,line string)partitioned by (dt String,country string)
```

创建表后不会生成分区目录

![](http://image-picgo.test.upcdn.net/img/20191224104106.png)

插入数据后才会有分区目录，可以看到数据只包含非分区列的值。

```sql
insert into logs values(1,'/root/hive/partitions/file1','2018','gz');

insert into logs partition (dt='2018',country='china') values(2,'/root/hive/partitions/file2') ;
```

![](http://image-picgo.test.upcdn.net/img/20191224134756.png)



#### 查看分区

```sql
show partitions table_name;
```



#### 导入数据并指定分区

（没有这个分区则会自动创建分区）

```
LOAD DATA LOCAL INPATH '/Users/huzekang/tmp/data' OVERWRITE  INTO TABLE logs PARTITION (dt='2018',country='UK')
```

有LOCAL表示从本地文件系统加载（文件会被拷贝到HDFS中）
无LOCAL表示从HDFS中加载数据（注意：文件直接被移动！！！而不是拷贝！！！ 并且。。文件名都不带改的。。）
OVERWRITE  表示是否覆盖表中数据（或指定分区的数据）（没有OVERWRITE  会直接APPEND，而不会滤重!）



#### 添加分区

```sql
ALTER TABLE table_name ADD PARTITION (partCol = 'value1') location 'loc1'; //示例
ALTER TABLE table_name ADD IF NOT EXISTS PARTITION (dt='20130101') LOCATION '/user/hadoop/warehouse/table_name/dt=20130101'; //一次添加一个分区

ALTER TABLE page_view ADD PARTITION (dt='2008-08-08', country='us') location '/path/to/us/part080808' PARTITION (dt='2008-08-09', country='us') location '/path/to/us/part080809';  //一次添加多个分区
```

 

#### 删除分区

```sql
ALTER TABLE login DROP IF EXISTS PARTITION (dt='2008-08-08');

ALTER TABLE page_view DROP IF EXISTS PARTITION (dt='2008-08-08', country='us');
```

 

#### 修改分区

```
ALTER TABLE table_name PARTITION (dt='2008-08-08') SET LOCATION "new location";
ALTER TABLE table_name PARTITION (dt='2008-08-08') RENAME TO PARTITION (dt='20080808');
```

 

## 

### 移除表

表结构，表数据都被移除。

```
drop table logs;
```



### 删除表数据

#### hive清空表中数据

```javascript
truncate table table_name;
```

表结构仍保留，分区仍保留，但里面的数据文件全没了。

![](http://image-picgo.test.upcdn.net/img/20200106143030.png)



#### hive按分区删除数据

```javascript
alter table table_name drop partition (partition_name='分区名')
```

整个分区文件夹都没了。

![](http://image-picgo.test.upcdn.net/img/20200106143212.png)



#### 删除部分满足条件的数据

##### 一、有partition表

删除具体partition

```
alter table table_name drop partition(partiton_name='value')
```

删除partition内的部分信息(INSERT OVERWRITE TABLE）

```
INSERT OVERWRITE TABLE table_name PARTITION(dt='v3') SELECT column1,column2 FROM alpha_sales_staff_info WHERE dt='v3' AND category is not null;
```


重新把对应的partition信息写一遍，通过WHERE 来限定需要留下的信息，没有留下的信息就被删除了。

##### 二、无partiton表

```
INSERT OVERWRITE TABLE dpc_test SELECT * FROM dpc_test WHERE age is not null;
```



### 函数

#### 创建udf

```SQL
add jar hdfs://cdh04:8020/escheduler/huzekang/udfs/hive-third-functions-2.2.1-shaded.jar
create temporary function pinyin as 'com.github.aaronshan.functions.string.UDFChineseToPinYin'
```

有temporary则为临时的。



#### 描述函数用法

```SQL
describe function extended md5;
```

![](http://image-picgo.test.upcdn.net/img/20200512174213.png)



## 读取外部表

### 读取kafka数据

```SQL
CREATE EXTERNAL TABLE test_kafka2
(`message` string)
 STORED BY 'org.apache.hadoop.hive.kafka.KafkaStorageHandler'
TBLPROPERTIES
 ("kafka.topic" = "maxwell",
 "kafka.bootstrap.servers"="cdh05:6667"
);

SELECT * from test_kafka2
```



## 开窗函数

参考：https://www.jianshu.com/p/a2f22af39072

### 1. 介绍

普通聚合函数聚合的行集是组，开窗函数聚合的行集是窗口。
因此，普通聚合函数每组（Group by）只有一个返回值，而开窗函数则可以为窗口中的每行都返回一个值。

### 1.1 基础结构

```swift
分析函数(如：sum(), max(), row_number()...) + 窗口子句(over函数)
```

### 1.2 over函数

`over(partition by [column_n] order by [column_m])`先按照`column_n`分区，相同的`column_n`分为一区，每个分区根据`column_m`排序（默认升序）。

### 1.3 测试数据

建表并插入数据



```csharp
-- 建表
create table student_scores(
  id int,
  studentId int,
  language int,
  math int,
  english int,
  classId string,
  departmentId string
);
-- 写入数据
insert into table student_scores values 
  (1,111,68,69,90,'class1','department1'),
  (2,112,73,80,96,'class1','department1'),
  (3,113,90,74,75,'class1','department1'),
  (4,114,89,94,93,'class1','department1'),
  (5,115,99,93,89,'class1','department1'),
  (6,121,96,74,79,'class2','department1'),
  (7,122,89,86,85,'class2','department1'),
  (8,123,70,78,61,'class2','department1'),
  (9,124,76,70,76,'class2','department1'),
  (10,211,89,93,60,'class1','department2'),
  (11,212,76,83,75,'class1','department2'),
  (12,213,71,94,90,'class1','department2'),
  (13,214,94,94,66,'class1','department2'),
  (14,215,84,82,73,'class1','department2'),
  (15,216,85,74,93,'class1','department2'),
  (16,221,77,99,61,'class2','department2'),
  (17,222,80,78,96,'class2','department2'),
  (18,223,79,74,96,'class2','department2'),
  (19,224,75,80,78,'class2','department2'),
  (20,225,82,85,63,'class2','department2');
```

### 1.4 窗口含义



```csharp
select studentId,math,departmentId,classId,
-- 符合所有条件的行作为窗口，这里符合department1的有9个
count(math) over() as count1,
-- 按照classId分组的所有行作为窗口
count(math) over(partition by classId) as count2,
-- 按照classId分组、按照math排序的所有行作为窗口
count(math) over(partition by classId order by math) as count3,
-- 按照classId分组、按照math排序，当前行向前1行向后2行的行作为窗口
count(math) over(partition by classId order by math rows between 1 preceding and 2 following) as count4,
-- 按照classId分组、按照math排序，当前行向后所有行作为窗口
count(math) over(partition by classId order by math rows between current row and unbounded following) as count5
from student_scores where departmentId='department1';
```

结果：

```undefined
studentId    math    departmentId    classId    count1    count2    count3    count4    count5
111          69      department1     class1     9         5         1         3         5
113          74      department1     class1     9         5         2         4         4
112          80      department1     class1     9         5         3         4         3
115          93      department1     class1     9         5         4         3         2
114          94      department1     class1     9         5         5         2         1
124          70      department1     class2     9         4         1         3         4
121          74      department1     class2     9         4         2         4         3
123          78      department1     class2     9         4         3         3         2
122          86      department1     class2     9         4         4         2         1
```

结果解析：

```undefined
studentId=115，
count1为departmentId=department1的行数为9，
count2为分区class1中的行数5，
count3为分区class1中math<=93的行数4，
count4为分区class1中math值向前1行和向后2行的行数3，
count5为分区class1中当前math值到class1分区结束的行数2。
```

上面可以看到：如果不指定`ROWS BETWEEN`，默认统计窗口是从起点到当前行
 **关键是`ROWS BETWEEN`，也叫做`window子句`。**
 `PRECEDING`：向前
 `FOLLOWING`：向后
 `CURRENT ROW`：当前行
 `UNBOUNDED`：无边界，`UNBOUNDED PRECEDING`表示从最前面的起点开始，`UNBOUNDED FOLLOWING`表示到最后面的终点

##### 开窗函数可以粗略地分为两类：聚合开窗函数和排序开窗函数。



### 2. 聚合开窗函数

### 2.1 sum函数



```csharp
select studentId,math,departmentId,classId,
sum(math) over() as sum1,
sum(math) over(partition by classId) as sum2,
sum(math) over(partition by classId order by math) as sum3,
sum(math) over(partition by classId order by math rows between 1 preceding and 2 following) as sum4,
sum(math) over(partition by classId order by math rows between current row and unbounded following) as sum5
from student_scores where departmentId='department1';

-- 结果解析：类似count()函数
```

> min()，max()，avg()都与count()类似



## Hive 日志记录

### 输出日志文件

```
hive --hiveconf hive.log.dir=/tmp/hive/huzekang
```

hive cli启动后可以看到日志输出到文件。

![image-20200915145339156](http://image-picgo.test.upcdn.net/img/20200915145339.png)

### 详细日志打印到控制台

```
hive --hiveconf hive.root.logger=INFO,console
```

### 日志中输出查询计划

```
hive --hiveconf hive.log.explain.output=true
```





## HA的连接方式

使用beeline连接

```sql
!connect jdbc:hive2://localhost:2181/;serviceDiscoveryMode=zooKeeper;zooKeeperNamespace=kyuubi-huzekang

```

其中zooKeeperNamespace自行上zk上看。



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



## Hive Cli 查看orc文件元数据

```
hive --orcfiledump hdfs:///user/hive/warehouse/user_copy1/ds=20200911/DATAX_15802138783242_1_0
```



## 使用LineageLogger获取血源关系

启动hive cli，并打印详细日志。

```
hive --hiveconf hive.root.logger=INFO,console
```

LineageLogger是hive默认就有的。只要注册到post的hook中即可。

```
set hive.exec.post.hooks=org.apache.hadoop.hive.ql.hooks.LineageLogger;
```

使用查询sql创建表

```sql
create table t_x_0 as select u.id,city,concat(name,",",cc) from tutu_user u left join (
    select id ,concat_ws("|",collect_list(concat(year," ",month," ",traffic))) cc
    from tutu_access group by id) a on a.id = u.id;
```

![image-20200915173306818](http://image-picgo.test.upcdn.net/img/20200915173306.png)



## Hook开发

### 参考文档

https://www.slideshare.net/julingks/apache-hive-hooksminwookim130813

![image-20200915164208344](http://image-picgo.test.upcdn.net/img/20200915164208.png)

![image-20200915170514946](http://image-picgo.test.upcdn.net/img/20200915170515.png)

### 依赖

```
  <dependency>
            <groupId>org.apache.hive</groupId>
            <artifactId>hive-exec</artifactId>
            <version>${hive.version}</version>
        </dependency>

```

### 计算sql执行的hook代码参考

可以参考 `org.apache.hadoop.hive.ql.hooks.LineageLogger`

```java
public class HiveExampleHook implements ExecuteWithHookContext {
	@Override
	public void run(HookContext hookContext) throws Exception {
		System.out.println("==========  " + System.getProperty("user.name") + "正在执行hive作业  ========");
		final ArrayList<ReadEntity> readEntities = new ArrayList<>(hookContext.getInputs());
		readEntities.forEach(e -> {
			System.out.println("读取表：" + e.getName());
			final List<String> accessedColumns = e.getAccessedColumns();
			System.out.println("读取到的字段：");
			accessedColumns.forEach(System.out::println);
		});
		hookContext.getOutputs().forEach(e -> System.out.println("写出表：" + e.getName()));

	}
}

```



### DDL的sql执行的hook代码参考

```JAVA
public class HiveMetaStoreHook extends MetaStoreEventListener {


  public HiveMetaStoreHook(Configuration config) {
    super(config);
  }

  @Override
  public void onCreateTable(CreateTableEvent tableEvent) throws MetaException {
    System.out.println("======================== HiveMetastoreHook (onCreateTable) =========================");
    System.out.println("已建表："+tableEvent.getTable().getTableName());
    System.out.println("表存储路径："+tableEvent.getTable().getSd().getLocation());
  }
}
```



### 运行Cli

添加hook的jar到hive中，然后指定hook的周期节点。

```shell
add jar /Volumes/Samsung_T5/huzekang/study/bigdata-hadoop/target/bigdata-hadoop-1.0.jar;

set hive.exec.pre.hooks=com.data.hive.hook.HiveExampleHook;

set hive.metastore.event.listeners=com.data.hive.hook.HiveMetaStoreHook;
```

### 效果

执行sql根据查询结果创建表。

```sql
create table t_x_0 as select u.id,city,concat(name,",",cc) from tutu_user u left join (
    select id ,concat_ws("|",collect_list(concat(year," ",month," ",traffic))) cc
    from tutu_access group by id) a on a.id = u.id;
```

![image-20200915170335837](http://image-picgo.test.upcdn.net/img/20200915170336.png)

```SQL
create table hoook(id int);
```

![image-20200915170924021](http://image-picgo.test.upcdn.net/img/20200915170924.png)