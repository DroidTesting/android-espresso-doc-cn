# Espresso 概览
> **声明：**本系列文章是对 [Android Testing Support Library](https://google.github.io/android-testing-support-library/docs/espresso/index.html)官方文档的翻译，水平有限，欢迎批评指正。

使用 Espresso，书写简洁、优雅、可信赖的 Android UI 测试：

```java
@Test
public void greeterSaysHello() {
  onView(withId(R.id.name_field)).perform(typeText("Steve"));
  onView(withId(R.id.greet_button)).perform(click());
  onView(withText("Hello Steve!")).check(matches(isDisplayed()));
}
```

核心 API 小巧、可预测、易于学习并且依然保持对定制的开放。Espresso 测试清晰的描述异常、交互和断言，而没有样板内容、自定义基础设施或凌乱的实现细节的干扰。

Espresso 测试运行非常快！它会在应用 UI 处于静止时对其进行操作和断言，而使你远离了等待、同步、睡眠以及民调（polls behind，不知如何翻译...）。

使用群体
----

Espresso 的使用群体为坚信自动化测试是开发周期中必不可少的一部分的开发者。虽然它可被用来做黑盒测试，但 Espresso 会在对被测代码库熟悉的人手中火力全开。

向后兼容
----

Espresso 支持如下 API：

|代号|API|
|----|:---:|
|Froyo|8|
|Gingerbread|10|
|Ice Cream Sandwich|15|
|Jelly Bean|16, 17 ,18|
|KitKat|19|
|Lollipop|21|

**注意：**

* 我们通过[平台版本仪表盘](http://developer.android.com/about/dashboards/index.html#Platform)来决定支持哪些 API 版本。随着使用旧 API 版本用户数量的跌落，我们会放弃支持这些 API 版本（Froyo 即将被放弃）。
* 我们会支持Android 未来的版本
