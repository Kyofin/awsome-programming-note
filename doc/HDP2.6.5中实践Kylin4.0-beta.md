# HDP2.6.5中实践Kylin4.0-beta

## 安装

[下载地址](https://mirrors.bfsu.edu.cn/apache/kylin/apache-kylin-4.0.0-beta/apache-kylin-4.0.0-beta-bin.tar.gz)

下载后，将tar包解压到服务中。

由于kylin4移除了hbase组件，所以元数据是保存在mysql中的，所以需要将mysql驱动放`ext`目录。

![image-20210625111916975](http://image-picgo.test.upcdn.net/img/20210625111917.png)

## 配置

启动kylin前，需要手动修改一下`conf/kylin.properties`文件。

### 配置元数据的mysql连接

需要提前在mysql中建好kylin数据库。

```
kylin.metadata.url=kylin_metadata@jdbc,url=jdbc:mysql://10.93.6.6:33066/kylin,username=root,password= Root@123,maxActive=10,maxIdle=10
```

### 配置kylin使用的hdfs目录

```
kylin.env.hdfs-working-dir=/kylin4
```

### 配置zk的连接

```
kylin.env.zookeeper-base-path=/kylin4

kylin.env.zookeeper-connect-string=hadoop002
```

### 配置engine的spark参数

kylin4的cube构建是使用spark的，而且kylin不自带spark客户端，因此需要配置spark的参数。

```
kylin.engine.spark-conf.spark.yarn.queue=kylin

kylin.engine.spark-conf.spark.eventLog.enabled=true
kylin.engine.spark-conf.spark.eventLog.dir=hdfs:///spark2-history/
kylin.engine.spark-conf.spark.history.fs.logDirectory=hdfs:///spark2-history/
kylin.engine.spark-conf.spark.hadoop.yarn.timeline-service.enabled=false


kylin.engine.spark-conf.spark.driver.extraJavaOptions=-Dhdp.version=current
kylin.engine.spark-conf.spark.yarn.am.extraJavaOptions=-Dhdp.version=current
```

### 配置query的spark参数

kylin在执行接收到的查询sql时，第一次会启动一个spark程序作为常驻服务，所有的查询都会由它去执行。

```
kylin.query.spark-conf.spark.driver.extraJavaOptions=-Dhdp.version=current
kylin.query.spark-conf.spark.yarn.am.extraJavaOptions=-Dhdp.version=current
```

### 配置查询下压

```
kylin.query.pushdown.runner-class-name=org.apache.kylin.query.pushdown.PushDownRunnerSparkImpl
```



### 环境变量配置spark_home

在启动kylin前，需要在shell中指定SPARK_HOME变量。这里指向我们基于HDP2.6.5编译的spark2.4.6目录。

```
export SPARK_HOME=/opt/spark-2.4.6-bin-2.11-2.7.3-hdp2.6.5
```



## 使用ssb-kylin测试使用

### 启动kylin

在启动kylin前，需要在shell中指定SPARK_HOME变量。这里指向我们基于HDP2.6.5编译的spark2.4.6目录。

```
export SPARK_HOME=/opt/spark-2.4.6-bin-2.11-2.7.3-hdp2.6.5
```

### 导入ssb-kylin自带的元数据

```
 /opt/apache-kylin-4.0.0-beta-bin/bin/metastore.sh restore cubemeta
```

![image-20210625145647003](http://image-picgo.test.upcdn.net/img/20210625145647.png)

### 刷新kylin的元数据

重启kylin或者在kylin UI中点击`Reload Metadata`。

![image-20210625145800466](http://image-picgo.test.upcdn.net/img/20210625145800.png)

### 手动ssb model的构建engine

![image-20210625145844992](http://image-picgo.test.upcdn.net/img/20210625145845.png)

![image-20210625145857403](http://image-picgo.test.upcdn.net/img/20210625145857.png)

### 执行构建

我这里已经构建过了，所以status是ready的。

![image-20210625145944359](http://image-picgo.test.upcdn.net/img/20210625145944.png)

![image-20210625145929392](http://image-picgo.test.upcdn.net/img/20210625145929.png)

### 执行查询

```
select c_nation, s_nation, d_year, sum(lo_revenue) as lo_revenue
from p_lineorder
left join dates on lo_orderdate = d_datekey
left join customer on lo_custkey = c_custkey
left join supplier on lo_suppkey = s_suppkey
where c_region = 'ASIA' and s_region = 'ASIA'and d_year >= 1992 and d_year <= 1997
group by c_nation, s_nation, d_year
order by d_year asc, lo_revenue desc;
```

![image-20210625150042697](http://image-picgo.test.upcdn.net/img/20210625150042.png)

第一次查询时会比较慢，因为kylin会向yarn申请资源，启动一个spark程序。

![image-20210625145209323](http://image-picgo.test.upcdn.net/img/20210625145209.png)