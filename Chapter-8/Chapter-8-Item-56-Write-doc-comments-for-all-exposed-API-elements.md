## Chapter 8. Methods（方法）

### Item 56: Write doc comments for all exposed API elements（为所有公开的 API 元素编写文档注释）

If an API is to be usable, it must be documented. Traditionally, API documentation was generated manually, and keeping it in sync with code was a chore. The Java programming environment eases this task with the Javadoc utility. Javadoc generates API documentation automatically from source code with specially formatted documentation comments, more commonly known as doc comments.

如果 API 要可用，就必须对其进行文档化。传统上，API 文档是手工生成的，保持与代码的同步是一件苦差事。Java 编程环境使用 Javadoc 实用程序简化了这一任务。Javadoc 使用特殊格式的文档注释（通常称为文档注释）从源代码自动生成 API 文档。

While the doc comment conventions are not officially part of the language, they constitute a de facto API that every Java programmer should know. These conventions are described in the How to Write Doc Comments web page [Javadoc-guide]. While this page has not been updated since Java 4 was released, it is still an invaluable resource. One important doc tag was added in Java 9, {@index}; one in Java 8, {@implSpec}; and two in Java 5, {@literal} and {@code}. These tags are missing from the aforementioned web page, but are discussed in this item.

虽然文档注释约定不是正式语言的一部分，但它们实际上构成了每个 Java 程序员都应该知道的 API。这些约定在如何编写文档注释的 web 页面 [Javadoc-guide] 中进行了描述。虽然自 Java 4 发布以来这个页面没有更新，但它仍然是一个非常宝贵的资源。在 Java 9 中添加了一个重要的文档标签，`{@index}`；Java 8 有一个重要标签，`{@implSpec}`；Java 5 中有两个重要标签，`{@literal}` 和 `{@code}`。上述 web 页面中缺少这些标签，但将在本项目中讨论。

**To document your API properly, you must precede every exported class, interface, constructor, method, and field declaration with a doc comment.** If a class is serializable, you should also document its serialized form (Item 87). In the absence of a doc comment, the best that Javadoc can do is to reproduce the declaration as the sole documentation for the affected API element. It is frustrating and error-prone to use an API with missing documentation comments. Public classes should not use default constructors because there is no way to provide doc comments for them. To write maintainable code, you should also write doc comments for most unexported classes, interfaces, constructors, methods, and fields, though these comments needn’t be as thorough as those for exported API elements.

**要正确地编写 API 文档，必须在每个公开的类、接口、构造函数、方法和字段声明之前加上文档注释。** 如果一个类是可序列化的，还应该记录它的序列化形式（[Item-87](/Chapter-12/Chapter-12-Item-87-Consider-using-a-custom-serialized-form.md)）。在缺少文档注释的情况下，Javadoc 所能做的最好的事情就是重新生成该声明，作为受影响的 API 元素的唯一文档。使用缺少文档注释的 API 是令人沮丧和容易出错的。公共类不应该使用默认构造函数，因为无法为它们提供文档注释。要编写可维护的代码，还应该为大多数未公开的类、接口、构造函数、方法和字段编写文档注释，尽管这些注释不需要像公开 API 元素那样完整。

**The doc comment for a method should describe succinctly the contract between the method and its client.** With the exception of methods in classes designed for inheritance (Item 19), the contract should say what the method does rather than how it does its job. The doc comment should enumerate all of the method’s preconditions, which are the things that have to be true in order for a client to invoke it, and its postconditions, which are the things that will be true after the invocation has completed successfully. Typically, preconditions are described implicitly by the @throws tags for unchecked exceptions; each unchecked exception corresponds to a precondition violation. Also, preconditions can be specified along with the affected parameters in their @param tags.

**方法的文档注释应该简洁地描述方法与其客户端之间的约定。** 除了为继承而设计的类中的方法（[Item-19](/Chapter-4/Chapter-4-Item-19-Design-and-document-for-inheritance-or-else-prohibit-it.md)），约定应该说明方法做什么，而不是它如何做它的工作。文档注释应该列举方法的所有前置条件（这些条件必须为真，以便客户端调用它们）和后置条件（这些条件是在调用成功完成后才为真）。通常，对于 unchecked 的异常，前置条件由 `@throw` 标记隐式地描述；每个 unchecked 异常对应于一个先决条件反例。此外，可以在前置条件及其 `@param` 标记中指定受影响的参数。

In addition to preconditions and postconditions, methods should document any side effects. A side effect is an observable change in the state of the system that is not obviously required in order to achieve the postcondition. For example, if a method starts a background thread, the documentation should make note of it.

除了前置条件和后置条件外，方法还应该文档中描述产生的任何副作用。副作用是系统状态的一个可观察到的变化，它不是实现后置条件所明显需要的。例如，如果一个方法启动了一个后台线程，文档应该说明。

To describe a method’s contract fully, the doc comment should have an @param tag for every parameter, an @return tag unless the method has a void return type, and an @throws tag for every exception thrown by the method, whether checked or unchecked (Item 74). If the text in the @return tag would be identical to the description of the method, it may be permissible to omit it, depending on the coding standards you are following.

要完整地描述方法的约定，文档注释应该为每个参数设置一个 `@param` 标记和一个 `@return` 标记（除非方法返回类型是 void），以及一个 `@throw` 标记（对于方法抛出的每个异常，无论 checked 或 unchecked）（[Item-74](/Chapter-10/Chapter-10-Item-74-Document-all-exceptions-thrown-by-each-method.md)。如果 `@return` 标记中的文本与方法的描述相同，则可以忽略它，这取决于你所遵循的标准。

By convention, the text following an @param tag or @return tag should be a noun phrase describing the value represented by the parameter or return value. Rarely, arithmetic expressions are used in place of noun phrases; see BigInteger for examples. The text following an @throws tag should consist of the word “if,” followed by a clause describing the conditions under which the exception is thrown. By convention, the phrase or clause following an @param, @return, or @throws tag is not terminated by a period. All of these conventions are illustrated by the following doc comment:

按照惯例，`@param` 标记或 `@return` 标记后面的文本应该是一个名词短语，描述参数或返回值所表示的值。算术表达式很少用来代替名词短语；有关示例，请参见 BigInteger。`@throw` 标记后面的文本应该包含单词「if」，后面跟着一个描述抛出异常的条件的子句。按照惯例，`@param`、`@return` 或 `@throw` 标记后面的短语或子句不以句号结束。以下的文档注释展示了所有这些约定：

```Java
/**
* Returns the element at the specified position in this list.
**
<p>This method is <i>not</i> guaranteed to run in constant
* time. In some implementations it may run in time proportional
* to the element position.
**
@param index index of element to return; must be
* non-negative and less than the size of this list
* @return the element at the specified position in this list
* @throws IndexOutOfBoundsException if the index is out of range
* ({@code index < 0 || index >= this.size()})
*/
E get(int index);
```

Notice the use of HTML tags in this doc comment (&lt;p&gt; and &lt;i&gt;). The Javadoc utility translates doc comments into HTML, and arbitrary HTML elements in doc comments end up in the resulting HTML document. Occasionally, programmers go so far as to embed HTML tables in their doc comments, although this is rare.

注意，在这个文档注释中使用 HTML 标签（`<p>` 和 `<i>`）。Javadoc 实用程序将文档注释转换为 HTML，文档注释中的任意 HTML 元素最终会出现在生成的 HTML 文档中。有时候，程序员甚至会在他们的文档注释中嵌入 HTML 表，尽管这种情况很少见。

Also notice the use of the Javadoc {@code} tag around the code fragment in the @throws clause. This tag serves two purposes: it causes the code fragment to be rendered in code font, and it suppresses processing of HTML markup and nested Javadoc tags in the code fragment. The latter property is what allows us to use the less-than sign (<) in the code fragment even though it’s an HTML metacharacter. To include a multiline code example in a doc comment, use a Javadoc {@code} tag wrapped inside an HTML &lt;pre&gt; tag. In other words, precede the code example with the characters &lt;pre&gt;{@code and follow it with }&lt;/pre&gt;. This preserves line breaks in the code, and eliminates the need to escape HTML metacharacters, but not the at sign (@), which must be escaped if the code sample uses annotations.

还要注意在 `@throw` 子句中的代码片段周围使用 Javadoc 的 `{@code}` 标记。这个标记有两个目的：它使代码片段以代码字体呈现，并且它抑制了代码片段中 HTML 标记和嵌套 Javadoc 标记的处理。后一个属性允许我们在代码片段中使用小于号 `(<)`，即使它是一个 HTML 元字符。要在文档注释中包含多行代码，请将其包装在 `<pre>` 标签中。换句话说，在代码示例之前加上字符 `<pre>{@code}</pre>`。这保留了代码中的换行符，并消除了转义 HTML 元字符的需要，但不需要转义 at 符号 `(@)`，如果代码示例使用注释，则必须转义 at 符号 `(@)`。

Finally, notice the use of the words “this list” in the doc comment. By convention, the word “this” refers to the object on which a method is invoked when it is used in the doc comment for an instance method.

最后，请注意文档注释中使用的单词「this list」。按照惯例，「this」指的是调用实例方法的对象。

As mentioned in Item 15, when you design a class for inheritance, you must document its self-use patterns, so programmers know the semantics of overriding its methods. These self-use patterns should be documented using the @implSpec tag, added in Java 8. Recall that ordinary doc comments describe the contract between a method and its client; @implSpec comments, by contrast, describe the contract between a method and its subclass, allowing subclasses to rely on implementation behavior if they inherit the method or call it via super. Here's how it looks in practice:

正如 [Item-15](/Chapter-4/Chapter-4-Item-15-Minimize-the-accessibility-of-classes-and-members.md) 中提到的，当你为继承设计一个类时，你必须记录它的自用模式，以便程序员知道覆盖它的方法的语义。这些自用模式应该使用在 Java 8 中添加的 `@implSpec` 标记来记录。回想一下，普通的文档注释描述了方法与其客户机之间的约定；相反，`@implSpec` 注释描述了方法与其子类之间的约定，允许子类依赖于实现行为（如果它们继承了方法或通过 super 调用方法）。下面是它在实际使用时的样子：

```Java
/**
* Returns true if this collection is empty.
**
@implSpec
* This implementation returns {@code this.size() == 0}.
**
@return true if this collection is empty
*/
public boolean isEmpty() { ... }
```

As of Java 9, the Javadoc utility still ignores the @implSpec tag unless you pass the command line switch -tag "implSpec: a :Implementation Requirements:". Hopefully this will be remedied in a subsequent release.

从 Java 9 开始，Javadoc 实用程序仍然忽略 `@implSpec` 标记，除非你通过命令行开关 `-tag "implSpec: a :Implementation Requirements:"`。希望在后续的版本中可以纠正这个错误。

Don’t forget that you must take special action to generate documentation that contains HTML metacharacters, such as the less-than sign (<), the greater-than sign (>), and the ampersand (&). The best way to get these characters into documentation is to surround them with the {@literal} tag, which suppress processing of HTML markup and nested Javadoc tags. It is like the {@code} tag, except that it doesn’t render the text in code font. For example, this Javadoc fragment:

不要忘记，你必须采取特殊的操作来生成包含 HTML 元字符的文档，比如小于号 `(<)`、大于号 `(>)`、与号 `(&)`。将这些字符放到文档中最好的方法是用 `{@literal}` 标记包围它们，这将抑制 HTML 标记和嵌套 Javadoc 标记的处理。它类似于 `{@code}` 标记，只是它不以代码字体呈现文本。例如，这个 Javadoc 片段：

```Java
* A geometric series converges if {@literal |r| < 1}.
```

generates the documentation: “A geometric series converges if |r| < 1.” The {@literal} tag could have been placed around just the less-than sign rather than the entire inequality with the same resulting documentation, but the doc comment would have been less readable in the source code. This illustrates the general principle that **doc comments should be readable both in the source code and in the generated documentation.** If you can’t achieve both, the readability of the generated documentation trumps that of the source code.

生成文档:「如果 |r| < 1，则几何级数收敛。」`{@literal}` 标签可以放在小于号周围，而不是整个不等式周围，得到的文档是相同的，但是文档注释在源代码中可读性会更差。这说明了 **一条原则，文档注释应该在源代码和生成的文档中都具备可读性。** 如果不能同时实现这两个目标，要保证生成的文档的可读性超过源代码的可读性。

The first “sentence” of each doc comment (as defined below) becomes the summary description of the element to which the comment pertains. For example, the summary description in the doc comment on page 255 is “Returns the element at the specified position in this list.” The summary description must stand on its own to describe the functionality of the element it summarizes. To avoid confusion, **no two members or constructors in a class or interface should have the same summary description.** Pay particular attention to overloadings, for which it is often natural to use the same first sentence (but unacceptable in doc comments).

每个文档注释的第一个「句子」（定义如下）成为注释所属元素的摘要描述。例如，255 页文档注释中的摘要描述是「返回列表中指定位置的元素」。摘要描述必须独立地描述它总结的元素的功能。为了避免混淆，**类或接口中的任何两个成员或构造函数都不应该具有相同的摘要描述。** 特别注意重载，对于重载，使用相同的摘要描述是很正常的（但在文档注释中是不可接受的）。

Be careful if the intended summary description contains a period, because the period can prematurely terminate the description. For example, a doc comment that begins with the phrase “A college degree, such as B.S., M.S. or Ph.D.” will result in the summary description “A college degree, such as B.S., M.S.” The problem is that the summary description ends at the first period that is followed by a space, tab, or line terminator (or at the first block tag) [Javadoc-ref]. Here, the second period in the abbreviation “M.S.” is followed by a space. The best solution is to surround the offending period and any associated text with an {@literal} tag, so the period is no longer followed by a space in the source code:

如果预期的摘要描述包含句点，请小心，因为句点可能会提前终止描述。例如，以「A college degree, such as B.S., M.S. or Ph.D.」开头的文档注释，会产生这样的概要描述「A college degree, such as B.S., M.S.」，问题在于，摘要描述在第一个句点结束，然后是空格、制表符或行结束符（或第一个块标记）[Javadoc-ref]。在这种情况下，缩写「M.S.」中的第二个句点就要接着用一个空格。最好的解决方案是用 `{@literal}` 标记来包围不当的句点和任何相关的文本，这样源代码中的句点后面就不会有空格了：

```Java
/**
* A college degree, such as B.S., {@literal M.S.} or Ph.D.
*/
public class Degree { ... }
```

It is a bit misleading to say that the summary description is the first sentence in a doc comment. Convention dictates that it should seldom be a complete sentence. For methods and constructors, the summary description should be a verb phrase (including any object) describing the action performed by the method. For example:

将摘要描述说成是是文档注释中的第一句话有点误导人。按照惯例，它通常不是一个完整的句子。对于方法和构造函数，摘要描述应该是一个动词短语（包括任何对象），描述方法执行的动作。例如：

- ArrayList(int initialCapacity)—Constructs an empty list with the specified initial capacity.

构造具有指定初始容量的空 List。

- Collection.size()—Returns the number of elements in this collection.

返回此集合中的元素数量。

As shown in these examples, use the third person declarative tense (“returns the number”) rather than the second person imperative (“return the number”).

如这些例子所示，应使用第三人称陈述句时态「returns the number」而不是第二人称祈使句「return the number」。

For classes, interfaces, and fields, the summary description should be a noun phrase describing the thing represented by an instance of the class or interface or by the field itself. For example:

对于类、接口和字段，摘要描述应该是一个名词短语，描述由类或接口的实例或字段本身表示的事物。例如：

- Instant—An instantaneous point on the time-line.

时间线上的瞬时点。

- Math.PI—The double value that is closer than any other to pi, the ratio of the circumference of a circle to its diameter.

这个 double 类型的值比任何其它值的都更接近于圆周率（圆周长与直径之比）。

In Java 9, a client-side index was added to the HTML generated by Javadoc. This index, which eases the task of navigating large API documentation sets, takes the form of a search box in the upper-right corner of the page. When you type into the box, you get a drop-down menu of matching pages. API elements, such as classes, methods, and fields, are indexed automatically. Occasionally you may wish to index additional terms that are important to your API. The {@index} tag was added for this purpose. Indexing a term that appears in a doc comment is as simple as wrapping it in this tag, as shown in this fragment:

在 Java 9 中，客户端索引被添加到 Javadoc 生成的 HTML 中。这个索引以页面右上角的搜索框的形式出现，它简化了导航大型 API 文档集的任务。当你在框中键入时，你将得到一个匹配页面的下拉菜单。API 元素（如类、方法和字段）是自动索引的。有时，你可能希望索引 API 中很重要的术语。为此添加了 `{@index}` 标记。对文档注释中出现的术语进行索引，就像将其包装在这个标签中一样简单，如下面的片段所示：

```Java
* This method complies with the {@index IEEE 754} standard.
```

Generics, enums, and annotations require special care in doc comments. **When documenting a generic type or method, be sure to document all type parameters:**

泛型、枚举和注解在文档注释中需要特别注意。**为泛型类型或方法编写文档时，请确保说明所有类型参数:**

```Java
/**
* An object that maps keys to values. A map cannot contain
* duplicate keys; each key can map to at most one value.
**
(Remainder omitted)
**
@param <K> the type of keys maintained by this map
* @param <V> the type of mapped values
*/
public interface Map<K, V> { ... }
```

**When documenting an enum type, be sure to document the constants** as well as the type and any public methods. Note that you can put an entire doc comment on one line if it’s short:

**编写枚举类型的文档时，一定要说明常量** 以及类型、任何公共方法。注意，如果文档很短，你可以把整个文档注释放在一行：

```Java
/**
* An instrument section of a symphony orchestra.
*/
public enum OrchestraSection {
/** Woodwinds, such as flute, clarinet, and oboe. */
WOODWIND,
/** Brass instruments, such as french horn and trumpet. */
BRASS,
/** Percussion instruments, such as timpani and cymbals. */
PERCUSSION,
/** Stringed instruments, such as violin and cello. */
STRING;
}
```

**When documenting an annotation type, be sure to document any Members** as well as the type itself. Document members with noun phrases, as if they were fields. For the summary description of the type, use a verb phrase that says what it means when a program element has an annotation of this type:

**为注释类型的文档时，请确保说明全部成员** 以及类型本身。用名词短语描述成员，就当它们是字段一样。对于类型的摘要描述，请使用动词短语，它表示当程序元素具有此类注解时的含义：

```Java
/**
* Indicates that the annotated method is a test method that
* must throw the designated exception to pass.
*/
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
/**
* The exception that the annotated test method must throw
* in order to pass. (The test is permitted to throw any
* subtype of the type described by this class object.)
*/
Class<? extends Throwable> value();
}
```

Package-level doc comments should be placed in a file named package-info.java. In addition to these comments, package-info.java must contain a package declaration and may contain annotations on this declaration. Similarly, if you elect to use the module system (Item 15), module-level comments should be placed in the module-info.java file.

包级别的文档注释应该放在名为 package info.java 的文件中。除了这些注释之外，package info.java 必须包含一个包声明，并且可能包含关于这个声明的注释。类似地，如果你选择使用模块系统（[Item-15](/Chapter-4/Chapter-4-Item-15-Minimize-the-accessibility-of-classes-and-members.md)），模块级别的注释应该放在 module-info.java 文件中。

Two aspects of APIs that are often neglected in documentation are threadsafety and serializability. **Whether or not a class or static method is threadsafe, you should document its thread-safety** level, as described in Item 82. If a class is serializable, you should document its serialized form, as described in Item 87.

在文档中经常忽略的 API 的两个方面是线程安全性和可序列化性。**无论类或静态方法是否线程安全，你都应该说明它的线程安全级别**，如 [Item-82](/Chapter-11/Chapter-11-Item-82-Document-thread-safety.md) 所述。如果一个类是可序列化的，你应该说明它的序列化形式，如 [Item-87](/Chapter-12/Chapter-12-Item-87-Consider-using-a-custom-serialized-form.md) 中所述。

Javadoc has the ability to “inherit” method comments. If an API element does not have a doc comment, Javadoc searches for the most specific applicable doc comment, giving preference to interfaces over superclasses. The details of the search algorithm can be found in The Javadoc Reference Guide [Javadoc-ref]. You can also inherit parts of doc comments from supertypes using the {@inheritDoc} tag. This means, among other things, that classes can reuse doc comments from interfaces they implement, rather than copying these comments. This facility has the potential to reduce the burden of maintaining multiple sets of nearly identical doc comments, but it is tricky to use and has some limitations. The details are beyond the scope of this book.

Javadoc 具有「继承」方法注释的能力。如果 API 元素没有文档注释，Javadoc 将搜索最适用的文档注释，优先选择接口而不是超类。搜索算法的详细信息可以在《The Javadoc Reference Guide》 [Javadoc-ref] 中找到。你还可以使用 `{@inheritDoc}` 标记从超类型继承部分文档注释。这意味着类可以复用它们实现的接口中的文档注释，而不是复制这些注释。这个工具有能力减少维护多个几乎相同的文档注释集的负担，但是它使用起来很棘手，并且有一些限制。这些细节超出了这本书的范围。

One caveat should be added concerning documentation comments. While it is necessary to provide documentation comments for all exported API elements, it is not always sufficient. For complex APIs consisting of multiple interrelated classes, it is often necessary to supplement the documentation comments with an external document describing the overall architecture of the API. If such a document exists, the relevant class or package documentation comments should include a link to it.

关于文档注释，有一点需要特别注意。虽然有必要为所有公开的 API 元素提供文档注释，但这并不总是足够的。对于由多个相互关联的类组成的复杂 API，通常需要用描述 API 总体架构的外部文档来补充文档注释。如果存在这样的文档，相关的类或包文档注释应该包含到它的链接。

Javadoc automatically checks for adherence to many of the recommendations in this item. In Java 7, the command line switch -Xdoclint was required to get this behavior. In Java 8 and 9, checking is enabled by default. IDE plug-ins such as checkstyle go further in checking for adherence to these recommendations [Burn01]. You can also reduce the likelihood of errors in doc comments by running the HTML files generated by Javadoc through an HTML validity checker. This will detect many incorrect uses of HTML tags. Several such checkers are available for download, and you can validate HTML on the web using the W3C markup validation service [W3C-validator]. When validating generated HTML, keep in mind that as of Java 9, Javadoc is capable of generating HTML5 as well as HTML 4.01, though it still generates HTML 4.01 by default. Use the -html5 command line switch if you want Javadoc to generate HTML5.

Javadoc 会自动检查文档是否符合本项目中提及的许多建议。在 Java 7 中，需要命令行开关 `-Xdoclint` 来获得这种特性。在 Java 8 和 Java 9 中，默认情况下启用了该机制。诸如 checkstyle 之类的 IDE 插件在检查是否符合这些建议方面做得更好 [Burn01]。还可以通过 HTML 有效性检查器运行 Javadoc 生成的 HTML 文件来降低文档注释中出现错误的可能性。这将检测 HTML 标记的许多不正确用法。有几个这样的检查器可供下载，你可以使用 W3C 标记验证服务 [W3C-validator] 在 web 上验证 HTML。在验证生成的 HTML 时，请记住，从 Java 9 开始，Javadoc 就能够生成 HTML 5 和 HTML 4.01，尽管默认情况下它仍然生成 HTML 4.01。如果希望 Javadoc 生成 HTML5，请使用 `-html5` 命令行开关。

The conventions described in this item cover the basics. Though it is fifteen years old at the time of this writing, the definitive guide to writing doc comments is still How to Write Doc Comments [Javadoc-guide]. If you adhere to the guidelines in this item, the generated documentation should provide a clear description of your API. The only way to know for sure, however, is to **read the web pages generated by the Javadoc utility.** It is worth doing this for every API that will be used by others. Just as testing a program almost inevitably results in some changes to the code, reading the documentation generally results in at least a few minor changes to the doc comments.

本条目中描述的约定涵盖了基本内容。尽管撰写本文时已经有 15 年的历史，但编写文档注释的最终指南仍然是《How to Write Doc Comments》[Javadoc-guide]。如果你遵循本条目中的指导原则，生成的文档应该提供对 API 的清晰描述。然而，唯一确定的方法是 **读取 Javadoc 实用程序生成的 web 页面。** 对于其他人将使用的每个 API 都值得这样做。正如测试程序几乎不可避免地会导致对代码的一些更改一样，阅读文档通常也会导致对文档注释的一些较小更改。

To summarize, documentation comments are the best, most effective way to document your API. Their use should be considered mandatory for all exported API elements. Adopt a consistent style that adheres to standard conventions. Remember that arbitrary HTML is permissible in documentation comments and that HTML metacharacters must be escaped.

总之，文档注释是记录API的最佳、最有效的方法。应该认为，所有公开的 API 元素都必须使用文档注释，并采用符合标准约定的统一样式。请记住，在文档注释中允许使用任意 HTML 标签，并且必须转义 HTML 元字符。

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-8/Chapter-8-Introduction.md)**
- **Previous Item（上一条目）：[Item 55: Return optionals judiciously（明智地的返回 Optional）](/Chapter-8/Chapter-8-Item-55-Return-optionals-judiciously.md)**
- **Next Item（下一条目）：[Chapter 9 Introduction（章节介绍）](/Chapter-9/Chapter-9-Introduction.md)**
