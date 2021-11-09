# 查找hdfs存储文件在本地磁盘的路径

## 创建测试数据上传hdfs

```
echo "hello hadoop" > hello.txt
hdfs dfs -put hello.txt /tmp/test


```

## 查看hdfs的block存储信息

```
hdfs fsck /tmp/test/hello.txt  -files -blocks -locations -replicaDetails
```

![](http://image-picgo.test.upcdn.net/img/20210918173226.png)

可以看到红色框就是我们存储在本地文件名，划线的就是副本存储所在的服务器节点。

## 查找存储本地磁盘中的block

```
find /data/hadoop/ |grep blk_1075354439
```

![](http://image-picgo.test.upcdn.net/img/20210918173506.png)

可以看到和之前上传时的内容是一致的。

![](http://image-picgo.test.upcdn.net/img/20210918173641.png)