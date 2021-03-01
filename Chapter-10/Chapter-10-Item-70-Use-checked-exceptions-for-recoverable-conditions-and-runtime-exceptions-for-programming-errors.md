## Chapter 10. Exceptions（异常）

### Item 70: Use checked exceptions for recoverable conditions and runtime exceptions for programming errors（对可恢复情况使用 checked 异常，对编程错误使用运行时异常）

Java provides three kinds of throwables: checked exceptions, runtime exceptions, and errors. There is some confusion among programmers as to when it is appropriate to use each kind of throwable. While the decision is not always clear-cut, there are some general rules that provide strong guidance.

Java 提供了三种可抛出项：checked 异常、运行时异常和错误。程序员们对什么时候使用这些可抛出项比较困惑。虽然决策并不总是明确的，但是有一些通用规则可以提供强有力的指导。

The cardinal rule in deciding whether to use a checked or an unchecked exception is this: **use checked exceptions for conditions from which the caller can reasonably be expected to recover.** By throwing a checked exception, you force the caller to handle the exception in a catch clause or to propagate it outward. Each checked exception that a method is declared to throw is therefore a potent indication to the API user that the associated condition is a possible outcome of invoking the method.

决定是使用 checked 异常还是 unchecked 异常的基本规则是：**使用 checked 异常的情况是为了合理地期望调用者能够从中恢复。** 通过抛出一个 checked 的异常，你可以强制调用者在 catch 子句中处理异常，或者将其传播出去。因此，方法中声明的要抛出的每个 checked 异常，都清楚的向 API 用户表明：the associated condition is a possible outcome of invoking the method.

By confronting the user with a checked exception, the API designer presents a mandate to recover from the condition. The user can disregard the mandate by catching the exception and ignoring it, but this is usually a bad idea (Item 77).

通过向用户提供 checked 异常，API 设计者提供了从条件中恢复的要求。用户为了无视强制要求，可以捕获异常并忽略，但这通常不是一个好主意（[Item-77](/Chapter-10/Chapter-10-Item-77-Don’t-ignore-exceptions.md)）
。

There are two kinds of unchecked throwables: runtime exceptions and errors. They are identical in their behavior: both are throwables that needn’t, and generally shouldn’t, be caught. If a program throws an unchecked exception or an error, it is generally the case that recovery is impossible and continued execution would do more harm than good. If a program does not catch such a throwable, it will cause the current thread to halt with an appropriate error message.

有两种 unchecked 的可抛出项：运行时异常和错误。它们在行为上是一样的：都是可抛出的，通常不需要也不应该被捕获。如果程序抛出 unchecked 异常或错误，通常情况下是不可能恢复的，如果继续执行，弊大于利。如果程序没有捕获到这样的可抛出项，它将导致当前线程停止，并发出适当的错误消息。

**Use runtime exceptions to indicate programming errors.** The great majority of runtime exceptions indicate precondition violations. A precondition violation is simply a failure by the client of an API to adhere to the contract established by the API specification. For example, the contract for array access specifies that the array index must be between zero and the array length minus one, inclusive. ArrayIndexOutOfBoundsException indicates that this precondition was violated.

**使用运行时异常来指示编程错误。** 绝大多数运行时异常都表示操作违反了先决条件。违反先决条件是指使用 API 的客户端未能遵守 API 规范所建立的约定。例如，数组访问约定指定数组索引必须大于等于 0 并且小于等于 length-1 （length：数组长度）。ArrayIndexOutOfBoundsException 表示违反了此先决条件。

One problem with this advice is that it is not always clear whether you’re dealing with a recoverable conditions or a programming error. For example, consider the case of resource exhaustion, which can be caused by a programming error such as allocating an unreasonably large array, or by a genuine shortage of resources. If resource exhaustion is caused by a temporary shortage or by temporarily heightened demand, the condition may well be recoverable. It is a matter of judgment on the part of the API designer whether a given instance of resource exhaustion is likely to allow for recovery. If you believe a condition is likely to allow for recovery, use a checked exception; if not, use a runtime exception. If it isn’t clear whether recovery is possible, you’re probably better off using an unchecked exception, for reasons discussed in Item 71.

这个建议存在的问题是，并不总能清楚是在处理可恢复的条件还是编程错误。例如，考虑资源耗尽的情况，这可能是由编程错误（如分配一个不合理的大数组）或真正的资源短缺造成的。如果资源枯竭是由于暂时短缺或暂时需求增加造成的，这种情况很可能是可以恢复的。对于 API 设计人员来说，判断给定的资源耗尽实例是否允许恢复是一个问题。如果你认为某个条件可能允许恢复，请使用 checked 异常；如果没有，则使用运行时异常。如果不清楚是否可以恢复，最好使用 unchecked 异常，原因将在 [Item-71](/Chapter-10/Chapter-10-Item-71-Avoid-unnecessary-use-of-checked-exceptions.md) 中讨论。

While the Java Language Specification does not require it, there is a strong convention that errors are reserved for use by the JVM to indicate resource deficiencies, invariant failures, or other conditions that make it impossible to continue execution. Given the almost universal acceptance of this convention, it’s best not to implement any new Error subclasses. Therefore, **all of the unchecked throwables you implement should subclass RuntimeException** (directly or indirectly). Not only shouldn’t you define Error subclasses, but with the exception of AssertionError, you shouldn’t throw them either.

虽然 Java 语言规范没有要求，但有一个约定俗成的约定，即错误保留给 JVM 使用，以指示：资源不足、不可恢复故障或其他导致无法继续执行的条件。考虑到这种约定被大众认可，所以最好不要实现任何新的 Error 子类。因此，**你实现的所有 unchecked 可抛出项都应该继承 RuntimeException**（直接或间接）。不仅不应该定义 Error 子类，而且除了 AssertionError 之外，不应该抛出它们。

It is possible to define a throwable that is not a subclass of Exception, RuntimeException, or Error. The JLS doesn’t address such throwables directly but specifies implicitly that they behave as ordinary checked exceptions (which are subclasses of Exception but not RuntimeException). So when should you use such a beast? In a word, never. They have no benefits over ordinary checked exceptions and would serve merely to confuse the user of your API.

可以自定义一种可抛出的异常，它不是 Exception、RuntimeException 或 Error 的子类。JLS 不直接处理这些可抛出项，而是隐式地指定它们作为普通 checked 异常（普通 checked 异常是 Exception 的子类，但不是 RuntimeException 的子类）。那么，什么时候应该使用这样的「猛兽」呢？总之，永远不要。与普通 checked 异常相比，它们没有任何好处，只会让 API 的用户感到困惑。

API designers often forget that exceptions are full-fledged objects on which arbitrary methods can be defined. The primary use of such methods is to provide code that catches the exception with additional information concerning the condition that caused the exception to be thrown. In the absence of such methods, programmers have been known to parse the string representation of an exception to ferret out additional information. This is extremely bad practice (Item 12). Throwable classes seldom specify the details of their string representations, so string representations can differ from implementation to implementation and release to release. Therefore, code that parses the string representation of an exception is likely to be nonportable and fragile.

API 设计人员常常忘记异常是成熟对象，可以为其定义任意方法。此类方法的主要用途是提供捕获异常的代码，并提供有关引发异常的附加信息。如果缺乏此类方法，程序员需要自行解析异常的字符串表示以获取更多信息。这是极坏的做法（[Item-12](/Chapter-3/Chapter-3-Item-12-Always-override-toString.md)）。这种类很少指定其字符串表示的细节，因此字符串表示可能因实现而异，也可能因版本而异。因此，解析异常的字符串表示形式的代码可能是不可移植且脆弱的。

Because checked exceptions generally indicate recoverable conditions, it’s especially important for them to provide methods that furnish information to help the caller recover from the exceptional condition. For example, suppose a checked exception is thrown when an attempt to make a purchase with a gift card fails due to insufficient funds. The exception should provide an accessor method to query the amount of the shortfall. This will enable the caller to relay the amount to the shopper. See Item 75 for more on this topic.

因为 checked 异常通常表示可恢复的条件，所以这类异常来说，设计能够提供信息的方法来帮助调用者从异常条件中恢复尤为重要。例如，假设当使用礼品卡购物由于资金不足而失败时，抛出一个 checked 异常。该异常应提供一个访问器方法来查询差额。这将使调用者能够将金额传递给购物者。有关此主题的更多信息，请参见 [Item-75](/Chapter-10/Chapter-10-Item-75-Include-failure-capture-information-in-detail-messages.md)。

To summarize, throw checked exceptions for recoverable conditions and unchecked exceptions for programming errors. When in doubt, throw unchecked exceptions. Don’t define any throwables that are neither checked exceptions nor runtime exceptions. Provide methods on your checked exceptions to aid in recovery.

总而言之，为可恢复条件抛出 checked 异常，为编程错误抛出 unchecked 异常。当有疑问时，抛出 unchecked 异常。不要定义任何既不是 checked 异常也不是运行时异常的自定义异常。应该为 checked 异常设计相关的方法，如提供异常信息，以帮助恢复。

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-10/Chapter-10-Introduction.md)**
- **Previous Item（上一条目）：[Item 69: Use exceptions only for exceptional conditions（仅在确有异常条件下使用异常）](/Chapter-10/Chapter-10-Item-69-Use-exceptions-only-for-exceptional-conditions.md)**
- **Next Item（下一条目）：[Item 71: Avoid unnecessary use of checked exceptions（避免不必要地使用 checked 异常）](/Chapter-10/Chapter-10-Item-71-Avoid-unnecessary-use-of-checked-exceptions.md)**
