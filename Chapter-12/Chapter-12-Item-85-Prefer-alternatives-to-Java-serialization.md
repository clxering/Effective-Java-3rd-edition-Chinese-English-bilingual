## Chapter 12. Serialization（序列化）

### Item 85: Prefer alternatives to Java serialization（优先选择 Java 序列化的替代方案）

When serialization was added to Java in 1997, it was known to be somewhat risky. The approach had been tried in a research language (Modula-3) but never in a production language. While the promise of distributed objects with little effort on the part of the programmer was appealing, the price was invisible constructors and blurred lines between API and implementation, with the potential for problems with correctness, performance, security, and maintenance. Proponents believed the benefits outweighed the risks, but history has shown otherwise.

当序列化在 1997 年添加到 Java 中时，它被认为有一定的风险。这种方法曾在研究语言（Modula-3）中尝试过，但从未在生产语言中使用过。虽然程序员不费什么力气就能实现分布式对象，这一点很吸引人，但代价也不小，如：不可见的构造函数、API 与实现之间模糊的界线，还可能会出现正确性、性能、安全性和维护方面的问题。支持者认为收益大于风险，但历史证明并非如此。

The security issues described in previous editions of this book turned out to be every bit as serious as some had feared. The vulnerabilities discussed in the early 2000s were transformed into serious exploits over the next decade, famously including a ransomware attack on the San Francisco Metropolitan Transit Agency Municipal Railway (SFMTA Muni) that shut down the entire fare collection system for two days in November 2016 [Gallagher16].

在本书之前的版本中描述的安全问题，和人们担心的一样严重。21 世纪初仅停留在讨论的漏洞在接下来的 10 年间变成了真实严重的漏洞，其中最著名的包括 2016 年 11 月对旧金山大都会运输署市政铁路（SFMTA Muni）的勒索软件攻击，导致整个收费系统关闭了两天 [Gallagher16]。

A fundamental problem with serialization is that its attack surface is too big to protect, and constantly growing: Object graphs are deserialized by invoking the readObject method on an ObjectInputStream. This method is essentially a magic constructor that can be made to instantiate objects of almost any type on the class path, so long as the type implements the Serializable interface. In the process of deserializing a byte stream, this method can execute code from any of these types, so the code for all of these types is part of the attack surface.

序列化的一个根本问题是它的可攻击范围太大，且难以保护，而且问题还在不断增多：通过调用 ObjectInputStream 上的 readObject 方法反序列化对象图。这个方法本质上是一个神奇的构造函数，可以用来实例化类路径上几乎任何类型的对象，只要该类型实现 Serializable 接口。在反序列化字节流的过程中，此方法可以执行来自任何这些类型的代码，因此所有这些类型的代码都在攻击范围内。

The attack surface includes classes in the Java platform libraries, in third-party libraries such as Apache Commons Collections, and in the application itself. Even if you adhere to all of the relevant best practices and succeed in writing serializable classes that are invulnerable to attack, your application may still be vulnerable. To quote Robert Seacord, technical manager of the CERT Coordination Center:

攻击可涉及 Java 平台库、第三方库（如 Apache Commons collection）和应用程序本身中的类。即使坚持履行实践了所有相关的最佳建议，并成功地编写了不受攻击的可序列化类，应用程序仍然可能是脆弱的。引用 CERT 协调中心技术经理 Robert Seacord 的话：

Java deserialization is a clear and present danger as it is widely used both directly by applications and indirectly by Java subsystems such as RMI (Remote Method Invocation), JMX (Java Management Extension), and JMS (Java Messaging System). Deserialization of untrusted streams can result in remote code execution (RCE), denial-of-service (DoS), and a range of other exploits. Applications can be vulnerable to these attacks even if they did nothing wrong. [Seacord17]

Java 反序列化是一个明显且真实的危险源，因为它被应用程序直接和间接地广泛使用，比如 RMI（远程方法调用）、JMX（Java 管理扩展）和 JMS（Java 消息传递系统）。不可信流的反序列化可能导致远程代码执行（RCE）、拒绝服务（DoS）和一系列其他攻击。应用程序很容易受到这些攻击，即使它们本身没有错误。[Seacord17]

Attackers and security researchers study the serializable types in the Java libraries and in commonly used third-party libraries, looking for methods invoked during deserialization that perform potentially dangerous activities. Such methods are known as gadgets. Multiple gadgets can be used in concert, to form a gadget chain. From time to time, a gadget chain is discovered that is sufficiently powerful to allow an attacker to execute arbitrary native code on the underlying hardware, given only the opportunity to submit a carefully crafted byte stream for deserialization. This is exactly what happened in the SFMTA Muni attack. This attack was not isolated. There have been others, and there will be more.

攻击者和安全研究人员研究 Java 库和常用的第三方库中的可序列化类型，寻找在反序列化过程中调用的潜在危险活动的方法称为 gadget。多个小工具可以同时使用，形成一个小工具链。偶尔会发现一个小部件链，它的功能足够强大，允许攻击者在底层硬件上执行任意的本机代码，允许提交精心设计的字节流进行反序列化。这正是 SFMTA Muni 袭击中发生的事情。这次袭击并不是孤立的。不仅已经存在，而且还会有更多。

Without using any gadgets, you can easily mount a denial-of-service attack by causing the deserialization of a short stream that requires a long time to deserialize. Such streams are known as deserialization bombs [Svoboda16]. Here’s an example by Wouter Coekaerts that uses only hash sets and a string [Coekaerts15]:

不使用任何 gadget，你都可以通过对需要很长时间才能反序列化的短流进行反序列化，轻松地发起拒绝服务攻击。这种流被称为反序列化炸弹 [Svoboda16]。下面是 Wouter Coekaerts 的一个例子，它只使用哈希集和字符串 [Coekaerts15]：

```
// Deserialization bomb - deserializing this stream takes forever
static byte[] bomb() {
    Set<Object> root = new HashSet<>();
    Set<Object> s1 = root;
    Set<Object> s2 = new HashSet<>();
    for (int i = 0; i < 100; i++) {
        Set<Object> t1 = new HashSet<>();
        Set<Object> t2 = new HashSet<>();
        t1.add("foo"); // Make t1 unequal to t2
        s1.add(t1); s1.add(t2);
        s2.add(t1); s2.add(t2);
        s1 = t1;
        s2 = t2;
    }
    return serialize(root); // Method omitted for brevity
}
```

The object graph consists of 201 HashSet instances, each of which contains 3 or fewer object references. The entire stream is 5,744 bytes long, yet the sun would burn out long before you could deserialize it. The problem is that deserializing a HashSet instance requires computing the hash codes of its elements. The 2 elements of the root hash set are themselves hash sets containing 2 hash-set elements, each of which contains 2 hash-set elements, and so on, 100 levels deep. Therefore, deserializing the set causes the hashCode method to be invoked over 2100 times. Other than the fact that the deserialization is taking forever, the deserializer has no indication that anything is amiss. Few objects are produced, and the stack depth is bounded.

对象图由 201 个 HashSet 实例组成，每个实例包含 3 个或更少的对象引用。整个流的长度为 5744 字节，但是在你对其进行反序列化之前，资源就已经耗尽了。问题在于，反序列化 HashSet 实例需要计算其元素的哈希码。根哈希集的 2 个元素本身就是包含 2 个哈希集元素的哈希集，每个哈希集元素包含 2 个哈希集元素，以此类推，深度为 100。因此，反序列化 Set 会导致 hashCode 方法被调用超过 2100 次。除了反序列化会持续很长时间之外，反序列化器没有任何错误的迹象。生成的对象很少，并且堆栈深度是有界的。

So what can you do defend against these problems? You open yourself up to attack whenever you deserialize a byte stream that you don’t trust. **The best way to avoid serialization exploits is never to deserialize anything.** In the words of the computer named Joshua in the 1983 movie WarGames, “the only winning move is not to play.” **There is no reason to use Java serialization in any new system you write.** There are other mechanisms for translating between objects and byte sequences that avoid many of the dangers of Java serialization, while offering numerous advantages, such as cross-platform support, high performance, a large ecosystem of tools, and a broad community of expertise. In this book, we refer to these mechanisms as cross-platform structured-data representations. While others sometimes refer to them as serialization systems, this book avoids that usage to prevent confusion with Java serialization.

那么你能做些什么来抵御这些问题呢？当你反序列化一个你不信任的字节流时，你就会受到攻击。**避免序列化利用的最好方法是永远不要反序列化任何东西。** 用 1983 年电影《战争游戏》（WarGames）中名为约书亚（Joshua）的电脑的话来说，「唯一的制胜绝招就是不玩。」**没有理由在你编写的任何新系统中使用 Java 序列化。** 还有其他一些机制可以在对象和字节序列之间进行转换，从而避免了 Java 序列化的许多危险，同时还提供了许多优势，比如跨平台支持、高性能、大量工具和广泛的专家社区。在本书中，我们将这些机制称为跨平台结构数据表示。虽然其他人有时将它们称为序列化系统，但本书避免使用这种说法，以免与 Java 序列化混淆。

What these representations have in common is that they’re far simpler than Java serialization. They don’t support automatic serialization and deserialization of arbitrary object graphs. Instead, they support simple, structured data-objects consisting of a collection of attribute-value pairs. Only a few primitive and array data types are supported. This simple abstraction turns out to be sufficient for building extremely powerful distributed systems and simple enough to avoid the serious problems that have plagued Java serialization since its inception.

以上所述技术的共同点是它们比 Java 序列化简单得多。它们不支持任意对象图的自动序列化和反序列化。相反，它们支持简单的结构化数据对象，由一组「属性-值」对组成。只有少数基本数据类型和数组数据类型得到支持。事实证明，这个简单的抽象足以构建功能极其强大的分布式系统，而且足够简单，可以避免 Java 序列化从一开始就存在的严重问题。

The leading cross-platform structured data representations are JSON [JSON] and Protocol Buffers, also known as protobuf [Protobuf]. JSON was designed by Douglas Crockford for browser-server communication, and protocol buffers were designed by Google for storing and interchanging structured data among its servers. Even though these representations are sometimes called languageneutral, JSON was originally developed for JavaScript and protobuf for C++; both representations retain vestiges of their origins.

领先的跨平台结构化数据表示是 JSON 和 Protocol Buffers，也称为 protobuf。JSON 由 Douglas Crockford 设计用于浏览器与服务器通信，Protocol Buffers 由谷歌设计用于在其服务器之间存储和交换结构化数据。尽管这些技术有时被称为「中性语言」，但 JSON 最初是为 JavaScript 开发的，而 protobuf 是为 c++ 开发的；这两种技术都保留了其起源的痕迹。

The most significant differences between JSON and protobuf are that JSON is text-based and human-readable, whereas protobuf is binary and substantially more efficient; and that JSON is exclusively a data representation, whereas protobuf offers schemas (types) to document and enforce appropriate usage. Although protobuf is more efficient than JSON, JSON is extremely efficient for a text-based representation. And while protobuf is a binary representation, it does provide an alternative text representation for use where human-readability is desired (pbtxt).

JSON 和 protobuf 之间最显著的区别是 JSON 是基于文本的，并且是人类可读的，而 protobuf 是二进制的，但效率更高；JSON 是一种专门的数据表示，而 protobuf 提供模式（类型）来记录和执行适当的用法。虽然 protobuf 比 JSON 更有效，但是 JSON 对于基于文本的表示非常有效。虽然 protobuf 是一种二进制表示，但它确实提供了另一种文本表示，可用于需要具备人类可读性的场景（pbtxt）。

If you can’t avoid Java serialization entirely, perhaps because you’re working in the context of a legacy system that requires it, your next best alternative is to **never deserialize untrusted data.** In particular, you should never accept RMI traffic from untrusted sources. The official secure coding guidelines for Java say “Deserialization of untrusted data is inherently dangerous and should be avoided.” This sentence is set in large, bold, italic, red type, and it is the only text in the entire document that gets this treatment [Java-secure].

如果你不能完全避免 Java 序列化，可能是因为你需要在遗留系统环境中工作，那么你的下一个最佳选择是 **永远不要反序列化不可信的数据。** 特别要注意，你不应该接受来自不可信来源的 RMI 流量。Java 的官方安全编码指南说：「反序列化不可信的数据本质上是危险的，应该避免。」这句话是用大号、粗体、斜体和红色字体设置的，它是整个文档中唯一得到这种格式处理的文本。[Java-secure]

If you can’t avoid serialization and you aren’t absolutely certain of the safety of the data you’re deserializing, use the object deserialization filtering added in Java 9 and backported to earlier releases (java.io.ObjectInputFilter). This facility lets you specify a filter that is applied to data streams before they’re deserialized. It operates at the class granularity, letting you accept or reject certain classes. Accepting classes by default and rejecting a list of potentially dangerous ones is known as blacklisting; rejecting classes by default and accepting a list of those that are presumed safe is known as whitelisting. **Prefer whitelisting to blacklisting,** as blacklisting only protects you against known threats. A tool called Serial Whitelist Application Trainer (SWAT) can be used to automatically prepare a whitelist for your application [Schneider16]. The filtering facility will also protect you against excessive memory usage, and excessively deep object graphs, but it will not protect you against serialization bombs like the one shown above.

如果无法避免序列化，并且不能绝对确定反序列化数据的安全性，那么可以使用 Java 9 中添加的对象反序列化筛选，并将其移植到早期版本（java.io.ObjectInputFilter）。该工具允许你指定一个过滤器，该过滤器在反序列化数据流之前应用于数据流。它在类粒度上运行，允许你接受或拒绝某些类。默认接受所有类，并拒绝已知潜在危险类的列表称为黑名单；在默认情况下拒绝其他类，并接受假定安全的类的列表称为白名单。**优先选择白名单而不是黑名单，** 因为黑名单只保护你免受已知的威胁。一个名为 Serial Whitelist Application Trainer（SWAT）的工具可用于为你的应用程序自动准备一个白名单 [Schneider16]。过滤工具还将保护你免受过度内存使用和过于深入的对象图的影响，但它不能保护你免受如上面所示的序列化炸弹的影响。

Unfortunately, serialization is still pervasive in the Java ecosystem. If you are maintaining a system that is based on Java serialization, seriously consider migrating to a cross-platform structured-data representation, even though this may be a time-consuming endeavor. Realistically, you may still find yourself having to write or maintain a serializable class. It requires great care to write a serializable class that is correct, safe, and efficient. The remainder of this chapter provides advice on when and how to do this.

不幸的是，序列化在 Java 生态系统中仍然很普遍。如果你正在维护一个基于 Java 序列化的系统，请认真考虑迁移到跨平台的结构化数据，尽管这可能是一项耗时的工作。实际上，你可能仍然需要编写或维护一个可序列化的类。编写一个正确、安全、高效的可序列化类需要非常小心。本章的其余部分将提供何时以及如何进行此操作的建议。

In summary, serialization is dangerous and should be avoided. If you are designing a system from scratch, use a cross-platform structured-data representation such as JSON or protobuf instead. Do not deserialize untrusted data. If you must do so, use object deserialization filtering, but be aware that it is not guaranteed to thwart all attacks. Avoid writing serializable classes. If you must do so, exercise great caution.

总之，序列化是危险的，应该避免。如果你从头开始设计一个系统，可以使用跨平台的结构化数据，如 JSON 或 protobuf。不要反序列化不可信的数据。如果必须这样做，请使用对象反序列化过滤，但要注意，它不能保证阻止所有攻击。避免编写可序列化的类。如果你必须这样做，一定要非常小心。

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-12/Chapter-12-Introduction.md)**
- **Previous Item（上一条目）：[Item 84: Don’t depend on the thread scheduler（不要依赖线程调度器）](/Chapter-11/Chapter-11-Item-84-Don’t-depend-on-the-thread-scheduler.md)**
- **Next Item（下一条目）：[Item 86: Implement Serializable with great caution（非常谨慎地实现 Serializable）](/Chapter-12/Chapter-12-Item-86-Implement-Serializable-with-great-caution.md)**
