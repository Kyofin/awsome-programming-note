# ssh终端复用神器——tmux

## 使用场景：

在使用ssh远程登录服务器后，工作到一半，例如使用wget下载文件下一半，但此时你电脑💻突然没电了，你仍然能保证远程终端不丢失! 



## 功能

- 提供了强劲的、易于使用的命令行界面。
- 可横向和纵向分割窗口。
- 窗格可以自由移动和调整大小，或直接利用四个预设布局之一。
- 支持 UTF-8 编码及 256 色终端。
- 可在多个缓冲区进行复制和粘贴。
- 可通过交互式菜单来选择窗口、会话及客户端。
- 支持跨窗口搜索。
- 支持自动及手动锁定窗口。



## 安装

##### 在 Mac OS 中，通过 brew 安装
```
brew install tmux
```

##### ubuntu版本下直接apt-get安装
```
sudo apt-get install tmux
```

##### centos7版本下直接yum安装
```
yum install -y tmux
```



## 常用操作

##### 1. 在后台建立会话 

```
tmux new -s session -d 
```

##### 2. 然后进入该会话

此时已经进入tmux面板了。可以使用前缀`ctrl+b`加上➕其他按键来触发tmux的功能。详情看快捷键说明。

```
tmux attach -t session 
```

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190506233306.png)

在操作完成后我们使用`ctrl+b+q`暂时退出会话。

##### 3. 列出会话

```
tmux ls
```

此处可以看到我创建的叫session的会话。 ![](https://raw.githubusercontent.com/huzekang/picbed/master/20190506232744.png)



##### 4. 只要该会话不删除，则每次ssh连接远程服务器后，只要进入该会话，就可以继续之前的工作状态了。



## 常用快捷键

`ctrl+b+[` 	然后用↕️键像操作vi一样来滚动屏幕 ，退出直接按`q`键即可。

`ctrl+b+?`	显示快捷键帮助
`ctrl+b+"`	模向分隔窗口

`ctrl+b+%` 	 纵向分隔窗口

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190506234740.png)

`ctrl+b+x`	关闭光标所在的面板

根据提示按y则关闭该面板。

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190506235126.png)

`ctrl+b+q` 	显示分隔窗口的编号
`ctrl+b+上下键`	上一个及下一个分隔窗口