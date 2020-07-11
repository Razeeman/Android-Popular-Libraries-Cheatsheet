# Testing

- [JUnit](#junit---)
- [Mockito](#mockito---)
- [PowerMock](#powermock---)
- [Espresso](#espresso--)
- [Sources](#sources)



# JUnit [![Maven][junit-mavenbadge]][junit-maven] [![Source][junit-sourcebadge]][junit-source] ![junit-starsbadge]

Popular unit testing framework.

```gradle
dependencies {
    testImplementation 'junit:junit:4.12'
}
```

### Unit tests

Located at module-name/src/test/java/ .

Unit tests run on local machine only. These tests are compiled to run locally on the Java Virtual Machine (JVM) to minimize execution time.

Running unit tests against your code, you can easily verify that the logic of individual units is correct. Running unit tests after every build helps you to quickly catch and fix software regressions introduced by code changes to your app. A unit test generally exercises the functionality of the smallest possible unit of code (which could be a method, class, or component) in a repeatable way.

JUnit assumes that all test methods can be executed in an arbitrary order. Well-written test code should not assume any order, i.e., tests should not depend on other tests.

By default, the Android Plug-in for Gradle executes your local unit tests against a modified version of the android.jar library, which does not contain any actual code. Instead, method calls to Android classes from your unit test throw an exception. This is to make sure you test only your code and do not depend on any particular behavior of the Android platform (that you have not explicitly built or mocked).

### JUnit. Basic test

```java
public class CalculatorTest {
    @Test
    public void calculatorTest_multiplicationOfZeroShouldReturnZero() {
        MyCalculator calc = new MyCalculator();
        assertEquals(0, calc.multiply(10, 0), "10 x 0 must be 0");
    }
}
```

### JUnit. Annotations

- **@Test** Identifies a method as a test method.

- **@BeforeClass** Executed once, before the start of all tests. It is used to perform time intensive activities, for example, to connect to a database. Methods need to be static.

- **@Before** Executed before each test. It is used to prepare the test environment (e.g., read input data, initialize the class).

- **@After** Executed after each test. It is used to cleanup the test environment (e.g., delete temporary data, restore defaults). It can also save memory by cleaning up expensive memory structures.

- **@AfterClass** Executed once, after all tests have been finished. It is used to perform clean-up activities, for example, to disconnect from a database. Methods need to be static.

- **@Ignore or @Ignore("Why disabled")** Marks that the test should be disabled. This is useful when the underlying code has been changed and the test case has not yet been adapted. Or if the execution time of this test is too long to be included.

- **@Test (expected = Exception.class)** Fails if the method does not throw the named exception.

- **@Test(timeout = 100)** Fails if the method takes longer than 100 milliseconds.

### JUnit. Assert statements

- fail([message])
- assertTrue([message,] boolean condition)
- assertFalse([message,] boolean condition)
- assertEquals([message,] expected, actual)
- assertEquals([message,] expected, actual, tolerance)
- assertNull([message,] object)
- assertNotNull([message,] object)
- assertSame([message,] expected, actual)
- assertNotSame([message,] expected, actual)

[Content](#testing)
# Mockito [![Maven][mockito-mavenbadge]][mockito-maven] [![Source][mockito-sourcebadge]][mockito-source] ![mockito-starsbadge]

Most popular Mocking framework for unit tests written in Java. Cannot mock static methods and private methods.

```gradle
dependencies {
    testImplementation 'junit:junit:4.12'
    testImplementation 'org.mockito:mockito-core:2.24.5'
}
```

If you have minimal Android dependencies and need to test specific interactions between a component and its dependency within your app, use a mocking framework to stub out external dependencies in your code. That way, you can easily test that your component interacts with the dependency in an expected way. By substituting Android dependencies with mock objects, you can isolate your unit test from the rest of the Android system while verifying that the correct methods in those dependencies are called.

A mock object is a dummy implementation for an interface or a class in which you define the output of certain method calls. Mock objects are configured to perform a certain behavior during a test. They typically record the interaction with the system and tests can validate that.

### Mockito. Mock dependencies

- To create a mock object for an Android dependency, add the @Mock annotation before the field declaration or use mock() method.
- Initialize mocks with MockitoAnnotations.initMocks(this).
- To stub the behavior of the dependency, you can specify a condition and return value when the condition is met by using the when().thenReturn() or doReturn().when(). The latter doesn't call the real method and can be used to avoid excepcions.

```java
public class PreferencesHelperTest {
    @Mock private SharedPreferences mSharedPreferences;
    private PreferencesHelper mPreferencesHelper;

    @Before
    public void setUp() throws Exception {
        MockitoAnnotations.initMocks(this);
        mPreferencesHelper = new PreferencesHelper(mSharedPreferences);
    }

    @Test
    public void getPreference() {
        when(mSharedPreferences.getString(eq(SharedPreferencesHelper.KEY_NAME), anyString()))
			.thenReturn(TEST_STRING);

        assertEquals(mPreferencesHelper.getSetting(), TEST_STRING);
    }
}
```

### Mockito. Spy

**@Spy** or the **spy()** method can be used to wrap a real object. Every call, unless specified otherwise, is delegated to the object.

```java
@Test
public void testSpy() {
    List<String> list = new LinkedList<>();
    List<String> spy = spy(list);

    doReturn("foo").when(spy).get(0);

    assertEquals("foo", spy.get(0));
}
```

### Mockito. Behavior testing

You can use the verify() method on the mock object to verify that the specified conditions are met. For example, you can verify that a method has been called with certain parameters.

```java
@Test
public void testVerify()  {
    MyClass test = Mockito.mock(Someclass.class);

    test.someEntryMethod(12);

    verify(test).someMethod(ArgumentMatchers.eq(12));
    verify(test, times(2)).someMethod();
    verify(test, never()).someMethod("never called");
    verify(test, atLeastOnce()).someMethod("called at least once");
    verify(test, atLeast(2)).someMethod("called at least twice");
    verify(test, times(5)).someMethod("called five times");
    verify(test, atMost(3)).someMethod("called at most 3 times");
    verifyNoMoreInteractions(test);
}
```

[Content](#testing)
# PowerMock [![Maven][powermock-mavenbadge]][powermock-maven] [![Source][powermock-sourcebadge]][powermock-source] ![powermock-starsbadge]

PowerMock is a framework that extends other mock libraries such as Mockito and EasyMock, but adds more capacity. PowerMock uses custom classloader and bytecode manipulation to allow mocking of static methods, builders, final classes, private methods and more. So we can test our code without modifying it, thanks to the fact that the mock objects are hooked to the code to be tested in execution time. By using reflection this framework allows you to modify even private attributes of the class.

```gradle
dependencies {
    testImplementation 'org.powermock:powermock-core:2.0.0' 
}
```

### PowerMock. Test setup

```java
@RunWith(PowerMockRunner.class)
@PrepareForTest( { YourClassWithEgStaticMethod.class })
public class YourTestCase {
...
}
```

### PowerMock. Bypass encapsulation

**Whitebox** class provides a set of methods which could you help bypass encapsulation if it required. Usually, it's not a good idea to get/modify non-public fields, but sometimes it's only way to cover code by test for future refactoring.

- **Whitebox.setInternalState(..)** to set a non-public member of an instance or class.
- **Whitebox.getInternalState(..)** to get a non-public member of an instance or class.
- **Whitebox.invokeMethod(..)** to invoke a non-public method of an instance or class.
- **Whitebox.invokeConstructor(..)** to create an instance of a class with a private constructor.
- **Whitebox.invokeMethod(..)** to invoke a private method.
- **WhiteBox.invokeConstructor(..)** to instantiate a class with a private constructor.

```java
public class ServiceHolder {
    private final Set<Object> services = new HashSet<Object>();

    public void addService(Object service) { services.add(service); }
    public void removeService(Object service) { services.remove(service); }
}

@Test
public void testAddService() throws Exception {
    ServiceHolder tested = new ServiceHolder();
    final Object service = new Object();

    tested.addService(service);

    // This is how you get the private services set using PowerMock
    Set<String> services = Whitebox.getInternalState(tested, "services");
    // Or
    Set<String> services = Whitebox.getInternalState(tested, Set.class);

    assertEquals("Size of the \"services\" Set should be 1",
        1, services.size());
    assertSame("The services Set should didn't contain the expect service",
        service, services.iterator().next());
}
```

### PowerMock. Suppressing unwanted behavior

- **Whitebox.newInstance(ClassWithEvilConstructor.class)** to instantiate a class without invoking the constructor what so ever.
- **suppress(constructor(EvilParent.class))** to suppress all constructors for the EvilParent class.
- **suppress(method(ClassWithEvilMethod.class, "methodName"))** to suppress the method in the class.
- **suppress(field(ClassWithEvilField.class, "fieldName"))** to suppress the field in the class.
- **@SuppressStaticInitializationFor("org.mycompany.SomeClass")** annotation to remove the static initializer for the class.

[Content](#testing)
# Espresso [![Maven][espresso-mavenbadge]][espresso-maven] [![Source][espresso-sourcebadge]][espresso-source]

UI testing framework. Provides APIs for writing UI tests to simulate user interactions within an app. Provides automatic synchronization of test actions with the UI of the app you are testing. Espresso detects when the main thread is idle, so it is able to run test commands at the appropriate time, improving the reliability of tests. Uses a separate apk to run tests.

```gradle
dependencies {
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.1.2-alpha01'
    
    // old namespace 'com.android.support.test.espresso:espresso-core:3.0.2'
}
```

### Instrumented (or Instrumentation, or UI) tests

Located at module-name/src/androidTest/java/ .

You can run instrumented unit tests on a physical device or emulator, which doesn't involve any mocking or stubbing of the framework. Because this form of testing involves significantly slower execution times than local unit tests, however, it's best to rely on this method only when it's essential to evaluate your app's behavior against actual device hardware.

### Espresso. Tests structure

- Activity will be launched using the @Rule annotation. By default the rule will be initialised and the activity will be launched before running every @Before method. Destroyed after running the @After method.

- Use JUnit annotations: @Test, @BeforeClass, @Before, @After, @AfterClass.

- Find the UI component you want to test in an Activity (for example, a sign-in button in the app) by calling the onView() method, or the onData() method for AdapterView controls, with specified matcher parameter.

- Simulate a specific user interaction to perform on that UI component, by calling the ViewInteraction.perform() or DataInteraction.perform() method and passing in the user action (for example, click on the sign-in button). To sequence multiple actions on the same UI component, chain them using a comma-separated list in your method argument.

- Use the ViewAssertions methods with ViewInteraction.check() or DataInteraction.check() to check that the UI reflects the expected state or behavior, after these user interactions are performed.

### Espresso. Basic test

```java
onView(ViewMatcher)
    .perform(ViewAction)
    .check(ViewAssertion);
```

```java
@RunWith(AndroidJUnit4.class)
public class ActivityTest {

    @Rule
    public final ActivityTestRule<MyActivity> mActivityTestRule
		= new ActivityTestRule<>(MyActivity.class);

    @Test
    public void basicTest() {
        onView(withId(R.id.my_view)).perform(click()).check(matches(isDisplayed()));
    }
}
```

### Espresso. Test examples

```java
// Testing view visibility.
onView(withText(R.string.some_text)).check(matches(isDisplayed()));
onView(withId(R.id.some_view)).check(matches(not(isDisplayed())));
onView(withText(containsString("some_word"))).check(doesNotExist());
onView(allOf(
    hasSibling(withText(R.string.prefs_date)),
    withText(R.string.prefs_not_set)))
    .check(matches(isDisplayed()));

// Checking view positioning.
onView(withText(R.string.some_text)).check(isCompletelyAbove(withText(R.string.other_text)));

// Clicking view on a ViewPager.
onView(allOf(withText(R.string.some_text), isCompletelyDisplayed())).perform(click());

// Typing text in EditText.
onView(withId(android.R.id.edit)).perform(typeText(text));

// Picking date on DatePicker.
onView(withClassName(equalTo(DatePicker.class.getName())))
    .perform(PickerActions.setDate(year, month, day));

// Scrolling ScrollView.
onView(withId(R.id.some_view)).perform(scrollTo()).check(matches(isDisplayed()));

// Scrolling RecyclerView.
onView(withId(R.id.recycler_view))
    .perform(RecyclerViewActions.scrollTo(hasDescendant(withText(R.string.some_text))));
    
// Check for keyboard
static boolean isKeyboardShown() {
    Context targetContext = ApplicationProvider.getApplicationContext();
    InputMethodManager inputMethodManager = (InputMethodManager) targetContext.getSystemService(Context.INPUT_METHOD_SERVICE);
    return inputMethodManager != null && inputMethodManager.isActive() && inputMethodManager.isAcceptingText();
}

// Check for toast
static void toastTextShowing(String text) {
    onView(withText(text)).inRoot(isToast()).check(matches(isDisplayed()));
}

private static Matcher<Root> isToast() {
    return new TypeSafeMatcher<Root>() {
        @Override
        protected boolean matchesSafely(Root item) {
            int type = item.getWindowLayoutParams().get().type;
            return type == WindowManager.LayoutParams.TYPE_TOAST;
        }

        @Override
        public void describeTo(Description description) {
            description.appendText("is toast");
        }
    };
}
```



[Content](#testing)
# Sources



[junit-maven]: https://search.maven.org/artifact/junit/junit
[junit-mavenbadge]: https://maven-badges.herokuapp.com/maven-central/junit/junit/badge.svg
[junit-source]: https://github.com/junit-team/junit4
[junit-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg
[junit-starsbadge]: https://img.shields.io/github/stars/junit-team/junit4

[mockito-maven]: https://search.maven.org/artifact/org.mockito/mockito-core
[mockito-mavenbadge]: https://maven-badges.herokuapp.com/maven-central/org.mockito/mockito-core/badge.svg
[mockito-source]: https://github.com/mockito/mockito
[mockito-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg
[mockito-starsbadge]: https://img.shields.io/github/stars/mockito/mockito

[powermock-maven]: https://search.maven.org/artifact/org.powermock/powermock-core
[powermock-mavenbadge]: https://maven-badges.herokuapp.com/maven-central/org.powermock/powermock-core/badge.svg
[powermock-source]: https://github.com/powermock/powermock
[powermock-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg
[powermock-starsbadge]: https://img.shields.io/github/stars/powermock/powermock

[espresso-maven]: https://mvnrepository.com/artifact/androidx.test.espresso/espresso-core
[espresso-mavenbadge]: https://img.shields.io/badge/maven%20google--brightgreen.svg
[espresso-source]: https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso
[espresso-sourcebadge]: https://img.shields.io/badge/source-google-orange.svg
