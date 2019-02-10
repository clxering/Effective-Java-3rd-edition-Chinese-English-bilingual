## Chapter 4. Classes and Interfaces（类和接口）

### Item 17: Minimize mutability（减少可变性）

An immutable（adj. 不变的；不可变的；不能变的） class is simply a class whose instances cannot be modified（v. 修改；adj. 改进的，修改的）. All of the information contained in each instance is fixed for the lifetime of the object,so no changes can ever be observed. The Java platform libraries contain many immutable classes, including String, the boxed primitive classes, and BigInteger and BigDecimal. There are many good reasons for this:Immutable classes are easier to design, implement, and use than mutable classes.They are less prone（adj. 俯卧的；有…倾向的，易于…的） to error and are more secure.

不可变类就是一个实例不能被修改的类。每个实例中包含的所有信息在对象的生命周期内都是固定的，因此永远不会观察到任何更改。Java 平台库包含许多不可变的类，包括String、基本数据类型的包装类、BigInteger和 BigDecimal。有很多很好的理由：不可变类比可变类更容易设计、实现和使用。它们不太容易出错，而且更安全。

To make a class immutable, follow these five rules:

要使类不可变，请遵循以下 5 条规则:

1. **Don’t provide methods that modify the object’s state** (known as mutators).

**不要提供修改对象状态的方法**（被称为调整器）

2. **Ensure that the class can’t be extended.** This prevents careless or malicious（adj. 恶意的；恶毒的；蓄意的；怀恨的） subclasses from compromising（adj. 妥协的；让步的） the immutable behavior of the class by behaving as if the object’s state has changed. Preventing subclassing is generally accomplished by making the class final, but there is an alternative that we’ll discuss later.

**确保类不能被扩展。** 这可以防止粗心或恶意的子类以对象状态发生改变的方式行为，从而损害类的不可变行为。防止子类化通常通过使类成为 final 来完成，但是还有一种替代方法，我们将在后面讨论。

3. **Make all fields final.** This clearly expresses your intent in a manner that is enforced（v. 执行） by the system. Also, it is necessary to ensure correct behavior if a reference to a newly created instance is passed from one thread to another without synchronization, as spelled out in the memory model [JLS, 17.5;Goetz06, 16].

**使所有字段为终态。** 这清楚地表达了您的意图，方式是由系统执行的。同样，如果引用新创建的实例在没有同步的情况下从一个线程传递到另一个线程，那么就有必要确保正确的行为，就像内存模型中描述的那样[JLS, 17.5;Goetz06, 16]。

4. **Make all fields private.** This prevents clients from obtaining access to mutable objects referred to by fields and modifying these objects directly.While it is technically permissible for immutable classes to have public final fields containing primitive values or references to immutable objects, it is not recommended because it precludes changing the internal representation in a later release (Items 15 and 16).

**使所有字段为私有。** 这将阻止客户端访问字段引用的可变对象并直接修改这些对象。虽然在技术上允许不可变类拥有包含基元值或对不可变对象的引用的公共 final 字段，但不建议这样做，因为它排除了在以后的版本中更改内部表示([Item-15](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-4/Chapter-4-Item-15-Minimize-the-accessibility-of-classes-and-members.md)和[Item-16](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-4/Chapter-4-Item-16-In-public-classes-use-accessor-methods-not-public-fields.md))。

5. **Ensure exclusive access to any mutable components.** If your class has any fields that refer to mutable objects, ensure that clients of the class cannot obtain references to these objects. Never initialize such a field to a clientprovided object reference or return the field from an accessor. Make defensive copies (Item 50) in constructors, accessors, and readObject methods (Item 88).

**确保对任何可变组件的独占访问。** 如果您的类有任何引用可变对象的字段，请确保该类的客户端无法获得对这些对象的引用。永远不要将这样的字段初始化为 clientprovide 对象引用或从访问器返回字段。在构造函数、访问器和 readObject 方法（[Item-88](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-12/Chapter-12-Item-88-Write-readObject-methods-defensively.md)）中创建防御性副本（[Item-50](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-8/Chapter-8-Item-50-Make-defensive-copies-when-needed.md)）。

Many of the example classes in previous items are immutable. One such class is PhoneNumber in Item 11, which has accessors for each attribute but no corresponding mutators. Here is a slightly more complex example:

前面项目中的许多示例类都是不可变的。其中一个类是[Item-11](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-3/Chapter-3-Item-11-Always-override-hashCode-when-you-override-equals.md)中的 PhoneNumber，它对每个属性都有访问器，但没有相应的 mutators。下面是一个稍微复杂一点的例子:

```
// Immutable complex number class
public final class Complex {
    private final double re;
    private final double im;

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }
    public double realPart() { return re; }
    public double imaginaryPart() { return im; }
    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }
    public Complex minus(Complex c) {
        return new Complex(re - c.re, im - c.im);
    }
    public Complex times(Complex c) {
        return new Complex(re * c.re - im * c.im,re * c.im + im * c.re);
    }
    public Complex dividedBy(Complex c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re * c.re + im * c.im) / tmp, (im * c.re - re * c.im) / tmp);
    }

    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Complex))
            return false;
        Complex c = (Complex) o;

        // See page 47 to find out why we use compare instead of ==
        return Double.compare(c.re, re) == 0
                && Double.compare(c.im, im) == 0;
    }

    @Override public int hashCode() {
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }

    @Override public String toString() {
        return "(" + re + " + " + im + "i)";
    }
}
```

This class represents a complex number (a number with both real and imaginary parts). In addition to the standard Object methods, it provides accessors for the real and imaginary parts and provides the four basic arithmetic operations: addition, subtraction, multiplication, and division. Notice how the arithmetic operations create and return a new Complex instance rather than modifying this instance. This pattern is known as the functional approach because methods return the result of applying a function to their operand,without modifying it. Contrast it to the procedural or imperative approach in which methods apply a procedure to their operand, causing its state to change.Note that the method names are prepositions (such as plus) rather than verbs (such as add). This emphasizes the fact that methods don’t change the values of the objects. The BigInteger and BigDecimal classes did not obey this naming convention, and it led to many usage errors.

这个类表示一个复数(一个包含实部和虚部的数)。除了标准的对象方法之外，它还为实部和虚部提供访问器，并提供四种基本的算术运算：加法、减法、乘法和除法。需要注意的是，算术操作如何创建和返回一个新的复杂实例，而不是修改这个实例。这种模式称为泛函方法，因为方法返回对其操作数应用函数的结果，而不修改它。将其与方法将过程应用于其操作数的过程或命令式方法进行对比，使其状态发生变化。注意，方法名是介词(如plus)，而不是动词(如add)。这强调了一个事实，方法不会改变对象的值。BigInteger 和 BigDecimal 类不遵守这个命名约定，导致了许多使用错误。

The functional approach（n. 方法；途径；接近） may appear unnatural if you’re not familiar（adj. 熟悉的；常见的；亲近的） with it,but it enables immutability, which has many advantages. **Immutable objects are simple.** An immutable object can be in exactly one state, the state in which it was created. If you make sure that all constructors establish class invariants, then it is guaranteed that these invariants will remain true for all time, with no further effort on your part or on the part of the programmer who uses the class. Mutable objects, on the other hand, can have arbitrarily complex state spaces. If the documentation does not provide a precise description of the state transitions performed by mutator methods, it can be difficult or impossible to use a mutable class reliably.

如果您不熟悉函数方法，那么它可能看起来不自然，但是它支持不变性，这有很多优点。**不可变对象很简单。** 一个不可变的对象可以恰好处于一种状态，即创建它的状态。如果您确保所有构造函数都建立了类不变量，那么就可以保证这些不变量将一直保持为真，而您和使用该类的程序员无需再做任何努力。另一方面，可变对象可以具有任意复杂的状态空间。如果文档没有提供 mutator 方法执行的状态转换的精确描述，那么可能很难或不可能可靠地使用可变类。

**Immutable objects are inherently thread-safe; they require no synchronization.** They cannot be corrupted by multiple threads accessing them concurrently. This is far and away the easiest approach to achieve thread safety.Since no thread can ever observe any effect of another thread on an immutable object, **immutable objects can be shared freely.** Immutable classes should therefore encourage clients to reuse existing instances wherever possible. One easy way to do this is to provide public static final constants for commonly used values. For example, the Complex class might provide these constants:

不可变对象本质上是线程安全的；它们不需要同步。它们不能被多线程并发地访问而损坏。这无疑是实现线程安全的最简单方法。由于任何线程都无法观察到另一个线程对不可变对象的任何影响，因此 **可以自由共享不可变对象。** 因此，不可变类应该鼓励客户尽可能重用现有的实例。一种简单的方法是为常用值提供公共静态最终常量。例如，复杂类可能提供以下常量:

```
public static final Complex ZERO = new Complex(0, 0);
public static final Complex ONE = new Complex(1, 0);
public static final Complex I = new Complex(0, 1);
```

This approach can be taken one step further. An immutable class can provide static factories (Item 1) that cache frequently requested instances to avoid creating new instances when existing ones would do. All the boxed primitive classes and BigInteger do this. Using such static factories causes clients to share instances instead of creating new ones, reducing memory footprint and garbage collection costs. Opting for static factories in place of public constructors when designing a new class gives you the flexibility to add caching later, without modifying clients.

这种方法可以更进一步。不可变类可以提供静态工厂（[Item-1](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-2/Chapter-2-Item-1-Consider-static-factory-methods-instead-of-constructors.md)），这些工厂缓存经常请求的实例，以避免在现有实例可用时创建新实例。所有包装类和 BigInteger 都是这样做的。使用这种静态工厂会导致客户机共享实例而不是创建新实例，从而减少内存占用和垃圾收集成本。在设计新类时，选择静态工厂而不是公共构造函数，这使您能够灵活地在以后添加缓存，而无需修改客户机。

A consequence（n. 结果；重要性；推论） of the fact that immutable objects can be shared freely is that you never have to make defensive copies of them (Item 50). In fact, you never have to make any copies at all because the copies would be forever equivalent to the originals. Therefore, you need not and should not provide a clone method or copy constructor (Item 13) on an immutable class. This was not well understood in the early days of the Java platform, so the String class does have a copy constructor, but it should rarely, if ever, be used (Item 6).

不可变对象可以自由共享这一事实的一个后果是，您永远不需要对它们进行防御性的复制（[Item-50](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-8/Chapter-8-Item-50-Make-defensive-copies-when-needed.md)）。事实上，你根本不需要做任何拷贝，因为拷贝将永远等同于原件。因此，您不需要也不应该在不可变类上提供克隆方法或复制构造函数（[Item-13](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-3/Chapter-3-Item-13-Override-clone-judiciously.md)）。这在 Java 平台的早期并没有得到很好的理解，因此 String 类确实有一个复制构造函数，但是如果有的话，应该很少使用它（[Item-6](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-2/Chapter-2-Item-6-Avoid-creating-unnecessary-objects.md)）。

**Not only can you share immutable objects, but they can share their internals.** For example, the BigInteger class uses a sign-magnitude representation internally. The sign is represented by an int, and the magnitude is represented by an int array. The negate method produces a new BigInteger of like magnitude and opposite sign. It does not need to copy the array even though it is mutable; the newly created BigInteger points to the same internal array as the original.

**您不仅可以共享不可变对象，而且可以共享它们的内部。** 例如，BigInteger 类在内部使用符号幅度表示。符号用 int 表示，星等用 int 数组表示。反求法产生了一个大小相同、符号相反的大整数。即使数组是可变的，它也不需要复制;新创建的 BigInteger 指向与原始的内部数组相同的内部数组。

**Immutable objects make great building blocks for other objects,** whether mutable or immutable. It’s much easier to maintain the invariants of a complex object if you know that its component objects will not change underneath it. A special case of this principle is that immutable objects make great map keys and set elements: you don’t have to worry about their values changing once they’re in the map or set, which would destroy the map or set’s invariants.

**不可变对象是其他对象的很好的构建模块，** 无论是可变的还是不可变的。如果知道复杂对象的组件对象不会在其下面发生更改，那么维护复杂对象的不变性就会容易得多。这个原则的一个特殊情况是，不可变对象会生成很棒的 map 键和 set 元素:您不必担心它们的值在 map 或 set 中发生变化，这会破坏 map 或 set 的不变量。

**Immutable objects provide failure atomicity for free** (Item 76). Their state never changes, so there is no possibility of a temporary inconsistency.

不可变对象免费提供故障原子性（[Item-76](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-10/Chapter-10-Item-76-Strive-for-failure-atomicity.md)）。他们的状态从未改变，所以不可能出现暂时的不一致。

**The major disadvantage of immutable classes is that they require a separate object for each distinct value.** Creating these objects can be costly,especially if they are large. For example, suppose that you have a million-bit BigInteger and you want to change its low-order bit:

不可变类的主要缺点是每个不同的值都需要一个单独的对象。创建这些对象的成本可能很高，尤其是如果对象很大的话。例如，假设你有一个百万比特的大整数，你想改变它的低阶位:

```
BigInteger moby = ...;
moby = moby.flipBit(0);
```

The flipBit method creates a new BigInteger instance, also a million bits long, that differs from the original in only one bit. The operation requires time and space proportional to the size of the BigInteger. Contrast this to java.util.BitSet. Like BigInteger, BitSet represents an arbitrarily long sequence of bits, but unlike BigInteger, BitSet is mutable. The BitSet class provides a method that allows you to change the state of a single bit of a million-bit instance in constant time:

flipBit方法创建了一个新的 BigInteger 实例，也有 100 万比特长，只在一个比特上与原始的不同。该操作需要与 BigInteger 的大小成比例的时间和空间。与java.util.BitSet形成对比。与 BigInteger 一样，BitSet 表示任意长的位序列，但与 BigInteger 不同，BitSet 是可变的。BitSet 类提供了一种方法，可以让你在固定的时间内改变百万比特实例的单个位的状态:

```
BitSet moby = ...;
moby.flip(0);
```

The performance problem is magnified if you perform a multistep operation that generates a new object at every step, eventually discarding all objects except the final result. There are two approaches to coping with this problem. The first is to guess which multistep operations will be commonly required and to provide them as primitives. If a multistep operation is provided as a primitive, the immutable class does not have to create a separate object at each step. Internally,the immutable class can be arbitrarily clever. For example, BigInteger has a package-private mutable “companion class” that it uses to speed up multistep operations such as modular exponentiation. It is much harder to use the mutable companion class than to use BigInteger, for all of the reasons outlined earlier. Luckily, you don’t have to use it: the implementors of BigInteger did the hard work for you.

如果执行多步操作，在每一步生成一个新对象，最终丢弃除最终结果之外的所有对象，那么性能问题就会增大。有两种方法可以解决这个问题。第一种方法是猜测通常需要哪些多步操作，并将它们作为原语提供。如果将多步操作作为基元提供，则不可变类不必在每个步骤中创建单独的对象。在内部，不可变类可以任意聪明。例如，BigInteger 有一个包私有的可变“同伴类”，它使用这个类来加速多步操作，比如模块化求幂。由于前面列出的所有原因，使用可变同伴类要比使用 BigInteger 难得多。幸运的是，您不必使用它:BigInteger的实现者为您做了艰苦的工作。

The package-private mutable companion class approach works fine if you can accurately predict which complex operations clients will want to perform on your immutable class. If not, then your best bet is to provide a public mutable companion class. The main example of this approach in the Java platform libraries is the String class, whose mutable companion is StringBuilder (and its obsolete predecessor, StringBuffer).

如果您能够准确地预测客户机希望在不可变类上执行哪些复杂操作，那么包私有可变伴侣类方法就可以很好地工作。如果不是，那么您最好的选择就是提供一个公共可变伴侣类。这种方法在 Java 平台库中的主要示例是 String 类，它的可变伙伴是StringBuilder(及其过时的前身StringBuffer)。

Now that you know how to make an immutable class and you understand the pros and cons of immutability, let’s discuss a few design alternatives. Recall that to guarantee immutability, a class must not permit itself to be subclassed. This can be done by making the class final, but there is another, more flexible alternative. Instead of making an immutable class final, you can make all of its constructors private or package-private and add public static factories in place of the public constructors (Item 1). To make this concrete, here’s how Complex would look if you took this approach:

既然您已经知道了如何创建不可变类，并且了解了不可变性的优缺点，那么让我们来讨论一些设计方案。回想一下，为了保证不变性，类不允许自己被子类化。这可以通过期末考试来完成，但是还有另外一个更灵活的选择。与使不可变类成为 final 不同，您可以将其所有构造函数变为私有或包-私有，并在公共构造函数的位置添加公共静态工厂（[Item-1](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-2/Chapter-2-Item-1-Consider-static-factory-methods-instead-of-constructors.md)）。

```
// Immutable class with static factories instead of constructors
public class Complex {
    private final double re;
    private final double im;
    private Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }
    public static Complex valueOf(double re, double im) {
        return new Complex(re, im);
    }
    ... // Remainder unchanged
}
```

This approach is often the best alternative. It is the most flexible because it allows the use of multiple package-private implementation classes. To its clients that reside outside its package, the immutable class is effectively final because it is impossible to extend a class that comes from another package and that lacks a public or protected constructor. Besides allowing the flexibility of multiple implementation classes, this approach makes it possible to tune the performance of the class in subsequent releases by improving the object-caching capabilities of the static factories.

这种方法通常是最好的选择。它是最灵活的，因为它允许使用多个包私有实现类。对于驻留在包之外的客户端，不可变类实际上是最终类，因为不可能扩展来自另一个包的类，因为它缺少公共或受保护的构造函数。除了允许多实现类的灵活性之外，这种方法还通过改进静态工厂的对象缓存功能，使在后续版本中调优该类的性能成为可能。

It was not widely understood that immutable classes had to be effectively final when BigInteger and BigDecimal were written, so all of their methods may be overridden. Unfortunately, this could not be corrected after the fact while preserving backward compatibility. If you write a class whose security depends on the immutability of a BigInteger or BigDecimal argument from an untrusted client, you must check to see that the argument is a “real” BigInteger or BigDecimal, rather than an instance of an untrusted subclass. If it is the latter, you must defensively copy it under the assumption that it might be mutable (Item 50):

当编写 BigInteger 和 BigDecimal 时，不可变类必须是有效的 final，因此可以重写它们的所有方法，这一点没有得到广泛的理解。遗憾的是，在保留向后兼容性的情况下，这一问题无法在事后得到纠正。如果您编写的类的安全性依赖于来自不受信任客户端的 BigInteger 或 BigDecimal 参数的不可变性，那么您必须检查该参数是否是“真正的”BigInteger或 BigDecimal，而不是不受信任子类的实例。如果是后者，你必须防御地复制它，假设它可能是可变的（[Item-50](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-8/Chapter-8-Item-50-Make-defensive-copies-when-needed.md)）:

```
public static BigInteger safeInstance(BigInteger val) {
return val.getClass() == BigInteger.class ?
val : new BigInteger(val.toByteArray());
}
```

The list of rules for immutable classes at the beginning of this item says that no methods may modify the object and that all its fields must be final. In fact these rules are a bit stronger than necessary and can be relaxed to improve performance. In truth, no method may produce an externally visible change in the object’s state. However, some immutable classes have one or more nonfinal fields in which they cache the results of expensive computations the first time they are needed. If the same value is requested again, the cached value is returned, saving the cost of recalculation. This trick works precisely because the object is immutable, which guarantees that the computation would yield the same result if it were repeated.

这个项目开头的不可变类的规则列表说，没有方法可以修改对象，它的所有字段必须是 final 的。实际上，这些规则比必要的要强大一些，可以通过放松来提高性能。实际上，任何方法都不能在对象的状态中产生外部可见的更改。然而，一些不可变类有一个或多个非最终字段，它们在第一次需要这些字段时，就会在其中缓存昂贵计算的结果。如果再次请求相同的值，则返回缓存的值，从而节省了重新计算的成本。这个技巧之所以有效，是因为对象是不可变的，这保证了如果重复计算，计算将产生相同的结果。

For example, PhoneNumber’s hashCode method (Item 11, page 53) computes the hash code the first time it’s invoked and caches it in case it’s invoked again. This technique, an example of lazy initialization (Item 83), is also used by String.

例如，PhoneNumber 的 hashCode 方法([Item-11](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-3/Chapter-3-Item-11-Always-override-hashCode-when-you-override-equals.md)，第 53 页)在第一次调用时计算哈希代码，并缓存它，以防再次调用它。这个技术是一个延迟初始化的例子（[Item-83](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-11/Chapter-11-Item-83-Use-lazy-initialization-judiciously.md)），String 也使用这个技术。

One caveat should be added concerning serializability. If you choose to have your immutable class implement Serializable and it contains one or more fields that refer to mutable objects, you must provide an explicit readObject or readResolve method, or use the ObjectOutputStream.writeUnshared and ObjectInputStream.readUnshared methods, even if the default serialized form is acceptable. Otherwise an attacker could create a mutable instance of your class. This topic is covered in detail in Item 88.

关于可序列化性，应该添加一个注意事项。如果您选择让不可变类实现 Serializable，并且它包含一个或多个引用可变对象的字段，那么您必须提供一个显式的 readObject 或 readResolve 方法，或者使用 ObjectOutputStream。writeUnshared ObjectInputStream。readUnshared 方法，即使默认的序列化形式是可以接受的。否则攻击者可能创建类的可变实例。[Item-88](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-12/Chapter-12-Item-88-Write-readObject-methods-defensively.md)详细讨论了这个主题。

To summarize, resist the urge to write a setter for every getter. **Classes should be immutable unless there’s a very good reason to make them mutable.** Immutable classes provide many advantages, and their only disadvantage is the potential for performance problems under certain circumstances. You should always make small value objects, such as PhoneNumber and Complex, immutable. (There are several classes in the Java platform libraries, such as java.util.Date and java.awt.Point, that should have been immutable but aren’t.) You should seriously consider making larger value objects, such as String and BigInteger, immutable as well. You should provide a public mutable companion class for your immutable class only once you’ve confirmed that it’s necessary to achieve satisfactory performance (Item 67).

总结一下，抵制为每个 getter 编写 setter 的冲动。类应该是不可变的，除非有很好的理由让它们可变。不可变类提供了许多优点，它们唯一的缺点是在某些情况下可能出现性能问题。您应该始终创建小的值对象，例如 PhoneNumber 和Complex、stable。(Java平台库中有几个类，比如java.util.Date和java.awt.Point，这本来是不可改变的，但事实并非如此。您应该认真考虑将较大的值对象(如 String 和BigInteger)设置为不可变的。只有在确认了实现满意性能的必要性之后，才应该为不可变类提供一个公共可变伴侣类（[Item-67](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-9/Chapter-9-Item-67-Optimize-judiciously.md)）。

There are some classes for which immutability is impractical. **If a class cannot be made immutable, limit its mutability as much as possible.** Reducing the number of states in which an object can exist makes it easier to reason about the object and reduces the likelihood of errors. Therefore, make every field final unless there is a compelling reason to make it nonfinal. Combining the advice of this item with that of Item 15, your natural inclination should be to **declare every field private final unless there’s a good reason to do otherwise.**

有些类的不变性是不切实际的。**如果一个类不能成为不可变的，那么就尽可能地限制它的可变性。** 减少对象可能存在的状态数可以使对对象进行推理更容易，并减少出错的可能性。因此，除非有令人信服的理由使每个字段成为 final。将此项目的建议与项目 15 的建议结合起来，您的自然倾向应该是 **声明每个字段为私有 final，除非有很好的理由这样做。**

**Constructors should create fully initialized objects with all of their invariants established.** Don’t provide a public initialization method separate from the constructor or static factory unless there is a compelling reason to do so. Similarly, don’t provide a “reinitialize” method that enables an object to be reused as if it had been constructed with a different initial state. Such methods generally provide little if any performance benefit at the expense of increased complexity.

**构造函数应该创建完全初始化的对象，并建立所有的不变量。** 不要提供与构造函数或静态工厂分离的公共初始化方法，除非有充分的理由这样做。类似地，不要提供“重新初始化”的方法，该方法允许重用对象，就好像它是用不同的初始状态构造的一样。这些方法通常只提供很少的性能收益，而代价是增加了复杂性。

The CountDownLatch class exemplifies these principles. It is mutable, but its state space is kept intentionally small. You create an instance, use it once, and it’s done: once the countdown latch’s count has reached zero, you may not reuse it.

CountDownLatch类演示了这些原理。它是可变的，但它的状态空间故意保持小。您创建一个实例，使用一次，就完成了:一旦倒计时锁的计数达到零，您可能不会重用它。

A final note should be added concerning the Complex class in this item. This example was meant only to illustrate immutability. It is not an industrial-strength complex number implementation. It uses the standard formulas for complex multiplication and division, which are not correctly rounded and provide poor semantics for complex NaNs and infinities [Kahan91, Smith62, Thomas94].

关于这个项目中的复杂类，应该添加最后一个注意事项。这个例子只是为了说明不变性。它不是一个工业强度的复数实现。它使用了复杂乘法和除法的标准公式，这些公式没有被正确地四舍五入，并且为复杂的 NaNs 和 infinities 提供了糟糕的语义[Kahan91, Smith62, Thomas94]。
