# Kudu start



## 模拟数据

```
kudu perf loadgen kudu.master:7051 -keep_auto_table
```



## 用impala-shell加载kudu表

```sql
CREATE EXTERNAL TABLE kudu_gen1
STORED AS KUDU
TBLPROPERTIES (
'kudu.master_addresses' = 'cdh02:7051',
'kudu.table_name' = 'loadgen_auto_7113fbdc7e6c424cb6c458b1f771eaee'
);
```

