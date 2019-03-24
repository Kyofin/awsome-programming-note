# 线上cpu占用过高的线程分析

生产环境的一个java服务占用cpu200%多。该服务里面跑了很多线程，于是想找到是谁引起的。

###1. 先查出java应用的pid

```
jps
```

![image-20190323174107943](/Users/huzekang/Library/Application Support/typora-user-images/image-20190323174107943.png)



###2. 首先dump出该进程的所有线程及状态。

#####使用命令 jstack PID 命令打印出CPU占用过高进程的线程栈

```
jstack -l 10 > 10.stack
```

将进程id为10的线程栈输出到了文件



###3. 使用top命令找到耗cpu的线程
  使用`top -H -p PID` 命令查看对应进程是哪个线程占用CPU过高

```
[goocar@LoginSVR ~]$ top -H -p 5683

 PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND                                                                
 76 ecar      16   0 2442m 1.3g 288m R 38.3  8.4 208:06.62 java                                                                   
 72 ecar      16   0 2442m 1.3g 288m S 37.3  8.4 209:08.91 java                                                                   
 71 ecar      16   0 2442m 1.3g 288m R 37.3  8.4 213:14.04 java                                                                   
 73 ecar      16   0 2442m 1.3g 288m S 35.6  8.4 211:39.23 java                                                                   
 17 ecar      16   0 2442m 1.3g 288m S  0.0  8.4   0:00.00 java                                                                   
 20 ecar      18   0 2442m 1.3g 288m S  0.0  8.4   0:01.62 java                                                                   
 22 ecar      16   0 2442m 1.3g 288m S  0.0  8.4  21:13.33 java    
```

可以看到是76这个线程占用的cpu比较高。

### 4. 检查文件中线程信息

将线程的pid 转成16进制，比如76 = 0x4c

在dump出来的10.stack文件中找0x4c，就知道是哪个线程占用cpu这么高了

- #### 用sed命令

```shell
 sed -n '474,494p' 10.stack
```

这样你就可以只查看文件的第474行到第494行。

- ####  用awk处理

```shell
awk 'NR>474 && NR<494 {print}'  10.stack
```

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190323173805.png)

可以看出该线程是在查oracle的数据