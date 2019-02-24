## Chapter 7. Lambdas and Streams（λ 表达式和流）

### Item 42: Prefer lambdas to anonymous classes（λ 表达式优于匿名类）

Historically, interfaces (or, rarely, abstract classes) with a single abstract method were used as function types. Their instances, known as function objects, represent functions or actions. Since JDK 1.1 was released in 1997, the primary means of creating a function object was the anonymous class (Item 24). Here’s a code snippet to sort a list of strings in order of length, using an anonymous class to create the sort’s comparison function (which imposes the sort order):

在历史上，带有单个抽象方法的接口（或者抽象类，但这种情况很少）被用作函数类型。它们的实例（称为函数对象）表示函数或操作。自从 JDK 1.1 在 1997 年发布以来，创建函数对象的主要方法就是匿名类（[Item-24](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-4/Chapter-4-Item-24-Favor-static-member-classes-over-nonstatic.md)）。下面是一个按长度对字符串列表进行排序的代码片段，使用一个匿名类来创建排序的比较函数（它强制执行排序顺序）：

```
// Anonymous class instance as a function object - obsolete!
Collections.sort(words, new Comparator<String>() {
    public int compare(String s1, String s2) {
        return Integer.compare(s1.length(), s2.length());
    }
});
```

Anonymous classes were adequate for the classic objected-oriented design patterns requiring function objects, notably the Strategy pattern [Gamma95]. The Comparator interface represents an abstract strategy for sorting; the anonymous class above is a concrete strategy for sorting strings. The verbosity of anonymous classes, however, made functional programming in Java an unappealing prospect.

匿名类对于需要函数对象的典型面向对象设计模式来说已经足够了，尤其是策略模式 [Gamma95]。比较器接口表示排序的抽象策略；上述匿名类是对字符串排序的一种具体策略。然而，匿名类的冗长使函数式编程在 Java 中变得毫无吸引力。

In Java 8, the language formalized the notion that interfaces with a single abstract method are special and deserve special treatment. These interfaces are now known as functional interfaces, and the language allows you to create instances of these interfaces using lambda expressions, or lambdas for short. Lambdas are similar in function to anonymous classes, but far more concise. Here’s how the code snippet above looks with the anonymous class replaced by a lambda. The boilerplate is gone, and the behavior is clearly evident:

在 Java 8 中，该语言形式化了一个概念，即具有单个抽象方法的接口是特殊的，应该得到特殊处理。这些接口现在被称为函数式接口，该语言允许您使用 lambda 表达式创建这些接口的实例。Lambda 表达式在功能上类似于匿名类，但是更加简洁。下面的代码片段，匿名类被 lambda 表达式替换。样板已经没有了，行为非常明显：

```
// Lambda expression as function object (replaces anonymous class)
Collections.sort(words,(s1, s2) -> Integer.compare(s1.length(), s2.length()));
```

Note that the types of the lambda (`Comparator<String>`), of its parameters (s1 and s2, both String), and of its return value (int) are not present in the code. The compiler deduces these types from context, using a process known as type inference. In some cases, the compiler won’t be able to determine the types, and you’ll have to specify them. The rules for type inference are complex: they take up an entire chapter in the JLS [JLS, 18]. Few programmers understand these rules in detail, but that’s OK. **Omit the types of all lambda parameters unless their presence makes your program clearer.** If the compiler generates an error telling you it can’t infer the type of a lambda parameter, then specify it. Sometimes you may have to cast the return value or the entire lambda expression, but this is rare.

注意，lambda 表达式（`Comparator<String>`）、它的参数（s1 和 s2，都是字符串）及其返回值（int）的类型在代码中不存在。编译器使用称为类型推断的过程从上下文中推断这些类型。在某些情况下，编译器无法确定类型，您必须指定它们。类型推断的规则很复杂：它们在 JLS 中占了整整一章 [JLS, 18]。很少有程序员能详细理解这些规则，但这没有关系。省略所有 lambda 表达式参数的类型，除非它们的存在使您的程序更清晰。如果编译器生成一个错误，告诉您它不能推断 lambda 表达式参数的类型，那么就指定它。有时您可能必须强制转换返回值或整个 lambda 表达式，但这种情况很少见。

One caveat should be added concerning type inference. Item 26 tells you not to use raw types, Item 29 tells you to favor generic types, and Item 30 tells you to favor generic methods. This advice is doubly important when you’re using lambdas, because the compiler obtains most of the type information that allows it to perform type inference from generics. If you don’t provide this information, the compiler will be unable to do type inference, and you’ll have to specify types manually in your lambdas, which will greatly increase their verbosity. By way of example, the code snippet above won’t compile if the variable words is declared to be of the raw type List instead of the parameterized type `List<String>`.

关于类型推断，应该添加一个警告。[Item-26](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-5/Chapter-5-Item-26-Do-not-use-raw-types.md) 告诉您不要使用原始类型，[Item-29](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-5/Chapter-5-Item-29-Favor-generic-types.md) 告诉您要优先使用泛型，[Item-30](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-5/Chapter-5-Item-30-Favor-generic-methods.md) 告诉您要优先使用泛型方法。在使用 lambda 表达式时，这些建议尤其重要，因为编译器获得了允许它从泛型中执行类型推断的大部分类型信息。如果不提供此信息，编译器将无法进行类型推断，并且必须在 lambda 表达式中手动指定类型，这将大大增加它们的冗长。举例来说，如果变量声明为原始类型 List 而不是参数化类型 `List<String>`，那么上面的代码片段将无法编译。

Incidentally, the comparator in the snippet can be made even more succinct if a comparator construction method is used in place of a lambda (Items 14. 43):

```
Collections.sort(words, comparingInt(String::length));
```

In fact, the snippet can be made still shorter by taking advantage of the sort method that was added to the List interface in Java 8:

```
words.sort(comparingInt(String::length));
```

The addition of lambdas to the language makes it practical to use function objects where it would not previously have made sense. For example, consider the Operation enum type in Item 34. Because each enum required different behavior for its apply method, we used constant-specific class bodies and overrode the apply method in each enum constant. To refresh your memory, here is the code:

```
// Enum type with constant-specific class bodies & data (Item 34)
public enum Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };

    private final String symbol;

    Operation(String symbol) { this.symbol = symbol; }

    @Override public String toString() { return symbol; }

    public abstract double apply(double x, double y);
}
```

Item 34 says that enum instance fields are preferable to constant-specific class bodies. Lambdas make it easy to implement constant-specific behavior using the former instead of the latter. Merely pass a lambda implementing each enum constant’s behavior to its constructor. The constructor stores the lambda in an instance field, and the apply method forwards invocations to the lambda. The resulting code is simpler and clearer than the original version:

```
// Enum with function object fields & constant-specific behavior
public enum Operation {
    PLUS ("+", (x, y) -> x + y),
    MINUS ("-", (x, y) -> x - y),
    TIMES ("*", (x, y) -> x * y),
    DIVIDE("/", (x, y) -> x / y);

    private final String symbol;

    private final DoubleBinaryOperator op;

    Operation(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }

    @Override public String toString() { return symbol; }

    public double apply(double x, double y) {
        return op.applyAsDouble(x, y);
    }
}
```

Note that we’re using the DoubleBinaryOperator interface for the lambdas that represent the enum constant’s behavior. This is one of the many predefined functional interfaces in java.util.function (Item 44). It represents a function that takes two double arguments and returns a double result.

Looking at the lambda-based Operation enum, you might think constantspecific method bodies have outlived their usefulness, but this is not the case. Unlike methods and classes, **lambdas lack names and documentation; if a computation isn’t self-explanatory, or exceeds a few lines, don’t put it in a lambda.** One line is ideal for a lambda, and three lines is a reasonable maximum. If you violate this rule, it can cause serious harm to the readability of your programs. If a lambda is long or difficult to read, either find a way to simplify it or refactor your program to eliminate it. Also, the arguments passed to enum constructors are evaluated in a static context. Thus, lambdas in enum constructors can’t access instance members of the enum. Constant-specific class bodies are still the way to go if an enum type has constant-specific behavior that is difficult to understand, that can’t be implemented in a few lines, or that requires access to instance fields or methods.

Likewise, you might think that anonymous classes are obsolete in the era of lambdas. This is closer to the truth, but there are a few things you can do with anonymous classes that you can’t do with lambdas. Lambdas are limited to functional interfaces. If you want to create an instance of an abstract class, you can do it with an anonymous class, but not a lambda. Similarly, you can use anonymous classes to create instances of interfaces with multiple abstract methods. Finally, a lambda cannot obtain a reference to itself. In a lambda, the this keyword refers to the enclosing instance, which is typically what you want. In an anonymous class, the this keyword refers to the anonymous class instance. If you need access to the function object from within its body, then you must use an anonymous class.

Lambdas share with anonymous classes the property that you can’t reliably serialize and deserialize them across implementations. Therefore, **you should rarely, if ever, serialize a lambda** (or an anonymous class instance). If you have a function object that you want to make serializable, such as a Comparator, use an instance of a private static nested class (Item 24).

In summary, as of Java 8, lambdas are by far the best way to represent small function objects. **Don’t use anonymous classes for function objects unless you have to create instances of types that aren’t functional interfaces. Also, remember that lambdas make it so easy to represent small function objects that it opens the door to functional programming techniques that were not previously practical in Java.
