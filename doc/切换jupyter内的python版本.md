# 结合conda切换jupyter内的python版本

> 参考：<https://stackoverflow.com/questions/28831854/how-do-i-add-python3-kernel-to-jupyter-ipython>

由于我macbook的aconada是3版本的，默认的jupyter内核就是python3的。

现在需要为jupyter安装上python2.7的内核。

首先需要先安装了基于python2.7的虚拟环境sanke2。

1. 安裝ipykernel到pip

   ```bash
   python -m pip install ipykernel
   ```

2. 使用pip安裝python到jupyter

   ```bash
   python -m ipykernel install --user
   ```

现在重新打开`jupyter notebook`，就可以看到有`python2`和`python3`兩個kernel了。

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190527164119.png)



------

云服务器centos7装的是aconada2版本，需要为jupyter装python3的内核。

1. 创建一个python3的环境并切换到该环境

   ```shell
   conda create -n py36-test python=3.6
   source activate py36-test	
   ```

2. 安装ipykernetl到python3的pip包管理

   ```shell
   python3 -m pip install ipykernel
   ```

3. 安装python3内核到jupyter

   ```shell
   python3 -m ipykernel install --user
   ```

4. 完全退出jupyter

   ![](https://raw.githubusercontent.com/huzekang/picbed/master/20190611183917.png)

5. 重启jupyter

   ```shell
   LANG=zn jupyter notebook --allow-root
   ```

   