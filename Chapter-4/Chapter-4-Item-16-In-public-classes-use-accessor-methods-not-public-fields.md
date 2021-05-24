## Chapter 4. Classes and Interfaces（类和接口）

### Item 16: In public classes, use accessor methods, not public fields（在公共类中，使用访问器方法，而不是公共字段）

Occasionally, you may be tempted to write degenerate classes that serve no purpose other than to group instance fields:

有时候，可能会编写一些退化类，这些类除了对实例字段进行分组之外，没有其他用途：

```Java
// Degenerate classes like this should not be public!
class Point {
    public double x;
    public double y;
}
```

Because the data fields of such classes are accessed directly, these classes do not offer the benefits of encapsulation (Item 15). You can’t change the representation without changing the API, you can’t enforce invariants, and you can’t take auxiliary action when a field is accessed. Hard-line object-oriented programmers feel that such classes are anathema and should always be replaced by classes with private fields and public accessor methods (getters) and, for mutable classes, mutators (setters):

因为这些类的数据字段是直接访问的，所以这些类没有提供封装的好处（[Item-15](/Chapter-4/Chapter-4-Item-15-Minimize-the-accessibility-of-classes-and-members.md)）。不改变 API 就不能改变表现形式，不能实现不变量，也不能在访问字段时采取辅助操作。坚持面向对象思维的程序员会认为这样的类是令人厌恶的，应该被使用私有字段和公共访问方法 getter 的类所取代，对于可变类，则是赋值方法 setter：

```Java
// Encapsulation of data by accessor methods and mutators
class Point {
    private double x;
    private double y;
    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }
    public double getX() { return x; }
    public double getY() { return y; }
    public void setX(double x) { this.x = x; }
    public void setY(double y) { this.y = y; }
}
```

Certainly, the hard-liners are correct when it comes to public classes: if a class is accessible outside its package, provide accessor methods to preserve the flexibility to change the class’s internal representation. If a public class exposes its data fields, all hope of changing its representation is lost because client code can be distributed far and wide.

当然，当涉及到公共类时，强硬派是正确的：如果类可以在包之外访问，那么提供访问器方法来保持更改类内部表示的灵活性。如果一个公共类公开其数据字段，那么改变其表示形式的所有希望都将落空，因为客户端代码可以广泛分发。

However, if a class is package-private or is a private nested class, there is nothing inherently wrong with exposing its data fields—assuming they do an adequate job of describing the abstraction provided by the class. This approach generates less visual clutter than the accessor-method approach, both in the class definition and in the client code that uses it. While the client code is tied to the class’s internal representation, this code is confined to the package containing the class. If a change in representation becomes desirable, you can make the change without touching any code outside the package. In the case of a private nested class, the scope of the change is further restricted to the enclosing class.

但是，如果一个类是包级私有的或者是私有嵌套类，那么公开它的数据字段并没有什么本质上的错误（假设它们能够很好地描述类提供的抽象）。无论是在类定义还是在使用它的客户端代码中，这种方法产生的视觉混乱都比访问方法少。虽然客户端代码与类的内部表示绑定在一起，但这段代码仅限于包含该类的包。如果想要对表示形式进行更改，你可以在不接触包外部任何代码的情况下进行更改。对于私有嵌套类，更改的范围进一步限制在封闭类中。

Several classes in the Java platform libraries violate the advice that public classes should not expose fields directly. Prominent examples include the Point and Dimension classes in the java.awt package. Rather than examples to be emulated, these classes should be regarded as cautionary tales.As described in Item 67, the decision to expose the internals of the Dimension class resulted in a serious performance problem that is still with us today.

Java 库中的几个类违反了公共类不应该直接公开字段的建议。突出的例子包括 `java.awt` 包中的 Point 和 Dimension。这些类不应被效仿，而应被视为警示。正如 [Item-67](/Chapter-9/Chapter-9-Item-67-Optimize-judiciously.md) 所述，公开 Dimension 类的内部结构导致了严重的性能问题，这种问题至今仍存在。

While it’s never a good idea for a public class to expose fields directly, it is less harmful if the fields are immutable. You can’t change the representation of such a class without changing its API, and you can’t take auxiliary actions when a field is read, but you can enforce invariants. For example, this class guarantees that each instance represents a valid time:

虽然公共类直接公开字段从来都不是一个好主意，但是如果字段是不可变的，那么危害就会小一些。你不能在不更改该类的 API 的情况下更改该类的表现形式，也不能在读取字段时采取辅助操作，但是你可以实施不变量。例如，这个类保证每个实例代表一个有效的时间：

```Java
// Public class with exposed immutable fields - questionable
public final class Time {
    private static final int HOURS_PER_DAY = 24;
    private static final int MINUTES_PER_HOUR = 60;
    public final int hour;
    public final int minute;

    public Time(int hour, int minute) {
        if (hour < 0 || hour >= HOURS_PER_DAY)
            throw new IllegalArgumentException("Hour: " + hour);
        if (minute < 0 || minute >= MINUTES_PER_HOUR)
            throw new IllegalArgumentException("Min: " + minute);
        this.hour = hour;
        this.minute = minute;
    } ... // Remainder omitted
}
```

In summary, public classes should never expose mutable fields. It is less harmful, though still questionable, for public classes to expose immutable fields.It is, however, sometimes desirable for package-private or private nested classes to expose fields, whether mutable or immutable.

总之，公共类不应该公开可变字段。对于公共类来说，公开不可变字段的危害要小一些，但仍然存在潜在的问题。然而，有时候包级私有或私有嵌套类需要公开字段，无论这个类是可变的还是不可变的。

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-4/Chapter-4-Introduction.md)**
- **Previous Item（上一条目）：[Item 15: Minimize the accessibility of classes and members（尽量减少类和成员的可访问性）](/Chapter-4/Chapter-4-Item-15-Minimize-the-accessibility-of-classes-and-members.md)**
- **Next Item（下一条目）：[Item 17: Minimize mutability（减少可变性）](/Chapter-4/Chapter-4-Item-17-Minimize-mutability.md)**
