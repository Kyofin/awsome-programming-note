# impala使用自定义函数生成唯一id





## 使用java编写udf

```java
package com.wugui.hive;

import cn.hutool.core.util.IdUtil;
import org.apache.hadoop.hive.ql.exec.UDF;

public class IdStringUDF extends UDF {
    public static String evaluate(int workerId,int datacenterId) {

        return IdUtil.getSnowflake(workerId,datacenterId).nextIdStr();
    }
 
 
    public static void main(String[] args) {
        System.out.println("下一个id：" + evaluate(1,1));
    }
}

```



## 使用maven打包，这里将使用的依赖都一并打包。

- POM文件加入以下配置

  ```XML
   <dependencies>
  
          <dependency>
              <groupId>org.apache.hive</groupId>
              <artifactId>hive-exec</artifactId>
              <version>1.1.0</version>
          </dependency>
  
  
          <dependency>
              <groupId>cn.hutool</groupId>
              <artifactId>hutool-core</artifactId>
              <version>4.6.0</version>
          </dependency>
  
      </dependencies> 
  <build>
          <plugins>
              <!--将所有依赖都打到jar包中，要是有命令mvn assembly:assembly-->
              <plugin>
                  <artifactId>maven-assembly-plugin</artifactId>
                  <configuration>
                      <descriptorRefs>
                          <descriptorRef>jar-with-dependencies</descriptorRef>
                      </descriptorRefs>
                  </configuration>
              </plugin>
          </plugins>
      </build>
  ```

  

- 使用idea插件打包

  ![](https://raw.githubusercontent.com/huzekang/picbed/master/20191023145507.png)

  

  

## impala shell中执行或者hue上执行创建udf。

```sql
create function if not exists getSerial(int,int) returns string location 'hdfs:///user/impala/udf/hive-starter-1.0-SNAPSHOT-jar-with-dependencies.jar' symbol='com.wugui.hive.IdStringUDF';

```



## 使用udf创建唯一序列号。

![](https://raw.githubusercontent.com/huzekang/picbed/master/20191023145049.png)