# Espresso Web

> **声明：**本系列文章是对 [Android Testing Support Library](https://google.github.io/android-testing-support-library/docs/espresso/index.html)官方文档的翻译，水平有限，欢迎批评指正。

> [下载 Espresso-Web](https://google.github.io/android-testing-support-library/docs/espresso/web/index.html#download-espresso-web)

Espresso-web 是测试 Android 上 WebView 的切入点。它使用流行的 WebDriver API 原子内省并控制 WebView 的行为。

与 onData 类似，WebView 的交互实际上是几个视图原子的组合。一个原子可以被看作一个 ViewAction，一个在 UI 上执行操作的自包含单元。然而，它们需要适当的精心策划并且十分啰嗦。Web 和 WebInteraction 对此样本进行了包装，提供了 Espresso 风格的 WebView 交互体验。

WebView 经常在 Java/JavaScript 之间跨界工作，由于没有机会将 JavaScript 中的数据引入到竞态机制(Espresso 得到的所有 Java 端的数据都一个独立的副本)，WebInteractions 全面支持数据的返回。

常规 WebInteractions
------------------

* `​withElement(ElementReference)`​ 将把 `​ElementReference`​ 提交到原子中，示例如下：

```java
onWebView().withElement(findElement(Locator.ID, "teacher"))
```

* `withContextualElement(Atom<ElementReference>)` 将把 ElementReference 提交到原子钟，示例如下：

```java
onWebView()
  .withElement(findElement(Locator.ID, "teacher"))
  .withContextualElement(findElement(Locator.ID, "person_name"))
```

* `​check(WebAssertion)`​ 将会检查断言的真假性。示例如下：

```java
onWebView()
  .withElement(findElement(Locator.ID, "teacher"))
  .withContextualElement(findElement(Locator.ID, "person_name"))
  .check(webMatches(getText(), containsString("Socrates")));
```

* `​perform(Atom)`​ 将在当前的上下文中执行提供的原子操作，示例如下：

```java
onWebView()
  .withElement(findElement(Locator.ID, "teacher"))
  .perform(webClick());
```

* 当之前的操作（如点击）改变了界面导航，从而使 ElementReference 和 WindowReference 点失效时，必须使用 `​reset()`​。

WebView 示例
----------

Espresso web 需要启用 JavaScript 来控制 WebView。你可以通过覆写 ActivityTestRule 类中的 afterActivityLaunched 方法强制启用它。

```java
@Rule
public ActivityTestRule<WebViewActivity> mActivityRule = new ActivityTestRule<WebViewActivity>(WebViewActivity.class, false, false) {
    @Override
    protected void afterActivityLaunched() {
        // Enable JS!
        onWebView().forceJavascriptEnabled();
    }
}

@Test
public void typeTextInInput_clickButton_SubmitsForm() {
   // Lazily launch the Activity with a custom start Intent per test
   mActivityRule.launchActivity(withWebFormIntent());

   // Selects the WebView in your layout. If you have multiple WebViews you can also use a
   // matcher to select a given WebView, onWebView(withId(R.id.web_view)).
   onWebView()
       // Find the input element by ID
       .withElement(findElement(Locator.ID, "text_input"))
       // Clear previous input
       .perform(clearElement())
       // Enter text into the input element
       .perform(DriverAtoms.webKeys(MACCHIATO))
       // Find the submit button
       .withElement(findElement(Locator.ID, "submitBtn"))
       // Simulate a click via javascript
       .perform(webClick())
       // Find the response element by ID
       .withElement(findElement(Locator.ID, "response"))
       // Verify that the response page contains the entered text
       .check(webMatches(getText(), containsString(MACCHIATO)));
}
```

在 GitHub 上查看 [Espresso Web sample](https://github.com/googlesamples/android-testing/tree/master/ui/espresso/WebBasicSample)。

下载 Espresso-Web
---------------

* 确保你已经安装了 *Android Support Repository*（查看[说明](https://google.github.io/android-testing-support-library/downloads/index.html)）
* 打开应用的 `​build.gradle`​ 文件。它通常不是顶级 `​build.gradle`​，而是​`​app/build.gradle`​。

在 dependencies 中添加以下行：

```java
androidTestCompile 'com.android.support.test.espresso:espresso-web:2.2.2'
```

Espresso-Web 只兼容 Espresso 2.2+ 和 testing supprot library 0.3+，所以你也要更新如下行：

```java
androidTestCompile 'com.android.support.test:runner:0.5'
androidTestCompile 'com.android.support.test:rules:0.5'
androidTestCompile 'com.android.support.test.espresso:espresso-core:2.2.2'
```
