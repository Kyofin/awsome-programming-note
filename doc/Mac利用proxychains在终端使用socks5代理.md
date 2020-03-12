# Mac利用proxychains在终端使用socks5代理

一些国外的maven仓库依赖在命令行中下载很慢时，可以用这个方法解决。

### 1.proxychains安装

```
git clone https://github.com/rofl0r/proxychains-ng.git
cd proxychains-ng
./configure
make && make install
cp ./src/proxychains.conf /etc/proxychains.conf
cd .. && rm -rf proxychains-ng
```

也可以用`brew install proxychains-ng`安装。

### 2.编辑proxychains配置

```
vim /usr/local/etc/proxychains.conf 
```

### 3.将`socks4 127.0.0.1 9095`改为

```
socks5 127.0.0.1 1080
```

ps: 默认的`socks4 127.0.0.1 9095`是tor代理，而`socks5 127.0.0.1 1080`是shadowsocks的代理。

`proxychains.conf`文件说明了代理配置格式,如下,这里根据自己使用的代理来配置就行了。

```
ProxyList format
 94 #       type  ip  port  [user pass]
 95 #       (values separated by 'tab' or 'blank')
 96 #
 97 #       only numeric ipv4 addresses are valid
 98 #
 99 #
100 #        Examples:
101 #
102 #       socks5  192.168.67.78   1080    lamer   secret
103 #       http    192.168.89.3    8080    justu   hidden
104 #       socks4  192.168.1.49    1080
105 #       http    192.168.39.93   8080
```

### 4.使用方法

**在需要代理的命令前加上 proxychains4 ，如：**

```shell
/Volumes/Samsung_T5/huzekang/opensource/hdp-3.1.0.0-78/phoenix-release(03913a975*) » proxychains4 mvn -T 4  clean  install -DskipTests 
```