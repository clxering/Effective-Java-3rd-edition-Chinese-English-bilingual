## Chapter 7. Lambdas and Streams（λ 表达式和流）

### Item 47: Prefer Collection to Stream as a return type（优先选择 Collection 而不是流作为返回类型）

Many methods return sequences of elements. Prior to Java 8, the obvious return types for such methods were the collection interfaces Collection, Set, and List; Iterable; and the array types. Usually, it was easy to decide which of these types to return. The norm was a collection interface. If the method existed solely to enable for-each loops or the returned sequence couldn’t be made to implement some Collection method (typically, contains(Object)), the Iterable interface was used. If the returned elements were primitive values or there were stringent performance requirements, arrays were used. In Java 8, streams were added to the platform, substantially complicating the task of choosing the appropriate return type for a sequence-returning method.

许多方法都返回元素序列。在 Java 8 之前，此类方法常见的返回类型是 Collection 集合接口，如 Set 和 List，另外还有 Iterable 以及数组类型。通常，很容易决定使用哪一种类型。标准是一个集合接口。如果方法的存在仅仅是为了支持 for-each 循环，或者无法使返回的序列实现某个集合方法（通常是 `contains(Object)`），则使用 Iterable 接口。如果返回的元素是基本数据类型或有严格的性能要求，则使用数组。在 Java 8 中，流被添加进来，这大大增加了为序列返回方法选择适当返回类型的复杂性。

You may hear it said that streams are now the obvious choice to return a sequence of elements, but as discussed in Item 45, streams do not make iteration obsolete: writing good code requires combining streams and iteration judiciously. If an API returns only a stream and some users want to iterate over the returned sequence with a for-each loop, those users will be justifiably upset. It is especially frustrating because the Stream interface contains the sole abstract method in the Iterable interface, and Stream’s specification for this method is compatible with Iterable’s. The only thing preventing programmers from using a for-each loop to iterate over a stream is Stream’s failure to extend Iterable.

你可能听说现在流是返回元素序列的明显选择，但是正如 [Item-45](/Chapter-7/Chapter-7-Item-45-Use-streams-judiciously.md) 中所讨论的，流不会让迭代过时：编写好的代码需要明智地将流和迭代结合起来。如果一个 API 只返回一个流，而一些用户希望使用 for-each 循环遍历返回的序列，那么这些用户将会感到不适。这尤其令人沮丧，因为流接口包含 Iterable 接口中惟一的抽象方法，而且流对该方法的规范与 Iterable 的规范兼容。唯一阻止程序员使用 for-each 循环在流上迭代的是流不能扩展 Iterable。

Sadly, there is no good workaround for this problem. At first glance, it might appear that passing a method reference to Stream’s iterator method would work. The resulting code is perhaps a bit noisy and opaque, but not unreasonable:

遗憾的是，这个问题没有好的解决办法。乍一看，似乎将方法引用传递给流的 iterator 方法是可行的。生成的代码可能有点繁琐，不易理解，但并非不合理：

```Java
// Won't compile, due to limitations on Java's type inference
for (ProcessHandle ph : ProcessHandle.allProcesses()::iterator) {
    // Process the process
}
```

Unfortunately, if you attempt to compile this code, you’ll get an error message:

不幸的是，如果你试图编译这段代码，你会得到一个错误消息：

```Java
Test.java:6: error: method reference not expected here
for (ProcessHandle ph : ProcessHandle.allProcesses()::iterator) {
^
```

In order to make the code compile, you have to cast the method reference to an appropriately parameterized Iterable:

为了编译代码，你必须将方法引用转换为适当参数化的 Iterable：

```Java
// Hideous workaround to iterate over a stream
for (ProcessHandle ph : (Iterable<ProcessHandle>)ProcessHandle.allProcesses()::iterator)
```

This client code works, but it is too noisy and opaque to use in practice. A better workaround is to use an adapter method. The JDK does not provide such a method, but it’s easy to write one, using the same technique used in-line in the snippets above. Note that no cast is necessary in the adapter method because Java’s type inference works properly in this context:

这个客户端代码可以工作，但是它太过繁琐并不易理解，无法在实践中使用。更好的解决方案是使用适配器方法。JDK 没有提供这样的方法，但是使用上面代码片段中使用的内联技术编写方法很容易。注意，适配器方法中不需要强制转换，因为 Java 的类型推断在此上下文中工作正常：

```Java
// Adapter from Stream<E> to Iterable<E>
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
    return stream::iterator;
}
```

With this adapter, you can iterate over any stream with a for-each statement:

使用此适配器，你可以使用 for-each 语句遍历任何流：

```Java
for (ProcessHandle p : iterableOf(ProcessHandle.allProcesses())) {
    // Process the process
}
```

Note that the stream versions of the Anagrams program in Item 34 use the Files.lines method to read the dictionary, while the iterative version uses a scanner. The Files.lines method is superior to a scanner, which silently swallows any exceptions encountered while reading the file. Ideally, we would have used Files.lines in the iterative version too. This is the sort of compromise that programmers will make if an API provides only stream access to a sequence and they want to iterate over the sequence with a for-each statement.

注意，[Item-34](/Chapter-6/Chapter-6-Item-34-Use-enums-instead-of-int-constants.md) 中 Anagrams 程序的流版本使用 `Files.lines` 读取字典，而迭代版本使用扫描器。`Files.lines` 方法优于扫描器，扫描器在读取文件时静默地接收任何异常。理想情况下，我们在 `Files.lines` 的迭代版本也应该如此。如果一个 API 只提供对一个序列的流访问，而程序员希望用 for-each 语句遍历该序列，那么这是程序员会做出的一种妥协。

Conversely, a programmer who wants to process a sequence using a stream pipeline will be justifiably upset by an API that provides only an Iterable. Again the JDK does not provide an adapter, but it’s easy enough to write one:

相反，如果程序员希望使用流管道来处理序列，那么只提供可迭代的 API 就会有理由让他心烦。JDK 同样没有提供适配器，但是编写适配器非常简单：

```Java
// Adapter from Iterable<E> to Stream<E>
public static <E> Stream<E> streamOf(Iterable<E> iterable) {
    return StreamSupport.stream(iterable.spliterator(), false);
}
```

If you’re writing a method that returns a sequence of objects and you know that it will only be used in a stream pipeline, then of course you should feel free to return a stream. Similarly, a method returning a sequence that will only be used for iteration should return an Iterable. But if you’re writing a public API that returns a sequence, you should provide for users who want to write stream pipelines as well as those who want to write for-each statements, unless you have a good reason to believe that most of your users will want to use the same mechanism.

如果你正在编写一个返回对象序列的方法，并且你知道它只会在流管道中使用，那么你当然应该可以随意返回流。类似地，返回仅用于迭代的序列的方法应该返回一个 Iterable。但是如果你写一个公共 API，它返回一个序列，你应该兼顾想写流管道以及想写 for-each 语句的用户，除非你有充分的理由相信大多数用户想要使用相同的机制。

The Collection interface is a subtype of Iterable and has a stream method, so it provides for both iteration and stream access. Therefore, **Collection or an appropriate subtype is generally the best return type for a public, sequence-returning method.** Arrays also provide for easy iteration and stream access with the Arrays.asList and Stream.of methods. If the sequence you’re returning is small enough to fit easily in memory, you’re probably best off returning one of the standard collection implementations, such as ArrayList or HashSet. But **do not store a large sequence in memory just to return it as a collection.**

Collection 接口是 Iterable 的一个子类型，它有一个流方法，因此它提供了迭代和流两种访问方式。因此，**Collection 或其适当的子类通常是公共序列返回方法的最佳返回类型。** 数组还提供了使用 `Arrays.asList` 和 `Stream.of` 方法进行简单迭代和流访问。如果返回的序列足够小，可以轻松地装入内存，那么最好返回标准集合实现之一，例如 ArrayList 或 HashSet。但是 **不要将一个大的序列存储在内存中，只是为了将它作为一个集合返回。**

If the sequence you’re returning is large but can be represented concisely, consider implementing a special-purpose collection. For example, suppose you want to return the power set of a given set, which consists of all of its subsets. The power set of {a, b, c} is {{}, {a}, {b}, {c}, {a, b}, {a, c}, {b, c}, {a, b, c}}. If a set has n elements, its power set has 2n. Therefore, you shouldn’t even consider storing the power set in a standard collection implementation. It is, however, easy to implement a custom collection for the job with the help of AbstractList.

如果返回的序列比较大，但是可以有规律地表示，那么可以考虑实现一个特殊用途的集合。例如，假设你想要返回给定集合的幂集，该集合由它的所有子集组成。`{a, b, c}` 的排列组合有 `{{}, {a}, {b}, {c}, {a, b}, {a, c}, {b, c}, {a, b, c}}`。如果一个集合有 n 个元素，它的幂集有 2<sup>n</sup>。因此，你甚至不应该考虑在标准集合实现中存储全部排列组合。然而，在 AbstractList 的帮助下，可以很容易实现这个需求的自定义集合。

The trick is to use the index of each element in the power set as a bit vector, where the nth bit in the index indicates the presence or absence of the nth element from the source set. In essence, there is a natural mapping between the binary numbers from 0 to 2n − 1 and the power set of an n-element set. Here’s the code:

诀窍是使用索引幂集的每个元素设置一个位向量，在该指数的 n 位表示第 n 个元素的存在与否从源。在本质上，之间有一个自然的映射二进制数字从 0 到 2n−1 和一组 n 元的幂集。这是代码：

```Java
// Returns the power set of an input set as custom collection
public class PowerSet {
    public static final <E> Collection<Set<E>> of(Set<E> s) {
        List<E> src = new ArrayList<>(s);
        if (src.size() > 30)
            throw new IllegalArgumentException("Set too big " + s);

        return new AbstractList<Set<E>>() {
            @Override
            public int size() {
                return 1 << src.size(); // 2 to the power srcSize
            }

            @Override
            public boolean contains(Object o) {
                return o instanceof Set && src.containsAll((Set)o);
            }

            @Override
            public Set<E> get(int index) {
                Set<E> result = new HashSet<>();
                for (int i = 0; index != 0; i++, index >>= 1)
                    if ((index & 1) == 1)
                        result.add(src.get(i));
                return result;
            }
        };
    }
}
```

Note that PowerSet.of throws an exception if the input set has more than 30 elements. This highlights a disadvantage of using Collection as a return type rather than Stream or Iterable: Collection has an int-returning size method, which limits the length of the returned sequence to Integer.MAX_VALUE, or 231 − 1. The Collection specification does allow the size method to return 231 − 1 if the collection is larger, even infinite, but this is not a wholly satisfying solution.

注意，如果输入集包含超过 30 个元素，`PowerSet.of` 将抛出异常。这突出的缺点使用 Collection 作为返回类型而不是流或 Iterable：收集 int-returning 大小的方法,这限制了 Integer.MAX_VALUE 返回序列的长度，或 231−1。收集规范允许大小方法返回 231−1 如果集合更大，甚至是无限的,但这不是一个完全令人满意的解决方案。

In order to write a Collection implementation atop AbstractCollection, you need implement only two methods beyond the one required for Iterable: contains and size. Often it’s easy to write efficient implementations of these methods. If it isn’t feasible, perhaps because the contents of the sequence aren’t predetermined before iteration takes place, return a stream or iterable, whichever feels more natural. If you choose, you can return both using two separate methods.

为了在 AbstractCollection 之上编写 Collection 实现，除了 Iterable 所需的方法外，只需要实现两个方法：contains 和 size。通常很容易编写这些方法的有效实现。如果它是不可行的，可能是因为序列的内容在迭代发生之前没有预先确定，那么返回一个流或 iterable，以感觉更自然的方式返回。如果你选择，你可以使用两个不同的方法返回这两个值。

There are times when you’ll choose the return type based solely on ease of implementation. For example, suppose you want to write a method that returns all of the (contiguous) sublists of an input list. It takes only three lines of code to generate these sublists and put them in a standard collection, but the memory required to hold this collection is quadratic in the size of the source list. While this is not as bad as the power set, which is exponential, it is clearly unacceptable. Implementing a custom collection, as we did for the power set, would be tedious, more so because the JDK lacks a skeletal Iterator implementation to help us.

有时，你将仅根据实现的易用性来选择返回类型。例如，假设你想编写一个返回输入列表的所有（连续的）子列表的方法。生成这些子列表并将它们放入标准集合中只需要三行代码，但是保存该集合所需的内存是源列表大小的二次方。虽然这没有幂集那么糟糕，幂集是指数的，但显然是不可接受的。实现自定义集合（就像我们为 power 集所做的那样）将会非常繁琐，因为 JDK 缺少一个框架迭代器实现来帮助我们。

It is, however, straightforward to implement a stream of all the sublists of an input list, though it does require a minor insight. Let’s call a sublist that contains the first element of a list a prefix of the list. For example, the prefixes of (a, b, c) are (a), (a, b), and (a, b, c). Similarly, let’s call a sublist that contains the last element a suffix, so the suffixes of (a, b, c) are (a, b, c), (b, c), and (c). The insight is that the sublists of a list are simply the suffixes of the prefixes (or identically, the prefixes of the suffixes) and the empty list. This observation leads directly to a clear, reasonably concise implementation:

然而，实现一个输入列表的所有子列表的流是很简单的，尽管它确实需要一些深入的了解。让我们将包含列表的第一个元素的子列表称为列表的前缀。例如，`(a,b,c)` 的前缀 `(a)`、`(a、b)` 和 `(a,b,c)`。类似地，让我们调用包含最后一个元素后缀的子列表，因此 `(a, b, c)` 的后缀是 `(a, b, c)`、`(b, c)` 和 `(c)`。我们的理解是，列表的子列表仅仅是前缀的后缀（或后缀的前缀相同）和空列表。这个观察直接导致了一个清晰、合理、简洁的实现：

```Java
// Returns a stream of all the sublists of its input list
public class SubLists {
    public static <E> Stream<List<E>> of(List<E> list) {
        return Stream.concat(Stream.of(Collections.emptyList()),prefixes(list).flatMap(SubLists::suffixes));
    }

    private static <E> Stream<List<E>> prefixes(List<E> list) {
        return IntStream.rangeClosed(1, list.size()).mapToObj(end -> list.subList(0, end));
    }

    private static <E> Stream<List<E>> suffixes(List<E> list) {
        return IntStream.range(0, list.size()).mapToObj(start -> list.subList(start, list.size()));
    }
}
```

Note that the Stream.concat method is used to add the empty list into the returned stream. Also note that the flatMap method (Item 45) is used to generate a single stream consisting of all the suffixes of all the prefixes. Finally, note that we generate the prefixes and suffixes by mapping a stream of consecutive int values returned by IntStream.range and IntStream.rangeClosed. This idiom is, roughly speaking, the stream equivalent of the standard for-loop on integer indices. Thus, our sublist implementation is similar in spirit to the obvious nested for-loop:

注意 `Stream.concat` 方法将空列表添加到返回的流中。还要注意，flatMap 方法（[Item-45](/Chapter-7/Chapter-7-Item-45-Use-streams-judiciously.md)）用于生成由所有前缀的所有后缀组成的单一流。最后，请注意，我们通过映射由 `IntStream.range` 和 `IntStream.rangeClosed` 返回的连续 int 值流来生成前缀和后缀。因此，我们的子列表实现在本质上类似于嵌套的 for 循环：

```Java
for (int start = 0; start < src.size(); start++)
    for (int end = start + 1; end <= src.size(); end++)
        System.out.println(src.subList(start, end));
```

It is possible to translate this for-loop directly into a stream. The result is more concise than our previous implementation, but perhaps a bit less readable. It is similar in spirit to the streams code for the Cartesian product in Item 45:

可以将这个 for 循环直接转换为流。结果比我们以前的实现更简洁，但可读性可能稍差。它在形态上类似于 [Item-45](/Chapter-7/Chapter-7-Item-45-Use-streams-judiciously.md) 中 Cartesian 的 streams 代码：

```Java
// Returns a stream of all the sublists of its input list
public static <E> Stream<List<E>> of(List<E> list) {
    return IntStream.range(0, list.size())
    .mapToObj(start ->
    IntStream.rangeClosed(start + 1, list.size())
    .mapToObj(end -> list.subList(start, end)))
    .flatMap(x -> x);
}
```

Like the for-loop that precedes it, this code does not emit the empty list. In order to fix this deficiency, you could either use concat, as we did in the previous version, or replace 1 by (int) Math.signum(start) in the rangeClosed call.

与前面的 for 循环一样，该代码不发出空列表。为了修复这个缺陷，你可以使用 concat，就像我们在上一个版本中所做的那样，或者在 rangeClosed 调用中将 1 替换为 `(int) Math.signum(start)`。

Either of these stream implementations of sublists is fine, but both will require some users to employ a Stream-to-Iterable adapter or to use a stream in places where iteration would be more natural. Not only does the Stream-to- Iterable adapter clutter up client code, but it slows down the loop by a factor of 2.3 on my machine. A purpose-built Collection implementation (not shown here) is considerably more verbose but runs about 1.4 times as fast as our stream-based implementation on my machine.

子列表的这两种流实现都可以，但是都需要一些用户使用流到迭代的适配器，或者在迭代更自然的地方使用流。流到迭代适配器不仅打乱了客户机代码，而且在我的机器上，它还将循环速度降低了 2.3 倍。专门构建的集合实现（这里没有显示）非常冗长，但是运行速度是我的机器上基于流的实现的 1.4 倍。

In summary, when writing a method that returns a sequence of elements, remember that some of your users may want to process them as a stream while others may want to iterate over them. Try to accommodate both groups. If it’s feasible to return a collection, do so. If you already have the elements in a collection or the number of elements in the sequence is small enough to justify creating a new one, return a standard collection such as ArrayList. Otherwise, consider implementing a custom collection as we did for the power set. If it isn’t feasible to return a collection, return a stream or iterable, whichever seems more natural. If, in a future Java release, the Stream interface declaration is modified to extend Iterable, then you should feel free to return streams because they will allow for both stream processing and iteration.

总之，在编写返回元素序列的方法时，请记住，有些用户可能希望将它们作为流处理，而有些用户可能希望对它们进行迭代。试着适应这两个群体。如果可以返回集合，那么就这样做。如果你已经在一个集合中拥有了元素，或者序列中的元素数量足够小，可以创建一个新的元素，那么返回一个标准集合，例如 ArrayList 。否则，请考虑像对 power 集那样实现自定义集合。如果返回集合不可行，则返回流或 iterable，以看起来更自然的方式返回。如果在未来的 Java 版本中，流接口声明被修改为可迭代的，那么你应该可以随意返回流，因为它们将允许流处理和迭代。

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-7/Chapter-7-Introduction.md)**
- **Previous Item（上一条目）：[Item 46: Prefer side effect free functions in streams（在流中使用无副作用的函数）](/Chapter-7/Chapter-7-Item-46-Prefer-side-effect-free-functions-in-streams.md)**
- **Next Item（下一条目）：[Item 48: Use caution when making streams parallel（谨慎使用并行流）](/Chapter-7/Chapter-7-Item-48-Use-caution-when-making-streams-parallel.md)**
