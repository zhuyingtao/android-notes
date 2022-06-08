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

  > 采用 Android 5.0 之前源码，Android 5.0 之后源码有较大改动，但原理相同

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

#### View 的事件分发机制

从上面 ViewGroup 的事件分发机制可以看出，View 事件分发机制是从 dispatchTouchEvent() 开始的。

- View 事件分发相关源码

  >  采用 Android 5.0 之前源码，Android 5.0 之后源码有较大改动，但原理相同

  ```java
  /**
       * 源码分析：View.dispatchTouchEvent()
       */
      public boolean dispatchTouchEvent(MotionEvent event) {
          if (mOnTouchListener != null && (mViewFlags & ENABLED_MASK) == ENABLED &&
                  mOnTouchListener.onTouch(this, event)) {
              return true;
          }
          return onTouchEvent(event);
      }
      // 说明：只有以下3个条件都为真，dispatchTouchEvent()才返回true；否则执行onTouchEvent()
      //     1. mOnTouchListener != null
      //     2. (mViewFlags & ENABLED_MASK) == ENABLED
      //     3. mOnTouchListener.onTouch(this, event)
      // 下面对这3个条件逐个分析
  
      /**
       * 条件1：mOnTouchListener != null
       * 说明：mOnTouchListener变量在View.setOnTouchListener（）方法里赋值
       */
      public void setOnTouchListener(OnTouchListener l) {
          mOnTouchListener = l;
          // 即只要我们给控件注册了Touch事件，mOnTouchListener就一定被赋值（不为空）
      }
  
      /**
       * 条件2：(mViewFlags & ENABLED_MASK) == ENABLED
       * 说明：
       *     a. 该条件是判断当前点击的控件是否enable
       *     b. 由于很多View默认enable，故该条件恒定为true
       */
  
      /**
       * 条件3：mOnTouchListener.onTouch(this, event)
       * 说明：即 回调控件注册Touch事件时的onTouch（）；需手动复写设置，具体如下（以按钮Button为例）
       */
      public void sample() {
          button.setOnTouchListener(new View.OnTouchListener() {
              @Override
              public boolean onTouch(View v, MotionEvent event) {
                  return false;
              }
          });
          // 若在onTouch（）返回true，就会让上述三个条件全部成立，从而使得View.dispatchTouchEvent（）直接返回true，事件分发结束
          // 若在onTouch（）返回false，就会使得上述三个条件不全部成立，从而使得View.dispatchTouchEvent（）中跳出If，执行onTouchEvent(event)
      }
  ```

  接下来，继续看 onTouchEvent(MotionEvent) 的源码分析

  ```java
  /**
       * 源码分析：View.onTouchEvent（）
       */
      public boolean onTouchEvent(MotionEvent event) {
          final int viewFlags = mViewFlags;
  
          if ((viewFlags & ENABLED_MASK) == DISABLED) {
  
              return (((viewFlags & CLICKABLE) == CLICKABLE ||
                      (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE));
          }
          if (mTouchDelegate != null) {
              if (mTouchDelegate.onTouchEvent(event)) {
                  return true;
              }
          }
  
          // 若该控件可点击，则进入switch判断中
          if (((viewFlags & CLICKABLE) == CLICKABLE ||
                  (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)) {
  
              switch (event.getAction()) {
  
                  // a. 若当前的事件 = 抬起View（主要分析）
                  case MotionEvent.ACTION_UP:
                      boolean prepressed = (mPrivateFlags & PREPRESSED) != 0;
  
                              ...// 经过种种判断，此处省略
  
                      // 执行performClick() ->>分析1
                      performClick();
                      break;
  
                  // b. 若当前的事件 = 按下View
                  case MotionEvent.ACTION_DOWN:
                      if (mPendingCheckForTap == null) {
                          mPendingCheckForTap = new CheckForTap();
                      }
                      mPrivateFlags |= PREPRESSED;
                      mHasPerformedLongPress = false;
                      postDelayed(mPendingCheckForTap, ViewConfiguration.getTapTimeout());
                      break;
  
                  // c. 若当前的事件 = 结束事件（非人为原因）
                  case MotionEvent.ACTION_CANCEL:
                      mPrivateFlags &= ~PRESSED;
                      refreshDrawableState();
                      removeTapCallback();
                      break;
  
                  // d. 若当前的事件 = 滑动View
                  case MotionEvent.ACTION_MOVE:
                      final int x = (int) event.getX();
                      final int y = (int) event.getY();
  
                      int slop = mTouchSlop;
                      if ((x < 0 - slop) || (x >= getWidth() + slop) ||
                              (y < 0 - slop) || (y >= getHeight() + slop)) {
                          // Outside button
                          removeTapCallback();
                          if ((mPrivateFlags & PRESSED) != 0) {
                              // Remove any future long press/tap checks
                              removeLongPressCallback();
                              // Need to switch from pressed to not pressed
                              mPrivateFlags &= ~PRESSED;
                              refreshDrawableState();
                          }
                      }
                      break;
              }
              // 若该控件可点击，就一定返回true
              return true;
          }
          // 若该控件不可点击，就一定返回false
          return false;
      }
  
      /**
       * 分析1：performClick（）
       */
      public boolean performClick() {
          if (mOnClickListener != null) {
              playSoundEffect(SoundEffectConstants.CLICK);
              mOnClickListener.onClick(this);
              return true;
              // 只要我们通过setOnClickListener（）为控件View注册1个点击事件
              // 那么就会给mOnClickListener变量赋值（即不为空）
              // 则会往下回调onClick（） & performClick（）返回true
          }
          return false;
      }
  ```

- View 事件分发机制流程图

  ![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWdjb252ZXJ0LmNzZG5pbWcuY24vYUhSMGNITTZMeTkxYzJWeUxXZHZiR1F0WTJSdUxuaHBkSFV1YVc4dk1qQXhPUzgyTHpFeEx6RTJZalEyTjJVMVpUQmhabVEzTldZ)

#### 额外知识

1. Touch 事件的后续事件（MOVE/UP）层级传递

   当 dispatchTouchEvent() 事件分发时，只有前一个事件（如 ACTION_DOWN ）返回 true，才会收到后一个事件（如 ACTION_MOVE/ACTION_UP）如果执行事件时返回 false，则后面的一系列事件都不会执行了。

2. onTouch() 和 onTouchEvent() 的区别

   这 2 个方法都是在 View.dispatchTouchEvent() 中调用。

   但 onTouch() 优先于 onTouchEvent() 执行，如果手动复写在 onTouch() 中返回 true，则不再执行 onTouchEvent()

### 滑动冲突

#### 场景

产生滑动冲突的场景主要有两种:

- 父ViewGroup和子View的滑动方向一致
- 父ViewGroup和子View的滑动方向不一致

#### 原因

ViewGroup的**onInterceptTouchEvent**方法默认情况下是返回false，也就是ViewGroup默认情况下是不会拦截事件的。当ViewGroup接收到事件时，由于不拦截事件，会去寻找能够处理事件的子View。此时，一旦子View处理了DOWN事件，默认情况下接下来同一事件序列的其他事件都交由子View处理，此时可以看到的效果是子View可以滑动，但是父ViewGroup始终滑动不了，此时滑动冲突就出现了。

#### 解决方式

- 外部拦截法

  所谓外部拦截法，就是当事件传递到父容器时，通过父容器去判断自己是否需要此事件，若需要则拦截事件，不需要则不拦截事件，将事件传递给子View。

  以 ViewPager 和 ListView 滑动冲突为例，我们要的效果是在水平方向上滑动时ViewPager可以水平滚动，在竖直方向上滑动时，ListView可以滚动但ViewPager不动。

  ```java
  @Override
  public boolean onInterceptTouchEvent(MotionEvent ev) {
      switch (ev.getAction()) {
          case MotionEvent.ACTION_MOVE:
              if (ViewPager需要此事件) {
                  return true;
              }
              break;
          default:
              break;
      }
      return false;
  }
  ```
