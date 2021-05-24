## Chapter 11. Concurrency（并发）

### Item 79: Avoid excessive synchronization（避免过度同步）

Item 78 warns of the dangers of insufficient synchronization. This item concerns the opposite problem. Depending on the situation, excessive synchronization can cause reduced performance, deadlock, or even nondeterministic behavior.

[Item-78](/Chapter-11/Chapter-11-Item-78-Synchronize-access-to-shared-mutable-data.md) 警告我们同步不到位的危险。本条目涉及相反的问题。根据不同的情况，过度的同步可能导致性能下降、死锁甚至不确定行为。

**To avoid liveness and safety failures, never cede control to the client within a synchronized method or block.** In other words, inside a synchronized region, do not invoke a method that is designed to be overridden, or one provided by a client in the form of a function object (Item 24). From the perspective of the class with the synchronized region, such methods are alien. The class has no knowledge of what the method does and has no control over it. Depending on what an alien method does, calling it from a synchronized region can cause exceptions, deadlocks, or data corruption.

**为避免活性失败和安全故障，永远不要在同步方法或块中将控制权交给客户端。** 换句话说，在同步区域内，不要调用一个设计为被覆盖的方法，或者一个由客户端以函数对象的形式提供的方法（[Item-24](/Chapter-4/Chapter-4-Item-24-Favor-static-member-classes-over-nonstatic.md)）。从具有同步区域的类的角度来看，这种方法是不一样的。类不知道该方法做什么，也无法控制它。Depending on what an alien method does，从同步区域调用它可能会导致异常、死锁或数据损坏。

To make this concrete, consider the following class, which implements an observable set wrapper. It allows clients to subscribe to notifications when elements are added to the set. This is the Observer pattern [Gamma95]. For brevity’s sake, the class does not provide notifications when elements are removed from the set, but it would be a simple matter to provide them. This class is implemented atop the reusable ForwardingSet from Item 18 (page 90):

要使这个问题具体化，请考虑下面的类，它实现了一个可视 Set 包装器。当元素被添加到集合中时，它允许客户端订阅通知。这是观察者模式 [Gamma95]。为了简单起见，当元素从集合中删除时，该类不提供通知，即使要提供通知也很简单。这个类是在 [Item-18](/Chapter-4/Chapter-4-Item-18-Favor-composition-over-inheritance.md)（第 90 页）的可复用 ForwardingSet 上实现的：

```Java
// Broken - invokes alien method from synchronized block!
public class ObservableSet<E> extends ForwardingSet<E> {
    public ObservableSet(Set<E> set) { super(set); }

    private final List<SetObserver<E>> observers= new ArrayList<>();

    public void addObserver(SetObserver<E> observer) {
        synchronized(observers) {
            observers.add(observer);
        }
    }

    public boolean removeObserver(SetObserver<E> observer) {
        synchronized(observers) {
            return observers.remove(observer);
        }
    }

    private void notifyElementAdded(E element) {
        synchronized(observers) {
            for (SetObserver<E> observer : observers)
                observer.added(this, element);
        }
    }

    @Override
    public boolean add(E element) {
        boolean added = super.add(element);
        if (added)
            notifyElementAdded(element);
        return added;
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (E element : c)
            result |= add(element); // Calls notifyElementAdded
        return result;
    }
}
```

Observers subscribe to notifications by invoking the addObserver method and unsubscribe by invoking the removeObserver method. In both cases, an instance of this callback interface is passed to the method.

观察者通过调用 addObserver 方法订阅通知，通过调用 removeObserver 方法取消订阅。在这两种情况下，都会将此回调接口的实例传递给方法。

```Java
@FunctionalInterface
public interface SetObserver<E> {
    // Invoked when an element is added to the observable set
    void added(ObservableSet<E> set, E element);
}
```

This interface is structurally identical to `BiConsumer<ObservableSet<E>,E>`. We chose to define a custom functional interface because the interface and method names make the code more readable and because the interface could evolve to incorporate multiple callbacks. That said, a reasonable argument could also be made for using BiConsumer (Item 44).

这个接口在结构上与 `BiConsumer<ObservableSet<E>,E>` 相同。我们选择定义一个自定义函数式接口，因为接口和方法名称使代码更具可读性，而且接口可以演化为包含多个回调。也就是说，使用 BiConsumer 也是合理的（[Item-44](/Chapter-7/Chapter-7-Item-44-Favor-the-use-of-standard-functional-interfaces.md)）。

On cursory inspection, ObservableSet appears to work fine. For example, the following program prints the numbers from 0 through 99:

粗略地检查一下，ObservableSet 似乎工作得很好。例如，下面的程序打印从 0 到 99 的数字：

```Java
public static void main(String[] args) {
    ObservableSet<Integer> set =new ObservableSet<>(new HashSet<>());
    set.addObserver((s, e) -> System.out.println(e));
    for (int i = 0; i < 100; i++)
        set.add(i);
}
```

Now let’s try something a bit fancier. Suppose we replace the addObserver call with one that passes an observer that prints the Integer value that was added to the set and removes itself if the value is 23:

现在让我们尝试一些更奇特的东西。假设我们将 addObserver 调用替换为一个传递观察者的调用，该观察者打印添加到集合中的整数值，如果该值为 23，则该调用将删除自身：

```Java
set.addObserver(new SetObserver<>() {
    public void added(ObservableSet<Integer> s, Integer e) {
        System.out.println(e);
        if (e == 23)
            s.removeObserver(this);
    }
});
```

Note that this call uses an anonymous class instance in place of the lambda used in the previous call. That is because the function object needs to pass itself to s.removeObserver, and lambdas cannot access themselves (Item 42).

注意，这个调用使用一个匿名类实例来代替前面调用中使用的 lambda 表达式。这是因为函数对象需要将自己传递给 `s.removeObserver`，而 lambda 表达式不能访问自身（[Item-42](/Chapter-7/Chapter-7-Item-42-Prefer-lambdas-to-anonymous-classes.md)）。

You might expect the program to print the numbers 0 through 23, after which the observer would unsubscribe and the program would terminate silently. In fact, it prints these numbers and then throws a ConcurrentModificationException. The problem is that notifyElementAdded is in the process of iterating over the observers list when it invokes the observer’s added method. The added method calls the observable set’s removeObserver method, which in turn calls the method observers.remove. Now we’re in trouble. We are trying to remove an element from a list in the midst of iterating over it, which is illegal. The iteration in the notifyElementAdded method is in a synchronized block to prevent concurrent modification, but it doesn’t prevent the iterating thread itself from calling back into the observable set and modifying its observers list.

你可能希望程序打印数字 0 到 23，然后观察者将取消订阅，程序将无声地终止。实际上，它打印这些数字，然后抛出 ConcurrentModificationException。问题在于 notifyElementAdded 在调用观察者的 added 方法时，正在遍历 observers 列表。added 方法调用可观察集的 removeObserver 方法，该方法反过来调用方法 `observers.remove`。现在我们有麻烦了。我们试图在遍历列表的过程中从列表中删除一个元素，这是非法的。notifyElementAdded 方法中的迭代位于一个同步块中，以防止并发修改，但是无法防止迭代线程本身回调到可观察的集合中，也无法防止修改它的 observers 列表。

Now let’s try something odd: let’s write an observer that tries to unsubscribe, but instead of calling removeObserver directly, it engages the services of another thread to do the deed. This observer uses an executor service (Item 80):

现在让我们尝试一些奇怪的事情：让我们编写一个观察者来尝试取消订阅，但是它没有直接调用 removeObserver，而是使用另一个线程的服务来执行这个操作。该观察者使用 executor 服务（[Item-80](/Chapter-11/Chapter-11-Item-80-Prefer-executors,-tasks,-and-streams-to-threads.md)）：

```Java
// Observer that uses a background thread needlessly
set.addObserver(new SetObserver<>() {
    public void added(ObservableSet<Integer> s, Integer e) {
        System.out.println(e);
        if (e == 23) {
            ExecutorService exec = Executors.newSingleThreadExecutor();
            try {
                exec.submit(() -> s.removeObserver(this)).get();
            } catch (ExecutionException | InterruptedException ex) {
                throw new AssertionError(ex);
            } finally {
                exec.shutdown();
            }
        }
    }
});
```

Incidentally, note that this program catches two different exception types in one catch clause. This facility, informally known as multi-catch, was added in Java 7. It can greatly increase the clarity and reduce the size of programs that behave the same way in response to multiple exception types.

顺便提一下，注意这个程序在一个 catch 子句中捕获了两种不同的异常类型。这个功能在 Java 7 中添加了，非正式名称为 multi-catch。它可以极大地提高清晰度，并减少在响应多种异常类型时表现相同的程序的大小。

When we run this program, we don’t get an exception; we get a deadlock. The background thread calls s.removeObserver, which attempts to lock observers, but it can’t acquire the lock, because the main thread already has the lock. All the while, the main thread is waiting for the background thread to finish removing the observer, which explains the deadlock.

当我们运行这个程序时，我们不会得到异常；而是遭遇了死锁。后台线程调用 `s.removeObserver`，它试图锁定观察者，但无法获取锁，因为主线程已经拥有锁。一直以来，主线程都在等待后台线程完成删除观察者的操作，这就解释了死锁的原因。

This example is contrived because there is no reason for the observer to use a background thread to unsubscribe itself, but the problem is real. Invoking alien methods from within synchronized regions has caused many deadlocks in real systems, such as GUI toolkits.

这个例子是人为设计的，因为观察者没有理由使用后台线程来取消订阅本身，但是问题是真实的。在实际系统中，从同步区域内调用外来方法会导致许多死锁，比如 GUI 工具包。

In both of the previous examples (the exception and the deadlock) we were lucky. The resource that was guarded by the synchronized region (observers) was in a consistent state when the alien method (added) was invoked. Suppose you were to invoke an alien method from a synchronized region while the invariant protected by the synchronized region was temporarily invalid. Because locks in the Java programming language are reentrant, such calls won’t deadlock. As in the first example, which resulted in an exception, the calling thread already holds the lock, so the thread will succeed when it tries to reacquire the lock, even though another conceptually unrelated operation is in progress on the data guarded by the lock. The consequences of such a failure can be catastrophic. In essence, the lock has failed to do its job. Reentrant locks simplify the construction of multithreaded object-oriented programs, but they can turn liveness failures into safety failures.

在前面的两个例子中（异常和死锁），我们都很幸运。调用外来方法（added）时，由同步区域（观察者）保护的资源处于一致状态。假设你要从同步区域调用一个外来方法，而同步区域保护的不变量暂时无效。因为 Java 编程语言中的锁是可重入的，所以这样的调用不会死锁。与第一个导致异常的示例一样，调用线程已经持有锁，所以当它试图重新获得锁时，线程将成功，即使另一个概念上不相关的操作正在对锁保护的数据进行中。这种失败的后果可能是灾难性的。从本质上说，这把锁没能发挥它的作用。可重入锁简化了多线程面向对象程序的构造，但它们可以将活动故障转化为安全故障。

Luckily, it is usually not too hard to fix this sort of problem by moving alien method invocations out of synchronized blocks. For the notifyElementAdded method, this involves taking a “snapshot” of the observers list that can then be safely traversed without a lock. With this change, both of the previous examples run without exception or deadlock:

幸运的是，通过将外来方法调用移出同步块来解决这类问题通常并不难。对于 notifyElementAdded 方法，这涉及到获取观察者列表的「快照」，然后可以在没有锁的情况下安全地遍历该列表。有了这个改变，前面的两个例子都可以再也不会出现异常或者死锁了：

```Java
// Alien method moved outside of synchronized block - open calls
private void notifyElementAdded(E element) {
    List<SetObserver<E>> snapshot = null;
    synchronized(observers) {
        snapshot = new ArrayList<>(observers);
    }
    for (SetObserver<E> observer :snapshot)
        observer.added(this, element);
}
```

In fact, there’s a better way to move the alien method invocations out of the synchronized block. The libraries provide a concurrent collection (Item 81) known as CopyOnWriteArrayList that is tailor-made for this purpose. This List implementation is a variant of ArrayList in which all modification operations are implemented by making a fresh copy of the entire underlying array. Because the internal array is never modified, iteration requires no locking and is very fast. For most uses, the performance of CopyOnWriteArrayList would be atrocious, but it’s perfect for observer lists, which are rarely modified and often traversed.

实际上，有一种更好的方法可以将外来方法调用移出同步块。库提供了一个名为 CopyOnWriteArrayList 的并发集合（[Item-81](/Chapter-11/Chapter-11-Item-81-Prefer-concurrency-utilities-to-wait-and-notify.md)），该集合是为此目的量身定制的。此列表实现是 ArrayList 的变体，其中所有修改操作都是通过复制整个底层数组来实现的。因为从不修改内部数组，所以迭代不需要锁定，而且速度非常快。如果大量使用，CopyOnWriteArrayList 的性能会很差，但是对于很少修改和经常遍历的观察者列表来说，它是完美的。

The add and addAll methods of ObservableSet need not be changed if the list is modified to use CopyOnWriteArrayList. Here is how the remainder of the class looks. Notice that there is no explicit synchronization whatsoever:

如果将 list 修改为使用 CopyOnWriteArrayList，则不需要更改 ObservableSet 的 add 和 addAll 方法。下面是类的其余部分。请注意，没有任何显式同步：

```Java
// Thread-safe observable set with CopyOnWriteArrayList
private final List<SetObserver<E>> observers =new CopyOnWriteArrayList<>();

public void addObserver(SetObserver<E> observer) {
    observers.add(observer);
}

public boolean removeObserver(SetObserver<E> observer) {
    return observers.remove(observer);
}

private void notifyElementAdded(E element) {
    for (SetObserver<E> observer : observers)
        observer.added(this, element);
}
```

An alien method invoked outside of a synchronized region is known as an open call [Goetz06, 10.1.4]. Besides preventing failures, open calls can greatly increase concurrency. An alien method might run for an arbitrarily long period. If the alien method were invoked from a synchronized region, other threads would be denied access to the protected resource unnecessarily.

在同步区域之外调用的外来方法称为 open call [Goetz06, 10.1.4]。除了防止失败之外，开放调用还可以极大地提高并发性。一个陌生的方法可以运行任意长的时间。如果从同步区域调用了外来方法，其他线程对受保护资源的访问就会遭到不必要的拒绝。

**As a rule, you should do as little work as possible inside synchronized regions.** Obtain the lock, examine the shared data, transform it as necessary, and drop the lock. If you must perform some time-consuming activity, find a way to move it out of the synchronized region without violating the guidelines in Item 78.

**作为规则，你应该在同步区域内做尽可能少的工作。** 获取锁，检查共享数据，根据需要进行转换，然后删除锁。如果你必须执行一些耗时的活动，请设法将其移出同步区域，而不违反 [Item-78](/Chapter-11/Chapter-11-Item-78-Synchronize-access-to-shared-mutable-data.md) 中的指导原则。

The first part of this item was about correctness. Now let’s take a brief look at performance. While the cost of synchronization has plummeted since the early days of Java, it is more important than ever not to oversynchronize. In a multicore world, the real cost of excessive synchronization is not the CPU time spent getting locks; it is contention: the lost opportunities for parallelism and the delays imposed by the need to ensure that every core has a consistent view of memory. Another hidden cost of oversynchronization is that it can limit the VM’s ability to optimize code execution.

本条目的第一部分是关于正确性的。现在让我们简要地看一下性能。虽然自 Java 早期以来，同步的成本已经大幅下降，但比以往任何时候都更重要的是：不要过度同步。在多核世界中，过度同步的真正代价不是获得锁所花费的 CPU 时间；这是一种争论：而是失去了并行化的机会，以及由于需要确保每个核心都有一个一致的内存视图而造成的延迟。过度同步的另一个隐藏成本是，它可能限制 VM 优化代码执行的能力。

If you are writing a mutable class, you have two options: you can omit all synchronization and allow the client to synchronize externally if concurrent use is desired, or you can synchronize internally, making the class thread-safe (Item 82). You should choose the latter option only if you can achieve significantly higher concurrency with internal synchronization than you could by having the client lock the entire object externally. The collections in java.util (with the exception of the obsolete Vector and Hashtable) take the former approach, while those in java.util.concurrent take the latter (Item 81).

如果你正在编写一个可变的类，你有两个选择：你可以省略所有同步并允许客户端在需要并发使用时在外部进行同步，或者你可以在内部进行同步，从而使类是线程安全的（[Item-82](/Chapter-11/Chapter-11-Item-82-Document-thread-safety.md)）。只有当你能够通过内部同步实现比通过让客户端在外部锁定整个对象获得高得多的并发性时，才应该选择后者。`java.util` 中的集合（废弃的 Vector 和 Hashtable 除外）采用前一种方法，而 `java.util.concurrent` 中的方法则采用后者（[Item-81](/Chapter-11/Chapter-11-Item-81-Prefer-concurrency-utilities-to-wait-and-notify.md)）。

In the early days of Java, many classes violated these guidelines. For example, StringBuffer instances are almost always used by a single thread, yet they perform internal synchronization. It is for this reason that StringBuffer was supplanted by StringBuilder, which is just an unsynchronized StringBuffer. Similarly, it’s a large part of the reason that the thread-safe pseudorandom number generator in java.util.Random was supplanted by the unsynchronized implementation in java.util.concurrent.ThreadLocalRandom. When in doubt, do not synchronize your class, but document that it is not thread-safe.

在 Java 的早期，许多类违反了这些准则。例如，StringBuffer 实例几乎总是由一个线程使用，但是它们执行内部同步。正是由于这个原因，StringBuffer 被 StringBuilder 取代，而 StringBuilder 只是一个未同步的 StringBuffer。类似地，同样，`java.util.Random` 中的线程安全伪随机数生成器被 `java.util.concurrent.ThreadLocalRandom` 中的非同步实现所取代，这也是原因之一。如果有疑问，不要同步你的类，但要记录它不是线程安全的。

If you do synchronize your class internally, you can use various techniques to achieve high concurrency, such as lock splitting, lock striping, and nonblocking concurrency control. These techniques are beyond the scope of this book, but they are discussed elsewhere [Goetz06, Herlihy08].

如果你在内部同步你的类，你可以使用各种技术来实现高并发性，例如分拆锁、分离锁和非阻塞并发控制。这些技术超出了本书的范围，但是在其他地方也有讨论 [Goetz06, Herlihy08]。

If a method modifies a static field and there is any possibility that the method will be called from multiple threads, you must synchronize access to the field internally (unless the class can tolerate nondeterministic behavior). It is not possible for a multithreaded client to perform external synchronization on such a method, because unrelated clients can invoke the method without synchronization. The field is essentially a global variable even if it is private because it can be read and modified by unrelated clients. The nextSerialNumber field used by the method generateSerialNumber in Item 78 exemplifies this situation.

如果一个方法修改了一个静态字段，并且有可能从多个线程调用该方法，则必须在内部同步对该字段的访问（除非该类能够容忍不确定性行为）。多线程客户端不可能对这样的方法执行外部同步，因为不相关的客户端可以在不同步的情况下调用该方法。字段本质上是一个全局变量，即使它是私有的，因为它可以被不相关的客户端读取和修改。[Item-78](/Chapter-11/Chapter-11-Item-78-Synchronize-access-to-shared-mutable-data.md) 中的 generateSerialNumber 方法使用的 nextSerialNumber 字段演示了这种情况。

In summary, to avoid deadlock and data corruption, never call an alien method from within a synchronized region. More generally, keep the amount of work that you do from within synchronized regions to a minimum. When you are designing a mutable class, think about whether it should do its own synchronization. In the multicore era, it is more important than ever not to oversynchronize. Synchronize your class internally only if there is a good reason to do so, and document your decision clearly (Item 82).

总之，为了避免死锁和数据损坏，永远不要从同步区域内调用外来方法。更一般地说，将你在同步区域内所做的工作量保持在最小。在设计可变类时，请考虑它是否应该执行自己的同步。在多核时代，比以往任何时候都更重要的是不要过度同步。只有在有充分理由时，才在内部同步类，并清楚地记录你的决定（[Item-82](/Chapter-11/Chapter-11-Item-82-Document-thread-safety.md)）。

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-11/Chapter-11-Introduction.md)**
- **Previous Item（上一条目）：[Item 78: Synchronize access to shared mutable data（对共享可变数据的同步访问）](/Chapter-11/Chapter-11-Item-78-Synchronize-access-to-shared-mutable-data.md)**
- **Next Item（下一条目）：[Item 80: Prefer executors, tasks, and streams to threads（Executor、task、流优于直接使用线程）](/Chapter-11/Chapter-11-Item-80-Prefer-executors,-tasks,-and-streams-to-threads.md)**
