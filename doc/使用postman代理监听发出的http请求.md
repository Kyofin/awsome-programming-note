# 使用postman代理监听发出的http请求

启动postman代理

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190513140142.png)



浏览器开启代理转发所有请求到postman代理

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190513140228.png)



测试浏览器发出请求

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190513140416.png)



此时postman已经记录了发出的请求了。

如果postman捕获不了本地的请求，例如：`127.0.0.1:8080/data`可以尝试使用你的内网ip进行访问，postman即可监听到请求

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190513140449.png)