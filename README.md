# Spring

## @JsonFormat、@DateTimeFormat、@JsonField 注意事项

### @JsonFormat

源于Jackson，需要引入对应包

#### 请求接收

<font color='red'>**用于`content-type为json`的请求参数时间格式化**</font>

将前端传入时间，如`2022-05-18 11:00:00`转换为时间格式

`@JsonFormat(pattern = “yyyy-MM-dd HH:mm:ss”)`

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

源于fastjson，需要额外引入包，请求和响应均可使用(application/json)