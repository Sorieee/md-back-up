---
title: String源码阅读
date: 2020-03-30 20:51:45
tags: java,java源代码,String
---

[TOC]



jdk8

# 类定义

实现了java.io.Serializable, Comparable<String>, CharSequence

# 成员变量

value:存储字符串

hash: 缓存hashCode避免重复计算

serialPersistentFields:  待研究

https://docs.oracle.com/en/java/javase/14/docs/specs/serialization/protocol.html#stream-elements

```java
	/** The value is used for character storage. */   
	private final char value[];

    /** Cache the hash code for the string */
    private int hash; // Default to 0

    /** use serialVersionUID from JDK 1.0.2 for interoperability */
    private static final long serialVersionUID = -6849794470754667710L;

    /**
     * Class String is special cased within the Serialization Stream Protocol.
     *
     * A String instance is written into an ObjectOutputStream according to
     * <a href="{@docRoot}/../platform/serialization/spec/output.html">
     * Object Serialization Specification, Section 6.2, "Stream Elements"</a>
     */
    private static final ObjectStreamField[] serialPersistentFields =
        new ObjectStreamField[0];
```

# 构造方法

大部分略，个人认为值得关注的一个

```
 /*
    * Package private constructor which shares value array for speed.
    * this constructor is always expected to be called with share==true.
    * a separate constructor is needed because we already have a public
    * String(char[]) constructor that makes a copy of the given char[].
    */
    String(char[] value, boolean share) {
        // assert share : "unshared not supported";
        this.value = value;
    }
```

有几点值得关注:

* 访问修饰符为default
* 直接赋值
* 参数share只为区分其他构造方法，并不支持unshare。

这个的意义是为了节约内存，String中用到该方法的有三处:

* replace(char, char)
* concat(String)
* valueof(char)

其中的value全是在这三个方法定义的，没有外界引用，所以可以直接赋予给this.value

# 重要成员方法

## Equals

主要有地址比较，类型比较，长度比较以提高效率，我们在复写一些Equals方法时，可以有所启发。

```java
 /**
     * Compares this string to the specified object.  The result is {@code
     * true} if and only if the argument is not {@code null} and is a {@code
     * String} object that represents the same sequence of characters as this
     * object.
     *
     * @param  anObject
     *         The object to compare this {@code String} against
     *
     * @return  {@code true} if the given object represents a {@code String}
     *          equivalent to this string, {@code false} otherwise
     *
     * @see  #compareTo(String)
     * @see  #equalsIgnoreCase(String)
     */
    public boolean equals(Object anObject) {
        //地址相同，代表相同直接返回true
        if (this == anObject) {
            return true;
        }
        // 如果不是String类型的数据，就直接返回false，
        if (anObject instanceof String) {
            String anotherString = (String)anObject;
            int n = value.length;
            //长度一样才有可能相等
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                //逐个比较
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
```

## contentEquals(CharSequence)

如果是StringBuffer，使用一个synchronized修饰，猜想是为了避免在比较时有地方修改了它。

```java
 /**
     * Compares this string to the specified {@code CharSequence}.  The
     * result is {@code true} if and only if this {@code String} represents the
     * same sequence of char values as the specified sequence. Note that if the
     * {@code CharSequence} is a {@code StringBuffer} then the method
     * synchronizes on it.
     *
     * @param  cs
     *         The sequence to compare this {@code String} against
     *
     * @return  {@code true} if this {@code String} represents the same
     *          sequence of char values as the specified sequence, {@code
     *          false} otherwise
     *
     * @since  1.5
     */
    public boolean contentEquals(CharSequence cs) {
        // Argument is a StringBuffer, StringBuilder
        if (cs instanceof AbstractStringBuilder) {
            if (cs instanceof StringBuffer) {
                synchronized(cs) {
                   return nonSyncContentEquals((AbstractStringBuilder)cs);
                }
            } else {
                return nonSyncContentEquals((AbstractStringBuilder)cs);
            }
        }
        // Argument is a String
        if (cs instanceof String) {
            return equals(cs);
        }
        // Argument is a generic CharSequence
        char v1[] = value;
        int n = v1.length;
        if (n != cs.length()) {
            return false;
        }
        for (int i = 0; i < n; i++) {
            if (v1[i] != cs.charAt(i)) {
                return false;
            }
        }
        return true;
    }
```

## compareTo

如果短字符串是长字符串开始部分，那么返回this.value.length - anotherString.value.length,

否则返回第一个不一样的char c1 - c2

```java
/**
 * Compares two strings lexicographically.
 * The comparison is based on the Unicode value of each character in
 * the strings. The character sequence represented by this
 * {@code String} object is compared lexicographically to the
 * character sequence represented by the argument string. The result is
 * a negative integer if this {@code String} object
 * lexicographically precedes the argument string. The result is a
 * positive integer if this {@code String} object lexicographically
 * follows the argument string. The result is zero if the strings
 * are equal; {@code compareTo} returns {@code 0} exactly when
 * the {@link #equals(Object)} method would return {@code true}.
 * <p>
 * This is the definition of lexicographic ordering. If two strings are
 * different, then either they have different characters at some index
 * that is a valid index for both strings, or their lengths are different,
 * or both. If they have different characters at one or more index
 * positions, let <i>k</i> be the smallest such index; then the string
 * whose character at position <i>k</i> has the smaller value, as
 * determined by using the &lt; operator, lexicographically precedes the
 * other string. In this case, {@code compareTo} returns the
 * difference of the two character values at position {@code k} in
 * the two string -- that is, the value:
 * <blockquote><pre>
 * this.charAt(k)-anotherString.charAt(k)
 * </pre></blockquote>
 * If there is no index position at which they differ, then the shorter
 * string lexicographically precedes the longer string. In this case,
 * {@code compareTo} returns the difference of the lengths of the
 * strings -- that is, the value:
 * <blockquote><pre>
 * this.length()-anotherString.length()
 * </pre></blockquote>
 *
 * @param   anotherString   the {@code String} to be compared.
 * @return  the value {@code 0} if the argument string is equal to
 *          this string; a value less than {@code 0} if this string
 *          is lexicographically less than the string argument; and a
 *          value greater than {@code 0} if this string is
 *          lexicographically greater than the string argument.
 */
public int compareTo(String anotherString) {
    int len1 = value.length;
    int len2 = anotherString.value.length;
    int lim = Math.min(len1, len2);
    char v1[] = value;
    char v2[] = anotherString.value;

    int k = 0;
    while (k < lim) {
        char c1 = v1[k];
        char c2 = v2[k];
        if (c1 != c2) {
            return c1 - c2;
        }
        k++;
    }
    return len1 - len2;
}
```

## equalsIgnoreCase

值得注意的是regionMatches中，如果忽略大小写，先比较大写，再比较去比较小写(Georgian alphabet转换规则有点问题)

```java
  /**
     * Compares this {@code String} to another {@code String}, ignoring case
     * considerations.  Two strings are considered equal ignoring case if they
     * are of the same length and corresponding characters in the two strings
     * are equal ignoring case.
     *
     * <p> Two characters {@code c1} and {@code c2} are considered the same
     * ignoring case if at least one of the following is true:
     * <ul>
     *   <li> The two characters are the same (as compared by the
     *        {@code ==} operator)
     *   <li> Applying the method {@link
     *        java.lang.Character#toUpperCase(char)} to each character
     *        produces the same result
     *   <li> Applying the method {@link
     *        java.lang.Character#toLowerCase(char)} to each character
     *        produces the same result
     * </ul>
     *
     * @param  anotherString
     *         The {@code String} to compare this {@code String} against
     *
     * @return  {@code true} if the argument is not {@code null} and it
     *          represents an equivalent {@code String} ignoring case; {@code
     *          false} otherwise
     *
     * @see  #equals(Object)
     */
    public boolean equalsIgnoreCase(String anotherString) {
        return (this == anotherString) ? true
                : (anotherString != null)
                && (anotherString.value.length == value.length)
                && regionMatches(true, 0, anotherString, 0, value.length);
    }
    
      /**
     * Tests if two string regions are equal.
     * <p>
     * A substring of this {@code String} object is compared to a substring
     * of the argument {@code other}. The result is {@code true} if these
     * substrings represent character sequences that are the same, ignoring
     * case if and only if {@code ignoreCase} is true. The substring of
     * this {@code String} object to be compared begins at index
     * {@code toffset} and has length {@code len}. The substring of
     * {@code other} to be compared begins at index {@code ooffset} and
     * has length {@code len}. The result is {@code false} if and only if
     * at least one of the following is true:
     * <ul><li>{@code toffset} is negative.
     * <li>{@code ooffset} is negative.
     * <li>{@code toffset+len} is greater than the length of this
     * {@code String} object.
     * <li>{@code ooffset+len} is greater than the length of the other
     * argument.
     * <li>{@code ignoreCase} is {@code false} and there is some nonnegative
     * integer <i>k</i> less than {@code len} such that:
     * <blockquote><pre>
     * this.charAt(toffset+k) != other.charAt(ooffset+k)
     * </pre></blockquote>
     * <li>{@code ignoreCase} is {@code true} and there is some nonnegative
     * integer <i>k</i> less than {@code len} such that:
     * <blockquote><pre>
     * Character.toLowerCase(this.charAt(toffset+k)) !=
     Character.toLowerCase(other.charAt(ooffset+k))
     * </pre></blockquote>
     * and:
     * <blockquote><pre>
     * Character.toUpperCase(this.charAt(toffset+k)) !=
     *         Character.toUpperCase(other.charAt(ooffset+k))
     * </pre></blockquote>
     * </ul>
     *
     * @param   ignoreCase   if {@code true}, ignore case when comparing
     *                       characters.
     * @param   toffset      the starting offset of the subregion in this
     *                       string.
     * @param   other        the string argument.
     * @param   ooffset      the starting offset of the subregion in the string
     *                       argument.
     * @param   len          the number of characters to compare.
     * @return  {@code true} if the specified subregion of this string
     *          matches the specified subregion of the string argument;
     *          {@code false} otherwise. Whether the matching is exact
     *          or case insensitive depends on the {@code ignoreCase}
     *          argument.
     */
    public boolean regionMatches(boolean ignoreCase, int toffset,
            String other, int ooffset, int len) {
        char ta[] = value;
        int to = toffset;
        char pa[] = other.value;
        int po = ooffset;
        // Note: toffset, ooffset, or len might be near -1>>>1.
        if ((ooffset < 0) || (toffset < 0)
                || (toffset > (long)value.length - len)
                || (ooffset > (long)other.value.length - len)) {
            return false;
        }
        while (len-- > 0) {
            char c1 = ta[to++];
            char c2 = pa[po++];
            if (c1 == c2) {
                continue;
            }
            if (ignoreCase) {
                // If characters don't match but case may be ignored,
                // try converting both characters to uppercase.
                // If the results match, then the comparison scan should
                // continue.
                char u1 = Character.toUpperCase(c1);
                char u2 = Character.toUpperCase(c2);
                if (u1 == u2) {
                    continue;
                }
                // Unfortunately, conversion to uppercase does not work properly
                // for the Georgian alphabet, which has strange rules about case
                // conversion.  So we need to make one last check before
                // exiting.
                if (Character.toLowerCase(u1) == Character.toLowerCase(u2)) {
                    continue;
                }
            }
            return false;
        }
        return true;
    }

```



## startsWith&endWith

其中endWith依赖starsWith的实现，直接从value.length - suffix.value.length开始比较，避免代码重复。

```java
/**
 * Tests if the substring of this string beginning at the
 * specified index starts with the specified prefix.
 *
 * @param   prefix    the prefix.
 * @param   toffset   where to begin looking in this string.
 * @return  {@code true} if the character sequence represented by the
 *          argument is a prefix of the substring of this object starting
 *          at index {@code toffset}; {@code false} otherwise.
 *          The result is {@code false} if {@code toffset} is
 *          negative or greater than the length of this
 *          {@code String} object; otherwise the result is the same
 *          as the result of the expression
 *          <pre>
 *          this.substring(toffset).startsWith(prefix)
 *          </pre>
 */
public boolean startsWith(String prefix, int toffset) {
    char ta[] = value;
    int to = toffset;
    char pa[] = prefix.value;
    int po = 0;
    int pc = prefix.value.length;
    // Note: toffset might be near -1>>>1.
    if ((toffset < 0) || (toffset > value.length - pc)) {
        return false;
    }
    while (--pc >= 0) {
        if (ta[to++] != pa[po++]) {
            return false;
        }
    }
    return true;
}

/**
 * Tests if this string starts with the specified prefix.
 *
 * @param   prefix   the prefix.
 * @return  {@code true} if the character sequence represented by the
 *          argument is a prefix of the character sequence represented by
 *          this string; {@code false} otherwise.
 *          Note also that {@code true} will be returned if the
 *          argument is an empty string or is equal to this
 *          {@code String} object as determined by the
 *          {@link #equals(Object)} method.
 * @since   1. 0
 */
public boolean startsWith(String prefix) {
    return startsWith(prefix, 0);
}

/**
 * Tests if this string ends with the specified suffix.
 *
 * @param   suffix   the suffix.
 * @return  {@code true} if the character sequence represented by the
 *          argument is a suffix of the character sequence represented by
 *          this object; {@code false} otherwise. Note that the
 *          result will be {@code true} if the argument is the
 *          empty string or is equal to this {@code String} object
 *          as determined by the {@link #equals(Object)} method.
 */
public boolean endsWith(String suffix) {
    return startsWith(suffix, value.length - suffix.value.length);
}
```





## hashCode	

参考文献: https://segmentfault.com/a/1190000010799123

```
/**
 * Returns a hash code for this string. The hash code for a
 * {@code String} object is computed as
 * <blockquote><pre>
 * s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]
 * </pre></blockquote>
 * using {@code int} arithmetic, where {@code s[i]} is the
 * <i>i</i>th character of the string, {@code n} is the length of
 * the string, and {@code ^} indicates exponentiation.
 * (The hash value of the empty string is zero.)
 *
 * @return  a hash code value for this object.
 */
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;

        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```

## indexOf

### indexOf(int ch, int fromIndex)

值得注意的地方是:

当 ch >= Character.MIN_SUPPLEMENTARY_CODE_POINT是，使用的indexOfSupplementary方法

原因是为了应对UTF-16的情况

**//todo 这里还需要下来深入研究 **

参考 https://www.zhihu.com/question/42176549的回答



```java
/**
 * Returns the index within this string of the first occurrence of the
 * specified character, starting the search at the specified index.
 * <p>
 * If a character with value {@code ch} occurs in the
 * character sequence represented by this {@code String}
 * object at an index no smaller than {@code fromIndex}, then
 * the index of the first such occurrence is returned. For values
 * of {@code ch} in the range from 0 to 0xFFFF (inclusive),
 * this is the smallest value <i>k</i> such that:
 * <blockquote><pre>
 * (this.charAt(<i>k</i>) == ch) {@code &&} (<i>k</i> &gt;= fromIndex)
 * </pre></blockquote>
 * is true. For other values of {@code ch}, it is the
 * smallest value <i>k</i> such that:
 * <blockquote><pre>
 * (this.codePointAt(<i>k</i>) == ch) {@code &&} (<i>k</i> &gt;= fromIndex)
 * </pre></blockquote>
 * is true. In either case, if no such character occurs in this
 * string at or after position {@code fromIndex}, then
 * {@code -1} is returned.
 *
 * <p>
 * There is no restriction on the value of {@code fromIndex}. If it
 * is negative, it has the same effect as if it were zero: this entire
 * string may be searched. If it is greater than the length of this
 * string, it has the same effect as if it were equal to the length of
 * this string: {@code -1} is returned.
 *
 * <p>All indices are specified in {@code char} values
 * (Unicode code units).
 *
 * @param   ch          a character (Unicode code point).
 * @param   fromIndex   the index to start the search from.
 * @return  the index of the first occurrence of the character in the
 *          character sequence represented by this object that is greater
 *          than or equal to {@code fromIndex}, or {@code -1}
 *          if the character does not occur.
 */
public int indexOf(int ch, int fromIndex) {
    final int max = value.length;
    if (fromIndex < 0) {
        fromIndex = 0;
    } else if (fromIndex >= max) {
        // Note: fromIndex might be near -1>>>1.
        return -1;
    }

    if (ch < Character.MIN_SUPPLEMENTARY_CODE_POINT) {
        // handle most cases here (ch is a BMP code point or a
        // negative value (invalid code point))
        final char[] value = this.value;
        for (int i = fromIndex; i < max; i++) {
            if (value[i] == ch) {
                return i;
            }
        }
        return -1;
    } else {
        return indexOfSupplementary(ch, fromIndex);
    }
}

/**
 * Handles (rare) calls of indexOf with a supplementary character.
 */
private int indexOfSupplementary(int ch, int fromIndex) {
    if (Character.isValidCodePoint(ch)) {
        final char[] value = this.value;
        final char hi = Character.highSurrogate(ch);
        final char lo = Character.lowSurrogate(ch);
        final int max = value.length - 1;
        for (int i = fromIndex; i < max; i++) {
            if (value[i] == hi && value[i + 1] == lo) {
                return i;
            }
        }
    }
    return -1;
}
```





## substring

当beginIndex为0是返回的是自己，否则是一个新的字符串。

jdk1.6和1.7subString发生了一些变化:

jdk1.6可能会导致一些内存泄露问题

所以一般用x = x.substring(x, y) + ""解决

参考文献:http://hollischuang.gitee.io/tobetopjavaer/#/basics/java-basic/substring

```java
/**
 * Returns a string that is a substring of this string. The
 * substring begins with the character at the specified index and
 * extends to the end of this string. <p>
 * Examples:
 * <blockquote><pre>
 * "unhappy".substring(2) returns "happy"
 * "Harbison".substring(3) returns "bison"
 * "emptiness".substring(9) returns "" (an empty string)
 * </pre></blockquote>
 *
 * @param      beginIndex   the beginning index, inclusive.
 * @return     the specified substring.
 * @exception  IndexOutOfBoundsException  if
 *             {@code beginIndex} is negative or larger than the
 *             length of this {@code String} object.
 */
public String substring(int beginIndex) {
    if (beginIndex < 0) {
        throw new StringIndexOutOfBoundsException(beginIndex);
    }
    int subLen = value.length - beginIndex;
    if (subLen < 0) {
        throw new StringIndexOutOfBoundsException(subLen);
    }
    return (beginIndex == 0) ? this : new String(value, beginIndex, subLen);
}
```

```java
//JDK 6
String(int offset, int count, char value[]) {
    this.value = value;
    this.offset = offset;
    this.count = count;
}

public String substring(int beginIndex, int endIndex) {
    //check boundary
    return  new String(offset + beginIndex, endIndex - beginIndex, value);
}
```

## replace



## split



## join



## trim

截取字符串中间的非空白字符，如果两段没有空白字符，则返回this

参考文献:https://blog.csdn.net/leeqihe/article/details/81006611

```java
/**
 * Returns a string whose value is this string, with any leading and trailing
 * whitespace removed.
 * <p>
 * If this {@code String} object represents an empty character
 * sequence, or the first and last characters of character sequence
 * represented by this {@code String} object both have codes
 * greater than {@code '\u005Cu0020'} (the space character), then a
 * reference to this {@code String} object is returned.
 * <p>
 * Otherwise, if there is no character with a code greater than
 * {@code '\u005Cu0020'} in the string, then a
 * {@code String} object representing an empty string is
 * returned.
 * <p>
 * Otherwise, let <i>k</i> be the index of the first character in the
 * string whose code is greater than {@code '\u005Cu0020'}, and let
 * <i>m</i> be the index of the last character in the string whose code
 * is greater than {@code '\u005Cu0020'}. A {@code String}
 * object is returned, representing the substring of this string that
 * begins with the character at index <i>k</i> and ends with the
 * character at index <i>m</i>-that is, the result of
 * {@code this.substring(k, m + 1)}.
 * <p>
 * This method may be used to trim whitespace (as defined above) from
 * the beginning and end of a string.
 *
 * @return  A string whose value is this string, with any leading and trailing white
 *          space removed, or this string if it has no leading or
 *          trailing white space.
 */
public String trim() {
    int len = value.length;
    int st = 0;
    char[] val = value;    /* avoid getfield opcode */

    while ((st < len) && (val[st] <= ' ')) {
        st++;
    }
    while ((st < len) && (val[len - 1] <= ' ')) {
        len--;
    }
    return ((st > 0) || (len < value.length)) ? substring(st, len) : this;
}
```

## valueOf

为空返回null，不为空返回toString()

```java
 /**
     * Returns the string representation of the {@code Object} argument.
     *
     * @param   obj   an {@code Object}.
     * @return  if the argument is {@code null}, then a string equal to
     *          {@code "null"}; otherwise, the value of
     *          {@code obj.toString()} is returned.
     * @see     java.lang.Object#toString()
     */
    public static String valueOf(Object obj) {
        return (obj == null) ? "null" : obj.toString();
    }

```



## copyValueOf

不清楚这两个方法的意义，因为已经有相同的代码了。确实是一模一样，而且不同版本之前也没有区别。

摘自stackOverFlow中Stephen C的回答

https://stackoverflow.com/questions/48134493/java-valueof-vs-copyvalueof

![image-20200330233354777](C:\Users\81929\AppData\Roaming\Typora\typora-user-images\image-20200330233354777.png)



```java

    /**
     * Returns the string representation of the {@code char} array
     * argument. The contents of the character array are copied; subsequent
     * modification of the character array does not affect the returned
     * string.
     *
     * @param   data     the character array.
     * @return  a {@code String} that contains the characters of the
     *          character array.
     */
    public static String valueOf(char data[]) {
        return new String(data);
    }

    /**
     * Returns the string representation of a specific subarray of the
     * {@code char} array argument.
     * <p>
     * The {@code offset} argument is the index of the first
     * character of the subarray. The {@code count} argument
     * specifies the length of the subarray. The contents of the subarray
     * are copied; subsequent modification of the character array does not
     * affect the returned string.
     *
     * @param   data     the character array.
     * @param   offset   initial offset of the subarray.
     * @param   count    length of the subarray.
     * @return  a {@code String} that contains the characters of the
     *          specified subarray of the character array.
     * @exception IndexOutOfBoundsException if {@code offset} is
     *          negative, or {@code count} is negative, or
     *          {@code offset+count} is larger than
     *          {@code data.length}.
     */
    public static String valueOf(char data[], int offset, int count) {
        return new String(data, offset, count);
    }

    /**
     * Equivalent to {@link #valueOf(char[], int, int)}.
     *
     * @param   data     the character array.
     * @param   offset   initial offset of the subarray.
     * @param   count    length of the subarray.
     * @return  a {@code String} that contains the characters of the
     *          specified subarray of the character array.
     * @exception IndexOutOfBoundsException if {@code offset} is
     *          negative, or {@code count} is negative, or
     *          {@code offset+count} is larger than
     *          {@code data.length}.
     */
    public static String copyValueOf(char data[], int offset, int count) {
        return new String(data, offset, count);
    }

    /**
     * Equivalent to {@link #valueOf(char[])}.
     *
     * @param   data   the character array.
     * @return  a {@code String} that contains the characters of the
     *          character array.
     */
    public static String copyValueOf(char data[]) {
        return new String(data);
    }
```



## intern

intern() 方法返回字符串对象的规范化表示形式。

# 各个版本代码重大改变





# 面试常见问题





# 参考文献

[1]: https://segmentfault.com/a/1190000010799123	"科普：为什么 String hashCode 方法选择数字31作为乘子"
[2]: https://www.zhihu.com/question/42176549	"Java中字符的高代理highSurrogate和低代理lowSurrogate分别是什么？"
[3]: http://hollischuang.gitee.io/tobetopjavaer/#/basics/java-basic/substring	"JDK 6和JDK 7中substring的原理及区别"
[4]: https://blog.csdn.net/leeqihe/article/details/81006611	"string.trim()究竟去掉了什么？"

