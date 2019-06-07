# conda切换多版本Python

> 参考：<https://www.jianshu.com/p/d2e15200ee9b>

## conda使用

### 检测安装的conda版本

```bash
conda --version
```

Conda会返回你安装Anaconda软件的版本。



### 升级当前版本conda

```bash
conda update conda
```

更新conda到最新版本之后，利用conda进行环境管理



### 创建新环境

```bash
conda create -n snake2 python=2
```

这里创建了名为snake2的虚拟环境，python版本为二。如果需要三版本的，可以改为`python=3`



### 切换到新环境

```bash
conda activate snake2
```

此时检测一下环境内的python版本

```bash
python --version                                  

Python 2.7.16 :: Anaconda, Inc.
```

可以发现已经切换成python2了。



### 切换回原来的环境

```bash
conda deactivate
```

此时检测一下环境内的python版本

```bash
python --version 

Python 3.6.8 :: Anaconda custom (64-bit)
```



### 列出所有的环境：

```
conda info --envs
```

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190527160942.png)

可以看到`snake2`后有个`*`号，代表当前激活环境。



### 检查环境内的包：

```bash
conda list
```

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190527160737.png)

这里由于snake2是新安装的环境，所以可以看出新安装的包并不多。



### 查找一个包是否能安装的版本：

```bash
conda search beautifulsoup4
```



### 安装包，必须告知安装环境的名字：

```bash
 conda install --name snake2 pandas
```

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190527165903.png)

这里安装pandas包到我的python2虚拟环境sanke2中。



检查一下在snake环境中是否安装成功。

```bash
conda list
```

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190527170053.png)



### 移除安装的包，必须告知移除包的环境：

```bash
conda remove -n base beautifulsoup4
```



### 从Anaconda.org安装一个包

如果一个包不能使用conda安装，我们接下来将在Anaconda.org网站查找。Anaconda.org向公开和私有包仓库提供包管理服务。Anaconda.org是一个连续分析产品。
 **提示：**你在Anaconda.org下载东西的时候不强制要求注册。
 **为了从Anaconda.org下载到当前的环境中，我们需要通过指定Anaconda.org为一个特定通道，通过输入这个包的完整路径来实现。**
 在浏览器中，去 <http://anaconda.org> 网站。我们查找一个叫“bottleneck”的包，所以在左上角的叫“Search Anaconda Cloud”搜索框中输入“bottleneck”并点击search按钮。
 Anaconda.org上会有超过一打的bottleneck包的版本可用，但是我们想要那个被下载最频繁的版本。所以你可以通过下载量来排序，通过点击Download栏。
 点击包的名字来选择最常被下载的包。它会链接到Anaconda.org详情页显示下载的具体命令：

```bash
conda install --channel https：//conda .anaconda.ort/pandas bottleneck
```

### 



### 通过pip命令来安装包

对于那些无法通过conda安装或者从Anaconda.org获得的包，我们通常可以用pip（“pip install packages”的简称）来安装包。
 **提示：** pip只是一个包管理器，所以它不能为你管理环境。pip甚至不能升级python，因为它不像conda一样把python当做包来处理。但是它可以安装一些conda安装不了的包，和vice versa（此处不会翻译）。**pip和conda都集成在Anaconda或miniconda里边。**

我们激活我们想放置程序的环境

```bash
activate snake2
```

然后通过pip安装一个叫“See”的程序。

```bash
pip install see
```



### 移除一个虚拟环境：

```bash
conda remove -n python3 --all
```





