# Iceberg0.12.1快速体验

## 环境准备

- spark3.1.2
- hive2.3.9
- hadoop2.7.3
- jdk1.8
- iceberg0.12.1

可以使用docker来启动上述服务。

```
docker pull peterpoker/spark3-all-in-one
```

参考地址：https://hub.docker.com/r/peterpoker/spark3-all-in-one



## 下载Iceberg包

```
wget https://repo1.maven.org/maven2/org/apache/iceberg/iceberg-spark3-runtime/0.12.1/iceberg-spark3-runtime-0.12.1.jar
```





## 启动spark-shell测试

```shell
bin/spark-shell  --master local \
  --conf spark.sql.extensions=org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions \
    --conf spark.sql.catalog.spark_catalog=org.apache.iceberg.spark.SparkSessionCatalog \
    --conf spark.sql.catalog.spark_catalog.type=hive \
    --conf spark.sql.catalog.ice_catalog=org.apache.iceberg.spark.SparkCatalog  \
    --conf spark.sql.catalog.ice_catalog.type=hive \
    --conf spark.sql.catalog.local=org.apache.iceberg.spark.SparkCatalog \
    --conf spark.sql.catalog.local.type=hadoop \
    --conf spark.sql.catalog.local.warehouse=/apps/iceberg/warehouse  \
  --jars  iceberg/iceberg-spark3-runtime-0.12.1.jar
```

该命令启动后，会存在三个catalog。

一个是`spark_catalog`，该catalog是hive表和Iceberg表共存的，而且元数据和spark内置的一样。如果是hive环境的，就存储在hive的mysql里，因此对hive的版本有要求。

一个是`local`，该catalog只能管理Iceberg表，而且元数据信息是存储在hadoop的文件系统里的。

一个是`ice_catalog`，该catalog只能管理Iceberg表，而元数据存储和spark内置的一样，具体同spark_catalog。



## 快速体验名为Spark_Catalog的Catalog

### createTable

建表时如果不指定catalog,默认会在`spark_catlog`中建表。

```scala
// 默认会在spark_catalog.default里建表
spark.sql("  create table  ice_t1 (id int) using iceberg;")  
  // 不使用using Iceberg，就是建的hive表
  spark.sql("    create table  hive_t1 (id int) ;")
  
```

这里的spark_catalog使用的元数据信息是和hive一起的，所以建表完成后，在hive的warehouse路径里和hive的元数据里是可以看到这张表的。

![image-20220106170707267](http://image-picgo.test.upcdn.net/img/20220106170707.png)

![image-20220106170812530](http://image-picgo.test.upcdn.net/img/20220106170812.png)

可以看到ice_t1表是外部表。当执行删表后，hdfs里还是存在的。

```scala
spark.sql(" drop table ice_t1")
```

![image-20220106171242048](http://image-picgo.test.upcdn.net/img/20220106171242.png)



### MergeTable

merge Into可以用一个sql执行更新和追加的操作。

Merge INTO比Insert overwrite更推荐使用，因为Iceberg可以实现只更新替换影响到的数据文件。

```scala
scala> spark.sql(" create table target (id int,count int) using iceberg;")
scala> spark.sql(" create table update (id int,count int) using iceberg;")
// 初始化数据
scala> spark.sql("insert into target values(1,10),(2,10),(3,20)")
scala> spark.sql("insert into update values(5,100),(2,20),(3,10)")
// merge操作
scala> spark.sql("MERGE INTO target t USING (SELECT * FROM update) u ON t.id = u.id WHEN MATCHED THEN UPDATE SET t.count = t.count + u.count WHEN NOT MATCHED THEN INSERT * ")

// 结果
scala> spark.table("target").show
+---+-----+
| id|count|
+---+-----+
|  1|   10|
|  2|   30|
|  3|   30|
|  5|  100|
+---+-----+
```



### 查看Iceberg表的元数据

在Iceberg0.12.1版本和spark3.0使用中，如果Iceberg表是存在于`spark_catalog`中的，是不能直接用sql访问元数据的，只能通过`DataFrameReade`r的API。

```scala
scala> spark.read.format("iceberg").load("default.target.manifests").show
```

![image-20220107115603067](http://image-picgo.test.upcdn.net/img/20220107115603.png)

```scala
scala> spark.read.format("iceberg").load("default.target.history").show
```

![image-20220107115628967](http://image-picgo.test.upcdn.net/img/20220107115629.png)

```
scala> spark.read.format("iceberg").load("default.target.files").show
```

![image-20220107143827940](http://image-picgo.test.upcdn.net/img/20220107143828.png)



### TimeTravel

时间旅行功能能访问某个时间点的数据快照，可以通过时间戳或者快照id访问。

通过Iceberg的元数据查询到历史数据。

```scala
spark.read.format("iceberg").load("default.target.history").show(false)
```

![image-20220107144609572](http://image-picgo.test.upcdn.net/img/20220107144609.png)

上图可知道快照id为`3925874089226408627`，通过DataFrameReader读取。

```scala
scala> spark.read.option("snapshot-id", "3925874089226408627").format("iceberg").load("default.target").show
```

![image-20220107144655676](http://image-picgo.test.upcdn.net/img/20220107144655.png)





### Write With Dataframe

Spark3支持DataFrameWriterV2的API。

下面的操作等同于 SQL：

- `df.writeTo(t).create()` is equivalent to `CREATE TABLE AS SELECT`
- `df.writeTo(t).replace()` is equivalent to `REPLACE TABLE AS SELECT`
- `df.writeTo(t).append()` is equivalent to `INSERT INTO`
- `df.writeTo(t).overwritePartitions()` is equivalent to dynamic `INSERT OVERWRITE`



读一个DF的数据追加到一张Iceberg表

```
scala> spark.read.format("iceberg").load("default.target").writeTo("target").append
```

由于在spark_catalog下，默认是建的Parquet表不是Iceberg表，因此如果执行`df.writeTo(t).create()`建的将不是一张Iceberg表。



### Update

行级别更新。

```
scala> spark.sql("update target set count = 4000 where id =1")
res56: org.apache.spark.sql.DataFrame = []

scala> spark.table("target").show
+---+-----+
| id|count|
+---+-----+
|  1| 4000|
|  2|   30|
|  3|   30|
|  5|  100|
|  1| 4000|
|  2|   30|
|  3|   30|
|  5|  100|
+---+-----+
```





### Delete From

行级别删除。

```
scala> spark.sql("delete from target  where id =1")
res59: org.apache.spark.sql.DataFrame = []

scala> spark.table("target").show
+---+-----+
| id|count|
+---+-----+
|  2|   30|
|  3|   30|
|  5|  100|
|  2|   30|
|  3|   30|
|  5|  100|
+---+-----+
```



## 快速体验名为Local的Catalog

### createTable

当在名为local的catalog中建表，注意这里不用说明`using iceberg`也是可以的。

```
scala> spark.sql("create table local.ice_t3 (id int)")
```

建表成功后，可以在hdfs上看到该表的存储空间

![image-20220107152706541](http://image-picgo.test.upcdn.net/img/20220107152706.png)

以及在hive的元数据里是不能看到该表的信息的。

![image-20220107152743822](http://image-picgo.test.upcdn.net/img/20220107152743.png)



### Write With Dataframe

当在名为local的catalog中，使用datafarme的API，是可以直接将DF的数据创建一张新的Iceberg表的。

```
scala> spark.read.format("iceberg").load("spark_catalog.default.target").writeTo("local.ice_t4").create
```

![image-20220107153015714](http://image-picgo.test.upcdn.net/img/20220107153015.png)

如果建表时，指定多级的namespace，则会变成层级目录。

```
scala> spark.sql("create table local.default.ice_t6 (id int)")


scala> spark.sql("create table local.ice_t6 (id int)")
```

上面这两个语句，就是会在路径`/ice_t6`和`/default/ice_t6`里存储Iceberg表。

![image-20220107175530406](http://image-picgo.test.upcdn.net/img/20220107175530.png)

当我们执行下面语句，则会按照这个多级namespace去建多级目录。这个特性，是只有用`spark.sql.catalog.local.type=hadoop`才会存在。

```
scala> spark.sql("create table local.default.hh.ice_t6 (id int)")
```

![image-20220107175646069](http://image-picgo.test.upcdn.net/img/20220107175646.png)

## 快速体验名为ice_catalog的Catalog

### createTable

当在名为ice_catalog的catalog中建表的话，是不需要声明`using iceberg`，默认就是Iceberg表。

```
scala> spark.sql("create table ice_catalog.default.ice_t88 (id int)")
```

![image-20220108113334319](http://image-picgo.test.upcdn.net/img/20220108113334.png)

当尝试在这个catalog下建hive表时，是不行的。

```
scala> spark.sql("CREATE TABLE IF NOT EXISTS ice_catalog.default.src (key INT, value STRING) USING hive")
```

报错如下：

![image-20220108113737625](http://image-picgo.test.upcdn.net/img/20220108113737.png)

所以该catalog只是将元数据存储到hive的元数据库mysql里，但是该catalog只能建Iceberg表，不能建hive表。