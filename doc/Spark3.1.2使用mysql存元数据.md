# Spark3使用mysql存元数据

## 介绍

本文主要使用spark3内置hive metastore来直接初始化元数据，存储在mysql中。注意下面的操作是不需要提前下载hive的安装包来安装到环境中的。



## 环境依赖

hadoop 2.7

spark3.1.2

不需要安装hive



### 下载spark3.12

```
wget https://www.apache.org/dyn/closer.lua/spark/spark-3.1.2/spark-3.1.2-bin-hadoop2.7.tgz
```



## 拷贝mysql驱动到jar目录

```
ll jars/
-rw-r--r-- 1 root root   992805 Jan 15 18:19 mysql-connector-java-5.1.41.jar
```



### 配置spark

在spark目录的conf文件夹中配置hive-site.xml

```xml
<property>
      <name>hive.metastore.warehouse.dir</name>
      <value>/apps/spark3_hive_hudi/warehouse</value>
    </property>
 <property>
      <name>javax.jdo.option.ConnectionDriverName</name>
      <value>com.mysql.jdbc.Driver</value>
    </property>

    <property>
      <name>javax.jdo.option.ConnectionURL</name>
      <value>jdbc:mysql://hadoop001:33066/spark3_hive_hudi?createDatabaseIfNotExist=true&amp;useSSL=false</value>
    </property>

    <property>
      <name>javax.jdo.option.ConnectionUserName</name>
      <value>root</value>
    </property>

 <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>Root@123</value>
        <description>password to use against metastore database</description>
    </property>
   <property>
        <name>hive.metastore.schema.verification</name>
        <value>false</value>
    </property>

  <property>
        <name>datanucleus.schema.autoCreateTables</name>
        <value>true</value>
    </property>

```



在spark目录的conf文件夹中配置spark-default.conf

```
spark.sql.catalogImplementation hive
```

