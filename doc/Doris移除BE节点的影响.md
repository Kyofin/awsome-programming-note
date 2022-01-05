# Doris移除BE节点的影响

结论：直接影响查询性能，下降会比较严重。



数据表本来是3副本的，下面模拟be节点挂了1台和2台的情况。



所需配置：Centos7.4   3台  8core 32GB

部署方式：1FE+3BE混合部署

测试版本：Doris0.14.13.1

数据量为：214931640条10.387 GB



数据样例：

| person_name | gender_code | birthdate  | admin_org              | work_unit | live_addres | person_type | inject_date | inject_time | com_short_n | vaccine_typ | organ_name             | input_area_ | record_area | id                          |
| ----------- | ----------- | ---------- | ---------------------- | --------- | ----------- | ----------- | ----------- | ----------- | ----------- | ----------- | ---------------------- | ----------- | ----------- | --------------------------- |
| 丁三        | 1           | 1953-12-22 | xx镇卫生院预防接种门诊 | xxx村     |             | 04_29       | 2021-09-10  | 2           | 深圳xx药业  | 9           | xx镇卫生院预防接种门诊 | 44098322    | 44098322    | 233148002021-09-11 03:12:49 |

```sql
show create table  maoming_info

/**2.7s**/
SELECT count(1) from maoming_info ;
/**1.5s**/
SELECT  count(1) from maoming_info where inject_date BETWEEN '2021-06-11' and '2021-07-01';
/**2.8s**/
SELECT  count(DISTINCT person_name) from maoming_info where inject_date BETWEEN '2021-06-01' and '2021-07-01'
/**6s**/
SELECT  count(DISTINCT concat(person_name,com_short_n)) from maoming_info where inject_date BETWEEN '2021-06-01' and '2021-07-01'



/**be2台2.7s**/
SELECT count(1) from maoming_info ;
/**be2台1.4s**/
SELECT  count(1) from maoming_info where inject_date BETWEEN '2021-06-11' and '2021-07-01';
/**be2台5.6s**/
SELECT  count(DISTINCT person_name) from maoming_info where inject_date BETWEEN '2021-06-01' and '2021-07-01';
/**be2台9.1s**/
SELECT  count(DISTINCT concat(person_name,com_short_n)) from maoming_info where inject_date BETWEEN '2021-06-01' and '2021-07-01'


/**be1台7.3s**/
SELECT count(1) from maoming_info ;
/**be1台2.2s**/
SELECT  count(1) from maoming_info where inject_date BETWEEN '2021-06-11' and '2021-07-01';
/**be1台6.7s**/
SELECT  count(DISTINCT person_name) from maoming_info where inject_date BETWEEN '2021-06-01' and '2021-07-01';
/**be1台16.1s**/
set exec_mem_limit=21474836480;
SELECT  count(DISTINCT concat(person_name,com_short_n)) from maoming_info where inject_date BETWEEN '2021-06-01' and '2021-07-01'




```

