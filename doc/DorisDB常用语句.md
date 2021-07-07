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





## 建表

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