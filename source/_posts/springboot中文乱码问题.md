---
title: springboot中文乱码问题
date: 2019-07-02 17:52:10
tags:
    -中文乱码
---
# springboot中文乱码问题
## 统一utf-8编码
```
spring.http.encoding.force=true
spring.http.encoding.charset=UTF-8
spring.http.encoding.enabled=true
```
## 自定义web配置转码

```
 @Bean
    public HttpMessageConverter<String> responseBodyConverter(){
        StringHttpMessageConverter converter = new StringHttpMessageConverter(Charset.forName("UTF-8"));
        return converter;
    }


    @Bean
    public ObjectMapper getObjectMapper() {
        return new ObjectMapper();
    }


    @Bean
    public MappingJackson2HttpMessageConverter messageConverter() {
        MappingJackson2HttpMessageConverter converter = new MappingJackson2HttpMessageConverter();
        converter.setObjectMapper(getObjectMapper());
        return converter;
    }

    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        
        //解决中文乱码
        converters.add(responseBodyConverter());
        //解决 添加解决中文乱码后 上述配置之后，返回json数据直接报错 500：no convertter for return value of type
        converters.add(messageConverter());
        super.configureMessageConverters(converters);
    }
```
## 设置Tomcat编码
```
   // factory.setUriEncoding(Charset.forName("UTF-8"));
      factory.setUriEncoding(Charset.defaultCharset());
```
## 设置rest编码
```
     // MediaType type = MediaType.parseMediaType("application/x-www-form-urlencoded; charset=UTF-8");
        //headers.setContentType(type);
        //headers.setContentType(MediaType.APPLICATION_FORM_URLENCODED) ;
```
## jar 运行时设置编码
```
 java -Dfile.encoding=UTF-8  -jar
```