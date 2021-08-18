# 使用FreeIPA搭建kerberos环境

### 介绍

使用freeIPA可以在创建kerberos用户时，一并自动创建linux用户、ldap用户，并且三者的密码都是一样的。



## 参考资料

https://www.cnblogs.com/yanghehe/p/12294172.html





### 注意事项

hostname一定要是小写，且是一个域名：如`ozone001.hadoop.com`



### 1. 扩充秘钥长度

[下载地址](https://www.oracle.com/java/technologies/javase-jce8-downloads.html)

解压后得到如下文件:

![image-20210817102443667](http://image-picgo.test.upcdn.net/img/20210817102443.png)

将 local_policy.jar和US_export_policy.jar复制到每台服务器JDK路径下的jre/lib/security目录下即可。

![image-20210817105436831](http://image-picgo.test.upcdn.net/img/20210817105436.png)

### 2. 安装IPA-Sever

在ozone001.hadoop.com服务器上安装ipa-server。

#### 安装需要的工具

```
yum install -y ntp ipa-server bind bind-dyndb-ldap ipa-server-dns
```



#### 检查dns是否能解析域名

```
dig ozone001.hadoop.com
```





#### 安装ipa-server

```
ipa-server-install 
```

出现错误：

```
    from .connectionpool import (
  File "/usr/lib/python2.7/site-packages/urllib3/connectionpool.py", line 31, in <module>
    from .connection import (
  File "/usr/lib/python2.7/site-packages/urllib3/connection.py", line 45, in <module>
    from .util.ssl_ import (
  File "/usr/lib/python2.7/site-packages/urllib3/util/__init__.py", line 4, in <module>
    from .request import make_headers
  File "/usr/lib/python2.7/site-packages/urllib3/util/request.py", line 5, in <module>
    from ..exceptions import UnrewindableBodyError
ImportError: cannot import name UnrewindableBodyError
```

解决办法：

```
 pip install --ignore-installed urllib3
```



继续安装还是出现错误：

```
IPv6 stack is enabled in the kernel but there is no interface that has ::1 address assigned. Add ::1 address resolution to 'lo' interface. You might need to enable IPv6 on the interface 'lo' in sysctl.conf.
```

解决办法：

```
vim /etc/sysctl.conf
```

修改成：

```
net.ipv6.conf.lo.disable_ipv6 = 0
```

刷新配置

```
sysctl -p
```

查看是否成功

![image-20210817112653004](http://image-picgo.test.upcdn.net/img/20210817112653.png)



继续安装

```
ipa-server-install 
```

![image-20210817133400755](http://image-picgo.test.upcdn.net/img/20210817133400.png)

设置ipa-server的admin密码为：admin123

![image-20210817133447130](http://image-picgo.test.upcdn.net/img/20210817133447.png)

等待几分钟后，安装完成。

![image-20210817133539591](http://image-picgo.test.upcdn.net/img/20210817133539.png)



#### 安装失败处理办法

第一回安装IPA失败的话需要执行下` ipa-server-install --uninstall`这条命令。先卸载第一回安装的包，否则第二回执行错误





### 3. 验证ipa-server安装成功

访问：https://ozone001.hadoop.com/ipa/ui/#/e/user/search

![image-20210817133112611](http://image-picgo.test.upcdn.net/img/20210817133112.png)



### 4. 安装ipa-client

在ozone002.hadoop.com上安装ipa-client。

````
yum install ipa-client
````



#### 检查是否能解析域名

```
dig ozone002.hadoop.com
```

![image-20210817135909294](http://image-picgo.test.upcdn.net/img/20210817135909.png)

#### 进行安装ipa-client

```
ipa-client-install
```

![image-20210817140126114](http://image-picgo.test.upcdn.net/img/20210817140126.png)

输入ipa-server的admin账号密码

![image-20210817140625692](http://image-picgo.test.upcdn.net/img/20210817140625.png)

#### 检查是否安装成功

打开ipa-server的web界面里的主机可以看到主机注册进来了

![image-20210817140234581](http://image-picgo.test.upcdn.net/img/20210817140234.png)



### 5. 安装ipa-admintools

在ozone002.hadoop.com上安装ipa-admintools。

```
yum install ipa-admintools
```

在终端创建ipa用户。

```
ipa user-add nau --first=nau --last=nau --password 
```

![image-20210817180700863](http://image-picgo.test.upcdn.net/img/20210817180700.png)

创建成功后，可以在ipa-server上看到该用户

![image-20210817180735924](http://image-picgo.test.upcdn.net/img/20210817180735.png)

删除用户，需要先登录admin账号

```shell
[huzekang@ozone002 security]$ kinit admin
Password for admin@HADOOP.COM: 
[huzekang@ozone002 security]$ ipa user-del nau
----------
已删除用户"nau"
----------

```

