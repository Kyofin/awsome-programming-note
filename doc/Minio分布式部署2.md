# Minio分布式部署2

## 必要知道的知识





纠删码 Erasure Code
可以简单的理解为一种数据保护机制，当过半节点数损毁的情况下仍然可以恢复数据。minio就是使用了纠删码的技术来保证数据的安全。为此其驱动器的数量必须是4或16的倍数。如上我们已经准备好了3台机器，那么要在3台机器上部署4个minio的驱动器，就需要其中一台机器上要部署两个。

## 安装的服务器清单

这里选了三台服务器。

```
10.81.16.168
10.81.16.172
10.81.16.199
```







## 部署安装

下载minio

```
wget https://dl.min.io/server/minio/release/linux-amd64/minio
```

创建安装目录。

```
mkdir -p /opt/minio_home
```

下载好的minio放到安装目录里。

在`10.81.16.168`和`10.81.16.199`创建数据目录。

```
mkdir -p /data/minio/data
```

在`10.81.16.172`创建数据目录。

```
mkdir -p /data/minio/data1
mkdir -p /data/minio/data2
```

在mino安装目录内编写启动脚本`startup.sh`。

```
#!/bin/bash

export MINIO_ACCESS_KEY=minio
export MINIO_SECRET_KEY=minio123
nohup /opt/minio_home/minio server --address :9000 --console-address :9001 http://10.81.16.168/data/minio/data http://10.81.16.199/data/minio/data  \
http://10.81.16.172/data/minio/data1 \
http://10.81.16.172/data/minio/data2   > /opt/minio_home/minio.log 2>&1 &

```

该脚本内设置了minio的访问账号和密码，并且指定了各个服务器的ip和驱动数据目录地址。可以看到，10.81.16.168和10.81.16.199都各自有一个驱动数据目录，10.81.16.172有两个个驱动数据目录。