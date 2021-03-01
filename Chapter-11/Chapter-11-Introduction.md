## Chapter 11. Concurrency（并发）

### Chapter 11 Introduction（章节介绍）

THREADS allow multiple activities to proceed concurrently. Concurrent programming is harder than single-threaded programming, because more things can go wrong, and failures can be hard to reproduce. You can’t avoid concurrency. It is inherent in the platform and a requirement if you are to obtain good performance from multicore processors, which are now ubiquitous. This chapter contains advice to help you write clear, correct, well-documented concurrent programs.

线程允许多个活动并发进行。并发编程比单线程编程更困难，容易出错的地方更多，而且失败很难重现。你无法避开并发。它是平台中固有的，并且多核处理器现在也是无处不在，而你会有从多核处理器获得良好的性能的需求。本章包含一些建议，帮助你编写清晰、正确、文档良好的并发程序。

### Contents of the chapter（章节目录）
- [Item 78: Synchronize access to shared mutable data（对共享可变数据的同步访问）](/Chapter-11/Chapter-11-Item-78-Synchronize-access-to-shared-mutable-data.md)
- [Item 79: Avoid excessive synchronization（避免过度同步）](/Chapter-11/Chapter-11-Item-79-Avoid-excessive-synchronization.md)
- [Item 80: Prefer executors, tasks, and streams to threads（Executor、task、流优于直接使用线程）](/Chapter-11/Chapter-11-Item-80-Prefer-executors,-tasks,-and-streams-to-threads.md)
- [Item 81: Prefer concurrency utilities to wait and notify（并发实用工具优于 wait 和 notify）](/Chapter-11/Chapter-11-Item-81-Prefer-concurrency-utilities-to-wait-and-notify.md)
- [Item 82: Document thread safety（文档应包含线程安全属性）](/Chapter-11/Chapter-11-Item-82-Document-thread-safety.md)
- [Item 83: Use lazy initialization judiciously（明智地使用延迟初始化）](/Chapter-11/Chapter-11-Item-83-Use-lazy-initialization-judiciously.md)
- [Item 84: Don’t depend on the thread scheduler（不要依赖线程调度器）](/Chapter-11/Chapter-11-Item-84-Don’t-depend-on-the-thread-scheduler.md)
