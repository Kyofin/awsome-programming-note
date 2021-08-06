# Hadoop的kerberos认证

## 一、hadoop安装

### 1、下载安装包并解压

```
wget https://archive.apache.org/dist/hadoop/common/hadoop-2.7.1/hadoop-2.7.1.tar.gz
tar -zxvf hadoop-2.7.1.tar.gz -C /usr/local/
cd /usr/local/
mv hadoop-2.7.1/ hadoop
```



### 2、设置hadoop的环境变量

```
vim /etc/profile
```

末尾增加下面语句

```
export HADOOP_HOME=/usr/local/hadoop
export PATH=$PATH:/usr/local/hadoop/bin
```



### 3、添加hdfs用户并修改hdfs的属组

```
groupadd hdfs
useradd hdfs -g hdfs
cat /etc/passwd
chown -R hdfs:hdfs /usr/local/hadoop/
```

![image-20210730160147949](http://image-picgo.test.upcdn.net/img/20210730160148.png)

### 4、修改hdfs配置文件

core-site.xml 

```XML
<configuration>
      <property>
        <name>fs.default.name</name>
        <value>hdfs://kerberos010:9000</value>
      </property>
      <property>
        <name>hadoop.tmp.dir</name>
        <value>/data/vdb1/tmp</value>
      </property>
    </configuration>
```



hdfs-site.xml

```xml
<configuration>
      <property>
        <name>dfs.replication</name>
        <value>2</value>
      </property>
      <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:/data/vdb1/name</value>
      </property>
      <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:/data/vdb1/data</value>
      </property>
      <property>
        <name>dfs.secondary.http.address</name>
        <value>kerberos010:50090</value>
      </property>
    </configuration>
```



mapred-site.xml

```xml
<configuration>
      <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
      </property>
      <property>
        <name>mapreduce.jobhistory.address</name>
        <value>kerberos010:10020</value>
      </property>
      <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>kerberos010:19888</value>
      </property>
</configuration>
```



yarn-site.xml

```xml
<configuration>
    <!-- Site specific YARN configuration properties-->
      <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
      </property>
      <property>
        <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
      </property>
      <property>
        <name>yarn.resourcemanager.address</name>
        <value>kerberos010:8032</value>
      </property>
      <property>
    <name>yarn.resourcemanager.scheduler.address</name>
        <value>kerberos010:8030</value>
      </property>
      <property>
        <name>yarn.resourcemanager.resource-tracker.address</name>
        <value>kerberos010:8031</value>
      </property>
      <property>
        <name>yarn.resourcemanager.admin.address</name>
        <value>kerberos010:8033</value>
      </property>
      <property>
        <name>yarn.resourcemanager.webapp.address</name>
        <value>kerberos010:8088</value>
      </property>
</configuration>
```



slaves

```
kerberos010
kerberos011
```



hadoop-env.sh

```
export JAVA_HOME=/usr/java/jdk1.8.0_181/
```





### 5、创建目录和修改属组

```
mkdir -p /data/vdb1/tmp
mkdir -p /data/vdb1/data
mkdir -p /data/vdb1/name
chown -R hdfs:hdfs /data/vdb1/tmp
chown -R hdfs:hdfs /data/vdb1/data
chown -R hdfs:hdfs /data/vdb1/name
```



### 6、拷贝hadoop安装目录到其他节点

在主节点执行

```
scp -r /usr/local/hadoop/ root@kerberos011:/usr/local/
```

在另外的从节点执行

```
groupadd hdfs
useradd hdfs -g hdfs
mkdir -p /data/vdb1/tmp
mkdir -p /data/vdb1/data
mkdir -p /data/vdb1/name
chown -R hdfs:hdfs /data/vdb1/tmp
chown -R hdfs:hdfs /data/vdb1/data
chown -R hdfs:hdfs /data/vdb1/name
chown -R hdfs:hdfs /usr/local/hadoop/
```

在另外的从节点上配置环境变量。修改`/etc/profile`

```
export HADOOP_HOME=/usr/local/hadoop
export PATH=$PATH:/usr/local/hadoop/bin
```

每个节点间需要配置ssh免密登录，下面只是一个示例。

```
[root@kerberos010 hadoop]# ssh-keygen -b 4096 -t rsa -C gp002
[root@kerberos010 hadoop]# ssh-copy-id -i ~/.ssh/id_rsa.pub -p 22 -f root@kerberos011
```



### 7、格式化hdfs

```
[root@kerberos010 java]# hdfs namenode -format
```



### 8、启动yarn

```
cd /usr/local/hadoop/sbin/
./start-yarn.sh
```





## 二、hdfs配置kerberos认证

### 1、所有节点安装autoconf

```
yum install -y autoconf
```



### 2、所有节点安装gcc

```
yum install -y gcc
```



### 3、安装jsvc

```
wget https://archive.apache.org/dist/commons/daemon/source/commons-daemon-1.2.2-src.tar.gz

tar-zxvf commons-daemon-1.2.2-src.tar.gz

cd /opt/commons-daemon-1.2.2-src/src/native/unix

./support/buildconf.sh
./configure
make
```

检查是否安装完成

```
[root@kerberos010 unix]# ./jsvc -help
```

![image-20210730171435704](http://image-picgo.test.upcdn.net/img/20210730171435.png)

做软连接

```
ln -s /opt/commons-daemon-1.2.2-src/src/native/unix/jsvc  /usr/local/bin/jsvc

```



### 4、修改hdfs-env.sh的配置文件

```
vim /usr/local/hadoop/etc/hadoop/hadoop-env.sh
```

增加以下内容

```
export JSVC_HOME=/opt/commons-daemon-1.2.2-src/src/native/unix
export HADOOP_SECURE_DN_USER=hdfs
```



### 5、创建hdfs的principal并分配免密key

密码都为123456

```
kadmin.local:  addprinc hdfs/kerberos010
kadmin.local:  addprinc hdfs/kerberos011

kadmin.local:  addprinc http/kerberos010
kadmin.local:  addprinc http/kerberos011


kadmin.local:  ktadd -norandkey -k /root/hdfs.keytab hdfs/kerberos010
kadmin.local:  ktadd -norandkey -k /root/hdfs.keytab hdfs/kerberos011

kadmin.local:  ktadd -norandkey -k /root/http.keytab http/kerberos010
kadmin.local:  ktadd -norandkey -k /root/http.keytab http/kerberos011

```



### 6、分发秘钥文件

```
scp /root/hdfs.keytab /root/http.keytab root@kerberos010:/usr/local/hadoop/etc/
scp /root/hdfs.keytab /root/http.keytab root@kerberos011:/usr/local/hadoop/etc/
```





### 7、修改hadoop的配置文件

修改core-site.xml文件，增加以下

```xml
<property>
     <name>hadoop.security.authentication</name>
     <value>kerberos</value>
   </property>
   <property>
     <name>hadoop.security.authorization</name>
     <value>true</value>
   </property>
```



修改修改hdfs-site.xml，加入以下内容

```xml
<property>
    <name>dfs.block.access.token.enable</name>
    <value>true</value>
</property>
<property>
    <name>dfs.namenode.kerberos.principal</name>
    <value>hdfs/kerberos010@HADOOP.COM</value>
</property>
<property>
    <name>dfs.namenode.keytab.file</name>
    <value>/usr/local/hadoop/etc/hdfs.keytab</value>
</property>
<property>
    <name>dfs.namenode.kerberos.internal.spnego.principal</name>
    <value>http/hadoop@HADOOP.COM</value>
</property>
<property>
    <name>dfs.namenode.kerberos.internal.spnego.keytab</name>
    <value>http/kerberos010@HADOOP.COM</value>
</property>
<property>
    <name>dfs.web.authentication.kerberos.principal</name>
    <value>hdfs/kerberos010@HADOOP.COM</value>
</property>
<property>
    <name>dfs.web.authentication.kerberos.keytab</name>
    <value>/usr/local/hadoop/etc/hdfs.keytab</value>
</property>
<property>
    <name>dfs.datanode.kerberos.principal</name>
    <value>hdfs/kerberos010@HADOOP.COM</value>
</property>
<property>
    <name>dfs.datanode.keytab.file</name>
    <value>/usr/local/hadoop/etc/hdfs.keytab</value>
</property>
<property>
    <name>dfs.datanode.address</name>
    <value>0.0.0.0:1004</value>
</property>
<property>
    <name>dfs.datanode.http.address</name>
    <value>0.0.0.0:1006</value>
</property>
```







如果有secondnamenode，则还需要加下面的配置

```xml
<property>
    <name>dfs.secondary.namenode.keytab.file</name>
    <value>/usr/local/hadoop/etc/hdfs.keytab</value>
</property>
<property>
    <name>dfs.secondary.namenode.kerberos.principal</name>
    <value>hdfs/kerberos010@HADOOP.COM</value>
</property>
```

修改yarn-site.xml

```
<property>
    <name>yarn.resourcemanager.principal</name>
    <value>hdfs/kerberos010@HADOOP.COM</value>
</property>
<property>
    <name>yarn.resourcemanager.keytab</name>
    <value>/usr/local/hadoop/etc/hdfs.keytab</value>
</property>
<property>
    <name>yarn.nodemanager.keytab</name>
    <value>/usr/local/hadoop/etc/hdfs.keytab</value>
</property>
<property>
    <name>yarn.nodemanager.principal</name>
    <value>hdfs/kerberos010@HADOOP.COM</value>
</property>
```





分发配置文件到其他节点

```
scp core-site.xml hdfs-site.xml yarn-site.xml root@kerberos011:/usr/local/hadoop/etc/hadoop/
```



### 8、启动hdfs

启动前要用root用户修改keytab的权限，否则使用hdfs用户用keytab登录kerberos时会下面错误。

![image-20210731115823523](http://image-picgo.test.upcdn.net/img/20210731115823.png)

```
chown hdfs:hdfs /usr/local/hadoop/etc/hdfs.keytab 
chmod 660 /usr/local/hadoop/etc/hdfs.keytab

chown hdfs:hdfs /usr/local/hadoop/etc/http.keytab 
chmod 660 /usr/local/hadoop/etc/http.keytab

```



切换root用户为hdfs设置ssh密码

```
passwd hdfs
```

为hdfs配置每天服务器的ssh免密

```
su hdfs
ssh-keygen -b 4096 -t rsa -C gp002
ssh-copy-id -i ~/.ssh/id_rsa.pub -p 22 -f hdfs@kerberos010
ssh-copy-id -i ~/.ssh/id_rsa.pub -p 22 -f hdfs@kerberos011
```





切换Hdfs用户执行下面的脚本

```
su hdfs
/usr/local/hadoop/sbin/start-dfs.sh
```

![image-20210731122633388](http://image-picgo.test.upcdn.net/img/20210731122633.png)

开启kerberos后需要手动启动datanode

```
[root@kerberos010 hadoop]#  /usr/local/hadoop/sbin/start-secure-dns.sh 
```

![image-20210731122432518](http://image-picgo.test.upcdn.net/img/20210731122432.png)

但没成功。namenode日志。

```
2021-07-31 12:51:41,983 WARN org.apache.hadoop.security.authentication.server.KerberosAuthenticationHandler: Failed to login as [hdfs/kerberos010@HADOOP.COM]
javax.security.auth.login.LoginException: No key to store
        at com.sun.security.auth.module.Krb5LoginModule.commit(Krb5LoginModule.java:1119)
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:498)
        at javax.security.auth.login.LoginContext.invoke(LoginContext.java:755)
        at javax.security.auth.login.LoginContext.access$000(LoginContext.java:195)
        at javax.security.auth.login.LoginContext$4.run(LoginContext.java:682)
        at javax.security.auth.login.LoginContext$4.run(LoginContext.java:680)
        at java.security.AccessController.doPrivileged(Native Method)
        at javax.security.auth.login.LoginContext.invokePriv(LoginContext.java:680)
        at javax.security.auth.login.LoginContext.login(LoginContext.java:588)
        at org.apache.hadoop.security.authentication.server.KerberosAuthenticationHandler.init(KerberosAuthenticationHandler.java:221)
        at org.apache.hadoop.security.authentication.server.AuthenticationFilter.initializeAuthHandler(AuthenticationFilter.java:238)
        at org.apache.hadoop.security.authentication.server.AuthenticationFilter.init(AuthenticationFilter.java:227)
        at org.mortbay.jetty.servlet.FilterHolder.doStart(FilterHolder.java:97)
        at org.mortbay.component.AbstractLifeCycle.start(AbstractLifeCycle.java:50)
        at org.mortbay.jetty.servlet.ServletHandler.initialize(ServletHandler.java:713)
        at org.mortbay.jetty.servlet.Context.startContext(Context.java:140)
        at org.mortbay.jetty.webapp.WebAppContext.startContext(WebAppContext.java:1282)
        at org.mortbay.jetty.handler.ContextHandler.doStart(ContextHandler.java:518)
        at org.mortbay.jetty.webapp.WebAppContext.doStart(WebAppContext.java:499)
        at org.mortbay.component.AbstractLifeCycle.start(AbstractLifeCycle.java:50)
        at org.mortbay.jetty.handler.HandlerCollection.doStart(HandlerCollection.java:152)
        at org.mortbay.jetty.handler.ContextHandlerCollection.doStart(ContextHandlerCollection.java:156)
        at org.mortbay.component.AbstractLifeCycle.start(AbstractLifeCycle.java:50)
        at org.mortbay.jetty.handler.HandlerWrapper.doStart(HandlerWrapper.java:130)
        at org.mortbay.jetty.Server.doStart(Server.java:224)
        at org.mortbay.component.AbstractLifeCycle.start(AbstractLifeCycle.java:50)
        at org.apache.hadoop.http.HttpServer2.start(HttpServer2.java:857)
        at org.apache.hadoop.hdfs.server.namenode.NameNodeHttpServer.start(NameNodeHttpServer.java:142)
        at org.apache.hadoop.hdfs.server.namenode.NameNode.startHttpServer(NameNode.java:752)
        at org.apache.hadoop.hdfs.server.namenode.NameNode.initialize(NameNode.java:638)
        at org.apache.hadoop.hdfs.server.namenode.NameNode.<init>(NameNode.java:811)
        at org.apache.hadoop.hdfs.server.namenode.NameNode.<init>(NameNode.java:795)
        at org.apache.hadoop.hdfs.server.namenode.NameNode.createNameNode(NameNode.java:1488)
        at org.apache.hadoop.hdfs.server.namenode.NameNode.main(NameNode.java:1554)
2021-07-31 12:51:41,985 WARN org.mortbay.log: failed SpnegoFilter: javax.servlet.ServletException: org.apache.hadoop.security.authentication.client.AuthenticationException: javax.security.auth.login.LoginException: No key to store
2021-07-31 12:51:41,985 WARN org.mortbay.log: Failed startup of context org.mortbay.jetty.webapp.WebAppContext@16eccb2e{/,file:/usr/local/hadoop/share/hadoop/hdfs/webapps/hdfs}
```

