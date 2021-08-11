# Hive之——权限管理(授权)

HIVE授权管理，类似于操作系统权限可以授予给不同的主题，如**用户(USER)，组(GROUP)，角色(ROLES)**。Hive还是支持相当多的权限管理功能，满足一般数据仓库的使用，同时HIVE能支持自定义权限。

HIVE授权并不是完全安全，在其目前的形式来看，授权方案的目的是主要是为了防止用户不小心好做了不合法的操作，但不承诺防止用户恶意破坏。

## 一、HIVE新建文件权限

  Hive由一个默认的设置来配置新建文件的默认权限。

```html
<property>  
  <name>hive.files.umask.value</name>  
  <value>0002</value>  
  <description>The dfs.umask value for the hive created folders</description>  
</property> 
```


创建文件授权掩码为0002，即664权限，具体要看hadoop与hive用户配置。

##  二、HIVE授权存储检查

   当hive.metastore.authorization.storage.checks属性被设置成true时，Hive将会阻止没有权限的用户进行表删除操作。不过这个配置的默认值是false，应该设置成true。

```html
<property>  
  <name>hive.metastore.authorization.storage.checks</name>  
  <value>true</value>  
  <description>Should the metastore do authorization checks against  
  the underlying storage for operations like drop-partition (disallow  
  the drop-partition if the user in question doesn't have permissions  
  to delete the corresponding directory on the storage).
  </description>  
</property>
```


 同时，Hive会尽可能地将hive.metastore.execute.setugi设置成true。在不安全的模式,将这个属性设置为true将导致metastore执行DFS操作定义用户和组权限。

##  三、HIVE身份验证

1.开启Hive的身份认证功能，默认是false

```html
<property>  
  <name>hive.security.authorization.enabled</name>   
  <value>true</value>  
  <description>Enable or disable the hive client authorization</description>  
</property>  
```


 2.表创建者用于的权限配置项

```html
<property>  
  <name>hive.security.authorization.createtable.owner.grants</name>  
  <value>ALL</value>  
  <description>The privileges automatically granted to the owner whenever  
  a table gets created.An example like "select,drop" will grant select  
  and drop privilege to the owner of the table</description>  
</property>  
```


 这个配置默认是NULL，建议将其设置成ALL，让用户能够访问自己创建的表。

##  四、案例说明

### 注意：

```
hive>  GRANT CREATE ON DATABASE default TO USER huzekang;
FAILED: SemanticException The current builtin authorization in Hive is incomplete and disabled.
```

使用grant语法时，如出现FAILED: SemanticException The current builtin authorization in Hive is incomplete and disabled.这个异常。

**解决方案:**

配置

```
set hive.security.authorization.task.factory = org.apache.hadoop.hive.ql.parse.authorization.HiveAuthorizationTaskFactoryImpl;
```

------

在命令行环境开启用户认证

```sql
hive> set hive.security.authorization.enabled=true;    

hive> CREATE TABLE auth_test (key INT, value STRING);  
Authorization failed:No privilege 'Create' found for outputs { database:default}.    
Use show grant to get more details. 
```


提示建表需要权限了。

权限可以授予给不同的主题，如用户(USER)，组(GROUP)，角色(ROLES)

### 1、授权给用户

现在通过授权方式，将权限授予给当前用户：

```sql
hive> set system:user.name;  
system:user.name=huzekang  


hive> GRANT CREATE ON DATABASE default TO USER huzekang;  

hive> CREATE TABLE auth_test (key INT, value STRING);  

通过SHOW GRANT命令确认我们拥有的权限:

hive> SHOW GRANT USER huzekang ON DATABASE default;
OK
database	table	partition	column	principal_name	principal_type	privilege	grant_option	grant_time	grantor
default				huzekang	USER	CREATE	false	1628503664000	huzekang
default				huzekang	USER	DROP	false	1628503710000	huzekang
Time taken: 0.131 seconds, Fetched: 2 row(s)
```

### 2、授权给组

当Hive里面用于N多用户和N多张表的时候，管理员给每个用户授权每张表会让他崩溃的。
所以，这个时候就可以进行组(GROUP)授权。
Hive里的用户组的定义等价于POSIX里面的用户组。

```sql
hive> CREATE TABLE auth_test_group(a int,b int);  

hive> SELECT * FROM auth_test_group; 
Authorization failed:No privilege 'Select' found for inputs { database:default, table:auth_test_group, columnName:a}. Use SHOW GRANT to get more details.

hive> GRANT SELECT on table auth_test_group to group hadoop;  

hive> SELECT * FROM auth_test_group;  
OK  
Time taken: 0.119 seconds 
```

### 3、授权给角色

当给用户组授权变得不够灵活的时候，角色(ROLES)就派上用途了。
用户可以被放在某个角色之中，然后角色可以被授权。
角色不同于用户组，是由Hadoop控制的，它是由Hive内部进行管理的。

```sql
hive> CREATE TABLE auth_test_role (a int , b int);  

hive> SELECT * FROM auth_test_role;  
Authorization failed:No privilege 'Select' found for inputs  
{ database:default, table:auth_test_role, columnName:a}.  
Use show grant to get more details.  


hive> CREATE ROLE users_who_can_select_auth_test_role;  

hive> GRANT ROLE users_who_can_select_auth_test_role TO USER huzekang;  

hive> GRANT SELECT ON TABLE auth_test_role  TO ROLE users_who_can_select_auth_test_role;  

hive> SELECT * FROM auth_test_role;  

OK  
Time taken: 0.103 seconds 
```

##  五、分区表级别的授权

默认情况下，分区表的授权将会跟随表的授权，也可以给每一个分区建立一个授权机制，只需要设置表的属性PARTITION_LEVEL_PRIVILEGE设置成TRUE:

```sql
hive> ALTER TABLE auth_part  
> SET TBLPROPERTIES ("PARTITION_LEVEL_PRIVILEGE"="TRUE");  

Authorization failed:No privilege 'Alter' found for inputs  
{database:default, table:auth_part}.  
Use show grant to get more details. 
```

##  六、自动授权

属性`hive.security.authorization.createtable.owner.grants`决定了建表者对表拥有的权限。

一般情况下，有select和drop

```html
<property>  
  <name>hive.security.authorization.createtable.owner.grants</name>  
  <value>select,drop</value>  
</property> 
```


类似的，特定的用户可以被在表创建的时候自动授予其权限。

```html
<property>  
  <name>hive.security.authorization.createtable.user.grants</name>  
  <value>irwin,hadoop:select;tom:create</value>  
</property>   
```


当表建立的时候，管理员irwin和用户hadoop授予读所有表的权限。而tom只能创建表
同样的配置也可以作用于组授权和角色授权

```
hive.security.authorization.createtable.group.grants  
hive.security.authorization.createtable.role.grants 
```

##  七、删除授权

- 回收用户hadoop的create授权  

```sql
revoke create on database default from user hadoop;  
 
```

- 回收组hadoop的select授权  

```sql
revoke select on database default from group hadoop;  
```





## 八、Hive权限列表

| ALL           | 所有权限                                                    |
| ------------- | ----------------------------------------------------------- |
| ALTER         | 允许修改元数据（modify metadata data of object）—表信息数据 |
| UPDATE        | 允许修改物理数据（modify physical data of object）—实际数据 |
| CREATE        | 允许进行Create操作                                          |
| DROP          | 允许进行DROP操作                                            |
| INDEX         | 允许建索引（目前还没有实现）                                |
| LOCK          | 当出现并发的使用允许用户进行LOCK和UNLOCK操作                |
| SELECT        | 允许用户进行SELECT操作                                      |
| SHOW_DATABASE | 允许用户查看可用的数据库                                    |