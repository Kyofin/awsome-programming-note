# Mac下安装Kylin3.1.1

## 环境准备

本地安装大数据cdh5.16.2环境

spark2.4.6-cdh5.16.2。

kylin官网下载apache-kylin-3.1.1-bin-cdh57.tar.gz



## 修改kylin检查脚本

Hive、Spark、Flink检查环境时会类似下面的错误

```
find: -printf: unknown primary or operator
Current HIVE_LIB is not valid, please export HIVE_LIB='YOUR_LOCAL_HIVE_LIB'
```

错误原因：mac 下面 find 查找文件命令行不支持 -printf ‘%p:’ 

修改`find-hive-dependency.sh`

```
hive_lib=`find -L ${hive_lib_dir} -name '*.jar' ! -name '*druid*' ! -name '*slf4j*' ! -name '*avatica*' ! -name '*calcite*' ! -name '*jackson-datatype-joda*' ! -name '*derby*' | awk '{printf "%s:", $1}' | sed 's/:$//'`
```

修改`find-spark-dependency.sh`

```
 spark_dependency=`find -L $spark_home/jars -name '*.jar' ! -name '*slf4j*' ! -name '*calcite*' ! -name '*doc*' ! -name '*test*' ! -name '*sources*' | awk '{printf "%s:", $1}' | sed 's/:$//'`
```

修改`find-flink-dependency.sh`

```
    flink_dependency=`find -L $flink_home/lib -name '*.jar' ! -name '*shaded-hadoop*' ! -name 'kafka*' ! -name '*log4j*' ! -name '*slf4j*' ! -name '*calcite*' ! -name '*doc*' ! -name '*test*' ! -name '*sources*' | awk '{printf "%s:", $1}' | sed 's/:$//'`
```



## 补充缺失依赖到webapp

启动时报错：

```
java.lang.ClassCastException: com.fasterxml.jackson.datatype.jdk8.Jdk8Module cannot be cast to com.fasterxml.jackson.databind.Module



java.lang.ClassCastException: com.fasterxml.jackson.datatype.jsr310.JavaTimeModule cannot be cast to com.fasterxml.jackson.databind.Module 缺失依赖包

```



maven仓库中下载依赖包：

jackson-datatype-jdk8-2.10.0.jar

jackson-datatype-jsr310-2.10.0.jar



放入目录:

/Users/huzekang/cdh5.16/apache-kylin-3.1.1-bin-cdh57/tomcat/webapps/kylin/WEB-INF/lib

## 启动大数据环境

hbase需要启动。

hdfs和yarn需要启动。

mr-jobhistory也要启动，不然在构建cube时会报错！

```
/mr-jobhistory-daemon.sh start historyserver                            
```



## 启动kylin

```
~/cdh5.16/apache-kylin-3.1.1-bin-cdh57 » bin/kylin.sh start
```

访问主机: http://hostname:7070
使用用户名登陆：ADMIN
使用密码登陆：KYLIN



