## Chapter 11. Concurrency（并发）

### Item 81: Prefer concurrency utilities to wait and notify（并发实用工具优于 wait 和 notify）

The first edition of this book devoted an item to the correct use of wait and notify [Bloch01, Item 50]. Its advice is still valid and is summarized at end of this item, but this advice is far less important than it once was. This is because there is far less reason to use wait and notify. Since Java 5, the platform has provided higher-level concurrency utilities that do the sorts of things you formerly had to hand-code atop wait and notify. **Given the difficulty of using wait and notify correctly, you should use the higher-level concurrency utilities instead.**

这本书的第一版专门介绍了 wait 和 notify 的正确用法 [Bloch01, item 50]。这些建议仍然有效，并在本条目末尾作了总结，但这一建议已远不如从前重要。这是因为使用 wait 和 notify 的理由要少得多。自 Java 5 以来，该平台提供了更高级别的并发实用工具，可以执行以前必须在 wait 和 notify 上手工编写代码的操作。**考虑到正确使用 wait 和 notify 的困难，你应该使用更高级别的并发实用工具。**

The higher-level utilities in java.util.concurrent fall into three categories: the Executor Framework, which was covered briefly in Item 80; concurrent collections; and synchronizers. Concurrent collections and synchronizers are covered briefly in this item.

`java.util.concurrent` 中级别较高的实用工具可分为三类：Executor 框架，[Item-80](/Chapter-11/Chapter-11-Item-80-Prefer-executors,-tasks,-and-streams-to-threads.md) 简要介绍了该框架；并发集合；同步器。本条目简要介绍并发集合和同步器。

The concurrent collections are high-performance concurrent implementations of standard collection interfaces such as List, Queue, and Map. To provide high concurrency, these implementations manage their own synchronization internally (Item 79). Therefore, **it is impossible to exclude concurrent activity from a concurrent collection; locking it will only slow the program.**

并发集合是标准集合接口，如 List、Queue 和 Map 的高性能并发实现。为了提供高并发性，这些实现在内部管理它们自己的同步（[Item-79](/Chapter-11/Chapter-11-Item-79-Avoid-excessive-synchronization.md)）。因此，**不可能从并发集合中排除并发活动；锁定它只会使程序变慢。**

Because you can’t exclude concurrent activity on concurrent collections, you can’t atomically compose method invocations on them either. Therefore, concurrent collection interfaces were outfitted with state-dependent modify operations, which combine several primitives into a single atomic operation. These operations proved sufficiently useful on concurrent collections that they were added to the corresponding collection interfaces in Java 8, using default methods (Item 21).

因为不能排除并发集合上的并发活动，所以也不能原子地组合对它们的方法调用。因此，并发集合接口配备了依赖于状态的修改操作，这些操作将多个基本操作组合成单个原子操作。这些操作在并发集合上非常有用，因此使用默认方法（[Item-21](/Chapter-4/Chapter-4-Item-21-Design-interfaces-for-posterity.md)）将它们添加到 Java 8 中相应的集合接口。

For example, Map’s putIfAbsent(key, value) method inserts a mapping for a key if none was present and returns the previous value associated with the key, or null if there was none. This makes it easy to implement thread-safe canonicalizing maps. This method simulates the behavior of String.intern:

例如，Map 的 `putIfAbsent(key, value)` 方法为一个没有映射的键插入一个映射，并返回与键关联的前一个值，如果没有，则返回 null。这使得实现线程安全的规范化 Map 变得很容易。这个方法模拟了 `String.intern` 的行为。

```Java
// Concurrent canonicalizing map atop ConcurrentMap - not optimal
private static final ConcurrentMap<String, String> map =new ConcurrentHashMap<>();
public static String intern(String s) {
    String previousValue = map.putIfAbsent(s, s);
    return previousValue == null ? s : previousValue;
}
```

In fact, you can do even better. ConcurrentHashMap is optimized for retrieval operations, such as get. Therefore, it is worth invoking get initially and calling putIfAbsent only if get indicates that it is necessary:

事实上，你可以做得更好。ConcurrentHashMap 针对 get 等检索操作进行了优化。因此，只有在 get 表明有必要时，才值得首先调用 get 再调用 putIfAbsent:

```Java
// Concurrent canonicalizing map atop ConcurrentMap - faster!
public static String intern(String s) {
    String result = map.get(s);
    if (result == null) {
        result = map.putIfAbsent(s, s);
        if (result == null)
        result = s;
    }
    return result;
}
```

Besides offering excellent concurrency, ConcurrentHashMap is very fast. On my machine, the intern method above is over six times faster than String.intern (but keep in mind that String.intern must employ some strategy to keep from leaking memory in a long-lived application). Concurrent collections make synchronized collections largely obsolete. For example, **use ConcurrentHashMap in preference to Collections.synchronizedMap.** Simply replacing synchronized maps with concurrent maps can dramatically increase the performance of concurrent applications.

除了提供优秀的并发性，ConcurrentHashMap 还非常快。在我的机器上，上面的 intern 方法比 `String.intern` 快六倍多（但是请记住，`String.intern` 必须使用一些策略来防止在长时间运行的应用程序中内存泄漏）。并发集合使同步集合在很大程度上过时。例如，**使用 ConcurrentHashMap 而不是 `Collections.synchronizedMap`。** 只要用并发 Map 替换同步 Map 就可以显著提高并发应用程序的性能。

Some of the collection interfaces were extended with blocking operations, which wait (or block) until they can be successfully performed. For example, BlockingQueue extends Queue and adds several methods, including take, which removes and returns the head element from the queue, waiting if the queue is empty. This allows blocking queues to be used for work queues (also known as producer-consumer queues), to which one or more producer threads enqueue work items and from which one or more consumer threads dequeue and process items as they become available. As you’d expect, most ExecutorService implementations, including ThreadPoolExecutor, use a BlockingQueue (Item 80).

一些集合接口使用阻塞操作进行了扩展，这些操作将等待（或阻塞）成功执行。例如，BlockingQueue 扩展了 Queue 并添加了几个方法，包括 take，它从队列中删除并返回首个元素，如果队列为空，则等待。这允许将阻塞队列用于工作队列（也称为生产者-消费者队列），一个或多个生产者线程将工作项添加到该工作队列中，一个或多个消费者线程将工作项从该工作队列中取出并在这些工作项可用时处理它们。正如你所期望的，大多数 ExecutorService 实现，包括 ThreadPoolExecutor，都使用 BlockingQueue（[Item-80](/Chapter-11/Chapter-11-Item-80-Prefer-executors,-tasks,-and-streams-to-threads.md)）。

Synchronizers are objects that enable threads to wait for one another, allowing them to coordinate their activities. The most commonly used synchronizers are CountDownLatch and Semaphore. Less commonly used are CyclicBarrier and Exchanger. The most powerful synchronizer is Phaser.

同步器是允许线程彼此等待的对象，允许它们协调各自的活动。最常用的同步器是 CountDownLatch 和 Semaphore。较不常用的是 CyclicBarrier 和 Exchanger。最强大的同步器是 Phaser。

Countdown latches are single-use barriers that allow one or more threads to wait for one or more other threads to do something. The sole constructor for CountDownLatch takes an int that is the number of times the countDown method must be invoked on the latch before all waiting threads are allowed to proceed.

Countdown latches are single-use barriers，允许一个或多个线程等待一个或多个其他线程执行某些操作。CountDownLatch 的惟一构造函数接受一个 int，这个 int 是在允许所有等待的线程继续之前，必须在锁存器上调用倒计时方法的次数。

It is surprisingly easy to build useful things atop this simple primitive. For example, suppose you want to build a simple framework for timing the concurrent execution of an action. This framework consists of a single method that takes an executor to execute the action, a concurrency level representing the number of actions to be executed concurrently, and a runnable representing the action. All of the worker threads ready themselves to run the action before the timer thread starts the clock. When the last worker thread is ready to run the action, the timer thread “fires the starting gun,” allowing the worker threads to perform the action. As soon as the last worker thread finishes performing the action, the timer thread stops the clock. Implementing this logic directly on top of wait and notify would be messy to say the least, but it is surprisingly straightforward on top of CountDownLatch:

在这个简单的基本类型上构建有用的东西非常容易。例如，假设你想要构建一个简单的框架来为一个操作的并发执行计时。这个框架由一个方法组成，该方法使用一个 executor 来执行操作，一个并发级别表示要并发执行的操作的数量，一个 runnable 表示操作。所有工作线程都准备在 timer 线程启动时钟之前运行操作。当最后一个工作线程准备好运行该操作时，计时器线程「发令枪」，允许工作线程执行该操作。一旦最后一个工作线程完成该操作，计时器线程就停止时钟。在 wait 和 notify 的基础上直接实现这种逻辑至少会有点麻烦，但是在 CountDownLatch 的基础上实现起来却非常简单：

```Java
// Simple framework for timing concurrent execution
public static long time(Executor executor, int concurrency,Runnable action) throws InterruptedException {
    CountDownLatch ready = new CountDownLatch(concurrency);
    CountDownLatch start = new CountDownLatch(1);
    CountDownLatch done = new CountDownLatch(concurrency);

    for (int i = 0; i < concurrency; i++) {
        executor.execute(() -> {
            ready.countDown(); // Tell timer we're ready
            try {
                start.await(); // Wait till peers are ready
                action.run();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } finally {
                done.countDown(); // Tell timer we're done
            }
        });
    }

    ready.await(); // Wait for all workers to be ready
    long startNanos = System.nanoTime();
    start.countDown(); // And they're off!
    done.await(); // Wait for all workers to finish
    return System.nanoTime() - startNanos;
}
```

Note that the method uses three countdown latches. The first, ready, is used by worker threads to tell the timer thread when they’re ready. The worker threads then wait on the second latch, which is start. When the last worker thread invokes ready.countDown, the timer thread records the start time and invokes start.countDown, allowing all of the worker threads to proceed. Then the timer thread waits on the third latch, done, until the last of the worker threads finishes running the action and calls done.countDown. As soon as this happens, the timer thread awakens and records the end time.

注意，该方法使用三个倒计时锁。第一个是 ready，工作线程使用它来告诉 timer 线程它们什么时候准备好了。工作线程然后等待第二个锁存器，即 start。当最后一个工作线程调用 `ready.countDown` 时。timer 线程记录开始时间并调用 `start.countDown`，允许所有工作线程继续。然后计时器线程等待第三个锁存器 done，直到最后一个工作线程运行完操作并调用 `done.countDown`。一旦发生这种情况，timer 线程就会唤醒并记录结束时间。

A few more details bear noting. The executor passed to the time method must allow for the creation of at least as many threads as the given concurrency level, or the test will never complete. This is known as a thread starvation deadlock [Goetz06, 8.1.1]. If a worker thread catches an InterruptedException, it reasserts the interrupt using the idiom Thread.currentThread().interrupt() and returns from its run method. This allows the executor to deal with the interrupt as it sees fit. Note that System.nanoTime is used to time the activity. **For interval timing, always use System.nanoTime rather than System.currentTimeMillis.** System.nanoTime is both more accurate and more precise and is unaffected by adjustments to the system’s realtime clock. Finally, note that the code in this example won’t yield accurate timings unless action does a fair amount of work, say a second or more. Accurate microbenchmarking is notoriously hard and is best done with the aid of a specialized framework such as jmh [JMH].

还有一些细节值得注意。传递给 time 方法的 executor 必须允许创建至少与给定并发级别相同数量的线程，否则测试将永远不会完成。这被称为线程饥饿死锁 [Goetz06, 8.1.1]。如果工作线程捕捉到 InterruptedException，它使用习惯用法 `Thread.currentThread().interrupt()` 重申中断，并从它的 run 方法返回。这允许执行程序按照它认为合适的方式处理中断。请注意，`System.nanoTime` 是用来计时的。**对于间隔计时，始终使用 `System.nanoTime` 而不是 `System.currentTimeMillis`。** `System.nanoTime` 不仅更准确，而且更精确，而且不受系统实时时钟调整的影响。最后，请注意，本例中的代码不会产生准确的计时，除非 action 做了相当多的工作，比如一秒钟或更长时间。准确的微基准测试是出了名的困难，最好是借助诸如 jmh 这样的专业框架来完成。

This item only scratches the surface of what you can do with the concurrency utilities. For example, the three countdown latches in the previous example could be replaced by a single CyclicBarrier or Phaser instance. The resulting code would be a bit more concise but perhaps more difficult to understand.

本条目只涉及到你可以使用并发实用工具做什么。例如，前面示例中的三个倒计时锁存器可以替换为单个 CyclicBarrier 或 Phaser 实例。生成的代码可能更简洁，但可能更难于理解。

While you should always use the concurrency utilities in preference to wait and notify, you might have to maintain legacy code that uses wait and notify. The wait method is used to make a thread wait for some condition. It must be invoked inside a synchronized region that locks the object on which it is invoked. Here is the standard idiom for using the wait method:

虽然你应该始终优先使用并发实用工具，而不是使用 wait 和 notify，但是你可能必须维护使用 wait 和 notify 的遗留代码。wait 方法用于使线程等待某个条件。它必须在同步区域内调用，该同步区域将锁定调用它的对象。下面是使用 wait 方法的标准形式：

```Java
// The standard idiom for using the wait method
synchronized (obj) {
    while (<condition does not hold>)
        obj.wait(); // (Releases lock, and reacquires on wakeup)
    ... // Perform action appropriate to condition
}
```

**Always use the wait loop idiom to invoke the wait method; never invoke it outside of a loop.** The loop serves to test the condition before and after waiting.

**始终使用 wait 习惯用法，即循环来调用 wait 方法；永远不要在循环之外调用它。** 循环用于在等待之前和之后测试条件。

Testing the condition before waiting and skipping the wait if the condition already holds are necessary to ensure liveness. If the condition already holds and the notify (or notifyAll) method has already been invoked before a thread waits, there is no guarantee that the thread will ever wake from the wait.

在等待之前测试条件，如果条件已经存在，则跳过等待，以确保活性。如果条件已经存在，并且在线程等待之前已经调用了 notify（或 notifyAll）方法，则不能保证线程将从等待中唤醒。

Testing the condition after waiting and waiting again if the condition does not hold are necessary to ensure safety. If the thread proceeds with the action when the condition does not hold, it can destroy the invariant guarded by the lock. There are several reasons a thread might wake up when the condition does not hold:

为了确保安全，需要在等待之后再测试条件，如果条件不成立，则再次等待。如果线程在条件不成立的情况下继续执行该操作，它可能会破坏由锁保护的不变性。当条件不成立时，有一些理由唤醒线程：

- Another thread could have obtained the lock and changed the guarded state between the time a thread invoked notify and the waiting thread woke up.

另一个线程可以获得锁，并在线程调用 notify 和等待线程醒来之间更改保护状态。

- Another thread could have invoked notify accidentally or maliciously when the condition did not hold. Classes expose themselves to this sort of mischief by waiting on publicly accessible objects. Any wait in a synchronized method of a publicly accessible object is susceptible to this problem.

当条件不成立时，另一个线程可能意外地或恶意地调用 notify。类通过等待公共可访问的对象来暴露自己。公共可访问对象的同步方法中的任何 wait 都容易受到这个问题的影响。

- The notifying thread could be overly “generous” in waking waiting threads. For example, the notifying thread might invoke notifyAll even if only some of the waiting threads have their condition satisfied.

通知线程在唤醒等待线程时可能过于「慷慨」。例如，即使只有一些等待线程的条件得到满足，通知线程也可能调用 notifyAll。

- The waiting thread could (rarely) wake up in the absence of a notify. This is known as a spurious wakeup [POSIX, 11.4.3.6.1; Java9-api].

在没有通知的情况下，等待的线程可能（很少）醒来。这被称为伪唤醒 [POSIX, 11.4.3.6.1; Java9-api]。

A related issue is whether to use notify or notifyAll to wake waiting threads. (Recall that notify wakes a single waiting thread, assuming such a thread exists, and notifyAll wakes all waiting threads.) It is sometimes said that you should always use notifyAll. This is reasonable, conservative advice. It will always yield correct results because it guarantees that you’ll wake the threads that need to be awakened. You may wake some other threads, too, but this won’t affect the correctness of your program. These threads will check the condition for which they’re waiting and, finding it false, will continue waiting.

一个相关的问题是，是使用 notify 还是 notifyAll 来唤醒等待的线程。（回想一下 notify 唤醒一个等待线程，假设存在这样一个线程，notifyAll 唤醒所有等待线程)。有时人们会说，应该始终使用 notifyAll。这是合理的、保守的建议。它总是会产生正确的结果，因为它保证你将唤醒需要唤醒的线程。你可能还会唤醒其他一些线程，但这不会影响程序的正确性。这些线程将检查它们正在等待的条件，如果发现为条件不满足，将继续等待。

As an optimization, you may choose to invoke notify instead of notifyAll if all threads that could be in the wait-set are waiting for the same condition and only one thread at a time can benefit from the condition becoming true.

作为一种优化，如果在等待状态的所有线程都在等待相同的条件，并且每次只有一个线程可以从条件中获益，那么你可以选择调用 notify 而不是 notifyAll。

Even if these preconditions are satisfied, there may be cause to use notifyAll in place of notify. Just as placing the wait invocation in a loop protects against accidental or malicious notifications on a publicly accessible object, using notifyAll in place of notify protects against accidental or malicious waits by an unrelated thread. Such waits could otherwise “swallow” a critical notification, leaving its intended recipient waiting indefinitely.

即使满足了这些先决条件，也可能有理由使用 notifyAll 来代替 notify。正如将 wait 调用放在循环中可以防止公共访问对象上的意外或恶意通知一样，使用 notifyAll 代替 notify 可以防止不相关线程的意外或恶意等待。否则，这样的等待可能会「吞下」一个关键通知，让预期的接收者无限期地等待。

In summary, using wait and notify directly is like programming in “concurrency assembly language,” as compared to the higher-level language provided by java.util.concurrent. **There is seldom, if ever, a reason to use wait and notify in new code.** If you maintain code that uses wait and notify, make sure that it always invokes wait from within a while loop using the standard idiom. The notifyAll method should generally be used in preference to notify. If notify is used, great care must be taken to ensure liveness.

总之，与 `java.util.concurrent` 提供的高级语言相比，直接使用 wait 和 notify 就像使用「并发汇编语言」编程一样原始。**在新代码中很少有理由使用 wait 和 notify。** 如果维护使用 wait 和 notify 的代码，请确保它始终使用标准的习惯用法，即在 while 循环中调用 wait。另外，notifyAll 方法通常应该优先于 notify。如果使用 notify，则必须非常小心以确保其活性。

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-11/Chapter-11-Introduction.md)**
- **Previous Item（上一条目）：[Item 80: Prefer executors, tasks, and streams to threads（Executor、task、流优于直接使用线程）](/Chapter-11/Chapter-11-Item-80-Prefer-executors,-tasks,-and-streams-to-threads.md)**
- **Next Item（下一条目）：[Item 82: Document thread safety（文档应包含线程安全属性）](/Chapter-11/Chapter-11-Item-82-Document-thread-safety.md)**
