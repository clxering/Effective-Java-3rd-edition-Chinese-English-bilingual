## Chapter 3. Methods Common to All Objects（对象的通用方法）

### Item 10: Obey the general contract when overriding equals（覆盖 equals 方法时应遵守的约定）

Overriding the equals method seems simple, but there are many ways to get it wrong, and consequences can be dire. The easiest way to avoid problems is not to override the equals method, in which case each instance of the class is equal only to itself. This is the right thing to do if any of the following conditions apply:

覆盖 equals 方法似乎很简单，但是有很多覆盖的方式会导致出错，而且后果可能非常严重。避免问题的最简单方法是不覆盖 equals 方法，在这种情况下，类的每个实例都只等于它自己。如果符合下列任何条件，就是正确的做法：

- **Each instance of the class is inherently unique.** This is true for classes such as Thread that represent active entities rather than values. The equals implementation provided by Object has exactly the right behavior for these classes.

**类的每个实例本质上都是唯一的。** 对于像 Thread 这样表示活动实体类而不是值类来说也是如此。Object 提供的 equals 实现对于这些类具有完全正确的行为。

- **There is no need for the class to provide a “logical equality” test.** For example, java.util.regex.Pattern could have overridden equals to check whether two Pattern instances represented exactly the same regular expression, but the designers didn’t think that clients would need or want this functionality. Under these circumstances, the equals implementation inherited from Object is ideal.

**该类不需要提供「逻辑相等」测试。** 例如，`java.util.regex.Pattern` 可以覆盖 equals 来检查两个 Pattern 实例是否表示完全相同的正则表达式，但设计人员认为客户端不需要或不需要这个功能。在这种情况下，从 Object 继承的 equals 实现是理想的。

- **A superclass has already overridden equals, and the superclass behavior is appropriate for this class.** For example, most Set implementations inherit their equals implementation from AbstractSet, List implementations from AbstractList, and Map implementations from AbstractMap.

**超类已经覆盖了 equals，超类行为适合于这个类。** 例如，大多数 Set 的实现从 AbstractSet 继承其对等实现，List 从 AbstractList 继承实现，Map 从 AbstractMap 继承实现。

- **The class is private or package-private, and you are certain that its equals method will never be invoked.** If you are extremely risk-averse,you can override the equals method to ensure that it isn’t invoked accidentally:

**类是私有的或包私有的，并且你确信它的 equals 方法永远不会被调用。** 如果你非常厌恶风险，你可以覆盖 equals 方法，以确保它不会意外调用：

```
@Override
public boolean equals(Object o) {
    throw new AssertionError(); // Method is never called
}
```

So when is it appropriate to override equals? It is when a class has a notion of logical equality that differs from mere object identity and a superclass has not already overridden equals. This is generally the case for value classes. A value class is simply a class that represents a value, such as Integer or String. A programmer who compares references to value objects using the equals method expects to find out whether they are logically equivalent, not whether they refer to the same object. Not only is overriding the equals method necessary to satisfy programmer expectations, it enables instances to serve as map keys or set elements with predictable, desirable behavior.

什么时候覆盖 equals 方法是合适的？当一个类有一个逻辑相等的概念，而这个概念不同于仅判断对象的同一性（相同对象的引用），并且超类还没有覆盖 equals。对于值类通常是这样。值类只是表示值的类，例如 Integer 或 String。使用 equals 方法比较引用和值对象的程序员希望发现它们在逻辑上是否等价，而不是它们是否引用相同的对象。覆盖 equals 方法不仅是为了满足程序员的期望，它还使实例能够作为 Map 的键或 Set 元素时，具有可预测的、理想的行为。

**译注 1：有一个表示状态的内部类。没有覆盖 equals 方法时，equals 的结果与 s1==s2 相同，为 false，即两者并不是相同对象的引用。**
```
public static void main(String[] args) {

    class Status {
        public String status;
    }

    Status s1 = new Status();
    Status s2 = new Status();

    System.out.println(s1==s2); // false
    System.out.println(s1.equals(s2)); // false
}
```
**译注 2：覆盖 equals 方法后，以业务逻辑来判断是否相同，具备相同 status 字段即为相同。在使用去重功能时，也以此作为判断依据。**
```
public static void main(String[] args) {

    class Status {
        public String status;

        @Override
        public boolean equals(Object o) {
            return Objects.equals(status, ((Status) o).status);
        }
    }

    Status s1 = new Status();
    Status s2 = new Status();

    System.out.println(s1==s2); // false
    System.out.println(s1.equals(s2)); // true
}
```

One kind of value class that does not require the equals method to be overridden is a class that uses instance control (Item 1) to ensure that at most one object exists with each value. Enum types (Item 34) fall into this category. For these classes, logical equality is the same as object identity, so Object’s equals method functions as a logical equals method.

不需要覆盖 equals 方法的一种值类是使用实例控件（[Item-1](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-2/Chapter-2-Item-1-Consider-static-factory-methods-instead-of-constructors.md)）来确保每个值最多只存在一个对象的类。枚举类型（[Item-34](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-6/Chapter-6-Item-34-Use-enums-instead-of-int-constants.md)）属于这一类。对于这些类，逻辑相等与对象标识相同，因此对象的 equals 方法函数与逻辑 equals 方法相同。

When you override the equals method, you must adhere to its general contract. Here is the contract, from the specification（n.规范，说明书） for Object :

当你覆盖 equals 方法时，你必须遵守它的通用约定。以下是具体内容，来自 Object 规范：

The equals method implements an equivalence relation. It has these properties:

equals 方法实现了等价关系。它应有这些属性：

- Reflexive: For any non-null reference value x, x.equals(x) must return true.

反身性：对于任何非空的参考值 x，`x.equals(x)` 必须返回 true。

- Symmetric: For any non-null reference values x and y, x.equals(y) must return true if and only if y.equals(x) returns true.

对称性：对于任何非空参考值 x 和 y，`x.equals(y)` 必须在且仅当 `y.equals(x)` 返回 true 时返回 true。

- Transitive: For any non-null reference values x, y, z, if x.equals(y) returns true and y.equals(z) returns true, then x.equals(z) must return true.

传递性：对于任何非空的引用值 x, y, z，如果 `x.equals(y)` 返回 true，`y.equals(z)` 返回 true，那么 `x.equals(z)` 必须返回 true。

- Consistent: For any non-null reference values x and y, multiple invocations of x.equals(y) must consistently return true or consistently return false, provided no information used in equals comparisons is modified.

一致性：对于任何非空的引用值 x 和 y, `x.equals(y)` 的多次调用必须一致地返回 true 或一致地返回 false，前提是不修改 equals 中使用的信息。

- For any non-null reference value x, x.equals(null) must return false.

对于任何非空引用值 x，`x.equals(null)` 必须返回 false。

Unless you are mathematically inclined（v.使…倾向；adj.趋向于…的）, this might look a bit scary, but do not ignore it! If you violate it, you may well find that your program behaves erratically or crashes, and it can be very difficult to pin down the source of the failure. To paraphrase John Donne, no class is an island. Instances of one class are frequently passed to another. Many classes, including all collections classes,depend on the objects passed to them obeying the equals contract.

除非你有数学方面的倾向，否则这些起来有点可怕，但不要忽略它！如果你违反了它，你的程序很可能会出现行为异常或崩溃，并且很难确定失败的根源。用 John Donne 的话来说，没有一个类是孤立的。一个类的实例经常被传递给另一个类。许多类（包括所有集合类）依赖于传递给它们的对象遵守 equals 约定。

Now that you are aware of the dangers of violating the equals contract, let’s go over the contract in detail. The good news is that, appearances notwithstanding, it really isn’t very complicated. Once you understand it, it’s not hard to adhere to it.

既然你已经意识到了违反 equals 约定的危险，让我们详细讨论一下。好消息是，尽管表面上看起来很复杂，但其实并不复杂。一旦你明白了，就不难坚持下去了。

So what is an equivalence relation? Loosely speaking, it’s an operator that partitions a set of elements into subsets whose elements are deemed equal to one another. These subsets are known as equivalence classes. For an equals method to be useful, all of the elements in each equivalence class must be interchangeable from the perspective of the user. Now let’s examine the five requirements in turn:

什么是等价关系？简单地说，它是一个操作符，它将一组元素划分为子集，子集的元素被认为是彼此相等的。这些子集被称为等价类。为了使 equals 方法有用，从用户的角度来看，每个等价类中的所有元素都必须是可互换的。现在让我们依次检查以下五个需求：

**Reflexivity** —The first requirement says merely that an object must be equal to itself. It’s hard to imagine violating this one unintentionally. If you were to violate it and then add an instance of your class to a collection, the contains method might well say that the collection didn’t contain the instance that you just added.

**反身性** ，第一个要求仅仅是说一个对象必须等于它自己。很难想象会无意中违反了这条规则。如果你违反了它，然后将类的一个实例添加到集合中，contains 方法很可能会说该集合不包含你刚才添加的实例。

**Symmetry** —The second requirement says that any two objects must agree on whether they are equal. Unlike the first requirement, it’s not hard to imagine violating this one unintentionally. For example, consider the following class,which implements a case-insensitive string. The case of the string is preserved by toString but ignored in equals comparisons:

**对称性** ，第二个要求是任何两个对象必须在是否相等的问题上达成一致。与第一个要求不同，无意中违反了这个要求的情况不难想象。例如，考虑下面的类，它实现了不区分大小写的字符串。字符串的情况是保留的 toString，但忽略在 equals 的比较：

```
// Broken - violates symmetry!
public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
}

// Broken - violates symmetry!
@Override
public boolean equals(Object o) {
    if (o instanceof CaseInsensitiveString)
    return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);

    if (o instanceof String) // One-way interoperability!
        return s.equalsIgnoreCase((String) o);

    return false;
    } ... // Remainder omitted
}
```

The well-intentioned equals method in this class naively attempts to interoperate with ordinary strings. Let’s suppose that we have one caseinsensitive string and one ordinary one:

这个类中的 equals 方法天真地尝试与普通字符串进行互操作。假设我们有一个不区分大小写的字符串和一个普通字符串：

```
CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
String s = "polish";
```

As expected, cis.equals(s) returns true. The problem is that while the equals method in CaseInsensitiveString knows about ordinary strings, the equals method in String is oblivious to case-insensitive strings.Therefore, s.equals(cis) returns false, a clear violation of symmetry.Suppose you put a case-insensitive string into a collection:

正如预期的那样，`cis.equals(s)` 返回 true。问题是，虽然 CaseInsensitiveString 中的 equals 方法知道普通字符串，但是 String 中的 equals 方法对不区分大小写的字符串不知情。因此，`s.equals(cis)` 返回 false，这明显违反了对称性。假设你将不区分大小写的字符串放入集合中：

```
List<CaseInsensitiveString> list = new ArrayList<>();
list.add(cis);
```

What does list.contains(s) return at this point? Who knows? In the current OpenJDK implementation, it happens to return false, but that’s just an implementation artifact. In another implementation, it could just as easily return true or throw a runtime exception. **Once you’ve violated the equals contract, you simply don’t know how other objects will behave when confronted with your object.**

此时 `list.contains(s)` 返回什么？谁知道呢？在当前的 OpenJDK 实现中，它碰巧返回 false，但这只是一个实现案例。在另一个实现中，它可以很容易地返回 true 或抛出运行时异常。一旦你违反了 equals 约定，就不知道当其他对象面对你的对象时，会如何表现。

**译注：contains 方法在 ArrayList 中的实现源码如下（省略了源码中的多行注释）：**
```
// ArrayList 的大小
private int size;

// 保存 ArrayList 元素的容器，一个 Object 数组
transient Object[] elementData; // non-private to simplify nested class access

public boolean contains(Object o) {
    return indexOf(o) >= 0;
}

public int indexOf(Object o) {
    return indexOfRange(o, 0, size);
}

int indexOfRange(Object o, int start, int end) {
    Object[] es = elementData;
    if (o == null) {
        for (int i = start; i < end; i++) {
            if (es[i] == null) {
                return i;
            }
        }
    } else {
        for (int i = start; i < end; i++) {
            if (o.equals(es[i])) {
                return i;
            }
        }
    }
    return -1;
}
```

To eliminate the problem, merely remove the ill-conceived attempt to interoperate with String from the equals method. Once you do this, you can refactor the method into a single return statement:

为了消除这个问题，只需从 equals 方法中删除与 String 互操作的错误尝试。一旦你这样做了，你可以重构方法为一个单一的返回语句：

```
@Override
public boolean equals(Object o) {
    return o instanceof CaseInsensitiveString && ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
}
```

**Transitivity** —The third requirement of the equals contract says that if one object is equal to a second and the second object is equal to a third, then the first object must be equal to the third. Again, it’s not hard to imagine violating this requirement unintentionally. Consider the case of a subclass that adds a new value component to its superclass. In other words, the subclass adds a piece of information that affects equals comparisons. Let’s start with a simple immutable two-dimensional integer point class:

**传递性** ，equals 约定的第三个要求是，如果一个对象等于第二个对象，而第二个对象等于第三个对象，那么第一个对象必须等于第三个对象。同样，无意中违反了这个要求的情况不难想象。考虑向超类添加新的值组件时，子类的情况。换句话说，子类添加了一条影响 equals 比较的信息。让我们从一个简单的不可变二维整数点类开始：

```
public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;
        Point p = (Point)o;
        return p.x == x && p.y == y;
    }
    ... // Remainder omitted
}
```

Suppose you want to extend this class, adding the notion of color to a point:

假设你想继承这个类，对一个点添加颜色的概念：

```
public class ColorPoint extends Point {
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }
    ... // Remainder omitted
}
```

How should the equals method look? If you leave it out entirely, the implementation is inherited from Point and color information is ignored in equals comparisons. While this does not violate the equals contract, it is clearly unacceptable. Suppose you write an equals method that returns true only if its argument is another color point with the same position and color:

equals 方法应该是什么样子？如果你完全忽略它，则实现将从 Point 类继承而来，在 equals 比较中颜色信息将被忽略。虽然这并不违反 equals 约定，但显然是不可接受的。假设你写了一个 equals 方法，该方法只有当它的参数是另一个颜色点，且位置和颜色相同时才返回 true：

```
// Broken - violates symmetry!
@Override
public boolean equals(Object o) {
    if (!(o instanceof ColorPoint))
        return false;
    return super.equals(o) && ((ColorPoint) o).color == color;
}
```

The problem with this method is that you might get different results when comparing a point to a color point and vice versa. The former comparison ignores color, while the latter comparison always returns false because the type of the argument is incorrect. To make this concrete, let’s create one point and one color point:

这种方法的问题是，当你比较一个点和一个颜色点时，你可能会得到不同的结果，反之亦然。前者比较忽略颜色，而后者比较总是返回 false，因为参数的类型是不正确的。为了使问题更具体，让我们创建一个点和一个颜色点：

```
Point p = new Point(1, 2);
ColorPoint cp = new ColorPoint(1, 2, Color.RED);
```

Then p.equals(cp) returns true, while cp.equals(p) returns false. You might try to fix the problem by having ColorPoint.equals ignore color when doing “mixed comparisons”:

然后，`p.equals(cp)` 返回 true，而 `cp.equals(p)` 返回 false。当你做「混合比较」的时候，你可以通过让 `ColorPoint.equals` 忽略颜色来解决这个问题：

```
// Broken - violates transitivity!
@Override
public boolean equals(Object o) {
    if (!(o instanceof Point))
        return false;

    // If o is a normal Point, do a color-blind comparison
    if (!(o instanceof ColorPoint))
        return o.equals(this);

    // o is a ColorPoint; do a full comparison
    return super.equals(o) && ((ColorPoint) o).color == color;
}
```

This approach does provide symmetry, but at the expense of transitivity:

这种方法确实提供了对称性，但牺牲了传递性：

```
ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
Point p2 = new Point(1, 2);
ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
```

Now p1.equals(p2) and p2.equals(p3) return true, while p1.equals(p3) returns false, a clear violation of transitivity. The first two comparisons are “color-blind,” while the third takes color into account.

现在，`p1.equals(p2)` 和 `p2.equals(p3)` 返回 true，而 `p1.equals(p3)` 返回 false，这明显违反了传递性。前两个比较是「色盲」，而第三个比较考虑了颜色。

Also, this approach can cause infinite recursion: Suppose there are two subclasses of Point, say ColorPoint and SmellPoint, each with this sort of equals method. Then a call to myColorPoint.equals(mySmellPoint) will throw a StackOverflowError.

同样，这种方法会导致无限的递归：假设有两个点的子类，比如 ColorPoint 和 SmellPoint，每个都使用这种 equals 方法。然后调用 `myColorPoint.equals(mySmellPoint)` 会抛出 StackOverflowError。

So what’s the solution? It turns out that this is a fundamental problem of equivalence relations in object-oriented languages. **There is no way to extend an instantiable class and add a value component while preserving the equals contract,** unless you’re willing to forgo the benefits of object-oriented abstraction.

那么解决方案是什么？这是面向对象语言中等价关系的一个基本问题。**除非你愿意放弃面向对象的抽象优点，否则无法继承一个可实例化的类并添加一个值组件，同时保留 equals 约定。**

You may hear it said that you can extend an instantiable class and add a value component while preserving the equals contract by using a getClass test in place of the instanceof test in the equals method:

你可能会听到它说你可以继承一个实例化的类并添加一个值组件，同时通过在 equals 方法中使用 getClass 测试来代替 instanceof 测试来保持 equals 约定：

```
// Broken - violates Liskov substitution principle (page 43)
@Override
public boolean equals(Object o) {

    if (o == null || o.getClass() != getClass())
        return false;

    Point p = (Point) o;
    return p.x == x && p.y == y;
}
```

This has the effect of equating objects only if they have the same implementation class. This may not seem so bad, but the consequences are unacceptable: An instance of a subclass of Point is still a Point, and it still needs to function as one, but it fails to do so if you take this approach! Let’s suppose we want to write a method to tell whether a point is on the unit circle. Here is one way we could do it:

只有当对象具有相同的实现类时，才会产生相等的效果。这可能看起来不是很糟糕，但其后果是不可接受的：Point 的子类的实例仍然是一个 Point，并且它仍然需要作为一个函数来工作，但是如果采用这种方法，它就不会这样做！假设我们要写一个方法来判断一个点是否在单位圆上。我们可以这样做：

```
// Initialize unitCircle to contain all Points on the unit circle
private static final Set<Point> unitCircle = Set.of(
        new Point( 1, 0), new Point( 0, 1),
        new Point(-1, 0), new Point( 0, -1)
    );

    public static boolean onUnitCircle(Point p) {
        return unitCircle.contains(p);
}
```

While this may not be the fastest way to implement the functionality, it works fine. Suppose you extend Point in some trivial way that doesn’t add a value component, say, by having its constructor keep track of how many instances have been created:

虽然这可能不是实现功能的最快方法，但它工作得很好。假设你以一种不添加值组件的简单方式继承 Point，例如，让它的构造函数跟踪创建了多少实例：

```
public class CounterPoint extends Point {
    private static final AtomicInteger counter = new AtomicInteger();

    public CounterPoint(int x, int y) {
        super(x, y);
        counter.incrementAndGet();
    }

    public static int numberCreated() {
        return counter.get();
    }
}
```

The Liskov substitution principle says that any important property of a type should also hold for all its subtypes so that any method written for the type should work equally well on its subtypes [Liskov87]. This is the formal statement of our earlier claim that a subclass of Point (such as CounterPoint) is still a Point and must act as one. But suppose we pass a CounterPoint to the onUnitCircle method. If the Point class uses a getClass-based equals method, the onUnitCircle method will return false regardless of the CounterPoint instance’s x and y coordinates. This is so because most collections, including the HashSet used by the onUnitCircle method, use the equals method to test for containment, and no CounterPoint instance is equal to any Point. If, however, you use a proper instanceof-based equals method on Point, the same onUnitCircle method works fine when presented with a CounterPoint instance.

Liskov 替换原则指出，类型的任何重要属性都应该适用于所有子类型，因此为类型编写的任何方法都应该在其子类型上同样有效 [Liskov87]。这是我们先前做的正式声明，即点的子类（如 CounterPoint）仍然是一个 Point，并且必须作为一个 Point。但假设我们传递了一个 CounterPoint 给 onUnitCircle 方法。如果 Point 类使用基于 getclass 的 equals 方法，那么不管 CounterPoint 实例的 x 和 y 坐标如何，onUnitCircle 方法都会返回 false。这是因为大多数集合，包括 onUnitCircle 方法使用的 HashSet，都使用 equals 方法来测试包含性，没有一个 CounterPoint 实例等于任何一个点。但是，如果你在 Point 上使用了正确的基于实例的 equals 方法，那么在提供对位实例时，相同的 onUnitCircle 方法就可以很好地工作。

**译注：里氏替换原则（Liskov Substitution Principle，LSP）面向对象设计的基本原则之一。里氏替换原则指出：任何父类可以出现的地方，子类一定可以出现。LSP 是继承复用的基石，只有当衍生类可以替换掉父类，软件单位的功能不受到影响时，父类才能真正被复用，而衍生类也能够在父类的基础上增加新的行为。**

While there is no satisfactory way to extend an instantiable class and add a value component, there is a fine workaround: Follow the advice of Item 18,“Favor composition over inheritance.” Instead of having ColorPoint extend Point, give ColorPoint a private Point field and a public view method (Item 6) that returns the point at the same position as this color point:

虽然没有令人满意的方法来继承一个可实例化的类并添加一个值组件，但是有一个很好的解决方案：遵循 [Item-18](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-4/Chapter-4-Item-18-Favor-composition-over-inheritance.md) 的建议，「Favor composition over inheritance.」。给 ColorPoint 一个私有的 Point 字段和一个 public 视图方法（[Item-6](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-2/Chapter-2-Item-6-Avoid-creating-unnecessary-objects.md)），而不是让 ColorPoint 继承 Point，该方法返回与这个颜色点相同位置的点：

```
// Adds a value component without violating the equals contract
public class ColorPoint {
    private final Point point;
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
    }

    /**
    * Returns the point-view of this color point.
    */
    public Point asPoint() {
        return point;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
            return false;

        ColorPoint cp = (ColorPoint) o;
        return cp.point.equals(point) && cp.color.equals(color);
    }
    ... // Remainder omitted
}
```

There are some classes in the Java platform libraries that do extend an instantiable class and add a value component. For example,java.sql.Timestamp extends java.util.Date and adds a nanoseconds field. The equals implementation for Timestamp does violate symmetry and can cause erratic behavior if Timestamp and Date objects are used in the same collection or are otherwise intermixed. The Timestamp class has a disclaimer cautioning programmers against mixing dates and timestamps. While you won’t get into trouble as long as you keep them separate, there’s nothing to prevent you from mixing them, and the resulting errors can be hard to debug. This behavior of the Timestamp class was a mistake and should not be emulated.

Java 库中有一些类确实继承了一个可实例化的类并添加了一个值组件。例如，`java.sql.Timestamp` 继承 `java.util.Date` 并添加了纳秒字段。如果在同一个集合中使用时间戳和日期对象，或者以其他方式混合使用时间戳和日期对象，那么时间戳的 equals 实现确实违反了对称性，并且可能导致不稳定的行为。Timestamp 类有一个免责声明，警告程序员不要混合使用日期和时间戳。虽然只要将它们分开，就不会遇到麻烦，但是没有什么可以阻止你将它们混合在一起，因此产生的错误可能很难调试。时间戳类的这种行为是错误的，不应该效仿。

Note that you can add a value component to a subclass of an abstract class without violating the equals contract. This is important for the sort of class hierarchies that you get by following the advice in Item 23, “Prefer class hierarchies to tagged classes.” For example, you could have an abstract class Shape with no value components, a subclass Circle that adds a radius field, and a subclass Rectangle that adds length and width fields.Problems of the sort shown earlier won’t occur so long as it is impossible to create a superclass instance directly.

注意，你可以向抽象类的子类添加一个值组件，而不违反 equals 约定。这对于遵循 [Item-23](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-4/Chapter-4-Item-23-Prefer-class-hierarchies-to-tagged-classes.md) 中的建议而得到的类层次结构很重要，「Prefer class hierarchies to tagged classes.」。例如，可以有一个没有值组件的抽象类形状、一个添加半径字段的子类圆和一个添加长度和宽度字段的子类矩形。只要不可能直接创建超类实例，前面显示的那种问题就不会发生。

**Consistency—** The fourth requirement of the equals contract says that if two objects are equal, they must remain equal for all time unless one (or both) of them is modified. In other words, mutable objects can be equal to different objects at different times while immutable objects can’t. When you write a class,think hard about whether it should be immutable (Item 17). If you conclude that it should, make sure that your equals method enforces the restriction that equal objects remain equal and unequal objects remain unequal for all time.

**一致性** ，对等约定的第四个要求是，如果两个对象相等，它们必须一直保持相等，除非其中一个（或两个）被修改。换句话说，可变对象可以等于不同时间的不同对象，而不可变对象不能。在编写类时，仔细考虑它是否应该是不可变的（[Item-17](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-4/Chapter-4-Item-17-Minimize-mutability.md)）。如果你认为应该这样做，那么请确保你的 equals 方法执行了这样的限制，即相等的对象始终是相等的，而不等的对象始终是不等的。

Whether or not a class is immutable, **do not write an equals method that depends on unreliable resources.** It’s extremely difficult to satisfy the consistency requirement if you violate this prohibition. For example,java.net.URL’s equals method relies on comparison of the IP addresses of the hosts associated with the URLs. Translating a host name to an IP address can require network access, and it isn’t guaranteed to yield the same results over time. This can cause the URL equals method to violate the equals contract and has caused problems in practice. The behavior of URL’s equals method was a big mistake and should not be emulated. Unfortunately, it cannot be changed due to compatibility requirements. To avoid this sort of problem,equals methods should perform only deterministic computations on memoryresident objects.

无论一个类是否不可变，都不要编写依赖于不可靠资源的 equals 方法。如果你违反了这个禁令，就很难满足一致性要求。例如，`java.net.URL` 的 equals 方法依赖于与 url 相关联的主机的 IP 地址的比较。将主机名转换为 IP 地址可能需要网络访问，而且不能保证随着时间的推移产生相同的结果。这可能会导致 URL 的 equals 方法违反约定，并在实践中造成问题。URL 的 equals 方法的行为是一个很大的错误，不应该被模仿。不幸的是，由于兼容性需求，它不能更改。为了避免这种问题，equals 方法应该只对 memoryresident 对象执行确定性计算。

**Non-nullity—** The final requirement lacks an official name, so I have taken the liberty of calling it “non-nullity.” It says that all objects must be unequal to null. While it is hard to imagine accidentally returning true in response to the invocation o.equals(null), it isn’t hard to imagine accidentally throwing a NullPointerException. The general contract prohibits this.Many classes have equals methods that guard against it with an explicit test for null:

**非无效性** ，最后的要求没有一个正式的名称，所以我冒昧地称之为「非无效性」。它说所有对象都不等于 null。虽然很难想象在响应调用 `o.equals(null)` 时意外地返回 true，但不难想象意外地抛出 NullPointerException。一般约定中禁止这样做。许多类都有相等的方法，通过显式的 null 测试来防止它：

```
@Override
public boolean equals(Object o) {
    if (o == null)
        return false;
    ...
}
```

This test is unnecessary. To test its argument for equality, the equals method must first cast its argument to an appropriate type so its accessors can be invoked or its fields accessed. Before doing the cast, the method must use the instanceof operator to check that its argument is of the correct type:

这个测试是不必要的。要测试其参数是否相等，equals 方法必须首先将其参数转换为适当的类型，以便能够调用其访问器或访问其字段。在执行转换之前，方法必须使用 instanceof 运算符来检查其参数的类型是否正确：

```
@Override
public boolean equals(Object o) {
    if (!(o instanceof MyType))
        return false;
    MyType mt = (MyType) o;
    ...
}
```

If this type check were missing and the equals method were passed an argument of the wrong type, the equals method would throw a ClassCastException, which violates the equals contract. But the instanceof operator is specified to return false if its first operand is null,regardless of what type appears in the second operand [JLS, 15.20.2]. Therefore,the type check will return false if null is passed in, so you don’t need an explicit null check.

如果缺少这个类型检查，并且 equals 方法传递了一个错误类型的参数，equals 方法将抛出 ClassCastException，这违反了 equals 约定。但是，如果 instanceof 操作符的第一个操作数为空，则指定该操作符返回 false，而不管第二个操作数 [JLS, 15.20.2] 中出现的是什么类型。因此，如果传入 null，类型检查将返回 false，因此不需要显式的 null 检查。

Putting it all together, here’s a recipe for a high-quality equals method:

综上所述，这里有一个高质量构建 equals 方法的秘诀：

1、**Use the == operator to check if the argument is a reference to this object.** If so, return true. This is just a performance optimization but one that is worth doing if the comparison is potentially expensive.

**使用 == 运算符检查参数是否是对该对象的引用。** 如果是，返回 true。这只是一种性能优化，但如果比较的代价可能很高，那么这种优化是值得的。

2、**Use the instanceof operator to check if the argument has the correct type.** If not, return false. Typically, the correct type is the class in which the method occurs. Occasionally, it is some interface implemented by this class. Use an interface if the class implements an interface that refines the equals contract to permit comparisons across classes that implement the interface. Collection interfaces such as Set, List, Map, and Map.Entry have this property.

**使用 instanceof 运算符检查参数是否具有正确的类型。** 如果不是，返回 false。通常，正确的类型是方法发生的类。有时候，它是由这个类实现的某个接口。如果类实现了一个接口，该接口对 equals 约定进行了改进，以允许跨实现该接口的类进行比较，则使用该接口。集合接口，如 Set、List、Map 和 Map.Entry 具有此属性。

3、**Cast the argument to the correct type.** Because this cast was preceded by an instanceof test, it is guaranteed to succeed.

**将参数转换为正确的类型。** 因为在这个强制类型转换之前有一个实例测试，所以它肯定会成功。

4、**For each “significant” field in the class, check if that field of the argument matches the corresponding field of this object.** If all these tests succeed, return true; otherwise, return false. If the type in Step 2 is an interface, you must access the argument’s fields via interface methods; if the type is a class, you may be able to access the fields directly, depending on their accessibility.

**对于类中的每个「重要」字段，检查参数的字段是否与该对象的相应字段匹配。** 如果所有这些测试都成功，返回 true；否则返回 false。如果第 2 步中的类型是接口，则必须通过接口方法访问参数的字段；如果是类，你可以根据字段的可访问性直接访问它们。

For primitive fields whose type is not float or double, use the == operator for comparisons; for object reference fields, call the equals method recursively; for float fields, use the static Float.compare(float,float) method; and for double fields, use Double.compare(double, double). The special treatment of float and double fields is made necessary by the existence of Float.NaN, -0.0f and the analogous double values; see JLS 15.21.1 or the documentation of Float.equals for details. While you could compare float and double fields with the static methods Float.equals and Double.equals, this would entail autoboxing on every comparison, which would have poor performance. For array fields, apply these guidelines to each element. If every element in an array field is significant, use one of the Arrays.equals methods.

对于类型不是 float 或 double 的基本类型字段，使用 == 运算符进行比较；对于对象引用字段，递归调用 equals 方法；对于 float 字段，使用 `static Float.compare(float,float)` 方法；对于 double 字段，使用 `Double.compare(double, double)`。float 和 double 字段的特殊处理是由于 `Float.NaN`、-0.0f 和类似的双重值的存在而必须的；请参阅 JLS 15.21.1 或 `Float.equals` 文档。虽然你可以将 float 和 double 字段与静态方法 Float.equals 和 Double.equals 进行比较，这将需要在每个比较上进行自动装箱，这将有较差的性能。对于数组字段，将这些指导原则应用于每个元素。如果数组字段中的每个元素都很重要，那么使用 `Arrays.equals` 方法之一。

Some object reference fields may legitimately contain null. To avoid the possibility of a NullPointerException, check such fields for equality using the static method Objects.equals(Object, Object).

一些对象引用字段可能合法地包含 null。为了避免可能出现 NullPointerException，请使用静态方法 `Objects.equals(Object, Object)` 检查这些字段是否相等。

For some classes, such as CaseInsensitiveString above, field comparisons are more complex than simple equality tests. If this is the case,you may want to store a canonical form of the field so the equals method can do a cheap exact comparison on canonical forms rather than a more costly nonstandard comparison. This technique is most appropriate for immutable classes (Item 17); if the object can change, you must keep the canonical form up to date.

对于某些类，例如上面的 CaseInsensitiveString，字段比较比简单的 equal 测试更复杂。如果是这样，你可能希望存储字段的规范形式，以便 equals 方法可以对规范形式进行廉价的精确比较，而不是更昂贵的非标准比较。这种技术最适合于不可变类（[Item-17](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-4/Chapter-4-Item-17-Minimize-mutability.md)）；如果对象可以更改，则必须使规范形式保持最新。

The performance of the equals method may be affected by the order in which fields are compared. For best performance, you should first compare fields that are more likely to differ, less expensive to compare, or, ideally,both. You must not compare fields that are not part of an object’s logical state,such as lock fields used to synchronize operations. You need not compare derived fields, which can be calculated from “significant fields,” but doing so may improve the performance of the equals method. If a derived field amounts to a summary description of the entire object, comparing this field will save you the expense of comparing the actual data if the comparison fails.For example, suppose you have a Polygon class, and you cache the area. If two polygons have unequal areas, you needn’t bother comparing their edges and vertices.

equals 方法的性能可能会受到字段比较顺序的影响。为了获得最佳性能，你应该首先比较那些更可能不同、比较成本更低的字段，或者理想情况下两者都比较。不能比较不属于对象逻辑状态的字段，例如用于同步操作的锁字段。你不需要比较派生字段（可以从「重要字段」计算），但是这样做可能会提高 equals 方法的性能。如果派生字段相当于整个对象的摘要描述，那么如果比较失败，比较该字段将节省比较实际数据的开销。例如，假设你有一个多边形类，你缓存这个区域。如果两个多边形的面积不相等，你不需要比较它们的边和顶点。

**When you are finished writing your equals method, ask yourself three questions: Is it symmetric? Is it transitive? Is it consistent?** And don’t just ask yourself; write unit tests to check, unless you used AutoValue (page 49) to generate your equals method, in which case you can safely omit the tests. If the properties fail to hold, figure out why, and modify the equals method accordingly. Of course your equals method must also satisfy the other two properties (reflexivity and non-nullity), but these two usually take care of themselves.

**写完 equals 方法后，问自己三个问题：它具备对称性吗？具备传递性吗？具备一致性吗？** 不要只问自己，要编写单元测试来检查，除非使用 AutoValue（第 49 页）来生成 equals 方法，在这种情况下，你可以安全地省略测试。如果属性不能保持，请找出原因，并相应地修改 equals 方法。当然，equals 方法还必须满足其他两个属性（反身性和非无效性），但这两个通常会自己处理。

An equals method constructed according to the previous recipe（n.食谱，配方） is shown in this simplistic PhoneNumber class:

在这个简单的 PhoneNumber 类中，根据前面的方法构造了一个 equals 方法：

```
// Class with a typical equals method
public final class PhoneNumber {
    private final short areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = rangeCheck(areaCode, 999, "area code");
        this.prefix = rangeCheck(prefix, 999, "prefix");
        this.lineNum = rangeCheck(lineNum, 9999, "line num");
    }

    private static short rangeCheck(int val, int max, String arg) {
        if (val < 0 || val > max)
            throw new IllegalArgumentException(arg + ": " + val);
        return (short) val;
    }

    @Override
    public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof PhoneNumber))
            return false;

        PhoneNumber pn = (PhoneNumber)o;
        return pn.lineNum == lineNum && pn.prefix == prefix && pn.areaCode == areaCode;
    } ... // Remainder omitted
}
```

Here are a few final caveats:

以下是一些最后的警告：

- **Always override hashCode when you override equals (Item 11).**

**当你覆盖 equals 时，也覆盖 hashCode。**（[Item-11](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-3/Chapter-3-Item-11-Always-override-hashCode-when-you-override-equals.md)）

- **Don’t try to be too clever.** If you simply test fields for equality, it’s not hard to adhere to the equals contract. If you are overly aggressive in searching for equivalence, it’s easy to get into trouble. It is generally a bad idea to take any form of aliasing into account. For example, the File class shouldn’t attempt to equate symbolic links referring to the same file. Thankfully, it doesn’t.

**不要自作聪明。** 如果你只是为了判断相等性而测试字段，那么遵循 equals 约定并不困难。如果你在寻求对等方面过于激进，很容易陷入麻烦。一般来说，考虑到任何形式的混叠都不是一个好主意。例如，File 类不应该尝试将引用同一文件的符号链接等同起来。值得庆幸的是，它不是。

- **Don’t substitute another type for Object in the equals declaration.** It is not uncommon for a programmer to write an equals method that looks like this and then spend hours puzzling over why it doesn’t work properly:

**不要用另一种类型替换 equals 声明中的对象。** 对于程序员来说，编写一个类似于这样的 equals 方法，然后花上几个小时思考为什么它不能正常工作是很常见的：

```
// Broken - parameter type must be Object!
public boolean equals(MyClass o) {
    ...
}
```

The problem is that this method does not override Object.equals,whose argument is of type Object, but overloads it instead (Item 52). It is unacceptable to provide such a “strongly typed” equals method even in addition to the normal one, because it can cause Override annotations in subclasses to generate false positives and provide a false sense of security.

这里的问题是，这个方法没有覆盖其参数类型为 Object 的 Object.equals，而是重载了它（[Item-52](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-8/Chapter-8-Item-52-Use-overloading-judiciously.md)）。即使是普通的方法，提供这样一个「强类型的」equals 方法是不可接受的，因为它会导致子类中的重写注释产生误报并提供错误的安全性。

Consistent use of the Override annotation, as illustrated throughout this item, will prevent you from making this mistake (Item 40). This equals method won’t compile, and the error message will tell you exactly what is wrong:

如本条目所示，一致使用 Override 注释将防止你犯此错误（[Item-40](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-6/Chapter-6-Item-40-Consistently-use-the-Override-annotation.md)）。这个 equals 方法不会编译，错误消息会告诉你什么是错误的：

```
// Still broken, but won’t compile
@Override
public boolean equals(MyClass o) {
    ...
}
```

Writing and testing equals (and hashCode) methods is tedious, and the resulting code is mundane. An excellent alternative to writing and testing these methods manually is to use Google’s open source AutoValue framework, which automatically generates these methods for you, triggered by a single annotation on the class . In most cases, the methods generated by AutoValue are essentially identical to those you’d write yourself.

编写和测试 equals （和 hashCode）方法很乏味，生成的代码也很单调。手动编写和测试这些方法的一个很好的替代方法是使用谷歌的开源 AutoValue 框架，它会自动为你生成这些方法，由类上的一个注释触发。在大多数情况下，AutoValue 生成的方法与你自己编写的方法基本相同。

IDEs, too, have facilities to generate equals and hashCode methods, but the resulting source code is more verbose and less readable than code that uses AutoValue, does not track changes in the class automatically, and therefore requires testing. That said, having IDEs generate equals (and hashCode)methods is generally preferable to implementing them manually because IDEs do not make careless mistakes, and humans do.

IDE 也有生成 equals 和 hashCode 方法的功能，但是生成的源代码比使用 AutoValue 的代码更冗长，可读性更差，不会自动跟踪类中的变化，因此需要进行测试。也就是说，让 IDE 生成 equals（和 hashCode）方法通常比手动实现更可取，因为 IDE 不会出现粗心的错误，而人会。

In summary, don’t override the equals method unless you have to: in many cases, the implementation inherited from Object does exactly what you want.If you do override equals, make sure to compare all of the class’s significant fields and to compare them in a manner that preserves all five provisions of the equals contract.

总之，除非必须，否则不要覆盖 equals 方法：在许多情况下，从 Object 继承而来的实现正是你想要的。如果你确实覆盖了 equals，那么一定要比较类的所有重要字段，并以保留 equals 约定的所有 5 项规定的方式进行比较。
