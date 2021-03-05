## Chapter 11. Concurrency（并发）

### Item 80: Prefer executors, tasks, and streams to threads（Executor、task、流优于直接使用线程）

The first edition of this book contained code for a simple work queue [Bloch01, Item 49]. This class allowed clients to enqueue work for asynchronous processing by a background thread. When the work queue was no longer needed, the client could invoke a method to ask the background thread to terminate itself gracefully after completing any work that was already on the queue. The implementation was little more than a toy, but even so, it required a full page of subtle, delicate code, of the sort that is prone to safety and liveness failures if you don’t get it just right. Luckily, there is no reason to write this sort of code anymore.

本书的第一版包含一个简单工作队列的代码 [Bloch01, Item 49]。这个类允许客户端通过后台线程为异步处理排队。当不再需要工作队列时，客户端可以调用一个方法，要求后台线程在完成队列上的任何工作后优雅地终止自己。这个实现只不过是一个玩具，但即便如此，它也需要一整页的代码，如果你做得不对，就很容易出现安全和活性失败。幸运的是，没有理由再编写这种代码了。

By the time the second edition of this book came out, java.util.concurrent had been added to Java. This package contains an Executor Framework, which is a flexible interface-based task execution facility. Creating a work queue that is better in every way than the one in the first edition of this book requires but a single line of code:

当这本书的第二版出版时，`java.util.concurrent` 已经添加到 Java 中。这个包有一个 Executor 框架，它是一个灵活的基于接口的任务执行工具。创建一个工作队列，它在任何方面都比在这本书的第一版更好，只需要一行代码：

```
ExecutorService exec = Executors.newSingleThreadExecutor();

Here is how to submit a runnable for execution:
exec.execute(runnable);

And here is how to tell the executor to terminate gracefully (if you fail to do this,it is likely that your VM will not exit):
exec.shutdown();
```

You can do many more things with an executor service. For example, you can wait for a particular task to complete (with the get method, as shown in Item 79, page 319), you can wait for any or all of a collection of tasks to complete (using the invokeAny or invokeAll methods), you can wait for the executor service to terminate (using the awaitTermination method), you can retrieve the results of tasks one by one as they complete (using an ExecutorCompletionService), you can schedule tasks to run at a particular time or to run periodically (using a ScheduledThreadPoolExecutor), and so on.

你可以使用 executor 服务做更多的事情。例如，你可以等待一个特定任务完成（使用 get 方法，参见 [Item-79](/Chapter-11/Chapter-11-Item-79-Avoid-excessive-synchronization.md)，319 页），你可以等待任务集合中任何或全部任务完成（使用 invokeAny 或 invokeAll 方法），你可以等待 executor 服务终止（使用 awaitTermination 方法），你可以一个接一个检索任务，获取他们完成的结果（使用一个 ExecutorCompletionService），还可以安排任务在特定时间运行或定期运行（使用 ScheduledThreadPoolExecutor），等等。

If you want more than one thread to process requests from the queue, simply call a different static factory that creates a different kind of executor service called a thread pool. You can create a thread pool with a fixed or variable number of threads. The java.util.concurrent.Executors class contains static factories that provide most of the executors you’ll ever need. If, however, you want something out of the ordinary, you can use the ThreadPoolExecutor class directly. This class lets you configure nearly every aspect of a thread pool’s operation.

如果希望多个线程处理来自队列的请求，只需调用一个不同的静态工厂，该工厂创建一种称为线程池的不同类型的 executor 服务。你可以使用固定或可变数量的线程创建线程池。`java.util.concurrent.Executors` 类包含静态工厂，它们提供你需要的大多数 executor。但是，如果你想要一些不同寻常的东西，你可以直接使用 ThreadPoolExecutor 类。这个类允许你配置线程池操作的几乎每个方面。

Choosing the executor service for a particular application can be tricky. For a small program, or a lightly loaded server, Executors.newCachedThreadPool is generally a good choice because it demands no configuration and generally “does the right thing.” But a cached thread pool is not a good choice for a heavily loaded production server! In a cached thread pool, submitted tasks are not queued but immediately handed off to a thread for execution. If no threads are available, a new one is created. If a server is so heavily loaded that all of its CPUs are fully utilized and more tasks arrive, more threads will be created, which will only make matters worse. Therefore, in a heavily loaded production server, you are much better off using Executors.newFixedThreadPool, which gives you a pool with a fixed number of threads, or using the ThreadPoolExecutor class directly, for maximum control.

为特定的应用程序选择 executor 服务可能比较棘手。对于小程序或负载较轻的服务器，`Executors.newCachedThreadPool` 通常是一个不错的选择，因为它不需要配置，而且通常「做正确的事情」。但是对于负载沉重的生产服务器来说，缓存的线程池不是一个好的选择！在缓存的线程池中，提交的任务不会排队，而是立即传递给线程执行。如果没有可用的线程，则创建一个新的线程。如果服务器负载过重，所有 CPU 都被充分利用，并且有更多的任务到达，就会创建更多的线程，这只会使情况变得更糟。因此，在负载沉重的生产服务器中，最好使用 `Executors.newFixedThreadPool`，它为你提供一个线程数量固定的池，或者直接使用 ThreadPoolExecutor 类来实现最大限度的控制。

Not only should you refrain from writing your own work queues, but you should generally refrain from working directly with threads. When you work directly with threads, a Thread serves as both a unit of work and the mechanism for executing it. In the executor framework, the unit of work and the execution mechanism are separate. The key abstraction is the unit of work, which is the task. There are two kinds of tasks: Runnable and its close cousin, Callable (which is like Runnable, except that it returns a value and can throw arbitrary exceptions). The general mechanism for executing tasks is the executor service. If you think in terms of tasks and let an executor service execute them for you, you gain the flexibility to select an appropriate execution policy to meet your needs and to change the policy if your needs change. In essence, the Executor Framework does for execution what the Collections Framework did for aggregation.

你不仅应该避免编写自己的工作队列，而且通常还应该避免直接使用线程。当你直接使用线程时，线程既是工作单元，又是执行它的机制。在 executor 框架中，工作单元和执行机制是分开的。关键的抽象是工作单元，即任务。有两种任务：Runnable 和它的近亲 Callable（与 Runnable 类似，只是它返回一个值并可以抛出任意异常）。执行任务的一般机制是 executor 服务。如果你从任务的角度考虑问题，并让 executor 服务为你执行这些任务，那么你就可以灵活地选择合适的执行策略来满足你的需求，并在你的需求发生变化时更改策略。本质上，Executor 框架执行的功能与 Collections 框架聚合的功能相同。

In Java 7, the Executor Framework was extended to support fork-join tasks, which are run by a special kind of executor service known as a fork-join pool. A fork-join task, represented by a ForkJoinTask instance, may be split up into smaller subtasks, and the threads comprising a ForkJoinPool not only process these tasks but “steal” tasks from one another to ensure that all threads remain busy, resulting in higher CPU utilization, higher throughput, and lower latency. Writing and tuning fork-join tasks is tricky. Parallel streams (Item 48) are written atop fork join pools and allow you to take advantage of their performance benefits with little effort, assuming they are appropriate for the task at hand.

在 Java 7 中，Executor 框架被扩展为支持 fork-join 任务，这些任务由一种特殊的 Executor 服务（称为 fork-join 池）运行。由 ForkJoinTask 实例表示的 fork-join 任务可以划分为更小的子任务，由 ForkJoinPool 组成的线程不仅处理这些任务，而且还从其他线程「窃取」任务，以确保所有线程都处于繁忙状态，从而提高 CPU 利用率、更高的吞吐量和更低的延迟。编写和调优 fork-join 任务非常棘手。并行流（[Item-48](/Chapter-7/Chapter-7-Item-48-Use-caution-when-making-streams-parallel.md)）
是在 fork 连接池之上编写的，假设它们适合当前的任务，那么你可以轻松地利用它们的性能优势。

A complete treatment of the Executor Framework is beyond the scope of this book, but the interested reader is directed to Java Concurrency in Practice [Goetz06].

对 Executor 框架的完整处理超出了本书的范围，但是感兴趣的读者可以在实践中可以参阅《Java Concurrency in Practice》 [Goetz06]。

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-11/Chapter-11-Introduction.md)**
- **Previous Item（上一条目）：[Item 79: Avoid excessive synchronization（避免过度同步）](/Chapter-11/Chapter-11-Item-79-Avoid-excessive-synchronization.md)**
- **Next Item（下一条目）：[Item 81: Prefer concurrency utilities to wait and notify（并发实用工具优于 wait 和 notify）](/Chapter-11/Chapter-11-Item-81-Prefer-concurrency-utilities-to-wait-and-notify.md)**
