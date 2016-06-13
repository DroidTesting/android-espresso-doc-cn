# AndroidJUnitRunner

> **声明：**本系列文章是对 [Android Testing Support Library](https://google.github.io/android-testing-support-library/docs/espresso/index.html)官方文档的翻译，水平有限，欢迎批评指正。


AndroidJUnitRunner 是一个为 Android 而生的全新非绑定测试执行器。它是 [Android 测试支持库](http://developer.android.com/tools/testing-support-library/index.html#features)的一部分，可以通过 Android Support Repository 下载。

以下是几个最常用到的特性：

* JUnit 4 支持
* Instrumentation 注册
* 测试过滤器
* 测试超时设定
* 切分测试
* [RunListener](http://junit.sourceforge.net/javadoc/org/junit/runner/notification/RunListener.html) 支持挂钩到测试运行的生命周期
* Activity 和 应用生命周期监控
* 意图监控和存根

此文只涵盖了 AndroidJUnitRunner 的高级特性。基础使用文档请参考 [developer.android.com](http://developer.android.com/) 上的 [AndroidJUnitRunner](http://developer.android.com/tools/testing-support-library/index.html#AndroidJUnitRunner) 章节。想要了解更过关于使用测试运行器的技巧请查看 [API reference](http://developer.android.com/reference/android/support/test/runner/package-summary.html)。