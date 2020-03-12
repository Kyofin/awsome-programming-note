# **SQLServer开启change Tracking**

# 第一步：

### 1、对库级别操作

```
ALTER DATABASE 数据库名 SET CHANGE_TRACKING = ON (CHANGE_RETENTION = 2 DAYS,AUTO_CLEANUP = ON)
GO
```

### 2、查看是否生效

```
SELECT DB_NAME(database_id) DataBaseName,is_auto_cleanup_on,retention_period,retention_period_units_desc FROM sys.change_tracking_databases
GO
```

# 第二步：

### 1、对表级别操作

### --对表启用更改跟踪

```
ALTER TABLE [dbo].[Department] ENABLE CHANGE_TRACKING WITH (TRACK_COLUMNS_UPDATED = ON)
GO
```

### --批量执行开启dbo下表级别change tracking （如果已开启会自动跳过）

```
begin 
  declare @name varchar(500) 
  declare @sql varchar(500) 
  declare ct_cur cursor for 
  select 
    '['+schema_name(a.schema_id)+']'+'.'+'['+a.name+']' from sys.objects a join sys.objects b on a.object_id = b.parent_object_id 
    and a.type = 'U' and b.type = 'PK' 
    and schema_name(a.schema_id) = 'dbo' 
    and b.name is not null 
  open ct_cur
  fetch next from ct_cur into @name 
  while @@FETCH_STATUS = 0 
    begin 
      set @sql = 'alter table ' + @name + ' enable change_tracking with (track_columns_updated = on)' 
    print @sql 
    begin try 
      begin tran 
      execute (@sql) 
      commit tran 
    end try 
    begin catch 
      rollback 
      select '已对表'+ @name +' 启用了更改跟踪' 
    end catch 
    set @sql = ''; 
    fetch next from ct_cur into @name 
   end 
  close ct_cur 
  deallocate ct_cur 
end 
go
```

### 2、查看是否生效

```
SELECT OBJECT_NAME(object_id) TableName,is_track_columns_updated_on FROM sys.change_tracking_tables
GO
```

# 第三步：（可针对个别表进行开启或者全部开启，可选择执行）

### 针对表级别:

```
grant select on table_name to user
go
grant view change tracking on object::table_name to user
go
```

### 或对schema级别:

```
grant select on schema::dbo to user
go
grant view change tracking on schema::dbo to user
go
```