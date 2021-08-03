本篇文章主要是对官方文档中没有列举出来的细节
以及编译过程中会遇到问题的一个补充

第一手资料请参照官方文档:

[ambari源码编译安装官方文档](https://cwiki.apache.org/confluence/display/AMBARI/Ambari+Development)

注:文档以2.6版本为主

2.7.5版本对应的源码下载命令为:

```shell
wget http://archive.apache.org/dist/ambari/ambari-2.7.5/apache-ambari-2.7.5-src.tar.gz

```
对应的编译命令为:
```shell
mvn versions:set-property -Dproperty=revision -DnewVersion=2.7.5.0.0
mvn -B clean install package rpm:rpm -DnewVersion=2.7.5.0.0 -DskipTests -Drat.skip -Dpython.ver="python >= 2.6" -Preplaceurl
```

第三部分3.9及之后的问题为ambari2.7.5版本编译中单独碰到的问题

## 文章目录


[**一. 准备工作**](#1)
   - [1.1 获取Ambari2.6.0源码](#11)
   - [1.2 搭建编译环境](#12)
     - [1.2.1 配置java环境](#121)
     - [1.2.2 配置maven环境](#122)
     - [1.2.3 安装rpm和rpmbuild](#123)
     - [1.2.4 安装g++](#124)
     - [1.2.5 检查python版本](#125)
     - [1.2.6 初始化python-devel](#126)
     - [1.2.7 安装bower, gulp](#127)
     - [1.2.8 安装git](#128)
     - [1.2.9 安装python setuptools](#129)
     - [1.2.10 设置最大打开文件数](#1210)
   
 [**二. 源码编译**](#2)
   - [2.1 配置阿里云远程仓库](#21)  
   - [2.2 Ambari2.6.0源码编译](#22)

 [**三. ambari2.6.0源码编译遇到的问题及解决方案**](#3)
   - [3.1 Ambari Main编译报错:  RPM build execution returned: '127' executing](#31)
   - [3.2 Ambari Web编译报错: requires Maven version 3.1.0](#32)
   - [3.3 Ambari Admin View编译失败: Command execution failed. Process exited with an error: 1](#33)
   - [3.4 Ambari Metrics Storm Sink编译失败: Could not resolve dependencies for project](#34)
   - [3.5 Ambari Server编译失败: Unable to parse configuration of mojo](#35)
   - [3.6 Ambari Server编译失败: The entity name must immediately follow the '&' in the entity reference](#36)
   - [3.7 Ambari Server编译失败: Failed to execute goal org.codehaus.mojo:rpm-maven-plugin:2.1.4:rpm (default-cli) on project ambari-server](#37)
   - [3.8 Ambari Groovy Shell编译失败: Could not resolve dependencies for project org.apache.ambari:ambari-groovy-shell:jar:2.6.0.0.0](#38)
   - [3.9 Ambari Web编译失败
 Failed to run task: 'yarn install --ignore-engines --pure-lockfile' failed](#39)
   - [3.10 Ambari Admin View编译失败: Failed to run task: 'npm install --unsafe-perm' failed](#310)
   - [3.11 Ambari Admin View编译失败: ECMDERR Failed to execute "git ls-remote --tags --heads](#311)
   - [3.12 Ambari Metrics Collector编译失败 An Ant BuildException has occured:](#312)

[**四. 安装和配置Ambari**](#4)
  - [4.1 安装配置Ambari Server](#41)
  - [4.2 安装配置Ambari Agent](#42)
  - [4.3 修改yum源,离线安装HDP及HDP-UTILS](#43)
 
[**五. ambari2.6.0安装配置遇到的问题及解决方案**](#5)

  - [5.1 ambari server启动报错: java.net.BindException: Address already in use](#51)
  - [5.2 ambari agent启动报错: SSLError: Failed to connect. Please check openssl library versions](#52)


<div id ="1"></div>


### 一. 准备工作

<div id ="11"></div>



#### 1.1 获取Ambari2.6.0源码
```shell
wget http://archive.apache.org/dist/ambari/ambari-2.6.0/apache-ambari-2.6.0-src.tar.gz

# 附1 wget命令安装方法:  yum install wget
# 附2 使用wget命令如遇报错:Unable to locally verify the issuer's authority  需要在wget命令后面加上--no-check-certificate 参数
```
```shell
tar -zxvf apache-ambari-2.6.0-src.tar.gz
```


<div id ="12"></div>


#### 1.2 搭建编译环境




<div id ="121"></div>


##### 1.2.1 配置java环境

**注意Ambari2.6要求的版本为JDK7,Ambari2.7要求的最低版本为JDK8**

```shell
vi ~/.bashrc
```

```shell
export JAVA_HOME=/opt/modules/jdk1.8.0_151 #以自己配置为准
export PATH=$JAVA_HOME/bin:$PATH
export _JAVA_OPTIONS="-Xmx2048m -XX:MaxPermSize=512m -Djava.awt.headless=true"

```
```shell
source ~/.bashrc
```


<div id ="122"></div>


##### 1.2.2 配置maven环境

**注意:ambari适配的maven版本最低为3.3.9,但3.5.4之后的版本可能出现兼容问题(具体参看问题3.5)**

```shell
vi ~/.bashrc
```
```shell
export MAVEN_HOME=/opt/modules/apache-maven-3.5.3 #以自己配置为准
export PATH=$MAVEN_HOME/bin:$PATH

```
```shell
source ~/.bashrc
```


<div id ="123"></div>

##### 1.2.3 安装rpm和rpmbuild

```shell
yum install rpm
yum install rpm-build  
```

<div id ="124"></div>


##### 1.2.4 安装g++

```shell
yum install gcc-c++
```

<div id ="125"></div>


##### 1.2.5 检查python版本

**注意:ambari要求python的版本为2.6+**
一般使用系统自带的就OK

```shell
# 检查python版本,低于2.6的重新安装
python -V 
```

<div id ="126"></div>


##### 1.2.6 初始化python-devel

```shell
yum install python-devel
```

<div id ="127"></div>


##### 1.2.7 安装bower, gulp

```shell

# 附 npm命令安装方式: yum install npm
npm install -g bower
npm install -g gulp
```

<div id ="128"></div>


##### 1.2.8 安装git

```shell
yum install git
```

<div id ="129"></div>

##### 1.2.9 安装python setuptools

```shell
wget https://pypi.python.org/packages/45/29/8814bf414e7cd1031e1a3c8a4169218376e284ea2553cc0822a6ea1c2d78/setuptools-36.6.0.zip#md5=74663b15117d9a2cc5295d76011e6fd1

unzip setuptools-36.6.0.zip  # 附 unzip安装命令:yum install -y unzip zip
cd setuptools-36.6.0
python setup.py install
```


<div id ="1210"></div>

##### 1.2.10 设置最大打开文件数

linux系统默认最大打开文件数为1024
在源码编译过程中最大打开文件数可能超过该限制报错:

**too many files are open**而导致编译失败,所以需要修改最大打开文件数限制

```shell
ulimit -n 10000  
```



<div id ="2"></div>
   
### 二. 源码编译

<div id ="21"></div>

#### 2.1 配置阿里云远程仓库

为了能够提高编译过程中依赖包下载效率,maven可以配置阿里远程仓库
    
 
```shell
vi $MAVEN_HOME/conf/settings.xml
```

```xml

<!--在mirrors标签里添加mirror标签,内容如下:-->
    <mirror>
        <id>alimaven</id>
        <mirrorOf>central</mirrorOf>
        <name>aliyun maven</name>
        <url>http://maven.aliyun.com/nexus/content/repositories/central/</url>
    </mirror>

```

<div id ="22"></div>

#### 2.2 源码编译安装(以下命令需要在ambari源码包的根目录下执行)

```shell
# 设置ambari版本
mvn versions:set-property -Dproperty=revision -DnewVersion=2.6.0.0.0 
```

**执行源码编译命令前,请先阅读第三部分,遇到的问题及解决方案,以免踩坑**

 ```shell
 # -DskipTests参数是为了跳过单元测试,提高编译效率
 # -Drat.skip参数是为了跳过licensing 检查,防止报错Too many files with unapproved license
mvn -B clean install package rpm:rpm -DnewVersion=2.6.0.0.0 -DskipTests -Drat.skip -Dpython.ver="python >= 2.6" -Preplaceurl 
```

出现如下界面,证明源码编译成功!

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020091717055069.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDg2MTcwNw==,size_16,color_FFFFFF,t_70#pic_center)




<div id ="3"></div>


### 三. 遇到的问题及解决方案

<div id ="31"></div>

#### 3.1 Ambari Main编译报错:  RPM build execution returned: '127' executing

**报错详情:**

```shell
[INFO] Ambari Main ....................................... FAILURE [58:16.141s]

[ERROR] Failed to execute goal org.codehaus.mojo:rpm-maven-plugin:2.0.1:rpm (default-cli) on project ambari: RPM build execution returned: '127' executing '/bin/sh -c cd /opt/ambari/apache-ambari-2.6.0-src/target/rpm/ambari/SPECS && rpmbuild -bb --buildroot /opt/ambari/apache-ambari-2.6.0-src/target/rpm/ambari/buildroot --define '_topdir /opt/ambari/apache-ambari-2.6.0-src/target/rpm/ambari' --target noarch-redhat-linux ambari.spec' -> [Help 1]
[ERROR] 
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/MojoExecutionException
```

**报错原因:**: rpm,或者rpm-build没有安装



**解决方案:** 

```shell
# 安装rpm,rpm-build
yum install rpm
yum install rpm-build  
```

之后使用2.2的编译命令重新编译安装即可

**解决结果:**
```shell
# Ambari Main编译成功
[INFO] Ambari Main 2.6.0.0.0 .............................. SUCCESS [  0.824 s]
```


<div id ="32"></div>


#### 3.2 Ambari Web编译报错: requires Maven version 3.1.0

**报错详情:**

```shell
[INFO] Reactor Summary:
[INFO] 
[INFO] Ambari Main ....................................... SUCCESS [0.873s]
[INFO] Apache Ambari Project POM ......................... SUCCESS [0.121s]
[INFO] Ambari Web ........................................ FAILURE [37.713s]

[ERROR] Failed to execute goal com.github.eirslett:frontend-maven-plugin:1.4:install-node-and-yarn (install node and yarn) on project ambari-web: The plugin com.github.eirslett:frontend-maven-plugin:1.4 requires Maven version 3.1.0 -> [Help 1]
[ERROR] 
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/PluginIncompatibleException
[ERROR] 
[ERROR] After correcting the problems, you can resume the build with the command
[ERROR]   mvn <goals> -rf :ambari-web

```

**报错原因:** maven版本的问题, **ambari要求, maven版本最低为3.3.9**

我这里报该错误的原因是配置了高版本的maven环境,但是没有使用**source ~/.bashrc**命令使改动生效

```shell
mvn -version
Apache Maven 3.0.5 (Red Hat 3.0.5-17)
Maven home: /usr/share/maven
Java version: 1.8.0_262, vendor: Oracle Corporation
Java home: /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.262.b10-0.el7_8.x86_64/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "3.10.0-693.21.1.el7.x86_64", arch: "amd64", family: "unix"
```

**解决方案**

配置高版本maven,并使用**source ~/.bashrc**命令使改动生效

**解决结果:**

```shell
#Ambari Web编译成功
[INFO] Ambari Web ......................................... SUCCESS [ 55.379 s]
```

<div id ="33"></div>

#### 3.3 Ambari Admin View编译失败: Command execution failed. Process exited with an error: 1

**问题详情:**
```shell
error Unexpected token {

Stack trace:
SyntaxError: Unexpected token {
    at exports.runInThisContext (vm.js:53:16)
    at Module._compile (module.js:373:25)
    at Object.Module._extensions..js (module.js:416:10)
    at Module.load (module.js:343:32)
    at Function.Module._load (module.js:300:12)
    at Module.require (module.js:353:17)
    at require (internal/module.js:12:17)
    at Object.<anonymous> (/opt/ambari/apache-ambari-2.6.0-src/ambari-admin/src/main/resources/ui/admin-web/node_modules/bower/node_modules/bower-registry-client/node_modules/request/lib/cookies.js:3:13)
    at Module._compile (module.js:409:26)
    at Object.Module._extensions..js (module.js:416:10)

Console trace:
Trace
    at StandardRenderer.error (/opt/ambari/apache-ambari-2.6.0-src/ambari-admin/src/main/resources/ui/admin-web/node_modules/bower/lib/renderers/StandardRenderer.js:72:17)
    at Logger.<anonymous> (/opt/ambari/apache-ambari-2.6.0-src/ambari-admin/src/main/resources/ui/admin-web/node_modules/bower/bin/bower:111:22)
    at emitOne (events.js:77:13)
    at Logger.emit (events.js:169:7)
    at Logger.emit (/opt/ambari/apache-ambari-2.6.0-src/ambari-admin/src/main/resources/ui/admin-web/node_modules/bower/node_modules/bower-logger/lib/Logger.js:29:39)
    at /opt/ambari/apache-ambari-2.6.0-src/ambari-admin/src/main/resources/ui/admin-web/node_modules/bower/lib/commands/index.js:40:20
    at _rejected (/opt/ambari/apache-ambari-2.6.0-src/ambari-admin/src/main/resources/ui/admin-web/node_modules/bower/node_modules/q/q.js:797:24)
    at /opt/ambari/apache-ambari-2.6.0-src/ambari-admin/src/main/resources/ui/admin-web/node_modules/bower/node_modules/q/q.js:823:30
    at Promise.when (/opt/ambari/apache-ambari-2.6.0-src/ambari-admin/src/main/resources/ui/admin-web/node_modules/bower/node_modules/q/q.js:1035:31)
    at Promise.promise.promiseDispatch (/opt/ambari/apache-ambari-2.6.0-src/ambari-admin/src/main/resources/ui/admin-web/node_modules/bower/node_modules/q/q.js:741:41)

System info:
Bower version: 1.3.8
Node version: 4.5.0
OS: Linux 3.10.0-693.21.1.el7.x86_64 x64
[ERROR] Command execution failed.
org.apache.commons.exec.ExecuteException: Process exited with an error: 1 (Exit value: 1)
    at org.apache.commons.exec.DefaultExecutor.executeInternal (DefaultExecutor.java:404)
    at org.apache.commons.exec.DefaultExecutor.execute (DefaultExecutor.java:166)
    at org.codehaus.mojo.exec.ExecMojo.executeCommandLine (ExecMojo.java:982)
    at org.codehaus.mojo.exec.ExecMojo.executeCommandLine (ExecMojo.java:929)
    at org.codehaus.mojo.exec.ExecMojo.execute (ExecMojo.java:457)
    at org.apache.maven.plugin.DefaultBuildPluginManager.executeMojo (DefaultBuildPluginManager.java:137)
    at org.apache.maven.lifecycle.internal.MojoExecutor.execute (MojoExecutor.java:210)
    at org.apache.maven.lifecycle.internal.MojoExecutor.execute (MojoExecutor.java:156)
    at org.apache.maven.lifecycle.internal.MojoExecutor.execute (MojoExecutor.java:148)
    at org.apache.maven.lifecycle.internal.LifecycleModuleBuilder.buildProject (LifecycleModuleBuilder.java:117)
    at org.apache.maven.lifecycle.internal.LifecycleModuleBuilder.buildProject (LifecycleModuleBuilder.java:81)
    at org.apache.maven.lifecycle.internal.builder.singlethreaded.SingleThreadedBuilder.build (SingleThreadedBuilder.java:56)
    at org.apache.maven.lifecycle.internal.LifecycleStarter.execute (LifecycleStarter.java:128)
    at org.apache.maven.DefaultMaven.doExecute (DefaultMaven.java:305)
    at org.apache.maven.DefaultMaven.doExecute (DefaultMaven.java:192)
    at org.apache.maven.DefaultMaven.execute (DefaultMaven.java:105)
    at org.apache.maven.cli.MavenCli.execute (MavenCli.java:957)
    at org.apache.maven.cli.MavenCli.doMain (MavenCli.java:289)
    at org.apache.maven.cli.MavenCli.main (MavenCli.java:193)
    at sun.reflect.NativeMethodAccessorImpl.invoke0 (Native Method)
    at sun.reflect.NativeMethodAccessorImpl.invoke (NativeMethodAccessorImpl.java:62)
    at sun.reflect.DelegatingMethodAccessorImpl.invoke (DelegatingMethodAccessorImpl.java:43)
    at java.lang.reflect.Method.invoke (Method.java:498)
    at org.codehaus.plexus.classworlds.launcher.Launcher.launchEnhanced (Launcher.java:282)
    at org.codehaus.plexus.classworlds.launcher.Launcher.launch (Launcher.java:225)
    at org.codehaus.plexus.classworlds.launcher.Launcher.mainWithExitCode (Launcher.java:406)
    at org.codehaus.plexus.classworlds.launcher.Launcher.main (Launcher.java:347)

[INFO] Ambari Main 2.6.0.0.0 .............................. SUCCESS [  0.814 s]
[INFO] Apache Ambari Project POM 2.6.0.0.0 ................ SUCCESS [  0.131 s]
[INFO] Ambari Web 2.6.0.0.0 ............................... SUCCESS [03:07 min]
[INFO] Ambari Views 2.6.0.0.0 ............................. SUCCESS [08:32 min]
[INFO] Ambari Admin View 2.6.0.0.0 ........................ FAILURE [02:46 min]

[ERROR] Failed to execute goal org.codehaus.mojo:exec-maven-plugin:3.0.0:exec (Bower install) on project ambari-admin: Command execution failed. Process exited with an error: 1 (Exit value: 1) -> [Help 1]
[ERROR] 
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/MojoExecutionException
[ERROR] 
[ERROR] After correcting the problems, you can resume the build with the command
[ERROR]   mvn <args> -rf :ambari-admin
[ERROR] Failed to execute goal org.codehaus.mojo:exec-maven-plugin:3.0.0:exec (Bower install) on project ambari-admin: Command execution failed. Process exited with an error: 1 (Exit value: 1) -> [Help 1]
[ERROR] 
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/MojoExecutionException
[ERROR] 
[ERROR] After correcting the problems, you can resume the build with the command
[ERROR]   mvn <args> -rf :ambari-admin

```

**问题原因:**

ambari-admin/pom.xml里默认的配置存在一些问题
**nodeVersion,npmVersion**配置项与实际不符


默认配置如下图:
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200916201004516.png#pic_center)

使用如下命令查看npm和node的版本
```shell
npm -v  # 我的版本为3.10.10
node -v # 我的版本为v6.17.1
```

**解决办法:**

```xml
<!--修改ambari-admin模块下的pom.xml文件 -->
<nodeVersion>3.10.10</nodeVersion>
<npmVersion>v6.17.1</npmVersion>

```

**解决结果:**
```shell
# Ambari Admin View编译成功 
[INFO] Ambari Admin View .................................. SUCCESS [ 15.976 s]
```


<div id ="34"></div>

#### 3.4 Ambari Metrics Storm Sink编译失败: Could not resolve dependencies for project

**报错详情:**
```shell
[INFO] Ambari Main 2.6.0.0.0 .............................. SUCCESS [  0.816 s]
[INFO] Apache Ambari Project POM 2.6.0.0.0 ................ SUCCESS [  0.131 s]
[INFO] Ambari Web 2.6.0.0.0 ............................... SUCCESS [ 55.226 s]
[INFO] Ambari Views 2.6.0.0.0 ............................. SUCCESS [  0.957 s]
[INFO] Ambari Admin View 2.6.0.0.0 ........................ SUCCESS [01:18 min]
[INFO] utility 1.0.0.0-SNAPSHOT ........................... SUCCESS [02:47 min]
[INFO] ambari-metrics 2.6.0.0.0 ........................... SUCCESS [ 10.792 s]
[INFO] Ambari Metrics Common 2.6.0.0.0 .................... SUCCESS [02:56 min]
[INFO] Ambari Metrics Hadoop Sink 2.6.0.0.0 ............... SUCCESS [03:15 min]
[INFO] Ambari Metrics Flume Sink 2.6.0.0.0 ................ SUCCESS [01:38 min]
[INFO] Ambari Metrics Kafka Sink 2.6.0.0.0 ................ SUCCESS [05:19 min]
[INFO] Ambari Metrics Storm Sink 2.6.0.0.0 ................ FAILURE [ 22.982 s]
.......................................................................................................................
[ERROR] Failed to execute goal on project ambari-metrics-storm-sink: Could not resolve dependencies for project org.apache.ambari:ambari-metrics-storm-sink:jar:2.6.0.0.0: Could not find artifact org.apache.storm:storm-core:jar:1.1.0-SNAPSHOT in apache-hadoop (http://repo.hortonworks.com/content/groups/public/) -> [Help 1]
[ERROR] 
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/DependencyResolutionException
[ERROR] 
[ERROR] After correcting the problems, you can resume the build with the command
[ERROR]   mvn <args> -rf :ambari-metrics-storm-sink


```

**报错原因:**

ambari-metrics模块下,ambari-metrics-storm-sink部分的pom.xml文件里
storm的版本号配置有误,按照配置的版本号无法在仓库找到对应jar包

下图为其默认配置:

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200916201933512.png#pic_center)



**解决方案:**

修改**ambari-metrics**模块下,**ambari-metrics-storm-sink**部分的**pom.xml**文件,修改如下:

```xml
 <properties>
    <storm.version>1.1.0</storm.version>
  </properties>
```

**解决结果:**
```shell
# Ambari Metrics Storm Sink编译成功
[INFO] Ambari Metrics Storm Sink .......................... SUCCESS [  2.970 s]
[INFO] Ambari Metrics Storm Sink (Legacy) ................. SUCCESS [  2.400 s]
```


<div id ="35"></div>

#### 3.5 Ambari Server编译失败: Unable to parse configuration of mojo


**详细报错:**

```shell
[INFO] Downloaded from alimaven: http://maven.aliyun.com/nexus/content/repositories/central/org/codehaus/groovy/groovy-all/2.4.3/groovy-all-2.4.3.jar (7.0 MB at 2.9 MB/s)
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary:
[INFO] 
[INFO] Ambari Main 2.6.0.0.0 .............................. SUCCESS [  0.828 s]
[INFO] Apache Ambari Project POM 2.6.0.0.0 ................ SUCCESS [  0.126 s]
[INFO] Ambari Web 2.6.0.0.0 ............................... SUCCESS [ 55.103 s]
[INFO] Ambari Views 2.6.0.0.0 ............................. SUCCESS [  0.458 s]
[INFO] Ambari Admin View 2.6.0.0.0 ........................ SUCCESS [ 16.401 s]
[INFO] utility 1.0.0.0-SNAPSHOT ........................... SUCCESS [  0.079 s]
[INFO] ambari-metrics 2.6.0.0.0 ........................... SUCCESS [  0.319 s]
[INFO] Ambari Metrics Common 2.6.0.0.0 .................... SUCCESS [  5.443 s]
[INFO] Ambari Metrics Hadoop Sink 2.6.0.0.0 ............... SUCCESS [  2.458 s]
[INFO] Ambari Metrics Flume Sink 2.6.0.0.0 ................ SUCCESS [  2.046 s]
[INFO] Ambari Metrics Kafka Sink 2.6.0.0.0 ................ SUCCESS [  1.832 s]
[INFO] Ambari Metrics Storm Sink 2.6.0.0.0 ................ SUCCESS [  3.541 s]
[INFO] Ambari Metrics Storm Sink (Legacy) 2.6.0.0.0 ....... SUCCESS [  2.670 s]
[INFO] Ambari Metrics Collector 2.6.0.0.0 ................. SUCCESS [01:40 min]
[INFO] Ambari Metrics Monitor 2.6.0.0.0 ................... SUCCESS [  1.243 s]
[INFO] Ambari Metrics Grafana 2.1.0.0.0 ................... SUCCESS [  8.008 s]
[INFO] Ambari Metrics Assembly 2.6.0.0.0 .................. SUCCESS [01:01 min]
[INFO] Ambari Server 2.6.0.0.0 ............................ FAILURE [53:05 min]
[INFO] Ambari Functional Tests 2.6.0.0.0 .................. SKIPPED
[INFO] Ambari Agent 2.6.0.0.0 ............................. SKIPPED
[INFO] Ambari Client 2.6.0.0.0 ............................ SKIPPED
[INFO] Ambari Python Client 2.6.0.0.0 ..................... SKIPPED
[INFO] Ambari Groovy Client 2.6.0.0.0 ..................... SKIPPED
[INFO] Ambari Shell 2.6.0.0.0 ............................. SKIPPED
[INFO] Ambari Python Shell 2.6.0.0.0 ...................... SKIPPED
[INFO] Ambari Groovy Shell 2.6.0.0.0 ...................... SKIPPED
[INFO] ambari-logsearch 2.6.0.0.0 ......................... SKIPPED
[INFO] Ambari Logsearch Appender 2.6.0.0.0 ................ SKIPPED
[INFO] Ambari Logsearch Portal 2.6.0.0.0 .................. SKIPPED
[INFO] Ambari Logsearch Log Feeder 2.6.0.0.0 .............. SKIPPED
[INFO] Ambari Logsearch Solr Client 2.6.0.0.0 ............. SKIPPED
[INFO] Ambari Infra Solr Plugin 2.6.0.0.0 ................. SKIPPED
[INFO] Ambari Logsearch Assembly 2.6.0.0.0 ................ SKIPPED
[INFO] Ambari Logsearch Integration Test 2.6.0.0.0 ........ SKIPPED
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  57:28 min
[INFO] Finished at: 2020-09-16T21:18:21+08:00
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal org.codehaus.mojo:findbugs-maven-plugin:3.0.3:findbugs (findbugs) on project ambari-server: Unable to parse configuration of mojo org.codehaus.mojo:findbugs-maven-plugin:3.0.3:findbugs for parameter pluginArtifacts: Cannot assign configuration entry 'pluginArtifacts' with value '${plugin.artifacts}' of type java.util.Collections.UnmodifiableRandomAccessList to property of type java.util.ArrayList -> [Help 1]
[ERROR] 
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/PluginConfigurationException
[ERROR] 
[ERROR] After correcting the problems, you can resume the build with the command
[ERROR]   mvn <args> -rf :ambari-server

```

**问题原因:**

这个是maven导致的错误,可能的原因有两个:

1. 如果maven版本是3.5.4以上,应该是版本过高导致的
2. 还有一种可能就是Maven下载成功之后，修改了镜像或者是配置文件,有些jar包下载不下来导致的

**解决方案: 降低maven版本(我最初配置的是3.6.3版本), 在3.3.9到3.5.4版本之间选择一个**


配置之后的maven版本:
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200917092413637.png#pic_center)

**解决结果:**

```shell
# 不再报该错误,报错The entity name must immediately follow the '&' in the entity reference
# 该错误请参考3.6
```

<div id ="36"></div>

#### 3.6 Ambari Server编译失败: The entity name must immediately follow the '&' in the entity reference

**详细报错:**

```shell
[INFO] Reactor Summary:
[INFO] 
[INFO] Ambari Main 2.6.0.0.0 .............................. SUCCESS [  1.405 s]
[INFO] Apache Ambari Project POM .......................... SUCCESS [  0.131 s]
[INFO] Ambari Web ......................................... SUCCESS [ 55.516 s]
[INFO] Ambari Views ....................................... SUCCESS [  0.691 s]
[INFO] Ambari Admin View .................................. SUCCESS [ 17.159 s]
[INFO] utility 1.0.0.0-SNAPSHOT ........................... SUCCESS [  0.160 s]
[INFO] ambari-metrics ..................................... SUCCESS [  0.493 s]
[INFO] Ambari Metrics Common .............................. SUCCESS [  5.426 s]
[INFO] Ambari Metrics Hadoop Sink ......................... SUCCESS [  2.823 s]
[INFO] Ambari Metrics Flume Sink .......................... SUCCESS [  2.278 s]
[INFO] Ambari Metrics Kafka Sink .......................... SUCCESS [  1.986 s]
[INFO] Ambari Metrics Storm Sink .......................... SUCCESS [  2.937 s]
[INFO] Ambari Metrics Storm Sink (Legacy) ................. SUCCESS [  2.428 s]
[INFO] Ambari Metrics Collector ........................... SUCCESS [01:33 min]
[INFO] Ambari Metrics Monitor ............................. SUCCESS [  1.271 s]
[INFO] Ambari Metrics Grafana 2.1.0.0.0 ................... SUCCESS [  2.801 s]
[INFO] Ambari Metrics Assembly ............................ SUCCESS [01:14 min]
[INFO] Ambari Server ...................................... FAILURE [02:34 min]
[INFO] Ambari Functional Tests ............................ SKIPPED
[INFO] Ambari Agent ....................................... SKIPPED
[INFO] Ambari Client ...................................... SKIPPED
[INFO] Ambari Python Client ............................... SKIPPED
[INFO] Ambari Groovy Client ............................... SKIPPED
[INFO] Ambari Shell ....................................... SKIPPED
[INFO] Ambari Python Shell ................................ SKIPPED
[INFO] Ambari Groovy Shell ................................ SKIPPED
[INFO] ambari-logsearch ................................... SKIPPED
[INFO] Ambari Logsearch Appender .......................... SKIPPED
[INFO] Ambari Logsearch Portal ............................ SKIPPED
[INFO] Ambari Logsearch Log Feeder ........................ SKIPPED
[INFO] Ambari Logsearch Solr Client ....................... SKIPPED
[INFO] Ambari Infra Solr Plugin ........................... SKIPPED
[INFO] Ambari Logsearch Assembly .......................... SKIPPED
[INFO] Ambari Logsearch Integration Test 2.6.0.0.0 ........ SKIPPED
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 07:00 min
[INFO] Finished at: 2020-09-17T09:57:12+08:00
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal org.codehaus.mojo:xml-maven-plugin:1.0:transform (default) on project ambari-server: Failed to transform input file /opt/ambari/apache-ambari-2.6.0-src/ambari-server/target/findbugs/findbugsXml.html: The entity name must immediately follow the '&' in the entity reference. -> [Help 1]
[ERROR] 
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/MojoExecutionException
[ERROR] 
[ERROR] After correcting the problems, you can resume the build with the command
[ERROR]   mvn <goals> -rf :ambari-server

```

**问题原因:**

从报错来看是&这个符号使用的问题,导致文件编译出现问题

具体的原因可参考stackoverflow的这篇问答:

[The entity name must immediately follow the '&' in the entity reference](https://stackoverflow.com/questions/16303779/the-entity-name-must-immediately-follow-the-in-the-entity-reference)


**解决方案:**
因为报错的findbugsXml.html文件内容过多,找出文件问题并改正成本太高,所以选择了**修改ambari-server的pom.xml文件,将findbugs部分内容注释掉:**


需要注释部分的原文:
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200917100939479.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDg2MTcwNw==,size_16,color_FFFFFF,t_70#pic_center)

修改之后:
```xml
      <executions>
          <!-- <execution>
            <phase>verify</phase>
            <goals> 
              <goal>check</goal>
            </goals>
          </execution> -->
        </executions>
      </plugin>
      <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>xml-maven-plugin</artifactId>
        <version>1.0</version>
        <executions>
          <!--    
           <execution>
            <phase>verify</phase>
            <goals> 
              <goal>transform</goal>
            </goals>
          </execution>
          -->     
        </executions>
```

**解决结果**
```shell
# 该问题得到解决,编译时抛出了新的错误:
Failed to execute goal org.codehaus.mojo:rpm-maven-plugin:2.1.4:rpm (default-cli) on project ambari-server

# 该问题的处理方法请参照3.7
```

<div id ="37"></div>

#### 3.7 Ambari Server编译失败: Failed to execute goal org.codehaus.mojo:rpm-maven-plugin:2.1.4:rpm (default-cli) on project ambari-server


```shell
[INFO] Bytecompiling .py files below /opt/ambari/apache-ambari-2.6.0-src/ambari-server/target/rpm/ambari-server/buildroot/usr/lib/python2.6 using /usr/bin/python2.6
[INFO] /usr/lib/rpm/brp-python-bytecompile: line 38: /usr/bin/python2.6: No such file or directory
[INFO] error: Bad exit status from /var/tmp/rpm-tmp.c817Ww (%install)
[INFO] 
[INFO] 
[INFO] RPM build errors:
[INFO]     Bad exit status from /var/tmp/rpm-tmp.c817Ww (%install)
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary:
[INFO] 
[INFO] Ambari Main 2.6.0.0.0 .............................. SUCCESS [  0.780 s]
[INFO] Apache Ambari Project POM .......................... SUCCESS [  0.132 s]
[INFO] Ambari Web ......................................... SUCCESS [ 55.137 s]
[INFO] Ambari Views ....................................... SUCCESS [  0.432 s]
[INFO] Ambari Admin View .................................. SUCCESS [ 16.003 s]
[INFO] utility 1.0.0.0-SNAPSHOT ........................... SUCCESS [  0.076 s]
[INFO] ambari-metrics ..................................... SUCCESS [  0.287 s]
[INFO] Ambari Metrics Common .............................. SUCCESS [  4.914 s]
[INFO] Ambari Metrics Hadoop Sink ......................... SUCCESS [  2.738 s]
[INFO] Ambari Metrics Flume Sink .......................... SUCCESS [  2.263 s]
[INFO] Ambari Metrics Kafka Sink .......................... SUCCESS [  1.938 s]
[INFO] Ambari Metrics Storm Sink .......................... SUCCESS [  2.701 s]
[INFO] Ambari Metrics Storm Sink (Legacy) ................. SUCCESS [  2.364 s]
[INFO] Ambari Metrics Collector ........................... SUCCESS [01:35 min]
[INFO] Ambari Metrics Monitor ............................. SUCCESS [  1.146 s]
[INFO] Ambari Metrics Grafana 2.1.0.0.0 ................... SUCCESS [  8.122 s]
[INFO] Ambari Metrics Assembly ............................ SUCCESS [01:24 min]
[INFO] Ambari Server ...................................... FAILURE [01:15 min]
[INFO] Ambari Functional Tests ............................ SKIPPED
[INFO] Ambari Agent ....................................... SKIPPED
[INFO] Ambari Client ...................................... SKIPPED
[INFO] Ambari Python Client ............................... SKIPPED
[INFO] Ambari Groovy Client ............................... SKIPPED
[INFO] Ambari Shell ....................................... SKIPPED
[INFO] Ambari Python Shell ................................ SKIPPED
[INFO] Ambari Groovy Shell ................................ SKIPPED
[INFO] ambari-logsearch ................................... SKIPPED
[INFO] Ambari Logsearch Appender .......................... SKIPPED
[INFO] Ambari Logsearch Portal ............................ SKIPPED
[INFO] Ambari Logsearch Log Feeder ........................ SKIPPED
[INFO] Ambari Logsearch Solr Client ....................... SKIPPED
[INFO] Ambari Infra Solr Plugin ........................... SKIPPED
[INFO] Ambari Logsearch Assembly .......................... SKIPPED
[INFO] Ambari Logsearch Integration Test 2.6.0.0.0 ........ SKIPPED
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 05:55 min
[INFO] Finished at: 2020-09-17T10:53:16+08:00
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal org.codehaus.mojo:rpm-maven-plugin:2.1.4:rpm (default-cli) on project ambari-server: RPM build execution returned: '1' executing '/bin/sh -c cd '/opt/ambari/apache-ambari-2.6.0-src/ambari-server/target/rpm/ambari-server/SPECS' && 'rpmbuild' '-bb' '--target' 'x86_64-redhat-linux' '--buildroot' '/opt/ambari/apache-ambari-2.6.0-src/ambari-server/target/rpm/ambari-server/buildroot' '--define' '_topdir /opt/ambari/apache-ambari-2.6.0-src/ambari-server/target/rpm/ambari-server' '--define' '_build_name_fmt %%{ARCH}/%%{NAME}-%%{VERSION}-%%{RELEASE}.%%{ARCH}.rpm' '--define' '_builddir %{_topdir}/BUILD' '--define' '_rpmdir %{_topdir}/RPMS' '--define' '_sourcedir %{_topdir}/SOURCES' '--define' '_specdir %{_topdir}/SPECS' '--define' '_srcrpmdir %{_topdir}/SRPMS' 'ambari-server.spec'' -> [Help 1]
[ERROR] 
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/MojoExecutionException
[ERROR] 
[ERROR] After correcting the problems, you can resume the build with the command
[ERROR]   mvn <goals> -rf :ambari-server

```

**问题原因:**

开始看到报错信息有**rpm**,认为是**rpm,或者rpm-build没有安装**导致的,使用命令行查看rpm,rpm-build都是正常的

后面在报错信息上面的INFO日志中看到如下信息:
**/usr/lib/rpm/brp-python-bytecompile: line 38: /usr/bin/python2.6: No such file or directory**

推测是该问题导致的编译失败

**解决方案**

因为装的是python2.7，有个python2.7的文件，所以建立软连接指向该文件

```shell
# 建立软连接指向/usr/bin/python2.6
ln -s /usr/bin/python2.7 /usr/bin/python2.6
```
之后重新编译


**解决结果:**
```shell
# Ambari Server终于编译成功!
[INFO] Ambari Server ...................................... SUCCESS [02:30 min]
```

<div id ="38"></div>

#### 3.8 Ambari Groovy Shell编译失败: Could not resolve dependencies for project org.apache.ambari:ambari-groovy-shell:jar:2.6.0.0.0

```shell
[INFO] Reactor Summary:
[INFO] 
[INFO] Ambari Main 2.6.0.0.0 .............................. SUCCESS [  0.810 s]
[INFO] Apache Ambari Project POM .......................... SUCCESS [  0.134 s]
[INFO] Ambari Web ......................................... SUCCESS [ 55.390 s]
[INFO] Ambari Views ....................................... SUCCESS [  0.421 s]
[INFO] Ambari Admin View .................................. SUCCESS [ 16.168 s]
[INFO] utility 1.0.0.0-SNAPSHOT ........................... SUCCESS [  0.094 s]
[INFO] ambari-metrics ..................................... SUCCESS [  0.287 s]
[INFO] Ambari Metrics Common .............................. SUCCESS [  5.606 s]
[INFO] Ambari Metrics Hadoop Sink ......................... SUCCESS [  2.669 s]
[INFO] Ambari Metrics Flume Sink .......................... SUCCESS [  2.226 s]
[INFO] Ambari Metrics Kafka Sink .......................... SUCCESS [  1.962 s]
[INFO] Ambari Metrics Storm Sink .......................... SUCCESS [  2.767 s]
[INFO] Ambari Metrics Storm Sink (Legacy) ................. SUCCESS [  2.400 s]
[INFO] Ambari Metrics Collector ........................... SUCCESS [01:31 min]
[INFO] Ambari Metrics Monitor ............................. SUCCESS [  1.156 s]
[INFO] Ambari Metrics Grafana 2.1.0.0.0 ................... SUCCESS [  7.677 s]
[INFO] Ambari Metrics Assembly ............................ SUCCESS [01:13 min]
[INFO] Ambari Server ...................................... SUCCESS [02:30 min]
[INFO] Ambari Functional Tests ............................ SUCCESS [  1.422 s]
[INFO] Ambari Agent ....................................... SUCCESS [02:05 min]
[INFO] Ambari Client ...................................... SUCCESS [  0.136 s]
[INFO] Ambari Python Client ............................... SUCCESS [  1.058 s]
[INFO] Ambari Groovy Client ............................... SUCCESS [ 13.309 s]
[INFO] Ambari Shell ....................................... SUCCESS [  0.133 s]
[INFO] Ambari Python Shell ................................ SUCCESS [  0.672 s]
[INFO] Ambari Groovy Shell ................................ FAILURE [04:39 min]
[INFO] ambari-logsearch ................................... SKIPPED
[INFO] Ambari Logsearch Appender .......................... SKIPPED
[INFO] Ambari Logsearch Portal ............................ SKIPPED
[INFO] Ambari Logsearch Log Feeder ........................ SKIPPED
[INFO] Ambari Logsearch Solr Client ....................... SKIPPED
[INFO] Ambari Infra Solr Plugin ........................... SKIPPED
[INFO] Ambari Logsearch Assembly .......................... SKIPPED
[INFO] Ambari Logsearch Integration Test 2.6.0.0.0 ........ SKIPPED
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 13:58 min
[INFO] Finished at: 2020-09-17T11:52:52+08:00
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal on project ambari-groovy-shell: Could not resolve dependencies for project org.apache.ambari:ambari-groovy-shell:jar:2.6.0.0.0: Failed to collect dependencies at org.springframework.shell:spring-shell:jar:1.1.0.RC3: Failed to read artifact descriptor for org.springframework.shell:spring-shell:jar:1.1.0.RC3: Could not transfer artifact org.springframework.shell:spring-shell:pom:1.1.0.RC3 from/to spring-milestones (http://repo.spring.io/milestone): Access denied to: http://repo.spring.io/milestone/org/springframework/shell/spring-shell/1.1.0.RC3/spring-shell-1.1.0.RC3.pom , ReasonPhrase:Forbidden. -> [Help 1]
[ERROR] 
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/DependencyResolutionException
[ERROR] 
[ERROR] After correcting the problems, you can resume the build with the command
[ERROR]   mvn <goals> -rf :ambari-groovy-shell
```

**报错原因:**

这种报错是在jar包下载过程中出现的问题

类似的报错,主要有以下几种可能性
可以参照报错提示给出的链接:
[DependencyResolutionException](https://cwiki.apache.org/confluence/display/MAVEN/DependencyResolutionException)




首先根据报错提示访问了**Access denied to:** 指向的链接

界面显示如下:

![在这里插入图片描述](https://img-blog.csdnimg.cn/202009171551573.png#pic_center)

去maven本地仓库查看该jar包,果然有问题:

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200917155805283.png#pic_center)

到这里,开始怀疑是 **ambari-shell/ambari-groovy-shell**模块的pom.xml文件文件有问题
发现该配置文件下和报错有关的配置如下:
```xml
 <dependency>
      <groupId>org.springframework.shell</groupId>
      <artifactId>spring-shell</artifactId>
    </dependency>
```



根据关键字**spring-shell**去maven资料库查询,maven资料库链接如下:

[maven资料库查询链接](https://mvnrepository.com/artifact/org.springframework.shell/spring-shell/1.1.0.RELEASE)


发现**spring-shell**没有对应于上述pom.xml文件的配置

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200917160949127.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDg2MTcwNw==,size_16,color_FFFFFF,t_70#pic_center)

**解决方案:**
```shell
 vi ambari-shell/ambari-groovy-shell/pom.xml

```
修改spring-shell部分配置为如下:
```xml
 <dependency>
      <groupId>org.springframework.shell</groupId>
      <artifactId>spring-shell</artifactId>
      <version>1.1.0.RELEASE</version>
    </dependency>
```
修改之后重新编译

**解决结果:**
```shell
#  Ambari Groovy Shell编译成功
[INFO] Ambari Groovy Shell ................................ SUCCESS [07:49 min]

```
<div id = 39></div>

#### 3.9 Ambari Web编译失败
 Failed to run task: 'yarn install --ignore-engines --pure-lockfile' failed
 
 详细报错:
```
[INFO] Ambari Web ......................................... FAILURE [02:07 min]
....
[INFO] Ambari Infra Manager Integration Tests 2.7.5.0.0 ... SKIPPED
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 02:09 min
[INFO] Finished at: 2021-04-23T14:19:59+08:00
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal com.github.eirslett:frontend-maven-plugin:1.4:yarn (yarn install) on project ambari-web: Failed to run task: 'yarn install --ignore-engines --pure-lockfile' failed. org.apache.commons.exec.ExecuteException: Process exited with an error: 1 (Exit value: 1) -> [Help 1]
[ERROR]
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR]
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/MojoFailureException
[ERROR]
[ERROR] After correcting the problems, you can resume the build with the command
[ERROR]   mvn <goals> -rf :ambari-web
```

问题原因:nodejs会通过 yarn去加载依赖包，
解决方式是保证网络环境好

<div id ="310"></div>

#### 3.10 Ambari Admin View编译失败: Failed to run task: 'npm install --unsafe-perm' failed

详细报错如下:
```
[INFO] Ambari Admin View .................................. FAILURE [08:28 min]
.....
[INFO] Ambari Infra Manager Integration Tests 2.7.5.0.0 ... SKIPPED
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 10:11 min
[INFO] Finished at: 2021-04-23T14:45:59+08:00
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal com.github.eirslett:frontend-maven-plugin:1.3:npm (npm install) on project ambari-admin: Failed to run task: 'npm install --unsafe-perm' failed. (error code 1) -> [Help 1]
[ERROR]
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR]
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/MojoFailureException
[ERROR]
[ERROR] After correcting the problems, you can resume the build with the command
[ERROR]   mvn <goals> -rf :ambari-admin

```

解决方案: 

[Failed to run task: 'npm install --unsafe-perm' failed](https://community.cloudera.com/t5/Support-Questions/Ambari-2-7-5-installation-fail/td-p/308784)

修改.bowerrc文件
```shell
vim /opt/modules/ambari/apache-ambari-2.7.5-src/ambari-admin/src/main/resources/ui/admin-web/.bowerrc
```
添加一行:**"allow_root": true**

修改后的文件内容如下:
```
{
    "directory": "app/bower_components",
    "allow_root": true
}

```



文件修改好之后,执行命令:
```
find ~/.m2/repository/ -name "*.lastUpdated" -exec rm -rf {} \;
```


之后重新编译

<div id ="311"></div>

#### 3.11 Ambari Admin View编译失败: ECMDERR Failed to execute "git ls-remote --tags --heads


编译中抛出错误: 
```
bower angular#*                         
ECMDERR Failed to execute "git ls-remote --tags --heads https://github.com/angular/bower-angular.git", exit code of #128

```
之后编译失败:

```
[INFO] Ambari Views ....................................... SUCCESS [  0.933 s]
[INFO] Ambari Admin View .................................. FAILURE [ 41.829 s]
...
[INFO] Ambari Infra Manager Integration Tests 2.7.5.0.0 ... SKIPPED
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 02:21 min
[INFO] Finished at: 2021-04-25T10:03:28+08:00
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal org.codehaus.mojo:exec-maven-plugin:1.2.1:exec (Bower install) on project ambari-admin: Command execution failed. Process exited with an error: 1 (Exit value: 1) -> [Help 1]
```
**解决方案:**
用https的协议去访问和下载，而不用bower默认的git协议</br>
对全局的git做如下的配置：
```
git config --global url."https://".insteadOf git://
```
之后重新编译:

在执行bower install过程中,会遇到诸如timeout之类的问题,这种问题是由于网络原因导致的,我执行了很多遍都没有成功.最后通过配置代理解决了这个问题

代理配置方法:
```
export http_proxy=""
export https_proxy=""
```

<div id ="312"></div>

####　3.12  Ambari Metrics Collector编译失败 An Ant BuildException has occured:

详细报错:
```
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary:
[INFO]
[INFO] Ambari Main 2.7.5.0.0 .............................. SUCCESS [  1.332 s]
...
[INFO] Ambari Metrics Collector ........................... FAILURE [06:00 min]
...
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 40:02 min
[INFO] Finished at: 2021-05-07T11:02:22+08:00
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-antrun-plugin:1.7:run (default) on project ambari-metrics-timelineservice: An Ant BuildException has occured: Can't get https://s3.amazonaws.com/dev.hortonworks.com/HDP/centos7/3.x/BUILDS/3.1.4.0-315/tars/hbase/hbase-2.0.2.3.1.4.0-315-bin.tar.gz to /opt/modules/apache-ambari-2.7.5-src/ambari-metrics/ambari-metrics-timelineservice/target/embedded/hbase.tar.gz
[ERROR] around Ant part ...<get usetimestamp="true" src="https://s3.amazonaws.com/dev.hortonworks.com/HDP/centos7/3.x/BUILDS/3.1.4.0-315/tars/hbase/hbase-2.0.2.3.1.4.0-315-bin.tar.gz" dest="/opt/modules/apache-ambari-2.7.5-src/ambari-metrics/ambari-metrics-timelineservice/target/embedded/hbase.tar.gz"/>... @ 5:280 in /opt/modules/apache-ambari-2.7.5-src/ambari-metrics/ambari-metrics-timelineservice/target/antrun/build-Download HBase.xml
```
原因: ambari2.7.5编译的时候,需要下载4个大的依赖包,下载过程中很容易因为网络问题失败,分别是:

    
```
hbase jar包: hbase-2.0.2.3.1.4.0-315-bin.tar.gz
grafana jar包:grafana-6.4.2.linux-amd64.tar.gz
hadoop jar包: hadoop-3.1.1.3.1.4.0-315.tar.gz
phoenix jar包:phoenix-5.0.0.3.1.4.0-315.tar.gz
```
**解决方案:** 提前把这几个包下载好
并修改:ambari-metrics/pom.xml文件,将这四个包的下载路径指向本地文件所在路径即可

如下图:

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210507144859806.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDg2MTcwNw==,size_16,color_FFFFFF,t_70)

如下图: Ambari-Server编译成功

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210507154234633.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDg2MTcwNw==,size_16,color_FFFFFF,t_70)


<div id ="4"></div>

### 四. 安装配置Ambari

<div id ="41"></div>


#### 4.1 安装配置Ambari Server
  

执行安装命令前,首先修改**ambari-server**文件
```shell
vi /usr/sbin/ambari-server
```
将文件标红部分,如下图:
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200917173523588.png#pic_center)
修改为:
```shell
HASH="${VERSION}"
```
这是为了防止执行**ambari-server setup**出现问题:
**/usr/sbin/ambari-server: line 34: buildNumber: unbound variable**

**安装ambari-server**

```shell

yum install ambari-server/target/rpm/ambari-server/RPMS/x86_64/ambari-server*.rpm
```
```shell
ambari-server setup
```
注意:**设置jdk时,应该选择3,然后输入1.2.1过程中设置的JAVA_HOME路径**

执行情况如下:
```shell
Using python  /usr/bin/python
Setup ambari-server
Checking SELinux...
SELinux status is 'disabled'
Customize user account for ambari-server daemon [y/n] (n)? n
Adjusting ambari-server permissions and ownership...
Checking firewall status...
Checking JDK...
[1] Oracle JDK 1.8 + Java Cryptography Extension (JCE) Policy Files 8
[2] Oracle JDK 1.7 + Java Cryptography Extension (JCE) Policy Files 7
[3] Custom JDK
==============================================================================
Enter choice (1): 3
WARNING: JDK must be installed on all hosts and JAVA_HOME must be valid on all hosts.
WARNING: JCE Policy files are required for configuring Kerberos security. If you plan to use Kerberos,please make sure JCE Unlimited Strength Jurisdiction Policy Files are valid on all hosts.
Path to JAVA_HOME: /opt/modules/jdk1.8.0_151
Validating JDK on Ambari Server...done.
Completing setup...
Configuring database...
Enter advanced database configuration [y/n] (n)? n
Configuring database...
Default properties detected. Using built-in database.
Configuring ambari database...
Checking PostgreSQL...
Running initdb: This may take up to a minute.
Initializing database ... OK


About to start PostgreSQL
Configuring local database...
Configuring PostgreSQL...
Restarting PostgreSQL
Creating schema and user...
done.
Creating tables...
done.
Extracting system views...
ambari-admin-2.6.0.0.0.jar

Adjusting ambari-server permissions and ownership...
Ambari Server 'setup' completed successfully.

```




**启动ambari server**
```shell
ambari-server start
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200917174450125.png#pic_center)

成功启动后在浏览器输入Ambari地址：
http://ip:<port_number> 

注意: **port_number默认8080**,我这里设置的8083,**页面访问的账户名和密码都是admin**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200917180418643.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDg2MTcwNw==,size_16,color_FFFFFF,t_70#pic_center)


<div id ="42"></div>


#### 4.2 安装配置Ambari Agent

**安装ambari-agent**

```shell
yum install ambari-agent/target/rpm/ambari-agent/RPMS/x86_64/ambari-agent-*.rpm
```

启动ambari-agent之前,首先修改该文件,
防止出现类似4.1中类似ambari-server出现的错误
```shell
vi /var/lib/ambari-agent/bin/ambari-agent
# 将HASH="${buildNumber}"修改为:  HASH="${VERSION}"
```

**编辑ambari server的位置**
```shell
vi /etc/ambari-agent/conf/ambari-agent.ini
# 修改hostname一行为ambari-server对应的主机名

# 在security下面添加如下行:force_https_protocol=PROTOCOL_TLSv1_2,确保有下图所示的两个配置
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918145547598.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDg2MTcwNw==,size_16,color_FFFFFF,t_70#pic_center)

**启动ambari-agent**
```shell
ambari-agent start
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200920091219703.png#pic_center)




如果有异常请查看日志:**/var/log/ambari-agent/ambari-agent.log**


<div id ="43"></div>

### 4.3 修改yum源,离线安装HDP及HDP-UTILS

**安装httpd服务（主服务器）**
```shell
yum -y install httpd
service httpd restart
chkconfig httpd on
```



**将HDP,HDP-UTILS安装包放到/var/www/html目录下**



[HDP-2.6.3.0版本下载链接](http://public-repo-1.hortonworks.com/HDP/centos7/2.x/updates/2.6.3.0/HDP-2.6.3.0-centos7-rpm.tar.gz)

[HDP-UTILS-1.1.0.21版本下载链接](http://public-repo-1.hortonworks.com/HDP-UTILS-1.1.0.21/repos/centos7/HDP-UTILS-1.1.0.21-centos7.tar.gz)

```shell
# 将从上述下载链接下载的文件拷贝到该目录下
mkdir -p /var/www/html/ambari
cd /var/www/html/ambari
tar -zxvf HDP-2.6.3.0-centos7-rpm.tar.gz 
mkdir HDP-UTILS
tar -zxvf HDP-UTILS-1.1.0.21-centos7.tar.gz -C HDP-UTILS
rm -rf  HDP-2.6.3.0-centos7-rpm.tar.gz HDP-UTILS-1.1.0.21-centos7.tar.gz
```

现在可以通过http://ip/ambari/查看能否成功访问

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200920090731192.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDg2MTcwNw==,size_16,color_FFFFFF,t_70#pic_center)

**制作本地源**
1. 安装本地源制作相关工具
```shell
yum install yum-utils createrepo yum-plugin-priorities -y
createrepo  ./
```
2. 修改文件里面的源地址
```shell
vi HDP/centos7/2.6.3.0-235/hdp.repo
```
修改如下:
```shell
#VERSION_NUMBER=2.6.3.0-235
[HDP-2.6.3.0]
name=HDP Version - HDP-2.6.3.0
baseurl=http://ip/ambari/HDP/centos7/2.6.3.0-235
gpgcheck=1
gpgkey=http://ip/ambari/HDP/centos7/2.6.3.0-235/RPM-GPG-KEY/RPM-GPG-KEY-Jenkins
enabled=1
priority=1


[HDP-UTILS-1.1.0.21]
name=HDP-UTILS Version - HDP-UTILS-1.1.0.21
baseurl=http://ip/ambari/HDP-UTILS
gpgcheck=1
gpgkey=http://ip/ambari/HDP-UTILS/RPM-GPG-KEY/RPM-GPG-KEY-Jenkins
enabled=1
priority=1

```
注意: 修改时需要将我这里的**ip**换为自己实际ip地址

3. yum缓存
```shell
yum clean all
yum makecache
yum repolist
```

#### 4.4 安装部署hdp

1. 登录
默认管理员登录,账户密码均为**admin**
2. 安装
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020092009484922.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDg2MTcwNw==,size_16,color_FFFFFF,t_70#pic_center)

**Get Started配置集群名字**

**选择版本并修改为本地源地址**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200920095208322.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDg2MTcwNw==,size_16,color_FFFFFF,t_70#pic_center)

**Install Options配置主机名和ssh秘钥文件id_rsa**

注意:秘钥文件在~/.ssh目录下

后续部分按照页面提示一一配置即可~
具体细节可以参照博客:

[Ambari2.6.0安装HDP2.6.3链接](https://www.cnblogs.com/zhang-ke/p/8944240.html)

遇到问题可参考我的博客:

[Ambari踩过的坑及解决方案](https://blog.csdn.net/weixin_40861707/article/details/108350022)


<div id ="5"></div>

### 五. ambari2.6.0安装配置遇到的问题及解决方案

<div id ="51"></div>

#### 5.1 ambari server启动报错: java.net.BindException: Address already in use


**详细报错:**
```shell
# 查看server日志
cat /var/log/ambari-server/ambari-server.log
```
```
17 Sep 2020 17:44:38,714 ERROR [main] AmbariServer:1073 - Failed to run the Ambari Server
java.net.BindException: Address already in use
	at sun.nio.ch.Net.bind0(Native Method)
	at sun.nio.ch.Net.bind(Net.java:433)
	at sun.nio.ch.Net.bind(Net.java:425)
	at sun.nio.ch.ServerSocketChannelImpl.bind(ServerSocketChannelImpl.java:223)
	at sun.nio.ch.ServerSocketAdaptor.bind(ServerSocketAdaptor.java:74)
	at org.eclipse.jetty.server.nio.SelectChannelConnector.open(SelectChannelConnector.java:187)
	at org.eclipse.jetty.server.AbstractConnector.doStart(AbstractConnector.java:316)
	at org.eclipse.jetty.server.nio.SelectChannelConnector.doStart(SelectChannelConnector.java:265)
	at org.eclipse.jetty.util.component.AbstractLifeCycle.start(AbstractLifeCycle.java:64)
	at org.eclipse.jetty.server.Server.doStart(Server.java:293)
	at org.eclipse.jetty.util.component.AbstractLifeCycle.start(AbstractLifeCycle.java:64)
	at org.apache.ambari.server.controller.AmbariServer.run(AmbariServer.java:558)
	at org.apache.ambari.server.controller.AmbariServer.main(AmbariServer.java:1067)
```

**报错原因:**

    8080端口号被占用
    
如图,端口号被k8s应用占用
    
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200917180032567.png#pic_center)

**解决办法:**

1. 杀掉占用8080端口号的进程
2. 更换ambari-server端口号,我这里选用的是第二种

```shell
# 修改ambari配置文件
vi /etc/ambari-server/conf/ambari.properties
```
```shell
# 增加这一行
client.api.port=<port_number>
```
执行结束之后,重启ambari-server
```shell
ambari-server start
```


**解决结果:**

如图,界面可以访问!

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200917180418643.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDg2MTcwNw==,size_16,color_FFFFFF,t_70#pic_center)


<div id ="52"></div>

#### 5.2 ambari agent启动报错: SSLError: Failed to connect. Please check openssl library versions

**报错详情:agent启动后无法注册成为ambari节点,界面显示Fail**

日志如下:

```shell
cat /var/log/ambari-agent/ambari-agent.log 
```
```shell
WARNING 2020-09-18 14:46:52,865 NetUtil.py:124 - Server at https://hdpamb154:8440 is not reachable, sleeping for 10 seconds...
INFO 2020-09-18 14:47:02,865 NetUtil.py:70 - Connecting to https://hdpamb154:8440/ca
ERROR 2020-09-18 14:47:02,867 NetUtil.py:96 - EOF occurred in violation of protocol (_ssl.c:618)
ERROR 2020-09-18 14:47:02,867 NetUtil.py:97 - SSLError: Failed to connect. Please check openssl library versions. 
Refer to: https://bugzilla.redhat.com/show_bug.cgi?id=1022468 for more details.
```

**报错原因:**
https连接失败

**解决方案:**
1. 首先按照提示检查openssl版本
```shell
openssl version # 如果低于 openssl-1.0.1e-16.el6.x86_64 版本，则需要更新到 openssl-1.0.1e-16.el6.x86_64 及以上版本
```
2.将 [https] 节的 verify 项设为禁用
```
vi /etc/python/cert-verification.cfg

# 在配置中更改下列参数
[https]
verify=disable
```
3. 编辑 /etc/ambari-agent/conf/ambari-agent.ini 配置文件，在 [security] 节部分，确保设置如下两个值，其它值保持不变：
```shell
vi /etc/ambari-agent/conf/ambari-agent.ini


[security]
ssl_verify_cert=0
force_https_protocol=PROTOCOL_TLSv1_2
```

之后重启ambari-agent
```shell
ambari-agent restart
```

**解决详情:**

节点注册成功
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918145418858.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDg2MTcwNw==,size_16,color_FFFFFF,t_70#pic_center)


ambari-server编译卡住问题
添加-o命令

https://www.itranslater.com/qa/details/2124842491173143552

