# prometheus+ pushgateway +grafana监控

## 参考文档

https://github.com/yunlzheng/prometheus-book

https://www.cnblogs.com/xiaobaozi-95/p/10684524.html



## 原理

客户端使用push的方式上报监控数据到pushgateway，prometheus会定期从pushgateway拉取数据。使用它的原因主要是:

- Prometheus 采用 pull 模式，可能由于不在一个子网或者防火墙原因，导致Prometheus 无法直接拉取各个 target数据。
- 在监控业务数据的时候，需要将不同数据汇总, 由 Prometheus 统一收集。

拓扑图

![img](https://img2018.cnblogs.com/blog/1364152/201904/1364152-20190410162338318-1205085018.png)

## 安装

### prometheus

```
docker run -p 9090:9090    prom/prometheus
```



### pushgateway

```
docker run -d -p 9091:9091 prom/pushgateway
```



### grafana

```
docker run -d -p 3000:3000 grafana/grafana
```



## 配置

**prometheus.yml添加target**

```
 - job_name: 'push-metrics'
    static_configs:
    - targets: ['10.0.0.17:9091']
    honor_labels: true
# 因为prometheus配置pushgateway 的时候,也会指定job和instance,但是它只表示pushgateway实例,不能真正表达收集数据的含义。所以配置pushgateway需要添加honor_labels:true,避免收集数据本身的job和instance被覆盖。
```

 **注意:**为了防止 pushgateway 重启或意外挂掉，导致数据丢失，可以通过 -persistence.file 和 -persistence.interval 参数将数据持久化下来。



## 业务代码发送指标到pushgateway

### maven依赖

```java
  <dependency>
            <groupId> io.prometheus</groupId>
            <artifactId>simpleclient_pushgateway</artifactId>
            <version>0.3.0</version>
        </dependency>
```

### PushGatewayIntegration

```java
/**
* 将监控指标推送到PushGateway供prometheus采集
*
* @author: huzekang
* @Date: 2020/1/19
*/
public enum  PushGatewayIntegration {

    //枚举元素本身就是单例
     INSTANCE;

    public  void push(String address,Long jobId)  {
        try {
            CollectorRegistry registry = CollectorRegistry.defaultRegistry;
            PushGateway pg = new PushGateway(address);
            // 根据jobid区分
            pg.pushAdd(registry, String.format("datax_job_%s",jobId));
        } catch (Exception e) {

        }

    }

}
```



### 注册指标收集

```java
 static final Gauge TOTAL_READ_RECORDS_METRICS = Gauge.build().name(CommunicationTool.TOTAL_READ_RECORDS)
            .help(CommunicationTool.TOTAL_READ_RECORDS).register();
            
            
```

### 设置指标

```JAVA
TOTAL_READ_RECORDS_METRICS.set(communication.getLongCounter(CommunicationTool.TOTAL_READ_RECORDS));
```



