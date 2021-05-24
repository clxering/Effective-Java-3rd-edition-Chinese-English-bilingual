## Chapter 9. General Programming（通用程序设计）

### Item 58: Prefer for-each loops to traditional for loops（for-each 循环优于传统的 for 循环）

As discussed in Item 45, some tasks are best accomplished with streams, others with iteration. Here is a traditional for loop to iterate over a collection:

正如在 [Item-45](/Chapter-7/Chapter-7-Item-45-Use-streams-judiciously.md) 中所讨论的，一些任务最好使用流来完成，其他任务最好使用 iteration。下面是使用一个传统的 for 循环来遍历一个集合：

```Java
// Not the best way to iterate over a collection!
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
    Element e = i.next();
    ... // Do something with e
}
```

and here is a traditional for loop to iterate over an array:

这是使用传统的 for 循环来遍历数组：

```Java
// Not the best way to iterate over an array!
for (int i = 0; i < a.length; i++) {
    ... // Do something with a[i]
}
```

These idioms are better than while loops (Item 57), but they aren’t perfect. The iterator and the index variables are both just clutter—all you need are the elements. Furthermore, they represent opportunities for error. The iterator occurs three times in each loop and the index variable four, which gives you many chances to use the wrong variable. If you do, there is no guarantee that the compiler will catch the problem. Finally, the two loops are quite different, drawing unnecessary attention to the type of the container and adding a (minor) hassle to changing that type.

这些习惯用法比 while 循环更好（[Item-57](/Chapter-9/Chapter-9-Item-57-Minimize-the-scope-of-local-variables.md)），但是它们并不完美。迭代器和索引变量都很混乱（你只需要元素）。此外，它们有出错的可能。迭代器在每个循环中出现三次，索引变量出现四次，这使得有很多机会使用到错误的变量。如果这样做，就不能保证编译器会捕捉到问题。最后，这两个循环区别很大，（第一个例子）还需要额外注意容器类型，并给类型转换增加小麻烦。

The for-each loop (officially known as the “enhanced for statement”) solves all of these problems. It gets rid of the clutter and the opportunity for error by hiding the iterator or index variable. The resulting idiom applies equally to collections and arrays, easing the process of switching the implementation type of a container from one to the other:

for-each 循环（官方称为「enhanced for 语句」）解决了所有这些问题。它通过隐藏迭代器或索引变量来消除混乱和出错的机会。由此产生的习惯用法同样适用于集合和数组，从而简化了将容器的实现类型从一种转换为另一种的过程：

```Java
// The preferred idiom for iterating over collections and arrays
for (Element e : elements) {
    ... // Do something with e
}
```

When you see the colon (:), read it as “in.” Thus, the loop above reads as “for each element e in elements.” There is no performance penalty for using for-each loops, even for arrays: the code they generate is essentially identical to the code you would write by hand.

当你看到冒号 `(:)` 时，请将其读作「in」。因此，上面的循环读作「对元素集的每个元素 e 进行操作」。使用 for-each 循环不会降低性能，对于数组也是如此：它们生成的代码本质上与你手工编写的 for 循环代码相同。

The advantages of the for-each loop over the traditional for loop are even greater when it comes to nested iteration. Here is a common mistake that people make when doing nested iteration:

当涉及到嵌套迭代时，for-each 循环相对于传统 for 循环的优势甚至更大。下面是人们在进行嵌套迭代时经常犯的一个错误：

```Java
// Can you spot the bug?
enum Suit { CLUB, DIAMOND, HEART, SPADE }
enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT,NINE, TEN, JACK, QUEEN, KING }
...
static Collection<Suit> suits = Arrays.asList(Suit.values());
static Collection<Rank> ranks = Arrays.asList(Rank.values());
List<Card> deck = new ArrayList<>();
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
deck.add(new Card(i.next(), j.next()));
```

Don’t feel bad if you didn’t spot the bug. Many expert programmers have made this mistake at one time or another. The problem is that the next method is called too many times on the iterator for the outer collection (suits). It should be called from the outer loop so that it is called once per suit, but instead it is called from the inner loop, so it is called once per card. After you run out of suits, the loop throws a NoSuchElementException.

如果你没有发现这个bug，不要感到难过。许多专业程序员都曾犯过这样的错误。问题是，迭代器对外部的集合 suits 调用了太多次 next 方法。它应该从外部循环调用，因此每种花色调用一次，但它是从内部循环调用的，因此每一张牌调用一次。在用完所有花色之后，循环抛出 NoSuchElementException。

If you’re really unlucky and the size of the outer collection is a multiple of the size of the inner collection—perhaps because they’re the same collection—the loop will terminate normally, but it won’t do what you want. For example, consider this ill-conceived attempt to print all the possible rolls of a pair of dice:

如果真的很不幸，外部集合的大小是内部集合大小的几倍（可能因为它们是相同的集合），循环将正常终止，但是它不会执行你想要的操作。例如，考虑一个打印一对骰子所有可能的组合值的错误尝试：

```Java
// Same bug, different symptom!
enum Face { ONE, TWO, THREE, FOUR, FIVE, SIX }
...
Collection<Face> faces = EnumSet.allOf(Face.class);
for (Iterator<Face> i = faces.iterator(); i.hasNext(); )
for (Iterator<Face> j = faces.iterator(); j.hasNext(); )
System.out.println(i.next() + " " + j.next());
```

The program doesn’t throw an exception, but it prints only the six “doubles” (from “ONE ONE” to “SIX SIX”), instead of the expected thirty-six combinations.

程序不会抛出异常，但它只打印 6 个重复数值（从「ONE ONE」到「SIX SIX」），而不是预期的 36 个组合。

To fix the bugs in these examples, you must add a variable in the scope of the outer loop to hold the outer element:

要修复这些例子中的错误，必须在外部循环的作用域内添加一个变量来保存外部元素：

```Java
// Fixed, but ugly - you can do better!
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); ) {
    Suit suit = i.next();
    for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
        deck.add(new Card(suit, j.next()));
}
```

If instead you use a nested for-each loop, the problem simply disappears. The resulting code is as succinct as you could wish for:

相反，如果使用嵌套 for-each 循环，问题就会消失。生成的代码更简洁：

```Java
// Preferred idiom for nested iteration on collections and arrays
for (Suit suit : suits)
for (Rank rank : ranks)
deck.add(new Card(suit, rank));
```

Unfortunately, there are three common situations where you can’t use foreach:

不幸的是，有三种常见的情况你不应使用 for-each：

- **Destructive filtering** —If you need to traverse a collection removing selected elements, then you need to use an explicit iterator so that you can call its remove method. You can often avoid explicit traversal by using Collection’s removeIf method, added in Java 8.

**破坏性过滤**，如果需要遍历一个集合并删除选定元素，则需要使用显式的迭代器，以便调用其 remove 方法。通过使用 Collection 在 Java 8 中添加的 removeIf 方法，通常可以避免显式遍历。

- **Transforming** —If you need to traverse a list or array and replace some or all of the values of its elements, then you need the list iterator or array index in order to replace the value of an element.

**转换**，如果需要遍历一个 List 或数组并替换其中部分或全部元素的值，那么需要 List 迭代器或数组索引来替换元素的值。

- **Parallel iteration** —If you need to traverse multiple collections in parallel, then you need explicit control over the iterator or index variable so that all iterators or index variables can be advanced in lockstep (as demonstrated unintentionally in the buggy card and dice examples above). If you find yourself in any of these situations, use an ordinary for loop and be wary of the traps mentioned in this item.

**并行迭代**，如果需要并行遍历多个集合，那么需要显式地控制迭代器或索引变量，以便所有迭代器或索引变量都可以同步执行（如上述牌和骰子示例中无意中演示的错误那样）。如果发现自己处于这些情况中的任何一种，请使用普通的 for 循环，并警惕本条目中提到的陷阱。

Not only does the for-each loop let you iterate over collections and arrays, it lets you iterate over any object that implements the Iterable interface, which consists of a single method. Here is how the interface looks:

for-each 循环不仅允许遍历集合和数组，还允许遍历实现 Iterable 接口的任何对象，该接口由一个方法组成。如下所示：

```Java
public interface Iterable<E> {
    // Returns an iterator over the elements in this iterable
    Iterator<E> iterator();
}
```

It is a bit tricky to implement Iterable if you have to write your own Iterator implementation from scratch, but if you are writing a type that represents a group of elements, you should strongly consider having it implement Iterable, even if you choose not to have it implement Collection. This will allow your users to iterate over your type using the foreach loop, and they will be forever grateful.

如果必须从头开始编写自己的 Iterator 实现，确实有点棘手，但是如果正在编写的类型表示一组元素，即使选择不让它实现 Collection，那么也应该强烈考虑让它实现 Iterable。这将允许用户使用 foreach 循环遍历类型，他们将永远感激不尽。

In summary, the for-each loop provides compelling advantages over the traditional for loop in clarity, flexibility, and bug prevention, with no performance penalty. Use for-each loops in preference to for loops wherever you can.

总之，for-each 循环在清晰度、灵活性和 bug 预防方面比传统的 for 循环更有优势，并且没有性能损失。尽可能使用 for-each 循环而不是 for 循环。

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-9/Chapter-9-Introduction.md)**
- **Previous Item（上一条目）：[Item 57: Minimize the scope of local variables（将局部变量的作用域最小化）](/Chapter-9/Chapter-9-Item-57-Minimize-the-scope-of-local-variables.md)**
- **Next Item（下一条目）：[Item 59: Know and use the libraries（了解并使用库）](/Chapter-9/Chapter-9-Item-59-Know-and-use-the-libraries.md)**
