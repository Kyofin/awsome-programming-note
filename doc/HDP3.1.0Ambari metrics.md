# HDP3.1.0Ambari metrics

## 原理

ambari metrics会将采集的数据用phoenix存放到hbase中。



## 参考文档

https://docs.cloudera.com/HDPDocuments/Ambari-2.7.3.0/using-ambari-core-services/content/amb_using_ambari_core_services.html



## 用curl请求collector的rest api获取监控指标

![image-20201023150821478](http://image-picgo.test.upcdn.net/img/20201023150821.png)

对应指标名可以参考grafana。

```
 curl -v "http://10.93.6.8:6188/ws/v1/timeline/metrics?metricNames=bytes_in._avg&hostnaame=&appId=HOST&instanceId=&startTime=1451630974&endTime=1603436573"
```

获取当前时间的主机的内存占用指标。

```
curl -v "http://10.93.6.8:6188/ws/v1/timeline/metrics?metricNames=mem_used._avg&hostnaame=&appId=HOST&instanceId="
```

获取当前时间namenode的总block。

```
curl -v "http://10.93.6.8:6188/ws/v1/timeline/metrics?metricNames=dfs.FSNamesystem.BlocksTotal._avg&hostnaame=&appId=namenode&instanceId="
```





## 用ambari metrics内置hbase访问数据

需要登录metrics collector部署的节点。

```
/usr/lib/ams-hbase/bin/hbase --config /etc/ams-hbase/conf shell 
```



## 用phonenix客户端查看监控数据

需要登录部署了phoenix的节点。

```
cd /usr/hdp/current/phoenix-client

bin/sqlline.py localhost:2181:/ams-hbase-unsecure

```

![image-20201022124526616](http://image-picgo.test.upcdn.net/img/20201022124526.png)

| 表名                    | 描述                                                         | 清理的时间间隔 |
| ----------------------- | ------------------------------------------------------------ | -------------- |
| METRIC_RECORD           | 用于记录每个机器上收集的每个 Metrics 属性，时间精度为 10 妙。 | 1 天           |
| METRIC_RECORD_MINUTE    | 同上，时间精度为 5 分钟。                                    | 1 周           |
| METRIC_RECORD_HOURLY    | 同上，时间精度为 1 小时                                      | 30 天          |
| METRIC_RECORD_DAILY     | 同上，时间精度为为 1 天                                      | 1 年           |
| METRIC_AGGREGATE        | 集群级别的 Metrics 属性（聚和计算每个机器的 Metrics），取样精度为 30 妙 | 1 周           |
| METRIC_AGGREGATE_MINUTE | 同上，精度为 5 分钟                                          | 30 天          |
| METRIC_AGGREGATE_HOURLY | 同上，精度为 1 小时                                          | 1 年           |
| METRIC_AGGREGATE_DAILY  | 同上，精度为 1 天                                            | 2 年           |





## 在grafana中展示采集到的监控指标

访问Ambari metrics服务的grafana的页面。

需要登录admin/admin

找到HDFS dashboard。

![image-20201022155247041](http://image-picgo.test.upcdn.net/img/20201022155247.png)

添加一行

![image-20201022155538651](http://image-picgo.test.upcdn.net/img/20201022155538.png)

添加graph

![image-20201022155632776](http://image-picgo.test.upcdn.net/img/20201022155632.png)

选择namenode，即可选择对应的metrics。





## 移除 Metric Collector 

停止metrics服务。

![image-20201022161222948](http://image-picgo.test.upcdn.net/img/20201022161223.png)

可以看到现在有两台服务器部署了collector。我们需要删除一台。

等待服务停止后。执行下面命令。

```
curl -u admin:admin -H "X-Requested-By:ambari" -i -X DELETE http://10.93.6.6:8080/api/v1/clusters/hadoop_cluster/hosts/hadoop001.master.hdp/host_components/METRICS_COLLECTOR

```

执行后，可以观察到一个collector被删除了。

![image-20201022161612607](http://image-picgo.test.upcdn.net/img/20201022161612.png)