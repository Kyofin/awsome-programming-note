# Ozone1.0.0伪分布式集群部署

## 下载解压

https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/ozone/ozone-1.0.0/hadoop-ozone-1.0.0.tar.gz



## docker-compose启动集群

进入 docker compose 文件所在的目录，执行 `docker-compose` 命令，你的机器会启动一个ozone 伪集群。

```
cd compose/ozone
export OZONE_REPLICATION_FACTOR=3
./run.sh
```

如果想后台启动，可以加上-d参数

```
./run.sh -d
```

启动成功后，可以在scm的页面看到3个datanode。

![image-20210805143137434](http://image-picgo.test.upcdn.net/img/20210805143137.png)



## 测试

使用`docker-compose`命令进入其中一个datanode容器中可以执行ozone的命令进行测试。

```
docker-compose exec datanode bash
```

执行下面命令，可以观察到该ozone集群默认就会创建一个volume。

```
ozone sh volume list
```

![image-20210805143520454](http://image-picgo.test.upcdn.net/img/20210805143520.png)

使用ozone内置测试工具产生大量文件。

```
ozone freon randomkeys --numOfVolumes=10 --numOfBuckets 10 --numOfKeys 1000  --replicationType=RATIS --factor=THREE
```

