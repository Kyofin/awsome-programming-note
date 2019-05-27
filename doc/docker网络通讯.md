**默认是桥接模式，网络地址为172.17.0.0/16，同一主机的容器实例能够通信，但不能跨主机通信。**

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190514154748.png)

在主机1.2中我进入spider-2容器中与sample-executor-3容器通讯是可以的。

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190514154655.png)

而在机架图中可以看到即使不同主机使用桥接后ip也会重复的，证实了桥接跨主机不能通讯。