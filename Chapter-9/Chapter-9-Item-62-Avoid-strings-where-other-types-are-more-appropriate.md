## Chapter 9. General Programming（通用程序设计）

### Item 62: Avoid strings where other types are more appropriate（其他类型更合适时应避免使用字符串）

Strings are designed to represent text, and they do a fine job of it. Because strings are so common and so well supported by the language, there is a natural tendency to use strings for purposes other than those for which they were designed. This item discusses a few things that you shouldn’t do with strings.

字符串被设计用来表示文本，它们在这方面做得很好。因为字符串是如此常见，并且受到 Java 的良好支持，所以很自然地会将字符串用于其他目的，而不是它们适用的场景。本条目讨论了一些不应该使用字符串的场景。

**Strings are poor substitutes for other value types.** When a piece of data comes into a program from a file, from the network, or from keyboard input, it is often in string form. There is a natural tendency to leave it that way, but this tendency is justified only if the data really is textual in nature. If it’s numeric, it should be translated into the appropriate numeric type, such as int, float, or BigInteger. If it’s the answer to a yes-or-no question, it should be translated into an appropriate enum type or a boolean. More generally, if there’s an appropriate value type, whether primitive or object reference, you should use it; if there isn’t, you should write one. While this advice may seem obvious, it is often violated.

字符串是其他值类型的糟糕替代品。当一段数据从文件、网络或键盘输入到程序时，它通常是字符串形式的。有一种很自然的倾向是保持这种格式不变，但是这种倾向只有在数据本质上是文本的情况下才合理。如果是数值类型，则应将其转换为适当的数值类型，如 int、float 或 BigInteger。如果是问题的答案，如「是」或「否」这类形式，则应将其转换为适当的枚举类型或布尔值。更一般地说，如果有合适的值类型，无论是基本类型还是对象引用，都应该使用它；如果没有，你应该写一个。虽然这条建议似乎很多余，但经常被违反。

**Strings are poor substitutes for enum types.** As discussed in Item 34, enums make far better enumerated type constants than strings.

**字符串是枚举类型的糟糕替代品。** 正如 [Item-34](/Chapter-6/Chapter-6-Item-34-Use-enums-instead-of-int-constants.md) 中所讨论的，枚举类型常量比字符串更适合于枚举类型常量。

**Strings are poor substitutes for aggregate types.** If an entity has multiple components, it is usually a bad idea to represent it as a single string. For example, here’s a line of code that comes from a real system—identifier names have been changed to protect the guilty:

**字符串是聚合类型的糟糕替代品。** 如果一个实体有多个组件，将其表示为单个字符串通常是一个坏主意。例如，下面这行代码来自一个真实的系统标识符，它的名称已经被更改，以免引发罪责：

```
// Inappropriate use of string as aggregate type
String compoundKey = className + "#" + i.next();
```

This approach has many disadvantages. If the character used to separate fields occurs in one of the fields, chaos may result. To access individual fields, you have to parse the string, which is slow, tedious, and error-prone. You can’t provide equals, toString, or compareTo methods but are forced to accept the behavior that String provides. A better approach is simply to write a class to represent the aggregate, often a private static member class (Item 24).

这种方法有很多缺点。如果用于分隔字段的字符出现在其中一个字段中，可能会导致混乱。要访问各个字段，你必须解析字符串，这是缓慢的、冗长的、容易出错的过程。你不能提供 equals、toString 或 compareTo 方法，但必须接受 String 提供的行为。更好的方法是编写一个类来表示聚合，通常是一个私有静态成员类（[Item-24](/Chapter-4/Chapter-4-Item-24-Favor-static-member-classes-over-nonstatic.md)）。

**Strings are poor substitutes for capabilities.** Occasionally, strings are used to grant access to some functionality. For example, consider the design of a thread-local variable facility. Such a facility provides variables for which each thread has its own value. The Java libraries have had a thread-local variable facility since release 1.2, but prior to that, programmers had to roll their own. When confronted with the task of designing such a facility many years ago, several people independently came up with the same design, in which clientprovided string keys are used to identify each thread-local variable:

**字符串不能很好地替代 capabilities。** 有时，字符串用于授予对某些功能的访问权。例如，考虑线程本地变量机制的设计。这样的机制提供了每个线程都有自己的变量值。自 1.2 版以来，Java 库就有了一个线程本地变量机制，但在此之前，程序员必须自己设计。许多年前，当面临设计这样一个机制的任务时，有人提出了相同的设计，其中客户端提供的字符串键，用于标识每个线程本地变量：

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

这种方法的问题在于，字符串键表示线程本地变量的共享全局名称空间。为了使这种方法有效，客户端提供的字符串键必须是惟一的：如果两个客户端各自决定为它们的线程本地变量使用相同的名称，它们无意中就会共享一个变量，这通常会导致两个客户端都失败。而且，安全性很差。恶意客户端可以故意使用与另一个客户端相同的字符串密钥来非法访问另一个客户端的数据。

This API can be fixed by replacing the string with an unforgeable key (sometimes called a capability):

这个 API 可以通过用一个不可伪造的键（有时称为 capability）替换字符串来修复：

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

虽然这解决了 API 中基于字符串的两个问题，但是你可以做得更好。你不再真正需要静态方法。它们可以变成键上的实例方法，此时键不再是线程局部变量的键值：而是成为线程局部变量本身。此时，顶层类不再为你做任何事情，所以你可以删除它，并将嵌套类重命名为 ThreadLocal：

```
public final class ThreadLocal {
    public ThreadLocal();
    public void set(Object value);
    public Object get();
}
```

This API isn’t typesafe, because you have to cast the value from Object to its actual type when you retrieve it from a thread-local variable. It is impossible to make the original String-based API typesafe and difficult to make the Keybased API typesafe, but it is a simple matter to make this API typesafe by making ThreadLocal a parameterized class (Item 29):

这个 API 不是类型安全的，因为在从线程本地变量检索值时，必须将值从 Object 转换为它的实际类型。原始的基于 String 类型 API 的类型安全是不可能实现的，基于键的 API 的类型安全也是很难实现的，但是通过将 ThreadLocal 作为一个参数化的类来实现这个 API 的类型安全很简单（[Item-29](/Chapter-5/Chapter-5-Item-29-Favor-generic-types.md)）：

```
public final class ThreadLocal<T> {
    public ThreadLocal();
    public void set(T value);
    public T get();
}
```

This is, roughly speaking, the API that java.lang.ThreadLocal provides. In addition to solving the problems with the string-based API, it is faster and more elegant than either of the key-based APIs.

粗略地说，这就是 `java.lang.ThreadLocal` 提供的 API，除了解决基于字符串的问题之外，它比任何基于键的 API 都更快、更优雅。

To summarize, avoid the natural tendency to represent objects as strings when better data types exist or can be written. Used inappropriately, strings are more cumbersome, less flexible, slower, and more error-prone than other types. Types for which strings are commonly misused include primitive types, enums, and aggregate types.

总之，当存在或可以编写更好的数据类型时，应避免将字符串用来表示对象。如果使用不当，字符串比其他类型更麻烦、灵活性更差、速度更慢、更容易出错。字符串经常被误用的类型包括基本类型、枚举和聚合类型。

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-9/Chapter-9-Introduction.md)**
- **Previous Item（上一条目）：[Item 61: Prefer primitive types to boxed primitives（基本数据类型优于包装类）](/Chapter-9/Chapter-9-Item-61-Prefer-primitive-types-to-boxed-primitives.md)**
- **Next Item（下一条目）：[Item 63: Beware the performance of string concatenation（当心字符串连接引起的性能问题）](/Chapter-9/Chapter-9-Item-63-Beware-the-performance-of-string-concatenation.md)**
