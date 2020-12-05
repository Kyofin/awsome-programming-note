# HDP3.1.0离线安装hadoop集群

## 官方文档

https://docs.cloudera.com/HDPDocuments/Ambari-2.7.4.0/bk_ambari-installation/content/ch_Getting_Ready.html

https://docs.cloudera.com/HDPDocuments/



## 各个组件的tutorial

https://www.cloudera.com/tutorials.html



## 各个组件的版本

https://docs.cloudera.com/HDPDocuments/HDP3/HDP-3.1.0/release-notes/content/comp_versions.html



## 安装包下载

ambari-2.7.3.0：

http://public-repo-1.hortonworks.com/ambari/centos7/2.x/updates/2.7.3.0/ambari-2.7.3.0-centos7.tar.gz

HDP-3.1.0：

http://public-repo-1.hortonworks.com/HDP/centos7/3.x/updates/3.1.0.0/HDP-3.1.0.0-centos7-rpm.tar.gz

HDP-UTILS-1.1.0.22：

http://public-repo-1.hortonworks.com/HDP-UTILS-1.1.0.22/repos/centos7/HDP-UTILS-1.1.0.22-centos7.tar.gz




## centos网络设置

### 虚拟机nat模式和桥接模式的网络配置区别

![](http://image-picgo.test.upcdn.net/img/20191228120402.png)

![](http://image-picgo.test.upcdn.net/img/20191228120739.png)

### 手动设置网络

如果在安装centos系统时就如同上面那样按了打开网络，则安装完成后就可以上网和连通我的mac了。

但是如果没有打开，进入安装完成的精简版系统后就需要**手动设置网络**。

```
vi /etc/sysconfig/network-scripts/ifcfg-ens33
```

![](http://image-picgo.test.upcdn.net/img/20191228122244.png)

这里设置了静态ip。

如果不设置DNS1指定路由器的网关则会ping不通baidu.com。



## hosts设置

```
vim /etc/hosts
```

![](http://image-picgo.test.upcdn.net/img/20191228122837.png)



## 禁用SELinux

使用sed命令替换文件`/etc/selinux/config`中的`SELINUX=enforcing`变成`SELINUX=disabled`

```
sed -i "s/SELINUX=enforcing/SELINUX=disabled/" /etc/selinux/config
```

![](http://image-picgo.test.upcdn.net/img/20191228124611.png)



## umask值(所有机器)

 umask值用于设置用户在创建文件时的默认权限，当我们在系统中创建目录或文件时，目录或文件所具有的默认权限就是由umask值决定的。

 ```
 echo umask 0022 >> /etc/profile
 ```

使环境变量生效

```
 source /etc/profile
```



## 文件句柄修改

```ruby
[root@master ~]# vi /etc/security/limits.conf
```

输入到结尾处。

```
* soft nofile 65536
* hard nofile 65536
* soft nproc 131072
* hard nproc 131072
```



![](http://image-picgo.test.upcdn.net/img/20191228145001.png)

检查是否设置成功。看到数值变大了。

```
ulimit -a
```

![](http://image-picgo.test.upcdn.net/img/20191228145239.png)



## yum拓展安装

像pssh等软件一开始是不能用yum安装的,需要执行下面命令先。

```
yum install -y epel-release
```



## 各节点间ssh免密登录

每个节点都先执行命令生成钥匙，一直回车就可以了

```
ssh-keygen -b 4096 -t rsa
```

各个节点都需执行下面三个命令，配ssh免密登录。

```
ssh-copy-id -i /root/.ssh/id_rsa.pub -p 22 -f root@node1
ssh-copy-id -i /root/.ssh/id_rsa.pub -p 22 -f root@node2
ssh-copy-id -i /root/.ssh/id_rsa.pub -p 22 -f root@node3
```





## JDK安装

### 卸载原有的openjdk

先看看有没有安装java -version

```ruby
[root@java-test-01 ~]# java -version
openjdk version "1.8.0_101"
OpenJDK Runtime Environment (build 1.8.0_101-b13)
OpenJDK 64-Bit Server VM (build 25.101-b13, mixed mode)
```

查找他们的安装位置

```ruby
[root@java-test-01 ~]# rpm -qa | grep java
java-1.8.0-openjdk-headless-1.8.0.101-3.b13.el7_2.x86_64
tzdata-java-2016f-1.el7.noarch
java-1.8.0-openjdk-1.8.0.101-3.b13.el7_2.x86_64
javapackages-tools-3.4.1-11.el7.noarch
java-1.7.0-openjdk-headless-1.7.0.111-2.6.7.2.el7_2.x86_64
java-1.7.0-openjdk-1.7.0.111-2.6.7.2.el7_2.x86_64
python-javapackages-3.4.1-11.el7.noarch
```

删除全部，noarch文件可以不用删除

```ruby
[root@java-test-01 ~]# rpm -e --nodeps java-1.8.0-openjdk-headless-1.8.0.101-3.b13.el7_2.x86_64
[root@java-test-01 ~]# rpm -e --nodeps java-1.8.0-openjdk-1.8.0.101-3.b13.el7_2.x86_64
[root@java-test-01 ~]# rpm -e --nodeps java-1.7.0-openjdk-headless-1.7.0.111-2.6.7.2.el7_2.x86_64
[root@java-test-01 ~]# rpm -e --nodeps java-1.7.0-openjdk-1.7.0.111-2.6.7.2.el7_2.x86_64
```

检查有没有删除成功

```ruby
[root@java-test-01 ~]# java -version
-bash: /usr/bin/java: 没有那个文件或目录
```

注：如果还没有删除，则用`yum -y remove`去删除他们

### 安装jdk8

上传jdk rpm包到node1。

![](http://image-picgo.test.upcdn.net/img/20191228140428.png)

```ruby
[root@node1 ~]# rpm -ivh  jdk-8u231-linux-x64.rpm
```

设置环境变量

```
echo export JAVA_HOME=/usr/java/latest >> /etc/profile
```

生效环境变量

```
source /etc/profile
```





## 关闭防火墙

全部节点都关闭。

```
service firewalld stop
```

防火墙自启动关闭

```
 systemctl disable firewalld
```





## 设置ntp同时钟

所有节点安装NTP：

```ruby
yum -y install ntp
```

配置开机启动：

```ruby
chkconfig ntpd on 
```

设置同步：

```
ntpdate -u ntp.sjtu.edu.cn
```

（时钟服务器根据实际环境设置、本文采用210.72.145.44-国家授时中心服务器IP地址）



## 修改yum源，实现离线安装

### 安装httpd服务（主服务器node1）

```ruby
[root@master ~]# yum -y install httpd

[root@master ~]# service httpd restart
Redirecting to /bin/systemctl restart httpd.service

[root@master ~]# chkconfig httpd on
Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.
```

安装完可以发现多了目录`/var/www/html`,此时打开浏览器访问node1



### 将上面下载的三个包放到/var/www/html目录下（主服务器node1）

 在http服务默认目录下添加目录

```ruby
[root@master ~]# mkdir /var/www/html/hdp 
```

进入添加的目录

```ruby
[root@master ~]# cd /var/www/html/hdp    
```

创建hdp-util解压到的目录

```ruby
[root@master ~]# mkdir /var/www/html/hdp/HDP-UTILS-1.1.0.22/ 
```

上传之前下载包到`/var/www/html/hdp`，后解压

```ruby
[root@master hdp]# tar -zxvf ambari-2.7.3.0-centos7.tar.gz    
[root@master hdp]# tar -zxvf HDP-3.1.0.0-centos7-rpm.tar.gz 
[root@master hdp]# tar -zxvf HDP-UTILS-1.1.0.22-centos7.tar.gz -C /var/www/html/hdp/HDP-UTILS-1.1.0.22/ 
```

此时浏览器打开`http://192.168.5.112/hdp/`即可看到这些文件了。

![](http://image-picgo.test.upcdn.net/img/20191228214028.png)



### 制作本地源（主服务器node1）

#### （1）安装本地源制作相关工具

```
[root@master hdp]# yum install yum-utils createrepo yum-plugin-priorities -y
[root@master hdp]# createrepo  ./
```

####  

#### （2）设置ambari的yum源

```
vim ambari/centos7/2.7.3.0-139/ambari.repo
```

替换掉这两个地址

![](http://image-picgo.test.upcdn.net/img/20191228210155.png)

改成下面内容。

```SHELL
#VERSION_NUMBER=2.7.3.0-139
[ambari-2.7.3.0]
#json.url = http://public-repo-1.hortonworks.com/HDP/hdp_urlinfo.json
name=ambari Version - ambari-2.7.3.0
baseurl=http://192.168.5.112/hdp/ambari/centos7/2.7.3.0-139
gpgcheck=1
gpgkey=http://192.168.5.112/hdp/ambari/centos7/2.7.3.0-139/RPM-GPG-KEY/RPM-GPG-KEY-Jenkins
enabled=1
priority=1


```

把设置好的yum文件，复制到各个节点的`/etc/yum.repos.d/`目录

```ruby
[root@node1 hdp]# cp ambari/centos7/2.7.3.0-139/ambari.repo /etc/yum.repos.d/

[root@node1 hdp]# scp -r ambari/centos7/2.7.3.0-139/ambari.repo root@node2:/etc/yum.repos.d/

[root@node1 hdp]# scp -r ambari/centos7/2.7.3.0-139/ambari.repo root@node3:/etc/yum.repos.d/
```



#### （3） 设置HDP的yum源

```ruby
[root@master hdp]# vim HDP/centos7/3.1.0.0-78/hdp.repo
```

修改红色框的内容变成我们在浏览器中可访问的地址。

![](http://image-picgo.test.upcdn.net/img/20191228210929.png)

改成下面这样。

```SHELL
#VERSION_NUMBER=3.1.0.0-78
[HDP-3.1.0.0]
name=HDP Version - HDP-3.1.0.0
baseurl=http://192.168.5.112/hdp/HDP/centos7/3.1.0.0-78
gpgcheck=1
gpgkey=http://192.168.5.112/hdp/HDP/centos7/3.1.0.0-78/RPM-GPG-KEY/RPM-GPG-KEY-Jenkins
enabled=1
priority=1


[HDP-UTILS-1.1.0.22]
name=HDP-UTILS Version - HDP-UTILS-1.1.0.22
baseurl=http://192.168.5.112/hdp/HDP-UTILS-1.1.0.22/HDP-UTILS/centos7/1.1.0.22
gpgcheck=1
gpgkey=http://192.168.5.112/hdp/HDP-UTILS-1.1.0.22/HDP-UTILS/centos7/1.1.0.22/RPM-GPG-KEY/RPM-GPG-KEY-Jenkins
enabled=1
priority=1
```

把设置好的yum文件，复制到各个节点的`/etc/yum.repos.d/`目录

```ruby
[root@master hdp]# cp HDP/centos7/3.1.0.0-78/hdp.repo /etc/yum.repos.d/

[root@master hdp]# scp -r HDP/centos7/3.1.0.0-78/hdp.repo root@node2:/etc/yum.repos.d/

[root@master hdp]# scp -r HDP/centos7/3.1.0.0-78/hdp.repo root@node3:/etc/yum.repos.d/
```



#### （5）把各个节点设置好的yum重置

只要上面地址不出错就没问题

```ruby
[root@master ambari]# yum clean all
[root@master ambari]# yum makecache
[root@master ambari]# yum repolist
```



## 创建软连接(非必)

home目录最大，可以先创建文件夹hadoop

```
[root@cdh01 home]# mkdir hadoop
```

再在根目录创建软连接，指向home的hadoop目录。

```
ln -s /home/hadoop/   /
```





## 安装ambari-server(主节点node1)

### 安装命令

```
yum -y install ambari-server
```

### 开始配置ambari(手动建表)

```
ambari-server setup
```

然后就在控制台填信息。留空则会选择括号中的默认值。

这里主要填自定义的**jdk**地址`/usr/java/default`，还有选择ambari的数据库。

数据库这里选择了**pg**，因为pg编码没那么多问题，而且也不用上传驱动。

需手动创建pg数据库`ambari`，然后把提示的脚本拿去数据库中执行**建表**。

```ruby
[root@node1 /]# ambari-server setup
Using python  /usr/bin/python
Setup ambari-server
Checking SELinux...
SELinux status is 'disabled'
Customize user account for ambari-server daemon [y/n] (n)? 
Adjusting ambari-server permissions and ownership...
Checking firewall status...
Checking JDK...
Do you want to change Oracle JDK [y/n] (n)? 
Check JDK version for Ambari Server...
JDK version found: 8
Minimum JDK version is 8 for Ambari. Skipping to setup different JDK for Ambari Server.
Checking GPL software agreement...
Completing setup...
Configuring database...
Enter advanced database configuration [y/n] (n)? y
Configuring database...
==============================================================================
Choose one of the following options:
[1] - PostgreSQL (Embedded)
[2] - Oracle
[3] - MySQL / MariaDB
[4] - PostgreSQL
[5] - Microsoft SQL Server (Tech Preview)
[6] - SQL Anywhere
[7] - BDB
==============================================================================
Enter choice (4): 4
Hostname (118.25.102.17): 
Port (12345): 
Database name (ambari): 
Postgres schema (ambari): public
Username (postgres): 
Enter Database Password (CTQ3gt2tGiZfZwx7up3cc3Rj3h54ZvFB): 
Configuring ambari database...
Configuring remote database connection properties...
WARNING: Before starting Ambari Server, you must run the following DDL directly from the database shell to create the schema: /var/lib/ambari-server/resources/Ambari-DDL-Postgres-CREATE.sql
Proceed with configuring remote database connection properties [y/n] (y)? y
Extracting system views...
.....
Ambari repo file contains latest json url http://public-repo-1.hortonworks.com/HDP/hdp_urlinfo.json, updating stacks repoinfos with it...
Adjusting ambari-server permissions and ownership...
Ambari Server 'setup' completed successfully.
```



### 安装ambari-agent(每个节点都要)

```
yum -y install ambari-agent
```



### Ambari-server设置pg驱动地址

 之后安装hive时需要。**如果不安hive则不用**。

这里先手动上传驱动到`/var/www/html/hdp/`。

```ruby
ambari-server setup --jdbc-db=postgres --jdbc-driver=/var/www/html/hdp/postgresql-42.2.4.jar
```



### 修改ambari-server默认端口（自选）

Ambari 使用 `8080` 端口提供服务，这个端口很多情况下会被 tomcat 等其他应用所占用。修改的方法如下：

修改配置文件 `/etc/ambari-server/conf/ambari.properties`

```shell
client.api.port=<port_number>
```

默认情况下配置文件中没有这个选项，添加上就可以。



### 启动ambari-server（主节点）

```ruby
[root@node1 /]# ambari-server start
```

成功后访问`http://192.168.5.112:8080/`。

默认账号密码：admin/admin。

进入后界面如下。

![](http://image-picgo.test.upcdn.net/img/20191228231133.png)

点击开始创建集群。

![](http://image-picgo.test.upcdn.net/img/20191228231234.png)

选择hdp 3.1.0.0版本。

选择自定义仓库，使用离线yum包。

![](http://image-picgo.test.upcdn.net/img/20191228233553.png)

GPL较小，直接用官方的就可以了。

```
http://public-repo-1.hortonworks.com/HDP-GPL/centos7/3.x/updates/3.1.0.0
```

其他仅供参考，实际要写httpd安装的ip：

```
http://192.168.1.95/hdp/HDP/centos7/3.1.0.0-78/
http://192.168.1.95/hdp/HDP-UTILS-1.1.0.22/HDP-UTILS/centos7/1.1.0.22/
```



添加安装的节点和ssh信息。

![](http://image-picgo.test.upcdn.net/img/20191228234155.png)

检查各个节点状态。

![](http://image-picgo.test.upcdn.net/img/20191228234231.png)

选择需要安装的组件。

![](http://image-picgo.test.upcdn.net/img/20191228234520.png)

组件资源安装到各个节点的分配。

![](http://image-picgo.test.upcdn.net/img/20191228234630.png)

分配从属和客户端。

![](http://image-picgo.test.upcdn.net/img/20191228234836.png)

定制服务配置。

![](http://image-picgo.test.upcdn.net/img/20191228235928.png)

需要在pg数据库中手动创建名为hive的数据库。

检测连接成功则可以下一步。

![](http://image-picgo.test.upcdn.net/img/20191229000005.png)

![](http://image-picgo.test.upcdn.net/img/20191229000205.png)

![](http://image-picgo.test.upcdn.net/img/20191229000211.png)

![](http://image-picgo.test.upcdn.net/img/20191229000413.png)

整体配置。

![](http://image-picgo.test.upcdn.net/img/20191229000431.png)

![](http://image-picgo.test.upcdn.net/img/20191229000719.png)

## 注意事项

### 安装集群过程中如果出现错误可以执行以下操作

1.清除数据库。
2.执行以下命令

```ruby
[root@master ~]# ambari-server stop
[root@master ~]# ambari-server reset
[root@master ~]# ambari-server setup
```

3.清除浏览器的缓存。
4.重新运行整个安装向导。

5.本次安装遇到的几个问题，这些文中都有提到，别遗漏了：
	1）密钥的一定别弄错了是**id_rsa**这个文件，而不是id_rsa.pub。
	2）**ambari-agent一定要先安装**，不然确定主机那一波非常慢。





## 卸载（未完成）

1、通过ambari将集群中的所用组件都关闭，如果关闭不了，直接kill-9 XXX

2、 关闭ambari-server，ambari-agent

```sql
ambari-server stop
ambari-agent stop
```

3、yum删除所有Ambari组件

```sql
yum remove -y hadoop_2* hdp-select* ranger_2* zookeeper* bigtop*atlas-metadata* ambari* spark* slide* strom* hive*
```

以上命令可能不全，执行完一下命令后，再执行

```sql
 rpm –qa|grep Ambari版本号
```

 如版本号：2.7.3.0

 查看是否还有没有卸载的，如果有，继续通过#yum remove XXX卸载

4、删除postgresql数据库中各组件的数据

​     postgresql软件卸载后，其数据还保留在硬盘中，需要把这部分数据删除掉，如果不删除掉，重新安装ambari-server后，有可能还应用以前的安装数据，而这些数据时错误数据，所以需要删除掉。





## 项目中使用hdp的依赖

pom文件中需要加入

```XML
<repositories>
        <repository>
            <id>HDP</id>
            <url>https://repo.hortonworks.com/content/repositories/releases/</url>
        </repository>
    </repositories>
```

