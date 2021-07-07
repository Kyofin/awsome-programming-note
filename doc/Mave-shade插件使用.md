# Mave-shade插件使用

## 介绍

Apache Maven项目提供的Shade插件，能够将Maven应用打包为超级的uber-jar（也称为fat jar，或shaded jar）。即在打包的过程中，可以：

- 包含依赖库
- 重命名依赖库的包名（以避免类库的冲突）
- 有选择地打包



## 完整pom

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>quick-maven-shade</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-sql_2.11</artifactId>
            <version>2.4.7</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.75</version>
        </dependency>
        
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.0.0</version>
                <executions>
                    <!-- Run shade goal on package phase -->
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <createDependencyReducedPom>true</createDependencyReducedPom>
                            <!--   需要改写引用到该依赖中类的package-->
                            <relocations>
                                <relocation>
                                    <pattern>com.alibaba.fastjson</pattern>
                                    <shadedPattern>com.data.shade.com.alibaba.fastjson</shadedPattern>
                                </relocation>
                            </relocations>
                            <artifactSet>
                                <!--   需要打进jar包中的依赖-->
                                <includes>
                                    <include>com.alibaba:fastjson</include>
                                </includes>
                            </artifactSet>
                            <filters>
                                <filter>
                                    <!-- Do not copy the signatures in the META-INF folder.
                                    Otherwise, this might cause SecurityExceptions when using the JAR. -->
                                    <artifact>*:*</artifact>
                                    <excludes>
                                        <exclude>META-INF/*.SF</exclude>
                                        <exclude>META-INF/*.DSA</exclude>
                                        <exclude>META-INF/*.RSA</exclude>
                                    </excludes>
                                </filter>
                            </filters>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```



## 示例代码准备

![image-20210617092241975](http://image-picgo.test.upcdn.net/img/20210617092242.png)

这里写了一个简单Spark程序，里面引用到了Fastjson依赖里的类。



## 执行编译

```
mvn clean package   
```

编译完后，可以看到编译后的jar包中包含了fastjson的代码，而且它的路径是`com.data.shade.com.fastjson`。

![image-20210617092550867](http://image-picgo.test.upcdn.net/img/20210617092550.png)

查看我们自己编写的Spark程序，可以观察到引用的fastjson路径变了。

![image-20210617092631284](http://image-picgo.test.upcdn.net/img/20210617092631.png)



## 结论

该插件可以将外部依赖连同自己写的代码一同打包，而且可以把外部依赖转成像自己写的代码一样。这样，自己写的代码里用到这个依赖的，都可以像引用自己写的类一样。