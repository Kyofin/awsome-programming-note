# Antlr4入门学习-安装配置（Mac）

本文为antlr的入门安装教程。



##### Mac&Linux的配置说明

首先是下载，进入到系统配置目录下，使用curl命令对文件进行下载：

```shell
knightdeMacBook-Pro-2:~ knight$ cd /usr/local/lib
knightdeMacBook-Pro-2:lib knight$ sudo curl -O https://www.antlr.org/download/antlr-4.7.1-complete.jar
```

下载完成后，注意重启bash，在这一路径下进行vim的操作

```shell
knightdeMacBook-Pro-2:~ knight$ vim .bash_profile
```

当然也可以cd /Users/YourName 来进行操作

```shell
knightdeMacBook-Pro-2:lib knight$ cd /Users/Knight
knightdeMacBook-Pro-2:Knight knight$ cd ~
knightdeMacBook-Pro-2:~ knight$ vim .bash_profile
```

然后进入到vim编辑器中，修改路径并设置快捷名（如果出现下列情况，选择edit anyway 或者先 quit 后再 edit）[![img](https://kingganzeng.github.io/Antlr4%E5%85%A5%E9%97%A8%E5%AD%A6%E4%B9%A0-%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE/355ABCC9A1785007329E01444EC9EBAE.png)](https://kingganzeng.github.io/Antlr4入门学习-安装配置/355ABCC9A1785007329E01444EC9EBAE.png)

进入之后由于现在vim处于命令模式，不能对内容进行修改，我们输入i即可（insert模式）

```
*
export CLASSPATH=".:/usr/local/lib/antlr-4.0-complete.jar:$CLASSPATH"
alias antlr4='java -jar /usr/local/lib/antlr-4.0-complete.jar'
alias grun='java org.antlr.v4.runtime.misc.TestRig'
```

如下图：

[![img](https://kingganzeng.github.io/Antlr4%E5%85%A5%E9%97%A8%E5%AD%A6%E4%B9%A0-%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE/3A1597632D78FA77A21796364BD36155.png)](https://kingganzeng.github.io/Antlr4入门学习-安装配置/3A1597632D78FA77A21796364BD36155.png)

这一步配置完成后，按esc后退回到命令模式，然后输入qw，接着输入两个大写Z即可保存退出。

接下来运行

```shell
knightdeMacBook-Pro-2:~ knight$ source .bash_profile
```

source 命令是为了重新执行刚修改的初始化文件，使之立即生效，而不必注销并重新登录。这时候antlr的系统配置已经完成了。

我们输入antlr4或者grun可以查看版本信息：

[![img](https://kingganzeng.github.io/Antlr4%E5%85%A5%E9%97%A8%E5%AD%A6%E4%B9%A0-%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE/5ECDBD935BBD18B22A601D581A794B8A.png)](https://kingganzeng.github.io/Antlr4入门学习-安装配置/5ECDBD935BBD18B22A601D581A794B8A.png)

[![img](https://kingganzeng.github.io/Antlr4%E5%85%A5%E9%97%A8%E5%AD%A6%E4%B9%A0-%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE/C05BAD4D21491C0596D9E5862243BC9E.png)](https://kingganzeng.github.io/Antlr4入门学习-安装配置/C05BAD4D21491C0596D9E5862243BC9E.png)

关键的操作是快捷配置这一步，之前按照官网直接在usr/local/lib下输入export并不能使得配置生效，导致每次在重启项目后都需要重新配置，后来才发现是导入的位置不对。