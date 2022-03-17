# centos7离线安装clickhouse

## 版本

21.12.3.32



## 下载文件
下载地址：https://packages.clickhouse.com/rpm/stable/



主要下如下文件：


```
clickhouse-common-static-21.12.3.32-2.x86_64.rpm    
clickhouse-server-21.12.3.32-2.noarch.rpm      
clickhouse-client-21.12.3.32-2.noarch.rpm   
```

先离线下载到本地





## rpm安装clickhouse

![image-20220317152228000](http://image-picgo.test.upcdn.net/img/20220317152228.png)

```
rpm -ivh *.rpm
```

安装后的目录文件：

/etc/clickhouse-server ：clickhouse 服务端配置文件目录
/etc/clickhouse-client ：clickhouse 客户端配置文件目录
/var/lib/clickhouse ：clickhouse 默认数据目录
/var/log/clickhouse-server ：clickhouse 默认日志目录
/etc/init.d/clickhouse-server ：clickhouse 服务端启动脚本

## Clickhouse 启动与验证

安装完成后，需要手动启动服务：

```
systemctl stop clickhouse-server
systemctl status clickhouse-server
systemctl start clickhouse-server

```



进入 Clickhouse 客户端交互界面：

```
clickhouse-client --password 123456
```



## 开启grpc接口

```
vim /etc/clickhouse-server/config.xml 
```

取消如下注释。

![](http://image-picgo.test.upcdn.net/img/20220317165442.png)



## 开启远程访问

默认clickhouse是监听本地请求的，可以通过如下命令看到

```
lsof -i :8123
```

![image-20220317165622709](http://image-picgo.test.upcdn.net/img/20220317165622.png)

修改配置文件，开启远程访问

```
vim /etc/clickhouse-server/config.xml 
```

取消如下注释。

![image-20220317165830288](http://image-picgo.test.upcdn.net/img/20220317165830.png)

修改完重启后，可以看到变成监听所有ip的请求了。

![image-20220317170120041](http://image-picgo.test.upcdn.net/img/20220317170120.png)



## 卸载clickhouse

```
yum remove -y clickhouse-common-static
yum remove -y clickhouse-server-common
yum list installed | grep clickhouse
rm -rf /var/lib/clickhouse
rm -rf /etc/clickhouse-*
rm -rf /var/log/clickhouse-server
```

