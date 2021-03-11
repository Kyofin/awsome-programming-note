# IDEA中远程debug调试Hive UDF

## 进入hive终端

```
hive --debug -e "add jar file:///Users/huzekang/code/liantong/chinaunicom-hive/hive-udf/target/hive-udf-1.0-SNAPSHOT-jar-with-dependencies.jar;create temporary function udfb as 'com.chinaunicom.hive.udf.MedicalDictUDF';select udfb('CT01.00.001','01')"
```

可以看到在监控8000端口

![image-20210128113310869](http://image-picgo.test.upcdn.net/img/20210128113310.png)

## 打开idea添加remote调试

![image-20210128113340650](http://image-picgo.test.upcdn.net/img/20210128113340.png)

## idea中运行remote调试

![image-20210128113416677](http://image-picgo.test.upcdn.net/img/20210128113416.png)

此时hive终端会继续执行，当执行到断点处即可。