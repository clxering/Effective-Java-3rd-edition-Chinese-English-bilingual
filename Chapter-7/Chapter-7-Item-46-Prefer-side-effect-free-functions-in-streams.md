## Chapter 7. Lambdas and Streams（λ 表达式和流）

### Item 46: Prefer side-effect-free functions in streams（在流中使用无副作用的函数）

If you’re new to streams, it can be difficult to get the hang of them. Merely expressing your computation as a stream pipeline can be hard. When you succeed, your program will run, but you may realize little if any benefit. Streams isn’t just an API, it’s a paradigm based on functional programming. In order to obtain the expressiveness, speed, and in some cases parallelizability that streams have to offer, you have to adopt the paradigm as well as the API.

如果您是流的新手，可能很难掌握它们。仅仅将计算表示为流管道是困难的。当你成功时，你的程序将运行，但你可能意识不到什么好处。流不仅仅是一个 API，它是一个基于函数式编程的范例。为了获得流必须提供的可表达性、速度以及在某些情况下的并行性，您必须采用范例和 API。

The most important part of the streams paradigm is to structure your computation as a sequence of transformations where the result of each stage is as close as possible to a pure function of the result of the previous stage. A pure function is one whose result depends only on its input: it does not depend on any mutable state, nor does it update any state. In order to achieve this, any function objects that you pass into stream operations, both intermediate and terminal, should be free of side-effects.

streams 范例中最重要的部分是将计算构造为一系列转换，其中每个阶段的结果都尽可能地接近上一阶段结果的纯函数。纯函数的结果只依赖于它的输入：它不依赖于任何可变状态，也不更新任何状态。为了实现这一点，您传递到流操作（包括中间操作和终端操作）中的任何函数对象都应该没有副作用。

Occasionally, you may see streams code that looks like this snippet, which builds a frequency table of the words in a text file:

偶尔，您可能会看到如下的 streams 代码片段，它用于构建文本文件中单词的频率表：

```
// Uses the streams API but not the paradigm--Don't do this!
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()) {
    words.forEach(word -> {
        freq.merge(word.toLowerCase(), 1L, Long::sum);
    });
}
```

What’s wrong with this code? After all, it uses streams, lambdas, and method references, and gets the right answer. Simply put, it’s not streams code at all; it’s iterative code masquerading as streams code. It derives no benefits from the streams API, and it’s (a bit) longer, harder to read, and less maintainable than the corresponding iterative code. The problem stems from the fact that this code is doing all its work in a terminal forEach operation, using a lambda that mutates external state (the frequency table). A forEach operation that does anything more than present the result of the computation performed by a stream is a “bad smell in code,” as is a lambda that mutates state. So how should this code look?

这段代码怎么了？毕竟，它使用了流、lambda 表达式和方法引用，并得到了正确的答案。简单地说，它根本不是流代码；它是伪装成流代码的迭代代码。它没有从 streams API 中获得任何好处，而且它（稍微）比相应的迭代代码更长、更难于阅读和更难以维护。这个问题源于这样一个事实：这段代码在一个终端 forEach 操作中执行它的所有工作，使用一个会改变外部状态的 lambda 表达式（频率表）。forEach 操作除了显示流执行的计算结果之外，还会执行其他操作，这是一种「代码中的异味」，就像 lambda 表达式会改变状态一样。那么这段代码应该是什么样的呢？

```
// Proper use of streams to initialize a frequency table
Map<String, Long> freq;
try (Stream<String> words = new Scanner(file).tokens()) {
    freq = words.collect(groupingBy(String::toLowerCase, counting()));
}
```

This snippet does the same thing as the previous one but makes proper use of the streams API. It’s shorter and clearer. So why would anyone write it the other way? Because it uses tools they’re already familiar with. Java programmers know how to use for-each loops, and the forEach terminal operation is similar. But the forEach operation is among the least powerful of the terminal operations and the least stream-friendly. It’s explicitly iterative, and hence not amenable to parallelization. **The forEach operation should be used only to report the result of a stream computation, not to perform the computation.** Occasionally, it makes sense to use forEach for some other purpose, such as adding the results of a stream computation to a preexisting collection.

这个代码片段与前面的代码片段做了相同的事情，但是正确地使用了 streams API。它更短更清晰。为什么有人会用另一种方式写呢？因为它使用了他们已经熟悉的工具。Java 程序员知道如何使用 for-each 循环，并且 forEach 终端操作是类似的。但是 forEach 操作是终端操作中功能最弱的操作之一，对流最不友好。它是显式迭代的，因此不适合并行化。forEach 操作应该只用于报告流计算的结果，而不是执行计算。有时候，将 forEach 用于其他目的是有意义的，例如将流计算的结果添加到现有集合中。

The improved code uses a collector, which is a new concept that you have to learn in order to use streams. The Collectors API is intimidating: it has thirty-nine methods, some of which have as many as five type parameters. The good news is that you can derive most of the benefit from this API without delving into its full complexity. For starters, you can ignore the Collector interface and think of a collector as an opaque object that encapsulates a reduction strategy. In this context, reduction means combining the elements of a stream into a single object. The object produced by a collector is typically a collection (which accounts for the name collector).

改进后的代码使用了 collector，这是使用流必须学习的新概念。collector 的 API 令人生畏：它有 39 个方法，其中一些方法有多达 5 个类型参数。好消息是，您可以从这个 API 中获得大部分好处，而不必深入研究它的全部复杂性。对于初学者，可以忽略 Collector 接口，将 collector 视为封装了缩减策略的不透明对象。在这种情况下，缩减意味着将流的元素组合成单个对象。collector 生成的对象通常是一个集合（它解释了名称 collector）。

The collectors for gathering the elements of a stream into a true Collection are straightforward. There are three such collectors: toList(), toSet(), and toCollection(collectionFactory). They return, respectively, a set, a list, and a programmer-specified collection type. Armed with this knowledge, we can write a stream pipeline to extract a top-ten list from our frequency table.

将流的元素收集到一个真正的集合中的收集器非常简单。这样的收集器有三种：toList()、toSet() 和 toCollection(collectionFactory)。它们分别返回集合、列表和程序员指定的集合类型。有了这些知识，我们就可以编写一个流管道来从频率表中提取前 10 个列表。

```
// Pipeline to get a top-ten list of words from a frequency table
List<String> topTen = freq.keySet().stream()
    .sorted(comparing(freq::get).reversed())
    .limit(10)
    .collect(toList());
```

Note that we haven’t qualified the toList method with its class, Collectors. **It is customary and wise to statically import all members of Collectors because it makes stream pipelines more readable.**

注意，我们还没有用它的类 Collectors 对 toList 方法进行限定。静态导入收集器的所有成员是习惯和明智的，因为它使流管道更具可读性。

The only tricky part of this code is the comparator that we pass to sorted, comparing(freq::get).reversed(). The comparing method is a comparator construction method (Item 14) that takes a key extraction function. The function takes a word, and the “extraction” is actually a table lookup: the bound method reference freq::get looks up the word in the frequency table and returns the number of times the word appears in the file. Finally, we call reversed on the comparator, so we’re sorting the words from most frequent to least frequent. Then it’s a simple matter to limit the stream to ten words and collect them into a list.

这段代码中唯一棘手的部分是我们传递给ordered的comparator, compare (freq::get).reverse()。比较方法是一种比较器构造方法（[Item-14](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-3/Chapter-3-Item-14-Consider-implementing-Comparable.md)），它具有键提取功能。函数接受一个单词，而“提取”实际上是一个表查找:绑定方法引用freq::get在频率表中查找该单词，并返回该单词在文件中出现的次数。最后，我们在比较器上调用reverse函数，我们将单词从最频繁排序到最不频繁排序。然后，将流限制为10个单词并将它们收集到一个列表中就很简单了。

The previous code snippets use Scanner’s stream method to get a stream over the scanner. This method was added in Java 9. If you’re using an earlier release, you can translate the scanner, which implements Iterator, into a stream using an adapter similar to the one in Item 47 (streamOf(Iterable<E>)).

So what about the other thirty-six methods in Collectors? Most of them exist to let you collect streams into maps, which is far more complicated than collecting them into true collections. Each stream element is associated with a key and a value, and multiple stream elements can be associated with the same key.

The simplest map collector is toMap(keyMapper, valueMapper), which takes two functions, one of which maps a stream element to a key, the other, to a value. We used this collector in our fromString implementation in Item 34 to make a map from the string form of an enum to the enum itself:

```
// Using a toMap collector to make a map from string to enum
private static final Map<String, Operation> stringToEnum =Stream.of(values()).collect(toMap(Object::toString, e -> e));
```

This simple form of toMap is perfect if each element in the stream maps to a unique key. If multiple stream elements map to the same key, the pipeline will terminate with an IllegalStateException.

The more complicated forms of toMap, as well as the groupingBy method, give you various ways to provide strategies for dealing with such collisions. One way is to provide the toMap method with a merge function in addition to its key and value mappers. The merge function is a BinaryOperator<V>, where V is the value type of the map. Any additional values associated with a key are combined with the existing value using the merge function, so, for example, if the merge function is multiplication, you end up with a value that is the product of all the values associated with the key by the value mapper.

The three-argument form of toMap is also useful to make a map from a key to a chosen element associated with that key. For example, suppose we have a stream of record albums by various artists, and we want a map from recording artist to best-selling album. This collector will do the job.

```
// Collector to generate a map from key to chosen element for key
Map<Artist, Album> topHits = albums.collect(
        toMap(Album::artist, a->a, maxBy(comparing(Album::sales)
    )
));
```

Note that the comparator uses the static factory method maxBy, which is statically imported from BinaryOperator. This method converts a Comparator<T> into a BinaryOperator<T> that computes the maximum implied by the specified comparator. In this case, the comparator is returned by the comparator construction method comparing, which takes the key extractor function Album::sales. This may seem a bit convoluted, but the code reads nicely. Loosely speaking, it says, “convert the stream of albums to a map, mapping each artist to the album that has the best album by sales.” This is surprisingly close to the problem statement.

Another use of the three-argument form of toMap is to produce a collector that imposes a last-write-wins policy when there are collisions. For many streams, the results will be nondeterministic, but if all the values that may be associated with a key by the mapping functions are identical, or if they are all acceptable, this collector’s s behavior may be just what you want:

```
// Collector to impose last-write-wins policy
toMap(keyMapper, valueMapper, (v1, v2) -> v2)
```

The third and final version of toMap takes a fourth argument, which is a map factory, for use when you want to specify a particular map implementation such as an EnumMap or a TreeMap.

There are also variant forms of the first three versions of toMap, named toConcurrentMap, that run efficiently in parallel and produce ConcurrentHashMap instances.

In addition to the toMap method, the Collectors API provides the groupingBy method, which returns collectors to produce maps that group elements into categories based on a classifier function. The classifier function takes an element and returns the category into which it falls. This category serves as the element’s map key. The simplest version of the groupingBy method takes only a classifier and returns a map whose values are lists of all the elements in each category. This is the collector that we used in the Anagram program in Item 45 to generate a map from alphabetized word to a list of the words sharing the alphabetization:

```
words.collect(groupingBy(word -> alphabetize(word)))
```

If you want groupingBy to return a collector that produces a map with values other than lists, you can specify a downstream collector in addition to a classifier. A downstream collector produces a value from a stream containing all the elements in a category. The simplest use of this parameter is to pass toSet(), which results in a map whose values are sets of elements rather than lists.

Alternatively, you can pass toCollection(collectionFactory), which lets you create the collections into which each category of elements is placed. This gives you the flexibility to choose any collection type you want. Another simple use of the two-argument form of groupingBy is to pass counting() as the downstream collector. This results in a map that associates each category with the number of elements in the category, rather than a collection containing the elements. That’s what you saw in the frequency table example at the beginning of this item:

```
Map<String, Long> freq = words.collect(groupingBy(String::toLowerCase, counting()));
```

The third version of groupingBy lets you specify a map factory in addition to a downstream collector. Note that this method violates the standard telescoping argument list pattern: the mapFactory parameter precedes, rather than follows, the downStream parameter. This version of groupingBy gives you control over the containing map as well as the contained collections, so, for example, you can specify a collector that returns a TreeMap whose values are TreeSets.

The groupingByConcurrent method provides variants of all three overloadings of groupingBy. These variants run efficiently in parallel and produce ConcurrentHashMap instances. There is also a rarely used relative of groupingBy called partitioningBy. In lieu of a classifier method, it takes a predicate and returns a map whose key is a Boolean. There are two overloadings of this method, one of which takes a downstream collector in addition to a predicate.

The collectors returned by the counting method are intended only for use as downstream collectors. The same functionality is available directly on Stream, via the count method, so **there is never a reason to say collect(counting()).** There are fifteen more Collectors methods with this property. They include the nine methods whose names begin with summing, averaging, and summarizing (whose functionality is available on the corresponding primitive stream types). They also include all overloadings of the reducing method, and the filtering, mapping, flatMapping, and collectingAndThen methods. Most programmers can safely ignore the majority of these methods. From a design perspective, these collectors represent an attempt to partially duplicate the functionality of streams in collectors so that downstream collectors can act as “ministreams.”

There are three Collectors methods we have yet to mention. Though they are in Collectors, they don’t involve collections. The first two are minBy and maxBy, which take a comparator and return the minimum or maximum element in the stream as determined by the comparator. They are minor generalizations of the min and max methods in the Stream interface and are the collector analogues of the binary operators returned by the like-named methods in BinaryOperator. Recall that we used BinaryOperator.maxBy in our best-selling album example.

The final Collectors method is joining, which operates only on streams of CharSequence instances such as strings. In its parameterless form, it returns a collector that simply concatenates the elements. Its one argument form takes a single CharSequence parameter named delimiter and returns a collector that joins the stream elements, inserting the delimiter between adjacent elements. If you pass in a comma as the delimiter, the collector returns a comma-separated values string (but beware that the string will be ambiguous if any of the elements in the stream contain commas). The three argument form takes a prefix and suffix in addition to the delimiter. The resulting collector generates strings like the ones that you get when you print a collection, for example [came, saw, conquered].

In summary, the essence of programming stream pipelines is side-effect-free function objects. This applies to all of the many function objects passed to streams and related objects. The terminal operation forEach should only be used to report the result of a computation performed by a stream, not to perform the computation. In order to use streams properly, you have to know about collectors. The most important collector factories are toList, toSet, toMap, groupingBy, and joining.

---
**[Back to contents of the chapter（返回章节目录）](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-7/Chapter-7-Introduction.md)**
- **Previous Item（上一条目）：[Item 45: Use streams judiciously（明智地使用流）](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-7/Chapter-7-Item-45-Use-streams-judiciously.md)**
- **Next Item（下一条目）：[Item 47: Prefer Collection to Stream as a return type（优先选择 Collection 而不是流作为返回类型）](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-7/Chapter-7-Item-47-Prefer-Collection-to-Stream-as-a-return-type.md)**
