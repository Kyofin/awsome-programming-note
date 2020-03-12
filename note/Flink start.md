# Flink starter



## 下载1.9版本

http://us.mirrors.quenda.co/apache/flink/flink-1.9.1/flink-1.9.1-bin-scala_2.11.tgz



## 参考文档

**[ververica flink forward](https://ververica.cn/developers-resources/)**

**[Flink on Yarn原理](https://www.infoq.cn/article/UFeFrdqSlqI3HyRHbPNm)**

##### [flink客户端操作的 5 种模式](https://mp.weixin.qq.com/s/KfuAZv2G0682NNzHv0iFfQ)

###### [Apache Flink 零基础入门](https://segmentfault.com/a/1190000020300020)

##### [Apache Flink 1.10.0 最新发布，年度最大规模版本升级！](https://blog.csdn.net/qq_18769269/article/details/104290984)

##### [5 个 TableEnvironment 我该用哪个？](https://mp.weixin.qq.com/s?__biz=MzU3Mzg4OTMyNQ==&mid=2247484827&idx=1&sn=0b9da36050532b8adb4e6ad11a0f4a23&chksm=fd3b8bd9ca4c02cf310a84c464e7a882742fbcaa1ed62520c2d045eb2e0bcd28278e85433c38&scene=0&xtrack=1&clicktime=1569728441&enterid=1569728441&ascene=7&)



## flink shell使用方法

```
~/opt/flink-1.9.1 » bin/start-scala-shell.sh local
```

成功启动如下图：

![](https://i.loli.net/2019/11/20/mdFv5GAn3gQ8EkU.png)



## 提交flink作业

#### ON YARN

在本地提交作业到远程yarn，不用在本地启动flink的任何服务。

```
 ./bin/flink run -m yarn-cluster -yn 1 -yjm 1024 -ytm 1024 ./examples/batch/WordCount.jar
```

通过环境变量的指定可以在本地以client模式提交到远程的yarn集群中执行。**注意的是提交到yarn需要hadoop的一些jar包，所以本地环境要装hadoop，并在环境变量中指定。**

![](http://image-picgo.test.upcdn.net/img/20200121093101.png)

由于提交到yarn，输出返回都直接打印在控制台，虽然执行过程中可以在yarn上跳转到该作业的flink web查看每个步骤的耗时等参数，但是作业结束后就不能查看了，因为作业结束flink web也会随之关闭。

![](http://image-picgo.test.upcdn.net/img/20200121093930.png)

这时候就需要在**本地启动history sever来记录作业**了。

在flink yml中配置如下：

```yml
jobmanager.archive.fs.dir: hdfs://cdh04:8020/flink/v1.0copy/completed-jobs/
 
# The address under which the web-based HistoryServer listens.
historyserver.web.address: 0.0.0.0
 
# The port under which the web-based HistoryServer listens.
historyserver.web.port: 8088
 
# Comma separated list of directories to monitor for completed jobs.
historyserver.archive.fs.dir: hdfs://cdh04:8020/flink/v1.0copy/completed-jobs/,hdfs://cdh04:8020/flink/v1.0copy/h-completed-jobs/,hdfs://cdh04:8020/flink/completed-jobs/
 
# Interval in milliseconds for refreshing the monitored directories.
historyserver.archive.fs.refresh-interval: 1000

```

![](http://image-picgo.test.upcdn.net/img/20200121094159.png)

然后单独启动flink history sever即可，其他服务不用在本地启动。

```
~/opt/flink-1.9.1 » bin/historyserver.sh start                                                                
```

这时候跑的作业就会记录下来了。

![](http://image-picgo.test.upcdn.net/img/20200121094414.png)

实际的记录则根据配置保存在远程的hdfs上。

![](http://image-picgo.test.upcdn.net/img/20200121094550.png)



## 运行long time flink集群的方式



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

