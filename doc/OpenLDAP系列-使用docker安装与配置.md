# OpenLDAP系列-使用docker安装与配置

## 启动docker容器

```
docker run -p 389:389 -p 636:636 \
--name my-openldap-container \
--env LDAP_TLS_VERIFY_CLIENT="never" \
--env LDAP_ORGANISATION="hadoopOrg"  \
--env LDAP_DOMAIN="hadoop.cn" \
--env LDAP_ADMIN_PASSWORD="hadoop.cn123" \
-v /Users/huzekang/docker_data/ldap/data:/var/lib/ldap \
-v /Users/huzekang/docker_data/ldap/conf:/etc/ldap/slapd.d \
--detach osixia/openldap:1.3.0

```



## 测试

使用Apache Directory Studio进行登录、验证测试。

[下载地址](https://archive.apache.org/dist/directory/studio/2.0.0.v20180908-M14/ApacheDirectoryStudio-2.0.0.v20180908-M14-macosx.cocoa.x86_64.dmg)

打开studio后，点击左下角的connection小窗口中的创建连接。

![image-20210715145543889](http://image-picgo.test.upcdn.net/img/20210715145543.png)



docker容器默认的管理源账号是admin。

![image-20210715145549984](http://image-picgo.test.upcdn.net/img/20210715145550.png)

登录成功后，可以看到如下：

![image-20210715145614792](http://image-picgo.test.upcdn.net/img/20210715145614.png)





