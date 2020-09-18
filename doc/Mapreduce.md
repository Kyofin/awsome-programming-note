# Mapreduce



## 远程debug

参考文档：

https://pravinchavan.wordpress.com/2013/04/05/remote-debugging-of-hadoop-job-with-eclipse/



1. 修改hadoop配置文件

   ![image-20200716010902099](http://image-picgo.test.upcdn.net/img/20200716010902.png)

   增加配置。

   ```
   export HADOOP_OPTS="-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=5432 $HADOOP_OPTS"
   
   ```

2. 重启yarn

   ```
   stop-yarn.sh
   start-yarn.sh
   ```

3. idea中配置remote debug

   ![image-20200716011046110](http://image-picgo.test.upcdn.net/img/20200716011046.png)

4. 运行mr程序

   ```
   hadoop jar /home/huzekang/tmp/bigdata-start-1.0-SNAPSHOT.jar com.data.hdfs.mr.outputformat.DbInputFormatDriver
   ```

5. 当看到命令行输出监听端口时，启动idea的debug即可。

   ![image-20200716011132019](http://image-picgo.test.upcdn.net/img/20200716011132.png)

