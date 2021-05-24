## Chapter 10. Exceptions（异常）

### Item 69: Use exceptions only for exceptional conditions（仅在确有异常条件下使用异常）

Someday, if you are unlucky, you may stumble across a piece of code that looks something like this:

你可能会偶然发现这样一段代码：

```Java
// Horrible abuse of exceptions. Don't ever do this!
try {
    int i = 0;
    while(true)
        range[i++].climb();
    }
    catch (ArrayIndexOutOfBoundsException e) {
}
```

What does this code do? It’s not at all obvious from inspection, and that’s reason enough not to use it (Item 67). It turns out to be a horribly ill-conceived idiom for looping through the elements of an array. The infinite loop terminates by throwing, catching, and ignoring an ArrayIndexOutOfBoundsException when it attempts to access the first array element outside the bounds of the array. It’s supposed to be equivalent to the standard idiom for looping through an array, which is instantly recognizable to any Java programmer:

这段代码是做什么的？从表面上看，一点也不明显，这足以成为不使用它的充分理由（[Item-67](/Chapter-9/Chapter-9-Item-67-Optimize-judiciously.md)）。事实证明，这是一个用于遍历数组的元素的非常糟糕的习惯用法。当试图访问数组边界之外的数组元素时，通过抛出、捕获和忽略 ArrayIndexOutOfBoundsException 来终止无限循环。如下循环遍历数组的标准习惯用法，任何 Java 程序员都可以立即识别它：

```Java
for (Mountain m : range)
    m.climb();
```

So why would anyone use the exception-based loop in preference to the tried and true? It’s a misguided attempt to improve performance based on the faulty reasoning that, since the VM checks the bounds of all array accesses, the normal loop termination test—hidden by the compiler but still present in the for-each loop—is redundant and should be avoided. There are three things wrong with this reasoning:

那么，为什么会有人使用基于异常的循环而不使用习惯的循环模式呢？由于 VM 检查所有数组访问的边界，所以误认为正常的循环终止测试被编译器隐藏了，但在 for-each 循环中仍然可见，这无疑是多余的，应该避免，因此利用错误判断机制来提高性能是错误的。这种思路有三点误区：

- Because exceptions are designed for exceptional circumstances, there is little incentive for JVM implementors to make them as fast as explicit tests.

因为异常是为特殊情况设计的，所以 JVM 实现几乎不会让它们像显式测试一样快。

- Placing code inside a try-catch block inhibits certain optimizations that JVM implementations might otherwise perform.

将代码放在 try-catch 块中会抑制 JVM 可能执行的某些优化。

- The standard idiom for looping through an array doesn’t necessarily result in redundant checks. Many JVM implementations optimize them away.

遍历数组的标准习惯用法不一定会导致冗余检查。许多 JVM 实现对它们进行了优化。

In fact, the exception-based idiom is far slower than the standard one. On my machine, the exception-based idiom is about twice as slow as the standard one for arrays of one hundred elements.

事实上，基于异常的用法比标准用法慢得多。在我的机器上，用 100 个元素的数组测试，基于异常的用法与标准用法相比速度大约慢了两倍。

Not only does the exception-based loop obfuscate the purpose of the code and reduce its performance, but it’s not guaranteed to work. If there is a bug in the loop, the use of exceptions for flow control can mask the bug, greatly complicating the debugging process. Suppose the computation in the body of the loop invokes a method that performs an out-of-bounds access to some unrelated array. If a reasonable loop idiom were used, the bug would generate an uncaught exception, resulting in immediate thread termination with a full stack trace. If the misguided exception-based loop were used, the bug-related exception would be caught and misinterpreted as a normal loop termination.

基于异常的循环不仅混淆了代码的目的，降低了代码的性能，而且不能保证它能正常工作。如果循环中存在 bug，使用异常进行流程控制会掩盖该 bug，从而大大增加调试过程的复杂性。假设循环体中的计算步骤调用一个方法，该方法对一些不相关的数组执行越界访问。如果使用合理的循环习惯用法，该 bug 将生成一个未捕获的异常，导致线程立即终止，并带有完整的堆栈跟踪。相反，如果使用了基于异常的循环，当捕获与 bug 相关的异常时，会将其误判为正常的循环终止条件。

The moral of this story is simple: **Exceptions are, as their name implies, to be used only for exceptional conditions; they should never be used for ordinary control flow.** More generally, use standard, easily recognizable idioms in preference to overly clever techniques that purport to offer better performance. Even if the performance advantage is real, it may not remain in the face of steadily improving platform implementations. The subtle bugs and maintenance headaches that come from overly clever techniques, however, are sure to remain.

这个案例的寓意很简单：**顾名思义，异常只适用于确有异常的情况；它们不应该用于一般的控制流程。** 更进一步说，使用标准的、易于识别的习惯用法，而不是声称能够提供更好性能的过于抖机灵的技术。即使性能优势是真实存在的，但在稳步改进平台实现的前提下，这种优势也并不可靠。而且，来自抖机灵的技术存在的细微缺陷和维护问题肯定会继续存在。

This principle also has implications for API design. **A well-designed API must not force its clients to use exceptions for ordinary control flow.** A class with a “state-dependent” method that can be invoked only under certain unpredictable conditions should generally have a separate “state-testing” method indicating whether it is appropriate to invoke the state-dependent method. For example, the Iterator interface has the state-dependent method next and the corresponding state-testing method hasNext. This enables the standard idiom for iterating over a collection with a traditional for loop (as well as the for-each loop, where the hasNext method is used internally):

这个原则对 API 设计也有影响。一个设计良好的 API 不能迫使其客户端为一般的控制流程使用异常。只有在某些不可预知的条件下才能调用具有「状态依赖」方法的类，通常应该有一个单独的「状态测试」方法，表明是否适合调用「状态依赖」方法。例如，Iterator 接口具有「状态依赖」的 next 方法和对应的「状态测试」方法 hasNext。这使得传统 for 循环（在 for-each 循环内部也使用了 hasNext 方法）在集合上进行迭代成为标准习惯用法：

```Java
for (Iterator<Foo> i = collection.iterator(); i.hasNext(); ) {
    Foo foo = i.next();
    ...
}
```

If Iterator lacked the hasNext method, clients would be forced to do this instead:

如果 Iterator 缺少 hasNext 方法，客户端将被迫这样做：

```Java
// Do not use this hideous code for iteration over a collection!
try {
    Iterator<Foo> i = collection.iterator();
    while(true) {
        Foo foo = i.next();
        ...
    }
}
catch (NoSuchElementException e) {
}
```

This should look very familiar after the array iteration example that began this item. In addition to being wordy and misleading, the exception-based loop is likely to perform poorly and can mask bugs in unrelated parts of the system.

这与一开始举例的对数组进行迭代的例子非常相似，除了冗长和误导之外，基于异常的循环执行效果可能很差，并且会掩盖系统中不相关部分的 bug。

An alternative to providing a separate state-testing method is to have the statedependent method return an empty optional (Item 55) or a distinguished value such as null if it cannot perform the desired computation.

提供单独的「状态测试」方法的另一种方式，就是让「状态依赖」方法返回一个空的 Optional 对象（[Item-55](/Chapter-8/Chapter-8-Item-55-Return-optionals-judiciously.md)），或者在它不能执行所需的计算时返回一个可识别的值，比如 null。

Here are some guidelines to help you choose between a state-testing method and an optional or distinguished return value. If an object is to be accessed concurrently without external synchronization or is subject to externally induced state transitions, you must use an optional or distinguished return value, as the object’s state could change in the interval between the invocation of a state-testing method and its state-dependent method. Performance concerns may dictate that an optional or distinguished return value be used if a separate statetesting method would duplicate the work of the state-dependent method. All other things being equal, a state-testing method is mildly preferable to a distinguished return value. It offers slightly better readability, and incorrect use may be easier to detect: if you forget to call a state-testing method, the statedependent method will throw an exception, making the bug obvious; if you forget to check for a distinguished return value, the bug may be subtle. This is not an issue for optional return values.

有一些指导原则，帮助你在「状态测试」方法、Optional、可识别的返回值之间进行选择。（1）如果要在没有外部同步的情况下并发地访问对象，或者受制于外部条件的状态转换，则必须使用 Optional 或可识别的返回值，因为对象的状态可能在调用「状态测试」方法与「状态依赖」方法的间隔中发生变化。（2）如果一个单独的「状态测试」方法重复「状态依赖」方法的工作，从性能问题考虑，可能要求使用 Optional 或可识别的返回值。（3）在所有其他条件相同的情况下，「状态测试」方法略优于可识别的返回值。它提供了较好的可读性，而且不正确的使用可能更容易被检测：如果你忘记调用「状态测试」方法，「状态依赖」方法将抛出异常，使错误显而易见；（4）如果你忘记检查一个可识别的返回值，那么这个 bug 可能很难发现。但是这对于返回 Optional 对象的方式来说不是问题。

In summary, exceptions are designed for exceptional conditions. Don’t use them for ordinary control flow, and don’t write APIs that force others to do so.

总之，异常是为确有异常的情况设计的。不要将它们用于一般的控制流程，也不要编写强制其他人这样做的 API。

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-10/Chapter-10-Introduction.md)**
- **Next Item（下一条目）：[Item 70: Use checked exceptions for recoverable conditions and runtime exceptions for programming errors（对可恢复情况使用 checked 异常，对编程错误使用运行时异常）](/Chapter-10/Chapter-10-Item-70-Use-checked-exceptions-for-recoverable-conditions-and-runtime-exceptions-for-programming-errors.md)**
