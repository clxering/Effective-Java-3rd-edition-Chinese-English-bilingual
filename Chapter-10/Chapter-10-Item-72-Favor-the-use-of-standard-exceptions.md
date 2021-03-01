## Chapter 10. Exceptions（异常）

### Item 72: Favor the use of standard exceptions（鼓励复用标准异常）

An attribute that distinguishes expert programmers from less experienced ones is that experts strive for and usually achieve a high degree of code reuse. Exceptions are no exception to the rule that code reuse is a good thing. The Java libraries provide a set of exceptions that covers most of the exception-throwing needs of most APIs.

专家程序员与经验较少的程序员之间的一个区别是，专家力求实现高度的代码复用。代码复用是一件好事，异常也不例外。Java 库提供了一组异常，涵盖了大多数 API 的大多数异常抛出需求。

Reusing standard exceptions has several benefits. Chief among them is that it makes your API easier to learn and use because it matches the established conventions that programmers are already familiar with. A close second is that programs using your API are easier to read because they aren’t cluttered with unfamiliar exceptions. Last (and least), fewer exception classes means a smaller memory footprint and less time spent loading classes.

复用标准异常有几个好处。其中最主要的是，它使你的 API 更容易学习和使用，因为它符合程序员已经熟悉的既定约定。其次，使用你的 API 的程序更容易阅读，因为它们不会因为不熟悉的异常而混乱。最后（也是最不重要的），更少的异常类意味着更小的内存占用和更少的加载类的时间。

The most commonly reused exception type is IllegalArgumentException (Item 49). This is generally the exception to throw when the caller passes in an argument whose value is inappropriate. For example, this would be the exception to throw if the caller passed a negative number in a parameter representing the number of times some action was to be repeated.

最常见的复用异常类型是 IllegalArgumentException（[Item-49](/Chapter-8/Chapter-8-Item-49-Check-parameters-for-validity.md)）。这通常是调用者传入不合适的参数时抛出的异常。例如，如果调用者在表示某个操作要重复多少次的参数中传递了一个负数，则抛出这个异常。

Another commonly reused exception is IllegalStateException. This is generally the exception to throw if the invocation is illegal because of the state of the receiving object. For example, this would be the exception to throw if the caller attempted to use some object before it had been properly initialized.

另一个常被复用异常是 IllegalStateException。如果接收对象的状态导致调用非法，则通常会抛出此异常。例如，如果调用者试图在对象被正确初始化之前使用它，那么这将是抛出的异常。

Arguably, every erroneous method invocation boils down to an illegal argument or state, but other exceptions are standardly used for certain kinds of illegal arguments and states. If a caller passes null in some parameter for which null values are prohibited, convention dictates that NullPointerException be thrown rather than IllegalArgumentException. Similarly, if a caller passes an out-of-range value in a parameter representing an index into a sequence, IndexOutOfBoundsException should be thrown rather than IllegalArgumentException.

可以说，每个错误的方法调用都归结为参数非法或状态非法，但是有一些异常通常用于某些特定的参数非法和状态非法。如果调用者在禁止空值的参数中传递 null，那么按照惯例，抛出 NullPointerException 而不是 IllegalArgumentException。类似地，如果调用者将表示索引的参数中的超出范围的值传递给序列，则应该抛出 IndexOutOfBoundsException，而不是 IllegalArgumentException。

Another reusable exception is ConcurrentModificationException. It should be thrown if an object that was designed for use by a single thread (or with external synchronization) detects that it is being modified concurrently. This exception is at best a hint because it is impossible to reliably detect concurrent modification.

另一个可复用异常是 ConcurrentModificationException。如果一个对象被设计为由单个线程使用（或与外部同步），并且检测到它正在被并发地修改，则应该抛出该异常。因为不可能可靠地检测并发修改，所以该异常充其量只是一个提示。

A last standard exception of note is UnsupportedOperationException. This is the exception to throw if an object does not support an attempted operation. Its use is rare because most objects support all of their methods. This exception is used by classes that fail to implement one or more optional operations defined by an interface they implement. For example, an append-only List implementation would throw this exception if someone tried to delete an element from the list.

最后一个需要注意的标准异常是 UnsupportedOperationException。如果对象不支持尝试的操作，则抛出此异常。它很少使用，因为大多数对象都支持它们的所有方法。此异常用于一个类没有实现由其实现的接口定义的一个或多个可选操作。例如，对于只支持追加操作的 List 实现，试图从中删除元素时就会抛出这个异常。

**Do not reuse Exception, RuntimeException, Throwable, or Error directly.** Treat these classes as if they were abstract. You can't reliably test for these exceptions because they are superclasses of other exceptions that a method may throw.

**不要直接复用 Exception、RuntimeException、Throwable 或 Error。** 应当将这些类视为抽象类。你不能对这些异常进行可靠的测试，因为它们是方法可能抛出的异常的超类。

This table summarizes the most commonly reused exceptions:

此表总结了最常见的可复用异常：


|    Exception    |       Occasion for Use       |
|:-------:|:-------:|
|   IllegalArgumentException  |     Non-null parameter value is inappropriate（非空参数值不合适）    |
|   IllegalStateException  |     Object state is inappropriate for method invocation（对象状态不适用于方法调用）    |
|   NullPointerException  |     Parameter value is null where prohibited（禁止参数为空时仍传入 null）    |
|   IndexOutOfBoundsException  |     Index parameter value is out of range（索引参数值超出范围）    |
|   ConcurrentModificationException  |     Concurrent modification of an object has been detected where it is prohibited（在禁止并发修改对象的地方检测到该动作）    |
|   UnsupportedOperationException  |     Object does not support method（对象不支持该方法调用）    |

While these are by far the most commonly reused exceptions, others may be reused where circumstances warrant. For example, it would be appropriate to reuse ArithmeticException and NumberFormatException if you were implementing arithmetic objects such as complex numbers or rational numbers. If an exception fits your needs, go ahead and use it, but only if the conditions under which you would throw it are consistent with the exception’s documentation: reuse must be based on documented semantics, not just on name. Also, feel free to subclass a standard exception if you want to add more detail (Item 75), but remember that exceptions are serializable (Chapter 12). That alone is reason not to write your own exception class without good reason.

虽然到目前为止，这些是最常见的复用异常，但是在环境允许的情况下也可以复用其他异常。例如，如果你正在实现诸如复数或有理数之类的算术对象，那么复用 ArithmeticException 和 NumberFormatException 是合适的。如果一个异常符合你的需要，那么继续使用它，但前提是你抛出它的条件与异常的文档描述一致：复用必须基于文档化的语义，而不仅仅是基于名称。另外，如果你想添加更多的细节，可以随意子类化标准异常(第75项)，但是请记住，异常是可序列化的（Chapter 12）。如果没有充分的理由，不要编写自己的异常类。

Choosing which exception to reuse can be tricky because the “occasions for use” in the table above do not appear to be mutually exclusive. Consider the case of an object representing a deck of cards, and suppose there were a method to deal a hand from the deck that took as an argument the size of the hand. If the caller passed a value larger than the number of cards remaining in the deck, it could be construed as an IllegalArgumentException (the handSize parameter value is too high) or an IllegalStateException (the deck contains too few cards). Under these circumstances, the rule is to throw IllegalStateException if no argument values would have worked, otherwise throw IllegalArgumentException.

选择复用哪个异常可能比较棘手，因为上表中的「使用场合」似乎并不相互排斥。考虑一个对象，表示一副牌，假设有一个方法代表发牌操作，该方法将手牌多少作为参数。如果调用者传递的值大于牌堆中剩余的牌的数量，则可以将其解释为 IllegalArgumentException （handSize 参数值太大）或 IllegalStateException（牌堆中包含的牌太少）。在这种情况下，规则是：如果没有参数值，抛出 IllegalStateException，否则抛出 IllegalArgumentException。

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-10/Chapter-10-Introduction.md)**
- **Previous Item（上一条目）：[Item 71: Avoid unnecessary use of checked exceptions（避免不必要地使用 checked 异常）](/Chapter-10/Chapter-10-Item-71-Avoid-unnecessary-use-of-checked-exceptions.md)**
- **Next Item（下一条目）：[Item 73: Throw exceptions appropriate to the abstraction（抛出能用抽象解释的异常）](/Chapter-10/Chapter-10-Item-73-Throw-exceptions-appropriate-to-the-abstraction.md)**
