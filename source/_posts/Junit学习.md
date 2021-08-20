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
             <artifac tId>maven-surefire-plugin</artifactId>
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

