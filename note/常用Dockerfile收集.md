# Dockerfile收集

## 集成tomcat

```dockerfile
#  容器构建镜像 ：docker build -t deploy/tomcat8 .
#  容器启动：docker run  -p 8093:8080 -v /data/metadataFiles:/data/metadataFiles -d --restart=always  --name metadata deploy/tomcat8

FROM tomcat:8-jre8

MAINTAINER "youname youemail"

RUN rm  -rf /usr/local/tomcat/webapps/ROOT

COPY metadata.war /usr/local/tomcat/webapps/ROOT.war

CMD ["catalina.sh","run"]
```



## 集成springboot

```dockerfile

# java镜像
FROM daocloud.io/java:8

# 将本地文件夹挂载到当前容器
# 创建/tmp目录并持久化到Docker数据文件夹，因为Spring Boot使用的内嵌Tomcat容器默认使用/tmp作为工作目录。
VOLUME ["/tmp"]

# 解决容器时间和宿主主机时间不一致问题
RUN /bin/cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo 'Asia/Shanghai' >/etc/timezone


# 拷贝文件到容器
COPY casServer.jar /opt/app.jar

# 打开服务端口
EXPOSE 8080 8080

# 配置环境变量 todo jvm优化参数可以设置这里
ENV JAVA_OPTS='-Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=8077' APP_OPTS=''

# 配置容器启动后执行的命令
ENTRYPOINT java $JAVA_OPTS -server -Dfile.encoding=UTF-8 -Duser.language=zh -Duser.region=CN -Djava.security.egd=file:/dev/./urandom -jar /opt/app.jar $APP_OPTS
```

