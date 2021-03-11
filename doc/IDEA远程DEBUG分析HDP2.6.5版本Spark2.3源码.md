# IDEA远程DEBUG分析HDP2.6.5版本Spark2.3源码

## 下载HDP2.6.5对应Spark源码

https://codeload.github.com/hortonworks/spark2-release/zip/HDP-2.6.5.0-292-tag





## client端debug

```
 bin/spark-shell --master yarn --conf spark.hadoop.hadoop.caller.context.enabled=true --driver-java-options "-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=5010"
```

对应的idea中需要设置remote监听，因为client端是在启动spark-shell的服务器的，所以写它的ip即可。

![image-20210219165449739](http://image-picgo.test.upcdn.net/img/20210219165449.png)

这里断点提前打在添加自定义hadoop conf的地方，可以看到在命令行输入的hadoop conf在变量区出现。

![image-20210219165838669](http://image-picgo.test.upcdn.net/img/20210219165838.png)

在prepareContext方法中可以看到context是client的。

![image-20210219170208598](http://image-picgo.test.upcdn.net/img/20210219170208.png)



## AM端debug

```
 bin/spark-shell --master yarn --conf spark.hadoop.hadoop.caller.context.enabled=true \
--conf spark.yarn.am.extraJavaOptions="-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=5011"
```

在启动后，需要打开yarn的ui，看看启动的进程的am在哪台服务器。

![image-20210219170941005](http://image-picgo.test.upcdn.net/img/20210219170941.png)

可以看到是在hadoop-master服务器上。

![image-20210219171011860](http://image-picgo.test.upcdn.net/img/20210219171011.png)

所以在idea中设置对应的服务器ip即可。

![image-20210219171127165](http://image-picgo.test.upcdn.net/img/20210219171127.png)

在prepareContext方法中可以看到context是AppMaster的。

![image-20210219171217747](http://image-picgo.test.upcdn.net/img/20210219171217.png)

## Executor端debug

```
 bin/spark-shell --master yarn  --conf spark.hadoop.hadoop.caller.context.enabled=true \
--conf spark.executor.extraJavaOptions="-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=5012"
```

启动后，在spark-shell中输入count算子，并执行。

![image-20210219173142544](http://image-picgo.test.upcdn.net/img/20210219173142.png)

可以看到prepareContext中是task。

![image-20210219173212671](http://image-picgo.test.upcdn.net/img/20210219173212.png)