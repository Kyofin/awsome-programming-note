# Livy spark集群交互服务

## 官方文档

https://livy.incubator.apache.org/docs/latest/rest-api.html



## 下载

apache-livy-0.6.0-incubating-bin



## 配置系统环境变量

```
export SPARK_HOME=/opt/spark-2.4.4-bin-hadoop2.6
export HADOOP_CONF_DIR=/etc/hadoop/conf
```



## 配置`conf/livy-env.sh`

```
SPARK_HOME=/opt/spark-2.4.4-bin-hadoop2.6
HADOOP_CONF_DIR=/etc/hadoop/conf
```



## 配置`conf/livy.conf`

指定spark  master地址

````
 livy.spark.master = spark://cdh01:7077
````



## 相关命令

启动命令

```
bin/livy-server
```

后台启动命令

```
bin/livy-server start
```

后续打开web ui可以看到启动成功

http://cdh01:8998/ui

![](https://i.loli.net/2019/10/09/qBF5EH8T7GbrKQ4.png)



## postman测试api

https://www.getpostman.com/collections/ee87d500b01a06343d03



## 交互编程示例

1. 创建session

   获得返回来的id

   ![](https://i.loli.net/2019/10/09/DsBJOFmd9u2LElo.png)

   

   打开spark ui可以看到启动spark 应用

   ![](https://i.loli.net/2019/10/09/DnRY9iBVO8Ga3h7.png)

2. 编写交互代码

   ![](https://i.loli.net/2019/10/09/HbFzGuVdfDwJeiE.png)

3. 查看交互代码计算完的结果

![](https://i.loli.net/2019/10/09/GrYL86IxHhyTbjU.png)

## 批处理jar包示例

spark作业的jar包要先上传到hdfs上。



![](https://i.loli.net/2019/10/09/HvocU4OID5dFkZ3.png)

打开spark ui可以看到批处理作业正在执行。

![](https://i.loli.net/2019/10/09/cGHO8W1NSKRAZeX.png)