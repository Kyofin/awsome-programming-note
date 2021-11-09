# Ozone1.0.0开启kerberos认证

## ipa-server上创建服务

需要创建如下服务：

omservice

scmservice

ozonehttpservice

s3gservice

dn

![image-20210817185048707](http://image-picgo.test.upcdn.net/img/20210817185048.png)

![image-20210817191913072](http://image-picgo.test.upcdn.net/img/20210817191913.png)



## 生成kerberos账号的keytab文件

进入`kadmin.local`终端。

```
kadmin.local:  xst -k /home/huzekang/ozone.keytab omservice/ozone001.hadoop.com@HADOOP.COM

kadmin.local:  xst -k /home/huzekang/ozone.keytab scmservice/ozone001.hadoop.com@HADOOP.COM

kadmin.local:  xst -k /home/huzekang/ozone.keytab ozonehttpservice/ozone001.hadoop.com@HADOOP.COM

kadmin.local:  xst -kt /home/huzekang/ozone.keytab dn/ozone001.hadoop.com@HADOOP.COM
```

查看keytab中的principal

```
klist -kt ~/ozone.keytab 
```

![image-20210817184831405](http://image-picgo.test.upcdn.net/img/20210817184831.png)

## ozone配置kerberos

ozone-site.xm

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<configuration>
    <property>
        <name>ozone.om.address</name>
        <value>ozone001.hadoop.com</value>
    </property>

    <property>
        <name>ozone.scm.client.address</name>
        <value>ozone001.hadoop.com</value>
    </property>
    <property>
        <name>ozone.scm.names</name>
        <value>ozone001.hadoop.com</value>
    </property>



    <property>
        <name>ozone.security.enabled</name>
        <value>true</value>
    </property>
    <property>
        <name>hadoop.security.authentication</name>
        <value>kerberos</value>
    </property>

    <property>
        <name>hdds.scm.kerberos.principal</name>
        <value>scmservice/ozone001.hadoop.com@HADOOP.COM</value>
    </property>
    <property>
        <name>hdds.scm.kerberos.keytab.file</name>
        <value>/Users/huzekang/study/bigdata-spark/spark-ozone/src/main/resources/ozone.keytab</value>
    </property>
    <property>
        <name>hdds.scm.http.kerberos.principal</name>
        <value>ozonehttpservice/ozone001.hadoop.com@HADOOP.COM</value>
    </property>
    <property>
        <name>hdds.scm.http.kerberos.keytab</name>
        <value>/Users/huzekang/study/bigdata-spark/spark-ozone/src/main/resources/ozone.keytab</value>
    </property>



    <property>
        <name>ozone.om.kerberos.principal</name>
        <value>omservice/ozone001.hadoop.com@HADOOP.COM</value>
    </property>
    <property>
        <name>ozone.om.kerberos.keytab.file</name>
        <value>/Users/huzekang/study/bigdata-spark/spark-ozone/src/main/resources/ozone.keytab</value>
    </property>
    <property>
        <name>ozone.om.http.kerberos.principal</name>
        <value>ozonehttpservice/ozone001.hadoop.com@HADOOP.COM</value>
    </property>
    <property>
        <name>ozone.om.http.kerberos.keytab</name>
        <value>/Users/huzekang/study/bigdata-spark/spark-ozone/src/main/resources/ozone.keytab</value>
    </property>



    <property>
        <name>ozone.s3g.authentication.kerberos.principal</name>
        <value>s3g/ozone001.hadoop.com@HADOOP.COM</value>
    </property>
    <property>
        <name>ozone.s3g.keytab.file</name>
        <value>/Users/huzekang/study/bigdata-spark/spark-ozone/src/main/resources/ozone.keytab</value>
    </property>

  
</configuration>
```



hdfs-site.xml

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<configuration>
    <property>
        <name>dfs.datanode.kerberos.principal</name>
        <value>dn/ozone001.hadoop.com@HADOOP.COM</value>
    </property>
 <property>
        <name>dfs.datanode.keytab.file</name>
        <value>/home/huzekang/ozone.keytab</value>
    </property>

</configuration>
```





## 测试安全的ozone集群

### Java客户端

使用OzoneClient进行测试

![image-20210818120541140](http://image-picgo.test.upcdn.net/img/20210818120541.png)

准备好如下文件：

![image-20210818120600980](http://image-picgo.test.upcdn.net/img/20210818120601.png)

可以看到运行后的结果：

![image-20210818120653202](http://image-picgo.test.upcdn.net/img/20210818120653.png)