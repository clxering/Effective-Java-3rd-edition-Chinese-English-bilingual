## Chapter 8. Methods（方法）

### Item 49: Check parameters for validity（检查参数的有效性）

Most methods and constructors have some restrictions on what values may be passed into their parameters. For example, it is not uncommon that index values must be non-negative and object references must be non-null. You should clearly document all such restrictions and enforce them with checks at the beginning of the method body. This is a special case of the general principle that you should attempt to detect errors as soon as possible after they occur. Failing to do so makes it less likely that an error will be detected and makes it harder to determine the source of an error once it has been detected.

大多数方法和构造函数都对传递给它们的参数值有一些限制。例如，索引值必须是非负的，对象引用必须是非空的，这种情况并不少见。你应该清楚地在文档中记录所有这些限制，并在方法主体的开头使用检查来实施它们。你应该在错误发生后尽快找到它们，这是一般原则。如果不这样做，就不太可能检测到错误，而且即使检测到错误，确定错误的来源也很难。

If an invalid parameter value is passed to a method and the method checks its parameters before execution, it will fail quickly and cleanly with an appropriate exception. If the method fails to check its parameters, several things could happen. The method could fail with a confusing exception in the midst of processing. Worse, the method could return normally but silently compute the wrong result. Worst of all, the method could return normally but leave some object in a compromised state, causing an error at some unrelated point in the code at some undetermined time in the future. In other words, failure to validate parameters, can result in a violation of failure atomicity (Item 76).

如果一个无效的参数值被传递给一个方法，如果该方法在执行之前会检查它的参数，那么这个过程将迅速失败，并引发适当的异常。如果方法未能检查其参数，可能会发生以下几件事。该方法可能会在处理过程中出现令人困惑的异常而失败。更糟的是，该方法可以正常返回，但会静默计算错误的结果。最糟糕的是，该方法可以正常返回，但会使某个对象处于隐患状态，从而在将来某个不确定的时间在代码中某个不相关的位置上导致错误。换句话说，如果没有验证参数，可能会违反故障原子性（[Item-76](/Chapter-10/Chapter-10-Item-76-Strive-for-failure-atomicity.md)）。

For public and protected methods, use the Javadoc @throws tag to document the exception that will be thrown if a restriction on parameter values is violated (Item 74). Typically, the resulting exception will be IllegalArgumentException, IndexOutOfBoundsException, or NullPointerException (Item 72). Once you’ve documented the restrictions on a method’s parameters and you’ve documented the exceptions that will be thrown if these restrictions are violated, it is a simple matter to enforce the restrictions. Here’s a typical example:

对于公共方法和受保护的方法，如果在方法说明使用 Javadoc 的 `@throws` 标签记录异常，表明如果违反了对参数值的限制，将会引发该异常（[Item-74](/Chapter-10/Chapter-10-Item-74-Document-all-exceptions-thrown-by-each-method.md)）。通常，生成的异常将是 IllegalArgumentException、IndexOutOfBoundsException 或 NullPointerException（[Item-72](/Chapter-10/Chapter-10-Item-72-Favor-the-use-of-standard-exceptions.md)）。一旦你在文档中记录了方法参数上的限制，并且记录了如果违反这些限制将引发的异常，那么实施这些限制就很简单了。这里有一个典型的例子：

```Java
/**
* Returns a BigInteger whose value is (this mod m). This method
* differs from the remainder method in that it always returns a
* non-negative BigInteger.
**
@param m the modulus, which must be positive
* @return this mod m
* @throws ArithmeticException if m is less than or equal to 0
*/
public BigInteger mod(BigInteger m) {
    if (m.signum() <= 0)
        throw new ArithmeticException("Modulus <= 0: " + m);
    ... // Do the computation
}
```

Note that the doc comment does not say “mod throws NullPointerException if m is null,” even though the method does exactly that, as a byproduct of invoking m.signum(). This exception is documented in the class-level doc comment for the enclosing BigInteger class. The classlevel comment applies to all parameters in all of the class’s public methods. This is a good way to avoid the clutter of documenting every NullPointerException on every method individually. It may be combined with the use of @Nullable or a similar annotation to indicate that a particular parameter may be null, but this practice is not standard, and multiple annotations are in use for this purpose.

注意，文档注释并没有说「如果 m 为空，mod 将抛出NullPointerException」，尽管方法确实是这样做的，这是调用 `m.signum()` 的副产品。这个异常记录在类级别的文档注释中，用于包含 BigInteger 类。类级别注释适用于类的所有公共方法中的所有参数。这是避免在每个方法上分别记录每个 NullPointerException 而造成混乱的好方法。它可以与 `@Nullable` 或类似的注释结合使用，以指示某个特定参数可能为 null，但这种做法并不标准，为此使用了多个注释。

**The Objects.requireNonNull method, added in Java 7, is flexible and convenient, so there’s no reason to perform null checks manually anymore.** You can specify your own exception detail message if you wish. The method returns its input, so you can perform a null check at the same time as you use a value:

**在 Java 7 中添加的 `Objects.requireNonNull` 方法非常灵活和方便，因此不再需要手动执行空检查。** 如果愿意，可以指定自己的异常详细信息。该方法返回它的输入，所以你可以执行一个空检查，同时你使用一个值：

```Java
// Inline use of Java's null-checking facility
this.strategy = Objects.requireNonNull(strategy, "strategy");
```

You can also ignore the return value and use Objects.requireNonNull as a freestanding null check where that suits your needs.

你还可以忽略返回值并使用 `Objects.requireNonNull` 作为一个独立的 null 检查来满足你的需要。

In Java 9, a range-checking facility was added to java.util.Objects. This facility consists of three methods: checkFromIndexSize, checkFromToIndex, and checkIndex. This facility is not as flexible as the null-checking method. It doesn’t let you specify your own exception detail message, and it is designed solely for use on list and array indices. It does not handle closed ranges (which contain both of their endpoints). But if it does what you need, it’s a useful convenience.

在 Java 9 中，范围检查功能被添加到 `java.util.Objects` 中。这个功能由三个方法组成：checkFromIndexSize、checkFromToIndex 和 checkIndex。这个工具不如空检查方法灵活。它不允许你指定自己的异常详细信息，而且它仅用于 List 和数组索引。它不处理封闭范围（包含两个端点）。但如果它满足你的需求，它仍是一个有用的工具。

For an unexported method, you, as the package author, control the circumstances under which the method is called, so you can and should ensure that only valid parameter values are ever passed in. Therefore, nonpublic methods can check their parameters using assertions, as shown below:

对于未导出的方法，作为包的作者，你应该定制方法调用的环境，因此你可以并且应该确保只传递有效的参数值。因此，非公共方法可以使用断言检查它们的参数，如下所示：

```Java
// Private helper function for a recursive sort
private static void sort(long a[], int offset, int length) {
    assert a != null;
    assert offset >= 0 && offset <= a.length;
    assert length >= 0 && length <= a.length - offset;
    ... // Do the computation
}
```

In essence, these assertions are claims that the asserted condition will be true, regardless of how the enclosing package is used by its clients. Unlike normal validity checks, assertions throw AssertionError if they fail. And unlike normal validity checks, they have no effect and essentially no cost unless you enable them, which you do by passing the -ea (or -enableassertions) flag to the java command. For more information on assertions, see the tutorial [Asserts].

从本质上说，这些断言在声明时，条件将为 true，而不管其客户端如何使用所包含的包。与普通的有效性检查不同，如果断言失败，则会抛出 AssertionError。与普通的有效性检查不同，如果断言没有起到作用，本质上不存在成本，除非你启用它们，你可以通过将 `-ea`（ 或 `-enableassertion`）标志传递给 java 命令来启用它们。有关断言的更多信息，请参见教程 [Asserts]。

It is particularly important to check the validity of parameters that are not used by a method, but stored for later use. For example, consider the static factory method on page 101, which takes an int array and returns a List view of the array. If a client were to pass in null, the method would throw a NullPointerException because the method has an explicit check (the call to Objects.requireNonNull). Had the check been omitted, the method would return a reference to a newly created List instance that would throw a NullPointerException as soon as a client attempted to use it. By that time, the origin of the List instance might be difficult to determine, which could greatly complicate the task of debugging.

特别重要的是，应检查那些不是由方法使用，而是存储起来供以后使用的参数的有效性。例如，考虑第 101 页中的静态工厂方法，它接受一个 int 数组并返回数组的 List 视图。如果客户端传入 null，该方法将抛出 NullPointerException，因为该方法具有显式检查(调用 `Objects.requireNonNull`)。如果省略了检查，该方法将返回对新创建的 List 实例的引用，该实例将在客户端试图使用它时抛出 NullPointerException。到那时，List 实例的起源可能很难确定，这可能会使调试任务变得非常复杂。

Constructors represent a special case of the principle that you should check the validity of parameters that are to be stored away for later use. It is critical to check the validity of constructor parameters to prevent the construction of an object that violates its class invariants.

构造函数代表了一种特殊的情况，即，你应该检查要存储起来供以后使用的参数的有效性。检查构造函数参数的有效性对于防止构造生成实例对象时，违背类的对象的不变性非常重要。

There are exceptions to the rule that you should explicitly check a method’s parameters before performing its computation. An important exception is the case in which the validity check would be expensive or impractical and the check is performed implicitly in the process of doing the computation. For example, consider a method that sorts a list of objects, such as Collections.sort(List). All of the objects in the list must be mutually comparable. In the process of sorting the list, every object in the list will be compared to some other object in the list. If the objects aren’t mutually comparable, one of these comparisons will throw a ClassCastException, which is exactly what the sort method should do. Therefore, there would be little point in checking ahead of time that the elements in the list were mutually comparable. Note, however, that indiscriminate reliance on implicit validity checks can result in the loss of failure atomicity (Item 76).

在执行方法的计算任务之前，应该显式地检查方法的参数，这条规则也有例外。一个重要的例外是有效性检查成本较高或不切实际，或者检查是在计算过程中隐式执行了。例如，考虑一个为对象 List 排序的方法，比如 `Collections.sort(List)`。List 中的所有对象必须相互比较。在对 List 排序的过程中，List 中的每个对象都会与列表中的其他对象进行比较。如果对象不能相互比较，将抛出 ClassCastException，这正是 sort 方法应该做的。因此，没有必要预先检查列表中的元素是否具有可比性。但是，请注意，不加区别地依赖隐式有效性检查可能导致失败原子性的丢失（[Item-76](/Chapter-10/Chapter-10-Item-76-Strive-for-failure-atomicity.md)）。

Occasionally, a computation implicitly performs a required validity check but throws the wrong exception if the check fails. In other words, the exception that the computation would naturally throw as the result of an invalid parameter value doesn’t match the exception that the method is documented to throw. Under these circumstances, you should use the exception translation idiom, described in Item 73, to translate the natural exception into the correct one.

有时，计算任务会隐式地执行所需的有效性检查，但如果检查失败，则抛出错误的异常。换句话说，计算任务由于无效参数值抛出的异常，与文档中记录的方法要抛出的异常不匹配。在这种情况下，你应该使用 [Item-73](/Chapter-10/Chapter-10-Item-73-Throw-exceptions-appropriate-to-the-abstraction.md) 中描述的异常转译技术来将计算任务抛出的异常转换为正确的异常。

Do not infer from this item that arbitrary restrictions on parameters are a good thing. On the contrary, you should design methods to be as general as it is practical to make them. The fewer restrictions that you place on parameters, the better, assuming the method can do something reasonable with all of the parameter values that it accepts. Often, however, some restrictions are intrinsic to the abstraction being implemented.

不要从本条目推断出：对参数的任意限制都是一件好事。相反，你应该把方法设计得既通用又实用。对参数施加的限制越少越好，假设该方法可以对它所接受的所有参数值进行合理的处理。然而，一些限制常常是实现抽象的内在限制。

To summarize, each time you write a method or constructor, you should think about what restrictions exist on its parameters. You should document these restrictions and enforce them with explicit checks at the beginning of the method body. It is important to get into the habit of doing this. The modest work that it entails will be paid back with interest the first time a validity check fails.

总而言之，每次编写方法或构造函数时，都应该考虑参数存在哪些限制。你应该在文档中记录这些限制，并在方法主体的开头显式地检查。养成这样的习惯是很重要的。它所涉及的这一少量工作及其所花费的时间，将在有效性检查出现第一次失败时连本带利地偿还。

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-8/Chapter-8-Introduction.md)**
- **Next Item（下一条目）：[Item 50: Make defensive copies when needed（在需要时制作防御性副本）](/Chapter-8/Chapter-8-Item-50-Make-defensive-copies-when-needed.md)**
