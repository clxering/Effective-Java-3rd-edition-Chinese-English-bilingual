## Chapter 5. Generics（泛型）

### Item 31: Use bounded wildcards to increase API flexibility（使用有界通配符增加 API 的灵活性）

As noted in Item 28, parameterized types are invariant. In other words, for any two distinct types `Type1` and `Type2`, `List<Type1>` is neither a subtype nor a supertype of `List<Type2>`. Although it is counterintuitive that `List<String>` is not a subtype of `List<Object>`, it really does make sense. You can put any object into a `List<Object>`, but you can put only strings into a `List<String>`. Since a `List<String>` can’t do everything a `List<Object>` can, it isn’t a subtype (by the Liskov substitution principal, Item 10).

如 [Item-28](/Chapter-5/Chapter-5-Item-28-Prefer-lists-to-arrays.md) 所示，参数化类型是不可变的。换句话说，对于任意两种不同类型 `Type1` 和 `Type2`，`List<Type1>` 既不是 `List<Type2>` 的子类型，也不是它的父类。虽然 `List<String>` 不是 `List<Object>` 的子类型，这和习惯的直觉不符，但它确实有意义。你可以将任何对象放入 `List<Object>`，但只能将字符串放入 `List<String>`。因为 `List<String>` 不能做 `List<Object>` 能做的所有事情，所以它不是子类型（可通过 Liskov 替换原则来理解这一点，[Item-10](/Chapter-3/Chapter-3-Item-10-Obey-the-general-contract-when-overriding-equals.md)）。

**译注：里氏替换原则（Liskov Substitution Principle，LSP）面向对象设计的基本原则之一。里氏替换原则指出：任何父类可以出现的地方，子类一定可以出现。LSP 是继承复用的基石，只有当衍生类可以替换掉父类，软件单位的功能不受到影响时，父类才能真正被复用，而衍生类也能够在父类的基础上增加新的行为。**

Sometimes you need more flexibility than invariant typing can provide. Consider the Stack class from Item 29. To refresh your memory, here is its public API:

有时你需要获得比不可变类型更多的灵活性。考虑 [Item-29](/Chapter-5/Chapter-5-Item-29-Favor-generic-types.md) 中的堆栈类。以下是它的公共 API：

```Java
public class Stack<E> {
    public Stack();
    public void push(E e);
    public E pop();
    public boolean isEmpty();
}
```

Suppose we want to add a method that takes a sequence of elements and pushes them all onto the stack. Here’s a first attempt:

假设我们想添加一个方法，该方法接受一系列元素并将它们全部推入堆栈。这是第一次尝试：

```Java
// pushAll method without wildcard type - deficient!
public void pushAll(Iterable<E> src) {
    for (E e : src)
        push(e);
}
```

This method compiles cleanly, but it isn’t entirely satisfactory. If the element type of the `Iterable src` exactly matches that of the stack, it works fine. But suppose you have a `Stack<Number>` and you invoke `push(intVal)`, where `intVal` is of type `Integer`. This works because `Integer` is a subtype of `Number`. So logically, it seems that this should work, too:

该方法能够正确编译，但并不完全令人满意。如果 `Iterable src` 的元素类型与堆栈的元素类型完全匹配，那么它正常工作。但是假设你有一个 `Stack<Number>`，并且调用 `push(intVal)`，其中 `intVal` 的类型是 `Integer`。这是可行的，因为 `Integer` 是 `Number` 的子类型。因此，从逻辑上讲，这似乎也应该奏效：

```Java
Stack<Number> numberStack = new Stack<>();
Iterable<Integer> integers = ... ;
numberStack.pushAll(integers);
```

If you try it, however, you’ll get this error message because parameterized types are invariant:

但是，如果你尝试一下，将会得到这个错误消息，因为参数化类型是不可变的：

```Java
StackTest.java:7: error: incompatible types: Iterable<Integer>
cannot be converted to Iterable<Number>
        numberStack.pushAll(integers);
                    ^
```

Luckily, there’s a way out. The language provides a special kind of parameterized type call a bounded wildcard type to deal with situations like this. The type of the input parameter to pushAll should not be “Iterable of E” but “Iterable of some subtype of E,” and there is a wildcard type that means precisely that: Iterable<? extends E>. (The use of the keyword extends is slightly misleading: recall from Item 29 that subtype is defined so that every type is a subtype of itself, even though it does not extend itself.) Let’s modify pushAll to use this type:

幸运的是，有一种解决方法。Java 提供了一种特殊的参数化类型，`有界通配符类型`来处理这种情况。pushAll 的输入参数的类型不应该是「E 的 Iterable 接口」，而应该是「E 的某个子类型的 Iterable 接口」，并且有一个通配符类型，它的确切含义是：`Iterable<? extends E>`（关键字 extends 的使用稍微有些误导：回想一下 [Item-29](/Chapter-5/Chapter-5-Item-29-Favor-generic-types.md)，定义了子类型，以便每个类型都是其本身的子类型，即使它没有扩展自己。）让我们修改 pushAll 来使用这种类型：

```Java
// Wildcard type for a parameter that serves as an E producer
public void pushAll(Iterable<? extends E> src) {
    for (E e : src)
        push(e);
}
```

With this change, not only does Stack compile cleanly, but so does the client code that wouldn’t compile with the original pushAll declaration. Because Stack and its client compile cleanly, you know that everything is typesafe. Now suppose you want to write a popAll method to go with pushAll. The popAll method pops each element off the stack and adds the elements to the given collection. Here’s how a first attempt at writing the popAll method might look:

更改之后，不仅 Stack 可以正确编译，而且不能用原始 pushAll 声明编译的客户端代码也可以正确编译。因为 Stack 和它的客户端可以正确编译，所以你知道所有东西都是类型安全的。现在假设你想编写一个与 pushAll 一起使用的 popAll 方法。popAll 方法将每个元素从堆栈中弹出，并将这些元素添加到给定的集合中。下面是编写 popAll 方法的第一次尝试：

```Java
// popAll method without wildcard type - deficient!
public void popAll(Collection<E> dst) {
    while (!isEmpty())
        dst.add(pop());
}
```

Again, this compiles cleanly and works fine if the element type of the destination collection exactly matches that of the stack. But again, it isn’t entirely satisfactory. Suppose you have a `Stack<Number>` and variable of type Object. If you pop an element from the stack and store it in the variable, it compiles and runs without error. So shouldn’t you be able to do this, too?

同样，如果目标集合的元素类型与堆栈的元素类型完全匹配，那么这种方法可以很好地编译。但这也不是完全令人满意。假设你有一个 `Stack<Number>` 和 Object 类型的变量。如果从堆栈中取出一个元素并将其存储在变量中，那么它将编译并运行，不会出错。所以你不能也这样做吗？

```Java
Stack<Number> numberStack = new Stack<Number>();
Collection<Object> objects = ... ;
numberStack.popAll(objects);
```

If you try to compile this client code against the version of popAll shown earlier, you’ll get an error very similar to the one that we got with our first version of pushAll: `Collection<Object>` is not a subtype of `Collection<Number>`. Once again, wildcard types provide a way out. The type of the input parameter to popAll should not be “collection of E” but “collection of some supertype of E” (where supertype is defined such that E is a supertype of itself [JLS, 4.10]). Again, there is a wildcard type that means precisely that: Collection<? super E>. Let’s modify popAll to use it:

如果你尝试根据前面显示的 popAll 版本编译此客户端代码，你将得到一个与第一个版本的 pushAll 非常相似的错误：`Collection<Object>`不是 `Collection<Number>` 的子类型。同样，通配符类型提供解决方法。popAll 的输入参数的类型不应该是「E 的集合」，而应该是「E 的某个超类型的集合」（其中的超类型定义为 E 本身是一个超类型[JLS, 4.10]）。同样，有一个通配符类型，它的确切含义是：`Collection<? super E>`。让我们修改 popAll 来使用它：

```Java
// Wildcard type for parameter that serves as an E consumer
public void popAll(Collection<? super E> dst) {
  while (!isEmpty())
    dst.add(pop());
}
```

With this change, both Stack and the client code compile cleanly.

通过此更改，Stack 类和客户端代码都可以正确编译。

The lesson is clear. **For maximum flexibility, use wildcard types on input parameters that represent producers or consumers.** If an input parameter is both a producer and a consumer, then wildcard types will do you no good: you need an exact type match, which is what you get without any wildcards. Here is a mnemonic to help you remember which wildcard type to use:

教训是清楚的。为了获得最大的灵活性，应在表示生产者或消费者的输入参数上使用通配符类型。如果输入参数既是生产者又是消费者，那么通配符类型对你没有任何好处：你需要一个精确的类型匹配，这就是在没有通配符的情况下得到的结果。这里有一个助记符帮助你记住使用哪种通配符类型：

**PECS stands for producer-extends, consumer-super.**

PECS 表示生产者应使用 extends，消费者应使用 super。

In other words, if a parameterized type represents a T producer, use `<? extends T>`; if it represents a T consumer, use `<? super T>`. In our Stack example, pushAll’s src parameter produces E instances for use by the Stack, so the appropriate type for src is `Iterable<? extends E>`; popAll’s dst parameter consumes E instances from the Stack, so the appropriate type for dst is `Collection<? super E>`. The PECS mnemonic captures the fundamental principle that guides the use of wild-card types. Naftalin and Wadler call it the Get and Put Principle [Naftalin07, 2.4].

换句话说，如果参数化类型表示 T 生成器，则使用 `<? extends T>`；如果它表示一个 T 消费者，则使用 `<? super T>`。在我们的 Stack 示例中，pushAll 的 src 参数生成 E 的实例供 Stack 使用，因此 src 的适当类型是 `Iterable<? extends E>`；popAll 的 dst 参数使用 Stack 中的 E 实例，因此适合 dst 的类型是 `Collection<? super E>`。PECS 助记符捕获了指导通配符类型使用的基本原则。Naftalin 和 Wadler 称之为 Get and Put 原则[Naftalin07, 2.4]。

With this mnemonic in mind, let’s take a look at some method and constructor declarations from previous items in this chapter. The Chooser constructor in Item 28 has this declaration:

记住这个助记符后，再让我们看一看本章前面提及的一些方法和构造函数声明。[Item-28](/Chapter-5/Chapter-5-Item-28-Prefer-lists-to-arrays.md) 中的 Chooser 构造函数有如下声明：

```Java
public Chooser(Collection<T> choices)
```

This constructor uses the collection choices only to produce values of type T (and stores them for later use), so its declaration should use a wildcard type that **extends T.** Here’s the resulting constructor declaration:

这个构造函数只使用集合选项来生成类型 T 的值（并存储它们以供以后使用），因此它的声明应该使用扩展 T 的通配符类型 **extends T**。下面是生成的构造函数声明：

```Java
// Wildcard type for parameter that serves as an T producer
public Chooser(Collection<? extends T> choices)
```

And would this change make any difference in practice? Yes, it would. Suppose you have a `List<Integer>`, and you want to pass it in to the constructor for a Chooser<Number>. This would not compile with the original declaration, but it does once you add the bounded wildcard type to the declaration.

这种改变在实践中会有什么不同吗？是的，它会。假设你有一个 `List<Integer>`，并且希望将其传递给 `Chooser<Number>` 的构造函数。这不会与原始声明一起编译，但是一旦你将有界通配符类型添加到声明中，它就会编译。

Now let’s look at the union method from Item 30. Here is the declaration:

现在让我们看看 [Item-30](/Chapter-5/Chapter-5-Item-30-Favor-generic-methods.md) 中的 union 方法。以下是声明：

```Java
public static <E> Set<E> union(Set<E> s1, Set<E> s2)
```

Both parameters, s1 and s2, are E producers, so the PECS mnemonic tells us that the declaration should be as follows:

参数 s1 和 s2 都是 E 的生产者，因此 PECS 助记符告诉我们声明应该如下：

```Java
public static <E> Set<E> union(Set<? extends E> s1,Set<? extends E> s2)
```

Note that the return type is still `Set<E>`. **Do not use bounded wildcard types as return types.** Rather than providing additional flexibility for your users, it would force them to use wildcard types in client code. With the revised declaration, this code will compile cleanly:

注意，返回类型仍然设置为 `Set<E>`。**不要使用有界通配符类型作为返回类型。** 它将强制用户在客户端代码中使用通配符类型，而不是为用户提供额外的灵活性。经修订后的声明可正确编译以下代码：

```Java
Set<Integer> integers = Set.of(1, 3, 5);
Set<Double> doubles = Set.of(2.0, 4.0, 6.0);
Set<Number> numbers = union(integers, doubles);
```

Properly used, wildcard types are nearly invisible to the users of a class. They cause methods to accept the parameters they should accept and reject those they should reject. **If the user of a class has to think about wildcard types, there is probably something wrong with its API.**

如果使用得当，通配符类型对于类的用户几乎是不可见的。它们让方法接受它们应该接受的参数，拒绝应该拒绝的参数。**如果类的用户必须考虑通配符类型，那么它的 API 可能有问题。**

Prior to Java 8, the type inference rules were not clever enough to handle the previous code fragment, which requires the compiler to use the contextually specified return type (or target type) to infer the type of E. The target type of the union invocation shown earlier is `Set<Number>`. If you try to compile the fragment in an earlier version of Java (with an appropriate replacement for the Set.of factory), you’ll get a long, convoluted error message like this:

在 Java 8 之前，类型推断规则还不足以处理前面的代码片段，这要求编译器使用上下文指定的返回类型（或目标类型）来推断 E 的类型。前面显示的 union 调用的目标类型设置为 `Set<Number>` 如果你尝试在 Java 的早期版本中编译该片段（使用 Set.of factory 的适当替代），你将得到一条长而复杂的错误消息，如下所示：

```Java
Union.java:14: error: incompatible types
Set<Number> numbers = union(integers, doubles);
^ required: Set<Number>
found: Set<INT#1>
where INT#1,INT#2 are intersection types:
INT#1 extends Number,Comparable<? extends INT#2>
INT#2 extends Number,Comparable<?>
```

Luckily there is a way to deal with this sort of error. If the compiler doesn’t infer the correct type, you can always tell it what type to use with an explicit type argument [JLS, 15.12]. Even prior to the introduction of target typing in Java 8, this isn’t something that you had to do often, which is good because explicit type arguments aren’t very pretty. With the addition of an explicit type argument, as shown here, the code fragment compiles cleanly in versions prior to Java 8:

幸运的是，有一种方法可以处理这种错误。如果编译器没有推断出正确的类型，你总是可以告诉它使用显式类型参数[JLS, 15.12]使用什么类型。即使在 Java 8 中引入目标类型之前，这也不是必须经常做的事情，这很好，因为显式类型参数不是很漂亮。通过添加显式类型参数，如下所示，代码片段可以在 Java 8 之前的版本中正确编译：

```Java
// Explicit type parameter - required prior to Java 8
Set<Number> numbers = Union.<Number>union(integers, doubles);
```

Next let’s turn our attention to the max method in Item 30. Here is the original declaration:

接下来让我们将注意力转到 [Item-30](/Chapter-5/Chapter-5-Item-30-Favor-generic-methods.md) 中的 max 方法。以下是原始声明：

```Java
public static <T extends Comparable<T>> T max(List<T> list)
```

Here is a revised declaration that uses wildcard types:

下面是使用通配符类型的修正声明：

```Java
public static <T extends Comparable<? super T>> T max(List<? extends T> list)
```

To get the revised declaration from the original, we applied the PECS heuristic twice. The straightforward application is to the parameter list. It produces T instances, so we change the type from `List<T>` to `List<? extends T>`. The tricky application is to the type parameter T. This is the first time we’ve seen a wildcard applied to a type parameter. Originally, T was specified to extend `Comparable<T>`, but a comparable of T consumes T instances (and produces integers indicating order relations). Therefore, the parameterized type `Comparable<T>` is replaced by the bounded wildcard type `Comparable<? super T>`. Comparables are always consumers, so you should generally **use `Comparable<? super T>` in preference to `Comparable<T>`.** The same is true of comparators; therefore, you should generally **use `Comparator<? super T>` in preference to `Comparator<T>`.**

为了从原始声明中得到修改后的声明，我们两次应用了 PECS 启发式。直接的应用程序是参数列表。它生成 T 的实例，所以我们将类型从 `List<T>` 更改为 `List<? extends T>`。复杂的应用是类型参数 T。这是我们第一次看到通配符应用于类型参数。最初，T 被指定为扩展 `Comparable<T>`，但是 T 的 Comparable 消费 T 实例（并生成指示顺序关系的整数）。因此，将参数化类型 `Comparable<T>` 替换为有界通配符类型 `Comparable<? super T>`，Comparables 始终是消费者，所以一般应**优先使用 `Comparable<? super T>` 而不是 `Comparable<T>`**，比较器也是如此；因此，通常应该**优先使用 `Comparator<? super T>` 而不是 `Comparator<T>`。**

The revised max declaration is probably the most complex method declaration in this book. Does the added complexity really buy you anything? Again, it does. Here is a simple example of a list that would be excluded by the original declaration but is permitted by the revised one:

修订后的 max 声明可能是本书中最复杂的方法声明。增加的复杂性真的能给你带来什么好处吗？是的，它再次生效。下面是一个简单的列表案例，它在原来的声明中不允许使用，但经订正的声明允许：

```Java
List<ScheduledFuture<?>> scheduledFutures = ... ;
```

The reason that you can’t apply the original method declaration to this list is that ScheduledFuture does not implement `Comparable<ScheduledFuture>`. Instead, it is a subinterface of Delayed, which extends `Comparable<Delayed>`. In other words, a ScheduledFuture instance isn’t merely comparable to other ScheduledFuture instances; it is comparable to any Delayed instance, and that’s enough to cause the original declaration to reject it. More generally, the wildcard is required to support types that do not implement Comparable (or Comparator) directly but extend a type that does.

不能将原始方法声明应用于此列表的原因是 ScheduledFuture 没有实现 `Comparable<ScheduledFuture>`。相反，它是 Delayed 的一个子接口，扩展了 `Comparable<Delayed>`。换句话说，ScheduledFuture 的实例不仅仅可以与其他 ScheduledFuture 实例进行比较；它可以与任何 Delayed 实例相比较，这足以导致初始声明时被拒绝。更通俗来说，通配符用于支持不直接实现 Comparable（或 Comparator）但扩展了实现 Comparable（或 Comparator）的类型的类型。

There is one more wildcard-related topic that bears discussing. There is a duality between type parameters and wildcards, and many methods can be declared using one or the other. For example, here are two possible declarations for a static method to swap two indexed items in a list. The first uses an unbounded type parameter (Item 30) and the second an unbounded wildcard:

还有一个与通配符相关的主题值得讨论。类型参数和通配符之间存在对偶性，可以使用其中一种方法声明许多方法。例如，下面是静态方法的两种可能声明，用于交换列表中的两个索引项。第一个使用无界类型参数（[Item-30](/Chapter-5/Chapter-5-Item-30-Favor-generic-methods.md)），第二个使用无界通配符：

```Java
// Two possible declarations for the swap method
public static <E> void swap(List<E> list, int i, int j);
public static void swap(List<?> list, int i, int j);
```

Which of these two declarations is preferable, and why? In a public API, the second is better because it’s simpler. You pass in a list—any list—and the method swaps the indexed elements. There is no type parameter to worry about. As a rule, **if a type parameter appears only once in a method declaration, replace it with a wildcard.** If it’s an unbounded type parameter, replace it with an unbounded wildcard; if it’s a bounded type parameter, replace it with a bounded wildcard.

这两个声明中哪个更好，为什么？在公共 API 中第二个更好，因为它更简单。传入一个列表（任意列表），该方法交换索引元素。不需要担心类型参数。通常，如果类型参数在方法声明中只出现一次，则用通配符替换它。**如果它是一个无界类型参数，用一个无界通配符替换它；** 如果它是有界类型参数，则用有界通配符替换它。

There’s one problem with the second declaration for swap. The straightforward implementation won’t compile:

交换的第二个声明有一个问题。这个简单的实现无法编译：

```Java
public static void swap(List<?> list, int i, int j) {
  list.set(i, list.set(j, list.get(i)));
}
```

Trying to compile it produces this less-than-helpful error message:

试图编译它会产生一个不太有用的错误消息：

```Java
Swap.java:5: error: incompatible types: Object cannot be
converted to CAP#1
list.set(i, list.set(j, list.get(i)));
^ where CAP#1
is a fresh type-variable: CAP#1 extends Object from capture of ?
```

It doesn’t seem right that we can’t put an element back into the list that we just took it out of. The problem is that the type of list is `List<?>`, and you can’t put any value except null into a `List<?>`. Fortunately, there is a way to implement this method without resorting to an unsafe cast or a raw type. The idea is to write a private helper method to capture the wildcard type. The helper method must be a generic method in order to capture the type. Here’s how it looks:

我们不能把一个元素放回刚刚取出的列表中，这看起来是不正确的。问题是 list 的类型是 `List<?>`，你不能在 `List<?>` 中放入除 null 以外的任何值。幸运的是，有一种方法可以实现，而无需求助于不安全的强制转换或原始类型。其思想是编写一个私有助手方法来捕获通配符类型。为了捕获类型，helper 方法必须是泛型方法。它看起来是这样的：

```Java
public static void swap(List<?> list, int i, int j) {
  swapHelper(list, i, j);
}
// Private helper method for wildcard capture
private static <E> void swapHelper(List<E> list, int i, int j) {
  list.set(i, list.set(j, list.get(i)));
}
```

The swapHelper method knows that list is a `List<E>`. Therefore, it knows that any value it gets out of this list is of type E and that it’s safe to put any value of type E into the list. This slightly convoluted implementation of swap compiles cleanly. It allows us to export the nice wildcard-based declaration, while taking advantage of the more complex generic method internally. Clients of the swap method don’t have to confront the more complex swapHelper declaration, but they do benefit from it. It is worth noting that the helper method has precisely the signature that we dismissed as too complex for the public method.

swapHelper 方法知道 list 是一个 `List<E>`。因此，它知道它从这个列表中得到的任何值都是 E 类型的，并且将 E 类型的任何值放入这个列表中都是安全的。这个稍微复杂的实现能够正确编译。它允许我们导出基于 通配符的声明，同时在内部利用更复杂的泛型方法。swap 方法的客户端不必面对更复杂的 swapHelper 声明，但它们确实从中受益。值得注意的是，helper 方法具有我们认为对于公共方法过于复杂而忽略的签名。

In summary, using wildcard types in your APIs, while tricky, makes the APIs far more flexible. If you write a library that will be widely used, the proper use of wildcard types should be considered mandatory. Remember the basic rule: producer-extends, consumer-super (PECS). Also remember that all comparables and comparators are consumers.

总之，在 API 中使用通配符类型虽然很棘手，但可以使其更加灵活。如果你编写的库将被广泛使用，则必须考虑通配符类型的正确使用。记住基本规则：生产者使用 extends，消费者使用 super（PECS）。还要记住，所有的 comparable 和 comparator 都是消费者。

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-5/Chapter-5-Introduction.md)**
- **Previous Item（上一条目）：[Item 30: Favor generic methods（优先使用泛型方法）](/Chapter-5/Chapter-5-Item-30-Favor-generic-methods.md)**
- **Next Item（下一条目）：[Item 32: Combine generics and varargs judiciously（明智地合用泛型和可变参数）](/Chapter-5/Chapter-5-Item-32-Combine-generics-and-varargs-judiciously.md)**
