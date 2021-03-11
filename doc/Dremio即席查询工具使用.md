# Dremio即席查询工具使用

## 下载linux版本

https://download.dremio.com/community-server/13.1.0-202102110202430875-3e6f3e7c/dremio-community-13.1.0-202102110202430875-3e6f3e7c.tar.gz



## 上传服务器解压

![image-20210226155140032](http://image-picgo.test.upcdn.net/img/20210226155140.png)



## 修改配置

dremio启动时默认会启动内嵌的zk。由于hdp集群内部已经有zk，所以dremio启动时会报错，提示端口占用。这里将dremio的zk配置用集群的zk地址，并且关闭内嵌的zk。

```
[root@hadoop-slave1 dremio-community-13.1.0-202102110202430875-3e6f3e7c]# vim conf/dremio.conf 
```

加入以下内容。

```
zookeeper: "10.93.6.167:2181"
services.coordinator.master.embedded-zookeeper.enabled: false
```

![image-20210226155220691](http://image-picgo.test.upcdn.net/img/20210226155220.png)





## 启动dremio

```
[root@hadoop-slave1 dremio-community-13.1.0-202102110202430875-3e6f3e7c]# bin/dremio start 
```

访问地址：http://10.93.6.167:9047 即可。

![image-20210226155447723](http://image-picgo.test.upcdn.net/img/20210226155447.png)