## Chapter 9. General Programming（通用程序设计）

### Item 57: Minimize the scope of local variables（将局部变量的作用域最小化）

This item is similar in nature to Item 15, “Minimize the accessibility of classes and members.” By minimizing the scope of local variables, you increase the readability and maintainability of your code and reduce the likelihood of error.

本条目在性质上类似于 [Item-15](/Chapter-4/Chapter-4-Item-15-Minimize-the-accessibility-of-classes-and-members.md)，即「最小化类和成员的可访问性」。通过最小化局部变量的范围，可以提高代码的可读性和可维护性，并降低出错的可能性。

Older programming languages, such as C, mandated that local variables must be declared at the head of a block, and some programmers continue to do this out of habit. It’s a habit worth breaking. As a gentle reminder, Java lets you declare variables anywhere a statement is legal (as does C, since C99).

较老的编程语言，如 C 语言，强制要求必须在代码块的头部声明局部变量，一些程序员出于习惯目前继续这样做。这是一个应改变的习惯。温馨提醒，Java 允许你在任何能够合法使用语句的地方声明变量（这与 C99 标准后 C 语言一样）。

**The most powerful technique for minimizing the scope of a local variable is to declare it where it is first used.** If a variable is declared before it is used, it’s just clutter—one more thing to distract the reader who is trying to figure out what the program does. By the time the variable is used, the reader might not remember the variable’s type or initial value.

**将局部变量的作用域最小化，最具说服力的方式就是在第一次使用它的地方声明。** 如果一个变量在使用之前声明了，代码会变得很混乱，这是另一件分散读者注意力的事情，因为读者正在试图弄清楚程序的功能。在使用到该变量时，读者可能不记得变量的类型或初始值。

Declaring a local variable prematurely can cause its scope not only to begin too early but also to end too late. The scope of a local variable extends from the point where it is declared to the end of the enclosing block. If a variable is declared outside of the block in which it is used, it remains visible after the program exits that block. If a variable is used accidentally before or after its region of intended use, the consequences can be disastrous.

过早地声明局部变量会导致其作用域开始得太早，而且结束得过晚。局部变量的范围应该从声明它的地方直到封闭块的末尾。如果变量在使用它的代码块外部声明，则在程序退出该块之后它仍然可见。如果一个变量在其预期使用区域之前或之后意外使用，其后果可能是灾难性的。

**Nearly every local variable declaration should contain an initializer.** If you don’t yet have enough information to initialize a variable sensibly, you should postpone the declaration until you do. One exception to this rule concerns try-catch statements. If a variable is initialized to an expression whose evaluation can throw a checked exception, the variable must be initialized inside a try block (unless the enclosing method can propagate the exception). If the value must be used outside of the try block, then it must be declared before the try block, where it cannot yet be “sensibly initialized.” For an example, see page 283.

**每个局部变量声明都应该包含一个初始化表达式。** 如果你还没有足够的信息来合理地初始化一个变量，你应该推迟声明，直到条件满足。这个规则的一个例外是 try-catch 语句。如果一个变量被初始化为一个表达式，该表达式的计算结果可以抛出一个 checked 异常，那么该变量必须在 try 块中初始化（除非所包含的方法可以传播异常）。如果该值必须在 try 块之外使用，那么它必须在 try 块之前声明，此时它还不能「合理地初始化」。例子可参见 283 页。

Loops present a special opportunity to minimize the scope of variables. The for loop, in both its traditional and for-each forms, allows you to declare loop variables, limiting their scope to the exact region where they’re needed. (This region consists of the body of the loop and the code in parentheses between the for keyword and the body.) Therefore, prefer for loops to while loops, assuming the contents of the loop variable aren’t needed after the loop terminates.

循环提供了一个特殊的机会来最小化变量的范围。for 循环的传统形式和 for-each 形式都允许声明循环变量，将它们的作用域精确限制在需要它们的区域。（这个区域由循环的主体以及 for 关键字和主体之间括号中的代码组成。）因此，假设循环结束后不再需要循环变量，for 循环就优于 while 循环。

For example, here is the preferred idiom for iterating over a collection (Item 58):

例如，下面是遍历集合的首选习惯用法（[Item-58](/Chapter-9/Chapter-9-Item-58-Prefer-for-each-loops-to-traditional-for-loops.md)）：

```Java
// Preferred idiom for iterating over a collection or array
for (Element e : c) {
    ... // Do Something with e
}
```

If you need access to the iterator, perhaps to call its remove method, the preferred idiom uses a traditional for loop in place of the for-each loop:

如果你需要访问 iterator，或者调用它的 remove 方法，首选的习惯用法是使用传统的 for 循环来代替 for-each 循环：

```Java
// Idiom for iterating when you need the iterator
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
    Element e = i.next();
    ... // Do something with e and i
}
```

To see why these for loops are preferable to a while loop, consider the following code fragment, which contains two while loops and one bug:

要弄清楚为什么 for 循环比 while 循环更好，请考虑下面的代码片段，其中包含两个 while 循环和一个 bug：

```Java
Iterator<Element> i = c.iterator();
while (i.hasNext()) {
    doSomething(i.next());
}
...
Iterator<Element> i2 = c2.iterator();
while (i.hasNext()) { // BUG!
    doSomethingElse(i2.next());
}
```

The second loop contains a copy-and-paste error: it initializes a new loop variable, i2, but uses the old one, i, which is, unfortunately, still in scope. The resulting code compiles without error and runs without throwing an exception, but it does the wrong thing. Instead of iterating over c2, the second loop terminates immediately, giving the false impression that c2 is empty. Because the program errs silently, the error can remain undetected for a long time.

第二个循环包含一个复制粘贴错误：它计划初始化一个新的循环变量 i2，却误用了旧的变量 i，不幸的是，i 仍然在作用域中。生成的代码编译时没有错误，运行时没有抛出异常，但是它做了错误的事情。第二个循环并没有遍历 c2，而是立即终止，从而产生 c2 为空的假象。因为程序会静默地出错，所以很长一段时间内都无法检测到错误。

If a similar copy-and-paste error were made in conjunction with either of the for loops (for-each or traditional), the resulting code wouldn’t even compile. The element (or iterator) variable from the first loop would not be in scope in the second loop. Here’s how it looks with the traditional for loop:

如果将类似的复制粘贴错误发生在 for 循环（for-each 循环或传统循环），则生成的代码甚至无法编译。对于第二个循环，第一个循环中的（或 iterator）变量已经不在作用域中。下面是它与传统 for 循环的样子：

```Java
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
Element e = i.next();
... // Do something with e and i
}
...
// Compile-time error - cannot find symbol i
for (Iterator<Element> i2 = c2.iterator(); i.hasNext(); ) {
Element e2 = i2.next();
... // Do something with e2 and i2
}
```

Moreover, if you use a for loop, it’s much less likely that you’ll make the copy-and-paste error because there’s no incentive to use different variable names in the two loops. The loops are completely independent, so there’s no harm in reusing the element (or iterator) variable name. In fact, it’s often stylish to do so. The for loop has one more advantage over the while loop: it is shorter, which enhances readability. Here is another loop idiom that minimizes the scope of local variables:

此外，如果你使用 for 循环，那么发生复制粘贴错误的可能性要小得多，因为这两种循环中没有使用不同变量名称的动机。循环是完全独立的，所以复用循环（或 iterator）变量名没有害处。事实上，这样做通常很流行。for 循环相比 while 循环还有一个优点：它更短，这增强了可读性。下面是另一个循环习惯用法，它也最小化了局部变量的范围：

```Java
for (int i = 0, n = expensiveComputation(); i < n; i++) {
    ... // Do something with i;
}
```

The important thing to notice about this idiom is that it has two loop variables, i and n, both of which have exactly the right scope. The second variable, n, is used to store the limit of the first, thus avoiding the cost of a redundant computation in every iteration. As a rule, you should use this idiom if the loop test involves a method invocation that is guaranteed to return the same result on each iteration.

关于这个用法需要注意的重要一点是，它有两个循环变量，i 和 n，它们都具有完全正确的作用域。第二个变量 n 用于存储第一个变量的极限，从而避免了每次迭代中冗余计算的成本。作为一个规则，如果循环测试涉及一个方法调用，并且保证在每次迭代中返回相同的结果，那么应该使用这个习惯用法。

A final technique to minimize the scope of local variables is to keep methods small and focused. If you combine two activities in the same method, local variables relevant to one activity may be in the scope of the code performing the other activity. To prevent this from happening, simply separate the method into two: one for each activity.

最小化局部变量范围的最后一种技术是保持方法小而集中。如果在同一方法中合并两个操作，与一个操作相关的局部变量可能位于执行另一个操作的代码的范围内。为了防止这种情况发生，只需将方法分成两个部分：每个操作一个。

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-9/Chapter-9-Introduction.md)**
- **Next Item（下一条目）：[Item 58: Prefer for-each loops to traditional for loops（for-each 循环优于传统的 for 循环）](/Chapter-9/Chapter-9-Item-58-Prefer-for-each-loops-to-traditional-for-loops.md)**
