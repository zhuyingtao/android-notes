#### 单图片加载

- BitmapFactory

#### 多图片加载

使用内存缓存技术。内存缓存技术对那些大量占用应用程序宝贵内存的图片提供了快速访问的方法。其中最核心的类是LruCache (此类在android-support-v4的包中提供) 。这个类非常适合用来缓存图片，它的主要算法原理是把最近使用的对象用强引用存储在 LinkedHashMap 中，并且把最近最少使用的对象在缓存值达到预设定值之前从内存中移除。

## 开源库

#### Glide

源码地址：https://github.com/bumptech/glide

核心用法：

```java
// 加载本地图片
File file = new File(getExternalCacheDir() + "/image.jpg");
Glide.with(this).load(file).into(imageView);

// 加载应用资源
int resource = R.drawable.image;
Glide.with(this).load(resource).into(imageView);

// 加载二进制流
byte[] image = getImageBytes();
Glide.with(this).load(image).into(imageView);

// 加载Uri对象
Uri imageUri = getImageUri();
Glide.with(this).load(imageUri).into(imageView);
```

其实就是关键的三步走：先with()，再load()，最后into()。

具体解析： 

[Android图片加载框架最全解析——Glide](http://blog.csdn.net/guolin_blog/article/details/53759439)

[Google推荐的图片加载库Glide介绍](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0327/2650.html)

#### Picasso

#### Fresco

#### UniversalImageLoader

#### 