# HDFS文件数、容量空间限额配置

## 介绍

 hdfs文件的限额配置，允许我们以文件大小或者文件个数来限制某个目录下上传的文件数量或者文件内容总量，以便达到我们类似百度网盘网盘等限制每个用户允许上传的最大的文件的量。



## 数量限额

给该文件夹下面设置最多上传两个文件

```
hdfs dfsadmin -setQuota 2 /user/huzekang
```

![image-20210809114811708](http://image-picgo.test.upcdn.net/img/20210809114811.png)

清除文件数量限制

```
hdfs dfsadmin -clrQuota /user/huzekang     
```



## 容量空间限额

限制空间大小4KB

```
hdfs dfsadmin -setSpaceQuota 4k  /user/huzekang 
```

上传超过4k的文件到hdfs。

```
hdfs dfs -put table2_data /user/huzekang
```

可以看到提示文件超出限额了。

![image-20210809113745120](http://image-picgo.test.upcdn.net/img/20210809113745.png)

清除限额后再重新上传。

```
hdfs dfsadmin -clrSpaceQuota /user/huzekang
hdfs dfs -put table2_data /user/huzekang
```

![image-20210809113915946](http://image-picgo.test.upcdn.net/img/20210809113915.png)

## 查看hfds文件限额

```
hdfs dfs -count -q -h /user/huzekang
```

![image-20210809114256531](http://image-picgo.test.upcdn.net/img/20210809114256.png)