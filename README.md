# Spring

## @JsonFormat、@DateTimeFormat、@JsonField 注意事项

### @JsonFormat

源于Jackson，springboot默认序列化

#### 请求接收

<font color='red'>**用于`content-type为json`的请求参数时间格式化**</font>

将前端传入时间，如`2022-05-18 11:00:00或者时间戳`转换为时间格式，需要注意的是：如果不是时间戳，需要保持入参和pattern的格式一致；对于时间戳，不能接收字符串形式的时间戳，可以接收数字类型时间戳

`@JsonField对上述情况都能支持`

##### 踩坑1

请求

```json
{
	"startTime":"2022-05-18",
    "endTime":"2022-05-18"
}
```

接收

```java
@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss",timezone = "GMT+8")
private Date startTime;
```

异常

无法转换成带时分秒格式的

`Cannot deserialize value of type java.util.Date from String "2022-05-18": expected format "yyyy-MM-dd HH:mm:ss";`

**如果请求的是"2020-05-18 11:00:00" 接收的是 pattern = "yyyy-MM-dd" 是可以接收成功的**

**如果不配置@JsonFormat也是可以接收成功的，但是响应格式就需要另寻他法**



##### 踩坑2

请求

```json
{
	"startTime":"1652716800000",
    "endTime":"2022-05-18"
}
```

接收

```java
@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss",timezone = "GMT+8")
private Date startTime;
```

异常

无法转换字符串类型的时间戳

`Cannot deserialize value of type java.util.Date from String "1652716800000": expected format "yyyy-MM-dd HH:mm:ss";`

**如果是数字类型的时间戳可以转换成功**



#### 请求响应

@JsonFormat 默认的时区是Greenwich Time， 默认的是格林威治时间，而我们是在东八区上，所以时间会比实际我们想得到的时间少八个小时。

解决方式`@JsonFormat(pattern = “yyyy-MM-dd HH:mm:ss”, timezone = "GMT+8")`

### @DateTimeFormat

#### 请求接收

<font color='red'>**用于接收表单参数(键值对，非json)**</font>

将传入时间为 yyyy-MM-dd HH:mm:ss 形式的时间(字符串)转换为时间类型

如请求为

```json
{
	"time":"2020-05-18 11:00:00"
}
```

接收

```
@Data
public class Test {
	@DateTimeFormat("yyyy-MM-dd HH:mm:ss")
	private Date time;
}
```

<font color = 'red'>**如果传入时间格式非对应格式 如 yyyy/MM/dd HH/mm/ss 或者 yyyy-MM-dd之类的非对应格式， 则会解析失败**</font>



### @JsonField

源于fastjson，需要额外引入包并将其配置为默认的Json转换器，否则只能通过JSON、JSONObject进行序列化

请求和响应均可使用(application/json)

还可用于字段不匹配的转换映射

<font color='red'>**如果没有配置转换器，则还是使用默认的jackson进行请求响应**</font>

```xml
<dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.80</version>
</dependency>
```



```java
/**
 * fastJson替换JackJson
 */
@Configuration
public class HttpConverterConfig {
    @Bean
    public HttpMessageConverters fastJsonHttpMessageConverters() {
        // 1.定义一个converters转换消息的对象
        FastJsonHttpMessageConverter fastConverter = new FastJsonHttpMessageConverter();
        // 2.添加fastjson的配置信息，比如: 是否需要格式化返回的json数据
        FastJsonConfig fastJsonConfig = new FastJsonConfig();
        fastJsonConfig.setSerializerFeatures(SerializerFeature.PrettyFormat);
        // 3.在converter中添加配置信息
        fastConverter.setFastJsonConfig(fastJsonConfig);
        //处理中文乱码问题
        List<MediaType> oFastMediaTypeList = new ArrayList<>();
        oFastMediaTypeList.add(MediaType.APPLICATION_JSON_UTF8);
        fastConverter.setSupportedMediaTypes(oFastMediaTypeList);
        // 4.将converter赋值给HttpMessageConverter
	    // 5.返回HttpMessageConverters对象
        return new HttpMessageConverters(fastConverter);
    }
}
```

