# hdp sandbox教程

## 参考文档

[账号权限相关文档](https://www.cloudera.com/tutorials/learning-the-ropes-of-the-hdp-sandbox.html#login-credentials)

[Quickstart](https://www.cloudera.com/tutorials/learning-the-ropes-of-the-hdp-sandbox.html)

## 安装

1. docker加载本地备份的镜像

   ![](http://image-picgo.test.upcdn.net/img/20200612102505.png)

   ```SHELL
   docker load --input hdp-box.tar
   docker load --input hdp-proxy.tar
   ```

2. 解压自动化部署包

   ![image-20200612102814824](http://image-picgo.test.upcdn.net/img/20200612102814.png)

3. 修改代理脚本，开放自定义端口5432，方便后续远程访问容器内的pg

   hdp-sandbox/assets/generate-proxy-deploy-script.sh

   ![image-20200612134358495](http://image-picgo.test.upcdn.net/img/20200612134358.png)

4. 运行脚本 `docker-deploy-hdp30.sh`



## 使用

本地mac配置sanbox的host和ip

```
192.168.1.97 sandbox-hdp.hortonworks.com 
192.168.1.97 sandbox-hdf.hortonworks.com
```

### 远程终端访问

mac 上进行ssh sandbox

```shell
ssh -p '2222' 'root@sandbox-hdp.hortonworks.com'
```

初始密码是hadoop。第一次登陆需要修改密码。

修改root登陆密码为**hadoop123456**



### 远程welcome页面访问

http://sandbox-hdp.hortonworks.com:1080/splash.html



### 常用账号

| admin      | 需要登录终端后使用`ambari-admin-password-reset`命令设置 |
| ---------- | ------------------------------------------------------- |
| maria_dev  | maria_dev                                               |
| raj_ops    | raj_ops                                                 |
| holger_gov | holger_gov                                              |
| amy_ds     | amy_ds                                                  |



### 远程访问容器内数据库

![image-20200612134534715](http://image-picgo.test.upcdn.net/img/20200612134534.png)