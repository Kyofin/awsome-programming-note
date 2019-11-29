# confluent使用教程

## 参考文档

https://docs.confluent.io/current/quickstart/ce-quickstart.html#step-6-stop-cp



## 安装说明	

### 下载压缩包

https://packages.confluent.io/archive/5.3/confluent-5.3.1-2.12.zip

解压完成如下图

![](https://i.loli.net/2019/11/22/1PYgTwGcDXN5ma9.png)

进入解压后的文件夹。创建一个目录叫`confluentCli`。

### 安装confluent Cli工具

然后执行命令安装confluent Cli工具。它会设置一些环境变量，让系统知道启动的参数。

```SHELL
~/Downloads/confluent-5.3.1 » curl -L https://cnfl.io/cli | sh -s -- -b /Users/huzekang/Downloads/confluent-5.3.1/confluentCli/bin
```

等待几分钟，安装完成后可以看到。

![](https://i.loli.net/2019/11/22/2za1knwf5mYSRuE.png)



### 设置环境变量

![](https://i.loli.net/2019/11/22/Ciwgz65v4Xth2nJ.png)

### 启动confluent

```
~/Downloads/confluent-5.3.1 » ./confluentCli/bin/confluent local start
```

看到控制台输出，等待一两分钟后打开http://localhost:9021/clusters即可。

```
Starting zookeeper
zookeeper is [UP]
Starting kafka
kafka is [UP]
Starting schema-registry
schema-registry is [UP]
Starting kafka-rest
kafka-rest is [UP]
Starting connect
connect is [UP]
Starting ksql-server
ksql-server is [UP]
Starting control-center
control-center is [UP]

```

可以看到页面打开如下

![](https://i.loli.net/2019/11/22/1kh9SePm4dFtXRA.png)

### 关闭confluent

Stop Confluent Platform using the [Confluent CLI](https://docs.confluent.io/current/cli/index.html#cli) [confluent local stop](https://docs.confluent.io/current/cli/command-reference/confluent-local/confluent_local_stop.html#confluent-local-stop) command.

```
<path-to-confluent-cli>/bin/confluent local stop
```

![](https://i.loli.net/2019/11/22/Ft7lyCTxdLkz8qO.png)

### 销毁confluent实例数据

Destroy the data in the Confluent Platform instance with the [confluent local destroy](https://docs.confluent.io/current/cli/command-reference/confluent-local/confluent_local_destroy.html#confluent-local-destroy) command.

```
<path-to-confluent-cli>/bin/confluent local destroy
```



## 使用Debezium监听mysql CDC

### 说明

参考文档https://debezium.io/documentation/reference/0.10/tutorial.html

Debezium是一个分布式平台，可将您现有的数据库转换为事件流，因此应用程序可以查看并立即响应数据库中的每个行级更改。Debezium建立在[Apache Kafka](http://kafka.apache.org/)之上，并提供[Kafka Connect](http://kafka.apache.org/documentation.html#connect)兼容连接器，用于监视特定的数据库管理系统。Debezium在Kafka日志中记录数据更改的历史记录，您的应用程序将在此位置使用它们。这使您的应用程序可以轻松，正确，完整地使用所有事件。即使您的应用程序停止（或崩溃），在重新启动时，它也会开始使用中断处的事件，因此不会丢失任何内容。

Debezium 0.10.0.Final支持使用其[MySQL连接器](https://debezium.io/documentation/reference/0.10/connectors/mysql.html)监视MySQL数据库服务器。

### 安装

```
~/opt/confluent-5.3.1 » confluent-hub install debezium/debezium-connector-mysql:0.10.0
```

其中有一些选项，选择y即可

![](https://i.loli.net/2019/11/22/tA7uQpiYRaeZT5M.png)

安装完成可以看到目录下多了组件

![](https://i.loli.net/2019/11/22/XYcTMUfjx1Or6Ke.png)

重启confluent后即可使用该connector



### 使用docker启动一个开启binlog的mysql

```
~/opt/confluent-5.3.1 » docker run -it --rm --name mysql_binlog -p 3307:3306 -e MYSQL_ROOT_PASSWORD=debezium -e MYSQL_USER=mysqluser -e MYSQL_PASSWORD=mysqlpw debezium/example-mysql:0.10
```





### 配置使用mysql connector

可以通过web页面添加

![](https://i.loli.net/2019/11/22/qb1H6C5sAdQZziw.png)

也可以用curl添加。

```shell
curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" localhost:8083/connectors/ -d '{ "name": "inventory-connector", "config": { "connector.class": "io.debezium.connector.mysql.MySqlConnector", "tasks.max": "1", "database.hostname": "localhost", "database.port": "3307", "database.user": "debezium", "database.password": "dbz", "database.server.id": "184054", "database.server.name": "dbserver1", "database.whitelist": "inventory", "database.history.kafka.bootstrap.servers": "localhost:9092", "database.history.kafka.topic": "dbhistory.inventory" } }'
```

其中`"database.server.id"`和`database.server.name`都是可以随便写的。

`database.whitelist`则是要监听的数据库名。

`database.history.kafka.bootstrap.servers`是kafka server的地址。这里用的是confluent自带的，所以写本地地址即可。

`database.history.kafka.topic`是根据数据库对应的topic名，后续connector会把表名都读出来建成一个个topic。

**用curl命令添加完后，可以看到mysql控制台多了监听信息。**

![](https://i.loli.net/2019/11/22/EwD95R7vYth2Qkr.png)

此时web页面也多了一个运行中的connector。

![](https://i.loli.net/2019/11/22/fQgBHKyMSiAJ3OL.png)

打开topic页面，也能看到connector帮我们把所有表都建成了一个个topic。

![](https://i.loli.net/2019/11/22/qBDzwPvZnftE83X.png)

### 测试使用

1. 打开address topic页面。

![](https://i.loli.net/2019/11/22/2xmYc6iHbI1OL8C.png)

2. 使用navicat网address表中添加一条数据。

   ![](https://i.loli.net/2019/11/22/XLzr4oHmnsYcV2R.png)

3. 可以看到web 页面中多了一条message。

   ![](https://i.loli.net/2019/11/22/MZog8pieESXBsbc.png)