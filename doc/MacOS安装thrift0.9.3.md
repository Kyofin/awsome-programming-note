# MacOS安装thrift0.9.3

最近在编译`spark-doris-connector`源码，由于spark在读取doris数据时，需要和BE进行通信，而通信的协议是用thrift写的，因为在mac上编译`spark-doris-connector`时，会报如下错误。

```
thrift not found
```

## 安装

先查找brew库中的thrift有哪个版本。

```
brew search thrift
```

![image-20210715182630705](http://image-picgo.test.upcdn.net/img/20210715182630.png)

执行安装。

```
brew install thrift@0.9
```

配置环境变量。

```
code ~/.zshrc
```

增加在最后一行。

```
export PATH=/usr/local/Cellar/thrift@0.9/0.9.3.1/bin:$PATH
```

让配置生效。

```
source ~/.zshrc
```



## 验证

![image-20210715182845279](http://image-picgo.test.upcdn.net/img/20210715182845.png)