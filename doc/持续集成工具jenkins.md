
程序员开发应用，开发后需要提交svn，然后从svn拉取代码，进行构建，发布到tomcat中，发布，然后看呈现效果，这样的工作是频繁反复的在进行的，浪费了程序员的大量时间，那么能不能把这些工作自动化呢，只需要程序员更新代码到svn，然后自动的构建，发布，呈现效果，当然是可以的，通过jenkins来实现。
## 什么是持续集成
互联网软件的开发和发布，已经形成了一套标准流程，最重要的组成部分就是持续集成（Continuous integration，简称CI）。
持续集成指的是，频繁地（一天多次）将代码集成到主干。
它的好处主要有两个。
（1）快速发现错误。每完成一点更新，就集成到主干，可以快速发现错误，定位错误也比较容易。
（2）防止分支大幅偏离主干。如果不是经常集成，主干又在不断更新，会导致以后集成的难度变大，甚至难以集成。
持续集成的目的，就是让产品可以快速迭代，同时还能保持高质量。它的核心措施是，代码集成到主干之前，必须通过自动化测试。只要有一个测试用例失败，就不能集成。

这种做法的核心思想在于：既然事实上难以做到事先完全了解完整的、正确的需求，那么就干脆一小块一小块的做，并且加快交付的速度和频率，使得交付物尽早在下个环节得到验证。早发现问题早返工。

**持续集成**
![图片](https://images-cdn.shimo.im/737vd8eXqlQXQ34d/image.png!thumbnail)
持续集成强调开发人员提交了新代码之后，立刻进行构建、（单元）测试。根据测试结果，我们可以确定新代码和原有代码能否正确地集成在一起。
**持续交付**
![图片](https://images-cdn.shimo.im/YmelRpypaccvrMrH/image.png!thumbnail)
持续交付在持续集成的基础上，将集成后的代码部署到更贴近真实运行环境的「类生产环境」（*production-like environments*）中。比如，我们完成单元测试后，可以把代码部署到连接数据库的 Staging 环境中更多的测试。如果代码没有问题，可以继续手动部署到生产环境中。
**持续部署**
![图片](https://images-cdn.shimo.im/3b6FqXOzKrcVwbeI/image.png!thumbnail)
持续部署则是在持续交付的基础上，把部署到生产环境的过程自动化。

### 总结就是
**持续**，就是说每完成一个完整的部分，就向下个环节交付，发现问题可以马上调整。是的问题不会放大到其他部分和后面的环节。
**集成，**是指软件个人研发的部分向软件整体部分交付，以便尽早发现个人开发部分的问题；
**部署，**是代码尽快向可运行的开发/测试节交付，以便尽早测试；
**交付，**是指研发尽快向客户交付，以便尽早发现生产环境中存在的问题。

1）核心价值体现在：
a、持续集成中的任何一个环节都是自动完成的，无需太多的人工干预，有利于减少重复过程以节省时间、费用和工作量；
b、持续集成保障了每个时间点上团队成员提交的代码是能成功集成的。换言之，任何时间点都能第一时间发现软件的集成问题，使任意时间发布可部署的软件成为了可能；
c、持续集成还能利于软件本身的发展趋势，这点在需求不明确或是频繁性变更的情景中尤其重要，持续集成的质量能帮助团队进行有效决策，同时建立团队对开发产品的信心。

2）持续集成系统的组成
由此可见，一个完整的构建系统必须包括：
A、一个自动构建过程，包括自动编译、分发、部署和测试等。
B、 一个代码存储库，即需要版本控制软件来保障代码的可维护性，同时作为构建过程的素材库。 
C、一个持续集成服务器。本文中介绍的 Jenkins/Jenkins 就是一个配置简单和使用方便的持续集成服务器。
## 什么是jenkins
Jenkins是一个开源的持续集成的服务器，Jenkins开源帮助我们自动构建各类项目。Jenkins强大的插件式，使得Jenkins可以集成很多软件，可能帮助我们持续集成我们的工程项目。

Jenkins对于maven工程完整的编译和发布流程如下：
1、Jenkins从SVN上拉取代码到指定的编译机器上。
2、在编译机器上触发编译命令或脚本。
3、编译得到的结果文件。
4、把结果文件传到指定的服务器上。
5、重启服务
## jenkins环境安装
【如果都会安装可以跳过，仅供参考】
### 首先安装java环境
1、本地下载**jdk8.tar.gz**包，然后通过xshell和xftp工具上传到**/opt**【可自定义】目录。
2、使用**tar -zxvf jdk8.tar.gz **解压文件。
3、打开**/etc/profile**文件中配置java环境
```shell
export JAVA_HOME=/opt/jdk1.8.0_161
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```
使用命令**source /etc/profile **让环境重新加载，输入**java -version**坚持是否配置成功
![图片](https://images-cdn.shimo.im/YBpMwlwsPOsXbsgX/image.png!thumbnail)
### 安装maven环境
1、查看地址
```
https://maven.apache.org/download.cgi
```
2、复制要下载的链接地址，使用linux的wget命令下载。切换到**/opt**目录，直接下载：
```
wget http://mirrors.hust.edu.cn/apache/maven/maven-3/3.5.3/binaries/apache-maven-3.5.3-bin.tar.gz
```
3、使用**tar -zxvf apache-maven-3.5.3-bin.tar.gz **解压文件得到**apache-maven-3.5.3**
4、**/etc/profile**配置环境，并使用命令**source /etc/profile **让环境重新加载
```
export MAVEN_HOME=/opt/apache-maven-3.5.3
export PATH=${MAVEN_HOME}/bin:$PATH
```
5、检验是否配置成功。
![图片](https://images-cdn.shimo.im/p63TOsQeH8Y1mEhs/image.png!thumbnail)
maven的配置setting.xml最好修改镜像源：
```java
<mirror>  
   <id>alimaven</id>  
   <name>aliyun maven</name>  
   <url>http://maven.aliyun.com/nexus/content/groups/public/</url>  
   <mirrorOf>central</mirrorOf>          
 </mirror>  
```
### 安装git环境
按照以下命令安装：
```bash
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils-MakeMaker

wget https://www.kernel.org/pub/software/scm/git/git-1.8.3.1.tar.gz

tar xzf git-1.8.3.1.tar.gz

cd git-1.8.3.1
make prefix=/usr/local/git all
make prefix=/usr/local/git install
echo "export PATH=$PATH:/usr/local/git/bin" >>/etc/bashrc
source /etc/bashrc
git --version
```
### 
## 安装jenkins
### 下载jenkins.war
直接下载war包，windows和linux环境通用。linux可以通过wget命令直接下载到指定目录。windows直接把链接复制到浏览器链接栏即可下载。
```
wget http://ftp-chi.osuosl.org/pub/jenkins/war-stable/2.164.1/jenkins.war
```
下载完了之后可以在当前目录看到jenkins.war包，表示已经下载完成。
### 运行命令
jenkins.war包可以有两种运行方式：
* 第一种是把war包放到tomcat的webapps目录下运行；
* 第二种是直接通过java -jar来启动项目，因为jenkins.war包里面已经内置了jetty服务器，可以直接启动，通过--httpPort来指定启动端口，添加&表示以服务形式启动。
```bash
nohup java -jar jenkins.war --httpPort=8081 &
```
### 安装过程
打开链接地址[http://你的ip](http://你的ip):8081,
![图片](https://images-cdn.shimo.im/NO68eFVOhVAZIDnm/image.png!thumbnail)
获取秘钥：
```bash
cat /root/.jenkins/secrets/initialAdminPassword
```
新手最好直接选择安装推荐的插件就可以，熟悉以后下次就可以自定义插件安装即可。
![图片](https://images-cdn.shimo.im/vgryHTJFXaYceKDu/image.png!thumbnail)
正在下载推荐的额插件
![图片](https://images-cdn.shimo.im/v9HGSdXQtWIefB2H/image.png!thumbnail)
插件安装完毕，跳转到创建管理员。填入用户名密码。
![图片](https://images-cdn.shimo.im/ilnAHgiUfLw9EtjT/image.png!thumbnail)
随便建个账号密码：admin/admin

![图片](https://images-cdn.shimo.im/CAe28rIy690lalvh/image.png!thumbnail)
jenkins安装成功。
![图片](https://images-cdn.shimo.im/FiW7CM9XlaIgvpaF/image.png!thumbnail)
### 
如果有bug， 就重新启动一下jenkins，用刚才创建的用户重新登录。

**设置中文**
安装插件 Locale plugin
>在可选插件中搜索Locale plugin，我已经安装好了，所以没搜索到。

在 系统管理 - 系统设置 - Locale 为zh_CN
记得打钩！！！！
### 
### jenkins插件管理

打开管理插件页面：
![图片](https://images-cdn.shimo.im/IwxBZQ8YGN82fCrg/image.png!thumbnail)
* 一个构建maven项目插件

![图片](https://images-cdn.shimo.im/3CEjJga1gjcodKoq/image.png!thumbnail)
* 一个发布到tomcat的插件

![图片](https://images-cdn.shimo.im/Adnn9dv6Amoall7F/image.png!thumbnail)

选择，直接安装！
### jenkins全局配置
系统管理--》全局工具配置，需要把服务器的jdk、maven、git等环境配置好。

![图片](https://images-cdn.shimo.im/9wN9RLhN7mwldckM/image.png!thumbnail)
![图片](https://uploader.shimo.im/f/pATMkwDRwrMZ2F4Y.png!thumbnail)


![图片](https://uploader.shimo.im/f/FVWGNFkXvogRYFzs.png!thumbnail)

![图片](https://uploader.shimo.im/f/ey3EuaXrTkIa94B6.png!thumbnail)
![图片](https://uploader.shimo.im/f/qqYFvvBTLGotb6fn.png!thumbnail)

配置好后记得点应用和保存！！！！
## 
## 构建jar项目
### 共同配置
* 新建一个maven项目。

![图片](https://uploader.shimo.im/f/ReTvH4wYbrUuRtYd.png!thumbnail)

* 配置git或者svn地址，jenkins会自动从远程仓库拉去最新代码。

![图片](https://images-cdn.shimo.im/bDKekdg9XZck8i7e/image.png!thumbnail)
如果是私有项目的话，需要加入用户名密码
![图片](https://uploader.shimo.im/f/ouMmi64z4BwOBHnQ.png!thumbnail)
![图片](https://images-cdn.shimo.im/8i0cRSO7EOQzck7Y/image.png!thumbnail)
![图片](https://images-cdn.shimo.im/kg6FIMSZLbMk2GEF/image.png!thumbnail)
* 开始maven打包构建的命令，可以去掉test测试
```
clean install -Dmaven.test.skip=true -Ptest
```
![图片](https://images-cdn.shimo.im/V86jrVzTrY8SwM4w/image.png!thumbnail)

### jenkins与应用服务器同一台机器实现自动部署启动项目
* 经过上一个步骤之后，jenkins会在默认路径/root/.jenkins/workspace/上看到打包的项目，/root/.jenkins/workspace/homework/target目录下有打包的jar文件。spring-boot-homework-0.0.1-SNAPSHOT.jar。因此我们打包完成之后的工作就是要替换原来的jar文件，然后重启项目。

![图片](https://images-cdn.shimo.im/ADu38UgvWQkizAeZ/image.png!thumbnail)
这里我们使用shell脚本来完成。
1. DIR表示存放jar文件、和启动项目的文件夹。JAEFILE是指jar包的名称。
2. 接下来的工作其实就是杀掉原来的spring-boot-homework-0.0.1-SNAPSHOT.jar进程
3. 备份原来的jar包
4. 把新生成的jar覆盖原来的
5. 然后java -jar启动项目

* BUILD_ID=dontKillMe表示jenkins不杀掉衍生的线程
* nohup表示不用提示

详细shell脚本如下：
```shell
#!/bin/bash
echo '~~~~~~~~~开始启动项目~~~~~~~~~'

DATE=$(date +%Y%m%d_%H%M)
export JAVA_HOME PATH CLASSPATH
JAVA_HOME=/java/jdk1.8.0_181
PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH
CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$CLASSPATH
DIR=/opt/jar
JARFILE=internetHospital.jar

if [ ! -d $DIR/backup ];then
   mkdir -p $DIR/backup
fi
cd $DIR

varpid=`ps aux|grep "internetHospital"|grep -v grep`
if test -z ${varpid}
then
        echo "没有查找到$JARFILE进程"

else
        echo "查找到进程$JARFILE"
        ps -ef | grep $JARFILE | grep -v grep | awk '{print $2}' | xargs kill -9

fi
mv $JARFILE backup/$JARFILE$DATE
mv -f /root/.jenkins/workspace/internethospital/target/$JARFILE .

BUILD_ID=dontKillMe nohup java -jar $JARFILE > out.log &
if [ $? = 0 ];then
        sleep 30
        tail -n 150 out.log
fi

cd backup/
```
ls -lt|awk 'NR>5{print $NF}'|xargs rm -rf

echo '~~~~~~~~~结束启动项目~~~~~~~~~'

致此，项目就会自动部署，只需要点击构建按钮，jenkins就会自动拉去最新代码，然后备份、重启项目。

代码注释：
获得已经在执行程序的pid ，并kill掉
```bash
ps -ef | grep $JARFILE | grep -v grep | awk '{print $2}' | xargs kill -9
```

代码流程：
1. 先创建存放jar包的目录，如果该目录不存在，然后进入该目录
2. 找出之前已经在运行的jar，并kill掉
3. 把之前的jar包备份
4. 将jenkins最新编译出来的jar包移动到当前目录
5. 启动jar包


## 回顾一下流程：
* 首先创建maven项目
* 配置项目的git仓库地址
* 配置maven构建命令（打包命令）
* 打包文成之后执行shell实现切换jar，重新部署项目。

### 
