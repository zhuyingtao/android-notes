### Android 网络

Android系统中主要提供了两种方式来进行HTTP通信，HttpURLConnection和HttpClient，不过HttpURLConnection和HttpClient的用法还是稍微有些复杂的，如果不进行适当封装的话，很容易就会写出不少重复代码。

#### Volley

Volley 是 2013年Google I/O大会上推出了一个新的网络通信框架。除了简单易用之外，Volley在性能方面也进行了大幅度的调整，它的设计目标就是非常适合去进行**数据量不大，但通信频繁的网络操作**，而对于大数据量的网络操作，比如说下载文件等，Volley的表现就会非常糟糕。

- StringRequest

Volley 的用法非常简单，首先需要获取到一个RequestQueue对象，可以调用如下方法获取到：

```java
RequestQueue mQueue = Volley.newRequestQueue(context); 
```

这里拿到的RequestQueue是一个请求队列对象，它可以缓存所有的HTTP请求，然后按照一定的算法并发地发出这些请求。

接下来为了要发出一条HTTP请求，我们还需要创建一个StringRequest对象：

```java
StringRequest stringRequest = new StringRequest("http://www.baidu.com",  
                        new Response.Listener<String>() {  
                            @Override  
                            public void onResponse(String response) {  
                                Log.d("TAG", response);  
                            }  
                        }, new Response.ErrorListener() {  
                            @Override  
                            public void onErrorResponse(VolleyError error) {  
                                Log.e("TAG", error.getMessage(), error);  
                            }  
                        });  
```

最后，将这个StringRequest对象添加到RequestQueue里面就可以了。

```java
mQueue.add(stringRequest); 
```

如果要发送 POST 请求，则需要重写父类的getParams()方法：

```java
StringRequest stringRequest = new StringRequest(Method.POST, url,  listener, errorListener) {  
    @Override  
    protected Map<String, String> getParams() throws AuthFailureError {  
        Map<String, String> map = new HashMap<String, String>();  
        map.put("params1", "value1");  
        map.put("params2", "value2");  
        return map;  
    }  
};  
```

- JsonRequest

JsonRequest是一个抽象类，有两个直接的子类，JsonObjectRequest和JsonArrayRequest。

```java
JsonObjectRequest jsonObjectRequest = new JsonObjectRequest("http://m.weather.com.cn/data/101010100.html", null,  
        new Response.Listener<JSONObject>() {  
            @Override  
            public void onResponse(JSONObject response) {  
                Log.d("TAG", response.toString());  
            }  
        }, new Response.ErrorListener() {  
            @Override  
            public void onErrorResponse(VolleyError error) {  
                Log.e("TAG", error.getMessage(), error);  
            }  
        });  
```

- ImageRequest

```java
ImageRequest imageRequest = new ImageRequest(  
        "http://developer.android.com/images/home/aw_dac.png",  
        new Response.Listener<Bitmap>() {  
            @Override  
            public void onResponse(Bitmap response) {  
                imageView.setImageBitmap(response);  
            }  
        }, 0, 0, Config.RGB_565, new Response.ErrorListener() {  
            @Override  
            public void onErrorResponse(VolleyError error) {  
                imageView.setImageResource(R.drawable.default_image);  
            }  
        });  
```

除了基本的ImageRequest外，Volley 还提供了更高级的 ImageLoader 和 NetworkImageView

#### 开源库

[Retrofit](https://github.com/square/retrofit)