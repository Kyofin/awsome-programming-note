# 轻量级运维利器：pssh

>   pssh命令是一个python编写可以在多台服务器上执行命令的工具，同时文件的并行复制，杀掉远程主机上的进程等。要使用pssh工具包，必须配置本地主机和被管理的远程主机之间ssh单向的免秘钥认证。

## 前提：ssh免秘钥认证

## 一、安装pssh工具包

```bash
# 安装好 epel 源
yum install -y epel-release
# 安装 pssh 工具包
yum install -y pssh  
```

安装完成后的 pssh 工具包包含以下5个命令

- pssh：在多台远程主机上并行执行命令
- pscp：把文件并行复制到多台远程主机上
- pslurp：把文件从多台远程主机上复制到本地
- pnuke：在多台远程主机上并行杀掉某一进程（类似于killall命令）
- prsync：使用rsync协议将文件从本地主机同步到多台远程主机上

```bash
# pssh 命令常用参数
-h host_file ：host_file 为远程主机列表文件，内容格式如下：test@192.168.1.1:2222
-H host_info ：操作单个远程主机
-o ：将输出的内容保持到指定文件中
-O ：指定ssh参数的具体配置，具体参照ssh服务的配置文件，例如：pssh -O StrictHostKeyChecking=no
-P ：显示命令结果
-i ：显示命令执行的标准输出和错误输出
```



## 二、实际应用

### 1. 操作单台远程主机执行命令

```dart
pssh -H root@node2 -P date
pssh -H root@node2 -i date
```

![img](https://upload-images.jianshu.io/upload_images/5083227-3a1d606149d06415.png)



### 2. 操作多台远程主机执行命令

设置host文件，指定要操作的机子

```ruby
root@node2:22
root@node3:22
```

获取多太机子的日期。

```ruby
pssh -h host -P date
```

![img](https://upload-images.jianshu.io/upload_images/5083227-1de1133750ad90b0.png)



### 3. 在远程主机上使用sudo权限安装软件

```bash
pssh -P -i -h host sudo yum install -y install tree
```



### 4. pscp 和 pslurp 应用实例

####  同步单个文件到多台远程主机

```
pscp.pssh -h host jdk-8u231-linux-x64.rpm /root/
```

这里同步jdk8到node2和node3的`/root`目录下。

![](http://image-picgo.test.upcdn.net/img/20191228141750.png)





```bash

# 同步目录到多台远程主机
pscp -h host -r /usr/src/sc  /tmp 
# 同步远程文件到本地，同步完成后会在本地创建以远程主机名命令的目录
pslurp -h host -L /tmp/remote /etc/hosts hosts
```

![img](https://upload-images.jianshu.io/upload_images/5083227-185012b9e99fc2ff.png)

