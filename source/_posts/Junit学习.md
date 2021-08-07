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

