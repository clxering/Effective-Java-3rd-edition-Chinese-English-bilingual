## Chapter 11. Concurrency（并发）

### Item 78: Synchronize access to shared mutable data

The synchronized keyword ensures that only a single thread can execute a method or block at one time. Many programmers think of synchronization solely as a means of mutual exclusion, to prevent an object from being seen in an inconsistent state by one thread while it’s being modified by another. In this view, an object is created in a consistent state (Item 17) and locked by the methods that access it. These methods observe the state and optionally cause a state transition, transforming the object from one consistent state to another. Proper use of synchronization guarantees that no method will ever observe the object in an inconsistent state.

This view is correct, but it’s only half the story. Without synchronization, one thread’s changes might not be visible to other threads. Not only does synchronization prevent threads from observing an object in an inconsistent state, but it ensures that each thread entering a synchronized method or block sees the effects of all previous modifications that were guarded by the same lock.

The language specification guarantees that reading or writing a variable is atomic unless the variable is of type long or double [JLS, 17.4, 17.7]. In other words, reading a variable other than a long or double is guaranteed to return a value that was stored into that variable by some thread, even if multiple threads modify the variable concurrently and without synchronization.

You may hear it said that to improve performance, you should dispense with synchronization when reading or writing atomic data. This advice is dangerously wrong. While the language specification guarantees that a thread will not see an arbitrary value when reading a field, it does not guarantee that a value written by one thread will be visible to another. **Synchronization is required for reliable communication between threads as well as for mutual exclusion.** This is due to a part of the language specification known as the memory model, which specifies when and how changes made by one thread become visible to others [JLS, 17.4; Goetz06, 16].

The consequences of failing to synchronize access to shared mutable data can be dire even if the data is atomically readable and writable. Consider the task of stopping one thread from another. The libraries provide the Thread.stop method, but this method was deprecated long ago because it is inherently unsafe —its use can result in data corruption. **Do not use Thread.stop.** A recommended way to stop one thread from another is to have the first thread poll a boolean field that is initially false but can be set to true by the second thread to indicate that the first thread is to stop itself. Because reading and writing a boolean field is atomic, some programmers dispense with synchronization when accessing the field:

```
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

The problem is that in the absence of synchronization, there is no guarantee as to when, if ever, the background thread will see the change in the value of stopRequested made by the main thread. In the absence of synchronization, it’s quite acceptable for the virtual machine to transform this code:

```
while (!stopRequested)
i++;
into this code:
if (!stopRequested)
while (true)
i++;
```

This optimization is known as hoisting, and it is precisely what the OpenJDK Server VM does. The result is a liveness failure: the program fails to make progress. One way to fix the problem is to synchronize access to the stopRequested field. This program terminates in about one second, as expected:

```
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

The actions of the synchronized methods in StopThread would be atomic even without synchronization. In other words, the synchronization on these methods is used solely for its communication effects, not for mutual exclusion. While the cost of synchronizing on each iteration of the loop is small, there is a correct alternative that is less verbose and whose performance is likely to be better. The locking in the second version of StopThread can be omitted if stopRequested is declared volatile. While the volatile modifier performs no mutual exclusion, it guarantees that any thread that reads the field will see the most recently written value:

```
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

```
// Broken - requires synchronization!
private static volatile int nextSerialNumber = 0;
public static int generateSerialNumber() {
    return nextSerialNumber++;
}
```

The intent of the method is to guarantee that every invocation returns a unique value (so long as there are no more than 232 invocations). The method’s state consists of a single atomically accessible field, nextSerialNumber, and all possible values of this field are legal. Therefore, no synchronization is necessary to protect its invariants. Still, the method won’t work properly without synchronization.

The problem is that the increment operator (++) is not atomic. It performs two operations on the nextSerialNumber field: first it reads the value, and then it writes back a new value, equal to the old value plus one. If a second thread reads the field between the time a thread reads the old value and writes back a new one, the second thread will see the same value as the first and return the same serial number. This is a safety failure: the program computes the wrong results.

One way to fix generateSerialNumber is to add the synchronized modifier to its declaration. This ensures that multiple invocations won’t be interleaved and that each invocation of the method will see the effects of all previous invocations. Once you’ve done that, you can and should remove the volatile modifier from nextSerialNumber. To bulletproof the method, use long instead of int, or throw an exception if nextSerialNumber is about to wrap.

Better still, follow the advice in Item 59 and use the class AtomicLong, which is part of java.util.concurrent.atomic. This package provides primitives for lock-free, thread-safe programming on single variables. While volatile provides only the communication effects of synchronization, this package also provides atomicity. This is exactly what we want for generateSerialNumber, and it is likely to outperform the synchronized version:

```
// Lock-free synchronization with java.util.concurrent.atomic
private static final AtomicLong nextSerialNum = new AtomicLong();
public static long generateSerialNumber() {
    return nextSerialNum.getAndIncrement();
}
```

The best way to avoid the problems discussed in this item is not to share mutable data. Either share immutable data (Item 17) or don’t share at all. In other words, **confine mutable data to a single thread.** If you adopt this policy, it is important to document it so that the policy is maintained as your program evolves. It is also important to have a deep understanding of the frameworks and libraries you’re using because they may introduce threads that you are unaware of.

It is acceptable for one thread to modify a data object for a while and then to share it with other threads, synchronizing only the act of sharing the object reference. Other threads can then read the object without further synchronization, so long as it isn’t modified again. Such objects are said to be effectively immutable [Goetz06, 3.5.4]. Transferring such an object reference from one thread to others is called safe publication [Goetz06, 3.5.3]. There are many ways to safely publish an object reference: you can store it in a static field as part of class initialization; you can store it in a volatile field, a final field, or a field that is accessed with normal locking; or you can put it into a concurrent collection (Item 81).

In summary, **when multiple threads share mutable data, each thread that reads or writes the data must perform synchronization.** In the absence of synchronization, there is no guarantee that one thread’s changes will be visible to another thread. The penalties for failing to synchronize shared mutable data are liveness and safety failures. These failures are among the most difficult to debug. They can be intermittent and timing-dependent, and program behavior can vary radically from one VM to another. If you need only inter-thread communication, and not mutual exclusion, the volatile modifier is an acceptable form of synchronization, but it can be tricky to use correctly.


