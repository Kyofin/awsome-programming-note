# HBase Starter

[TOC]

## 下载

```
wget https://archive.apache.org/dist/hbase/1.2.4/hbase-1.2.4-bin.tar.gz
```



## 伪分布式运行修改hbase相关文件

### conf/hbase-env.sh

 

- 对于HBase 0.98.5和更高版本，需要在启动HBase之前设置JAVA_HOME环境变量  

```
export JAVA_HOME=/usr/java/jdk1.7.0_80
```

假定集群的每个节点使用相同的配置。如果不是这样，您需要为每个节点单独设置JAVA_HOME。

**这个环境变量设置不是必须的**

 

- 告诉HBase是否应该管理自己的Zookeeper实例，设置为true让它自己管理

```
export HBASE_MANAGES_ZK=true
```

 

- 设置PID文件存储路径，缺省是在/tmp 下

```
export HBASE_PID_DIR=/data/hbase/tmp/pids
```

 

- 使用JDK8 ，需要在HBase的配置文件中hbase-env.sh，注释掉两行

```
# Configure PermSize. Only needed in JDK7. You can safely remove it for JDK8+
export HBASE_MASTER_OPTS="$HBASE_MASTER_OPTS -XX:PermSize=128m -XX:MaxPermSize=128m"
export HBASE_REGIONSERVER_OPTS="$HBASE_REGIONSERVER_OPTS -XX:PermSize=128m -XX:MaxPermSize=128m"
```

 

### conf/hbase-site.xml 

这是HBase的主配置文件

缺省配置：<http://hbase.apache.org/book.html#hbase_default_configurations>

 

- 配置本地临时目录，缺省值是'/tmp'

```
<property>
  <name>hbase.tmp.dir</name>
  <value>/opt/data/hbase/tmp</value>
</property>
```

- 指定zookeeper数据目录

  ```
  <property>
    <name>hbase.zookeeper.property.dataDir</name>
    <value>/opt/data/zookeeper</value>
  </property>
  ```

  

- 指定分布式运行

```
<property>
  <name>hbase.cluster.distributed</name>
  <value>true</value>
</property>
```

 

- 配置hbase目录，让HDFS生成该目录给hbase使用

```
<property>
  <name>hbase.rootdir</name>
  <value>hdfs://localhost:8020/hbase</value>
</property>
```

 

> 注意：

> $HBASE_HOME/conf/hbase-site.xml 的 hbase.rootdir的主机和端口号
>
> $HADOOP_HOME/conf/core-site.xml 的 fs.default.name的主机和端口号一致



## 配置环境变量方便使用

```
export HBASE_HOME=/opt/hadoop/hbase-1.2.4
export PATH=$HBASE_HOME/bin:$PATH
```



## 访问地址

HBase的Web界面 <http://134.175.29.194:16010/>





------

## shell操作hbase

### 准备导入数据脚本

hbase_test.sh

```
for ((i=1; i<=100; i ++))
do
        echo  put \'region_test\', \'row$i\', \'cf:a\', \'value1\' >> test.txt
        echo  put \'region_test\', \'row$i\', \'cf:b\', \'value2\' >> test.txt
        echo  put \'region_test\', \'row$i\', \'cf:c\', \'value3\' >> test.txt
        echo  put \'region_test\', \'row$i\', \'cf:d\', \'value4\' >> test.txt
done
```

### 执行脚本sh hbase_test.txt

```
sh hbase_test.txt
```

### hbase shell导入

```
hbase shell test.txt
```

### 计数

```
count 'region_test'
```



## 协处理器开发

这里将写入hbase的数据都同步到redis。

依赖：

```xml
 <dependency>
     <groupId>org.apache.hbase</groupId>
     <artifactId>hbase-server</artifactId>
     <version>1.2.0</version>
</dependency>
<dependency>
      <groupId>redis.clients</groupId>
      <artifactId>jedis</artifactId>
      <version>3.2.0</version>
</dependency>
<dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>fastjson</artifactId>
      <version>1.2.61</version>
 </dependency>
```



代码：

```java
package com.gavin.observer;

import com.alibaba.fastjson.JSONObject;
import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.apache.hadoop.hbase.Cell;
import org.apache.hadoop.hbase.CellUtil;
import org.apache.hadoop.hbase.client.Delete;
import org.apache.hadoop.hbase.client.Durability;
import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.coprocessor.BaseRegionObserver;
import org.apache.hadoop.hbase.coprocessor.ObserverContext;
import org.apache.hadoop.hbase.coprocessor.RegionCoprocessorEnvironment;
import org.apache.hadoop.hbase.regionserver.wal.WALEdit;
import org.apache.hadoop.hbase.util.Bytes;
import redis.clients.jedis.Jedis;

import java.io.IOException;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.NavigableMap;

/**
 * @program: observer
 * @author: huzekang
 * @create: 2020-03-24 13:56
 **/
public class RedisObserver extends BaseRegionObserver {
	private static final Log LOG = LogFactory.getLog(RedisObserver.class);

	@Override
	public void postPut(ObserverContext<RegionCoprocessorEnvironment> e, Put put, WALEdit edit, Durability durability) throws IOException {
		try {
			String indexId = new String(put.getRow());
			NavigableMap<byte[], List<Cell>> familyMap = put.getFamilyCellMap();
			Map<String, Object> json = new HashMap<String, Object>(5);
			for (Map.Entry<byte[], List<Cell>> entry : familyMap.entrySet()) {
				for (Cell cell : entry.getValue()) {
					String key = Bytes.toString(CellUtil.cloneQualifier(cell));
					String value = Bytes.toString(CellUtil.cloneValue(cell));
					json.put(key, value);
				}
			}
			Jedis jedis = new Jedis("localhost");
			jedis.set(indexId, JSONObject.toJSONString(json));
			LOG.info("observer -- add new doc: " + indexId );

		} catch (Exception ex) {
			LOG.error(ex);
		}
	}

	@Override
	public void postDelete(ObserverContext<RegionCoprocessorEnvironment> e, Delete delete, WALEdit edit, Durability durability) throws IOException {
		String indexId = new String(delete.getRow());
		Jedis jedis = new Jedis("localhost");
		jedis.del(indexId);
		LOG.info("observer -- delete a doc: " + indexId);	}
}

```

打包成jar包后，在hbase shell中加载协处理器。

这里需要先建好表`table1`

```
alter 'table1',  METHOD => 'table_att', 'coprocessor' => 'file:///Users/huzekang/study/HBaseObserver/target/observer-1.0-SNAPSHOT-jar-with-dependencies.jar|com.gavin.observer.RedisObserver|1001|'
```

使用hbase shell 插入一条数据后，可以在redis中看到。

```
hbase(main):007:0> put 'table1' ,'row23','cf:name',"12321xkjkdf"
```

![](http://image-picgo.test.upcdn.net/img/20200324144337.png)