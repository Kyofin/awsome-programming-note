# 统一配置

账号密码root/19950721

都有oh-my-zsh，autojump

互相免密登录

/etc/hosts都设置了各个节点的host









# 主机vwware

## 安装mysql

mysql5.7，安装参考<https://www.jianshu.com/p/7cccdaa2d177>

账号 root

密码 MyNewPass4!



账号 temp **(用于临时创建CM Server数据库)**

密码 MyNewPass4!

```shell
# root @ vmware in /opt/cloudera-manager/cm-5.13.3/share/cmf/schema [19:49:57]
$ ./scm_prepare_database.sh mysql  -h vmware -utemp -pMyNewPass4! --scm-host vmware scm scm scm
```



账号scm

密码sc

访问数据库scm



## 安装jdk

用rpm安装官网下载的rpm安装包。

目录 /usr/java/jdk1.8.0_211-amd64

## 创建cm用户

```
useradd --system --no-create-home --shell=/bin/false --comment "Cloudera SCM User" cloudera-scm
```







# 主机dell

## 安装jdk

目录

/usr/java/jdk1.8.0_211-amd64

## 创建cm用户

```
useradd --system --no-create-home --shell=/bin/false --comment "Cloudera SCM User" cloudera-scm
```

# 





# 主机vmware2

## 安装jdk

目录

/usr/java/jdk1.8.0_211-amd64

## 创建cm用户

```
useradd --system --no-create-home --shell=/bin/false --comment "Cloudera SCM User" cloudera-scm
```

# 