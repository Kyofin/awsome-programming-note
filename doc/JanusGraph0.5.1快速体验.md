

# JanusGraph0.5.1快速体验

## 下载

https://github.com/JanusGraph/janusgraph/releases/download/v0.5.1/janusgraph-0.5.1.zip



## 安装

只需要解压，有Java环境就可以使用。



## 修改配置

`conf/gremlin-server`

改graph指向用到的的propeties

![image-20211112152116929](http://image-picgo.test.upcdn.net/img/20211112152117.png)

## 启动server

bin/gremlin-server.sh



## 启动命令行工具

bin/gremlin.sh

```
:remote connect tinkerpop.server conf/remote.yaml

:remote console

graph

g

g.V().count()

g.addV('person').property('name','p1')

g.addV('person').property('name','p2')

g.V().count()

:remote close

:quit
```





