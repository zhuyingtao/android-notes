## Android测试

Android 的测试种类:

- Unit Test（单元测试）
  - Local Tests —— 用来测试无关 Android 平台的代码。
  - Instrument Tests —— 运行在 Android 系统中, 这些测试可以获取到测试应用的上下文信息，用来测试有 Android API 的代码。
- Integration Test（集成测试）

Android Studio 中的测试

- Local Test 的默认测试文件夹在 `src/test/java`。
- Instrument Test 的默认测试文件夹在`src/androidTest/java`。

### 单元测试

#### 1. 是什么？

单元测试，是为了测试某一个类的某一个方法能否正常工作，而写的测试代码。写单元测试，就是给你的每个类的每个public方法写对应的测试方法，非public方法是这个类的实现细节，我们并不关心，我们只关心某一个public方法的输入、输出。

- 减少bug
- 快速定位bug
- 放心重构，便于解耦，提高代码质量

#### 2. 怎么写？
一般来说，一个方法对应的测试方法主要分为3部分：

1. setup。一般是new出你要测试的那个类，以及其他一些前提条件的设置：`Calculator calculator = new Calculator();`
2. 执行操作。一般是调用你要测试的那个方法，获得运行结果：`int sum = calculator.add(1, 2);`
3. 验证结果。验证得到的结果跟预期中是一样的：`Assert.assertEquals(3, sum);`

- 在 Android Studio 中写单元测试

#### 3. 区别

单元测试只是测试一个方法单元，它不是测试一整个流程。这种整个流程的测试叫做集成测试。

一个类的方法可以分为两种，一种是有返回值的，另一种是没有返回值的。有返回值用 JUnit，无返回值的用Mockito。

#### 4. JUnit 用法

#### 5. Mockito 用法

[Mockito 框架的使用](https://blog.csdn.net/qq_17766199/article/details/78450007)

#### 6. 基本测试框架

- Java单元测试框架：**Junit、Mockito、Powermockito**
- Android单元测试框架：**Robolectric、AndroidJUnitRunner、Espresso、UI Automator**

- AndroidJUnitRunner
> AndroidJUnitRunner 类是一个 JUnit 测试运行器，可让您在 Android 设备上运行 JUnit 3 或 JUnit 4 样式测试类，包括使用 Espresso 和 UI Automator 测试框架的设备。测试运行器可以将测试软件包和要测试的应用加载到设备、运行测试并报告测试结果。
- Espresso
> Espresso 测试框架提供了一组 API 来构建 UI 测试，用于测试应用中的用户流。利用这些 API，您可以编写简洁、运行可靠的自动化 UI 测试。Espresso 非常适合编写白盒自动化测试，其中测试代码将利用所测试应用的实现代码详情。
- UI Automator
> UI Automator 测试框架提供了一组 API 来构建 UI 测试，用于在用户应用和系统应用中执行交互。利用 UI Automator API，您可以执行在测试设备中打开“设置”菜单或应用启动器等操作。UI Automator 测试框架非常适合编写黑盒自动化测试，其中的测试代码不依赖于目标应用的内部实现详情。
- Robolectric
> 主要是解决仪器化测试中耗时的缺陷，仪器化测试需要安装以及跑在Android系统上，也就是需要在Android虚拟机或真机上面，所以十分的耗时，基本上每次来来回回都需要几分钟时间。

```
dependencies {
  // Required -- JUnit 4 framework
  testImplementation 'junit:junit:4.12'
  // Optional -- Mockito framework（可选，用于模拟一些依赖对象，以达到隔离依赖的效果）
  testImplementation 'org.mockito:mockito-core:2.19.0'

  androidTestCompile 'com.android.support.test:runner:0.4'
  // Set this dependency to use JUnit 4 rules
  androidTestCompile 'com.android.support.test:rules:0.4'
  // Set this dependency to build and run Espresso tests
  androidTestCompile 'com.android.support.test.espresso:espresso-core:2.2.1'
  // Set this dependency to build and run UI Automator tests
  androidTestCompile 'com.android.support.test.uiautomator:uiautomator-v18:2.1.2'
}
```

[各种测试框架比较](https://davidleee.com/2017/07/04/Android-unit-test-lib-compare/)

### 参考资料

[关于安卓单元测试，你需要知道的一切](http://chriszou.com/2016/06/07/android-unit-testing-everything-you-need-to-know.html)

[Android单元测试在蘑菇街支付金融部门的实践](http://chriszou.com/2016/04/25/android-unit-testing-wechat-group-share.html)

[Android单元测试专题](https://blog.csdn.net/qq_17766199/article/details/78243176#commentBox)

[Android自动化测试--学习浅谈](https://www.jianshu.com/p/cb06c4be07fa) ++++

[Android 单元测试只看这一篇就够了](https://mp.weixin.qq.com/s?__biz=MzIwMTAzMTMxMg==&mid=2649492976&idx=1&sn=4bd263ca5e2f323648d8c3c670d68f93&chksm=8eec860fb99b0f195c98acabca6152306d95da626a03702a99d2c623976122bb7a3e8ab89a43&mpshare=1&scene=1&srcid=0725lVabnvtGaPRefBuRHVnB&pass_ticket=PrmhlMsETyCbtkVppItoPf2NsezymOWa7X5IDHGSqEIGdeb7FY9HHUkspypBW4RY#rd) +++++

[使用gradle或adb运行测试用例](https://developer.android.com/studio/test/command-line)
[gradle test-filter 使用说明](https://docs.gradle.org/current/userguide/java_testing.html#test_filtering)

### Github

- [Junit4](https://github.com/junit-team/junit4)

- [Mockito](https://github.com/mockito/mockito)

- [Robolectric](https://github.com/robolectric/robolectric)