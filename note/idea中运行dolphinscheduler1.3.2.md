# idea中运行dolphinscheduler1.3.2

## 拉取代码

```
gti clone 	https://github.com/apache/incubator-dolphinscheduler.git
```



## 导入idea

![image-20200923111131019](http://image-picgo.test.upcdn.net/img/20200923111131.png)



## 修改本地配置

![image-20200923111513255](http://image-picgo.test.upcdn.net/img/20200923111513.png)

主要修改上面这些配置。

其中common文件要配置hdfs相关，yarn相关信息。注意其中`dolphinscheduler.env.path`要指向worker加载的环境变量文件(即上图中的sh文件)



## 配置启动类相关

![image-20200923111628430](http://image-picgo.test.upcdn.net/img/20200923111628.png)

worker配置了logback文件才能有job日志生成。

![image-20200923111648510](http://image-picgo.test.upcdn.net/img/20200923111648.png)