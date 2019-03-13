## Chapter 10. Exceptions（异常）

### Item 69: Use exceptions only for exceptional conditions （只在异常的情况下才使用异常）

Someday, if you are unlucky, you may stumble across a piece of code that looks something like this:

某一天，假如你可能很不幸地碰到下面这样的代码：

```
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

这段代码有什么作用？ 看起来很不明显，这就是他不真正被使用的原因（见第67条）。 事实证明，作为一种循环遍历数组元素的实现方式，这是一种很糟糕的构想。 当这个无限循环在尝试访问数组边界外的第一个数组元素时，使用抛出，捕获和忽略ArrayIndexOutOfBoundsException的的方式从而使循环中止。其实任何Java程序员都可以很快意识到，它等价于下面的标准循环用法：


```
for (Mountain m : range)
    m.climb();
```

So why would anyone use the exception-based loop in preference to the tried and true? It’s a misguided attempt to improve performance based on the faulty reasoning that, since the VM checks the bounds of all array accesses, the normal loop termination test—hidden by the compiler but still present in the for-each loop—is redundant and should be avoided. There are three things wrong with this reasoning:

那么为什么有人会使用基于异常的循环而不是使用上面这种行之有效的方案呢？ 他们企图使用错误判断机制可以提高性能，因为VM对每次数组访问都会检查是否越界，他们就认为正常的循环终止测试被编译器隐藏了，但在for-each循环着仍然可见，这显然是多余的，应该避免。 这个想法有以下三个错误：

- Because exceptions are designed for exceptional circumstances, there is little incentive for JVM implementors to make them as fast as explicit tests.

- Placing code inside a try-catch block inhibits certain optimizations that JVM implementations might otherwise perform.

- The standard idiom for looping through an array doesn’t necessarily result in redundant checks. Many JVM implementations optimize them away.

 - 因为异常是针对异常的情况而设计的，所以JVM实现者很少会对他们进行优化，使它们像显式的测试一样快。

 - 将代码放在try-catch块中会禁止JVM实现本来可能会执行的某些优化。

 - 循环遍历数组的标准用法不一定会导致JVM进行冗余检查。 一些现代的JVM会实现此类优化。

In fact, the exception-based idiom is far slower than the standard one. On my machine, the exception-based idiom is about twice as slow as the standard one for arrays of one hundred elements.

事实上，在现代的JVM实现上，基于异常的模式远比标准模式慢。在我的机器上，同样是100个元素数组遍历，基于异常的模式比标准模式慢了两倍。

Not only does the exception-based loop obfuscate the purpose of the code and reduce its performance, but it’s not guaranteed to work. If there is a bug in the loop, the use of exceptions for flow control can mask the bug, greatly complicating the debugging process. Suppose the computation in the body of the loop invokes a method that performs an out-of-bounds access to some unrelated array. If a reasonable loop idiom were used, the bug would generate an uncaught exception, resulting in immediate thread termination with a full stack trace. If the misguided exception-based loop were used, the bug-related exception would be caught and misinterpreted as a normal loop termination.

基于异常的循环不仅会混淆代码的目的并降低其性能，而且不能保证它能够正常工作。如果循环中存在无关的错误，流控制异常会掩盖错误，从而使调试过程变得非常复杂。假设循环体中的计算调用一个方法，该方法对一些不相关的数组执行越界访问。如果使用了合理的循环模式，则该错误将生成未捕获的异常，从而导致立即终止线程，产生完整的堆栈轨迹。如果使用了这个误导的基于异常的循环模式，则会捕获与错误相关的异常并将其误解为正常的循环终止。

The moral of this story is simple: **Exceptions are, as their name implies, to be used only for exceptional conditions; they should never be used for ordinary control flow.** More generally, use standard, easily recognizable idioms in preference to overly clever techniques that purport to offer better performance. Even if the performance advantage is real, it may not remain in the face of steadily improving platform implementations. The subtle bugs and maintenance headaches that come from overly clever techniques, however, are sure to remain.

这个例子的教训很简单：**顾名思义，异常仅用于异常情况;它们永远不应该用于普通的控制流程。**更一般地说，使用标准的，易于识别的模式，而不是声称能提供更好的性能实际上回弄巧成拙的模式。即使性能提升是真的，面对平台实现的不断改进，这种提升也不可能会一直继续保持下去。然而，这种过度聪明的模式所带来的微妙Bug和难以维护的问题反而会一直存在。

This principle also has implications for API design. **A well-designed API must not force its clients to use exceptions for ordinary control flow.** A class with a “state-dependent” method that can be invoked only under certain unpredictable conditions should generally have a separate “state-testing” method indicating whether it is appropriate to invoke the state-dependent method. For example, the Iterator interface has the state-dependent method next and the corresponding state-testing method hasNext. This enables the standard idiom for iterating over a collection with a traditional for loop (as well as the for-each loop, where the hasNext method is used internally):

这个原则对API设计也有影响。 **精心设计的API不得强制其客户端为了实现正常的控制流而使用异常。**如果类具有“状态依赖”方法，也就是只能在某些不可预测的条件下调用的方法，那它通常应该有一个单独的“状态测试”来指示是否适合调用依赖于此状态的方法。例如，Iterator有一个状态依赖方法next，和相应的状态测试方法hasNext。这使得利用传统的for循环来遍历集合的标准模式成为可能（以及for-each循环，在这里是指在内部使用hasNext方法）：

```
for (Iterator<Foo> i = collection.iterator(); i.hasNext(); ) {
    Foo foo = i.next();
    ...
}
```

If Iterator lacked the hasNext method, clients would be forced to do this instead:

如果Iterator缺少hasNext方法，客户端将被迫改用下面的做法：
```
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

这看起来应该非常像本条目刚开始时的数组迭代示例。除了冗长和误导之外，基于异常的循环模式可能执行起来性能不佳并且还可能会掩盖系统中其他不相关部分中的错误。

An alternative to providing a separate state-testing method is to have the statedependent method return an empty optional (Item 55) or a distinguished value such as null if it cannot perform the desired computation.

另一种代替单独的“状态测试”的方法是使"状态依赖"方法返回空的optional（第55条）或者在它不能执行所需的计算的情况下返回一个可识别的值，例如null。

Here are some guidelines to help you choose between a state-testing method and an optional or distinguished return value. If an object is to be accessed concurrently without external synchronization or is subject to externally induced state transitions, you must use an optional or distinguished return value, as the object’s state could change in the interval between the invocation of a statetesting method and its state-dependent method. Performance concerns may dictate that an optional or distinguished return value be used if a separate statetesting method would duplicate the work of the state-dependent method. All other things being equal, a state-testing method is mildly preferable to a distinguished return value. It offers slightly better readability, and incorrect use may be easier to detect: if you forget to call a state-testing method, the statedependent method will throw an exception, making the bug obvious; if you forget to check for a distinguished return value, the bug may be subtle. This is not an issue for optional return values.

有些指导原则，可帮助你在“状态测试方法（state-testing method）”和“可选的（optional）或可识别的（distinguished）返回值”之间进行选择。如果对象在没有外部同步的情况下被并发访问或者可被外界改变状态，则必须使用可选或可区分的返回值，因为对象的状态可能会在调用“状态测试”方法与调用对应的“状态相关”方法的时间间隔内发生变化。如果单独的“状态测试”方法将复制与状态相关的方法的工作，从性能的角度看，应该要求使用optional或可区分（null）的返回值。在所有其他条件相同的情况下，“状态测试”方法稍微优于可别识别的返回值。它提供了更好的可读性，并且更可能检测到使用上的错误，例如：如果您忘记调用状态测试方法，则“状态依赖”方法将抛出异常，使错误变得明显；如果你忘记检查可识别的返回值（null），这个Bug可能很难被发现。如果使用可选返回值（optional）则不会遇到这个问题。

In summary, exceptions are designed for exceptional conditions. Don’t use them for ordinary control flow, and don’t write APIs that force others to do so.

总之，异常是针对异常的情况而设计的。不要将它们用于普通的控制流程，也不要编写迫使这么做的API。





