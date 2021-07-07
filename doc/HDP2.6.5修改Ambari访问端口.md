# HDP2.6.5修改Ambari访问端口

## 描述

由于ambari 默认的页面访问端口是8080，比较容易和其他应用端口冲突，所以要改成8081端口。



## 调整配置

在安装ambari server的服务器中，配置`/etc/ambari-server/conf/ambari.properties`文件。

修改如下：

```
client.api.port=8081
```

修改成功后，需要重启ambari-server服务。

## 效果

修改成功后，可以通过访问8081端口，访问ambari ui。

![image-20210621101532886](http://image-picgo.test.upcdn.net/img/20210621101533.png)