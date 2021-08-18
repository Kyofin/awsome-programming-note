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
        <name>ozone.metadata.dirs</name>
        <value>/home/huzekang/software/ozone-1.0.0/metadata</value>
    </property>

<property>
    <name>dfs.datanode.data.dir</name>
    <value>file:///home/huzekang/software/ozone-1.0.0/data</value>
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
    <value>/home/huzekang/ozone.keytab</value>
</property>
<property>
    <name>hdds.scm.http.kerberos.principal</name>
    <value>ozonehttpservice/ozone001.hadoop.com@HADOOP.COM</value>
</property>
<property>
    <name>hdds.scm.http.kerberos.keytab</name>
    <value>/home/huzekang/ozone.keytab</value>
</property>



<property>
    <name>ozone.om.kerberos.principal</name>
    <value>omservice/ozone001.hadoop.com@HADOOP.COM</value>
</property>
<property>
    <name>ozone.om.kerberos.keytab.file</name>
    <value>/home/huzekang/ozone.keytab</value>
</property>
<property>
    <name>ozone.om.http.kerberos.principal</name>
    <value>ozonehttpservice/ozone001.hadoop.com@HADOOP.COM</value>
</property>
<property>
    <name>ozone.om.http.kerberos.keytab</name>
    <value>/home/huzekang/ozone.keytab</value>
</property>






</configuration>
```



hdfs-site.xml

```
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

使用ozone自带测试工具写入数据

```
 bin/ozone freon rk --numOfVolumes=1 --numOfBuckets=1 --numOfKeys=100
```

发现无法写入，提示需要kerberos认证。

![image-20210817190358241](http://image-picgo.test.upcdn.net/img/20210817190358.png)