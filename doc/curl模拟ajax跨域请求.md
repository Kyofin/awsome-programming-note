## curl模拟ajax跨域请求

```
curl -H "Origin: http://example.com" \
  -H "Access-Control-Request-Method: GET" \
  -H "Access-Control-Request-Headers: X-Requested-With" \
  -X OPTIONS --verbose  http://192.168.2.28:8080/login?clientUrl=http://10.91.1.7:8082/cas
```



## 响应结果显示不允许跨域

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190409205622.png)



## 解决办法

在springboot项目中加入如下配置

```java
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {


    /**
     * 允许跨域配置
     * @param registry
     */
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("*")
                .allowedMethods("GET", "POST", "DELETE", "PUT", "OPTIONS")
                .allowCredentials(false).maxAge(3600);
    }
  }
```



再次用curl模拟请求，请求成功。

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190409210202.png)