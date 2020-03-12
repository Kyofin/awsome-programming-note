# 使用portainer管理docker主机

>参考文档：
>https://www.portainer.io/installation/
>http://www.senra.me/docker-management-panel-series-portainer/
## 管理多台单机
### 1. Deploying Portainer 
```
$ docker volume create portainer_data

$ docker run -d -p 9000:9000 --name portainer --restart always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer
```

##### ![](https://raw.githubusercontent.com/huzekang/picbed/master/20190608201557.png)
## 
### 2. 加入远程主机到portainer
#### 方法一

1. 远程主机docker开启对外监听

以134.175.29.194这台主机为例。
首先需要开启docker的2375端口
```
vi /etc/sysconfig/docker  
```
添加上
centos6下使用
```
 other_args='-Htcp://0.0.0.0:2375 -H unix:///var/run/docker.sock'
```
centos7下使用 
```
OPTIONS='-Htcp://0.0.0.0:2375 -H unix:///var/run/docker.sock'
```

1. 把主机添加到portainer

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190608201738.png)

#### 方法二（适用新版本，我就是用这个的）

修改文件

```
vim /lib/systemd/system/docker.service
```

修改这行内容
```
ExecStart=/usr/bin/docker daemon -H fd:// -H tcp://0.0.0.0:2375
```
重点是tcp://0.0.0.0:2375

然后
```
sudo systemctl daemon-reload
sudo systemctl restart docker.service
```

## 管理swarm集群

主节点建立swarm集群，然后让各个节点都需要加入swarm集群。
### 1. 创建swarm机群
* manager上执行：
```
docker swarm init --advertise-addr 10.45.41.211
```
* 查看状态：
```
docker info
docker node ls
```
* 如果忘记了token，可执行如下查看：
```
docker swarm join-token worker
```
* 然后添加woker
```
docker swarm join \
--token SWMTKN-1-2ch5ug4apnq2gge6j8dq9o9mxu392afcj2gcsvnc2jc913qmoz-35tv88g9bhptxejxc0pg5synk \
10.45.41.211:2377
```
* 再次查看状态：
```
docker node ls  
```


### 2. 使用portainer管理swarm集群
主节点需要安装好docker-compose
```
$  curl -L https://github.com/docker/compose/releases/download/1.24.0-rc1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

$  chmod +x /usr/local/bin/docker-compose
```

在主节点启动如下命令：
```
$ curl -L https://downloads.portainer.io/portainer-agent-stack.yml -o portainer-agent-stack.yml

$ docker stack deploy --compose-file=portainer-agent-stack.yml portainer
```


