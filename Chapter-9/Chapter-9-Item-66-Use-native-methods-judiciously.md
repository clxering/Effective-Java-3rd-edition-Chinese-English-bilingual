## Chapter 9. General Programming（通用程序设计）

### Item 66: Use native methods judiciously

The Java Native Interface (JNI) allows Java programs to call native methods, which are methods written in native programming languages such as C or C++. Historically, native methods have had three main uses. They provide access to platform-specific facilities such as registries. They provide access to existing libraries of native code, including legacy libraries that provide access to legacy data. Finally, native methods are used to write performance-critical parts of applications in native languages for improved performance.

It is legitimate to use native methods to access platform-specific facilities, but it is seldom necessary: as the Java platform matured, it provided access to many features previously found only in host platforms. For example, the process API, added in Java 9, provides access to OS processes. It is also legitimate to use native methods to use native libraries when no equivalent libraries are available in Java.

**It is rarely advisable to use native methods for improved performance.** In early releases (prior to Java 3), it was often necessary, but JVMs have gotten much faster since then. For most tasks, it is now possible to obtain comparable performance in Java. For example, when java.math was added in release 1.1, BigInteger relied on a then-fast multiprecision arithmetic library written in C. In Java 3, BigInteger was reimplemented in Java, and carefully tuned to the point where it ran faster than the original native implementation.

A sad coda to this story is that BigInteger has changed little since then, with the exception of faster multiplication for large numbers in Java 8. In that time, work continued apace on native libraries, notably GNU Multiple Precision arithmetic library (GMP). Java programmers in need of truly high-performance multiprecision arithmetic are now justified in using GMP via native methods [Blum14].

The use of native methods has serious disadvantages. Because native languages are not safe (Item 50), applications using native methods are no longer immune to memory corruption errors. Because native languages are more platform-dependent than Java, programs using native methods are less portable. They are also harder to debug. If you aren’t careful, native methods can decrease performance because the garbage collector can’t automate, or even track, native memory usage (Item 8), and there is a cost associated with going into and out of native code. Finally, native methods require “glue code” that is difficult to read and tedious to write.

In summary, think twice before using native methods. It is rare that you need to use them for improved performance. If you must use native methods to access low-level resources or native libraries, use as little native code as possible and test it thoroughly. A single bug in the native code can corrupt your entire application.

---
**[Back to contents of the chapter（返回章节目录）](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-9/Chapter-9-Introduction.md)**
- **Previous Item（上一条目）：[Item 65: Prefer interfaces to reflection](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-9/Chapter-9-Item-65-Prefer-interfaces-to-reflection.md)**
- **Next Item（下一条目）：[Item 67: Optimize judiciously](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-9/Chapter-9-Item-67-Optimize-judiciously.md)**
