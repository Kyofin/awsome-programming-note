# spark必知必会

## Spark submit 不同的模式运行

### 1. 运行在 yarn集群上

系统环境变量需要有yarn的配置。

![](https://i.loli.net/2019/11/20/bLxrwyP3DJ6QglY.png)

因为spark提交作业时会找yarn的配置去获取资源。

- Spark on YARN 集群上 yarn-cluster 模式运行

  ```SHELL
  ~/opt/spark-2.4.4-bin-hadoop2.6 » ./bin/spark-submit --class org.apache.spark.examples.SparkPi \                                                              huzekang@huzekangdeMacBook-Pro
      --master yarn \
      --deploy-mode cluster \
      --driver-memory 2g \
      --executor-memory 1g \
      --executor-cores 1 \
      --queue thequeue \
      examples/jars/spark-examples*.jar \
      10
  ```

  

- spark on yarn 集群上 yarn-client模式运行

  ```SHELL
  ~/opt/spark-2.4.4-bin-hadoop2.6 » ./bin/spark-submit --class org.apache.spark.examples.SparkPi \                                                              huzekang@huzekangdeMacBook-Pro
      --master yarn \
      --deploy-mode client \
      --driver-memory 2g \
      --executor-memory 1g \
      --executor-cores 1 \
      --queue thequeue \
      examples/jars/spark-examples*.jar \
      100
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