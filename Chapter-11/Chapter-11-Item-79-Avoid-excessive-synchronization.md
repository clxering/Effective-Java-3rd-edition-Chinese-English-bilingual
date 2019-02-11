## Chapter 11. Concurrency（并发）

### Item 79: Avoid excessive synchronization

Item 78 warns of the dangers of insufficient synchronization. This item concerns the opposite problem. Depending on the situation, excessive synchronization can cause reduced performance, deadlock, or even nondeterministic behavior.

**To avoid liveness and safety failures, never cede control to the client within a synchronized method or block.** In other words, inside a synchronized region, do not invoke a method that is designed to be overridden, or one provided by a client in the form of a function object (Item 24). From the perspective of the class with the synchronized region, such methods are alien. The class has no knowledge of what the method does and has no control over it. Depending on what an alien method does, calling it from a synchronized region can cause exceptions, deadlocks, or data corruption.

To make this concrete, consider the following class, which implements an observable set wrapper. It allows clients to subscribe to notifications when elements are added to the set. This is the Observer pattern [Gamma95]. For brevity’s sake, the class does not provide notifications when elements are removed from the set, but it would be a simple matter to provide them. This class is implemented atop the reusable ForwardingSet from Item 18 (page 90):

```
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

```
@FunctionalInterface public interface SetObserver<E> {
// Invoked when an element is added to the observable set
void added(ObservableSet<E> set, E element);
}
```

This interface is structurally identical to `BiConsumer<ObservableSet<E>,E>`. We chose to define a custom functional interface because the interface and method names make the code more readable and because the interface could evolve to incorporate multiple callbacks. That said, a reasonable argument could also be made for using BiConsumer (Item 44).

On cursory inspection, ObservableSet appears to work fine. For example, the following program prints the numbers from 0 through 99:

```
public static void main(String[] args) {
    ObservableSet<Integer> set =new ObservableSet<>(new HashSet<>());
    set.addObserver((s, e) -> System.out.println(e));
    for (int i = 0; i < 100; i++)
        set.add(i);
}
```

Now let’s try something a bit fancier. Suppose we replace the addObserver call with one that passes an observer that prints the Integer value that was added to the set and removes itself if the value is 23:

```
set.addObserver(new SetObserver<>() {
    public void added(ObservableSet<Integer> s, Integer e) {
        System.out.println(e);
        if (e == 23)
            s.removeObserver(this);
    }
});
```

Note that this call uses an anonymous class instance in place of the lambda used in the previous call. That is because the function object needs to pass itself to s.removeObserver, and lambdas cannot access themselves (Item 42).

You might expect the program to print the numbers 0 through 23, after which the observer would unsubscribe and the program would terminate silently. In fact, it prints these numbers and then throws a ConcurrentModificationException. The problem is that notifyElementAdded is in the process of iterating over the observers list when it invokes the observer’s added method. The added method calls the observable set’s removeObserver method, which in turn calls the method observers.remove. Now we’re in trouble. We are trying to remove an element from a list in the midst of iterating over it, which is illegal. The iteration in the notifyElementAdded method is in a synchronized block to prevent concurrent modification, but it doesn’t prevent the iterating thread itself from calling back into the observable set and modifying its observers list.

Now let’s try something odd: let’s write an observer that tries to unsubscribe, but instead of calling removeObserver directly, it engages the services of another thread to do the deed. This observer uses an executor service (Item 80):

```
// Observer that uses a background thread needlessly
set.addObserver(new SetObserver<>() {
    public void added(ObservableSet<Integer> s, Integer e) {
        System.out.println(e);
        if (e == 23) {
            ExecutorService exec =Executors.newSingleThreadExecutor();
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

When we run this program, we don’t get an exception; we get a deadlock. The background thread calls s.removeObserver, which attempts to lock observers, but it can’t acquire the lock, because the main thread already has the lock. All the while, the main thread is waiting for the background thread to finish removing the observer, which explains the deadlock.

This example is contrived because there is no reason for the observer to use a background thread to unsubscribe itself, but the problem is real. Invoking alien methods from within synchronized regions has caused many deadlocks in real systems, such as GUI toolkits.

In both of the previous examples (the exception and the deadlock) we were lucky. The resource that was guarded by the synchronized region (observers) was in a consistent state when the alien method (added) was invoked. Suppose you were to invoke an alien method from a synchronized region while the invariant protected by the synchronized region was temporarily invalid. Because locks in the Java programming language are reentrant, such calls won’t deadlock. As in the first example, which resulted in an exception, the calling thread already holds the lock, so the thread will succeed when it tries to reacquire the lock, even though another conceptually unrelated operation is in progress on the data guarded by the lock. The consequences of such a failure can be catastrophic. In essence, the lock has failed to do its job. Reentrant locks simplify the construction of multithreaded object-oriented programs, but they can turn liveness failures into safety failures.

Luckily, it is usually not too hard to fix this sort of problem by moving alien method invocations out of synchronized blocks. For the notifyElementAdded method, this involves taking a “snapshot” of the observers list that can then be safely traversed without a lock. With this change, both of the previous examples run without exception or deadlock:

```
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

The add and addAll methods of ObservableSet need not be changed if the list is modified to use CopyOnWriteArrayList. Here is how the remainder of the class looks. Notice that there is no explicit synchronization whatsoever:

```
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

**As a rule, you should do as little work as possible inside synchronized regions.** Obtain the lock, examine the shared data, transform it as necessary, and drop the lock. If you must perform some time-consuming activity, find a way to move it out of the synchronized region without violating the guidelines in Item 78.

The first part of this item was about correctness. Now let’s take a brief look at performance. While the cost of synchronization has plummeted since the early days of Java, it is more important than ever not to oversynchronize. In a multicore world, the real cost of excessive synchronization is not the CPU time spent getting locks; it is contention: the lost opportunities for parallelism and the delays imposed by the need to ensure that every core has a consistent view of memory. Another hidden cost of oversynchronization is that it can limit the VM’s ability to optimize code execution.

If you are writing a mutable class, you have two options: you can omit all synchronization and allow the client to synchronize externally if concurrent use is desired, or you can synchronize internally, making the class thread-safe (Item 82). You should choose the latter option only if you can achieve significantly higher concurrency with internal synchronization than you could by having the client lock the entire object externally. The collections in java.util (with the exception of the obsolete Vector and Hashtable) take the former approach, while those in java.util.concurrent take the latter (Item 81).

In the early days of Java, many classes violated these guidelines. For example, StringBuffer instances are almost always used by a single thread, yet they perform internal synchronization. It is for this reason that StringBuffer was supplanted by StringBuilder, which is just an unsynchronized StringBuffer. Similarly, it’s a large part of the reason that the thread-safe pseudorandom number generator in java.util.Random was supplanted by the unsynchronized implementation in java.util.concurrent.ThreadLocalRandom. When in doubt, do not synchronize your class, but document that it is not thread-safe.

If you do synchronize your class internally, you can use various techniques to achieve high concurrency, such as lock splitting, lock striping, and nonblocking concurrency control. These techniques are beyond the scope of this book, but they are discussed elsewhere [Goetz06, Herlihy08].

If a method modifies a static field and there is any possibility that the method will be called from multiple threads, you must synchronize access to the field internally (unless the class can tolerate nondeterministic behavior). It is not possible for a multithreaded client to perform external synchronization on such a method, because unrelated clients can invoke the method without synchronization. The field is essentially a global variable even if it is private because it can be read and modified by unrelated clients. The nextSerialNumber field used by the method generateSerialNumber in Item 78 exemplifies this situation.

In summary, to avoid deadlock and data corruption, never call an alien method from within a synchronized region. More generally, keep the amount of work that you do from within synchronized regions to a minimum. When you are designing a mutable class, think about whether it should do its own synchronization. In the multicore era, it is more important than ever not to oversynchronize. Synchronize your class internally only if there is a good reason to do so, and document your decision clearly (Item 82).
