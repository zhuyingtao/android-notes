## Android 

### 一、基础概念

#### 显示系统

在一个典型的显示系统中，一般包括CPU、GPU、Display三个部分

- CPU负责计算帧数据，把计算好的数据交给GPU
- GPU会对图形数据进行渲染，渲染好后放到buffer(图像缓冲区)里存起来
- Display（屏幕或显示器）负责把buffer里的数据呈现到屏幕上



- 屏幕刷新率

  一秒内屏幕刷新的次数（一秒内显示了多少帧的图像），单位 Hz，如常见的 60 Hz。**刷新频率取决于硬件的固定参数**（不会变的）。

- 逐行扫描

  显示器并不是一次性将画面显示到屏幕上，而是从左到右边，从上到下逐行扫描，顺序显示整屏的一个个像素点，不过这一过程快到人眼无法察觉到变化。以 60 Hz 刷新率的屏幕为例，这一过程即 1000 / 60 ≈ 16ms。

- 帧率

  一秒内 GPU 绘制操作的帧数，单位 fps，帧率是动态变化的，例如当画面静止时，GPU 是没有绘制操作的，屏幕刷新的还是buffer中的数据，即GPU最后操作的帧数据。

- 画面撕裂

  一个屏幕内的数据来自2个不同的帧，画面会出现撕裂感









### 四、Choreographer

#### 1. 概述

Google 在 Android 4.1系统中对 Android Display 系统进行了优化：在收到 VSync pulse 后，将马上开始下一帧的渲染。即一旦收到VSync通知，CPU和GPU就立刻开始计算然后把数据写入buffer。本节就来讲 “drawing with VSync” 的实现——Choreographer。

- Choreographer，本意为 舞蹈编导、编舞者。在这里就是指 对CPU/GPU绘制的指导—— 收到VSync信号才开始绘制，保证绘制拥有完整的16.6ms，避免绘制的随机性。
- Choreographer，是一个Java类，包路径为 android.view.Choreographer。协调动画、输入和绘制的时机。
- 通常应用层不会直接使用Choreographer，而是使用更高级的API，例如动画和View绘制相关的ValueAnimator.start()、View.invalidate()等。
- 业界一般通过Choreographer来监控应用的帧率。

#### 2. 源码分析









参考资料：https://blog.csdn.net/hfy8971613/article/details/108041504