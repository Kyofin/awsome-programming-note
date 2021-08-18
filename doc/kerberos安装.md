# kerberos安装

## 1. 安装

```bash
yum install -y  krb5-workstation krb5-libs krb5-auth-dialog krb5-server

```

## 2. 执行完命令后，系统会生成kerberos配置文件
`/etc/krb5.conf`和`/var/kerberos/krb5kdc/kdc.conf`

![image-20210730094713167](http://image-picgo.test.upcdn.net/img/20210730094713.png)

##3. 修改kdc.conf配置文件
```bash
vim /var/kerberos/krb5kdc/kdc.conf
```
修改成下面这样。

```bash
[kdcdefaults]
 kdc_ports= 88
 kdc_tcp_ports= 88
 
[realms]
 HADOOP.COM= {
  #master_key_type = aes256-cts
  acl_file= /var/kerberos/krb5kdc/kadm5.acl
  dict_file= /usr/share/dict/words
  admin_keytab= /var/kerberos/krb5kdc/kadm5.keytab
  supported_enctypes= aes128-cts:normal des3-hmac-sha1:normal arcfour-hmac:normal camellia256-cts:normal camellia128-cts:normal des-hmac-sha1:normal des-cbc-md5:normal des-cbc-crc:normal
  max_life= 1d
  max_renewable_life= 7d
 }
```

## 4. 修改krb5.conf配置文件
```bash
vim /etc/krb5.conf
```
修改成下面这样。
```bash
# Configuration snippets may be placed in this directory as well
includedir/etc/krb5.conf.d/
 
[logging]
 default= FILE:/var/log/krb5libs.log
 kdc= FILE:/var/log/krb5kdc.log
 admin_server= FILE:/var/log/kadmind.log
 
[libdefaults]
 dns_lookup_realm= false
 ticket_lifetime= 24h              
 renew_lifetime= 7d              
 forwardable= true
 rdns= false
 pkinit_anchors= /etc/pki/tls/certs/ca-bundle.crt
 default_realm= HADOOP.COM  
 udp_preference_limit= 1
#default_ccache_name = KEYRING:persistent:%{uid}  
 
[realms]
 HADOOP.COM= {
 	# 注意要修改成kdc部署服务器的hostname
  kdc= kerberos010
  admin_server= kerberos010
 }
 
[domain_realm]  
# 指定域名和域的映射关系
# .example.com = EXAMPLE.COM
# example.com = EXAMPLE.COM
```


## 5. 配置kerberos的数据库
```bash
kdb5_util create -s -r HADOOP.COM
```
这里我设置的密码都是123456。
成功后可以检查生成的数据库文件。
```bash
ll /var/kerberos/krb5kdc/
```

![image-20210730101212313](http://image-picgo.test.upcdn.net/img/20210730101212.png)

添加database administrator (即能够管理database的principals)
```bash
kadmin.local -q "addprinc admin/admin"
```
并设置密码为admin。

![image-20210730110554542](http://image-picgo.test.upcdn.net/img/20210730110554.png)

## 6. 设置krb5kdc服务和kadmin服务开机自启动
```bash
service krb5kdc start

service kadmin start

chkconfig krb5kdc on

chkconfig kadmin on

```

![image-20210730111059854](http://image-picgo.test.upcdn.net/img/20210730111059.png)

## 7. 创建test principal
设置密码为123456
```bash
kadmin.local
```

![image-20210730111457845](http://image-picgo.test.upcdn.net/img/20210730111457.png)

验证是否创建成功，可以使用`klist`命令验证。
如果没有登录，会报错，如果使用`kinit`登录账户，就不会。
如果使用`kdestory`注销账户，在使用`klist`就会报错。
```bash
[root@kerberos010 kerberos]# klist 
klist: No credentials cache found (filename: /tmp/krb5cc_0)
[root@kerberos010 kerberos]# kinit test/test
Password for test/test@HADOOP.COM: 
[root@kerberos010 kerberos]# klist 
Ticket cache: FILE:/tmp/krb5cc_0
Default principal: test/test@HADOOP.COM

Valid starting       Expires              Service principal
07/30/2021 14:22:27  07/31/2021 14:22:27  krbtgt/HADOOP.COM@HADOOP.COM
        renew until 08/06/2021 14:22:27
[root@kerberos010 kerberos]# kdestroy 
[root@kerberos010 kerberos]# klist 
klist: No credentials cache found (filename: /tmp/krb5cc_0)
```

## 8. 另外两个节点安装kerberos的client
```bash
yum install krb5-workstation krb5-libs krb5-auth-dialog -y
```
另外两个节点设置配置/etc/krb5.conf和server端保持一致
```bash
scp /etc/krb5.conf root@kerberos011:/etc/krb5.conf
```

## 9. 使用用户名和密码的方式验证kerberos配置在客户端通过用户名和密码认证
```bash
[root@kerberos011 ~]# klist 
klist: No credentials cache found (filename: /tmp/krb5cc_0)
[root@kerberos011 ~]# kinit test/test
Password for test/test@HADOOP.COM: 
[root@kerberos011 ~]# klist 
Ticket cache: FILE:/tmp/krb5cc_0
Default principal: test/test@HADOOP.COM

Valid starting       Expires              Service principal
07/30/2021 15:03:36  07/31/2021 15:03:36  krbtgt/HADOOP.COM@HADOOP.COM
        renew until 08/06/2021 15:03:36
```



## 10. 密钥的方式认证
在server端生成秘钥，并拷贝到client

```bash
kadmin.local -q "xst -k /root/test.keytab test/test@HADOOP.COM"
```

![image-20210730152904735](http://image-picgo.test.upcdn.net/img/20210730152904.png)

成功后可以看到生成的文件`/root/test.keytab`

将该文件拷贝到另外两台服务器后，可以使用`kinit`通过秘钥登录。

```bash
[root@kerberos011 ~]# kinit -kt /root/test.keytab test/test
[root@kerberos011 ~]# klist 
Ticket cache: FILE:/tmp/krb5cc_0
Default principal: test/test@HADOOP.COM

Valid starting       Expires              Service principal
07/30/2021 15:28:11  07/31/2021 15:28:11  krbtgt/HADOOP.COM@HADOOP.COM
        renew until 08/06/2021 15:28:11
```

这里要注意：

通过秘钥登录后，就不能通过用户名和密码登录了



## 查看keytab中的principal

```
klist -kt ozone.keytab
```

![image-20210817144932509](http://image-picgo.test.upcdn.net/img/20210817144932.png)