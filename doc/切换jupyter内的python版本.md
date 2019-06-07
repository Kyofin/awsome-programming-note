# 结合conda切换jupyter内的python版本

由于我的aconda是3版本的，默认的jupyter内核就是python3的。现在需要为它安装上python2.7的内核。

参考上述已经安装了基于python2.7的虚拟环境sanke2。

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