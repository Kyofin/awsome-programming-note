# 360QuickSql项目源码分析

## 原理 

- metastore

    通过metastore将外部数据源的连接配置和元信息维护在项目的sqllite中，后续查询时直接使用建立连接
   
- server

    根据sql去选择执行引擎查询，返回结果
   
- client

    提供jdbc给应用使用，再将sql给server运行
    
## 源码编译
```
 mvn   -DskipTests clean package -e        
```

编译完记得解压`target`中的**压缩包**。



## 测试使用idea进行元数据收集

元数据是通过sqllite文件存储的。
`/Quicksql/metastore/schema.db`

参考`com.qihoo.qsql.metadata.collector.JdbcCollectorTest`



## 测试使用idea执行sql

### 不指定执行引擎
1. 需要先元数据导入sqllite
2. 将sql进行base64编码（原因可以参考`quicksql.sh`）

   ![](http://image-picgo.test.upcdn.net/img/20200114155611.png)

3. idea启动类设置查询sql参数

   ![](http://image-picgo.test.upcdn.net/img/20200114155727.png)

4. 运行即可

   ![](http://image-picgo.test.upcdn.net/img/20200114155933.png)



### 指定spark执行引擎

#### 普通查询

执行上面的sql，不过这次用spark运行。

![](http://image-picgo.test.upcdn.net/img/20200114175607.png)

**运行前提：**

1. 本地要有hive，且hive要启动**metastore**服务，因为程序提交给spark的植入代码会用到spark-sql，其他不用
2. 本地要有spark，并**SPARK_HOME**在系统环境中，无需启动



idea设置启动参数。

这些参数是根据执行`./bin/quicksql.sh -e "SELECT 1" --runner spark`时，将脚本最后的command打印获得的。

参考：

```
--sql
"U0VMRUNUICBjb3VudCgxKSBmcm9tIGNoYXJ0"
--runner
spark
--master=
--worker_memory=
--driver_memory=
--worker_num=
--jar_name=/Users/huzekang/openSource/Quicksql/target/qsql-0.7.0/lib/qsql-core-0.7.0.jar
--class_name=com.qihoo.quicksql.sh.cli.QSqlSubmit
--jar=/Users/huzekang/openSource/Quicksql/target/qsql-0.7.0/lib/spark/commons-cli-1.3.1.jar,/Users/huzekang/openSource/Quicksql/target/qsql-0.7.0/lib/spark/elasticsearch-spark-20_2.11-6.2.4.jar,/Users/huzekang/openSource/Quicksql/target/qsql-0.7.0/lib/spark/ojdbc6-11.2.0.3.jar,/Users/huzekang/openSource/Quicksql/target/qsql-0.7.0/lib/spark/derby-10.10.2.0.jar,/Users/huzekang/openSource/Quicksql/target/qsql-0.7.0/lib/spark/imc-0.2.jar,/Users/huzekang/openSource/Quicksql/target/qsql-0.7.0/lib/spark/mysql-connector-java-5.1.20.jar
```

![](http://image-picgo.test.upcdn.net/img/20200114165018.png)

idea点击运行main方法即可。

程序会把我们的连接配置和sql变成spark的植入代码。

```java
import com.qihoo.qsql.exec.Requirement;
import org.apache.spark.sql.catalyst.expressions.Attribute;
import org.apache.spark.sql.Row;
import java.util.stream.Collectors;
import org.apache.spark.sql.SparkSession;
import java.util.Collections;
import java.util.Map;
import com.qihoo.qsql.exec.spark.SparkRequirement;
import scala.collection.JavaConversions;
import java.util.AbstractMap.SimpleEntry;
import com.qihoo.qsql.codegen.spark.SparkJdbcGenerator;
import org.apache.spark.sql.Dataset;
import java.util.Arrays;
import java.util.Properties;
import java.util.List;

public class Requirement42319 extends SparkRequirement { 
		public Requirement42319(SparkSession spark){
			super(spark);
		}

		public Object execute() throws Exception {
			Dataset<Row> tmp;
			{
			tmp = spark.read().jdbc("jdbc:mysql://192.168.1.150:3306/yiboard", "(select count(*) as expr_col__0 from yiboard.chart) yiboard_chart_0", SparkJdbcGenerator.config("root", "root", "com.mysql.jdbc.Driver"));
			tmp.createOrReplaceTempView("yiboard_chart_0");
			}
			String sql = "SELECT expr_col__0 FROM yiboard_chart_0";
			tmp = spark.sql(sql);
			tmp.show();

			return null;

		}
}

```

然后通过命令行工具`spark-submit`运行。

命令精简如下：

```shell
/Users/huzekang/opt/spark-2.4.4-bin-hadoop2.6/bin/spark-submit --jars /Users/huzekang/openSource/Quicksql/target/qsql-0.7.0/lib/spark/commons-cli-1.3.1.jar,/Users/huzekang/openSource/Quicksql/target/qsql-0.7.0/lib/spark/elasticsearch-spark-20_2.11-6.2.4.jar,/Users/huzekang/openSource/Quicksql/target/qsql-0.7.0/lib/spark/ojdbc6-11.2.0.3.jar,/Users/huzekang/openSource/Quicksql/target/qsql-0.7.0/lib/spark/derby-10.10.2.0.jar,/Users/huzekang/openSource/Quicksql/target/qsql-0.7.0/lib/spark/imc-0.2.jar,/Users/huzekang/openSource/Quicksql/target/qsql-0.7.0/lib/spark/mysql-connector-java-5.1.20.jar --master local[*] --executor-memory 1G --driver-memory 3G --num-executors 20 --conf spark.driver.userClassPathFirst=true --class com.qihoo.qsql.launcher.ProcessExecutor /Users/huzekang/openSource/Quicksql/target/qsql-0.7.0/lib/qsql-core-0.7.0.jar --jar /Users/huzekang/openSource/Quicksql/target/qsql-0.7.0/lib/spark/commons-cli-1.3.1.jar,/Users/huzekang/openSource/Quicksql/target/qsql-0.7.0/lib/spark/elasticsearch-spark-20_2.11-6.2.4.jar,/Users/huzekang/openSource/Quicksql/target/qsql-0.7.0/lib/spark/ojdbc6-11.2.0.3.jar,/Users/huzekang/openSource/Quicksql/target/qsql-0.7.0/lib/spark/derby-10.10.2.0.jar,/Users/huzekang/openSource/Quicksql/target/qsql-0.7.0/lib/spark/imc-0.2.jar,/Users/huzekang/openSource/Quicksql/target/qsql-0.7.0/lib/spark/mysql-connector-java-5.1.20.jar --master local[*] --runner spark --class_name Requirement41430 --source aW1wb3J0IGNvbS5xaWhvby5xc3FsLmV4ZWMuUmVxdWlyZW1lbnQ7CmltcG9ydCBvcmcuYXBhY2hlLnNwYXJrLnNxbC5jYXRhbHlzdC5leHByZXNzaW9ucy5BdHRyaWJ1dGU7CmltcG9ydCBvcmcuYXBhY2hlLnNwYXJrLnNxbC5Sb3c7CmltcG9ydCBqYXZhLnV0aWwuc3RyZWFtLkNvbGxlY3RvcnM7CmltcG9ydCBvcmcuYXBhY2hlLnNwYXJrLnNxbC5TcGFya1Nlc3Npb247CmltcG9ydCBqYXZhLnV0aWwuQ29sbGVjdGlvbnM7CmltcG9ydCBqYXZhLnV0aWwuTWFwOwppbXBvcnQgY29tLnFpaG9vLnFzcWwuZXhlYy5zcGFyay5TcGFya1JlcXVpcmVtZW50OwppbXBvcnQgc2NhbGEuY29sbGVjdGlvbi5KYXZhQ29udmVyc2lvbnM7CmltcG9ydCBqYXZhLnV0aWwuQWJzdHJhY3RNYXAuU2ltcGxlRW50cnk7CmltcG9ydCBjb20ucWlob28ucXNxbC5jb2RlZ2VuLnNwYXJrLlNwYXJrSmRiY0dlbmVyYXRvcjsKaW1wb3J0IG9yZy5hcGFjaGUuc3Bhcmsuc3FsLkRhdGFzZXQ7CmltcG9ydCBqYXZhLnV0aWwuQXJyYXlzOwppbXBvcnQgamF2YS51dGlsLlByb3BlcnRpZXM7CmltcG9ydCBqYXZhLnV0aWwuTGlzdDsKCnB1YmxpYyBjbGFzcyBSZXF1aXJlbWVudDQxNDMwIGV4dGVuZHMgU3BhcmtSZXF1aXJlbWVudCB7IAoJCXB1YmxpYyBSZXF1aXJlbWVudDQxNDMwKFNwYXJrU2Vzc2lvbiBzcGFyayl7CgkJCXN1cGVyKHNwYXJrKTsKCQl9CgoJCXB1YmxpYyBPYmplY3QgZXhlY3V0ZSgpIHRocm93cyBFeGNlcHRpb24gewoJCQlEYXRhc2V0PFJvdz4gdG1wOwoJCQl7CgkJCXRtcCA9IHNwYXJrLnJlYWQoKS5qZGJjKCJqZGJjOm15c3FsOi8vMTkyLjE2OC4xLjE1MDozMzA2L3lpYm9hcmQiLCAiKHNlbGVjdCBjb3VudCgqKSBhcyBleHByX2NvbF9fMCBmcm9tIHlpYm9hcmQuY2hhcnQpIHlpYm9hcmRfY2hhcnRfMCIsIFNwYXJrSmRiY0dlbmVyYXRvci5jb25maWcoInJvb3QiLCAicm9vdCIsICJjb20ubXlzcWwuamRiYy5Ecml2ZXIiKSk7CgkJCXRtcC5jcmVhdGVPclJlcGxhY2VUZW1wVmlldygieWlib2FyZF9jaGFydF8wIik7CgkJCX0KCQkJU3RyaW5nIHNxbCA9ICJTRUxFQ1QgZXhwcl9jb2xfXzAgRlJPTSB5aWJvYXJkX2NoYXJ0XzAiOwoJCQl0bXAgPSBzcGFyay5zcWwoc3FsKTsKCQkJdG1wLnNob3coKTsKCgkJCXJldHVybiBudWxsOwoKCQl9Cn0K
```

可以看到默认是使用local模式。 

spark作业的启动类是`com.qihoo.qsql.launcher.ProcessExecutor`



#### 数据落地sql

这里将聚合后的数据写入远程hdfs中。

```
INSERT INTO `hdfs://cdh04:8020/hello/world` IN HDFS SELECT approx_count_distinct(city), state FROM account GROUP BY state LIMIT 10
```

![](http://image-picgo.test.upcdn.net/img/20200114175418.png)

将编码后的sql放入idea的启动参数中替换就可以了。

可以debug获取提交作业的class。

```java
import com.qihoo.qsql.exec.Requirement;
import org.apache.spark.sql.catalyst.expressions.Attribute;
import org.apache.spark.sql.Row;
import java.util.stream.Collectors;
import org.apache.spark.sql.SparkSession;
import java.util.Collections;
import java.util.Map;
import java.util.HashMap;
import com.qihoo.qsql.exec.spark.SparkRequirement;
import java.util.regex.Pattern;
import scala.collection.JavaConversions;
import java.util.AbstractMap.SimpleEntry;
import java.util.regex.Matcher;
import com.qihoo.qsql.codegen.spark.SparkElasticsearchGenerator;
import org.elasticsearch.spark.sql.api.java.JavaEsSparkSQL;
import org.apache.spark.sql.Dataset;
import java.util.Arrays;
import org.apache.commons.lang.StringEscapeUtils;
import java.util.List;

public class Requirement40757 extends SparkRequirement { 
		public Requirement40757(SparkSession spark){
			super(spark);
		}

		public Object execute() throws Exception {
			Dataset<Row> tmp;
			{
			Map<String, String> config = SparkElasticsearchGenerator.config("192.168.1.25", "9200", "", "", "bank/account", "{\"_source\":[\"state\",\"city\"],\"query\":{\"match_all\":{}}}", "1");
			tmp = JavaEsSparkSQL.esDF(spark, config);
			tmp.createOrReplaceTempView("bank_account_0");
			}
			String sql = "SELECT COUNT(DISTINCT city) AS expr_col__0, state FROM bank_account_0 GROUP BY state LIMIT 10";
			tmp = spark.sql(sql);
			tmp.write().format("com.databricks.spark.csv").save("hdfs://cdh04:8020/hello/world");

			return null;

		}
}

```

运行完成后，可以打开hdfs看到保存落地的数据。

![](http://image-picgo.test.upcdn.net/img/20200114175732.png)