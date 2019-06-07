[TOC]



## 实验环境

#### 主机信息：

- 主机win10。
- 网络连接的是无线网络。网关是192.168.5.1。

- 虚拟机软件是VMware。虚拟机是centos7阿里的精简版。

- 主机ip：192.168.5.25

#### centos虚拟机的ip：

1. 【nat模式下】192.168.148.40

2. 【桥接模式下】192.168.5.70

#### 虚拟机账号：

- 账号：root
- 密码：19950721

## 阿里云镜像下载地址：
[http://mirrors.aliyun.com/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-1810.iso](http://mirrors.aliyun.com/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-1810.iso)

## 桥接模式和nat模式区别
* 当我们要在局域网使用虚拟机，对局域网其他pc提供服务时，例如提供ftp，提供ssh，提供http服务，那么就要选择桥接模式。
*  由于NAT的网络在vmware提供的一个虚拟网络里，所以局域网其他主机是无法访问虚拟机的，而宿主机可以访问虚拟机。虚拟机也可以访问局域网的所有主机，因为真实的局域网相对于NAT的虚拟网络，就是NAT的虚拟网络的外网。
## Nat模式下设置静态ip（与下面二选一）
我这里是用VMware虚拟机软件的。
1. 需要先设置【虚拟机网络编辑】

![图片](https://uploader.shimo.im/f/odhL5HOKxO8tbdqP.png!thumbnail)

1. 设置该虚拟机网络用nat模式。所以这里要编辑一下nat模式。

![图片](https://uploader.shimo.im/f/bqMCw5D4EhoyPUEC.png!thumbnail)

1. 在虚拟机centos系统内设置虚拟机IP

/etc/sysconfig/network-scripts/ifcfg-ens33（修改网卡）
```bash
vi /etc/sysconfig/network-scripts/ifcfg-ens33
```

![图片](https://uploader.shimo.im/f/n13SnKGyQng0ynms.png!thumbnail)
参数说明：
* ONBOOT：开机启动。
* NM_CONTROLLED：网络管理组件是否启用，精简版的是没有这个组件的。所以就不需要开启。
* BOOTPROTO：网络分配方式，静态。
* IPPADDR：手动指定ip地址。
* NETMASK：子网掩码。
* GATEWAY：网关ip。编辑好以后保存退出。

/etc/resolv.conf（设置DNS）
```bash
vi /etc/resolv.conf
```
![图片](https://uploader.shimo.im/f/jh4rZIvJYksadxmm.png!thumbnail)

1. 重启网卡
```bash
service network restart
```
### 
1. 测试ping效果
* ping我的主机

![图片](https://uploader.shimo.im/f/Al85whgSv1cKuoOt.png!thumbnail)
* ping百度

![图片](https://uploader.shimo.im/f/OHlg4suXyfcka1wO.png!thumbnail)

#### *遇见问题*
此时centos可以ping同我的windows主机。
然而我用windows去ping虚拟机时却不通。
![图片](https://uploader.shimo.im/f/A3mYlYe40Ig8vNVA.png!thumbnail)

用ipconfig查看网络情况。
发现ip的网段不是192.168.148.x的。
显然与虚拟机的IP不在同一网段
![图片](https://uploader.shimo.im/f/7Ans231KAqU2RAme.png!thumbnail)

修改vmnet8网络适配器
![图片](https://uploader.shimo.im/f/xNuX8HLGYCMDWQtm.png!thumbnail)

之后就可以ping通centos了。
![图片](https://uploader.shimo.im/f/MTEdhoxPlbcP5Sxr.png!thumbnail)
## 桥接模式下设置静态ip（与上面二选一）
1. 设置虚拟机网络编辑。

点击Vmnet0
## ![图片](https://uploader.shimo.im/f/RNlZcMRzzYUgbTC0.png!thumbnail)
1. 参考主机所属网段。

![图片](https://uploader.shimo.im/f/UViCdTpp4ygiwPxB.png!thumbnail)

1. 改变虚拟机ip和网关。

/etc/sysconfig/network-scripts/ifcfg-ens33（修改网卡）
```bash
vi /etc/sysconfig/network-scripts/ifcfg-ens33
```
![图片](https://uploader.shimo.im/f/uSrAL4hjfL0rMzfb.png!thumbnail)
其他操作和nat模式基本一致。

## 设置hostname
1. 要查看主机名相关的设置：
```bash
[root@localhost ~]# hostnamectl
```
  ![图片](https://uploader.shimo.im/f/pIfLcgcL13wdxuv0.png!thumbnail)

1. 同时修改静态、瞬态和灵活主三个主机名：
```bash
[root@localhost ~]# hostnamectl set-hostname node1
```
一旦修改了静态主机名，/etc/hostname 将被自动更新。
然而，/etc/hosts 不会更新以保存所做的修改，所以你每次在修改主机名后一定要手动更新/etc/hosts，之后再重启CentOS 7。
否则系统再启动时会很慢。

1. 手动更新/etc/hosts 
```bash
vim /etc/hosts

127.0.0.1 localhost localhost.localdomain localhost4 localhost4.localdomain4 node1
::1 localhost localhost.localdomain localhost6 localhost6.localdomain6 node1
```

1. 重启观察效果
```
reboot -f 
```
* 可观察到@后面变成了node1

![图片](https://uploader.shimo.im/f/uKZl8K5aVC0ViE6R.png!thumbnail)
* ping node1也有响应

![图片](https://uploader.shimo.im/f/E3Ha0QjjVsM2KK1P.png!thumbnail)

## CentOS7系统配置国内yum源和epel源
### 1.首先进入/etc/yum.repos.d/目录下，新建一个repo_bak目录，用于保存系统中原来的repo文件
```bash
cd /etc/yum.repos.d/
mkdir repo_bak
mv *.repo repo_bak/
```

### 2.在CentOS中配置使用网易和阿里的开源镜像
到网易和阿里开源镜像站点下载系统对应版本的repo文件
```bash
 wget http://mirrors.aliyun.com/repo/Centos-7.repo &&
 wget http://mirrors.163.com/.help/CentOS7-Base-163.repo
```
查看当前目录文件可以看到新下载的.repo
```
 ls
```
Centos-7.repo  CentOS-Base-163.repo  repo.bak

或者手动下载repo文件并上传到/etc/yum.repos.d/目录。
[网易开源镜像站](http://mirrors.163.com/.help/centos.html)
[阿里开源镜像站](https://opsx.alibaba.com/mirror?lang=zh-cn)
### 3.清除系统yum缓存并生成新的yum缓存
列出/etc/yum.repos.d/目录下的文件
```
 ls      
```
Centos-7.repo  CentOS-Base-163.repo  repo.bak

清除系统所有的yum缓存
```
yum clean all     
```
生成yum缓存
```
yum makecache    
```

### 4.安装epel源
```bash
 yum list | grep epel-release
 yum install -y epel-release
```

 epel源安装成功，比原来多了一个epel.repo和epel-testing.repo文件
```
 ls           
```
Centos-7.repo  CentOS-Base-163.repo  epel.repo  epel-testing.repo  repo.bak
### 5.使用阿里开源镜像提供的epel源
```
wget -O /etc/yum.repos.d/epel-7.repo http://mirrors.aliyun.com/repo/epel-7.repo    
```

```
 ls
```
CentOS7-Base-163.repo  Centos-7.repo  epel-7.repo  epel.repo  epel-testing.repo  repo_bak
### 6.再次清除系统yum缓存，并重新生成新的yum缓存
```
 yum clean all &&
 yum makecache
```
### 7.查看系统可用的yum源和所有的yum源
```
yum repolist enabled
yum repolist all
```




