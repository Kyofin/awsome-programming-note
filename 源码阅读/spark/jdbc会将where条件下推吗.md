# jdbc如何实现谓语下推和字段裁剪

测试代码：

```java
public class SparkJdbcApp {
  public static void main(String[] args) throws AnalysisException {
    SparkSession spark = SparkSession.builder().master("local").appName("Simple Application").getOrCreate();
    // 增加自定义listener
    spark.sparkContext().addSparkListener(new SparkListenerDemo(spark.sparkContext().conf()));
    final Properties properties = new Properties();
    properties.setProperty("user", "root");
    properties.setProperty("password", "eWJmP7yvpccHCtmVb61Gxl2XLzIrRgmT");
    spark.read().jdbc("jdbc:mysql://localhost:3306/yiboard", "chart", properties).createOrReplaceTempView("t1");
    spark.sql("select id from t1 where id = 3206").show();

    spark.stop();
  }
}
```



参考代码：org.apache.spark.sql.execution.datasources.jdbc.JDBCRDD

![image-20200610094825324](http://image-picgo.test.upcdn.net/img/20200610094825.png)

这里将filter条件和分区条件组成**where字句**，下推到数据库中执行。



![image-20200610095400882](http://image-picgo.test.upcdn.net/img/20200610095400.png)

这里就会提取用户查询的字段，然后拼接到select后面，下推到数据库执行。