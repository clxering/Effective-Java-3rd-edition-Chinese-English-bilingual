## Chapter 2. Creating and Destroying Objects（创建和销毁对象）

### Item 9: Prefer try-with-resources to try-finally（使用try-with-resources优于try-finally）

The Java libraries include many resources that must be closed manually（adv.手动地） by invoking a close method. Examples include InputStream,OutputStream, and java.sql.Connection. Closing resources is often overlooked（n.被忽视的；v.忽视） by clients, with predictably（adv.可以预见的是） dire performance consequences. While many of these resources use finalizers as a safety net, finalizers don’t work very well (Item 8).

Java库包含许多必须通过调用close方法手动关闭的资源。常见的有InputStream、OutputStream和java.sql.Connection。关闭资源常常会被客户忽略，这会导致可怕的性能后果。虽然这些资源中的许多都使用终结器作为安全网，但终结器并不能很好地工作（[Item-8](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-2-Item-8-Avoid-finalizers-and-cleaners.md)）。

Historically, a try-finally statement was the best way to guarantee that a resource would be closed properly, even in the face of an exception or return:

从历史上看，try-finally语句是确保正确关闭资源的最佳方法，即使在出现异常或返回时也是如此：

```
// try-finally - No longer the best way to close resources!
static String firstLineOfFile(String path) throws IOException {
BufferedReader br = new BufferedReader(new FileReader(path));
try {
return br.readLine();
} finally {
br.close();
}}
```

This may not look bad, but it gets worse when you add a second resource:

这可能看起来不坏，但当您添加第二个资源时，情况会变得更糟：

```
// try-finally is ugly when used with more than one resource!
static void copy(String src, String dst) throws IOException {
InputStream in = new FileInputStream(src);
try {
OutputStream out = new FileOutputStream(dst);
try {
byte[] buf = new byte[BUFFER_SIZE];
int n;
while ((n = in.read(buf)) >= 0)
out.write(buf, 0, n);
} finally {
out.close();
}}
finally {
in.close();
}}
```

It may be hard to believe, but even good programmers got this wrong most of the time. For starters, I got it wrong on page 88 of Java Puzzlers [Bloch05], and no one noticed for years. In fact, two-thirds of the uses of the close method in the Java libraries were wrong in 2007.

这可能难以置信，但即使是优秀的程序员在大多数时候也会犯这种错误。首先，我在第88页的Java谜题[Bloch05]中弄错了，多年来没人注意到。事实上，2007年Java库中三分之二的close方法的使用都是错误的。

Even the correct code for closing resources with try-finally statements,as illustrated（v.阐明） in the previous two code examples, has a subtle deficiency. The code in both the try block and the finally block is capable of throwing exceptions. For example, in the firstLineOfFile method, the call to readLine could throw an exception due to a failure in the underlying physical device, and the call to close could then fail for the same reason. Under these circumstances, the second exception completely obliterates（vt.抹去） the first one. There is no record of the first exception in the exception stack trace, which can greatly complicate debugging in real systems—usually it’s the first exception that you want to see in order to diagnose the problem. While it is possible to write code to suppress the second exception in favor of the first, virtually no one did because it’s just too verbose.

即使使用try-finally语句关闭资源的正确代码（如前两个代码示例所示）也有一个细微的缺陷。try块和finally块中的代码都能够抛出异常。例如，在firstLineOfFile方法中，由于底层物理设备发生故障，对readLine的调用可能会抛出异常，而关闭的调用也可能出于同样的原因而失败。在这种情况下，第二个异常完全覆盖了第一个异常。异常堆栈跟踪中没有第一个异常的记录，这可能会使实际系统中的调试变得非常复杂——通常这是您希望看到的第一个异常，以便诊断问题。虽然可以通过编写代码来抑制第二个异常而支持第一个异常，但实际上没有人这样做，因为它太过冗长。

All of these problems were solved in one fell swoop when Java 7 introduced the try-with-resources statement [JLS, 14.20.3]. To be usable with this construct, a resource must implement the AutoCloseable interface, which consists of a single void-returning close method. Many classes and interfaces in the Java libraries and in third-party libraries now implement or extend AutoCloseable. If you write a class that represents a resource that must be closed, your class should implement AutoCloseable too.

当Java 7引入try-with-resources语句[JLS, 14.20.3]时，所有这些问题都一次性解决了。要使用这个构造，资源必须实现AutoCloseable接口，它由一个单独的void-return close方法组成。Java库和第三方库中的许多类和接口现在都实现或扩展了AutoCloseable。如果您编写的类表示必须关闭的资源，那么类也应该实现AutoCloseable。

Here’s how our first example looks using try-with-resources:

下面是第一个使用try-with-resources的示例：

```
// try-with-resources - the the best way to close resources!
static String firstLineOfFile(String path) throws IOException {
try (BufferedReader br = new BufferedReader(
new FileReader(path))) {
return br.readLine();
}}
```

And here’s how our second example looks using try-with-resources:

下面是使用try-with-resources的第二个示例：

```
// try-with-resources on multiple resources - short and sweet
static void copy(String src, String dst) throws IOException {
try (InputStream in = new FileInputStream(src);
OutputStream out = new FileOutputStream(dst)) {
byte[] buf = new byte[BUFFER_SIZE];
int n;
while ((n = in.read(buf)) >= 0)
out.write(buf, 0, n);
}}
```

originals, but they provide far better diagnostics. Consider the firstLineOfFile method. If exceptions are thrown by both the readLine call and the (invisible) close, the latter exception is suppressed in favor of the former. In fact, multiple exceptions may be suppressed in order to preserve the exception that you actually want to see. These suppressed exceptions are not merely discarded; they are printed in the stack trace with a notation saying that they were suppressed. You can also access them programmatically with the getSuppressed method, which was added to Throwable in Java 7.

（在功能实现上，两者没有差别），但后者（为开发者）提供了更好的诊断。考虑firstLineOfFile方法。如果异常是由readLine调用和（不可见的）close抛出的，则后一个异常将被禁止，以支持前一个异常。实际上，可能会禁止多个异常，以保留您实际希望看到的异常。这些被抑制的异常不仅被抛弃；它们被打印在堆栈跟踪中，标记表示它们被抑制。您还可以通过编程方式使用get方法访问它们，该方法是在Java 7中添加到Throwable中的。

You can put catch clauses on try-with-resources statements, just as you can on regular try-finally statements. This allows you to handle exceptions without sullying your code with another layer of nesting. As a slightly contrived example, here’s a version our firstLineOfFile method that does not throw exceptions, but takes a default value to return if it can’t open the file or read from it:

您可以在带有资源的try-with-resources语句中放置catch子句，就像在常规的try-finally语句上一样。这允许您处理异常，而不会用另一层嵌套影响您的代码。作为一个略显做作的示例，下面是我们的firstLineOfFile方法的一个版本，它不抛出异常，但如果无法打开文件或从中读取文件，则返回一个默认值：

```
// try-with-resources with a catch clause
static String firstLineOfFile(String path, String defaultVal) {
try (BufferedReader br = new BufferedReader(
new FileReader(path))) {
return br.readLine();
} catch (IOException e) {
return defaultVal;
}}
```

The lesson is clear: Always use try-with-resources in preference to tryfinally when working with resources that must be closed. The resulting code is shorter and clearer, and the exceptions that it generates are more useful. The try-with-resources statement makes it easy to write correct code using resources that must be closed, which was practically impossible using tryfinally.

教训很清楚：在使用必须关闭的资源时，总是优先使用try-with-resources，而不是try- finally。生成的代码更短、更清晰，生成的异常更有用。使用try-with-resources语句可以很容易地使用必须关闭的资源编写正确的代码，而使用try- finally几乎是不可能的。
