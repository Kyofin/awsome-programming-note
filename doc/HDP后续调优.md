# HDP后续调优

## spark

### spark无法查询hive命令行建的表

通过Ambari2.7安装好HDP3.1后，发现在spark-sql中无法读到hive命令行创建的数据库和表。

后来查了网上资料，发现hive 3.0之后默认开启ACID功能，而且新建的表默认是ACID表。而spark目前还不支持hive的ACID功能，因此无法读取ACID表的数据。


1. 在ambari页面中对hive服务修改配置

   ```
   hive.strict.managed.tables=false
   hive.create.as.insert.only=false
   metastore.create.as.acid=false
   ```

   重启hive。

2. 在ambari页面中对spark2服务修改配置

   `metastore.catalog.default`配置为：Hive

   这个选项默认为Spark， 即读取SparkSQL自己的metastore_db，修改完后重启spark服务。

   Spark Shell会去读取Hive的metastore，这样就可以实现以Spark Shell方式访问Hive SQL方式创建的databases/tables。

   

3. 在hive中建表时使用下面格式

   ```
   create table xxx.***(....) stored as orc TBLPROPERTIES('transactional'='false')
   ```

   

