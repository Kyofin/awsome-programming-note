# kafka starter

## 常用命令

### **1.kafka server启动:**  

```
./kafka-server-start.sh ../config/server.properties &
```



### **2.创建topic:** 

```
 ./kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic tes
```



### **3.查看kafka的topic：**

```
./kafka-topics.sh --zookeeper localhost:2181 --list
```



### **4.查看kafka某个topic下partition信息:** 

```
./kafka-topics.sh --describe --zookeeper localhost:2181 --topic test-topic
```



### **5.查看kafka的指定topic的描述:**  

```
./kafka-topics.sh --zookeeper localhost:2181 --describe --topic yq20171220
```



### **6.控制台向kafka生产数据:**  

```
./kafka-console-producer.sh --broker-list localhost:9092 --topic jason_20180519
```



### **7.控制台消费kafka的数据（也可以读全部数据）:**  

```
./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic dbhistory.inventory --from-beginning
```

加入参数`--from-beginning`会把数据全部都读出来，不加只读启动脚本后采集到的。



### 8.查看topic下某分区偏移量的最小值: 

```
./kafka-run-class.sh kafka.tools.GetOffsetShell --topic test-topic  --time -1 --broker-list localhost:9092 --partitions 0
```



### 9.增加topic的partition:

```
./kafka-topics.sh --alter --topic jason_20180519 --zookeeper 10.200.10.24:2181,10.200.10.26:2181,10.200.10.29:2181 --partitions 5  
```



### 10.删除topic，慎用，只会删除zookeeper中的元数据，消息文件须手动删除:  

```
./kafka-run-class.sh kafka.admin.DeleteTopicCommand --zookeeper localhost:2181 --topic yq20171220
```



### 11.彻底删除topic:

```
 rmr /brokers/topics/【topic name】即可
```

