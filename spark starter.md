# spark必知必会

## Spark submit 不同的模式运行

### 1. 运行在 yarn集群上

系统环境变量需要有yarn的配置。

![](https://i.loli.net/2019/11/20/bLxrwyP3DJ6QglY.png)

因为spark提交作业时会找yarn的配置去获取资源。

- Spark on YARN 集群上 yarn-cluster 模式运行

  ```SHELL
  ~/opt/spark-2.4.4-bin-hadoop2.6 » ./bin/spark-submit --class org.apache.spark.examples.SparkPi \                                                              huzekang@huzekangdeMacBook-Pro
      --master yarn \
      --deploy-mode cluster \
      --driver-memory 2g \
      --executor-memory 1g \
      --executor-cores 1 \
      --queue thequeue \
      examples/jars/spark-examples*.jar \
      10
  ```

  

- spark on yarn 集群上 yarn-client模式运行

  ```SHELL
  ~/opt/spark-2.4.4-bin-hadoop2.6 » ./bin/spark-submit --class org.apache.spark.examples.SparkPi \                                                              huzekang@huzekangdeMacBook-Pro
      --master yarn \
      --deploy-mode client \
      --driver-memory 2g \
      --executor-memory 1g \
      --executor-cores 1 \
      --queue thequeue \
      examples/jars/spark-examples*.jar \
      100
  ```



注意 Spark on YARN 支持两种运行模式，分别为`yarn-cluster`和`yarn-client`，具体的区别可以看[这篇博文](http://www.iteblog.com/archives/1223)，从广义上讲，yarn-cluster适用于生产环境；而yarn-client适用于交互和调试，也就是希望快速地看到application的输出。



## spark-shell连接phoenix

- 需要用到的jar到服务器上
  下载地址：https://pan.baidu.com/s/1bpGtcficrxJVd_RxDbfTsw
  ![](http://183.6.50.10:4999/server/../Public/Uploads/2019-11-20/5dd49b3d1a745.png)

```shell
scp spark-extra-lib/phoenix-*.jar root@agent1:/root/spark-extra-libs
```

- 启动spark shell

  在spark-shell 启动时指定加载的 lib，解决在 spark-shell中使用 phoenix 的问题

```shell
spark-shell --driver-library-path=/root/spark-extra-libs/* --driver-class-path=/root/spark-extra-libs/*
```

- 测试连接 phoenix

首先输入 `:paste`

```
val df = spark.read 
      .format("jdbc") 
      .option("driver", "org.apache.phoenix.jdbc.PhoenixDriver") 
      .option("url", "jdbc:phoenix:cdh01:2181")
      .option("dbtable", "gzhonghui_new.patient_basic_information") 
      .option("phoenix.schema.isNamespaceMappingEnabled", "true") 
      .load() 
```

Ctrl +D 执行

```
df.show(1,20,true)
```

![](https://i.loli.net/2019/11/20/GYyEVTj8I7b4gS6.png)