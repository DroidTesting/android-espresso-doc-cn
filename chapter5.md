# Espresso 意图

> **声明：**本系列文章是对 [Android Testing Support Library](https://google.github.io/android-testing-support-library/docs/espresso/index.html)官方文档的翻译，水平有限，欢迎批评指正。

Espresso-Intents 是 Espresso 的一个扩展，它使验证和存根待测应用向外发出的意图成为可能。它类似于 [Mockito](http://mockito.org/)，但是针对的是 Android 的意图。

下载 Espresso-Intents
-------------------

* 确保你已经安装了 *Android Support Repository*(参考 [instructions](https://google.github.io/android-testing-support-library/downloads/index.html))。
* 打开你应用的 `​build.gradle`​ 文件。这一般不是顶级的 `​build.gralde`​ 文件，而是 `​app/build.gradle`​。

在 dependencies 中添加以下行：

```java
androidTestCompile 'com.android.support.test.espresso:espresso-intents:2.2.2'
```

Espresso-Intents 只兼容 Espresso 2.1+ 和 testing support library 0.3。所以确保你也更新了以下行：

```java
androidTestCompile 'com.android.support.test:runner:0.5'
androidTestCompile 'com.android.support.test:rules:0.5'
androidTestCompile 'com.android.support.test.espresso:espresso-core:2.2.2'
```

IntentsTestRule
---------------

 当使用 Espresso-Intents 时，应当用 `​IntentsTestRule`​ 替换 `​ActivityTestRule`​。`IntentsTestRule`​ 使得在 UI 功能测试中使用 Espresso-Intents API 变得简单。该类是 `​ActivityTestRule`​ 的扩展，它会在每一个被 `​@Test`​ 注解的测试执行前初始化 Espresso-Intents，然后在测试执行完后释放 Espresso-Intents。被启动的 activity 会在每个测试执行完后被终止掉，此规则也适用于 `​ActivityTestRule`​。

验证意图（Intent validation）
-----------------------

Espresso-Intents 会记录待测应用里所有尝试启动 Activity 意图。使用 intended API（与 `​Mockito.verify`​ 类似）你可以断言特定的意图是否被发出。

验证外发意图的简单示例：

```java
@Test
public void validateIntentSentToPackage() {
    // User action that results in an external "phone" activity being launched.
    user.clickOnView(system.getView(R.id.callButton));

    // Using a canned RecordedIntentMatcher to validate that an intent resolving
    // to the "phone" activity has been sent.
    intended(toPackage("com.android.phone"));
}
```

意图存根（Intent stubbing）
---------------------

使用 intending API（与 `​Mockito.when`​ 类似）你可以为通过 startActivityForResult 启动的 Activity 提供一个响应结果（尤其是外部的 Activity，因为我们不能操作外部 activity 的用户界面，也不能控制 `​ActivityResult`​ 返回给待测 Activity）。

使用意图存根的示例：

```java
@Test
public void activityResult_IsHandledProperly() {
    // Build a result to return when a particular activity is launched.
    Intent resultData = new Intent();
    String phoneNumber = "123-345-6789";
    resultData.putExtra("phone", phoneNumber);
    ActivityResult result = new ActivityResult(Activity.RESULT_OK, resultData);

    // Set up result stubbing when an intent sent to "contacts" is seen.
    intending(toPackage("com.android.contacts")).respondWith(result));

    // User action that results in "contacts" activity being launched.
    // Launching activity expects phoneNumber to be returned and displays it on the screen.
    onView(withId(R.id.pickButton)).perform(click());

    // Assert that data we set up above is shown.
    onView(withId(R.id.phoneNumber).check(matches(withText(phoneNumber)));
}
```

### 意图匹配器（Intent Matchers）

`​intending`​ 和 `​intended`​ 方法用一个 hamcrest `​Matcher<Intent>`​ 作为参数。 Hamcrest 是匹配器对象（也称为约束或断言）库。有以下选项：

* 使用现有的意图匹配器：最简单的选择，绝大多数情况的首选。
* 自己实现意图匹配器，最灵活的选择（参考 [Hamcrest 教程](http://code.google.com/p/hamcrest/wiki/Tutorial) 的 “Writing custom matchers” 章节）

以下是一个使用现有的意图匹配器验证意图的示例：

```java
intended(allOf(
    hasAction(equalTo(Intent.ACTION_VIEW)),
    hasCategories(hasItem(equalTo(Intent.CATEGORY_BROWSABLE))),
    hasData(hasHost(equalTo("www.google.com"))),
    hasExtras(allOf(
        hasEntry(equalTo("key1"), equalTo("value1")),
        hasEntry(equalTo("key2"), equalTo("value2")))),
        toPackage("com.android.browser")));
```
