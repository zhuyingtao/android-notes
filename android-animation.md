#### 逐帧动画

逐帧动画(frame-by-frame animation)，是将一个完整的动画拆分成一张张单独的图片，然后再将它们连贯起来进行播放。

#### 补间动画

补间动画(tweened animation)，是可以对View进行一系列的动画操作，包括淡入淡出、缩放、平移、旋转四种。

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

