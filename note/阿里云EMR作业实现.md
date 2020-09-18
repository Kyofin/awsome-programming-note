# 阿里云EMR作业实现

## flink

**说明:** Flink命令行任务，请填写flink命令行参数, 实际会执行如下命令: flink <PARAMS>

**示例:** run -m yarn-cluster -yD taskmanager.network.memory.fraction=0.4 -yD akka.ask.timeout=60s -yjm 2048 -ytm 2048 -ys 4 -yn 14 -c com.aliyun.emr.checklist.benchmark.FlinkWordCount -p 56 ossref://sample-bucket/sample.jar --input oss://sample-bucket/flink-input --output oss://sample-bucket/flink-output



## hive

**说明:** Hive命令行任务，请填写hive cli命令行参数，实际会执行如下命令:hive <PARAMS>。

**示例:** -f ./example.hql



## hivesql

**说明:** Hive SQL，请填写SQL代码，实际会执行如下命令:hive -e <SQL>。

**示例:**

--- sql example

--- sql1

SELECT * from test1

WHERE id='xxxx';

--- sql2

SELECT count(1) from test1

WHERE id='xxxx';



## spark

**说明:** Spark命令行任务，请填写spark-submit命令行参数, 实际会执行如下命令: spark-submit <PARAMS>.

**示例:** --class org.apache.spark.examples.SparkPi --master local[8] /path/to/examples.jar 100



## spark shell

**说明:** Spark Shell任务，请填写Scala代码, 其中Spark context对应的变量名为'sc', Spark session对应的变量为'spark'. 系统会将该scala代码保存到集群上,使用`cat <path/to/the/scala/source.scala>|spark-shell`的方式执行.

**示例:** 
val t = spark.read.textFile("README.md") println(t.count())



## spark sql

**说明:** Spark SQL，请填写SQL代码，实际会执行如下命令:spark-sql -e <SQL>。

**示例:**

--- sql example

--- sql1

SELECT * from test1

WHERE id='xxxx';

--- sql2

SELECT count(1) from test1

WHERE id='xxxx';



## spark streaming

**说明:** Spark命令行任务，请填写spark-submit命令行参数, 实际会执行如下命令: spark-submit <PARAMS>.

**示例:** --class org.apache.spark.examples.SparkPi --master local[8] /path/to/examples.jar 100



## streaming sql

Spark Streaming SQL基于Spark Structured Streaming开发完成，所有语法功能和使用限制遵循Spark Structured Streaming。

**说明:** Streaming&#x00a0;SQL，请填写SQL代码，实际会执行如下命令:streaming-sql -e <SQL>。

**示例:** 
--- 创建SLS数据表

 CREATE TABLE IF NOT EXISTS ${slsTableName} USING loghub OPTIONS ( sls.project = '${logProjectName}', sls.store = '${logStoreName}', access.key.id = '${accessKeyId}', access.key.secret = '${accessKeySecret}', endpoint = '${endpoint}'); 

--- 将数据导入HDFS 

INSERT INTO ${hdfsTableName} SELECT col1, col2 FROM ${slsTableName} WHERE ${condition}



## mr

**说明:** MapReduce作业, 实际会执行如下命令: hadoop jar <PARAMS>.

**示例:** /usr/lib/hadoop-current/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar



## presto

**说明:** Presto SQL，请填写SQL代码，实际会执行如下命令: presto -f <SQL in file>

**示例:** 
--- sql example --- sql1 SELECT * from test1 WHERE id='xxxx'; --- sql2 SELECT count(1) from test1 WHERE id='xxxx';



## impala

**说明:** Impala SQL，请填写SQL代码，实际会执行如下命令: impala-shell -f <SQL in file>

**示例:** 
--- sql example --- sql1 SELECT * from test1 WHERE id='xxxx'; --- sql2 SELECT count(1) from test1 WHERE id='xxxx';