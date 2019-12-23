# Vscode常见玩法

## Mac快捷键

### 使用插件`IntelliJ IDEA Keybindings`

- option+shift+f **格式化**

- option+ F12  **显示终端**

- Crtrl + g **光标选择当前一样内容**

- ##### 命令框

  1. F1 或 Cmd+Shift+P: 打开命令面板。在打开的输入框内，可以输入任何命令
     - 在Cmd+P下输入 > 可以进入 Cmd+Shift+P 模式
  2. 在 Cmd+P 窗口下还可以:
     - 直接输入文件名，跳转到文件
     - ? 列出当前可执行的动作
     - : 跳转到行数，也可以 Cmd+G 直接进入

  

  

  



## 远程开发

### 安装插件

![](http://image-picgo.test.upcdn.net/img/20191206110858.png)

### 设置vscode。

![](http://image-picgo.test.upcdn.net/img/20191206110949.png)

### 配置要远程开发的服务器

![](http://image-picgo.test.upcdn.net/img/20191206111713.png)

当config文件保存后，左边的target就会多一个agent1的图标。我这里之前已经设置了，所以已经存在。

### 连接服务器

- 可以使用F1快捷键显示所有命令后输入ssh

- 也可以对ssh target右键进行连接。

  ![](http://image-picgo.test.upcdn.net/img/20191206112049.png)

### 选择工作目录

![](http://image-picgo.test.upcdn.net/img/20191206112926.png)

### 常用功能

![](http://image-picgo.test.upcdn.net/img/20191206113148.png)

### 连接本地docker容器也是可以的

![](http://image-picgo.test.upcdn.net/img/20191206113617.png)



## 连接jupyter notebook

### 安装插件

![](http://image-picgo.test.upcdn.net/img/20191207144505.png)

### 控制台启动jupyter notebook

![](http://image-picgo.test.upcdn.net/img/20191207144717.png)

控制台会打印连接地址。

### 打开jupyter文件

![](http://image-picgo.test.upcdn.net/img/20191207144547.png)

vscode会帮我们渲染成jupyter的样式。



### 设置vscode连接jupyter notebook

![](http://image-picgo.test.upcdn.net/img/20191207144749.png)



### 运行

![](http://image-picgo.test.upcdn.net/img/20191207144838.png)





## python超强提示kite

## 下载kite到Mac中

https://azure-cdn-kite-downloads.azureedge.net/Kite-0.20191205.2.dmg



### vscode装kite插件

![](http://image-picgo.test.upcdn.net/img/20191207152055.png)



### 测试python自动补全功能

#### without kite

![](http://image-picgo.test.upcdn.net/img/20191207152225.png)

#### with kite

![](http://image-picgo.test.upcdn.net/img/20191207152238.png)

