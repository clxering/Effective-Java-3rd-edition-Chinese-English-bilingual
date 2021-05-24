## Chapter 9. General Programming（通用程序设计）

### Item 63: Beware the performance of string concatenation（当心字符串连接引起的性能问题）

The string concatenation operator (+) is a convenient way to combine a few strings into one. It is fine for generating a single line of output or constructing the string representation of a small, fixed-size object, but it does not scale. Using **the string concatenation operator repeatedly to concatenate n strings requires time quadratic in n.** This is an unfortunate consequence of the fact that strings are immutable (Item 17). When two strings are concatenated, the contents of both are copied.

字符串连接操作符 `(+)` 是将几个字符串组合成一个字符串的简便方法。对于生成单行输出或构造一个小的、固定大小的对象的字符串表示形式，它是可以的，但是它不能伸缩。使用 **字符串串联运算符重复串联 n 个字符串需要 n 的平方级时间。** 这是字符串不可变这一事实导致的结果（[Item-17](/Chapter-4/Chapter-4-Item-17-Minimize-mutability.md)）。当连接两个字符串时，将复制这两个字符串的内容。

For example, consider this method, which constructs the string representation of a billing statement by repeatedly concatenating a line for each item:

例如，考虑这个方法，它通过将每个账单项目重复连接到一行来构造账单语句的字符串表示：

```Java
// Inappropriate use of string concatenation - Performs poorly!
public String statement() {
    String result = "";
    for (int i = 0; i < numItems(); i++)
        result += lineForItem(i); // String concatenation
    return result;
}
```

The method performs abysmally if the number of items is large. **To achieve acceptable performance, use a StringBuilder in place of a String** to store the statement under construction:

如果项的数量很大，则该方法的性能非常糟糕。**要获得能接受的性能，请使用 StringBuilder 代替 String** 来存储正在构建的语句：

```Java
public String statement() {
    StringBuilder b = new StringBuilder(numItems() * LINE_WIDTH);
    for (int i = 0; i < numItems(); i++)
        b.append(lineForItem(i));
    return b.toString();
}
```

A lot of work has gone into making string concatenation faster since Java 6, but the difference in the performance of the two methods is still dramatic: If numItems returns 100 and lineForItem returns an 80-character string, the second method runs 6.5 times faster than the first on my machine. Because the first method is quadratic in the number of items and the second is linear, the performance difference gets much larger as the number of items grows. Note that the second method preallocates a StringBuilder large enough to hold the entire result, eliminating the need for automatic growth. Even if it is detuned to use a default-sized StringBuilder, it is still 5.5 times faster than the first method.

自 Java 6 以来，为了使字符串连接更快，已经做了大量工作，但是这两个方法在性能上的差异仍然很大：如果 numItems 返回 100，lineForItem 返回 80 个字符串，那么第二个方法在我的机器上运行的速度是第一个方法的 6.5 倍。由于第一种方法在项目数量上是平方级的，而第二种方法是线性的，所以随着项目数量的增加，性能差异会变得越来越大。注意，第二个方法预先分配了一个足够大的 StringBuilder 来保存整个结果，从而消除了自动增长的需要。即使使用默认大小的 StringBuilder，它仍然比第一个方法快 5.5 倍。

The moral is simple: **Don’t use the string concatenation operator to combine more than a few strings** unless performance is irrelevant. Use StringBuilder’s append method instead. Alternatively, use a character array, or process the strings one at a time instead of combining them.

道理很简单：**不要使用字符串连接操作符合并多个字符串**，除非性能无关紧要。否则使用 StringBuilder 的 append 方法。或者，使用字符数组，再或者一次只处理一个字符串，而不是组合它们。

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-9/Chapter-9-Introduction.md)**
- **Previous Item（上一条目）：[Item 62: Avoid strings where other types are more appropriate（其他类型更合适时应避免使用字符串）](/Chapter-9/Chapter-9-Item-62-Avoid-strings-where-other-types-are-more-appropriate.md)**
- **Next Item（下一条目）：[Item 64: Refer to objects by their interfaces（通过接口引用对象）](/Chapter-9/Chapter-9-Item-64-Refer-to-objects-by-their-interfaces.md)**
