# 鲲鹏ARM服务器部署juicefs

## 环境依赖

- ARM架构
- 华为云服务器
- 统信OS 20



## docker

yum install docker



## minio

```ruby
wget https://dl.minio.io/server/minio/release/linux-arm64/minio
 chmod +x ./minio
mkdir -p /minio/minio_data

nohup ./minio server /minio/minio_data  --console-address :9002 > /minio/minio_data/minio.log 2>&1 &


```



## redis

docker load -i redis-kunpeng-5.0.9.tar.gz 

docker run -p 6379:6379 -d --restart=always redis-arm:5.0.9



## juicefs

```
JFS_LATEST_TAG=$(curl -s https://api.github.com/repos/juicedata/juicefs/releases/latest | grep 'tag_name' | cut -d '"' -f 4 | tr -d 'v')

wget "https://github.com/juicedata/juicefs/releases/download/v${JFS_LATEST_TAG}/juicefs-${JFS_LATEST_TAG}-linux-arm64.tar.gz"

tar -zxf "juicefs-${JFS_LATEST_TAG}-linux-arm64.tar.gz"

```

注意：该解压会直接解压所有文件到当前目录。

初始化文件系统：

```
./juicefs format \
    --storage minio \
    --bucket http://127.0.0.1:9000/unjfs \
    --access-key minioadmin \
    --secret-key minioadmin \
    redis://localhost:6379/1 \
    unjfs
```

挂载本地目录

```
./juicefs mount redis://localhost:6379/1   unjfs
```





## java

yum list installed |grep java

yum -y remove java-1.8.0-openjdk*

rpm -ivh jdk-8u321-linux-aarch64.rpm 



vim /etc/profile

```
# 末尾追加以下内容
export JAVA_HOME=/usr/java/default
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

```

source /etc/profile 



## maven

wget https://dlcdn.apache.org/maven/maven-3/3.8.4/binaries/apache-maven-3.8.4-bin.tar.gz

tar zxvf apache-maven-3.8.4-bin.tar.gz

vim /etc/profile

```
export MAVEN_HOME=/root/apache-maven-3.8.4
export PATH=$PATH:$MAVEN_HOME/bin

```

source /etc/profile 





## go

rm -rf /usr/local/go

 tar zxf go1.15.15.linux-arm64.tar.gz -C /usr/local

tar zxf go1.16.1.linux-arm64.tar.gz   -C /usr/local



vim /etc/profile

```
#golang env config
export GO111MODULE=on
export GOROOT=/usr/local/go 
export GOPATH=/home/gopath
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
```

source /etc/profile 



go version

go env -w GOPROXY=https://goproxy.cn,direct 

go env



```
mkdir gopath
cd gopath


```

vim hello.go

```
package main  
import "fmt"  
func main() {  
    fmt.Printf("Hello, world!\n")  
}  
```

go run hello.go



## 编译juicefs hadoop sdk

官网现在都没ARM架构下的hadoop java sdk，所以需要自行编译。准备好maven、jdk、go的环境就可以进行编译了。

```
$ git clone https://github.com/juicedata/juicefs.git
$ cd juicefs/sdk/java
$ make
```

编译完成后，可以在 `sdk/java/target` 目录中找到编译好的 `JAR` 文件，包括两个版本：

- 包含第三方依赖的包：`juicefs-hadoop-X.Y.Z.jar`
- 不包含第三方依赖的包：`original-juicefs-hadoop-X.Y.Z.jar`

建议使用包含第三方依赖的版本。





## 安装hadoop客户端测试

```
wget https://archive.apache.org/dist/hadoop/hadoop-2.8.0.tar.gz
```

修改core-site.xml

````
<property>
  <name>fs.jfs.impl</name>
  <value>io.juicefs.JuiceFileSystem</value>
</property>
<property>
  <name>fs.AbstractFileSystem.jfs.impl</name>
  <value>io.juicefs.JuiceFS</value>
</property>
<property>
  <name>juicefs.meta</name>
  <value>redis://localhost:6379/1</value>
</property>
<property>
  <name>juicefs.cache-dir</name>
  <value>/data*/jfs</value>
</property>
<property>
  <name>juicefs.cache-size</name>
  <value>1024</value>
</property>
<property>
  <name>juicefs.access-log</name>
  <value>/tmp/juicefs.access.log</value>
</property>
````

hadoop fs  -ls  jfs://unjfs/