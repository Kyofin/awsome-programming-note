# Spark性能调优工具Sparklens实践

## 介绍

Sparklens 是 Spark 的分析工具，带有内置的 Spark 调度程序模拟器。它的主要目标是让人们更容易理解 Spark 应用程序的可扩展性限制。它有助于了解给定的 Spark 应用程序使用提供给它的计算资源的效率如何。也许你的应用程序会随着更多的执行程序运行得更快，但也可能不会。Sparklens 可以通过查看应用程序的单次运行来回答这个问题。

它可以帮助您缩小到限制您的应用程序向外扩展的几个阶段（或驱动程序、倾斜或缺少任务），并提供有关这些阶段可能出现问题的上下文信息。主要是它可以帮助您将 Spark 应用程序调优视为一种明确定义的方法/过程，而不是您通过反复试验学习的东西，从而节省开发人员和计算时间。

## 下载源码编译

下载源码：https://github.com/huzk8/Sparklens

执行编译。

```
 mvn clean package -DskipTests    
```

编译成功后可以看到编译好的jar包。

![image-20210707173528440](http://image-picgo.test.upcdn.net/img/20210707173528.png)

## 使用实践

这里以一个sql作为spark任务以client模式提交到yarn中运行。

在提交spark任务时加入以下配置。

```
--jars /path/to/sparklens_2.11-0.3.2.jar 
--conf spark.extraListeners=com.qubole.sparklens.QuboleJobListener
```

并指导启动executor和driver的配置如下：

```
"numExecutors":2,"driverMemory":"2G","executorMemory":"1G","driverCores":1
```

在任务执行完后，sparklens会将监测到的指标打印在控制台中。

![image-20210707174432602](http://image-picgo.test.upcdn.net/img/20210707174432.png)

可以看到启动了两个executor。

![image-20210707175608009](http://image-picgo.test.upcdn.net/img/20210707175608.png)

可以看到总的耗时为4分49s。

sparklens会推测不同executor数量的执行耗时和集群资源利用率。如下图：

![image-20210707175908226](http://image-picgo.test.upcdn.net/img/20210707175908.png)

可以看到sparklens推测：当executor设置数量为4时，耗时2分多钟就可以完成作业。

重新跑该sql，以如下配置提交，只调大了executor的数量为4，其余不变。

```
"numExecutors":4,"driverMemory":"2G","executorMemory":"1G","driverCores":1
```

![image-20210707180123242](http://image-picgo.test.upcdn.net/img/20210707180123.png)

可以看到执行的时间只需要2分多钟，和之前的推测是一致的。

## 总结

根据sparklens给我们分析的指标，可以方便的调整spark的调优参数。