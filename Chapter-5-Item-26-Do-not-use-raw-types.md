## Chapter 5. Generics（泛型）

### Item 26: Don’t use raw types（不要使用原始类型）

First, a few terms. A class or interface whose declaration has one or more type parameters is a generic class or interface [JLS, 8.1.2, 9.1.2]. For example, the List interface has a single type parameter, E, representing its element type. The full name of the interface is List<E> (read “list of E”), but people often call it List for short. Generic classes and interfaces are collectively known as generic types.

首先（介绍）一些术语。声明中具有一个或多个类型参数的类或接口就是泛型类或泛型接口[JLS, 8.1.2, 9.1.2]。例如，List接口有一个类型参数E，用于表示其元素类型。该接口的全名是List<E>（读作“List of E”），但人们通常简称为List。泛型类和泛型接口统称为泛型。

Each generic type defines a set of parameterized types, which consist of the class or interface name followed by an angle-bracketed list of actual type parameters corresponding to the generic type’s formal type parameters [JLS, 4.4, 4.5]. For example, List<String> (read “list of string”) is a parameterized type representing a list whose elements are of type String. (String is the actual type parameter corresponding to the formal type parameter E.)

每个泛型都定义了一组参数化类型，该类型为类或接口名后由尖括号括起来的对应于泛型形式类型参数的实际类型参数列表组成[JLS, 4.4, 4.5]。例如，List<String>（读作“List of String”）是一个参数化类型，表示元素类型为String类型的List。(String是与形式类型参数E对应的实际类型参数。)

Finally, each generic type defines a raw type, which is the name of the generic type used without any accompanying type parameters [JLS, 4.8]. For example, the raw type corresponding to List<E> is List. Raw types behave as if all of the generic type information were erased from the type declaration. They exist primarily for compatibility with pre-generics code.

最后，每个泛型类型都定义了一个原始类型，它是没有任何相关类型参数的泛型类型的名称[JLS, 4.8]。例如，List<E>对应的原始类型是List。原始类型的行为就好像所有泛型类型信息都从类型声明中删除了一样。它们的存在主要是为了与之前的泛型代码兼容。

Before generics were added to Java, this would have been an exemplary collection declaration. As of Java 9, it is still legal, but far from exemplary:

在将泛型添加到Java之前，这是一个典型的集合声明。就Java 9而言，它仍然是合法的，但（不应效仿）：

```
// Raw collection type - don't do this!
// My stamp collection. Contains only Stamp instances.
private final Collection stamps = ... ;
```

If you use this declaration today and then accidentally put a coin into your stamp collection, the erroneous insertion compiles and runs without error (though the compiler does emit a vague warning):

如果你今天使用这个声明，然后意外地将coin放入stamp集合中，这一错误的插入（依然能够）编译并没有错误地运行（尽管编译器确实发出了模糊的警告）：

```
// Erroneous insertion of coin into stamp collection
stamps.add(new Coin( ... )); // Emits "unchecked call" warning
```

You don’t get an error until you try to retrieve the coin from the stamp collection:

直到从stamp集合中获取coin时才会收到错误提示：

```
// Raw iterator type - don't do this!
for (Iterator i = stamps.iterator(); i.hasNext(); )
Stamp stamp = (Stamp) i.next(); // Throws ClassCastException
stamp.cancel();
```

As mentioned throughout this book, it pays to discover errors as soon as possible after they are made, ideally at compile time. In this case, you don’t discover the error until runtime, long after it has happened, and in code that may be distant from the code containing the error. Once you see the ClassCastException, you have to search through the codebase looking for the method invocation that put the coin into the stamp collection. The compiler can’t help you, because it can’t understand the comment that says, “Contains only Stamp instances.”

正如在本书中提到的，在出现错误之后尽快发现错误是（有价值）的，最好是在编译时。在（本例）这种情况下，直到运行时（在错误发生很久之后）才发现错误，而且（报错）代码可能与包含错误的代码相距很远。一旦看到ClassCastException，就必须在代码中搜索将coin放进stamp集合的方法调用。编译器不能帮助您，因为它不能理解注释“Contains only Stamp instances.”。

With generics, the type declaration contains the information, not the comment:

对于泛型，类型声明包含（类型）信息，而不是注释:

```
// Parameterized collection type - typesafe
private final Collection<Stamp> stamps = ... ;
```

From this declaration, the compiler knows that stamps should contain only Stamp instances and guarantees it to be true, assuming your entire codebase compiles without emitting (or suppressing; see Item 27) any warnings. When stamps is declared with a parameterized type declaration, the erroneous insertion generates a compile-time error message that tells you exactly what is wrong:

从这个声明中，编译器知道stamps应该只包含Stamp实例，并保证它是真实的，假设您的整个代码库编译没有发出（或禁止；详见[Item-27](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-5-Item-27-Eliminate-unchecked-warnings.md)）任何警告。当stamps利用一个参数化的类型进行声明时，错误的插入将生成编译时错误消息，该消息将确切地告诉您哪里出了问题:

```
Test.java:9: error: incompatible types: Coin cannot be converted
to Stamp
c.add(new Coin());
^
```

The compiler inserts invisible casts for you when retrieving（v. [计] 检索，retrieve的ing形式；恢复；取回；补偿） elements from collections and guarantees that they won’t fail (assuming, again, that all of your code did not generate or suppress any compiler warnings). While the prospect of accidentally inserting a coin into a stamp collection may appear far-fetched, the problem is real. For example, it is easy to imagine putting a BigInteger into a collection that is supposed to contain only BigDecimal instances.

当从集合中检索元素时，编译器会为你（执行）不可见的强制类型转换，并确保它们不会失败（再次假设您的所有代码没有产生或禁止任何编译器警告）。虽然不小心将coin插入stamp集合看起来有些牵强，但问题是真的。例如，很容易想象将一个BigInteger放入一个只包含BigDecimal实例的集合中。

As noted earlier, it is legal to use raw types (generic types without their type parameters), but you should never do it. **If you use raw types, you lose all the safety and expressiveness benefits of generics.** Given that you shouldn’t use them, why did the language designers permit raw types in the first place? For compatibility. Java was about to enter its second decade when generics were added, and there was an enormous amount of code in existence that did not use generics. It was deemed critical that all of this code remain legal and interoperate with newer code that does use generics. It had to be legal to pass instances of parameterized types to methods that were designed for use with raw types, and vice versa. This requirement, known as migration compatibility, drove the decisions to support raw types and to implement generics using erasure (Item 28).

如前所述，使用原始类型（没有类型参数的泛型类型）是合法的，但是你永远不应该这样做。**如果使用原始类型，就会失去泛型的安全性和表现力。** 既然您不应该使用它们，那么为什么语言设计者一开始就允许原始类型呢？（答案是）为了兼容性。Java即将进入第二个十年，泛型被添加进来（时），（还）存在大量不使用泛型的代码。保持所有这些代码合法并与使用泛型的新代码（兼容）被认为是关键的。将参数化类型的实例传递给设计用于原始类型的方法必须是合法的，反之亦然。这种被称为迁移兼容性的需求促使（了）支持原始类型并使用擦除实现泛型（[Item-28](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-5-Item-28-Prefer-lists-to-arrays.md)）。

While you shouldn’t use raw types such as List, it is fine to use types that are parameterized to allow insertion of arbitrary objects, such as List<Object>. Just what is the difference between the raw type List and the parameterized type List<Object>? Loosely speaking, the former has opted out of the generic type system, while the latter has explicitly told the compiler that it is capable of holding objects of any type. While you can pass a List<String> to a parameter of type List, you can’t pass it to a parameter of type List<Object>. There are sub-typing rules for generics, and List<String> is a subtype of the raw type List, but not of the parameterized type List<Object> (Item 28). As a consequence, **you lose type safety if you use a raw type such as List, but not if you use a parameterized type such as List&lt;Object&gt;.**

虽然您不应该使用原始类型（如List），但是可以使用参数化的类型来允许插入任意对象（如List<Object>）。原始类型列表和参数化类型List<Object>之间的区别是什么？粗略地说，前者选择了不使用泛型类型系统，而后者明确地告诉编译器它能够保存任何类型的对象。虽然可以将List<String>传递给类型列表的参数，但不能将其传递给类型List<Object>的参数。泛型有子类型规则，List<String>是原始类型列表的子类型，而不是参数化类型列表<object>（[Item-28](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-5-Item-28-Prefer-lists-to-arrays.md)）。因此，**如果使用原始类型（如List），就会失去类型安全性，但如果使用参数化类型（如List&lt;Object&gt;）则不会。**

To make this concrete, consider the following program:

为了使这一点具体化，考虑下面的程序：

```
// Fails at runtime - unsafeAdd method uses a raw type (List)!

public static void main(String[] args) {
  List<String> strings = new ArrayList<>();
  unsafeAdd(strings, Integer.valueOf(42));
  String s = strings.get(0); // Has compiler-generated cast
}

private static void unsafeAdd(List list, Object o) {
  list.add(o);
}
```

This program compiles, but because it uses the raw type List, you get a warning:

该程序将编译，但因为它使用原始类型List，所以您会得到一个警告:

```
Test.java:10: warning: [unchecked] unchecked call to add(E) as a
member of the raw type List
list.add(o);
^
```

And indeed, if you run the program, you get a ClassCastException when the program tries to cast the result of the invocation strings.get(0), which is an Integer, to a String. This is a compiler-generated cast, so it’s normally guaranteed to succeed, but in this case we ignored a compiler warning and paid the price.

实际上，如果您运行程序，当程序试图将调用strings.get(0)的结果转换为字符串时，您会得到一个ClassCastException。这是一个编译器生成的强制类型转换，因此它通常保证会成功，但在本例中，我们忽略了编译器的警告，并为此付出了代价。

If you replace the raw type List with the parameterized type List<Object> in the unsafeAdd declaration and try to recompile the program, you’ll find that it no longer compiles but emits the error message:

如果将unsafeAdd声明中的原始类型List替换为参数化类型List<Object>，并尝试重新编译程序，您会发现它不再编译，而是发出错误消息：

```
Test.java:5: error: incompatible types: List<String> cannot be
converted to List<Object>
unsafeAdd(strings, Integer.valueOf(42));
^
```

You might be tempted to use a raw type for a collection whose element type is unknown and doesn’t matter. For example, suppose you want to write a method that takes two sets and returns the number of elements they have in common. Here’s how you might write such a method if you were new to generics:

对于元素类型未知且无关紧要的集合，您可能会尝试使用原始类型。例如，假设您希望编写一个方法，该方法接受两个集合并返回它们共有的元素数量。如果您是（使用）泛型（的）新手，那么您可以这样编写一个方法:

```
// Use of raw type for unknown element type - don't do this!
static int numElementsInCommon(Set s1, Set s2) {
  int result = 0;
  for (Object o1 : s1)
    if (s2.contains(o1))
  result++;
  return result;
}
```

This method works but it uses raw types, which are dangerous. The safe alternative is to use unbounded wildcard types. If you want to use a generic type but you don’t know or care what the actual type parameter is, you can use a question mark instead. For example, the unbounded wildcard type for the generic type Set<E> is Set<?> (read “set of some type”). It is the most general parameterized Set type, capable of holding any set. Here is how the numElementsInCommon declaration looks with unbounded wildcard types:

这种方法是可行的，但是它使用的是原始类型，这是很危险的。安全的替代方法是使用无界通配符类型。如果您想使用泛型，但不知道或不关心实际的类型参数是什么，那么可以使用问号代替。例如，泛型类型集Set<E>的无界通配符类型是Set<?>(读作“set of some type”)。它是最通用的参数化集合类型，能够容纳任何集合：

```
// Uses unbounded wildcard type - typesafe and flexible
static int numElementsInCommon(Set<?> s1, Set<?> s2) { ... }
```

What is the difference between the unbounded wildcard type Set<?> and the raw type Set? Does the question mark really buy you anything? Not to belabor the point, but the wildcard type is safe and the raw type isn’t. You can put any element into a collection with a raw type, easily corrupting the collection’s type invariant (as demonstrated by the unsafeAdd method on page 119); you can’t put any element (other than null) into a Collection<?>. Attempting to do so will generate a compile-time error message like this:

无界通配符类型Set<?>和原始类型Set之间的区别是什么？问号真的能（起作用吗）？我并不是在强调这一点，但是通配符类型是安全的，而原始类型则不是。可以将任何元素放入具有原始类型的集合中，很容易破坏集合的类型不变量（如上述的unsafeAdd方法所示）；您不能将任何元素（除了null）放入Collection<?>。尝试这样做将生成这样的编译时错误消息:

```
WildCard.java:13: error: incompatible types: String cannot be
converted to CAP#1
c.add("verboten");
^ where CAP#1
is a
fresh type-variable:
CAP#1 extends Object from capture of ?
```

Admittedly this error message leaves something to be desired, but the compiler has done its job, preventing you from corrupting（vt. 使腐烂；使堕落，使恶化） the collection’s type invariant（n. [数] 不变量；[计] 不变式）, whatever its element type may be. Not only can’t you put any element (other than null) into a Collection<?>, but you can’t assume anything about the type of the objects that you get out. If these restrictions are unacceptable, you can use generic methods (Item 30) or bounded wildcard types (Item 31).

无可否认，这个错误消息让人不满意，但是编译器已经完成了它的工作，防止您破坏集合的类型（约束），不管它的元素类型是什么。您不仅不能将任何元素（除null之外）放入Collection<?>，（而且）不能假设你得到的对象的类型。如果这些限制是不可接受的，您可以使用泛型方法（[Item-30](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-5-Item-30-Favor-generic-methods.md)）或有界通配符类型（[Item-31](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-5-Item-31-Use-bounded-wildcards-to-increase-API-flexibility.md)）。

There are a few minor exceptions to the rule that you should not use raw types. **You must use raw types in class literals.** The specification does not permit the use of parameterized types (though it does permit array types and primitive types) [JLS, 15.8.2]. In other words, List.class, String[].class, and int.class are all legal, but List<String>.class and List<?>.class are not.

对于不应该使用原始类型的规则，有一些小的例外。**必须在类文字中使用原始类型。** 该规范不允许使用参数化类型（尽管它允许数组类型和基本类型）[JLS, 15.8.2]。换句话说，List.class，String[].class和int.class都是合法的，但是List<String>.class和List<?>.class不是。

A second exception to the rule concerns the instanceof operator. Because generic type information is erased at runtime, it is illegal to use the instanceof operator on parameterized types other than unbounded wildcard types. The use of unbounded wildcard types in place of raw types does not affect the behavior of the instanceof operator in any way. In this case, the angle brackets and question marks are just noise. **This is the preferred way to use the instanceof operator with generic types:**

规则的第二个例外是instanceof运算符。由于泛型类型信息在运行时被删除，因此在非无界通配符类型之外的参数化类型上使用instanceof操作符是非法的。使用无界通配符类型代替原始类型不会以任何方式影响instanceof运算符的行为。在这种情况下，尖括号和问号只是（多余的）。**（下面的例子）是使用通用类型instanceof运算符的首选方法:**

```
// Legitimate use of raw type - instanceof operator
if (o instanceof Set) { // Raw type
Set<?> s = (Set<?>) o; // Wildcard type
...
}
```

Note that once you’ve determined that o is a Set, you must cast it to the wildcard type Set<?>, not the raw type Set. This is a checked cast, so it will not cause a compiler warning.

注意，一旦确定o是一个Set，就必须将其转换为通配符类型Set<?>，而不是原始类型Set。这是一个（经过检查的）强制类型转换，所以不会引起编译器警告。

In summary, using raw types can lead to exceptions at runtime, so don’t use them. They are provided only for compatibility and interoperability with legacy code that predates the introduction of generics. As a quick review, Set<Object> is a parameterized type representing a set that can contain objects of any type, Set<?> is a wildcard（n. 通配符） type representing（v. 代表；表示，表现） a set that can contain only objects of some unknown type, and Set is a raw type, which opts out of the generic type system. The first two are safe, and the last is not.

总之，使用原始类型可能会在运行时导致异常，所以不要（轻易）使用它们。它们仅用于与引入泛型之前的遗留代码进行兼容和互操作。快速回顾一下，Set<Object>是一个参数化类型，表示可以包含任何类型的对象的集合，Set<?>是一个通配符类型，表示只能包含某种未知类型的对象的集合，Set是一个原始类型，它选择了泛型类型系统。前两个是安全的，后一个就不安全了。

For quick reference, the terms introduced in this item (and a few introduced later in this chapter) are summarized in the following table:

为了（便于）参考，本条目中介绍的术语（以及后面将要介绍的一些术语）总结如下表:

|    Term    |       Example       |      Item     |
|:-------:|:-------:|:-------:|
|   Parameterized type  |     List<String>    |   [Item-26](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-5-Item-26-Do-not-use-raw-types.md)   |
|   Actual type parameter  |     String    |   [Item-26](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-5-Item-26-Do-not-use-raw-types.md)   |
|   Generic type  |     List<E>    |   [Item-26](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-5-Item-26-Do-not-use-raw-types.md), [Item-29](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-5-Item-29-Favor-generic-types.md)   |
|   Formal type parameter  |     E    |   [Item-26](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-5-Item-26-Do-not-use-raw-types.md)   |
|   Unbounded wildcard type  |     List<?>    |   [Item-26](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-5-Item-26-Do-not-use-raw-types.md)   |
|   Raw type  |     List    |   [Item-26](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-5-Item-26-Do-not-use-raw-types.md)   |
|   Bounded type parameter  |     &lt;E extends Number&gt;    |   [Item-29](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-5-Item-29-Favor-generic-types.md)   |
|   Recursive type bound  |     <T extends Comparable<T>>    |   [Item-30](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-5-Item-30-Favor-generic-methods.md)   |
|   Bounded wildcard type  |     List<? extends Number>    |   [Item-31](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-5-Item-31-Use-bounded-wildcards-to-increase-API-flexibility.md)   |
|   Generic method  |     static <E> List<E> asList(E[] a)    |   [Item-30](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-5-Item-30-Favor-generic-methods.md)   |
|   Type token  |     String.class    |   [Item-33](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-5-Item-33-Consider-typesafe-heterogeneous-containers.md)   |
