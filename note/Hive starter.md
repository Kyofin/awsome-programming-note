# Hive starter

添加分区

```sql
ALTER TABLE table_name ADD PARTITION (partCol = 'value1') location 'loc1'; //示例
ALTER TABLE table_name ADD IF NOT EXISTS PARTITION (dt='20130101') LOCATION '/user/hadoop/warehouse/table_name/dt=20130101'; //一次添加一个分区

ALTER TABLE page_view ADD PARTITION (dt='2008-08-08', country='us') location '/path/to/us/part080808' PARTITION (dt='2008-08-09', country='us') location '/path/to/us/part080809';  //一次添加多个分区
```

 

删除分区

```sql
ALTER TABLE login DROP IF EXISTS PARTITION (dt='2008-08-08');

ALTER TABLE page_view DROP IF EXISTS PARTITION (dt='2008-08-08', country='us');
```

 

修改分区

```
ALTER TABLE table_name PARTITION (dt='2008-08-08') SET LOCATION "new location";
ALTER TABLE table_name PARTITION (dt='2008-08-08') RENAME TO PARTITION (dt='20080808');
```

 

添加列

```
ALTER TABLE table_name ADD COLUMNS (col_name STRING);  //在所有存在的列后面，但是在分区列之前添加一列
```

 

修改列

```
CREATE TABLE test_change (a int, b int, c int);

// will change column a's name to a1
ALTER TABLE test_change CHANGE a a1 INT; 

// will change column a's name to a1, a's data type to string, and put it after column b. The new table's structure is: b int, a1 string, c int
ALTER TABLE test_change CHANGE a a1 STRING AFTER b; 

// will change column b's name to b1, and put it as the first column. The new table's structure is: b1 int, a string, c int
ALTER TABLE test_change CHANGE b b1 INT FIRST; 
```



修改表属性:

```
alter table table_name set TBLPROPERTIES ('EXTERNAL'='TRUE');  //内部表转外部表 
alter table table_name set TBLPROPERTIES ('EXTERNAL'='FALSE');  //外部表转内部表
```

 

表的重命名

```
ALTER TABLE table_name RENAME TO new_table_name
```