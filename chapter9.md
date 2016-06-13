# ATSL 中的 JUnit4 规则

> **声明：**本系列文章是对 [Android Testing Support Library](https://google.github.io/android-testing-support-library/docs/espresso/index.html)官方文档的翻译，水平有限，欢迎批评指正。

我们在 Android 测试支持库中提供了一套供 `AndroidJUnitRunner` 使用的 JUnit 规则。JUnit 规则提供了更多的灵活性，并且减少了测试需要的样板代码。

​为了拥抱 `ActivityTestRule` 和 `ServiceTestRule`​，我们不再使用 `​​ActivityInstruentationTestCase2`​ 和 `ServiceTestCase` 类型的 `TestCase` 。

ActivityTestRule
----------------

此规则提供了针对单个 activity 的功能测试。待测 activity 将在每个带 `@Test` 注解的测试和每个带 `​@Before`​ 注解的方法执行前被启动。它将在测试结束并且所有带 `​@After`​ 注解的方法执行完后被销毁。待测 activity 在测试期间可以通过调用 `​ActivityTestRule#getActivity`​ 访问。

```java
@RunWith(AndroidJUnit4.class)
@LargeTest
public class MyClassTest {

    @Rule
    public ActivityTestRule<MyClass> mActivityRule = new ActivityTestRule(MyClass.class);

    @Test
    public void myClassMethod_ReturnsTrue() { ... }
}
```

ServiceTestRule
---------------

此规则提供了在测试前后启动和终止服务的简单机制。它也会保证当启动（或绑定）一个服务时，该服务被成功的连接。服务可以使用它的一个帮助方法来启动（或绑定)。它将在测试结束并且所有带 `​@After`​ 注解的方法执行完后自动被停止（或解绑）。

注意：此规则不适用于 `​IntentService`​，因为它会在 `​IntentService#onHandleIntent(android.content.Intent)`​ 结束了所有外发指令时自动销毁。所以无法保证及时建立一个成功的连接。

```java
@RunWith(AndroidJUnit4.class)
@MediumTest
public class MyServiceTest {

    @Rule
    public final ServiceTestRule mServiceRule = new ServiceTestRule();

    @Test
    public void testWithStartedService() {
        mServiceRule.startService(
            new Intent(InstrumentationRegistry.getTargetContext(), MyService.class));
        // test code
    }

    @Test
    public void testWithBoundService() {
        IBinder binder = mServiceRule.bindService(
            new Intent(InstrumentationRegistry.getTargetContext(), MyService.class));
        MyService service = ((MyService.LocalBinder) binder).getService();
        assertTrue("True wasn't returned", service.doSomethingToReturnTrue());
    }
}
```