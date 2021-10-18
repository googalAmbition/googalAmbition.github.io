# Spring多次request.getReader()解决方法

> `ServletRequest`的`getReader`和`getInputStream`只能调用一次


开发中遇到一个需求,对http请求进行打点,其中有个字段是请求body

本来想的是写一个拦截器拦截所有请求,读取请求中的body,然后写入MQ

```java
        StringBuilder data = new StringBuilder();
        String line;
        BufferedReader reader;
        reader = request.getReader();
        while (null != (line = reader.readLine())) {
            data.append(line);
        }
```



然后报错了`requesst.getReader`只能调用一次,因为`@RequestBody`中读取过一次,拦截器中在读取一次会报错.


网上有说重写`HttpServletRequestWrapper`备份

多方查找发现Spring自带了一个对`HttpServletRequestWrapper`的重写(`org.springframework.web.util.ContentCachingRequestWrapper`),可以满足需求,记录一下解决方法.


下面demo

编写filter,把request设置成ContentCachingRequestWrapper

```java
import org.springframework.web.filter.GenericFilterBean;
import org.springframework.web.util.ContentCachingRequestWrapper;
import java.io.IOException;
import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;
public class CachingRequestBodyFilter extends GenericFilterBean {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
        throws IOException, ServletException {
        HttpServletRequest currentRequest = (HttpServletRequest) request;
        ContentCachingRequestWrapper wrappedRequest = new ContentCachingRequestWrapper(currentRequest);
        chain.doFilter(wrappedRequest, response);
    }
}
```

使filter生效

```java
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
@Configuration
public class FilterConfig {

    @Bean
    public FilterRegistrationBean<CachingRequestBodyFilter> filterRegistration() {
        FilterRegistrationBean<CachingRequestBodyFilter> registration = new FilterRegistrationBean<>(new CachingRequestBodyFilter());
        registration.setOrder(2);
        return registration;
    }
}
```

使用

```java
    ContentCachingRequestWrapper wrapper = org.springframework.web.util.WebUtils.getNativeRequest(request,ContentCachingRequestWrapper.class);
    if(Objects.isNull(wrapper)){
        return null;
    }
    String s=new String(wrapper.getContentAsByteArray());

```


### 注意
ContentCachingRequestWrapper 只有request调用过`getReader`或`getInputStream`时才`ContentAsByteArray`有数据,
当controller没有`@RequestBody`时使用`ContentCachingRequestWrapper`是读取不到数据的,
需要直接调用处理

```java
        StringBuilder data = new StringBuilder();
        String line;
        BufferedReader reader;
        reader = request.getReader();
        while (null != (line = reader.readLine())) {
            data.append(line);
        }
```


