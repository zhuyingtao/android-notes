## Android 事件分发机制

### 概述

#### 事件分发的对象

当用户触摸屏幕时，将产生点击事件（Touch 事件），Touch 事件的相关细节被封装成 MotionEvent 对象。

- 事件类型

  |         事件类型          |          具体动作          |
  | :-----------------------: | :------------------------: |
  |  MotionEvent.ACTION_DOWN  | 按下View（所有事件的开始） |
  |   MotionEvent.ACTION_UP   |   抬起View（与DOWN对应）   |
  |  MotionEvent.ACTION_MOVE  |          滑动View          |
  | MotionEvent.ACTION_CANCEL |   结束事件（非人为原因）   |

- 事件列

  从手指接触屏幕到手指离开屏幕，整个过程会产生一系列事件。一般情况下，事件列都是以 DOWN 事件开始，以 UP 事件结束，中间有无数个 MOVE 事件。

#### 事件分发的本质

将点击事件（MotionEvent）传递到某个 View 处理的整个过程

#### 事件分发的顺序

Activity -> ViewGroup -> View，即 1 个点击事件发生后，先传到 Activity，再传到 ViewGroup，最终传到 View。

#### 事件分发涉及的方法

| 方法                    | 作用               | 调用时刻                                                     |
| ----------------------- | ------------------ | ------------------------------------------------------------ |
| dispatchTouchEvent()    | 分发点击事件       | 当点击事件传递给当前 View 时，该方法就会被调用               |
| onTouchEvent()          | 处理点击事件       | 在 dispatchTouchEvent() 内部调用                             |
| onInterceptTouchEvent() | 判断是否拦截了事件 | 在 ViewGroup 的 dispatchTouchEvent() 内部调用，普通 View 无该方法 |

### 源码分析

Android 事件分发流程是 Activity -> ViewGroup -> View，所以要充分理解 Android 分发机制，本质上是了解这其中每一步的过程。

#### Activity 的事件分发机制

- Activity 事件分发相关源码

```java
		/**
 	 	 * 源码分析：Activity.dispatchTouchEvent（）
  	 */ 
    public boolean dispatchTouchEvent(MotionEvent ev) {

            // 一般事件列开始都是DOWN事件 = 按下事件，故此处基本是true
            if (ev.getAction() == MotionEvent.ACTION_DOWN) {
                onUserInteraction();
                // ->>分析1
            }

            // ->>分析2
            if (getWindow().superDispatchTouchEvent(ev)) {
                return true;
                // 若getWindow().superDispatchTouchEvent(ev)的返回true
                // 则Activity.dispatchTouchEvent（）就返回true，则方法结束。即 ：该点击事件停止往下传递 & 事件传递过程结束
                // 否则：继续往下调用Activity.onTouchEvent

            }
            // ->>分析4
            return onTouchEvent(ev);
        }


/**
  * 分析1：onUserInteraction()
  * 作用：实现屏保功能
  * 注：
  *    a. 该方法为空方法
  *    b. 当此activity在栈顶时，触屏点击按home，back，menu键等都会触发此方法
  */
      public void onUserInteraction() { 

      }
      // 回到最初的调用原处

/**
  * 分析2：getWindow().superDispatchTouchEvent(ev)
  * 说明：
  *     a. getWindow() = 获取Window类的对象
  *     b. Window类是抽象类，其唯一实现类 = PhoneWindow类；即此处的Window类对象 = PhoneWindow类对象
  *     c. Window类的superDispatchTouchEvent() = 1个抽象方法，由子类PhoneWindow类实现
  */
    @Override
    public boolean superDispatchTouchEvent(MotionEvent event) {

        return mDecor.superDispatchTouchEvent(event);
        // mDecor = 顶层View（DecorView）的实例对象
        // ->> 分析3
    }

/**
  * 分析3：mDecor.superDispatchTouchEvent(event)
  * 定义：属于顶层View（DecorView）
  * 说明：
  *     a. DecorView类是PhoneWindow类的一个内部类
  *     b. DecorView继承自FrameLayout，是所有界面的父类
  *     c. FrameLayout是ViewGroup的子类，故DecorView的间接父类 = ViewGroup
  */
    public boolean superDispatchTouchEvent(MotionEvent event) {

        return super.dispatchTouchEvent(event);
        // 调用父类的方法 = ViewGroup的dispatchTouchEvent()
        // 即 将事件传递到ViewGroup去处理，详细请看ViewGroup的事件分发机制

    }
    // 回到最初的调用原处

/**
  * 分析4：Activity.onTouchEvent（）
  * 定义：属于顶层View（DecorView）
  * 说明：
  *     a. DecorView类是PhoneWindow类的一个内部类
  *     b. DecorView继承自FrameLayout，是所有界面的父类
  *     c. FrameLayout是ViewGroup的子类，故DecorView的间接父类 = ViewGroup
  */
  public boolean onTouchEvent(MotionEvent event) {

        // 当一个点击事件未被Activity下任何一个View接收 / 处理时
        // 应用场景：处理发生在Window边界外的触摸事件
        // ->> 分析5
        if (mWindow.shouldCloseOnTouch(this, event)) {
            finish();
            return true;
        }
        
        return false;
        // 即 只有在点击事件在Window边界外才会返回true，一般情况都返回false，分析完毕
    }

/**
  * 分析5：mWindow.shouldCloseOnTouch(this, event)
  */
    public boolean shouldCloseOnTouch(Context context, MotionEvent event) {
    // 主要是对于处理边界外点击事件的判断：是否是DOWN事件，event的坐标是否在边界内等
    if (mCloseOnTouchOutside && event.getAction() == MotionEvent.ACTION_DOWN
            && isOutOfBounds(context, event) && peekDecorView() != null) {
        return true;
    }
    return false;
    // 返回true：说明事件在边界外，即 消费事件
    // 返回false：未消费（默认）
}
// 回到分析4调用原处
```

- Activity 事件分发流程图

  ![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWdjb252ZXJ0LmNzZG5pbWcuY24vYUhSMGNITTZMeTkxYzJWeUxXZHZiR1F0WTJSdUxuaHBkSFV1YVc4dk1qQXhPUzgyTHpFeEx6RTJZalEyTjJVMVlUUmlZVE13WmpF)



#### ViewGroup 的事件分发机制

从上面的 Activity 事件分发机制可知，ViewGroup 事件分发机制从 dispatchTouchEvent() 开始。

- ViewGroup 事件分发相关源码

  >  Android 5.0 之前源码，Android 5.0 之后源码有较大改动，但原理相同

  ```java
  /**
       * 源码分析：ViewGroup.dispatchTouchEvent（）
       */
      public boolean dispatchTouchEvent(MotionEvent ev) {
  
          // 仅贴出关键代码
          // 重点分析1：ViewGroup每次事件分发时，都需调用onInterceptTouchEvent()询问是否拦截事件
          if (disallowIntercept || !onInterceptTouchEvent(ev)) {
              // 判断值1：disallowIntercept = 是否禁用事件拦截的功能(默认是false)，可通过调用requestDisallowInterceptTouchEvent（）修改
              // 判断值2： !onInterceptTouchEvent(ev) = 对onInterceptTouchEvent()返回值取反
              // a. 若在onInterceptTouchEvent()中返回false（即不拦截事件），就会让第二个值为true，从而进入到条件判断的内部
              // b. 若在onInterceptTouchEvent()中返回true（即拦截事件），就会让第二个值为false，从而跳出了这个条件判断
              // c. 关于onInterceptTouchEvent() ->>分析1
  
              ev.setAction(MotionEvent.ACTION_DOWN);
              final int scrolledXInt = (int) scrolledXFloat;
              final int scrolledYInt = (int) scrolledYFloat;
              final View[] children = mChildren;
              final int count = mChildrenCount;
  
              // 重点分析2
              // 通过for循环，遍历了当前ViewGroup下的所有子View
              for (int i = count - 1; i >= 0; i--) {
                  final View child = children[i];
                  if ((child.mViewFlags & VISIBILITY_MASK) == VISIBLE
                          || child.getAnimation() != null) {
                      child.getHitRect(frame);
  
                      // 判断当前遍历的View是不是正在点击的View，从而找到当前被点击的View
                      // 若是，则进入条件判断内部
                      if (frame.contains(scrolledXInt, scrolledYInt)) {
                          final float xc = scrolledXFloat - child.mLeft;
                          final float yc = scrolledYFloat - child.mTop;
                          ev.setLocation(xc, yc);
                          child.mPrivateFlags &= ~CANCEL_NEXT_UP_EVENT;
  
                          // 条件判断的内部调用了该View的dispatchTouchEvent()
                          // 即 实现了点击事件从ViewGroup到子View的传递（具体请看下面的View事件分发机制）
                          if (child.dispatchTouchEvent(ev)) {
                              mMotionTarget = child;
                              return true;
                              // 调用子View的dispatchTouchEvent后是有返回值的
                              // 若该控件可点击，那么点击时，dispatchTouchEvent的返回值必定是true，因此会导致条件判断成立
                              // 于是给ViewGroup的dispatchTouchEvent（）直接返回了true，即直接跳出
                              // 即把ViewGroup的点击事件拦截掉
                          }
                      }
                  }
              }
          }
  
          boolean isUpOrCancel = (action == MotionEvent.ACTION_UP) ||
                  (action == MotionEvent.ACTION_CANCEL);
          if (isUpOrCancel) {
              mGroupFlags &= ~FLAG_DISALLOW_INTERCEPT;
          }
  
          final View target = mMotionTarget;
  
          // 重点分析3
          // 若点击的是空白处（即无任何View接收事件） / 拦截事件（手动复写onInterceptTouchEvent（），从而让其返回true）
          if (target == null) {
              ev.setLocation(xf, yf);
              if ((mPrivateFlags & CANCEL_NEXT_UP_EVENT) != 0) {
                  ev.setAction(MotionEvent.ACTION_CANCEL);
                  mPrivateFlags &= ~CANCEL_NEXT_UP_EVENT;
              }
  
              return super.dispatchTouchEvent(ev);
              // 调用ViewGroup父类的dispatchTouchEvent()，即View.dispatchTouchEvent()
              // 因此会执行ViewGroup的onTouch() ->> onTouchEvent() ->> performClick（） ->> onClick()，即自己处理该事件，事件不会往下传递（具体请参考View事件的分发机制中的View.dispatchTouchEvent（））
              // 此处需与上面区别：子View的dispatchTouchEvent（）
          }
  
          //...
  
      }
  
      /**
       * 分析1：ViewGroup.onInterceptTouchEvent()
       * 作用：是否拦截事件
       * 说明：
       * a. 返回true = 拦截，即事件停止往下传递（需手动设置，即复写onInterceptTouchEvent（），从而让其返回true）
       * b. 返回false = 不拦截（默认）
       */
      public boolean onInterceptTouchEvent(MotionEvent ev) {
          return false;
      }
      // 回到调用原处
  
  ```

- ViewGroup 事件分发流程图

  ![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWdjb252ZXJ0LmNzZG5pbWcuY24vYUhSMGNITTZMeTkxYzJWeUxXZHZiR1F0WTJSdUxuaHBkSFV1YVc4dk1qQXhPUzgyTHpFeEx6RTJZalEyTjJVMVlqY3haamMzTXpZ)

