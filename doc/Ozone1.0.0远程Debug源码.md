# Ozone1.0.0远程Debug源码

## 远程Debug断点Ozone s3 gateway

命令行输入下面命令

```
export HADOOP_OPTS="-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=5005"
```

然后再启动s3g组件。

```
bin/ozone --daemon start s3g
```

启动过程中，idea要启动远程断点程序。设置如下：

![image-20210804182125369](http://image-picgo.test.upcdn.net/img/20210804182125.png)

浏览器访问s3g的地址，获取某个key的信息。

![image-20210804182208515](http://image-picgo.test.upcdn.net/img/20210804182208.png)

可以看到代码断点成功。

![image-20210804182232183](http://image-picgo.test.upcdn.net/img/20210804182232.png)