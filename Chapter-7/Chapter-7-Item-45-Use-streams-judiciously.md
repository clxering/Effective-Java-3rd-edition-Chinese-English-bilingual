## Chapter 7. Lambdas and Streams（λ 表达式和流）

### Item 45: Use streams judiciously（明智地使用流）

The streams API was added in Java 8 to ease the task of performing bulk operations, sequentially or in parallel. This API provides two key abstractions: the stream, which represents a finite or infinite sequence of data elements, and the stream pipeline, which represents a multistage computation on these elements. The elements in a stream can come from anywhere. Common sources include collections, arrays, files, regular expression pattern matchers, pseudorandom number generators, and other streams. The data elements in a stream can be object references or primitive values. Three primitive types are supported: int, long, and double.

在 Java 8 中添加了流 API，以简化序列或并行执行批量操作的任务。这个 API 提供了两个关键的抽象：流（表示有限或无限的数据元素序列）和流管道（表示对这些元素的多阶段计算）。流中的元素可以来自任何地方。常见的源包括集合、数组、文件、正则表达式的 Pattern 匹配器、伪随机数生成器和其他流。流中的数据元素可以是对象的引用或基本数据类型。支持三种基本数据类型：int、long 和 double。

A stream pipeline consists of a source stream followed by zero or more intermediate operations and one terminal operation. Each intermediate operation transforms the stream in some way, such as mapping each element to a function of that element or filtering out all elements that do not satisfy some condition. Intermediate operations all transform one stream into another, whose element type may be the same as the input stream or different from it. The terminal operation performs a final computation on the stream resulting from the last intermediate operation, such as storing its elements into a collection, returning a certain element, or printing all of its elements.

流管道由源流、零个或多个 Intermediate 操作和一个 Terminal 操作组成。每个 Intermediate 操作以某种方式转换流，例如将每个元素映射到该元素的一个函数，或者过滤掉不满足某些条件的所有元素。中间操作都将一个流转换为另一个流，其元素类型可能与输入流相同，也可能与输入流不同。Terminal 操作对最后一次 Intermediate 操作所产生的流进行最终计算，例如将其元素存储到集合中、返回特定元素、或打印其所有元素。

Stream pipelines are evaluated lazily: evaluation doesn’t start until the terminal operation is invoked, and data elements that aren’t required in order to complete the terminal operation are never computed. This lazy evaluation is what makes it possible to work with infinite streams. Note that a stream pipeline without a terminal operation is a silent no-op, so don’t forget to include one.

流管道的计算是惰性的：直到调用 Terminal 操作时才开始计算，并且对完成 Terminal 操作不需要的数据元素永远不会计算。这种惰性的求值机制使得处理无限流成为可能。请注意，没有 Terminal 操作的流管道是无动作的，因此不要忘记包含一个 Terminal 操作。

The streams API is fluent: it is designed to allow all of the calls that comprise a pipeline to be chained into a single expression. In fact, multiple pipelines can be chained together into a single expression.

流 API 是流畅的：它被设计成允许使用链式调用将组成管道的所有调用写到单个表达式中。实际上，可以将多个管道链接到一个表达式中。

By default, stream pipelines run sequentially. Making a pipeline execute in parallel is as simple as invoking the parallel method on any stream in the pipeline, but it is seldom appropriate to do so (Item 48).

默认情况下，流管道按顺序运行。让管道并行执行与在管道中的任何流上调用并行方法一样简单，但是这样做不一定合适（[Item-48](/Chapter-7/Chapter-7-Item-48-Use-caution-when-making-streams-parallel.md)）。

The streams API is sufficiently versatile that practically any computation can be performed using streams, but just because you can doesn’t mean you should. When used appropriately, streams can make programs shorter and clearer; when used inappropriately, they can make programs difficult to read and maintain. There are no hard and fast rules for when to use streams, but there are heuristics.

流 API 非常通用，实际上任何计算都可以使用流来执行，但这并不意味着你就应该这样做。如果使用得当，流可以使程序更短、更清晰；如果使用不当，它们会使程序难以读取和维护。对于何时使用流没有硬性的规则，但是有一些启发式的规则。

Consider the following program, which reads the words from a dictionary file and prints all the anagram groups whose size meets a user-specified minimum. Recall that two words are anagrams if they consist of the same letters in a different order. The program reads each word from a user-specified dictionary file and places the words into a map. The map key is the word with its letters alphabetized, so the key for "staple" is "aelpst", and the key for "petals" is also "aelpst": the two words are anagrams, and all anagrams share the same alphabetized form (or alphagram, as it is sometimes known). The map value is a list containing all of the words that share an alphabetized form. After the dictionary has been processed, each list is a complete anagram group. The program then iterates through the map’s values() view and prints each list whose size meets the threshold:

考虑下面的程序，它从字典文件中读取单词并打印所有大小满足用户指定最小值的变位组。回想一下，如果两个单词以不同的顺序由相同的字母组成，那么它们就是字谜。该程序从用户指定的字典文件中读取每个单词，并将这些单词放入一个 Map 中。Map 的键是按字母顺序排列的单词，因此「staple」的键是「aelpst」，而「petals」的键也是「aelpst」：这两个单词是字谜，所有的字谜都有相同的字母排列形式（有时称为字母图）。Map 的值是一个列表，其中包含共享按字母顺序排列的表单的所有单词。在字典被处理之后，每个列表都是一个完整的字谜组。然后，该程序遍历 Map 的 values() 视图，并打印大小满足阈值的每个列表：

```Java
// Prints all large anagram groups in a dictionary iteratively
public class Anagrams {
    public static void main(String[] args) throws IOException {
        File dictionary = new File(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);
        Map<String, Set<String>> groups = new HashMap<>();
        try (Scanner s = new Scanner(dictionary)) {
            while (s.hasNext()) {
                String word = s.next();
                groups.computeIfAbsent(alphabetize(word),(unused) -> new TreeSet<>()).add(word);
            }
        }
        for (Set<String> group : groups.values())
        if (group.size() >= minGroupSize)
            System.out.println(group.size() + ": " + group);
    }

    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```

One step in this program is worthy of note. The insertion of each word into the map, which is shown in bold, uses the computeIfAbsent method, which was added in Java 8. This method looks up a key in the map: If the key is present, the method simply returns the value associated with it. If not, the method computes a value by applying the given function object to the key, associates this value with the key, and returns the computed value. The computeIfAbsent method simplifies the implementation of maps that associate multiple values with each key.

这个程序中的一个步骤值得注意。将每个单词插入到 Map 中（以粗体显示）使用 computeIfAbsent 方法，该方法是在 Java 8 中添加的。此方法在 Map 中查找键：如果键存在，则该方法仅返回与其关联的值。若不存在，则该方法通过将给定的函数对象应用于键来计算一个值，将该值与键关联，并返回计算的值。computeIfAbsent 方法简化了将多个值与每个键关联的 Map 的实现。

Now consider the following program, which solves the same problem, but makes heavy use of streams. Note that the entire program, with the exception of the code that opens the dictionary file, is contained in a single expression. The only reason the dictionary is opened in a separate expression is to allow the use of the try-with-resources statement, which ensures that the dictionary file is closed:

现在考虑下面的程序，它解决了相同的问题，但是大量使用了流。注意，除了打开字典文件的代码之外，整个程序都包含在一个表达式中。在单独的表达式中打开字典的唯一原因是允许使用 `try with-resources` 语句，该语句确保字典文件是关闭的：

```Java
// Overuse of streams - don't do this!
public class Anagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);
        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(
            groupingBy(word -> word.chars().sorted()
            .collect(StringBuilder::new,(sb, c) -> sb.append((char) c),
            StringBuilder::append).toString()))
            .values().stream()
            .filter(group -> group.size() >= minGroupSize)
            .map(group -> group.size() + ": " + group)
            .forEach(System.out::println);
        }
    }
}
```

If you find this code hard to read, don’t worry; you’re not alone. It is shorter, but it is also less readable, especially to programmers who are not experts in the use of streams. Overusing streams makes programs hard to read and maintain. Luckily, there is a happy medium. The following program solves the same problem, using streams without overusing them. The result is a program that’s both shorter and clearer than the original:

如果你发现这段代码难以阅读，不要担心；不单是你有这样的感觉。它虽然更短，但可读性也更差，特别是对于不擅长流使用的程序员来说。过度使用流会使得程序难以读取和维护。幸运的是，有一个折衷的办法。下面的程序解决了相同的问题，在不过度使用流的情况下使用流。结果，这个程序比原来的程序更短，也更清晰：

```Java
// Tasteful use of streams enhances clarity and conciseness
public class Anagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);
        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(groupingBy(word -> alphabetize(word)))
            .values().stream()
            .filter(group -> group.size() >= minGroupSize)
            .forEach(g -> System.out.println(g.size() + ": " + g));
        }
    }
    // alphabetize method is the same as in original version
}
```

Even if you have little previous exposure to streams, this program is not hard to understand. It opens the dictionary file in a try-with-resources block, obtaining a stream consisting of all the lines in the file. The stream variable is named words to suggest that each element in the stream is a word. The pipeline on this stream has no intermediate operations; its terminal operation collects all the words into a map that groups the words by their alphabetized form (Item 46). This is exactly the same map that was constructed in both previous versions of the program. Then a new `Stream<List<String>>` is opened on the values() view of the map. The elements in this stream are, of course, the anagram groups. The stream is filtered so that all of the groups whose size is less than minGroupSize are ignored, and finally, the remaining groups are printed by the terminal operation forEach.

即使你以前很少接触流，这个程序也不难理解。它在带有资源的 try 块中打开字典文件，获得由文件中所有行组成的流。流变量名为 words，表示流中的每个元素都是一个单词。此流上的管道没有 Intermediate 操作；它的 Terminal 操作将所有单词收集到一个 Map 中，该 Map 按字母顺序将单词分组（[Item-46](/Chapter-7/Chapter-7-Item-46-Prefer-side-effect-free-functions-in-streams.md)）。这与在程序的前两个版本中构造的 Map 完全相同。然后在 Map 的 values() 视图上打开一个新的 `Stream<List<String>>`。这个流中的元素当然是字谜组。对流进行过滤，以便忽略所有大小小于 minGroupSize 的组，最后，Terminal 操作 forEach 打印其余组。

Note that the lambda parameter names were chosen carefully. The parameter g should really be named group, but the resulting line of code would be too wide for the book. **In the absence of explicit types, careful naming of lambda parameters is essential to the readability of stream pipelines.**

注意，lambda 表达式参数名称是经过仔细选择的。参数 g 实际上应该命名为 group，但是生成的代码行对于本书排版来说太宽了。**在没有显式类型的情况下，lambda 表达式参数的谨慎命名对于流管道的可读性至关重要。**

Note also that word alphabetization is done in a separate alphabetize method. This enhances readability by providing a name for the operation and keeping implementation details out of the main program. **Using helper methods is even more important for readability in stream pipelines than in iterative code** because pipelines lack explicit type information and named temporary variables.

还要注意，单词的字母化是在一个单独的字母化方法中完成的。这通过为操作提供一个名称并将实现细节排除在主程序之外，从而增强了可读性。**在流管道中使用 helper 方法比在迭代代码中更重要**，因为管道缺少显式类型信息和命名的临时变量。

The alphabetize method could have been reimplemented to use streams, but a stream-based alphabetize method would have been less clear, more difficult to write correctly, and probably slower. These deficiencies result from Java’s lack of support for primitive char streams (which is not to imply that Java should have supported char streams; it would have been infeasible to do so). To demonstrate the hazards of processing char values with streams, consider the following code:

本来可以重新实现字母顺序方法来使用流，但是基于流的字母顺序方法就不那么清晰了，更难于正确地编写，而且可能会更慢。这些缺陷是由于 Java 不支持基本字符流（这并不意味着 Java 应该支持字符流；这样做是不可行的）。要演示使用流处理 char 值的危害，请考虑以下代码：

```Java
"Hello world!".chars().forEach(System.out::print);
```

You might expect it to print Hello world!, but if you run it, you’ll find that it prints 721011081081113211911111410810033. This happens because the elements of the stream returned by "Hello world!".chars() are not char values but int values, so the int overloading of print is invoked. It is admittedly confusing that a method named chars returns a stream of int values. You could fix the program by using a cast to force the invocation of the correct overloading:

你可能希望它打印 Hello world!，但如果运行它，你会发现它打印 721011081081113211911111410810033。这是因为 `"Hello world!".chars()` 返回的流元素不是 char 值，而是 int 值，因此调用了 print 的 int 重载。无可否认，一个名为 chars 的方法返回一个 int 值流是令人困惑的。你可以通过强制调用正确的重载来修复程序：

```Java
"Hello world!".chars().forEach(x -> System.out.print((char) x));
```

but ideally you should refrain from using streams to process char values. When you start using streams, you may feel the urge to convert all your loops into streams, but resist the urge. While it may be possible, it will likely harm the readability and maintainability of your code base. As a rule, even moderately complex tasks are best accomplished using some combination of streams and iteration, as illustrated by the Anagrams programs above. So **refactor existing code to use streams and use them in new code only where it makes sense to do so.**

但理想情况下，你应该避免使用流来处理 char 值。当你开始使用流时，你可能会有将所有循环转换为流的冲动，但是要抵制这种冲动。虽然这是可实施的，但它可能会损害代码库的可读性和可维护性。通常，即使是中等复杂的任务，也最好使用流和迭代的某种组合来完成，如上面的 Anagrams 程序所示。因此，**重构现有代码或是在新代码中，都应该在合适的场景使用流。**

As shown in the programs in this item, stream pipelines express repeated computation using function objects (typically lambdas or method references), while iterative code expresses repeated computation using code blocks. There are some things you can do from code blocks that you can’t do from function objects:

如本项中的程序所示，流管道使用函数对象（通常是 lambda 表达式或方法引用）表示重复计算，而迭代代码使用代码块表示重复计算。有些事情你可以对代码块做，而你不能对函数对象做：

- From a code block, you can read or modify any local variable in scope; from a lambda, you can only read final or effectively final variables [JLS 4.12.4], and you can’t modify any local variables.

从代码块中，可以读取或修改作用域中的任何局部变量；在 lambda 表达式中，只能读取 final 或有效的 final 变量 [JLS 4.12.4]，不能修改任何局部变量。

- From a code block, you can return from the enclosing method, break or continue an enclosing loop, or throw any checked exception that this method is declared to throw; from a lambda you can do none of these things.

从代码块中，可以从封闭方法返回、中断或继续封闭循环，或抛出声明要抛出的任何已检查异常；在 lambda 表达式中，你不能做这些事情。

If a computation is best expressed using these techniques, then it’s probably not a good match for streams. Conversely, streams make it very easy to do some things:

如果使用这些技术最好地表达计算，那么它可能不适合流。相反，流使做一些事情变得非常容易：

- Uniformly transform sequences of elements

元素序列的一致变换

- Filter sequences of elements

过滤元素序列

- Combine sequences of elements using a single operation (for example to add them, concatenate them, or compute their minimum)

使用单个操作组合元素序列（例如添加它们、连接它们或计算它们的最小值）

- Accumulate sequences of elements into a collection, perhaps grouping them by some common attribute

将元素序列累积到一个集合中，可能是按某个公共属性对它们进行分组

- Search a sequence of elements for an element satisfying some criterion

在元素序列中搜索满足某些条件的元素

If a computation is best expressed using these techniques, then it is a good candidate for streams.

如果使用这些技术能够最好地表达计算，那么它就是流的一个很好的使用场景。

One thing that is hard to do with streams is to access corresponding elements from multiple stages of a pipeline simultaneously: once you map a value to some other value, the original value is lost. One workaround is to map each value to a pair object containing the original value and the new value, but this is not a satisfying solution, especially if the pair objects are required for multiple stages of a pipeline. The resulting code is messy and verbose, which defeats a primary purpose of streams. When it is applicable, a better workaround is to invert the mapping when you need access to the earlier-stage value.

使用流很难做到的一件事是从管道的多个阶段同时访问相应的元素：一旦你将一个值映射到另一个值，原始值就会丢失。一个解决方案是将每个值映射到包含原始值和新值的 pair 对象，但这不是一个令人满意的解决方案，特别是如果管道的多个阶段都需要 pair 对象的话。生成的代码混乱而冗长，这违背了流的主要目的。当它适用时，更好的解决方案是在需要访问早期阶段值时反转映射。

For example, let’s write a program to print the first twenty Mersenne primes. To refresh your memory, a Mersenne number is a number of the form 2p − 1. If p is prime, the corresponding Mersenne number may be prime; if so, it’s a Mersenne prime. As the initial stream in our pipeline, we want all the prime numbers. Here’s a method to return that (infinite) stream. We assume a static import has been used for easy access to the static members of BigInteger:

例如，让我们编写一个程序来打印前 20 个 Mersenne 素数。刷新你的记忆,一个 Mersenne 素数的数量是一个数字形式 2p − 1。如果 p 是素数，对应的 Mersenne 数可以是素数；如果是的话，这就是 Mersenne 素数。作为管道中的初始流，我们需要所有质数。这里有一个返回（无限）流的方法。我们假设已经使用静态导入来方便地访问 BigInteger 的静态成员：

```Java
static Stream<BigInteger> primes() {
    return Stream.iterate(TWO, BigInteger::nextProbablePrime);
}
```

The name of the method (primes) is a plural noun describing the elements of the stream. This naming convention is highly recommended for all methods that return streams because it enhances the readability of stream pipelines. The method uses the static factory Stream.iterate, which takes two parameters: the first element in the stream, and a function to generate the next element in the stream from the previous one. Here is the program to print the first twenty Mersenne primes:

方法的名称（素数）是描述流元素的复数名词。强烈推荐所有返回流的方法使用这种命名约定，因为它增强了流管道的可读性。该方法使用静态工厂 `Stream.iterate`，它接受两个参数：流中的第一个元素和一个函数，用于从前一个元素生成流中的下一个元素。下面是打印前 20 个 Mersenne 素数的程序：

```Java
public static void main(String[] args) {
    primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
    .filter(mersenne -> mersenne.isProbablePrime(50))
    .limit(20)
    .forEach(System.out::println);
}
```

This program is a straightforward encoding of the prose description above: it starts with the primes, computes the corresponding Mersenne numbers, filters out all but the primes (the magic number 50 controls the probabilistic primality test), limits the resulting stream to twenty elements, and prints them out.

这个程序是上述散文描述的一个简单编码：它从素数开始，计算相应的 Mersenne 数，过滤掉除素数以外的所有素数（魔法数字 50 控制概率素数测试），将结果流限制为 20 个元素，并打印出来。

Now suppose that we want to precede each Mersenne prime with its exponent (p). This value is present only in the initial stream, so it is inaccessible in the terminal operation, which prints the results. Luckily, it’s easy to compute the exponent of a Mersenne number by inverting the mapping that took place in the first intermediate operation. The exponent is simply the number of bits in the binary representation, so this terminal operation generates the desired result:

现在假设我们想要在每个 Mersenne 素数之前加上它的指数 (p)，这个值只在初始流中存在，因此在输出结果的终端操作中是不可访问的。幸运的是，通过对第一个中间操作中发生的映射求逆，可以很容易地计算出 Mersenne 数的指数。指数仅仅是二进制表示的比特数，因此这个终端操作产生了想要的结果：

```Java
.forEach(mp -> System.out.println(mp.bitLength() + ": " + mp));
```

There are plenty of tasks where it is not obvious whether to use streams or iteration. For example, consider the task of initializing a new deck of cards. Assume that Card is an immutable value class that encapsulates a Rank and a Suit, both of which are enum types. This task is representative of any task that requires computing all the pairs of elements that can be chosen from two sets. Mathematicians call this the Cartesian product of the two sets. Here’s an iterative implementation with a nested for-each loop that should look very familiar to you:

在许多任务中，使用流还是迭代并不明显。例如，考虑初始化一副新纸牌的任务。假设 Card 是一个不可变的值类，它封装了 Rank 和 Suit，它们都是 enum 类型。此任务代表需要计算可从两个集合中选择的所有元素对的任何任务。数学家称之为这两个集合的笛卡尔积。下面是一个嵌套 for-each 循环的迭代实现，你应该非常熟悉它：

```Java
// Iterative Cartesian product computation
private static List<Card> newDeck() {
    List<Card> result = new ArrayList<>();
    for (Suit suit : Suit.values())
    for (Rank rank : Rank.values())
    result.add(new Card(suit, rank));
    return result;
}
```

And here is a stream-based implementation that makes use of the intermediate operation flatMap. This operation maps each element in a stream to a stream and then concatenates all of these new streams into a single stream (or flattens them). Note that this implementation contains a nested lambda, shown in boldface:

这是一个基于流的实现，它使用了中间操作 flatMap。此操作将流中的每个元素映射到流，然后将所有这些新流连接到单个流中（或将其扁平化）。注意，这个实现包含一个嵌套 lambda 表达式，用粗体显示:

```Java
// Stream-based Cartesian product computation
private static List<Card> newDeck() {
    return Stream.of(Suit.values())
    .flatMap(suit ->Stream.of(Rank.values())
    .map(rank -> new Card(suit, rank)))
    .collect(toList());
}
```

Which of the two versions of newDeck is better? It boils down to personal preference and the environment in which you’re programming. The first version is simpler and perhaps feels more natural. A larger fraction of Java programmers will be able to understand and maintain it, but some programmers will feel more comfortable with the second (stream-based) version. It’s a bit more concise and not too difficult to understand if you’re reasonably well-versed in streams and functional programming. If you’re not sure which version you prefer, the iterative version is probably the safer choice. If you prefer the stream version and you believe that other programmers who will work with the code will share your preference, then you should use it.

两个版本的 newDeck 哪个更好？这可以归结为个人偏好和编程环境。第一个版本更简单，可能感觉更自然。大部分 Java 程序员将能够理解和维护它，但是一些程序员将对第二个（基于流的）版本感到更舒服。如果你相当精通流和函数式编程，那么它会更简洁，也不会太难理解。如果你不确定你更喜欢哪个版本，迭代版本可能是更安全的选择。如果你更喜欢流版本，并且相信与代码一起工作的其他程序员也会分享你的偏好，那么你应该使用它。

In summary, some tasks are best accomplished with streams, and others with iteration. Many tasks are best accomplished by combining the two approaches. There are no hard and fast rules for choosing which approach to use for a task, but there are some useful heuristics. In many cases, it will be clear which approach to use; in some cases, it won’t. If you’re not sure whether a task is better served by streams or iteration, try both and see which works better.

总之，有些任务最好使用流来完成，有些任务最好使用迭代来完成。许多任务最好通过结合这两种方法来完成。对于选择任务使用哪种方法，没有硬性的规则，但是有一些有用的启发。在许多情况下，使用哪种方法是清楚的；在某些情况下很难决定。如果你不确定一个任务是通过流还是通过迭代更好地完成，那么同时尝试这两种方法，看看哪一种更有效。

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-7/Chapter-7-Introduction.md)**
- **Previous Item（上一条目）：[Item 44: Favor the use of standard functional interfaces（优先使用标准函数式接口）](/Chapter-7/Chapter-7-Item-44-Favor-the-use-of-standard-functional-interfaces.md)**
- **Next Item（下一条目）：[Item 46: Prefer side effect free functions in streams（在流中使用无副作用的函数）](/Chapter-7/Chapter-7-Item-46-Prefer-side-effect-free-functions-in-streams.md)**
