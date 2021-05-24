## Chapter 7. Lambdas and Streams（λ 表达式和流）

### Item 46: Prefer side-effect-free functions in streams（在流中使用无副作用的函数）

If you’re new to streams, it can be difficult to get the hang of them. Merely expressing your computation as a stream pipeline can be hard. When you succeed, your program will run, but you may realize little if any benefit. Streams isn’t just an API, it’s a paradigm based on functional programming. In order to obtain the expressiveness, speed, and in some cases parallelizability that streams have to offer, you have to adopt the paradigm as well as the API.

如果你是流的新手，可能很难掌握它们。仅仅将计算表示为流管道是困难的。当你成功时，你的程序可以运行，但你可能意识不到什么好处。流不仅仅是一个 API，它是一个基于函数式编程的范式。为了获得流提供的可表达性、速度以及在某些情况下的并行性，你必须采纳范式和 API。

The most important part of the streams paradigm is to structure your computation as a sequence of transformations where the result of each stage is as close as possible to a pure function of the result of the previous stage. A pure function is one whose result depends only on its input: it does not depend on any mutable state, nor does it update any state. In order to achieve this, any function objects that you pass into stream operations, both intermediate and terminal, should be free of side-effects.

流范式中最重要的部分是将计算构造为一系列转换，其中每个阶段的结果都尽可能地接近上一阶段结果的纯函数。纯函数的结果只依赖于它的输入：它不依赖于任何可变状态，也不更新任何状态。为了实现这一点，传递到流操作（包括 Intermediate 操作和 Terminal 操作）中的任何函数对象都应该没有副作用。

**译注：流的操作类型分为以下几种：**

**1、Intermediate**
- 一个流可以后面跟随零个或多个 intermediate 操作。其目的主要是打开流，做出某种程度的数据映射/过滤，然后返回一个新的流，交给下一个操作使用。这类操作都是惰性化的（lazy），就是说，仅仅调用到这类方法，并没有真正开始流的遍历。常见的操作：map（mapToInt、flatMap 等）、filter、distinct、sorted、peek、limit、skip、parallel、sequential、unordered

**2、Terminal**
- 一个流只能有一个 terminal 操作，当这个操作执行后，流就被使用「光」了，无法再被操作。所以这必定是流的最后一个操作。Terminal 操作的执行，才会真正开始流的遍历，并且会生成一个结果，或者一个 side effect。常见的操作：forEach、forEachOrdered、toArray、reduce、collect、min、max、count、anyMatch、allMatch、noneMatch、findFirst、findAny、iterator

- 在对于一个流进行多次转换操作 (Intermediate 操作)，每次都对流的每个元素进行转换，而且是执行多次，这样时间复杂度就是 N（转换次数）个 for 循环里把所有操作都做掉的总和吗？其实不是这样的，转换操作都是 lazy 的，多个转换操作只会在 Terminal 操作的时候融合起来，一次循环完成。我们可以这样简单的理解，流里有个操作函数的集合，每次转换操作就是把转换函数放入这个集合中，在 Terminal 操作的时候循环流对应的集合，然后对每个元素执行所有的函数。

**3、short-circuiting**
- 对于一个 intermediate 操作，如果它接受的是一个无限大（infinite/unbounded）的流，但返回一个有限的新流。

- 对于一个 terminal 操作，如果它接受的是一个无限大的流，但能在有限的时间计算出结果。当操作一个无限大的流，而又希望在有限时间内完成操作，则在管道内拥有一个 short-circuiting 操作是必要非充分条件。常见的操作：anyMatch、allMatch、 noneMatch、findFirst、findAny、limit

Occasionally, you may see streams code that looks like this snippet, which builds a frequency table of the words in a text file:

偶尔，你可能会看到如下使用流的代码片段，它用于构建文本文件中单词的频率表：

```Java
// Uses the streams API but not the paradigm--Don't do this!
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()) {
    words.forEach(word -> {
        freq.merge(word.toLowerCase(), 1L, Long::sum);
    });
}
```

What’s wrong with this code? After all, it uses streams, lambdas, and method references, and gets the right answer. Simply put, it’s not streams code at all; it’s iterative code masquerading as streams code. It derives no benefits from the streams API, and it’s (a bit) longer, harder to read, and less maintainable than the corresponding iterative code. The problem stems from the fact that this code is doing all its work in a terminal forEach operation, using a lambda that mutates external state (the frequency table). A forEach operation that does anything more than present the result of the computation performed by a stream is a “bad smell in code,” as is a lambda that mutates state. So how should this code look?

这段代码有什么问题？毕竟，它使用了流、lambda 表达式和方法引用，并得到了正确的答案。简单地说，它根本不是流代码，而是伪装成流代码的迭代代码。它没有从流 API 中获得任何好处，而且它（稍微）比相应的迭代代码更长、更难于阅读和更难以维护。这个问题源于这样一个事实：这段代码在一个 Terminal  操作中（forEach）执行它的所有工作，使用一个会改变外部状态的 lambda 表达式（频率表）。forEach 操作除了显示流执行的计算结果之外，还会执行其他操作，这是一种「代码中的不良习惯」，就像 lambda 表达式会改变状态一样。那么这段代码应该是什么样的呢？

```Java
// Proper use of streams to initialize a frequency table
Map<String, Long> freq;
try (Stream<String> words = new Scanner(file).tokens()) {
    freq = words.collect(groupingBy(String::toLowerCase, counting()));
}
```

This snippet does the same thing as the previous one but makes proper use of the streams API. It’s shorter and clearer. So why would anyone write it the other way? Because it uses tools they’re already familiar with. Java programmers know how to use for-each loops, and the forEach terminal operation is similar. But the forEach operation is among the least powerful of the terminal operations and the least stream-friendly. It’s explicitly iterative, and hence not amenable to parallelization. **The forEach operation should be used only to report the result of a stream computation, not to perform the computation.** Occasionally, it makes sense to use forEach for some other purpose, such as adding the results of a stream computation to a preexisting collection.

这个代码片段与前面的代码片段做了相同的事情，但是正确地使用了流 API。它更短更清晰。为什么有人会用另一种方式写呢？因为它使用了他们已经熟悉的工具。Java 程序员知道如何使用 for-each 循环，并且与 forEach 操作是类似的。但是 forEach 操作是 Terminal 操作中功能最弱的操作之一，对流最不友好。它是显式迭代的，因此不适合并行化。**forEach 操作应该只用于报告流计算的结果，而不是执行计算。** 有时候，将 forEach 用于其他目的是有意义的，例如将流计算的结果添加到现有集合中。

The improved code uses a collector, which is a new concept that you have to learn in order to use streams. The Collectors API is intimidating: it has thirty-nine methods, some of which have as many as five type parameters. The good news is that you can derive most of the benefit from this API without delving into its full complexity. For starters, you can ignore the Collector interface and think of a collector as an opaque object that encapsulates a reduction strategy. In this context, reduction means combining the elements of a stream into a single object. The object produced by a collector is typically a collection (which accounts for the name collector).

改进后的代码使用了 collector，这是使用流必须学习的新概念。Collectors 的 API 令人生畏：它有 39 个方法，其中一些方法有多达 5 个类型参数。好消息是，你可以从这个 API 中获得大部分好处，而不必深入研究它的全部复杂性。对于初学者，可以忽略 Collector 接口，将 collector 视为封装了缩减策略的不透明对象。在这种情况下，缩减意味着将流的元素组合成单个对象。collector 生成的对象通常是一个集合（这也解释了为何命名为 collector）。

The collectors for gathering the elements of a stream into a true Collection are straightforward. There are three such collectors: toList(), toSet(), and toCollection(collectionFactory). They return, respectively, a set, a list, and a programmer-specified collection type. Armed with this knowledge, we can write a stream pipeline to extract a top-ten list from our frequency table.

将流的元素收集到一个真正的 Collection 中的 collector 非常简单。这样的 collector 有三种：`toList()`、`toSet()` 和 `toCollection(collectionFactory)`。它们分别返回 List、Set 和程序员指定的集合类型。有了这些知识，我们就可以编写一个流管道来从 freq 表中提取前 10 个元素来构成一个新 List。

```Java
// Pipeline to get a top-ten list of words from a frequency table
List<String> topTen = freq.keySet().stream()
    .sorted(comparing(freq::get).reversed())
    .limit(10)
    .collect(toList());
```

Note that we haven’t qualified the toList method with its class, Collectors. **It is customary and wise to statically import all members of Collectors because it makes stream pipelines more readable.**

注意，我们还没有用它的类 Collectors 对 toList 方法进行限定。**静态导入 Collectors 的所有成员是习惯用法，也是明智的，因为这使流管道更具可读性。**

The only tricky part of this code is the comparator that we pass to sorted, comparing(freq::get).reversed(). The comparing method is a comparator construction method (Item 14) that takes a key extraction function. The function takes a word, and the “extraction” is actually a table lookup: the bound method reference freq::get looks up the word in the frequency table and returns the number of times the word appears in the file. Finally, we call reversed on the comparator, so we’re sorting the words from most frequent to least frequent. Then it’s a simple matter to limit the stream to ten words and collect them into a list.

这段代码中唯一棘手的部分是我们传递给 sorted 的 `comparing(freq::get).reversed()`。comparing 方法是 comparator 的一种构造方法（[Item-14](/Chapter-3/Chapter-3-Item-14-Consider-implementing-Comparable.md)），它具有键提取功能。函数接受一个单词，而「提取」实际上是一个表查找：绑定方法引用 `freq::get` 在 freq 表中查找该单词，并返回该单词在文件中出现的次数。最后，我们在比较器上调用 reverse 函数，我们将单词从最频繁排序到最不频繁进行排序。然后，将流限制为 10 个单词并将它们收集到一个列表中。

The previous code snippets use Scanner’s stream method to get a stream over the scanner. This method was added in Java 9. If you’re using an earlier release, you can translate the scanner, which implements Iterator, into a stream using an adapter similar to the one in Item 47 (`streamOf(Iterable<E>)`).

前面的代码片段使用 Scanner 的流方法在扫描器上获取流。这个方法是在 Java 9 中添加的。如果使用的是较早的版本，则可以使用类似于 [Item-47](/Chapter-7/Chapter-7-Item-47-Prefer-Collection-to-Stream-as-a-return-type.md)（`streamOf(Iterable<E>)`）中的适配器将实现 Iterator 的扫描程序转换为流。

So what about the other thirty-six methods in Collectors? Most of them exist to let you collect streams into maps, which is far more complicated than collecting them into true collections. Each stream element is associated with a key and a value, and multiple stream elements can be associated with the same key.

那么 Collectors 中的其他 36 个方法呢？它们中的大多数都允许你将流收集到 Map 中，这比将它们收集到真正的集合要复杂得多。每个流元素与一个键和一个值相关联，多个流元素可以与同一个键相关联。

The simplest map collector is toMap(keyMapper, valueMapper), which takes two functions, one of which maps a stream element to a key, the other, to a value. We used this collector in our fromString implementation in Item 34 to make a map from the string form of an enum to the enum itself:

最简单的 Map 收集器是 `toMap(keyMapper, valueMapper)`，它接受两个函数，一个将流元素映射到键，另一个映射到值。我们在 [Item-34](/Chapter-6/Chapter-6-Item-34-Use-enums-instead-of-int-constants.md) 中的 fromString 实现中使用了这个收集器来创建枚举的字符串形式到枚举本身的映射：

```Java
// Using a toMap collector to make a map from string to enum
private static final Map<String, Operation> stringToEnum =Stream.of(values()).collect(toMap(Object::toString, e -> e));
```

This simple form of toMap is perfect if each element in the stream maps to a unique key. If multiple stream elements map to the same key, the pipeline will terminate with an IllegalStateException.

如果流中的每个元素映射到唯一的键，那么这种简单的 toMap 形式就是完美的。如果多个流元素映射到同一个键，管道将以 IllegalStateException 结束。

The more complicated forms of toMap, as well as the groupingBy method, give you various ways to provide strategies for dealing with such collisions. One way is to provide the toMap method with a merge function in addition to its key and value mappers. The merge function is a `BinaryOperator<V>`, where V is the value type of the map. Any additional values associated with a key are combined with the existing value using the merge function, so, for example, if the merge function is multiplication, you end up with a value that is the product of all the values associated with the key by the value mapper.

toMap 更为复杂的形式，以及 groupingBy 方法，提供了各种方法来提供处理此类冲突的策略。一种方法是为 toMap 方法提供一个 merge 函数，以及它的键和值映射器。merge 函数是一个 `BinaryOperator<V>`，其中 V 是 Map 的值类型。与键关联的任何附加值都将使用 merge 函数与现有值组合，因此，例如，如果 merge 函数是乘法，那么你将得到一个值，该值是 value mapper 与键关联的所有值的乘积。

The three-argument form of toMap is also useful to make a map from a key to a chosen element associated with that key. For example, suppose we have a stream of record albums by various artists, and we want a map from recording artist to best-selling album. This collector will do the job.

toMap 的三参数形式对于从键到与该键关联的所选元素的映射也很有用。例如，假设我们有一个由不同艺术家录制的唱片流，并且我们想要一个从唱片艺术家到畅销唱片的映射。这个 collector 将完成这项工作。

```Java
// Collector to generate a map from key to chosen element for key
Map<Artist, Album> topHits = albums.collect(
        toMap(Album::artist, a->a, maxBy(comparing(Album::sales)
    )
));
```

Note that the comparator uses the static factory method maxBy, which is statically imported from BinaryOperator. This method converts a `Comparator<T>` into a `BinaryOperator<T>` that computes the maximum implied by the specified comparator. In this case, the comparator is returned by the comparator construction method comparing, which takes the key extractor function Album::sales. This may seem a bit convoluted, but the code reads nicely. Loosely speaking, it says, “convert the stream of albums to a map, mapping each artist to the album that has the best album by sales.” This is surprisingly close to the problem statement.

注意，比较器使用静态工厂方法 maxBy，该方法从 BinaryOperator 静态导入。此方法将 `Comparator<T>` 转换为 `BinaryOperator<T>`，该操作符计算指定比较器所隐含的最大值。在这种情况下，比较器是通过比较器构造方法返回的，比较器构造方法取 `Album::sales`。这看起来有点复杂，但是代码可读性很好。粗略地说，代码是这样描述的:「将专辑流转换为 Map，将每个艺人映射到销量最好的专辑。」这与问题的文字陈述惊人地接近。

Another use of the three-argument form of toMap is to produce a collector that imposes a last-write-wins policy when there are collisions. For many streams, the results will be nondeterministic, but if all the values that may be associated with a key by the mapping functions are identical, or if they are all acceptable, this collector’s s behavior may be just what you want:

toMap 的三参数形式的另一个用途是生成一个 collector，当发生冲突时，它强制执行 last-write-wins 策略。对于许多流，结果将是不确定的，但如果映射函数可能与键关联的所有值都是相同的，或者它们都是可接受的，那么这个 collector 的行为可能正是你想要的：

```Java
// Collector to impose last-write-wins policy
toMap(keyMapper, valueMapper, (v1, v2) -> v2)
```

The third and final version of toMap takes a fourth argument, which is a map factory, for use when you want to specify a particular map implementation such as an EnumMap or a TreeMap.

toMap 的第三个也是最后一个版本采用了第四个参数，这是一个 Map 工厂，当你想要指定一个特定的 Map 实现（如 EnumMap 或 TreeMap）时，可以使用它。

There are also variant forms of the first three versions of toMap, named toConcurrentMap, that run efficiently in parallel and produce ConcurrentHashMap instances.

还有前三个版本的 toMap 的变体形式，名为 toConcurrentMap，它们可以有效地并行运行，同时生成 ConcurrentHashMap 实例。

In addition to the toMap method, the Collectors API provides the groupingBy method, which returns collectors to produce maps that group elements into categories based on a classifier function. The classifier function takes an element and returns the category into which it falls. This category serves as the element’s map key. The simplest version of the groupingBy method takes only a classifier and returns a map whose values are lists of all the elements in each category. This is the collector that we used in the Anagram program in Item 45 to generate a map from alphabetized word to a list of the words sharing the alphabetization:

除了 toMap 方法之外，collector API 还提供 groupingBy 方法，该方法返回 collector，以生成基于分类器函数将元素分组为类别的映射。分类器函数接受一个元素并返回它所属的类别。这个类别用作元素的 Map 键。groupingBy 方法的最简单版本只接受一个分类器并返回一个 Map，其值是每个类别中所有元素的列表。这是我们在 [Item-45](/Chapter-7/Chapter-7-Item-45-Use-streams-judiciously.md) 的字谜程序中使用的收集器，用于生成从按字母顺序排列的单词到共享字母顺序的单词列表的映射：

```Java
words.collect(groupingBy(word -> alphabetize(word)))
```

If you want groupingBy to return a collector that produces a map with values other than lists, you can specify a downstream collector in addition to a classifier. A downstream collector produces a value from a stream containing all the elements in a category. The simplest use of this parameter is to pass toSet(), which results in a map whose values are sets of elements rather than lists.

如果你希望 groupingBy 返回一个使用列表之外的值生成映射的收集器，你可以指定一个下游收集器和一个分类器。下游收集器从包含类别中的所有元素的流中生成一个值。这个参数最简单的用法是传递 toSet()，这会生成一个 Map，其值是 Set，而不是 List。

Alternatively, you can pass toCollection(collectionFactory), which lets you create the collections into which each category of elements is placed. This gives you the flexibility to choose any collection type you want. Another simple use of the two-argument form of groupingBy is to pass counting() as the downstream collector. This results in a map that associates each category with the number of elements in the category, rather than a collection containing the elements. That’s what you saw in the frequency table example at the beginning of this item:

或者，你可以传递 `toCollection(collectionFactory)`，它允许你创建集合，将每个类别的元素放入其中。这使你可以灵活地选择所需的任何集合类型。groupingBy 的两参数形式的另一个简单用法是将 `counting()` 作为下游收集器传递。这将生成一个 Map，该 Map 将每个类别与类别中的元素数量相关联，而不是包含元素的集合。这是你在这一项开始的 freq 表例子中看到的：

```Java
Map<String, Long> freq = words.collect(groupingBy(String::toLowerCase, counting()));
```

The third version of groupingBy lets you specify a map factory in addition to a downstream collector. Note that this method violates the standard telescoping argument list pattern: the mapFactory parameter precedes, rather than follows, the downStream parameter. This version of groupingBy gives you control over the containing map as well as the contained collections, so, for example, you can specify a collector that returns a TreeMap whose values are TreeSets.

groupingBy 的第三个版本允许你指定除了下游收集器之外的 Map 工厂。注意，这个方法违反了标准的可伸缩参数列表模式：mapFactory 参数位于下游参数之前，而不是之后。groupingBy 的这个版本允许你控制包含的 Map 和包含的集合，因此，例如，你可以指定一个收集器，该收集器返回一个 TreeMap，其值为 TreeSet。

The groupingByConcurrent method provides variants of all three overloadings of groupingBy. These variants run efficiently in parallel and produce ConcurrentHashMap instances. There is also a rarely used relative of groupingBy called partitioningBy. In lieu of a classifier method, it takes a predicate and returns a map whose key is a Boolean. There are two overloadings of this method, one of which takes a downstream collector in addition to a predicate.

groupingByConcurrent 方法提供了 groupingBy 的所有三种重载的变体。这些变体可以有效地并行运行，并生成 ConcurrentHashMap 实例。还有一个与 groupingBy 关系不大的词，叫做 partitioningBy 。代替分类器方法，它接受一个 Predicate 并返回一个键为布尔值的 Map。此方法有两个重载，其中一个除了 Predicate 外还接受下游收集器。

The collectors returned by the counting method are intended only for use as downstream collectors. The same functionality is available directly on Stream, via the count method, so **there is never a reason to say collect(counting()).** There are fifteen more Collectors methods with this property. They include the nine methods whose names begin with summing, averaging, and summarizing (whose functionality is available on the corresponding primitive stream types). They also include all overloadings of the reducing method, and the filtering, mapping, flatMapping, and collectingAndThen methods. Most programmers can safely ignore the majority of these methods. From a design perspective, these collectors represent an attempt to partially duplicate the functionality of streams in collectors so that downstream collectors can act as “ministreams.”

计数方法返回的收集器仅用于作为下游收集器。相同的功能可以通过 count 方法直接在流上使用，**所以永远没有理由说 `collect(counting())`。** 还有 15 个具有此属性的收集器方法。它们包括 9 个方法，它们的名称以求和、平均和汇总开头（它们的功能在相应的原始流类型上可用）。它们还包括 reduce 方法的所有重载，以及过滤、映射、平面映射和 collectingAndThen 方法。大多数程序员可以安全地忽略这些方法中的大多数。从设计的角度来看，这些收集器试图部分复制收集器中的流的功能，以便下游收集器可以充当「迷你存储器」。

There are three Collectors methods we have yet to mention. Though they are in Collectors, they don’t involve collections. The first two are minBy and maxBy, which take a comparator and return the minimum or maximum element in the stream as determined by the comparator. They are minor generalizations of the min and max methods in the Stream interface and are the collector analogues of the binary operators returned by the like-named methods in BinaryOperator. Recall that we used BinaryOperator.maxBy in our best-selling album example.

我们还没有提到三种 Collectors 方法。虽然它们是在 Collectors 中，但它们不涉及收集。前两个是 minBy 和 maxBy，它们接受 comparator 并返回由 comparator 确定的流中的最小或最大元素。它们是流接口中最小和最大方法的一些小泛化，是 BinaryOperator 中同名方法返回的二进制操作符的 collector 类似物。回想一下，在我们最畅销的专辑示例中，我们使用了 `BinaryOperator.maxBy`。

The final Collectors method is joining, which operates only on streams of CharSequence instances such as strings. In its parameterless form, it returns a collector that simply concatenates the elements. Its one argument form takes a single CharSequence parameter named delimiter and returns a collector that joins the stream elements, inserting the delimiter between adjacent elements. If you pass in a comma as the delimiter, the collector returns a comma-separated values string (but beware that the string will be ambiguous if any of the elements in the stream contain commas). The three argument form takes a prefix and suffix in addition to the delimiter. The resulting collector generates strings like the ones that you get when you print a collection, for example [came, saw, conquered].

最后一个 Collectors 方法是 join，它只对 CharSequence 实例流（如字符串）执行操作。在其无参数形式中，它返回一个收集器，该收集器只是将元素连接起来。它的一个参数形式接受一个名为 delimiter 的 CharSequence 参数，并返回一个连接流元素的收集器，在相邻元素之间插入分隔符。如果传入逗号作为分隔符，收集器将返回逗号分隔的值字符串（但是要注意，如果流中的任何元素包含逗号，该字符串将是不明确的）。除了分隔符外，三参数形式还接受前缀和后缀。生成的收集器生成的字符串与打印集合时得到的字符串类似，例如 `[came, saw, conquer]`。

In summary, the essence of programming stream pipelines is side-effect-free function objects. This applies to all of the many function objects passed to streams and related objects. The terminal operation forEach should only be used to report the result of a computation performed by a stream, not to perform the computation. In order to use streams properly, you have to know about collectors. The most important collector factories are toList, toSet, toMap, groupingBy, and joining.

总之，流管道编程的本质是无副作用的函数对象。这适用于传递给流和相关对象的所有函数对象。Terminal 操作 forEach 只应用于报告由流执行的计算结果，而不应用于执行计算。为了正确使用流，你必须了解 collector。最重要的 collector 工厂是 toList、toSet、toMap、groupingBy 和 join。

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-7/Chapter-7-Introduction.md)**
- **Previous Item（上一条目）：[Item 45: Use streams judiciously（明智地使用流）](/Chapter-7/Chapter-7-Item-45-Use-streams-judiciously.md)**
- **Next Item（下一条目）：[Item 47: Prefer Collection to Stream as a return type（优先选择 Collection 而不是流作为返回类型）](/Chapter-7/Chapter-7-Item-47-Prefer-Collection-to-Stream-as-a-return-type.md)**
