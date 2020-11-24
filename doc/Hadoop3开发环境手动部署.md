## Hadoop3开发环境手动部署

## 相关版本下载

hadoop3.1.1

https://archive.apache.org/dist/hadoop/common/hadoop-3.1.1/hadoop-3.1.1.tar.gz

hive3.1.0

http://archive.apache.org/dist/hive/hive-3.1.0/apache-hive-3.1.0-bin.tar.gz



## 环境配置

解压好jdk、hive、hadoop后。



### 配置环境变量

```
export HADOOP_HOME=/home/bigdata/hadoop
export HIVE_HOME=/home/bigdata/hive
export PATH=${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin:${HIVE_HOME}/bin:$PATH
export JAVA_HOME=/usr/java/jdk1.8.0_181
CLASSPATH=$JAVA_HOME/lib/
PATH=$PATH:$JAVA_HOME/bin
export PATH JAVA_HOME CLASSPATH

```



### java环境配置

```
[bigdata@server-0001 ~]$ which java
/usr/java/jdk1.8.0_181/bin/java
```





### hosts 用内网ip配置

![image-20201119171433768](http://image-picgo.test.upcdn.net/img/20201119171433.png)



### 配置本机免密登录

```
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 0600 ~/.ssh/authorized_keys

```





## 部署hadoop

### 配置修改

#### 配置workers文件。

```
server-0001
```



#### 配置hadoop-env.sh文件。

```
export JAVA_HOME=/usr/java/jdk1.8.0_181
export HADOOP_SSH_OPTS="-p 22088"      
```

由于本机ssh的端口是22088，所以需要上面这样配置。



#### 配置core-site.xml

```
<property>
        <name>fs.defaultFS</name>
        <value>hdfs://server-0001:8020</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/home/bigdata/hadoop/tmp</value>
    </property>
```



#### 配置hdfs-site.xml

```
<property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>

    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>server-0001:9868</value>
    </property>
    
    <property>
        <name>dfs.namenode.secondary.https-address</name>
        <value>server-0001:9869</value>
    </property>
```



#### 配置yarn-site.xml

```
 <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.resourcemanager.webapp.address</name>
        <value>server-0001:18088</value>
    </property>
```



#### 配置mapred-site.xml

```
 <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>

<property>
  <name>yarn.app.mapreduce.am.env</name>
  <value>HADOOP_MAPRED_HOME=/home/bigdata/hadoop</value>
</property>
<property>
  <name>mapreduce.map.env</name>
  <value>HADOOP_MAPRED_HOME=/home/bigdata/hadoop</value>
</property>
<property>
  <name>mapreduce.reduce.env</name>
  <value>HADOOP_MAPRED_HOME=/home/bigdata/hadoop</value>
</property>
```



### 格式化

```
hdfs namenode -format
```



### 启动

```
start-dfs.sh
start-yarn.sh
```





### 浏览器访问UI

http://10.93.6.164:9870/explorer.html#/

http://10.93.6.164:18088/cluster/nodes









## 部署hive



### 初始化meta表

#### 使用derby做元数据库

测试时可以使用内嵌的数据库来做元数据库。

```
  $ $HIVE_HOME/bin/schematool -dbType derby -initSchema
```



#### 使用mysql做元数据库

一般我们都是用mysqll做元数据库。

修改`conf/hive-site.xml`

```
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
<property>
  <name>javax.jdo.option.ConnectionURL</name>
  <value>jdbc:mysql://10.93.6.6:3306/hzk_hive</value>
  <description>the URL of the MySQL database</description>
</property>

<property>
  <name>javax.jdo.option.ConnectionDriverName</name>
  <value>com.mysql.jdbc.Driver</value>
</property>

<property>
  <name>javax.jdo.option.ConnectionUserName</name>
  <value>root</value>
</property>

<property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>Root@123</value>
    </property>

   <property>
        <name>hive.cli.print.header</name>
        <value>true</value>
    </property>

 
</configuration>

```

加mysql驱动加入lib目录。

![image-20201119153312598](http://image-picgo.test.upcdn.net/img/20201119153312.png)

执行命令初始schema。

```
  $ $HIVE_HOME/bin/schematool -dbType mysql -initSchema
```

