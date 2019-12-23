## 用户
### 添加用户
1. 先切换到 root 用户，在终端输入

```
sudo su
```
按照提示输入密码。
2. 创建新用户账户，在终端输入

```
adduser hzk
```
这句执行成功后，会自动在 home 目录下新建一个 hzk 目录。
### 修改用户密码
3. 为新账户设置密码，在终端输入

```
passwd hzk
```
按照提示输入密码。密码设置成功。
此时已经可以用这个用户 ssh 登录了。

### 查看用户所在用户组
![图片](https://uploader.shimo.im/f/N5VYLwdQhNwiOEAn.png!thumbnail)

可以看到 hive 用户在 hive 组、supergroup 组。

也可以用下面命令查看。

![图片](https://uploader.shimo.im/f/XAyZAGIq1TEUrN2i.png!thumbnail)

### 将用户添加到用户组中
```
usermod -a -G supergroup root
usermod -a -G hive root
```
将root用户添加进hdfs的超级管理员组supergroup

将root用户添加进hive组



### 删除用户

```
userdel -r hzk
```
不带选项，只会删除用户。用户的家目录将仍会在/home 目录下。
要完全的删除用户信息，使用-r 选项

### 为新账户添加 sudo 权限
vim /etc/sudoers

找到

![图片](https://uploader.shimo.im/f/rE8qqSvbxBQbmYnA.png!thumbnail)

在其后添加如下内容，

![图片](https://uploader.shimo.im/f/4RkAJEmE5swKj0BV.png!thumbnail)

保存退出。

### sudo 操作不需要输入密码：
```
 sudo visudo
```
把下面一行内容添加到文档的最后并保存文件：
```
hzk   ALL=(ALL:ALL) NOPASSWD: ALL
```
### 本机用户的 ssh public key 添加到目标主机上
```
ssh-copy-id -i ~/.ssh/id_rsa.pub nick@xxx.xxx.xxx.xxx
```
之后就可以 ssh 连接时不用输入密码了。



## 用户组

### 查看系统已有的组。
```
vim /etc/group
```
![图片](https://uploader.shimo.im/f/C8tNFYVRCfckwxIq.png!thumbnail)


