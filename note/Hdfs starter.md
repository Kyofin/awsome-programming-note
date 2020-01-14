# Hdfs starter

## 介绍

Hadoop本身的用户和组的关系，都是同步Linux系统中的，但是HDFS和Linux的超级用户组又有一点差别，HDFS中的超级用户组是supergroup，但是Linux中默认是没有supergoup这个组，这个时候只需要在Linux中增加supergroup这个组，然后将要在HDFS中加入到supergroup中的用户加到这个组中，再同步HDFS用户和组即可。

```
 groupadd supergroup
```



## linux下将用户添加到用户组中

```
usermod -a -G supergroup root
usermod -a -G hive root
```



## 将linux的用户和用户组同步到hdfs

```
hdfs dfsadmin -refreshUserToGroupsMappings
```



## 解除namenode的safemode

使用hdfs用户执行

```
hadoop dfsadmin -safemode leave
```



## 修改hdfs的权限

```
sudo bin/hadoop dfs -chmod -R 755 /
```



## 修改hdfs文件的所有者

```
sudo bin/hadoop fs -chown -R larry /
```

