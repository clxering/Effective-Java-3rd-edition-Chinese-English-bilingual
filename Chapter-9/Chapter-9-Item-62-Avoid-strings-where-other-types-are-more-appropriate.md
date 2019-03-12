## Chapter 9. General Programming（通用程序设计）

### Item 62: Avoid strings where other types are more appropriate（其他类型更合适时应避免使用字符串）

Strings are designed to represent text, and they do a fine job of it. Because strings are so common and so well supported by the language, there is a natural tendency to use strings for purposes other than those for which they were designed. This item discusses a few things that you shouldn’t do with strings.

**Strings are poor substitutes for other value types.** When a piece of data comes into a program from a file, from the network, or from keyboard input, it is often in string form. There is a natural tendency to leave it that way, but this tendency is justified only if the data really is textual in nature. If it’s numeric, it should be translated into the appropriate numeric type, such as int, float, or BigInteger. If it’s the answer to a yes-or-no question, it should be translated into an appropriate enum type or a boolean. More generally, if there’s an appropriate value type, whether primitive or object reference, you should use it; if there isn’t, you should write one. While this advice may seem obvious, it is often violated.

**Strings are poor substitutes for enum types.** As discussed in Item 34, enums make far better enumerated type constants than strings.

**Strings are poor substitutes for aggregate types.** If an entity has multiple components, it is usually a bad idea to represent it as a single string. For example, here’s a line of code that comes from a real system—identifier names have been changed to protect the guilty:

```
// Inappropriate use of string as aggregate type
String compoundKey = className + "#" + i.next();
```

This approach has many disadvantages. If the character used to separate fields occurs in one of the fields, chaos may result. To access individual fields, you have to parse the string, which is slow, tedious, and error-prone. You can’t provide equals, toString, or compareTo methods but are forced to accept the behavior that String provides. A better approach is simply to write a class to represent the aggregate, often a private static member class (Item 24).

**Strings are poor substitutes for capabilities.** Occasionally, strings are used to grant access to some functionality. For example, consider the design of a thread-local variable facility. Such a facility provides variables for which each thread has its own value. The Java libraries have had a thread-local variable facility since release 1.2, but prior to that, programmers had to roll their own. When confronted with the task of designing such a facility many years ago, several people independently came up with the same design, in which clientprovided string keys are used to identify each thread-local variable:

```
// Broken - inappropriate use of string as capability!
public class ThreadLocal {
    private ThreadLocal() { } // Noninstantiable

    // Sets the current thread's value for the named variable.
    public static void set(String key, Object value);

    // Returns the current thread's value for the named variable.
    public static Object get(String key);
}
```

The problem with this approach is that the string keys represent a shared global namespace for thread-local variables. In order for the approach to work, the client-provided string keys have to be unique: if two clients independently decide to use the same name for their thread-local variable, they unintentionally share a single variable, which will generally cause both clients to fail. Also, the security is poor. A malicious client could intentionally use the same string key as another client to gain illicit access to the other client’s data.

This API can be fixed by replacing the string with an unforgeable key (sometimes called a capability):

```
public class ThreadLocal {
    private ThreadLocal() { } // Noninstantiable

    public static class Key { // (Capability)
        Key() { }
}

// Generates a unique, unforgeable key
public static Key getKey() {
    return new Key();
}

public static void set(Key key, Object value);

public static Object get(Key key);
}
```

While this solves both of the problems with the string-based API, you can do much better. You don’t really need the static methods anymore. They can instead become instance methods on the key, at which point the key is no longer a key for a thread-local variable: it is a thread-local variable. At this point, the toplevel class isn’t doing anything for you anymore, so you might as well get rid of it and rename the nested class to ThreadLocal:

```
public final class ThreadLocal {
public ThreadLocal();
public void set(Object value);
public Object get();
}
```

This API isn’t typesafe, because you have to cast the value from Object to its actual type when you retrieve it from a thread-local variable. It is impossible to make the original String-based API typesafe and difficult to make the Keybased API typesafe, but it is a simple matter to make this API typesafe by making ThreadLocal a parameterized class (Item 29):

```
public final class ThreadLocal<T> {
public ThreadLocal();
public void set(T value);
public T get();
}
```

This is, roughly speaking, the API that java.lang.ThreadLocal provides. In addition to solving the problems with the string-based API, it is faster and more elegant than either of the key-based APIs.

To summarize, avoid the natural tendency to represent objects as strings when better data types exist or can be written. Used inappropriately, strings are more cumbersome, less flexible, slower, and more error-prone than other types. Types for which strings are commonly misused include primitive types, enums, and aggregate types.

---
**[Back to contents of the chapter（返回章节目录）](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-9/Chapter-9-Introduction.md)**
- **Previous Item（上一条目）：[Item 61: Prefer primitive types to boxed primitives（基本数据类型优于包装类）](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-9/Chapter-9-Item-61-Prefer-primitive-types-to-boxed-primitives.md)**
- **Next Item（下一条目）：[Item 63: Beware the performance of string concatenation（当心字符串连接引起的性能问题）](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-9/Chapter-9-Item-63-Beware-the-performance-of-string-concatenation.md)**
