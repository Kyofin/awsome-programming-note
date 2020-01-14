在debug datax源码过程中，发现了个奇怪现象。就是：

idea中修改reader和writer插件的代码后，新增的代码在利用idea断点时无法访问。

![](http://image-picgo.test.upcdn.net/img/20191208131636.png)

但是在core包中加了这句又可以断点到。

![](http://image-picgo.test.upcdn.net/img/20191208131824.png)

其实上图已经有答案，因为每个task在初始化它的reader和writer线程的时候，都对`thread`对象使用了`setContextClassLoader`方法改变了它的classpath，读的路径是`Datax home`中打包好的目录里的jar包。

![](http://image-picgo.test.upcdn.net/img/20191208132113.png)

所以在reader和writer线程的classpath和主线程中的classpath是不一样的。

下图是streamwriter中代码debug时计算的。

![](http://image-picgo.test.upcdn.net/img/20191208132505.png)

下图是TaskGroupContainer中代码debug时计算的。

![](http://image-picgo.test.upcdn.net/img/20191208132427.png)

而我们直接在idea中改writer的代码，除非编译后输出到`DATAX HOME`的目录，否则都不会在idea中debug得到。