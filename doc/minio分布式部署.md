# minio分布式部署

## 下载minio

```
wget http://dl.minio.org.cn/server/minio/release/linux-amd64/minio
```

赋予执行权限

```
chmod +x minio
```



## 网络拓扑

172.30.74.0  

172.30.74.1

172.30.74.2

## 节点1创建目录

```
mkdir data1 data2 data3 data3
```



## 节点2创建目录

```
mkdir data1 data2 data3 data3
```



## 节点3创建目录

```
mkdir data1 data2 data3 data3
```



## 节点1启动minio

```
./minio server  http://172.30.74.2/root/data1 http://172.30.74.2/root/data2 http://172.30.74.2/root/data3 http://172.30.74.2/root/data4 \
             http://172.30.74.1/root/data1 http://172.30.74.1/root/data2 http://172.30.74.1/root/data3 http://172.30.74.1/root/data4  \
             http://172.30.74.0/root/data1 http://172.30.74.0/root/data2 http://172.30.74.0/root/data3 http://172.30.74.0/root/data4  
```



## 节点2启动minio

```
./minio server  http://172.30.74.2/root/data1 http://172.30.74.2/root/data2 http://172.30.74.2/root/data3 http://172.30.74.2/root/data4 \
             http://172.30.74.1/root/data1 http://172.30.74.1/root/data2 http://172.30.74.1/root/data3 http://172.30.74.1/root/data4  \
             http://172.30.74.0/root/data1 http://172.30.74.0/root/data2 http://172.30.74.0/root/data3 http://172.30.74.0/root/data4  
```



## 节点3启动minio

```
./minio server  http://172.30.74.2/root/data1 http://172.30.74.2/root/data2 http://172.30.74.2/root/data3 http://172.30.74.2/root/data4 \
             http://172.30.74.1/root/data1 http://172.30.74.1/root/data2 http://172.30.74.1/root/data3 http://172.30.74.1/root/data4  \
             http://172.30.74.0/root/data1 http://172.30.74.0/root/data2 http://172.30.74.0/root/data3 http://172.30.74.0/root/data4  
```

启动成功后可以看到如下：

![image-20210819172700755](http://image-picgo.test.upcdn.net/img/20210819172700.png)

其中9000端口为s3访问的，36573（可能会变）是web控制台的访问。





## 本地awscli访问远程服务器minio

本地配置aws的key和secret。

```
~/Downloads/minio-distributed » aws configure                                                                                                                

AWS Access Key ID [****************park]: minioadmin
AWS Secret Access Key [****************k123]: minioadmin
Default region name [us-east-1]:
Default output format [json]:
```

列出所有的bucket

```
aws --endpoint-url http://8.130.167.145:9000   s3 ls
```

创建bucket

```
aws --endpoint-url http://8.130.167.145:9000 s3 mb s3://mybucket
```

创建完成后，可以在web控制台看到

![image-20210819173035564](http://image-picgo.test.upcdn.net/img/20210819173035.png)

上传文件到bucket

```
aws --endpoint-url  http://8.130.167.145:9000  s3 cp docker-compose.yaml s3://mybucket
```

上传完成后，可以在web控制台上看到该文件

![image-20210819173230500](http://image-picgo.test.upcdn.net/img/20210819173230.png)

列出bucket中的文件列表

```
aws --endpoint-url http://8.130.167.145:9000 s3 ls s3://mybucket
```



