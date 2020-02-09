### Android 动画

#### 逐帧动画

逐帧动画(frame-by-frame animation)，是将一个完整的动画拆分成一张张单独的图片，然后再将它们连贯起来进行播放。

#### 补间动画

补间动画(tweened animation)，是可以对View进行一系列的动画操作，包括淡入淡出、缩放、平移、旋转四种。

##### 使用场景

基础动画效果：

- 平移动画（Translate）
- 缩放动画（Scale）
- 旋转动画（Rotate）
- 透明度动画（Alpha）

特殊应用场景：

- Activity 的切换效果
- Fragment 的切换效果
- ViewGroup 中子元素的出场效果

##### 平移动画

对应的核心类是 TranslateAnimation 类。

- xml 设置方式

1. 在 res/anim 的文件夹里创建动画效果文件.xml
2. 根据不同动画效果的语法设置不同的动画参数，从而实现动画效果。

```xml
<?xml version="1.0" encoding="utf-8"?>
// 采用<translate /> 标签表示平移动画
<translate xmlns:android="http://schemas.android.com/apk/res/android"

    // 以下参数是4种动画效果的公共属性,即都有的属性
    android:duration="3000" // 动画持续时间（ms），必须设置，动画才有效果
    android:startOffset ="1000" // 动画延迟开始时间（ms）
    android:fillBefore = “true” // 动画播放完后，视图是否会停留在动画开始的状态，默认为true
    android:fillAfter = “false” // 动画播放完后，视图是否会停留在动画结束的状态，优先于fillBefore值，默认为false
    android:fillEnabled= “true” // 是否应用fillBefore值，对fillAfter值无影响，默认为true
    android:repeatMode= “restart” // 选择重复播放动画模式，restart代表正序重放，reverse代表倒序回放，默认为restart|
    android:repeatCount = “0” // 重放次数（所以动画的播放次数=重放次数+1），为infinite时无限重复
    android:interpolator = @[package:]anim/interpolator_resource // 插值器，即影响动画的播放速度,下面会详细讲
    
    // 以下参数是平移动画特有的属性
    android:fromXDelta="0" // 视图在水平方向x 移动的起始值
    android:toXDelta="500" // 视图在水平方向x 移动的结束值

    android:fromYDelta="0" // 视图在竖直方向y 移动的起始值
    android:toYDelta="500" // 视图在竖直方向y 移动的结束值
    /> 
```

3. 在 Java 代码中创建 Animation 对象并播放动画

```java
Button mButton = (Button) findViewById(R.id.Button);
// 步骤1:创建 需要设置动画的 视图View
Animation translateAnimation = AnimationUtils.loadAnimation(this, R.anim.view_animation);
// 步骤2:创建 动画对象 并传入设置的动画效果xml文件
mButton.startAnimation(translateAnimation);
// 步骤3:播放动画
```

- Java 设置方式

```java
Button mButton = (Button) findViewById(R.id.Button);
// 步骤1:创建 需要设置动画的 视图View

Animation translateAnimation = new TranslateAnimation(0，500，0，500);
// 步骤2：创建平移动画的对象：平移动画对应的Animation子类为TranslateAnimation
// 参数分别是：
// 1. fromXDelta ：视图在水平方向x 移动的起始值
// 2. toXDelta ：视图在水平方向x 移动的结束值
// 3. fromYDelta ：视图在竖直方向y 移动的起始值
// 4. toYDelta：视图在竖直方向y 移动的结束值

translateAnimation.setDuration(3000);
// 固定属性的设置都是在其属性前加“set”，如setDuration（）
mButton.startAnimation(translateAnimation);
// 步骤3:播放动画
```

##### 缩放动画

对应的核心类是 ScaleAnimation 类。

- xml 设置方式

1. 在 res/anim 的文件夹里创建动画效果文件.xml
2. 根据不同动画效果的语法设置不同的动画参数，从而实现动画效果。

```xml
<?xml version="1.0" encoding="utf-8"?>
// 采用<scale/> 标签表示是缩放动画
<scale  xmlns:android="http://schemas.android.com/apk/res/android"

    // 以下参数是4种动画效果的公共属性,即都有的属性
    android:duration="3000" // 动画持续时间（ms），必须设置，动画才有效果
    android:startOffset ="1000" // 动画延迟开始时间（ms）
    android:fillBefore = “true” // 动画播放完后，视图是否会停留在动画开始的状态，默认为true
    android:fillAfter = “false” // 动画播放完后，视图是否会停留在动画结束的状态，优先于fillBefore值，默认为false
    android:fillEnabled= “true” // 是否应用fillBefore值，对fillAfter值无影响，默认为true
    android:repeatMode= “restart” // 选择重复播放动画模式，restart代表正序重放，reverse代表倒序回放，默认为restart|
    android:repeatCount = “0” // 重放次数（所以动画的播放次数=重放次数+1），为infinite时无限重复
    android:interpolator = @[package:]anim/interpolator_resource // 插值器，即影响动画的播放速度,下面会详细讲
    
    // 以下参数是缩放动画特有的属性
    android:fromXScale="0.0" 
    // 动画在水平方向X的起始缩放倍数
    // 0.0表示收缩到没有；1.0表示正常无伸缩
    // 值小于1.0表示收缩；值大于1.0表示放大

    android:toXScale="2"  //动画在水平方向X的结束缩放倍数

    android:fromYScale="0.0" //动画开始前在竖直方向Y的起始缩放倍数
    android:toYScale="2" //动画在竖直方向Y的结束缩放倍数

    android:pivotX="50%" // 缩放轴点的x坐标
    android:pivotY="50%" // 缩放轴点的y坐标
    // 轴点 = 视图缩放的中心点

    // pivotX pivotY,可取值为数字，百分比，或者百分比p
    // 设置为数字时（如50），轴点为View的左上角的原点在x方向和y方向加上50px的点。在Java代码里面设置这个参数的对应参数是Animation.ABSOLUTE。
    // 设置为百分比时（如50%），轴点为View的左上角的原点在x方向加上自身宽度50%和y方向自身高度50%的点。在Java代码里面设置这个参数的对应参数是Animation.RELATIVE_TO_SELF。
    // 设置为百分比p时（如50%p），轴点为View的左上角的原点在x方向加上父控件宽度50%和y方向父控件高度50%的点。在Java代码里面设置这个参数的对应参数是Animation.RELATIVE_TO_PARENT

    // 两个50%表示动画从自身中间开始，具体如下图

    /> 

```

3. 在 Java 代码中创建 Animation 对象并播放动画

```java
Button mButton = (Button) findViewById(R.id.Button);
// 步骤1:创建 需要设置动画的 视图View
Animation rotateAnimation = AnimationUtils.loadAnimation(this, R.anim.view_animation);
// 步骤2:创建 动画对象 并传入设置的动画效果xml文件
mButton.startAnimation(scaleAnimation);
// 步骤3:播放动画

```

- Java 设置方式

```java
Button mButton = (Button) findViewById(R.id.Button);
// 步骤1:创建 需要设置动画的 视图View

Animation rotateAnimation = new ScaleAnimation(0,2,0,2,Animation.RELATIVE_TO_SELF,0.5f,Animation.RELATIVE_TO_SELF,0.5f);
// 步骤2：创建缩放动画的对象 & 设置动画效果：缩放动画对应的Animation子类为RotateAnimation
// 参数说明:
// 1. fromX ：动画在水平方向X的结束缩放倍数
// 2. toX ：动画在水平方向X的结束缩放倍数
// 3. fromY ：动画开始前在竖直方向Y的起始缩放倍数
// 4. toY：动画在竖直方向Y的结束缩放倍数
// 5. pivotXType:缩放轴点的x坐标的模式
// 6. pivotXValue:缩放轴点x坐标的相对值
// 7. pivotYType:缩放轴点的y坐标的模式
// 8. pivotYValue:缩放轴点y坐标的相对值

// pivotXType = Animation.ABSOLUTE:缩放轴点的x坐标 =  View左上角的原点 在x方向 加上 pivotXValue数值的点(y方向同理)
// pivotXType = Animation.RELATIVE_TO_SELF:缩放轴点的x坐标 = View左上角的原点 在x方向 加上 自身宽度乘上pivotXValue数值的值(y方向同理)
// pivotXType = Animation.RELATIVE_TO_PARENT:缩放轴点的x坐标 = View左上角的原点 在x方向 加上 父控件宽度乘上pivotXValue数值的值 (y方向同理)

scaleAnimation.setDuration(3000);
// 固定属性的设置都是在其属性前加“set”，如setDuration（）

mButton.startAnimation(scaleAnimation);
// 步骤3：播放动画

```

##### 旋转动画

对应的核心类是 RotateAnimation 类

- Xml 设置方式

```xml
<?xml version="1.0" encoding="utf-8"?>
// 采用<rotate/> 标签表示是旋转动画
<rotate xmlns:android="http://schemas.android.com/apk/res/android"

    // 以下参数是4种动画效果的公共属性,即都有的属性
    android:duration="3000" // 动画持续时间（ms），必须设置，动画才有效果
    android:startOffset ="1000" // 动画延迟开始时间（ms）
    android:fillBefore = “true” // 动画播放完后，视图是否会停留在动画开始的状态，默认为true
    android:fillAfter = “false” // 动画播放完后，视图是否会停留在动画结束的状态，优先于fillBefore值，默认为false
    android:fillEnabled= “true” // 是否应用fillBefore值，对fillAfter值无影响，默认为true
    android:repeatMode= “restart” // 选择重复播放动画模式，restart代表正序重放，reverse代表倒序回放，默认为restart|
    android:repeatCount = “0” // 重放次数（所以动画的播放次数=重放次数+1），为infinite时无限重复
    android:interpolator = @[package:]anim/interpolator_resource // 插值器，即影响动画的播放速度,下面会详细讲
    
    // 以下参数是旋转动画特有的属性
    android:duration="1000"
    android:fromDegrees="0" // 动画开始时 视图的旋转角度(正数 = 顺时针，负数 = 逆时针)
    android:toDegrees="270" // 动画结束时 视图的旋转角度(正数 = 顺时针，负数 = 逆时针)
    android:pivotX="50%" // 旋转轴点的x坐标
    android:pivotY="0" // 旋转轴点的y坐标
    // 轴点 = 视图缩放的中心点

// pivotX pivotY,可取值为数字，百分比，或者百分比p
    // 设置为数字时（如50），轴点为View的左上角的原点在x方向和y方向加上50px的点。在Java代码里面设置这个参数的对应参数是Animation.ABSOLUTE。
    // 设置为百分比时（如50%），轴点为View的左上角的原点在x方向加上自身宽度50%和y方向自身高度50%的点。在Java代码里面设置这个参数的对应参数是Animation.RELATIVE_TO_SELF。
    // 设置为百分比p时（如50%p），轴点为View的左上角的原点在x方向加上父控件宽度50%和y方向父控件高度50%的点。在Java代码里面设置这个参数的对应参数是Animation.RELATIVE_TO_PARENT
    // 两个50%表示动画从自身中间开始，具体如下图

    /> 

```

```java
Button mButton = (Button) findViewById(R.id.Button);
// 步骤1:创建 需要设置动画的 视图View
Animation scaleAnimation = AnimationUtils.loadAnimation(this, R.anim.view_animation);
// 步骤2:创建 动画对象 并传入设置的动画效果xml文件
mButton.startAnimation(scaleAnimation);
// 步骤3:播放动画

```

- Java 设置方式

```java
Button mButton = (Button) findViewById(R.id.Button);
// 步骤1:创建 需要设置动画的 视图View

Animation rotateAnimation = new RotateAnimation(0,270,Animation.RELATIVE_TO_SELF,0.5f,Animation.RELATIVE_TO_SELF,0.5f);
// 步骤2：创建旋转动画的对象 & 设置动画效果：旋转动画对应的Animation子类为RotateAnimation
// 参数说明:
// 1. fromDegrees ：动画开始时 视图的旋转角度(正数 = 顺时针，负数 = 逆时针)
// 2. toDegrees ：动画结束时 视图的旋转角度(正数 = 顺时针，负数 = 逆时针)
// 3. pivotXType：旋转轴点的x坐标的模式
// 4. pivotXValue：旋转轴点x坐标的相对值
// 5. pivotYType：旋转轴点的y坐标的模式
// 6. pivotYValue：旋转轴点y坐标的相对值

// pivotXType = Animation.ABSOLUTE:旋转轴点的x坐标 =  View左上角的原点 在x方向 加上 pivotXValue数值的点(y方向同理)
// pivotXType = Animation.RELATIVE_TO_SELF:旋转轴点的x坐标 = View左上角的原点 在x方向 加上 自身宽度乘上pivotXValue数值的值(y方向同理)
// pivotXType = Animation.RELATIVE_TO_PARENT:旋转轴点的x坐标 = View左上角的原点 在x方向 加上 父控件宽度乘上pivotXValue数值的值 (y方向同理)

rotateAnimation.setDuration(3000);
// 固定属性的设置都是在其属性前加“set”，如setDuration（）

mButton.startAnimation(rotateAnimation);
// 步骤3：播放动画

```

##### 透明度动画

对应的核心类是 AlphaAnimation 类

- xml 设置方式

```xml
<?xml version="1.0" encoding="utf-8"?>
// 采用<alpha/> 标签表示是透明度动画
<alpha xmlns:android="http://schemas.android.com/apk/res/android"

    // 以下参数是4种动画效果的公共属性,即都有的属性
    android:duration="3000" // 动画持续时间（ms），必须设置，动画才有效果
    android:startOffset ="1000" // 动画延迟开始时间（ms）
    android:fillBefore = “true” // 动画播放完后，视图是否会停留在动画开始的状态，默认为true
    android:fillAfter = “false” // 动画播放完后，视图是否会停留在动画结束的状态，优先于fillBefore值，默认为false
    android:fillEnabled= “true” // 是否应用fillBefore值，对fillAfter值无影响，默认为true
    android:repeatMode= “restart” // 选择重复播放动画模式，restart代表正序重放，reverse代表倒序回放，默认为restart|
    android:repeatCount = “0” // 重放次数（所以动画的播放次数=重放次数+1），为infinite时无限重复
    android:interpolator = @[package:]anim/interpolator_resource // 插值器，即影响动画的播放速度,下面会详细讲
    
    // 以下参数是透明度动画特有的属性
    android:fromAlpha="1.0" // 动画开始时视图的透明度(取值范围: -1 ~ 1)
    android:toAlpha="0.0"// 动画结束时视图的透明度(取值范围: -1 ~ 1)

    /> 

```

```java
Button mButton = (Button) findViewById(R.id.Button);
// 步骤1:创建 需要设置动画的 视图View
Animation alphaAnimation = AnimationUtils.loadAnimation(this, R.anim.view_animation);
// 步骤2:创建 动画对象 并传入设置的动画效果xml文件
mButton.startAnimation(alphaAnimation);
// 步骤3:播放动画

```

- Java 设置方式

```java
Button mButton = (Button) findViewById(R.id.Button);
// 步骤1:创建 需要设置动画的 视图View

Animation alphaAnimation = new AlphaAnimation(1,0);
// 步骤2：创建透明度动画的对象 & 设置动画效果：透明度动画对应的Animation子类为AlphaAnimation
// 参数说明:
// 1. fromAlpha:动画开始时视图的透明度(取值范围: -1 ~ 1)
// 2. toAlpha:动画结束时视图的透明度(取值范围: -1 ~ 1)

alphaAnimation.setDuration(3000);
// 固定属性的设置都是在其属性前加“set”，如setDuration（）

mButton.startAnimation(alphaAnimation);
// 步骤3：播放动画     

```

##### Activity 的切换动画

启动动画

```java
Intent intent = new Intent (this,Acvtivity.class);
startActivity(intent);
overridePendingTransition(R.anim.enter_anim,R.anim.exit_anim);
// 采用overridePendingTransition（int enterAnim, int exitAnim）进行设置
// enterAnim：从Activity a跳转到Activity b，进入b时的动画效果资源ID
// exitAnim：从Activity a跳转到Activity b，离开a时的动画效果资源Id

// 特别注意
// overridePendingTransition（）必须要在startActivity(intent)后被调用才能生效

```

退出动画

```java
@Override
public void finish(){
    super.finish();
    
    overridePendingTransition(R.anim.enter_anim,R.anim.exit_anim);
// 采用overridePendingTransition（int enterAnim, int exitAnim）进行设置
// enterAnim：从Activity a跳转到Activity b，进入b时的动画效果资源ID
// exitAnim：从Activity a跳转到Activity b，离开a时的动画效果资源Id

// 特别注意
// overridePendingTransition（）必须要在finish()后被调用才能生效
}

```

动画资源可以使用系统预设的，也可以使用自定义的。

系统自带的有`android.R.anmi.xxx`

自定义的方法如下

*fade_in.xml*

```xml
<?xml version="1.0" encoding="utf-8"?>  
<set xmlns:android="http://schemas.android.com/apk/res/android" >  
  
    <alpha  
        android:duration="1500"  
        android:fromAlpha="0.0"  
        android:toAlpha="1.0" />  
  
</set>  

```

##### Fragment 的切换动画

类似于 Activity，使用系统预设动画

```java
FragmentTransaction fragmentTransaction = mFragmentManager
                .beginTransaction();

fragmentTransaction.setTransition(int transit)；
// 通过setTransition(int transit)进行设置
// transit参数说明
// 1. FragmentTransaction.TRANSIT_NONE：无动画
// 2. FragmentTransaction.TRANSIT_FRAGMENT_OPEN：标准的打开动画效果
// 3. FragmentTransaction.TRANSIT_FRAGMENT_CLOSE：标准的关闭动画效果

// 标准动画设置好后，在Fragment添加和移除的时候都会有。

```

使用自定义动画效果

```java
// 采用FragmentTransavtion的 setCustomAnimations（）进行设置

FragmentTransaction fragmentTransaction = mFragmentManager
                .beginTransaction();

fragmentTransaction.setCustomAnimations(
                R.anim.in_from_right,
                R.anim.out_to_left);

// 此处的自定义动画效果同Activity，此处不再过多描述

```

##### ViewGroup 中子元素的出场效果

1. 设置子元素的出场动画
2. 设置 ViewGroup 的动画文件

```xml
<?xml version="1.0" encoding="utf-8"?>
// 采用LayoutAnimation标签
<layoutAnimation xmlns:android="http://schemas.android.com/apk/res/android"
    android:delay="0.5"
    // 子元素开始动画的时间延迟
    // 如子元素入场动画的时间总长设置为300ms
    // 那么 delay = "0.5" 表示每个子元素都会延迟150ms才会播放动画效果
    // 第一个子元素延迟150ms播放入场效果；第二个延迟300ms，以此类推

    android:animationOrder="normal"
    // 表示子元素动画的顺序
    // 可设置属性为：
    // 1. normal ：顺序显示，即排在前面的子元素先播放入场动画
    // 2. reverse：倒序显示，即排在后面的子元素先播放入场动画
    // 3. random：随机播放入场动画

    android:animation="@anim/view_animation"
    // 设置入场的具体动画效果
    // 将步骤1的子元素出场动画设置到这里
    />

```

3. 为 ViewGroup 指定动画效果

```java
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#FFFFFF"
    android:orientation="vertical" >
    <ListView
        android:id="@+id/listView1"
        android:layoutAnimation="@anim/anim_layout"
        // 指定layoutAnimation属性用以指定子元素的入场动画
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
</LinearLayout>

```

##### 高级使用

- 组合动画

- 监听动画

  Animation 类通过监听动画开始/结束/重复来进行一系列操作

  ```java
  
  Animation.setAnimationListener(new Animation.AnimationListener() {
              @Override
              public void onAnimationStart(Animation animation) {
                  // 动画开始时回调
              }
  
              @Override
              public void onAnimationEnd(Animation animation) {
                  // 动画结束时回调
              }
  
              @Override
              public void onAnimationRepeat(Animation animation) {
                  //动画重复执行的时候回调
              }
          });
  
  
  ```

- 插值器

  插值器的作用是确定动画从初始值过渡到结束值的变化规律，也就是确定了动画效果变化的模式，如匀速、加速、减速等。它实现了非线性运动的动画效果。

  插值器设置方式

  xml 设置方式

  ```xml
  android:interpolator="@android:anim/overshoot_interpolator"
  ```

  java 设置方式

  ```java
  Interpolator overshootInterpolator = new OvershootInterpolator();
  alphaAnimation.setInterpolator(overshootInterpolator);
  ```

- 估值器

  插值器决定动画变化规律，估值器决定其中具体的数值，用来协助插值器实现非线性动画效果。

  估值器是属性动画特有的属性。

  估值器设置方式

  ```java
  ObjectAnimator anim = ObjectAnimator.ofObject(myView2, "height", new Evaluator()，1，3);
  // 在第4个参数中传入对应估值器类的对象
  // 系统内置的估值器有3个：
  // IntEvaluator：以整型的形式从初始值 - 结束值 进行过渡
  // FloatEvaluator：以浮点型的形式从初始值 - 结束值 进行过渡
  // ArgbEvaluator：以Argb类型的形式从初始值 - 结束值 进行过渡
  ```

#### 属性动画

属性动画(property animation)，它的功能非常强大，弥补了之前补间动画的一些缺陷，几乎是可以完全替代掉补间动画了。

- ValueAnimator

ValueAnimator是整个属性动画机制当中最核心的一个类。属性动画的运行机制是通过不断地对值进行操作来实现的，而初始值和结束值之间的动画过渡就是由ValueAnimator这个类来负责计算的。

```java
ValueAnimator anim = ValueAnimator.ofFloat(0f, 1f);  
anim.setDuration(300);  
anim.start();  
```

- ObjectAnimator

相比于ValueAnimator，ObjectAnimator可能才是我们最常接触到的类。因为ValueAnimator只不过是对值进行了一个平滑的动画过渡，而 ObjectAnimator可以直接对任意对象的任意属性进行动画操作的，比如说View的alpha属性。

ObjectAnimator虽然更加常用一些，但是它其实是继承自ValueAnimator的，底层的动画实现机制也是基于ValueAnimator来完成的。

```java
ObjectAnimator animator = ObjectAnimator.ofFloat(textview, "rotation", 0f, 360f);  
animator.setDuration(5000);  
animator.start();  
```

