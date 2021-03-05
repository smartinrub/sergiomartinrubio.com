---
title: Take Unit Testing to the Next Level with JUnit 5
image: /assets/images/shubham-beeharry-223969.jpg
categories:
    - Testing
    - Java Framework
mermaid: false
layout: post
---

**JUnit** is the most popular testing framework in Java, and with [**JUnit 5**](https://junit.org/junit5/){:target="_blank"} testing in **Java 8** and beyond takes another step forward. This version was release in September 2017 and has been actively updated to fix bugs and add new features. Moreover, JUnit 5 is also compatible with version 3 and 4 by adding [`junit-vintage-engine`](https://mvnrepository.com/artifact/org.junit.vintage/junit-vintage-engine){:target="_blank"} to your classpath path.

### Migrating from JUnit 4

When [migrating from JUnit 4](https://junit.org/junit5/docs/current/user-guide/#migrating-from-junit4){:target="_blank"} there are a few considerations to bear in mind:

- To have **JUnit 5** annotations you need to add to your pom file:

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>${junit.jupiter.version}</version>
    <scope>test</scope>
</dependency>
```

- Replace `@BeforeClass`, `@Before`, `@AfterClass` and `@After` annotations with `@BeforeAll`, `@BeforeEach`, `@AfterAll` and `@AfterEach` alternatively.

```java
@BeforeAll
static void initAll() {
    LOGGER.info("@BeforeAll runs once before all tests");
}

@BeforeEach
void init() {
    LOGGER.info("@BeforeEach runs before each test");
}

@AfterAll
static void tearDownAll() {
    LOGGER.info("@AfterAll runs once after all tests");
}

@AfterEach
void tearDown() {
    LOGGER.info("@AfterEach runs after each test");
}
```

- Replace `@Ignore` with `@Disabled`. _JUnit 5_ also provides a more powerful filter in order to ignore or disable tests by _JavaScript_ expressions, _OS_, _Java_ version or system properties

```java
@Test
@Disabled("Disabled test (it used to be @Ignore in jUnit4)")
void skippedTest() {
    // not executed
}

@Test
@EnabledIf(value = "true", reason = "test runs because is true")
void isExecuted() {
}

@DisabledIf("Math.random() < 0.5")
@RepeatedTest(10)
void sometimesIsExecuted() {
    System.out.println("Running " + counter++);
}
```

- `@Category` need to be replaced with `@Tag`.

```java
@Tag("myTestSuite")
class TagsTests {

    @Test
    @Tag("myTest")
    void tagExample() {

    }
}
```

- `@RunWith` does not exist anymore, use `@ExtendWith` instead.
- `@Rule` and `@ClassRule` were also removed, use `@ExtendWith` instead.

### New JUnit 5 Features

#### @DisplayName

`@DisplayName` allows you to override a test class or method with a custom message, special characters or even Emojis (😄).

```java
@DisplayName("Display name test suite")
class DisplayNamesTests {
    @Test
    @DisplayName("My Custom Test Name")
    void displayNameTest() {
    }
    @Test
    @DisplayName("\uD83D\uDE03") // Happy Emoji
    void displayEmojiTest() {
    }
}
```

#### Assertions

Assertions include four new improvements:

1. Compatibility with lambda expression.

```java
assertEquals(
        Stream.of(1, 4, 5).mapToInt(Integer::intValue).sum(),
        10,
        "Sum should be 10");
```

2. Group assertions.

- **assertAll()**

```java
List<String> names = Arrays.asList("Sergio", "Juan", "Adolfo");
assertAll("names",
        () -> assertEquals("Sergio", names.get(0)),
        () -> assertEquals("Juan", names.get(1)),
        () -> assertEquals("Adolfo", names.get(2)));
```

3. More control over exceptions. Now you can even inspect the returning exception to verify message, cause, stacktrace…

- **assertThrows()**

```java
Throwable runtimeException = assertThrows(RuntimeException.class, () -> {
    throw new RuntimeException("exception");
});
assertEquals("exception", runtimeException.getMessage());
```

4. Timeout assertions:

- **assertTimeout()**

```java
@Test
void timeoutNotExceeded() {
    assertTimeout(Duration.ofMinutes(2), () -> System.out.println("Hello"));
}
@Test
void timeoutNotExceededWithResult() {
    String result  = assertTimeout(Duration.ofMinutes(2), () -> "hello");
    assertEquals("hello", result);
}
@Test
@Disabled("Assertion fails with exceeded timeout of 2ms by 8ms")
void timeoutExceeded() {
    assertTimeout(Duration.ofMillis(2), () -> {
        sleep(10);
    });
}
```

#### Assumptions

- `assumeTrue()`
- `assumeFalse()`
- `assumingThat()`

Assumptions are preconditions that need to be satisfied to run subsequent assertions. If the assumption fails TestAbortedException is thrown and the complete test is skipped.

```java
@Test
void trueAssumption() {
    assumeTrue(true);
}
@Test
void falseAssumption() {
    assumeFalse(false);
}
@Test
void assumptionThat() {
    String word = "word";
    assumingThat(
            "word".equals(word),
            () -> assertEquals(3, 2 + 1)
    );
}
```

#### @Nested

`@Nested` gives your more freedom to create groups of related test in the same test suite.

```java
class NestedTests {
    private List<String> strings;
    @Nested
    class listIsInstantiated {
        @BeforeEach
        void init() {
            strings = new ArrayList<>();
        }
        @Test
        void listIsEmpty() {
            assertTrue(strings.isEmpty());
        }
        @Nested
        class afterAddingString {
            @BeforeEach
            void init() {
                strings.add("hello");
            }
            @Test
            void listIsNotEmpty() {
                assertFalse(strings.isEmpty());
            }
        }
    }
}
```

> Note: `@BeforeAll` and `@AfterAll` are only allow if the test class is annotated with `@TestInstance(Lifecycle.PER_CLASS)`." %}

#### Dependency Injection

**JUnit 5** allows to add parameters in constructors and test methods. Therefore, now constructors and methods annotated with `@Test`, `@TestFactory`, `@BeforeEach`, `@AfterEach`, `@BeforeAll`, `@AfterAll` accept parameters.

There are three kind of allowed parameters:

- **TestInfo**: allows you to get information related to a test suite or test method such as display name, tags, test class…

```java
DependencyInjectionTests(TestInfo testInfo) {
    assertEquals("Dependency Injection Test Suite", testInfo.getDisplayName());
}
@Nested
class testInfoExamples {
    @BeforeEach
    void init(TestInfo testInfo) {
        String displayName = testInfo.getDisplayName();
        assertTrue(displayName.equals("Test Example") || displayName.equals("myTest()"));
    }
    @Test
    @DisplayName("Test Example")
    @Tag("my-tag")
    void test(TestInfo testInfo) {
        assertTrue(testInfo.getTags().contains("my-tag"));
    }
    @Test
    void myTest() {
    }
}
```

- **TestReporter**: is used to print test information to stout or stderr.

```java
@Test
void testReporterString(TestReporter testReporter) {
    // Use test reporter to print information to stout or stderr
    testReporter.publishEntry("my message");
}
@Test
void testReporterKeyValue(TestReporter testReporter) {
    testReporter.publishEntry("myKey", "myValue");
}
@Test
void testReporterMap(TestReporter testReporter) {
    Map<String, String> myMap = new HashMap<>();
    myMap.put("myKey", "myValue");
    testReporter.publishEntry(myMap);
}
```

- **RepetitionInfo**: can be used only on test annotated with @RepeatedTest and provides information regarding current repetition number or total of repetitions.

```java
private static int repetition = 1;
@RepeatedTest(10)
@DisplayName("repetition")
void repetitionTest(RepetitionInfo repetitionInfo) {
    assertEquals(repetition++, repetitionInfo.getCurrentRepetition());
    assertEquals(10, repetitionInfo.getTotalRepetitions());
}
```

#### Interfaces and Default methods

Interfaces create contracts to implement in your test suites. Methods declared as default in an interface will always run in a test suite.

```java
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
interface Interfaces {
    @BeforeAll
    default void initAll() {
        System.out.println("@BeforeAll runs once before all tests");
    }
    @BeforeEach
    default void init() {
        System.out.println("@BeforeEach runs before each test");
    }
    @AfterAll
    default void tearDownAll() {
        System.out.println("@AfterAll runs once after all tests");
    }
    @AfterEach
    default void tearDown() {
        System.out.println("@AfterEach runs after each test");
    }
}
class ClassImplementingInterfaceTests implements Interfaces {
    @Test
    void myTest() {
    }
}
```

> Note: `@TestInstance(TestInstance.Lifecycle.PER_CLASS)` is required to make `@BeforeAll` and `@AfterAll` work.

#### @RepeatedTest

`@RepeatedTest` allows you to repeat a test as many times as you want by passing the number of desire repetitions to the annotation.

```java
@RepeatedTest(value = 10, name = "{displayName} {currentRepetition}/{totalRepetitions}")
void repetitionTest() {
}
```

#### @ParameterizedTest

When using `@ParameterizedTest` annotation in combination with

- `@ValueSource`
- `@EnumSource`
- `@MethodSource`
- `@CsvSource`
- `@CsvFileSource`
- `@ArgumentsSource`

you can run a test multiple times with different inputs each time.

```java
class ParameterizedTests {
    enum MyEnum {RED, GREEN, BLACK}
    @ParameterizedTest
    @ValueSource(ints = {2, 4, 8})
    void integerParameters(int value) {
        assertEquals(0, value % 2);
    }
    @ParameterizedTest
    @EnumSource(value = MyEnum.class, names = {"RED", "GREEN"})
    void integerParameters(MyEnum myEnum) {
        assertTrue(EnumSet.of(MyEnum.RED, MyEnum.GREEN).contains(myEnum));
    }
    @ParameterizedTest
    @MethodSource("myStrings")
    void methodSourceProvider(String string) {
        assertTrue(Arrays.asList("hello", "world").contains(string));
    }
    static List<String> myStrings() {
        return Arrays.asList("hello", "world");
    }
}
```

> Note: For `@MethodSource`, if method has same name as test, parameter name is not required.
To use this feature, `junit-jupiter-params` needs to be added to the classpath.

### Conclusion

As you can see, JUnit 5 is a huge improvement over previous versions and introduces many new testing features. It also allows you to write more expressive units test when using in combination with lambda expressions from Java 8.

<p class="text-center">
{% include elements/button.html link="https://github.com/smartinrub/junit5-features" text="Source Code" %}
</p>
