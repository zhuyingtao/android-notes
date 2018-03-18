#### 反编译

是指对APK文件进行反编译。Android的反编译主要又分为两个部分，一个是对代码的反编译，一个是对资源的反编译。

##### 反编译代码

要想将APK文件中的代码反编译出来，我们需要用到以下两款工具：

- **dex2jar** 这个工具用于将dex文件转换成jar文件 
  下载地址：<http://sourceforge.net/projects/dex2jar/files/>
- **jd-gui** 这个工具用于将jar文件转换成java代码 
  下载地址：<http://jd.benow.ca/>

##### 反编译资源

直接对APK包进行解压是无法得到它的原始资源文件的，因此我们还需要对资源进行反编译才行。 

要想将APK文件中的资源反编译出来，又要用到另外一个工具了：

- **apktool** 这个工具用于最大幅度地还原APK文件中的9-patch图片、布局、字符串等等一系列的资源。 
  下载地址：<http://ibotpeaches.github.io/Apktool/install/>

##### 文章：

[Android APK 反编译实践](https://www.jianshu.com/p/9e0d1c3e342e)

[Android 反编译利器，jadx 的高级技巧](https://segmentfault.com/a/1190000012180752)

##### 工具：

- [jadx](https://github.com/skylot/jadx)

#### 代码混淆

混淆代码并不是让代码无法被反编译，而是将代码中的类、方法、变量等信息进行重命名，把它们改成一些毫无意义的名字。

在Android Studio当中混淆APK，只需要修改build.gradle中的一行配置即可。

```groovy
release {
    minifyEnabled true
    proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
}
```

其中minifyEnabled用于设置是否启用混淆，proguardFiles用于选定混淆配置文件。

minifyEnabled除了混淆代码之外，还可以起到压缩APK包的作用。

一份默认的混淆配置文件如下：

```properties
# This is a configuration file for ProGuard.
# http://proguard.sourceforge.net/index.html#manual/usage.html

-dontusemixedcaseclassnames
-dontskipnonpubliclibraryclasses
-verbose

# Optimization is turned off by default. Dex does not like code run
# through the ProGuard optimize and preverify steps (and performs some
# of these optimizations on its own).
-dontoptimize
-dontpreverify
# Note that if you want to enable optimization, you cannot just
# include optimization flags in your own project configuration file;
# instead you will need to point to the
# "proguard-android-optimize.txt" file instead of this one from your
# project.properties file.

-keepattributes *Annotation*
-keep public class com.google.vending.licensing.ILicensingService
-keep public class com.android.vending.licensing.ILicensingService

# For native methods, see http://proguard.sourceforge.net/manual/examples.html#native
-keepclasseswithmembernames class * {
    native <methods>;
}

# keep setters in Views so that animations can still work.
# see http://proguard.sourceforge.net/manual/examples.html#beans
-keepclassmembers public class * extends android.view.View {
   void set*(***);
   *** get*();
}

# We want to keep methods in Activity that could be used in the XML attribute onClick
-keepclassmembers class * extends android.app.Activity {
   public void *(android.view.View);
}

# For enumeration classes, see http://proguard.sourceforge.net/manual/examples.html#enumerations
-keepclassmembers enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}

-keepclassmembers class * implements android.os.Parcelable {
  public static final android.os.Parcelable$Creator CREATOR;
}

-keepclassmembers class **.R$* {
    public static <fields>;
}

# The support library contains references to newer platform versions.
# Dont warn about those in case this app is linking against an older
# platform version.  We know about them, and they are safe.
-dontwarn android.support.**
```

##### 混淆规则

proguard中一共有三组六个keep关键字，分别的作用是：

| 关键字                        | 描述                                       |
| -------------------------- | ---------------------------------------- |
| keep                       | 保留类和类中的成员，防止它们被混淆或移除。                    |
| keepnames                  | 保留类和类中的成员，防止它们被混淆，但当成员没有被引用时会被移除。        |
| keepclassmembers           | 只保留类中的成员，防止它们被混淆或移除。                     |
| keepclassmembernames       | 只保留类中的成员，防止它们被混淆，但当成员没有被引用时会被移除。         |
| keepclasseswithmembers     | 保留类和类中的成员，防止它们被混淆或移除，前提是指名的类中的成员必须存在，如果不存在则还是会混淆。 |
| keepclasseswithmembernames | 保留类和类中的成员，防止它们被混淆，但当成员没有被引用时会被移除，前提是指名的类中的成员必须存在，如果不存在则还是会混淆。 |

proguard 中通配符的作用：

| 通配符      | 描述                                       |
| -------- | ---------------------------------------- |
| <field>  | 匹配类中的所有字段                                |
| <method> | 匹配类中的所有方法                                |
| <init>   | 匹配类中的所有构造函数                              |
| *        | 匹配任意长度字符，但不含包名分隔符(.)。比如说我们的完整类名是com.example.test.MyActivity，使用com.*，或者com.exmaple.*都是无法匹配的，因为*无法匹配包名中的分隔符，正确的匹配方式是com.exmaple.*.*，或者com.exmaple.test.*，这些都是可以的。但如果你不写任何其它内容，只有一个*，那就表示匹配所有的东西。 |
| **       | 匹配任意长度字符，并且包含包名分隔符(.)。比如proguard-android.txt中使用的-dontwarn android.support.**就可以匹配android.support包下的所有内容，包括任意长度的子包。 |
| ***      | 匹配任意参数类型。比如void set*(***)就能匹配任意传入的参数类型，*** get*()就能匹配任意返回值的类型。 |
| …        | 匹配任意长度的任意类型参数。比如void test(…)就能匹配任意void test(String a)或者是void test(int a, String b)这些方法。 |

