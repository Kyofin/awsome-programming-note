## 1. 启动minikube（需翻墙）

启动集群，初始化服务，这个过程会下载虚拟机镜像，启动virtualbox的虚拟机。

```
minikube start --docker-env HTTP_PROXY=http://192.168.5.19:1087 --docker-env HTTPS_PROXY=http://192.168.5.19:1087
```

- 加上代理解决翻墙问题HTTP_PROXY和HTTPS_PROXY写的都是ssr的http代理地址

- 另外第一次启动失败之后，要 minikube delete 之后再 start，否则不会重新拉镜像



## 2. 部署应用(需翻墙)

启动一个镜像，hello-minikube。网站上下载echoserver镜像1.10版，发布到的容器(Pod)端口8080。

```
kubectl run hello-minikube --image=k8s.gcr.io/echoserver:1.10 --port=8080
```

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190413231310.png)



部署完毕后，可以通过以下命令查看

```
kubectl get pods
```

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190413231322.png)

Pod容器组已经有一个hello-minikube-xxx的应用，xxx是默认加上的后缀。

```
kubectl get deployments
```

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190413231452.png)

部署也可以看到已经有一个hello-minikube应用了。

此时打开 dashboard可以看到三个hello-minikube相关的，如下：

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190413231641.png)



## 3. 发布服务

将之前部署的应用，发布为可以访问的服务。将hello-minikube应用发布成为服务，将之前容器(Pod)的8080端口，在节点（当前mac）中映射物理端口NodePort，

```
kubectl expose deployment hello-minikube --type=NodePort
```

发布后，通过命令查看已经发布的内容。

```
kubectl get services
```

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190413232528.png)

注意服务的集群ip是每人都不同的，这样服务就已经发布出来了，我们可以在minikube创建的虚拟机中通过的ip地址cluster-ip（**10.97.116.251**）访问服务。



## 4. 访问服务

有两种方式访问刚才发布的服务：

- 登录节点(minikube mac节点）访问
- 通过minikube service命令访问

### VM节点访问

 可以登录到virutalbox控制台（minikube虚拟机）上直接访问：

```
curl http://10.97.116.251:8080/
```

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190413232335.png)

### minikube service方式

 也可以通过minikube的命令查看数据

```
curl $(minikube service hello-minikube --url)
```

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190413232605.png)

实际上服务的访问没有那么简单，先临时这么用着，有个感官认识。



## 5. docker容器

整个过程中，部署应用，其实就是启动了docker应用。

```
docker ps
```

默认是没有东西的，执行下面命令

```
eval $(minikube docker-env)
```

再重新看docker ps，就可以看见很多容器启动了。

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190413232718.png)

第一个容器对象k8s_hello-minikube_hello-minikube-xxx就是在第3步部署的，而k8s_POD_hello-minikube-xxx是kubenertes pod里面默认的pause容器，后面有机会再展开说。

**kubernetes容器可以使用docker，并在此基础上封装了很多内容。**



## 6. 控制台dashboard

minikube 提供了dashboard可视化界面

```
minikube dashboard
```

默认会弹出浏览器 <http://192.168.99.100:30000/#!/overview?namespace=default>

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190413230652.png)

可以直观查看节点（node）、部署（deployment）、容器组（pod）、副本集（ReplicaSet）等。



## 7. 删除服务

```
kubectl delete services hello-minikube
```



## 8. 删除应用

```
kubectl delete deployment hello-minikube
```



## 9. 停止minikube

```
minikube stop
```

