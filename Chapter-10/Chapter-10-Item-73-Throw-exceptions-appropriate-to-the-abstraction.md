## Chapter 10. Exceptions（异常）

### Item 73: Throw exceptions appropriate to the abstraction（抛出能用抽象解释的异常）

It is disconcerting when a method throws an exception that has no apparent connection to the task that it performs. This often happens when a method propagates an exception thrown by a lower-level abstraction. Not only is it disconcerting, but it pollutes the API of the higher layer with implementation details. If the implementation of the higher layer changes in a later release, the exceptions it throws will change too, potentially breaking existing client programs.

当一个方法抛出一个与它所执行的任务没有明显关联的异常时，这是令人不安的。这种情况经常发生在由方法传播自低层抽象抛出的异常。它不仅令人不安，而且让实现细节污染了上层的 API。如果高层实现在以后的版本中发生变化，那么它抛出的异常也会发生变化，可能会破坏现有的客户端程序。

To avoid this problem, **higher layers should catch lower-level exceptions and, in their place, throw exceptions that can be explained in terms of the higher-level abstraction.** This idiom is known as exception translation:

为了避免这个问题，**高层应该捕获低层异常，并确保抛出的异常可以用高层抽象解释。** 这个习惯用法称为异常转换：

```Java
// Exception Translation
try {
    ... // Use lower-level abstraction to do our bidding
} catch (LowerLevelException e) {
    throw new HigherLevelException(...);
}
```

Here is an example of exception translation taken from the AbstractSequentialList class, which is a skeletal implementation (Item 20) of the List interface. In this example, exception translation is mandated by the specification of the get method in the `List<E>` interface:

下面是来自 AbstractSequentialList 类的异常转换示例，该类是 List 接口的一个框架实现（[Item-20](/Chapter-4/Chapter-4-Item-20-Prefer-interfaces-to-abstract-classes.md)）。在本例中，异常转换是由 `List<E>` 接口中的 get 方法规范强制执行的：

```Java
/**
* Returns the element at the specified position in this list.
* @throws IndexOutOfBoundsException if the index is out of range
* ({@code index < 0 || index >= size()}).
*/
public E get(int index) {
    ListIterator<E> i = listIterator(index);
    try {
        return i.next();
    }
    catch (NoSuchElementException e) {
        throw new IndexOutOfBoundsException("Index: " + index);
    }
}
```

A special form of exception translation called exception chaining is called for in cases where the lower-level exception might be helpful to someone debugging the problem that caused the higher-level exception. The lower-level exception (the cause) is passed to the higher-level exception, which provides an accessor method (Throwable’s getCause method) to retrieve the lower-level exception:

如果低层异常可能有助于调试高层异常的问题，则需要一种称为链式异常的特殊异常转换形式。低层异常（作为原因）传递给高层异常，高层异常提供一个访问器方法（Throwable 的 getCause 方法）来检索低层异常：

```Java
// Exception Chaining
try {
    ... // Use lower-level abstraction to do our bidding
}
catch (LowerLevelException cause) {
    throw new HigherLevelException(cause);
}
```

The higher-level exception’s constructor passes the cause to a chaining-aware superclass constructor, so it is ultimately passed to one of Throwable’s chaining-aware constructors, such as Throwable(Throwable):

高层异常的构造函数将原因传递给能够接收链式异常的超类构造函数，因此它最终被传递给 Throwable 的一个接收链式异常的构造函数，比如 `Throwable(Throwable)`：

```Java
// Exception with chaining-aware constructor
class HigherLevelException extends Exception {
    HigherLevelException(Throwable cause) {
        super(cause);
    }
}
```

Most standard exceptions have chaining-aware constructors. For exceptions that don’t, you can set the cause using Throwable’s initCause method. Not only does exception chaining let you access the cause programmatically (with getCause), but it integrates the cause’s stack trace into that of the higher-level exception.

大多数标准异常都有接收链式异常的构造函数。对于不支持链式异常的异常，可以使用 Throwable 的 initCause 方法设置原因。异常链接不仅允许你以编程方式访问原因（使用 getCause），而且还将原因的堆栈跟踪集成到更高层异常的堆栈跟踪中。

**While exception translation is superior to mindless propagation of exceptions from lower layers, it should not be overused.** Where possible, the best way to deal with exceptions from lower layers is to avoid them, by ensuring that lower-level methods succeed. Sometimes you can do this by checking the validity of the higher-level method’s parameters before passing them on to lower layers.

虽然异常转换优于底层异常的盲目传播，但它不应该被过度使用。在可能的情况下，处理低层异常的最佳方法是确保低层方法避免异常。有时，你可以在将高层方法的参数传递到低层之前检查它们的有效性。

If it is impossible to prevent exceptions from lower layers, the next best thing is to have the higher layer silently work around these exceptions, insulating the caller of the higher-level method from lower-level problems. Under these circumstances, it may be appropriate to log the exception using some appropriate logging facility such as java.util.logging. This allows programmers to investigate the problem, while insulating client code and the users from it.

如果不可能从低层防止异常，那么下一个最好的方法就是让高层静默处理这些异常，使较高层方法的调用者免受低层问题的影响。在这种情况下，可以使用一些适当的日志工具（如 `java.util.logging`）来记录异常。这允许程序员研究问题，同时将客户端代码和用户与之隔离。

In summary, if it isn’t feasible to prevent or to handle exceptions from lower layers, use exception translation, unless the lower-level method happens to guarantee that all of its exceptions are appropriate to the higher level. Chaining provides the best of both worlds: it allows you to throw an appropriate higherlevel exception, while capturing the underlying cause for failure analysis (Item 75).

总之，如果无法防止或处理来自低层的异常，则使用异常转换，但要保证低层方法的所有异常都适用于较高层。链式异常提供了兼顾两方面的最佳服务：允许抛出适当的高层异常，同时捕获并分析失败的潜在原因（[Item-75](/Chapter-10/Chapter-10-Item-75-Include-failure-capture-information-in-detail-messages.md)）。

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-10/Chapter-10-Introduction.md)**
- **Previous Item（上一条目）：[Item 72: Favor the use of standard exceptions（鼓励复用标准异常）](/Chapter-10/Chapter-10-Item-72-Favor-the-use-of-standard-exceptions.md)**
- **Next Item（下一条目）：[Item 74: Document all exceptions thrown by each method（为每个方法记录会抛出的所有异常）](/Chapter-10/Chapter-10-Item-74-Document-all-exceptions-thrown-by-each-method.md)**
