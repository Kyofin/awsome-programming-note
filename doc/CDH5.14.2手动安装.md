[TOC]

# CDH5.14.2手动安装

## 下载

| 软件       | 版本                      | 下载地址                                                     |
| ---------- | ------------------------- | ------------------------------------------------------------ |
| jdk        | jdk-8u172-linux-x64       | [点击下载](http://download.oracle.com/otn-pub/java/jdk/8u172-b11/a58eab1ec242421181065cdc37240b08/jdk-8u172-linux-x64.tar.gz) |
| hadoop     | hadoop-2.6.0-cdh5.14.2    | [点击下载](http://archive.cloudera.com/cdh5/cdh/5/hadoop-2.6.0-cdh5.14.2.tar.gz) |
| zookeeeper | zookeeper-3.4.5-cdh5.14.2 | [点击下载](http://archive.cloudera.com/cdh5/cdh/5/zookeeper-3.4.5-cdh5.14.2.tar.gz) |
| hbase      | hbase-1.2.0-cdh5.14.2     | [点击下载](http://archive.cloudera.com/cdh5/cdh/5/hbase-1.2.0-cdh5.14.2.tar.gz) |
| hive       | hive-1.1.0-cdh5.14.2      | [点击下载](http://archive.cloudera.com/cdh5/cdh/5/hive-1.1.0-cdh5.14.2.tar.gz) |
| hue        | hue-3.9.0-cdh5.14.2       | [点击下载](http://archive.cloudera.com/cdh5/cdh/5/hue-3.9.0-cdh5.14.2.tar.gz) |
| presto     | presto-server-0.221       | [点击下载](https://repo1.maven.org/maven2/com/facebook/presto/presto-server/0.221/presto-server-0.221.tar.gz) |
| kafka      | kafka_2.12-2.1.1          | [点击下载](http://mirrors.tuna.tsinghua.edu.cn/apache/kafka/2.1.1/kafka_2.12-2.1.1.tgz) |



注： CDH5的所有软件可以在此下载：<http://archive.cloudera.com/cdh5/cdh/5/>



## 预备环境

1. jdk8

2. 关闭防火墙

3. 设置hostname，设置hosts文件

   **设置成hadoop0**

4. 设置ssh

   Now check that you can ssh to the localhost without a passphrase:

   ```
     $ ssh localhost
   ```

   If you cannot ssh to localhost without a passphrase, execute the following commands:

   ```
     $ ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
     $ cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys
   ```

   

## Hdfs

### 参考官方文档

<http://archive.cloudera.com/cdh5/cdh/5/hadoop-2.6.0-cdh5.14.2/hadoop-project-dist/hadoop-common/SingleCluster.html>

### 配置

etc/hadoop/core-site.xml:

```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:8020</value>
    </property>
   <!-- 本机默认存储路径：/tmp/hadoop-huzekang/dfs/name -->
     <!-- 防止重启电脑临时文件删除清楚所有hdfs保存的文件,该目录需手动创建 -->
      <property>
	        <name>hadoop.tmp.dir</name>
	        <value>/Users/huzekang/opt/hadoop-cdh/data/hadoop/tmp</value>
	    </property>
</configuration>
```

etc/hadoop/hdfs-site.xml:

```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```

etc/hadoop/hadoop-env.sh 

```
# The java implementation to use.
export JAVA_HOME=/usr/java/jdk1.8.0_211-amd64/
```

### 运行

The following instructions are to run a **MapReduce job locally**. If you want to execute a job on YARN, see [YARN on Single Node](http://archive.cloudera.com/cdh5/cdh/5/hadoop-2.6.0-cdh5.14.2/hadoop-project-dist/hadoop-common/SingleCluster.html#YARN_on_Single_Node).

1. Format the filesystem:

   ```
     $ bin/hdfs namenode -format
   ```

   ![](https://raw.githubusercontent.com/huzekang/picbed/master/20190618172845.png)

2. Start NameNode daemon and DataNode daemon:

   ```
     $ sbin/start-dfs.sh
   ```

   The hadoop daemon log output is written to the `$HADOOP_LOG_DIR` directory (defaults to `$HADOOP_HOME/logs`).

   ![](https://raw.githubusercontent.com/huzekang/picbed/master/20190618172745.png)

3. Browse the web interface for the NameNode; by default it is available at:

   - NameNode - `http://localhost:50070/`

     ![](https://raw.githubusercontent.com/huzekang/picbed/master/20190623160757.png)

4. Make the HDFS directories required to execute MapReduce jobs:

   ```
     $ bin/hdfs dfs -mkdir /user
     $ bin/hdfs dfs -mkdir /user/<username>
   ```

5. Copy the README.txt into the distributed filesystem:

   ```
     $ bin/hdfs dfs -put README.txt /user/<username>
   ```

6. Run some of the examples provided:

   ```
     $ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.6.0-cdh5.14.2.jar grep /user/huzekang/README.txt /user/huzekang/output 'dfs[a-z.]+'
   ```

7. Examine the output files:

   Copy the output files **from the distributed filesystem** to the **local filesystem** and examine them:

   ```
     $ bin/hdfs dfs -get /user/huzekang/output output
     $ ll output/
   ```

   or

   View the output files on the distributed filesystem:

   ```
     bin/hdfs dfs -ls /user/huzekang/output
   ```

8. When you're done, stop the daemons with:

   ```
     $ sbin/stop-dfs.sh
   ```

## YARN on Single Node

You can run a MapReduce job on YARN in a **pseudo-distributed mode** by setting a few parameters and running ResourceManager daemon and NodeManager daemon in addition.

### 配置

The following instructions assume that 1. ~ 4. steps of [the above instructions](http://archive.cloudera.com/cdh5/cdh/5/hadoop-2.6.0-cdh5.14.2/hadoop-project-dist/hadoop-common/SingleCluster.html#Execution) are already executed.

1. Configure parameters as follows:

   etc/hadoop/mapred-site.xml:

   **如果没有则复制它的template创建一个**

   ```xml
   <configuration>
       <property>
           <name>mapreduce.framework.name</name>
           <value>yarn</value>
       </property>
   </configuration>
   ```

   etc/hadoop/yarn-site.xml:

   ```xml
   <configuration>
       <property>
           <name>yarn.nodemanager.aux-services</name>
           <value>mapreduce_shuffle</value>
       </property>
   </configuration>
   ```

### 运行

1. Start ResourceManager daemon and NodeManager daemon:

   ```
     $ sbin/start-yarn.sh
   ```

2. Browse the web interface for the ResourceManager; by default it is available at:

   - ResourceManager - `http://localhost:8088/`

     ![](https://raw.githubusercontent.com/huzekang/picbed/master/20190618175325.png)

3. Run a MapReduce job.

   ![](https://raw.githubusercontent.com/huzekang/picbed/master/20190618175408.png)

4. When you're done, stop the daemons with:

   ```
     $ sbin/stop-yarn.sh
   ```



**设置了yarn启动作业之后就不能再使用local模式了，所以如果关闭yarn再启动作业就会出现下面提示。如果想用回local模式，可以把上面yarn的配置注释掉即可。**

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190618180037.png)



## Zookeeper

### 参考文档

<http://archive.cloudera.com/cdh5/cdh/5/zookeeper-3.4.5-cdh5.14.2/zookeeperStarted.html#sc_Prerequisites>



### Standalone Operation安装

Setting up a ZooKeeper server in standalone mode is straightforward. The server is contained in a single JAR file, so installation consists of creating a configuration.

Once you've downloaded a stable ZooKeeper release unpack it and cd to the root

To start ZooKeeper you need a configuration file. Here is a sample, create it in **conf/zoo.cfg**:

```
tickTime=2000
dataDir=/Users/huzekang/opt/hadoop-cdh/data/zookeeper
clientPort=2181
```

This file can be called anything, but for the sake of this discussion call it **conf/zoo.cfg**. Change the value of **dataDir** to specify an existing (empty to start with) directory. Here are the meanings for each of the fields:

- **tickTime**

  the basic time unit in milliseconds used by ZooKeeper. It is used to do heartbeats and the minimum session timeout will be twice the tickTime.

- **dataDir**

  the location to store the in-memory database snapshots and, unless specified otherwise, the transaction log of updates to the database.

- **clientPort**

  the port to listen for client connections

Now that you created the configuration file, you can **start ZooKeeper**:

```
bin/zkServer.sh start
```

启动的是一个java进程。

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190626145350.png)

ZooKeeper logs messages using log4j -- more detail available in the [Logging](http://archive.cloudera.com/cdh5/cdh/5/zookeeper-3.4.5-cdh5.14.2/zookeeperProgrammers.html#Logging) section of the Programmer's Guide. You will see log messages coming to the console (default) and/or a log file depending on the log4j configuration.

The steps outlined here run ZooKeeper in standalone mode. There is no replication, so if ZooKeeper process fails, the service will go down. This is fine for most development situations, but to run ZooKeeper in replicated mode, please see [Running Replicated ZooKeeper](http://archive.cloudera.com/cdh5/cdh/5/zookeeper-3.4.5-cdh5.14.2/zookeeperStarted.html#sc_RunningReplicatedZooKeeper).



### Managing ZooKeeper Storage

For long running production systems ZooKeeper storage must be managed externally (dataDir and logs). See the section on [maintenance](http://archive.cloudera.com/cdh5/cdh/5/zookeeper-3.4.5-cdh5.14.2/zookeeperAdmin.html#sc_maintenance) for more details.



### Connecting to ZooKeeper

```
$ bin/zkCli.sh -server 127.0.0.1:2181
```

This lets you perform simple, file-like operations.

启动的是一个java进程。

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190626145423.png)

Once you have connected, you should see something like:

```shell
Connecting to localhost:2181
log4j:WARN No appenders could be found for logger (org.apache.zookeeper.ZooKeeper).
log4j:WARN Please initialize the log4j system properly.
Welcome to ZooKeeper!
JLine support is enabled
[zkshell: 0]
        
```

From the shell, type help to get a listing of commands that can be executed from the client, as in:

```shell
[zkshell: 0] help
ZooKeeper host:port cmd args
        get path [watch]
        ls path [watch]
        set path data [version]
        delquota [-n|-b] path
        quit
        printwatches on|off
        createpath data acl
        stat path [watch]
        listquota path
        history
        setAcl path acl
        getAcl path
        sync path
        redo cmdno
        addauth scheme auth
        delete path [version]
        setquota -n|-b val path

        
```

From here, you can try a few simple commands to get a feel for this simple command line interface. First, start by issuing the list command, as in ls, yielding:

```shell
[zkshell: 8] ls /
[zookeeper]
        
```

Next, create a new znode by running create /zk_test my_data. This creates a new znode and associates the string "my_data" with the node. You should see:

```shell
[zkshell: 9] create /zk_test my_data
Created /zk_test
      
```

Issue another ls / command to see what the directory looks like:

```shell
[zkshell: 11] ls /
[zookeeper, zk_test]

        
```

Notice that the zk_test directory has now been created.

Next, verify that the data was associated with the znode by running the get command, as in:

```shell
[zkshell: 12] get /zk_test
my_data
cZxid = 5
ctime = Fri Jun 05 13:57:06 PDT 2009
mZxid = 5
mtime = Fri Jun 05 13:57:06 PDT 2009
pZxid = 5
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0
dataLength = 7
numChildren = 0
        
```

We can change the data associated with zk_test by issuing the set command, as in:

```shell
[zkshell: 14] set /zk_test junk
cZxid = 5
ctime = Fri Jun 05 13:57:06 PDT 2009
mZxid = 6
mtime = Fri Jun 05 14:01:52 PDT 2009
pZxid = 5
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0
dataLength = 4
numChildren = 0
[zkshell: 15] get /zk_test
junk
cZxid = 5
ctime = Fri Jun 05 13:57:06 PDT 2009
mZxid = 6
mtime = Fri Jun 05 14:01:52 PDT 2009
pZxid = 5
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0
dataLength = 4
numChildren = 0
      
```

(Notice we did a get after setting the data and it did, indeed, change.

Finally, let's delete the node by issuing:

```
[zkshell: 16] delete /zk_test
[zkshell: 17] ls /
[zookeeper]
[zkshell: 18]
```

如果需要递归删除则可以使用`rmr /hbase`



## Hbase

### 配置conf目录下文件

#### hbase-env.sh

配置java环境

```shell
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_202.jdk/Contents/Home
```

注释jdk7需要的配置，因为我使用的是jdk8

```shell
#export HBASE_MASTER_OPTS="$HBASE_MASTER_OPTS -XX:PermSize=128m -XX:MaxPermSize=128m -XX:ReservedCodeCacheSize=256m"
#export HBASE_REGIONSERVER_OPTS="$HBASE_REGIONSERVER_OPTS -XX:PermSize=128m -XX:MaxPermSize=128m -XX:ReservedCodeCacheSize=256m"
```

设置hbase pid文件路径，用于关闭hbase使用

```shell
export HBASE_PID_DIR=/Users/huzekang/opt/hadoop-cdh/data/hbase/tmp/pids
```

使用外部zookeeper

```shell
export HBASE_MANAGES_ZK=false
```



#### hbase-site.xml

```xml
<configuration>
    <!-- 配置本地临时目录，缺省值是'/tmp' -->
    <property>
        <name>hbase.tmp.dir</name>
        <value>/Users/huzekang/opt/hadoop-cdh/data/hbase/tmp</value>
    </property>


    <!-- 指定外部zookeeper连接 -->
    <property>
        <name>hbase.zookeeper.quorum</name>
        <value>localhost:2181</value>
    </property>


    <!-- 指定分布式运行 -->
    <property>
        <name>hbase.cluster.distributed</name>
        <value>true</value>
    </property>

    <!-- 配置hbase目录，让HDFS生成该目录给hbase使用 -->
    <property>
        <name>hbase.rootdir</name>
        <value>hdfs://localhost:8020/hbase</value>
    </property>
</configuration>
```



### 启动Hbase

```
~/opt/hadoop-cdh/hbase-1.2.0-cdh5.14.2 » bin/start-hbase.sh                                  
```

观察启动进程，其中hmaster和hregionserver就是hbase的。

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190626145232.png)

打开zookeeper客户端

```
~/opt/hadoop-cdh/zookeeper-3.4.5-cdh5.14.2 » bin/zkCli.sh -server 127.0.0.1:2181
```

观察`/`目录下多了一个hbase目录

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190626144956.png)



### 关闭hbase

```
~/opt/hadoop-cdh/hbase-1.2.0-cdh5.14.2 » bin/stop-hbase.sh
```



### 启动hbase thrift

```
~/opt/hadoop-cdh/hbase-1.2.0-cdh5.14.2 » bin/hbase-daemon.sh start thrift
```

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190627100355.png)

该进程可以方便其他语言例如python使用hbase的api。





## Hive

### 参考文档：

官方文档：<https://cwiki.apache.org/confluence/display/Hive/GettingStarted>

hiveserver搭建详解：<http://www.coderli.com/setup-hiveserver-step-details/>

### Metadata Store postgres安装

Hive有三种元数据存储的模式。

- Embedded mode
- Local mode
- Remote mode

很多文章里都有介绍，但是很少有人具体解释其区别和原理。

这里找到一篇介绍，至少对我来说，算是能看懂他所介绍的。 [Metastore Deployment Modes](http://www.cloudera.com/documentation/enterprise/5-2-x/topics/cdh_ig_hive_metastore_configure.html#topic_18_4_1)

这里采用**Remote mode**。因此我们先要部署**postgres**。



### 配置Hive

**Hive**在**conf**目录下提供了一个配置文件模版**hive-default.xml.template**供我们修改。如果没，则创建一份。

**Hive**会默认加载**hive-site.xml**配置文件。

```
cp conf/hive-default.xml.template conf/hive-site.xml
```

同样，按照了解，我们需要配置**metastore**的相关信息，如数据库连接，驱动，用户名，密码等。

#### 配置hive-site.xml配置文件

```xml
<?xml version="1.0"?>

<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>

    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:postgresql://192.168.5.148:5432/hive_metadata</value>
    </property>

    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>org.postgresql.Driver</value>
    </property>

    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>postgres</value>
    </property>

    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>eLN8QGV4g3LINDrFrsDKvCCyHapLOPCR</value>
    </property>

    <property>
        <name>hive.metastore.uris</name>
        <value>thrift://localhost:9083</value>
        <description>Thrift URI for the remote metastore. Used by metastore client to connect to remote metastore.</description>
    </property>

    <!-- 设置hdfs上保存的数据仓库路径 -->
    <property>
        <name>hive.metastore.warehouse.dir</name>
        <value>/hive/warehouse</value>
        <description>location of default database for the warehouse</description>
    </property>




</configuration>


```

这里，**metastore**服务的默认端口为**9083**。

由于我们需要连接**pg10**数据库，而**Hive**并没有提供相应的驱动，所以需要下载**Pg  JDBC**驱动，并放在**Hive**的**lib**目录下。直接进入**lib**目录执行

```
wget https://repo1.maven.org/maven2/org/postgresql/postgresql/42.1.4/postgresql-42.1.4.jar
```



#### 初始化元数据的数据库

1. 在pg中创建数据库`hive_metadata`

2. 执行hive自带的初始化脚本

   ```
   bin/schematool -dbType postgres -initSchema
   ```

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190626203349.png)



#### 修改log日志文件位置(option)

**Hive**默认将日志**存放在/tmp/${user.name}**下。为了方便维护和查看，修改日志文件位置。

```
cp conf/hive-log4j.properties.template conf/hive-log4j.properties
```

修改日志级别和输出日志文件的地址。

```
hive.log.dir=/Users/huzekang/opt/hadoop-cdh/hive-1.1.0-cdh5.14.2/logs/hive
hive.root.logger=INFO,DRFA
```



### 启动HiveServer服务

启动MetaStore Service

```
bin/hive --service metastore &
```

第一次启动会在mysql中创建表，可能速度会慢点。可以通过看日志文件观察是否报错。



启动HiveServer

```
bin/hive --service hiveserver2 &
```

查看日志一切正常。可以看到多了两个java进程。

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190623160521.png)

可以打开浏览器网址<http://localhost:10002/hiveserver2.jsp>进行访问hiveserver2。

启动成功后，hive会在hdfs上的tmp目录建立相关文件，但是使用浏览器打开hdfs的tmp目录会发现权限不足无法访问。修改tmp目录权限就可以了。

```
bin/hdfs dfs -chmod -R 755 /tmp
```

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190623163014.png)



### 使用hive客户端测试

手动创建一个table

```sql
create table helloword(id int,name string) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';
```

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190623155043.png)



插入一条数据。

```sql
insert into helloword values(11,'peter');
```

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190623161213.png)

打开hdfs可以观察到数据存放到指定的目录了。

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190623161342.png)



### 使用beeline连接

####  嵌入模式

```
bin/beeline -u jdbc:hive2://
```

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190623220356.png)

此模式可以查询，也可以插入。



####    远程模式

```
bin/beeline      

beeline> !connect jdbc:hive2://localhost:10000
```

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190626172239.png)

此模式下可以正常查询select操作，但是插入操作和创建表都会报错。

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190623220827.png)

错误信息：

```java
Error: Error while processing statement: FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. MetaException(message:Got exception: org.apache.hadoop.security.AccessControlException Permission denied: user=anonymous, access=WRITE, inode="/Users/huzekang/opt/hadoop-cdh/hive-1.1.0-cdh5.14.2/warehouse":huzekang:supergroup:drwxr-xr-x
```

此时只要对错误信息中提到的**hive在hdfs上的warehouse目录**进行change the permission即可。

```
bin/hdfs dfs -chmod -R 777 /Users/huzekang/
```



### 使用Java代码连接

#### 引入maven依赖

```
 <dependency>
           <groupId>org.apache.hive</groupId>
           <artifactId>hive-jdbc</artifactId>
           <version>0.11.0</version>
       </dependency>
```

#### 代码

```java
/**
 * 如果频繁出现下面错误，试试更换hive目录下的lib目录下的mysql驱动
 * Error while compiling statement: FAILED: SemanticException Unable to fetch table hive_table1. Could not retrieve transation read-only status server
 */

import java.sql.*;

public class HiveJdbcTest {

    private static String driverName = "org.apache.hive.jdbc.HiveDriver";
   
    public static void main(String[] args) throws SQLException {
        try {
            Class.forName(driverName);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
            System.exit(1);
        }
        // 设置hive 的jdbc连接
        Connection con = DriverManager.getConnection("jdbc:hive2://localhost:10000/default", "", "");
        Statement stmt = con.createStatement();
        String tableName = "hive_table1";
        stmt.execute("drop table if exists " + tableName);
        stmt.execute("create table " + tableName +  " (key int, value string)");
        System.out.println("Create table success!");
        // show tables
        String sql = "show tables '" + tableName + "'";
        System.out.println("=======Running: " + sql);
        ResultSet res = stmt.executeQuery(sql);
        if (res.next()) {
            System.out.println(res.getString(1));
        }
 
        // describe table
        sql = "describe " + tableName;
        System.out.println("=======Running: " + sql);
        res = stmt.executeQuery(sql);
        while (res.next()) {
            System.out.println("表字段名："+res.getString(1) + "\t" +"表字段类型："+ res.getString(2));
        }
 

        sql = "select * from " + tableName;
        res = stmt.executeQuery(sql);
        while (res.next()) {
            System.out.println(res.getInt(1) + "\t" + res.getString(2));
        }

        // 插数据到hive_table1
        sql = "insert into  " + tableName + " values (22,'xxded')";
        System.out.println("=======Running: " + sql);
        stmt.executeUpdate( sql);

        // zh
        sql = "select count(1) from " + tableName;
        System.out.println("Running: " + sql);
        res = stmt.executeQuery(sql);
        while (res.next()) {
            System.out.println(res.getString(1));
        }

    }
}
```





## Spark

### Spark standalone集群模式部署

1. 将conf目录下的`slaves.template`文件复制成`slaves`，并写入worker部署的主机![](https://i.loli.net/2019/09/30/qNhwvczkUrZO45e.png)

2. 将spark目录分发到其他主机

   ```
   scp -r /opt/spark cdh02:/opt
   ```

3. 在主节点的spark目录下启动集群

   ```
   sbin/start-all.sh
   ```

   

### Spark集群启动命令汇总

1、在主节点启动所有服务（包括slave节点，需要做免密码登录）

```
sbin/start-all.sh
```


2、单独启动主节点

```
sbin/start-master.sh
```

3、单独启动slave节点

启动所有的slaves节点

```
sbin/start-slaves.sh spark://10.130.2.220:7077
```

启动单台的slaves节点

```
sbin/start-slave.sh spark://10.130.2.220:7077
```

启动后可以打开浏览器`http://localhost:8080/`看到spark master。

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190627113457.png)

可以观察到起来了一个master和worker进程。

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190626112610.png)





## Presto

### 服务端安装

#### 1. 下载

#### 2. 配置

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190627204309.png)

1. `etc/config.properties`

   ```properties
   coordinator=true
   node-scheduler.include-coordinator=true
   http-server.http.port=8082
   query.max-memory=5GB
   query.max-memory-per-node=1GB
   query.max-total-memory-per-node=2GB
   discovery-server.enabled=true
   discovery.uri=http://localhost:8082
   ```

2. `etc/jvm.config`

   ```java
   -server
   -Xmx16G
   -XX:+UseG1GC
   -XX:G1HeapRegionSize=32M
   -XX:+UseGCOverheadLimit
   -XX:+ExplicitGCInvokesConcurrent
   -XX:+HeapDumpOnOutOfMemoryError
   -XX:+ExitOnOutOfMemoryError
   ```

3. `etc/node.properties`

   ```properties
   node.environment=production
   node.id=ffffffff-ffff-ffff-ffff-ffffffffffff
   node.data-dir=/Users/huzekang/opt/hadoop-cdh/data/presto
   ```

#### 3. 启动

daemon运行：`bin/launcher start` 
foreground运行：`bin/launcher run`

```
~/opt/hadoop-cdh/presto-server-0.221 » bin/launcher run
```



### 客户端安装

1. 下载https://repo1.maven.org/maven2/com/facebook/presto/presto-cli/0.196/presto-cli-0.196-executable.jar

2. 移动到你喜欢的目录

   ```shell
   ~/opt/hadoop-cdh/presto-server-0.221/bin » mv ~/Downloads/presto-cli-0.196-executable.jar ./presto            
   ```

3. 赋予它执行的权限

   ```
    chmod +x presto
   ```



### 配置连接器

#### mysql

参考配置<https://prestodb.github.io/docs/current/connector/mysql.html>

在server的主目录下创建配置文件`etc/catalog/mysql.properties`

```properties
connector.name=mysql
connection-url=jdbc:mysql://192.168.1.150:3306
connection-user=root
connection-password=root
```

重启完server后启动客户端

```
~/opt/hadoop-cdh/presto-server-0.221/bin » ./presto --server localhost:8082 --catalog mysql --schema yiboard  
```

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190627165459.png)



#### postgres

参考配置：<https://prestodb.github.io/docs/current/connector/postgresql.html>

在server的主目录下创建配置文件`etc/catalog/posgresql.properties`

```properties
connector.name=postgresql
connection-url=jdbc:postgresql://192.168.1.150:5432/crawl
connection-user=postgres
connection-password=123456
```

重启完server后启动客户端。

```
~/opt/hadoop-cdh/presto-server-0.221/bin » ./presto --server localhost:8082 --catalog postgresql --schema public
```



#### mongoDB

参考配置：<https://prestodb.github.io/docs/current/connector/mongodb.html>>

在server的主目录下创建配置文件`etc/catalog/mongodb.properties`

```properties
connector.name=mongodb
mongodb.seeds=192.168.1.150:27017
```

重启完server后启动客户端。

```
~/opt/hadoop-cdh/presto-server-0.221/bin » ./presto --server localhost:8082 --catalog mongodb --schema yibo
```



#### hive

参考配置：<https://prestodb.github.io/docs/current/connector/hive.html>>

在server的主目录下创建配置文件`etc/catalog/posgresql.properties`

```properties
connector.name=hive-hadoop2
hive.metastore.uri=thrift://localhost:9083
```

重启完server后启动客户端。

```
~/opt/hadoop-cdh/presto-server-0.221/bin » ./presto --server localhost:8082 --catalog hive --schema default
```



### 跨源查询

这里我使用mysql的表和pg的表进行关联查询。

```sql
SELECT a.employee_id,b.id, a.full_name,b.repo_name FROM mysql.foodmart.employee a,postgresql.public.github_repo b where a.employee_id = b.id;
```

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190628141632.png)



## Kafka

#### 下载解压

#### 配置

配置`config/server.properties`文件。

```
broker.id=1
log.dirs=/Users/huzekang/opt/hadoop-cdh/data/kafka/kafka-logs
```



#### 启动

首先打开zookeeper。

然后启动kafka server。

```
~/opt/hadoop-cdh/kafka_2.12-2.1.1 » bin/kafka-server-start.sh  config/server.properties
```

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190627230614.png)



#### 测试

1. 启动控制台producer，创建一个test2的topic，输出几个hello

   ```shell
   ~/opt/hadoop-cdh/kafka_2.12-2.1.1 »  bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test2         
   >hello1
   [2019-06-27 23:08:54,369] WARN [Producer clientId=console-producer] Error while fetching metadata with correlation id 1 : {test2=LEADER_NOT_AVAILABLE} (org.apache.kafka.clients.NetworkClient)
   >hello2
   >hello3
   >
   ```

2. 启动控制台consumer，监听test2的topic。

   ```
   bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test2 --from-beginning
   ```

   可以看到👆上面输入的都显示出来了。

   ![](https://raw.githubusercontent.com/huzekang/picbed/master/20190627231059.png)

3. **查看有哪些topic**

4. ```
   bin/kafka-topics.sh --list --zookeeper localhost:2181
   ```

   ![](https://raw.githubusercontent.com/huzekang/picbed/master/20190701213535.png)