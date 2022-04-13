http://www.groovy-lang.org/documentation.html

http://www.groovy-lang.org/single-page-documentation.html

# 安装

下载安装包解压，配置环境变量即可。

Windows:

配置GROOVY_HOME
配置%GROOVY_HOME%\bin

```
groovy -version
```

# 与Java的区别

## 1.默认导入

​	会默认导入如下包:

- java.io.*
- java.lang.*
- java.math.BigDecimal
- java.math.BigInteger
- java.net.*
- java.util.*
- groovy.lang.*
- groovy.util.*

### 2. Multi-methods

​	Groovy中，方法调用会在运行时进行选择，这叫做运行时分派或者multi-methods。

```
int method(String arg) {
    return 1;
}
int method(Object arg) {
    return 2;
}
Object o = "Object";
int result = method(o);
```

In Java, you would have:

```
assertEquals(2, result);
```

Whereas in Groovy:

```
assertEquals(1, result);
```

​	这是因为 Java 将使用静态信息类型，即 o 被声明为 Object，而 Groovy 将在运行时选择，即实际调用该方法时。既然是用字符串调用的，那么就调用字符串版本。

## 3. Array initializers

In Java, array initializers take either of these two forms:

```java
int[] array = {1, 2, 3};             // Java array initializer shorthand syntax
int[] array2 = new int[] {4, 5, 6};  // Java array initializer long syntax
```

In Groovy, the `{ … }` block is reserved for closures. That means that you cannot create array literals using Java’s array initializer shorthand syntax. You instead borrow Groovy’s literal list notation like this:

```groovy
int[] array = [1, 2, 3]
```

For Groovy 3+, you can optionally use the Java’s array initializer long syntax:

```groovy
def array2 = new int[] {1, 2, 3} // Groovy 3.0+ supports the Java-style array initialization long syntax
```

## 4. Package scope visibility

In Groovy, omitting a modifier on a field doesn’t result in a package-private field like in Java:

```groovy
class Person {
    String name
}
```

Instead, it is used to create a *property*, that is to say a *private field*, an associated *getter* and an associated *setter*.

It is possible to create a package-private field by annotating it with `@PackageScope`:

```groovy
class Person {
    @PackageScope String name
}
```

## 5. ARM blocks

Java 7 introduced ARM (Automatic Resource Management) blocks like this:

```java
Path file = Paths.get("/path/to/file");
Charset charset = Charset.forName("UTF-8");
try (BufferedReader reader = Files.newBufferedReader(file, charset)) {
    String line;
    while ((line = reader.readLine()) != null) {
        System.out.println(line);
    }

} catch (IOException e) {
    e.printStackTrace();
}
```

Such blocks are supported from Groovy 3+. However, Groovy provides various methods relying on closures, which have the same effect while being more idiomatic. For example:

```groovy
new File('/path/to/file').eachLine('UTF-8') {
   println it
}
```

or, if you want a version closer to Java:

```groovy
new File('/path/to/file').withReader('UTF-8') { reader ->
   reader.eachLine {
       println it
   }
}
```

## 6. Inner Class

### 6.1 Static inner classes

Here’s an example of static inner class:

```groovy
class A {
    static class B {}
}

new A.B()
```

The usage of static inner classes is the best supported one. If you absolutely need an inner class, you should make it a static one.

### 6.2. Anonymous Inner Classes

```gr'
import java.util.concurrent.CountDownLatch
import java.util.concurrent.TimeUnit

CountDownLatch called = new CountDownLatch(1)

Timer timer = new Timer()
timer.schedule(new TimerTask() {
    void run() {
        called.countDown()
    }
}, 0)

assert called.await(10, TimeUnit.SECONDS)
```

### 6.3. Creating Instances of Non-Static Inner Classes

In Java you can do this:

```java
public class Y {
    public class X {}
    public X foo() {
        return new X();
    }
    public static X createX(Y y) {
        return y.new X();
    }
}
```

Before 3.0.0, Groovy doesn’t support the `y.new X()` syntax. Instead, you have to write `new X(y)`, like in the code below:

```groovy
public class Y {
    public class X {}
    public X foo() {
        return new X()
    }
    public static X createX(Y y) {
        return new X(y)
    }
}
```

> Caution though, Groovy supports calling methods with one parameter without giving an argument. The parameter will then have the value null. Basically the same rules apply to calling a constructor. There is a danger that you will write new X() instead of new X(this) for example. Since this might also be the regular way we have not yet found a good way to prevent this problem.
>
> Groovy 3.0.0 supports Java style syntax for creating instances of non-static inner classes.

## 7. Lambda expressions and the method reference operator

Java 8+ supports lambda expressions and the method reference operator (`::`):

```java
Runnable run = () -> System.out.println("Run");  // Java
list.forEach(System.out::println);
```

Groovy 3 and above also support these within the Parrot parser. In earlier versions of Groovy you should use closures instead:

```groovy
Runnable run = { println 'run' }
list.each { println it } // or list.each(this.&println)
```

## 8. GStrings

​	As double-quoted string literals are interpreted as `GString` values, Groovy may fail with compile error or produce subtly different code if a class with `String` literal containing a dollar character is compiled with Groovy and Java compiler.

​	While typically, Groovy will auto-cast between `GString` and `String` if an API declares the type of a parameter, beware of Java APIs that accept an `Object` parameter and then check the actual type.

## 9. String and Character literals

Singly-quoted literals in Groovy are used for `String`, and double-quoted result in `String` or `GString`, depending whether there is interpolation in the literal.

```groovy
assert 'c'.getClass()==String
assert "c".getClass()==String
assert "c${1}".getClass() in GString
```

Groovy will automatically cast a single-character `String` to `char` only when assigning to a variable of type `char`. When calling methods with arguments of type `char` we need to either cast explicitly or make sure the value has been cast in advance.

```groovy
char a='a'
assert Character.digit(a, 16)==10 : 'But Groovy does boxing'
assert Character.digit((char) 'a', 16)==10

try {
  assert Character.digit('a', 16)==10
  assert false: 'Need explicit cast'
} catch(MissingMethodException e) {
}
```

Groovy supports two styles of casting and in the case of casting to `char` there are subtle differences when casting a multi-char strings. The Groovy style cast is more lenient and will take the first character, while the C-style cast will fail with exception.

```groovy
// for single char strings, both are the same
assert ((char) "c").class==Character
assert ("c" as char).class==Character

// for multi char strings they are not
try {
  ((char) 'cx') == 'c'
  assert false: 'will fail - not castable'
} catch(GroovyCastException e) {
}
assert ('cx' as char) == 'c'
assert 'cx'.asType(char) == 'c'
```

## 10. Behaviour of `==`

In Java, `==` means equality of primitive types or identity for objects. In Groovy, `==` means equality in all places. For non-primitives, it translates to `a.compareTo(b) == 0`, when evaluating equality for `Comparable` objects, and `a.equals(b)` otherwise.

To check for identity (reference equality), use the `is` method: `a.is(b)`. From Groovy 3, you can also use the `===` operator (or negated version): `a === b` (or `c !== d`).

## 11. Primitives and wrappers

In a pure object-oriented language, everything would be an object. Java takes the stance that primitive types, such as int, boolean and double, are used very frequently and worthy of special treatment. Primitives can be efficiently stored and manipulated but can’t be used in all contexts where an object could be used. Luckily, Java auto boxes and unboxes primitives when they are passed as parameters or used as return types:

```
jshell> class Main {
   ...>   float f1 = 1.0f;
   ...>   Float f2 = 2.0f;
   ...>   float add(Float a1, float a2) { return a1 + a2; }
   ...>   Float calc() { return add(f1, f2); } 
   ...> }
|  created class Main

jshell> new Main().calc()
$2 ==> 3.0
```

>The `add` method expects wrapper then primitive type arguments, but we are supplying parameters with a primitive then wrapper type. Similarly, the return type from `add` is primitive, but we need the wrapper type.



Groovy does the same:

```
groovy:000> class Main {
groovy:001>   float f1 = 1.0f
groovy:002>   Float f2 = 2.0f
groovy:003>   float add(Float a1, float a2) { a1 + a2 }
groovy:004>   Float calc() { add(f1, f2) }
groovy:005> }
===> true
groovy:000> new Main().calc()
===> 3.0
```

Groovy, also supports primitives and object types, however, it goes a little further in pushing OO purity; it tries hard to treat *everything* as an object. Any primitive typed variable or field can be treated like an object and it will be [autowrapped](https://docs.groovy-lang.org/latest/html/documentation/core-object-orientation.html#_primitive_types) as needed. While primitive types might be used under the covers, their use should be indistinguishable from normal object use whenever possible and they will be boxed/unboxed as needed.

Here is a little example using Java trying to (incorrectly for Java) dereference a primitive `float`:

```
jshell> class Main {
   ...>     public float z1 = 0.0f;
   ...> }
|  created class Main

jshell> new Main().z1.equals(1.0f)
|  Error:
|  float cannot be dereferenced
|  new Main().z1.equals(1.0f)
|  ^------------------^
```

The same example using Groovy compiles and runs successfully:

```
groovy:000> class Main {
groovy:001>     float z1 = 0.0f
groovy:002> }
===> true
groovy:000> new Main().z1.equals(1.0f)
===> false
```

Because of Groovy’s additional use of un/boxing, it does not follow Java’s behavior of widening taking priority over boxing. Here’s an example using `int`

```
int i
m(i)

void m(long l) { // 1 
    println "in m(long)"
}

void m(Integer i) {   // 2   
    println "in m(Integer)"
}
```

1. This is the method that Java would call, since widening has precedence over unboxing.

2. This is the method Groovy actually calls, since all primitive references use their wrapper class.

### 11.1. Numeric Primitive Optimisation with `@CompileStatic`

Since Groovy converts to wrapper classes in more places, you might wonder whether it produces less efficient bytecode for numeric expressions. Groovy has a highly optimised set of classes for doing math computations. When using `@CompileStatic`, expressions involving only primitives uses the same bytecode that Java would use.

### 11.2. Positive/Negative zero edge case

Java float/double operations for both primitives and wrapper classes follow the IEEE 754 standard but there is an interesting edge case involving positive and negative zero. The standard supports distinguishing between these two cases and while in many scenarios programmers may not care about the difference, in some mathematical or data science scenarios it is important to cater for the distinction.

For primitives, Java maps down onto a special [bytecode instruction](https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-6.html#jvms-6.5.dcmp_op) when comparing such values which has the property that "Positive zero and negative zero are considered equal".

```
jshell> float f1 = 0.0f
f1 ==> 0.0

jshell> float f2 = -0.0f
f2 ==> -0.0

jshell> f1 == f2
$3 ==> true
```

For the wrapper classes, e.g. [java.base/java.lang.Float#equals(java.lang.Object)](https://docs.oracle.com/en/java/javase/11/docs/api/index.html?java/base/java/lang/Float.html#equals(java.lang.Object)), the result is `false` for this same case.

```
jshell> Float f1 = 0.0f
f1 ==> 0.0

jshell> Float f2 = -0.0f
f2 ==> -0.0

jshell> f1.equals(f2)
$3 ==> false
```

Groovy on the one hand tries to follow Java behavior closely, but on the other switches automatically between primitives and wrapped equivalents in more places. To avoid confusion we recommend the following guidelines:

- If you wish to distinguish between positive and negative zero, use the `equals` method directly or cast any primitives to their wrapper equivalent before using `==`.
- If you wish to ignore the difference between positive and negative zero, use the `equalsIgnoreZeroSign` method directly or cast any non-primitives to their primitive equivalent before using `==`.

These guidelines are illustrated in the following example:

```
float f1 = 0.0f
float f2 = -0.0f
Float f3 = 0.0f
Float f4 = -0.0f

assert f1 == f2
assert (Float) f1 != (Float) f2

assert f3 != f4         
assert (float) f3 == (float) f4

assert !f1.equals(f2)
assert !f3.equals(f4)

assert f1.equalsIgnoreZeroSign(f2)
assert f3.equalsIgnoreZeroSign(f4)
```

Recall that for non-primitives, `==` maps to `.equals()`



## 12. Conversions

Java does automatic widening and narrowing [conversions](https://docs.oracle.com/javase/specs/jls/se7/html/jls-5.html).

|                   | **Converts to** |          |           |          |         |          |           |            |
| ----------------- | --------------- | -------- | --------- | -------- | ------- | -------- | --------- | ---------- |
| **Converts from** | **boolean**     | **byte** | **short** | **char** | **int** | **long** | **float** | **double** |
| **boolean**       | -               | N        | N         | N        | N       | N        | N         | N          |
| **byte**          | N               | -        | Y         | C        | Y       | Y        | Y         | Y          |
| **short**         | N               | C        | -         | C        | Y       | Y        | Y         | Y          |
| **char**          | N               | C        | C         | -        | Y       | Y        | Y         | Y          |
| **int**           | N               | C        | C         | C        | -       | Y        | T         | Y          |
| **long**          | N               | C        | C         | C        | C       | -        | T         | T          |
| **float**         | N               | C        | C         | C        | C       | C        | -         | Y          |
| **double**        | N               | C        | C         | C        | C       | C        | C         | -          |

\* 'Y' indicates a conversion Java can make, 'C' indicates a conversion Java can make when there is an explicit cast, 'T` indicates a conversion Java can make but data is truncated, 'N' indicates a conversion Java can’t make.

Groovy expands greatly on this.

|                   | **Converts to** |             |          |          |           |           |          |               |         |             |          |          |                |           |           |            |            |                |
| ----------------- | --------------- | ----------- | -------- | -------- | --------- | --------- | -------- | ------------- | ------- | ----------- | -------- | -------- | -------------- | --------- | --------- | ---------- | ---------- | -------------- |
| **Converts from** | **boolean**     | **Boolean** | **byte** | **Byte** | **short** | **Short** | **char** | **Character** | **int** | **Integer** | **long** | **Long** | **BigInteger** | **float** | **Float** | **double** | **Double** | **BigDecimal** |
| **boolean**       | -               | B           | N        | N        | N         | N         | N        | N             | N       | N           | N        | N        | N              | N         | N         | N          | N          | N              |
| **Boolean**       | B               | -           | N        | N        | N         | N         | N        | N             | N       | N           | N        | N        | N              | N         | N         | N          | N          | N              |
| **byte**          | T               | T           | -        | B        | Y         | Y         | Y        | D             | Y       | Y           | Y        | Y        | Y              | Y         | Y         | Y          | Y          | Y              |
| **Byte**          | T               | T           | B        | -        | Y         | Y         | Y        | D             | Y       | Y           | Y        | Y        | Y              | Y         | Y         | Y          | Y          | Y              |
| **short**         | T               | T           | D        | D        | -         | B         | Y        | D             | Y       | Y           | Y        | Y        | Y              | Y         | Y         | Y          | Y          | Y              |
| **Short**         | T               | T           | D        | T        | B         | -         | Y        | D             | Y       | Y           | Y        | Y        | Y              | Y         | Y         | Y          | Y          | Y              |
| **char**          | T               | T           | Y        | D        | Y         | D         | -        | D             | Y       | D           | Y        | D        | D              | Y         | D         | Y          | D          | D              |
| **Character**     | T               | T           | D        | D        | D         | D         | D        | -             | D       | D           | D        | D        | D              | D         | D         | D          | D          | D              |
| **int**           | T               | T           | D        | D        | D         | D         | Y        | D             | -       | B           | Y        | Y        | Y              | Y         | Y         | Y          | Y          | Y              |
| **Integer**       | T               | T           | D        | D        | D         | D         | Y        | D             | B       | -           | Y        | Y        | Y              | Y         | Y         | Y          | Y          | Y              |
| **long**          | T               | T           | D        | D        | D         | D         | Y        | D             | D       | D           | -        | B        | Y              | T         | T         | T          | T          | Y              |
| **Long**          | T               | T           | D        | D        | D         | T         | Y        | D             | D       | T           | B        | -        | Y              | T         | T         | T          | T          | Y              |
| **BigInteger**    | T               | T           | D        | D        | D         | D         | D        | D             | D       | D           | D        | D        | -              | D         | D         | D          | D          | T              |
| **float**         | T               | T           | D        | D        | D         | D         | T        | D             | D       | D           | D        | D        | D              | -         | B         | Y          | Y          | Y              |
| **Float**         | T               | T           | D        | T        | D         | T         | T        | D             | D       | T           | D        | T        | D              | B         | -         | Y          | Y          | Y              |
| **double**        | T               | T           | D        | D        | D         | D         | T        | D             | D       | D           | D        | D        | D              | D         | D         | -          | B          | Y              |
| **Double**        | T               | T           | D        | T        | D         | T         | T        | D             | D       | T           | D        | T        | D              | D         | T         | B          | -          | Y              |
| **BigDecimal**    | T               | T           | D        | D        | D         | D         | D        | D             | D       | D           | D        | D        | D              | T         | D         | T          | D          | -              |

\* 'Y' indicates a conversion Groovy can make, 'D' indicates a conversion Groovy can make when compiled dynamically or explicitly cast, 'T' indicates a conversion Groovy can make but data is truncated, 'B' indicates a boxing/unboxing operation, 'N' indicates a conversion Groovy can’t make.

The truncation uses [Groovy Truth](https://docs.groovy-lang.org/latest/html/documentation/core-semantics.html#the-groovy-truth) when converting to `boolean`/`Boolean`. Converting from a number to a character casts the `Number.intvalue()` to `char`. Groovy constructs `BigInteger` and `BigDecimal` using `Number.doubleValue()` when converting from a `Float` or `Double`, otherwise it constructs using `toString()`. Other conversions have their behavior defined by `java.lang.Number`.

## 13. Extra keywords

Groovy has many of the same keywords as Java and Groovy 3 and above also has the same `var` reserved type as Java. In addition, Groovy has the following keywords:

- `as`
- `def`
- `in`
- `trait`
- `it` // within closures

Groovy is less stringent than Java in that it allows some keywords to appear in places that would be illegal in Java, e.g. the following is valid: `var var = [def: 1, as: 2, in: 3, trait: 4]`. Never-the-less, you are discouraged from using the above keywords in places that might cause confusion even when the compiler might be happy. In particular, avoid using them for variable, method and class names, so our previous `var var` example would be considered poor style.

Additional documentation is available for [keywords](https://docs.groovy-lang.org/latest/html/documentation/core-syntax.html#_keywords).

# Syntax

## 注释

```groovy
// a standalone single line comment
println "hello" // a comment till the end of the line


/* a standalone multiline comment
   spanning two lines */
println "hello" /* a multiline comment starting
                   at the end of a statement */
println 1 /* one */ + 2 /* two */
```

### Groovdoc注释

```groovy
/**
 * A Class description
 */
class Person {
    /** the name of the person */
    String name

    /**
     * Creates a greeting method for a certain person.
     *
     * @param otherPerson the person to greet
     * @return a greeting message
     */
    String greet(String otherPerson) {
       "Hello ${otherPerson}"
    }
}
```

In addition, Groovy supports **Runtime Groovydoc** since 3.0.0, i.e. Groovydoc can be retained at runtime.

> Runtime Groovydoc is disabled by default. It can be enabled by adding JVM option `-Dgroovy.attach.runtime.groovydoc=true`

```groovy
/**@
 * Some class groovydoc for Foo
 */
class Foo {
    /**@
     * Some method groovydoc for bar
     */
    void bar() {
    }
}
assert Foo.class.groovydoc.content.contains('Some class groovydoc for Foo') // 1 
assert Foo.class.getMethod('bar', new Class[0]).groovydoc.content.contains('Some method groovydoc for bar') // 2
```

### 1.4. Shebang line

Beside the single-line comment, there is a special line comment, often called the *shebang* line understood by UNIX systems which allows scripts to be run directly from the command-line, provided you have installed the Groovy distribution and the `groovy` command is available on the `PATH`.

```groovy
#!/usr/bin/env groovy
println "Hello from the shebang line"
```

## 2. 关键字

Groovy has the following reserved keywords:

| abstract     | assert     | break      | case       |
| ------------ | ---------- | ---------- | ---------- |
| catch        | class      | const      | continue   |
| def          | default    | do         | else       |
| enum         | extends    | final      | finally    |
| for          | goto       | if         | implements |
| import       | instanceof | interface  | native     |
| new          | null       | non-sealed | package    |
| public       | protected  | private    | return     |
| static       | strictfp   | super      | switch     |
| synchronized | this       | threadsafe | throw      |
| throws       | transient  | try        | while      |

Of these, `const`, `goto`, `strictfp`, and `threadsafe` are not currently in use.

A trick allows methods to be defined having the same name as a keyword by surrounding the name in quotes as shown in the following example:

```groovy
// reserved keywords can be used for method names if quoted
def "abstract"() { true }
// when calling such methods, the name must be qualified using "this."
this.abstract()
```

n addition, Groovy has the following contextual keywords:

| as     | in    | permits | record |
| ------ | ----- | ------- | ------ |
| sealed | trait | var     | yields |

```groovy
// contextual keywords can be used for field and variable names
def as = true
assert as

// contextual keywords can be used for method names
def in() { true }
// when calling such methods, the name only needs to be qualified using "this." in scenarios which would be ambiguous
this.in()
```



| null | true  | false  | boolean |
| ---- | ----- | ------ | ------- |
| char | byte  | short  | int     |
| long | float | double |         |

```groovy
def "null"() { true }  // not recommended; potentially confusing
assert this.null()     // must be qualified
```

## 3. 标识符

### 3.1 普通标识符

Identifiers start with a letter, a dollar or an underscore. They cannot start with a number.

A letter can be in the following ranges:

- 'a' to 'z' (lowercase ascii letter)
- 'A' to 'Z' (uppercase ascii letter)
- '\u00C0' to '\u00D6'
- '\u00D8' to '\u00F6'
- '\u00F8' to '\u00FF'
- '\u0100' to '\uFFFE'

```groovy
def name
def item3
def with_underscore
def $dollarStart
```

But the following ones are invalid identifiers:

```groovy
def 3tier
def a+b
def a#b
```

All keywords are also valid identifiers when following a dot:

```groovy
foo.as
foo.assert
foo.break
foo.case
foo.catch
```

### 3.2 引号标识符

​	Quoted identifiers appear after the dot of a dotted expression. For instance, the `name` part of the `person.name` expression can be quoted with `person."name"` or `person.'name'`. This is particularly interesting when certain identifiers contain illegal characters that are forbidden by the Java Language Specification, but which are allowed by Groovy when quoted. For example, characters like a dash, a space, an exclamation mark, etc.

```groovy
def map = [:]

map."an identifier with a space and double quotes" = "ALLOWED"
map.'with-dash-signs-and-single-quotes' = "ALLOWED"

assert map."an identifier with a space and double quotes" == "ALLOWED"
assert map.'with-dash-signs-and-single-quotes' == "ALLOWED"
```

As we shall see in the [following section on strings](http://www.groovy-lang.org/syntax.html#all-strings), Groovy provides different string literals. All kind of strings are actually allowed after the dot:

```groovy
map.'single quote'
map."double quote"
map.'''triple single quote'''
map."""triple double quote"""
map./slashy string/
map.$/dollar slashy string/$
```

There’s a difference between plain character strings and Groovy’s GStrings (interpolated strings), as in that the latter case, the interpolated values are inserted in the final string for evaluating the whole identifier:

```groovy
def firstname = "Homer"
map."Simpson-${firstname}" = "Homer Simpson"

assert map.'Simpson-Homer' == "Homer Simpson"
```

## 4. Strings

### 4.1. Single-quoted string

Single-quoted strings are a series of characters surrounded by single quotes:

```groovy
'a single-quoted string'
```

### 4.2. String concatenation

All the Groovy strings can be concatenated with the `+` operator:

```groovy
assert 'ab' == 'a' + 'b'
```

### 4.3. Triple-single-quoted string

Triple-single-quoted strings are a series of characters surrounded by triplets of single quotes:

```
'''a triple-single-quoted string'''
```

> Triple-single-quoted strings are plain `java.lang.String` and don’t support interpolation.

Triple-single-quoted strings may span multiple lines. The content of the string can cross line boundaries without the need to split the string in several pieces and without concatenation or newline escape characters:

```
def aMultilineString = '''line one
line two
line three'''
```

If your code is indented, for example in the body of the method of a class, your string will contain the whitespace of the indentation. The Groovy Development Kit contains methods for stripping out the indentation with the `String#stripIndent()` method, and with the `String#stripMargin()` method that takes a delimiter character to identify the text to remove from the beginning of a string.

When creating a string as follows:

```groovy
def startingAndEndingWithANewline = '''
line one
line two
line three
'''
```

You will notice that the resulting string contains a newline character as first character. It is possible to strip that character by escaping the newline with a backslash:

```groovy
def strippedFirstNewline = '''\
line one
line two
line three
'''

assert !strippedFirstNewline.startsWith('\n')
```

#### 4.3.1. 转义符

You can escape single quotes with the backslash character to avoid terminating the string literal:

```
'an escaped single quote: \' needs a backslash'
```

And you can escape the escape character itself with a double backslash:

```
'an escaped escape character: \\ needs a double backslash'
```

Some special characters also use the backslash as escape character:

| Escape sequence | Character                                                    |
| :-------------- | :----------------------------------------------------------- |
| \b              | backspace                                                    |
| \f              | formfeed                                                     |
| \n              | newline                                                      |
| \r              | carriage return                                              |
| \s              | single space                                                 |
| \t              | tabulation                                                   |
| \\              | backslash                                                    |
| \'              | single quote within a single-quoted string (and optional for triple-single-quoted and double-quoted strings) |
| \"              | double quote within a double-quoted string (and optional for triple-double-quoted and single-quoted strings) |

We’ll see some more escaping details when it comes to other types of strings discussed later.

#### 4.3.2. Unicode escape sequence

For characters that are not present on your keyboard, you can use unicode escape sequences: a backslash, followed by 'u', then 4 hexadecimal digits.

For example, the Euro currency symbol can be represented with:

```
'The Euro currency symbol: \u20AC'
```

### 4.4. Double-quoted string

Double-quoted strings are a series of characters surrounded by double quotes:

```groovy
"a double-quoted string"
```

#### 4.4.1. String interpolation



Any Groovy expression can be interpolated in all string literals, apart from single and triple-single-quoted strings. Interpolation is the act of replacing a placeholder in the string with its value upon evaluation of the string. The placeholder expressions are surrounded by `${}`. The curly braces may be omitted for unambiguous dotted expressions, i.e. we can use just a $ prefix in those cases. If the GString is ever passed to a method taking a String, the expression value inside the placeholder is evaluated to its string representation (by calling `toString()` on that expression) and the resulting String is passed to the method.

Here, we have a string with a placeholder referencing a local variable:

```groovy
def name = 'Guillaume' // a plain string
def greeting = "Hello ${name}"

assert greeting.toString() == 'Hello Guillaume'
```

Any Groovy expression is valid, as we can see in this example with an arithmetic expression:

```groovy
def sum = "The sum of 2 and 3 equals ${2 + 3}"
assert sum.toString() == 'The sum of 2 and 3 equals 5'
```

> Not only are expressions allowed in between the `${}` placeholder, but so are statements. However, a statement’s value is just `null`. So if several statements are inserted in that placeholder, the last one should somehow return a meaningful value to be inserted. For instance, "The sum of 1 and 2 is equal to ${def a = 1; def b = 2; a + b}" is supported and works as expected but a good practice is usually to stick to simple expressions inside GString placeholders.



In addition to `${}` placeholders, we can also use a lone `$` sign prefixing a dotted expression:

```
def person = [name: 'Guillaume', age: 36]
assert "$person.name is $person.age years old" == 'Guillaume is 36 years old'
```

But only dotted expressions of the form `a.b`, `a.b.c`, etc, are valid. Expressions containing parentheses like method calls, curly braces for closures, dots which aren’t part of a property expression or arithmetic operators would be invalid. Given the following variable definition of a number:

```
def number = 3.14
```

The following statement will throw a `groovy.lang.MissingPropertyException` because Groovy believes you’re trying to access the `toString` property of that number, which doesn’t exist:

```groovy
shouldFail(MissingPropertyException) {
    println "$number.toString()"
}
```

> You can think of `"$number.toString()"` as being interpreted by the parser as `"${number.toString}()"`.



```
String thing = 'treasure'
assert 'The x-coordinate of the treasure is represented by treasure.x' ==
    "The x-coordinate of the $thing is represented by $thing.x"   // <= Not allowed: ambiguous!!
assert 'The x-coordinate of the treasure is represented by treasure.x' ==
        "The x-coordinate of the $thing is represented by ${thing}.x"  // <= Curly braces required
```

If you need to escape the `$` or `${}` placeholders in a GString so they appear as is without interpolation, you just need to use a `\` backslash character to escape the dollar sign:

```groovy
assert '$5' == "\$5"
assert '${name}' == "\${name}"
```

#### 4.4.2. Special case of interpolating closure expressions

​	So far, we’ve seen we could interpolate arbitrary expressions inside the `${}` placeholder, but there is a special case and notation for closure expressions. When the placeholder contains an arrow, `${→}`, the expression is actually a closure expression — you can think of it as a closure with a dollar prepended in front of it:

```
def sParameterLessClosure = "1 + 2 == ${-> 3}" // 1
assert sParameterLessClosure == '1 + 2 == 3'

def sOneParamClosure = "1 + 2 == ${ w -> w << 3}" // 2
assert sOneParamClosure == '1 + 2 == 3'
```

1. The closure is a parameterless closure which doesn’t take arguments.
2. Here, the closure takes a single `java.io.StringWriter` argument, to which you can append content with the `<<` leftShift operator. In either case, both placeholders are embedded closures.

​	In appearance, it looks like a more verbose way of defining expressions to be interpolated, but closures have an interesting advantage over mere expressions: lazy evaluation.

```groovy
def number = 1 
def eagerGString = "value == ${number}"
def lazyGString = "value == ${ -> number }"

assert eagerGString == "value == 1" 
assert lazyGString ==  "value == 1" 

number = 2 
assert eagerGString == "value == 1" 
assert lazyGString ==  "value == 2" 
```

#### 4.4.3. Interoperability with Java

When a method (whether implemented in Java or Groovy) expects a `java.lang.String`, but we pass a `groovy.lang.GString` instance, the `toString()` method of the GString is automatically and transparently called.

```
String takeString(String message) {         
    assert message instanceof String        
    return message
}

def message = "The message is ${'hello'}"   
assert message instanceof GString           

def result = takeString(message)            
assert result instanceof String
assert result == 'The message is hello'
```

#### 4.4.4. GString and String hashCodes

Although interpolated strings can be used in lieu of plain Java strings, they differ with strings in a particular way: their hashCodes are different. Plain Java strings are immutable, whereas the resulting String representation of a GString can vary, depending on its interpolated values. Even for the same resulting string, GStrings and Strings don’t have the same hashCode.

```
assert "one: ${1}".hashCode() != "one: 1".hashCode()
```

GString and Strings having different hashCode values, using GString as Map keys should be avoided, especially if we try to retrieve an associated value with a String instead of a GString.

```
def key = "a"
def m = ["${key}": "letter ${key}"]     // 1

assert m["a"] == null     // 2
```

### 4.5. Triple-double-quoted string

Triple-double-quoted strings behave like double-quoted strings, with the addition that they are multiline, like the triple-single-quoted strings.

```groovy
def name = 'Groovy'
def template = """
    Dear Mr ${name},

    You're the winner of the lottery!

    Yours sincerly,

    Dave
"""

assert template.toString().contains('Groovy')
```

> Neither double quotes nor single quotes need be escaped in triple-double-quoted strings.

### 4.6. Slashy string

Beyond the usual quoted strings, Groovy offers slashy strings, which use `/` as the opening and closing delimiter. Slashy strings are particularly useful for defining regular expressions and patterns, as there is no need to escape backslashes.

Example of a slashy string:

```groovy
def fooPattern = /.*foo.*/
assert fooPattern == '.*foo.*'
```

Only forward slashes need to be escaped with a backslash:

```groovy
def escapeSlash = /The character \/ is a forward slash/
assert escapeSlash == 'The character / is a forward slash'
```

Slashy strings are multiline:

```groovy
def multilineSlashy = /one
    two
    three/

assert multilineSlashy.contains('\n')
```

Slashy strings can be thought of as just another way to define a GString but with different escaping rules. They hence support interpolation:

```groovy
def color = 'blue'
def interpolatedSlashy = /a ${color} car/

assert interpolatedSlashy == 'a blue car'
```

#### 4.6.1. Special cases

​	An empty slashy string cannot be represented with a double forward slash, as it’s understood by the Groovy parser as a line comment. That’s why the following assert would actually not compile as it would look like a non-terminated statement:

```
assert '' == //
```

​	As slashy strings were mostly designed to make regexp easier so a few things that are errors in GStrings like `$()` or `$5` will work with slashy strings.

​	Remember that escaping backslashes is not required. An alternative way of thinking of this is that in fact escaping is not supported. The slashy string `/\t/` won’t contain a tab but instead a backslash followed by the character 't'. Escaping is only allowed for the slash character, i.e. `/\/folder/` will be a slashy string containing `'/folder'`. A consequence of slash escaping is that a slashy string can’t end with a backslash. Otherwise that will escape the slashy string terminator. You can instead use a special trick, `/ends with slash ${'\'}/`. But best just avoid using a slashy string in such a case.

### 4.7. Dollar slashy string

Dollar slashy strings are multiline GStrings delimited with an opening `$/` and a closing `/$`. The escaping character is the dollar sign, and it can escape another dollar, or a forward slash. Escaping for the dollar and forward slash characters is only needed where conflicts arise with the special use of those characters. The characters `$foo` would normally indicate a GString placeholder, so those four characters can be entered into a dollar slashy string by escaping the dollar, i.e. `$$foo`. Similarly, you will need to escape a dollar slashy closing delimiter if you want it to appear in your string.

Here are a few examples:

```groovy
def name = "Guillaume"
def date = "April, 1st"

def dollarSlashy = $/
    Hello $name,
    today we're ${date}.

    $ dollar sign
    $$ escaped dollar sign
    \ backslash
    / forward slash
    $/ escaped forward slash
    $$$/ escaped opening dollar slashy
    $/$$ escaped closing dollar slashy
/$

assert [
    'Guillaume',
    'April, 1st',
    '$ dollar sign',
    '$ escaped dollar sign',
    '\\ backslash',
    '/ forward slash',
    '/ escaped forward slash',
    '$/ escaped opening dollar slashy',
    '/$ escaped closing dollar slashy'
].every { dollarSlashy.contains(it) }
```

### 4.8. String summary table

| String name          | String syntax | Interpolated | Multiline | Escape character |
| -------------------- | ------------- | ------------ | --------- | ---------------- |
| Single-quoted        | `'…'`         |              |           | `\`              |
| Triple-single-quoted | `'''…'''`     |              |           | `\`              |
| Double-quoted        | `"…"`         |              |           | `\`              |
| Triple-double-quoted | `"""…"""`     |              |           | `\`              |
| Slashy               | `/…/`         |              |           | `\`              |
| Dollar slashy        | `$/…/$`       |              |           | `$`              |

### 4.9. Characters

​	Unlike Java, Groovy doesn’t have an explicit character literal. However, you can be explicit about making a Groovy string an actual character, by three different means:

```groovy
char c1 = 'A' 
assert c1 instanceof Character

def c2 = 'B' as char 
assert c2 instanceof Character

def c3 = (char)'C' 
assert c3 instanceof Character
```

## 5. Numbers

​	Groovy supports different kinds of integral literals and decimal literals, backed by the usual `Number` types of Java.

### 5.1. Integral literals

The integral literal types are the same as in Java:

* `byte`
* `char`
* `short`
* `int`
* `long`
* `java.math.BigInteger`

You can create integral numbers of those types with the following declarations:

```groovy
// primitive types
byte  b = 1
char  c = 2
short s = 3
int   i = 4
long  l = 5

// infinite precision
BigInteger bi =  6
```

If you use optional typing by using the `def` keyword, the type of the integral number will vary: it’ll adapt to the capacity of the type that can hold that number.

For positive numbers:

```gr
def a = 1
assert a instanceof Integer

// Integer.MAX_VALUE
def b = 2147483647
assert b instanceof Integer

// Integer.MAX_VALUE + 1
def c = 2147483648
assert c instanceof Long

// Long.MAX_VALUE
def d = 9223372036854775807
assert d instanceof Long

// Long.MAX_VALUE + 1
def e = 9223372036854775808
assert e instanceof BigInteger
```

As well as for negative numbers:

```groovy
def na = -1
assert na instanceof Integer

// Integer.MIN_VALUE
def nb = -2147483648
assert nb instanceof Integer

// Integer.MIN_VALUE - 1
def nc = -2147483649
assert nc instanceof Long

// Long.MIN_VALUE
def nd = -9223372036854775808
assert nd instanceof Long

// Long.MIN_VALUE - 1
def ne = -9223372036854775809
assert ne instanceof BigInteger
```

#### 5.1.1. Alternative non-base 10 representations

Numbers can also be represented in binary, octal, hexadecimal and decimal bases.

##### Binary literal

Binary numbers start with a `0b` prefix:

```groovy
int xInt = 0b10101111
assert xInt == 175

short xShort = 0b11001001
assert xShort == 201 as short

byte xByte = 0b11
assert xByte == 3 as byte

long xLong = 0b101101101101
assert xLong == 2925l

BigInteger xBigInteger = 0b111100100001
assert xBigInteger == 3873g

int xNegativeInt = -0b10101111
assert xNegativeInt == -175
```

##### Octal literal

Octal numbers are specified in the typical format of `0` followed by octal digits.

```groovy
int xInt = 077
assert xInt == 63

short xShort = 011
assert xShort == 9 as short

byte xByte = 032
assert xByte == 26 as byte

long xLong = 0246
assert xLong == 166l

BigInteger xBigInteger = 01111
assert xBigInteger == 585g

int xNegativeInt = -077
assert xNegativeInt == -63
```

##### Hexadecimal literal

Hexadecimal numbers are specified in the typical format of `0x` followed by hex digits.

```groovy
int xInt = 0x77
assert xInt == 119

short xShort = 0xaa
assert xShort == 170 as short

byte xByte = 0x3a
assert xByte == 58 as byte

long xLong = 0xffff
assert xLong == 65535l

BigInteger xBigInteger = 0xaaaa
assert xBigInteger == 43690g

Double xDouble = new Double('0x1.0p0')
assert xDouble == 1.0d

int xNegativeInt = -0x77
assert xNegativeInt == -119
```

### 5.2. Decimal literals

The decimal literal types are the same as in Java:

* `float`
* `double`
* `java.math.BigDecimal`

You can create decimal numbers of those types with the following declarations:

```groovy
// primitive types
float  f = 1.234
double d = 2.345

// infinite precision
BigDecimal bd =  3.456
```

Decimals can use exponents, with the `e` or `E` exponent letter, followed by an optional sign, and an integral number representing the exponent:

```groovy
assert 1e3  ==  1_000.0
assert 2E4  == 20_000.0
assert 3e+1 ==     30.0
assert 4E-2 ==      0.04
assert 5e-1 ==      0.5
```

Conveniently for exact decimal number calculations, Groovy chooses `java.math.BigDecimal` as its decimal number type. In addition, both `float` and `double` are supported, but require an explicit type declaration, type coercion or suffix. Even if `BigDecimal` is the default for decimal numbers, such literals are accepted in methods or closures taking `float` or `double` as parameter types.

>  Decimal numbers can’t be represented using a binary, octal or hexadecimal representation.

### 5.3. Underscore in literals

When writing long literal numbers, it’s harder on the eye to figure out how some numbers are grouped together, for example with groups of thousands, of words, etc. By allowing you to place underscore in number literals, it’s easier to spot those groups:

```groovy
long creditCardNumber = 1234_5678_9012_3456L
long socialSecurityNumbers = 999_99_9999L
double monetaryAmount = 12_345_132.12
long hexBytes = 0xFF_EC_DE_5E
long hexWords = 0xFFEC_DE5E
long maxLong = 0x7fff_ffff_ffff_ffffL
long alsoMaxLong = 9_223_372_036_854_775_807L
long bytes = 0b11010010_01101001_10010100_10010010
```

### 5.4. Number type suffixes

We can force a number (including binary, octals and hexadecimals) to have a specific type by giving a suffix (see table below), either uppercase or lowercase.

| Type       | Suffix     |
| :--------- | :--------- |
| BigInteger | `G` or `g` |
| Long       | `L` or `l` |
| Integer    | `I` or `i` |
| BigDecimal | `G` or `g` |
| Double     | `D` or `d` |
| Float      | `F` or `f` |

Examples:

```
assert 42I == Integer.valueOf('42')
assert 42i == Integer.valueOf('42') // lowercase i more readable
assert 123L == Long.valueOf("123") // uppercase L more readable
assert 2147483648 == Long.valueOf('2147483648') // Long type used, value too large for an Integer
assert 456G == new BigInteger('456')
assert 456g == new BigInteger('456')
assert 123.45 == new BigDecimal('123.45') // default BigDecimal type used
assert .321 == new BigDecimal('.321')
assert 1.200065D == Double.valueOf('1.200065')
assert 1.234F == Float.valueOf('1.234')
assert 1.23E23D == Double.valueOf('1.23E23')
assert 0b1111L.class == Long // binary
assert 0xFFi.class == Integer // hexadecimal
assert 034G.class == BigInteger // octal
```

### 5.5. Math operations

Although [operators](https://docs.groovy-lang.org/latest/html/documentation/core-operators.html#groovy-operators) are covered in more detail elsewhere, it’s important to discuss the behavior of math operations and what their resulting types are.

Division and power binary operations aside (covered below),

* binary operations between `byte`, `char`, `short` and `int` result in `int`
* binary operations involving `long` with `byte`, `char`, `short` and `int` result in `long`
* binary operations involving `BigInteger` and any other integral type result in `BigInteger`
* binary operations involving `BigDecimal` with `byte`, `char`, `short`, `int` and `BigInteger` result in `BigDecimal`
* binary operations between `float`, `double` and `BigDecimal` result in `double`
* binary operations between two `BigDecimal` result in `BigDecimal`

The following table summarizes those rules:

|                | byte | char | short | int  | long | BigInteger | float  | double | BigDecimal |
| :------------- | :--- | :--- | :---- | :--- | :--- | :--------- | :----- | :----- | :--------- |
| **byte**       | int  | int  | int   | int  | long | BigInteger | double | double | BigDecimal |
| **char**       |      | int  | int   | int  | long | BigInteger | double | double | BigDecimal |
| **short**      |      |      | int   | int  | long | BigInteger | double | double | BigDecimal |
| **int**        |      |      |       | int  | long | BigInteger | double | double | BigDecimal |
| **long**       |      |      |       |      | long | BigInteger | double | double | BigDecimal |
| **BigInteger** |      |      |       |      |      | BigInteger | double | double | BigDecimal |
| **float**      |      |      |       |      |      |            | double | double | double     |
| **double**     |      |      |       |      |      |            |        | double | double     |
| **BigDecimal** |      |      |       |      |      |            |        |        | BigDecimal |

> Thanks to Groovy’s operator overloading, the usual arithmetic operators work as well with `BigInteger` and `BigDecimal`, unlike in Java where you have to use explicit methods for operating on those numbers.

#### 5.5.1. The case of the division operator

The division operators `/` (and `/=` for division and assignment) produce a `double` result if either operand is a `float` or `double`, and a `BigDecimal` result otherwise (when both operands are any combination of an integral type `short`, `char`, `byte`, `int`, `long`, `BigInteger` or `BigDecimal`).

`BigDecimal` division is performed with the `divide()` method if the division is exact (i.e. yielding a result that can be represented within the bounds of the same precision and scale), or using a `MathContext` with a [precision](http://docs.oracle.com/javase/7/docs/api/java/math/BigDecimal.html#precision()) of the maximum of the two operands' precision plus an extra precision of 10, and a [scale](http://docs.oracle.com/javase/7/docs/api/java/math/BigDecimal.html#scale()) of the maximum of 10 and the maximum of the operands' scale.

> For integer division like in Java, you should use the `intdiv()` method, as Groovy doesn’t provide a dedicated integer division operator symbol.

#### 5.5.2. The case of the power operator

The power operation is represented by the `**` operator, with two parameters: the base and the exponent. The result of the power operation depends on its operands, and the result of the operation (in particular if the result can be represented as an integral value).

The following rules are used by Groovy’s power operation to determine the resulting type:

* If the exponent is a decimal value
  * if the result can be represented as an `Integer`, then return an `Integer`
  * else if the result can be represented as a `Long`, then return a `Long`
  * otherwise return a `Double`
* If the exponent is an integral value
  * if the exponent is strictly negative, then return an `Integer`, `Long` or `Double` if the result value fits in that type
  * if the exponent is positive or zero
    * if the base is a `BigDecimal`, then return a `BigDecimal` result value
    * if the base is a `BigInteger`, then return a `BigInteger` result value
    * if the base is an `Integer`, then return an `Integer` if the result value fits in it, otherwise a `BigInteger`
    * if the base is a `Long`, then return a `Long` if the result value fits in it, otherwise a `BigInteger`

We can illustrate those rules with a few examples:

```
// base and exponent are ints and the result can be represented by an Integer
assert    2    **   3    instanceof Integer    //  8
assert   10    **   9    instanceof Integer    //  1_000_000_000

// the base is a long, so fit the result in a Long
// (although it could have fit in an Integer)
assert    5L   **   2    instanceof Long       //  25

// the result can't be represented as an Integer or Long, so return a BigInteger
assert  100    **  10    instanceof BigInteger //  10e20
assert 1234    ** 123    instanceof BigInteger //  170515806212727042875...

// the base is a BigDecimal and the exponent a negative int
// but the result can be represented as an Integer
assert    0.5  **  -2    instanceof Integer    //  4

// the base is an int, and the exponent a negative float
// but again, the result can be represented as an Integer
assert    1    **  -0.3f instanceof Integer    //  1

// the base is an int, and the exponent a negative int
// but the result will be calculated as a Double
// (both base and exponent are actually converted to doubles)
assert   10    **  -1    instanceof Double     //  0.1

// the base is a BigDecimal, and the exponent is an int, so return a BigDecimal
assert    1.2  **  10    instanceof BigDecimal //  6.1917364224

// the base is a float or double, and the exponent is an int
// but the result can only be represented as a Double value
assert    3.4f **   5    instanceof Double     //  454.35430372146965
assert    5.6d **   2    instanceof Double     //  31.359999999999996

// the exponent is a decimal value
// and the result can only be represented as a Double value
assert    7.8  **   1.9  instanceof Double     //  49.542708423868476
assert    2    **   0.1f instanceof Double     //  1.0717734636432956
```

## 6. Booleans

Boolean is a special data type that is used to represent truth values: `true` and `false`. Use this data type for simple flags that track true/false [conditions](https://docs.groovy-lang.org/latest/html/documentation/core-operators.html#_conditional_operators).

Boolean values can be stored in variables, assigned into fields, just like any other data type:

```groovy
def myBooleanVariable = true
boolean untypedBooleanVar = false
booleanField = true
```

`true` and `false` are the only two primitive boolean values. But more complex boolean expressions can be represented using [logical operators](https://docs.groovy-lang.org/latest/html/documentation/core-operators.html#_logical_operators).

In addition, Groovy has [special rules](https://docs.groovy-lang.org/latest/html/documentation/core-semantics.html#the-groovy-truth) (often referred to as *Groovy Truth*) for coercing non-boolean objects to a boolean

## 7. Lists

Groovy uses a comma-separated list of values, surrounded by square brackets, to denote lists. Groovy lists are plain JDK `java.util.List`, as Groovy doesn’t define its own collection classes. The concrete list implementation used when defining list literals are `java.util.ArrayList` by default, unless you decide to specify otherwise, as we shall see later on.

```groovy
def numbers = [1, 2, 3]         

assert numbers instanceof List  
assert numbers.size() == 3      
```

* We define a list numbers delimited by commas and surrounded by square brackets, and we assign that list into a variable.
* The list is an instance of Java’s `java.util.List` interface.
* The size of the list can be queried with the `size()` method, and shows our list contains 3 elements.

In the above example, we used a homogeneous list, but you can also create lists containing values of heterogeneous types:

```sh
def heterogeneous = [1, "a", true]
```

1. Our list here contains a number, a string and a boolean value.

We mentioned that by default, list literals are actually instances of `java.util.ArrayList`, but it is possible to use a different backing type for our lists, thanks to using type coercion with the `as` operator, or with explicit type declaration for your variables:

```groovy
def arrayList = [1, 2, 3]
assert arrayList instanceof java.util.ArrayList

def linkedList = [2, 3, 4] as LinkedList    
assert linkedList instanceof java.util.LinkedList

LinkedList otherLinked = [3, 4, 5]          
assert otherLinked instanceof java.util.LinkedList
```

1. We use coercion with the `as` operator to explicitly request a `java.util.LinkedList` implementation.
2.  We can say that the variable holding the list literal is of type `java.util.LinkedList`.

You can access elements of the list with the `[]` subscript operator (both for reading and setting values) with positive indices or negative indices to access elements from the end of the list, as well as with ranges, and use the `<<` leftShift operator to append elements to a list:

```groovy
def letters = ['a', 'b', 'c', 'd']

assert letters[0] == 'a'     
assert letters[1] == 'b'

assert letters[-1] == 'd'    
assert letters[-2] == 'c'

letters[2] = 'C'             
assert letters[2] == 'C'

letters << 'e'               
assert letters[ 4] == 'e'
assert letters[-1] == 'e'

assert letters[1, 3] == ['b', 'd']         
assert letters[2..4] == ['C', 'd', 'e'] 
```

1. Access the first element of the list (zero-based counting)
2. Access the last element of the list with a negative index: -1 is the first element from the end of the list
3. Use an assignment to set a new value for the third element of the list
4. Use the `<<` leftShift operator to append an element at the end of the list
5. Access two elements at once, returning a new list containing those two elements
6.  Use a range to access a range of values from the list, from a start to an end element position

As lists can be heterogeneous in nature, lists can also contain other lists to create multi-dimensional lists:

```groovy
def multi = [[0, 1], [2, 3]]     
assert multi[1][0] == 2          
```

1.  Define a list of list of numbers
2. Access the second element of the top-most list, and the first element of the inner list

## 8. Arrays

Groovy reuses the list notation for arrays, but to make such literals arrays, you need to explicitely define the type of the array through coercion or type declaration.

```groovy
String[] arrStr = ['Ananas', 'Banana', 'Kiwi']  

assert arrStr instanceof String[]    
assert !(arrStr instanceof List)

def numArr = [1, 2, 3] as int[]      

assert numArr instanceof int[]       
assert numArr.size() == 3
```

1. Define an array of strings using explicit variable type declaration
2.  Assert that we created an array of strings
3. Create an array of ints with the `as` operator
4. Assert that we created an array of primitive ints

You can also create multi-dimensional arrays:

```groovy
def matrix3 = new Integer[3][3]         
assert matrix3.size() == 3

Integer[][] matrix2                     
matrix2 = [[1, 2], [3, 4]]
assert matrix2 instanceof Integer[][]
```

1. You can define the bounds of a new array
2. Or declare an array without specifying its bounds

Access to elements of an array follows the same notation as for lists:

```
String[] names = ['Cédric', 'Guillaume', 'Jochen', 'Paul']
assert names[0] == 'Cédric'     

names[2] = 'Blackdrag'          
assert names[2] == 'Blackdrag'
```

1. Retrieve the first element of the array
2. Set the value of the third element of the array to a new value

### 8.1. Java-style array initialization

Groovy has always supported literal list/array definitions using square brackets and has avoided Java-style curly braces so as not to conflict with closure definitions. In the case where the curly braces come immediately after an array type declaration however, there is no ambiguity with closure definitions, so Groovy 3 and above support that variant of the Java array initialization expression.

Examples:

```
def primes = new int[] {2, 3, 5, 7, 11}
assert primes.size() == 5 && primes.sum() == 28
assert primes.class.name == '[I'

def pets = new String[] {'cat', 'dog'}
assert pets.size() == 2 && pets.sum() == 'catdog'
assert pets.class.name == '[Ljava.lang.String;'

// traditional Groovy alternative still supported
String[] groovyBooks = [ 'Groovy in Action', 'Making Java Groovy' ]
assert groovyBooks.every{ it.contains('Groovy') }
```

## 9. Maps

Sometimes called dictionaries or associative arrays in other languages, Groovy features maps. Maps associate keys to values, separating keys and values with colons, and each key/value pairs with commas, and the whole keys and values surrounded by square brackets.

```
def colors = [red: '#FF0000', green: '#00FF00', blue: '#0000FF']   

assert colors['red'] == '#FF0000'    
assert colors.green  == '#00FF00'    

colors['pink'] = '#FF00FF'           
colors.yellow  = '#FFFF00'           

assert colors.pink == '#FF00FF'
assert colors['yellow'] == '#FFFF00'

assert colors instanceof java.util.LinkedHashMap
```

1. We define a map of string color names, associated with their hexadecimal-coded html colors
2. We use the subscript notation to check the content associated with the `red` key
3.  We can also use the property notation to assert the color green’s hexadecimal representation
4. Similarly, we can use the subscript notation to add a new key/value pair
5.  Or the property notation, to add the `yellow` color

> When using names for the keys, we actually define string keys in the map.
>
> Groovy creates maps that are actually instances of `java.util.LinkedHashMap`.

If you try to access a key which is not present in the map:

```groovy
assert colors.unknown == null

def emptyMap = [:]
assert emptyMap.anyKey == null
```

You will retrieve a `null` result.

In the examples above, we used string keys, but you can also use values of other types as keys:

```groovy
def numbers = [1: 'one', 2: 'two']

assert numbers[1] == 'one'
```

Here, we used numbers as keys, as numbers can unambiguously be recognized as numbers, so Groovy will not create a string key like in our previous examples. But consider the case you want to pass a variable in lieu of the key, to have the value of that variable become the key:

```groovy
def key = 'name'
def person = [key: 'Guillaume']      

assert !person.containsKey('name')   
assert person.containsKey('key')   
```

1. The `key` associated with the `'Guillaume'` name will actually be the `"key"` string, not the value associated with the `key` variable
2. The map doesn’t contain the `'name'` key
3.  Instead, the map contains a `'key'` key

> You can also pass quoted strings as well as keys: ["name": "Guillaume"]. This is mandatory if your key string isn’t a valid identifier, for example if you wanted to create a string key containing a dash like in: ["street-name": "Main street"].

When you need to pass variable values as keys in your map definitions, you must surround the variable or expression with parentheses:

```groovy
person = [(key): 'Guillaume']        

assert person.containsKey('name')    
assert !person.containsKey('key')    
```

1. This time, we surround the `key` variable with parentheses, to instruct the parser we are passing a variable rather than defining a string key
2. The map does contain the `name` key
3. But the map doesn’t contain the `key` key as before

# Operators

## 1. Arithmetic operators

### 1.1. Normal arithmetic operators

The following binary arithmetic operators are available in Groovy:

| Operator | Purpose        | Remarks                                                      |
| :------- | :------------- | :----------------------------------------------------------- |
| `+`      | addition       |                                                              |
| `-`      | subtraction    |                                                              |
| `*`      | multiplication |                                                              |
| `/`      | division       | Use `intdiv()` for integer division, and see the section about [integer division](https://docs.groovy-lang.org/latest/html/documentation/core-syntax.html#integer_division) for more information on the return type of the division. |
| `%`      | remainder      |                                                              |
| `**`     | power          | See the section about [the power operation](https://docs.groovy-lang.org/latest/html/documentation/core-syntax.html#power_operator) for more information on the return type of the operation. |

Here are a few examples of usage of those operators:

```groovy
assert  1  + 2 == 3
assert  4  - 3 == 1
assert  3  * 5 == 15
assert  3  / 2 == 1.5
assert 10  % 3 == 1
assert  2 ** 3 == 8
```

### 1.2. Unary operators

The `+` and `-` operators are also available as unary operators:

```
assert +3 == 3
assert -4 == 0 - 4

assert -(-1) == 1  
```



In terms of unary arithmetics operators, the `++` (increment) and `--` (decrement) operators are available, both in prefix and postfix notation:

```groovy
def a = 2
def b = a++ * 3             

assert a == 3 && b == 6

def c = 3
def d = c-- * 2             

assert c == 2 && d == 6

def e = 1
def f = ++e + 3             

assert e == 2 && f == 5

def g = 4
def h = --g + 1             

assert g == 3 && h == 4
```

### 1.3. Assignment arithmetic operators

The binary arithmetic operators we have seen above are also available in an assignment form:

* `+=`
* `-=`
* `*=`
* `/=`
* `%=`
* `**=`

Let’s see them in action:

```groovy
def a = 4
a += 3

assert a == 7

def b = 5
b -= 3

assert b == 2

def c = 5
c *= 3

assert c == 15

def d = 10
d /= 2

assert d == 5

def e = 10
e %= 3

assert e == 1

def f = 3
f **= 2

assert f == 9
```

## 2. Relational operators

Relational operators allow comparisons between objects, to know if two objects are the same or different, or if one is greater than, less than, or equal to the other.

The following operators are available:

| Operator | Purpose                            |
| :------- | :--------------------------------- |
| `==`     | equal                              |
| `!=`     | different                          |
| `<`      | less than                          |
| `<=`     | less than or equal                 |
| `>`      | greater than                       |
| `>=`     | greater than or equal              |
| `===`    | identical (Since Groovy 3.0.0)     |
| `!==`    | not identical (Since Groovy 3.0.0) |

Here are some examples of simple number comparisons using these operators:

```
assert 1 + 2 == 3
assert 3 != 4

assert -2 < 3
assert 2 <= 2
assert 3 <= 4

assert 5 > 1
assert 5 >= -2
```

Both `===` and `!==` are supported which are the same as calling the `is()` method, and negating a call to the `is()` method respectively.

```
import groovy.transform.EqualsAndHashCode

@EqualsAndHashCode
class Creature { String type }

def cat = new Creature(type: 'cat')
def copyCat = cat
def lion = new Creature(type: 'cat')

assert cat.equals(lion) // Java logical equality
assert cat == lion      // Groovy shorthand operator

assert cat.is(copyCat)  // Groovy identity
assert cat === copyCat  // operator shorthand
assert cat !== lion     // negated operator shorthand
```

## 3. Logical operators

Groovy offers three logical operators for boolean expressions:

* `&&`: logical "and"
* `||`: logical "or"
* `!`: logical "not"

Let’s illustrate them with the following examples:

```
assert !false           
assert true && true     
assert true || false    
```

### 3.1. Precedence

The logical "not" has a higher priority than the logical "and".

```
assert (!false && false) == false   
```

The logical "and" has a higher priority than the logical "or".

```
assert true || true && false        
```

### 3.2. Short-circuiting

The logical `||` operator supports short-circuiting: if the left operand is true, it knows that the result will be true in any case, so it won’t evaluate the right operand. The right operand will be evaluated only if the left operand is false.

Likewise for the logical `&&` operator: if the left operand is false, it knows that the result will be false in any case, so it won’t evaluate the right operand. The right operand will be evaluated only if the left operand is true.

```
boolean checkIfCalled() {   
    called = true
}

called = false
true || checkIfCalled()
assert !called              

called = false
false || checkIfCalled()
assert called               

called = false
false && checkIfCalled()
assert !called              

called = false
true && checkIfCalled()
assert called               
```

## 4. Bitwise and bit shift operators

### 4.1. Bitwise operators

Groovy offers four bitwise operators:

* `&`: bitwise "and"
* `|`: bitwise "or"
* `^`: bitwise "xor" (exclusive "or")
* `~`: bitwise negation

Bitwise operators can be applied on arguments which are of type `byte`, `short`, `int`, `long`, or `BigInteger`. If one of the arguments is a `BigInteger`, the result will be of type `BigInteger`; otherwise, if one of the arguments is a `long`, the result will be of type `long`; otherwise, the result will be of type `int`:

```
int a = 0b00101010
assert a == 42
int b = 0b00001000
assert b == 8
assert (a & a) == a                     
assert (a & b) == b                     
assert (a | a) == a                     
assert (a | b) == a                     

int mask = 0b11111111                   
assert ((a ^ a) & mask) == 0b00000000   
assert ((a ^ b) & mask) == 0b00100010   
assert ((~a) & mask)    == 0b11010101   
```

### 4.2. Bit shift operators

Groovy offers three bit shift operators:

* `<<`: left shift
* `>>`: right shift
* `>>>`: right shift unsigned

All three operators are applicable where the left argument is of type `byte`, `short`, `int`, or `long`. The first two operators can also be applied where the left argument is of type `BigInteger`. If the left argument is a `BigInteger`, the result will be of type `BigInteger`; otherwise, if the left argument is a `long`, the result will be of type `long`; otherwise, the result will be of type `int`:

```groovy
assert 12.equals(3 << 2)           
assert 24L.equals(3L << 3)         
assert 48G.equals(3G << 4)         

assert 4095 == -200 >>> 20
assert -1 == -200 >> 20
assert 2G == 5G >> 1
assert -3G == -5G >> 1
```

## 5. Conditional operators

### 5.1. Not operator

The "not" operator is represented with an exclamation mark (`!`) and inverts the result of the underlying boolean expression. In particular, it is possible to combine the `not` operator with the [Groovy truth](https://docs.groovy-lang.org/latest/html/documentation/core-semantics.html#the-groovy-truth):

```groovy
assert (!true)    == false                      
assert (!'foo')   == false                      
assert (!'')      == true                       
```

### 5.2. Ternary operator

The ternary operator is a shortcut expression that is equivalent to an if/else branch assigning some value to a variable.

Instead of:

```groovy
if (string!=null && string.length()>0) {
    result = 'Found'
} else {
    result = 'Not found'
}
```

You can write:

```groovy
result = (string!=null && string.length()>0) ? 'Found' : 'Not found'
```

The ternary operator is also compatible with the [Groovy truth](https://docs.groovy-lang.org/latest/html/documentation/core-semantics.html#the-groovy-truth), so you can make it even simpler:

```groovy
result = string ? 'Found' : 'Not found'
```

### 5.3. Elvis operator

The "Elvis operator" is a shortening of the ternary operator. One instance of where this is handy is for returning a 'sensible default' value if an expression resolves to `false`-ish (as in [Groovy truth](https://docs.groovy-lang.org/latest/html/documentation/core-semantics.html#the-groovy-truth)). A simple example might look like this:

```
displayName = user.name ? user.name : 'Anonymous'   
displayName = user.name ?: 'Anonymous'       
```

### 5.4. Elvis assignment operator

Groovy 3.0.0 introduces the Elvis operator, for example:

```
import groovy.transform.ToString

@ToString
class Element {
    String name
    int atomicNumber
}

def he = new Element(name: 'Helium')
he.with {
    name = name ?: 'Hydrogen'   // existing Elvis operator
    atomicNumber ?= 2           // new Elvis assignment shorthand
}
assert he.toString() == 'Element(Helium, 2)'
```

## 6. Object operators

### 6.1. Safe navigation operator

The Safe Navigation operator is used to avoid a `NullPointerException`. Typically when you have a reference to an object you might need to verify that it is not `null` before accessing methods or properties of the object. To avoid this, the safe navigation operator will simply return `null` instead of throwing an exception, like so:

```groovy
def person = Person.find { it.id == 123 }    
def name = person?.name                      
assert name == null       
```

### 6.2. Direct field access operator

Normally in Groovy, when you write code like this:

```groovy
class User {
    public final String name                 
    User(String name) { this.name = name}
    String getName() { "Name: $name" }       
}
def user = new User('Bob')
assert user.name == 'Name: Bob'              
```

The `user.name` call triggers a call to the property of the same name, that is to say, here, to the getter for `name`. If you want to retrieve the field instead of calling the getter, you can use the direct field access operator:

```
assert user.@name == 'Bob' 
```

### 6.3. Method pointer operator

The method pointer operator (`.&`) can be used to store a reference to a method in a variable, in order to call it later:

```
def str = 'example of method reference'            
def fun = str.&toUpperCase                         
def upper = fun()                                  
assert upper == str.toUpperCase()                  
```

There are multiple advantages in using method pointers. First of all, the type of such a method pointer is a `groovy.lang.Closure`, so it can be used in any place a closure would be used. In particular, it is suitable to convert an existing method for the needs of the strategy pattern:

```groovy
def transform(List elements, Closure action) {                    
    def result = []
    elements.each {
        result << action(it)
    }
    result
}
String describe(Person p) {                                       
    "$p.name is $p.age"
}
def action = this.&describe                                       
def list = [
    new Person(name: 'Bob',   age: 42),
    new Person(name: 'Julia', age: 35)]                           
assert transform(list, action) == ['Bob is 42', 'Julia is 35']    
```

Method pointers are bound by the receiver and a method name. Arguments are resolved at runtime, meaning that if you have multiple methods with the same name, the syntax is not different, only resolution of the appropriate method to be called will be done at runtime:

```groovy
def doSomething(String str) { str.toUpperCase() }    
def doSomething(Integer x) { 2*x }                   
def reference = this.&doSomething                    
assert reference('foo') == 'FOO'                     
assert reference(123)   == 246                       
```

To align with Java 8 method reference expectations, in Groovy 3 and above, you can use `new` as the method name to obtain a method pointer to the constructor:

```groovy
def foo  = BigInteger.&new
def fortyTwo = foo('42')
assert fortyTwo == 42G
```

Also in Groovy 3 and above, you can obtain a method pointer to an instance method of a class. This method pointer takes an additional parameter being the receiver instance to invoke the method on:

```groovy
def instanceMethod = String.&toUpperCase
assert instanceMethod('foo') == 'FOO'
```

### 6.4. Method reference operator

The Parrot parser in Groovy 3+ supports the Java 8+ method reference operator. The method reference operator (`::`) can be used to reference a method or constructor in contexts expecting a functional interface. This overlaps somewhat with the functionality provided by Groovy’s method pointer operator. Indeed, for dynamic Groovy, the method reference operator is just an alias for the method pointer operator. For static Groovy, the operator results in bytecode similar to the bytecode that Java would produce for the same context.

Some examples highlighting various supported method reference cases are shown in the following script:

```groovy
import groovy.transform.CompileStatic
import static java.util.stream.Collectors.toList

@CompileStatic
void methodRefs() {
    assert 6G == [1G, 2G, 3G].stream().reduce(0G, BigInteger::add)                           

    assert [4G, 5G, 6G] == [1G, 2G, 3G].stream().map(3G::add).collect(toList())              

    assert [1G, 2G, 3G] == [1L, 2L, 3L].stream().map(BigInteger::valueOf).collect(toList())  

    assert [1G, 2G, 3G] == [1L, 2L, 3L].stream().map(3G::valueOf).collect(toList())          
}

methodRefs()
```

Some examples highlighting various supported constructor reference cases are shown in the following script:

```groovy
@CompileStatic
void constructorRefs() {
    assert [1, 2, 3] == ['1', '2', '3'].stream().map(Integer::new).collect(toList())  

    def result = [1, 2, 3].stream().toArray(Integer[]::new)                           
    assert result instanceof Integer[]
    assert result.toString() == '[1, 2, 3]'
}

constructorRefs()
```

## 7. Regular expression operators

### 7.1. Pattern operator

The pattern operator (`~`) provides a simple way to create a `java.util.regex.Pattern` instance:

```
def p = ~/foo/
assert p instanceof Pattern
```

while in general, you find the pattern operator with an expression in a slashy-string, it can be used with any kind of `String` in Groovy:

```groovy
p = ~'foo'                                                        
p = ~"foo"                                                        
p = ~$/dollar/slashy $ string/$                                   
p = ~"${pattern}"                                             
```

### 7.2. Find operator

Alternatively to building a pattern, you can use the find operator `=~` to directly create a `java.util.regex.Matcher` instance:

```
def text = "some text to match"
def m = text =~ /match/                                           
assert m instanceof Matcher                                       
if (!m) {                                                         
    throw new RuntimeException("Oops, text not found!")
}
```

Since a `Matcher` coerces to a `boolean` by calling its `find` method, the `=~` operator is consistent with the simple use of Perl’s `=~` operator, when it appears as a predicate (in `if`, `?:`, etc.). When the intent is to iterate over matches of the specified pattern (in `while`, etc.) call `find()` directly on the matcher or use the `iterator` DGM.

### 7.3. Match operator

The match operator (`==~`) is a slight variation of the find operator, that does not return a `Matcher` but a boolean and requires a strict match of the input string:

```
m = text ==~ /match/                                              
assert m instanceof Boolean                                       
if (m) {                                                          
    throw new RuntimeException("Should not reach that point!")
```

### 7.4. Comparing Find vs Match operators

Typically, the match operator is used when the pattern involves a single exact match, otherwise the find operator might be more useful.

```
assert 'two words' ==~ /\S+\s+\S+/
assert 'two words' ==~ /^\S+\s+\S+$/         
assert !(' leading space' ==~ /\S+\s+\S+/)   

def m1 = 'two words' =~ /^\S+\s+\S+$/
assert m1.size() == 1                          
def m2 = 'now three words' =~ /^\S+\s+\S+$/    
assert m2.size() == 0                          
def m3 = 'now three words' =~ /\S+\s+\S+/
assert m3.size() == 1                          
assert m3[0] == 'now three'
def m4 = ' leading space' =~ /\S+\s+\S+/
assert m4.size() == 1                          
assert m4[0] == 'leading space'
def m5 = 'and with four words' =~ /\S+\s+\S+/
assert m5.size() == 2                          
assert m5[0] == 'and with'
assert m5[1] == 'four words'
```



## [10. Operator overloading](http://www.groovy-lang.org/operators.html#Operator-Overloading)

Groovy allows you to overload the various operators so that they can be used with your own classes. Consider this simple class:

```
class Bucket {
    int size

    Bucket(int size) { this.size = size }

    Bucket plus(Bucket other) {                     
        return new Bucket(this.size + other.size)
    }
}
```

|      | `Bucket` implements a special method called `plus()` |
| ---- | ---------------------------------------------------- |
|      |                                                      |

Just by implementing the `plus()` method, the `Bucket` class can now be used with the `+` operator like so:

```
def b1 = new Bucket(4)
def b2 = new Bucket(11)
assert (b1 + b2).size == 15                         
```

>  The two `Bucket` objects can be added together with the `+` operator

All (non-comparator) Groovy operators have a corresponding method that you can implement in your own classes. The only requirements are that your method is public, has the correct name, and has the correct number of arguments. The argument types depend on what types you want to support on the right hand side of the operator. For example, you could support the statement

```
assert (b1 + 11).size == 15
```

by implementing the `plus()` method with this signature:

```
Bucket plus(int capacity) {
    return new Bucket(this.size + capacity)
}
```

Here is a complete list of the operators and their corresponding methods:

| Operator | Method        | Operator   | Method                  |
| :------- | :------------ | :--------- | :---------------------- |
| `+`      | a.plus(b)     | `a[b]`     | a.getAt(b)              |
| `-`      | a.minus(b)    | `a[b] = c` | a.putAt(b, c)           |
| `*`      | a.multiply(b) | `a in b`   | b.isCase(a)             |
| `/`      | a.div(b)      | `<<`       | a.leftShift(b)          |
| `%`      | a.mod(b)      | `>>`       | a.rightShift(b)         |
| `**`     | a.power(b)    | `>>>`      | a.rightShiftUnsigned(b) |
| `|`      | a.or(b)       | `++`       | a.next()                |
| `&`      | a.and(b)      | `--`       | a.previous()            |
| `^`      | a.xor(b)      | `+a`       | a.positive()            |
| `as`     | a.asType(b)   | `-a`       | a.negative()            |
| `a()`    | a.call()      | `~a`       | a.bitwiseNegate()       |