# pg导出导入

## 导出

```
pg_dump -U postgres  zhengzhou | gzip > filename.gz
```

第一个参数是用户名 

第二个参数是数据库名

## 导入

```
psql -U postgres zhengzhou < zzszxyy.dump
```

第一个参数是用户名 

第二个参数是数据库名