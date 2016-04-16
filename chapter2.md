# Espresso 设置说明

> **声明：**本系列文章是对 [Android Testing Support Library](https://google.github.io/android-testing-support-library/docs/espresso/index.html)官方文档的翻译，水平有限，欢迎批评指正。

本指南涵盖了使用 SDK Manager 安装 Espresso 和使用 Gradle 构建 Espresso 测试两部分内容。推荐使用 Android Studio。

配置测试环境
------

为了避免花屏，我们强烈建议在虚拟机或真实设备上测试时关闭系统动画。

* 在设备上的设置-\>开发者选项中禁用一下三项设置：
> 窗口动画缩放
> 过渡动画缩放
> 动画程序时长缩放

下载 Espresso
-----------

* 确保你已经安装了最新的 *Extras 下的* *Android Support Repository* （查看[使用说明](https://google.github.io/android-testing-support-library/downloads/index.html)）
* 打开应用的 `build.gradle` 文件。这通常不是顶级 `build.gradle`，而是 `app/build.gradle`。
* 在 dependencies 节点下添加以下行：

```java
androidTestCompile 'com.android.support.test.espresso:espresso-core:2.2.2'
androidTestCompile 'com.android.support.test:runner:0.5'
```

* 参考[下载](https://google.github.io/android-testing-support-library/downloads/index.html)小节查看更多的工件（espresso-contrib, espresso-web 等）

设置 instrumentation runner
-------------------------

* 在 `​android.defaultConfig`​ 下添加下面的代码：

```java
testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
```

build.gradle 示例文件
-----------------

```groovy
apply plugin: 'com.android.application'

android {
    compileSdkVersion 22
    buildToolsVersion "22"

    defaultConfig {
        applicationId "com.my.awesome.app"
        minSdkVersion 10
        targetSdkVersion 22.0.1
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
}

dependencies {
    // App's dependencies, including test
    compile 'com.android.support:support-annotations:22.2.0'

    // Testing-only dependencies
    androidTestCompile 'com.android.support.test:runner:0.5'
    androidTestCompile 'com.android.support.test.espresso:espresso-core:2.2.2'
}
```

分析数据
----

为了确保每一个新发布的版本正常工作，test runner 会收集分析数据。具体而言，每次调用它都会上传待测应用包名的一个 hash 值。这可以使我们在 Espresso 特性包的数量以及它的体积之间做出权衡。

如果你不希望上传此类数据，可以通过给 test runner 传入 `​disableAnalytics “true”`​ 来禁止（参考 [如何传入自定义参数](https://github.com/googlesamples/android-testing-templates/tree/master/AndroidTestingBlueprint#custom-gradle-command-line-arguments)）

添加第一个测试
-------

Android Studio 默认在 `​src/androidTest/java/com.example.package/`​ 中创建测试。

使用 Rules 创建的 JUnit4 测试实例：

```java
@RunWith(AndroidJUnit4.class)
@LargeTest
public class HelloWorldEspressoTest {

    @Rule
    public ActivityTestRule<MainActivity> mActivityRule = new ActivityTestRule(MainActivity.class);

    @Test
    public void listGoesOverTheFold() {
        onView(withText("Hello world!")).check(matches(isDisplayed()));
    }
}
```

执行测试
----

### 使用 Android Studio 执行

创建测试配置

在 Android Studio中：

* 打开菜单 Run -\> Edit Configurations
* 添加一个新的 Android Tests 配置
* 选择 module
* 添加一个指定的 instrumentation runner：

```groovy
android.support.test.runner.AndroidJUnitRunner
```

执行这个新创建的配置

### 在命令行中通过 Gradle 执行

执行 `​./gradlew connectedAndroidTest`
