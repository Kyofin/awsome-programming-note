# Flink starter



## 下载1.9版本

http://us.mirrors.quenda.co/apache/flink/flink-1.9.1/flink-1.9.1-bin-scala_2.11.tgz



## 参考文档

https://mp.weixin.qq.com/s/KfuAZv2G0682NNzHv0iFfQ【flink客户端操作的 5 种模式】



## flink shell使用方法

```
~/opt/flink-1.9.1 » bin/start-scala-shell.sh local
```

成功启动如下图：

![](https://i.loli.net/2019/11/20/mdFv5GAn3gQ8EkU.png)



## 运行flink集群的方式



### 1. 使用yarn模式运行flink集群



##### 启动yarn

```
~/opt/hadoop-cdh/hadoop-2.6.0-cdh5.14.2/logs » ../sbin/start-all.sh              
```

##### 启动flink集群（用yarn模式）

```
~/Downloads/flink-1.9.1 » bin/yarn-session.sh
```

![](https://i.loli.net/2019/11/19/F3ia5ZXrv8cbLlk.png)

这里分配了yarn资源的1g内存和1核开启一个flink 集群。上面的YARN session是在Hadoop YARN环境下启动一个Flink cluster集群，里面的资源是可以共享给其他的Flink作业

此时打开flink jobmanager web interface地址可以看到如下画面。

![](https://i.loli.net/2019/11/19/ymRBopgDqevCVsE.png)

查看resourcemanager也可以资源被分配出去了。

![](https://i.loli.net/2019/11/19/dClNqknAaSIvguO.png)



##### 跑一个example

上传文件到hdfs

```
~/opt/hadoop-cdh/hadoop-2.6.0-cdh5.14.2 » bin/hadoop fs -copyFromLocal LICENSE.txt hdfs:///tmp/
```



使用flink 集群计算wordcount

```
~/Downloads/flink-1.9.1 » ./bin/flink run ./examples/batch/WordCount.jar --input hdfs:///tmp/LICENSE.txt
```

![](https://i.loli.net/2019/11/19/ULWmFqvOilxHIMV.png)

可以看到作业正在执行。

一切顺利的话，可以看到在终端会显示出计算的结果：

```
(0,9)
(1,6)
(10,3)
(12,1)
(15,1)
(17,1)
(2,9)
(2004,1)
(2010,2)
(2011,2)
(2012,5)
(2013,4)
(2014,6)
(2015,7)
(2016,2)
(3,6)
(4,4)
(5,3)
(50,1)
(6,3)
(7,3)
(8,2)
(9,2)
(a,25)
(above,4)
(acceptance,1)
(accepting,3)
(act,1)
```



### 2. Start a local  flink cluster

##### 启动flink

```
~/opt/flink-1.9.1 » bin/start-cluster.sh                                                                               
Starting cluster.
Starting standalonesession daemon on host huzekangdembp.
Starting taskexecutor daemon on host huzekangdembp.
```

打开`http://localhost:8081/#/job/running`即可看到flink 集群。

![](http://image-picgo.test.upcdn.net/img/20191204222452.png)

##### 跑example

使用flink 集群计算wordcount

```
 flink run ~/opt/flink-1.9.1/examples/batch/WordCount.jar --input ~/opt/flink-1.9.1/README.txt
```

