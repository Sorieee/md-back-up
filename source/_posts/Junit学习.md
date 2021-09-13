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

