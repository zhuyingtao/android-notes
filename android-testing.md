## Android测试

Android 的测试种类:

- Unit Test（单元测试）
  - Junit Test —— 用来测试无关 Android 平台的代码。
  - Instrument Unit Test —— 运行在 Android 系统中, 这些测试可以获取到测试应用的上下文信息，用来测试有 Android API 的代码。
- Integration Test（集成测试）



Android Studio 中的测试

- Junit Test 的默认测试文件夹在 `src/test/java`。
- Instrument Unit Test 的默认测试文件夹在`src/androidTest/java`。

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

[关于安卓单元测试，你需要知道的一切](http://chriszou.com/2016/06/07/android-unit-testing-everything-you-need-to-know.html)

[Android单元测试在蘑菇街支付金融部门的实践](http://chriszou.com/2016/04/25/android-unit-testing-wechat-group-share.html)

#### 6. 基本测试框架

- Java单元测试框架：**Junit、Mockito、Powermockito**
- Android单元测试框架：**Robolectric、AndroidJUnitRunner、Espresso**