[TOC]

# iTerm2使用技巧

## 同时输出命令到多个tab

只需要输入快捷键 `⌘(command) + ⇧(shift) + i`

这里我尝试在键盘输出ifconfi，可以看到两个tab都显示了我输的命令。

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190501115706.png)



## 自动提示补全

输入若干字符，按`⌘+;`弹出自动补齐窗口，列出曾经使用过的命令。

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190501111744.png)



设置终端历史行数为不限制

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190501111843.png)



## 查看输过哪些历史命令

按`⌘+Shift+h`弹出历史粘贴记录窗口

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190501112032.png)



## 命令回滚快照

按下 `Cmd + Option + B` 就会在界面上显示一个时间轴。

这时候，我们按下键盘的左右箭头，时间轴就会自由的穿梭，这时 iTerm 上的命令行界面也随着变化成你选中的时间点的内容了。

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190501112456.png)



## 一键调出命令行

我们有时会遇上这样一种情况，就是我们只想用命令行执行某一个特定的操作，然后就不需要它了。使用自定义快捷键`⌘+\`

### 效果如下：

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190501113902.png)

### 设置如下：

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190501113739.png)

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190501113722.png)



## 巧用快捷键

### 按住⌘键

1. 可以拖拽选中的字符串；

2. 点击 url：调用默认浏览器访问该网址

3. 点击文件：调用默认程序打开文件；

4. 如果文件名是filename:42，且默认文本编辑器是 Macvim、Textmate或BBEdit，将会直接打开到这一行；

5. 点击文件夹：在 finder 中打开该文件夹；

   ![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190501112555.png)

6. 同时按住option键，可以以矩形选中，类似于vim中的ctrl v操作。

### control 的使用

1. control + a 光标切换到本行开头
2. control + e 光标切换到本行结尾
3. control + w 删除一个单词
4. control + u 删除一行

### 常用快捷键

- cmd + n：新建窗口

- cmd + w：关闭窗口

- 切换 tab：⌘+←, ⌘+→, ⌘+{, ⌘+}。⌘+数字直接定位到该 tab；

- 新建 tab：⌘+t；

- 切分屏幕：⌘+d 水平切分，⌘+Shift+d 垂直切分；

- 智能查找，支持正则查找：⌘+f。

  ![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190501112741.png)

- 粘贴：⌘+f查找后，按住 Tab 可以选择性全部选中，并且已经自动复制到剪切板了。这一个小小的功能，让我们不必在键盘和鼠标之间频繁切换了，非常的实用。

## 配置同时打开多个ssh会话

我这里配置了两个，一个是vmware一个是dell。

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190501132424.png)

配置完成后按`⌘+o`会显示profiles

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190501133608.png)

选择要打开的，可以看到右边有悬浮的标记

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190501133626.png)

