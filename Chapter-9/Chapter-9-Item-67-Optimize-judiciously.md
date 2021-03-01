## Chapter 9. General Programming（通用程序设计）

### Item 67: Optimize judiciously（明智地进行优化）

There are three aphorisms concerning optimization that everyone should know:

有三条关于优化的格言是每个人都应该知道的：

More computing sins are committed in the name of efficiency (without necessarily achieving it) than for any other single reason—including blind stupidity.

比起其他任何单一的原因（包括盲目的愚蠢），很多计算上的过失都被归昝于效率（不一定能实现）。

—William A. Wulf [Wulf72]

We should forget about small efficiencies, say about 97% of the time: premature optimization is the root of all evil.

不要去计较效率上的一些小小的得失，在 97% 的情况下，不成熟的优化才是一切问题的根源。

—Donald E. Knuth [Knuth74]

We follow two rules in the matter of optimization:
Rule 1. Don’t do it.
Rule 2 (for experts only). Don’t do it yet—that is, not until you have a perfectly clear and unoptimized solution.

在优化方面，我们应该遵守两条规则：

规则 1：不要进行优化。

规则 2 （仅针对专家）：还是不要进行优化，也就是说，在你还没有绝对清晰的未优化方案之前，请不要进行优化。

—M. A. Jackson [Jackson75]

All of these aphorisms predate the Java programming language by two decades. They tell a deep truth about optimization: it is easy to do more harm than good, especially if you optimize prematurely. In the process, you may produce software that is neither fast nor correct and cannot easily be fixed.

所有这些格言都比 Java 编程语言早了 20 年。它们告诉我们关于优化的一个深刻的事实：很容易弊大于利，尤其是如果过早地进行优化。在此过程中，你可能会生成既不快速也不正确且无法轻松修复的软件。

Don’t sacrifice sound architectural principles for performance. Strive to write **good programs rather than fast ones.** If a good program is not fast enough, its architecture will allow it to be optimized. Good programs embody the principle of information hiding: where possible, they localize design decisions within individual components, so individual decisions can be changed without affecting the remainder of the system (Item 15).

不要为了性能而牺牲合理的架构。努力编写 **好的程序，而不是快速的程序。** 如果一个好的程序不够快，它的架构将允许它被优化。好的程序体现了信息隐藏的原则：在可能的情况下，它们在单个组件中本地化设计决策，因此可以在不影响系统其余部分的情况下更改单个决策（[Item-15](/Chapter-4/Chapter-4-Item-15-Minimize-the-accessibility-of-classes-and-members.md)）。

This does not mean that you can ignore performance concerns until your program is complete. Implementation problems can be fixed by later optimization, but pervasive architectural flaws that limit performance can be impossible to fix without rewriting the system. Changing a fundamental facet of your design after the fact can result in an ill-structured system that is difficult to maintain and evolve. Therefore you must think about performance during the design process.

这并不意味着在程序完成之前可以忽略性能问题。实现上的问题可以通过以后的优化来解决，但是对于架构缺陷，如果不重写系统，就不可能解决限制性能的问题。在系统完成之后再改变设计的某个基本方面可能导致结构不良的系统难以维护和进化。因此，你必须在设计过程中考虑性能。

**Strive to avoid design decisions that limit performance.** The components of a design that are most difficult to change after the fact are those specifying interactions between components and with the outside world. Chief among these design components are APIs, wire-level protocols, and persistent data formats. Not only are these design components difficult or impossible to change after the fact, but all of them can place significant limitations on the performance that a system can ever achieve.

**尽量避免限制性能的设计决策。** 设计中最难以更改的组件是那些指定组件之间以及与外部世界的交互的组件。这些设计组件中最主要的是 API、线路层协议和持久数据格式。这些设计组件不仅难以或不可能在事后更改，而且所有这些组件都可能对系统能够达到的性能造成重大限制。

**Consider the performance consequences of your API design decisions.** Making a public type mutable may require a lot of needless defensive copying (Item 50). Similarly, using inheritance in a public class where composition would have been appropriate ties the class forever to its superclass, which can place artificial limits on the performance of the subclass (Item 18). As a final example, using an implementation type rather than an interface in an API ties you to a specific implementation, even though faster implementations may be written in the future (Item 64).

**考虑API设计决策的性能结果。** 使公共类型转化为可变，可能需要大量不必要的防御性复制（[Item-50](/Chapter-8/Chapter-8-Item-50-Make-defensive-copies-when-needed.md)）。类似地，在一个公共类中使用继承（在这个类中组合将是合适的）将该类永远绑定到它的超类，这会人为地限制子类的性能（[Item-18](/Chapter-4/Chapter-4-Item-18-Favor-composition-over-inheritance.md)）。最后一个例子是，在 API 中使用实现类而不是接口将你绑定到特定的实现，即使将来可能会编写更快的实现也无法使用（[Item-64](/Chapter-9/Chapter-9-Item-64-Refer-to-objects-by-their-interfaces.md)）。

The effects of API design on performance are very real. Consider the getSize method in the java.awt.Component class. The decision that this performance-critical method was to return a Dimension instance, coupled with the decision that Dimension instances are mutable, forces any implementation of this method to allocate a new Dimension instance on every invocation. Even though allocating small objects is inexpensive on a modern VM, allocating millions of objects needlessly can do real harm to performance.

API 设计对性能的影响是非常实际的。考虑 `java.awt.Component` 中的 getSize 方法。该性能很关键方法返回 Dimension 实例的决定，加上维度实例是可变的决定，强制该方法的任何实现在每次调用时分配一个新的 Dimension 实例。尽管在现代 VM 上分配小对象并不昂贵，但不必要地分配数百万个对象也会对性能造成实际损害。

Several API design alternatives existed. Ideally, Dimension should have been immutable (Item 17); alternatively, getSize could have been replaced by two methods returning the individual primitive components of a Dimension object. In fact, two such methods were added to Component in Java 2 for performance reasons. Preexisting client code, however, still uses the getSize method and still suffers the performance consequences of the original API design decisions.

存在几种 API 设计替代方案。理想情况下，Dimension 应该是不可变的（[Item-17](/Chapter-4/Chapter-4-Item-17-Minimize-mutability.md)）；或者，getSize 可以被返回 Dimension 对象的原始组件的两个方法所替代。事实上，出于性能原因，在 Java 2 的组件中添加了两个这样的方法。然而，现有的客户端代码仍然使用 getSize 方法，并且仍然受到原始 API 设计决策的性能影响。

Luckily, it is generally the case that good API design is consistent with good performance. **It is a very bad idea to warp an API to achieve good performance.** The performance issue that caused you to warp the API may go away in a future release of the platform or other underlying software, but the warped API and the support headaches that come with it will be with you forever.

幸运的是，通常情况下，好的 API 设计与好的性能是一致的。**为了获得良好的性能而改变 API 是一个非常糟糕的想法。** 导致你改变 API 的性能问题，可能在平台或其他底层软件的未来版本中消失，但是改变的 API 和随之而来的问题将永远伴随着你。

Once you’ve carefully designed your program and produced a clear, concise, and well-structured implementation, then it may be time to consider optimization, assuming you’re not already satisfied with the performance of the program.

一旦你仔细地设计了你的程序，成了一个清晰、简洁、结构良好的实现，那么可能是时候考虑优化了，假设此时你还不满意程序的性能。

Recall that Jackson’s two rules of optimization were “Don’t do it,” and “(for experts only). Don’t do it yet.” He could have added one more: **measure performance before and after each attempted optimization.** You may be surprised by what you find. Often, attempted optimizations have no measurable effect on performance; sometimes, they make it worse. The main reason is that it’s difficult to guess where your program is spending its time. The part of the program that you think is slow may not be at fault, in which case you’d be wasting your time trying to optimize it. Common wisdom says that programs spend 90 percent of their time in 10 percent of their code.

记得 Jackson 的两条优化规则是「不要做」和「（只针对专家）」。先别这么做。他本可以再加一个：**在每次尝试优化之前和之后测量性能。** 你可能会对你的发现感到惊讶。通常，试图做的优化通常对于性能并没有明显的影响；有时候，还让事情变得更糟。主要原因是很难猜测程序将时间花费在哪里。程序中你认为很慢的部分可能并没有问题，在这种情况下，你是在浪费时间来优化它。一般认为，程序将 90% 的时间花费在了 10% 的代码上。

Profiling tools can help you decide where to focus your optimization efforts. These tools give you runtime information, such as roughly how much time each method is consuming and how many times it is invoked. In addition to focusing your tuning efforts, this can alert you to the need for algorithmic changes. If a quadratic (or worse) algorithm lurks inside your program, no amount of tuning will fix the problem. You must replace the algorithm with one that is more efficient. The more code in the system, the more important it is to use a profiler. It’s like looking for a needle in a haystack: the bigger the haystack, the more useful it is to have a metal detector. Another tool that deserves special mention is jmh, which is not a profiler but a microbenchmarking framework that provides unparalleled visibility into the detailed performance of Java code [JMH].

分析工具可以帮助你决定将优化工作的重点放在哪里。这些工具提供了运行时信息，比如每个方法大约花费多少时间以及调用了多少次。除了关注你的调优工作之外，这还可以提醒你是否需要改变算法。如果程序中潜伏着平方级（或更差）的算法，那么再多的调优也无法解决这个问题。你必须用一个更有效的算法来代替这个算法。系统中的代码越多，使用分析器就越重要。这就像大海捞针：大海越大，金属探测器就越有用。另一个值得特别提及的工具是 jmh，它不是一个分析器，而是一个微基准测试框架，提供了对 Java 代码性能无与伦比的预测性。

The need to measure the effects of attempted optimization is even greater in Java than in more traditional languages such as C and C++, because Java has a weaker performance model: The relative cost of the various primitive operations is less well defined. The “abstraction gap” between what the programmer writes and what the CPU executes is greater, which makes it even more difficult to reliably predict the performance consequences of optimizations. There are plenty of performance myths floating around that turn out to be half-truths or outright lies.

与 C 和 C++ 等更传统的语言相比，Java 甚至更需要度量尝试优化的效果，因为 Java 的性能模型更弱：各种基本操作的相对成本没有得到很好的定义。程序员编写的内容和 CPU 执行的内容之间的「抽象鸿沟」更大，这使得可靠地预测优化的性能结果变得更加困难。有很多关于性能的传说流传开来，但最终被证明是半真半假或彻头彻尾的谎言。

Not only is Java’s performance model ill-defined, but it varies from implementation to implementation, from release to release, and from processor to processor. If you will be running your program on multiple implementations or multiple hardware platforms, it is important that you measure the effects of your optimization on each. Occasionally you may be forced to make trade-offs between performance on different implementations or hardware platforms.

Java 的性能模型不仅定义不清，而且在不同的实现、不同的发布版本、不同的处理器之间都有所不同。如果你要在多个实现或多个硬件平台上运行程序，那么度量优化对每个平台的效果是很重要的。有时候，你可能会被迫在不同实现或硬件平台上的性能之间进行权衡。

In the nearly two decades since this item was first written, every component of the Java software stack has grown in complexity, from processors to VMs to libraries, and the variety of hardware on which Java runs has grown immensely. All of this has combined to make the performance of Java programs even less predictable now than it was in 2001, with a corresponding increase in the need to measure it.

自本条目首次编写以来的近 20 年里，Java 软件栈的每个组件都变得越来越复杂，从处理器到 vm 再到库，Java 运行的各种硬件都有了极大的增长。所有这些加在一起，使得 Java 程序的性能比 2001 年更难以预测，而对它进行度量的需求也相应增加。

To summarize, do not strive to write fast programs—strive to write good ones; speed will follow. But do think about performance while you’re designing systems, especially while you’re designing APIs, wire-level protocols, and persistent data formats. When you’ve finished building the system, measure its performance. If it’s fast enough, you’re done. If not, locate the source of the problem with the aid of a profiler and go to work optimizing the relevant parts of the system. The first step is to examine your choice of algorithms: no amount of low-level optimization can make up for a poor choice of algorithm. Repeat this process as necessary, measuring the performance after every change, until you’re satisfied.

总而言之，不要努力写快的程序，要努力写好程序；速度自然会提高。但是在设计系统时一定要考虑性能，特别是在设计API、线路层协议和持久数据格式时。当你完成了系统的构建之后，请度量它的性能。如果足够快，就完成了。如果没有，利用分析器找到问题的根源，并对系统的相关部分进行优化。第一步是检查算法的选择：再多的底层优化也不能弥补算法选择的不足。根据需要重复这个过程，在每次更改之后测量性能，直到你满意为止。

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-9/Chapter-9-Introduction.md)**
- **Previous Item（上一条目）：[Item 66: Use native methods judiciously（明智地使用本地方法）](/Chapter-9/Chapter-9-Item-66-Use-native-methods-judiciously.md)**
- **Next Item（下一条目）：[Item 68: Adhere to generally accepted naming conventions（遵守被广泛认可的命名约定）](/Chapter-9/Chapter-9-Item-68-Adhere-to-generally-accepted-naming-conventions.md)**
