# 组件仓库

http://archive.cloudera.com/cdh5/cdh/5/

组件下载：上面地址加组件加.tar.gz

如下载hive:http://archive.cloudera.com/cdh5/cdh/5/hive-1.1.0-cdh5.16.2.tar.gz

源码下载：上面地址加组件加-src.tar.gz

如下载hive源码：http://archive.cloudera.com/cdh5/cdh/5/hive-1.1.0-cdh5.16.2-src.tar.gz

# hadoop部署

## java部署 

rpm 安装。

## 校验java环境

校验java是否安装成功

```
[hzk@bigdata001 ~]$  which java
/usr/java/jdk1.8.0_121/bin/java
```

JAVA部署必须是 /usr/java/   CDH环境配套



## 个人环境变量文件

[hzk@bigdata001 ~]$ vi .bashrc

```
export HADOOP_HOME=/home/hzk/app/hadoop
export HIVE_HOME=/home/hzk/app/hive
export PATH=${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin:${HIVE_HOME}/bin:$PATH
export JAVA_HOME=/usr/java/default

```

环境生效，校验环境

```
[hzk@bigdata001 ~]$ source .bashrc
[hzk@bigdata001 ~]$ which hdfs
~/app/hadoop/bin/hdfs
[hzk@bigdata001 ~]$ which hive
~/app/hive/bin/hive
[hzk@bigdata001 ~]$ 
```



##  信任关系

```
[hzk@bigdata001 ~]$ ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
Generating public/private rsa key pair.
Created directory '/home/hzk/.ssh'.
Your identification has been saved in /home/hzk/.ssh/id_rsa.
Your public key has been saved in /home/hzk/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:ZTBfyZWJtzdLnqubfvooTwu3QP/OQnw+L5tBDqEOeqM hzk@bigdata001
The key's randomart image is:
+---[RSA 2048]----+
|        o  ..+.o |
|         + .+ +  |
|          + .. . |
|         o . ..o.|
|        S ..o +.+|
|       . o. .* = |
|      . o .o.+= .|
|       o . .=.OB |
|      E     +@@B+|
+----[SHA256]-----+
[hzk@bigdata001 ~]$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
[hzk@bigdata001 ~]$ chmod 0600 ~/.ssh/authorized_keys

[hzk@bigdata001 ~]$ cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.0.3 bigdata001

[hzk@bigdata001 ~]$ ssh bigdata001 date
The authenticity of host 'bigdata001 (192.168.0.3)' can't be established.
ECDSA key fingerprint is SHA256:OLqoaMxlGFbCq4sC9pYgF+FdbcXHbEbtSrnMiGGFbVw.
ECDSA key fingerprint is MD5:d3:5b:4a:ef:8e:00:41:a0:5e:80:ef:75:76:8a:a3:49.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'bigdata001,192.168.0.3' (ECDSA) to the list of known hosts.
Thu Oct 31 21:31:32 CST 2019
[hzk@bigdata001 ~]$ ssh bigdata001 date
Thu Oct 31 21:31:41 CST 2019
[hzk@bigdata001 ~]$ 


```



## 配置hadoop下的env文件

一定要显示配置路径。不要用`${JAVA_HOME}`

理由看注释。

![image-20200724131436556](http://image-picgo.test.upcdn.net/img/20200724131436.png)

hadoop-env.sh

```
# export JAVA_HOME=${JAVA_HOME}
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_202.jdk/Contents/Home   
```

如果不加，后续可能后有java home找不到的错误。



## 配置文件及 NN SNN DN RM NM都以bigdata001启动

[hzk@bigdata001 hadoop]$ vi core-site.xml

```
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://bigdata001:8020</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/home/hzk/tmp</value>
    </property>
    
</configuration>
```



[hzk@bigdata001 hadoop]$ vi hdfs-site.xml

```
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>

    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>bigdata001:9868</value>
    </property>
    
    <property>
        <name>dfs.namenode.secondary.https-address</name>
        <value>bigdata001:9869</value>
    </property>

</configuration>
```



[hzk@bigdata001 hadoop]$ vi slaves 

```
bigdata001
```





[hzk@bigdata001 hadoop]$ vi mapred-site.xml

```
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
 <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>

</configuration>
```



[hzk@bigdata001 hadoop]$ vi yarn-site.xml

```
<?xml version="1.0"?>
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.resourcemanager.webapp.address</name>
        <value>bigdata001:18088</value>
    </property>
</configuration>
```



## 关闭防火墙

```
service firewalld stop
```





## 格式化 及启动

[hzk@bigdata001 hadoop]$ hdfs namenode -format

[hzk@bigdata001 hadoop]$ start-dfs.sh

[hzk@bigdata001 hadoop]$ start-yarn.sh

[hzk@bigdata001 hadoop]$ jps

3.6 open
http://bigdata001:50070
http://bigdata001:18088







# hive部署启动



## hive配置

https://docs.cloudera.com/documentation/enterprise/5-16-x/topics/cdh_ig_hive_metastore_configure.html

[hzk@bigdata001 conf]$ vi hive-site.xml 

```
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
<property>
  <name>javax.jdo.option.ConnectionURL</name>
  <value>jdbc:postgresql://192.168.1.237:5432/demo_hive_spark</value>
  <description>the URL of the MySQL database</description>
</property>

<property>
  <name>javax.jdo.option.ConnectionDriverName</name>
  <value>org.postgresql.Driver</value>
</property>

<property>
  <name>javax.jdo.option.ConnectionUserName</name>
  <value>postgres</value>
</property>

<property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>izA8v8gFIXqsXXwa4BZWahrW6dtQ21T8</value>
    </property>
</configuration>
```

将pg依赖放入hive目录的lib文件夹中。



##  启动metastore + hiveserver2服务

```
nohup hive --service  metastore > ~/log/metastore.log 2>&1 &
nohup  hiveserver2  > ~/log/hiveserver2.log 2>&1 &
```

hiveserver 开放端口10000 ----》 jdbc连接

hivemetastore 开放端口9083  ——》 thrift协议

## 测试hiveserver2服务是否ok

[hzk@bigdata001 ~]$ beeline
which: no hbase in (/home/hzk/app/hadoop/bin:/home/hzk/app/hadoop/sbin:/home/hzk/app/hive/bin:/usr/java/jdk1.8.0_121/bin:/usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/hzk/.local/bin:/home/hzk/bin)
Beeline version 1.1.0-cdh5.16.2 by Apache Hive
beeline> !connect jdbc:hive2://bigdata001:10000/default


scan complete in 1ms
Connecting to jdbc:hive2://bigdata001:10000/default
Enter username for jdbc:hive2://bigdata001:10000/default: hzk   输入hiveserver2进程启动的用户名称
Enter password for jdbc:hive2://bigdata001:10000/default: 无需输入密码

0: jdbc:hive2://bigdata001:10000/default> show databases;
+----------------+--+
| database_name  |
+----------------+--+
| default        |
+----------------+--+
1 row selected (0.782 seconds)
0: jdbc:hive2://bigdata001:10000/default> 





# idea开发



```
<properties>
        <hadoop.version>2.6.0-cdh5.16.2</hadoop.version>
    </properties>

    <repositories>
        <repository>
            <id>cloudera</id>
            <url>https://repository.cloudera.com/artifactory/cloudera-repos/</url>
        </repository>
    </repositories>

    <dependencies>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>${hadoop.version}</version>
        </dependency>
    </dependencies>

```



