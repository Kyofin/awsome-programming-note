# Spark写入数据到Hudi并同步到Hive

## 版本说明

Hudi使用的是孵化版本 0.8

Apache Spark-2.4.6-2.11

HDP集群2.6.5.0：

1. hive-1.2.0
2. hdfs-2.7.3

## 下载hudi的hive依赖放上服务器

https://repo1.maven.org/maven2/org/apache/hudi/hudi-hadoop-mr-bundle/0.8.0/hudi-hadoop-mr-bundle-0.8.0.jar

复制到集群的hive的lib目录`/usr/hdp/2.6.5.0-292/hive`下即可。



## 用spark写入数据并同步元数据到hive

引入依赖。

```
 <!--    hudi同步元数据到hive-->
    <dependency>
        <groupId>org.apache.hive</groupId>
        <artifactId>hive-jdbc</artifactId>
        <version>1.2.1</version>
    </dependency>
     <!--        hudi需要的依赖开始-->
    <dependency>
        <groupId>org.apache.hudi</groupId>
        <artifactId>hudi-spark-bundle_${scala.version}</artifactId>
        <version>0.8.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.spark</groupId>
        <artifactId>spark-avro_${scala.version}</artifactId>
        <version>${spark.version}</version>
    </dependency>
```

执行代码如下。

```scala
package com.data.spark.datalake.hudi

import org.apache.hudi.DataSourceWriteOptions
import org.apache.hudi.config.HoodieWriteConfig
import org.apache.hudi.hive.MultiPartKeysValueExtractor
import org.apache.spark.sql.{SaveMode, SparkSession}

/**
 * 写入hudi数据时，同步元数据到hive。
 * 需要先将https://repo1.maven.org/maven2/org/apache/hudi/hudi-hadoop-mr-bundle/0.8.0/hudi-hadoop-mr-bundle-0.8.0.jar
 * 放入服务器hive的lib包中。
 */
object HiveSupportApp {
  def main(args: Array[String]): Unit = {

    System.setProperty("HADOOP_USER_NAME", "hive")

    val spark = SparkSession.builder()
      .master("local[*]")
      .enableHiveSupport()
      //hudi只支持kryo做序列化
      .config("spark.serializer", "org.apache.spark.serializer.KryoSerializer")
      // Uses Hive SerDe, this is mandatory for MoR tables
      .config("spark.sql.hive.convertMetastoreParquet", "false")
      // 调试线上HDFS
      .config("spark.hadoop.fs.defaultFS", "hdfs://hadoop-master:8020")
      // 调试线上Hive Metastore
      .config("spark.hadoop.hive.metastore.uris", "thrift://10.93.6.167:9083")
      .getOrCreate()

    import spark.implicits._

    val tableName = "hudi_archive_hive"
    val basePath = "/apps/hive/warehouse/hzk.db/" + tableName

    // Create a DataFrame
    val inputDF = Seq(
      ("100", "2015-01-01", "2015-01-01T13:51:39.340396Z"),
      ("101", "2015-01-01", "2015-01-01T12:14:58.597216Z"),
      ("102", "2015-01-01", "2015-01-01T13:51:40.417052Z"),
      ("103", "2015-01-01", "2015-01-01T13:51:40.519832Z"),
      ("104", "2015-01-02", "2015-01-01T12:15:00.512679Z"),
      ("105", "2015-01-02", "2015-01-01T13:51:42.248818Z")
    ).toDF("id", "creation_date", "last_update_time")

    //Specify common DataSourceWriteOptions in the single hudiOptions variable
    val hudiOptions = Map[String, String](
      HoodieWriteConfig.TABLE_NAME -> tableName,
      DataSourceWriteOptions.TABLE_TYPE_OPT_KEY -> "COPY_ON_WRITE",
      DataSourceWriteOptions.RECORDKEY_FIELD_OPT_KEY -> "id",
      DataSourceWriteOptions.PARTITIONPATH_FIELD_OPT_KEY -> "creation_date",
      DataSourceWriteOptions.PRECOMBINE_FIELD_OPT_KEY -> "last_update_time",
      // 以hive的分区目录命名方式
      DataSourceWriteOptions.HIVE_STYLE_PARTITIONING_OPT_KEY -> "true",
      //配置hive相关
      DataSourceWriteOptions.HIVE_SYNC_ENABLED_OPT_KEY -> "true",
      DataSourceWriteOptions.HIVE_DATABASE_OPT_KEY -> "hzk",
      DataSourceWriteOptions.HIVE_TABLE_OPT_KEY -> tableName,
      DataSourceWriteOptions.HIVE_PARTITION_FIELDS_OPT_KEY -> "creation_date",
      DataSourceWriteOptions.HIVE_URL_OPT_KEY -> "jdbc:hive2://10.93.6.167:10000",
      DataSourceWriteOptions.HIVE_USER_OPT_KEY -> "hive",
      DataSourceWriteOptions.HIVE_PASS_OPT_KEY -> "hive",
      DataSourceWriteOptions.HIVE_PARTITION_EXTRACTOR_CLASS_OPT_KEY -> classOf[MultiPartKeysValueExtractor].getName
    )

    // Write the DataFrame as a Hudi dataset
    inputDF.write
      .format("org.apache.hudi")
      .option(DataSourceWriteOptions.OPERATION_OPT_KEY, DataSourceWriteOptions.INSERT_OPERATION_OPT_VAL)
      .options(hudiOptions)
      .mode(SaveMode.Overwrite)
      .save(basePath)

    spark.sql("drop table hzk.hudi_cts")
    spark.sql("create table hzk.hudi_cts(id int,cnt int )stored as parquet")
    spark.sql("insert OVERWRITE table hzk.hudi_cts  select id ,count(1) cnt from hzk.hudi_archive_hive group by id")

    spark.stop()
  }


}

```

运行spark作业后，登录服务器查看hdfs的数据。

![image-20210423152326378](http://image-picgo.test.upcdn.net/img/20210423152326.png)

![image-20210423152332138](http://image-picgo.test.upcdn.net/img/20210423152332.png)

登录可以看到hive中已经建了对应的hudi表了。

![image-20210423152058349](http://image-picgo.test.upcdn.net/img/20210423152058.png)