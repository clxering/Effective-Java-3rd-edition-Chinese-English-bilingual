## Chapter 5. Generics（泛型）

### Item 30: Favor generic methods（优先使用泛型方法）

Just as classes can be generic, so can methods. Static utility methods that operate on parameterized types are usually generic. All of the “algorithm” methods in Collections (such as binarySearch and sort) are generic.

类可以是泛型的，方法也可以是泛型的。操作参数化类型的静态实用程序方法通常是泛型的。Collections 类中的所有「算法」方法（如 binarySearch 和 sort）都是泛型的。

Writing generic methods is similar to writing generic types. Consider this deficient method, which returns the union of two sets:

编写泛型方法类似于编写泛型类型。考虑这个有缺陷的方法，它返回两个集合的并集：

```Java
// Uses raw types - unacceptable! (Item 26)
public static Set union(Set s1, Set s2) {
    Set result = new HashSet(s1);
    result.addAll(s2);
    return result;
}
```

This method compiles but with two warnings:

该方法可进行编译，但有两个警告：

```Java
Union.java:5: warning: [unchecked] unchecked call to
HashSet(Collection<? extends E>) as a member of raw type HashSet
        Set result = new HashSet(s1);
                      ^

Union.java:6: warning: [
unchecked] unchecked call to
addAll(Collection<? extends E>) as a member of raw type Set
        result.addAll(s2);
                      ^
```

To fix these warnings and make the method typesafe, modify its declaration to declare a type parameter representing the element type for the three sets (the two arguments and the return value) and use this type parameter throughout the method. **The type parameter list, which declares the type parameters, goes between a method’s modifiers and its return type.** In this example, the type parameter list is `<E>`, and the return type is `Set<E>`. The naming conventions for type parameters are the same for generic methods and generic types (Items 29, 68):

要修复这些警告并使方法类型安全，请修改其声明，以声明表示三个集合（两个参数和返回值）的元素类型的类型参数，并在整个方法中使用该类型参数。类型参数列表声明类型参数，它位于方法的修饰符与其返回类型之间。在本例中，类型参数列表为 `<E>`，返回类型为 `Set<E>`。类型参数的命名约定与泛型方法和泛型类型的命名约定相同（[Item-29](/Chapter-5/Chapter-5-Item-29-Favor-generic-types.md)、[Item-68](/Chapter-9/Chapter-9-Item-68-Adhere-to-generally-accepted-naming-conventions.md)）:

```Java
// Generic method
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
    Set<E> result = new HashSet<>(s1);
    result.addAll(s2);
    return result;
}
```

At least for simple generic methods, that’s all there is to it. This method compiles without generating any warnings and provides type safety as well as ease of use. Here’s a simple program to exercise the method. This program contains no casts and compiles without errors or warnings:

至少对于简单的泛型方法，这就是（要注意细节的）全部。该方法编译时不生成任何警告，并且提供了类型安全性和易用性。这里有一个简单的程序来演示。这个程序不包含转换，编译时没有错误或警告：

```Java
// Simple program to exercise generic method
public static void main(String[] args) {
    Set<String> guys = Set.of("Tom", "Dick", "Harry");
    Set<String> stooges = Set.of("Larry", "Moe", "Curly");
    Set<String> aflCio = union(guys, stooges);
    System.out.println(aflCio);
}
```

When you run the program, it prints [Moe, Tom, Harry, Larry, Curly, Dick]. (The order of the elements in the output is implementation-dependent.)

当你运行程序时，它会打印出 [Moe, Tom, Harry, Larry, Curly, Dick]。（输出元素的顺序可能不同）。

A limitation of the union method is that the types of all three sets (both input parameters and the return value) have to be exactly the same. You can make the method more flexible by using bounded wildcard types (Item 31).

union 方法的一个限制是，所有三个集合（输入参数和返回值）的类型必须完全相同。你可以通过使用有界通配符类型（[Item-31](/Chapter-5/Chapter-5-Item-31-Use-bounded-wildcards-to-increase-API-flexibility.md)）使方法更加灵活。

On occasion, you will need to create an object that is immutable but applicable to many different types. Because generics are implemented by erasure (Item 28), you can use a single object for all required type parameterizations, but you need to write a static factory method to repeatedly dole out the object for each requested type parameterization. This pattern, called the generic singleton factory, is used for function objects (Item 42) such as Collections.reverseOrder, and occasionally for collections such as Collections.emptySet.

有时，你需要创建一个对象，该对象是不可变的，但适用于许多不同类型。因为泛型是由擦除（[Item-28](/Chapter-5/Chapter-5-Item-28-Prefer-lists-to-arrays.md)）实现的，所以你可以为所有需要的类型参数化使用单个对象，但是你需要编写一个静态工厂方法，为每个请求的类型参数化重复分配对象。这种模式称为泛型单例工厂，可用于函数对象（[Item-42](/Chapter-7/Chapter-7-Item-42-Prefer-lambdas-to-anonymous-classes.md)），如 Collections.reverseOrder，偶尔也用于集合，如 Collections.emptySet。

Suppose that you want to write an identity function dispenser. The libraries provide Function.identity, so there’s no reason to write your own (Item 59), but it is instructive. It would be wasteful to create a new identity function object time one is requested, because it’s stateless. If Java’s generics were reified, you would need one identity function per type, but since they’re erased a generic singleton will suffice. Here’s how it looks:

假设你想要编写一个恒等函数分发器。这些库提供 Function.identity，所以没有理由编写自己的库（[Item-59](/Chapter-9/Chapter-9-Item-59-Know-and-use-the-libraries.md)），但是它很有指导意义。在请求标识函数对象时创建一个新的标识函数对象是浪费时间的，因为它是无状态的。如果 Java 的泛型被具体化了，那么每个类型都需要一个标识函数，但是由于它们已经被擦除，一个泛型单例就足够了。它是这样的：

```Java
// Generic singleton factory pattern
private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

@SuppressWarnings("unchecked")
public static <T> UnaryOperator<T> identityFunction() {
    return (UnaryOperator<T>) IDENTITY_FN;
}
```

The cast of IDENTITY_FN to (`UnaryFunction<T>`) generates an unchecked cast warning, as `UnaryOperator<Object>` is not a `UnaryOperator<T>` for every T. But the identity function is special: it returns its argument unmodified, so we know that it is typesafe to use it as a `UnaryFunction<T>`, whatever the value of T. Therefore, we can confidently suppress the unchecked cast warning generated by this cast. Once we’ve done this, the code compiles without error or warning.

IDENTITY_FN 到（`UnaryFunction<T>`）的转换会生成一个 unchecked 转换警告，因为 `UnaryOperator<Object>` 并不是每个 T 都是 `UnaryOperator<T>`，但是恒等函数是特殊的：它会返回未修改的参数，所以我们知道，无论 T 的值是多少，都可以将其作为 `UnaryFunction<T>` 使用，这是类型安全的。一旦我们这样做了，代码编译就不会出现错误或警告。

Here is a sample program that uses our generic singleton as a `UnaryOperator<String>` and a `UnaryOperator<Number>`. As usual, it contains no casts and compiles without errors or warnings:

下面是一个示例程序，它使用我们的泛型单例作为 `UnaryOperator<String>` 和 `UnaryOperator<Number>`。像往常一样，它不包含类型转换和编译，没有错误或警告：

```Java
// Sample program to exercise generic singleton
public static void main(String[] args) {
    String[] strings = { "jute", "hemp", "nylon" };
    UnaryOperator<String> sameString = identityFunction();

    for (String s : strings)
        System.out.println(sameString.apply(s));

    Number[] numbers = { 1, 2.0, 3L };
    UnaryOperator<Number> sameNumber = identityFunction();

    for (Number n : numbers)
        System.out.println(sameNumber.apply(n));
}
```

It is permissible, though relatively rare, for a type parameter to be bounded by some expression involving that type parameter itself. This is what’s known as a recursive type bound. A common use of recursive type bounds is in connection with the Comparable interface, which defines a type’s natural ordering (Item 14). This interface is shown here:

允许类型参数被包含该类型参数本身的表达式限制，尽管这种情况比较少见。这就是所谓的递归类型限定。递归类型边界的一个常见用法是与 Comparable 接口相关联，后者定义了类型的自然顺序（[Item-14](/Chapter-3/Chapter-3-Item-14-Consider-implementing-Comparable.md)）。该界面如下图所示：

```Java
public interface Comparable<T> {
    int compareTo(T o);
}
```

The type parameter T defines the type to which elements of the type implementing `Comparable<T>` can be compared. In practice, nearly all types can be compared only to elements of their own type. So, for example, String implements `Comparable<String>`, Integer implements `Comparable<Integer>`, and so on.

类型参数 T 定义了实现 `Comparable<T>` 的类型的元素可以与之进行比较的类型。在实践中，几乎所有类型都只能与它们自己类型的元素进行比较。例如，String 实现 `Comparable<String>`， Integer 实现 `Comparable<Integer>`，等等。

Many methods take a collection of elements implementing Comparable to sort it, search within it, calculate its minimum or maximum, and the like. To do these things, it is required that every element in the collection be comparable to every other element in it, in other words, that the elements of the list be mutually comparable. Here is how to express that constraint:

许多方法采用实现 Comparable 的元素集合，在其中进行搜索，计算其最小值或最大值，等等。要做到这些，需要集合中的每个元素与集合中的每个其他元素相比较，换句话说，就是列表中的元素相互比较。下面是如何表达这种约束（的示例）：

```Java
// Using a recursive type bound to express mutual comparability
public static <E extends Comparable<E>> E max(Collection<E> c);
```

The type bound `<E extends Comparable<E>>` may be read as “any type E that can be compared to itself,” which corresponds more or less precisely to the notion of mutual comparability.

类型限定 `<E extends Comparable<E>>` 可以被理解为「可以与自身进行比较的任何类型 E」，这或多或少与相互可比性的概念相对应。

Here is a method to go with the previous declaration. It calculates the maximum value in a collection according to its elements’ natural order, and it compiles without errors or warnings:

下面是一个与前面声明相同的方法。它根据元素的自然顺序计算集合中的最大值，编译时没有错误或警告：

```Java
// Returns max value in a collection - uses recursive type bound
public static <E extends Comparable<E>> E max(Collection<E> c) {
    if (c.isEmpty())
        throw new IllegalArgumentException("Empty collection");

    E result = null;

    for (E e : c)
        if (result == null || e.compareTo(result) > 0)

    result = Objects.requireNonNull(e);
    return result;
}
```

Note that this method throws IllegalArgumentException if the list is empty. A better alternative would be to return an `Optional<E>` (Item 55).

注意，如果列表为空，该方法将抛出 IllegalArgumentException。更好的选择是返回一个 `Optional<E>`（[Item-55](/Chapter-8/Chapter-8-Item-55-Return-optionals-judiciously.md)）。

Recursive type bounds can get much more complex, but luckily they rarely do. If you understand this idiom, its wildcard variant (Item 31), and the simulated self-type idiom (Item 2), you’ll be able to deal with most of the recursive type bounds you encounter in practice.

递归类型限定可能会变得复杂得多，但幸运的是，这种情况很少。如果你理解这个习惯用法、它的通配符变量（[Item-31](/Chapter-5/Chapter-5-Item-31-Use-bounded-wildcards-to-increase-API-flexibility.md)）和模拟的自类型习惯用法（[Item-2](/Chapter-2/Chapter-2-Item-2-Consider-a-builder-when-faced-with-many-constructor-parameters.md)），你就能够处理在实践中遇到的大多数递归类型限定。

In summary, generic methods, like generic types, are safer and easier to use than methods requiring their clients to put explicit casts on input parameters and return values. Like types, you should make sure that your methods can be used without casts, which often means making them generic. And like types, you should generify existing methods whose use requires casts. This makes life easier for new users without breaking existing clients (Item 26).

总之，与要求客户端对输入参数和返回值进行显式转换的方法相比，泛型方法与泛型一样，更安全、更容易使用。与类型一样，你应该确保你的方法可以在不使用类型转换的情况下使用，这通常意味着要使它们具有通用性。与类型类似，你应该将需要强制类型转换的现有方法泛型化。这使得新用户在不破坏现有客户端的情况下更容易使用（[Item-26](/Chapter-5/Chapter-5-Item-26-Do-not-use-raw-types.md)）。

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-5/Chapter-5-Introduction.md)**
- **Previous Item（上一条目）：[Item 29: Favor generic types（优先使用泛型）](/Chapter-5/Chapter-5-Item-29-Favor-generic-types.md)**
- **Next Item（下一条目）：[Item 31: Use bounded wildcards to increase API flexibility（使用有界通配符增加 API 的灵活性）](/Chapter-5/Chapter-5-Item-31-Use-bounded-wildcards-to-increase-API-flexibility.md)**
