## Chapter 11. Concurrency（并发）

### Item 80: Prefer executors, tasks, and streams to threads

The first edition of this book contained code for a simple work queue [Bloch01, Item 49]. This class allowed clients to enqueue work for asynchronous processing by a background thread. When the work queue was no longer needed, the client could invoke a method to ask the background thread to terminate itself gracefully after completing any work that was already on the queue. The implementation was little more than a toy, but even so, it required a full page of subtle, delicate code, of the sort that is prone to safety and liveness failures if you don’t get it just right. Luckily, there is no reason to write this sort of code anymore.

By the time the second edition of this book came out, java.util.concurrent had been added to Java. This package contains an Executor Framework, which is a flexible interface-based task execution facility. Creating a work queue that is better in every way than the one in the first edition of this book requires but a single line of code:

```
ExecutorService exec = Executors.newSingleThreadExecutor();
Here is how to submit a runnable for execution:
exec.execute(runnable);
And here is how to tell the executor to terminate gracefully (if you fail to do this,it is likely that your VM will not exit):
exec.shutdown();
```

You can do many more things with an executor service. For example, you can wait for a particular task to complete (with the get method, as shown in Item 79, page 319), you can wait for any or all of a collection of tasks to complete (using the invokeAny or invokeAll methods), you can wait for the executor service to terminate (using the awaitTermination method), you can retrieve the results of tasks one by one as they complete (using an ExecutorCompletionService), you can schedule tasks to run at a particular time or to run periodically (using a ScheduledThreadPoolExecutor), and so on.

If you want more than one thread to process requests from the queue, simply call a different static factory that creates a different kind of executor service called a thread pool. You can create a thread pool with a fixed or variable number of threads. The java.util.concurrent.Executors class contains static factories that provide most of the executors you’ll ever need. If, however, you want something out of the ordinary, you can use the ThreadPoolExecutor class directly. This class lets you configure nearly every aspect of a thread pool’s operation.

Choosing the executor service for a particular application can be tricky. For a small program, or a lightly loaded server, Executors.newCachedThreadPool is generally a good choice because it demands no configuration and generally “does the right thing.” But a cached thread pool is not a good choice for a heavily loaded production server! In a cached thread pool, submitted tasks are not queued but immediately handed off to a thread for execution. If no threads are available, a new one is created. If a server is so heavily loaded that all of its CPUs are fully utilized and more tasks arrive, more threads will be created, which will only make matters worse. Therefore, in a heavily loaded production server, you are much better off using Executors.newFixedThreadPool, which gives you a pool with a fixed number of threads, or using the ThreadPoolExecutor class directly, for maximum control.

Not only should you refrain from writing your own work queues, but you should generally refrain from working directly with threads. When you work directly with threads, a Thread serves as both a unit of work and the mechanism for executing it. In the executor framework, the unit of work and the execution mechanism are separate. The key abstraction is the unit of work, which is the task. There are two kinds of tasks: Runnable and its close cousin, Callable (which is like Runnable, except that it returns a value and can throw arbitrary exceptions). The general mechanism for executing tasks is the executor service. If you think in terms of tasks and let an executor service execute them for you, you gain the flexibility to select an appropriate execution policy to meet your needs and to change the policy if your needs change. In essence, the Executor Framework does for execution what the Collections Framework did for aggregation.

In Java 7, the Executor Framework was extended to support fork-join tasks, which are run by a special kind of executor service known as a fork-join pool. A fork-join task, represented by a ForkJoinTask instance, may be split up into smaller subtasks, and the threads comprising a ForkJoinPool not only process these tasks but “steal” tasks from one another to ensure that all threads remain busy, resulting in higher CPU utilization, higher throughput, and lower latency. Writing and tuning fork-join tasks is tricky. Parallel streams (Item 48) are written atop fork join pools and allow you to take advantage of their performance benefits with little effort, assuming they are appropriate for the task at hand.

A complete treatment of the Executor Framework is beyond the scope of this book, but the interested reader is directed to Java Concurrency in Practice [Goetz06].


