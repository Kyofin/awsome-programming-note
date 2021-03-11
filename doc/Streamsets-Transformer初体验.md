# Streamsets-Transformer初体验

## 下载streamset-transformer部署包

https://archives.streamsets.com/transformer/3.17.0/tarball/aster/streamsets-transformer-all_2.11-3.17.0.tgz

## 下载spark部署包

http://archive.apache.org/dist/spark/spark-2.4.7/spark-2.4.7-bin-hadoop2.6.tgz

注意scala版本是2.11



## 配置`libexec/transformer-env.sh`

![image-20210209092458818](http://image-picgo.test.upcdn.net/img/20210209092458.png)

```
export SPARK_HOME=/Volumes/Samsung_T5/opt/spark-2.4.7-bin-hadoop2.6

```



## 启动transformer

```
/Volumes/Samsung_T5/opt/streamsets/streamsets-transformer_2.11-3.17.0 » bin/streamsets transformer
```

控制台会显示浏览器访问的URL。



## 运行sample例子

![image-20210209092310654](http://image-picgo.test.upcdn.net/img/20210209092310.png)

配置local模式

![image-20210209092403477](http://image-picgo.test.upcdn.net/img/20210209092403.png)

