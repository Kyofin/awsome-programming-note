## 更新yum 

```
 yum -y update && yum -y upgrade 
```
## 新系统可能只能用ip addr看ip，安装下面ifconfig就可以用了 

```
yum install net-tools -y 
```
## yum没可用安装包时可以使用 

```
yum -y install epel-release 
```
## 网络流量监听工具 

```
yum install -y iftop 
```
## netcat网络数据交流工具 

```
yum install -y nc 
```
## vim 

```
yum install vim -y 
```
## vimrc配置 

vim  ~/.vimrc 

```
syntax enable                  "语法高亮 
syntax on 
set runtimepath+=~/.vim_runtime 
set nocompatible               " 去掉有关vi一致性模式，避免以前版本的bug和局限 
set nu!                        " 显示行号 
set history=1000               " 记录历史的行数 
set autoindent                 " vim使用自动对齐，也就是把当前行的对齐格式应用到下一行(自动缩进） 
set cindent                    " cindent是特别针对 C语言语法自动缩进 
set smartindent                " 依据上面的对齐格式，智能的选择对齐方式，对于类似C语言编写上有用 
set tabstop=4                  " 设置tab键为4个空格， 
set ai! 
set showmatch                  " 设置匹配模式，类似当输入一个左括号时会匹配相应的右括号 
set guioptions-=T              " 去除vim的GUI版本中得toolbar 
set vb t_vb=                   " 当vim进行编辑时，如果命令错误，会发出警报，该设置去掉警报 
set ruler                      " 在编辑过程中，在右下角显示光标位置的状态行 
set incsearch 
```
## 操作系统调优 

```
cat >> /etc/sysctl.conf <<EOF 
```
粘贴下面内容 
```
fs.file-max=655360 
vm.max_map_count = 262144 
EOF 
```
```
sysctl -p 
```
![图片](https://uploader.shimo.im/f/cVfVn5W43YQnLeMB.png!thumbnail)

```
vim /etc/security/limits.conf 
```
* soft nproc 20480 

* hard nproc 20480 

* soft nofile 65536 

* hard nofile 65536 

* soft memlock unlimited 

* hard memlock unlimited 

 

```
vim /etc/security/limits.d/20-nproc.conf 
```
* soft nproc 20480 
## 

## htop系统监控 

```
yum install htop -y 
```
## tmux窗口管理 

```
yum install tmux -y 
```
## xshell上传下载文件 

```
yum install lrzsz -y 
```
## 安装fuser 

```
yum install -y psmisc  
```
## 安装docker 

如果你之前安装过 docker，请先删掉 

```
sudo yum remove docker docker-common docker-selinux docker-engine 
```
安装一些依赖 
```
sudo yum install -y yum-utils device-mapper-persistent-data lvm2 
```
**根据你的发行版下载repo文件:**  
CentOS/RHEL 

```
wget -O /etc/yum.repos.d/docker-ce.repo https://download.docker.com/linux/centos/docker-ce.repo 
```
把软件仓库地址替换为 TUNA: 
```
sudo sed -i 's+download.docker.com+mirrors.tuna.tsinghua.edu.cn/docker-ce+' /etc/yum.repos.d/docker-ce.repo 
```
最后安装: 
```
sudo yum makecache fast 
sudo yum install docker-ce 
```
