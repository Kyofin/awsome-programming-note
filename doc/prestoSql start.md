# prestoSql start

## 330版本

### 源码下载

https://github.com/prestosql/presto.git

我这里用的版本是330



### idea导入项目

启动类是`io.prestosql.server.PrestoServer`

idea启动类配置如下：

![image-20200902093852034](http://image-picgo.test.upcdn.net/img/20200902093852.png)

VM option：

```
-ea
-XX:+UseG1GC
-XX:G1HeapRegionSize=32M
-XX:+UseGCOverheadLimit
-XX:+ExplicitGCInvokesConcurrent
-Xmx2G
-Dconfig=etc/config.properties
-Dlog.levels-file=etc/log.properties
-Djdk.attach.allowAttachSelf=true
-Dpresto-temporarily-allow-java8=true
```



启动前需要先使用maven编译。

```
mvn clean install -DskipTests -X
```

需要注意的是：presto server启动时，会加载用户默认maven仓库(~/.m2/repository)中的依赖作为presto的插件。而我本地的maven仓库的地址是改了的，所以需要做软连接指向之前presto编译打包到的仓库地址。

![image-20200902094338760](http://image-picgo.test.upcdn.net/img/20200902094338.png)



启动前，先修改catalog的配置，使其可以访问我本地的mysql

![image-20200902094613320](http://image-picgo.test.upcdn.net/img/20200902094613.png)



presto sever启动成功后可以看到如下。

![image-20200902094734928](http://image-picgo.test.upcdn.net/img/20200902094734.png)



presto server启动后，就可以用cli连接了。

![image-20200902094036026](http://image-picgo.test.upcdn.net/img/20200902094036.png)

测试查mysql的数据库。

![image-20200902094649140](http://image-picgo.test.upcdn.net/img/20200902094649.png)

测试查mysql数据库test里的表

![image-20200902095314929](http://image-picgo.test.upcdn.net/img/20200902095314.png)



默认加载的连接器是没有kudu的。需要先修改配置`/Users/huzekang/openSource/prestosql/presto-main/etc/config.properties` 。增加kudu的pom。

![image-20200902094911577](http://image-picgo.test.upcdn.net/img/20200902094911.png)

然后新建kudu的catalog配置`/Users/huzekang/openSource/prestosql/presto-main/etc/catalog/kudu.properties`。

```properties
connector.name=kudu

## List of Kudu master addresses, at least one is needed (comma separated)
## Supported formats: example.com, example.com:7051, 192.0.2.1, 192.0.2.1:7051,
##                    [2001:db8::1], [2001:db8::1]:7051, 2001:db8::1
kudu.client.master-addresses=cdh02

## Kudu does not support schemas, but the connector can emulate them optionally.
## By default, this feature is disabled, and all tables belong to the default schema.
## For more details see connector documentation.
#kudu.schema-emulation.enabled=false

## Prefix to use for schema emulation (only relevant if `kudu.schema-emulation.enabled=true`)
## The standard prefix is `presto::`. Empty prefix is also supported.
## For more details see connector documentation.
#kudu.schema-emulation.prefix=

###########################################
### Advanced Kudu Java client configuration
###########################################

## Default timeout used for administrative operations (e.g. createTable, deleteTable, etc.)
#kudu.client.default-admin-operation-timeout = 30s

## Default timeout used for user operations
#kudu.client.default-operation-timeout = 30s

## Default timeout to use when waiting on data from a socket
#kudu.client.default-socket-read-timeout = 10s

## Disable Kudu client's collection of statistics.
#kudu.client.disable-statistics = false

```





## 308版本

### 源码下载

```
git clone https://github.com/prestosql/presto.git
git checkout 308
git checkout -b hzk_308
```

### 编译

```
mvn  clean install -DskipTests 
```

![image-20201017000018198](http://image-picgo.test.upcdn.net/img/20201017000018.png)

### 导入idea

![image-20201017000039430](http://image-picgo.test.upcdn.net/img/20201017000039.png)

```
-ea
-XX:+UseG1GC
-XX:G1HeapRegionSize=32M
-XX:+UseGCOverheadLimit
-XX:+ExplicitGCInvokesConcurrent
-Xmx2G
-Dconfig=etc/config.properties
-Dlog.levels-file=etc/log.properties
-Djdk.attach.allowAttachSelf=true
-Dpresto-temporarily-allow-java8=true
```

![image-20201017000145535](http://image-picgo.test.upcdn.net/img/20201017000145.png)

启动成功。访问`http://127.0.0.1:8080/ui/`

![image-20201017000200525](http://image-picgo.test.upcdn.net/img/20201017000200.png)

![image-20201017000214483](http://image-picgo.test.upcdn.net/img/20201017000214.png)

### 启动客户cli

```
~/openSource/prestosql(hzk_308*) » java -jar  presto-cli/target/presto-cli-308-executable.jar 
```

可以执行help查看命令帮助

### SQL执行debug

![image-20201017001216817](http://image-picgo.test.upcdn.net/img/20201017001216.png)

该sql会触发`ScanFilterAndProjectOperator`。

![image-20201017001258680](http://image-picgo.test.upcdn.net/img/20201017001258.png)

### 接口debug

该rest接口定时会触发

![image-20201017001822772](http://image-picgo.test.upcdn.net/img/20201017001822.png)

### 使用内置tpcds数据集测试

![image-20201017004802716](http://image-picgo.test.upcdn.net/img/20201017004802.png)