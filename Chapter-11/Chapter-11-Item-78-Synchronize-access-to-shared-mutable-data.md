## Chapter 11. Concurrency（并发）

### Item 78: Synchronize access to shared mutable data（对共享可变数据的同步访问）

The synchronized keyword ensures that only a single thread can execute a method or block at one time. Many programmers think of synchronization solely as a means of mutual exclusion, to prevent an object from being seen in an inconsistent state by one thread while it’s being modified by another. In this view, an object is created in a consistent state (Item 17) and locked by the methods that access it. These methods observe the state and optionally cause a state transition, transforming the object from one consistent state to another. Proper use of synchronization guarantees that no method will ever observe the object in an inconsistent state.

synchronized 关键字确保一次只有一个线程可以执行一个方法或块。许多程序员认为同步只是一种互斥的方法，是为防止一个线程在另一个线程修改对象时使对象处于不一致的状态。这样看来，对象以一致的状态创建（[Item-17](/Chapter-4/Chapter-4-Item-17-Minimize-mutability.md)），并由访问它的方法锁定。这些方法可以察觉当前状态，并引起状态转换，将对象从一致的状态转换为另一个一致的状态。正确使用同步可以保证没有方法会让对象处于不一致状态。

This view is correct, but it’s only half the story. Without synchronization, one thread’s changes might not be visible to other threads. Not only does synchronization prevent threads from observing an object in an inconsistent state, but it ensures that each thread entering a synchronized method or block sees the effects of all previous modifications that were guarded by the same lock.

这种观点是正确的，但它只是冰山一角。没有同步，一个线程所做的的更改可能对其他线程不可见。同步不仅阻止线程察觉到处于不一致状态的对象，而且确保每个进入同步方法或块的线程都能察觉由同一把锁保护的所有已修改的效果。

The language specification guarantees that reading or writing a variable is atomic unless the variable is of type long or double [JLS, 17.4, 17.7]. In other words, reading a variable other than a long or double is guaranteed to return a value that was stored into that variable by some thread, even if multiple threads modify the variable concurrently and without synchronization.

语言规范保证读取或写入变量是原子性的，除非变量的类型是 long 或 double [JLS, 17.4, 17.7]。换句话说，读取 long 或 double 之外的变量将保证返回某个线程存储在该变量中的值，即使多个线程同时修改该变量，并且没有同步时也是如此。

You may hear it said that to improve performance, you should dispense with synchronization when reading or writing atomic data. This advice is dangerously wrong. While the language specification guarantees that a thread will not see an arbitrary value when reading a field, it does not guarantee that a value written by one thread will be visible to another. **Synchronization is required for reliable communication between threads as well as for mutual exclusion.** This is due to a part of the language specification known as the memory model, which specifies when and how changes made by one thread become visible to others [JLS, 17.4; Goetz06, 16].

你可能听说过，为了提高性能，在读取或写入具有原子性的数据时应该避免同步。这种建议大错特错。虽然语言规范保证线程在读取字段时不会觉察任意值，但它不保证由一个线程编写的值对另一个线程可见。**线程之间能可靠通信以及实施互斥，同步是所必需的。** 这是由于语言规范中，称为内存模型的部分指定了一个线程所做的更改何时以及如何对其他线程可见 [JLS, 17.4; Goetz06, 16]。

The consequences of failing to synchronize access to shared mutable data can be dire even if the data is atomically readable and writable. Consider the task of stopping one thread from another. The libraries provide the Thread.stop method, but this method was deprecated long ago because it is inherently unsafe —its use can result in data corruption. **Do not use Thread.stop.** A recommended way to stop one thread from another is to have the first thread poll a boolean field that is initially false but can be set to true by the second thread to indicate that the first thread is to stop itself. Because reading and writing a boolean field is atomic, some programmers dispense with synchronization when accessing the field:

即使数据是原子可读和可写的，无法同步访问共享可变数据的后果也可能是可怕的。考虑从一个线程中使另一个线程停止的任务。库提供了 `Thread.stop` 方法，但是这个方法很久以前就被弃用了，因为它本质上是不安全的，它的使用可能导致数据损坏。**不要使用 `Thread.stop`。** 一个建议的方法是让第一个线程轮询一个 boolean 字段，该字段最初为 false，但第二个线程可以将其设置为 true，以指示第一个线程要停止它自己。由于读写布尔字段是原子性的，一些程序员在访问该字段时不需要同步：

```Java
// Broken! - How long would you expect this program to run?
public class StopThread {
    private static boolean stopRequested;

    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
        int i = 0;
        while (!stopRequested)
            i++;
        });

    backgroundThread.start();
    TimeUnit.SECONDS.sleep(1);
    stopRequested = true;
    }
}
```

You might expect this program to run for about a second, after which the main thread sets stopRequested to true, causing the background thread’s loop to terminate. On my machine, however, the program never terminates: the background thread loops forever!

你可能认为这个程序运行大约一秒钟，之后主线程将 stopRequested 设置为 true，从而导致后台线程的循环终止。然而，在我的机器上，程序永远不会终止：后台线程永远循环！

The problem is that in the absence of synchronization, there is no guarantee as to when, if ever, the background thread will see the change in the value of stopRequested made by the main thread. In the absence of synchronization, it’s quite acceptable for the virtual machine to transform this code:

问题在于在缺乏同步的情况下，无法保证后台线程何时（如果有的话）看到主线程所做的 stopRequested 值的更改。在缺乏同步的情况下，虚拟机可以很好地转换这段代码：

```Java
while (!stopRequested)
    i++;
into this code:
if (!stopRequested)
    while (true)
        i++;
```

This optimization is known as hoisting, and it is precisely what the OpenJDK Server VM does. The result is a liveness failure: the program fails to make progress. One way to fix the problem is to synchronize access to the stopRequested field. This program terminates in about one second, as expected:

这种优化称为提升，这正是 OpenJDK 服务器 VM 所做的。结果是活性失败：程序无法取得进展。解决此问题的一种方法是同步对 stopRequested 字段的访问。程序在大约一秒内结束，正如预期：

```Java
// Properly synchronized cooperative thread termination
public class StopThread {
    private static boolean stopRequested;

    private static synchronized void requestStop() {
        stopRequested = true;
    }

    private static synchronized boolean stopRequested() {
        return stopRequested;
    }

    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested())
            i++;
        });

        backgroundThread.start();
        TimeUnit.SECONDS.sleep(1);
        requestStop();
    }
}
```

Note that both the write method (requestStop) and the read method (stop-Requested) are synchronized. It is not sufficient to synchronize only the write method! **Synchronization is not guaranteed to work unless both read and write operations are synchronized.** Occasionally a program that synchronizes only writes (or reads) may appear to work on some machines, but in this case, appearances are deceiving.

注意，写方法（requestStop）和读方法（stopRequested）都是同步的。仅同步写方法是不够的！**除非读和写操作都同步，否则不能保证同步工作。** 有时，只同步写（或读）的程序可能在某些机器上显示有效，但在这种情况下，不能这么做。

The actions of the synchronized methods in StopThread would be atomic even without synchronization. In other words, the synchronization on these methods is used solely for its communication effects, not for mutual exclusion. While the cost of synchronizing on each iteration of the loop is small, there is a correct alternative that is less verbose and whose performance is likely to be better. The locking in the second version of StopThread can be omitted if stopRequested is declared volatile. While the volatile modifier performs no mutual exclusion, it guarantees that any thread that reads the field will see the most recently written value:

即使没有同步，StopThread 中同步方法的操作也是原子性的。换句话说，这些方法的同步仅用于其通信效果，而不是互斥。虽然在循环的每个迭代上同步的成本很小，但是有一种正确的替代方法，它不那么冗长，而且性能可能更好。如果 stopRequested 声明为 volatile，则可以省略 StopThread 的第二个版本中的锁。虽然 volatile 修饰符不执行互斥，但它保证任何读取字段的线程都会看到最近写入的值：

```Java
// Cooperative thread termination with a volatile field
public class StopThread {
    private static volatile boolean stopRequested;

    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
        int i = 0;
        while (!stopRequested)
            i++;
    });

    backgroundThread.start();
    TimeUnit.SECONDS.sleep(1);
    stopRequested = true;
    }
}
```

You do have to be careful when using volatile. Consider the following method, which is supposed to generate serial numbers:

在使用 volatile 时一定要小心。考虑下面的方法，它应该生成序列号：

```Java
// Broken - requires synchronization!
private static volatile int nextSerialNumber = 0;

public static int generateSerialNumber() {
    return nextSerialNumber++;
}
```

The intent of the method is to guarantee that every invocation returns a unique value (so long as there are no more than 2<sup>32</sup> invocations). The method’s state consists of a single atomically accessible field, nextSerialNumber, and all possible values of this field are legal. Therefore, no synchronization is necessary to protect its invariants. Still, the method won’t work properly without synchronization.

该方法的目的是确保每次调用返回一个唯一的值（只要不超过 2<sup>32</sup> 次调用）。方法的状态由一个原子可访问的字段 nextSerialNumber 组成，该字段的所有可能值都是合法的。因此，不需要同步来保护它的不变性。不过，如果没有同步，该方法将无法正常工作。

The problem is that the increment operator (++) is not atomic. It performs two operations on the nextSerialNumber field: first it reads the value, and then it writes back a new value, equal to the old value plus one. If a second thread reads the field between the time a thread reads the old value and writes back a new one, the second thread will see the same value as the first and return the same serial number. This is a safety failure: the program computes the wrong results.

问题在于增量运算符 `(++)` 不是原子性的。它对 nextSerialNumber 字段执行两个操作：首先读取值，然后返回一个新值，旧值再加 1。如果第二个线程在读取旧值和写入新值之间读取字段，则第二个线程将看到与第一个线程相同的值，并返回相同的序列号。这是一个安全故障：使程序计算错误的原因。

One way to fix generateSerialNumber is to add the synchronized modifier to its declaration. This ensures that multiple invocations won’t be interleaved and that each invocation of the method will see the effects of all previous invocations. Once you’ve done that, you can and should remove the volatile modifier from nextSerialNumber. To bulletproof the method, use long instead of int, or throw an exception if nextSerialNumber is about to wrap.

修复 generateSerialNumber 的一种方法是将 synchronized 修饰符添加到它的声明中。这确保了多个调用不会交叉，并且该方法的每次调用都将看到之前所有调用的效果。一旦你这样做了，你就可以并且应该从 nextSerialNumber 中删除 volatile 修饰符。为了使方法更可靠，应使用 long 而不是 int，或者在 nextSerialNumber 即将超限时抛出异常。

Better still, follow the advice in Item 59 and use the class AtomicLong, which is part of java.util.concurrent.atomic. This package provides primitives for lock-free, thread-safe programming on single variables. While volatile provides only the communication effects of synchronization, this package also provides atomicity. This is exactly what we want for generateSerialNumber, and it is likely to outperform the synchronized version:

更好的方法是，遵循 [Item-59](/Chapter-9/Chapter-9-Item-59-Know-and-use-the-libraries.md) 中的建议并使用 AtomicLong 类，它是 `java.util.concurrent.atomic` 的一部分。这个包为单变量的无锁、线程安全编程提供了基本类型。虽然 volatile 只提供同步的通信效果，但是这个包提供原子性。这正是我们想要的 generateSerialNumber，它很可能优于同步版本：

```Java
// Lock-free synchronization with java.util.concurrent.atomic
private static final AtomicLong nextSerialNum = new AtomicLong();

public static long generateSerialNumber() {
    return nextSerialNum.getAndIncrement();
}
```

The best way to avoid the problems discussed in this item is not to share mutable data. Either share immutable data (Item 17) or don’t share at all. In other words, **confine mutable data to a single thread.** If you adopt this policy, it is important to document it so that the policy is maintained as your program evolves. It is also important to have a deep understanding of the frameworks and libraries you’re using because they may introduce threads that you are unaware of.

为避免出现本条目中讨论的问题，最佳方法是不共享可变数据。要么共享不可变数据（[Item-17](/Chapter-4/Chapter-4-Item-17-Minimize-mutability.md)），要么完全不共享。换句话说，**应当将可变数据限制在一个线程中。** 如果采用此策略，重要的是对其进行文档化，以便随着程序的发展维护该策略。深入了解你正在使用的框架和库也很重要，因为它们可能会引入你不知道的线程。

It is acceptable for one thread to modify a data object for a while and then to share it with other threads, synchronizing only the act of sharing the object reference. Other threads can then read the object without further synchronization, so long as it isn’t modified again. Such objects are said to be effectively immutable [Goetz06, 3.5.4]. Transferring such an object reference from one thread to others is called safe publication [Goetz06, 3.5.3]. There are many ways to safely publish an object reference: you can store it in a static field as part of class initialization; you can store it in a volatile field, a final field, or a field that is accessed with normal locking; or you can put it into a concurrent collection (Item 81).

一个线程可以暂时修改一个数据对象，然后与其他线程共享，并且只同步共享对象引用的操作。然后，其他线程可以在没有进一步同步的情况下读取对象，只要不再次修改该对象。这些对象被认为是有效不可变的 [Goetz06, 3.5.4]。将这样的对象引用从一个线程转移到其他线程称为安全发布 [Goetz06, 3.5.3]。安全地发布对象引用的方法有很多：可以将它存储在静态字段中，作为类初始化的一部分；你可以将其存储在易失性字段、final 字段或使用普通锁定访问的字段中；或者你可以将其放入并发集合中（[Item-81](/Chapter-11/Chapter-11-Item-81-Prefer-concurrency-utilities-to-wait-and-notify.md)）。

In summary, **when multiple threads share mutable data, each thread that reads or writes the data must perform synchronization.** In the absence of synchronization, there is no guarantee that one thread’s changes will be visible to another thread. The penalties for failing to synchronize shared mutable data are liveness and safety failures. These failures are among the most difficult to debug. They can be intermittent and timing-dependent, and program behavior can vary radically from one VM to another. If you need only inter-thread communication, and not mutual exclusion, the volatile modifier is an acceptable form of synchronization, but it can be tricky to use correctly.

总之，**当多个线程共享可变数据时，每个读取或写入数据的线程都必须执行同步。** 在缺乏同步的情况下，不能保证一个线程的更改对另一个线程可见。同步共享可变数据失败的代价是活性失败和安全失败。这些故障是最难调试的故障之一。它们可能是间歇性的，并与时间相关，而且程序行为可能在不同 VM 之间发生根本的变化。如果只需要线程间通信，而不需要互斥，那么 volatile 修饰符是一种可接受的同步形式，但是要想正确使用它可能会比较棘手。

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-11/Chapter-11-Introduction.md)**
- **Previous Item（上一条目）：[Item 77: Don’t ignore exceptions（不要忽略异常）](/Chapter-10/Chapter-10-Item-77-Don’t-ignore-exceptions.md)**
- **Next Item（下一条目）：[Item 79: Avoid excessive synchronization（避免过度同步）](/Chapter-11/Chapter-11-Item-79-Avoid-excessive-synchronization.md)**
