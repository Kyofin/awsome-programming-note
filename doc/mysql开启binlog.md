# mysql开启binlog

## 参考文档

https://www.cnblogs.com/martinzhang/p/3454358.html

[腾讯工程师带你深入解析 MySQL binlog](https://zhuanlan.zhihu.com/p/33504555)



## 主要用途

- 监听数据变化

- 复制

- 恢复数据

  

## 用docker安装mysql

```
docker run --restart=always --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456  -d mysql:5.7

```

## 进入容器中修改mysql配置

```
~/openSource/phoenix-master » docker exec -it mysql bash                                                                                                
```

![](https://raw.githubusercontent.com/huzekang/picbed/master/20191012143758.png)

### 重启mysql后使用Navicat连接

```sql
-- 查看是否开启binlog
show variables like '%log_bin%'
-- 查看日志文件名
show master logs;
-- 查看监听到的日志
show binlog events ; 



--  比如：创建一个学生表并插入、修改了数据等等：
        CREATE TABLE IF NOT EXISTS `tt` (
          `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
          `name` varchar(16) NOT NULL,
          `sex` enum('m','w') NOT NULL DEFAULT 'm',
          `age` tinyint(3) unsigned NOT NULL,
          `classid` char(6) DEFAULT NULL,
          PRIMARY KEY (`id`)
         ) ENGINE=InnoDB DEFAULT CHARSET=utf8;

--       导入实验数据
insert into test.tt(`name`,`sex`,`age`,`classid`) values('yiyi','w',20,'cls1'),('xiaoer','m',22,'cls3'),('zhangsan','w',21,'cls5'),('lisi','m',20,'cls4'),('wangwu','w',26,'cls6');
-- 修改数据
update test.tt set name='李四' where id=4

```

![](https://raw.githubusercontent.com/huzekang/picbed/master/20191012144007.png)

可以看到我创建数据库和表都记录到binlog中了。但是新增和更新数据的日志却没有显示。可以通过`mysqlbinlog`命令来查看完整的日志，后面的参数可以解码编码后的sql。

```SHELL
root@f76dd9571f32:/var/lib/mysql# mysqlbinlog mysql-bin.000001 --base64-output=decode-rows -v

```

![](https://raw.githubusercontent.com/huzekang/picbed/master/20191012150021.png)