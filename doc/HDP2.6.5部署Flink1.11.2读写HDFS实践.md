# HDP2.6.5部署Flink1.11.2读写HDFS实践

## 编译flink-shade

1. 下载 [flink-shade](https://github.com/apache/flink-shaded/tree/master)源码到指定目录。

   ```
   git clone https://github.com/apache/flink-shaded.git  ~/flink-shade
   ```

2. 检出分支release-10.0

   ```
   git checkout release-10.0
   ```

3. 修改flink-shade源码中的pom文件。

   修改Hadoop版本为HDP的版本`2.7.3.2.6.5.0-292`

   ```
   vim  ~/flink-shaded/flink-shaded-hadoop-2-parent/pom.xml
   ```

   ![image-20210224151054697](http://image-picgo.test.upcdn.net/img/20210224151054.png)编译打包。

   ```
   cd ~/flink-shaded
   mvn clean package -Dshade-sources
   ```

## 添加缺失依赖到Flink的lib中

1. 拷贝flink-shade编译的flink-shaded-hadoop-2-uber-x.y.z.jar到Flink的lib目录下。

   ```
   cp  ~/flink-shaded/flink-shaded-hadoop-2-parent/flink-shaded-hadoop-2-uber/target/flink-shaded-hadoop-2-uber-2.7.2-11.0.jar /opt/flink-1.11.2/lib/
   ```

2. 下载[javax.ws.rs-api-2.0.jar](https://repo1.maven.org/maven2/javax/ws/rs/javax.ws.rs-api/2.0/javax.ws.rs-api-2.0.jar)放到Flink的lib目录下。

   ![image-20210311092052645](http://image-picgo.test.upcdn.net/img/20210311092052.png)

   如果不添加，使用yarn模式运行时，会报依赖缺失错误。

   ```
   java.lang.NoClassDefFoundError: javax/ws/rs/ext/MessageBodyReader
   ```



## 验证Apache Flink配置

1. 在集群中使用MR生成测试数据

   ```
   hadoop jar  /usr/hdp/2.6.5.0-292/hadoop-mapreduce/hadoop-mapreduce-examples.jar \
   randomtextwriter \
   -D mapreduce.randomtextwriter.totalbytes=10240 \
   -D mapreduce.randomtextwriter.bytespermap=1024 \
   -D mapreduce.job.maps=4  \
   -D mapreduce.job.reduces=2  \
   hdfs:///tmp/flink-test/input 
   ```

2. 检查MR生成的测试数据

   ![image-20210224152104563](http://image-picgo.test.upcdn.net/img/20210224152104.png)

3. 提交wordcount程序(local模式)

   ```
   /opt/flink-1.11.2/bin/flink run \
   -t local  \
   /opt/flink-1.11.2/examples/batch/WordCount.jar \
   --input hdfs://10.93.6.247:8020/tmp/flink-test/input  \
   --output hdfs://10.93.6.247:8020/tmp/flink-test/output 
   ```

   使用yarn-per模式。

   ```
   /opt/flink-1.11.2/bin/flink run \
   -t yarn-per-job \
   /opt/flink-1.11.2/examples/batch/WordCount.jar \
   --input hdfs://10.93.6.247:8020/tmp/flink-test/input  \
   --output hdfs://10.93.6.247:8020/tmp/flink-test/output 
   ```

   ![image-20210311092452717](http://image-picgo.test.upcdn.net/img/20210311092452.png)

4. 可以看到结果数据已经输出了

   ![image-20210224154006822](http://image-picgo.test.upcdn.net/img/20210224154006.png)

