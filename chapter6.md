# Espresso 高级示例

> **声明：**本系列文章是对 [Android Testing Support Library](https://google.github.io/android-testing-support-library/docs/espresso/index.html)官方文档的翻译，水平有限，欢迎批评指正。

视图匹配器
-----

### 匹配紧靠一个视图的另一个视图

一个布局可以包含多个本身并不唯一的视图（如联系人列表中的重播按钮，它们在视图结构中拥有相同的 R.id 值，相同的文字和属性）。

比如，在此 activity 中，包含文字“7”的视图在多行中重复出现：

![has sibling](http://upload-images.jianshu.io/upload_images/146569-e43687144ebca2a7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

此类非唯一的视图通常会与紧靠它的带有唯一标签的视图配对（如拨号按钮旁的姓名）。这种情况下，你可以使用 hasSibling 匹配器缩小选择范围：

```java
onView(allOf(withText("7"), hasSibling(withText("item 0")))).perform(click());
```

### 用 onData 和自定义 ViewMatcher 匹配数据

如下 activity 包含一个 ListView，它基于一个为每一行提供一个 `​Map<String, Object>`​ 类型的 `SimpleAdapter`​。每个 map 都一个键为 “STR” ，的元素（值为 String 类型，“item: x”）和一个键为 “LEN” 的元素（值为 Integer 类型，字符串的长度）。

![has sibling2](http://upload-images.jianshu.io/upload_images/146569-004157189cd3689d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击 “item: 50” 条目的代码如下：

```java
onData(allOf(is(instanceOf(Map.class)), hasEntry(equalTo("STR"), is("item: 50))).perform(click());
```

我们先来看 `​onData`​ 的一部分：

```java
is(instanceOf(Map.class))
```

限制搜索 AdapterView 中任意条目的条件为一个 Map。

在此例子中，ListView 的所有行都满足条件。但我们想要点击指定的条目 “item: 50”，所以我们需要继续缩小范围：

```java
hasEntry(equalTo("STR"), is("item: 50))
```

这个 Matcher\<String, Object\> 会匹配所有包含任意键，值=“item: 50” 的 Map 。鉴于查找此条目的代码较长，而且我们希望在其他地方重用它，我们可以写一个自定义的 matcher “withItemContent”：

```java
return new BoundedMatcher<Object, Map>(Map.class) {
    @Override
    public boolean matchesSafely(Map map) {
      return hasEntry(equalTo("STR"), itemTextMatcher).matches(map);
    }

    @Override
    public void describeTo(Description description) {
      description.appendText("with item content: ");
      itemTextMatcher.describeTo(description);
    }
  };
}
```

我们使用 `​BoundedMatcher`​ 作为基类，因为我们希望能够只匹配 Map 类型的对象。我们覆写了 matchesSafely 方法，将之前创建的匹配器带入，将其通过 `​Matcher<String>`​ 传入进行匹配。此方式将允许我们使用 `​withItemContent(equalTo(“foo”))`​。为了代码简洁性，我们创建了另一个已经做了 equalTo 操作并接收一个 String 的匹配器：

```java
public static Matcher<Object> withItemContent(String expectedText) {
  checkNotNull(expectedText);
  return withItemContent(equalTo(expectedText));
}
```

现在点击该条目的代码很简单了：

```java
onData(withItemContent("item: 50")).perform(click());
```

此测试的完整代码请查看 [AdapterViewText\#testClickOnItem50](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/sample/src/androidTest/java/android/support/test/testapp/AdapterViewTest.java) 和[自定义匹配器](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/sample/src/androidTest/java/android/support/test/testapp/LongListMatchers.java)。

### 匹配一个视图的指定子视图

上述示例陈述了在 ListView 中点击一行的中间位置。但我们该如何操作一行中指定的子视图？例如，我们我们想要点击 LongListActivity 其中一行的第二列，该列展示第一列的 `​String.lenth`​（具象而言，你可以想象一下 G+ 应用中的评论，在每一条评论旁有一个 +1 按钮）：

![](http://upload-images.jianshu.io/upload_images/146569-fadc1f9aa839f371.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在 `​DataInteraction`​ 中添加一个 `​onChildView`​ 说明：

```java
onData(withItemContent("item: 60"))
  .onChildView(withId(R.id.item_size))
  .perform(click());
```

**注意：**此示例使用了上例中的 `​withItemContent`​ 匹配器！参考 [ApdaterViewTest\#testClickOnSpecificChildOfRow60](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/sample/src/androidTest/java/android/support/test/testapp/AdapterViewTest.java)!

### 匹配 ListView 的 footer/header 视图

header 和 footer 通过 addHeaderView/addFooterView API 添加到 ListView 中。为了能够使用 Espresso.onData 加载它们，确保使用预置的值来设置数据对象（第二个参数）。

```java
public static final String FOOTER = "FOOTER";
...
View footerView = layoutInflater.inflate(R.layout.list_item, listView, false);
((TextView) footerView.findViewById(R.id.item_content)).setText("count:");
((TextView) footerView.findViewById(R.id.item_size)).setText(String.valueOf(data.size()));
listView.addFooterView(footerView, FOOTER, true);
```

然后，你可以写一个匹配器来匹配此对象：

```java
import static org.hamcrest.Matchers.allOf;
import static org.hamcrest.Matchers.instanceOf;
import static org.hamcrest.Matchers.is;

@SuppressWarnings("unchecked")
public static Matcher<Object> isFooter() {
  return allOf(is(instanceOf(String.class)), is(LongListActivity.FOOTER));
}
```

在测试中很轻易就能加载该视图：

```java
import static com.google.android.apps.common.testing.ui.espresso.Espresso.onData;
import static com.google.android.apps.common.testing.ui.espresso.action.ViewActions.click;
import static com.google.android.apps.common.testing.ui.espresso.sample.LongListMatchers.isFooter;

public void testClickFooter() {
  onData(isFooter())
    .perform(click());
  ...
}
```

请在 [AdapterViewtest\#testClickFooter](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/sample/src/androidTest/java/android/support/test/testapp/AdapterViewTest.java) 中查看完整示例代码。

### 匹配 ActionBar 中的视图

`​ActionBarActivity`​ 有两个不同的模式：普通的 ActionBar 和从[选项菜单](http://developer.android.com/guide/topics/ui/menus.html#options-menu)中创建的上下文 ActionBar。它们都有一个条目是一直可见的，而另外两个只在悬浮菜单中课件。当点击一个条目时，它会将一个 TextView 的内容改为点击条目的内容。

匹配两种 ActionBar 中的可见图标都很简单：

```java
public void testClickActionBarItem() {
  // We make sure the contextual action bar is hidden.
  onView(withId(R.id.hide_contextual_action_bar))
    .perform(click());

  // Click on the icon - we can find it by the r.Id.
  onView(withId(R.id.action_save))
    .perform(click());

  // Verify that we have really clicked on the icon by checking the TextView content.
  onView(withId(R.id.text_action_bar_result))
    .check(matches(withText("Save")));
}
```

![](http://upload-images.jianshu.io/upload_images/146569-ec1960155e576e61.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

针对上下文 ActionBar 的代码与之类似：

```java
public void testClickActionModeItem() {
  // Make sure we show the contextual action bar.
  onView(withId(R.id.show_contextual_action_bar))
    .perform(click());

  // Click on the icon.
  onView((withId(R.id.action_lock)))
    .perform(click());

  // Verify that we have really clicked on the icon by checking the TextView content.
  onView(withId(R.id.text_action_bar_result))
    .check(matches(withText("Lock")));
}
```

![](http://upload-images.jianshu.io/upload_images/146569-bc80589faf4d0174.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击普通 ActionBar 悬浮菜单中的条目有点棘手，由于某些设备带有硬件悬浮菜单按钮（它们会打开选项菜单的悬浮条目），而某些设备带有软件悬浮菜单按钮（它们会打开一个普通悬浮菜单）。幸运的是，Espresso 为我们解决了此问题。

对于普通的 ActionBar：

```java
public void testActionBarOverflow() {
  // Make sure we hide the contextual action bar.
  onView(withId(R.id.hide_contextual_action_bar))
    .perform(click());

  // Open the overflow menu OR open the options menu,
  // depending on if the device has a hardware or software overflow menu button.
  openActionBarOverflowOrOptionsMenu(getInstrumentation().getTargetContext());

  // Click the item.
  onView(withText("World"))
    .perform(click());

  // Verify that we have really clicked on the icon by checking the TextView content.
  onView(withId(R.id.text_action_bar_result))
    .check(matches(withText("World")));
}
```

![](http://upload-images.jianshu.io/upload_images/146569-da9e809348eaa406.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如下是在带硬件悬浮菜单按钮设备的显示：

![](http://upload-images.jianshu.io/upload_images/146569-5414ea15e479e094.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对于上下文 ActionBar 而言也很简单：

```java
public void testActionModeOverflow() {
  // Show the contextual action bar.
  onView(withId(R.id.show_contextual_action_bar))
    .perform(click());

  // Open the overflow menu from contextual action mode.
  openContextualActionModeOverflowMenu();

  // Click on the item.
  onView(withText("Key"))
    .perform(click());

  // Verify that we have really clicked on the icon by checking the TextView content.
  onView(withId(R.id.text_action_bar_result))
    .check(matches(withText("Key")));
  }
```

![](http://upload-images.jianshu.io/upload_images/146569-15bfa76ac58a4ef8.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

查看完整示例代码： [ActionBarTest.java](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/sample/src/androidTest/java/android/support/test/testapp/ActionBarTest.java)。

ViewAssertion
-------------

### 断言视图没有显示

执行过一系列操作之后，你会想要断言待测 UI 的状态。有时可是会是一个负面情况（例如，有些操作没有成功）。请记住，你可以通过 ViewAssertions.matches 将任意的 hamcrest 视图匹配器转换为一个 ViewAssertion。

在下面的例子中，我们使用了 isDisplayed 匹配器，并使用标准的 “not” 匹配器反转匹配结果：

```java
import static com.google.android.apps.common.testing.ui.espresso.Espresso.onView;
import static com.google.android.apps.common.testing.ui.espresso.assertion.ViewAssertions.matches;
import static com.google.android.apps.common.testing.ui.espresso.matcher.ViewMatchers.isDisplayed;
import static com.google.android.apps.common.testing.ui.espresso.matcher.ViewMatchers.withId;
import static org.hamcrest.Matchers.not;

onView(withId(R.id.bottom_left))
  .check(matches(not(isDisplayed())));
```

如果视图仍然是整个结构的一部分，上述方式可用。否则，你将得到一个 `​NoMatchingViewException`​，此时你需要使用 `​ViewAssertions.doesNotExit`​（见下面）。

### 断言一个视图不存在

如果视图不在视图结构中（如，界面切换到了另一个 activity），你应该使用 `ViewAssertions.doesNotExist`:

```java
import static com.google.android.apps.common.testing.ui.espresso.Espresso.onView;
import static com.google.android.apps.common.testing.ui.espresso.assertion.ViewAssertions.doesNotExist;
import static com.google.android.apps.common.testing.ui.espresso.matcher.ViewMatchers.withId;

onView(withId(R.id.bottom_left))
  .check(doesNotExist());
```

### 断言一个数据条目不在指定适配器中

为了证明一个指定的数据条目不在指定的 AdapterView 中，你需要做些其他操作。我们需要找到指定的 AdapterView 并得到它持有的数据。此处不需要使用 onData()，而是使用 onView 查找 AdapterView，然后使用另一个匹配器处理该视图中的数据。

首先创建匹配器：

```java
private static Matcher<View> withAdaptedData(final Matcher<Object> dataMatcher) {
  return new TypeSafeMatcher<View>() {

    @Override
    public void describeTo(Description description) {
      description.appendText("with class name: ");
      dataMatcher.describeTo(description);
    }

    @Override
    public boolean matchesSafely(View view) {
      if (!(view instanceof AdapterView)) {
        return false;
      }
      @SuppressWarnings("rawtypes")
      Adapter adapter = ((AdapterView) view).getAdapter();
      for (int i = 0; i < adapter.getCount(); i++) {
        if (dataMatcher.matches(adapter.getItem(i))) {
          return true;
        }
      }
      return false;
    }
  };
}
```

接下来我们只需要使用一个 onView 查找指定的 AdapterView：

```java
@SuppressWarnings("unchecked")
public void testDataItemNotInAdapter(){
  onView(withId(R.id.list))
      .check(matches(not(withAdaptedData(withItemContent("item: 168")))));
  }
```

如果 id 为 list 的 AdapterView 中有一个条目与 “item: 168” 相同，则断言失败。

参考网址示例代码：[AdapterViewTest\#testDataItemNotInAdapter](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/sample/src/androidTest/java/android/support/test/testapp/AdapterViewTest.java)。

闲置资源
----

### 使用 registerIdlingResource 与自定义资源同步

Espresso 的核心是它可以与待测应用无缝同步测试操作的能力。默认情况下，Espresso 会等待当前消息队列中的 UI 事件执行（默认是 AsyncTask）完毕再进行下一个测试操作。这应该能解决大部分应用与测试同步的问题。

然而，应用中有一些执行后台操作的对象（比如与网络服务交互）通过非标准方式实现；例如：直接创建和管理线程，以及使用自定义服务。

此种情况，我们建议你首先提出可测试性的概念，然后询问使用非标准后台操作是否必要。某些情况下，可能是由于对 Android 理解太少造成的，并且应用也会受益于重构（例如，将自定义创建的线程改为 AsyncTask）。然而，某些时候重构并不现实。庆幸的是 Espresso 仍然可以同步测试操作与你的自定义资源。

以下是我们需要完成的：

* 实现 `​IdlingResource`​ 接口并暴露给测试。
* 通过在 setUp 中调用 `​Espresso.registerIdlingResource`​ 注册一个或多个 IdlingResource 给 Espresso。

参考 [​AdvancedSynchronizationTest](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/sample/src/androidTest/java/android/support/test/testapp/AdvancedSynchronizationTest.java)​ 和 [CountingIdlingResource](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/contrib/src/main/java/android/support/test/espresso/contrib/CountingIdlingResource.java) 类以了解 IdlingResource 如何能使用。

需要注意的是 IdlingResource 接口是在待测应用中实现的，所以你需要谨慎的添加依赖：

```java
// IdlingResource is used in the app under test
compile 'com.android.support.test.espresso:espresso-idling-resource:2.2.2'

// For CountingIdlingResource:
compile 'com.android.support.test.espresso:espresso-contrib:2.2.2'
```

定制
--

### 使用自定义失败处理器

使用一个自定义的失败处理器来替换 Espresso 默认的 FailureHandler 以允许增强（或区分）错误的处理。比如：截图或转储特别的调试信息。

示例 [CustomFailureHandlerTest](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/sample/src/androidTest/java/android/support/test/testapp/CustomFailureHandlerTest.java?autodive=0%2F%2F%2F%2F) 演示了如何实现一个自定义的失败处理器：

```java
private static class CustomFailureHandler implements FailureHandler {
  private final FailureHandler delegate;

  public CustomFailureHandler(Context targetContext) {
    delegate = new DefaultFailureHandler(targetContext);
  }

  @Override
  public void handle(Throwable error, Matcher<View> viewMatcher) {
    try {
      delegate.handle(error, viewMatcher);
    } catch (NoMatchingViewException e) {
      throw new MySpecialException(e);
    }
  }
}
```

此失败处理器用 MySpecialException 代替了 NoMatchingViewException，并委托其他的失败给 DefaultFailureHandler。CustomFailureHandler 可以在 Espresso 测试的 setUp() 中注册：

```java
@Override
public void setUp() throws Exception {
  super.setUp();
  getActivity();
  setFailureHandler(new CustomFailureHandler(getInstrumentation().getTargetContext()));
}
```

参考 [FailureHandler](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/core/src/main/java/android/support/test/espresso/FailureHandler.java) 接口和 [Espresso.setFailureHandler](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/core/src/main/java/android/support/test/espresso/Espresso.java) 获取更多信息。

inRoot
------

### 使用 inRoot 来指定非默认窗口

很惊奇吧，但这是真的—— Android 支持多[窗口](http://developer.android.com/reference/android/view/Window.html)。通常这对于用户和 Android 开发者来说是透明的（transparent 双关意）,在某些情况下多窗口是可见的（例如：在搜索控件中自动补全窗口绘制在主窗口之上）。为了方便，Espresso 默认使用一个启发模式来推测你想要与哪个窗口交互。你可以通过自己提供根窗口（亦称为 [Root](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/core/src/main/java/android/support/test/espresso/Root.java) 匹配器）来决定要与哪个窗口交互。

```java
onView(withText("South China Sea"))
  .inRoot(withDecorView(not(is(getActivity().getWindow().getDecorView()))))
  .perform(click());
```

和 [ViewMatchers](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/core/src/main/java/android/support/test/espresso/matcher/ViewMatchers.java) 的情况类似，我们提供了一组封装好的 [RootMatchers](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/core/src/main/java/android/support/test/espresso/matcher/RootMatchers.java)。当然，你依然可以实现自己的匹配器。

更多信息请查看 [示例](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/sample/src/androidTest/java/android/support/test/testapp/MultipleWindowTest.java) 或者 [GitHub 上的示例](https://github.com/googlesamples/android-testing/tree/master/ui/espresso/MultiWindowSample)。
