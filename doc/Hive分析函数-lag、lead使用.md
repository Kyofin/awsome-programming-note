# Hive分析函数-lag、lead使用

lead是向下取n行。LEAD(col,n,DEFAULT) 用于统计窗口内往下第n行值

lag是向上取n行。LAG(col,n,DEFAULT) 用于统计窗口内往上第n行值

## 经典案例

求用户在每个网页逗留的时长？

### 样本数据如下

![image-20200731120009136](http://image-picgo.test.upcdn.net/img/20200731120009.png)



### 测试脚本如下

```sql
drop table user_access_log;
create table user_access_log
(
    user_id string,
    t_time  string,
    url     string
) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';

insert into user_access_log
values ('Peter', '2015-10-12 01:10:00', 'url1'),
       ('Peter', '2015-10-12 01:15:10', 'url2'),
       ('Peter', '2015-10-12 01:16:40', 'url3'),
       ('Peter', '2015-10-12 02:13:00', 'url4'),
       ('Peter', '2015-10-12 03:14:30', 'url5'),
       ('Marry', '2015-11-12 01:10:00', 'url1'),
       ('Marry', '2015-11-12 01:15:10', 'url2'),
       ('Marry', '2015-11-12 01:16:40', 'url3'),
       ('Marry', '2015-11-12 02:13:00', 'url4'),
       ('Marry', '2015-11-12 03:14:30', 'url5');

select *
from user_access_log;

-- 统计每个网页用户停留的总时间呢
select user_id,
       url,
       t_time as start_time,
       unix_timestamp(lead(t_time) over (partition by user_id order by t_time)) - unix_timestamp(t_time) as period
from user_access_log;
```

## 计算结果

![image-20200731115939977](http://image-picgo.test.upcdn.net/img/20200731115940.png)