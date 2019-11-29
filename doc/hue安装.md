# hue安装

由于我的mac无法编译源码安装，所以采用docker方式进行安装。

## 使用hue docker容器安装

1. 在当前目录创建好`hue.ini`文件

   简化版可以参考<https://github.com/cloudera/hue/blob/master/tools/docker/hue/conf/hue-overrides.ini>

   完全版可以参考<https://github.com/cloudera/hue/blob/master/desktop/conf.dist/hue.ini>

2. 在`hue.ini`文件中配置好数据库

   ![](https://raw.githubusercontent.com/huzekang/picbed/master/20190626222203.png)

3. 启动命令

   ```
   docker run -it -p 8889:8888 -v $PWD/hue.ini:/usr/share/hue/desktop/conf/z-hue.ini gethue/hue
   ```



## 配置需要hue连接的组件

### hdfs

```ini
[hadoop]

  # Configuration for HDFS NameNode
  # ------------------------------------------------------------------------
   [[hdfs_clusters]]
    # HA support by using HttpFs

    [[[default]]]
      # Enter the filesystem uri
       fs_defaultfs=hdfs://192.168.5.37:9000

      # Use WebHdfs/HttpFs as the communication mechanism.
      # Domain should be the NameNode or HttpFs host.
      # Default port is 14000 for HttpFs.
       webhdfs_url=http://192.168.5.37:50070/webhdfs/v1
```

然而配置完重启hue后，点击“File Browser”报错：

<u>Cannot access:/user/admin."注：您是hue管理员，但不是HDFS超级用户（即“”HDFS“”）</u>

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190626224752.png)

解决办法：

在hadoop的`core-site.xml`中配置代理用户。

 我一般配置是如下，在$HADOOP_HOME/etc/hadoop/下的core-site.xml里添加

```xml
<!-- 配置可以访问hdfs的用户 -->
    <property>
      <name>hadoop.proxyuser.hue.hosts</name>
      <value>*</value>
    </property>
    <property>
      <name>hadoop.proxyuser.hue.groups</name>
      <value>*</value>
    </property>
```

重启完hadoop就可以看到文件了。

但是如果在没有权限的目录下（如图当前文件夹属于huzekang用户）创建文件夹，则会出现如下错误：

<u>Permission denied: user=admin, access=WRITE, inode="/user":huzekang:supergroup:drwxr-xr-x</u>

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190626231038.png)

在hadoop的`core-site.xml`中配置代理用户

```xml
	<property>
      <name>hadoop.proxyuser.huzekang.hosts</name>
      <value>*</value>
    </property>
    <property>
      <name>hadoop.proxyuser.huzekang.groups</name>
      <value>*</value>
    </property>
```

并在hue中创建huzekang用户，并使用它登录hue则可以在上述目录中创建文件夹了。

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190626231658.png)



### hive

```
[beeswax]

  # Host where HiveServer2 is running.
  # If Kerberos security is enabled, use fully-qualified domain name (FQDN).
   hive_server_host=192.168.5.37

  # Port where HiveServer2 Thrift server runs on.
   hive_server_port=10000
```

配置好`hue.ini`文件后重启docker容器即可。

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190626232731.png)