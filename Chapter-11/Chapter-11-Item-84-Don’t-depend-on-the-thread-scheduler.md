## Chapter 11. Concurrency（并发）

### Item 84: Don’t depend on the thread scheduler（不要依赖线程调度器）

When many threads are runnable, the thread scheduler determines which ones get to run and for how long. Any reasonable operating system will try to make this determination fairly, but the policy can vary. Therefore, well-written programs shouldn’t depend on the details of this policy. **Any program that relies on the thread scheduler for correctness or performance is likely to be nonportable.**

当许多线程可以运行时，线程调度器决定哪些线程可以运行以及运行多长时间。任何合理的操作系统都会尝试公平地做出这个决定，但是策略可能会有所不同。因此，编写良好的程序不应该依赖于此策略的细节。**任何依赖线程调度器来保证正确性或性能的程序都可能是不可移植的。**

The best way to write a robust, responsive, portable program is to ensure that the average number of runnable threads is not significantly greater than the number of processors. This leaves the thread scheduler with little choice: it simply runs the runnable threads till they’re no longer runnable. The program’s behavior doesn’t vary too much, even under radically different thread-scheduling policies. Note that the number of runnable threads isn’t the same as the total number of threads, which can be much higher. Threads that are waiting are not runnable.

编写健壮、响应快、可移植程序的最佳方法是确保可运行线程的平均数量不显著大于处理器的数量。这使得线程调度器几乎没有选择：它只运行可运行线程，直到它们不再可运行为止。即使在完全不同的线程调度策略下，程序的行为也没有太大的变化。注意，可运行线程的数量与线程总数不相同，后者可能更高。正在等待的线程不可运行。

The main technique for keeping the number of runnable threads low is to have each thread do some useful work, and then wait for more. **Threads should not run if they aren’t doing useful work.** In terms of the Executor Framework (Item 80), this means sizing thread pools appropriately [Goetz06, 8.2] and keeping tasks short, but not too short, or dispatching overhead will harm performance.

保持可运行线程数量低的主要技术是让每个线程做一些有用的工作，然后等待更多的工作。**如果线程没有做有用的工作，它们就不应该运行。** 对于 Executor 框架（[Item-80](/Chapter-11/Chapter-11-Item-80-Prefer-executors,-tasks,-and-streams-to-threads.md)），这意味着适当调整线程池的大小 [Goetz06, 8.2]，并保持任务短小（但不要太短），否则分派开销依然会损害性能。

Threads should not busy-wait, repeatedly checking a shared object waiting for its state to change. Besides making the program vulnerable to the vagaries of the thread scheduler, busy-waiting greatly increases the load on the processor, reducing the amount of useful work that others can accomplish. As an extreme example of what not to do, consider this perverse reimplementation of CountDownLatch:

线程不应该处于循环检查共享对象状态变化。除了使程序容易受到线程调度器变化无常的影响之外，循环检查状态变化还大大增加了处理器的负载，还影响其他线程获取处理器进行工作。作为反面的极端例子，考虑一下 CountDownLatch 的不正确的重构实现：

```
// Awful CountDownLatch implementation - busy-waits incessantly!
public class SlowCountDownLatch {

    private int count;

    public SlowCountDownLatch(int count) {
        if (count < 0)
            throw new IllegalArgumentException(count + " < 0");
        this.count = count;
    }

    public void await() {
        while (true) {
            synchronized(this) {
                if (count == 0)
                return;
            }
        }
    }

    public synchronized void countDown() {
        if (count != 0)
            count--;
    }
}
```

On my machine, SlowCountDownLatch is about ten times slower than Java’s CountDownLatch when 1,000 threads wait on a latch. While this example may seem a bit far-fetched, it’s not uncommon to see systems with one or more threads that are unnecessarily runnable. Performance and portability are likely to suffer.

在我的机器上，当 1000 个线程等待一个锁存器时，SlowCountDownLatch 的速度大约是 Java 的 CountDownLatch 的 10 倍。虽然这个例子看起来有点牵强，但是具有一个或多个不必要运行的线程的系统并不少见。性能和可移植性可能会受到影响。

When faced with a program that barely works because some threads aren’t getting enough CPU time relative to others, **resist the temptation to “fix” the program by putting in calls to Thread.yield.** You may succeed in getting the program to work after a fashion, but it will not be portable. The same yield invocations that improve performance on one JVM implementation might make it worse on a second and have no effect on a third. **Thread.yield has no testable semantics.** A better course of action is to restructure the application to reduce the number of concurrently runnable threads.

当面对一个几乎不能工作的程序时，而原因是由于某些线程相对于其他线程没有获得足够的 CPU 时间，那么 **通过调用 `Thread.yield` 来「修复」程序** 你也许能勉强让程序运行起来，但它是不可移植的。在一个 JVM 实现上提高性能的相同的 yield 调用，在一些JVM 实现上可能会使性能变差，而在其他 JVM 实现上可能没有任何影响。**`Thread.yield` 没有可测试的语义。** 更好的做法是重构应用程序，以减少并发运行线程的数量。

A related technique, to which similar caveats apply, is adjusting thread priorities. **Thread priorities are among the least portable features of Java.** It is not unreasonable to tune the responsiveness of an application by tweaking a few thread priorities, but it is rarely necessary and is not portable. It is unreasonable to attempt to solve a serious liveness problem by adjusting thread priorities. The problem is likely to return until you find and fix the underlying cause.

一个相关的技术是调整线程优先级，类似的警告也适用于此技术，即，线程优先级是 Java 中最不可移植的特性之一。通过调整线程优先级来调优应用程序的响应性并非不合理，但很少情况下是必要的，而且不可移植。试图通过调整线程优先级来解决严重的活性问题是不合理的。在找到并修复潜在原因之前，问题很可能会再次出现。

In summary, do not depend on the thread scheduler for the correctness of your program. The resulting program will be neither robust nor portable. As a corollary, do not rely on Thread.yield or thread priorities. These facilities are merely hints to the scheduler. Thread priorities may be used sparingly to improve the quality of service of an already working program, but they should never be used to “fix” a program that barely works.

总之，不要依赖线程调度器来判断程序的正确性。生成的程序既不健壮也不可移植。因此，不要依赖 `Thread.yield` 或线程优先级。这些工具只是对调度器的提示。线程优先级可以少量地用于提高已经工作的程序的服务质量，但绝不应该用于「修复」几乎不能工作的程序。

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-11/Chapter-11-Introduction.md)**
- **Previous Item（上一条目）：[Item 83: Use lazy initialization judiciously（明智地使用延迟初始化）](/Chapter-11/Chapter-11-Item-83-Use-lazy-initialization-judiciously.md)**
- **Next Item（下一条目）：[Chapter 12 Introduction（章节介绍）](/Chapter-12/Chapter-12-Introduction.md)**
