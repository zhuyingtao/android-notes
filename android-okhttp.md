## OkHttp 深入理解

### OkHttp 请求流程

OkHttp 请求流程如下所示：

![img](assets/android-okhttp/okhttp.png)

如下为使用OkHttp进行Get请求的步骤：

```java
//1.新建OkHttpClient客户端
OkHttpClient client = new OkHttpClient();
//新建一个Request对象
Request request = new Request.Builder()
        .url(url)
        .build();
//2.Response为OKHttp中的响应
Response response = client.newCall(request).execute();
```

