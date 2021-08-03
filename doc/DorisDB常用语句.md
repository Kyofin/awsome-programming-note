# DorisDB常用语句

## 权限相关

创建角色并赋予角色权限，最后创建用户赋予对应角色

```
create ROLE read_and_write_test;

GRANT SELECT_PRIV,LOAD_PRIV,ALTER_PRIV,CREATE_PRIV,DROP_PRIV ON test.* TO ROLE 'read_and_write_test';

CREATE USER 'hzk'@'%' IDENTIFIED BY '123456' DEFAULT ROLE 'read_and_write_test';
```



删除用户

```
DROP USER 'hzk'
```









## 建表相关

### 建表注意事项

建表中有三个注意事项，这几个选择会比较大的影响到测试结果：

1. Bucket的数量选择和分桶键的选择

   Bucket的数量是对性能影响比较大的因素之一，首先我们希望选择合理的分桶键（DISTRIBUTED BY HASH(key))，来保证数据在各个bucket中尽可能均衡，如果碰到数据倾斜严重的数据可以使用多列作为分桶键，或者采用MD5 hash以后作为分桶键，具体可以参考[分桶键选择](http://doc.dorisdb.com/2142135)，这里我们都统一使用唯一的key列。

2. 建表的数据类型

   数据类型的选择对性能测试的结果是有一定影响的，比如Decimal/String的运算一般比int/bigint要慢，所以在实际场景中我们应该尽可能准确的使用数据类型，从而达到最好的效果，比如可以使用Int/Bigint的字段就尽量避免使用String，如果是日期类型也多使用Date/Datetime以及相对应的时间日期函数，而不是用string和相关字符串操作函数来处理

3. 字段是否可以为空

### 明细模型

建明细表，副本数为1，数据按event_time, event_type字段进行排序

```
CREATE TABLE IF NOT EXISTS detail (
    event_time DATETIME NOT NULL COMMENT "datetime of event",
    event_type INT NOT NULL COMMENT "type of event",
    user_id INT COMMENT "id of user",
    device_code INT COMMENT "device of ",
    channel INT COMMENT ""
)
DUPLICATE KEY(event_time, event_type)
DISTRIBUTED BY HASH(user_id) BUCKETS 8
PROPERTIES (
"replication_num" = "1"
);

```



## 集群管理

检查fe节点

```
SHOW PROC '/frontends'
```

检查be节点

```
 SHOW PROC '/backends'
```



增加be节点

```
ALTER SYSTEM ADD BACKEND "10.93.11.247:9050";
```

如果出现错误，可以移除be。

```
alter system decommission backend "10.93.11.247:9050";
alter system dropp backend "10.93.11.247:9050";
```



检查加载的plugin

```
show plugins
```



## 调优相关

```
SHOW TABLET FROM example_db.table_name;
```

该语句用于显示 tablet 相关的信息（仅管理员使用）。



```
SHOW DATA from ssb.lineorder 
```

该语句用于展示表的数据容量大小、副本数量以及统计行数。



```
use chinaunicom_medical_health2_dev2_1;
show data;
```

该语句用于展示整库的数据容量大小、副本数量以及统计行数。



## 常用参数配置

```
set global is_report_success = true
```

在mysql客户端中执行，fe页面上可以看到查询的SQL。

![image-20210624132529327](http://image-picgo.test.upcdn.net/img/20210624132529.png)



```
set  query_timeout = 259200;
```

用于设置查询超时，单位为秒。该变量会作用于当前连接中所有的查询语句，以及 INSERT 语句。默认为300秒，即 5 分钟。要在doris中执行etl的，就要调大这个参数。

 

```
SET exec_mem_limit =10147483648 ;
```

用于设置单个查询计划实例所能使用的内存限制。默认为 2GB，单位为B/K/KB/M/MB/G/GB/T/TB/P/PB, 默认为B。