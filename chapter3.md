# Espresso 基础

> **声明：**本系列文章是对 [Android Testing Support Library](https://google.github.io/android-testing-support-library/docs/espresso/index.html)官方文档的翻译，水平有限，欢迎批评指正。

Espresso API 鼓励测试者以用户会怎样与应用交互的方式进行思考来定位 UI 元素并与它们交互。同时，框架不允许直接使用应用的活动和视图，因为在非 UI 线程持有此类对象并对它们操作是造成测试花屏的主要原因。因此，你不会在 Espresso API 中看到诸如 getView 或 getCurrentActivity 等方法。但你仍然可以通过实现 `ViewAction` 和 `ViewAssertion` 来对视图进行安全操作。

以下是 Espresso 主要组件的概览：

* **Espresso** - 与视图交互的切入点（参考 `onView` 和 `onData`）。也暴露了与任何视图都没有必然联系的 API（如 `​pressBack`）。
* **ViewMatchers** - 实现了 `​Matcher<? super View>`​ 接口的对象集合。你可以在 `​onView`​ 方法中传入一个或多个此类对象来在当前的视图结构中定位一个视图。
* **ViewActions** - 可以作为参数传入 `​ViewInteraction.perform()`​ 方法中的 `ViewAction` 的集合（如 `​click()`）。
* **ViewAssertions** - 可以作为参数传入 `​ViewInteraction.check()`​ 方法中的 `ViewAssertion` 的集合。通常，你会使用带有视图匹配器的匹配断言来判断当前被选中视图的状态。

例如：

```java
onView(withId(R.id.my_view))      // withId(R.id.my_view) is a ViewMatcher
  .perform(click())               // click() is a ViewAction
  .check(matches(isDisplayed())); // matches(isDisplayed()) is a ViewAssertion
```

使用 onView 查找视图
--------------

多数情况下，onView 方法使用 hamcrest 匹配器以期望在当前视图结构里匹配一个（唯一的）视图。该匹配器十分强大而且对用过 Mockito 或 JUnit 的人而言并不陌生。如果你对 hamcrest 匹配器不熟悉，我们建议你先快速浏览一下[此报告](http://www.slideshare.net/shaiyallin/hamcrest-matchers)。（译注：译者本人表示打不开）

想要查找的视图一般会有唯一的 `​R.id`​ 值，使用简单的 `​withId`​ 匹配器可以缩小搜索范围。然而，当你在测试开发阶段，无法确定 `​R.id`值是合理的​。例如，指定的视图可能没有 `R.id`​ 值或该值不唯一。这将使一般的 instrumentation 测试变得脆弱而复杂，因为通用的获取视图方式（通过 f`indViewById()`）已经不适用了。因此，你可能需要获取持有视图的私有对象 Activity 或 Fragment，或者找到一个已知其 `​R.id`​ 值的父容器，然后在其中定位到特定的视图。

Espresso 处理该问题的方式很干脆，它允许你使用已存在的或自定义的 ViewMatcher 来限定视图查找。

通过 `​R.id`​ 查找视图：

```java
onView(withId(R.id.my_view))
```

有时，`​R.id`​值会被多个视图共享。此时，如果尝试使用该 `​R.id`​ 值将会抛出类似 `​AmbiguousViewMatcherException`​的异常。异常信息会给你提供文字描述形式的当前视图结构，你可以搜索并找出所有使用非唯一 `​R.id`​ 值的视图：

```java
java.lang.RuntimeException:
com.google.android.apps.common.testing.ui.espresso.AmbiguousViewMatcherException:
This matcher matches multiple views in the hierarchy: (withId: is <123456789>)
```

...

```java
+----->SomeView{id=123456789, res-name=plus_one_standard_ann_button, visibility=VISIBLE, width=523, height=48, has-focus=false, has-focusable=true, window-focus=true,
is-focused=false, is-focusable=false, enabled=true, selected=false, is-layout-requested=false, text=, root-is-layout-requested=false, x=0.0, y=625.0, child-count=1}
****MATCHES****
|
+------>OtherView{id=123456789, res-name=plus_one_standard_ann_button, visibility=VISIBLE, width=523, height=48, has-focus=false, has-focusable=true, window-focus=true,
is-focused=false, is-focusable=true, enabled=true, selected=false, is-layout-requested=false, text=Hello!, root-is-layout-requested=false, x=0.0, y=0.0, child-count=1}
****MATCHES****
```

通过查看视图丰富的属性，你兴许可以找到唯一可确认的属性（上例中，其中一个视图有一个“Hello!”文本）。你可以通过使用组合匹配器结合该属性来缩小搜索范围：

```java
onView(allOf(withId(R.id.my_view), withText("Hello!")))
```

你也可以使用 `​not`​ 反转匹配：

```java
onView(allOf(withId(R.id.my_view), not(withText("Unwanted"))))
```

你可以在 [ViewMatchers](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/core/src/main/java/android/support/test/espresso/matcher/ViewMatchers.java) 类中查看 Espresso 提供的视图匹配器。

*注意：*在一个良态的应用中，所有用户可与之交互的视图都应该包含说明文字或有一个内容描述（参考 [Android 可访问性指导](http://developer.android.com/guide/topics/ui/accessibility/apps.html)）。如果你不能通过使用 ‘withText’ 或 ‘withContentDescripiton’ 来缩小 onView 的搜索范围，可以认为这是一个可访问性的 bug。

*注意：*请使用最少的匹配器来定位视图。不要过指定，因为这将强制框架做无用功。例如，如果一个视图可以通过它的文字唯一确定，你不需要说明该视图也可以通过 `​TextView`​ 指定。对许多视图而言，使用它的 `​R.id`​ 值就足够了。

*注意：*如果目标视图在一个 `​AdapterView`​（如 `​ListView`​，`​GridView`​，`​Spinner`​）中，将不能使用 `onView`​ 方法，推荐使用 `​onData`​ 方法。

在视图上执行操作
--------

当为目标视图找到了合适的适配器后，你将可以通过 `​perform`​ 方法在该视图上执行 `​ViewAction`​。

例如，点击该视图：

```java
onView(…).perform(click());
```

你可以在一个 perform 方法中执行多个操作：

```java
onView(…).perform(typeText("Hello"), click());
```

如果操作的视图在 `​ScrollView`​（水平或垂直方向）中，需要考虑在对该视图执行操作（如 `​click()`​ 或 `​typeText()`​）之前通过 `​scrollTo()`​ 方法使其处于显示状态。这样就保证了视图在执行其他操作之前是显示着的。

```java
onView(…).perform(scrollTo(), click());
```

*注意：*如果视图已经是显示状态，* *`​scrollTo()`​ 将不会对界面有影响。因此，当视图的可见性取决于屏幕的大小时（例如，同时在大屏和小屏上执行测试时），你可以安全的使用该方法。

你可以在 [ViewActions](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/core/src/main/java/android/support/test/espresso/action/ViewActions.java) 类中产看 Espresso 提供的视图操作。

检查一个视图是否满足断言
------------

断言可以通过 `​check()`​ 方法应用在当前选中的视图上。最常用的是 `​matches()`​ 断言，它使用一个 `​ViewMatcher`​ 来判断当前选中视图的状态。

例如，检查一个视图拥有 “Hello!”文本：

```java
onView(…).check(matches(withText("Hello!")));
```

*注意：*不要将 “assertions” 作为 onView 的参数传入，而要在检查代码块中明确指定你检查的内容，例如：

如果你想要断言视图的内容是 “Hello!” ，以下做法是反面教材：

```java
// Don't use assertions like withText inside onView.
onView(allOf(withId(...), withText("Hello!"))).check(matches(isDisplayed()));
```

从另一个角度讲，如果你想要断言一个包含 “Hello!” 文本的视图是可见的（例如，在修改了该视图的可见性标志之后），这段代码是正确的。

**注意：**请留意断言一个视图没有显示和断言一个视图不在当前视图结构之间的区别。

使用 onView 编写一个简单的测试
-------------------

在此示例中，`​SimpleActivity`​ 包含一个 `​Button`​ 和一个 `​TextView`​。当点击按钮时，`​TextView`​ 的内容更改为 “Hello Espresso!”。以下是如何使用 Espresso 执行此测试的讲解：

### 1\. 点击按钮

第一步是检索一个能定位这个按钮的属性。`​SimpleActivity`​ 中的这个按钮拥有唯一的 `​R.id`​，赞！

```java
onView(withId(R.id.button_simple))
```

然后执行点击操作：

```java
onView(withId(R.id.button_simple)).perform(click());
```

### 2\. 检查 `​TextView`​ 中是否包含 “Hello Espresso!”

待验证的 `​TextView`​ 也包含唯一的 `​R.id`​：

```java
onView(withId(R.id.text_simple))
```

然后验证文本内容：

```java
onView(withId(R.id.text_simple)).check(matches(withText("Hello Espresso!")));
```

在 `​AdapterView`​ 控制器（`ListView`, `GridView`, ...）中使用 onData
------------------------------------------------------------

`​AdapterView`​ 是一个从适配器中动态加载数据的特殊控件。最常见的 `​AdapterView`​ 是 `ListView`​。与像 `​LinearLayout`​ 这样的静态控件相反，在当前视图结构中，可能只加载了 `​AdapterView`​ 子控件的一部分， 简单的 `​onview()`​ 搜索不能找到当前没有被加载的视图。Espresso 通过提供单独的 `onData()`​ 切入点处理此问题，它可以在操作适配器中有该问题的条目或该条目的子项之前将其加载（使其获取焦点）。

**注意：**你可能不会对初始状态就显示在屏幕上的适配器条目执行 `​onData()`​ 加载操作，因为它们已经被加载了。然而，一直使用 `​onData()`​ 会更安全。

**警告：**对于 `AdapterView`​ 的自定义实现，如果他们打破了继承契约（尤其是 `​getItem()`​ API），使用 `​onData()`​ 方法时会出现问题。此种情况，最好是重构你的应用代码。如果不能这样做，你可以实现一个匹配的自定义 `​AdapterViewProtocol`​。查看 Espresso 提供的默认的 [AdapterViewProtocols](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/core/src/main/java/android/support/test/espresso/action/AdapterViewProtocol.java) 获取供多信息。

使用 onData 编写一个简单的测试
-------------------

这个简单的测试演示了如何使用`​ onData()`​。

`​SimpleActivity`​ 包含一个 `​Spinner`​ ，该 `Spinner`​ 中有几个条目——代表咖啡类型的字符串。当选中其中一个条目时，`​TextView`​ 内容会变成 `​“One %s a day!”`​，其中 %s 代表选中的条目。此测试的目标是打开 `​Spinner`​，选中一个条目然后验证 `​TextView`​ 中包含该条目。由于 `​Spinner`​ 类基于 `​AdapterView`​，建议使用 `​onData()`​ 而不是 `​onView()`​ 来匹配条目。

### 1\. 点击 Spinner 打开条目选择框

```java
onView(withId(R.id.spinner_simple)).perform(click());
```

### 2\. 点击 “Americano” 条目

为了条目可供选择，Spinner 用它的内容创建了一个 `​ListView`​。该 `ListView` 可能会很长，而且它的元素不会出现在视图结构中。通过使用 `​onData()`​ 我们强制将想要得到的元素加入到视图结构中。Spinner 中的元素是字符串，我们想要匹配的条目是字符串类型并且值是 “Americano”。

```java
onData(allOf(is(instanceOf(String.class)), is("Americano"))).perform(click());
```

### 3\. 验证 `TextView`​ 包含 “Americano” 字符串

```java
onView(withId(R.id.spinnertext_simple).check(matches(withText(containsString("Americano"))));
```

调试
--

当测试失败时，Espresso 会提供有用的调试信息：

### 日志

Espresso 将所有视图操作记录到 logcat 中。例如：

```java
ViewInteraction: Performing ‘single click’ action on view with text: Espresso
```

### 视图结构

当 `onView()`​ 执行失败时，Espresso 会在异常字符串里打印视图结构。

* 如果 `​onView`​ 没有找到目标视图，会抛出 `​NoMatchingViewException`​。你可以检查异常字符串中的视图结构来分析为什么匹配器没有匹配到视图。
* 如果 `​onView()`​ 根据给出的匹配器找到了多个视图，会抛出 `​AmbiguousViewMatcherException`​。视图结构会被打印出来，并且所有被匹配的视图都会带有 MATCHES 标签：

```java
java.lang.RuntimeException:
com.google.android.apps.common.testing.ui.espresso.AmbiguousViewMatcherException:
This matcher matches multiple views in the hierarchy: (withId: is <123456789>)
```

...

```java
+----->SomeView{id=123456789, res-name=plus_one_standard_ann_button, visibility=VISIBLE, width=523, height=48, has-focus=false, has-focusable=true, window-focus=true,
is-focused=false, is-focusable=false, enabled=true, selected=false, is-layout-requested=false, text=, root-is-layout-requested=false, x=0.0, y=625.0, child-count=1}
****MATCHES****
|
+------>OtherView{id=123456789, res-name=plus_one_standard_ann_button, visibility=VISIBLE, width=523, height=48, has-focus=false, has-focusable=true, window-focus=true,
is-focused=false, is-focusable=true, enabled=true, selected=false, is-layout-requested=false, text=Hello!, root-is-layout-requested=false, x=0.0, y=0.0, child-count=1}
****MATCHES****
```

当处理一个完整的视图结构或控件异常行为时，使用 [Android 视图结构查看器](http://developer.android.com/tools/help/hierarchy-viewer.html)有利于你给出说明。

### `​AdapterView`​ 提醒

Espresso 会提醒用户 `AdapterView` 控件的出现。当 `​onView`​ 操作抛出 `​NoMatchingViewException`​ 异常而且 `​AdapterView`​ 控件在视图结构中时，最常见的解决方法是使用 `onData()`。异常信息中将会包含一个带有一列适配器视图的提醒。你可以通过此信息来调用 onData 加载目标视图。
