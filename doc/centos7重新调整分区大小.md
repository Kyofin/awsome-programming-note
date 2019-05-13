

[TOC]

# centos7重新调整分区大小

> 目标：将home目录多余空间给根目录使用



### 查看磁盘的空间大小： 

```
df -h
```

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190511142634.png)

```
fdisk -l
```

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190511143043.png)



### 备份/home :  

```
cp -r  /home/  homebak/ 
```



### 卸载挂载点 /home ：  

```
umount /home 
```

此时/home挂载点已经不存在了

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190511143306.png)

如果出现 home 存在进程，使用 fuser -m -v -i -k /home 终止 home 下的进程，最后使用 umount /home 卸载 /home 



### 删除/home所在的lv ：

```
lvremove /dev/mapper/centos-home 
```

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190511143454.png)



### 扩展/root所在的lv，增加920G ： 

```
lvextend -L +920G /dev/mapper/centos-root 
```

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190511143749.png)



### 扩展/root文件系统 ： 

```
xfs_growfs /dev/mapper/centos-root 
```

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190511143829.png)



### 查看剩余的空间

```
vgdisplay
```

可以看到剩余21G

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190511144058.png)



### 重新创建home lv：

重新创建home lv 分区的大小，根据 vgdisplay 中的free PE 的大小确定    

```
lvcreate -L 21G -n home centos 
```



### 创建centos-home文件系统:

```
mkfs.xfs /dev/mapper/centos-home 
```



### 挂载 home目录到centos-home文件系统：  

```
mount /dev/mapper/centos-home /home 
```



### 重新查看分配好的结果:

```
df -h
```

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190511144527.png)



### 将homebak目录下文件复制回home目录