# mysql导出导入所有数据库



## 导出所有数据库


mysqldump -uroot -p123456 --all-databases > /home/aa.sql

 

## 导入所有数据库

 

mysql -uroot -p123456 < /home/aa.sql