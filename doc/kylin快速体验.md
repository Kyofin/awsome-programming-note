[TOC]

## 测试环境

- Cdh5.13 **(使用的是cdh官方提供的quicstart vm 5.13)**

- centos6

- Kylin2.10



## 下载文件

```
wget https://archive.apache.org/dist/kylin/apache-kylin-2.1.0/apache-kylin-2.1.0-bin-cdh57.tar.gz
```

## 解压到/usr/local目录，建立软连接

```
tar -zxvf apache-kylin-2.1.0-bin-cdh57.tar.gz -C /usr/local
cd /usr/local
ln -s apache-kylin-2.1.0-bin-cdh57/ kylin
```

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190419235301.png)



## kylin环境变量配置

```
vim ~/.bashrc
```

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190419235441.png)



## 执行kylin检查脚本

```
bin/check-env.sh 
```

### 检查配置时报错

```
mkdir: Permission denied: user=cloudera, access=WRITE, inode="/":hdfs:supergroup:drwxr-xr-x
Failed to create /kylin. Please make sure the user has right to access /kylin

```



### 解决办法

1. 在hdfs手动创建该目录

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190419234133.png)

2. 在cloudera manager中，转到高级下的hdfs配置并将以下代码放在HDFS Service Configuration Safety Valve中：

```xml
<property> 
  <name> dfs.permissions </ name> 
  <value> false </ value> 
</property>
```



## 启动kylin

```
bin/kylin.sh start
```

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190419234456.png)



打开浏览器看看 http://quickstart.cloudera:7070/kylin/login

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190419234530.png)

默认账号密码 ADMIN / KYLIN



## 测试样本

### 1. 使用官方自带的测试样本

```
bin/sample.sh 
```

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190420000355.png)



### 2. 执行成功后打开hue看看hive下的table多了五张

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190420000443.png)

### 3. reload metadata

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190420001014.png)

### 4. 构建cube

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190420001212.png)

### 5. 选择数据分区范围

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190420001400.png)



### 6. 点击monitor ，查看构建中的cube

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190420001316.png)



此时打开yarn 的resource manager可以看到正在计算的mr

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190420001905.png)



### 7. 执行多表关联查询

#### 在kylin的insight中执行查询

```sql
select sum(KYLIN_SALES.PRICE) 
as price_sum,KYLIN_CATEGORY_GROUPINGS.META_CATEG_NAME,KYLIN_CATEGORY_GROUPINGS.CATEG_LVL2_NAME 
from KYLIN_SALES inner join KYLIN_CATEGORY_GROUPINGS
on KYLIN_SALES.LEAF_CATEG_ID = KYLIN_CATEGORY_GROUPINGS.LEAF_CATEG_ID and 
KYLIN_SALES.LSTG_SITE_ID = KYLIN_CATEGORY_GROUPINGS.SITE_ID
group by KYLIN_CATEGORY_GROUPINGS.META_CATEG_NAME,KYLIN_CATEGORY_GROUPINGS.CATEG_LVL2_NAME
order by KYLIN_CATEGORY_GROUPINGS.META_CATEG_NAME asc,KYLIN_CATEGORY_GROUPINGS.CATEG_LVL2_NAME desc
```



查询结果：

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190420003452.png)



之后在查询速度保持在 0.40s，因为有缓存。



#### 相同语句放到hive和impala测试

##### - impala

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190420003650.png)

impala之后查询会保持在0.37s左右，也是做了缓存。



##### - hive

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190420004221.png)



第二次查询仍是42s左右，没有更快。。。



## 手动导入测试数据

准备测试数据，下载地址：<https://gitee.com/huzekang/cdhproject/tree/master/kylin>

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190425171200.png)



将本地文件上传到hdf (前提是手动在hdfs上已经有如下目录)

```
hdfs dfs -put employee.csv /tmp/data/kylin/
hdfs dfs -put department.csv /tmp/data/kylin
```



导入外部数据到hive

```
beeline -u "jdbc:hive2://quickstart.cloudera:10000/default" -n hive -f create_table.sql
```



使用beeline命令查看hive中导进去的表

```
beeline -u "jdbc:hive2://quickstart.cloudera:10000/default" 
```

连接上hive后，执行查询语句

```
0: jdbc:hive2://quickstart.cloudera:10000/def> select * from employee;
```

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190425171924.png)



或者打开hue也可以看到hive中已经有导进去的表了

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190425171457.png)





## 使用restful方式访问kylin

> 参考官方文档：<http://kylin.apache.org/cn/docs/howto/howto_use_restapi.html#authentication>

### 首先需要获取认证

这里使用python根据账户密码生成一下认证信息。

```
python2.7 -c "import base64; print base64.standard_b64encode('ADMIN:KYLIN')"
```

ADMIN/KYLIN 是kylin的登录账户密码。



可以生成认证

```
QURNSU46S1lMSU4=
```



### 使用api访问kylin

要在请求头带上认证信息`QURNSU46S1lMSU4=`才可以正常访问。

```shell
curl -X POST -H "Authorization: Basic QURNSU46S1lMSU4=" -H "Content-Type: application/json" -d '{ "sql":"select count(*) from KYLIN_ACCOUNT", "project":"learn_kylin" }' http://10.0.0.62:7070/kylin/api/query
```

可以得到响应

```json
{
    "columnMetas": [
        {
            "isNullable": 0,
            "displaySize": 19,
            "label": "EXPR$0",
            "name": "EXPR$0",
            "schemaName": null,
            "catelogName": null,
            "tableName": null,
            "precision": 19,
            "scale": 0,
            "columnType": -5,
            "columnTypeName": "BIGINT",
            "readOnly": true,
            "autoIncrement": false,
            "caseSensitive": true,
            "currency": false,
            "definitelyWritable": false,
            "searchable": false,
            "signed": true,
            "writable": false
        }
    ],
    "results": [
        [
            "10000"
        ]
    ],
    "cube": "CUBE[name=kylin_sales_cube]",
    "affectedRowCount": 0,
    "isException": false,
    "exceptionMessage": null,
    "duration": 185,
    "totalScanCount": 0,
    "totalScanBytes": 0,
    "hitExceptionCache": false,
    "storageCacheUsed": false,
    "pushDown": false,
    "partial": false
}
```

