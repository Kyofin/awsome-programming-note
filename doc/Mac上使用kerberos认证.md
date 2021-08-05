# Mac上使用kerberos认证

## 说明

macbook是自带kerberos工具的，所以只要拿到krb5.conf和对应的keytab，就可以访问公司服务器开启了kerberos认证的hdp集群。

## 下载配置

登录hdp集群的服务器上拷贝`/ect/krb5.conf`文件到macbook的`/etc`目录下。

```
[libdefaults]
  renew_lifetime = 7d
  forwardable = true
  default_realm = EXAMPLE.COM
  ticket_lifetime = 24h
  dns_lookup_realm = false
  dns_lookup_kdc = false
  default_ccache_name = /tmp/krb5cc_%{uid}
  #default_tgs_enctypes = aes des3-cbc-sha1 rc4 des-cbc-md5
  #default_tkt_enctypes = aes des3-cbc-sha1 rc4 des-cbc-md5

[logging]
  default = FILE:/var/log/krb5kdc.log
  admin_server = FILE:/var/log/kadmind.log
  kdc = FILE:/var/log/krb5kdc.log

[realms]
  EXAMPLE.COM = {
    admin_server = 10.93.6.247
    kdc = 10.93.6.247
  }
```

再拷贝hive的keytab到macbook上。

![image-20210805181131484](http://image-picgo.test.upcdn.net/img/20210805181131.png)

## mac上认证hive用户

```
 kinit -kt ~/Downloads/hive.service.keytab hive/hadoop-master
```

![image-20210805181235784](http://image-picgo.test.upcdn.net/img/20210805181235.png)