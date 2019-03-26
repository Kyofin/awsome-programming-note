# SpringBoot引入第三方jar包

因项目需要用到oracle，但是在maven的公开仓库中已经没有ojdbc6这个依赖了，所以需要自己下载jar包引入到项目中。

- 在src/main/resources建立目录lib,然后将jar包扔进去；

  ![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190326222816.png)

  

- pom文件添加jar依赖

```

		<dependency>
			<groupId>oracle.jdbc</groupId>
			<artifactId>ojdbc</artifactId>
			<version>${ojdbc.version}</version>
			<scope>system</scope>
			<systemPath>${project.basedir}/src/main/resources/lib/oracle/jdbc/ojdbc-6.jar</systemPath>
		</dependency>
```



- pom文件中添加插件

```

<plugin>
  <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-maven-plugin</artifactId>
   <configuration>
       <includeSystemScope>true</includeSystemScope>
   </configuration>
</plugin>

```

打包后jar包的路径即在BOOT-INF\lib目录下。

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190326222526.png)

