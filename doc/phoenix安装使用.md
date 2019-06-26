[TOC]

>安装参考：[https://www.cnblogs.com/frankdeng/p/9536450.html](https://www.cnblogs.com/frankdeng/p/9536450.html)
>命令使用参考：[https://www.cnblogs.com/codeOfLife/p/7399188.html](https://www.cnblogs.com/codeOfLife/p/7399188.html)
## 介绍

sql on hbase

## 下载压缩包
```shell
wget http://mirror.bit.edu.cn/apache/phoenix/apache-phoenix-4.14.0-cdh5.14.2/bin/apache-phoenix-4.14.0-cdh5.14.2-bin.tar.gz
```


## 拷贝phoenix server jar包到hbase/lib包
进入到phoenix的安装目录把phoenix-4.12.0-HBase-1.2-server.jar 拷贝到集群中每个节点( 主节点也要拷贝 )的 hbase 的 lib 目录下
```
cp phoenix-4.14.1-HBase-1.2-server.jar /opt/hadoop/hbase-1.2.4/lib
```


## 重启Hbase,hdfs
```shell
$ ../hbase-1.2.4/bin/stop-hbase.sh
$ ../hadoop-2.7.3/sbin/stop-dfs.sh
```

## 启动phoenix 客户端
 启动命令：phoenix-4.14.0-HBase-1.2/bin/sqlline.py  
```
./sqlline.py localhost
```


## 验证phoenix

### 1）查看所有表
输入 !tables ，查看都有哪些表。以下显示均为Phoenix系统表，系统表中维护了用户表的元数据信息。
![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190418212032.png)



**注意**：上图中，我们使用了 sqlline.py 支持的 table 命令，该命令可以列出 HBase 中所有的表。这里需要注意**Phoenix 不支持直接显示 HBase Shell 中创建的表格**。

原因很简单，当在 Phoenix 创建一张表时，Phoenix 是将表进行了重组装。

而对 HBase Shell 创建的表 Phoenix 并未进行加工，所以无法直接显示。

如果需要将 HBase Shell 中创建的表格关联到 Phoenix 中查看，就需要在 Phoenix 中创建一个视图（View）做关联。

### 2）退出Phoenix
输入 !exit 命令(PS：Phoenix早期版本如(2.11版本)需输入!quilt才可退出，目前高版本已改为!exit命令)





## Phoenix的使用
 从命令行执行SQL的终端接口现在与Phoenix捆绑在一起。要启动它，请从bin目录执行以下命令：
```shell
[admin@node21 phoenix-4.14.0-HBase-1.2]$ ./bin/sqlline.py localhost
```
### **1）**建表插入数据：
这里演示官方案例，安装包目录下examples包下STOCK_SYMBOL.sql 内容如下：
```sql
CREATE TABLE IF NOT EXISTS STOCK_SYMBOL (SYMBOL VARCHAR NOT NULL PRIMARY KEY, COMPANY VARCHAR);
UPSERT INTO STOCK_SYMBOL VALUES ('CRM','SalesForce.com');
SELECT * FROM STOCK_SYMBOL;
```

### **2）**导入数据：
此外，您可以使用bin / psql.py加载CSV数据或执行SQL脚本。例如：
```shell
[admin@node21 phoenix-4.14.0-HBase-1.2]$ ./bin/psql.py -t STOCK_SYMBOL localhost ./examples/STOCK_SYMBOL.csv 
```
PS：其中 -t 后面是表名， ../examples/STOCK_SYMBOL.csv是csv数据（注意数据的分隔符需要是逗号）。


### **3）**查询数据
```sql
 select * from STOCK_SYMBOL;
```

### Java调用

引入依赖

```
<dependencies>
    <dependency>
      <groupId>org.apache.phoenix</groupId>
      <artifactId>phoenix-core</artifactId>
      <version>4.13.1-HBase-1.2</version>
    </dependency>
  </dependencies>
```



编写测试代码

```java
package com.xyg.phoenix;


import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;


/**
 * Desc: create table, create index, insert data, select table.
 */
public class TestPhoenixJDBC {


    private static String driver = "org.apache.phoenix.jdbc.PhoenixDriver";


    public static void main(String[] args) throws SQLException {


        Connection conn = null;
        Statement stmt = null;
        ResultSet rs = null;
        try {
              Class.forName(driver);
            } catch (ClassNotFoundException e) {
                e.printStackTrace();
            }
        conn = DriverManager.getConnection("jdbc:phoenix:node21,node22,node23:2181");
        stmt = conn.createStatement();
        stmt.execute("drop table if exists test");
        stmt.execute("create table test (mykey integer not null primary key, mycolumn varchar)");
        stmt.execute("create index test_idx on test(mycolumn)");
        stmt.executeUpdate("upsert into test values (1,'World!')");
        stmt.executeUpdate("upsert into test values (2,'Hello')");
        stmt.executeUpdate("upsert into test values (3,'World!')");
        conn.commit();
        rs = stmt.executeQuery("select mykey from test where mycolumn='Hello'");
        while (rs.next()) {
            System.out.println(rs.getInt(1));
        }
        stmt.close();
        rs.close();
        conn.close();


    }


}
```

