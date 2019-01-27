## Chapter 6. Enums and Annotations（枚举和注解）

### Item 34: Use enums instead of int constants（用枚举类型代替 int 常量）

An enumerated type is a type whose legal values consist of a fixed set of constants, such as the seasons of the year, the planets in the solar system, or the suits in a deck of playing cards. Before enum types were added to the language, a common pattern for representing enumerated types was to declare a group of named int constants, one for each member of the type:

枚举类型的合法值由一组固定的常量组成，如：一年中的季节、太阳系中的行星或扑克牌中的花色。在枚举类型被添加到 JAVA 之前，表示枚举类型的一种常见模式是声明一组 int 的常量，每个类型的成员都有一个：

```
// The int enum pattern - severely deficient!
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;
public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;
```

This technique, known as the int enum pattern, has many shortcomings. It provides nothing in the way of type safety and little in the way of expressive power. The compiler won’t complain if you pass an apple to a method that expects an orange, compare apples to oranges with the == operator, or worse:

这种技术称为 int 枚举模式，它有许多缺点。它没有提供任何类型安全性，也没有提供任何表达能力。如果你传递一个苹果给一个方法，希望得到一个橘子，使用 == 操作符比较苹果和橘子时编译器并不会提示错误，或更糟：

```
// Tasty citrus flavored applesauce!
int i = (APPLE_FUJI - ORANGE_TEMPLE) / APPLE_PIPPIN;
```

Note that the name of each apple constant is prefixed with APPLE_ and the name of each orange constant is prefixed with ORANGE_. This is because Java doesn’t provide namespaces for int enum groups. Prefixes prevent name clashes when two int enum groups have identically named constants, for example between ELEMENT_MERCURY and PLANET_MERCURY.

注意，每个 apple 常量的名称都以 APPLE_ 为前缀，每个 orange 常量的名称都以 ORANGE_ 为前缀。这是因为 Java 不为 int enum 组提供名称空间。当两个 int 枚举组具有相同的命名常量时，前缀可以防止名称冲突，例如 ELEMENT_MERCURY 和 PLANET_MERCURY 之间的冲突。

Programs that use int enums are brittle. Because int enums are constant variables [JLS, 4.12.4], their int values are compiled into the clients that use them [JLS, 13.1]. If the value associated with an int enum is changed, its clients must be recompiled. If not, the clients will still run, but their behavior will be incorrect.

使用 int 枚举的程序很脆弱。因为 int 枚举是常量变量 [JLS, 4.12.4]，所以它们的 int 值被编译到使用它们的客户端中 [JLS, 13.1]。如果与 int 枚举关联的值发生了更改，则必须重新编译客户端。如果不重新编译，客户端仍然可以运行，但是他们的行为将是错误的。

There is no easy way to translate int enum constants into printable strings. If you print such a constant or display it from a debugger, all you see is a number, which isn’t very helpful. There is no reliable way to iterate over all the int enum constants in a group, or even to obtain the size of an int enum group.

没有一种简单的方法可以将 int 枚举常量转换为可打印的字符串。如果你打印这样的常量或从调试器中显示它，你所看到的只是一个数字，这不是很有帮助。没有可靠的方法可以遍历组中的所有 int 枚举常量，甚至无法获得组的大小。

You may encounter a variant of this pattern in which String constants are used in place of int constants. This variant, known as the String enum pattern, is even less desirable. While it does provide printable strings for its constants, it can lead naive users to hard-code string constants into client code instead of using field names. If such a hard-coded string constant contains a typographical error, it will escape detection at compile time and result in bugs at runtime. Also, it might lead to performance problems, because it relies on string comparisons.

可能会遇到这种模式的变体，使用字符串常量代替 int 常量。这种称为 String 枚举模式的变体甚至更不可取。虽然它确实为常量提供了可打印的字符串，但是它可能会导致不知情的用户将字符串常量硬编码到客户端代码中，而不是使用字段名。如果这样一个硬编码的字符串常量包含一个排版错误，它将在编译时躲过检测，并在运行时导致错误。此外，它可能会导致性能问题，因为它依赖于字符串比较。

Luckily, Java provides an alternative that avoids all the shortcomings of the int and string enum patterns and provides many added benefits. It is the enum type [JLS, 8.9]. Here’s how it looks in its simplest form:

幸运的是，Java 提供了一种替代方案，它避免了 int 和 String 枚举模式的所有缺点，并提供了许多额外的好处。它是 enum 类型 [JLS, 8.9]。下面是它最简单的形式：

```
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```

On the surface, these enum types may appear similar to those of other languages, such as C, C++, and C#, but appearances are deceiving. Java’s enum types are full-fledged classes, far more powerful than their counterparts in these other languages, where enums are essentially int values.

从表面上看，这些枚举类型可能与其他语言（如 C、c++ 和 c#）的枚举类型类似，但这是「以貌取人」。Java的枚举类型是成熟的类，比其他语言中的枚举类型功能强大得多，在其他语言中枚举本质上是 int 值。

The basic idea behind Java’s enum types is simple: they are classes that export one instance for each enumeration constant via a public static final field. Enum types are effectively final, by virtue of having no accessible constructors. Because clients can neither create instances of an enum type nor extend it, there can be no instances but the declared enum constants. In other words, enum types are instance-controlled (page 6). They are a generalization of singletons (Item 3), which are essentially single-element enums.

Java 枚举类型背后的基本思想很简单：它们是通过 public static final 修饰的字段为每个枚举常量导出一个实例的类。枚举类型实际上是 final 类型，因为没有可访问的构造函数。客户端既不能创建枚举类型的实例，也不能扩展它，所以除了声明的枚举常量之外，不能有任何实例。换句话说，枚举类型是单例（[Item-6](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-2-Item-6-Avoid-creating-unnecessary-objects.md)）。

Enums provide compile-time type safety. If you declare a parameter to be of type Apple, you are guaranteed that any non-null object reference passed to the parameter is one of the three valid Apple values. Attempts to pass values of the wrong type will result in compile-time errors, as will attempts to assign an expression of one enum type to a variable of another, or to use the == operator to compare values of different enum types.

枚举提供编译时类型的安全性。如果将参数声明为 Apple 类型，则可以保证传递给该参数的任何非空对象引用都是三个有效 Apple 值之一。尝试传递错误类型的值将导致编译时错误，就像尝试将一个枚举类型的表达式赋值给另一个枚举类型的变量，或者使用 == 运算符比较不同枚举类型的值一样。

Enum types with identically named constants coexist（vi. 共存；和平共处） peacefully because each type has its own namespace. You can add or reorder constants in an enum type without recompiling its clients because the fields that export the constants provide a layer of insulation between an enum type and its clients: constant values are not compiled into the clients as they are in the int enum patterns. Finally, you can translate enums into printable strings by calling their toString method.

名称相同的枚举类型常量能和平共存，因为每种类型都有自己的名称空间。您可以在枚举类型中添加或重新排序常量，而无需重新编译其客户端，因为导出常量的字段在枚举类型及其客户端之间提供了一层隔离：常量值不会像在 int enum 模式中那样编译到客户端中。最后，您可以通过调用枚举的 toString 方法将其转换为可打印的字符串。

In addition to rectifying the deficiencies of int enums, enum types let you add arbitrary methods and fields and implement arbitrary interfaces. They provide high-quality implementations of all the Object methods (Chapter 3), they implement Comparable (Item 14) and Serializable (Chapter 12), and their serialized form is designed to withstand most changes to the enum type.

除了纠正 int 枚举的不足之外，枚举类型还允许您添加任意方法和字段并实现任意接口。它们提供了所有 Object 方法的高质量实现（参阅 Chapter 3），实现了 Comparable（[Item-14](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-3-Item-14-Consider-implementing-Comparable.md)）和 Serializable（参阅 Chapter 12），并且它们的序列化形式被设计成能够承受枚举类型的大多数更改。

So why would you want to add methods or fields to an enum type? For starters, you might want to associate data with its constants. Our Apple and Orange types, for example, might benefit from a method that returns the color of the fruit, or one that returns an image of it. You can augment an enum type with any method that seems appropriate. An enum type can start life as a simple collection of enum constants and evolve over time into a full-featured abstraction.

那么，为什么要向枚举类型添加方法或字段呢？首先，您可能希望将数据与其常量关联起来。例如，我们的 Apple 和 Orange 类型可能受益于返回水果颜色的方法，或者返回水果图像的方法。您可以使用任何适当的方法来扩充枚举类型。枚举类型可以从枚举常量的简单集合开始，并随着时间的推移演变为功能齐全的抽象。

For a nice example of a rich enum type, consider the eight planets of our solar system. Each planet has a mass and a radius, and from these two attributes you can compute its surface gravity. This in turn lets you compute the weight of an object on the planet’s surface, given the mass of the object. Here’s how this enum looks. The numbers in parentheses after each enum constant are parameters that are passed to its constructor. In this case, they are the planet’s mass and radius:

对于富枚举类型来说，有个很好的例子，考虑我们太阳系的八颗行星。每颗行星都有质量和半径，通过这两个属性你可以计算出它的表面引力。反过来让你计算出一个物体在行星表面的重量，给定物体的质量。这个枚举是这样的。每个枚举常量后括号中的数字是传递给其构造函数的参数。在这种情况下，它们是行星的质量和半径：

```
// Enum type with data and behavior
public enum Planet {
    MERCURY(3.302e+23, 2.439e6),
    VENUS (4.869e+24, 6.052e6),
    EARTH (5.975e+24, 6.378e6),
    MARS (6.419e+23, 3.393e6),
    JUPITER(1.899e+27, 7.149e7),
    SATURN (5.685e+26, 6.027e7),
    URANUS (8.683e+25, 2.556e7),
    NEPTUNE(1.024e+26, 2.477e7);

    private final double mass; // In kilograms
    private final double radius; // In meters
    private final double surfaceGravity; // In m / s^2
    // Universal gravitational constant in m^3 / kg s^2
    private static final double G = 6.67300E-11;
    // Constructor

    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }

    public double mass() { return mass; }
    public double radius() { return radius; }
    public double surfaceGravity() { return surfaceGravity; }

    public double surfaceWeight(double mass) {
        return mass * surfaceGravity; // F = ma
    }
}
```

It is easy to write a rich enum type such as Planet. **To associate data with enum constants, declare instance fields and write a constructor that takes the data and stores it in the fields.** Enums are by their nature immutable, so all fields should be final (Item 17). Fields can be public, but it is better to make them private and provide public accessors (Item 16). In the case of Planet, the constructor also computes and stores the surface gravity, but this is just an optimization. The gravity could be recomputed from the mass and radius each time it was used by the surfaceWeight method, which takes an object’s mass and returns its weight on the planet represented by the constant. While the Planet enum is simple, it is surprisingly powerful. Here is a short program that takes the earth weight of an object (in any unit) and prints a nice table of the object’s weight on all eight planets (in the same unit):

编写一个富枚举类型（如上述的 Planet）很容易。**要将数据与枚举常量关联，请声明实例字段并编写一个构造函数，该构造函数接受数据并将其存储在字段中。** 枚举本质上是不可变的，因此所有字段都应该是 final（[Item-17](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-4-Item-17-Minimize-mutability.md)）。字段可以是公共的，但是最好将它们设置为私有并提供公共访问器（[Item-16](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-4-Item-16-In-public-classes-use-accessor-methods-not-public-fields.md)）。在 Planet 的情况下，构造函数还计算和存储表面重力，但这只是一个优化。每一次使用 surfaceWeight 方法时，都可以从质量和半径重新计算重力。surfaceWeight 方法获取一个物体的质量，并返回其在该常数所表示的行星上的重量。虽然行星全会很简单，但它的力量惊人。下面是一个简短的程序，它获取一个物体的地球重量（以任何单位表示），并打印一个漂亮的表格，显示该物体在所有 8 个行星上的重量（以相同的单位表示）：

```
public class WeightTable {
    public static void main(String[] args) {
        double earthWeight = Double.parseDouble(args[0]);
        double mass = earthWeight / Planet.EARTH.surfaceGravity();
    for (Planet p : Planet.values())
        System.out.printf("Weight on %s is %f%n",p, p.surfaceWeight(mass));
    }
}
```

Note that Planet, like all enums, has a static values method that returns an array of its values in the order they were declared. Note also that the toString method returns the declared name of each enum value, enabling easy printing by println and printf. If you’re dissatisfied with this string representation, you can change it by overriding the toString method. Here is the result of running our WeightTable program (which doesn’t override toString) with the command line argument 185:

请注意，Planet 和所有枚举一样，有一个静态值方法，该方法按照声明值的顺序返回其值的数组。还要注意的是，toString 方法返回每个枚举值的声明名称，这样就可以通过 println 和 printf 轻松打印。如果您对这个字符串表示不满意，可以通过重写 toString 方法来更改它。下面是用命令行运行我们的 WeightTable 程序（未覆盖 toString）的结果：

```
Weight on MERCURY is 69.912739
Weight on VENUS is 167.434436
Weight on EARTH is 185.000000
Weight on MARS is 70.226739
Weight on JUPITER is 467.990696
Weight on SATURN is 197.120111
Weight on URANUS is 167.398264
Weight on NEPTUNE is 210.208751
```

Until 2006, two years after enums were added to Java, Pluto was a planet. This raises the question “what happens when you remove an element from an enum type?” The answer is that any client program that doesn’t refer to the removed element will continue to work fine. So, for example, our WeightTable program would simply print a table with one fewer row. And what of a client program that refers to the removed element (in this case, Planet.Pluto)? If you recompile the client program, the compilation will fail with a helpful error message at the line that refers to the erstwhile planet; if you fail to recompile the client, it will throw a helpful exception from this line at runtime. This is the best behavior you could hope for, far better than what you’d get with the int enum pattern.

直到 2006 年，也就是枚举被添加到 Java 的两年后，冥王星还是一颗行星。这就提出了一个问题:「从枚举类型中删除元素时会发生什么?」答案是，任何不引用被删除元素的客户端程序将继续正常工作。例如，我们的 WeightTable 程序只需打印一个少一行的表。那么引用被删除元素（在本例中是 Planet.Pluto）的客户机程序又如何呢？如果重新编译客户端程序，编译将失败，并在引用以前的行星的行中显示一条有用的错误消息；如果您未能重新编译客户端，它将在运行时从这行抛出一个有用的异常。这是您所希望的最佳行为，比 int enum 模式要好得多。

Some behaviors associated with enum constants may need to be used only from within the class or package in which the enum is defined. Such behaviors are best implemented as private or package-private methods. Each constant then carries with it a hidden collection of behaviors that allows the class or package containing the enum to react appropriately when presented with the constant. Just as with other classes, unless you have a compelling reason to expose an enum method to its clients, declare it private or, if need be, package-private (Item 15).

与枚举常量相关的一些行为可能只需要在定义枚举的类或包中使用。此类行为最好以私有或包私有方法来实现。然后，每个常量都带有一个隐藏的行为集合，允许包含枚举的类或包在使用该常量时做出适当的反应。与其他类一样，除非您有充分的理由向其客户端公开枚举方法，否则将其声明为私有的，或者在需要时声明为包私有（[Item-15](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-4-Item-15-Minimize-the-accessibility-of-classes-and-members.md)）。

***译注：Java 中访问级别规则如下：***
- ***类访问级别：public（公共）、无修饰符（package-private，包私有）***
- ***成员访问级别：public（公共）、protected（保护）、private（私有）、无修饰符（package-private，包私有）***

If an enum is generally useful, it should be a top-level class; if its use is tied to a specific top-level class, it should be a member class of that top-level class (Item 24). For example, the java.math.RoundingMode enum represents a rounding mode for decimal fractions. These rounding modes are used by the BigDecimal class, but they provide a useful abstraction that is not fundamentally tied to BigDecimal. By making RoundingMode a top-level enum, the library designers encourage any programmer who needs rounding modes to reuse this enum, leading to increased consistency across APIs.

通常，如果一个枚举用途广泛，那么它应该是顶级类；如果它被绑定到一个特定的顶级类使用，那么它应该是这个顶级类（[Item-24](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-4-Item-24-Favor-static-member-classes-over-nonstatic.md)）的成员类。例如，java.math.RoundingMode 枚举表示小数部分的舍入模式。BigDecimal 类使用这些四舍五入模式，但是它们提供了一个有用的抽象，这个抽象与 BigDecimal 没有本质上的联系。通过使 RoundingMode 成为顶级枚举，库设计人员支持任何需要舍入模式的程序员重用该枚举，从而提高 API 之间的一致性。

The techniques demonstrated in the Planet example are sufficient（adj. 足够的；充分的） for most enum types, but sometimes you need more. There is different data associated with each Planet constant, but sometimes you need to associate fundamentally different behavior with each constant. For example, suppose you are writing an enum type to represent the operations on a basic four-function calculator and you want to provide a method to perform the arithmetic operation represented by each constant. One way to achieve this is to switch on the value of the enum:

Planet 示例中演示的技术对于大多数枚举类型来说已经足够了，但有时还需要更多。每个行星常数都有不同的数据，但有时你需要将基本不同的行为与每个常数联系起来。例如，假设您正在编写一个枚举类型来表示一个基本的四函数计算器上的操作，并且您希望提供一个方法来执行由每个常量表示的算术操作。实现这一点的一种方法是切换枚举的值：

```
// Enum type that switches on its own value - questionable
public enum Operation {
    PLUS, MINUS, TIMES, DIVIDE;
    // Do the arithmetic operation represented by this constant
    public double apply(double x, double y) {
        switch(this) {
            case PLUS: return x + y;
            case MINUS: return x - y;
            case TIMES: return x * y;
            case DIVIDE: return x / y;
        }
    throw new AssertionError("Unknown op: "+this);
    }
}
```

This code works, but it isn’t very pretty. It won’t compile without the throw statement because the end of the method is technically reachable, even though it will never be reached [JLS, 14.21]. Worse, the code is fragile. If you add a new enum constant but forget to add a corresponding case to the switch, the enum will still compile, but it will fail at runtime when you try to apply the new operation.

这段代码可以工作，但不是很漂亮。如果没有抛出语句，它将无法编译，因为从技术上讲，方法的结尾是可到达的，即使它永远也无法到达[JLS, 14.21]。更糟糕的是，代码很脆弱。如果您添加了一个新的枚举常量，但忘记向 switch 添加相应的 case，则枚举仍将编译，但在运行时尝试应用新操作时将失败。

Luckily, there is a better way to associate a different behavior with each enum constant: declare an abstract apply method in the enum type, and override it with a concrete method for each constant in a constant-specific class body. Such methods are known as constant-specific method implementations:

幸运的是，有一种更好的方法可以将不同的行为与每个枚举常量关联起来：在枚举类型中声明一个抽象的 apply 方法，并用一个特定于常量的类体中的每个常量的具体方法覆盖它。这些方法称为特定于常量的方法实现：

```
// Enum type with constant-specific method implementations
public enum Operation {
    PLUS {public double apply(double x, double y){return x + y;}},
    MINUS {public double apply(double x, double y){return x - y;}},
    TIMES {public double apply(double x, double y){return x * y;}},
    DIVIDE{public double apply(double x, double y){return x / y;}};
    public abstract double apply(double x, double y);
}
```

If you add a new constant to the second version of Operation, it is unlikely that you’ll forget to provide an apply method, because the method immediately follows each constant declaration. In the unlikely event that you do forget, the compiler will remind you because abstract methods in an enum type must be overridden with concrete methods in all of its constants.

如果您在操作的第二个版本中添加一个新常量，那么您不太可能忘记提供一个 apply 方法，因为该方法紧跟每个常量声明。在不太可能忘记的情况下，编译器会提醒您，因为枚举类型中的抽象方法必须用其所有常量中的具体方法覆盖。

Constant-specific method implementations can be combined with constantspecific data. For example, here is a version of Operation that overrides the toString method to return the symbol commonly associated with the operation:

特定于常量的方法实现可以与特定于常量的数据相结合。例如，下面是一个操作的版本，它重写 toString 方法来返回通常与该操作关联的符号：

```
// Enum type with constant-specific class bodies and data
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

    @Override
    public String toString() { return symbol; }

    public abstract double apply(double x, double y);
}
```

The toString implementation shown makes it easy to print arithmetic expressions, as demonstrated by this little program:

所示的 toString 实现使得打印算术表达式变得很容易，如下面的小程序所示:

```
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    for (Operation op : Operation.values())
        System.out.printf("%f %s %f = %f%n",x, op, y, op.apply(x, y));
}
```

Running this program with 2 and 4 as command line arguments produces the following output:

以 2 和 4 作为命令行参数运行这个程序将产生以下输出：

```
2.000000 + 4.000000 = 6.000000
2.000000 - 4.000000 = -2.000000
2.000000 * 4.000000 = 8.000000
2.000000 / 4.000000 = 0.500000
```

Enum types have an automatically generated valueOf(String) method that translates a constant’s name into the constant itself. If you override the toString method in an enum type, consider writing a fromString method to translate the custom string representation back to the corresponding enum. The following code (with the type name changed appropriately) will do the trick for any enum, so long as each constant has a unique string representation:

枚举类型有一个自动生成的 valueOf(String) 方法，该方法将常量的名称转换为常量本身。如果在枚举类型中重写 toString 方法，可以考虑编写 fromString 方法将自定义字符串表示形式转换回相应的枚举。只要每个常量都有唯一的字符串表示形式，下面的代码（类型名称适当更改）就可以用于任何枚举：

```
// Implementing a fromString method on an enum type
private static final Map<String, Operation> stringToEnum =Stream.of(values()).collect(toMap(Object::toString, e -> e));

// Returns Operation for string, if any
public static Optional<Operation> fromString(String symbol) {
    return Optional.ofNullable(stringToEnum.get(symbol));
}
```

Note that the Operation constants are put into the stringToEnum map from a static field initialization that runs after the enum constants have been created. The previous code uses a stream (Chapter 7) over the array returned by the values() method; prior to Java 8, we would have created an empty hash map and iterated over the values array inserting the string-to-enum mappings into the map, and you can still do it that way if you prefer. But note that attempting to have each constant put itself into a map from its own constructor does not work. It would cause a compilation error, which is good thing because if it were legal, it would cause a NullPointerException at runtime. Enum constructors aren’t permitted to access the enum’s static fields, with the exception of constant variables (Item 34). This restriction is necessary because static fields have not yet been initialized when enum constructors run. A special case of this restriction is that enum constants cannot access one another from their constructors.

Also note that the fromString method returns an Optional<String>. This allows the method to indicate that the string that was passed in does not represent a valid operation, and it forces the client to confront this possibility (Item 55).

A disadvantage of constant-specific method implementations is that they make it harder to share code among enum constants. For example, consider an enum representing the days of the week in a payroll package. This enum has a method that calculates a worker’s pay for that day given the worker’s base salary (per hour) and the number of minutes worked on that day. On the five weekdays, any time worked in excess of a normal shift generates overtime pay; on the two weekend days, all work generates overtime pay. With a switch statement, it’s easy to do this calculation by applying multiple case labels to each of two code fragments:

```
// Enum that switches on its value to share code - questionable
enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY,SATURDAY, SUNDAY;
    private static final int MINS_PER_SHIFT = 8 * 60;
    int pay(int minutesWorked, int payRate) {
        int basePay = minutesWorked * payRate;
        int overtimePay;
        switch(this) {
            case SATURDAY: case SUNDAY: // Weekend
                overtimePay = basePay / 2;
                break;
            default: // Weekday
                overtimePay = minutesWorked <= MINS_PER_SHIFT ?0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
    }
    return basePay + overtimePay;
    }
}
```

This code is undeniably concise, but it is dangerous from a maintenance perspective. Suppose you add an element to the enum, perhaps a special value to represent a vacation day, but forget to add a corresponding case to the switch statement. The program will still compile, but the pay method will silently pay the worker the same amount for a vacation day as for an ordinary weekday.

To perform the pay calculation safely with constant-specific method implementations, you would have to duplicate the overtime pay computation for each constant, or move the computation into two helper methods, one for weekdays and one for weekend days, and invoke the appropriate helper method from each constant. Either approach would result in a fair amount of boilerplate code, substantially reducing readability and increasing the opportunity for error.

The boilerplate could be reduced by replacing the abstract overtimePay method on PayrollDay with a concrete method that performs the overtime calculation for weekdays. Then only the weekend days would have to override the method. But this would have the same disadvantage as the switch statement: if you added another day without overriding the overtimePay method, you would silently inherit the weekday calculation.

What you really want is to be forced to choose an overtime pay strategy each time you add an enum constant. Luckily, there is a nice way to achieve this. The idea is to move the overtime pay computation into a private nested enum, and to pass an instance of this strategy enum to the constructor for the PayrollDay enum. The PayrollDay enum then delegates the overtime pay calculation to the strategy enum, eliminating the need for a switch statement or constantspecific method implementation in PayrollDay. While this pattern is less concise than the switch statement, it is safer and more flexible:

```
// The strategy enum pattern
enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY,SATURDAY(PayType.WEEKEND), SUNDAY(PayType.WEEKEND);
    private final PayType payType;
    PayrollDay(PayType payType) { this.payType = payType; }
    PayrollDay() { this(PayType.WEEKDAY); } // Default

    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }

    // The strategy enum type
    private enum PayType {
        WEEKDAY {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked <= MINS_PER_SHIFT ? 0 :(minsWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked * payRate / 2;
            }
        };

        abstract int overtimePay(int mins, int payRate);

        private static final int MINS_PER_SHIFT = 8 * 60;

        int pay(int minsWorked, int payRate) {
            int basePay = minsWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
        }
    }
}
```

If switch statements on enums are not a good choice for implementing constant-specific behavior on enums, what are they good for? **Switches on enums are good for augmenting enum types with constant-specific behavior.** For example, suppose the Operation enum is not under your control and you wish it had an instance method to return the inverse of each operation. You could simulate the effect with the following static method:

```
// Switch on an enum to simulate a missing method
public static Operation inverse(Operation op) {
    switch(op) {
        case PLUS: return Operation.MINUS;
        case MINUS: return Operation.PLUS;
        case TIMES: return Operation.DIVIDE;
        case DIVIDE: return Operation.TIMES;
        default: throw new AssertionError("Unknown op: " + op);
    }
}
```

You should also use this technique on enum types that are under your control if a method simply doesn’t belong in the enum type. The method may be required for some use but is not generally useful enough to merit inclusion in the enum type.

Enums are, generally speaking, comparable in performance to int constants. A minor performance disadvantage of enums is that there is a space and time cost to load and initialize enum types, but it is unlikely to be noticeable in practice.

So when should you use enums? **Use enums any time you need a set of constants whose members are known at compile time.** Of course, this includes “natural enumerated types,” such as the planets, the days of the week, and the chess pieces. But it also includes other sets for which you know all the possible values at compile time, such as choices on a menu, operation codes, and command line flags. **It is not necessary that the set of constants in an enum type stay fixed for all time.** The enum feature was specifically designed to allow for binary compatible evolution of enum types.

In summary, the advantages of enum types over int constants are compelling. Enums are more readable, safer, and more powerful. Many enums require no explicit constructors or members, but others benefit from associating data with each constant and providing methods whose behavior is affected by this data. Fewer enums benefit from associating multiple behaviors with a single method. In this relatively rare case, prefer constant-specific methods to enums that switch on their own values. Consider the strategy enum pattern if some, but not all, enum constants share common behaviors.
