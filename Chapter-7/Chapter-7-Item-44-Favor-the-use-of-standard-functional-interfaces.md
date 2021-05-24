## Chapter 7. Lambdas and Streams（λ 表达式和流）

### Item 44: Favor the use of standard functional interfaces（优先使用标准函数式接口）

Now that Java has lambdas, best practices for writing APIs have changed considerably. For example, the Template Method pattern [Gamma95], wherein a subclass overrides a primitive method to specialize the behavior of its superclass, is far less attractive. The modern alternative is to provide a static factory or constructor that accepts a function object to achieve the same effect. More generally, you’ll be writing more constructors and methods that take function objects as parameters. Choosing the right functional parameter type demands care.

现在 Java 已经有了 lambda 表达式，编写 API 的最佳实践已经发生了很大的变化。例如，模板方法模式 [Gamma95]，其中子类覆盖基类方法以专门化其超类的行为，就没有那么有吸引力了。现代的替代方法是提供一个静态工厂或构造函数，它接受一个函数对象来实现相同的效果。更一般地，你将编写更多以函数对象为参数的构造函数和方法。选择正确的函数参数类型需要谨慎。

Consider LinkedHashMap. You can use this class as a cache by overriding its protected removeEldestEntry method, which is invoked by put each time a new key is added to the map. When this method returns true, the map removes its eldest entry, which is passed to the method. The following override allows the map to grow to one hundred entries and then deletes the eldest entry each time a new key is added, maintaining the hundred most recent entries:

考虑 LinkedHashMap。你可以通过覆盖受保护的 removeEldestEntry 方法将该类用作缓存，每当向映射添加新键时，put 都会调用该方法。当该方法返回 true 时，映射将删除传递给该方法的最老条目。下面的覆盖允许映射增长到 100 个条目，然后在每次添加新键时删除最老的条目，维护 100 个最近的条目：

```Java
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return size() > 100;
}
```

This technique works fine, but you can do much better with lambdas. If LinkedHashMap were written today, it would have a static factory or constructor that took a function object. Looking at the declaration for removeEldestEntry, you might think that the function object should take a `Map.Entry<K,V>` and return a boolean, but that wouldn’t quite do it: The removeEldestEntry method calls size() to get the number of entries in the map, which works because removeEldestEntry is an instance method on the map. The function object that you pass to the constructor is not an instance method on the map and can’t capture it because the map doesn’t exist yet when its factory or constructor is invoked. Thus, the map must pass itself to the function object, which must therefore take the map on input as well as its eldest entry. If you were to declare such a functional interface, it would look something like this:

这种技术工作得很好，但是使用 lambda 表达式可以做得更好。如果 LinkedHashMap 是现在编写的，它将有一个静态工厂或构造函数，它接受一个函数对象。看着 removeEldestEntry 的定义,你可能会认为这个函数对象应该 `Map.Entry<K,V>` 和返回一个布尔值，但不会完全做到：removeEldestEntry 方法调用 `size()` 地图中的条目的数量，这工作，因为 removeEldestEntry 在 Map 上是一个实例方法。传递给构造函数的函数对象不是 Map 上的实例方法，无法捕获它，因为在调用 Map 的工厂或构造函数时，Map 还不存在。因此，Map 必须将自身传递给函数对象，函数对象因此必须在输入端及其最老的条目上接受 Map。如果要声明这样一个函数式接口，它看起来是这样的：

```Java
// Unnecessary functional interface; use a standard one instead.
@FunctionalInterface interface EldestEntryRemovalFunction<K,V>{
    boolean remove(Map<K,V> map, Map.Entry<K,V> eldest);
}
```

This interface would work fine, but you shouldn’t use it, because you don’t need to declare a new interface for this purpose. The java.util.function package provides a large collection of standard functional interfaces for your use. **If one of the standard functional interfaces does the job, you should generally use it in preference to a purpose-built functional interface.** This will make your API easier to learn, by reducing its conceptual surface area, and will provide significant interoperability benefits, as many of the standard functional interfaces provide useful default methods. The Predicate interface, for instance, provides methods to combine predicates. In the case of our LinkedHashMap example, the standard `BiPredicate<Map<K,V>`, `Map.Entry<K,V>>` interface should be used in preference to a custom EldestEntryRemovalFunction interface.

这个接口可以很好地工作，但是你不应该使用它，因为你不需要为此声明一个新接口。`java.util.function` 包提供了大量的标准函数接口供你使用。**如果一个标准的函数式接口可以完成这项工作，那么你通常应该优先使用它，而不是使用专门构建的函数式接口。** 通过减少 API 的概念表面积，这将使你的 API 更容易学习，并将提供显著的互操作性优势，因为许多标准函数式接口提供了有用的默认方法。例如，Predicate 接口提供了组合谓词的方法。在我们的 LinkedHashMap 示例中，应该优先使用标准的 `BiPredicate<Map<K,V>`、`Map.Entry<K,V>>` 接口，而不是定制的 EldestEntryRemovalFunction 接口。

There are forty-three interfaces in java.util.Function. You can’t be expected to remember them all, but if you remember six basic interfaces, you can derive the rest when you need them. The basic interfaces operate on object reference types. The Operator interfaces represent functions whose result and argument types are the same. The Predicate interface represents a function that takes an argument and returns a boolean. The Function interface represents a function whose argument and return types differ. The Supplier interface represents a function that takes no arguments and returns (or “supplies”) a value. Finally, Consumer represents a function that takes an argument and returns nothing, essentially consuming its argument. The six basic functional interfaces are summarized below:

**译注：原文笔误，应为 `java.util.function`**

`java.util.function` 中有 43 个接口。不能期望你记住所有的接口，但是如果你记住了 6 个基本接口，那么你可以在需要时派生出其余的接口。基本接口操作对象引用类型。Operator 接口表示结果和参数类型相同的函数。Predicate 接口表示接受参数并返回布尔值的函数。Function 接口表示参数和返回类型不同的函数。Supplier 接口表示一个不接受参数并返回（或「供应」）值的函数。最后，Consumer 表示一个函数，该函数接受一个参数，但不返回任何内容，本质上是使用它的参数。六个基本的函数式接口总结如下：

|    Interface    |       Function Signature       |      Example     |
|:-------:|:-------:|:-------:|
|   `UnaryOperator<T>`  |     `T apply(T t)`    |   `String::toLowerCase`   |
|   `BinaryOperator<T>`  |     `T apply(T t1, T t2)`    |   `BigInteger::add`   |
|   `Predicate<T>`  |     `boolean test(T t)`    |   `Collection::isEmpty`   |
|   `Function<T,R>`  |     `R apply(T t)`    |   `Arrays::asList`   |
|   `Supplier<T>`  |     `T get()`    |   `Instant::now`   |
|   `Consumer<T>`  |     `void accept(T t)`    |   `System.out::println`   |

There are also three variants of each of the six basic interfaces to operate on the primitive types int, long, and double. Their names are derived from the basic interfaces by prefixing them with a primitive type. So, for example, a predicate that takes an int is an IntPredicate, and a binary operator that takes two long values and returns a long is a LongBinaryOperator. None of these variant types is parameterized except for the Function variants, which are parameterized by return type. For example, `LongFunction<int[]>` takes a long and returns an int[].

还有 6 个基本接口的 3 个变体，用于操作基本类型 int、long 和 double。它们的名称是通过在基本接口前面加上基本类型前缀而派生出来的。例如，一个接受 int 的 Predicate 就是一个 IntPredicate，一个接受两个 long 值并返回一个 long 的二元操作符就是一个 LongBinaryOperator。除了由返回类型参数化的函数变量外，这些变量类型都不是参数化的。例如，`LongFunction<int[]>` 使用 long 并返回一个 int[]。

There are nine additional variants of the Function interface, for use when the result type is primitive. The source and result types always differ, because a function from a type to itself is a UnaryOperator. If both the source and result types are primitive, prefix Function with SrcToResult, for example LongToIntFunction (six variants). If the source is a primitive and the result is an object reference, prefix Function with `<Src>ToObj`, for example DoubleToObjFunction (three variants).

Function 接口还有 9 个额外的变体，在结果类型为基本数据类型时使用。源类型和结果类型总是不同的，因为不同类型的函数本身都是 UnaryOperator。如果源类型和结果类型都是基本数据类型，则使用带有 SrcToResult 的前缀函数，例如 LongToIntFunction（六个变体）。如果源是一个基本数据类型，而结果是一个对象引用，则使用带前缀 `<Src>ToObj` 的 Function 接口，例如 DoubleToObjFunction（三个变体）。

There are two-argument versions of the three basic functional interfaces for which it makes sense to have them: `BiPredicate<T,U>`, `BiFunction<T,U,R>`, and `BiConsumer<T,U>`. There are also BiFunction variants returning the three relevant primitive types: `ToIntBiFunction<T,U>`, `ToLongBiFunction<T,U>`, and `ToDoubleBiFunction<T,U>`. There are two-argument variants of Consumer that take one object reference and one primitive type: `ObjDoubleConsumer<T>`, `ObjIntConsumer<T>`, and `ObjLongConsumer<T>`. In total, there are nine two-argument versions of the basic interfaces.

三个基本函数式接口有两个参数版本，使用它们是有意义的：`BiPredicate<T,U>`、`BiFunction<T,U,R>`、`BiConsumer<T,U>`。也有 BiFunction 变体返回三个相关的基本类型：`ToIntBiFunction<T,U>`、 `ToLongBiFunction<T,U>`、`ToDoubleBiFunction<T,U>`。Consumer 有两个参数变体，它们接受一个对象引用和一个基本类型：`ObjDoubleConsumer<T>`、`ObjIntConsumer<T>`、`ObjLongConsumer<T>`。总共有9个基本接口的双参数版本。

Finally, there is the BooleanSupplier interface, a variant of Supplier that returns boolean values. This is the only explicit mention of the boolean type in any of the standard functional interface names, but boolean return values are supported via Predicate and its four variant forms. The BooleanSupplier interface and the forty-two interfaces described in the previous paragraphs account for all forty-three standard functional interfaces. Admittedly, this is a lot to swallow, and not terribly orthogonal. On the other hand, the bulk of the functional interfaces that you’ll need have been written for you and their names are regular enough that you shouldn’t have too much trouble coming up with one when you need it.

最后是 BooleanSupplier 接口，它是 Supplier 的一个变体，返回布尔值。这是在任何标准函数接口名称中唯一显式提到布尔类型的地方，但是通过 Predicate 及其四种变体形式支持布尔返回值。前面描述的 BooleanSupplier 接口和 42 个接口占了全部 43 个标准函数式接口。不可否认，这有很多东西需要消化，而且不是非常直观。另一方面，你将需要的大部分函数式接口都是为你编写的，并且它们的名称足够常规，因此在需要时你应该不会遇到太多麻烦。

Most of the standard functional interfaces exist only to provide support for primitive types. **Don’t be tempted to use basic functional interfaces with boxed primitives instead of primitive functional interfaces.** While it works, it violates the advice of Item 61, “prefer primitive types to boxed primitives.” The performance consequences of using boxed primitives for bulk operations can be deadly.

大多数标准函数式接口的存在只是为了提供对基本类型的支持。**不要尝试使用带有包装类的基本函数式接口，而不是使用基本类型函数式接口。** 当它工作时，它违反了 [Item-61](/Chapter-9/Chapter-9-Item-61-Prefer-primitive-types-to-boxed-primitives.md) 的建议，“与盒装原语相比，更喜欢原语类型”。在批量操作中使用装箱原语的性能后果可能是致命的。

Now you know that you should typically use standard functional interfaces in preference to writing your own. But when should you write your own? Of course you need to write your own if none of the standard ones does what you need, for example if you require a predicate that takes three parameters, or one that throws a checked exception. But there are times you should write your own functional interface even when one of the standard ones is structurally identical.

现在你知道，与编写自己的接口相比，通常应该使用标准的函数式接口。但是你应该什么时候写你自己的呢？当然，如果标准的函数式接口都不能满足你的需要，那么你需要自行编写，例如，如果你需要一个接受三个参数的 Predicate，或者一个抛出已检查异常的 Predicate。但是有时候你应该编写自己的函数接口，即使其中一个标准接口在结构上是相同的。

Consider our old friend `Comparator<T>`, which is structurally identical to the `ToIntBiFunction<T,T>` interface. Even if the latter interface had existed when the former was added to the libraries, it would have been wrong to use it. There are several reasons that Comparator deserves its own interface. First, its name provides excellent documentation every time it is used in an API, and it’s used a lot. Second, the Comparator interface has strong requirements on what constitutes a valid instance, which comprise its general contract. By implementing the interface, you are pledging to adhere to its contract. Third, the interface is heavily outfitted with useful default methods to transform and combine comparators.

考虑我们的老朋友 `Comparator<T>`，它在结构上与 `ToIntBiFunction<T,T>` 接口相同。即使后者接口在将前者添加到库时已经存在，使用它也是错误的。有几个原因说明比较器应该有自己的接口。首先，每次在 API 中使用 Comparator 时，它的名称都提供了优秀的文档，而且它的使用非常频繁。通过实现接口，你保证遵守其契约。第三，该接口大量配备了用于转换和组合比较器的有用默认方法。

You should seriously consider writing a purpose-built functional interface in preference to using a standard one if you need a functional interface that shares one or more of the following characteristics with Comparator:

如果你需要与 Comparator 共享以下一个或多个特性的函数式接口，那么你应该认真考虑编写一个专用的函数式接口，而不是使用标准接口：

- It will be commonly used and could benefit from a descriptive name.

它将被广泛使用，并且可以从描述性名称中获益。

- It has a strong contract associated with it.

它有一个强有力的约定。

- It would benefit from custom default methods.

它将受益于自定义默认方法。

If you elect to write your own functional interface, remember that it’s an interface and hence should be designed with great care (Item 21).

如果你选择编写自己的函数式接口，请记住这是一个接口，因此应该非常小心地设计它（[Item-21](/Chapter-4/Chapter-4-Item-21-Design-interfaces-for-posterity.md)）。

Notice that the EldestEntryRemovalFunction interface (page 199) is labeled with the @FunctionalInterface annotation. This annotation type is similar in spirit to @Override. It is a statement of programmer intent that serves three purposes: it tells readers of the class and its documentation that the interface was designed to enable lambdas; it keeps you honest because the interface won’t compile unless it has exactly one abstract method; and it prevents maintainers from accidentally adding abstract methods to the interface as it evolves. **Always annotate your functional interfaces with the @FunctionalInterface annotation.**

注意 EldestEntryRemovalFunction 接口(第199页)使用 `@FunctionalInterface` 注释进行标记。这种注释类型在本质上类似于 `@Override`。它是程序员意图的声明，有三个目的：它告诉类及其文档的读者，接口的设计是为了启用 lambda 表达式；它使你保持诚实，因为接口不会编译，除非它只有一个抽象方法；它还可以防止维护者在接口发展过程中意外地向接口添加抽象方法。**总是用 `@FunctionalInterface` 注释你的函数接口。**

A final point should be made concerning the use of functional interfaces in APIs. Do not provide a method with multiple overloadings that take different functional interfaces in the same argument position if it could create a possible ambiguity in the client. This is not just a theoretical problem. The submit method of ExecutorService can take either a `Callable<T>` or a Runnable, and it is possible to write a client program that requires a cast to indicate the correct overloading (Item 52). The easiest way to avoid this problem is not to write overloadings that take different functional interfaces in the same argument position. This is a special case of the advice in Item 52, “use overloading judiciously.”

最后一点应该是关于 API 中函数式接口的使用。不要提供具有多个重载的方法，这些方法采用相同参数位置的不同函数式接口，否则会在客户机中造成可能的歧义。这不仅仅是一个理论问题。ExecutorService 的 submit 方法可以是 `Callable<T>` 级的，也可以是 Runnable 的，并且可以编写一个客户端程序，它需要一个类型转换来指示正确的重载(Item 52)。避免此问题的最简单方法是不要编写将不同函数式接口放在相同参数位置的重载。这是 [Item-52](/Chapter-8/Chapter-8-Item-52-Use-overloading-judiciously.md) 「明智地使用过载」建议的一个特例。

In summary, now that Java has lambdas, it is imperative that you design your APIs with lambdas in mind. Accept functional interface types on input and return them on output. It is generally best to use the standard interfaces provided in java.util.function.Function, but keep your eyes open for the relatively rare cases where you would be better off writing your own functional interface.

总之，既然 Java 已经有了 lambda 表达式，你必须在设计 API 时考虑 lambda 表达式。在输入时接受函数式接口类型，在输出时返回它们。一般情况下，最好使用 `java.util.function` 中提供的标准函数式接口，但请注意比较少见的一些情况，在这种情况下，你最好编写自己的函数式接口。

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-7/Chapter-7-Introduction.md)**
- **Previous Item（上一条目）：[Item 43: Prefer method references to lambdas（方法引用优于 λ 表达式）](/Chapter-7/Chapter-7-Item-43-Prefer-method-references-to-lambdas.md)**
- **Next Item（下一条目）：[Item 45: Use streams judiciously（明智地使用流）](/Chapter-7/Chapter-7-Item-45-Use-streams-judiciously.md)**
