

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
docker run -d -p 9090:9090 --name prometheus -v $PWD/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
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

```yml
 # Prometheus全局配置项
global:
  scrape_interval:     15s # 设定抓取数据的周期，默认为1min
  evaluation_interval: 15s # 设定更新rules文件的周期，默认为1min
  scrape_timeout: 15s # 设定抓取数据的超时时间，默认为10s


# scape配置
scrape_configs:

- job_name: 'pushgateway'
  static_configs:
  - targets: ['192.168.5.6:9091']

- job_name: 'node_exporter'
  static_configs:
  - targets: ['192.168.5.6:9100']

- job_name: 'docker'
  static_configs:
  - targets: ['192.168.5.6:8080']

- job_name: 'mysql'
  static_configs:
  - targets: ['192.168.5.6:9104']
 
- job_name: 'push-metrics'
  static_configs:
  - targets: ['10.0.0.17:9091']
    honor_labels: true
# 因为prometheus配置pushgateway 的时候,也会指定job和instance,但是它只表示pushgateway实例,不能真正表达收集数据的含义。所以配置pushgateway需要添加honor_labels:true,避免收集数据本身的job和instance被覆盖。
```

 **注意:**为了防止 pushgateway 重启或意外挂掉，导致数据丢失，可以通过` -persistence.file` 和` -persistence.interval `参数将数据持久化下来。



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



## 导入社区看板到grafana

地址：https://grafana.com/grafana/dashboards

![](http://image-picgo.test.upcdn.net/img/20200211111146.png)

将想要导入的看板id输入。

![](http://image-picgo.test.upcdn.net/img/20200211111048.png)



## 常用监控组件

#### 监控主机性能 node exporter

文档：https://prometheus.io/docs/guides/node-exporter/

下载地址：https://github.com/prometheus/node_exporter/releases/download/v0.18.1/node_exporter-0.18.1.darwin-amd64.tar.gz

解压后运行：

```shell
~/opt/prometheus/node_exporter-0.18.1.darwin-amd64(master*) » ./node_exporter
```

打开浏览器访问9100，可看到暴露出来的指标。

![](http://image-picgo.test.upcdn.net/img/20200211112447.png)

导入社区对应比较好的看板：https://grafana.com/grafana/dashboards/9276。

![](http://image-picgo.test.upcdn.net/img/20200211112817.png)

成功后可看到。

![](http://image-picgo.test.upcdn.net/img/20200211112834.png)



#### 监控docker cAdvisor

文档：https://github.com/google/cadvisor

启动命令：

```shell
docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:ro \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --volume=/dev/disk/:/dev/disk:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  google/cadvisor
```

导入社区对应比较好的看板：https://grafana.com/grafana/dashboards/193。

![](http://image-picgo.test.upcdn.net/img/20200211130307.png)





#### 监控mysql mysqld_exporter

文档：https://github.com/prometheus/mysqld_exporter

启动命令：

```
docker run -d \
  -p 9104:9104 \
  -e DATA_SOURCE_NAME="root:eWJmP7yvpccHCtmVb61Gxl2XLzIrRgmT@(192.168.5.6:3306)/" \
  prom/mysqld-exporter
```

导入社区对应比较好的看板：https://grafana.com/grafana/dashboards/7362。

![](http://image-picgo.test.upcdn.net/img/20200211131332.png)



# 告警设置

### 使用grafana实现钉钉告警

钉钉群组内创建机器人。

![](http://image-picgo.test.upcdn.net/img/20200211142454.png)

![](http://image-picgo.test.upcdn.net/img/20200211142458.png)



grafana中配置钉钉告警通知。

![](http://image-picgo.test.upcdn.net/img/20200211142605.png)



对某一个图表编辑添加告警规则。

![](http://image-picgo.test.upcdn.net/img/20200211142353.png)



触发告警规则后，可以收到钉钉的通知。

![](http://image-picgo.test.upcdn.net/img/20200211142642.png)