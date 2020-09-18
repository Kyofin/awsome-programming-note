# filebeat start

## 下载解压

```
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.8.0-darwin-x86_64.tar.gz
tar xzvf filebeat-7.8.0-darwin-x86_64.tar.gz
```



## 配置filebeat.yml

找到如下选项，参考配置即可

```yml
- type: log

  # Change to true to enable this input configuration.
  enabled: true

  # Paths that should be crawled and fetched. Glob based paths.
  paths:
    - /Users/huzekang/tmp/logs/*.log



output.elasticsearch:
  # Array of hosts to connect to.
  hosts: ["192.168.1.130:9200"]
```

这里是监听目录下的log文件，然后写入es。



## 启动

```
~/opt/filebeat-7.8.0-darwin-x86_64 » ./filebeat -e -c filebeat.yml
```

![image-20200723102810544](http://image-picgo.test.upcdn.net/img/20200723102810.png)