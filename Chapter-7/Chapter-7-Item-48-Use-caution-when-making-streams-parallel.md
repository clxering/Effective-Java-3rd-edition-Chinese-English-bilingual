## Chapter 7. Lambdas and Streams（λ 表达式和流）

### Item 48: Use caution when making streams parallel（谨慎使用并行流）

Among mainstream languages, Java has always been at the forefront of providing facilities to ease the task of concurrent programming. When Java was released in 1996, it had built-in support for threads, with synchronization and wait/notify. Java 5 introduced the java.util.concurrent library, with concurrent collections and the executor framework. Java 7 introduced the fork-join package, a high-performance framework for parallel decomposition. Java 8 introduced streams, which can be parallelized with a single call to the parallel method. Writing concurrent programs in Java keeps getting easier, but writing concurrent programs that are correct and fast is as difficult as it ever was. Safety and liveness violations are a fact of life in concurrent programming, and parallel stream pipelines are no exception.

在主流语言中，Java 一直走在提供简化并发编程任务工具的前列。当 Java 在 1996 年发布时，它内置了对线程的支持，支持同步和 wait/notify。Java 5 引入了 `java.util.concurrent`。具有并发集合和执行器框架的并发库。Java 7 引入了 fork-join 包，这是一个用于并行分解的高性能框架。Java 8 引入了流，它可以通过对 parallel 方法的一次调用来并行化。用 Java 编写并发程序变得越来越容易，但是编写正确且快速的并发程序却和以前一样困难。在并发编程中，安全性和活性的违反是不可避免的，并行流管道也不例外。

Consider this program from Item 45:

考虑 [Item-45](/Chapter-7/Chapter-7-Item-45-Use-streams-judiciously.md) 的程序：

```Java
// Stream-based program to generate the first 20 Mersenne primes
public static void main(String[] args) {
    primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
    .filter(mersenne -> mersenne.isProbablePrime(50))
    .limit(20)
    .forEach(System.out::println);
}

static Stream<BigInteger> primes() {
    return Stream.iterate(TWO, BigInteger::nextProbablePrime);
}
```

On my machine, this program immediately starts printing primes and takes 12.5 seconds to run to completion. Suppose I naively try to speed it up by adding a call to parallel() to the stream pipeline. What do you think will happen to its performance? Will it get a few percent faster? A few percent slower? Sadly, what happens is that it doesn’t print anything, but CPU usage spikes to 90 percent and stays there indefinitely (a liveness failure). The program might terminate eventually, but I was unwilling to find out; I stopped it forcibly after half an hour.

在我的机器上，这个程序立即开始打印素数，运行 12.5 秒完成。假设我天真地尝试通过向流管道添加对 `parallel()` 的调用来加速它。你认为它的性能会怎么样？它会快几个百分点吗？慢了几个百分点？遗憾的是，它不会打印任何东西，但是 CPU 使用率会飙升到 90%，并且会无限期地停留在那里（活跃性失败）。这个项目最终可能会终止，但我不愿意知道；半小时后我强行停了下来。

What’s going on here? Simply put, the streams library has no idea how to parallelize this pipeline and the heuristics fail. Even under the best of circumstances, **parallelizing a pipeline is unlikely to increase its performance if the source is from Stream.iterate, or the intermediate operation limit is used.** This pipeline has to contend with both of these issues. Worse, the default parallelization strategy deals with the unpredictability of limit by assuming there’s no harm in processing a few extra elements and discarding any unneeded results. In this case, it takes roughly twice as long to find each Mersenne prime as it did to find the previous one. Thus, the cost of computing a single extra element is roughly equal to the cost of computing all previous elements combined, and this innocuous-looking pipeline brings the automatic parallelization algorithm to its knees. The moral of this story is simple: **Do not parallelize stream pipelines indiscriminately.** The performance consequences may be disastrous.

这是怎么回事？简单地说，stream 库不知道如何并行化这个管道，因此启发式会失败。即使在最好的情况下，**如果源来自 `Stream.iterate` 或使用 Intermediate 操作限制，并行化管道也不太可能提高其性能。** 这条管道必须解决这两个问题。更糟糕的是，默认的并行化策略通过假设处理一些额外的元素和丢弃任何不需要的结果没有害处来处理极限的不可预测性。在这种情况下，找到每一个 Mersenne 素数所需的时间大约是找到上一个 Mersenne 素数所需时间的两倍。因此，计算单个额外元素的成本大致等于计算之前所有元素的总和，而这条看上去毫无问题的管道将自动并行化算法推到了极致。这个故事的寓意很简单：**不要不加区别地将流管道并行化。** 性能后果可能是灾难性的。

As a rule, **performance gains from parallelism are best on streams over ArrayList, HashMap, HashSet, and ConcurrentHashMap instances; arrays; int ranges; and long ranges.** What these data structures have in common is that they can all be accurately and cheaply split into subranges of any desired sizes, which makes it easy to divide work among parallel threads. The abstraction used by the streams library to perform this task is the spliterator, which is returned by the spliterator method on Stream and Iterable.

通常，**并行性带来的性能提升在 ArrayList、HashMap、HashSet 和 ConcurrentHashMap 实例上的流效果最好；int 数组和 long 数组也在其中。** 这些数据结构的共同之处在于，它们都可以被精确且廉价地分割成任意大小的子程序，这使得在并行线程之间划分工作变得很容易。stream 库用于执行此任务的抽象是 spliterator，它由流上的 spliterator 方法返回并可迭代。

Another important factor that all of these data structures have in common is that they provide good-to-excellent locality of reference when processed sequentially: sequential element references are stored together in memory. The objects referred to by those references may not be close to one another in memory, which reduces locality-of-reference. Locality-of-reference turns out to be critically important for parallelizing bulk operations: without it, threads spend much of their time idle, waiting for data to be transferred from memory into the processor’s cache. The data structures with the best locality of reference are primitive arrays because the data itself is stored contiguously in memory.

所有这些数据结构的另一个重要共同点是，当按顺序处理时，它们提供了从优秀到优秀的引用位置：顺序元素引用一起存储在内存中。这些引用引用的对象在内存中可能彼此不太接近，这降低了引用的位置。引用位置对于并行化批量操作非常重要：如果没有它，线程将花费大量时间空闲，等待数据从内存传输到处理器的缓存中。具有最佳引用位置的数据结构是基本数组，因为数据本身是连续存储在内存中的。

The nature of a stream pipeline’s terminal operation also affects the effectiveness of parallel execution. If a significant amount of work is done in the terminal operation compared to the overall work of the pipeline and that operation is inherently sequential, then parallelizing the pipeline will have limited effectiveness. The best terminal operations for parallelism are reductions, where all of the elements emerging from the pipeline are combined using one of Stream’s reduce methods, or prepackaged reductions such as min, max, count, and sum. The short-circuiting operations anyMatch, allMatch, and noneMatch are also amenable to parallelism. The operations performed by Stream’s collect method, which are known as mutable reductions, are not good candidates for parallelism because the overhead of combining collections is costly.

流管道 Terminal 操作的性质也会影响并行执行的有效性。如果与管道的总体工作相比，在 Terminal 操作中完成了大量的工作，并且该操作本质上是顺序的，那么管道的并行化将具有有限的有效性。并行性的最佳 Terminal 操作是缩减，其中来自管道的所有元素都使用流的缩减方法之一进行组合，或者使用预先打包的缩减，如最小、最大、计数和和。anyMatch、allMatch 和 noneMatch 的短路操作也适用于并行性。流的 collect 方法执行的操作称为可变缩减，它们不是并行性的好候选，因为组合集合的开销是昂贵的。

If you write your own Stream, Iterable, or Collection implementation and you want decent parallel performance, you must override the spliterator method and test the parallel performance of the resulting streams extensively. Writing high-quality spliterators is difficult and beyond the scope of this book.

如果你编写自己的流、Iterable 或 Collection 实现，并且希望获得良好的并行性能，则必须重写 spliterator 方法，并广泛地测试结果流的并行性能。编写高质量的 spliterator 是困难的，超出了本书的范围。

**Not only can parallelizing a stream lead to poor performance, including liveness failures; it can lead to incorrect results and unpredictable behavior** (safety failures). Safety failures may result from parallelizing a pipeline that uses mappers, filters, and other programmer-supplied function objects that fail to adhere to their specifications. The Stream specification places stringent requirements on these function objects. For example, the accumulator and combiner functions passed to Stream’s reduce operation must be associative, non-interfering, and stateless. If you violate these requirements (some of which are discussed in Item 46) but run your pipeline sequentially, it will likely yield correct results; if you parallelize it, it will likely fail, perhaps catastrophically. Along these lines, it’s worth noting that even if the parallelized Mersenne primes program had run to completion, it would not have printed the primes in the correct (ascending) order. To preserve the order displayed by the sequential version, you’d have to replace the forEach terminal operation with forEachOrdered, which is guaranteed to traverse parallel streams in encounter order.

**并行化流不仅会导致糟糕的性能，包括活动失败；它会导致不正确的结果和不可预知的行为（安全故障）。** 如果管道使用映射器、过滤器和其他程序员提供的函数对象，而这些对象没有遵守其规范，则并行化管道可能导致安全故障。流规范对这些功能对象提出了严格的要求。例如，传递给流的 reduce 操作的累加器和组合器函数必须是关联的、不干扰的和无状态的。如果你违反了这些要求（其中一些要求在 [Item-46](/Chapter-7/Chapter-7-Item-46-Prefer-side-effect-free-functions-in-streams.md) 中讨论），但是按顺序运行管道，则可能会产生正确的结果；如果你并行化它，它很可能会失败，可能是灾难性的。沿着这些思路，值得注意的是，即使并行化的 Mersenne 素数程序运行到完成，它也不会以正确的（升序）顺序打印素数。为了保留序列版本所显示的顺序，你必须将 forEach 这一 Terminal 操作替换为 forEachOrdered，它保证按顺序遍历并行流。

Even assuming that you’re using an efficiently splittable source stream, a parallelizable or cheap terminal operation, and non-interfering function objects, you won’t get a good speedup from parallelization unless the pipeline is doing enough real work to offset the costs associated with parallelism. As a very rough estimate, the number of elements in the stream times the number of lines of code executed per element should be at least a hundred thousand [Lea14].

即使假设你正在使用一个高效的可分割源流、一个可并行化的或廉价的 Terminal 操作，以及不受干扰的函数对象，你也不会从并行化中获得良好的加速，除非管道正在做足够的实际工作来抵消与并行性相关的成本。作为一个非常粗略的估计，流中的元素数量乘以每个元素执行的代码行数至少应该是 100000 [Lea14]。

It’s important to remember that parallelizing a stream is strictly a performance optimization. As is the case for any optimization, you must test the performance before and after the change to ensure that it is worth doing (Item 67). Ideally, you should perform the test in a realistic system setting. Normally, all parallel stream pipelines in a program run in a common fork-join pool. A single misbehaving pipeline can harm the performance of others in unrelated parts of the system.

重要的是要记住，并行化流严格来说是一种性能优化。与任何优化一样，你必须在更改之前和之后测试性能，以确保它值得进行（[Item-67](/Chapter-9/Chapter-9-Item-67-Optimize-judiciously.md)）。理想情况下，你应该在实际的系统设置中执行测试。通常，程序中的所有并行流管道都在公共 fork-join 池中运行。一个行为不当的管道可能会损害系统中不相关部分的其他管道的性能。

If it sounds like the odds are stacked against you when parallelizing stream pipelines, it’s because they are. An acquaintance who maintains a multimillionline codebase that makes heavy use of streams found only a handful of places where parallel streams were effective. This does not mean that you should refrain from parallelizing streams. Under the right circumstances, it is possible to achieve near-linear speedup in the number of processor cores simply by adding a parallel call to a stream pipeline. Certain domains, such as machine learning and data processing, are particularly amenable to these speedups.

如果在并行化流管道时，听起来你的胜算非常大，那是因为它们确实如此。一位熟悉的人维护着大量使用流的数百万在线代码库，他发现只有少数几个地方并行流是有效的。这并不意味着你应该避免并行化流。在适当的情况下，可以通过向流管道添加并行调用来实现处理器内核数量的近乎线性的加速。某些领域，如机器学习和数据处理，特别适合于这些加速。

As a simple example of a stream pipeline where parallelism is effective, consider this function for computing π(n), the number of primes less than or equal to n:

作为一个简单的例子，一个流管道并行性是有效的，考虑这个函数计算 `π(n)`，质数数目小于或等于 n：

```Java
// Prime-counting stream pipeline - benefits from parallelization
static long pi(long n) {
    return LongStream.rangeClosed(2, n)
    .mapToObj(BigInteger::valueOf)
    .filter(i -> i.isProbablePrime(50))
    .count();
}
```

On my machine, it takes 31 seconds to compute π(108) using this function. Simply adding a parallel() call reduces the time to 9.2 seconds:

在我的机器上，需要 31 秒计算 `π(108)` 使用这个函数。简单地添加 `parallel()` 调用将时间缩短到 9.2 秒：

```Java
// Prime-counting stream pipeline - parallel version
static long pi(long n) {
    return LongStream.rangeClosed(2, n)
    .parallel()
    .mapToObj(BigInteger::valueOf)
    .filter(i -> i.isProbablePrime(50))
    .count();
}
```

In other words, parallelizing the computation speeds it up by a factor of 3.7 on my quad-core machine. It’s worth noting that this is not how you’d compute π(n) for large values of n in practice. There are far more efficient algorithms, notably Lehmer’s formula.

换句话说，在我的四核计算机上，并行化的计算速度提高了 3.7 倍。值得注意的是，这不是你如何计算 `π(n)` 为大 n 的值。有更有效的算法，特别是 Lehmer 公式。

If you are going to parallelize a stream of random numbers, start with a SplittableRandom instance rather than a ThreadLocalRandom (or the essentially obsolete Random). SplittableRandom is designed for precisely this use, and has the potential for linear speedup. ThreadLocalRandom is designed for use by a single thread, and will adapt itself to function as a parallel stream source, but won’t be as fast as SplittableRandom. Random synchronizes on every operation, so it will result in excessive, parallelism-killing contention.

如果要并行化一个随机数流，可以从一个 SplittableRandom 实例开始，而不是从一个 ThreadLocalRandom（或者本质上已经过时的 random）开始。SplittableRandom 正是为这种用途而设计的，它具有线性加速的潜力。ThreadLocalRandom 是为单个线程设计的，它将自适应为并行流源，但速度没有 SplittableRandom 快。随机同步每个操作，因此它将导致过度的并行争用。

In summary, do not even attempt to parallelize a stream pipeline unless you have good reason to believe that it will preserve the correctness of the computation and increase its speed. The cost of inappropriately parallelizing a stream can be a program failure or performance disaster. If you believe that parallelism may be justified, ensure that your code remains correct when run in parallel, and do careful performance measurements under realistic conditions. If your code remains correct and these experiments bear out your suspicion of increased performance, then and only then parallelize the stream in production code.

总之，甚至不要尝试并行化流管道，除非你有充分的理由相信它将保持计算的正确性以及提高速度。不适当地并行化流的代价可能是程序失败或性能灾难。如果你认为并行性是合理的，那么请确保你的代码在并行运行时保持正确，并在实际情况下进行仔细的性能度量。如果你的代码保持正确，并且这些实验证实了你对提高性能的怀疑，那么，并且只有这样，才能在生产代码中并行化流。

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-7/Chapter-7-Introduction.md)**
- **Previous Item（上一条目）：[Item 47: Prefer Collection to Stream as a return type（优先选择 Collection 而不是流作为返回类型）](/Chapter-7/Chapter-7-Item-47-Prefer-Collection-to-Stream-as-a-return-type.md)**
- **Next Item（下一条目）：[Chapter 8 Introduction（章节介绍）](/Chapter-8/Chapter-8-Introduction.md)**
