# 基于docker一键部署大数据开发环境

## 组件镜像下载

```
docker pull bde2020/hadoop-namenode:2.0.0-hadoop2.7.4-java8
docker pull bde2020/hadoop-resourcemanager:2.0.0-hadoop2.7.4-java8
docker pull bde2020/hadoop-historyserver:2.0.0-hadoop2.7.4-java8
docker pull bde2020/hadoop-nodemanager:2.0.0-hadoop2.7.4-java8
docker pull bde2020/hadoop-datanode:2.0.0-hadoop2.7.4-java8
docker pull bde2020/hive:2.3.2-postgresql-metastore
docker pull bde2020/hive-metastore-postgresql:2.3.0
docker pull zookeeper:3.5.8
docker pull bde2020/hbase-master:1.0.0-hbase1.2.6
docker pull bde2020/hbase-regionserver:1.0.0-hbase1.2.6


```

