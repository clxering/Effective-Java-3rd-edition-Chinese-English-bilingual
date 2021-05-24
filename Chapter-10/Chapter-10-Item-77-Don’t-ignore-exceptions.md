## Chapter 10. Exceptions（异常）

### Item 77: Don’t ignore exceptions（不要忽略异常）

While this advice may seem obvious, it is violated often enough that it bears repeating. When the designers of an API declare a method to throw an exception, they are trying to tell you something. Don’t ignore it! It is easy to ignore exceptions by surrounding a method invocation with a try statement whose catch block is empty:

虽然这一建议似乎显而易见，但它经常被违反，因此值得强调。当 API 的设计人员声明一个抛出异常的方法时，他们试图告诉你一些事情。不要忽略它！如果在方法调用的周围加上一条 try 语句，其 catch 块为空，可以很容易忽略异常：

```Java
// Empty catch block ignores exception - Highly suspect!
try {
    ...
}
catch (SomeException e) {
}
```

**An empty catch block defeats the purpose of exceptions,** which is to force you to handle exceptional conditions. Ignoring an exception is analogous to ignoring a fire alarm—and turning it off so no one else gets a chance to see if there’s a real fire. You may get away with it, or the results may be disastrous. Whenever you see an empty catch block, alarm bells should go off in your head.

**空 catch 块违背了异常的目的，** 它的存在是为了强制你处理异常情况。忽略异常类似于忽略火灾警报一样，关掉它之后，其他人就没有机会看到是否真的发生了火灾。你可能侥幸逃脱，但结果可能是灾难性的。每当你看到一个空的 catch 块，你的脑海中应该响起警报。

There are situations where it is appropriate to ignore an exception. For example, it might be appropriate when closing a FileInputStream. You haven’t changed the state of the file, so there’s no need to perform any recovery action, and you’ve already read the information that you need from the file, so there’s no reason to abort the operation in progress. It may be wise to log the exception, so that you can investigate the matter if these exceptions happen often. **If you choose to ignore an exception, the catch block should contain a comment explaining why it is appropriate to do so, and the variable should be named ignored:**

在某些情况下，忽略异常是合适的。例如，在关闭 FileInputStream 时，忽略异常可能是合适的。你没有更改文件的状态，因此不需要执行任何恢复操作，并且已经从文件中读取了所需的信息，因此没有理由中止正在进行的操作。记录异常可能是明智的，这样如果这些异常经常发生，你应该研究起因。**如果你选择忽略异常，catch 块应该包含一条注释，解释为什么这样做是合适的，并且应该将变量命名为 ignore：**

```Java
Future<Integer> f = exec.submit(planarMap::chromaticNumber);
int numColors = 4; // Default; guaranteed sufficient for any map
try {
    numColors = f.get(1L, TimeUnit.SECONDS);
}
catch (TimeoutException | ExecutionException ignored) {
    // Use default: minimal coloring is desirable, not required
}
```

The advice in this item applies equally to checked and unchecked exceptions. Whether an exception represents a predictable exceptional condition or a programming error, ignoring it with an empty catch block will result in a program that continues silently in the face of error. The program might then fail at an arbitrary time in the future, at a point in the code that bears no apparent relation to the source of the problem. Properly handling an exception can avert failure entirely. Merely letting an exception propagate outward can at least cause the program to fail swiftly, preserving information to aid in debugging the failure.

本条目中的建议同样适用于 checked 异常和 unchecked异常。不管异常是表示可预测的异常条件还是编程错误，用空 catch 块忽略它将导致程序在错误面前保持静默。然后，程序可能会在未来的任意时间点，在与问题源没有明显关系的代码中失败。正确处理异常可以完全避免失败。仅仅让异常向外传播，可能会导致程序走向失败，保留信息有利于调试。

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-10/Chapter-10-Introduction.md)**
- **Previous Item（上一条目）：[Item 76: Strive for failure atomicity（尽力保证故障原子性）](/Chapter-10/Chapter-10-Item-76-Strive-for-failure-atomicity.md)**
- **Next Item（下一条目）：[Chapter 11 Introduction（章节介绍）](/Chapter-11/Chapter-11-Introduction.md)**
