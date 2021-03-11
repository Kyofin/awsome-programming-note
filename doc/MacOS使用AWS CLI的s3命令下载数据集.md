# MacOS使用AWS CLI的s3命令下载数据集

## 安装aws cli

```
python3 -m pip install -i https://pypi.tuna.tsinghua.edu.cn/simple/ awscli --upgrade --user
```

当安装好后使用aws发现没有这个命令。所以需要指定全路径。

```
~/Library/Python/3.7 » ~/Library/Python/3.7/bin/aws --version  

aws-cli/1.19.16 Python/3.7.4 Darwin/19.6.0 botocore/1.20.16
```

安装成功后如上面这样可以看到版本。

## aws上创建组

![image-20210226171607780](http://image-picgo.test.upcdn.net/img/20210226171607.png)

## aws上配置组权限

![image-20210226171611164](http://image-picgo.test.upcdn.net/img/20210226171611.png)

## aws上创建用cli登录的用户

```
~/Library/Python/3.7 » ~/Library/Python/3.7/bin/aws  configure                                                                                                huzekang@huzekangdeMacBook-Pro
AWS Access Key ID [****************AP2A]: 
AWS Secret Access Key [****************o9Tg]: 
Default region name [us-west-2]:
Default output format [csv]:
```

前两个按照之前的id和key输入，然后后两个直接回车即可。

## 列出aws上的数据集

![image-20210226172134443](http://image-picgo.test.upcdn.net/img/20210226172134.png)

## 下载aws上的数据集

```
~/Library/Python/3.7/bin/aws  s3 cp "s3://amazon-reviews-pds/parquet/product_category=Electronics/" ~/Downloads/datasets --recursive
```

![image-20210226172555728](http://image-picgo.test.upcdn.net/img/20210226172555.png)