当我们的json配置中设置`job.setting.speed.channel= 4`时，datax会启动一个job和一个taskgroup和4个task。对应的是4个reader线程和4个writer线程。

线程的命名是`jobid  - taskgroupid  -  taskid  -  reader/wrtier`

![](http://image-picgo.test.upcdn.net/img/20191208122647.png)

而每一对writer和reader通过一个channel对象来连接。上面4个task，所以就有四个channel对象。

下面两图对同task0断点可以看到reader和writer线程的channel对象都是一样的。

![](http://image-picgo.test.upcdn.net/img/20191208123140.png)

![](http://image-picgo.test.upcdn.net/img/20191208123209.png)

但是和task1的channel就不一样了。

![](http://image-picgo.test.upcdn.net/img/20191208123349.png)