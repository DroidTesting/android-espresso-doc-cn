# 可访问性检查

> **声明：**本系列文章是对 [Android Testing Support Library](https://google.github.io/android-testing-support-library/docs/espresso/index.html)官方文档的翻译，水平有限，欢迎批评指正。

类 [AccessibilityChecks](http://developer.android.com/reference/android/support/test/espresso/contrib/AccessibilityChecks.html) 允许你使用已有的测试代码来测试可访问性问题。作为测试测试中的一个视图，[可访问性测试框架](https://code.google.com/p/eyes-free/source/browse/trunk/devtools/accessibility-test-framework/src/main/java/com/google/android/apps/common/testing/accessibility/framework)会在它执行操作之前自动进行检查。你只需要导入该类，并将以下行添加到带有 `​@Before`​ 注解的 setup 方法中：

```java
import android.support.test.espresso.contrib.AccessibilityChecks;

@RunWith(AndroidJUnit4.class)
@LargeTest
public class AccessibilityChecksIntegrationTest {
    @BeforeClass
    public static void enableAccessibilityChecks() {
        AccessibilityChecks.enable();
    }
}
```

这将在每次调用 ViewActions 中的视图操作时触发对当前视图的可访问性检查。为了避免对视图结构中的所有视图进行检查，请使用：

```java
AccessibilityChecks.enable()
        .setRunChecksFromRootView(true);
```

当首次启用检查时，你可能会遇到一些你不想活不能立即处理的问题。你可以通过为你想要压制的结果设置一个匹配器来压制此类错误。可访问性测试框架中的 `AccessibilityCheckResultUtils`​ 中提供了 `​AccessibilityChechResults`​ 需要的匹配器。

例如，压制 id 为`​`​ `​R.id.example_view`​ 视图的所有错误：

```java
AccessibilityChecks.enable()
        .setSuppressingResultMatcher(matchingViews(withId(R.id.example_view)));
```

更多关于可访问性检查的搞基配置信息请参考可访问性测试框架中的 `​`​[AccessibilityValidator](https://code.google.com/p/eyes-free/source/browse/trunk/devtools/accessibility-test-framework/src/main/java/com/google/android/apps/common/testing/accessibility/framework/integrations/espresso/AccessibilityValidator.java)。
