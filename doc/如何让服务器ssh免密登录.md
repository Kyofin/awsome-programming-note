[TOC]

# 如何让服务器ssh免密登录

## 1.进入~/.ssh目录，在本地生成相关私钥与公钥。 

```
~/.ssh ⌚ 17:03:35 
$ ssh-keygen -b 4096 -t rsa -C huzekang@mac 
```

命令执行完成之后，在当前目录会生成id_rsa和id_rsa.pub两个key文件。 

参数`b`指定key的长度，本例中指定的长度为4096。参数t指定加密算法，本例中使用的RSA加密算法。 

参数`C`可以是邮件地址或域名等，会被添加到key的尾部，以示区分。 



## 2.查看本地生成的公钥 

```
ccat id_rsa.pub 
```

可以看到如下内容 

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCzJHp0kkSF8Y/77rGQuTW3lfS3roMt7TI57yhyQZtKqTzaWUMKwALUOD/mop0wuqEyY40EDRG/Wxu/VRzVoaWZ6COQ2q2jXyLU49W/dGtGBHBsrJPTq0Y+AZ2Lk3B4sQrSzqy+jRsum5nrv/vYiVQZBhJMEHMPAe9Tqs+fLccOqkEKaG9l6+n5bHBRsRd8vGw6TmyanCB7kWgWX8+umkMRM+aeYagw+ryktxs5R3t9FQhQuYXY8mYrOrbXmZH3jso7cJ6Bcv20tVDMjZbe0x+zNLcu3DRz0mdFytPrs2ZoWPphYrsDE7rBRSzNAQeR9zQkqQu0f6Nexu2PXPHsCrHcrTGhw2YKajCtMivrqDzF2b2DC6IZ3Y8Td04c+2RKB3572/sIO4o+/U4YpYiQoELmPYdav6s042D+Uk2HUdRHnAs3G4FSVI4V7vu1V4S0TsYCuhd7oIImya2IeHjof+yH9FGKTYtJ5JjMqXRYiZOTfvIFxQRrtdZHtEyzjvkkZPEhro1Df8u7jVVVSYbaUb9451nKAg3ew0+6h1LXeQM+lrZ9aZlszzJL36BKKeQQFg4qndPzXX4XdiyfhTo+yhvFHeAnAWLL6W3xtwhnEVXIwDRSG03bSTqbum41x9bFS1c3wzFXsAKIpMfhHUlgiW+A8Mk4aQ2phNSaFyX2xW3GcQ== huzekang@mac 
```

可以看到最后跟上的是之前加的标识huzekang@mac。 



## 3.将公钥上传到服务器 

```
ssh-copy-id -i id_rsa.pub -p 50000 hzk@134.175.29.194 


```

参数`i`是指定公钥文件。 

参数`p`是ssh的端口。 



过程中需要输入该账户ssh登录时的密码，输入正确后，可以看到如下内容输出。 

```
usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "id_rsa.pub" 

/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed 

/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys 

hzk@134.175.29.194's password: 



Number of key(s) added:        1 



Now try logging into the machine, with:   "ssh -p '50000' 'hzk@134.175.29.194'" 

and check to make sure that only the key(s) you wanted were added. 
```





## 4.此时打开服务器，检查授权登录文件 

```
-> # cat authorized_keys 
```

可以看到我们的公钥已经追加在文件中了。 

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCzJHp0kkSF8Y/77rGQuTW3lfS3roMt7TI57yhyQZtKqTzaWUMKwALUOD/mop0wuqEyY40EDRG/Wxu/VRzVoaWZ6COQ2q2jXyLU49W/dGtGBHBsrJPTq0Y+AZ2Lk3B4sQrSzqy+jRsum5nrv/vYiVQZBhJMEHMPAe9Tqs+fLccOqkEKaG9l6+n5bHBRsRd8vGw6TmyanCB7kWgWX8+umkMRM+aeYagw+ryktxs5R3t9FQhQuYXY8mYrOrbXmZH3jso7cJ6Bcv20tVDMjZbe0x+zNLcu3DRz0mdFytPrs2ZoWPphYrsDE7rBRSzNAQeR9zQkqQu0f6Nexu2PXPHsCrHcrTGhw2YKajCtMivrqDzF2b2DC6IZ3Y8Td04c+2RKB3572/sIO4o+/U4YpYiQoELmPYdav6s042D+Uk2HUdRHnAs3G4FSVI4V7vu1V4S0TsYCuhd7oIImya2IeHjof+yH9FGKTYtJ5JjMqXRYiZOTfvIFxQRrtdZHtEyzjvkkZPEhro1Df8u7jVVVSYbaUb9451nKAg3ew0+6h1LXeQM+lrZ9aZlszzJL36BKKeQQFg4qndPzXX4XdiyfhTo+yhvFHeAnAWLL6W3xtwhnEVXIwDRSG03bSTqbum41x9bFS1c3wzFXsAKIpMfhHUlgiW+A8Mk4aQ2phNSaFyX2xW3GcQ== huzekang@mac 
```



## 5.此时在mac中再次登录服务器 

```
$ ssh -p 50000 hzk@134.175.29.194 
Last login: Sat Feb 23 16:58:01 2019 from 183.48.88.227 

```

不用输入密码也完成登录了。 