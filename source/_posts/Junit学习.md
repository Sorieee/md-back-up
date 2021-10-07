《Junit实战》第3版

# Jump start

*  Each unit test should run independently of all other unit tests.
* The framework should detect and report errors test by test
* It should be easy to define which unit tests will run.

The team has defined three discrete goals for the framework:

* The framework must help us write useful tests.
* The framework must help us create tests that retain their value over time.
* The framework must help us lower the cost of writing tests by reusing code.



​	The dependencies that are always needed in the pom.xml file are shown in the following listing. In the beginning, you need only`junit-jupiter-api` and `junit-jupiter-engine`.

```xml
<dependency>
     <groupId>org.junit.jupiter</groupId>
     <artifactId>junit-jupiter-api</artifactId>
     <version>5.6.0</version>
     <scope>test</scope>
</dependency>
<dependency>
     <groupId>org.junit.jupiter</groupId>
     <artifactId>junit-jupiter-engine</artifactId>
     <version>5.6.0</version>
     <scope>test</scope>
</dependency>
<build>
    <plugins>
         <plugin>
             <artifactId>maven-surefire-plugin</artifactId>
             <version>2.22.2</version>
        </plugin>
    </plugins>
</build>
```

## Testing with JUnit

* Separate test class instances and class loaders for each unit test to prevent side effects
* JUnit annotations to provide resource initialization and cleanup methods: `@BeforeEach`, `@BeforeAll`, `@AfterEach`, and`@AfterAll` (starting from version 5); and `@Before`,`@BeforeClass`, `@After`, and `@AfterClass` (up to version 4)
* A variety of assert methods that make it easy to check the results of your tests
* Integration with popular tools such as Maven and Gradle, as well as popular integrated development environments (IDEs) such as Eclipse, NetBeans, and IntelliJ

```java
public class CalculatorTest {

    @Test
    public void testAdd() {
        Calculator calculator = new Calculator();
        double result = calculator.add(10, 50);
        assertEquals(60, result, 0);
    }
}
```

​	JUnit 3 required extending the `TestCase` class, but this requirement was removed with JUnit 4. Also, up to JUnit 4, the class had to be public; starting with version 5, the test class can be public or private, and you can name it whatever you want.

​	`delta` provides a range factor: if the actual value is within the range`expected - delta` and `expected + delta`, the test will pass. 

# Exploring Core Unit

* *A test class* may be a top-level class, static member class, or an inner class annotated as `@Nested` that contains one or more test methods. Test classes cannot be abstract and must have a single constructor. The constructor must have no arguments or arguments that can be dynamically resolved at runtime through dependency injection. (We discuss the details of dependency injection in section 2.6.) A test class is allowed to be package-private as a minimum requirement for visibility. It is no longer required that test classes be public, as was the case up to JUnit 4.x. In our example, because we do not define any other constructors, we do not need to define the no-arguments constructor; the Java compiler will supply a no-args constructor.
* *A test method* is an instance method that is annotated with `@Test`, `@RepeatedTest`, `@ParameterizedTest`,`@TestFactory`, or `@TestTemplate`.
* *A life cycle method* is a method that is annotated with `@BeforeAll`, `@AfterAll`, `@BeforeEach`, or `@AfterEach`



​	Test methods must not be abstract and must not return a value (the return type should be void).

​	If you annotate your test class with `@TestInstance(Lifecycle.PER_CLASS)`, JUnit 5 will execute all test methods on the same test instance. A new test instance will be created for each test class when using this annotation.

​	Listing 2.3 shows the use of the JUnit 5 life cycle methods in the `lifecycle.SUTTest` class.

```java
class SUTTest {
    private static ResourceForAllTests resourceForAllTests;
    private SUT systemUnderTest;

    @BeforeAll
    static void setUpClass() {
        resourceForAllTests = new ResourceForAllTests("Our resource for all tests");
    }

    @AfterAll
    static void tearDownClass() {
        resourceForAllTests.close();
    }

    @BeforeEach
    void setUp() {
        systemUnderTest = new SUT("Our system under test");
    }

    @AfterEach
    void tearDown() {
        systemUnderTest.close();
    }

    @Test
    void testRegularWork() {
        boolean canReceiveRegularWork = systemUnderTest.canReceiveRegularWork();

        assertTrue(canReceiveRegularWork);
    }

    @Test
    void testAdditionalWork() {
        boolean canReceiveAdditionalWork = systemUnderTest.canReceiveAdditionalWork();

        assertFalse(canReceiveAdditionalWork);
    }
}
```

```
	Our resource for all tests from class ResourceForAllTests is initializing.

    Our system under test from class SUT is initializing.
    Our system under test from class SUT can receive regular work.
    Our system under test from class SUT is closing.


    Our system under test from class SUT is initializing.
    Our system under test from class SUT cannot receive additional work.
    Our system under test from class SUT is closing.

    Our resource for all tests from class ResourceForAllTests is closing.
    Disconnected from the target VM, address: '127.0.0.1:57960', transport: 'socket'

    Process finished with exit code 0
```

* The method annotated with `@BeforeAll` #1 is executed once: before all tests. This method needs to be static unless the whole test class is annotated with `@TestInstance(Lifecycle.PER_CLASS)`.
* The method annotated with `@BeforeEach` #3 is executed before each test. In our case, it will be executed twice.
* The two methods annotated with `@Test` #5 are executed independently.
* The method annotated with `@AfterEach` #4 is executed after each test. In our case, it will be executed twice.
* The method annotated with `@AfterAll` #2 is executed once: after all tests. This method needs to be static unless the whole test class is annotated with `@TestInstance(Lifecycle.PER_CLASS)`.
*  In order to run this test class, you can execute from the command line: `mvn -Dtest=SUTTest.java clean install`

## **The @DisplayName annotation**

​	The `@DisplayName` annotation can be used over classes and test methods. It helps the engineers at Tested Data Systems declare their own display name for an annotated test class or test method.

```java
class DisplayNameTest {
    private SUT systemUnderTest = new SUT();

    @Test
    @DisplayName("Our system under test says hello.")
    void testHello() {
        assertEquals("Hello", systemUnderTest.hello());
    }

    @Test
    @DisplayName("😱")
    void testTalking() {
        assertEquals("How are you?", systemUnderTest.talk());
    }

    @Test
    void testBye() {
        assertEquals("Bye", systemUnderTest.bye());
    }
}
```

![](https://pic.imgdb.cn/item/610f45f45132923bf8c20fe2.jpg)

### The @Disabled annotation

```java
@Disabled("Feature is still under construction.")
class DisabledClassTest {
}

	@Test
    @Disabled("Feature still under construction.")
    void testAdditionalWork() {
        boolean canReceiveAdditionalWork = systemUnderTest.canReceiveAdditionalWork();

        assertFalse(canReceiveAdditionalWork);
    }
```

## Nested tests

```java
public class NestedTestsTest {
    private static final String FIRST_NAME = "John";
    private static final String LAST_NAME = "Smith";

    @Nested
    class BuilderTest {
        private String MIDDLE_NAME = "Michael";

        @Test
        void customerBuilder() throws ParseException {
            SimpleDateFormat simpleDateFormat = new SimpleDateFormat("MM-dd-yyyy");
            Date customerDate = simpleDateFormat.parse("04-21-2019");

            Customer customer = new Customer.Builder(Gender.MALE, FIRST_NAME, LAST_NAME)
                    .withMiddleName(MIDDLE_NAME)
                    .withBecomeCustomer(customerDate)
                    .build();


            assertAll(() -> {
                assertEquals(Gender.MALE, customer.getGender());
                assertEquals(FIRST_NAME, customer.getFirstName());
                assertEquals(LAST_NAME, customer.getLastName());
                assertEquals(MIDDLE_NAME, customer.getMiddleName());
                assertEquals(customerDate, customer.getBecomeCustomer());
            });
        }
    }
}
```

## Tagged tests

​	If you are familiar with JUnit 4, *tagged tests* are replacements for JUnit 4 categories. You can use the `@Tag` annotation over classes and test methods. Later, you can use tags to filter test discovery and execution.

```java
@Tag("individual")
public class CustomerTest {
    private String CUSTOMER_NAME = "John Smith";

    @Test
    void testCustomer() {
        Customer customer = new Customer(CUSTOMER_NAME);

        assertEquals("John Smith", customer.getName());
    }
}
```

```java
@Tag("repository")
public class CustomersRepositoryTest {
    private String CUSTOMER_NAME = "John Smith";
    private CustomersRepository repository = new CustomersRepository();

    @Test
    void testNonExistence() {
        boolean exists = repository.contains("John Smith");

        assertFalse(exists);
    }

    @Test
    void testCustomerPersistence() {
        repository.persist(new Customer(CUSTOMER_NAME));

        assertTrue(repository.contains("John Smith"));
    }
}
```

![](https://pic.imgdb.cn/item/610f47ba5132923bf8c43c2e.jpg)

​	To activate the tags, you have a few alternatives. One is to work at the level of the pom.xml configuration file. In this example, it’s enough to uncomment the configuration node of the Surefire plugin #1 and run `mvn clean install`.

​	Another alternative in the IntelliJ IDEA IDE is to activate the tags by creating a configuration by choosing Run > Run… > Edit Configurations > Tags (JUnit 5) as the test kind (figure 2.2). This is fine when you would like to quickly make some changes about which tests to run locally. However, it is strongly recommended that you make the changes at the level of the pom.xml file—otherwise, any automated build of the project will fail.

![](https://pic.imgdb.cn/item/610f47e05132923bf8c46afd.jpg)

## Assertions

| assert method                                    | What is it used for                                          |
| ------------------------------------------------ | ------------------------------------------------------------ |
| assertAll                                        | Overloaded method. It asserts that none of the supplied executables throw exceptions. An executable is an object of type org.junit.jupiter.api.function.Executable. |
| assertArrayEquals                                | Overloaded method. It asserts that the expected array and the actual array are equal. |
| assertEquals                                     | Overloaded method. It asserts that the expected values and the actual values are equal. |
| assert*X*(..., String message)                   | Assertion that delivers the supplied message to the test framework if the assertion fails |
| assert*X*(..., Supplier<String> messageSupplier) | Assertion that delivers the supplied message to the test framework if the assertion fails. The failure message will be retrieved lazily from the supplied messageSupplier. |

​	JUnit 5 provides a lot of overloaded assertion methods. It takes many assertion methods from JUnit 4 and adds a few that can use Java 8 lambdas. All JUnit Jupiter assertions belong to the `org.junit.jupiter.api.Assertions` class and are static methods. 



**assertAll**

```java
class AssertAllTest {
    @Test
    @DisplayName("SUT should default to not being under current verification")
    void testSystemNotVerified() {
        SUT systemUnderTest = new SUT("Our system under test");

        assertAll("By default, SUT is not under current verification",
                () -> assertEquals("Our system under test", systemUnderTest.getSystemName()),
                () -> assertFalse(systemUnderTest.isVerified())
        );
    }

    @Test
    @DisplayName("SUT should be under current verification")
    void testSystemUnderVerification() {
        SUT systemUnderTest = new SUT("Our system under test");

        systemUnderTest.verify();

        assertAll("SUT should be under current verification",
                () -> assertEquals("Our system under test", systemUnderTest.getSystemName()),
                () -> assertTrue(systemUnderTest.isVerified())
        );
    }
}
```

​	The `assertAll` method will always check all the assertions that are provided to it, even if some of them fail—if any of the executables fail, the remaining ones will still be run. 

**Some assertion methods with messages**

![](https://pic.imgdb.cn/item/610f49215132923bf8c5f892.jpg)



* A condition is verified with the help of the `assertTrue` method #1. In case of failure, a message is lazily created #2.
* A condition is verified with the help of the `assertFalse` method #3. In case of failure, a message is lazily created #4.
* The existence of an object is verified with the help of the `assertNull` method #5. In case of failure, a message is lazily created #6.

**Some assertTimeout methods**

```java
class AssertTimeoutTest {
    private SUT systemUnderTest = new SUT("Our system under test");

    @Test
    @DisplayName("A job is executed within a timeout")
    void testTimeout() throws InterruptedException {
        systemUnderTest.addJob(new Job("Job 1"));
        assertTimeout(ofMillis(500), () -> systemUnderTest.run(200));
    }

    @Test
    @DisplayName("A job is executed preemptively within a timeout")
    void testTimeoutPreemptively() throws InterruptedException {
        systemUnderTest.addJob(new Job("Job 1"));
        assertTimeoutPreemptively(ofMillis(500), () -> systemUnderTest.run(200));
    }
```

​	`assertTimeout` waits until the executable finishes #1. The failure message looks something like this: `execution exceeded timeout of 500 ms by 193 ms`.

​	`assertTimeoutPreemptively` stops the executable when the time has expired #2). The failure message looks like this:`execution timed out after 100 ms`.

**Some assertThrows methods**

```java
class AssertThrowsTest {
    private SUT systemUnderTest = new SUT("Our system under test");

    @Test
    @DisplayName("An exception is expected")
    void testExpectedException() {
        assertThrows(NoJobException.class, systemUnderTest::run);
    }

    @Test
    @DisplayName("An exception is caught")
    void testCatchException() {
        Throwable throwable = assertThrows(NoJobException.class, () -> systemUnderTest.run(1000));

        assertEquals("No jobs on the execution list!", throwable.getMessage());
    }
}
```

In this example:

*  We verify that the `systemUnderTest` object’s call of the `run` method throws a `NoJobException` #1.
* We verify that a call to `systemUnderTest.run(1000)` throws a `NoJobException`, and we keep a reference to the thrown exception in the `throwable` variable #2.
* We check the message kept in the `throwable` exception variable #3.

## Assumptions

​	Sometimes tests fail due to an external environment configuration or a date or time zone issue that we cannot control. We can prevent our tests from being executed under inappropriate conditions.

​	JUnit 5 comes with a set of assumption methods suitable for use with Java 8 lambdas. The JUnit 5 assumptions are static methods belonging to the `org.junit.jupiter.api.Assumptions` class. The `message` parameter is in the last position.

**Some assumption methods**

```java
class AssumptionsTest {
    private static String EXPECTED_JAVA_VERSION = "1.8";
    private TestsEnvironment environment = new TestsEnvironment(
            new JavaSpecification(System.getProperty("java.vm.specification.version")),
            new OperationSystem(System.getProperty("os.name"), System.getProperty("os.arch"))
    );

    private SUT systemUnderTest = new SUT();

    @BeforeEach
    void setUp() {
        assumeTrue(environment.isWindows());
    }

    @Test
    void testNoJobToRun() {
        assumingThat(
                () -> environment.getJavaVersion().equals(EXPECTED_JAVA_VERSION),
                () -> assertFalse(systemUnderTest.hasJobToRun()));
    }

    @Test
    void testJobToRun() {
        assumeTrue(environment.isAmd64Architecture());

        systemUnderTest.run(new Job());

        assertTrue(systemUnderTest.hasJobToRun());
    }
}
```

* The `@BeforeEach` annotated method is executed before each test. The test will not run unless the assumption that the current environment is Windows is true #1.
* The first test checks that the current Java version is the expected one #2. Only if this assumption is true does it verify that no job is currently being run by the SUT #3.)
* The second test checks the current environment architecture #4. Only if this architecture is the expected one does it run a new job on the SUT #5 and verify that the system has a job to run #6.

## Dependency injection in JUnit 5

​	The previous JUnit versions did not permit test constructors or methods to have parameters. JUnit 5 allows test constructors and methods to have parameters, but they need to be resolved through dependency injection.

**TestInfoParameterResolver**

```java
class TestInfoTest {

    TestInfoTest(TestInfo testInfo) {
        assertEquals("TestInfoTest", testInfo.getDisplayName());
    }

    @BeforeEach
    void setUp(TestInfo testInfo) {
        String displayName = testInfo.getDisplayName();
        assertTrue(displayName.equals("display name of the method") || displayName.equals("testGetNameOfTheMethod(TestInfo)"));
    }

    @Test
    void testGetNameOfTheMethod(TestInfo testInfo) {
        assertEquals("testGetNameOfTheMethod(TestInfo)", testInfo.getDisplayName());
    }

    @Test
    @DisplayName("display name of the method")
    void testGetNameOfTheMethodWithDisplayNameAnnotation(TestInfo testInfo) {
        assertEquals("display name of the method", testInfo.getDisplayName());
    }
}
```

* A `TestInfo` parameter is injected into the constructor and into three methods. The constructor verifies that the display name is `TestInfoTest`: its own name #1. This behavior is the default behavior, which we can vary using `@DisplayName`annotations.
* The `@BeforeEach` annotated method is executed before each test. It has an injected `TestInfo` parameter, and it verifies that the displayed name is the expected one: the name of the method or the name specified by the `@DisplayName`annotation #2.
*  Both tests have an injected `TestInfo` parameter. Each parameter verifies that the displayed name is the expected one: the name of the method in the first test #3 or the name specified by the `@DisplayName` annotation in the second test #4.
* The built-in `TestInfoParameterResolver` supplies an instance of `TestInfo` that corresponds to the current container or test as the value of the expected parameters of the constructor and of the methods.

**TestReporterParameterResolver**

```java
class TestReporterTest {

    @Test
    void testReportSingleValue(TestReporter testReporter) {
        testReporter.publishEntry("Single value");
    }

    @Test
    void testReportKeyValuePair(TestReporter testReporter) {
        testReporter.publishEntry("Key", "Value");
    }

    @Test
    void testReportMultipleKeyValuePairs(TestReporter testReporter) {
        Map<String, String> values = new HashMap<>();
        values.put("user", "John");
        values.put("password", "secret");

        testReporter.publishEntry(values);
    }
}
```

* In the first method, it is used to publish a single value entry 1).
*  In the second method, it is used to publish a key-value pair #2.
* In the third method, we construct a map #3, populate it with two key-value pairs #4, and then use it to publish the constructed map #5.
* The built-in `TestReporterParameterResolver` supplies the instance of `TestReporter` needed to publish the entries.

![](https://pic.imgdb.cn/item/610f69c05132923bf8f1a411.jpg)

**RepetitionInfoParameterResolve**r

​	If a parameter in a method annotated with `@RepeatedTest`, `@BeforeEach`, or `@AfterEach` is of type `RepetitionInfo`,`RepetitionInfoParameterResolver` supplies an instance of this type`.` Then `RepetitionInfo` gets information about the current repetition and the total number of repetitions for a test annotated with `@RepeatedTest`. Repeated tests and examples are discussed in the following section.

## Repeated tests

* `{displayName}`—Display name of the method annotated with `@RepeatedTest`
* `{currentRepetition}`—Current repetition number
* `{totalRepetitions}`—Total number of repetitions

```java
public class RepeatedTestsTest {

    private static Set<Integer> integerSet = new HashSet<>();
    private static List<Integer> integerList = new ArrayList<>();

    @RepeatedTest(value = 5, name = "{displayName} - repetition {currentRepetition}/{totalRepetitions}")
    @DisplayName("Test add operation")
    void addNumber() {
        Calculator calculator = new Calculator();
        assertEquals(2, calculator.add(1, 1), "1 + 1 should equal 2");
    }

    @RepeatedTest(value = 5, name = "the list contains {currentRepetition} elements(s), the set contains 1 element")
    void testAddingToCollections(TestReporter testReporter, RepetitionInfo repetitionInfo) {
        integerSet.add(1);
        integerList.add(repetitionInfo.getCurrentRepetition());

        testReporter.publishEntry("Repetition number", String.valueOf(repetitionInfo.getCurrentRepetition()));
        assertEquals(1, integerSet.size());
        assertEquals(repetitionInfo.getCurrentRepetition(), integerList.size());
    }
}
```

## Parameterized tests

​	`@ValueSource` lets us specify a single array of literal values. At execution, this array provides a single argument for each invocation of the parameterized test. The following test checks the number of words in some phrases that are provided as parameters.

```java
class ParameterizedWithValueSourceTest {
    private WordCounter wordCounter = new WordCounter();

    @ParameterizedTest
    @ValueSource(strings = {"Check three parameters", "JUnit in Action"})
    void testWordsInSentence(String sentence) {
        assertEquals(3, wordCounter.countWords(sentence));
    }
}
```

​	`@EnumSource` enables us to use `enum` instances. The annotation provides an optional `names` parameter that lets us specify which instances must be used or excluded. By default, all instances of an `enum` are used.

```java
class ParameterizedWithEnumSourceTest {
    private WordCounter wordCounter = new WordCounter();

    @ParameterizedTest
    @EnumSource(Sentences.class)
    void testWordsInSentence(Sentences sentence) {
        assertEquals(3, wordCounter.countWords(sentence.value()));
    }

    @ParameterizedTest
    @EnumSource(value = Sentences.class, names = {"JUNIT_IN_ACTION", "THREE_PARAMETERS"})
    void testSelectedWordsInSentence(Sentences sentence) {
        assertEquals(3, wordCounter.countWords(sentence.value()));
    }

    @ParameterizedTest
    @EnumSource(value = Sentences.class, mode = EXCLUDE, names = {"THREE_PARAMETERS"})
    void testExcludedWordsInSentence(Sentences sentence) {
        assertEquals(3, wordCounter.countWords(sentence.value()));
    }

    enum Sentences {
        JUNIT_IN_ACTION("JUnit in Action"),
        SOME_PARAMETERS("Check some parameters"),
        THREE_PARAMETERS("Check three parameters");

        private final String sentence;

        Sentences(String sentence) {
            this.sentence = sentence;
        }

        public String value() {
            return sentence;
        }
    }
}
```

**@CsvSource annotation**

```java
class ParameterizedWithCsvSourceTest {
    private WordCounter wordCounter = new WordCounter();

    @ParameterizedTest
    @CsvSource({"2, Unit testing", "3, JUnit in Action", "4, Write solid Java code"})
    void testWordsInSentence(int expected, String sentence) {
        assertEquals(expected, wordCounter.countWords(sentence));
    }
}
```

**@CsvFileSource annotation**

```java
class ParameterizedWithCsvFileSourceTest {
    private WordCounter wordCounter = new WordCounter();

    @ParameterizedTest
    @CsvFileSource(resources = "/word_counter.csv")
    void testWordsInSentence(int expected, String sentence) {
        assertEquals(expected, wordCounter.countWords(sentence));
    }
}
```

```
2, Unit testing
3, JUnit in Action
4, Write solid Java code
```

## Dynamic tests

​	JUnit 5 introduces a dynamic new programming model that can generate tests at runtime. We write a factory method, and at runtime, it creates a series of tests to be executed. Such a factory method must be annotated with `@TestFactory`.

​	A `@TestFactory` method is not a regular test but a factory that generates tests. A method annotated as `@TestFactory` must return one of the following:

* A `DynamicNode` (an abstract class; `DynamicContainer` and `DynamicTest` are the instantiable concrete classes)
* An array of `DynamicNode` objects
* A `Stream` of `DynamicNode` objects
* A `Collection` of `DynamicNode` objects
* An `Iterable` of `DynamicNode` objects
* An `Iterator` of `DynamicNode` objects

```java
class DynamicTestsTest {

    private PositiveNumberPredicate predicate = new PositiveNumberPredicate();

    @BeforeAll
    static void setUpClass() {
        System.out.println("@BeforeAll method");
    }

    @AfterAll
    static void tearDownClass() {
        System.out.println("@AfterAll method");
    }

    @BeforeEach
    void setUp() {
        System.out.println("@BeforeEach method");
    }

    @AfterEach
    void tearDown() {
        System.out.println("@AfterEach method");
    }

    @TestFactory
    Iterator<DynamicTest> positiveNumberPredicateTestCases() {
        return asList(
                dynamicTest("negative number", () -> assertFalse(predicate.check(-1))),
                dynamicTest("zero", () -> assertFalse(predicate.check(0))),
                dynamicTest("positive number", () -> assertTrue(predicate.check(1)))
        ).iterator();
    }
}
```

![](https://pic.imgdb.cn/item/610f6e315132923bf8f8727a.jpg)

## Using Hamcrest matchers

​	Statistics show that people easily become hooked on the unit-testing philosophy. When we get accustomed to writing tests and see how good it feels to be protected from possible mistakes, we wonder how it was possible to live without unit testing.

**Cumbersome JUnit assert method**

```java
public class HamcrestListTest {

    private List<String> values;

    @BeforeEach
    public void setUp() {
        values = new ArrayList<>();
        values.add("John");
        values.add("Michael");
        values.add("Edwin");
    }

    @Test
    @DisplayName("List without Hamcrest will intentionally fail to show how failing information is displayed")
    public void testListWithoutHamcrest() {
        assertEquals(3, values.size());
        assertTrue(values.contains("Oliver") || values.contains("Jack") || values.contains("Harry"));
    }

    @Test
    @DisplayName("List with Hamcrest will intentionally fail to show how failing information is displayed")
    public void testListWithHamcrest() {
        assertThat(values, hasSize(3));
        assertThat(values, hasItem(anyOf(equalTo("Oliver"), equalTo("Jack"),
                equalTo("Harry"))));
        assertThat("The list doesn't contain all the expected objects, in order", values, contains("Oliver", "Jack", "Harry"));
        assertThat("The list doesn't contain all the expected objects", values, containsInAnyOrder("Jack", "Harry", "Oliver"));
    }
}
```

​	To solve this problem, Tested Data Systems uses a library of matchers for building test expressions: Hamcrest. Hamcrest ([https://code.google.com/archive/p/hamcrest](https://code.google.com/archive/p/hamcrest/)) is a library that contains a lot of helpful *matcher* objects (also known as *constraints* or *predicates*) ported in several languages, such as Java, C++, Objective-C, Python, and PHP.

```java
public class HamcrestMatchersTest {

    private static String FIRST_NAME = "John";
    private static String LAST_NAME = "Smith";
    private static Customer customer = new Customer(FIRST_NAME, LAST_NAME);


    @Test
    @DisplayName("Hamcrest is, anyOf, allOf")
    public void testHamcrestIs() {
        int price1 = 1, price2 = 1, price3 = 2;

        assertThat(1, is(price1));
        assertThat(1, anyOf(is(price2), is(price3)));
        assertThat(1, allOf(is(price1), is(price2)));
    }

    @Test
    @DisplayName("Null expected")
    void testNull() {
        assertThat(null, nullValue());
    }

    @Test
    @DisplayName("Object expected")
    void testNotNull() {
        assertThat(customer, notNullValue());
    }

    @Test
    @DisplayName("Check correct customer properties")
    void checkCorrectCustomerProperties() {
        assertThat(customer, allOf(
                hasProperty("firstName", is(FIRST_NAME)),
                hasProperty("lastName", is(LAST_NAME))
        ));
    }

}
```

To use Hamcrest in our projects, we need to add the required dependency to the pom.xml file.

```java
<dependency>
    <groupId>org.hamcrest</groupId>
    <artifactId>hamcrest-library</artifactId>
    <version>2.1</version>
    <scope>test</scope>
</dependency>
```

​	To use Hamcrest in JUnit 4, you have to use the `assertThat` method from the `org.junit.Assert` class. But as explained earlier in this chapter, JUnit 5 removes the `assertThat()` method. The user guide justifies the decision this way:

**Sample of common Hamcrest static factory methods**

| Factory method                                               | Logical                                                      |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| anything                                                     | Matches to absolutely anything. Useful in some cases where you want to make the assert statement more readable. |
| is                                                           | This matcher is used only to improve the readability of your statements. |
| allOf                                                        | A matcher that checks if all contained matchers match (just like the && operator). |
| anyOf                                                        | A matcher that checks if any of the contained matchers match (like the operator \|\|). |
| not                                                          | A matcher that inverses the meaning of the contained matchers (just like the ! operator in Java). |
| instanceOf                                                   | A matchers that matches whether objects are instance of one another. |
| sameInstance                                                 | A matcher to test object identity.                           |
| nullValue , notNullValue                                     | A matcher to test for null values (or non-null values).      |
| hasProperty                                                  | A matcher to test whether a Java Bean has a certain property. |
| hasEntry, hasKey, hasValue                                   | This matcher tests whether a given Map has a given entry, key or value. |
| hasItem, hasItems                                            | This matchers tests a given collection for presence of item or items. |
| closeTo, greaterThan, greaterThanOrEqual, lessThan, lessThanOrEqual | Test given numbers are close to, greater than, greater than or equal, less than or less than or equal to a given value. |
| equalToIgnoringCase                                          | Tests a given string equals another one, ignoring the case.  |
| equalToIgnoringWhiteSpace                                    | Tests a given string equals another one, by ignoring the white spaces. |
| containsString, endsWith, startWith                          | Tests whether the given string contains, starts with or ends with a certain string. |

# JUnit architecture

* Demonstrating the concept and importance of software architecture
* Comparing the JUnit 4 and JUnit 5 architectures

## The concept and importance of software architecture

Junit5 vs Junit4

* smaller works better.
* modularity improves the work. 

## JUnit 4 architecture

**JUnit 4 modularity**

​	If a programmer wants to use JUnit 4 in a project, all they need to do is add that JAR file on the classpath.

**JUnit 4 runners**

​	A JUnit 4 *runner* is a class that extends the JUnit 4 abstract `Runner` class. A JUnit 4 runner is responsible for running JUnit tests. JUnit 4 remains a single JAR file, but it’s generally necessary to extend the functionality of this file. In other words, developers have the opportunity to add custom features to the functionality, such as executing additional things before and after running a test.

**JUnit 4 rules**

​	A JUnit 4 *rule* is a component that intercepts test method calls; it allows us to do something before a test method is run and something else after a test method has run. Rules are specific to JUnit 4.

```java
public class Calculator {
    public double add(double number1, double number2) {
        return number1 + number2;
    }

    public double sqrt(double x) {
        if (x < 0) {
            throw new IllegalArgumentException("Cannot extract the square root of a negative value");
        }
        return Math.sqrt(x);
    }

    public double divide(double x, double y) {
        if (y == 0) {
            throw new ArithmeticException("Cannot divide by zero");
        }
        return x / y;
    }
}
```

​	Listing 3.5 specifies which exception message is expected during the execution of the test code, using the new functionality of the `Calculator` class.

```java
public class RuleExceptionTester {
    @Rule
    public ExpectedException expectedException = ExpectedException.none();

    private Calculator calculator = new Calculator();

    @Test
    public void expectIllegalArgumentException() {
        expectedException.expect(IllegalArgumentException.class);
        expectedException.expectMessage("Cannot extract the square root of a negative value");
        calculator.sqrt(-1);
    }

    @Test
    public void expectArithmeticException() {
        expectedException.expect(ArithmeticException.class);
        expectedException.expectMessage("Cannot divide by zero");
        calculator.divide(1, 0);
    }
}
```

* We declare an `ExpectedException` field annotated with `@Rule`. The `@Rule` annotation must be applied to a public nonstatic field or a public nonstatic method (#A). The `ExpectedException.none()` factory method simply creates an unconfigured `ExpectedException`.
* We initialize an instance of the `Calculator` class whose functionality we are testing (#B).
* The `ExpectedException` is configured to keep the type of exception (#C) and message (#D) before it is thrown by invoking the `sqrt` method (#E).
* he `ExpectedException` is configured to keep the type of exception (#F) and message #G before it is thrown by invoking the `divide` method (#H).

**The RuleTester class**

```
public class RuleTester {
    @Rule
    public TemporaryFolder folder = new TemporaryFolder();
    private static File createdFolder;
    private static File createdFile;

    @Test
    public void testTemporaryFolder() throws IOException {
        createdFolder = folder.newFolder("createdFolder");
        createdFile = folder.newFile("createdFile.txt");
        assertTrue(createdFolder.exists());
        assertTrue(createdFile.exists());
    }

    @AfterClass
    public static void cleanUpAfterAllTestsRan() {
        assertFalse(createdFolder.exists());
        assertFalse(createdFile.exists());
    }
}
```

* We declare a `TemporaryFolder` field annotated with `@Rule` and initialize it. The `@Rule` annotation must be applied to a public field or a public method (#A).
*  We declare the static fields `createdFolder` and `createdFile` #B.
* We use the `TemporaryFolder` field to create a folder and a file (#C), which are located in the `Temp` folder of our user profile in the operating system.
* We check the existence of the temporary folder and of the temporary file (#D).
* At the end of the execution of the tests, we check that the temporary resources do not exist any longer (#E).



​	To write our own custom rule, we’ll have to create a class that implements the `TestRule` interface. Consequently, we’ll override the `apply(Statement, Description)` method, which must return an instance of `Statement`. Such an object represents the tests within the JUnit run time, and `Statement#evaluate()` runs them. The `Description` object describes the individual test; we can use this object to read information about the test through reflection.

```java
public class CustomRule implements TestRule {
    private Statement base;
    private Description description;

    @Override
    public Statement apply(Statement base, Description description) {
        this.base = base;
        this.description = description;
        return new CustomStatement(base, description);
    }
}
```

*  We declare a `CustomRule` class that implements the `TestRule` interface (#A).
* We keep references to a `Statement` field and to a `Description` field (#B), and we use them into the `apply` method that returns a `CustomStatement` (#C).



```java
public class CustomStatement extends Statement {
    private Statement base;
    private Description description;

    public CustomStatement(Statement base, Description description) {
        this.base = base;
        this.description = description;
    }

    @Override
    public void evaluate() throws Throwable {
        System.out.println(this.getClass().getSimpleName() + " " + description.getMethodName() + " has started");
        try {
            base.evaluate();
        } finally {
            System.out.println(this.getClass().getSimpleName() + " " + description.getMethodName() + " has finished");
        }
    }
}
```

* We declare our `CustomStatement` class that extends the `Statement` class (#A).
* We keep references to a `Statement` field and a `Description` field (#B), and we use them as arguments of the constructor (#C).
* We override the inherited `evaluate` method and call `base.evaluate()` inside it (#D).

```java
public class CustomRuleTester {

    @Rule
    public CustomRule myRule = new CustomRule();

    @Test
    public void myCustomRuleTest() {
        System.out.println("Call of a test method");
    }
}
```

* We declare a public `CustomRule` field and annotate it with `@Rule` (#A).
* We create the `myCustomRuleTest` method and annotate it with `@Test` (#B).

**CustomRuleTester2**

```java
public class CustomRuleTester2 {

    private CustomRule myRule = new CustomRule();

    @Rule
    public CustomRule getMyRule() {
        return myRule;
    }

    @Test
    public void myCustomRuleTest() {
        System.out.println("Call of a test method");
    }
}
```

```xml
<dependencies>
	<dependency>
		<groupId>org.junit.vintage</groupId>
		<artifactId>junit-vintage-engine</artifactId>
		<version>5.6.0</version>
		<scope>provided</scope>
	</dependency>
</dependencies>
```

**Shortcomings of the JUnit 4 architecture**

* The API provided by JUnit 4 was not flexible enough.

## JUnit 5 architecture

**JUnit 5 modularity**

​	The architecture had to allow JUnit to interact with different programmatic clients that used different tools and IDEs. The logical separation of concerns required

* An API to write tests, dedicated mainly to developers
* A mechanism for discovering and running the tests
* An API to allow easy interaction with IDEs and tools and to run the tests from them



​	As a consequence, the JUnit 5 architecture contained three modules (figure 3.7):

* *JUnit Platform*, which serves as a foundation for launching testing frameworks on the Java Virtual Machine (JVM). It also provides an API to launch tests from the console, IDEs, or build tools.
* *JUnit Jupiter*, the combination of the new programming and extension model for writing tests and extensions in JUnit 5. The name comes from the fifth planet of our solar system, which is also the largest.
* *JUnit Vintage*, a test engine for running JUnit 3- and JUnit 4-based tests on the platform, ensuring backward compatibility.

![](https://pic.imgdb.cn/item/6110f8765132923bf8123b0e.jpg)

**JUnit 5 platform**

* `junit-platform-commons`, an internal common library of JUnit intended solely for use within the JUnit framework. Any use by external parties is not supported.
* `junit-platform-console`, which provides support for discovering and executing tests on the JUnit Platform from the console.
* `junit-platform-console-standalone`, an executable JAR with all dependencies included. This artifact is used by Console Launcher, a command-line Java application that lets us launch the JUnit Platform from the console. It can be used to run JUnit Vintage and JUnit Jupiter tests, for example, and to print test execution results to the console.
* `junit-platform-engine`, a public API for test engines.
* `junit-platform-launcher`, a public API for configuring and launching test plans; typically used by IDEs and build tools.
* `junit-platform-runner`, a runner for executing tests and test suites on the JUnit Platform in a JUnit 4 environment.
* `junit-platform-suite-api`, which contains the annotations for configuring test suites on the JUnit Platform.
* `junit-platform-surefire-`provider, which provides support for discovering and executing tests on the JUnit Platform by using Maven Surefire.
* `junit-platform-gradle-plugin`, which provides support for discovering and executing tests on the JUnit Platform by using Gradle.



**JUnit 5 Jupiter**

​	JUnit Jupiter is the combination of the new programming (annotations, classes, and methods) and extension model for writing tests and extensions in JUnit 5. 

* `junit-jupiter-api`, the JUnit Jupiter API for writing tests and extensions
* `junit-jupiter-engine`, the JUnit Jupiter test engine implementation, required only at run time
* `junit-jupiter-params`, which provides support for parameterized tests in JUnit Jupiter
* `junit-jupiter-migrationsupport`, which provides migration support from JUnit 4 to JUnit Jupiter and is required only for running selected JUnit 4 rules

**JUnit 5 Vintage**

​	JUnit Vintage provides a `TestEngine` for running JUnit 3- and JUnit 4-based tests on the platform. JUnit 5 Vintage contains only`junit-vintage-engine`, the engine implementation to execute tests written in JUnit 3 or 4. For this purpose, of course, we also need the JUnit 3 or 4 JARs.

**The big picture of the JUnit 5 architecture**

![](https://pic.imgdb.cn/item/6110f9b85132923bf814edb8.jpg)

​	Detailing at the level of the jar files (figure 3.9):

* The test APIs provide the facilities for different test engines: `junit-jupiter-api` for JUnit 5 tests, `junit-4.12` for legacy tests, and custom engines for third-party tests.
*  The test engines mentioned earlier are created by extending the `junit-platform-engine` public API, which is part of the JUnit 5 Platform.
* The `junit-platform-launcher` public API provides the facilities to discover tests inside the JUnit 5 Platform for build tools such as Maven or Gradle or for IDEs.

![](https://pic.imgdb.cn/item/6110fa1a5132923bf815c884.jpg)





# Migrating from JUnit 4 to JUnit 5

* Implementing the migration from JUnit 4 to JUnit 5
* Working with a hybrid approach for mature projects
* Comparing the needed JUnit 4 and JUnit 5 dependencies
* Comparing the equivalent JUnit 4 and JUnit 5 annotations
* Comparing the JUnit 4 rules and JUnit 5 extensions

## The steps between JUnit 4 and JUnit 5

​	All classes and annotations specific to JUnit Jupiter are located in the new `org.junit.jupiter` base package. All classes and annotations specific to JUnit 4 are located in the old`org.junit` base package.

​	So, if both JUnit 4 and JUnit 5 Jupiter are in the classpath does not result in any conflict. 

​	Before developing and running the JUnit tests, programmers must have the following programs:

* JUnit 4 requires Java 5 or later.
* JUnit 5 requires Java 8 or later.



​	Consequently, migrating from JUnit 4 to JUnit 5 may require an update of the Java version in use inside the project.

​	Table 4.1 summarizes the most important steps in migrating from JUnit 4 to JUnit 5.

**Table 4.1 Migrating from JUnit 4 and JUnit 5**

| Main step                                                    | Comments                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Replace the needed dependencies                              | JUnit 4 needs a single dependency. JUnit 5 requires more dependencies, related to the features that are used. JUnit 5 uses JUnit Vintage to work with old JUnit 4 tests. |
| Replace the annotations and introduce the new ones           | Some JUnit 5 annotations mirror the old JUnit 4 ones. Some new ones introduce new facilities and help developers write better tests. |
| Replace the testing classes and methods                      | JUnit 5 assertions and assumptions have been moved to different classes from different packages. |
| Replace the JUnit 4 rules and the runners with the JUnit 5 extension model | This step generally requires more effort than the other steps in this table. Because JUnit 4 and JUnit 5 may coexist for a long period, however, the rules and runners may remain in the code or be replaced much later. |

## Needed dependencies

**The JUnit 4 Maven dependency**

```xml
<dependencies> 
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

​	The first dependency is `junit-vintage-engine` (listing 4.2). It belongs to JUnit 5 but ensures backward compatibility with previous versions of JUnit. 

```xml
<dependencies>
    <dependency>
        <groupId>org.junit.vintage</groupId>
        <artifactId>junit-vintage-engine</artifactId>
        <version>5.6.0</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

​	Running the JUnit 4 tests right now, we may notice that they are successfully executed (figure 4.1). Working with the JUnit 5 Vintage dependency instead of the old JUnit 4 one will not make any difference.

![](https://pic.imgdb.cn/item/611c9ed24907e2d39c6eab6c.jpg)

​	After the company’s programmers introduce the JUnit Vintage dependency, the migration path of Tested Data Systems’s projects may continue with the introduction of JUnit 5 Jupiter annotations and features. The required dependencies are shown in listing 4.3.

**The most useful JUnit Jupiter Maven dependencies**

```xml
<dependencies>
    <dependency>
       <groupId>org.junit.jupiter</groupId>
       <artifactId>junit-jupiter-api</artifactId>
       <version>5.6.0</version>
       <scope>test</scope>
    </dependency>
    <dependency>
       <groupId>org.junit.jupiter</groupId>
       <artifactId>junit-jupiter-engine</artifactId>
       <version>5.6.0</version>
       <scope>test</scope>
    </dependency>
</dependencies>
```

​	To write tests using JUnit 5, you will always need the `junit-jupiter-api` and the `junit-jupiter-engine` dependencies. The first one represents the API for writing tests with JUnit Jupiter (including the annotations, classes, and methods to be migrated to). The second one represents the core JUnit Jupiter package for the execution test engine.

## Annotations, classes, and methods

**Table 4.2 Annotations**

| JUnit 4                   | JUnit 5                 |
| ------------------------- | ----------------------- |
| @BeforeClass, @AfterClass | @BeforeAll, @AfterAll   |
| @Before, @After           | @BeforeEach, @AfterEach |
| @Ignore                   | @Disable                |
| @Category                 | @Tag                    |

**Table 4.3 Assertions**

| JUnit 4                                           | JUnit 5                                                      |
| ------------------------------------------------- | ------------------------------------------------------------ |
| Assert class                                      | Assertions class                                             |
| Optional assertion message is the first parameter | Optional assertion message is the last parameter             |
| assertThat method                                 | assertThat method removed. New methods: assertAll and assertThrows |

**Table 4.4 Assumptions**

| JUnit 4                             | JUnit 5                                     |
| ----------------------------------- | ------------------------------------------- |
| Assume class                        | Assumptions class                           |
| assumeNotNull and assumeNoException | assumeNotNull and assumeNoException removed |

​	We start with a class that simulates a system under test (SUT). This one can be initialized, can receive usual but cannot receive additional work to execute, and may close itself. Listing 4.4 presents the SUT class.

```java
public class SUT {
    private String systemName;

    public SUT(String systemName) {
        this.systemName = systemName;
        System.out.println(systemName + " from class " + getClass().getSimpleName() + " is initializing.");
    }

    public boolean canReceiveRegularWork() {
        System.out.println(systemName + " from class " + getClass().getSimpleName() + " can receive regular work.");
        return true;
    }

    public boolean canReceiveAdditionalWork() {
        System.out.println(systemName + " from class " + getClass().getSimpleName() + " cannot receive additional work.");
        return false;
    }

    public void close() {
        System.out.println(systemName + " from class " + getClass().getSimpleName() + " is closing.");
    }
}
```



```java
public class JUnit4SUTTest {

    private static ResourceForAllTests resourceForAllTests;
    private SUT systemUnderTest;

    @BeforeClass
    public static void setUpClass() {
        resourceForAllTests = new ResourceForAllTests("Our resource for all tests");
    }

    @AfterClass
    public static void tearDownClass() {
        resourceForAllTests.close();
    }

    @Before
    public void setUp() {
        systemUnderTest = new SUT("Our system under test");
    }

    @After
    public void tearDown() {
        systemUnderTest.close();
    }

    @Test
    public void testRegularWork() {
        boolean canReceiveRegularWork = systemUnderTest.canReceiveRegularWork();

        assertTrue(canReceiveRegularWork);
    }

    @Test
    public void testAdditionalWork() {
        boolean canReceiveAdditionalWork = systemUnderTest.canReceiveAdditionalWork();

        assertFalse(canReceiveAdditionalWork);
    }

    @Test
    @Ignore
    public void myThirdTest() {
        assertEquals("2 is not equal to 1", 2, 1);
    }
}
```

​	We previously replaced the JUnit 4 dependency with the JUnit Vintage one. The result of running the JUnit4SUTTest class is the same in both cases (figure 4.2), `mySecondTest` being marked with the `@Ignore` annotation. Now we can proceed to the effective migration of annotations, classes, and methods.

![](https://pic.imgdb.cn/item/611ca4244907e2d39c99bb5a.jpg)

**JUnit5SUTTest class**

```java
class JUnit5SUTTest {
    private static ResourceForAllTests resourceForAllTests;
    private SUT systemUnderTest;

    @BeforeAll
    static void setUpClass() {
        resourceForAllTests = new ResourceForAllTests("Our resource for all tests");
    }

    @AfterAll
    static void tearDownClass() {
        resourceForAllTests.close();
    }

    @BeforeEach
    void setUp() {
        systemUnderTest = new SUT("Our system under test");
    }

    @AfterEach
    void tearDown() {
        systemUnderTest.close();
    }

    @Test
    void testRegularWork() {
        boolean canReceiveRegularWork = systemUnderTest.canReceiveRegularWork();

        assertTrue(canReceiveRegularWork);
    }

    @Test
    void testAdditionalWork() {
        boolean canReceiveAdditionalWork = systemUnderTest.canReceiveAdditionalWork();

        assertFalse(canReceiveAdditionalWork);
    }

    @Test
    @Disabled
    void myThirdTest() {
        assertEquals(2, 1, "2 is not equal to 1");
    }
}
```

Comparing the JUnit 4 and JUnit 5 methods, we see that

* The methods annotated with `@BeforeClass` (#A in listing 4.5) and `@BeforeAll` (#A' in listing 4.6), respectively, are executed once, before all tests. These methods need to be static. In the JUnit 4 version, the method also needs to be public. In the JUnit 5 version, we can make the method nonstatic and annotate the whole test class with`@TestInstance(Life cycle.PER_CLASS)`.
* The methods annotated with `@AfterClass` (#B in listing 4.5) and `@AfterAll` (#B' in listing 4.6), respectively, are executed once, after all tests. These methods need to be static. In the JUnit 4 version, the method also needs to be public. In the JUnit 5 version, we can make the method nonstatic and annotate the whole test class with `@TestInstance(Life cycle.PER_CLASS)`.
* The methods annotated with `@Before` (#C in listing 4.5) and `@BeforeEach` (#C' in listing 4.6), respectively, are executed before each test. In the JUnit 4 version, the methods need to be public.
* The methods annotated with `@`After (#D in listing 4.5) and `@AfterEach` (#D' in listing 4.6), respectively, are executed after each test. In the JUnit 4 version, the methods need to be public.
* The methods annotated with `@Test` (#E in listing 4.5) and `@Test` (#E' in listing 4.6) are executed independently. In the JUnit 4 version, the methods need to be public. The two annotations belong to different packages: `org.junit.Test` and`org.junit.jupiter.api.Test`, respectively.
* To skip the execution of a test method, JUnit 4 uses the annotation `@Ignore (#F` in listing 4.5)`,` whereas JUnit 5 uses the annotation `@Disabled (`#F' in listing 4.6).



​	The access level has been relaxed for the test methods, from public to package-private. These methods are accessed only from within the package to which the test class belongs to, so they didn’t need to be made public.

​	Tested Data Systems needs to verify its customers’ information, as well as their existence or nonexistence. It wants to classify the verification tests in two groups: the ones that work with individual customers and the ones that check inside a repository. The company has used categories (in JUnit 4) and needs to switch to tags (in JUnit 5).

**The interfaces created to define categories with JUnit 4**

```java
public interface IndividualTests {
}
public interface RepositoryTests {
}
```

​	Listing 4.8 defines a JUnit 4 test that contains a method annotated as `@Category(IndividualTests.`class`).` This annotation assigns that test method as belonging to this category.

```java
public class JUnit4CustomerTest {
    private String CUSTOMER_NAME = "John Smith";

    @Category(IndividualTests.class)
    @Test
    public void testCustomer() {
        Customer customer = new Customer(CUSTOMER_NAME);

        assertEquals("John Smith", customer.getName());
    }
}
```

​	Listing 4.9 defines a JUnit 4 test class annotated as `@Category(IndividualTests.`class`, RepositoryTests.`class`).` This annotation assigns the two containing test methods as belonging to these two categories.

```java
@Category({IndividualTests.class, RepositoryTests.class})
public class JUnit4CustomersRepositoryTest {
    private String CUSTOMER_NAME = "John Smith";
    private CustomersRepository repository = new CustomersRepository();

    @Test
    public void testNonExistence() {
        boolean exists = repository.contains(CUSTOMER_NAME);

        assertFalse(exists);
    }

    @Test
    public void testCustomerPersistence() {
        repository.persist(new Customer(CUSTOMER_NAME));

        assertTrue(repository.contains("John Smith"));
    }
}
```

**The JUnit4IndividualTestsSuite class**

```java
@RunWith(Categories.class)
@Categories.IncludeCategory(IndividualTests.class)
@Suite.SuiteClasses({JUnit4CustomerTest.class, JUnit4CustomersRepositoryTest.class})
public class JUnit4IndividualTestsSuite {
}
```

​	In this example, the `JUnit4IndividualTestsSuite:`

* Is annotated with `@RunWith(Categories.`class`)` (#A), informing JUnit that it has to execute the tests with this particular runner.
* Includes the category of tests annotated with `IndividualTests` (#B).
* Looks for these annotated tests in the `JUnit4CustomerTest` and `JUnit4CustomersRepositoryTest` classes (#C).



​	The result of running this suite is shown in figure 4.3. All tests from the `JUnit4CustomerTest` and`JUnit4CustomersRepositoryTest` classes will be executed, as all of them are annotated with `IndividualTests`.

![](https://pic.imgdb.cn/item/611ca8c14907e2d39cbde971.jpg)

**The JUnit4RepositoryTestsSuite class**

```java
@RunWith(Categories.class)
@Categories.IncludeCategory(RepositoryTests.class)
@Suite.SuiteClasses({JUnit4CustomerTest.class, JUnit4CustomersRepositoryTest.class})
public class JUnit4RepositoryTestsSuite {
}
```

![](https://pic.imgdb.cn/item/611ca9fa4907e2d39cc6d78e.jpg)

**The JUnit4ExcludeRepositoryTestsSuite class**

```java
@RunWith(Categories.class)
@Categories.ExcludeCategory(RepositoryTests.class)
@Suite.SuiteClasses({JUnit4CustomerTest.class, JUnit4CustomersRepositoryTest.class})
public class JUnit4ExcludeRepositoryTestsSuite {
}
```

![](https://pic.imgdb.cn/item/611caa494907e2d39cc91536.jpg)

**The JUnit5CustomerTest tagged class**

```java
@Tag("individual")
public class JUnit5CustomerTest {
    private String CUSTOMER_NAME = "John Smith";

    @Test
    void testCustomer() {
        Customer customer = new Customer(CUSTOMER_NAME);

        assertEquals("John Smith", customer.getName());
    }
}
```

​	The `@Tag` annotation is added to the whole `JUnit5CustomerTest` class (#A).

**The JUnit5CustomerRepositoryTest tagged class**

```java
@Tag("repository")
public class JUnit5CustomersRepositoryTest {
    private String CUSTOMER_NAME = "John Smith";
    private CustomersRepository repository = new CustomersRepository();

    @Test
    void testNonExistence() {
        boolean exists = repository.contains("John Smith");

        assertFalse(exists);
    }

    @Test
    void testCustomerPersistence() {
        repository.persist(new Customer(CUSTOMER_NAME));

        assertTrue(repository.contains("John Smith"));
    }
}
```

​	Similarly, the `@Tag` annotation is added to the whole `JUnit5CustomerRepositoryTest` class (#A).

​	To active the JUnit 5 tags that replace the JUnit 4 categories, we have a few alternatives. For one, we can work at the level of the pom.xml configuration file. In listing 4.15, we can uncomment the configuration node of the Surefire plugin (#A) and run `mvn clean install`.

```java
<plugin>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.22.2</version>

    <configuration>
        <groups>individual</groups>
        <excludedGroups>repository</excludedGroups>
    </configuration>

</plugin>
```

​	In order to activate the usage of the JUnit 5 tags that replace the JUnit 4 categories, you have a few alternatives. You can work at the level of the pom.xml configuration file. In listing 4.15, it will be enough to uncomment the configuration node of the surefire plugin #A and run mvn clean install.

​	Or, from the IntelliJ IDEA IDE, you can activate the usage of the tags by going to Run -> Edit Configurations and choose Tags (JUnit 5) as test kind (figure 4.6). However, the recommended way to go is to change the pom.xml, so that the tests can be correctly executed from the command line

![](https://pic.imgdb.cn/item/611cacdd4907e2d39cdb4b1b.jpg)



​	To continue the comparison of JUnit 4 and JUnit 5, we look at the Hamcrest matchers functionality, introduced in chapter 2. In this chapter, we use our collections example to put the two versions face to face. We ill populate a list with values from Tested Data Systems’s internal information and then investigate whether its elements match some patterns, using JUnit 4 (listing 4.16) and JUnit 5 (listing 4.17).

**The JUnit4HamcrestListTest class**

```java
public class JUnit4HamcrestListTest {

    private List<String> values;

    @Before
    public void setUp() {
        values = new ArrayList<>();
        values.add("Oliver");
        values.add("Jack");
        values.add("Harry");
    }

    @Test
    public void testListWithHamcrest() {
        assertThat(values, hasSize(3));
        assertThat(values, hasItem(anyOf(equalTo("Oliver"), equalTo("Jack"),
                equalTo("Harry"))));
        assertThat("The list doesn't contain all the expected objects, in order", values, contains("Oliver", "Jack", "Harry"));
        assertThat("The list doesn't contain all the expected objects", values, containsInAnyOrder("Jack", "Harry", "Oliver"));
    }
}
```

**The JUnit5HamcrestListTest class**

```java
public class JUnit5HamcrestListTest {

    private List<String> values;

    @BeforeEach
    public void setUp() {
        values = new ArrayList<>();
        values.add("Oliver");
        values.add("Jack");
        values.add("Harry");
    }

    @Test
    @DisplayName("List with Hamcrest")
    public void testListWithHamcrest() {
        assertThat(values, hasSize(3));
        assertThat(values, hasItem(anyOf(equalTo("Oliver"), equalTo("Jack"),
                equalTo("Harry"))));
        assertThat("The list doesn't contain all the expected objects, in order", values, contains("Oliver", "Jack", "Harry"));
        assertThat("The list doesn't contain all the expected objects", values, containsInAnyOrder("Jack", "Harry", "Oliver"));
    }
}
```

These examples are very similar (except for the `@Before`/`@BeforeEach` and `@DisplayName` annotations). The old imports are gradually replaced by the new ones, which is why the annotations `org.junit.Test` and `org.junit.jupiter.api.Test`belong to different packages.

What do these programs do with the internal information managed at Tested Data Systems?

* We initialize the list to work with. The code is the same, but the annotations are different: `@Before` (#A) and `@BeforeEach`(#A').
* The test methods are annotated with `org.junit.Test` (#B) and `org.junit.jupiter.api.Test` (#B’), respectively.
* Verification (#C) uses the `org.junit.Assert.assertThat` method. JUnit 5 removes this method, so we use`org.hamcrest.MatcherAssert.assertThat` (#C’).
* We use the `anyOf` and `equalTo` methods from the `org.hamcrest.Matchers` class (D) and the `anyOf` and `equalTo`methods from the `org.hamcrest.CoreMatchers` class (#D’).
* We use the same `org.hamcrest.Matchers.contains` method (#E and #E’).
* We use the same `org.hamcrest.Matchers.containsInAnyOrder` method (#F and #F’).

### Rules vs. the extension model

​	A JUnit4 rule is a component that allows introducing additional actions when a method is executed, by intercepting its call and do something before and after the execution of the method.

​	One rule that the Tested Data Systems test code uses extensively is`ExpectedException`, which can easily be replaced by the JUnit 5 `assertThrows` method.

**The extended Calculator class**

```java
public class Calculator {
    public double add(double number1, double number2) {
        return number1 + number2;
    }

    public double sqrt(double x) {
        if (x < 0) {
            throw new IllegalArgumentException("Cannot extract the square root of a negative value");
        }
        return Math.sqrt(x);
    }

    public double divide(double x, double y) {
        if (y == 0) {
            throw new ArithmeticException("Cannot divide by zero");
        }
        return x / y;
    }
}
```

**The JUnit4RuleExceptionTester class**

```java
public class JUnit4RuleExceptionTester {
    @Rule
    public ExpectedException expectedException = ExpectedException.none();

    private Calculator calculator = new Calculator();

    @Test
    public void expectIllegalArgumentException() {
        expectedException.expect(IllegalArgumentException.class);
        expectedException.expectMessage("Cannot extract the square root of a negative value");
        calculator.sqrt(-1);
    }

    @Test
    public void expectArithmeticException() {
        expectedException.expect(ArithmeticException.class);
        expectedException.expectMessage("Cannot divide by zero");
        calculator.divide(1, 0);
    }
}
```

**The JUnit5ExceptionTester class**

```java
public class JUnit5ExceptionTester {
    private Calculator calculator = new Calculator();

    @Test
    public void expectIllegalArgumentException() {
        Throwable throwable = assertThrows(IllegalArgumentException.class, () -> calculator.sqrt(-1));
        assertEquals("Cannot extract the square root of a negative value", throwable.getMessage());
    }

    @Test
    public void expectArithmeticException() {
        Throwable throwable = assertThrows(ArithmeticException.class, () -> calculator.divide(1, 0));
        assertEquals("Cannot divide by zero", throwable.getMessage());
    }
}
```



​	Another rule that Tested Data Systems would like to migrate is `TemporaryFolder`. The `TemporaryFolder` rule allows the creation of files and folders that should be deleted when the test method finishes (whether it passes or fails). As the tests of the Tested Data Systems projects work intensively with temporary resources, this step is also a must. The JUnit 4 rule has been replaced by the `@TempDi`r annotation in JUnit 5. Listing 4.21 presents the JUnit 4 approach.

**The JUnit4RuleTester class**

```java
public class JUnit4RuleTester {
    @Rule
    public TemporaryFolder folder = new TemporaryFolder();

    @Test
    public void testTemporaryFolder() throws IOException {
        File createdFolder = folder.newFolder("createdFolder");
        File createdFile = folder.newFile("createdFile.txt");
        assertTrue(createdFolder.exists());
        assertTrue(createdFile.exists());
    }
}
```

**The JUnit5TempDirTester class**

```java
public class JUnit5TempDirTester {
    @TempDir
    Path tempDir;

    private static Path createdFile;

    @Test
    public void testTemporaryFolder() throws IOException {
        assertTrue(Files.isDirectory(tempDir));
        createdFile = Files.createFile(tempDir.resolve("createdFile.txt"));
        assertTrue(createdFile.toFile().exists());
    }

    @AfterAll
    public static void afterAll() {
        assertFalse(createdFile.toFile().exists());
    }
}
```

### Custom rules

​	In JUnit 4, the Tested Data Systems engineers needed their additional actions to be executed before and after the execution of a test. Consequently, they created their own classes that implement the `TestRule` interface. To do this, they had to override the `apply(Statement, Description)` method, which returns an instance of `Statement`. Such an object represents the tests within the JUnit run time, and `Statement#evaluate()` will run them. The `Description` object describes the individual test. This object can be used to read information about the test through reflection.

```java
public class CustomRule implements TestRule {
    private Statement base;
    private Description description;

    @Override
    public Statement apply(Statement base, Description description) {
        this.base = base;
        this.description = description;
        return new CustomStatement(base, description);
    }
}
```

```java
public class CustomStatement extends Statement {
    private Statement base;
    private Description description;

    public CustomStatement(Statement base, Description description) {
        this.base = base;
        this.description = description;
    }

    @Override
    public void evaluate() throws Throwable {
        System.out.println(this.getClass().getSimpleName() + " " + description.getMethodName() + " has started");
        try {
            base.evaluate();
        } finally {
            System.out.println(this.getClass().getSimpleName() + " " + description.getMethodName() + " has finished");
        }
    }
}
```

**The JUnit4CustomRuleTester class**

```java
public class JUnit4CustomRuleTester {

    @Rule
    public CustomRule myRule = new CustomRule();

    @Test
    public void myCustomRuleTest() {
        System.out.println("Call of a test method");
    }
}
```

​	The result of the execution of this test is shown in figure 4.7. As the engineers from Tested Data Systems required, the effective execution of the test is surrounded by the additional messages provided to the `evaluate` method of the `CustomStatement`class.

![](https://pic.imgdb.cn/item/611cb4764907e2d39c06cb2c.jpg)



​	The engineers from Tested Data Systems would like to migrate their own rules as well. JUnit 5 allows similar effects, as in the case of the JUnit 4 rules, by introducing custom extensions, that will extend the behavior of test classes and methods. The code will be shorter and will rely on the declarative annotations style. First, the engineers define the `CustomExtension` class, which will be used as an argument of the `@ExtendWith` annotation on the tested class.

```java
public class CustomExtension implements AfterEachCallback, BeforeEachCallback {
    @Override
    public void beforeEach(ExtensionContext extensionContext) throws Exception {
        System.out.println(this.getClass().getSimpleName() + " " + extensionContext.getDisplayName() + " has started");
    }

    @Override
    public void afterEach(ExtensionContext extensionContext) throws Exception {
        System.out.println(this.getClass().getSimpleName() + " " + extensionContext.getDisplayName() + " has finished");
    }
}
```

```java
@ExtendWith(CustomExtension.class)
public class JUnit5CustomExtensionTester {

    @Test
    public void myCustomRuleTest() {
        System.out.println("Call of a test method");
    }
}
```

![](https://pic.imgdb.cn/item/611cb51d4907e2d39c09cabf.jpg)

The JUnit 5 extension model may also be used to replace the runners from JUnit 4 gradually. For the extensions that have already been created, the migration process is simple:

* To migrate the Mockito tests, we need to replace in the tested class the annotation`@RunWith(MockitoJUnitRunner.class)` with the annotation `@ExtendWith(MockitoExtension.class)`.
* To migrate the Spring tests, we need to replace in the tested class the annotation`@RunWith(SpringJUnit4ClassRunner.class)` with the annotation `@ExtendWith(SpringExtension.class)`.

# Software Testing Principles

* Examining the need for unit tests
* Differentiating between types of software tests
* Comparing black-box and white-box testing

## The need for unit tests

​	The main goal of unit testing is to verify that your application works as expected and to catch bugs early. Although functional testing helps you accomplish the same goal, unit tests are extremely powerful and versatile and offer much more than simply verifying that the application works. Unit tests

* Allow greater test coverage than functional tests
* Increase team productivity
* Detect regressions and limit the need for debugging
* Give us the confidence to refactor, and, in general, make changes
* Improve implementation
* Document expected behavior
* Enable code coverage and other metrics

### Allowing greater test-coverage

​	Unit tests are the first type of tests any application should have. If you had to choose between writing unit tests and functional tests, you should choose to write the second kind. In my experience, functional tests cover about 70 percent of the application code. If you want to go further and provide more test coverage, you need to write unit tests.

​	Unit tests can easily simulate error conditions, which is extremely difficult for functional tests to do (and impossible in some instances). Unit tests provide much more than just testing, as explained in the following sections.

### Increasing team productivity

​	Imagine that you are on a team working on a large application. Unit tests allow you to deliver quality code (tested code) without waiting for all the other components to be ready. On the other hand, functional tests are more coarse-grained and need the full application, or a good part of it, to be ready before you can test it.

### Detecting regressions and limiting debugging

​	A passing unit test suite confirms that your code works and gives you the confidence to modify your existing code, either for refactoring or for adding and modifying new features. As a developer, you can have = no better feeling than knowing someone is watching your back and will warn you if you break something.

​	A suite of unit tests reduces the need to debug an application to find out why something is failing. Whereas a functional test tells you that a bug exists somewhere in the implementation of a use case, a unit test tells you that a specific method is failing for a specific reason. You no longer need to spend hours trying to find the problem.

### Refactoring with confidence

​	Without unit tests, it is difficult to justify refactoring, because there is always a relatively high chance that you will break something. Why would you risk spending hours of debugging time (and putting delivery at risk) only to improve the implementation or change a method name? As shown in figure 5.1, unit tests provide the safety net that gives you the confidence to refactor.

![](https://pic.imgdb.cn/item/611f59e74907e2d39c501156.jpg)

> **JUnit best practice: Refactor**
>
> ​	When you design and write code for a single use case or functional chain, your design may be adequate for this feature, but it may not be adequate for the next feature. To retain a design across features, agile methodologies encourage refactoring to adapt the code base as needed.
>
> ​	How do you ensure that *refactoring* (improving the design of existing code) does not break the existing code? The answer is that unit tests tell you when and where code breaks. In short, unit tests give you the confidence to refactor.

### Improving implementation

​	Unit tests are first-rate clients of the code they test. They force the API under test to be flexible and to be unit-testable in isolation. Sometimes, you have to refactor your code under test to make it unit-testable. You may eventually use the Test Driven Development (TDD) approach, which by its own conception generates code that can be unit-tested. We’ll experiment with TDD in detail later in the book (chapter 20).

​	It is important to monitor your unit tests as you create and modify them. If a unit test is too long and unwieldy, the code under test usually has a design smell, and you should refactor it. You may also be testing too many features in one test method. If a test cannot verify a feature in isolation, the code probably is not flexible enough, and you should refactor it. Modifying code to test it is normal.

### Documenting expected behavior

​	Imagine that you need to learn a new API. On one hand, you have a 300-page document describing the API, and on the other hand, you have some examples of how to use the API. Which would you choose?

​	The power of examples is well known. Unit tests are examples that show how to use the API. As such, they make excellent developer documentation. Because unit tests match the production code, they *must* be up to date, unlike other forms of documentation.

### Enabling code coverage and other metrics

​	Unit tests tell you, at the push of a button, whether everything still works. Furthermore, unit tests enable you to gather code coverage metrics (see chapter 6) that show, statement by statement, what code execution the tests triggered and what code the tests did not touch. You can also use tools to track the progress of passing versus failing tests from one build to the next. Further, you can monitor performance and cause a test to fail if its performance has degraded from a previous build.

## Test types

![](https://pic.imgdb.cn/item/611f60144907e2d39c5cc6ce.jpg)

* Unit tests
* Integration tests
* System tests
* Acceptance tests

###  Unit testing

​	Unit testing is a software testing method in which individual units of source code (methods or classes) are tested to determine whether they are fit for use. Unit testing increases developer confidence in changing the code because from the beginning, it serves as a safety net. If we have good unit tests and run them every time we change the code, we will be certain that our changes are not affecting the existing functionality.

###  Integration software testing

​	Individual unit tests are essential quality controls, but what happens when different units of work are combined into a workflow? When you have the tests for a class up and running, the next step is hooking up the class with other methods and services. Examining the interaction among components, possibly running in their target environment, is the job of integration testing. Table 5.1 differentiates the various cases under which components interact.

| Interaction | Test Description                                             |
| ----------- | ------------------------------------------------------------ |
| Objects     | The test instantiates objects and calls methods on these objects. You may use this test type when you would like to see how objects belonging to different classes cooperate to solve the problem. |
| Services    | The test runs while a container hosts the application, which may connect to a database or attach to any other external resource or device. You may use this test type when you are developing an application that is deployed into a software container. |
| Subsystems  | A layered application may have a frontend to handle the presentation and a backend to execute the business logic. Tests can verify that a request passes through the frontend and returns an appropriate response from the backend. You may use this test type in the case of an application with an architecture made of a presentation layer (web interface, for example) and a business service layer executing the logic. |

​	Just as more traffic collisions occur at intersections, the points where objects interact are major contributors to bugs. Ideally, you should define integration tests before you write the application code. Being able to code to the test strongly increases a programmer’s ability to write well-behaved objects. The engineers of Tested Data Systems will use integration testing to check whether the objects representing customers and offers cooperate well, such as when a customer is assigned only once to an offer. When the offer expires, the customer is automatically removed from the offer, if he hasn’t accepted it; if we have added an offer on the customer side, the customer is added on the offer side.

### System software testing

​	System testing of software is testing conducted on a complete, integrated system to evaluate the system’s compliance with its specified requirements. The objective is to detect inconsistencies among units that are already integrated.

### Acceptance software testing

​	It is important that an application performs well, but the application must also meet the customer's needs*.* Acceptance tests are our final level of testing. The customer or a proxy usually conduct acceptance tests to ensure that the application meets whatever goals the customer or stakeholder defined.

​	Acceptance tests may be expressed by the *Given*, *When*, and *Then* keywords. When you use these keywords, you are practically following a scenario: the interaction of the user with the system. The verification in the Then step may look like a unit test, but it checks the end of the scenario and answers the question “Are we addressing the business goals?”

​	The at from Tested Data Systems may implement some acceptance tests such as this:

*  Given that there is an economy offer,
* When we have a usual customer,
* We can add him to and remove him from the offer.



* Given that there is an economy offer,
* When we have a VIP customer,
* We can add him to the offer but cannot remove him from the offer.

## Black-box vs. white-box testing

### Black-box testing

​	A *black-box test* has no knowledge of the internal state or behavior of the system. The test relies solely on the external system interface to verify its correctness.

​	As the name of this methodology suggests, you treat the system as a black box. Imagine it with buttons and LEDs. You do not know what is inside or how the system operates; all you know is that when the correct input is provided, the system produces the desired output. All you need to know to test the system properly is the system's functional specification. The early stages of a project typically produce this kind of specification, which means that we can start testing early. Anyone can take part in testing the system, such as a QA engineer, a developer, or even a customer.

​	The simplest form of black-box testing tries to mimic actions in the user interface manually. A more sophisticated approach is to use a tool for this task, such as HTTPUnit, HTMLUnit, or Selenium. We will apply some of these tools in another part of the book.

​	At Tested Data Systems, black-box testing is used for applications that provide a web interface. The testing tool knows only that it has to interact with the frontend (selections, pushbuttons, and so on) and to verify the results (the result of the action, the content of the destination page, and so on).

### White-box testing

​	White-box testing can be implemented at an earlier stage. There is no need to wait for the GUI to be available. You may cover many execution paths.

​	Which of the two approaches should you use? There is no absolute answer, so I suggest that you use both approaches. In some situations, you need user-centric tests (no need for details), and in others, you need to test the implementation details of the system. The next section presents the pros and cons of both approaches.

***USERCENTRIC APPROACH***

​	Black-box testing first addresses user needs. We know that there is tremendous value in customer feedback, and one of our goals in extreme programming is to release early and release often. We are unlikely to get useful feedback, however, if we just tell the customer “Here it is. Let me know what you think.” It is far better to get a customer involved by providing a manual test script to run through. By making the customer think about the application, he can also clarify what the system should do. Interacting with the constructed GUI and comparing the results that are obtained with the expected ones is an example of testing that customers may run by themselves.

***TESTING DIFFICULTIES***

​	Black-box tests are more difficult to write and run[***\*[1\]\****](#id_ftn1) because they usually deal with a graphical frontend, whether it’s a web browser or desktop application. Another issue is that a valid result on the screen does not always mean that the application is correct. White-box tests usually are usually easier to write and run than black-box tests, but the developers must implement them.

**The pros of black-box and white-box tests**

| Black box tests                                              | White box tests                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Tests are usercentric and expose specifications to discrepancies. | Testing can be implemented from the early stages of the project. |
| The tester may be a nontechnical person.                     | There is no need for an existing GUI.                        |
| Tests can be conducted independently of the developers.      | Testing is controlled by the developer and can cover many execution paths. |

**The cons of black-box and white-box tests**

| Black box tests                                              | White box tests                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| A limited number of inputs may be tested.                    | Testing can be implemented only by skilled people with programming knowledge. |
| Many program paths may be left uncovered.                    | Tests will need to be rewritten if the implementation changes. |
| Tests may become redundant; the lack of details may mean covering the same execution paths. | Tests are tightly coupled to the implementation.             |

***TEST COVERAGE***

​	White-box testing provides better test coverage than black-box testing. On the other hand, black-box tests can bring more value than white-box tests. I focus on test coverage in chapter 6.

# Tests quality

* Measuring test coverage
* Writing testable code
* Investigating Test Driven Development
* Investigating Behavior Driven Development
* Introducing mutation testing
* Testing in the development cycle

## Measuring test coverage

​	Writing unit tests gives you the confidence to change and refactor an application. As you make changes, you run tests, which give you immediate feedback on new features under test and whether changes break the existing tests. The issue is that these changes may still break untested functionality.

​	To resolve this issue, you need to know precisely what code runs when you or the build invoke tests. Ideally, your tests should cover 100 percent of your application code. This section examines the test coverage in detail.

###  Introduction to test coverage

![](https://pic.imgdb.cn/item/612438f944eaada7394734e4.jpg)

​	Many metrics can be used to calculate test coverage. Some of the most basic is the percentage of program methods and the percentage of program lines called during execution of the test suite.

​	You *can* write a unit test with intimate knowledge of a method's implementation. If a method contains a conditional branch, you can write two unit tests. one for each branch. Because you need to see into the method to create such a test, this type of test falls in the category of white-box testing*.* Figure 6.2 shows 100 percent test coverage with white-box testing.

![](https://pic.imgdb.cn/item/61243d8244eaada7394bd943.jpg)

### Code coverage measuring tools

![](https://pic.imgdb.cn/item/61243dab44eaada7394c0c0e.jpg)

​	The HTML reports get at the level of the `com.manning.junitbook.ch06` package (figure 6.5), individual `Calculator` class (figure 6.6), or individual lines of code (figure 6.7).

![](https://pic.imgdb.cn/item/61243dc744eaada7394c2c7f.jpg)

​	The recommended way to work is to execute the tests from the command line with code coverage, it works fine in conjunction with the Continuous Integration/Continuous Development pipeline. You may use the JaCoCo (Java Code Coverage) tool. JaCoCo is an open-source toolkit that is updated frequently and has very good integration with Maven. To make Maven and JaCoCo work together, you need to insert the JaCoCo plugin information into the pom.xml file (figure 6.8).

![](https://pic.imgdb.cn/item/61243de744eaada7394c5186.jpg)

​	You need to go to the command prompt and run `mvn test` (figure 6.9). This command also generates—for the`com.manning.junitbook` package and for the `Calculator` class—the code coverage report, which you can access from the project folder, target\site\jacoco (figure 6.10). I’ve chosen to show an example that does not have 100 percent code coverage. You can also do this by removing some of the existing tests from the `CalculatorTest` class in the accompanying `ch06-quality` folder.

![](https://pic.imgdb.cn/item/61243e0444eaada7394c7398.jpg)

​	From here, you can access the reports at the level of the `com.manning.junitbook` package (figure 6.11), `Calculator` class (figure 6.12), methods (figure 6.13), and individual lines (figure 6.14).

![](https://pic.imgdb.cn/item/61243e1d44eaada7394c8ff5.jpg)

![](https://pic.imgdb.cn/item/61243e2c44eaada7394c9f2d.jpg)

## Writing testable code

​	This chapter is dedicated to best practices in software testing. Now you’re ready to get to the next level: writing code that is easy to test. Sometimes, writing a single test case is easy; sometimes, it is not. Everything depends on the complexity of the application. A best practice avoids complexity as much as possible; code should be readable and testable.

###  Understand that public APIs are contracts

​	One principle in providing backward-compatible software states “Never change the signature of a public method.” An application code review shows that most calls are made from external applications, which play the role of clients of your API. If you change the signature of a public method, you need to change every call site in the application and unit tests. Even with refactoring wizards in tools such as IntelliJ or Eclipse, you must always perform this task with care. During initial development, especially if working TDD (Test Driven Development), it is a great time to refactor the API as needed, because you do not yet have any public users. Things will change afterward!

### Reduce dependencies

​	Remember that unit tests verify your code in isolation. Your unit tests should instantiate the class you want to test, use it, and assert its correctness. Your test cases should be simple. What happens when your class instantiates a new set of objects, directly or indirectly? Now your class depends on these classes. To write testable code, you should reduce dependencies as much as possible. If your classes depend on many other classes that need to be instantiated and set up with some state, your tests will be very complicated; you may need to use a complicated mock-objects solution (see chapter 8).

![](https://pic.imgdb.cn/item/61243fec44eaada7394e8175.jpg)

​	This code allows you to produce a mock Driver object (see chapter 8) and pass it to the `Vehicle` class on instantiation. This process is *dependency injection*—a technique in which a dependency is supplied to another object. You can mock any other type of `Driver` implementation. The requirements for the engineers at Tested Data Systems *may introduce specific classes such as* `JuniorDriver` and `SeniorDriver`, and pass those classes to the `Vehicle` class. Through dependency injection, the code supplies the `Driver` object to the `Vehicle` object by invoking the `Vehicle(Driver d)` constructor.

### Create simple constructors

​	By striving for better test coverage, you add more test cases. In each of these test cases, you

* Instantiate the class to test.
* Set the class to a particular state.
* Assert the final state of the class.

### Follow the Law of Demeter (Principle of Least Knowledge)

​	The Law of Demeter, or *Principle of Least Knowledge*, states that one class should know only as much as it needs to know.

​	*Talk to your immediate friends.*

or

​	*Don't talk to strangers.*

![](https://pic.imgdb.cn/item/612440fd44eaada7394fbb04.jpg)

### Avoid hidden dependencies and global state

​	Be very careful with the global state, because the global state makes it possible for many clients to share the global object. This sharing can have unintended consequences if the global object is not coded for shared access or if clients expect exclusive access to the global object.

### Favor generic methods

​	Static methods, like factory methods, are very useful, but large groups of utility static methods can introduce issues of their own. Recall that unit testing is testing in isolation. To achieve isolation, you need some articulation points in your code, where you can easily substitute your code for the test code. These points use polymorphism. With *polymorphism* (the ability of one object to pass more than one IS-A tests), the method you are calling is not determined at compile time. You can easily use polymorphism to substitute application code for the test code to force certain code patterns to be tested.

### Favor composition over inheritance

​	Many people choose inheritance as a code-reuse mechanism. I think that composition can be easier to test. At run time, code cannot change an inheritance hierarchy, but you can compose objects differently. What you strive for is to make your code as flexible as possible at run time. This way, you can be sure that it is easy to switch from one state of your objects to another, which makes the code easily testable.

### Favor polymorphism over conditionals

As I mentioned previously, all you do in your tests is

* Instantiate the class to test.
* Set the class to a particular state.
*  Assert the final state of the class.

![](https://pic.imgdb.cn/item/6124e94944eaada739c1bdec.jpg)

![](https://pic.imgdb.cn/item/6124e96044eaada739c1f311.jpg)

## Test Driven Development

​	As you design an application, tests may help you improve the initial design. 

### Adapting the development cycle

​	When you develop code, you design an application programming interface (API) and then implement the behavior promised by the interface. When you unit-test code, you verify the promised behavior through an API. The test is a client of the API, just as your domain code is a client of the API.

​	The conventional development cycle goes something like this:

​		[code,test, (repeat)]

​	Developers who practice TDD make a seemingly slight but surprisingly effective adjustment:

​		[*test*, code, (repeat)]

​	This approach has a few advantages:

* You write code that is driven by clear goals and can be sure that you address exactly what your application needs to do. Tests represent a means to design the code.
* You can introduce new functionality much faster. Tests drive you to implement the code that does what it is supposed to do.
* Tests prevent you from introducing bugs into existing code that is working well.
* Tests serve as documentation, so you can follow them and understand what problems the code is supposed to solve.

### Doing the TDD two-step

Earlier, I said that TDD tweaks the development cycle to go something like

[test, code, (repeat)]

The problem with this process is that it leaves out a key step. Development should go more like this:

[test, code,refactor, (repeat)].

​	*Refactoring* is the process of changing a software system in such a way that it improves the code’s internal structure without altering its external behavior. To make sure that external behavior is not affected, you need to rely on tests.

​	The core tenets of TDD are

* Write a failing test before writing new code.
* Write the smallest piece of code that will make the new test pass.

## Behavior Driven Development

​	Behavior Driven Development, which emerged during the mid-2000s, is a methodology for developing IT solutions that satisfy business requirements directly. Its philosophy is driven by business strategy, requirements, and goals, which are refined and transformed into an IT solution.

​	Whereas TDD helps you build good-quality software, BDD helps you build software that’s worth building to solve the problems of the users.

## Mutation testing

​	*Mutation testing* (or *mutation analysis* or *program mutation*) is used to design new software tests and evaluate the quality of existing software tests.

​	The basic idea of mutation testing involves modifying a program in small ways. Each mutated version is called a *mutant*. The behavior of the original version differs from that of the mutant. Tests detect and reject mutants, which is called *killing the mutant*. Test suites are measured by the percentage of mutants that they kill. New tests can be designed to kill additional mutants.

​	Mutants are generated by well-defined mutation operators that change an existing operator for another one or invert some conditions. The goal is to support the creation of effective tests or to locate weaknesses in the test data used for the program or sections of the code that are seldom or never accessed during execution. Consequently, mutation testing is a form of white-box testing.

![](https://pic.imgdb.cn/item/6124ed0c44eaada739ca0689.jpg)

A strong mutation test fulfills the following three conditions:

* The test reaches the `if` condition that has been mutated.
* The test continues on a different branch from the initial correct one.
* The changed value of `b` propagates to the output of the program and is checked by the test.
* The test will fail because the method returned the wrong value for b.



​	Well-written tests must bring to the failure of mutated tests to demonstrate that they initially covered the necessary logical conditions.

## Testing in the development cycle

​	Testing occurs at different places and times during the development cycle. This section first introduces a development life cycle and then uses it as a basis for deciding what types of tests are executed when. Figure 6.15 shows a typical development cycle.

![](https://pic.imgdb.cn/item/6124ee4244eaada739ccc8c4.jpg)

* *Development*—This platform is where coding happens on developers’ workstations.
* *Integration*—This platform builds the application from its components (which may have been developed by different teams) and ensures that the components work together. 
* *Acceptance/stress test*—Depending on the resources available to your project, this platform can be one or two platforms. 
* *(Pre)production* —The preproduction platform is the last staging area before production. 



​	Next, I discuss how testing fits into the development cycle. Figure 6.16 highlights the types of tests you can perform on each platform.

![](https://pic.imgdb.cn/item/6124ef0f44eaada739ce8eeb.jpg)

* On the *development platform*, you execute *logic unit tests* (tests that can be executed in isolation from the environment).
* The *integration platform* usually runs the build process automatically to package and deploy the application; then it executes unit and functional tests.
* On the *acceptance platform**/stress test platform*, you execute the same tests executed by the integration platform; in addition, you run stress tests (to verify the robustness and performance of the software). 
* A good habit is to run on the (*pre)production platform* the tests you ran on the acceptance platform. 



​	Now, again, it is up to you. Are you going to strive for perfection, stick to everything that you’ve learned so far, and let your code benefit from that knowledge?

# Coarse-grained testing with stubs

* Testing with stubs
* Using an embedded server in place of a real webserver
*  Implementing unit tests of an HTTP connection with stubs



​	As you develop your applications, you will find that the code you want to test depends on other classes, which themselves depend on other classes, which then depend on the environment (figure 7.1). You might be developing an application that uses Hibernate to access a database, a Java EE application (one that relies on a Java EE container for security, persistence, and other services), an application that accesses a file system, or an application that connects to some resource by using HTTP or another protocol.

![](https://pic.imgdb.cn/item/6124f13744eaada739d3503c.jpg)

​	There are two strategies for providing these fake objects: stubbing and using mock objects.

​	When you write stubs, you provide a predetermined behavior right from the beginning. The stubbed code is written outside the test, and it will always have a fixed behavior, no matter how many times or where you will use the stub;– its methods will usually return hard-coded values. The pattern of testing with a stub is: *initialize stub > execute test > verify assertions*.

## Introducing stubs

​	Stubs are mechanisms for faking the behavior of real code or code that is not ready yet. In other words, stubs allow you to test a portion of a system when the other part is not available. Stubs usually do not change the code you are testing; instead, they adapt to provide seamless integration.

​	Here are some examples of when you might use stubs:

* You cannot modify an existing system because it is too complex and fragile.
* You are depending on an environment that you cannot control.
* You are replacing a full-blown external system such as a file system, a connection to a server, or a database.
* You are performing coarse-grained testing, such as integration testing between different subsystems.



​	Using stubs is not recommended in situations such as these:

* You need fine-grained tests to provide precise messages that underline the cause of the failure.
* You would like to test a small portion of the code in isolation.



​	In these situations, you should use mock objects (discussed in chapter 8).

​	On the downside, stubs are usually hard to write, especially when the system to fake is complex. The stub needs to implement, in a simplified and short way, the same logic as the code it is replacing, which is difficult to get right for complex logic. Here are some cons of stubbing:

* Stubs are often complex to write and need debugging themselves.
* Stubs can be difficult to maintain because they are complex.
* A stub does not lend itself well to fine-grained unit testing.
* Each situation requires a different stubbing strategy.

## Stubbing an HTTP connection

![](https://pic.imgdb.cn/item/6124f3dd44eaada739d9611c.jpg)



![](https://pic.imgdb.cn/item/6124f3ef44eaada739d98c6a.jpg)

```java
public class WebClient {
    public String getContent(URL url) {
        StringBuffer content = new StringBuffer();
        try {
            HttpURLConnection connection = (HttpURLConnection) url.openConnection();
            connection.setDoInput(true);
            InputStream is = connection.getInputStream();
            byte[] buffer = new byte[2048];
            int count;
            while (-1 != (count = is.read(buffer))) {
                content.append(new String(buffer, 0, count));
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
        return content.toString();
    }
}
```

In this listing:

* We start by opening an HTTP connection using the `HttpURLConnection` class (#A).
* We read the stream content until there is nothing more to read (#B).
* If an error occurs, we pack it into a `RuntimeException` and rethrow it `(#C)`.

### Choosing a stubbing solution

​	The example application has two possible scenarios: the remote web server (see figure 7.2) could be located outside the development platform (such as on a partner site), or it could be part of the platform where the application is to be deployed.

​	One relatively easy solution would be to install an Apache test server and drop some test web pages in its document root. This stubbing solution is typical and widely used, but it has several drawbacks, as shown in table 7.1.

| Drawback                    | Explanation                                                  |
| --------------------------- | ------------------------------------------------------------ |
| Reliance on the environment | We need to be sure that the full environment is up and running before the test starts.If the webserver is down, and we execute the test, it will fail, and we will spend time determining why it is failing.We will discover that the code is working fine and that the problem was only a setup issue generating a false failure.When we are unit testing, it is important to be able to control as much as possible of the environment in which the tests execute, such that test results are reproducible. |
| Separated test logic        | The test logic is scattered in two locations: in the JUnit 5 test case and on the test web page.We need to keep both types of resources in sync for the tests to succeed. |
| Difficult tests to automate | Automating the execution of the tests is difficult because it involves deploying the web pages on the webserver, starting the webserver, and running the unit tests. |

​	Fortunately, an easier solution exists: using an embedded web server. Because the testing is in Java, the easiest solution is to use a Java web server that can be embedded in the test case. You can use the free, open-source Jetty server for this purpose. The developers at Tested Data Systems Inc. will use Jetty to set up their stubs. (For more information about Jetty, visit [https://www.eclipse.org/jetty](https://www.eclipse.org/jetty/).)

### Using Jetty as an embedded server

​	We use Jetty because it is fast (important when running tests), it is lightweight, and the test cases can control it programmatically. Additionally, Jetty is a very good web, servlet, and JSP container that you can use in production.

```java
public class JettySample {
    public static void main(String[] args) throws Exception {
        Server server = new Server(8081);

        Context root = new Context(server, "/");
        root.setResourceBase("./pom.xml");
        root.setHandler(new ResourceHandler());

        server.setStopAtShutdown(true);
        server.start();
    }
}
```

In this listing:

* We start by creating the Jetty `Server` object (#A) and specifying in the constructor which port to listen to for HTTP requests (port 8080).

* Next, we create a `Context` object (#B) that processes the HTTP requests and passes them to various handlers. We map the context to the already-created server instance, and to the root (`/`) URL. The `setResourceBase` method sets the document root from which to serve resources. On the next line, we attach a `ResourceHandler` handler to the root to serve files from the file system.

* Finally, we start the server (#C).

## Stubbing the web server resources

​	To verify that the `WebClient` works with a valid URL, we need to start the Jetty server before the test, which we can implement in a test case `setUp` method. We can also stop the server in a `tearDown` method. Listing 7.3 shows the code.

```java
public class TestWebClientSkeleton {

    @BeforeAll
    public static void setUp() {
        // Start Jetty and configure it to return "It works" when
        // the http://localhost:8081/testGetContentOk URL is
        // called.
    }

    @AfterAll
    public static void tearDown() {
        // Stop Jetty.
    }

    @Test
    @Disabled(value = "This is just the initial skeleton of a test. Therefore, if we run it now, it will fail.")
    public void testGetContentOk() throws MalformedURLException {
        WebClient client = new WebClient();
        String workingContent = client.getContent(new URL("http://localhost:8081/testGetContentOk"));

        assertEquals("It works", workingContent);
    }
}
```

​	To implement the `@BeforeAll` and `@AfterAll` methods, we have are two options. We can prepare a static page containing the text `"It works",` which we put in the document root (controlled by the call to `root.setResourceBase(String)` in listing 7.2). Alternatively, we can configure Jetty to use a custom handler that returns the string `"It works"` instead of getting it from a file. This technique is much more powerful because it lets us unit-test the case when the remote HTTP server returns an error code to the `WebClient` client application.

```java
private static class TestGetContentOkHandler extends AbstractHandler {
    public void handle(String target, HttpServletRequest request, HttpServletResponse response, int dispatch) throws IOException {

        OutputStream out = response.getOutputStream();
        ByteArrayISO8859Writer writer = new ByteArrayISO8859Writer();
        writer.write("It works");
        writer.flush();
        response.setIntHeader(HttpHeaders.CONTENT_LENGTH, writer.size());
        writer.writeTo(out);
        out.flush();
    }
}
```

*WRITING THE TEST CLASS*

```java
public class TestWebClient {

    private WebClient client = new WebClient();

    @BeforeAll
    public static void setUp() throws Exception {
        Server server = new Server(8081);

        Context contentOkContext = new Context(server, "/testGetContentOk");
        contentOkContext.setHandler(new TestGetContentOkHandler());

        Context contentErrorContext = new Context(server, "/testGetContentError");
        contentErrorContext.setHandler(new TestGetContentServerErrorHandler());

        Context contentNotFoundContext = new Context(server, "/testGetContentNotFound");
        contentNotFoundContext.setHandler(new TestGetContentNotFoundHandler());

        server.setStopAtShutdown(true);
        server.start();
    }

    @AfterAll
    public static void tearDown() {
        // Empty
    }

    @Test
    public void testGetContentOk() throws MalformedURLException {
        String workingContent = client.getContent(new URL("http://localhost:8081/testGetContentOk"));
        assertEquals("It works", workingContent);
    }

    /**
     * Handler to handle the good requests to the server.
     */
    private static class TestGetContentOkHandler extends AbstractHandler {
        public void handle(String target, HttpServletRequest request, HttpServletResponse response, int dispatch) throws IOException {

            OutputStream out = response.getOutputStream();
            ByteArrayISO8859Writer writer = new ByteArrayISO8859Writer();
            writer.write("It works");
            writer.flush();
            response.setIntHeader(HttpHeaders.CONTENT_LENGTH, writer.size());
            writer.writeTo(out);
            out.flush();
        }
    }

    /**
     * Handler to handle bad requests to the server
     */
    private static class TestGetContentServerErrorHandler extends AbstractHandler {

        public void handle(String target, HttpServletRequest request, HttpServletResponse response, int dispatch) throws IOException {
            response.sendError(HttpServletResponse.SC_SERVICE_UNAVAILABLE);
        }
    }

    /**
     * Handler to handle requests that request unavailable content.
     */
    private static class TestGetContentNotFoundHandler extends AbstractHandler {

        public void handle(String target, HttpServletRequest request, HttpServletResponse response, int dispatch) throws IOException {
            response.sendError(HttpServletResponse.SC_NOT_FOUND);
        }
    }
}
```

## Stubbing the connection

### Producing a custom URL protocol handler

```java
public class TestWebClient1 {

    @BeforeAll
    public static void setUp() {
        URL.setURLStreamHandlerFactory(new StubStreamHandlerFactory());
    }

    private static class StubStreamHandlerFactory implements URLStreamHandlerFactory {
        @Override
        public URLStreamHandler createURLStreamHandler(String protocol) {
            return new StubHttpURLStreamHandler();
        }
    }

    private static class StubHttpURLStreamHandler extends URLStreamHandler {
        @Override
        protected URLConnection openConnection(URL url) {
            return new StubHttpURLConnection(url);
        }
    }

    @Test
    public void testGetContentOk() throws MalformedURLException {
        WebClient client = new WebClient();
        String workingContent = client.getContent(new URL("http://localhost/"));
        assertEquals("It works", workingContent);
    }
}
```

### Creating a JDK HttpURLConnection stub

```java
public class StubHttpURLConnection extends HttpURLConnection {
    private boolean isInput = true;

    protected StubHttpURLConnection(URL url) {
        super(url);
    }

    @Override
    public InputStream getInputStream() throws IOException {
        if (!isInput) {
            throw new ProtocolException("Cannot read from URLConnection" + " if doInput=false (call setDoInput(true))");
        }
        ByteArrayInputStream readStream = new ByteArrayInputStream(new String("It works").getBytes());
        return readStream;
    }

    @Override
    public void connect() throws IOException {
    }

    @Override
    public void disconnect() {
    }

    @Override
    public boolean usingProxy() {
        return false;
    }
}
```

### Running the test

![](https://pic.imgdb.cn/item/6124f9af44eaada739e71a9a.jpg)

# Testing with mock objects

## Introducing mock objects

​	Testing in isolation offers strong benefits, such as the ability to test code that has not yet been written (as long as you at least have an interface to work with). In addition, testing in isolation helps teams unit-test one part of the code without waiting for all the other parts.

## Unit testing with mock objects

![](https://pic.imgdb.cn/item/612c7df044eaada739b27227.jpg)

```java
public class Account {
    private String accountId;
    private long balance;

    public Account(String accountId, long initialBalance) {
        this.accountId = accountId;
        this.balance = initialBalance;
    }

    public void debit(long amount) {
        this.balance -= amount;
    }

    public void credit(long amount) {
        this.balance += amount;
    }

    public long getBalance() {
        return this.balance;
    }
}
```

```java
public interface AccountManager {
    /**
     * A method to find an account by the given userId.
     *
     * @param userId
     * @return
     */
    Account findAccountForUser(String userId);

    /**
     * A method to update the given accout.
     *
     * @param account
     */
    void updateAccount(Account account);


}
```

```java
public class AccountService {
    /**
     * The account manager implementation to use.
     */
    private AccountManager accountManager;

    /**
     * A setter method to set the account manager implementation.
     *
     * @param manager
     */
    public void setAccountManager(AccountManager manager) {
        this.accountManager = manager;
    }

    /**
     * A transfer method which transfers the amount of money
     * from the account with the senderId to the account of
     * beneficiaryId.
     *
     * @param senderId
     * @param beneficiaryId
     * @param amount
     */
    public void transfer(String senderId, String beneficiaryId, long amount) {
        Account sender = accountManager.findAccountForUser(senderId);
        Account beneficiary = accountManager.findAccountForUser(beneficiaryId);

        sender.debit(amount);
        beneficiary.credit(amount);
        this.accountManager.updateAccount(sender);
        this.accountManager.updateAccount(beneficiary);
    }
}
```

```java
public class MockAccountManager implements AccountManager {
    /**
     * A Map to hold all the <userId, account> values.
     */
    private Map<String, Account> accounts = new HashMap<String, Account>();

    /**
     * A method to add an account to the manager.
     *
     * @param userId
     * @param account
     */
    public void addAccount(String userId, Account account) {
        this.accounts.put(userId, account);
    }

    /**
     * A method to find an account for the user with the given ID.
     */
    public Account findAccountForUser(String userId) {
        return this.accounts.get(userId);
    }

    /**
     * A method to update the given account. Notice that we don't need this method and that's why we leave it with a
     * blank implementation.
     */
    public void updateAccount(Account account) {
        // do nothing
    }
}
```

​	Now you are ready to write a unit test for `AccountService.transfer`. Listing 8.4 shows a typical test that uses a mock.

```java
public class TestAccountService {
    @Test
    public void testTransferOk() {
        Account senderAccount = new Account("1", 200);
        Account beneficiaryAccount = new Account("2", 100);

        MockAccountManager mockAccountManager = new MockAccountManager();
        mockAccountManager.addAccount("1", senderAccount);
        mockAccountManager.addAccount("2", beneficiaryAccount);

        AccountService accountService = new AccountService();
        accountService.setAccountManager(mockAccountManager);

        accountService.transfer("1", "2", 50);

        assertEquals(150, senderAccount.getBalance());
        assertEquals(150, beneficiaryAccount.getBalance());
    }
}
```

## Refactoring with mock objects

​	Some people used to say that unit tests should be fully transparent to the code under test and that run-time code should not be changed to simplify testing. *This is wrong!* Unit tests are first-class users of the run-time code and deserve the same consideration as any other user. If the code is too inflexible for the tests to use, we should correct the code.

```java
public class DefaultAccountManager1
        implements AccountManager {
    /**
     * Logger instance.
     */
    private static final Log logger = LogFactory.getLog(DefaultAccountManager1.class);

    /**
     * Finds an account for user with the given userID.
     *
     * @param
     */
    public Account findAccountForUser(String userId) {
        logger.debug("Getting account for user [" + userId + "]");
        ResourceBundle bundle = PropertyResourceBundle.getBundle("technical");
        String sql = bundle.getString("FIND_ACCOUNT_FOR_USER");

        // Some code logic to load a user account using JDBC
        return null;
    }

    /**
     * Updates the given account.
     *
     * @param
     */
    public void updateAccount(Account account) {
        // Perform database access here
    }
}
```

### Refactoring example

```java
public class DefaultAccountManager2
        implements AccountManager {
    /**
     * Logger instance.
     */
    private Log logger;

    /**
     * Configuration to use.
     */
    private Configuration configuration;

    /**
     * Constructor with no parameters.
     */
    public DefaultAccountManager2() {
        this(LogFactory.getLog(DefaultAccountManager2.class),
                new DefaultConfiguration("technical"));
    }

    /**
     * Constructor with logger and configration parameters.
     *
     * @param logger
     * @param configuration
     */
    public DefaultAccountManager2(Log logger,
                                  Configuration configuration) {
        this.logger = logger;
        this.configuration = configuration;
    }

    /**
     * Finds an account for user with the given userID.
     *
     * @param
     */
    public Account findAccountForUser(String userId) {
        this.logger.debug("Getting account for user ["
                + userId + "]");
        this.configuration.getSQL("FIND_ACCOUNT_FOR_USER");

        // Some code logic to load a user account using JDBC
        return null;
    }

    /**
     * Updates the given account.
     */
    public void updateAccount(Account account) {
        // Perform database access here
    }
}
```

### Refactoring considerations

​	With this refactoring, we have provided a trap door for controlling the domain objects from the tests. We retain backward compatibility and pave an easy refactoring path for the future. Calling classes can start using the new constructor at their own pace.

```java
public class TestDefaultAccountManager {

    @Test
    public void testFindAccountByUser() {
        MockLog logger = new MockLog();
        MockConfiguration configuration = new MockConfiguration();
        configuration.setSQL("SELECT * [...]");
        DefaultAccountManager2 am = new DefaultAccountManager2(logger, configuration);

        @SuppressWarnings("unused")
        Account account = am.findAccountForUser("1234");

        // Perform asserts here
    }
}
```

## Mocking an HTTP connection

![](https://pic.imgdb.cn/item/612c804044eaada739b7e23d.jpg)

### Defining the mock objects

![](https://pic.imgdb.cn/item/612c805844eaada739b8184a.jpg)

### Testing a sample method

```java
[…]
import java.net.URL;
import java.net.HttpURLConnection;
import java.io.InputStream;
import java.io.IOException;
 
public class WebClient {
   public String getContent(URL url) {
      StringBuffer content = new StringBuffer();
             try {
         HttpURLConnection connection =                                 #A
            (HttpURLConnection) url.openConnection();                   #A
         connection.setDoInput(true);
         InputStream is = connection.getInputStream();                  #A
         int count;                                                     #A
         while (-1 != (count = is.read())) {                            #B
            content.append( new String( Character.toChars( count ) ) ); #B
         }                                                              #B
      } catch (IOException e) {
         return null;                                                   #C
      }
      return content.toString();
   }
}
```

### Try #1: easy method refactoring technique

```java
@Test
public void testGetContentOk() throws Exception {
   MockHttpURLConnection mockConnection = new MockHttpURLConnection();  
   mockConnection.setupGetInputStream(                                  
                     new ByteArrayInputStream("It works".getBytes()));  
   MockURL mockURL = new MockURL();                                   
   mockURL.setupOpenConnection(mockConnection);                        
   WebClient client = new WebClient();
   String workingContent = client.getContent(mockURL);                 
   assertEquals("It works", workingContent);                           
}
```

​	Unfortunately, this approach does not work! The JDK `URL` class is a final class, and no URL interface is available. So much for extensibility.

**Extracting retrieval of the connection object from getContent**

```java
public class WebClient1 {
    /**
     * A method to retrieve the content from the given URL.
     *
     * @param url
     * @return
     */
    public String getContent(URL url) {
        StringBuffer content = new StringBuffer();

        try {
            HttpURLConnection connection = createHttpURLConnection(url);
            InputStream is = connection.getInputStream();

            int count;
            while (-1 != (count = is.read())) {
                content.append(new String(Character.toChars(count)));
            }
        } catch (IOException e) {
            return null;
        }

        return content.toString();
    }

    /**
     * Creates an HTTP connection.
     *
     * @param url
     * @return
     * @throws IOException
     */
    protected HttpURLConnection createHttpURLConnection(URL url) throws IOException {
        return (HttpURLConnection) url.openConnection();
    }
}
```

​	How does this solution test `getContent` more effectively? It allows us to apply a useful trick, which writes a test helper class that extends the `WebClient` class and overrides its `createHttpURLConnection` method, as follows:

```java
private class TestableWebClient
        extends WebClient1 {
    /**
     * The connection.
     */
    private HttpURLConnection connection;

    /**
     * Setter method for the HttpURLConnection.
     *
     * @param connection
     */
    public void setHttpURLConnection(HttpURLConnection connection) {
        this.connection = connection;
    }

    /**
     * A method that we overwrite to create the URL connection.
     */
    public HttpURLConnection createHttpURLConnection(URL url)
            throws IOException {
        return this.connection;
    }
}
```

​	In the test, we can call the `setHttpURLConnection` method, passing it the mock `HttpURLConnection` object. Now the test becomes the following. (Differences are shown in bold.)

```java
@Test
public void testGetContentOk()
        throws Exception {
    MockHttpURLConnection mockConnection = new MockHttpURLConnection();
    mockConnection.setExpectedInputStream(new ByteArrayInputStream("It works".getBytes()));

    TestableWebClient client = new TestableWebClient();
    client.setHttpURLConnection(mockConnection);

    String result = client.getContent(new URL("http://localhost"));

    assertEquals("It works", result);
}
```

### Try #2: refactoring by using a class factory

​	The developers at Tested Data Systems want to give another refactoring try by applying the IoC pattern, which says that any resource we use needs to be passed to the `getContent` method or `WebClient` class. The only resource we use is the`HttpURLConnection` object. We could change the `WebClient.getContent` signature to

```java
public String getContent(URL url, HttpURLConnection connection);
```

```java
public interface ConnectionFactory {
    /**
     * Read the data from the connection.
     *
     * @return
     * @throws Exception
     */
    InputStream getData() throws Exception;
}
```

```java
public class WebClient2 {
    /**
     * Open a connection to the given URL and read the content
     * out of it. In case of an exception we return null.
     *
     * @param connectionFactory
     * @return
     */
    public String getContent(ConnectionFactory connectionFactory) {
        String workingContent;

        StringBuffer content = new StringBuffer();
        try (InputStream is = connectionFactory.getData()) {
            int count;
            while (-1 != (count = is.read())) {
                content.append(new String(Character.toChars(count)));
            }

            workingContent = content.toString();
        } catch (Exception e) {
            workingContent = null;
        }


        return workingContent;
    }
}
```

```java
public class HttpURLConnectionFactory
        implements ConnectionFactory {
    /**
     * URL for the connection.
     */
    private URL url;

    /**
     * Constructor with the url as a parameter.
     *
     * @param url
     */
    public HttpURLConnectionFactory(URL url) {
        this.url = url;
    }

    /**
     * Read the data from the HTTP input stream.
     *
     * @return
     */
    public InputStream getData()
            throws Exception {
        HttpURLConnection connection = (HttpURLConnection) this.url.openConnection();
        return connection.getInputStream();
    }
}
```

**MockConnectionFactory**

```java
public class MockConnectionFactory implements ConnectionFactory {
    /**
     * The input stream for the connection.
     */
    private InputStream inputStream;

    /**
     * Set the input stream.
     *
     * @param stream
     */
    public void setData(InputStream stream) {
        this.inputStream = stream;
    }

    /**
     * Get the input stream.
     *
     * @throws Exception
     */
    public InputStream getData() //throws Exception
    {
        return inputStream;
    }
}
```

**Refactored WebClient test using MockConnectionFactory**

```java
public class TestWebClient {
    @Test
    public void testGetContentOk()
            throws Exception {
        MockConnectionFactory mockConnectionFactory = new MockConnectionFactory();
        MockInputStream mockStream = new MockInputStream();
        mockStream.setBuffer("It works");

        mockConnectionFactory.setData(mockStream);

        WebClient2 client = new WebClient2();

        String workingContent = client.getContent(mockConnectionFactory);

        assertEquals("It works", workingContent);
        mockStream.verify();
    }
}
```

## Using mocks as Trojan horses

**Mock InputStream with an expectation on close**

```java
public class MockInputStream
        extends InputStream {
    /**
     * Buffer to read in.
     */
    private String buffer;

    /**
     * Current position in the stream.
     */
    private int position = 0;

    /**
     * How many times the close method was called.
     */
    private int closeCount = 0;

    /**
     * Sets the buffer.
     *
     * @param buffer
     */
    public void setBuffer(String buffer) {
        this.buffer = buffer;
    }

    /**
     * Reads from the stream.
     *
     * @return
     */
    public int read()
            throws IOException {
        if (position == this.buffer.length()) {
            return -1;
        }

        return buffer.charAt(this.position++);
    }

    /**
     * Close the stream.
     */
    public void close()
            throws IOException {
        closeCount++;
        super.close();
    }

    /**
     * Verify how many times the close method was called.
     *
     * @throws java.lang.AssertionError
     */
    public void verify()
            throws java.lang.AssertionError {
        if (closeCount != 1) {
            throw new AssertionError("close() should " + "have been called once and once only");
        }
    }
}
```

​	In the case of the `MockInputStream` class, the expectation for `close` is simple: we always want it to be called once. Most of the time, however, the expectation for `closeCount` depends on the code under test. A mock usually has a method such as`setExpectedCloseCalls` so that the test can tell the mock what to expect.

```java
public class TestWebClient {
    @Test
    public void testGetContentOk()
            throws Exception {
        MockConnectionFactory mockConnectionFactory = new MockConnectionFactory();
        MockInputStream mockStream = new MockInputStream();
        mockStream.setBuffer("It works");

        mockConnectionFactory.setData(mockStream);

        WebClient2 client = new WebClient2();

        String workingContent = client.getContent(mockConnectionFactory);

        assertEquals("It works", workingContent);
        mockStream.verify();
    }
}
```

## Introducing Mock frameworks

### Using EasyMock

​	EasyMock ([http://easymock.org](http://easymock.org/)) is an open-source framework that provides useful classes for mocking objects. To work with it, we need to add to the pom.xml file the dependencies shown in listing 8.18.

```xml
<dependency>
   <groupId>org.easymock</groupId>
   <artifactId>easymock</artifactId>
   <version>2.4</version>
</dependency>
<dependency>
   <groupId>org.easymock</groupId>
   <artifactId>easymockclassextension</artifactId>
   <version>2.4</version>
</dependency>
```

```java
public class TestWebClientEasyMock {
    private ConnectionFactory factory;

    private InputStream stream;

    @BeforeEach
    public void setUp() {
        factory = createMock("factory", ConnectionFactory.class);
        stream = createMock("stream", InputStream.class);
    }

    @Test
    public void testGetContentOk()
            throws Exception {
        expect(factory.getData()).andReturn(stream);
        expect(stream.read()).andReturn(Integer.valueOf((byte) 'W'));
        expect(stream.read()).andReturn(Integer.valueOf((byte) 'o'));
        expect(stream.read()).andReturn(Integer.valueOf((byte) 'r'));
        expect(stream.read()).andReturn(Integer.valueOf((byte) 'k'));
        expect(stream.read()).andReturn(Integer.valueOf((byte) 's'));
        expect(stream.read()).andReturn(Integer.valueOf((byte) '!'));

        expect(stream.read()).andReturn(-1);
        stream.close();

        replay(factory);
        replay(stream);

        WebClient2 client = new WebClient2();

        String workingContent = client.getContent(factory);

        assertEquals("Works!", workingContent);
    }

    @Test
    public void testGetContentInputStreamNull() throws Exception {
        expect(factory.getData()).andReturn(null);

        replay(factory);
        replay(stream);

        WebClient2 client = new WebClient2();

        String workingContent = client.getContent(factory);

        assertNull(workingContent);
    }

    @Test
    public void testGetContentCannotCloseInputStream() throws Exception {
        expect(factory.getData()).andReturn(stream);
        expect(stream.read()).andReturn(-1);
        stream.close();
        expectLastCall().andThrow(new IOException("cannot close"));

        replay(factory);
        replay(stream);

        WebClient2 client = new WebClient2();
        String workingContent = client.getContent(factory);

        assertNull(workingContent);
    }

    @AfterEach
    public void tearDown() {
        verify(factory);
        verify(stream);
    }
}
```

### Using JMock

​	So far, you’ve seen how to implement your own mock-objects and use the EasyMock framework. In this section, we introduce the JMock framework ([http://jmock.org](http://jmock.org/)). We’ll follow the same scenario that the engineers from Tested Data Systems are following to evaluate the capabilities of a mock framework and compare them with those of other frameworks: testing money transfer with the help of a mock `AccountManager`, this time using JMock.

```xml
<dependency>
   <groupId>org.jmock</groupId>
   <artifactId>jmock-junit5</artifactId>
   <version>2.12.0</version>
</dependency>
<dependency>
   <groupId>org.jmock</groupId>
   <artifactId>jmock-legacy</artifactId>
   <version>2.5.1</version>
</dependency>
```

**Reworking the TestAccountService test using JMock**

```java
public class TestWebClientJMock
{
    @RegisterExtension
    Mockery context = new JUnit5Mockery()
    {
        {
            setImposteriser( ClassImposteriser.INSTANCE );
        }
    };

    @Test
    public void testGetContentOk() throws Exception
    {
        ConnectionFactory factory = context.mock( ConnectionFactory.class );
        InputStream mockStream = context.mock( InputStream.class );

        context.checking( new Expectations()
        {
            {
                oneOf( factory ).getData();
                will( returnValue( mockStream ) );

                atLeast(1).of(mockStream).read();
                will( onConsecutiveCalls( returnValue( Integer.valueOf( (byte) 'W' ) ),
                                          returnValue( Integer.valueOf( (byte) 'o' ) ),
                                          returnValue( Integer.valueOf( (byte) 'r' ) ),
                                          returnValue( Integer.valueOf( (byte) 'k' ) ),
                                          returnValue( Integer.valueOf( (byte) 's' ) ),
                                          returnValue( Integer.valueOf( (byte) '!' ) ),
                                          returnValue( -1 ) ) );

                oneOf( mockStream ).close();
            }
        } );

        WebClient2 client = new WebClient2();

        String workingContent = client.getContent( factory );

        assertEquals( "Works!", workingContent );
    }

    @Test
    public void testGetContentCannotCloseInputStream()
        throws Exception
    {

        ConnectionFactory factory = context.mock( ConnectionFactory.class );
        InputStream mockStream = context.mock( InputStream.class );

        context.checking( new Expectations()
        {
            {
                oneOf( factory ).getData();
                will( returnValue( mockStream ) );
                oneOf( mockStream ).read();
                will( returnValue( -1 ) );
                oneOf( mockStream ).close();
                will( throwException( new IOException( "cannot close" ) ) );
            }
        } );

        WebClient2 client = new WebClient2();

        String workingContent = client.getContent( factory );

        assertNull( workingContent );
    }
}
```

### Using Mockito

This section introduces Mockito ([https://site.mockito.org](https://site.mockito.org/)), another popular mocking framework. The engineers at Tested Data Systems want to evaluate it and eventually introduce it into their projects.

To work with Mockito, you need to add to the pom.xml file the dependency shown in listing 8.24.

```xml
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-junit-jupiter</artifactId>
    <version>2.21.0</version>
    <scope>test</scope>
</dependency>
```

```java
@ExtendWith(MockitoExtension.class)
public class TestWebClientMockito {
    @Mock
    private ConnectionFactory factory;

    @Mock
    private InputStream mockStream;

    @Test
    public void testGetContentOk() throws Exception {
        when(factory.getData()).thenReturn(mockStream);
        when(mockStream.read()).thenReturn((int) 'W')
                .thenReturn((int) 'o')
                .thenReturn((int) 'r')
                .thenReturn((int) 'k')
                .thenReturn((int) 's')
                .thenReturn((int) '!')
                .thenReturn(-1);

        WebClient2 client = new WebClient2();

        String workingContent = client.getContent(factory);

        assertEquals("Works!", workingContent);
    }

    @Test
    public void testGetContentCannotCloseInputStream()
            throws Exception {
        when(factory.getData()).thenReturn(mockStream);
        when(mockStream.read()).thenReturn(-1);
        doThrow(new IOException("cannot close")).when(mockStream).close();

        WebClient2 client = new WebClient2();

        String workingContent = client.getContent(factory);

        assertNull(workingContent);
    }
}
```

# In-container testing

* Analyzing the limitations of mock objects
* Using in-container testing
* Evaluating and comparing stubs, mock objects, and in-container testing
* Working with Arquillian



​	This chapter examines one approach to unit-testing components in an application container: in-container unit testing, or integration testing. 

## Limitations of standard unit testing

​	Start with the example servlet in listing 9.1, which implements the `HttpServlet` method `isAuthenticated,` the method we want to unit-test. A *servlet* is a Java software application that extends the capabilities of a server. The example company Tested Data Systems uses servlets to develop web applications, one of which is an online shop that serves new customers. To access the online shop, users need to connect to the frontend interface. The online shop needs authentication mechanisms so that the client knows who is making the operations, and the Tested Data Systems engineers would like to test a method that verifies whether a user is authenticated.

```java
[…]
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;
 
public class SampleServlet extends HttpServlet {
   public boolean isAuthenticated(HttpServletRequest request) {
      HttpSession session = request.getSession(false);
      if (session == null) {
         return false;
      }
      String authenticationAttribute =
         (String) session.getAttribute("authenticated");
      return Boolean.valueOf(authenticationAttribute).booleanValue();
   }
}
```

​	This servlet, although it is simple enough, allows us to show the limitation of standard unit testing. To test the method`isAuthenticated`, we need a valid `HttpServletRequest`. Because `HttpServletRequest` is an interface, we cannot just call`new HttpServletRequest`. The `HttpServletRequest` life cycle and implementation are provided by the container (in this case, a servlet container). The same is true of other server-side objects, such as `HttpSession.` JUnit alone is not enough to write a test for the i`sAuthenticated` method and for servlets in general.

## The mock-objects solution

​	The engineers at Tested Data Systems have to test the authentication mechanisms of the online shop. The first solution they consider for unit-testing the `isAuthenticated` method (listing 9.1) is to mock the `HttpServletRequest` class, using the approach described in chapter 8. Although mocking works, we need to write a lot of code to create a test. We can achieve the same result more easily by using the open-source EasyMock[***\*[1\]\****](#id_ftn1) framework (see chapter 8), as listing 9.2 demonstrates.

```java
[…]
import javax.servlet.http.HttpServletRequest;                           #A
import static org.easymock.EasyMock.createStrictMock;                   #A
import static org.easymock.EasyMock.expect;                             #A
import static org.easymock.EasyMock.replay;                             #A
import static org.easymock.EasyMock.verify;                             #A
import static org.easymock.EasyMock.eq;                                 #A
import static org.junit.jupiter.api.Assertions.assertFalse;             #A
import static org.junit.jupiter.api.Assertions.assertTrue;              #A
[…]
public class EasyMockSampleServletTest {
    
   private SampleServlet servlet;
   private HttpServletRequest mockHttpServletRequest;                   #B
   private HttpSession mockHttpSession;                                 #B
      
   @BeforeEach
   public void setUp() {                                                #C
      servlet = new SampleServlet();
      mockHttpServletRequest = 
         createStrictMock(HttpServletRequest.class);                    #C
      mockHttpSession = createStrictMock(HttpSession.class);            #C
   }
 
   @Test
   public void testIsAuthenticatedAuthenticated() { 
      expect(mockHttpServletRequest.getSession(eq(false)))              #D
         .andReturn(mockHttpSession);                                   #D
      expect(mockHttpSession.getAttribute(eq("authenticated")))         #D
         .andReturn("true");                                            #D
      replay(mockHttpServletRequest);                                   #E
      replay(mockHttpSession);                                          #E
      assertTrue(servlet.isAuthenticated(mockHttpServletRequest));      #F
   }
 
   @Test
   public void testIsAuthenticatedNotAuthenticated() {
      expect(mockHttpSession.getAttribute(eq("authenticated")))
         .andReturn("false");          
      replay(mockHttpSession);
      expect(mockHttpServletRequest.getSession(eq(false)))
         .andReturn(mockHttpSession);
      replay(mockHttpServletRequest);
      assertFalse(servlet.isAuthenticated(mockHttpServletRequest));
   }
 
   @Test
   public void testIsAuthenticatedNoSession() {
      expect(mockHttpServletRequest.getSession(eq(false))).andReturn(null);
      replay(mockHttpServletRequest);
         replay(mockHttpSession);
         assertFalse(servlet.isAuthenticated(mockHttpServletRequest));
   }
 
   @AfterEach
   public void tearDown() {                                             #G
      verify(mockHttpServletRequest);                                   #H
      verify(mockHttpSession);                                          #H
   }
}
```

​	Mocking a minimal portion of a container is a valid approach for testing components. But mocking can be complicated and require a lot of code. The source code provided with this book also includes the servlet testing versions for the JMock and Mockito frameworks. As with other kinds of tests, when the servlet changes, the test expectations must change to match. In the next section, the engineers at Tested Data Systems attempt to ease the task of testing the online shop’s authentication mechanism.

## The step to in-container testing

​	The next approach in testing the `SampleServlet` is running the test cases where the `HttpServletRequest` and `HttpSession`objects live: in the container itself. This approach eliminates the need to mock any objects; we simply access the objects and methods we need in the real container.

​	For our example of testing the online shop’s authentication mechanism, we need the web request and the session to be real`HttpServletRequest` and `HttpSession` objects managed by the container. Using a mechanism to deploy and execute our tests in a container, we have in-container testing. The next section covers options for in-container tests.

### Implementation strategies

​	Two architectural choices drive in-container tests: server-side and client-side. As stated in section 9.3, we can drive the tests directly by controlling the server-side container and the unit tests. Alternatively, we can drive the tests from the client-side, as shown in figure 9.1.

![](https://pic.imgdb.cn/item/613090b344eaada739d1af8b.jpg)

### In-container testing frameworks

​	As we have just seen, in-container testing is applicable when code interacts with a container, and tests cannot create valid container objects (`HttpServletRequest` in the preceding section).

​	Our example uses a servlet container, but many other types of containers are available, including Java EE, web server, applets, and EJB. In all these cases, the in-container testing strategy can be applied.

## Comparing stubs, mock objects, and in-container testing

### Stubs evaluation

***Pros:***

·  Fast and lightweight

·  Easy to write and understand

·  Powerful

·  More coarse-grained tests

***Cons:***

·  Specialized methods required to verify the state

·  Do not test the behavior of faked objects

·  Time-consuming for complicated interactions

·  Require more maintenance when the code changes

### Mock-objects evaluation

***Advantages:***

·  Do not require a running container to execute tests

·  Are quick to set up and run

·  Allow fine-grained unit testing

***Drawbacks:***

·  Do not test interactions with the container or between the components

·  Do not test the deployment of components

·  Require good knowledge of the API to mock, which can be difficult (especially for external libraries)

·  Do not provide confidence that the code will run in the target container

·  Offer more fine-grained testing, which may lead to testing code being swamped with interfaces

·  Like stubs, require maintenance when the code changes

###  In-container testing evaluation

* *SPECIFIC TOOLS REQUIRED*
* *NO GOOD IDE SUPPORT*
* *LONGER EXECUTION TIME*
* *COMPLEX CONFIGURATION*

## Testing with Arquillian

​	Arquillian is a testing framework for Java that leverages JUnit to execute test cases against a Java container. The Arquillian framework is broken \into three major sections:

```java
public class Passenger {

    private String identifier;
    private String name;

    public Passenger(String identifier, String name) {
        this.identifier = identifier;
        this.name = name;
    }

    public String getIdentifier() {
        return identifier;
    }

    public String getName() {
        return name;
    }

    @Override
    public String toString() {
        return "Passenger " + getName() + " with identifier: " + getIdentifier();
    }
}
```

```java
public class Flight {

    private String flightNumber;
    private int seats;
    Set<Passenger> passengers = new HashSet<>();

    public Flight(String flightNumber, int seats) {
        this.flightNumber = flightNumber;
        this.seats = seats;
    }

    public String getFlightNumber() {
        return flightNumber;
    }

    public int getSeats() {
        return seats;
    }

    public void setSeats(int seats) {
        if (passengers.size() > seats) {
            throw new RuntimeException("Cannot reduce seats under the number of existing passengers!");
        }
        this.seats = seats;
    }

    public int getNumberOfPassengers() {
        return passengers.size();
    }

    public boolean addPassenger(Passenger passenger) {
        if (passengers.size() >= seats) {
            throw new RuntimeException("Cannot add more passengers than the capacity of the flight!");
        }
        return passengers.add(passenger);
    }

    public boolean removePassenger(Passenger passenger) {
        return passengers.remove(passenger);
    }

    @Override
    public String toString() {
        return "Flight " + getFlightNumber();
    }

}
```

The flights_information.csv file

```
1236789; John Smith
9006789; Jane Underwood
1236790; James Perkins
9006790; Mary Calderon
1236791; Noah Graves
9006791; Jake Chavez
1236792; Oliver Aguilar
9006792; Emma McCann
1236793; Margaret Knight
9006793; Amelia Curry
1236794; Jack Vaughn
9006794; Liam Lewis
1236795; Olivia Reyes
9006795; Samantha Poole
1236796; Patricia Jordan
9006796; Robert Sherman
1236797; Mason Burton
9006797; Harry Christensen
1236798; Jennifer Mills
9006798; Sophia Graham
```

```java
public class FlightBuilderUtil {

    public static Flight buildFlightFromCsv() throws IOException {
        Flight flight = new Flight("AA1234", 20);
        try (BufferedReader reader = new BufferedReader(new FileReader("src/test/resources/flights_information.csv"))) {
            String line = null;
            do {
                line = reader.readLine();
                if (line != null) {
                    String[] passengerString = line.toString().split(";");
                    Passenger passenger = new Passenger(passengerString[0].trim(), passengerString[1].trim());
                    flight.addPassenger(passenger);
                }
            } while (line != null);

        }

        return flight;
    }
}
```

```java
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.jboss.arquillian</groupId>
            <artifactId>arquillian-bom</artifactId>
            <version>1.4.0.Final</version>
            <scope>import</scope>
            <type>pom</type>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.jboss.spec</groupId>
        <artifactId>jboss-javaee-7.0</artifactId>
        <version>1.0.3.Final</version>
        <type>pom</type>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-api</artifactId>
        <version>5.6.0</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-engine</artifactId>
        <version>5.6.0</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.junit.vintage</groupId>
        <artifactId>junit-vintage-engine</artifactId>
        <version>5.6.0</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>servlet-api</artifactId>
        <version>2.5</version>
    </dependency>
    <dependency>
        <groupId>org.easymock</groupId>
        <artifactId>easymock</artifactId>
        <version>2.4</version>
    </dependency>
    <dependency>
        <groupId>org.jmock</groupId>
        <artifactId>jmock-junit5</artifactId>
        <version>2.12.0</version>
    </dependency>
    <dependency>
        <groupId>org.mockito</groupId>
        <artifactId>mockito-junit-jupiter</artifactId>
        <version>2.21.0</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.jboss.arquillian.junit</groupId>
        <artifactId>arquillian-junit-container</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.jboss.arquillian.container</groupId>
        <artifactId>arquillian-weld-ee-embedded-1.1</artifactId>
        <version>1.0.0.CR9</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.jboss.weld</groupId>
        <artifactId>weld-core</artifactId>
        <version>2.3.5.Final</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

```java
@RunWith(Arquillian.class)
public class FlightWithPassengersTest {

    @Deployment
    public static JavaArchive createDeployment() {
        return ShrinkWrap.create(JavaArchive.class)
                .addClasses(Passenger.class, Flight.class, FlightProducer.class)
                .addAsManifestResource(EmptyAsset.INSTANCE, "beans.xml");
    }

    @Inject
    Flight flight;

    @Test(expected = RuntimeException.class)
    public void testNumberOfSeatsCannotBeExceeded() throws IOException {
        assertEquals(20, flight.getNumberOfPassengers());
        flight.addPassenger(new Passenger("1247890", "Michael Johnson"));
    }

    @Test
    public void testAddRemovePassengers() throws IOException {
        flight.setSeats(21);
        Passenger additionalPassenger = new Passenger("1247890", "Michael Johnson");
        flight.addPassenger(additionalPassenger);
        assertEquals(21, flight.getNumberOfPassengers());
        flight.removePassenger(additionalPassenger);
        assertEquals(20, flight.getNumberOfPassengers());
        assertEquals(21, flight.getSeats());
    }
}
```

·  A @RunWith(Arquillian.class) annotation on the class (#A). The @RunWith annotation tells JUnit to use Arquillian as the test controller.

·  A public static method annotated with @Deployment that returns a ShrinkWrap archive (#B). The purpose of the test archive is to isolate the classes and resources that the test needs. The archive is defined with ShrinkWrap. The micro deployment strategy lets us focus on precisely the classes we want to test. As a result, the test remains very lean and manageable. For the moment, we have included only the Passenger and the Flight classes. We try to inject a Flight object as a class member, using the CDI @Inject annotation (#C). The @Inject annotation allows us to define injection points inside classes. In this case, @Inject instructs CDI to inject into the test a field of type reference to a Flight object.

·  At least one method annotated with @Test (#D and #E). Arquillian looks for a public static method annotated with the @Deployment annotation to retrieve the test archive. Then each @Test annotated method is run inside the container environment.



​	The error says `Unsatisfied dependencies for type Flight with qualifiers @Default`. It means that the container is trying to inject the dependency, as it has been instructed through the CDI `@Inject` annotation, but it is unsatisfied. Why? What have the developers from Tested Data Systems missed? The `Flight` class provides only a constructor with arguments, and it has no default constructor to be used by the container for the creation of the object. The container does not know how to invoke the constructor with parameters and which parameters to pass to it to create the `Flight` object that must be injected.

​	What is the solution in this case? Java EE offers the producer methods that are designed to inject objects that require custom initialization (listing 9.9). The solution fixes the issue and is easy to put into practice, even by a junior developer.

```java
public class FlightProducer {
 
    @Produces
    public Flight createFlight() throws IOException {
        return FlightBuilderUtil.buildFlightFromCsv();
    }
}
```

# 10. Running JUnit tests from Maven 3

* Creating a Maven project from the scratch
* Testing the Maven project with JUnit 5
* Using Maven plugins
* Using the Maven Surefire Plugins

## 10.1 Setting up a Maven project

​	First, create the `C:\junitbook\` folder. This directory is our work directory and where we will set up the Maven examples. Type the following on the command line:

```sh
mvn archetype:generate -DgroupId=com.manning.junitbook 
-DartifactId=maven-sampling 
-DarchetypeArtifactid=maven-artifact-mojo
```

​	After we press Enter, wait for the appropriate artifacts to be downloaded, and accept the default options, we should see a folder named `maven-sampling` being created. If we open the new project into IntelliJ IDEA, its structure should look like figure 10.1.

![](https://pic.imgdb.cn/item/6135d77144eaada739a295dd.jpg)

​	What happened here? We invoked the `maven-archetype-plugin` from the command line and told it to generate a new project from scratch with the given parameters. As a result, this Maven plugin created a new project with a new folder structure, following the convention of the folder structure. Further, it created a sample `App.java` class with the main method and a corresponding `AppTest.java` file that is a unit test for our application. After looking at this folder structure, you should understand what files stay in `src/main/java` and what files stay in `src/test/java.`

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" 
         xmlns:xsi=http://www.w3.org/2001/XMLSchema-instance
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/maven-v4_0_0.xsd">
  
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.manning.junitbook</groupId>
  <artifactId>maven-sampling</artifactId>
  <version>1.0-SNAPSHOT</version>
  <name>maven-sampling</name>
    <!-- FIXME change it to the project's website -->
  <url>http://www.example.com</url>
  […]
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
  […]
</project>
```

**Listing 10.2 Changes and additions to the pom.xml**

```xml
      <dependencies>
         <dependency>
            <groupId>org.junit.jupiter</groupId>
         <artifactId>junit-jupiter-api</artifactId>
         <version>5.4.2</version>
         <scope>test</scope>
      </dependency>
   <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter-engine</artifactId>
      <version>5.4.2</version>
      <scope>test</scope>
   </dependency>
   </dependencies>
 
<developers>
      <developer>
       <name>Catalin Tudose</name>
       <id>ctudose</id>
       <organization>Manning</organization>
       <roles>
          <role>Java Developer</role>
       </roles>
</developer>
<developer>
      <name>Petar Tahchiev</name>
     <id>ptahchiev</id>
     <organization>Apache Software Foundation</organization>
     <roles>
        <role>Java Developer</role>
     </roles>    
</developer>
</developers>
```

We also specify `organization`, `description` and `inceptionYear`:

```xml
<description>
   “JUnit in Action III” book, the sample project for the “Running Junit   
   tests from Maven” chapter.
</description>
<organization>
   <name>Manning Publications</name>
   <url>http://manning.com/</url>
</organization>
<inceptionYear>2019</inceptionYear>
```

​	Now we can start developing our software. What if we want to use a Java IDE other than IntelliJ IDEA, such as Eclipse? No problem. Maven offers additional plugins that let us import the project into our favorite IDE. If we want to use Eclipse, we open a terminal and navigate to the directory that contains the project descriptor (pom.xml). Then we type the following and press Enter:

```sh
mvn eclipse:eclipse
```

## 10.2 Using the Maven plugins

​	Whenever we would like to clean the project from the previous activities, we may execute the following command:

```sh
mvn clean
```

### 10.2.1 Maven compiler plugin

​	Like any other build system, Maven is supposed to build our projects (compile our software and package in an archive). Every task in Maven is performed by an appropriate plugin, the configuration of which is in the `<plugins>` section of the project descriptor. To compile the source code, all we need to do is invoke the compile phase on the command line

```sh
mvn compile
```

​	which causes Maven to execute all the plugins attached to *the compile* phase (in particular it will invoke the `maven-compiler-plugin`). But before invoking the compile phase, as already discussed, Maven goes through the validate phase, downloads all the dependencies listed in the pom.xml file, and includes them in the classpath of the project. When the compilation process is complete, we can go to the `target/classes/` folder and see the compiled classes there.

**Configuring the maven-compiler-plugin**

```xml
<build>
   <plugins>
      <plugin>
        <artifactId>maven-compiler-plugin</artifactId>
         <version>2.3.2</version>
        <configuration>
           <source>1.8</source>
           <target>1.8</target>
        </configuration>
     </plugin>
   </plugins>
</build>
```

### 10.2.2  Maven surefire plugin

​	To process the unit tests from our project, Maven uses (of course) a plugin. The Maven plugin that executes the unit tests is called `maven-surefire-plugin`. The surefire plugin executes the unit tests for our code, but these unit tests are not necessarily JUnit tests.

**The maven-surefire-plugin**

```xml
<build>
   <plugins>
      <plugin>
        <artifactId>maven-surefire-plugin</artifactId>
         <version>2.22.2</version>
     </plugin>
   </plugins>
</build>
```

​	The conventional way to start the `maven-surefire-plugin` is very simple: invoke the test phase of Maven. This way, Maven first invokes all the phases that are supposed to come before the test phase (validate and compile) and then invokes all the plugins that are attached to the test phase, this way invoking the `maven-surefire-plugin`. So by calling

```xml
mvn clean test
```

​	That’s great, but what if we want to execute only a single test case? This execution is unconventional*,* so we need to configure the `maven-surefire-plugin` to do it. Ideally, a parameter for the plugin allows us to specify the pattern of test cases that we want to execute. We configure the surefire plugin in absolutely the same way that we configure the compiler plugin, as shown in listing 10.6.

```xml
<build>
   <plugins>
   […]
      <plugin>
          <artifactId>maven-surefire-plugin</artifactId>
         <version>2.22.2</version>
          <configuration>
             <includes>**/*Test.java</includes>
          </configuration>
        […]
      </plugin>
   […]
   </plugins>
</build>
```

​	The next step is generating some documentation for the project. But wait for a second—how are we supposed to do that with no files to generate the documentation from? This is another one of Maven’s great benefits: with a little bit of configuration and description that we have, we can produce a fully functional website skeleton.

​	First, add the `maven-site-plugin` to the Maven `pom.xml` configuration file:

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-site-plugin</artifactId>
    <version>3.7.1</version>
</plugin>
```

Then type

```
mvn site
```

​	on the command line where our pom.xml file is. Maven should start downloading its plugins, and after their successful installation, it produces the nice website shown in figure 10.4.

### 10.2.3  HTML JUnit reports with Maven

​	As you already guess, the job of producing these reports is done by a Maven plugin. The name of the plugin is `maven-surefire-report-plugin`, and by default, it is not attached to any of the core phases that we already know. (Many people do not need HTML reports every time they build software.) We can’t invoke the plugin by running a certain phase (as we did with both the compiler plugin and the surefire plugin). Instead, we have to call it directly from the command line:

```xml
mvn surefire-report:report
```

​	Maven tries to compile the source files and the test cases; then it invokes the `surefire` plugin to produce the plain-text and XML-formatted output of the tests. After that, the `surefire-report` plugin tries to transform all the XMLs from the`target/surefire-reports/` directory into an HTML report that will be placed in the `target/site` directory. (Remember that this is the convention for the folder—to keep all the generated documentation of the project—and that the HTML reports are considered to be documentation.)

​	If we try to open the generated HTML report, it should look something like figure 10.5.

![](https://pic.imgdb.cn/item/6135da8f44eaada739a79bdb.jpg)

## 10.3 Putting it all together

略

## 10.4  Maven challenges

​	A lot of people who have used Maven agree that it is really easy to start and that the idea behind the project is amazing. Things seem to be more challenging when we need to do something unconventional, however.

​	What is great about Maven is that it sets up a frame for us and constrains us to think inside that frame—to think the Maven way and do things the Maven way.

​	In most cases, Maven will not let us execute any nonsense. It restricts us and shows us the way things need to be done. But these restrictions may be a real challenge if we are accustomed to doing things our own way and to having real freedom of choice as build engineers.

​	Chapter 11 shows how to run JUnit tests with another build automation tool inspired by Maven: Gradle.

# 12. JUnit 5 IDE support

​	When running the whole tests suite from IntelliJ IDEA, the tagged tests are also run. If you would like to run only some particular tagged tests, the IDE will provide this option. You have to go to *Run -> Edit Configurations…* and to choose from *Test kind -> Tags*, as shown in figure 12.9. Then, you will be able to run only that particular configuration addressing tagged tests. You would like to do this when you have a larger suite of tests and you need to focus on particular ones and not to spend time with the whole suite. Details about the functionality of the `@Tag` annotation are to be found in chapter 2.

![](https://pic.imgdb.cn/item/6135db5544eaada739a8e8ca.jpg)

# 14. JUnit 5 extension model

* Introducing the JUnit 5 extension model
* Creating JUnit 5 extensions
* Implementing JUnit 5 tests using available extension points
* Developing an application with tests extended by JUnit 5 extensions

## 14.1 Introducing the JUnit 5 extension model

Although JUnit 4 provides extension points through runners and rules (chapter 3), the JUnit 5 extension model consists of a single concept: the `Extension` API. `Extension` itself is just a *marker interface* (or *tag* or *token interface*)—an interface with no fields or methods inside. It is used to mark the fact that the class implementing an interface of this category has some special behavior. Among the best-known, Java marker interfaces are `Serializable` and `Cloneable`.

JUnit 5 can extend the behavior of classes or methods of test and these extensions can be reused by many tests.

A JUnit 5 extension is connected to the occurrence of some particular event during the execution of a test. This kind of event is called an extension point. At the moment when such a point in the lifecycle of a test has been reached, the JUnit engine automatically calls the registered extension.

The list of available extension points is made of:

*  *Conditional test execution*—Controls whether a test should be run
*  *Life-cycle callbacks*— Reacts to events in the life cycle of a test
* *Parameter resolution*—Resolves at run time the parameter received by a test
* *Exception handling*—Defines the behavior of a test when it encounters certain types of exceptions
*  *Test instance postprocessing*—Executed after an instance of a test is created

Please note that the extensions are largely used inside frameworks and build tools. They may also be used for application programming, but not to the same extent. The creation and usage of extensions follow common principles, our demonstration will use examples appropriate for regular applications development.

## 14.2  Creating the first JUnit 5 extension

```java
public class Passenger {
 
   private String identifier;                                            #A
   private String name;                                                  #A
 
   public Passenger(String identifier, String name) {                    #B
      this.identifier = identifier;                                      #B
      this.name = name;                                                  #B
   }                                                                     #B
 
   public String getIdentifier() {                                       #C
      return identifier;                                                 #C
   }                                                                     #C
 
   public String getName() {                                             #C
      return name;                                                       #C
   }                                                                     #C
 
 
   @Override                                                             #D
   public String toString() {                                            #D
      return "Passenger " + getName() + " with identifier: " +           #D
             getIdentifier();                                            #D
   }                                                                     #D
}
```

```java
public class PassengerTest {
 
    @Test                                                                  
    void testPassenger() throws IOException {                              
        Passenger passenger = new Passenger("123-456-789", "John Smith");  
        assertEquals("Passenger John Smith with identifier: 123-456-789",  
                     passenger.toString());                                
    }                                                                      
 
}
```

**The ExecutionContextExtension class**

```java
public class ExecutionContextExtension implements ExecutionCondition {   #A
 
    @Override                                                            #B
    public ConditionEvaluationResult                                     #B
           evaluateExecutionCondition(ExtensionContext context) {        #B
        Properties properties = new Properties();                        #C
        String executionContext = "";                                    #C
 
        try {
            properties.load(ExecutionContextExtension.class              
                                                     .getClassLoader()   #C
                      .getResourceAsStream("context.properties"));       #C
            executionContext = properties.getProperty("context");        #C
            if (!"regular".equalsIgnoreCase(executionContext) &&         #D
                !"low".equalsIgnoreCase(executionContext)) {             #D
                return ConditionEvaluationResult.disabled(               #D
    "Test disabled outside regular and low contexts");                   #D
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
        return ConditionEvaluationResult.enabled("Test enabled on the "+ #E
                                    executionContext + " context");      #E
    }
}
```

The context is configured through the resources/context.properties configuration file:

```
  context =regular 
```

For the current business logic, the `"regular"` value means that the tests will be executed in the current context.

The last thing to do is annotate the existing `PassengerTest` with the new extension:

```java
@ExtendWith({ExecutionContextExtension.class})
public class PassengerTest {
[…]
```

![](https://pic.imgdb.cn/item/6135dcda44eaada739ab38e8.jpg)

​	We may instruct the JVM (Java Virtual Machine) to bypass the effects of conditional execution, however, deactivating it by setting the `junit.jupiter.conditions.deactivate` configuration key to a pattern that matches the condition. From the Run -> Edit Configurations menu, we may set `junit.jupiter.conditions.deactivate=*`, for example, and the result will be the deactivation of all conditions (figure 14.2). The result of the execution will not be influenced by any of the conditions, so all tests will run.

![](https://pic.imgdb.cn/item/6135dd3b44eaada739abdaff.jpg)

## 14.3  Writing JUnit 5 tests using the available extension points

​	Harry’s next task is saving the passengers in a test database. Before the whole test suite is executed, the database must be reinitialized, and a connection to it must be open. At the end of the execution of the suite, the connection to the database must be closed. Before executing a test, we have to set up the database in a known state so we can be sure that its content will be tested correctly. Harry decides to use the H2 database, JDBC, and JUnit 5 extensions.

​	H2 is a relational database management system developed in Java that permits the creation of an in-memory database. It can also be embedded in Java applications when testing purposes require this.

​	JDBC (Java Database Connectivity) is a Java API that defines how a client accesses a database. JDBC is part of the Java Standard Edition platform.

​	To solve the task, the first thing Harry needs to do is add the H2 dependency to the pom.xml file, as shown in listing 14.4.

```mysql
<dependency>
  <groupId>com.h2database</groupId>
  <artifactId>h2</artifactId>
  <version>1.4.199</version>
</dependency>
```

To manage the connection to the database, Harry implements the `ConnectionManager` class (listing 14.5).

```java
public class ConnectionManager {
   private static Connection connection;                                 #A
 
   public static Connection getConnection() {                            #A
       return connection;                                                #A
    }                                                                    #A
   
   public static Connection openConnection() {
        try {
            Class.forName("org.h2.Driver"); // this is driver for H2     #B
            connection = DriverManager.getConnection("jdbc:h2:~/book",   #C
                "sa", // login                                           #C
                "" // password                                           #C
                );                                                       #C
            return connection;
        } catch(ClassNotFoundException | SQLException e) {
            throw new RuntimeException(e);
        }
    }
 
    public static void closeConnection() {
       if (null != connection) {
           try {
               connection.close();                                       #D
           } catch(SQLException e) {
               throw new RuntimeException(e);
           }
       }
    }
 
}
```

To manage the database tables, Harry implements the `TablesManager` class (listing 14.6).

```java
public class TablesManager {
   
    public static void createTable(Connection connection) {              #A
       String sql =                                                      #A
          "CREATE TABLE IF NOT EXISTS PASSENGERS (ID VARCHAR(50), " +    #A
          "NAME VARCHAR(50));";                                          #A
       
       executeStatement(connection, sql);                                #A
    }                                                                    #A
 
    public static void dropTable(Connection connection) {                #B
       String sql = "DROP TABLE IF EXISTS PASSENGERS;";                  #B
       
       executeStatement(connection, sql);                                #B
    }                                                                    #B
    
    private static void executeStatement(Connection connection,          #C
                                         String sql)                     #C
    {                                                                    #C
       try(PreparedStatement statement =                                 #C
           connection.prepareStatement(sql))                             #C
       {                                                                 #C
            statement.executeUpdate();                                   #C
       } catch (SQLException e) {                                        #C
            throw new RuntimeException(e);                               #C
       }                                                                 #C
    }                                                                    #C
 
}
```

​	To manage the execution of the queries against the database, Harry implements the `PassengerDao` interface (listing 14.7) and the `PassengerDaoImpl` class (listing 14.8). A *DAO* (data access object) provides an interface to a database and maps the application calls to specific database operations without exposing the details of the persistence layer.

```java
public interface PassengerDao {
   public void insert(Passenger passenger);                              #A
   public void update(String id, String name);                           #B
   public void delete(Passenger passenger);                              #C
   public Passenger getById(String id);                                  #D
}
```

**The PassengerDaoImpl class**

```java
public class PassengerDaoImpl implements PassengerDao {
 
    private Connection connection;                                         #A
 
    public PassengerDaoImpl(Connection connection) {                       #A
       this.connection = connection;                                       #A
    }                                                                      #A
 
    @Override
    public void insert(Passenger passenger) {
        String sql = "INSERT INTO PASSENGERS (ID, NAME) VALUES (?, ?)";    #B
 
        try (PreparedStatement statement = connection.prepareStatement(sql)){
            statement.setString(1, passenger.getIdentifier());             #C
            statement.setString(2, passenger.getName());                   #C
            statement.executeUpdate();                                     #D
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }
 
    @Override
    public void update(String id, String name) {
        String sql = "UPDATE PASSENGERS SET NAME = ? WHERE ID = ?";        #E
 
        try (PreparedStatement statement = connection.prepareStatement(sql)){
            statement.setString(1, name);                                  #F
            statement.setString(2, id);                                    #F
            statement.executeUpdate();                                     #G
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }
 
    @Override
    public void delete(Passenger passenger) {
        String sql = "DELETE FROM PASSENGERS WHERE ID = ?";                #H
 
        try (PreparedStatement statement = connection.prepareStatement(sql)){
            statement.setString(1, passenger.getIdentifier());             #I
            statement.executeUpdate();                                     #J
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }
 
    @Override
    public Passenger getById(String id) {
        String sql = "SELECT * FROM PASSENGERS WHERE ID = ?";              #K
        Passenger passenger = null;
 
        try (PreparedStatement statement = connection.prepareStatement(sql)){
            statement.setString(1, id);                                    #L
            ResultSet resultSet = statement.executeQuery();                #M
 
            if (resultSet.next()) {
                passenger = new Passenger(resultSet.getString(1),          #N
                                          resultSet.getString(2));         #N
            }
 
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
 
        return passenger;                                                  #O
    }
}
```

**The DatabaseOperationsExtension class**

```java
public class DatabaseOperationsExtension implements                      #A
             BeforeAllCallback, AfterAllCallback, BeforeEachCallback,    #A
             AfterEachCallback {                                         #A
 
    private Connection connection;                                       #B
    private Savepoint savepoint;                                         #B
 
    @Override                                                             
    public void beforeAll(ExtensionContext context) {                    
        connection = ConnectionManager.openConnection();                 #C
        TablesManager.dropTable(connection);                             #C
        TablesManager.createTable(connection);                           #C
    }                                                                     
 
    @Override                                                             
    public void afterAll(ExtensionContext context) {                      
        ConnectionManager.closeConnection();                             #D
    }                                                                     
 
    @Override                                                             
    public void beforeEach(ExtensionContext context) 
                           throws SQLException {
        connection.setAutoCommit(false);                                 #E
        savepoint = connection.setSavepoint("savepoint");                #F
    }                                                                     
 
    @Override                                                             
    public void afterEach(ExtensionContext context) 
                           throws SQLException { 
        connection.rollback(savepoint);                                  #G
    }                                                                     
 
}
```

**The updated PassengerTest class**

```java
@ExtendWith({ExecutionContextExtension.class,                            #A
             DatabaseOperationsExtension.class })                        #A
public class PassengerTest {
 
    private PassengerDao passengerDao;
 
    public PassengerTest(PassengerDao passengerDao) {                      
        this.passengerDao = passengerDao;                                #B
    }
 
    @Test
    void testPassenger(){
        Passenger passenger = new Passenger("123-456-789", "John Smith");
        assertEquals("Passenger John Smith with identifier: 123-456-789", 
                     passenger.toString());
    }
 
 
    @Test
    void testInsertPassenger() {
        Passenger passenger = new Passenger("123-456-789",               #C
                                           "John Smith");                #C
        passengerDao.insert(passenger);                                  #D
        assertEquals("John Smith",                                       #E
              passengerDao.getById("123-456-789").getName());            #E
    }
 
    @Test
    void testUpdatePassenger() {
        Passenger passenger = new Passenger("123-456-789",               #F
                                            "John Smith");               #F
        passengerDao.insert(passenger);                                  #G
        passengerDao.update("123-456-789", "Michael Smith");             #H
        assertEquals("Michael Smith",                                    #I
              passengerDao.getById("123-456-789").getName());            #I
    }
 
    @Test
    void testDeletePassenger() {
        Passenger passenger = new Passenger("123-456-789",               #J
                                            "John Smith");               #J
        passengerDao.insert(passenger);                                  #K
        passengerDao.delete(passenger);                                  #L
        assertNull(passengerDao.getById("123-456-789"));                 #M
    }
 
}
```

![](https://pic.imgdb.cn/item/6135de0544eaada739ad27b0.jpg)

**The DatabaseAccessObjectParameterResolver class**

```java
public class DatabaseAccessObjectParameterResolver implements            #A
          ParameterResolver{                                             #A
 
    @Override
    public boolean supportsParameter(ParameterContext parameterContext, 
                                     ExtensionContext extensionContext) 
           throws ParameterResolutionException {
        return parameterContext.getParameter()                           #B
            .getType()                                                   #B
            .equals(PassengerDao.class);                                 #B
    }
 
    @Override
    public Object resolveParameter(ParameterContext parameterContext, 
                                   ExtensionContext extensionContext) 
           throws ParameterResolutionException {
        return new PassengerDaoImpl(ConnectionManager.getConnection());  #C
    }
 
}
```

Additionally, Harry extends the `PassengerTest` class with the `DatabaseAccessObjectParameterResolver`. The first lines of the `PassengerTest` class look like listing 14.12.

```java
@ExtendWith({ExecutionContextExtension.class, DatabaseOperationsExtension.class, DatabaseAccessObjectParameterResolver.class})
public class PassengerTest {
[…]
```

![](https://pic.imgdb.cn/item/6135de3e44eaada739ad82be.jpg)

```java
public class PassengerExistsException extends Exception {
    private Passenger passenger;                                         #A
 
    public PassengerExistsException(Passenger passenger, String message) {
        super(message);                                                  #B
        this.passenger = passenger;                                      #C
    }
}
```

Next, Harry changes the `PassengerDao` interface and the `PassengerDaoImpl` class so that the insert method

```java
public void insert(Passenger passenger) throws PassengerExistsException;
```

hrows PassengerExistsException, as shown in listing 14.14.

Listing 14.14 The modified insert method from PassengerDaoImpl

```java
public void insert(Passenger passenger) throws PassengerExistsException {
    String sql = "INSERT INTO PASSENGERS (ID, NAME) VALUES (?, ?)";
 
    if (null != getById(passenger.getIdentifier()) ) {                   #A
      throw new PassengerExistsException                                 #A
               (passenger, passenger.toString());                        #A
    }                                                                    #A
 
    try (PreparedStatement statement = connection.prepareStatement(sql)){
        statement.setString(1, passenger.getIdentifier());
        statement.setString(2, passenger.getName());
        statement.executeUpdate();
    } catch (SQLException e) {
        throw new RuntimeException(e);
    }
}
```

**The LogPassengerExistsExceptionExtension class**

```java
public class LogPassengerExistsExceptionExtension implements             #A
             TestExecutionExceptionHandler {                             #A
    private Logger logger = Logger.getLogger(this.getClass().getName()); #B
 
    @Override
    public void handleTestExecutionException(ExtensionContext context,   #C
                   Throwable throwable) throws Throwable {               #C
        if (throwable instanceof PassengerExistsException) {             #D
            logger.severe("Passenger exists:" + throwable.getMessage()); #D
            return;                                                      #D
        }
        throw throwable;                                                 #E
    }
}
```

```java
@ExtendWith({ExecutionContextExtension.class, DatabaseOperationsExtension.class, DatabaseAccessObjectParameterResolver.class, LogPassengerExistsExceptionExtension.class})
public class PassengerTest {
[…]
```

Table 14.2 Extension points and corresponding interfaces

| Extension point              | Interfaces to implement                                      |
| ---------------------------- | ------------------------------------------------------------ |
| Conditional test execution   | ExecutionCondition                                           |
| Life-cycle callbacks         | BeforeAllCallback, AfterAllCalback, BeforeEachCallback, AfterEachCallback |
| Parameter resolution         | ParameterResolver                                            |
| Exception handling           | TestExecutionExceptionHandler                                |
| Test instance postprocessing | TestInstancePostProcessor                                    |

# 15. Presentation Layer Testing

  Introducing presentation layer testing

* Operating with HtmlUnit

* Developing HtmlUnit tests

* Operating with Selenium

* Developing Selenium tests

* Comparing HtmlUnit and Selenium

## 15.1  Choosing a Testing Framework

​	We will look at two free open source tools to implement presentation layer tests within JUnit 5: HtmlUnit and Selenium.

## 15.2  Introducing HtmlUnit

​	HtmlUnit is an open-source Java headless browser framework. It allows tests to imitate programmatically the user of a browser-based web application. HtmlUnit JUnit 5 tests do not display a user interface. In the remainder of this HtmlUnit section, when we talk about "testing with a web browser", it is with the understanding that we are really "testing by emulating a specific web browser".

### 15.2.1  A live example

```java
import com.gargoylesoftware.htmlunit.WebClient;
[…]
 
public abstract class ManagedWebClient {
    protected WebClient webClient;                              #A
 
    @BeforeEach                                                 #B
    public void setUp() {                                       #B
        webClient = new WebClient();                            #B
    }                                                           #B
 
    @AfterEach                                                  #C
    public void tearDown() {                                    #C
        webClient.close();                                      #C
    }                                                           #C
}
```

**Listing 15.2 Our first HtmlUnit example**

```java
public class HtmlUnitPageTest extends ManagedWebClient {
 
    @Test
    public void homePage() throws IOException {
      HtmlPage page = webClient.getPage("http://htmlunit.sourceforge.net"); #A
      assertEquals("HtmlUnit – Welcome to HtmlUnit", page.getTitleText());  #A
 
      String pageAsXml = page.asXml();                                      #B
      assertTrue(pageAsXml.contains("<div class=\"container-fluid\">"));    #B
 
      String pageAsText = page.asText();                                    #C
      assertTrue(pageAsText.contains(                                       #C
            "Support for the HTTP and HTTPS protocols"));                   #C
    }
 
    @Test
    public void testClassNav() throws IOException {
      HtmlPage mainPage = webClient.getPage(                                #D
            "http://htmlunit.sourceforge.net/apidocs/index.html");          #D
      HtmlPage packagePage = (HtmlPage)                                     #E
            mainPage.getFrameByName("packageFrame").getEnclosedPage();      #E
      HtmlListItem htmlListItem = (HtmlListItem)                            #F
            packagePage.getElementsByTagName("li").item(0);                 #F
      assertEquals("AboutURLConnection", htmlListItem.getTextContent());    #G
    }
}
```

## 15.3  Writing HtmlUnit tests

### 15.3.1  HTML Assertions

​	We know that JUnit 5 provides a class called `Assertions` to allow tests to fail when they detect an error condition. `Assertions`are the bread and butter of any unit test.

​	HtmlUnit may work with JUnit 5, but it also provides a class in the same spirit called `WebAssert`, which contains standard assertions for HTML like `assertTitleEquals`, `assertTextPresent` or `notNull`.

### 15.3.2  Testing for a specific web browser

**Table 15.1 HTMLUnit supported browsers.**

| Web Browser and Version                  | HtmlUnit BrowserVersion Constant |
| ---------------------------------------- | -------------------------------- |
| Internet Explorer 11                     | BrowserVersion.INTERNET_EXPLORER |
| Firefox 5.2 (deprecated)                 | BrowserVersion.FIREFOX_52        |
| Firefox 6.0                              | BrowserVersion.FIREFOX_60        |
| Latest Chrome                            | BrowserVersion.CHROME            |
| The best-supported browser at the moment | BrowserVersion.BEST_SUPPORTED    |

​	By default, `WebClient` emulates `BrowserVersion.BEST_SUPPORTED` which, at the time of writing this chapter, is Google Chrome, but may change in the future, depending on the evolution of each particular browser. In order to specify which browser to emulate, you provide the `WebClient` constructor with a `BrowserVersion`. For example, for Firefox 6.0, use:

```java
WebClient webClient = new WebClient(BrowserVersion.FIREFOX_60);
```

### 15.3.3  Testing more than one web browser

**Listing 15.3 Testing for all HtmlUnit supported browsers**

```java
public class JavadocPageAllBrowserTest {
 
    private static Collection<BrowserVersion[]> getBrowserVersions() {      #A
        return Arrays.asList(new BrowserVersion[][] {                       #A
                               { BrowserVersion.FIREFOX_60 },               #A
                               { BrowserVersion.INTERNET_EXPLORER },        #A
                               { BrowserVersion.CHROME },                   #A
                               { BrowserVersion.BEST_SUPPORTED } });        #A
    }
 
    @ParameterizedTest                                                      #B
    @MethodSource("getBrowserVersions")                                     #C
    public void testClassNav(BrowserVersion browserVersion)                 #D
                                            throws IOException              #D
    {
        WebClient webClient = new WebClient(browserVersion);                #E
 
        HtmlPage mainPage = (HtmlPage)webClient                             #F
                   .getPage(                                                #F
                   "http://htmlunit.sourceforge.net/apidocs/index.html");   #F
        WebAssert.notNull("Missing main page", mainPage);                   #G
 
        HtmlPage packagePage = (HtmlPage) mainPage                          #H
                       .getFrameByName("packageFrame").getEnclosedPage();   #H
        WebAssert.notNull("Missing package page", packagePage);             #I
 
        HtmlListItem htmlListItem = (HtmlListItem) packagePage              #J
                       .getElementsByTagName("li").item(0);                 #J
        assertEquals("AboutURLConnection", htmlListItem.getTextContent());  #K
    }
}
```

### 15.3.4  Creating stand-alone tests

**Listing 15.4 Configuring a stand-alone test**

```java
public class InLineHtmlFixtureTest extends ManagedWebClient {              
 
    @Test
    public void testInLineHtmlFixture() throws IOException {               
        final String expectedTitle = "Hello 1!";                           #A
        String html = "<html><head><title>" +                              #B
                      expectedTitle +                                      #B
                      "</title></head></html>";                            #B
        MockWebConnection connection = new MockWebConnection();            #C
        connection.setDefaultResponse(html);                               #D
        webClient.setWebConnection(connection);                            #E
        HtmlPage page = webClient.getPage("http://page");                  #F
        WebAssert.assertTitleEquals(page, expectedTitle);                  #G
    }
   
}
```

**Listing 15.5 Configuring a test with multiple page fixtures**

```java
    @Test
    public void testInLineHtmlFixtures() throws IOException {
        final URL page1Url = new URL("http://Page1/");                      #A
        final URL page2Url = new URL("http://Page2/");                      #A
        final URL page3Url = new URL("http://Page3/");                      #A
 
        MockWebConnection connection = new MockWebConnection();             #B
        connection.setResponse(page1Url, 
           "<html><head><title>Hello 1!</title></head></html>");            #C
        connection.setResponse(page2Url,   
                "<html><head><title>Hello 2!</title></head></html>");       #C
        connection.setResponse(page3Url, 
             "<html><head><title>Hello 3!</title></head></html>");          #C
        webClient.setWebConnection(connection);                             #D
 
        HtmlPage page1 = webClient.getPage(page1Url);                       #E
        WebAssert.assertTitleEquals(page1, "Hello 1!");                     #E
 
        HtmlPage page2 = webClient.getPage(page2Url);                       #E
        WebAssert.assertTitleEquals(page1, "Hello 2!");                     #E
 
        HtmlPage page3 = webClient.getPage(page3Url);                       #E
        WebAssert.assertTitleEquals(page1, "Hello 3!");                     #E
    }
```

### 15.3.5  Testing forms

**Listing 15.6 Example form page**

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<script>
function validate_form(form) {
    if (form.in_text.value=="") {
        alert("Please enter a value.");
        form.in_text.focus();
        return false;
    }
}
</script>
<title>Form</title></head>
<body>
<form name="validated_form" action="submit.html" onsubmit="return validate_form(this);" method="post">
  Value: 
  <input type="text" name="in_text" id="in_text" size="30"/>
  <input type="submit" value="Submit" id="submit"/>
</form>
</body>
</html>
```

**Listing 15.7 Testing a form**

```java
public class FormTest extends ManagedWebClient {
 
@Test
public void testForm() throws IOException {
   HtmlPage page = 
        webClient.getPage("file:src/main/webapp/formtest.html");    #A
   HtmlForm form = page.getFormByName("validated_form");            #B
   HtmlTextInput input = form.getInputByName("in_text");            #C
   input.setValueAttribute("typing...");                            #D
   HtmlSubmitInput submitButton = form.getInputByName("submit");    #E
   HtmlPage resultPage = submitButton.click();                      #E
   WebAssert.assertTitleEquals(resultPage, "Result");               #F
}
}
```

### 15.3.6  Testing JavaScript

​	HtmlUnit processes JavaScript automatically. Even when, for example, HTML is generated with `Document.write()`, you follow the usual pattern: call `getPage`, find an element, click on it, and check the result.

​	You can toggle JavaScript support on and off in a web client by calling:

```java
webClient.getOptions().setJavaScriptEnabled(true);
webClient.getOptions().setJavaScriptEnabled(false);
```

​	HtmlUnit enables JavaScript support by default. You can also set how long a script is allowed to run before being terminated by calling:

```java
webClient.setJavaScriptTimeout(timeout);
```

**Listing 15.8 Asserting expected alerts**

```java
public class FormTest extends ManagedWebClient {
[…]
 
@Test
public void testFormAlert() throws IOException {
    CollectingAlertHandler alertHandler =                         #A
            new CollectingAlertHandler();                         #A
    webClient.setAlertHandler(alertHandler);                      #B
    HtmlPage page = webClient.getPage(                            #C
             "file:src/main/webapp/formtest.html");               #C
    HtmlForm form = page.getFormByName("validated_form");         #D
    HtmlSubmitInput submitButton = form.getInputByName("submit"); #E
    HtmlPage resultPage = submitButton.click();                   #F
    WebAssert.assertTitleEquals(resultPage, page.getTitleText()); #G
    WebAssert.assertTextPresent(resultPage, page.asText());       #H
 
    List<String> collectedAlerts =                                #I
                 alertHandler.getCollectedAlerts();               #I
    List<String> expectedAlerts = Collections.singletonList(      #J
                                       "Please enter a value.");  #J
    assertEquals(expectedAlerts, collectedAlerts);                #K
}
}
```

**Listing 15.9 Asserting no alerts under normal operation**

```java
public class FormTest extends ManagedWebClient {
[…]
 
    @Test
    public void testFormNoAlert() throws IOException {
        CollectingAlertHandler alertHandler = new CollectingAlertHandler(); #A
        webClient.setAlertHandler(alertHandler);                            #A
        HtmlPage page = webClient.getPage(
                "file:src/main/webapp/formtest.html");
        HtmlForm form = page.getFormByName("validated_form");
        HtmlTextInput input = form.getInputByName("in_text");
        input.setValueAttribute("typing...");                               #B
        HtmlSubmitInput submitButton = form.getInputByName("submit");
        HtmlPage resultPage = submitButton.click();
        WebAssert.assertTitleEquals(resultPage, "Result");
        assertTrue(alertHandler.getCollectedAlerts().isEmpty(),             #C
                   "No alerts expected");                                   #C
    }
 
}
```

**Listing 15.10 Custom alert handler**

```
webClient.setAlertHandler((page, message) -> 
                           fail("JavaScript alert: " + message));
```

**Listing 15.11 Asserting the expected confirmation messages**

```java
public class WindowConfirmTest extends ManagedWebClient {
 
    @Test
    public void testWindowConfirm() throws FailingHttpStatusCodeException, 
                                           IOException {
        String html = "<html><head><title>Hello</title></head>            #A
                       <body onload='confirm(\"Confirm Message\")'>       #A
                       </body></html>";                                   #A
        URL testUrl = new URL("http://Page1/");                           #B
        MockWebConnection mockConnection = new MockWebConnection();       #C
        final List<String> confirmMessages = new ArrayList<String>();     #D
 
        webClient.setConfirmHandler((page, message) -> {                  #E
            confirmMessages.add(message);                                 #E
            return true;                                                  #E
        });                                                               #E
        mockConnection.setResponse(testUrl, html);                        #F
        webClient.setWebConnection(mockConnection);                       #G
 
        HtmlPage firstPage = webClient.getPage(testUrl);                  #H
        WebAssert.assertTitleEquals(firstPage, "Hello");                  #I
        assertArrayEquals(new String[] { "Confirm Message" },             #J
                          confirmMessages.toArray());                     #J
    }
}
```

**Listing 15.12 Asserting the expected confirmation messages from a JavaScript function**

```java
public class WindowConfirmTest extends ManagedWebClient {
 
  @Test
  public void testWindowConfirmAndAlert() throws 
                FailingHttpStatusCodeException, IOException {
    String html = "<html><head><title>Hello</title>                #A
               <script>function go(){                              #A
                  alert(confirm('Confirm Message'))                #A
               }</script>\n" +                                     #A
               "</head><body onload='go()'></body></html>";        #A
    URL testUrl = new URL("http://Page1/");
    MockWebConnection mockConnection = new MockWebConnection();
    final List<String> confirmMessages = new ArrayList<String>();
    webClient.setAlertHandler(new CollectingAlertHandler());       #B
    webClient.setConfirmHandler((page, message) -> {
        confirmMessages.add(message);
        return true;
    });
    mockConnection.setResponse(testUrl, html);
    webClient.setWebConnection(mockConnection);
 
    HtmlPage firstPage = webClient.getPage(testUrl);
    WebAssert.assertTitleEquals(firstPage, "Hello");
    assertArrayEquals(new String[] { "Confirm Message" },          #C
                  confirmMessages.toArray());                      #C
    assertArrayEquals(new String[] { "true" },                     #D
                  ((CollectingAlertHandler)                        #D
                   webClient.getAlertHandler())                    #D
                  .getCollectedAlerts().toArray());                #D
  }
}
```

## 15.4  Introducing Selenium

​	Selenium is a free open-source tool suite used to test web applications. Selenium’s strength lies in its ability to run tests against a real browser on a specific operating system, this is unlike HtmlUnit, which emulates the browser in the same VM (Virtual Machine) as your tests. Selenium lets you write tests in a few programming languages, including Java with JUnit 5.

![](https://pic.imgdb.cn/item/613f126144eaada73920cbe1.jpg)

Selenium WebDriver includes the following four components:

1. Selenium Client Libraries - Selenium supports multiple libraries for programming languages as Java, C#, PHP, Python, Ruby, etc.

2. JSON Wire Protocol Over HTTP - JSON (JavaScript Object Notation) is used to transfer data between servers and clients on the web. JSON Wire Protocol is a REST API that transfers the information to the HTTP server. Each WebDriver (such as FirefoxDriver, ChromeDriver, InternetExplorerDriver, etc.) has its own HTTP server.

3. Browser Drivers - Each browser contains a separate browser driver. Browser drivers communicate with the respective browser without revealing the internal logic of the browser functionality. When a browser driver has received any command, then that command will be executed on the respective browser and the response will go back in the form of an HTTP response.

4. Browsers - Selenium supports multiple browsers such as Firefox, Chrome, Internet Explorer, Safari, etc.

## 15.5  Writing Selenium Tests

```xml
<dependency>
     <groupId>org.seleniumhq.selenium</groupId>
     <artifactId>selenium-java</artifactId>
     <version>3.141.59</version>
</dependency>
```

**Table 15.2 Browsers supported by Selenium**

| Web Browser       | Browser driver class   |
| ----------------- | ---------------------- |
| Google Chrome     | ChromeDriver           |
| Internet Explorer | InternetExplorerDriver |
| Safari            | SafariDriver           |
| Opera             | OperaDriver            |
| Firefox           | FirefoxDriver          |
| Edge              | EdgeDriver             |

​	For our demonstration purposes, we are going to use three of the most popular browsers: Google Chrome, Internet Explorer and Mozilla Firefox. So, we have downloaded the Selenium drivers for them and copied them into a dedicated folder, as shown in fig. 15.3.

![](https://pic.imgdb.cn/item/613f154444eaada739241b5b.jpg)

​	Please note that you will need exactly the driver corresponding to the browser installed on your computer. For example, if your Google Chrome version is 79, you will be able to use only version 79 of the driver. Versions as 77, 78 or 80 will not work.

​	We need to include the folder with the Selenium drivers on the path of the operating system. On Windows, the most used operating system, you can do this by accessing This PC -> Properties -> Advanced system settings -> Environment variables -> Path -> Edit (fig. 15.4). For doing the same on other operating systems, consult their documentation.

![](https://pic.imgdb.cn/item/613f15d544eaada73924d885.jpg)

### 15.5.1  Testing for a specific web browser

**Listing 15.14 Accessing the Manning and the Google homepages with Chrome**

```java
public class ChromeSeleniumTest {
 
    private WebDriver driver;                                               #A
 
    @BeforeEach
    void setUp() {
        driver = new ChromeDriver();                                        #B
    }
 
    @Test
    void testChromeManning() {
        driver.get("https://www.manning.com/");                             #C
        assertThat(driver.getTitle(), is("Manning | Home"));                #C
    }
    
    @Test
    void testChromeGoogle() {
        driver.get("https://www.google.com");                               #D
        assertThat(driver.getTitle(), is("Google"));                        #D
    }
 
 
    @AfterEach
    void tearDown() {
        driver.quit();                                                      #E
    }
}
```

**Listing 15.15 Accessing the Manning and the Google homepages with Firefox**

```java
public class FirefoxSeleniumTest {
 
    private WebDriver driver;                                              #A’
 
    @BeforeEach
    void setUp() {
        driver = new FirefoxDriver();                                      #B’
    }
 
    @Test
    void testFirefoxManning() {
        driver.get("https://www.manning.com/");                            #C’
        assertThat(driver.getTitle(), is("Manning | Home"));               #C’
    }
 
    @Test
    void testFirefoxGoogle() {
        driver.get("https://www.google.com");                              #D’
        assertThat(driver.getTitle(), is("Google"));                       #D’
    }
 
    @AfterEach
    void tearDown() {
        driver.quit();                                                     #E’
    }
}
```

### 15.5.2  Testing navigation using a web browser

**Listing 15.16 Finding an element on a page with Firefox**

```java
public class WikipediaAccessTest {
 
    private RemoteWebDriver driver;                                         #A
 
    @BeforeEach
    void setUp() {
        driver = new FirefoxDriver();                                       #B
    }
 
    @Test
    void testWikipediaAccess() {
        driver.get("https://en.wikipedia.org/");                            #C
        assertThat(driver.getTitle(),                                       #C
                    is("Wikipedia, the free encyclopedia"));                #C
 
        WebElement contents = driver.findElementByLinkText("Contents");     #D
        assertTrue(meap.isDisplayed());                                     #D
 
        contents.click();                                                   #E
        assertThat(driver.getTitle(),                                       #E
                             is("Wikipedia:Contents - Wikipedia "));        #E
    }
 
    @AfterEach
    void tearDown() {
        driver.quit();                                                      #F
    }
}
```

### 15.5.3  Testing more than one web browser

```
public class MultiBrowserSeleniumTest {
 
    public static Collection<WebDriver> getBrowserVersions() {              #A
        return Arrays.asList(new WebDriver[] {new FirefoxDriver(), new      #A
               ChromeDriver(), new InternetExplorerDriver()});              #A
    }                                                                       #A
 
    @ParameterizedTest                                                      #B
    @MethodSource("getBrowserVersions")                                     #B
    void testManningAccess(WebDriver driver) {
        driver.get("https://www.manning.com/");                             #C
        assertThat(driver.getTitle(), is("Manning | Home"));                #C
        driver.quit();                                                      #D
    }
 
    @ParameterizedTest                                                      #B
    @MethodSource("getBrowserVersions")                                     #B
    void testGoogleAccess(WebDriver driver) {
        driver.get("https://www.google.com");                               #E
        assertThat(driver.getTitle(), is("Google"));                        #E
        driver.quit();                                                      #F
    }
 
}
```

### 15.5.4  Testing Google search and navigation using different web browsers

**Listing 15.18 Testing Google search and navigating inside the Wikipedia website**

```java
public class GoogleSearchTest {
 
    public static Collection<RemoteWebDriver> getBrowserVersions() {        #A
        return Arrays.asList(new RemoteWebDriver[] {new FirefoxDriver(),    #A
             new ChromeDriver(), new InternetExplorerDriver()});            #A
    }                                                                       #A
 
    @ParameterizedTest                                                      #B
    @MethodSource("getBrowserVersions")                                     #B
    void testGoogleSearch(RemoteWebDriver driver) {
        driver.get("http://www.google.com");                                #C
        WebElement element = driver.findElement(By.name("q"));              #C
        element.sendKeys("en.wikipedia.org");                               #D
        driver.findElement(By.name("q")).sendKeys(Keys.ENTER);              #D
 
        WebElement myDynamicElement = (new WebDriverWait(driver, 10))       #E
                 .until(ExpectedConditions                                  #E
                 .presenceOfElementLocated(By.id("resultStats")));          #E
 
        List<WebElement> findElements =                                     #F
                driver.findElements(By.xpath("//*[@id='rso']//a/h3"));      #F
 
        findElements.get(0).click();                                        #G
 
        assertEquals("https://en.wikipedia.org/wiki/Main_Page",             #G
                            driver.getCurrentUrl());                        #G
        assertThat(driver.getTitle(),                                       #G
                    is("Wikipedia, the free encyclopedia"));                #G
 
        WebElement contents = driver.findElementByLinkText("Contents");     #H
        assertTrue(contents.isDisplayed());                                 #H
        
        contents.click();                                                   #I
        assertThat(driver.getTitle(),                                       #I
                              is("Wikipedia:Contents - Wikipedia"));        #I
 
        driver.quit();                                                      #J
    }
}
```

![](https://pic.imgdb.cn/item/613f19db44eaada73929f7af.jpg)

### 15.5.5  Testing the authentication scenario to a website

![](https://pic.imgdb.cn/item/613f19fb44eaada7392a1cbd.jpg)

```java
public class Homepage {
    private WebDriver webDriver;                                            #A
 
    public Homepage(WebDriver webDriver) {
        this.webDriver = webDriver;                                         #B
    }
 
    public LoginPage openFormAuthentication() {
        webDriver.get("https://the-internet.herokuapp.com/");               #C
        webDriver.findElement(By.cssSelector("[href=\"/login\"]"))          #D
                 .click();
        return new LoginPage(webDriver);                                    #E
    }
}
```

![](https://pic.imgdb.cn/item/613f1a7544eaada7392aa817.jpg)

**Listing 15.20 The class describing the login page of the tested website**

```java
public class LoginPage {
 
    private WebDriver webDriver;                                            #A
 
    public LoginPage(WebDriver webDriver) {
        this.webDriver = webDriver;                                         #B
    }
 
    public LoginPage loginWith(String username, String password) {
        webDriver.findElement(By.id("username")).sendKeys(username);        #C
        webDriver.findElement(By.id("password")).sendKeys(password);        #C
        webDriver.findElement(By.cssSelector("#login button")).click();     #D
 
        return this;                                                        #E
    }
 
    public void thenLoginSuccessful() {
        assertTrue(webDriver                                                #F
                  .findElement(By.cssSelector("#flash.success"))            #F
                  .isDisplayed());                                          #F
        assertTrue(webDriver                                                #F
                   .findElement(By.cssSelector("[href=\"/logout\"]"))       #F
                   .isDisplayed());                                         #F
    }
 
    public void thenLoginUnsuccessful() {
        assertTrue(webDriver.findElement(By.id("username")).isDisplayed()); #G
        assertTrue(webDriver.findElement(By.id("password")).isDisplayed()); #G
    }
}
```

![](https://pic.imgdb.cn/item/613f1b0944eaada7392b5f06.jpg)

**Listing 15.21 The successful and unsuccessful login tests**

```java
public class LoginTest {
 
    private Homepage homepage;                                              #A
 
    public static Collection<WebDriver> getBrowserVersions() {              #B
        return Arrays.asList(new WebDriver[] {new FirefoxDriver(),          #B
               new ChromeDriver(), new InternetExplorerDriver()});          #B
    }                                                                       #B
 
    @ParameterizedTest                                                      #C
    @MethodSource("getBrowserVersions")                                     #C
    public void loginWithValidCredentials(WebDriver webDriver) {            #C
        homepage = new Homepage(webDriver);                                 #D
        homepage                                                            #E
                .openFormAuthentication()                                   #E
                .loginWith("tomsmith", "SuperSecretPassword!")              #E
                .thenLoginSuccessful();                                     #E
        webDriver.quit();                                                   #G
    }
 
    @ParameterizedTest                                                      #C
    @MethodSource("getBrowserVersions")                                     #C
    public void loginWithInvalidCredentials(WebDriver webDriver) {          #C
        homepage = new Homepage(webDriver);                                 #D
        homepage                                                            #F
                .openFormAuthentication()                                   #F
                .loginWith("tomsmith", "SuperSecretPassword")               #F
                .thenLoginUnsuccessful();                                   #F
        webDriver.quit();                                                   #G
    }
}
```

## 15.6  HtmlUnit vs. Selenium

​	The major difference between the two is that HtmlUnit emulates a specific web browser while Selenium drives a real web browser process.

Finally:

> Use HtmlUnit when…
>
> Use HtmlUnit when your application is independent of operating system features and browser-specific implementations not accounted for by HtmlUnit.

 

> Use Selenium when…
>
> Use Selenium when you require validation of specific browsers and operating systems, especially if the application takes advantage of or depends on a browser-specific implementation.

# 16. Testing Spring applications

```java
public class SimpleAppTest {                                                #A
   
   private static final String APPLICATION_CONTEXT_XML_FILE_NAME =          #B
          "classpath:application-context.xml";                              #B
 
   private ClassPathXmlApplicationContext context;                          #C
 
   private Passenger expectedPassenger;                                     #D
 
   @Before
   public void setUp() {
      context = new ClassPathXmlApplicationContext(                         #E
            APPLICATION_CONTEXT_XML_FILE_NAME);                             #E
      expectedPassenger = getExpectedPassenger();                           #F
   }
 
   @Test
   public void testInitPassenger() {
      Passenger passenger = (Passenger) context.getBean("passenger");       #G
      assertEquals(expectedPassenger, passenger);                           #H
   }
 
}
```

### 16.3.2  Using the Spring TestContext framework

```java
@RunWith(SpringJUnit4ClassRunner.class)                                     #A
@ContextConfiguration("classpath:application-context.xml")                  #B
public class SpringAppTest {
   
   @Autowired                                                               #C
   private Passenger passenger;                                             #C
   private Passenger expectedPassenger;
   
 
   @Before
   public void setUp() {
      expectedPassenger = getExpectedPassenger();
   }
 
   @Test
   public void testInitPassenger() {
      assertEquals(expectedPassenger, passenger);
   }
 
}
```

## 16.4  Using the SpringExtension for JUnit Jupiter

**Listing 16.12 The pom.xml file with the Spring 5 and JUnit 5 dependencies**

```xml
<dependency>                                                                #A
    <groupId>org.springframework</groupId>                                  #A
    <artifactId>spring-context</artifactId>                                 #A
    <version>5.2.0.RELEASE</version>                                        #A
</dependency>                                                               #A
<dependency>                                                                #B
    <groupId>org.springframework</groupId>                                  #B
    <artifactId>spring-test</artifactId>                                    #B
    <version>5.2.0.RELEASE</version>                                        #B
</dependency>                                                               #B
<dependency>                                                                #C
    <groupId>org.junit.jupiter</groupId>                                    #C
    <artifactId>junit-jupiter-api</artifactId>                              #C
    <version>5.6.0</version>                                                #C
    <scope>test</scope>                                                     #C
</dependency>                                                               #C
<dependency>                                                                #D
    <groupId>org.junit.jupiter</groupId>                                    #D
    <artifactId>junit-jupiter-engine</artifactId>                           #D
    <version>5.6.0</version>                                                #D
    <scope>test</scope>                                                     #D
</dependency>                                                               #D
```

**Listing 16.13 The SpringAppTest class**

```java
@ExtendWith(SpringExtension.class)                                          #A
@ContextConfiguration("classpath:application-context.xml")                  #B
public class SpringAppTest {
   
   @Autowired                                                               #C
   private Passenger passenger;                                             #C
   private Passenger expectedPassenger;
   
   @BeforeEach                                                              #D
   public void setUp() {
      expectedPassenger = getExpectedPassenger();
   }
 
   @Test
   public void testInitPassenger() {
      assertEquals(expectedPassenger, passenger);
      System.out.println(passenger);
   }
}
```

## 16.5  Adding a new feature and testing it with JUnit 5

​	略。

# 17.Testing Spring Boot applications

略

# 18.Testing a REST API

```java
@SpringBootTest                                                             #A
@AutoConfigureMockMvc                                                       #B
@Import(FlightBuilder.class)                                                #C
public class RestApplicationTest {
 
    @Autowired                                                              #D
    private MockMvc mvc;                                                    #D
 
    @Autowired                                                              #E
    private Flight flight;                                                  #E
 
    @Autowired                                                              #E
    private Map<String, Country> countriesMap;                              #E
 
    @MockBean                                                               #F
    private PassengerRepository passengerRepository;                        #F
 
 
    @MockBean                                                               #F
    private CountryRepository countryRepository;                            #F
 
    @Test
    void testGetAllCountries() throws Exception {
        when(countryRepository.findAll()).thenReturn(new                    #G
               ArrayList<>(countriesMap.values()));                         #G
        mvc.perform(get("/countries"))                                      #H
           .andExpect(status().isOk())                                      #I
           .andExpect(content().contentType(MediaType.APPLICATION_JSON))    #I
           .andExpect(jsonPath("$", hasSize(3)));                           #I
 
        verify(countryRepository, times(1)).findAll();                      #J
    }
 
    @Test
    void testGetAllPassengers() throws Exception {
        when(passengerRepository.findAll()).thenReturn(new                  #K
             ArrayList<>(flight.getPassengers()));                          #K
 
        mvc.perform(get("/passengers"))                                     #L
           .andExpect(status().isOk())                                      #L
           .andExpect(content().contentType(MediaType.APPLICATION_JSON))    #L
           .andExpect(jsonPath("$", hasSize(20)));                          #L
 
        verify(passengerRepository, times(1)).findAll();                    #M
    }
 
    @Test
    void testPassengerNotFound() {
        Throwable throwable = assertThrows(NestedServletException.class,    #N
                  () -> mvc.perform(get("/passengers/30"))                  #N
                               .andExpect(status().isNotFound()));          #N
        assertEquals(PassengerNotFoundException.class,                      #O
                     throwable.getCause().getClass());                      #O
    }
 
    @Test
    void testPostPassenger() throws Exception {
 
        Passenger passenger = new Passenger("Peter Michelsen");             #P
        passenger.setCountry(countriesMap.get("US"));                       #P
        passenger.setIsRegistered(false);                                   #P
        when(passengerRepository.save(passenger))                           #P
            .thenReturn(passenger);                                         #P
 
        mvc.perform(post("/passengers")                                     #Q
            .content(new ObjectMapper().writeValueAsString(passenger))      #Q
            .header(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON))  #Q
           .andExpect(status().isCreated())                                 #Q
           .andExpect(jsonPath("$.name", is("Peter Michelsen")))            #Q
           .andExpect(jsonPath("$.country.codeName", is("US")))             #Q
           .andExpect(jsonPath("$.country.name", is("USA")))                #Q
           .andExpect(jsonPath("$.registered", is(Boolean.FALSE)));         #Q
 
   verify(passengerRepository, times(1)).save(passenger);                   #R
  }
 
  @Test
  void testPatchPassenger() throws Exception {
      Passenger passenger = new Passenger("Sophia Graham");                 #S
      passenger.setCountry(countriesMap.get("UK"));                         #S
      passenger.setIsRegistered(false);                                     #S
      when(passengerRepository.findById(1L))                                #S
           .thenReturn(Optional.of(passenger));                             #S
      when(passengerRepository.save(passenger))                             #T
           .thenReturn(passenger);                                          #T
      String updates =                                                      #U
        "{\"name\":\"Sophia Jones\", \"country\":\"AU\",                    #U
         \"isRegistered\":\"true\"}";                                       #U
 
      mvc.perform(patch("/passengers/1")                                    #U
          .content(updates)                                                 #U
          .header(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON))    #U
          .andExpect(content().contentType(MediaType.APPLICATION_JSON))     #U
          .andExpect(status().isOk());
 
      verify(passengerRepository, times(1)).findById(1L);                   #V
      verify(passengerRepository, times(1)).save(passenger);                #V
  }
 
  @Test
  public void testDeletePassenger() throws Exception {
 
      mvc.perform(delete("/passengers/4"))                                  #W
             .andExpect(status().isOk());                                   #W
 
      verify(passengerRepository, times(1)).deleteById(4L);                 #X
  }
 
}
```

# 19. Testing database applications

* Examining the challenges of database testing
*  Implementing tests for a JDBC application
* Implementing tests for a Spring JDBC application
* Implementing tests for a Hibernate application
*  Implementing tests for a Spring Hibernate application
* Comparing the different approaches of building and testing database applications

## 19.1  The Database Unit Testing Impedance Mismatch

### 19.1.1  Unit tests must exercise code in isolation

* Unit tests are used to test classes that interact directly with the database (like DAOs). A DAO (*Data Access Object)* is an object that provides an interface to a database and maps application calls to the specific database operations without exposing details of the persistence layer. Such tests guarantee these classes execute the proper operation against the database. Although these tests depend on external entities (like the database and/or persistence frameworks), they exercise classes that are building blocks in a bigger application (and hence are units).
* Similarly, unit tests can be written to test the upper layers (like Facades), without the need of accessing the database – in these tests, the persistence layer can be emulated by mocks or stubs. As a facade in architecture, the Facade design pattern provides an object that serves as a front-facing interface masking a more complex underlying code.

### 19.1.2  Unit tests must be easy to write and run

略

### 19.1.3  Unit tests must be fast to run

略

## 19.2  Testing a JDBC application

```java
public class Country {
    private String name;                                                    #A
    private String codeName;                                                #A
 
    public Country(String name, String codeName) {                          #B
        this.name = name;                                                   #B
        this.codeName = codeName;                                           #B
    }                                                                       #B
 
    public String getName() {                                               #A
        return name;                                                        #A
    }                                                                       #A
 
    public void setName(String name) {                                      #A
        this.name = name;                                                   #A
    }                                                                       #A
 
    public String getCodeName() {                                           #A
        return codeName;                                                    #A
    }                                                                       #A
 
    public void setCodeName(String codeName) {                              #A
        this.codeName = codeName;                                           #A
    }                                                                       #A
 
    @Override                                                               #C
    public String toString() {                                              #C
        return "Country{" +                                                 #C
                "name='" + name + '\'' +                                    #C
                ", codeName='" + codeName + '\'' +                          #C
                '}';                                                        #C
    }                                                                       #C  
 
    @Override                                                               #D
    public boolean equals(Object o) {                                       #D
        if (this == o) return true;                                         #D
        if (o == null || getClass() != o.getClass()) return false;          #D
        Country country = (Country) o;                                      #D
        return Objects.equals(name, country.name) &&                        #D
                Objects.equals(codeName, country.codeName);                 #D
    }                                                                       #D
 
    @Override                                                               #D
    public int hashCode() {                                                 #D
        return Objects.hash(name, codeName);                                #D
    }                                                                       #D
}
```

**Listing 19.2 The Maven pom.xml dependencies**

```xml
<dependencies>
   <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter-api</artifactId>
      <version>5.6.0</version>
      <scope>test</scope>
   </dependency>
   <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter-engine</artifactId>
      <version>5.6.0</version>
      <scope>test</scope>
   </dependency>
   <dependency>
      <groupId>com.h2database</groupId>
      <artifactId>h2</artifactId>
      <version>1.4.193</version>
   </dependency>
</dependencies>
```

**Listing 19.3 The ConnectionManager class**

```java
public class ConnectionManager {
   
   private static Connection connection;                                    #A
   
   public static Connection openConnection() {
 
        try {
            Class.forName("org.h2.Driver");                                 #B
            connection = DriverManager.getConnection("jdbc:h2:~/country",   #C
                "sa",                                                       #C
                ""                                                          #C
                );                                                          #C
            return connection;                                              #D
        } catch(ClassNotFoundException | SQLException e) {                  #E
            throw new RuntimeException(e);                                  #E
        }                                                                   #E
    }
 
    public static void closeConnection() {
       if (null != connection) {                                            #F
           try {
               connection.close();                                          #G
           } catch(SQLException e) {                                        #H
               throw new RuntimeException(e);                               #H
           }                                                                #H
       }
    }
}
```

**Listing 19.4 The TablesManager class**

```java
public class TablesManager {
   
    public static void createTable() {
       String sql = "CREATE TABLE COUNTRY( ID IDENTITY,                     #A
                     NAME VARCHAR(255), CODE_NAME VARCHAR(255) );";         #A
       executeStatement(sql);                                               #B
    }
 
    public static void dropTable() {
       String sql = "DROP TABLE IF EXISTS COUNTRY;";                        #C
       executeStatement(sql);                                               #D
    }
    
    private static void executeStatement(String sql) {
      PreparedStatement statement;                                          #E
       
       try {
            Connection connection = openConnection();                       #F
            statement = connection.prepareStatement(sql);                   #G
            statement.executeUpdate();                                      #H
            statement.close();                                              #I
       } catch (SQLException e) {
            throw new RuntimeException(e);                                  #J
       } finally {
            closeConnection();                                              #K
       }
   }
 
}
```

**Listing 19.5 The CountryDao class**

```java
public class CountryDao {
   private static final String GET_ALL_COUNTRIES_SQL =                      #A
           "select * from country";                                         #A
   private static final String GET_COUNTRIES_BY_NAME_SQL =                  #B
           "select * from country where name like ?";                       #B
 
   public List<Country> getCountryList() {
      List<Country> countryList = new ArrayList<>();                        #C
 
      try {
         Connection connection = openConnection();                          #D
         PreparedStatement statement =                                      #E
                     connection.prepareStatement(GET_ALL_COUNTRIES_SQL);    #E
         ResultSet resultSet = statement.executeQuery();                    #E
 
         while (resultSet.next()) {                                         #F
            countryList.add(new Country(resultSet.getString(2),             #F
                            resultSet.getString(3)));                       #F
         }
         statement.close();                                                 #G
      } catch (SQLException e) {                                            #H
         throw new RuntimeException(e);                                     #H
      } finally {
         closeConnection();                                                 #I
      }
      return countryList;                                                   #J
   }
 
   public List<Country> getCountryListStartWith(String name) {
      List<Country> countryList = new ArrayList<>();                        #K
 
      try {
         Connection connection = openConnection();                          #L
         PreparedStatement statement =                                      #M
                   connection.prepareStatement(GET_COUNTRIES_BY_NAME_SQL);  #M
         statement.setString(1, name + "%");                                #M
         ResultSet resultSet = statement.executeQuery();                    #M
 
         while (resultSet.next()) {                                         #N
            countryList.add(new Country(resultSet.getString(2),             #N
                            resultSet.getString(3)));                       #N
         }
         statement.close();                                                 #O
      } catch (SQLException e) {                                            #P
         throw new RuntimeException(e);                                     #P
      } finally {
         closeConnection();                                                 #Q
      }
      return countryList;                                                   #R
   }
}
```

**Listing 19.6 The CountriesLoader class**

```java
public class CountriesLoader {
 
    private static final String LOAD_COUNTRIES_SQL =                        #A
            "insert into country (name, code_name) values ";
 
    public static final String[][] COUNTRY_INIT_DATA = {                    #B
          { "Australia", "AU"}, { "Canada", "CA" }, { "France", "FR" },     #B
          { "Germany", "DE" }, { "Italy", "IT" }, { "Japan", "JP" },        #B
          { "Romania", "RO" },{ "Russian Federation", "RU" },               #B
          { "Spain", "ES" }, { "Switzerland", "CH" },                       #B
          { "United Kingdom", "UK" }, { "United States", "US" } };          #B
 
    public void loadCountries() {
        for (String[] countryData : COUNTRY_INIT_DATA) {                    #C
            String sql = LOAD_COUNTRIES_SQL + "('" + countryData[0] + "',   #D
                           '" + countryData[1] + "');";                     #D
 
            try {
                Connection connection = openConnection();                   #E
                PreparedStatement statement =                               #F
                      connection.prepareStatement(sql);                     #F
                statement.executeUpdate();                                  #F
                statement.close();                                          #F
            } catch (SQLException e) {                                      #G
                throw new RuntimeException(e);                              #G 
            } finally {
                closeConnection();                                          #H
            }
        }
    }
}
```

**Listing 19.7 The CountriesDatabaseTest class**

```java
import static com.manning.junitbook.databases.CountriesLoader.COUNTRY_INIT_DATA;          #A
[…]
 
public class CountriesDatabaseTest {
    private CountryDao countryDao = new CountryDao();                       #A
    private CountriesLoader countriesLoader = new CountriesLoader();        #A
   
    private List<Country> expectedCountryList = new ArrayList<>();          #B
    private List<Country> expectedCountryListStartsWithA =                  #C
                          new ArrayList<>();                                #C
 
    @BeforeEach                                                             #D
    public void setUp() {
        TablesManager.createTable();                                        #E
        initExpectedCountryLists();                                         #F
        countriesLoader.loadCountries();                                    #G
    }
 
    @Test
    public void testCountryList() {
        List<Country> countryList = countryDao.getCountryList();            #H
        assertNotNull(countryList);                                         #I
        assertEquals(expectedCountryList.size(), countryList.size());       #J
        for (int i = 0; i < expectedCountryList.size(); i++) {              #K
            assertEquals(expectedCountryList.get(i), countryList.get(i));   #K
        }
    }
 
    @Test
    public void testCountryListStartsWithA() {
        List<Country> countryList =                                         #L
                      countryDao.getCountryListStartWith("A");              #L
        assertNotNull(countryList);                                         #M
        assertEquals(expectedCountryListStartsWithA.size(),                 #N
                     countryList.size());                                   #N
        for (int i = 0; i < expectedCountryListStartsWithA.size(); i++) {   #O
            assertEquals(expectedCountryListStartsWithA.get(i),             #O
                         countryList.get(i));                               #O
        }
    }
 
    @AfterEach                                                              #P
    public void dropDown() {
        TablesManager.dropTable();                                          #Q
    }
 
    private void initExpectedCountryLists() {
         for (int i = 0; i < COUNTRY_INIT_DATA.length; i++) {               #R
             String[] countryInitData = COUNTRY_INIT_DATA[i];               #S
             Country country = new Country(countryInitData[0],              #S
                                   countryInitData[1]);                     #S
             expectedCountryList.add(country);                              #T
             if (country.getName().startsWith("A")) {                       #U
                 expectedCountryListStartsWithA.add(country);               #U
             }
         }
     }
}
```

## 19.3  Testing a Spring JDBC application

**Listing 19.8 The new dependencies introduced into the Maven pom.xml file**

```xml
<dependency>                                                             #A
	<groupId>org.springframework</groupId>                            #A
	<artifactId>spring-context</artifactId>                           #A
	<version>5.2.1.RELEASE</version>                                  #A
</dependency>                                                            #A
<dependency>                                                             #B
	<groupId>org.springframework</groupId>                            #B
	<artifactId>spring-jdbc</artifactId>                              #B
	<version>5.2.1.RELEASE</version>                                  #B
</dependency>                                                            #B
<dependency>                                                             #C
	<groupId>org.springframework</groupId>                            #C
	<artifactId>spring-test</artifactId>                              #C
	<version>5.2.1.RELEASE</version>                                  #C
</dependency> 
```

**The db-schema.sql file**

```sql
create table country( id identity , name varchar (255) , code_name varchar (255) );
```

**Listing 19.10 The application-context.xml file**

```xml
<jdbc:embedded-database id="dataSource" type="H2">                          #A
   <jdbc:script location="classpath:db-schema.sql"/>                        #A
</jdbc:embedded-database>                                                   #A
 
<bean id="countryDao"                                                       #B
      class="com.manning.junitbook.databases.dao.CountryDao">               #B
       <property name="dataSource" ref="dataSource"/>                       #B
</bean>                                                                     #B
 
<bean id="countriesLoader"                                                  #C class="com.manning.junitbook.databases.CountriesLoader">             #C
   <property name="dataSource" ref="dataSource"/>                           #C
</bean>                                                                     #C
```

**Listing 19.11 The CountriesLoader class**

```java
public class CountriesLoader extends JdbcDaoSupport                         #A
{
 
    private static final String LOAD_COUNTRIES_SQL =                        #B
            "insert into country (name, code_name) values ";
 
    public static final String[][] COUNTRY_INIT_DATA = {                    #C
          { "Australia", "AU"}, { "Canada", "CA" }, { "France", "FR" },     #C
          { "Germany", "DE" }, { "Italy", "IT" }, { "Japan", "JP" },        #C
          { "Romania", "RO" },{ "Russian Federation", "RU" },               #C
          { "Spain", "ES" }, { "Switzerland", "CH" },                       #C
   { "United Kingdom", "UK" }, { "United States", "US" } };                 #C
 
    public void loadCountries() {
    for (String[] countryData : COUNTRY_INIT_DATA) {                        #D
      String sql = LOAD_COUNTRIES_SQL + "('" + countryData[0] + "',         #E
                     '" + countryData[1] + "');";                           #E
       getJdbcTemplate().execute(sql);                                      #E
    }
   }
 
}
```

**Listing 19.12 The CountryRowMapper class**

```java
public class CountryRowMapper implements RowMapper<Country> {               #A
   public static final String NAME = "name";                                #B
   public static final String CODE_NAME = "code_name";                      #B
 
   @Override
   public Country mapRow(ResultSet resultSet, int i) throws SQLException {  #C
      Country country = new Country(resultSet.getString(NAME),              #C
           resultSet.getString(CODE_NAME));                                 #C
      return country;                                                       #D
   }
}
```

**Listing 19.13 The CountryDao class**

```java
public class CountryDao extends JdbcDaoSupport                              #A
{
   private static final String GET_ALL_COUNTRIES_SQL =                      #B
           "select * from country";                                         #B
   private static final String GET_COUNTRIES_BY_NAME_SQL =                  #B
           "select * from country where name like :name";                   #B
 
   private static final CountryRowMapper COUNTRY_ROW_MAPPER =               #C
           new CountryRowMapper();                                          #C
 
   public List<Country> getCountryList() {
      List<Country> countryList =                                           #D
       getJdbcTemplate().query(GET_ALL_COUNTRIES_SQL, COUNTRY_ROW_MAPPER);  #D
      return countryList;                                                   #D
   }
 
   public List<Country> getCountryListStartWith(String name) {
        NamedParameterJdbcTemplate namedParameterJdbcTemplate =             #E
            new NamedParameterJdbcTemplate(getDataSource());                #E
        SqlParameterSource sqlParameterSource =                             #F
            new MapSqlParameterSource("name", name + "%");                  #F
        return namedParameterJdbcTemplate.query(GET_COUNTRIES_BY_NAME_SQL,  #G
           sqlParameterSource, COUNTRY_ROW_MAPPER);                         #G
   }
 
}
```

**Listing 19.14 The CountriesDatabaseTest class**

```java
@ExtendWith(SpringExtension.class)                                          #A
@ContextConfiguration("classpath:application-context.xml")                  #B
public class CountriesDatabaseTest {
 
    @Autowired                                                              #C
    private CountryDao countryDao;                                          #C
 
    @Autowired                                                              #D
    private CountriesLoader countriesLoader;                                #D
 
    private List<Country> expectedCountryList = new ArrayList<Country>();   #E
    private List<Country> expectedCountryListStartsWithA =                  #F
          new ArrayList<Country>();                                         #F
 
    @BeforeEach                                                             #G
    public void setUp() {
        initExpectedCountryLists();                                         #H
        countriesLoader.loadCountries();                                    #I
    }
 
    
    @Test
    @DirtiesContext                                                         #J
    public void testCountryList() {
        List<Country> countryList = countryDao.getCountryList();            #K
        assertNotNull(countryList);                                         #L
        assertEquals(expectedCountryList.size(), countryList.size());       #M
        for (int i = 0; i < expectedCountryList.size(); i++) {              #N
            assertEquals(expectedCountryList.get(i), countryList.get(i));   #N
        }                                                                   #N
    }
 
    @Test
    @DirtiesContext                                                         #J
    public void testCountryListStartsWithA() {
        List<Country> countryList = countryDao.getCountryListStartWith("A");#O
        assertNotNull(countryList);                                         #P 
        assertEquals(expectedCountryListStartsWithA.size(),                 #Q
                     countryList.size());                                   #Q
        for (int i = 0; i < expectedCountryListStartsWithA.size(); i++) {   #R
            assertEquals(expectedCountryListStartsWithA.get(i),             #R
                         countryList.get(i));                               #R
        }
    }
 
    private void initExpectedCountryLists() {
      for (int i = 0; i < CountriesLoader.COUNTRY_INIT_DATA.length; i++) {
          String[] countryInitData = CountriesLoader.COUNTRY_INIT_DATA[i];  #S
          Country country = new Country(countryInitData[0],                 #T
                  countryInitData[1]);                                      #T 
          expectedCountryList.add(country);                                 #U
          if (country.getName().startsWith("A")) {                          #V
              expectedCountryListStartsWithA.add(country);                  #V
          }                                                                 #V
      }
    }
}
```

## 19.4  Testing a Hibernate application

```xml
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>5.4.9.Final</version>
</dependency>
```

**Listing 19.16 The annotated Country class**

```java
@Entity                                                                     #A
@Table(name = "COUNTRY")                                                    #B
public class Country {
    @Id                                                                     #C
    @GeneratedValue(strategy = GenerationType.IDENTITY)                     #D
    @Column(name = "ID")                                                    #E
    private int id;
 
    @Column(name = "NAME")                                                  #F
    private String name;
 
    @Column(name = "CODE_NAME")                                             #F
    private String codeName;
 
    […]
}
```

**Listing 19.17 The persistence.xml file**

```xml
<persistence-unit name="manning.hibernate">                                 #A
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider> #B
        <class>com.manning.junitbook.databases.model.Country</class>        #C
 
        <properties>
            <property name="javax.persistence.jdbc.driver"                  #D
                            value="org.h2.Driver"/>                         #D
            <property name="javax.persistence.jdbc.url"                     #E
                            value="jdbc:h2:mem:test;DB_CLOSE_DELAY=-1"/>    #E
            <property name="javax.persistence.jdbc.user" value="sa"/>       #F
            <property name="javax.persistence.jdbc.password" value=""/>     #F
            <property name="hibernate.dialect"                              #G
                            value="org.hibernate.dialect.H2Dialect"/>       #G
            <property name="hibernate.show_sql" value="true"/>              #H
            <property name="hibernate.hbm2ddl.auto" value="create"/>        #I
        </properties>
</persistence-unit>
```

**Listing 19.18 The CountriesHibernateTest file**

```java
public class CountriesHibernateTest {
 
    private EntityManagerFactory emf;                                       #A
    private EntityManager em;                                               #B
 
    private List<Country> expectedCountryList =                             #C
                          new ArrayList<>();                                #C
    private List<Country> expectedCountryListStartsWithA =                  #C
                          new ArrayList<>();                                #C
 
    public static final String[][] COUNTRY_INIT_DATA = {                    #D
        { "Australia", "AU" }, { "Canada", "CA" }, { "France", "FR" },      #D
        { "Germany", "DE" }, { "Italy", "IT" }, { "Japan", "JP" },          #D
        { "Romania", "RO" }, { "Russian Federation", "RU" },                #D
        { "Spain", "ES" }, { "Switzerland", "CH" },                         #D
        { "United Kingdom", "UK" }, { "United States", "US" } };            #D
 
    @BeforeEach                                                             #E
    public void setUp() {
        initExpectedCountryLists();                                         #F
 
        emf = Persistence.createEntityManagerFactory("manning.hibernate");  #G
        em = emf.createEntityManager();                                     #G
 
        em.getTransaction().begin();                                        #H
 
        for (int i = 0; i < COUNTRY_INIT_DATA.length; i++) {                #I
            String[] countryInitData = COUNTRY_INIT_DATA[i];                #I
            Country country = new Country(countryInitData[0],               #I
                                  countryInitData[1]);                      #I
            em.persist(country);                                            #J
        }
 
        em.getTransaction().commit();                                       #H
    }
 
    @Test
    public void testCountryList() {
        List<Country> countryList = em.createQuery(                         #K
               "select c from Country c").getResultList();                  #K
        assertNotNull(countryList);                                         #L
        assertEquals(COUNTRY_INIT_DATA.length, countryList.size());         #M
        for (int i = 0; i < expectedCountryList.size(); i++) {              #N
            assertEquals(expectedCountryList.get(i), countryList.get(i));   #N
        }
 
    }
 
    @Test
    public void testCountryListStartsWithA() {
        List<Country> countryList = em.createQuery(                         #O
            "select c from Country c where c.name like 'A%'").              #O
                                              getResultList();              #O
        assertNotNull(countryList);                                         #P
        assertEquals(expectedCountryListStartsWithA.size(),                 #Q
                     countryList.size());                                   #Q
        for (int i = 0; i < expectedCountryListStartsWithA.size(); i++) {   #R
            assertEquals(expectedCountryListStartsWithA.get(i),             #R
                         countryList.get(i));                               #R
        }
    }
 
    @AfterEach                                                              #S
    public void dropDown() {
        em.close();                                                         #T
        emf.close();                                                        #T
    }
 
    private void initExpectedCountryLists() {
        for (int i = 0; i < COUNTRY_INIT_DATA.length; i++) {                #U
            String[] countryInitData = COUNTRY_INIT_DATA[i];                #V
            Country country = new Country(countryInitData[0],               #V
                                          countryInitData[1]);              #V
            expectedCountryList.add(country);                               #W
            if (country.getName().startsWith("A")) {                        #X
                expectedCountryListStartsWithA.add(country);                #X
            }
        }
    }
}
```

## 19.5  Testing a Spring Hibernate application

```xml
<dependency>                                                             #A
	<groupId>org.springframework</groupId>                            #A
	<artifactId>spring-context</artifactId>                           #A
	<version>5.2.1.RELEASE</version>                                  #A
</dependency>                                                            #A
<dependency>                                                             #B
<groupId>org.springframework</groupId>                            #B
	<artifactId>spring-orm</artifactId>                               #B
	<version>5.2.1.RELEASE</version>                                  #B
</dependency>                                                            #B
<dependency>                                                             #C
	<groupId>org.springframework</groupId>                            #C
	<artifactId>spring-test</artifactId>                              #C
	<version>5.2.1.RELEASE</version>                                  #C
</dependency>                                                            #C
<dependency>                                                             #D
	<groupId>org.hibernate</groupId>                                  #D
	<artifactId>hibernate-core</artifactId>                           #D
  	<version>5.4.9.Final</version>                                    #D
</dependency>                                                            #D
```

**Listing 19.20 The persistence.xml file**

```xml
<persistence-unit name="manning.hibernate">                                 #A
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider> #B
        <class>com.manning.junitbook.databases.model.Country</class>        #C
</persistence-unit>
```

**Listing 19.21 The application-context.xml file**

```xml
<tx:annotation-driven transaction-manager="txManager"/>                     #A
<bean id="dataSource" class=                                                #B
     "org.springframework.jdbc.datasource.DriverManagerDataSource">         #B
     <property name="driverClassName" value="org.h2.Driver"/>               #C
     <property name="url" value="jdbc:h2:mem:test;DB_CLOSE_DELAY=-1"/>      #D
     <property name="username" value="sa"/>                                 #E
     <property name="password" value=""/>                                   #E
</bean>
 
<bean id="entityManagerFactory" class=                                      #F
     "org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">  #F
     <property name="persistenceUnitName" value="manning.hibernate" />      #G
     <property name="dataSource" ref="dataSource"/>                         #H
     <property name="jpaProperties">
         <props>
           <prop key=                                                       #I
           "hibernate.dialect">org.hibernate.dialect.H2Dialect</prop>       #I
           <prop key="hibernate.show_sql">true</prop>                       #J
           <prop key="hibernate.hbm2ddl.auto">create</prop>                 #K
         </props>
     </property>
</bean>
 
<bean id="txManager" class=                                                 #L
     "org.springframework.orm.jpa.JpaTransactionManager">                   #L
     <property name="entityManagerFactory" ref="entityManagerFactory" />    #M
     <property name="dataSource" ref="dataSource" />                        #N
</bean>
 
<bean class="com.manning.junitbook.databases.CountryService"/>              #O
```

**Listing 19.22 The CountryService class**

```java
public class CountryService {
 
    @PersistenceContext                                                     #A
    private EntityManager em;                                               #A
 
    public static final String[][] COUNTRY_INIT_DATA =                      #B
      { { "Australia", "AU" }, { "Canada", "CA" }, { "France", "FR" },      #B
        { "Germany", "DE" }, { "Italy", "IT" }, { "Japan", "JP" },          #B
        { "Romania", "RO" }, { "Russian Federation", "RU" },                #B
        { "Spain", "ES" }, { "Switzerland", "CH" },                         #B
        { "United Kingdom", "UK" }, { "United States", "US" } };            #B
 
    @Transactional                                                          #C
    public void init() {
        for (int i = 0; i < COUNTRY_INIT_DATA.length; i++) {                #D
            String[] countryInitData = COUNTRY_INIT_DATA[i];                #D
            Country country = new Country(countryInitData[0],               #D
                                          countryInitData[1]);              #D
            em.persist(country);                                            #D
        }
    }
 
    @Transactional                                                          #C
    public void clear() {
        em.createQuery("delete from Country c").executeUpdate();            #E
    }
 
    public List<Country> getAllCountries() {
        return em.createQuery("select c from Country c")                    #F
                   .getResultList();                                        #F  
    }
 
    public List<Country> getCountriesStartingWithA() {
        return em.createQuery(                                              #G
            "select c from Country c where c.name like 'A%'")               #G
            .getResultList();                                               #G
    }
}
```

**Listing 19.23 The CountriesHibernateTest class**

```java
@ExtendWith(SpringExtension.class)                                          #A
@ContextConfiguration("classpath:application-context.xml")                  #B
public class CountriesHibernateTest {
 
    @Autowired                                                              #C
    private CountryService countryService;                                  #C
 
    private List<Country> expectedCountryList = new ArrayList<>();          #D
    private List<Country> expectedCountryListStartsWithA =                  #D
                              new ArrayList<>();                            #D
 
 
    @BeforeEach                                                             #E
    public void setUp() {
        countryService.init();                                              #F
        initExpectedCountryLists();                                         #G
    }
 
    @Test
    public void testCountryList() {
        List<Country> countryList = countryService.getAllCountries();       #H   
        assertNotNull(countryList);                                         #I
        assertEquals(COUNTRY_INIT_DATA.length, countryList.size());         #J
        for (int i = 0; i < expectedCountryList.size(); i++) {              #K
            assertEquals(expectedCountryList.get(i), countryList.get(i));   #K
        }
    }
 
    @Test
    public void testCountryListStartsWithA() {
        List<Country> countryList =                                         #L
               countryService.getCountriesStartingWithA();                  #L
        assertNotNull(countryList);                                         #M
        assertEquals(expectedCountryListStartsWithA.size(),                 #N
                     countryList.size());                                   #N
        for (int i = 0; i < expectedCountryListStartsWithA.size(); i++) {   #O
            assertEquals(expectedCountryListStartsWithA.get(i),             #O
                         countryList.get(i));                               #O
        }
    }
 
    @AfterEach                                                              #P
    public void dropDown() {
        countryService.clear();                                             #Q
    }
 
    private void initExpectedCountryLists() {
        for (int i = 0; i < COUNTRY_INIT_DATA.length; i++) {                #R
            String[] countryInitData = COUNTRY_INIT_DATA[i];                #S
            Country country = new Country(countryInitData[0],               #S
                                          countryInitData[1]);              #S
            expectedCountryList.add(country);                               #T
            if (country.getName().startsWith("A")) {                        #U
                expectedCountryListStartsWithA.add(country);                #U
            }
        }
    }
}
```

# 19    Testing database applications

This chapter covers:

·  Examining the challenges of database testing

·  Implementing tests for a JDBC application

·  Implementing tests for a Spring JDBC application

·  Implementing tests for a Hibernate application

·  Implementing tests for a Spring Hibernate application

·  Comparing the different approaches of building and testing database applications

Dependency is the key problem in software development at all scales…. Eliminating duplication in programs eliminates dependency.

—Kent Beck, Test-Driven Development: By Example

The persistence layer (or, roughly speaking, the database access code) is undoubtedly one of the most important parts of any enterprise project. Despite its importance, the persistence layer is hard to unit test, mainly because of the following three issues:

·  Unit tests must exercise code in isolation; the persistence layer requires interaction with an external entity, the database.

·  Unit tests must be easy to write and run; code that accesses the database can be cumbersome.

·  Unit tests must be fast to run; database access is relatively slow.

We call these issues the *Database Unit Testing Impedance Mismatch*, in a reference to the object-relational impedance mismatch (which describes the difficulties of using a relational database to persist data when an application is written using an object-oriented language).

## 19.1  The Database Unit Testing Impedance Mismatch

Let's take a deeper look at the three issues that comprise the Database Unit Testing Impedance Mismatch.

### 19.1.1  Unit tests must exercise code in isolation

From a purist point of view, tests that exercise database access code cannot be considered unit tests, as they depend on an external entity, the almighty database. What should they be called then? Integration Tests? Functional Tests? Non-Unit Unit Tests?

Well, the answer is: there is no secret ingredient! In other words, database tests can fit in many categories, depending on the context.

Pragmatically speaking, though, database access code can be exercised by both unit and integration tests:

·  Unit tests are used to test classes that interact directly with the database (like DAOs). A DAO (*Data Access Object)* is an object that provides an interface to a database and maps application calls to the specific database operations without exposing details of the persistence layer. Such tests guarantee these classes execute the proper operation against the database. Although these tests depend on external entities (like the database and/or persistence frameworks), they exercise classes that are building blocks in a bigger application (and hence are units).

·  Similarly, unit tests can be written to test the upper layers (like Facades), without the need of accessing the database – in these tests, the persistence layer can be emulated by mocks or stubs. As a facade in architecture, the Facade design pattern provides an object that serves as a front-facing interface masking a more complex underlying code.

There is still a practical question: can't the data present in the database get in the way of the tests? Yes, it is possible, so before we run the tests, we must assure the database is in a known state – and we’ll show how to do this in this chapter.

### 19.1.2  Unit tests must be easy to write and run

It does not matter how much a company, project manager, or technical leader praises unit tests – if they are not easy to write and run, developers will resist writing them. Moreover, writing code that accesses the database is not a straightforward task – one would have to write SQL statements, mix many levels of try/catch/finally code, convert SQL types to and from Java, etc.

Therefore, in order for database unit tests to thrive, it is necessary to alleviate the “database burden” on developers. We’ll start our work using pure JDBC (Java Database Connectivity). Then, we’ll introduce Spring as the framework used for our application. Finally, we’ll move to ORM (Object Relational Mapping) and Hibernate.

**JDBC** JDBC (Java Database Connectivity) is a Java API (application programming interface) that defines how a client may access a database. JDBC provides methods to query and update data in a relational database.

**ORM** ORM (Object Relational Mapping) is a programming technique for converting data between relational databases and object-oriented programming languages and vice-versa.

**HIBERNATE** Hibernate is an object-relational mapping framework for Java. It provides the facilities for mapping an object-oriented domain model to relational database tables. Hibernate manages the incompatibilities between the object-oriented model and the relational database model by replacing the direct database access with object manipulation.

### 19.1.3  Unit tests must be fast to run

Let's say you overcame the first two issues and have a nice environment, with hundreds of unit tests exercising the objects that access the database, and where a developer can easily add new ones. All seems nice, but when a developer runs the build (and they should do that many times a day, at least after updating their workspace and before submitting changes to the source control system), it takes ten minutes for the build to complete, nine of them spent in the database tests. What should you do then?

This is the hardest issue, as it cannot be always solved. Typically, the delay is caused by the database access per se, as the database is probably a remote server, accessed by dozens of users. A possible solution then is to move the database closer to the developer, by either using an embedded database (if either the application uses standard SQL that enables a database switch or the application uses an ORM framework) or locally installing lighter versions of the database.

**DEFINITION: EMBEDDED DATABASE** An embedded database is a database that is bundled within an application, instead of being managed by external servers (which is the typical scenario). There is a broad range of embedded databases available for Java applications, most of them based on open-source projects - like H2 ([http://h2database.com](http://h2database.com/)), HSQLDB ([http://hsqldb.org](http://hsqldb.org/)), Apache Derby (http://db.apache.org/derby). Notice that the fundamental characteristic of an embedded database is the fact that it is managed by the application, and not the language it is written in. For instance, both HSQLDB and Derby supports client/server mode (besides the embedded option), while SQLite (which is a C-based product) could also be embedded in a Java application.

In the next sections, we will see how we start with a pure JDBC application and with an embedded database. Then, we move to introduce Spring and Hibernate as ORM (Object Relational Mapping) framework and also take steps to solve the Database Unit Testing Mismatch.

## 19.2  Testing a JDBC application

JDBC (Java Database Connectivity) is a Java API (application programming interface) that defines how a client may access a database. JDBC provides methods to query and update data in a relational database.

It was first released as part of the Java Development Kit (JDK) 1.1 in 1997. Since then, it has been part of the Java Platform, Standard Edition (Java SE). Being one of the early API in use in Java and designed for the largely spread database applications, it may be still encountered in projects, even not mixed with any other technology.

In our demonstration, we’ll start from a pure JDBC application, then introduce Spring and Hibernate, and test all these applications. This will prove how these kinds of database applications can be tested, and also show how the Database Unit Testing Mismatch is reduced.

At Tested Data Systems, the old flights management application was providing the possibility to persist the information into a database. It is George’s job to take this application over, analyze it and then move it to the new present-day technologies.

The JDBC application that George receives and needs to take over contains a `Country` class, describing the countries of the passengers of the flights management application that we have already met across the chapters (listing 19.1).

Listing 19.1 The Country class

```
public class Country {``  private String name;                          #A``  private String codeName;                        #A`` ``  public Country(String name, String codeName) {             #B``    this.name = name;                          #B``    this.codeName = codeName;                      #B``  }                                    #B`` ``  public String getName() {                        #A``    return name;                            #A``  }                                    #A`` ``  public void setName(String name) {                   #A``    this.name = name;                          #A``  }                                    #A`` ``  public String getCodeName() {                      #A``    return codeName;                          #A``  }                                    #A`` ``  public void setCodeName(String codeName) {               #A``    this.codeName = codeName;                      #A``  }                                    #A`` ``  @Override                                #C``  public String toString() {                       #C``    return "Country{" +                         #C``        "name='" + name + '\'' +                  #C``        ", codeName='" + codeName + '\'' +             #C``        '}';                            #C``  }                                    #C `` ``  @Override                                #D``  public boolean equals(Object o) {                    #D``    if (this == o) return true;                     #D``    if (o == null || getClass() != o.getClass()) return false;     #D``    Country country = (Country) o;                   #D``    return Objects.equals(name, country.name) &&            #D``        Objects.equals(codeName, country.codeName);         #D``  }                                    #D`` ``  @Override                                #D``  public int hashCode() {                         #D``    return Objects.hash(name, codeName);                #D``  }                                    #D``}
```

In the listing above we do the following:

·  We declare the `name` and `codeName` fields of the `Country` class, together with the corresponding getters and setters (#A).

·  We create a constructor of the `Country` class, to initialize the `name` and `codeName` fields (#B).

·  We override the `toString` method to nicely display a country (#C).

·  We override the `equals` and `hashCode` methods, to take into account the `name` and `codeName` fields (#D).

The application that George is taking over is using the embedded H2 database for testing purposes. The Maven pom.xml file will include the JUnit 5 dependencies and the H2 dependencies (listing 19.2).

Listing 19.2 The Maven pom.xml dependencies

<dependencies>   <dependency>      <groupId>org.junit.jupiter</groupId>      <artifactId>junit-jupiter-api</artifactId>      <version>5.6.0</version>      <scope>test</scope>   </dependency>   <dependency>      <groupId>org.junit.jupiter</groupId>      <artifactId>junit-jupiter-engine</artifactId>      <version>5.6.0</version>      <scope>test</scope>   </dependency>   <dependency>      <groupId>com.h2database</groupId>      <artifactId>h2</artifactId>      <version>1.4.193</version>   </dependency></dependencies>

The application manages the connections to the database and the operations against the database tables through the `ConnectionManager` and `TablesManager` classes.

Listing 19.3 The ConnectionManager class

`public class ConnectionManager {``  ``  private static Connection connection;                  #A``  ``  public static Connection openConnection() {`` ``    try {``      Class.forName("org.h2.Driver");                 #B``      connection = DriverManager.getConnection("jdbc:h2:~/country",  #C``        "sa",                            #C``        ""                             #C``        );                             #C``      return connection;                       #D``    } catch(ClassNotFoundException | SQLException e) {         #E``      throw new RuntimeException(e);                 #E``    }                                  #E``  }`` ``  public static void closeConnection() {``    if (null != connection) {                      #F``      try {``        connection.close();                     #G``      } catch(SQLException e) {                    #H``        throw new RuntimeException(e);                #H``      }                                #H``    }``  }``}`

In the listing above, we do the following:

·  We declare a `Connection` type `connection` field (#A).

·  In the `openConnection` method, we load the H2 driver (#B) and we initialize the previously declared `connection` field to access the H2 `country` database using JDBC, `sa` as a user and no password (#C). If everything goes fine, we return the initialized `connection` (#D).

·  If the H2 driver class hasn’t been found or we encounter an `SQLException`, we catch it and re-throw a `RuntimeException` (#E).

·  In the `closeConnection` method, we first check the `connection` not to be null (#F), then we try to close it (#G). In case an `SQLException` occurs, we catch it and re-throw a `RuntimeException` (#H).

Listing 19.4 The TablesManager class

`public class TablesManager {``  ``  public static void createTable() {``    String sql = "CREATE TABLE COUNTRY( ID IDENTITY,           #A``           NAME VARCHAR(255), CODE_NAME VARCHAR(255) );";     #A``    executeStatement(sql);                        #B``  }`` ``  public static void dropTable() {``    String sql = "DROP TABLE IF EXISTS COUNTRY;";            #C``    executeStatement(sql);                        #D``  }``  ``  private static void executeStatement(String sql) {``   PreparedStatement statement;                     #E``    ``    try {``      Connection connection = openConnection();            #F``      statement = connection.prepareStatement(sql);          #G``      statement.executeUpdate();                   #H``      statement.close();                       #I``    } catch (SQLException e) {``      throw new RuntimeException(e);                 #J``    } finally {``      closeConnection();                       #K``    }``  }`` ``}`

In the listing above, we do the following:

·  In the `createTable` method, we declare an SQL CREATE TABLE statement that creates the COUNTRY table with an ID identity field and the NAME and CODE_NAME fields of type VARCHAR (#A). Then, we execute this statement (#B).

·  In the `dropTable` method, we declare an SQL DROP TABLE statement that drops the COUNTRY table, if it exists (#C). Then, we execute this statement (#D).

·  The `executeStatement` method declares a `PreparedStatement` variable (#E). Then, it opens a connection (#F), prepares the statement (#G), executes it (#H) and closes it (#I). If an `SQLException` is caught, we re-throw a `RuntimeException` (#J). No matter if the statement is successfully executed or not, we close the connection (#K).

The application declares a `CountryDao` class, an implementation of the DAO (Data Access Object) pattern, which provides an abstract interface to the database and executes queries against it.

 

Listing 19.5 The CountryDao class

`public class CountryDao {``  private static final String GET_ALL_COUNTRIES_SQL =           #A``      "select * from country";                     #A``  private static final String GET_COUNTRIES_BY_NAME_SQL =         #B``      "select * from country where name like ?";            #B`` ``  public List<Country> getCountryList() {``   List<Country> countryList = new ArrayList<>();            #C`` ``   try {``     Connection connection = openConnection();             #D``     PreparedStatement statement =                   #E``           connection.prepareStatement(GET_ALL_COUNTRIES_SQL);  #E``     ResultSet resultSet = statement.executeQuery();          #E`` ``     while (resultSet.next()) {                     #F``      countryList.add(new Country(resultSet.getString(2),       #F``              resultSet.getString(3)));            #F``     }``     statement.close();                         #G``   } catch (SQLException e) {                      #H``     throw new RuntimeException(e);                   #H``   } finally {``     closeConnection();                         #I``   }``   return countryList;                          #J``  }`` ``  public List<Country> getCountryListStartWith(String name) {``   List<Country> countryList = new ArrayList<>();            #K`` ``   try {``     Connection connection = openConnection();             #L``     PreparedStatement statement =                   #M``          connection.prepareStatement(GET_COUNTRIES_BY_NAME_SQL); #M``     statement.setString(1, name + "%");                #M``     ResultSet resultSet = statement.executeQuery();          #M`` ``     while (resultSet.next()) {                     #N``      countryList.add(new Country(resultSet.getString(2),       #N``              resultSet.getString(3)));            #N``     }``     statement.close();                         #O``   } catch (SQLException e) {                      #P``     throw new RuntimeException(e);                   #P``   } finally {``     closeConnection();                         #Q``   }``   return countryList;                          #R``  }``}`

In the listing above, we do the following:

·  We declare two SQL SELECT statements, to get all the countries from the COUNTRY table (#A) and to get the countries whose names match a pattern (#B).

·  In the `getCountryList` method we initialize an empty country list (#C), we open a connection (#D), prepare the statement and execute it (#E).

·  We pass through all the results returned from the database and add them to the countries list (#F). Then, we close the statement (#G). If an `SQLException` is caught, we re-throw a `RuntimeException` (#H). No matter if the statement is successfully executed or not, we close the connection (#I). We return the countries list at the end of the method (#J).

·  In the `getCountryListStartWith` method we initialize an empty country list (#K), we open a connection (#L), prepare the statement and execute it (#M).

·  We pass through all the results returned from the database and add them to the countries list (#N). Then, we close the statement (#O). If an `SQLException` is caught, we re-throw a `RuntimeException` (#P). No matter if the statement is successfully executed or not, we close the connection (#Q). We return the countries list at the end of the method (#R).

Moving to the side of the test, we have here two classes: `CountriesLoader`, which is populating the database and makes sure that it is in a known state and `CountriesDatabaseTest` that is effectively testing the interaction of the application with the database.

Listing 19.6 The CountriesLoader class

`public class CountriesLoader {`` ``  private static final String LOAD_COUNTRIES_SQL =            #A``      "insert into country (name, code_name) values ";`` ``  public static final String[][] COUNTRY_INIT_DATA = {          #B``     { "Australia", "AU"}, { "Canada", "CA" }, { "France", "FR" },   #B``     { "Germany", "DE" }, { "Italy", "IT" }, { "Japan", "JP" },    #B``     { "Romania", "RO" },{ "Russian Federation", "RU" },        #B``     { "Spain", "ES" }, { "Switzerland", "CH" },            #B``     { "United Kingdom", "UK" }, { "United States", "US" } };     #B`` ``  public void loadCountries() {``    for (String[] countryData : COUNTRY_INIT_DATA) {          #C``      String sql = LOAD_COUNTRIES_SQL + "('" + countryData[0] + "',  #D``              '" + countryData[1] + "');";           #D`` ``      try {``        Connection connection = openConnection();          #E``        PreparedStatement statement =                #F``           connection.prepareStatement(sql);           #F``        statement.executeUpdate();                 #F``        statement.close();                     #F``      } catch (SQLException e) {                   #G``        throw new RuntimeException(e);               #G ``      } finally {``        closeConnection();                     #H``      }``    }``  }``}`

In the listing above, we do the following:

·  We declare one SQL INSERT statement, to insert a country into the COUNTRY table (#A). We then declare the initialization data for the countries to be inserted (#B).

·  In the `loadCountries` method, we browse the initialization data for the countries (#C) and build the SQL query that is inserting each particular country (#D).

·  We open a connection (#E), prepare the statement. execute it and close it (#F). If an `SQLException` is caught, we re-throw a `RuntimeException` (#G). No matter if the statement is successfully executed or not, we close the connection (#H).

Listing 19.7 The CountriesDatabaseTest class

`import static com.manning.junitbook.databases.CountriesLoader.COUNTRY_INIT_DATA;     #A``[…]`` ``public class CountriesDatabaseTest {``  private CountryDao countryDao = new CountryDao();            #A``  private CountriesLoader countriesLoader = new CountriesLoader();    #A``  ``  private List<Country> expectedCountryList = new ArrayList<>();     #B``  private List<Country> expectedCountryListStartsWithA =         #C``             new ArrayList<>();                #C`` ``  @BeforeEach                               #D``  public void setUp() {``    TablesManager.createTable();                    #E``    initExpectedCountryLists();                     #F``    countriesLoader.loadCountries();                  #G``  }`` ``  @Test``  public void testCountryList() {``    List<Country> countryList = countryDao.getCountryList();      #H``    assertNotNull(countryList);                     #I``    assertEquals(expectedCountryList.size(), countryList.size());    #J``    for (int i = 0; i < expectedCountryList.size(); i++) {       #K``      assertEquals(expectedCountryList.get(i), countryList.get(i));  #K``    }``  }`` ``  @Test``  public void testCountryListStartsWithA() {``    List<Country> countryList =                     #L``           countryDao.getCountryListStartWith("A");       #L``    assertNotNull(countryList);                     #M``    assertEquals(expectedCountryListStartsWithA.size(),         #N``           countryList.size());                  #N``    for (int i = 0; i < expectedCountryListStartsWithA.size(); i++) {  #O``      assertEquals(expectedCountryListStartsWithA.get(i),       #O``             countryList.get(i));                #O``    }``  }`` ``  @AfterEach                               #P``  public void dropDown() {``    TablesManager.dropTable();                     #Q``  }`` ``  private void initExpectedCountryLists() {``     for (int i = 0; i < COUNTRY_INIT_DATA.length; i++) {        #R``       String[] countryInitData = COUNTRY_INIT_DATA[i];        #S``       Country country = new Country(countryInitData[0],       #S``                  countryInitData[1]);           #S``       expectedCountryList.add(country);               #T``       if (country.getName().startsWith("A")) {            #U``         expectedCountryListStartsWithA.add(country);        #U``       }``     }``   }``}`

In the listing above, we do the following:

·  We statically import the countries data from `CountriesLoader` and we initialize a `CountryDao` and a `CountriesLoader` (#A). We initialize an empty list of expected countries (#B) and an empty list of expected countries that start with ‘A’ (#C). We can do this for any letter, but our test is looking now for the countries with names starting with ‘A’.

·  We mark the `setUp` method with the `@BeforeEach` annotation, to be executed before each test (#D). Inside it, we create the empty COUNTRY table into the database (#E), we initialize the expected countries list (#F) and we load the countries into the database (#G).

·  In the `testCountryList` method, we initialize the countries list from the database by using the `getCountryList` from the `CountryDao` class (#H). Then, we check that the list we have obtained is not null (#I), it has the expected size (#J) and that its content is the expected one (#K).

·  In the `testCountryListStartsWithA` method, we initialize the countries list starting with ‘A’ from the database by using the `getCountryListStartWith` from the `CountryDao` class (#L). Then, we check that the list we have obtained is not null (#M), it has the expected size (#N) and that its content is the expected one (#O).

·  We mark the `dropDown` method with the `@AfterEach` annotation, to be executed after each test (#P). Inside it, we drop the COUNTRY table from the database (#Q).

·  In the `initExpectedCountryLists` method we browse the countries initialization data (#R), create a `Country` object at each step (#S) and add it to the expected countries list (#T). If the name of the country starts with ‘A’, we also add it to the expected countries list whose names start with ‘A’ (#U).

The tests are successfully running, as shown in fig. 19.1.

![img](http://localhost:8000/39e2591b-e785-4a53-9b15-9c0cdeb566c1/OEBPS/Images/19_img_0001.jpg)

Figure 19.1 Successfully running the tests from the JDBC application that checks the interaction with the COUNTRY table

This is the state of the application that George has to take over, in order to improve the way it is tested. The application is accessing and testing the database through JDBC, which requires a lot of tedious code to do the following:

·  Create and open the connection

·  Specify, prepare and execute the statement

·  Iterate through the results

·  Do the work for each iteration

·  Process the exceptions

·  Close the connection

George will now look for means to reduce the “database burden” from the developers so that they are able to improve the way tests are written and to reduce the Database Unit Testing Mismatch.

## 19.3  Testing a Spring JDBC application

Spring is a lightweight but at the same time flexible and universal framework used for creating Java applications. We have already introduced the testing of Spring applications in our previous chapters. George has decided to introduce Spring in the database application that he has to take over in order to reduce the “database burden” and to handle through the Spring Inversion of Control some of the tasks of the interaction with the database.

The application will keep the `Country` class untouched, and George will make some other changes for the migration to Spring. First, he will introduce the new dependencies into the Maven pom.xml file.

Listing 19.8 The new dependencies introduced into the Maven pom.xml file

<dependency>                                                             #A<groupId>org.springframework</groupId>                            #A<artifactId>spring-context</artifactId>                           #A<version>5.2.1.RELEASE</version>                                  #A</dependency>                                                            #A<dependency>                                                             #B<groupId>org.springframework</groupId>                            #B<artifactId>spring-jdbc</artifactId>                              #B<version>5.2.1.RELEASE</version>                                  #B</dependency>                                                            #B<dependency>                                                             #C<groupId>org.springframework</groupId>                            #C<artifactId>spring-test</artifactId>                              #C<version>5.2.1.RELEASE</version>                                  #C</dependency> 

In the listing above, we have added the following dependencies:

·  `spring-context`, the dependency for the Spring Inversion of Control container (#A).

·  `spring-jdbc`, as the application is still using JDBC to access the database. The control of working with connections, preparing and executing statements, processing exceptions is handled by Spring (#B).

·  `spring-test`, the dependency that provides support for writing tests with the help of Spring and that is necessary to use the `SpringExtension` and the `@ContextConfiguration` annotation. (#C).

In the `test/resources` project folder, George will insert two files, one to create the database schema and one to configure the Spring context of the application.

 

Listing 19.9 The db-schema.sql file

`create table country( id identity , name varchar (255) , code_name varchar (255) );`

In the listing above, we are creating the COUNTRY table with three fields: ID (identity field), NAME and CODE_NAME (VARCHAR type).

Listing 19.10 The application-context.xml file

`<jdbc:embedded-database id="dataSource" type="H2">             #A``  <jdbc:script location="classpath:db-schema.sql"/>            #A``</jdbc:embedded-database>                          #A`` ``<bean id="countryDao"                            #B``   class="com.manning.junitbook.databases.dao.CountryDao">        #B``    <property name="dataSource" ref="dataSource"/>            #B``</bean>                                   #B`` ``<bean id="countriesLoader"                         #C class="com.manning.junitbook.databases.CountriesLoader">       #C``  <property name="dataSource" ref="dataSource"/>              #C``</bean>                                   #C`

In the listing above, we instruct the Spring container to create three beans:

·  `dataSource`, pointing to a JDBC embedded database of type H2. We initialize the database with the help of the `db-schema.sql` file from listing 19.9, which is on the classpath (#A).

·  `countryDao`, the DAO bean for executing SELECT queries to the database (#B). It has a `dataSource` property pointing out to the previously declared `dataSource` bean.

·  `countriesLoader`, the bean that initializes the content of the database and brings it to a known state (#C). It also has a `dataSource` property pointing out to the previously declared `dataSource` bean.

George will change the `CountriesLoader` class that creates the countries from the database and sets it in a known state.

Listing 19.11 The CountriesLoader class

`public class CountriesLoader extends JdbcDaoSupport             #A``{`` ``  private static final String LOAD_COUNTRIES_SQL =            #B``      "insert into country (name, code_name) values ";`` ``  public static final String[][] COUNTRY_INIT_DATA = {          #C``     { "Australia", "AU"}, { "Canada", "CA" }, { "France", "FR" },   #C``     { "Germany", "DE" }, { "Italy", "IT" }, { "Japan", "JP" },    #C``     { "Romania", "RO" },{ "Russian Federation", "RU" },        #C``     { "Spain", "ES" }, { "Switzerland", "CH" },            #C``  { "United Kingdom", "UK" }, { "United States", "US" } };         #C`` ``  public void loadCountries() {``  for (String[] countryData : COUNTRY_INIT_DATA) {            #D``   String sql = LOAD_COUNTRIES_SQL + "('" + countryData[0] + "',     #E``           '" + countryData[1] + "');";              #E``    getJdbcTemplate().execute(sql);                   #E``  }``  }`` ``}`

In the listing above, we do the following:

·  We are declaring the `CountriesLoader` class as extending `JdbcDaoSupport` (#A).

·  We declare one SQL INSERT statement, to insert a country into the COUNTRY table (#B). We then declare the initialization data for the countries to be inserted (#C).

·  In the `loadCountries` method, we browse the initialization data for the countries (#D), build the SQL query that is inserting each particular country and execute it against the database (#E). With the Spring Inversion of Control approach, there is no need to do the previous tedious jobs: open the connection, prepare the statement. execute it and close it, treat the exceptions and close the connection.

Spring JDBC classes

JdbcDaoSupport is a Spring JDBC class that facilitates configuring and transferring the database parameters. If a class extends JdbcDaoSupport, JdbcDaoSupport hides how a JdbcTemplate is created.

JdbcTemplate is the central class in the package org.springframework.jdbc.core. getJdbcTemplate is a final method from the JdbcDaoSupport class that provides access to an already initialized JdbcTemplate object that executes SQL queries, iterates over results and catches JDBC exceptions.

 

George will also change the tested code to use Spring and to reduce here as well the “database burden” on the side of the developers. He will first implement the `CountryRowMapper` class that is taking care of the mapping rules between the columns from the COUNTRY database table and the fields of the application `Country` class.

Listing 19.12 The CountryRowMapper class

`public class CountryRowMapper implements RowMapper<Country> {        #A``  public static final String NAME = "name";                #B``  public static final String CODE_NAME = "code_name";           #B`` ``  @Override``  public Country mapRow(ResultSet resultSet, int i) throws SQLException { #C``   Country country = new Country(resultSet.getString(NAME),       #C``      resultSet.getString(CODE_NAME));                 #C``   return country;                            #D``  }``}`

In the listing above, we do the following:

·  We are declaring the `CountryRowMaper` class as implementing `RowMapper` (#A). `RowMapper` is a Spring JDBC interface that is doing mapping of the `ResultSet` obtained by accessing a database to certain objects.

·  We declare the string constants to be used in the class representing the names of the table columns (#B). The class will define once how to map the columns to the object fields and may be reused. There is no more need to set the parameters of the statement each time, as we were doing in the JDBC version.

·  We override the `mapRow` method inherited from the `RowMapper` interface. We are getting the two string parameters from the `ResultSet` coming from the database and build a `Country` object (#C) that we return at the end of the method (#D).

George will change the existing `CountryDao` class to use Spring to interact with the database.

Listing 19.13 The CountryDao class

`public class CountryDao extends JdbcDaoSupport               #A``{``  private static final String GET_ALL_COUNTRIES_SQL =           #B``      "select * from country";                     #B``  private static final String GET_COUNTRIES_BY_NAME_SQL =         #B``      "select * from country where name like :name";          #B`` ``  private static final CountryRowMapper COUNTRY_ROW_MAPPER =        #C``      new CountryRowMapper();                     #C`` ``  public List<Country> getCountryList() {``   List<Country> countryList =                      #D``    getJdbcTemplate().query(GET_ALL_COUNTRIES_SQL, COUNTRY_ROW_MAPPER); #D``   return countryList;                          #D``  }`` ``  public List<Country> getCountryListStartWith(String name) {``    NamedParameterJdbcTemplate namedParameterJdbcTemplate =       #E``      new NamedParameterJdbcTemplate(getDataSource());        #E``    SqlParameterSource sqlParameterSource =               #F``      new MapSqlParameterSource("name", name + "%");         #F``    return namedParameterJdbcTemplate.query(GET_COUNTRIES_BY_NAME_SQL, #G``      sqlParameterSource, COUNTRY_ROW_MAPPER);             #G``  }`` ``}`

In the listing above, we do the following:

·  We are declaring the `CountryDao` class as extending `JdbcDaoSupport` (#A).

·  We declare two SQL SELECT statements, to get all the countries from the COUNTRY table and to get the countries whose names match a pattern (#B). Into the second statement, we have replaced the parameter with a named parameter (:name) and we are going to use it this way into the class.

·  We are initializing a `CountryRowMapper` instance, the class that we have previously created (#C).

·  In the `getCountryList` method, we query the COUNTRY table using the SQL that returns all the countries and the `CountryRowMapper` that will match the columns from the table to the fields from the `Country` object. We are directly returning a list of `Country` objects (#D).

·  In the `getCountryListStartWith` method, we initialize a `NamedParameterJdbcTemplate` variable (#E). `NamedParameterJdbcTemplate` allows the use of named parameters instead of the previously used '?' placeholders. The `getDataSource` method which is the argument of the `NamedParameterJdbcTemplate` constructor is a final method inherited from `JdbcDaoSupport` and it returns the JDBC `DataSource` used by a DAO.

·  We initialize an `SqlParameterSource` variable (#F). `SqlParameterSource` defines the functionality of the objects that can offer parameter values for named SQL parameters and can serve as an argument for `NamedParameterJdbcTemplate` operations.

·  We query the COUNTRY table using the SQL that returns all the countries having names starting with ‘A’ and the `CountryRowMapper` that will match the columns from the table to the fields from the `Country` object (#G).

George will finally change the existing `CountriesDatabaseTest` to take advantage of the Spring JDBC approach.

Listing 19.14 The CountriesDatabaseTest class

`@ExtendWith(SpringExtension.class)                     #A``@ContextConfiguration("classpath:application-context.xml")         #B``public class CountriesDatabaseTest {`` ``  @Autowired                               #C``  private CountryDao countryDao;                     #C`` ``  @Autowired                               #D``  private CountriesLoader countriesLoader;                #D`` ``  private List<Country> expectedCountryList = new ArrayList<Country>();  #E``  private List<Country> expectedCountryListStartsWithA =         #F``     new ArrayList<Country>();                     #F`` ``  @BeforeEach                               #G``  public void setUp() {``    initExpectedCountryLists();                     #H``    countriesLoader.loadCountries();                  #I``  }`` ``  ``  @Test``  @DirtiesContext                             #J``  public void testCountryList() {``    List<Country> countryList = countryDao.getCountryList();      #K``    assertNotNull(countryList);                     #L``    assertEquals(expectedCountryList.size(), countryList.size());    #M``    for (int i = 0; i < expectedCountryList.size(); i++) {       #N``      assertEquals(expectedCountryList.get(i), countryList.get(i));  #N``    }                                  #N``  }`` ``  @Test``  @DirtiesContext                             #J``  public void testCountryListStartsWithA() {``    List<Country> countryList = countryDao.getCountryListStartWith("A");#O``    assertNotNull(countryList);                     #P ``    assertEquals(expectedCountryListStartsWithA.size(),         #Q``           countryList.size());                  #Q``    for (int i = 0; i < expectedCountryListStartsWithA.size(); i++) {  #R``      assertEquals(expectedCountryListStartsWithA.get(i),       #R``             countryList.get(i));                #R``    }``  }`` ``  private void initExpectedCountryLists() {``   for (int i = 0; i < CountriesLoader.COUNTRY_INIT_DATA.length; i++) {``     String[] countryInitData = CountriesLoader.COUNTRY_INIT_DATA[i]; #S``     Country country = new Country(countryInitData[0],         #T``         countryInitData[1]);                   #T ``     expectedCountryList.add(country);                 #U``     if (country.getName().startsWith("A")) {             #V``       expectedCountryListStartsWithA.add(country);         #V``     }                                 #V``   }``  }``}`

In the listing above, we do the following:

·  We are annotating the test class to be extended with `SpringExtension` (#A). `SpringExtension` is used to integrate the Spring TestContext with the JUnit 5 Jupiter Test.

·  We are also annotating the test class to look for the context configuration into the `application-context.xml` file from the classpath (#B).

·  We are auto-wiring a `CountryDao` (#C) and a `CountriesLoader` bean (#D), which are declared into the `application-context.xml` file.

·  We initialize an empty list of expected countries (#E) and an empty list of expected countries that start with ‘A’ (#F).

·  We mark the `setUp` method with the `@BeforeEach` annotation, to be executed before each test (#G). Inside it, we initialize the expected countries list (#H) and we load the countries into the database (#I). The database is initialized by Spring, and we have eliminated its manual initialization.

·  We annotate the test methods with the `@DirtiesContext` annotation (#J). This annotation is used when a test has modified the context (in our case, the state of the embedded database). This will reduce the “database burden” from the developers. Subsequent tests will be supplied with a new unmodified context.

·  In the `testCountryList` method, we initialize the countries list from the database by using the `getCountryList` from the `CountryDao` class (#K). Then, we check that the list we have obtained is not null (#L), it has the expected size (#M) and that its content is the expected one (#N).

·  In the `testCountryListStartsWithA` method, we initialize the countries list starting with ‘A’ from the database by using the `getCountryListStartWith` from the `CountryDao` class (#O). Then, we check that the list we have obtained is not null (#P), it has the expected size (#Q) and that its content is the expected one (#R).

·  In the `initExpectedCountryLists` method we browse the countries initialization data (#S), create a `Country` object at each step (#T) and add it to the expected countries list (#U). If the name of the country starts with ‘A’, we also add it to the expected countries list whose names start with ‘A’ (#V).

The tests are successfully running, as shown in fig. 19.2.

![img](http://localhost:8000/39e2591b-e785-4a53-9b15-9c0cdeb566c1/OEBPS/Images/19_img_0002.jpg)

Figure 19.2 Successfully running the tests from the Spring JDBC application that checks the interaction with the COUNTRY table

The application is now accessing and testing the database through Spring JDBC, this approach has a few advantages:

·  It no longer requires the previously existing large amount of tedious code.

·  We no longer create and open the connections by ourselves.

·  We no longer prepare and execute the statement, process the exceptions or close the connections.

·  Mainly, we must take care of the application context configuration to be handled by Spring and we also must take care of the row mapper. Otherwise, we have to specify only the statement and iterate through the results.

Spring allows configuration alternatives, as we have already demonstrated in the previous chapters. We have used here the XML-based configuration for the tests, as being easier to change, especially by less technical people. Be aware that the Spring Framework is a large topic and for this chapter, we are mostly discussing the things related to testing database applications with JUnit 5. For a comprehensive book on this topic, also showing in detail the configuration possibilities, please refer to the Manning “Spring in Action” by Craig Walls (https://www.manning.com/books/spring-in-action-fifth-edition).

George will consider further alternatives to test the interaction with the database and to keep a reduced “database burden” for the developers. We’ll see more in the next sections.

## 19.4  Testing a Hibernate application

JPA (Java Persistence API) is the specification describing the management of the relational data, the API that the client will operate with and the metadata for the object-relational mapping.

Hibernate is an object-relational mapping framework for Java, implementing the JPA specifications. It existed before the first publishing of the JPA specifications. Therefore, Hibernate has also retained its old native API, being able to offer some non-standard features. For our demonstration, we’ll use the standard Java Persistence API.

Hibernate provides the facilities for mapping an object-oriented domain model to relational database tables. Hibernate manages the incompatibilities between the object-oriented model and the relational database model by replacing the direct database access with object handling functions.

Working with Hibernate provides a series of advantages for accessing and testing the database:

·  Speeding development. It eliminates repetitive code like mapping query result columns to object fields and vice-versa.

·  Making data access more abstract and portable. The ORM implementation classes know how to write vendor-specific SQL, so we do not have to.

·  Cache management. Entities are cached in memory, thereby reducing the load on the database.

·  Generating boilerplate code for basic CRUD operations (Create, Read, Update, Delete).

George will first introduce the Hibernate dependency into the Maven pom.xml configuration (listing 19.15).

Listing 19.15 The Hibernate dependency introduced into the Maven pom.xml file

`<dependency>``  <groupId>org.hibernate</groupId>``  <artifactId>hibernate-core</artifactId>``  <version>5.4.9.Final</version>``</dependency>`

Then, George will change the Country class to annotate it as an entity and to annotate the fields from it as columns in a table.

Listing 19.16 The annotated Country class

`@Entity                                   #A``@Table(name = "COUNTRY")                          #B``public class Country {``  @Id                                   #C``  @GeneratedValue(strategy = GenerationType.IDENTITY)           #D``  @Column(name = "ID")                          #E``  private int id;`` ``  @Column(name = "NAME")                         #F``  private String name;`` ``  @Column(name = "CODE_NAME")                       #F``  private String codeName;`` ``  […]``}`

In the listing above we do the following:

·  We annotate the `Country` class with `@Entity`, meaning that it has the ability to represent objects in a database (#A). The corresponding table into the database is provided by the `@Table` annotation and is named COUNTRY (#B).

·  The `id` field is marked as the primary key (#C), its value is automatically generated using a database identity column (#D). The corresponding table column is ID (#E).

·  We also mark the corresponding columns of the `name` and `codeName` fields in the class by annotating them with `@Column` (#F).

The `persistence.xml` file is the standard configuration for Hibernate. It is located in the `test/resources/META-INF` folder.

Listing 19.17 The persistence.xml file

<persistence-unit name="manning.hibernate">                                 #A        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider> #B        <class>com.manning.junitbook.databases.model.Country</class>        #C         <properties>            <property name="javax.persistence.jdbc.driver"                  #D                            value="org.h2.Driver"/>                         #D            <property name="javax.persistence.jdbc.url"                     #E                            value="jdbc:h2:mem:test;DB_CLOSE_DELAY=-1"/>    #E            <property name="javax.persistence.jdbc.user" value="sa"/>       #F            <property name="javax.persistence.jdbc.password" value=""/>     #F            <property name="hibernate.dialect"                              #G                            value="org.hibernate.dialect.H2Dialect"/>       #G            <property name="hibernate.show_sql" value="true"/>              #H            <property name="hibernate.hbm2ddl.auto" value="create"/>        #I        </properties></persistence-unit>

In the listing above we do the following:

·  We specify the persistence unit as `manning.hibernate` (#A). The `persistence.xml` file must define a persistence unit with a unique name in the currently scoped classloader.

·  We specify the provider, meaning the underlying implementation of the JPA (Java Persistence API) EntityManager (#B). An EntityManager manages a set of persistent objects and has an API to insert new objects and read/update/delete the existing ones. In our case, the EntityManager is Hibernate, currently the most popular JPA implementation.

·  We define the entity class that is managed by Hibernate as the `Country` class from our application (#C).

·  We specify the JDBC driver as H2, as this is the database type is use (#D).

·  We specify the URL of the H2 database. In addition, the `DB_CLOSE_DELAY=-1` will keep the database open and the content of the in-memory database as long as the virtual machine is alive (#E).

·  We specify the credentials to access the database – user and password (#F).

·  We set the SQL dialect for the generated query to H2Dialect (#G), we show the generated SQL query on the console (#H).

·  We create the database schema from the scratch every time when we execute the tests (#I).

George will finally rewrite the test that verifies the functionality of the database application, this time using Hibernate.

Listing 19.18 The CountriesHibernateTest file

`public class CountriesHibernateTest {`` ``  private EntityManagerFactory emf;                    #A``  private EntityManager em;                        #B`` ``  private List<Country> expectedCountryList =               #C``             new ArrayList<>();                #C``  private List<Country> expectedCountryListStartsWithA =         #C``             new ArrayList<>();                #C`` ``  public static final String[][] COUNTRY_INIT_DATA = {          #D``    { "Australia", "AU" }, { "Canada", "CA" }, { "France", "FR" },   #D``    { "Germany", "DE" }, { "Italy", "IT" }, { "Japan", "JP" },     #D``    { "Romania", "RO" }, { "Russian Federation", "RU" },        #D``    { "Spain", "ES" }, { "Switzerland", "CH" },             #D``    { "United Kingdom", "UK" }, { "United States", "US" } };      #D`` ``  @BeforeEach                               #E``  public void setUp() {``    initExpectedCountryLists();                     #F`` ``    emf = Persistence.createEntityManagerFactory("manning.hibernate"); #G``    em = emf.createEntityManager();                   #G`` ``    em.getTransaction().begin();                    #H`` ``    for (int i = 0; i < COUNTRY_INIT_DATA.length; i++) {        #I``      String[] countryInitData = COUNTRY_INIT_DATA[i];        #I``      Country country = new Country(countryInitData[0],        #I``                 countryInitData[1]);           #I``      em.persist(country);                      #J``    }`` ``    em.getTransaction().commit();                    #H``  }`` ``  @Test``  public void testCountryList() {``    List<Country> countryList = em.createQuery(             #K``        "select c from Country c").getResultList();         #K``    assertNotNull(countryList);                     #L``    assertEquals(COUNTRY_INIT_DATA.length, countryList.size());     #M``    for (int i = 0; i < expectedCountryList.size(); i++) {       #N``      assertEquals(expectedCountryList.get(i), countryList.get(i));  #N``    }`` ``  }`` ``  @Test``  public void testCountryListStartsWithA() {``    List<Country> countryList = em.createQuery(             #O``      "select c from Country c where c.name like 'A%'").       #O``                       getResultList();       #O``    assertNotNull(countryList);                     #P``    assertEquals(expectedCountryListStartsWithA.size(),         #Q``           countryList.size());                  #Q``    for (int i = 0; i < expectedCountryListStartsWithA.size(); i++) {  #R``      assertEquals(expectedCountryListStartsWithA.get(i),       #R``             countryList.get(i));                #R``    }``  }`` ``  @AfterEach                               #S``  public void dropDown() {``    em.close();                             #T``    emf.close();                            #T``  }`` ``  private void initExpectedCountryLists() {``    for (int i = 0; i < COUNTRY_INIT_DATA.length; i++) {        #U``      String[] countryInitData = COUNTRY_INIT_DATA[i];        #V``      Country country = new Country(countryInitData[0],        #V``                     countryInitData[1]);       #V``      expectedCountryList.add(country);                #W``      if (country.getName().startsWith("A")) {            #X``        expectedCountryListStartsWithA.add(country);        #X``      }``    }``  }``}`

In the listing above we do the following:

·  We initialize an `EntityManagerFactory` (#A) and an `EntityManager` objects (#B). `EntityManagerFactory` provides instances of `EntityManager` for connecting to the same database, while an `EntityManager` is used to access a database in a particular application.

·  We initialize an empty list of expected countries and an empty list of expected countries that start with ‘A’ (#C). We then declare the initialization data for the countries to be inserted (#D).

·  We mark the `setUp` method with the `@BeforeEach` annotation, to be executed before each test (#E). Inside it, we initialize the expected countries list (#F) and we initialize the `EntityManagerFactory` and the `EntityManager` (#G).

·  Within a transaction (#H), we initialize each country, one after the other (#I) and we persist the newly created country to the database (#J).

·  In the `testCountryList` method, we initialize the countries list from the database by using the `EntityManager` and querying the Country entity using a JPQL SELECT (#K). JPQL (Java Persistence Query Language) is an object-oriented query language, independent of the platform. It is a part of the JPA (Java Persistence API) specification. Note that we need to spell “Country”, as this is the exact name of the class, starting in upper case and having the other letters are in lower case. Then, we check that the list we have obtained is not null (#L), it has the expected size (#M) and that its content is the expected one (#N).

·  In the `testCountryListStartsWithA` method, we initialize the countries list starting with ‘A’ from the database by using the `EntityManager` and querying the Country entity using a JPQL SELECT (#O). Then, we check that the list we have obtained is not null (#P), it has the expected size (#Q) and that its content is the expected one (#R).

·  We mark the `dropDown` method with the `@AfterEach` annotation, to be executed after each test (#S). Inside it, we close the `EntityManagerFactory` and the `EntityManager` (#T).

·  In the `initExpectedCountryLists` method we browse the countries initialization data (#U), create a `Country` object at each step (#V) and add it to the expected countries list (#W). If the name of the country starts with ‘A’, we also add it to the expected countries list whose names start with ‘A’ (#X).

The tests are successfully running, as shown in fig. 19.3.

![img](http://localhost:8000/39e2591b-e785-4a53-9b15-9c0cdeb566c1/OEBPS/Images/19_img_0003.jpg)

Figure 19.3 Successfully running the tests from the Hibernate application that checks the interaction with the COUNTRY table

The application is now accessing and testing the database through Hibernate. This comes with a few advantages:

·  No more SQL code to be written inside the application. The developers are working only with Java code.

·  No more mapping of the query result columns to object fields and vice-versa.

·  Hibernate knows how to transform the operations with the implemented classes into vendor-specific SQL. So, if we change the underlying database, we will not touch the existing code, we’ll only change the Hibernate configuration and the database dialect.

George will make one more step in considering alternatives to test the interaction with the database: combining Spring and Hibernate. We’ll see this approach in the next section.

## 19.5  Testing a Spring Hibernate application

Hibernate is an object-relational mapping framework for Java. It provides the facilities for mapping an object-oriented domain model to relational database tables. Spring can take advantage of the Inversion of Control to simplify the database interaction tasks.

Integrating Hibernate and Spring, George will add the needed dependencies into the Maven pom.xml configuration file.

Listing 19.19 The Spring and Hibernate dependencies in the Maven pom.xml file

<dependency>                                                             #A<groupId>org.springframework</groupId>                            #A<artifactId>spring-context</artifactId>                           #A<version>5.2.1.RELEASE</version>                                  #A</dependency>                                                            #A<dependency>                                                             #B<groupId>org.springframework</groupId>                            #B<artifactId>spring-orm</artifactId>                               #B<version>5.2.1.RELEASE</version>                                  #B</dependency>                                                            #B<dependency>                                                             #C<groupId>org.springframework</groupId>                            #C<artifactId>spring-test</artifactId>                              #C<version>5.2.1.RELEASE</version>                                  #C</dependency>                                                            #C<dependency>                                                             #D<groupId>org.hibernate</groupId>                                  #D<artifactId>hibernate-core</artifactId>                           #D  <version>5.4.9.Final</version>                                    #D</dependency>                                                            #D

In the listing above, we have added the following dependencies:

·  `spring-context`, the dependency for the Spring Inversion of Control container (#A).

·  `spring-orm`, as the application is still using Hibernate as ORM (Object Relational Mapping) framework to access the database. The control of working with connections, preparing and executing statements, processing exceptions is handled by Spring (#B).

·  `spring-test`, the dependency that provides support for writing tests with the help of Spring and that is necessary to use the `SpringExtension` and the `@ContextConfiguration` annotation. (#C).

·  The `hibernate-code` dependency, for the interaction with the database through Hibernate (#D).

George will make some changes to the `persistence.xml` file, the standard configuration for Hibernate. Only some minimal information will remain here, as the database access control will be handled by Spring.

Listing 19.20 The persistence.xml file

`<persistence-unit name="manning.hibernate">                 #A``    <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider> #B``    <class>com.manning.junitbook.databases.model.Country</class>    #C``</persistence-unit>`

In the listing above we do the following:

·  We specify the persistence unit as `manning.hibernate` (#A). The `persistence.xml` file must define a persistence unit with a unique name in the currently scoped classloader.

·  We specify the provider, meaning the underlying implementation of the JPA (Java Persistence API) EntityManager (#B). An EntityManager manages a set of persistent objects and has an API to insert new objects and read/update/delete the existing ones. In our case, the EntityManager is Hibernate.

·  We define the entity class that is managed by Hibernate as the `Country` class from our application (#C).

So, George will move the database access configuration to the application-context.xml file that is configuring the Spring container.

Listing 19.21 The application-context.xml file

<tx:annotation-driven transaction-manager="txManager"/>                     #A<bean id="dataSource" class=                                                #B     "org.springframework.jdbc.datasource.DriverManagerDataSource">         #B     <property name="driverClassName" value="org.h2.Driver"/>               #C     <property name="url" value="jdbc:h2:mem:test;DB_CLOSE_DELAY=-1"/>      #D     <property name="username" value="sa"/>                                 #E     <property name="password" value=""/>                                   #E</bean> <bean id="entityManagerFactory" class=                                      #F     "org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">  #F     <property name="persistenceUnitName" value="manning.hibernate" />      #G     <property name="dataSource" ref="dataSource"/>                         #H     <property name="jpaProperties">         <props>           <prop key=                                                       #I           "hibernate.dialect">org.hibernate.dialect.H2Dialect</prop>       #I           <prop key="hibernate.show_sql">true</prop>                       #J           <prop key="hibernate.hbm2ddl.auto">create</prop>                 #K         </props>     </property></bean> <bean id="txManager" class=                                                 #L     "org.springframework.orm.jpa.JpaTransactionManager">                   #L     <property name="entityManagerFactory" ref="entityManagerFactory" />    #M     <property name="dataSource" ref="dataSource" />                        #N</bean> <bean class="com.manning.junitbook.databases.CountryService"/>              #O

In the listing above we do the following:

·  `<tx:annotation-driven>` tells the Spring context that we are using annotation-based transaction management configuration (#A).

·  We are configuring the access to the data source (#B), by specifying the driver as H2, as this is the database type is use (#C). We specify the URL of the H2 database. In addition, the `DB_CLOSE_DELAY=-1` will keep the database open and the content of the in-memory database as long as the virtual machine is alive (#D).

·  We specify the credentials to access the database – user and password (#E).

·  We create and `EntityManagerFactory` bean (#F). We set its properties: the persistence unit name (as defined into the persistence.xml file) (#G), the data source (defined above) (#H), the SQL dialect for the generated query to H2Dialect (#I). We show the generated SQL query on the console (#J) and we create the database schema from the scratch every time when we execute the tests (#K).

·  To process the annotation-based transaction configuration, a transaction manager bean needs to be created. We declare it (#L) and set its entity manager factory (#M) and data source (#N) properties.

·  We declare a `CountryService` bean (#O), as we’ll create this class into the code and group here the logic of the interaction with the database.

George creates the `CountryService` class that contains the logic of the interaction with the database.

Listing 19.22 The CountryService class

`public class CountryService {`` ``  @PersistenceContext                           #A``  private EntityManager em;                        #A`` ``  public static final String[][] COUNTRY_INIT_DATA =           #B``   { { "Australia", "AU" }, { "Canada", "CA" }, { "France", "FR" },   #B``    { "Germany", "DE" }, { "Italy", "IT" }, { "Japan", "JP" },     #B``    { "Romania", "RO" }, { "Russian Federation", "RU" },        #B``    { "Spain", "ES" }, { "Switzerland", "CH" },             #B``    { "United Kingdom", "UK" }, { "United States", "US" } };      #B`` ``  @Transactional                             #C``  public void init() {``    for (int i = 0; i < COUNTRY_INIT_DATA.length; i++) {        #D``      String[] countryInitData = COUNTRY_INIT_DATA[i];        #D``      Country country = new Country(countryInitData[0],        #D``                     countryInitData[1]);       #D``      em.persist(country);                      #D``    }``  }`` ``  @Transactional                             #C``  public void clear() {``    em.createQuery("delete from Country c").executeUpdate();      #E``  }`` ``  public List<Country> getAllCountries() {``    return em.createQuery("select c from Country c")          #F``          .getResultList();                    #F ``  }`` ``  public List<Country> getCountriesStartingWithA() {``    return em.createQuery(                       #G``      "select c from Country c where c.name like 'A%'")        #G``      .getResultList();                        #G``  }``}`

In the listing above we do the following:

·  We declare an `EntityManager` bean and we annotate it with `@PersistenceContext` (#A). `The EntityManager` is used to access a database and is created by the container using the information in the persistence.xml. To use it at runtime, we simply need to request it be injected into one of our components, via `@PersistenceContext`.

·  We then declare the initialization data for the countries to be inserted (#B).

·  We annotate the `init` and `clear` methods as `@Transactional` (#C). We do this as these methods are modifying the content of the database, and all this kind of methods need to be executed within a transaction.

·  We browse the initialization data for the countries, create each `Country` object and persist it within the database (#D).

·  The `clear` method will delete all countries from the Country entity using a JPQL DELETE (#E). JPQL (*Java Persistence Query Language)* is a platform-independent object-oriented query language defined as part of the JPA (Java Persistence API) specification. Note that we need to spell “Country”, as this is the exact name of the class, starting in upper case and having the other letters are in lower case.

·  The `getAllCountries` method will select all the countries from the Country entity using a JPQL SELECT (#F).

·  The `getCountriesStartingWithA` method will select all the countries from the Country entity having names starting with ‘A”, using a JPQL SELECT (#G).

George will finally modify the `CountriesHibernateTest` class, testing the logic of the interaction with the database.

Listing 19.23 The CountriesHibernateTest class

`@ExtendWith(SpringExtension.class)                     #A``@ContextConfiguration("classpath:application-context.xml")         #B``public class CountriesHibernateTest {`` ``  @Autowired                               #C``  private CountryService countryService;                 #C`` ``  private List<Country> expectedCountryList = new ArrayList<>();     #D``  private List<Country> expectedCountryListStartsWithA =         #D``               new ArrayList<>();              #D`` `` ``  @BeforeEach                               #E``  public void setUp() {``    countryService.init();                       #F``    initExpectedCountryLists();                     #G``  }`` ``  @Test``  public void testCountryList() {``    List<Country> countryList = countryService.getAllCountries();    #H  ``    assertNotNull(countryList);                     #I``    assertEquals(COUNTRY_INIT_DATA.length, countryList.size());     #J``    for (int i = 0; i < expectedCountryList.size(); i++) {       #K``      assertEquals(expectedCountryList.get(i), countryList.get(i));  #K``    }``  }`` ``  @Test``  public void testCountryListStartsWithA() {``    List<Country> countryList =                     #L``        countryService.getCountriesStartingWithA();         #L``    assertNotNull(countryList);                     #M``    assertEquals(expectedCountryListStartsWithA.size(),         #N``           countryList.size());                  #N``    for (int i = 0; i < expectedCountryListStartsWithA.size(); i++) {  #O``      assertEquals(expectedCountryListStartsWithA.get(i),       #O``             countryList.get(i));                #O``    }``  }`` ``  @AfterEach                               #P``  public void dropDown() {``    countryService.clear();                       #Q``  }`` ``  private void initExpectedCountryLists() {``    for (int i = 0; i < COUNTRY_INIT_DATA.length; i++) {        #R``      String[] countryInitData = COUNTRY_INIT_DATA[i];        #S``      Country country = new Country(countryInitData[0],        #S``                     countryInitData[1]);       #S``      expectedCountryList.add(country);                #T``      if (country.getName().startsWith("A")) {            #U``        expectedCountryListStartsWithA.add(country);        #U``      }``    }``  }``}`

In the listing above we do the following:

·  We are annotating the test class to be extended with `SpringExtension` (#A). `SpringExtension` is used to integrate the Spring TestContext with the JUnit 5 Jupiter Test. This allows us to use other Spring annotations as well (e.g. `@ContextConfiguration`, `@Transactional`), but it also requires the usage of JUnit 5 and its annotations.

·  We are also annotating the test class to look for the context configuration into the `application-context.xml` file from the classpath (#B).

·  We declare and auto-wire a `CountryService` bean. This bean will be created and injected by the Spring container (#C).

·  We initialize an empty list of expected countries and an empty list of expected countries that start with ‘A’ (#D).

·  We mark the `setUp` method with the `@BeforeEach` annotation, to be executed before each test (#E). Inside it, we initialize the database content through the `init` method from the `CountryService` class (#F) and we initialize the expected countries list (#G).

·  In the `testCountryList` method, we initialize the countries list from the database by using the `getAllCountries` method from the `CountryService` class (#H). Then, we check that the list we have obtained is not null (#I), it has the expected size (#J) and that its content is the expected one (#K).

·  In the `testCountryListStartsWithA` method, we initialize the countries list starting with ‘A’ from the database by using the `getCountriesStartingWithA` method from the `CountryService` class (#L). Then, we check that the list we have obtained is not null (#M), it has the expected size (#N) and that its content is the expected one (#O).

·  We mark the `dropDown` method with the `@AfterEach` annotation, to be executed after each test (#P). Inside it, we clear the COUNTRY table content through the clear method from the `CountryService` class (#Q).

·  In the `initExpectedCountryLists` method we browse the countries initialization data (#R), create a `Country` object at each step (#S) and add it to the expected countries list (#T). If the name of the country starts with ‘A’, we also add it to the expected countries list whose names start with ‘A’ (#U).

The tests are successfully running, as shown in fig. 19.4.

![img](http://localhost:8000/39e2591b-e785-4a53-9b15-9c0cdeb566c1/OEBPS/Images/19_img_0004.jpg)

Figure 19.4 Successfully running the tests from the Spring Hibernate application that checks the interaction with the COUNTRY table

The application is now accessing and testing the database through Spring Hibernate. This comes with a few advantages:

·  No SQL code to be written inside the application. The developers are working only with Java code.

·  No creation/opening/closing of the connections by ourselves.

·  No exception process by ourselves.

·  We have to take care mainly of the application context to be handled by Spring and to include the data source, transaction manager and entity manager factory configuration.

·  Hibernate knows how to transform the operations with the implemented classes into vendor-specific SQL. So, if we change the underlying database, we will not touch the existing code, we’ll only change the Hibernate configuration and the database dialect.

## 19.6  Comparing the approaches of testing database applications

We have followed George as he started with a simple JDBC application and moved it for use with Spring and Hibernate. We have demonstrated how things have changed and how each approach has simplified the way we test and we interact in general with the database. Our purpose was to analyze how each approach works and how the “database burden” can be reduced from the side of the developers.

We will summarize in table 19.1 the characteristics of each approach.

| Application type | Characteristics                                              |
| ---------------- | ------------------------------------------------------------ |
| JDBC             | SQL code needs to be written inside the testsNo portability between databasesFull control on what the application is doingDeveloper manual work for interacting with the database, e.g.:Create and open the connectionsSpecify, prepare and execute the statementIterate through the resultsDo the work for each iterationProcess the exceptionsClose the connection |
| Spring JDBC      | SQL code needs to be written inside the testsNo portability between databasesNeed to take care mainly of the application context configuration to be handled by Spring and of the row mapperControl on the queries that the application is executing against the databaseReduces the manual work for interacting with the database:No creation/opening/closing the connections by ourselvesNo preparation and execution of the statementsNo processing of the exceptions |
| Hibernate        | No SQL code inside the applicationDevelopers working only with Java codeNo mapping of the query result columns to object fields and vice-versaPortability between databases by changing the Hibernate configuration and the database dialectDatabase configuration handled through Java code |
| Spring Hibernate | No SQL code inside the applicationDevelopers working only with Java codeNo mapping of the query result columns to object fields and vice-versaPortability between databases by changing the Hibernate configuration and the database dialectDatabase configuration is handled by Spring, based on the information from the application context |

Table 19.1 Comparison of the alternatives of working with a database application and testing it with JDBC, Spring JDBC, Hibernate, Spring Hibernate

We notice that the introduction of at least one framework as Spring or Hibernate into the application greatly simplifies the testing of the database and developing the application itself. As these are the most popular Java frameworks from today and they provide a lot of benefits (including the work and testing of the interaction with a database, as we have demonstrated), you may consider adopting them into your project (if you haven’t done it already).

This chapter has focused on covering the alternatives of testing a database application with JUnit 5, to support you with your possible current project. The tests have covered only the insertions and selections operations, which are already providing comprehensive information about how to do it and how each alternative works in detail. The reader may extend the tests offered here to include the update and delete operations.

The next chapter will start the last part of the book, dedicated to the systematic development of applications with the help of JUnit 5. It will introduce one of the most widely spread development techniques from today: TDD (Test Driven Development).

## 19.7  Summary

This chapter has covered the following:

·  Examining the database unit testing impedance mismatch, including as challenges the fact that: unit tests must exercise code in isolation; unit tests must be easy to write and run; unit tests must be fast to run.

·  Implementing tests for a JDBC application, which requires SQL code to be written inside the tests and a lot of tedious work to be done by the developer: creating/opening/closing the connections to the database; specifying, preparing and executing the statements; handling exceptions.

·  Implementing tests for a Spring JDBC application. This still requires SQL code to be written inside the tests, but it handles the Spring container the tasks of creating/opening/closing the connections to the database; specifying, preparing and executing the statements; handling exceptions.

·  Implementing tests for a Hibernate application. No more SQL code is required, developers work only with the Java code, the application is portable to another database with minimum configuration changes. The database configuration is handled through the Java code.

·  Implementing tests for a Spring Hibernate application. No SQL code is required, developers work only with the Java code, the application is portable to another database with minimum configuration changes. Additionally, the database configuration is handled by Spring, based on the information from the application context.





# 20. Test Driven Development with JUnit 5

略

# 21. Behavior Driven Development with JUnit 5

## 21.1  Introducing Behavior Driven Development

> **Behavior Driven Development**
>
> ​	Behavior Driven Development is a software development technique that directly addresses the business requirements. It starts from the business requirements and goals and then transforms them into working features.
>
> ​	BDD encourages teams to interact and use concrete examples to communicate how the application must behave. It emerged from Test Driven Development.
>
> ​	Test Driven Development helps us to build safe software. Behavior Driven Development helps us building software providing business value.



​	The communication between the people who are involved in the same project may bring up problems and misunderstandings. Usually, the flow works this way:

* The customer communicates to the business analyst his understanding about the functionality of a feature.
* The business analyst builds the requirements for the developers, describing the way the software must work.
* The developer creates the code based on the requirements and writes unit tests to implement the new feature.
* The tester creates the test cases based on the requirements and uses them to verify the way the new feature works.

### 21.1.1  Introducing a new feature

​	The business analyst discusses with the customer to decide the software features that will be able to address the business goals.

### 21.1.2  From requirements analysis to acceptance criteria

​	

In BDD, the definition of acceptance criteria is made using the Given/When/Then keywords. More exactly:

```
Given <a context>

When <an action occurs>

Then <expect a result>

As a concrete example:

Given the flights operated by the company

When I want to travel from Bucharest to London next Wednesday

Then I should be provided 2 possible flights: 10:35 and 16:2
```

### 21.1.3  BDD benefits and challenges

​	We’ll point out a few benefits of the BDD approach:

* Address user needs. The users care less about the implementation and they are mainly interested in the application functionality. Working BDD style you get closer to addressing these needs.
* Clarity. Scenarios will clarify what software should do. Scenarios are described in simple language, easy to understand by technical and non-technical people. Ambiguities can be clarified by analyzing the scenario or by adding another scenario.
* Supports change. The scenarios will represent a part of the documentation of the software – in fact, living documentation, as it evolves simultaneously with the application. It also helps locate an incoming change. The automated acceptance tests will hinder the introduction of regressions when new changes are introduced.
* Supports automation. Scenarios can be transformed into automated tests, as the steps of the scenario are already defined.
* Focuses on adding business value. BDD will prevent introducing features that are not useful to the project. You will also be able to prioritize the functionalities.
* Reduces costs. Prioritizing the importance of the functionalities and avoiding the unnecessary ones will hinder the waste of resources and concentrate them to do exactly what is needed.



​	As challenges, BDD requires engagement and strong collaboration. It requires interaction, direct communication, and constant feedback. This may be a challenge for some persons and, in the context of the present-day globalization and distributed teams, may require language skills and fitting time zones.

## 21.2  Working BDD with Cucumber and JUnit 5

![](https://pic.imgdb.cn/item/6157d52e2ab3f51d9195a6d0.jpg)

### 21.2.1  Introducing Cucumber

> Cucumber
>
> ​    Cucumber is a Behavior Driven Development testing tool framework. It describes the application scenarios in plain English text, using a language called Gherkin. Cucumber is easy to read and understand by stakeholders and allows automation.

The main capabilities of Cucumber are:

* Scenarios or examples describe the requirements.
* A scenario is defined through a list of steps to be executed by Cucumber.
* Cucumber executes the code corresponding to the scenarios, checks that the software follows these requirements and generates a report about the success or failure of each scenario.

The main capabilities of Gherkin are:

* Gherkin defines simple grammar rules that allow Cucumber to understand English plain text.
* Gherkin documents the behavior of the system. The requirements are always up to date, as they are provided through these scenarios which represent living specifications

A Cucumber acceptance test can look this way:

**Given there is an economy flight**

**When we have a regular passenger**

**Then you can add and remove him from an economy flight**

​	Cucumber will take care to interpret the sentences starting with these keywords and generate methods which it will annotate using exactly these annotations: `@Given`, `@When` and `@Then`.

```xml
<dependency>
     <groupId>info.cukes</groupId>
     <artifactId>cucumber-java</artifactId>
     <version>1.2.5</version>
     <scope>test</scope>
</dependency>
<dependency>
     <groupId>info.cukes</groupId>
     <artifactId>cucumber-junit</artifactId>
     <version>1.2.5</version>
     <scope>test</scope>
</dependency>
```

### 21.2.2  Moving a TDD feature to Cucumber

​	John will start creating the Cucumber features. He will follow the Maven standard folders structure and introduce the features into the test/resources folder. He will create the test/resources/features folder and, inside it, he will create the passengers_policy.feature file (fig. 21.2).

![](https://pic.imgdb.cn/item/6157d6e62ab3f51d9197d9ea.jpg)

**Listing 21.2 The passenger_policy.feature file**

```
Feature: Passengers Policy
  The company follows a policy of adding and removing passengers,
  depending on the passenger type and on the flight type
 
  Scenario: Economy flight, regular passenger
    Given there is an economy flight
    When we have a regular passenger
    Then you can add and remove him from an economy flight
    And you cannot add a regular passenger to an economy flight more than once
 
  Scenario: Economy flight, VIP passenger
    Given there is an economy flight
    When we have a VIP passenger
    Then you can add him but cannot remove him from an economy flight
    And you cannot add a VIP passenger to an economy flight more than once
 
  Scenario: Business flight, regular passenger
    Given there is a business flight
    When we have a regular passenger
    Then you cannot add or remove him from a business flight
 
  Scenario: Business flight, VIP passenger
    Given there is a business flight
    When we have a VIP passenger
    Then you can add him but cannot remove him from a business flight
    And you cannot add a VIP passenger to a business flight more than once
 
  Scenario: Premium flight, regular passenger
    Given there is a premium flight
    When we have a regular passenger
    Then you cannot add or remove him from a premium flight
 
  Scenario: Premium flight, VIP passenger
    Given there is a premium flight
    When we have a VIP passenger
    Then you can add and remove him from a premium flight
    And you cannot add a VIP passenger to a premium flight more than once
```

​	We see the keywords Feature, Scenario, Given, When, Then, And which are highlighted. If we right-click on this feature file, we see that we have the possibility to run it directly (fig. 21.3).

![](https://pic.imgdb.cn/item/6157d7082ab3f51d9198084e.jpg)

​	This is possible only if two things are fulfilled. First, the appropriate plugins must be activated. In order to do this, we must go to File -> Settings->Plugins, and we install from here the Cucumber for Java and Gherkin plugins (fig. 21.4 and 21.5).

![](https://pic.imgdb.cn/item/6157d71c2ab3f51d91982570.jpg)

![](https://pic.imgdb.cn/item/6157d72d2ab3f51d91983b1b.jpg)

​	Then, we must configure the way the feature is run. We need to go to Run -> Edit Configurations, and to set the following (fig. 21.6):

![](https://pic.imgdb.cn/item/6157d7532ab3f51d91986ba0.jpg)

​	Running the feature directly will generate the skeleton of the Java Cucumber tests (fig. 21.7).

![](https://pic.imgdb.cn/item/6157d8282ab3f51d9199881b.jpg)

**Listing 21.3 The initial PassengersPolicy class**

```java
public class PassengerPolicy {
   @Given("^there is an economy flight$")                                   #A
   public void there_is_an_economy_flight() throws Throwable {              #B
    // Write code here that turns the phrase above into concrete actions    #B
    throw new PendingException();                                           #B
   }                                                                        #B
 
   @When("^we have a regular passenger$")                                   #C
   public void we_have_a_regular_passenger() throws Throwable {             #D
      // Write code here that turns the phrase above into concrete actions  #D
      throw new PendingException();                                         #D
   }                                                                        #D
 
   @Then("^you can add and remove him from an economy flight$")             #E
   public void you_can_add_and_remove_him_from_an_economy_flight()          #F
      throws Throwable {                                                    #F
      // Write code here that turns the phrase above into concrete actions  #F
      throw new PendingException();                                         #F
   }                                                                        #F
 
   […]
 
}
```

**Listing 21.4 Implementing the business logic of the previously defined steps**

```java
public class PassengerPolicy {
    private Flight economyFlight;                                           #A
    private Passenger mike;                                                 #A
    […]
 
    @Given("^there is an economy flight$")                                  #B
    public void there_is_an_economy_flight() throws Throwable {             #B
        economyFlight = new EconomyFlight("1");                             #C
    }
 
    @When("^we have a regular passenger$")                                  #D
    public void we_have_a_regular_passenger() throws Throwable {            #D
        mike  = new Passenger("Mike", false);                               #E
    }
 
    @Then("^you can add and remove him from an economy flight$")            #F
    public void you_can_add_and_remove_him_from_an_economy_flight()         #F
           throws Throwable {
        assertAll("Verify all conditions for a regular passenger            #G
                   and an economy flight",                                  #G
          () -> assertEquals("1", economyFlight.getId()),                   #G
          () -> assertEquals(true, economyFlight.addPassenger(mike)),       #G
          () -> assertEquals(1,                                             #G
                  economyFlight.getPassengersSet().size()),                 #G
          () -> 
             assertTrue(economyFlight.getPassengersSet().contains(mike)),   #G
          () -> assertEquals(true, economyFlight.removePassenger(mike)),    #G
          () -> assertEquals(0, economyFlight.getPassengersSet().size())    #G
        );
    }
    […]
 }
```

**Listing 21.5 The CucumberTest class**

```java
@RunWith(Cucumber.class)                                                    #A
@CucumberOptions(                                                           #B
   plugin = {"pretty"},                                                     #C
   features = "classpath:features")                                         #D
public class CucumberTest {
 
   /**
    * This class should be empty, step definitions should be in separate classes.
    */
 
}
```

​	By running the tests, we see that the code coverage is 100% (fig. 21.8), so we have kept the existing test functionalities before moving to Cucumber.

![](https://pic.imgdb.cn/item/6157d8902ab3f51d919a1194.jpg)

**21.2.3  Adding a new feature with the help of Cucumber**

![](https://pic.imgdb.cn/item/6157d8b32ab3f51d919a3c86.jpg)

**Listing 21.6 The bonus_policy.feature file**

```
Feature: Bonus Policy
  The company follows a bonus policy, depending on the passenger type and on the mileage
 
  Scenario Outline: Regular passenger bonus policy                                 #A
    Given we have a regular passenger with a mileage
    When the regular passenger travels <mileage1> and <mileage2> and <mileage3>    #B
    Then the bonus points of the regular passenger should be <points>              #B
 
    Examples:                                                                      #C
      | mileage1 | mileage2 | mileage3| points |                                   #C
      |     349  |     319  |    623  |     64 |                                   #C
      |     312  |     356  |    135  |     40 |                                   #C
      |     223  |     786  |    503  |     75 |                                   #C
      |     482  |      98  |    591  |     58 |                                   #C
      |     128  |     176  |    304  |     30 |                                   #C
 
  Scenario Outline: VIP passenger bonus policy                                     #A
    Given we have a VIP passenger with a mileage
    When the VIP passenger travels <mileage1> and <mileage2> and <mileage3>        #B
    Then the bonus points of the VIP passenger should be <points>                  #B
 
    Examples:                                                                      #C
      | mileage1 | mileage2 | mileage3| points  |                                  #C
      |     349  |     319  |    623  |     129 |                                  #C
      |     312  |     356  |    135  |      80 |                                  #C
      |     223  |     786  |    503  |     151 |                                  #C
      |     482  |      98  |    591  |     117 |                                  #C
      |     128  |     176  |    304  |      60 |                                  #
```

We will configure the way the feature is run. We need to go to Run -> Edit Configurations, and to set the following (fig. 21.10):

![](https://pic.imgdb.cn/item/6157d91e2ab3f51d919ac87e.jpg)

![](https://pic.imgdb.cn/item/6157d91e2ab3f51d919ac87e.jpg)

**Listing 21.7 The initial BonusPolicy class**

```java
public class BonusPolicy {
   @Given("^we have a regular passenger with a mileage$")                   #A
   public void we_have_a_regular_passenger_with_a_mileage()                 #B
           throws Throwable {                                               #B
    // Write code here that turns the phrase above into concrete actions    #B
    throw new PendingException();                                           #B
   }                                                                        #B
 
   @When("^the regular passenger travels (\\d+) and (\\d+) and (\\d+)$")    #C
   public void the_regular_passenger_travels_and_and                        #D
         (int arg1, int arg2, int arg3) throws Throwable {                  #D
    // Write code here that turns the phrase above into concrete actions    #D
    throw new PendingException();                                           #D
   }                                                                        #D
 
   @Then("^the bonus points of the regular passenger should be (\\d+)$")    #E
   public void the_bonus_points_of_the_regular_passenger_should_be(int arg1)#F
       throws Throwable {                                                   #F
    // Write code here that turns the phrase above into concrete actions    #F
    throw new PendingException();                                           #F
   }                                                                        #F
   […]
 
}
```

**Listing 21.8 The Mileage class, with no implementation of the methods**

```java
public class Mileage {
 
    public static final int VIP_FACTOR = 10;                                #A
    public static final int REGULAR_FACTOR = 20;                            #A
 
    private Map<Passenger, Integer> passengersMileageMap = new HashMap<>(); #B
    private Map<Passenger, Integer> passengersPointsMap = new HashMap<>();  #B
 
    public void addMileage(Passenger passenger, int miles) {                #C
 
    }
 
    public void calculateGivenPoints() {                                    #D
 
    }
 
}
```

**Listing 21.9 The business logic of the steps from BonusPolicy**

```java
public class BonusPolicy {
    private Passenger mike;                                                 #A
    private Mileage mileage;                                                #A
    […]
 
    @Given("^we have a regular passenger with a mileage$")                  #B
    public void we_have_a_regular_passenger_with_a_mileage() 
                throws Throwable {
        mike  = new Passenger("Mike", false);                               #C
        mileage  = new Mileage();                                           #C
    }
 
    @When("^the regular passenger travels (\\d+) and (\\d+) and (\\d+)$")  #D
    public void the_regular_passenger_travels_and_and(int mileage1, int 
                mileage2, int mileage3) throws Throwable {
        mileage.addMileage(mike, mileage1);                                 #E
        mileage.addMileage(mike, mileage2);                                 #E
        mileage.addMileage(mike, mileage3);                                 #E
    }
 
    @Then("^the bonus points of the regular passenger should be (\\d+)$")  #F
    public void the_bonus_points_of_the_regular_passenger_should_be
                 (int points) throws Throwable {
        mileage.calculateGivenPoints();                                     #G
        assertEquals(points,                                                #H
           mileage.getPassengersPointsMap().get(mike).intValue());          #H
    }
    […]
}
```

​	If we run the bonus points tests now they will fail (fig. 21.12), as the business logic is not yet implemented (the `addMileage` and `calculateGivenPoints` methods are empty, the business logic is implemented after the tests).

![](https://pic.imgdb.cn/item/6157d9c72ab3f51d919bb83f.jpg)

**Listing 21.10 The implementation of business logic from the Mileage class**

```java
public void addMileage(Passenger passenger, int miles) {
    if (passengersMileageMap.containsKey(passenger)) {                      #A
        passengersMileageMap.put(passenger,                                 #B
           passengersMileageMap.get(passenger) + miles);                    #B
    } else {
        passengersMileageMap.put(passenger, miles);                         #C
    }
 
}
 
public void calculateGivenPoints() {
    for (Passenger passenger : passengersMileageMap.keySet()) {             #D
        if (passenger.isVip()) {                                            #E
            passengersPointsMap.put(passenger,                              #E
              passengersMileageMap.get(passenger)/ VIP_FACTOR);             #E
        } else {
            passengersPointsMap.put(passenger,                              #F
              passengersMileageMap.get(passenger)/ REGULAR_FACTOR);         #F
        }
    }
}
```

![](https://pic.imgdb.cn/item/6157d9e82ab3f51d919be2e0.jpg)

## 21.3  Working BDD with JBehave and JUnit 5

> **JBehave**
>
> JBehave is a Behavior Driven Development testing tool framework. As implementing the idea of Behavior Driven Development, JBehave allows us to write stories in plain text, to be understood by all persons involved in the project. Through the stories, we'll define scenarios that express the desired behavior.

### 21.3.2  Moving a TDD feature to JBehave

```xml
<dependency>
   <groupId>org.jbehave</groupId>
   <artifactId>jbehave-core</artifactId>
   <version>4.1</version>
</dependency>
```

![](https://pic.imgdb.cn/item/6157da1d2ab3f51d919c2870.jpg)

​	John will start creating the story. He will follow the Maven standard folders structure and introduce the stories into the test/resources folder. He will create a folders structure com/manning/junitbook/airport and insert here the passengers_policy_story.story file. He will also create, into the test folder, the com.manning.junitbook.airport package containing the PassengersPolicy class (fig. 21.15).

![](https://pic.imgdb.cn/item/6157daa42ab3f51d919ce2c3.jpg)

**Listing 21.12 The passengers_policy_story.story file**

```
Meta: Passengers Policy
      The company follows a policy of adding and removing passengers, 
      depending on the passenger type and on the flight type
 
Narrative:
As a company
I want to be able to manage passengers and flights
So that the policies of the company are followed
 
Scenario: Economy flight, regular passenger
Given there is an economy flight
When we have a regular passenger
Then you can add and remove him from an economy flight
And you cannot add a regular passenger to an economy flight more than once
 
Scenario: Economy flight, VIP passenger
Given there is an economy flight
When we have a VIP passenger
Then you can add him but cannot remove him from an economy flight
And you cannot add a VIP passenger to an economy flight more than once
 
Scenario: Business flight, regular passenger
Given there is a business flight
When we have a regular passenger
Then you cannot add or remove him from a business flight
 
Scenario: Business flight, VIP passenger
Given there is a business flight
When we have a VIP passenger
Then you can add him but cannot remove him from a business flight
And you cannot add a VIP passenger to a business flight more than once
 
Scenario: Premium flight, regular passenger
Given there is a premium flight
When we have a regular passenger
Then you cannot add or remove him from a premium flight
 
Scenario: Premium flight, VIP passenger
Given there is a premium flight
When we have a VIP passenger
Then you can add and remove him from a premium flight
And you cannot add a VIP passenger to a premium flight more than once
```

​	In order to generate the steps into a Java file, John will place the cursor on any not yet created test step (they are underlined in red) and press Alt+Enter (fig. 21.16).

![](https://pic.imgdb.cn/item/6157dad02ab3f51d919d1c5f.jpg)

![](https://pic.imgdb.cn/item/6157dade2ab3f51d919d2d9d.jpg)

```java
public class PassengersPolicy {
    @Given("there is an economy flight")
    public void givenThereIsAnEconomyFlight() {
 
    }
 
    @When("we have a regular passenger")
    public void whenWeHaveARegularPassenger() {
 
    }
 
    @Then("you can add and remove him from an economy flight")
    public void thenYouCanAddAndRemoveHimFromAnEconomyFlight() {
 
    }
    […]
}
```

**Listing 21.14 The implemented tests from PassengersPolicy**

```java
public class PassengersPolicy {
    private Flight economyFlight;                                           #A
    private Passenger mike;                                                 #A
    […]
 
    @Given("there is an economy flight")                                    #B
    public void givenThereIsAnEconomyFlight() {
        economyFlight = new EconomyFlight("1");                             #C
    }
 
    @When("we have a regular passenger")                                    #D
    public void whenWeHaveARegularPassenger() {
        mike  = new Passenger("Mike", false);                               #E
    }
 
    @Then("you can add and remove him from an economy flight")              #F
    public void thenYouCanAddAndRemoveHimFromAnEconomyFlight() {
        assertAll("Verify all conditions for a regular passenger            #G
                   and an economy flight",                                  #G
           () -> assertEquals("1", economyFlight.getId()),                  #G
           () -> assertEquals(true,                                         #G
                 economyFlight.addPassenger(mike)),                         #G
           () -> assertEquals(1,                                            #G
                 economyFlight.getPassengersSet().size()),                  #G
           () -> assertEquals("Mike", new ArrayList<>(                      #G
             economyFlight.                                                 #G
               getPassengersSet()).get(0).getName()),                       #G
           () -> assertEquals(true,                                         #G
                       economyFlight.removePassenger(mike)),                #G
           () -> assertEquals(0,                                            #G
                       economyFlight.getPassengersSet().size())             #G
        );
    }
[…]
 
}
```

**Listing 21.15 The PassengersPolicyStory class**

```java
public class PassengersPolicyStory extends JUnitStory {                     #A
 
    @Override
    public Configuration configuration() {                                  #B
        return new MostUsefulConfiguration()                                #C
                  .useStoryReporterBuilder(                                 #D
                     new StoryReporterBuilder().                            #D
                         withFormats(Format.CONSOLE));                      #D
    }
 
    @Override
    public InjectableStepsFactory stepsFactory() {                          #E
        return new InstanceStepsFactory(configuration(),                    #F
                      new PassengersPolicy());                              #F
    }
}
```

The result of running these tests is shown in fig. 21.18. The tests are successfully running, the code coverage is 100%. However, the reporting capabilities of JBehave do not allow the same nice display as in the case of Cucumber.

![](https://pic.imgdb.cn/item/6157db182ab3f51d919d79ee.jpg)

### 21.3.3  Adding a new feature with the help of JBehave

**Listing 21.16 The bonus_policy_story.story file**

```java
Meta: Bonus Policy
      The company follows a bonus policy, depending on the passenger type and on the mileage
 
Narrative:
As a company
I want to be able to manage the bonus awarding
So that the policies of the company are followed
 
Scenario: Regular passenger bonus policy                                   #A
Given we have a regular passenger with a mileage
When the regular passenger travels <mileage1> and <mileage2> and <mileage3>#B
Then the bonus points of the regular passenger should be <points>          #B
 
Examples:
| mileage1 | mileage2 | mileage3| points |                                 #C
|     349  |     319  |    623  |     64 |                                 #C
|     312  |     356  |    135  |     40 |                                 #C
|     223  |     786  |    503  |     75 |                                 #C
|     482  |      98  |    591  |     58 |                                 #C
|     128  |     176  |    304  |     30 |                                 #C
 
Scenario: VIP passenger bonus policy                                       #A
Given we have a VIP passenger with a mileage
When the VIP passenger travels <mileage1> and <mileage2> and <mileage3>    #B
Then the bonus points of the VIP passenger should be <points>              #B
 
Examples:
| mileage1 | mileage2 | mileage3| points  |                                #C
|     349  |     319  |    623  |     129 |                                #C
|     312  |     356  |    135  |      80 |                                #C
|     223  |     786  |    503  |     151 |                                #C
|     482  |      98  |    591  |     117 |                                #C
|     128  |     176  |    304  |      60 |                                #C
```

![](https://pic.imgdb.cn/item/6157db402ab3f51d919db353.jpg)

![](https://pic.imgdb.cn/item/6157dbe32ab3f51d919e90b0.jpg)

**Listing 21.17 The skeleton of the JBehave BonusPolicy test**

```java
public class BonusPolicy {
 
    @Given("we have a regular passenger with a mileage")                    #A
    public void givenWeHaveARegularPassengerWithAMileage() {
 
    }
 
    @When("the regular passenger travels <mileage1> and                     #B
           <mileage2> and <mileage3>")                                      #B
    public void whenTheRegularPassengerTravelsMileageAndMileageAndMileage(
           @Named("mileage1") int mileage1, @Named("mileage2") int mileage2,  
           @Named("mileage3") int mileage3) {
 
    }
 
    @Then("the bonus points of the regular passenger should be <points>")   #C
    public void thenTheBonusPointsOfTheRegularPassengerShouldBePoints(
      @Named("points") int points) {
 
   }
   […]
}
```

**Listing 21.18 The Mileage class, with no implementation of the methods**

```java
public class Mileage {
 
    public static final int VIP_FACTOR = 10;                                #A
    public static final int REGULAR_FACTOR = 20;                            #A
 
    private Map<Passenger, Integer> passengersMileageMap = new HashMap<>(); #B
    private Map<Passenger, Integer> passengersPointsMap = new HashMap<>();  #B
 
    public void addMileage(Passenger passenger, int miles) {                #C
 
    }
 
    public void calculateGivenPoints() {                                    #D
 
    }
 
}
```

**Listing 21.19 The business logic of the steps from BonusPolicy**

```java
public class BonusPolicy {
    private Passenger mike;                                                 #A
    private Mileage mileage;                                                #A
    […]
 
    @Given("we have a regular passenger with a mileage")                    #B
    public void givenWeHaveARegularPassengerWithAMileage() {
        mike  = new Passenger("Mike", false);                               #C
        mileage  = new Mileage();                                           #C
    }
 
    @When("the regular passenger travels <milege1> and <mileage2> and       #D
                                       <mileage3>")                         #D
    public void the_regular_passenger_travels_and_and(@Named(“mileage1”)
                int mileage1, @Named(“mileage2”) int mileage2, 
                @Named(“mileage3”) int mileage3) {
        mileage.addMileage(mike, mileage1);                                 #E
        mileage.addMileage(mike, mileage2);                                 #E
        mileage.addMileage(mike, mileage3);                                 #E
    }
 
    @Then("the bonus points of the regular passenger should be <points>")   #F
    public void the_bonus_points_of_the_regular_passenger_should_be
                 (@Named(“points”) int points) {
        mileage.calculateGivenPoints();                                     #G
        assertEquals(points,                                                #H
           mileage.getPassengersPointsMap().get(mike).intValue());          #H
    }
    […]
}
```

**Listing 21.20 The BonusPolicyStory class**

```java
public class BonusPolicyStory extends JUnitStory {                          #A
 
    @Override
    public Configuration configuration() {                                  #B
        return new MostUsefulConfiguration()                                #C
                  .useStoryReporterBuilder(                                 #D
                     new StoryReporterBuilder().                            #D
                         withFormats(Format.CONSOLE));                      #D
    }
 
    @Override
    public InjectableStepsFactory stepsFactory() {                          #E
        return new InstanceStepsFactory(configuration(),                    #F
                      new BonusPolicy());                                   #F
    }
}
```

​	If we run the bonus points tests now they will fail (fig. 21.22), as the business logic is not yet implemented (we have left the `addMileage` and `calculateGivenPoints` methods empty).

![](https://pic.imgdb.cn/item/6157e1a92ab3f51d91a7959a.jpg)

**Listing 21.21 The implementation of business logic from the Mileage class**

```java
public void addMileage(Passenger passenger, int miles) {
    if (passengersMileageMap.containsKey(passenger)) {                      #A
        passengersMileageMap.put(passenger,                                 #A
           passengersMileageMap.get(passenger) + miles);                    #A
    } else {
        passengersMileageMap.put(passenger, miles);                         #B
    }
 
}
 
public void calculateGivenPoints() {
    for (Passenger passenger : passengersMileageMap.keySet()) {             #C
        if (passenger.isVip()) {                                            #D
            passengersPointsMap.put(passenger,                              #D
              passengersMileageMap.get(passenger)/ VIP_FACTOR);             #D
        } else {
            passengersPointsMap.put(passenger,                              #E
              passengersMileageMap.get(passenger)/ REGULAR_FACTOR);         #E
        }
    }
}
```

![](https://pic.imgdb.cn/item/6157e1e42ab3f51d91a7f425.jpg)

## 21.4  Comparing Cucumber and JBehave

略。

# 21    Behavior Driven Development with JUnit 5

This chapter covers:

·  Introducing Behavior Driven Development

·  Analyzing the benefits and challenges of Behavior Driven Development

·  Moving a TDD application to BDD

·  Developing a BDD application with the help of Cucumber and JUnit 5

·  Developing a BDD application with the help of JBehave and JUnit 5

·  Comparing Cucumber and JBehave

Some people refer to BDD as "TDD done right." You can also think of BDD as "how we build the right thing" and TDD as "how we build the thing right."

\- Millard Ellingsworth

Kent Beck invented Test Driven Development in the early years of Agile. It is an effective technique that uses unit tests to verify the code. Working TDD style, the programmer will have first to write a test that checks the not yet implemented feature. The test is expected to fail. Then, the programmer will write the smallest piece of code that fixes the test. Eventually, the developer will refactor the code to be easier to understand and to maintain.

Despite its clear benefits, this usual loop

**[test, code,** **refactor****, (repeat)]**

can bring developers to lose the overall picture of the business goals of the application. The project will become larger and more complex, the numbers of unit tests will increase and will become harder to understand and to maintain. The tests may also be strongly coupled with the implementation. They focus on the unit (the class or the method) that is tested, while the business goals may not be considered.

Starting from TDD, a new technique has been created: the Behavior Driven Development. It focuses on the features themselves, it makes sure that these ones work as expected.

## 21.1  Introducing Behavior Driven Development

Behavior Driven Development

Behavior Driven Development is a software development technique that directly addresses the business requirements. It starts from the business requirements and goals and then transforms them into working features.

BDD encourages teams to interact and use concrete examples to communicate how the application must behave. It emerged from Test Driven Development.

Test Driven Development helps us to build safe software. Behavior Driven Development helps us building software providing business value.

 

Dan North originated the Behavior Driven Development in the mid-2000s. It is a software development technique that encourages teams to deliver software that matters, supporting the cooperation between stakeholders.

BDD helps us write software that really matters. We can find out the features that the organization really needs and we then focus on implementing them. We will be able to discover what the user actually needs and not only what he asks about.

BDD is a large topic and this chapter focuses on demonstrating how to use it in conjunction with JUnit 5 and how to effectively build features using this technique. For a comprehensive work on this subject, you may refer to another Manning book, “BDD in Action” by John Ferguson Smart. The second edition of this book is under development at the time of writing this chapter ([***https://www.manning.com/books/bdd-in-action-second-edition\***](https://www.manning.com/books/bdd-in-action-second-edition)).

The communication between the people who are involved in the same project may bring up problems and misunderstandings. Usually, the flow works this way:

·  The customer communicates to the business analyst his understanding about the functionality of a feature.

·  The business analyst builds the requirements for the developers, describing the way the software must work.

·  The developer creates the code based on the requirements and writes unit tests to implement the new feature.

·  The tester creates the test cases based on the requirements and uses them to verify the way the new feature works.

It is possible that the information gets misunderstood, modified or ignored. The new feature may not do exactly what it has been initially expected.

### 21.1.1  Introducing a new feature

The business analyst discusses with the customer to decide the software features that will be able to address the business goals. These features are general requirements, like: "Allow the traveler to choose the shortest way to the destination", "Allow the traveler to choose the cheapest way to the destination".

These features need to be broken into stories. The stories might look like: "Find the route between source and destination with the smallest number of flight changes", "Find the quickest route between source and destination".

Stories will be defined through concrete examples. These examples will become the acceptance criteria for a story.

Acceptance criteria may be expressed BDD style through the keywords "Given", "When", "Then".

As an example, we may present the following acceptance criteria:

Given the flights operated by company X

When I want to find the quickest route from Bucharest to New York on May 15 20...

Then I will be provided the route Bucharest - Frankfurt - New York, with a duration of...

### 21.1.2  From requirements analysis to acceptance criteria

For the company using the flights management application, one business goal that we can formulate is "Increase sales by providing higher quality overall flight services". This is a very general goal, and it can be detailed through requirements:

·  Provide an interactive application to choose flights

·  Provide an interactive application to change flights

·  Provide an interactive application to calculate the shortest route between source and destination

To make the customer happy, the features generated by the requirements analysis need to achieve the customer business goals or deliver business value. The initial ideas need to be described in more detail. One way to describe the previous requirements would be:

**As a passenger**

**I want to know the flights for a given destination within a given period of time**

**So that I can choose the flight(s) that suit(s) my needs**

or

**As a passenger**

**I want to be able to change my initial flight(s) to a different one(s)**

**So that I can follow the changes in my schedule**

A feature like "I can choose the flights that suit my needs" might be too large to be implemented at once - so, it must be divided. You may also want to get some feedback while passing through the milestones of the implementation of a feature.

The previous feature may be broken into smaller stories, such as the following:

·  Find the direct flights that suit my needs (if any).

·  Find the alternatives of flights with stopovers that suit my needs.

·  Find the one-way flights that suit my needs.

·  Find the there and back flights that suit my needs.

Generally, particular examples are used as acceptance criteria. Acceptance criteria will express what will make the stakeholder agree that the application is working the way it was expected.

In BDD, the definition of acceptance criteria is made using the Given/When/Then keywords. More exactly:

**Given <a context>**

**When <an action occurs>**

**Then <expect a result>**

As a concrete example:

**Given the flights operated by the company**

**When I want to travel from Bucharest to London next Wednesday**

**Then I should be provided 2 possible flights: 10:35 and 16:20**

### 21.1.3  BDD benefits and challenges

We’ll point out a few benefits of the BDD approach:

·  **Address user needs.** The users care less about the implementation and they are mainly interested in the application functionality. Working BDD style you get closer to addressing these needs.

·  **Clarity**. Scenarios will clarify what software should do. Scenarios are described in simple language, easy to understand by technical and non-technical people. Ambiguities can be clarified by analyzing the scenario or by adding another scenario.

·  **Supports change.** The scenarios will represent a part of the documentation of the software – in fact, living documentation, as it evolves simultaneously with the application. It also helps locate an incoming change. The automated acceptance tests will hinder the introduction of regressions when new changes are introduced.

·  **Supports automation.** Scenarios can be transformed into automated tests, as the steps of the scenario are already defined.

·  **Focuses on adding business value.** BDD will prevent introducing features that are not useful to the project. You will also be able to prioritize the functionalities.

·  **Reduces costs.** Prioritizing the importance of the functionalities and avoiding the unnecessary ones will hinder the waste of resources and concentrate them to do exactly what is needed.

As challenges, BDD requires engagement and strong collaboration. It requires interaction, direct communication, and constant feedback. This may be a challenge for some persons and, in the context of the present-day globalization and distributed teams, may require language skills and fitting time zones.

## 21.2  Working BDD with Cucumber and JUnit 5

Tested Data Systems Inc. is an outsourcing company creating software projects for different companies. The flights management application we have worked with is one of the projects under development.

Working TDD style, at the end of chapter 20, John, a programmer at Tested Data Systems, has left the development of the flights management application in a stage where it was able to work with three types of flights: economy, business, and premium. Besides this, he implemented the requirement that a passenger can be added only once to a flight. The functionality of the application can be quickly reviewed if we run the tests (fig. 21.1).

John has already introduced, in a discrete way, a first taste of the Behavior Driven Development way of working. We can easily read how the application works by following the tests using the "Given", "When", "Then" keywords. John will take it over from here, moving it to BDD with Cucumber and also introducing new features.

![img](http://localhost:8000/39e2591b-e785-4a53-9b15-9c0cdeb566c1/OEBPS/Images/21_img_0001.jpg)

Figure 21.1 Successfully running the tests of the TDD flights management application – the user may follow the execution of the annotated methods and read the scenarios

### 21.2.1  Introducing Cucumber

Cucumber

Cucumber is a Behavior Driven Development testing tool framework. It describes the application scenarios in plain English text, using a language called Gherkin. Cucumber is easy to read and understand by stakeholders and allows automation.

The main capabilities of Cucumber are:

·  Scenarios or examples describe the requirements.

·  A scenario is defined through a list of steps to be executed by Cucumber.

·  Cucumber executes the code corresponding to the scenarios, checks that the software follows these requirements and generates a report about the success or failure of each scenario.

The main capabilities of Gherkin are:

·  Gherkin defines simple grammar rules that allow Cucumber to understand English plain text.

·  Gherkin documents the behavior of the system. The requirements are always up to date, as they are provided through these scenarios which represent living specifications.

Cucumber ensures that technical and non-technical persons can easily read, write and understand the acceptance tests. The acceptance tests became an instrument of communication between the stakeholders of the project.

A Cucumber acceptance test can look this way:

**Given there is an economy flight**

**When we have a regular passenger**

**Then you can add and remove him from an economy flight**

We notice again the Given/When/Then words. We have already explained that they are the keywords for describing a scenario and we have already introduced them in our previous work with JUnit 5. But we have to know from the very beginning that we'll no longer use them just for labeling. Cucumber will take care to interpret the sentences starting with these keywords and generate methods which it will annotate using exactly these annotations: `@Given`, `@When` and `@Then`.

The acceptance tests will be written in Cucumber feature files. A feature file is an entry point to the Cucumber tests. It is a file where we will describe our tests in Gherkin. A feature file can contain one or many scenarios.

John makes plans for starting the work with Cucumber in the project. He will first introduce the Cucumber dependencies into the existing Maven configuration. He will create a first Cucumber feature and will generate the skeleton of the Cucumber tests. Then, he will move the previously existing JUnit 5 tests to fill in this Cucumber generated tests skeleton.

Listing 21.1 The Cucumber dependencies added to the pom.xml file

<dependency>     <groupId>info.cukes</groupId>     <artifactId>cucumber-java</artifactId>     <version>1.2.5</version>     <scope>test</scope></dependency><dependency>     <groupId>info.cukes</groupId>     <artifactId>cucumber-junit</artifactId>     <version>1.2.5</version>     <scope>test</scope></dependency>

In listing 21.1, John has introduced the two needed Maven dependencies: `cucumber-java` and `cucumber-junit`.

### 21.2.2  Moving a TDD feature to Cucumber

John will start creating the Cucumber features. He will follow the Maven standard folders structure and introduce the features into the test/resources folder. He will create the test/resources/features folder and, inside it, he will create the passengers_policy.feature file (fig. 21.2).

![img](http://localhost:8000/39e2591b-e785-4a53-9b15-9c0cdeb566c1/OEBPS/Images/21_img_0002.png)

Figure 21.2 The new Cucumber passengers_policy.feature file is created into the test/resources/features folder, following the Maven rules

John will follow the Gherkin syntax and introduce the feature named "Passengers Policy", together with a short description of what it intends to do. Then, he will follow the same Gherkin syntax and write the scenarios (listing 21.2).

Listing 21.2 The passenger_policy.feature file

`Feature: Passengers Policy`` The company follows a policy of adding and removing passengers,`` depending on the passenger type and on the flight type`` `` Scenario: Economy flight, regular passenger``  Given there is an economy flight``  When we have a regular passenger``  Then you can add and remove him from an economy flight``  And you cannot add a regular passenger to an economy flight more than once`` `` Scenario: Economy flight, VIP passenger``  Given there is an economy flight``  When we have a VIP passenger``  Then you can add him but cannot remove him from an economy flight``  And you cannot add a VIP passenger to an economy flight more than once`` `` Scenario: Business flight, regular passenger``  Given there is a business flight``  When we have a regular passenger``  Then you cannot add or remove him from a business flight`` `` Scenario: Business flight, VIP passenger``  Given there is a business flight``  When we have a VIP passenger``  Then you can add him but cannot remove him from a business flight``  And you cannot add a VIP passenger to a business flight more than once`` `` Scenario: Premium flight, regular passenger``  Given there is a premium flight``  When we have a regular passenger``  Then you cannot add or remove him from a premium flight`` `` Scenario: Premium flight, VIP passenger``  Given there is a premium flight``  When we have a VIP passenger``  Then you can add and remove him from a premium flight``  And you cannot add a VIP passenger to a premium flight more than once`

We see the keywords Feature, Scenario, Given, When, Then, And which are highlighted. If we right-click on this feature file, we see that we have the possibility to run it directly (fig. 21.3).

![img](http://localhost:8000/39e2591b-e785-4a53-9b15-9c0cdeb566c1/OEBPS/Images/21_img_0003.jpg)

Figure 21.3 Directly running the passengers_policy.feature file by right-clicking on the feature file

This is possible only if two things are fulfilled. First, the appropriate plugins must be activated. In order to do this, we must go to File -> Settings->Plugins, and we install from here the Cucumber for Java and Gherkin plugins (fig. 21.4 and 21.5).

![img](http://localhost:8000/39e2591b-e785-4a53-9b15-9c0cdeb566c1/OEBPS/Images/21_img_0004.jpg)

Figure 21.4 Installing the Cucumber for Java plugin from the File -> Settings -> Plugins menu

![img](http://localhost:8000/39e2591b-e785-4a53-9b15-9c0cdeb566c1/OEBPS/Images/21_img_0005.jpg)

Figure 21.5 Installing the Gherkin plugin from the File -> Settings -> Plugins menu

Then, we must configure the way the feature is run. We need to go to Run -> Edit Configurations, and to set the following (fig. 21.6):

![img](http://localhost:8000/39e2591b-e785-4a53-9b15-9c0cdeb566c1/OEBPS/Images/21_img_0006.jpg)

Figure 21.6 Setting the feature configuration by filling in the Main class, Glue, Feature or folder path, and Working directory

·  Main class: cucumber.api.cli.Main

·  Glue (the package where step definitions are stored): com.manning.junitbook.airport

·  Feature or folder path: the test/resources/features folder we have created

·  Working directory: the project folder

Running the feature directly will generate the skeleton of the Java Cucumber tests (fig. 21.7).

![img](http://localhost:8000/39e2591b-e785-4a53-9b15-9c0cdeb566c1/OEBPS/Images/21_img_0007.jpg)

Figure 21.7 Getting the skeleton of the passengers policy feature by directly running the feature file – the annotated methods will be executed in order to verify the scenarios

John will now create a new Java class into the test/java folder, into the `com.manning.junitbook.airport` package. This class will be named `PassengersPolicy` and, for the beginning, it will contain the tests skeleton (listing 21.3). The execution of such a test will follow the scenarios described in the passengers_policy.feature file. For example, when executing the step:

**Given** there is an economy flight

the program will execute the method annotated with:

@Given("^there is an economy flight$")

Listing 21.3 The initial PassengersPolicy class

`public class PassengerPolicy {``  @Given("^there is an economy flight$")                  #A``  public void there_is_an_economy_flight() throws Throwable {       #B``  // Write code here that turns the phrase above into concrete actions  #B``  throw new PendingException();                      #B``  }                                    #B`` ``  @When("^we have a regular passenger$")                  #C``  public void we_have_a_regular_passenger() throws Throwable {       #D``   // Write code here that turns the phrase above into concrete actions #D``   throw new PendingException();                     #D``  }                                    #D`` ``  @Then("^you can add and remove him from an economy flight$")       #E``  public void you_can_add_and_remove_him_from_an_economy_flight()     #F``   throws Throwable {                          #F``   // Write code here that turns the phrase above into concrete actions #F``   throw new PendingException();                     #F``  }                                    #F`` ``  […]`` ``}`

In the listing above we are doing the following:

·  The Cucumber plugin has generated a method annotated with `@Given(“^there is an economy flight$”)`, meaning that this method will be executed when the step “`Given there is an economy flight”` from the scenario will be executed (#A).

·  A method stub has been generated by the Cucumber plugin to be implemented with the code addressing the step “`Given there is an economy flight”` from the scenario (#B).

·  The Cucumber plugin has generated a method annotated with `@When(“^we have a regular passenger$”)`, meaning that this method will be executed when the step “`When we have a regular passenger”` from the scenario will be executed (#C).

·  A method stub has been generated by the Cucumber plugin to be implemented with the code addressing the step “`When we have a regular passenger”` from the scenario (#D).

·  The Cucumber plugin has generated a method annotated with `@Then(“^you can add and remove him from an economy flight$”)`, meaning that this method will be executed when the step “`Then you can add and remove him from an economy flight”` from the scenario will be executed (#E).

·  A method stub has been generated by the Cucumber plugin to be implemented with the code addressing the step “`Then you can add and remove him from an economy flight”` from the scenario (#F).

·  The rest of the methods are implemented in a similar way, we have covered the `Given`, `When` and `Then` steps of one scenario.

John follows the business logic of each step that has been defined and transposes it into the tests from listing 21.4 – the steps of the scenarios that need to be verified.

Listing 21.4 Implementing the business logic of the previously defined steps

`public class PassengerPolicy {``  private Flight economyFlight;                      #A``  private Passenger mike;                         #A``  […]`` ``  @Given("^there is an economy flight$")                 #B``  public void there_is_an_economy_flight() throws Throwable {       #B``    economyFlight = new EconomyFlight("1");               #C``  }`` ``  @When("^we have a regular passenger$")                 #D``  public void we_have_a_regular_passenger() throws Throwable {      #D``    mike = new Passenger("Mike", false);                #E``  }`` ``  @Then("^you can add and remove him from an economy flight$")      #F``  public void you_can_add_and_remove_him_from_an_economy_flight()     #F``      throws Throwable {``    assertAll("Verify all conditions for a regular passenger      #G``          and an economy flight",                 #G``     () -> assertEquals("1", economyFlight.getId()),          #G``     () -> assertEquals(true, economyFlight.addPassenger(mike)),    #G``     () -> assertEquals(1,                       #G``         economyFlight.getPassengersSet().size()),         #G``     () -> ``       assertTrue(economyFlight.getPassengersSet().contains(mike)),  #G``     () -> assertEquals(true, economyFlight.removePassenger(mike)),  #G``     () -> assertEquals(0, economyFlight.getPassengersSet().size())  #G``    );``  }``  […]`` }`

In the listing above we are doing the following:

·  We declare the instance variables for the test, among which `economyFlight` and `mike` as a `Passenger` (#A).

·  We write the method corresponding to the “`Given there is an economy flight”` business logic step (#B) by initializing the `economyFlight` (#C).

·  We write the method corresponding to the “`When we have a regular passenger”` business logic step (#D) by initializing the regular passenger `mike` (#E).

·  We write the method corresponding to the “`Then you can add and remove him from an economy flight”` business logic step (#F) by checking all the conditions by using the `assertAll` JUnit 5 method, which can now be read in a flow (#G).

·  The rest of the methods are implemented in a similar way, we have covered the `Given`, `When` and `Then` steps of one scenario.

In order to run our Cucumber tests, we'll need a special class. The name of the class could be anything, we have chosen `CucumberTest` (listing 21.5).

Listing 21.5 The CucumberTest class

`@RunWith(Cucumber.class)                          #A``@CucumberOptions(                              #B``  plugin = {"pretty"},                           #C``  features = "classpath:features")                     #D``public class CucumberTest {`` ``  /**``  * This class should be empty, step definitions should be in separate classes.``  */`` ``}`

In the listing above, we are doing the following:

·  We have annotated this class with `@RunWith(Cucumber.class)` annotation (#A). Executing it as any JUnit test class will run all features found on the classpath in the same package. As there is no Cucumber JUnit 5 extension at the moment of writing this chapter, we use the JUnit 4 runner.

·  The `@CucumberOptions` (#B) annotation provides the plugin option (#C), that is used to specify different formatting options for the output reports. Using "`pretty`", the Gherkin source will be printed with additional colors (fig. 21.8). Other plugin options include `“html”` and `“json”`, but `“pretty”` is appropriate for us now. And the `features` option (#D) helps Cucumber to locate the feature file in the project folder structure. It will look for the features folder on the classpath - and remember that the src/test/resources folder is maintained by Maven on the classpath!

By running the tests, we see that the code coverage is 100% (fig. 21.8), so we have kept the existing test functionalities before moving to Cucumber.

![img](http://localhost:8000/39e2591b-e785-4a53-9b15-9c0cdeb566c1/OEBPS/Images/21_img_0008.jpg)

Figure 21.8 Running CucumberTest. The Gherkin source will be pretty-printed, the successful tests are displayed in green, the code coverage is 100%

There is another advantage to the move to BDD. We compare the length of the pre-Cucumber `AirportTest` class, which has 207 lines, with the one of the `PassengersPolicy` class, which has 157 lines. So, the testing code is now at only 75% of the pre-Cucumber size, yet it has the same 100% coverage. Where does this gain come from? Remember that the AirportTest file contained 7 classes, on 3 levels: `AirportTest` at the top level; one `EconomyFlightTest` and one `BusinessFlightTest` at the second level; and, at the third level, two `RegularPassenger` and two `VipPassenger` classes. The code duplication is now really jumping to our attention, but that was the solution having only JUnit 5.

With Cucumber, each step is implemented only once, and if we have the same step in more than one scenario, we’ll avoid the code duplication.

### 21.2.3  Adding a new feature with the help of Cucumber

John receives a new feature to implement, concerning the policy of bonus points that are awarded to the passenger.

The specifications about calculating the bonus points consider the mileage, meaning the distance that is traveled by each passenger. The bonus will be calculated for all flights of the passenger and it depends on a factor: the mileage will be divided by 10 for VIP passengers and by 20 for regular ones (fig 21.9).

![img](http://localhost:8000/39e2591b-e785-4a53-9b15-9c0cdeb566c1/OEBPS/Images/21_img_0009.png)

Figure 21.9 The business logic of awarding bonus points: the mileage will be divided by 10 for VIP passenger and by 20 for regular ones

John will move to the BDD scenarios, tests and implementation. He will define the scenarios of awarding the bonus points (listing 21.6) and generate the Cucumber tests that describe the scenarios. In the beginning, they are expected to fail. Then, he will effectively add the code that implements the bonus award, run the tests and expect them to be green.

Listing 21.6 The bonus_policy.feature file

`Feature: Bonus Policy`` The company follows a bonus policy, depending on the passenger type and on the mileage`` `` Scenario Outline: Regular passenger bonus policy                 #A``  Given we have a regular passenger with a mileage``  When the regular passenger travels <mileage1> and <mileage2> and <mileage3>  #B``  Then the bonus points of the regular passenger should be <points>       #B`` ``  Examples:                                   #C``   | mileage1 | mileage2 | mileage3| points |                  #C``   |   349 |   319 |  623 |   64 |                  #C``   |   312 |   356 |  135 |   40 |                  #C``   |   223 |   786 |  503 |   75 |                  #C``   |   482 |   98 |  591 |   58 |                  #C``   |   128 |   176 |  304 |   30 |                  #C`` `` Scenario Outline: VIP passenger bonus policy                   #A``  Given we have a VIP passenger with a mileage``  When the VIP passenger travels <mileage1> and <mileage2> and <mileage3>    #B``  Then the bonus points of the VIP passenger should be <points>         #B`` ``  Examples:                                   #C``   | mileage1 | mileage2 | mileage3| points |                 #C``   |   349 |   319 |  623 |   129 |                 #C``   |   312 |   356 |  135 |   80 |                 #C``   |   223 |   786 |  503 |   151 |                 #C``   |   482 |   98 |  591 |   117 |                 #C``   |   128 |   176 |  304 |   60 |                 #C`

In the listing above, we are using the following new things compared to the previous Cucumber feature that we have implemented:

·  We introduce a new capability of Cucumber - Scenario Outline (#A). With Scenario Outline, values do not need to be hard-coded in the step definitions.

·  Values are replaced with parameters as into the step-definition itself - you can see <mileage1>, <mileage2>, <mileage3> and <points> as parameters (#B).

·  The effective values are defined in the Examples table, at the end of the Scenario Outline (#C). The first row in the first table defines the values of 3 mileages (349, 319, 623). Adding them and dividing them by 20 (the regular passenger factor), we get the integer part 64 (the number of bonus points). This successfully replaces the JUnit 5 parameterized tests, having the advantage that the values are kept inside the scenarios, and easy to be understood by everyone.

We will configure the way the feature is run. We need to go to Run -> Edit Configurations, and to set the following (fig. 21.10):

![img](http://localhost:8000/39e2591b-e785-4a53-9b15-9c0cdeb566c1/OEBPS/Images/21_img_0010.jpg)

Figure 21.10 Setting the configuration for the new bonus_policy.feature by filling in the Main class, Glue, Feature or folder path, and Working directory

·  Main class: cucumber.api.cli.Main

·  Glue (the package where step definitions are stored): com.manning.junitbook.airport

·  Feature or folder path: test/resources/features/bonus_policy.feature we have created

·  Working directory: the project folder

Running the feature directly will generate the skeleton of the Java Cucumber tests (fig. 21.11).

![img](http://localhost:8000/39e2591b-e785-4a53-9b15-9c0cdeb566c1/OEBPS/Images/21_img_0011.jpg)

Figure 21.11 Getting the skeleton of the bonus policy feature by directly running the feature file

John will now create a new Java class into the test/java folder, into the `com.manning.junitbook.airport` package. This class will be named `BonusPolicy` and, for the beginning, it will contain the tests skeleton (listing 21.7). The execution of such a test will follow the scenarios described in the bonus_policy.feature file.

Listing 21.7 The initial BonusPolicy class

`public class BonusPolicy {``  @Given("^we have a regular passenger with a mileage$")          #A``  public void we_have_a_regular_passenger_with_a_mileage()         #B``      throws Throwable {                        #B``  // Write code here that turns the phrase above into concrete actions  #B``  throw new PendingException();                      #B``  }                                    #B`` ``  @When("^the regular passenger travels (\\d+) and (\\d+) and (\\d+)$")  #C``  public void the_regular_passenger_travels_and_and            #D``     (int arg1, int arg2, int arg3) throws Throwable {         #D``  // Write code here that turns the phrase above into concrete actions  #D``  throw new PendingException();                      #D``  }                                    #D`` ``  @Then("^the bonus points of the regular passenger should be (\\d+)$")  #E``  public void the_bonus_points_of_the_regular_passenger_should_be(int arg1)#F``    throws Throwable {                          #F``  // Write code here that turns the phrase above into concrete actions  #F``  throw new PendingException();                      #F``  }                                    #F``  […]`` ``}`

In the listing above we are doing the following:

·  The Cucumber plugin has generated a method annotated with `@Given(“^we have a regular passenger with a mileage$”)`, meaning that this method will be executed when the step “`Given we have a regular passenger with a mileage”` from the scenario will be executed (#A).

·  A method stub has been generated by the Cucumber plugin to be implemented with the code addressing the step “`Given we have a regular passenger with a mileage”` from the scenario (#B).

·  The Cucumber plugin has generated a method annotated with `@When(“^the regular passenger travels (\\d+) and (\\d+) and (\\d+)$”)`, meaning that this method will be executed when the step “`When the regular passenger travels <mileage1> and <mileage2> and <mileage3>”` from the scenario will be executed (#C).

·  A method stub has been generated by the Cucumber plugin to be implemented with the code addressing the step “`When the regular passenger travels <mileage1> and <mileage2> and <mileage3>”` from the scenario (#D). This method has three parameters, corresponding to the three different mileages.

·  The Cucumber plugin has generated a method annotated with `@Then(“^the bonus points of the regular passenger should be (\\d+)$”)`, meaning that this method will be executed when the step “`Then the bonus points of the regular passenger should be <points>”` from the scenario will be executed (#E).

·  A method stub has been generated by the Cucumber plugin to be implemented with the code addressing the step “`Then the bonus points of the regular passenger should be <points>”` from the scenario (#F). This method has one parameter, corresponding to the points.

·  The rest of the methods are implemented in a similar way, we have covered the `Given`, `When` and `Then` steps of one scenario.

Now, John will create the Mileage class, declaring the fields and the methods, but not implementing them yet. John needs to use the methods of this class for the tests, make these tests initially fail, then implement the methods and make the tests pass.

Listing 21.8 The Mileage class, with no implementation of the methods

`public class Mileage {`` ``  public static final int VIP_FACTOR = 10;                #A``  public static final int REGULAR_FACTOR = 20;              #A`` ``  private Map<Passenger, Integer> passengersMileageMap = new HashMap<>(); #B``  private Map<Passenger, Integer> passengersPointsMap = new HashMap<>(); #B`` ``  public void addMileage(Passenger passenger, int miles) {        #C`` ``  }`` ``  public void calculateGivenPoints() {                  #D`` ``  }`` ``}`

In the listing above, we are doing the following:

·  We have declared the `VIP_FACTOR` and `REGULAR_FACTOR` constants, corresponding to the factor by which we divide the mileage for each type of passenger in order to get the bonus points (#A).

·  We have declared `passengersMileageMap` and `passengersPointsMap`, two maps having as key the passenger and keeping as value the mileage and the points for that passenger, respectively (#B).

·  We have declared the `addMileage` method, which will populate the `passengersMileageMap` with the mileage for each passenger (#C). The method does not do anything, for now, it will be written later, to fix the tests.

·  We have declared the `calculateGivenPoints` method, which will populate the `passengersPointsMap` with the bonus points for each passenger (#D). The method does not do anything, for now, it will be written later, to fix the tests.

John will now turn his attention to write the unimplemented tests from the BonusPolicy class, to follow the business logic of this feature (listing 21.9).

Listing 21.9 The business logic of the steps from BonusPolicy

`public class BonusPolicy {``  private Passenger mike;                         #A``  private Mileage mileage;                        #A``  […]`` ``  @Given("^we have a regular passenger with a mileage$")         #B``  public void we_have_a_regular_passenger_with_a_mileage() ``        throws Throwable {``    mike = new Passenger("Mike", false);                #C``    mileage = new Mileage();                      #C``  }`` ``  @When("^the regular passenger travels (\\d+) and (\\d+) and (**\\d+)$**") #D``  public void the_regular_passenger_travels_and_and(int mileage1, int ``        mileage2, int mileage3) throws Throwable {``    mileage.addMileage(mike, mileage1);                 #E``    mileage.addMileage(mike, mileage2);                 #E``    mileage.addMileage(mike, mileage3);                 #E``  }`` ``  @Then("^the bonus points of the regular passenger should be (**\\d+)$**") #F``  public void the_bonus_points_of_the_regular_passenger_should_be``         (int points) throws Throwable {``    mileage.calculateGivenPoints();                   #G``    assertEquals(points,                        #H``      mileage.getPassengersPointsMap().get(mike).intValue());     #H``  }``  […]``}`

In the listing above, we are doing the following:

·  We declare the instance variables for the test, among which `mileage` and `mike` as a `Passenger` (#A).

·  We write the method corresponding to the “`Given we have a regular passenger with a mileage”` business logic step (#B) by initializing the passenger and the mileage (#C).

·  We write the method corresponding to the “`When the regular passenger travels <mileage1> and <mileage2> and <mileage3>”` business logic step (#D) by adding mileages to the regular passenger `mike` (#E).

·  We write the method corresponding to the “`Then the bonus points of the regular passenger should be <points>”` business logic step (#F) by calculating the given points (#G) and checking that the calculated value is the expected one (#H).

·  The rest of the methods are implemented in a similar way, we have covered the `Given`, `When` and `Then` steps of one scenario.

If we run the bonus points tests now they will fail (fig. 21.12), as the business logic is not yet implemented (the `addMileage` and `calculateGivenPoints` methods are empty, the business logic is implemented after the tests).

![img](http://localhost:8000/39e2591b-e785-4a53-9b15-9c0cdeb566c1/OEBPS/Images/21_img_0012.jpg)

Figure 21.12 Running the bonus points tests before the business logic implementation will fail

John will move back to the implementation of the two remaining business logic methods from the `Mileage` class (`addMileage` and `calculateGivenPoints`), as shown in listing 21.10.

Listing 21.10 The implementation of business logic from the Mileage class

`public void addMileage(Passenger passenger, int miles) {``  if (passengersMileageMap.containsKey(passenger)) {           #A``    passengersMileageMap.put(passenger,                 #B``      passengersMileageMap.get(passenger) + miles);          #B``  } else {``    passengersMileageMap.put(passenger, miles);             #C``  }`` ``}`` ``public void calculateGivenPoints() {``  for (Passenger passenger : passengersMileageMap.keySet()) {       #D``    if (passenger.isVip()) {                      #E``      passengersPointsMap.put(passenger,               #E``       passengersMileageMap.get(passenger)/ VIP_FACTOR);       #E``    } else {``      passengersPointsMap.put(passenger,               #F``       passengersMileageMap.get(passenger)/ REGULAR_FACTOR);     #F``    }``  }``}`

In the listing above, we are doing the following:

·  In the `addMileage` method, we check if the `passengersMileageMap` already contains a passenger (#A). If that passenger already exists, we add the mileage to him (#B), otherwise, we create a new entry into the map having that passenger as a key and the miles as the initial value (#C).

·  In the `calculateGivenPoints` method, we browse the passengers set (#D) and, for each passenger, if he is a VIP, we calculate the bonus points by dividing the mileage to the VIP factor (#E). Otherwise, we calculate the bonus points by dividing the mileage to the regular factor (#F).

Running the bonus points tests now will be successful and nicely displayed, as shown in fig. 21.13.

![img](http://localhost:8000/39e2591b-e785-4a53-9b15-9c0cdeb566c1/OEBPS/Images/21_img_0013.jpg)

Figure 21.13 Running the bonus points tests after the business logic implementation will succeed

John has successfully implemented the bonus policy feature working BDD style, with JUnit 5 and Cucumber.

## 21.3  Working BDD with JBehave and JUnit 5

### 21.3.1  Introducing JBehave

There are a few alternatives in choosing a BDD framework. Besides the already introduced Cucumber, we'll also take a look at another very popular one, JBehave.

JBehave

JBehave is a Behavior Driven Development testing tool framework. As implementing the idea of Behavior Driven Development, JBehave allows us to write stories in plain text, to be understood by all persons involved in the project. Through the stories, we'll define scenarios that express the desired behavior.

Like other BDD frameworks, JBehave provides its terminology. We mention here:

·  Story - covers one or more scenarios and represents an increment of business functionality that can be automatically executed.

·  Scenario - a real-life situation to interact with the application.

·  Step - this is defined using the classic BDD keywords: Given, When and Then.

### 21.3.2  Moving a TDD feature to JBehave

John would like to implement, with the help of JBehave, the same features, and tests that he has implemented with Cucumber. This will allow the possibility to make some comparisons between the two BDD frameworks and conclude which one to use.

John will first introduce the JBehave dependency into the Maven configuration (listing 21.11). He will first create a JBehave story, generate the tests skeleton and fill it in.

Listing 21.11 The JBehave dependency added to the pom.xml file

`<dependency>``  <groupId>org.jbehave</groupId>``  <artifactId>jbehave-core</artifactId>``  <version>4.1</version>``</dependency>`

Then John will install the plugins for IntelliJ. He goes to File -> Settings -> Plugins -> Browse Repositories, type JBehave, and choose JBehave Step Generator and JBehave Support (fig 21.14).

![img](http://localhost:8000/39e2591b-e785-4a53-9b15-9c0cdeb566c1/OEBPS/Images/21_img_0014.jpg)

Figure 21.14 Installing the JBehave for Java plugins from the File -> Settings -> Plugins menu

John will start creating the story. He will follow the Maven standard folders structure and introduce the stories into the test/resources folder. He will create a folders structure com/manning/junitbook/airport and insert here the passengers_policy_story.story file. He will also create, into the test folder, the com.manning.junitbook.airport package containing the PassengersPolicy class (fig. 21.15).

![img](http://localhost:8000/39e2591b-e785-4a53-9b15-9c0cdeb566c1/OEBPS/Images/21_img_0015.jpg)

Figure 21.15 The newly introduced PassengersPolicy class corresponds to the story file to be found in the test/resources/com/manning/junitbook/airport folder

The story will contain meta-information about itself, the narrative (what it intends to do) and the scenarios (listing 21.12).

Listing 21.12 The passengers_policy_story.story file

`Meta: Passengers Policy``   The company follows a policy of adding and removing passengers, ``   depending on the passenger type and on the flight type`` ``Narrative:``As a company``I want to be able to manage passengers and flights``So that the policies of the company are followed`` ``Scenario: Economy flight, regular passenger``Given there is an economy flight``When we have a regular passenger``Then you can add and remove him from an economy flight``And you cannot add a regular passenger to an economy flight more than once`` ``Scenario: Economy flight, VIP passenger``Given there is an economy flight``When we have a VIP passenger``Then you can add him but cannot remove him from an economy flight``And you cannot add a VIP passenger to an economy flight more than once`` ``Scenario: Business flight, regular passenger``Given there is a business flight``When we have a regular passenger``Then you cannot add or remove him from a business flight`` ``Scenario: Business flight, VIP passenger``Given there is a business flight``When we have a VIP passenger``Then you can add him but cannot remove him from a business flight``And you cannot add a VIP passenger to a business flight more than once`` ``Scenario: Premium flight, regular passenger``Given there is a premium flight``When we have a regular passenger``Then you cannot add or remove him from a premium flight`` ``Scenario: Premium flight, VIP passenger``Given there is a premium flight``When we have a VIP passenger``Then you can add and remove him from a premium flight``And you cannot add a VIP passenger to a premium flight more than once`

In order to generate the steps into a Java file, John will place the cursor on any not yet created test step (they are underlined in red) and press Alt+Enter (fig. 21.16).

![img](http://localhost:8000/39e2591b-e785-4a53-9b15-9c0cdeb566c1/OEBPS/Images/21_img_0016.jpg)

Figure 21.16 Pressing Alt+ Enter and generating the BDD steps into a class

He will generate all steps into the newly created `PassengersPolicy` class (fig. 21.17).

![img](http://localhost:8000/39e2591b-e785-4a53-9b15-9c0cdeb566c1/OEBPS/Images/21_img_0017.jpg)

Figure 21.17 Choosing PassengersPolicy as the class to generate the steps of the story into

The skeleton will look as in listing 21.13, with the tests needing to be filled in.

Listing 21.13 The skeleton of the JBehave PassengersPolicy test

`public class PassengersPolicy {``  @Given("there is an economy flight")``  public void givenThereIsAnEconomyFlight() {`` ``  }`` ``  @When("we have a regular passenger")``  public void whenWeHaveARegularPassenger() {`` ``  }`` ``  @Then("you can add and remove him from an economy flight")``  public void thenYouCanAddAndRemoveHimFromAnEconomyFlight() {`` ``  }``  […]``}`

John will now implement the tests according to the business logic, as in listing 21.14. He will write the code corresponding to each step defined through a method.

Listing 21.14 The implemented tests from PassengersPolicy

`public class PassengersPolicy {``  private Flight economyFlight;                      #A``  private Passenger mike;                         #A``  […]`` ``  @Given("there is an economy flight")                  #B``  public void givenThereIsAnEconomyFlight() {``    economyFlight = new EconomyFlight("1");               #C``  }`` ``  @When("we have a regular passenger")                  #D``  public void whenWeHaveARegularPassenger() {``    mike = new Passenger("Mike", false);                #E``  }`` ``  @Then("you can add and remove him from an economy flight")       #F``  public void thenYouCanAddAndRemoveHimFromAnEconomyFlight() {``    assertAll("Verify all conditions for a regular passenger      #G``          and an economy flight",                 #G``      () -> assertEquals("1", economyFlight.getId()),         #G``      () -> assertEquals(true,                     #G``         economyFlight.addPassenger(mike)),             #G``      () -> assertEquals(1,                      #G``         economyFlight.getPassengersSet().size()),         #G``      () -> assertEquals("Mike", new ArrayList<>(           #G``       economyFlight.                         #G``        getPassengersSet()).get(0).getName()),            #G``      () -> assertEquals(true,                     #G``            economyFlight.removePassenger(mike)),        #G``      () -> assertEquals(0,                      #G``            economyFlight.getPassengersSet().size())       #G``    );``  }``[…]`` ``}`

In the listing above, we are doing the following:

·  We declare the instance variables for the test, among which `economyFlight` and `mike` as a `Passenger` (#A).

·  We write the method corresponding to the “`Given there is an economy flight”` business logic step (#B) by initializing the `economyFlight` (#C).

·  We write the method corresponding to the “`When we have a regular passenger”` business logic step (#D) by initializing the regular passenger `mike` (#E).

·  We write the method corresponding to the “`Then you can add and remove him from an economy flight”` business logic step (#F) by checking all the conditions by using the `assertAll` JUnit 5 method, which can now be read in a flow (#G).

·  The rest of the methods are implemented in a similar way, we have covered the `Given`, `When` and `Then` steps of one scenario.

In order to be able to run these tests, John will need a new special class that will represent the test configuration. He will name this class `PassengersPolicyStory` (listing 21.15).

Listing 21.15 The PassengersPolicyStory class

`public class PassengersPolicyStory extends JUnitStory {           #A`` ``  @Override``  public Configuration configuration() {                 #B``    return new MostUsefulConfiguration()                #C``         .useStoryReporterBuilder(                 #D``           new StoryReporterBuilder().              #D``             withFormats(Format.CONSOLE));           #D``  }`` ``  @Override``  public InjectableStepsFactory stepsFactory() {             #E``    return new InstanceStepsFactory(configuration(),          #F``           new PassengersPolicy());               #F``  }``}`

In the listing above, we are doing the following:

·  We are declaring the `PassengersPolicyStory` class that is extending `JUnitStory` (#A). A JBehave story class must extend this `JUnitStory.`

·  We override the `configuration` method (#B) and we tell that the configuration of the report is the one that works for the most situations that users are likely to encounter (#C), plus that the report will be displayed on the console (#D).

·  We override the `stepsFactory` method (#E) and we tell that the steps definition is to be found in the `PassengersPolicy` class (#F).

The result of running these tests is shown in fig. 21.18. The tests are successfully running, the code coverage is 100%. However, the reporting capabilities of JBehave do not allow the same nice display as in the case of Cucumber.

![img](http://localhost:8000/39e2591b-e785-4a53-9b15-9c0cdeb566c1/OEBPS/Images/21_img_0018.jpg)

Figure 21.18 The JBehave passengers policy tests run successfully, the code coverage is 100%

We compare the length of the pre-BDD `AirportTest` class, which has 207 lines, with the one of this JBehave `PassengersPolicy` class, which has 157 lines (just like for the Cucumber version). The testing code is now at only 75% of the pre-BDD size, while we have shown that it has the same 100% coverage. Where does this gain come from? Remember that the AirportTest file contained 7 classes, on 3 levels: `AirportTest` at the top level; one `EconomyFlightTest` and one `BusinessFlightTest` at the second level; and, at the third level, two `RegularPassenger` and two `VipPassenger` classes. The code duplication is now really jumping to our attention, but that was the solution having only JUnit 5.

### 21.3.3  Adding a new feature with the help of JBehave

John would like to implement, with the help of JBehave, the same new feature concerning the policy of bonus points that are awarded to the passenger.

The specifications about calculating the bonus points consider the mileage, meaning the distance that is traveled by each passenger. The bonus will be calculated for all flights of the passenger and it depends on a factor: the mileage will be divided by 10 for VIP passenger and by 20 for regular ones.

John will define the scenarios of awarding the bonus points into the bonus_policy_story.story file (listing 21.16) and generate the JBehave tests that describe the scenarios. In the beginning, they are expected to fail.

Listing 21.16 The bonus_policy_story.story file

`Meta: Bonus Policy``   The company follows a bonus policy, depending on the passenger type and on the mileage`` ``Narrative:``As a company``I want to be able to manage the bonus awarding``So that the policies of the company are followed`` ``Scenario: Regular passenger bonus policy                  #A``Given we have a regular passenger with a mileage``When the regular passenger travels <mileage1> and <mileage2> and <mileage3>#B``Then the bonus points of the regular passenger should be <points>     #B`` ``Examples:``| mileage1 | mileage2 | mileage3| points |                 #C``|   349 |   319 |  623 |   64 |                 #C``|   312 |   356 |  135 |   40 |                 #C``|   223 |   786 |  503 |   75 |                 #C``|   482 |   98 |  591 |   58 |                 #C``|   128 |   176 |  304 |   30 |                 #C`` ``Scenario: VIP passenger bonus policy                    #A``Given we have a VIP passenger with a mileage``When the VIP passenger travels <mileage1> and <mileage2> and <mileage3>  #B``Then the bonus points of the VIP passenger should be <points>       #B`` ``Examples:``| mileage1 | mileage2 | mileage3| points |                #C``|   349 |   319 |  623 |   129 |                #C``|   312 |   356 |  135 |   80 |                #C``|   223 |   786 |  503 |   151 |                #C``|   482 |   98 |  591 |   117 |                #C``|   128 |   176 |  304 |   60 |                #C`

In the listing above, we are doing the following:

·  We introduce new scenarios for the bonus policy, using the Given, When and Then keywords (#A).

·  Values are replaced with parameters as into the step-definition itself - you can see <mileage1>, <mileage2>, <mileage3> and <points> as parameters (#B).

·  The effective values are defined in the Examples table, at the end of each Scenario (#C). The first row in the first table defines the values of 3 mileages (349, 319, 623). Adding them and dividing them by 20 (the regular passenger factor), we get the integer part 64 (the number of bonus points). This successfully replaces the JUnit 5 parameterized tests, having the advantage that the values are kept inside the scenarios, and easy to be understood by everyone.

John will create, into the test folder, the `BonusPolicy` class into the com.manning.junitbook.airport package (fig. 21.19).

![img](http://localhost:8000/39e2591b-e785-4a53-9b15-9c0cdeb566c1/OEBPS/Images/21_img_0019.jpg)

Figure 21.19 The newly introduced BonusPolicy class corresponds to the story file to be found in the test/resources/com/manning/junitbook/airport folder

In order to generate the steps into a Java file, John will place the cursor on any not yet created step and press Alt+Enter (fig. 21.20).

![img](http://localhost:8000/39e2591b-e785-4a53-9b15-9c0cdeb566c1/OEBPS/Images/21_img_0020.jpg)

Figure 21.20 Pressing Alt+ Enter and generating the BDD steps into a class

He will generate all steps into the newly created `BonusPolicy` class (fig. 21.21).

![img](http://localhost:8000/39e2591b-e785-4a53-9b15-9c0cdeb566c1/OEBPS/Images/21_img_0021.jpg)

Figure 21.21 Choosing BonusPolicy as the class to generate the steps of the story into

The `BonusPolicy` class is shown in listing 21.17 with the tests skeleton filled in.

Listing 21.17 The skeleton of the JBehave BonusPolicy test

`public class BonusPolicy {`` ``  @Given("we have a regular passenger with a mileage")          #A``  public void givenWeHaveARegularPassengerWithAMileage() {`` ``  }`` ``  @When("the regular passenger travels <mileage1> and           #B``      <mileage2> and <mileage3>")                   #B``  public void whenTheRegularPassengerTravelsMileageAndMileageAndMileage(``      @Named("mileage1") int mileage1, @Named("mileage2") int mileage2, ``      @Named("mileage3") int mileage3) {`` ``  }`` ``  @Then("the bonus points of the regular passenger should be <points>")  #C``  public void thenTheBonusPointsOfTheRegularPassengerShouldBePoints(``   @Named("points") int points) {`` ``  }``  […]``}`

In the listing above we are doing the following:

·  The JBehave plugin has generated a method annotated with `@Given(“we have a regular passenger with a mileage”)`, meaning that this method will be executed when the step “`Given we have a regular passenger with a mileage”` from the scenario will be executed (#A).

·  The JBehave plugin has generated a method annotated with `@When(“the regular passenger travels <mileage1> and <mileage2> and <mileage3>”)`, meaning that this method will be executed when the step “`When the regular passenger travels <mileage1> and <mileage2> and <mileage3>”` from the scenario will be executed (#B).

·  The JBehave plugin has generated a method annotated with `@Then(“the bonus points of the regular passenger should be <points>”)`, meaning that this method will be executed when the step “`Then the bonus points of the regular passenger should be <points>”` from the scenario will be executed (#C).

·  The rest of the methods generated by the JBehave plugin are similar, we have covered the `Given`, `When` and `Then` steps of one scenario.

Now, John will create the Mileage class, declaring the fields and the methods, but not implementing them yet (listing 21.18). John needs to use the methods of this class for the tests, make these tests initially fail, then implement the methods and make the tests pass.

Listing 21.18 The Mileage class, with no implementation of the methods

`public class Mileage {`` ``  public static final int VIP_FACTOR = 10;                #A``  public static final int REGULAR_FACTOR = 20;              #A`` ``  private Map<Passenger, Integer> passengersMileageMap = new HashMap<>(); #B``  private Map<Passenger, Integer> passengersPointsMap = new HashMap<>(); #B`` ``  public void addMileage(Passenger passenger, int miles) {        #C`` ``  }`` ``  public void calculateGivenPoints() {                  #D`` ``  }`` ``}`

In the listing above, we are doing the following:

·  We have declared the `VIP_FACTOR` and `REGULAR_FACTOR` constants, corresponding to the factor by which we divide the mileage for each type of passenger in order to get the bonus points (#A).

·  We have declared `passengersMileageMap` and `passengersPointsMap`, two maps having as key the passenger and keeping as value the mileage and the points for that passenger, respectively (#B).

·  We have declared the `addMileage` method, which will populate the `passengersMileageMap` with the mileage for each passenger (#C). The method does not do anything, for now, it will be written later, to fix the tests.

·  We have declared the `calculateGivenPoints` method, which will populate the `passengersPointsMap` with the bonus points for each passenger (#D). The method does not do anything, for now, it will be written later, to fix the tests.

John will now turn his attention to write the unimplemented tests from the BonusPolicy class, to follow the business logic of this feature (listing 21.19).

Listing 21.19 The business logic of the steps from BonusPolicy

`public class BonusPolicy {``  private Passenger mike;                         #A``  private Mileage mileage;                        #A``  […]`` ``  @Given("we have a regular passenger with a mileage")          #B``  public void givenWeHaveARegularPassengerWithAMileage() {``    mike = new Passenger("Mike", false);                #C``    mileage = new Mileage();                      #C``  }`` ``  @When("the regular passenger travels <milege1> and <mileage2> and    #D``                    <mileage3>")             #D``  public void the_regular_passenger_travels_and_and(@Named(“mileage1”)``        int mileage1, @Named(“mileage2”) int mileage2, ``        @Named(“mileage3”) int mileage3) {``    mileage.addMileage(mike, mileage1);                 #E``    mileage.addMileage(mike, mileage2);                 #E``    mileage.addMileage(mike, mileage3);                 #E``  }`` ``  @Then("the bonus points of the regular passenger should be <points>")  #F``  public void the_bonus_points_of_the_regular_passenger_should_be``         (@Named(“points”) int points) {``    mileage.calculateGivenPoints();                   #G``    assertEquals(points,                        #H``      mileage.getPassengersPointsMap().get(mike).intValue());     #H``  }``  […]``}`

In the listing above, we are doing the following:

·  We declare the instance variables for the test, among which `mileage` and `mike` as a `Passenger` (#A).

·  We write the method corresponding to the “`Given we have a regular passenger with a mileage”` business logic step (#B) by initializing the passenger and the mileage (#C).

·  We write the method corresponding to the “`When the regular passenger travels <mileage1> and <mileage2> and <mileage3>”` business logic step (#D) by adding mileages to the regular passenger `mike` (#E).

·  We write the method corresponding to the “`Then the bonus points of the regular passenger should be <points>”` business logic step (#F) by calculating the given points (#G) and checking that the calculated value is the expected one (#H).

·  The rest of the methods are implemented in a similar way, we have covered the `Given`, `When` and `Then` steps of one scenario.

In order to be able to run these tests, John will need a new special class that will represent the test configuration. He will name this class `BonusPolicyStory` (listing 21.20).

Listing 21.20 The BonusPolicyStory class

`public class BonusPolicyStory extends JUnitStory {             #A`` ``  @Override``  public Configuration configuration() {                 #B``    return new MostUsefulConfiguration()                #C``         .useStoryReporterBuilder(                 #D``           new StoryReporterBuilder().              #D``             withFormats(Format.CONSOLE));           #D``  }`` ``  @Override``  public InjectableStepsFactory stepsFactory() {             #E``    return new InstanceStepsFactory(configuration(),          #F``           new BonusPolicy());                  #F``  }``}`

In the listing above, we are doing the following:

·  We are declaring the `BonusPolicyStory` class that is extending `JUnitStory` (#A). A JBehave story class must extend this `JUnitStory.`

·  We override the `configuration` method (#B) and we tell that the configuration of the report is the one that works for the most situations that users are likely to encounter (#C), plus that the report will be displayed on the console (#D).

·  We override the `stepsFactory` method (#E) and we tell that the steps definition is to be found into the `BonusPolicy` class (#F).

If we run the bonus points tests now they will fail (fig. 21.22), as the business logic is not yet implemented (we have left the `addMileage` and `calculateGivenPoints` methods empty).

![img](http://localhost:8000/39e2591b-e785-4a53-9b15-9c0cdeb566c1/OEBPS/Images/21_img_0022.jpg)

Figure 21.22 Running the JBehave bonus points tests before the business logic implementation will fail

John will move back to the implementation of the two remaining business logic methods from the `Mileage` class (`addMileage` and `calculateGivenPoints`), as shown in listing 21.21.

Listing 21.21 The implementation of business logic from the Mileage class

`public void addMileage(Passenger passenger, int miles) {``  if (passengersMileageMap.containsKey(passenger)) {           #A``    passengersMileageMap.put(passenger,                 #A``      passengersMileageMap.get(passenger) + miles);          #A``  } else {``    passengersMileageMap.put(passenger, miles);             #B``  }`` ``}`` ``public void calculateGivenPoints() {``  for (Passenger passenger : passengersMileageMap.keySet()) {       #C``    if (passenger.isVip()) {                      #D``      passengersPointsMap.put(passenger,               #D``       passengersMileageMap.get(passenger)/ VIP_FACTOR);       #D``    } else {``      passengersPointsMap.put(passenger,               #E``       passengersMileageMap.get(passenger)/ REGULAR_FACTOR);     #E``    }``  }``}`

In the listing above, we are doing the following:

·  In the `addMileage` method, we check if the `passengersMileageMap` already contains a passenger (#A). If that passenger already exists, we add the mileage to him (#B), otherwise, we create a new entry into the map having that passenger as a key and the miles as the initial value (#C).

·  In the `calculateGivenPoints` method, we browse the passengers set (#C) and, for each passenger, if he is a VIP, we calculate the bonus points by dividing the mileage to the VIP factor (#D). Otherwise, we calculate the bonus points by dividing the mileage to the regular factor (#E).

Running the bonus points tests now will be successful, as shown in fig. 21.23.

![img](http://localhost:8000/39e2591b-e785-4a53-9b15-9c0cdeb566c1/OEBPS/Images/21_img_0023.jpg)

Figure 21.23 Running the JBehave bonus points tests after the business logic implementation will succeed

John has successfully implemented the bonus policy feature working BDD style, with JUnit 5 and JBehave.

## 21.4  Comparing Cucumber and JBehave

Cucumber and JBehave have similar approaches, supporting the Behaviour Driven Development ideas. Cucumber and JBehave are different frameworks but built on the same well defined BDD principles that we have emphasized.

They are based around features (Cucumber) or stories (JBehave). A feature is a collection of stories, expressed from the point of view of a specific project stakeholder.

You may notice the same BDD keywords Given, When, Then, but also some variations in names like "Scenario Outline" for Cucumber, or simply Scenario with Examples for JBehave.

The IntelliJ IDE support for both Cucumber and JBehave is provided through plugins that help from the steps generation in Java code up to checking the code coverage. The Cucumber plugin has been able to produce a nicer output, allowing us to follow the full testing hierarchy at a glance, and displaying everything with significant colors. More, it allowed us to directly run a test from the feature text file, which is easier to follow, especially by non-technical persons.

JBehave has reached its maturity phase some time ago, while the Cucumber codebase is still updated very frequently. At the time of writing this chapter, the Cucumber Github code displays tens of commits from within the previous 7 days. The JBehave Github code displays one single commit within the last 7 days.

Cucumber has also a more active community at the time of writing this chapter, the articles on blogs and forums are more recent and frequent. Consequently, this makes troubleshooting easier for developers.

In terms of code size compared to the pre-BDD situation, both Cucumber and JBehave have shown similar performances, reducing the size of the initial code in the same proportion.

Choosing one of these frameworks may be a matter of habit or preference - personal preference or project preference. We have intended to put them face to face in a very practical and comparable way so that you can eventually make your own choice.

The next chapter will be dedicated to building a test pyramid strategy, from the low level to the high level, and applying it in working with JUnit 5.

## 21.5  Summary

This chapter has covered the following:

·  Introducing Behavior Driven Development, a software development technique that encourages teams to deliver software that matters, supporting the cooperation between stakeholders.

·  Analyzing the benefits of Behavior Driven Development: addressing user needs, clarity, change support, automation support, focus on adding business value, costs reduction.

·  Analyzing the challenges of Behavior Driven Development: it requires engagement and strong collaboration, interaction, direct communication, and constant feedback.

·  Moving a TDD application to BDD, by moving the passengers policy business logic to be implemented with the help of both Cucumber and JBehave.

·  Developing the business logic for the bonus policy with the help of Cucumber, by creating a separate feature, generating the skeleton of the testing code, writing the tests and implementing the code.

·  Developing the business logic for the bonus policy with the help of JBehave, by creating a separate story, generating the skeleton of the testing code, writing the tests and implementing the code.

·  Comparing Cucumber and JBehave in terms of approaching the BDD principles, ease of use, code size needed to implement some functionality.





# 22. Implementing a test pyramid strategy with JUnit 5

## 22.1  Software testing levels

**Describing the levels of software testing, we'll enumerate the following (from the lowest to the highest):**

* Unit testing - methods or classes (meaning individual units) are tested to determine whether they are working correctly
* Integration testing – the individual software components are combined and tested together.
* System testing - testing is performed on a complete, full system, in order to evaluate the system compliance with the specification.
* Acceptance testing – an application is verified to satisfy the expectations of the end-user.

If we ask ourselves what to test, we identify the following:

* Business logic - how, the program transposes the business rules from the real-world
* Bad input values – e.g, we cannot assign a negative number of seats to a flight.
* Boundary conditions - extremes of some input domain, e.g. maximum, minimum. We may test on flights having 0 passengers or the maximum allowed passengers.
* Unexpected conditions –conditions that are not part of the normal operation of a program. A flight cannot change its origin once it has taken off.
* Invariants - expressions whose values do not change during program execution, e.g., the identifier of a person cannot change during the execution of the program.
* Regressions - bugs introduced in an existing system after upgrades- or patches.

![](https://pic.imgdb.cn/item/6157e3d72ab3f51d91aafe98.jpg)

* The unit testing is at its foundation. It focuses on each piece of software by testing it in isolating, to determine if it is according to expectations.
* Integration testing takes its input from already verified units, groups them in larger aggregates and executes integration tests on them.
* System testing requires no knowledge of the design or of the code but focuses on the functionality of the whole system.
* Acceptance testing use scenarios and test cases to check if the application satisfies the expectations of the end-user.

## 22.2  Unit testing – our basic components work in isolation

略

## 22.3  Integration testing – units combined as a group

略

## 22.4  System testing – looking at the complete software

![](https://pic.imgdb.cn/item/6157e4b92ab3f51d91ac66ab.jpg)

## 22.5  Acceptance testing – compliance with the business requirements

略

