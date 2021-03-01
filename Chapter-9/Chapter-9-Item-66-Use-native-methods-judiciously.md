## Chapter 9. General Programming（通用程序设计）

### Item 66: Use native methods judiciously（明智地使用本地方法）

The Java Native Interface (JNI) allows Java programs to call native methods, which are methods written in native programming languages such as C or C++. Historically, native methods have had three main uses. They provide access to platform-specific facilities such as registries. They provide access to existing libraries of native code, including legacy libraries that provide access to legacy data. Finally, native methods are used to write performance-critical parts of applications in native languages for improved performance.

Java 本地接口（JNI）允许 Java 程序调用本地方法，这些方法是用 C 或 C++ 等本地编程语言编写的。从历史上看，本地方法主要有三种用途。它们提供对特定于平台的设施（如注册中心）的访问。它们提供对现有本地代码库的访问，包括提供对遗留数据访问。最后，本地方法可以通过本地语言编写应用程序中注重性能的部分，以提高性能。

It is legitimate to use native methods to access platform-specific facilities, but it is seldom necessary: as the Java platform matured, it provided access to many features previously found only in host platforms. For example, the process API, added in Java 9, provides access to OS processes. It is also legitimate to use native methods to use native libraries when no equivalent libraries are available in Java.

使用本地方法访问特定于平台的机制是合法的，但是很少有必要：随着 Java 平台的成熟，它提供了对许多以前只能在宿主平台中上找到的特性。例如，Java 9 中添加的流 API 提供了对 OS 流程的访问。在 Java 中没有等效库时，使用本地方法来使用本地库也是合法的。

**It is rarely advisable to use native methods for improved performance.** In early releases (prior to Java 3), it was often necessary, but JVMs have gotten much faster since then. For most tasks, it is now possible to obtain comparable performance in Java. For example, when java.math was added in release 1.1, BigInteger relied on a then-fast multiprecision arithmetic library written in C. In Java 3, BigInteger was reimplemented in Java, and carefully tuned to the point where it ran faster than the original native implementation.

**为了提高性能，很少建议使用本地方法。** 在早期版本（Java 3 之前），这通常是必要的，但是从那时起 JVM 变得更快了。对于大多数任务，现在可以在 Java 中获得类似的性能。例如，在版本 1.1 中添加了 `java.math`，BigInteger 是在一个用 C 编写的快速多精度运算库的基础上实现的。在当时，为了获得足够的性能这样做是必要的。在 Java 3 中，BigInteger 则完全用 Java 重写了，并且进行了性能调优，新的版本比原来的版本更快。

A sad coda to this story is that BigInteger has changed little since then, with the exception of faster multiplication for large numbers in Java 8. In that time, work continued apace on native libraries, notably GNU Multiple Precision arithmetic library (GMP). Java programmers in need of truly high-performance multiprecision arithmetic are now justified in using GMP via native methods [Blum14].

这个故事的一个可悲的结尾是，除了在 Java 8 中对大数进行更快的乘法运算之外，BigInteger 此后几乎没有发生什么变化。在此期间，对本地库的工作继续快速进行，尤其是 GNU 多精度算术库（GMP）。需要真正高性能多精度算法的 Java 程序员现在可以通过本地方法使用 GMP [Blum14]。

The use of native methods has serious disadvantages. Because native languages are not safe (Item 50), applications using native methods are no longer immune to memory corruption errors. Because native languages are more platform-dependent than Java, programs using native methods are less portable. They are also harder to debug. If you aren’t careful, native methods can decrease performance because the garbage collector can’t automate, or even track, native memory usage (Item 8), and there is a cost associated with going into and out of native code. Finally, native methods require “glue code” that is difficult to read and tedious to write.

使用本地方法有严重的缺点。由于本地语言不安全（[Item-50](/Chapter-8/Chapter-8-Item-50-Make-defensive-copies-when-needed.md)），使用本地方法的应用程序不再能免受内存毁坏错误的影响。由于本地语言比 Java 更依赖于平台，因此使用本地方法的程序的可移植性较差。它们也更难调试。如果不小心，本地方法可能会降低性能，因为垃圾收集器无法自动跟踪本地内存使用情况（[Item-8](/Chapter-2/Chapter-2-Item-8-Avoid-finalizers-and-cleaners.md)），而且进出本地代码会产生相关的成本。最后，本地方法需要「粘合代码」，这很难阅读，而且编写起来很乏味。

In summary, think twice before using native methods. It is rare that you need to use them for improved performance. If you must use native methods to access low-level resources or native libraries, use as little native code as possible and test it thoroughly. A single bug in the native code can corrupt your entire application.

总之，在使用本地方法之前要三思。一般很少需要使用它们来提高性能。如果必须使用本地方法来访问底层资源或本地库，请尽可能少地使用本地代码，并对其进行彻底的测试。本地代码中的一个错误就可以破坏整个应用程序。

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-9/Chapter-9-Introduction.md)**
- **Previous Item（上一条目）：[Item 65: Prefer interfaces to reflection（接口优于反射）](/Chapter-9/Chapter-9-Item-65-Prefer-interfaces-to-reflection.md)**
- **Next Item（下一条目）：[Item 67: Optimize judiciously（明智地进行优化）](/Chapter-9/Chapter-9-Item-67-Optimize-judiciously.md)**
