# Git超实用攻略

## 创建新分支并切换到新分支

```
git checkout -b yinchuang master 
```

这里是根据master创建新分支yinchuang。

可以使用`git branch`查看有的分支和当前分支。

![](https://i.loli.net/2019/10/31/hBi63ckxY5IeSwv.png)



## 将本地新分支推送到新远程库

先定义远程仓库别名和地址

```
 git remote add yinchuang http://139.9.4.32:8899/lijiakai/backend-oauthserver.git
```

这里是定义远程仓库别名为因创。

再推送本地指定分支到远程仓库。

```
 git push -u yinchuang  yinchuang  
```

**注意：第一个是定义的远程库名，方便不用记忆地址。第二个才是是本地分支名。**

![](https://i.loli.net/2019/10/31/hRGgseMqVK7nF8u.png)

可以看到新创的远程库中只有一个分支。



## push错了怎么办

1. 执行  git log查看日志，获取需要回退的版本号 

![img](https://img2018.cnblogs.com/blog/788599/201809/788599-20180927164303193-2084393469.png)

2. 执行 git reset –soft <版本号> ，如 

`git reset -soft 4f5e9a90edeadcc45d85f43bd861a837fa7ce4c7 `，重置至指定版本的提交，达到撤销提交的目的

然后执行 git log 查看

![img](https://img2018.cnblogs.com/blog/788599/201809/788599-20180927164827547-451137005.png)

此时，已重置至指定版本的提交，log中已经没有了需要撤销的提交

>  git reset 命令分为两种： git reset –soft 与 git reset –hard ，区别是：
>
>    前者表示只是改变了HEAD的指向，本地代码不会变化，我们使用git status依然可以看到，同时也可以git commit提交。后者直接回改变本地源码，不仅仅指向变化了，代码也回到了那个版本时的代码。

3. 执行` git push -f origin master` ，强制提交当前版本号。这里origin是定义的远程仓库的别名，master为本地分支名。

至此，撤销push提交完成。

![](https://i.loli.net/2019/10/31/MuFpbOAW6DVeYLy.png)

可以看到自己错误push的已经没了。

