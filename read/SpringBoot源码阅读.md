### BasicErrorController

- #### 处理html请求error

```java
@RequestMapping(produces = "text/html")
	public ModelAndView errorHtml(HttpServletRequest request,
			HttpServletResponse response) {
		HttpStatus status = getStatus(request);
		Map<String, Object> model = Collections.unmodifiableMap(getErrorAttributes(
				request, isIncludeStackTrace(request, MediaType.TEXT_HTML)));
		response.setStatus(status.value());
		ModelAndView modelAndView = resolveErrorView(request, response, status, model);
		return (modelAndView != null) ? modelAndView : new ModelAndView("error", model);
	}
```

- #### 处理json请求error

```java
@RequestMapping
	@ResponseBody
	public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
    // 这行代码调用完后body中会保存我们自定义的返回结果
		Map<String, Object> body = getErrorAttributes(request,
				isIncludeStackTrace(request, MediaType.ALL));
		HttpStatus status = getStatus(request);
		return new ResponseEntity<>(body, status);
	}
```

其中`getErrorAttributes`方法内实际调用我们自己实现的`GlobalErrorAttributes`中重写的`getErrorAttributes`方法。

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190522164616.png)

这里我们继承了`DefaultErrorAttributes`,返回的map中带上了我们自定义的一个`responseT`对象。

```java
@Component
public class GlobalErrorAttributes extends DefaultErrorAttributes {
    public GlobalErrorAttributes() {
    }

    public Map<String, Object> getErrorAttributes(WebRequest webRequest, boolean includeStackTrace) {
        boolean debug = webRequest.getParameter("debug") != null;
        ResponseT responseT = (ResponseT)webRequest.getAttribute("responseT", 0);
        Map<String, Object> errorMap = super.getErrorAttributes(webRequest, debug);
        if (responseT == null) {
            responseT = new ResponseT(ReturnCodeEnum.REQUEST_ERROR, (String)errorMap.get("error"), debug);
        }

        responseT.setUri((String)errorMap.get("path"));
        ResponseErrorMap<String, Object> dataMap = new ResponseErrorMap();
      // 设置自定义的返回结果responseT到方法返回值中
        dataMap.put("responseT", responseT);
        return dataMap;
    }
}
```

因此，在处理error请求的控制器里叫body的map中就存有`responseT`对象。

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190522165236.png)

所以最终返回的结果如下：

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190522165320.png)