## Chapter 2. Creating and Destroying Objects（创建和销毁对象）

### Item 6: Avoid creating unnecessary objects（避免创建不必要的对象）

It is often appropriate to reuse a single object instead of creating a new functionally equivalent object each time it is needed. Reuse can be both faster and more stylish. An object can always be reused if it is immutable (Item 17).

重用单个对象通常是合适的，不必每次需要时都创建一个新的功能等效对象。重用可以更快、更时尚。如果对象是不可变的，那么它总是可以被重用的（[Item-17](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-4/Chapter-4-Item-17-Minimize-mutability.md)）。

As an extreme example of what not to do, consider this statement:

作为一个不该做的极端例子，请考虑下面的语句：

```
String s = new String("bikini"); // DON'T DO THIS!
```

The statement creates a new String instance each time it is executed, and none of those object creations is necessary. The argument to the String constructor ("bikini") is itself a String instance, functionally identical to all of the objects created by the constructor. If this usage occurs in a loop or in a frequently invoked method, millions of String instances can be created needlessly.

该语句每次执行时都会创建一个新的 String 实例，而这些对象创建都不是必需的。String 构造函数的参数（"bikini"）本身就是一个 String 实例，在功能上与构造函数创建的所有对象相同。如果这种用法发生在循环或频繁调用的方法中，则不必要创建数百万个 String 实例。

The improved version is simply the following:

改进后的版本如下：

```
String s = "bikini";
```

This version uses a single String instance, rather than creating a new one each time it is executed. Furthermore, it is guaranteed that the object will be reused by any other code running in the same virtual machine that happens to contain the same string literal [JLS, 3.10.5].

这个版本使用单个 String 实例，而不是每次执行时都创建一个新的实例。此外，可以保证在同一虚拟机中运行的其他代码都可以重用该对象，只要恰好包含相同的字符串字面量 [JLS, 3.10.5]。

You can often avoid creating unnecessary objects by using static factory methods (Item 1) in preference to constructors on immutable classes that provide both. For example, the factory method Boolean.valueOf(String) is preferable to the constructor Boolean(String), which was deprecated in Java 9. The constructor must create a new object each time it’s called, while the factory method is never required to do so and won’t in practice. In addition to reusing immutable objects, you can also reuse mutable objects if you know they won’t be modified.

你通常可以通过使用静态工厂方法（[Item-1](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-2/Chapter-2-Item-1-Consider-static-factory-methods-instead-of-constructors.md)）来避免创建不必要的对象，而不是在提供这两种方法的不可变类上使用构造函数。例如，工厂方法 Boolean.valueOf(String) 比构造函数 ~~Boolean(String)~~ 更可取，**后者在 Java 9 中被弃用了** 。构造函数每次调用时都必须创建一个新对象，而工厂方法从来不需要这样做，在实际应用中也不会这样做。除了重用不可变对象之外，如果知道可变对象不会被修改，也可以重用它们。

Some object creations are much more expensive than others. If you’re going to need such an “expensive object” repeatedly, it may be advisable（adj.明智的，适当的） to cache it for reuse. Unfortunately, it’s not always obvious when you’re creating such an object. Suppose you want to write a method to determine（v.下决心；vt.确定） whether a string is a valid Roman numeral. Here’s the easiest way to do this using a regular expression:

有些对象的创建的代价相比而言要昂贵得多。如果你需要重复地使用这样一个「昂贵的对象」，那么最好将其缓存以供重用。不幸的是，当你创建这样一个对象时，并不总是很明显。假设你要编写一个方法来确定字符串是否为有效的罗马数字。下面是使用正则表达式最简单的方法：

```
// Performance can be greatly improved!
static boolean isRomanNumeral(String s) {
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3})" + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
```

The problem with this implementation is that it relies on the String.matches method. **While String.matches is the easiest way to check if a string matches a regular expression, it’s not suitable for repeated use in performance-critical situations.** The problem is that it internally creates a Pattern instance for the regular expression and uses it only once, after which it becomes eligible for garbage collection. Creating a Pattern instance is expensive because it requires compiling the regular expression into a finite state machine.

这个实现的问题是它依赖于 String.matches 方法。**虽然 String.matches 是检查字符串是否与正则表达式匹配的最简单方法，但它不适合在性能关键的情况下重复使用。** 问题是，它在内部为正则表达式创建了一个模式实例，并且只使用一次，之后就可以进行垃圾收集了。创建一个模式实例是很昂贵的因为它需要将正则表达式编译成有限的状态机制。

To improve the performance, explicitly compile the regular expression into a Pattern instance (which is immutable) as part of class initialization, cache it,and reuse the same instance for every invocation of the isRomanNumeral method:

为了提高性能，将正则表达式显式编译为模式实例（它是不可变的），作为类初始化的一部分，缓存它，并在每次调用 isRomanNumeral 方法时重用同一个实例：

```
// Reusing expensive object for improved performance
public class RomanNumerals {
    private static final Pattern ROMAN = Pattern.compile("^(?=.)M*(C[MD]|D?C{0,3})" + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
    static boolean isRomanNumeral(String s) {
        return ROMAN.matcher(s).matches();
    }
}
```

The improved version of isRomanNumeral provides significant performance gains if invoked frequently. On my machine, the original version takes 1.1 μs on an 8-character input string, while the improved version takes 0.17 μs, which is 6.5 times faster. Not only is the performance improved, but arguably, so is clarity. Making a static final field for the otherwise invisible Pattern instance allows us to give it a name, which is far more readable than the regular expression itself.

如果频繁调用 isRomanNumeral，改进版本将提供显著的性能提升。在我的机器上，原始版本输入 8 字符的字符串花费 1.1μs，而改进的版本需要 0.17μs，快 6.5 倍。不仅性能得到了改善，清晰度也得到了提高。为不可见的模式实例创建一个静态终态字段允许我们为它命名，这比正则表达式本身更容易阅读。

If the class containing the improved version of the isRomanNumeral method is initialized but the method is never invoked, the field ROMAN will be initialized needlessly. It would be possible to eliminate the initialization by lazily initializing the field (Item 83) the first time the isRomanNumeral method is invoked, but this is not recommended. As is often the case with lazy initialization, it would complicate the implementation with no measurable performance improvement (Item 67).

如果加载包含改进版 isRomanNumeral 方法的类时，该方法从未被调用过，那么初始化字段 ROMAN 是不必要的。因此，可以用延迟初始化字段（[Item-83](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-11/Chapter-11-Item-83-Use-lazy-initialization-judiciously.md)）的方式在第一次调用 isRomanNumeral 方法时才初始化字段，而不是在类加载时初始化，但不建议这样做。通常情况下，延迟初始化会使实现复杂化，而没有明显的性能改善（[Item-67](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-9/Chapter-9-Item-67-Optimize-judiciously.md)）。

**译注：类加载通常指的是类的生命周期中加载、连接、初始化三个阶段。当方法没有在类加载过程中被使用时，可以不初始化与之相关的字段**

When an object is immutable, it is obvious it can be reused safely, but there are other situations where it is far less obvious, even counterintuitive. Consider the case of adapters [Gamma95], also known as views. An adapter is an object that delegates to a backing object, providing an alternative interface. Because an adapter has no state beyond that of its backing object, there’s no need to create more than one instance of a given adapter to a given object.

当一个对象是不可变的，很明显，它可以安全地重用，但在其他情况下，它远不那么明显，甚至违反直觉。考虑适配器的情况 [Gamma95]，也称为视图。适配器是委托给支持对象的对象，提供了一个替代接口。因为适配器的状态不超过其支持对象的状态，所以不需要为给定对象创建一个给定适配器的多个实例。

For example, the keySet method of the Map interface returns a Set view of the Map object, consisting of all the keys in the map. Naively, it would seem that every call to keySet would have to create a new Set instance, but every call to keySet on a given Map object may return the same Set instance. Although the returned Set instance is typically mutable（adj.易变的）, all of the returned objects are functionally identical: when one of the returned objects changes, so do all the others, because they’re all backed by the same Map instance. While it is largely harmless to create multiple instances of the keySet view object, it is unnecessary and has no benefits.

例如，Map 接口的 keySet 方法返回 Map 对象的 Set 视图，其中包含 Map 中的所有键。天真的是，对 keySet 的每次调用都必须创建一个新的 Set 实例，但是对给定 Map 对象上的 keySet 的每次调用都可能返回相同的 Set 实例。虽然返回的 Set 实例通常是可变的，但所有返回的对象在功能上都是相同的：当返回的对象之一发生更改时，所有其他对象也会发生更改，因为它们都由相同的 Map 实例支持。虽然创建 keySet 视图对象的多个实例基本上是无害的，但这是不必要的，也没有好处。

Another way to create unnecessary objects is autoboxing, which allows the programmer to mix primitive and boxed primitive types, boxing and unboxing automatically as needed. **Autoboxing blurs but does not erase the distinction between primitive and boxed primitive types.** There are subtle semantic distinctions and not-so-subtle performance differences (Item 61). Consider the following method, which calculates the sum of all the positive int values. To do this, the program has to use long arithmetic because an int is not big enough to hold the sum of all the positive int values:

另一种创建不必要对象的方法是自动装箱，它允许程序员混合原始类型和包装类型，根据需要自动装箱和拆箱。**自动装箱模糊了原始类型和包装类型之间的区别，** 两者有细微的语义差别和不明显的性能差别（[Item-61](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-9/Chapter-9-Item-61-Prefer-primitive-types-to-boxed-primitives.md)）。考虑下面的方法，它计算所有正整数的和。为了做到这一点，程序必须使用 long，因为 int 值不够大，不足以容纳所有正整数值的和：

```
// Hideously slow! Can you spot the object creation?
private static long sum() {
    Long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; i++)
        sum += i;
    return sum;
}
```

This program gets the right answer, but it is much slower than it should be,due to a one-character typographical error. The variable sum is declared as a Long instead of a long, which means that the program constructs about 231 unnecessary Long instances (roughly one for each time the long i is added to the Long sum). Changing the declaration of sum from Long to long reduces the runtime from 6.3 seconds to 0.59 seconds on my machine. The lesson is clear: **prefer primitives to boxed primitives, and watch out for unintentional autoboxing.**

这个程序得到了正确的答案，但是由于一个字符的印刷错误，它的速度比实际要慢得多。变量 sum 被声明为 Long 而不是 long，这意味着程序将构造大约 231 个不必要的 Long 实例（大约每次将 Long i 添加到 Long sum 时都有一个实例）。将 sum 的声明从 Long 更改为 long，机器上的运行时间将从 6.3 秒减少到 0.59 秒。教训很清楚：**基本数据类型优于包装类，还应提防意外的自动装箱。**

This item should not be misconstrued（vt.误解，曲解） to imply that object creation is expensive and should be avoided. On the contrary, the creation and reclamation of small objects whose constructors do little explicit work is cheap, especially on modern JVM implementations. Creating additional objects to enhance the clarity,simplicity, or power of a program is generally a good thing.

这个项目不应该被曲解为是在暗示创建对象是昂贵的，应该避免。相反，创建和回收这些小对象的构造函数成本是很低廉的，尤其是在现代 JVM 实现上。创建额外的对象来增强程序的清晰性、简单性或功能通常是件好事。

Conversely, avoiding object creation by maintaining your own object pool is a bad idea unless the objects in the pool are extremely heavyweight. The classic example of an object that does justify an object pool is a database connection.The cost of establishing the connection is sufficiently high that it makes sense to reuse these objects. Generally speaking, however, maintaining your own object pools clutters your code, increases memory footprint, and harms performance.Modern JVM implementations have highly optimized garbage collectors that easily outperform such object pools on lightweight objects.

相反，通过维护自己的对象池来避免创建对象不是一个好主意，除非池中的对象非常重量级。证明对象池是合理的对象的典型例子是数据库连接。建立连接的成本非常高，因此重用这些对象是有意义的。然而，一般来说，维护自己的对象池会使代码混乱，增加内存占用，并损害性能。现代 JVM 实现具有高度优化的垃圾收集器，在轻量级对象上很容易胜过这样的对象池。

The counterpoint to this item is Item 50 on defensive copying. The present item says, “Don’t create a new object when you should reuse an existing one,”while Item 50 says, “Don’t reuse an existing object when you should create a new one.” Note that the penalty for reusing an object when defensive copying is called for is far greater than the penalty for needlessly creating a duplicate object. Failing to make defensive copies where required can lead to insidious bugs and security holes; creating objects unnecessarily merely affects style and performance.

与此项对应的条目是 [Item-50](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-8/Chapter-8-Item-50-Make-defensive-copies-when-needed.md)（防御性复制）。当前项的描述是：「在应该重用现有对象时不要创建新对象」，而 Item 50 的描述则是：「在应该创建新对象时不要重用现有对象」。请注意，当需要进行防御性复制时，重用对象所受到的惩罚远远大于不必要地创建重复对象所受到的惩罚。在需要时不制作防御性副本可能导致潜在的 bug 和安全漏洞；不必要地创建对象只会影响样式和性能。
