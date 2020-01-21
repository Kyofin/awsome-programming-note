# spark必知必会

## spark必须知道的概念

一个**Job**被拆分成若干个**Stage**，每个Stage执行一些计算，产生一些中间结果。它们的目的是最终生成这个Job的计算结果。而每个**Stage**是一个task set，包含若干个**task**。Task是Spark中最小的工作单元，在一个executor上完成一个特定的事情。



## spark源码编译

命令行设置代理翻墙

```
export http_proxy=10.0.0.17:1087
```

使用spark项目内置脚本执行编译

```
~/openSource/spark(02b510728c) » ./build/mvn -DskipTests clean package      
```

编译成功后使用idea打开，并将pom.xml移入maven栏目，项目会自动加载。

但如果直接启动example模块的例子还是会报错。

![](http://image-picgo.test.upcdn.net/img/20200115140027.png)

是因为pom.xml的spark依赖都是provided范围。需要修改idea的启动选项。

![](http://image-picgo.test.upcdn.net/img/20200115140042.png)

并指定master，就可以正常运行了。

![](http://image-picgo.test.upcdn.net/img/20200115140258.png)





## Spark submit 不同的模式运行

### 1. 运行在 yarn集群上

系统环境变量需要有yarn的配置。

![](https://i.loli.net/2019/11/20/bLxrwyP3DJ6QglY.png)

因为spark提交作业时会找yarn的配置去获取资源。

- Spark on YARN 集群上 yarn-cluster 模式运行

  ```SHELL
  ./bin/spark-submit --class org.apache.spark.examples.SparkPi \
      --master yarn \
      --deploy-mode cluster \
      --driver-memory 2g \
      --executor-memory 2g \
      --executor-cores 1 \
      examples/jars/spark-examples*.jar \
      10
  ```
  

  
- spark on yarn 集群上 yarn-client模式运行

  ```SHELL
  ./bin/spark-submit --class org.apache.spark.examples.SparkPi \
      --master yarn \
      --deploy-mode client \
      --driver-memory 2g \
      --executor-memory 2g \
      --executor-cores 1 \
      examples/jars/spark-examples*.jar \
      10
  ```



注意 Spark on YARN 支持两种运行模式，分别为`yarn-cluster`和`yarn-client`，具体的区别可以看[这篇博文](http://www.iteblog.com/archives/1223)，从广义上讲，yarn-cluster适用于生产环境；而yarn-client适用于交互和调试，也就是希望快速地看到application的输出。



## sparkSql中连接自定义sql外部源

使用sparksql通常都是建立连接时指定特定表，如下

```scala
sqlContext.read.format("jdbc").options(Map("url"->"jdbc:mysql://10.10.190.102:3306/db",
            "driver" -> "com.mysql.jdbc.Driver",
            "dbtable"->"temp",
            "user"->"root","password"->"root")).load()

```

但是我们也可以在建立该连接时指定自定义的sql作为一个表。如：

```scala
sqlContext.read.format("jdbc").option("driver","com.cloudera.impala.jdbc4.Driver").option("url","jdbc:impala://cdh02:21050/honghui_sqlserver").option("dbtable","(select  a.patid as patient_id,    b.xxdm as abo_code,    c.ysdm2 as assistant_ii_staff_name,    c.ysdm1 as assistant_i_staff_name,    b.bmy as coder_signature,    d.bah as medical_record_number,    b.bazl as med_record_code,    b.blbh as pathology_number,    b.blzd as pathologic_diagnosis_name,    d.csd_s as birth_place_province,    d.csd_x as birth_place_county,    d.birth as birth_date,    b.cych as discharge_room,    b.cyks as discharge_departmen,    a.cyrq as discharge_date,    e.zddm as dis_main_diagnosis_code,    e.zdmc as dis_main_diagnosis_name,    d.lxdh as telephone_number,    d.dwyb as work_unit_addr_postal_code,    d.dwdh as work_company_tel,    d.dwmc as work_company_name,    d.gjbm as nationality_code,    b.hkdz as registered_house_number,    b.hkyb as registered_postal_code,    a.sfzh as id_number,    d.hyzk as marital_status_code,    b.jgdm as native_place_province,    a.ysdm as training_doctor_signature,    f.cardno as health_card_number,    a.cyfs as leave_way_code,    b.lxdz as linkman_addr_house_number,    b.lxdh as linkman_tel,    b.lxrm as linkman_name,    b.lxgx as linkman_patient_relationship_code,    g.mzdm as anesth_type_code,    h.zddm as diagnostic_disease_code,    h.zdmc as diagnostic_name,    d.mzbm as nation,    b.brnl as age,    b.rych as admiss_room,    b.ryks as admission_department,    b.ryrq as into_date,    b.rytj as adm_way_code,    i.zyts as actual_hospital_stay,    g.ssdm as surgery_code,    g.ssmc as surgery_operation_name,    g.kssj as surgery_operation_target_name,    g.qkdj as op_incision_grade_code,    g.ysdm as op_doctor_name,    b.xyf as western_medicine_fee,    d.lxdz as residence_door,    d.lxyb as residence_post,    j.tz as newborn_weight,    k.tz as newborn_admiss_weight,    a.sex as gender_code,    a.hzxm as name,    b.xxdm as blood_type_code,    b.sxf as blood_fee,    d.zybm as occupational_category_code,    b.zkhs as qc_nurse_signs,    b.barq as qc_date,    b.zkys as quality_control_doctor_signature,    b.qtf4 + b.ssf as surgical_treatment_costs,    b.qtf4 as surgical_treatment_narcotize_costs,    b.ssf as surgical_treatment_operating_costs,    b.zyf as chinese_herbal_medicine_fee,    b.zyf as chinese_patent_medicine_fee,    b.zrys as chief_doctor_signature,    b.zzys as attending_doctor_signature,    a.zycs as hospitalization_times,    a.syxh as admisson_no,    m.ysdm1 as hospitalization_doctor_signature,    l.zje as total_hospitalization_fee,    l.zfje as total_hospitalization_self_fee,    b.zkks as transfer_branch,    b.hlf as self_care_code,    a.lrrq as createdate    from zy_brsyk as a     left join bq_ys_syjb as b on a.syxh = b.syxh    left join bq_ys_bassk as c on a.syxh = c.syxh    left join zy_brxxk as d on a.patid = d.patid    left join zy_brzdqk as e on a.syxh = e.syxh and e.zdlb = 2    left join sf_brcard as f on a.patid = f.patid    left join ss_ssdjk as g on a.syxh = g.syxh    left join zy_brzdqk as h on a.syxh = h.syxh and h.zdlb = 1    left join zy_brjsk as i on a.syxh = i.syxh    left join zy_babysyk as j on a.syxh = j.syxh    left join bq_yetzjlk as k on a.syxh = k.syxh    left join zy_brfymxk2 as l on a.syxh = l.syxh    left join bq_ys_cyxj as m on a.syxh = m.syxh) as aa").load
```



## spark-shell连接phoenix

- 需要用到的jar到服务器上
  下载地址：https://pan.baidu.com/s/1bpGtcficrxJVd_RxDbfTsw
  ![](http://183.6.50.10:4999/server/../Public/Uploads/2019-11-20/5dd49b3d1a745.png)

```shell
scp spark-extra-lib/phoenix-*.jar root@agent1:/root/spark-extra-libs
```

- 启动spark shell

  在spark-shell 启动时指定加载的 lib，解决在 spark-shell中使用 phoenix 的问题

```shell
spark-shell --driver-library-path=/root/spark-extra-libs/* --driver-class-path=/root/spark-extra-libs/*
```

- 测试连接 phoenix

首先输入 `:paste`

```
val df = spark.read 
      .format("jdbc") 
      .option("driver", "org.apache.phoenix.jdbc.PhoenixDriver") 
      .option("url", "jdbc:phoenix:cdh01:2181")
      .option("dbtable", "gzhonghui_new.patient_basic_information") 
      .option("phoenix.schema.isNamespaceMappingEnabled", "true") 
      .load() 
```

Ctrl +D 执行

```
df.show(1,20,true)
```

![](https://i.loli.net/2019/11/20/GYyEVTj8I7b4gS6.png)



## Spark-sql工具操作hive

在conf目录下放入hive-stie.xml后使用`bin/spark-sql`即可。

可以在启动时加入master即可指定spark standalone资源(非必须)。

![](http://image-picgo.test.upcdn.net/img/20191223092548.png)



### **创建表**

#### **1，以文本方式存储**

```
create external table mytest1(id bigint, name string)  row format delimited fields terminated by ','  location 'hdfs://bigserver1:9000/test/spark/tank3';  
```

**这种方式创建的表，是以文本的形式存储的**

#### **2，以parquet存储**

```
CREATE TABLE mytest3 (id bigint, name string)  USING HIVE OPTIONS(fileFormat 'PARQUET')  location 'hdfs://cdh04:8020/test/spark/tank4';  
```

**这种创建表的方式，指定了文件存储方式，在用scala读取时会方便一些。**但要注意有写入该文件夹**权限**。

**在这里要注意一点，如果没有指定location的话，默认会装到SPARK thrift的warehouse**。

写入数据（这样写很慢，建议用hdfs写入再查）

```
INSERT INTO mytest3 VALUES (1,"zhang"), (2,"tank")  
```



## 内置函数

共267个

![](http://image-picgo.test.upcdn.net/img/20200110185755.png)

```
--查看所有内置函数
SHOW FUNCTIONS ;
--查看某个函数的描述
DESCRIBE FUNCTION !;
 --查看某个函数的具体使用方法
DESCRIBE FUNCTION EXTENDED !;
```





## 开启thriftserver启动spark-sql远程服务

### 启动

- 使用yarn启动

  设置环境变量`export HADOOP_CONF_DIR=/etc/hadoop/conf`。

  ```shell
  ./sbin/start-thriftserver.sh \
  				--hiveconf hive.server2.thrift.port=14000 \
          --hiveconf hive.exec.mode.local.auto=true  \
          --hiveconf hive.auto.convert.join=true     \
          --hiveconf hive.mapjoin.smalltable.filesize=50000000 \
          --name thriftserver    \
          --master yarn-client \
          --driver-cores    5   \
          --driver-memory   5G  \
          --executor-memory 5g \
          --total-executor-cores 10\
          --conf spark.serializer=org.apache.spark.serializer.KryoSerializer \
          --conf spark.kryoserializer.buffer.max.mb=1024 \
          --conf spark.storage.memoryFraction=0.2
  ```

  启动端口是14000，即可用jdbc连接。

  ![](http://image-picgo.test.upcdn.net/img/20191223100247.png)

- 使用spark-alone模式启动

  ```
  ./sbin/start-thriftserver.sh \
  				--hiveconf hive.server2.thrift.port=14000 \
          --hiveconf hive.exec.mode.local.auto=true  \
          --hiveconf hive.auto.convert.join=true     \
          --hiveconf hive.mapjoin.smalltable.filesize=50000000 \
          --name thriftserver    \
          --master spark://cdh01:7077 \
          --driver-cores    5   \
          --driver-memory   5G  \
          --executor-memory 5g \
          --total-executor-cores 10\
          --conf spark.serializer=org.apache.spark.serializer.KryoSerializer \
          --conf spark.kryoserializer.buffer.max.mb=1024 \
          --conf spark.storage.memoryFraction=0.2
  ```

  ![](http://image-picgo.test.upcdn.net/img/20191223194248.png)

- 使用local模式启动

  ```
  ./sbin/start-thriftserver.sh --hiveconf hive.server2.thrift.port=14000
  ```

### 连接使用

```
./bin/beeline
Beeline version 1.2.1.spark2 by Apache Hive
beeline> !connect jdbc:hive2://localhost:14000
```

![](http://image-picgo.test.upcdn.net/img/20191223094140.png)

### 关闭服务

```
[root@cdh01 spark-2.4.4-bin-hadoop2.6]# sbin/stop-thriftserver.sh 
stopping org.apache.spark.sql.hive.thriftserver.HiveThriftServer2
```



### troubleshooting

使用beeline插入数据时报错。

![](http://image-picgo.test.upcdn.net/img/20191223132409.png)

#### 解决方法：

在使用beeline连接时输入用户名和密码。

此处的root用户是linux的用户，需先同步到hdfs才可以使用，即`hdfs dfsadmin -refreshUserToGroupsMappings`

![](http://image-picgo.test.upcdn.net/img/20191223132546.png)



## spark ui界面分析

部署spark alone

```
cdh01 (主节点)
cdh02 (slave)
cdh03 (slave)
cdh04 (slave)
cdh05 (slave)
```

在cdh01启动spark alone模式集群。

启动集群后，通过启动thriftserver的应用来观察UI。

```
./sbin/start-thriftserver.sh \
				--hiveconf hive.server2.thrift.port=14000 \
        --hiveconf hive.exec.mode.local.auto=true  \
        --hiveconf hive.auto.convert.join=true     \
        --hiveconf hive.mapjoin.smalltable.filesize=50000000 \
        --name thriftserver    \
        --master spark://cdh01:7077 \
        --driver-cores    5   \
        --driver-memory   5G  \
        --executor-memory 5g \
        --total-executor-cores 10\
        --conf spark.serializer=org.apache.spark.serializer.KryoSerializer \
        --conf spark.kryoserializer.buffer.max.mb=1024 \
        --conf spark.storage.memoryFraction=0.2
```

上面指定了每个executor内存为5g，总共核数10。

![](http://image-picgo.test.upcdn.net/img/20191223202259.png)

打开spark ui。可以发现drvier指定的内存和核数在spark首页都是看不到的。它统计的都是worker工作节点的占用资源，即非cdh01的节点。



测试一个很耗时的操作，读hive上的一张大数目为1000000010的表并写入到新表中。

```
create table hdfs2_id as select *,id as id2 from hdfswriter2;
```

![](http://image-picgo.test.upcdn.net/img/20191223203035.png)



再进入该sql 的作业页面。

![](http://image-picgo.test.upcdn.net/img/20191223203123.png)



可以看到该作业用了38分钟。只有一个stage4，而且stage4耗时最长。

![](http://image-picgo.test.upcdn.net/img/20191223203515.png)



点击进入stage4的监控页面。

可以看到每个工作节点都读了不同数目的数据。

并且task任务的总数和`--total-executor-cores 10`是相等的。即使这里有四个节点，但是只有0和1工作节点分到3个任务，而2和3都只分到2个。因此，我们能调整这个参数来挺高并发数。

![](http://image-picgo.test.upcdn.net/img/20191223204009.png)