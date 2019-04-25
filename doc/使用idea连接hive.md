[TOC]

# 使用idea连接hive

## 下载驱动

首先我们需要准备驱动等jar包。

可以到官网的下载 <https://www.cloudera.com/downloads/connectors/hive/jdbc/2-5-4.html>

可能网速有点慢，这里我也准备好了。

<https://gitee.com/huzekang/cdhproject/tree/master/hive>



## 新建数据源类型

打开idea新建一个数据源类型，定义name为hive

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190425223147.png)



## 引入驱动

点击增加驱动文件按钮，选择下载好的cdh hive的驱动

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190425223349.png)



选择和服务器hive版本对应的driver。我的cdh是5.13版本，hive是1.1.0。

你可以上服务器使用命令`locate */hive/lib/hive*jar`查看hive版本。

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190425223710.png)



## 设置模板

添加一个url template方便以后使用。

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190425223952.png)



模板格式为

```
jdbc:hive2://{host::localhost}?[:{port::10000}][/{database:::schema}?]
```



此时再打开idea，即可看到hive的连接可以使用

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190425224353.png)

