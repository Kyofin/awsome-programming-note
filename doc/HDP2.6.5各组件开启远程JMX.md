# HDP2.6.5各组件开启远程JMX

## kafka

在ambari中设置kafka的env。

```
export JMX_PORT=9988
```

![image-20210511164403582](http://image-picgo.test.upcdn.net/img/20210511164403.png)

在Mac上使用Jconsole命令，远程连接kafka jmx。

![image-20210511164643434](http://image-picgo.test.upcdn.net/img/20210511164643.png)



## kylin

打开bin/kylin.sh文件，添加如下三行。

```
-Dcom.sun.management.jmxremote.authenticate=false \
-Dcom.sun.management.jmxremote.port=10000 \
-Dcom.sun.management.jmxremote.ssl=false \
```

![image-20210511180311333](http://image-picgo.test.upcdn.net/img/20210511180311.png)

打开conf/kylin.properties

```
kylin.server.query-metrics-enabled=true
```

默认值为 FALSE，设为 TRUE 来将查询指标收集到 JMX。

重启kylin后，可以使用Jconsole查看。

![image-20210511180538268](http://image-picgo.test.upcdn.net/img/20210511180538.png)