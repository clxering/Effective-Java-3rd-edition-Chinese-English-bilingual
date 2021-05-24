## Chapter 12. Serialization（序列化）

### Item 89: For instance control, prefer enum types to readResolve（对于实例控制，枚举类型优于 readResolve）

Item 3 describes the Singleton pattern and gives the following example of a singleton class. This class restricts access to its constructor to ensure that only a single instance is ever created:

[Item-3](/Chapter-2/Chapter-2-Item-3-Enforce-the-singleton-property-with-a-private-constructor-or-an-enum-type.md) 描述了单例模式，并给出了下面的单例类示例。该类限制对其构造函数的访问，以确保只创建一个实例：

```Java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public void leaveTheBuilding() { ... }
}
```

As noted in Item 3, this class would no longer be a singleton if the words implements Serializable were added to its declaration. It doesn’t matter whether the class uses the default serialized form or a custom serialized form (Item 87), nor does it matter whether the class provides an explicit readObject method (Item 88). Any readObject method, whether explicit or default, returns a newly created instance, which will not be the same instance that was created at class initialization time.

如 [Item-3](/Chapter-2/Chapter-2-Item-3-Enforce-the-singleton-property-with-a-private-constructor-or-an-enum-type.md) 所述，如果实现 Serializable 接口，该类将不再是单例的。类使用默认序列化形式还是自定义序列化形式并不重要（[Item-87](/Chapter-12/Chapter-12-Item-87-Consider-using-a-custom-serialized-form.md)），类是否提供显式 readObject 方法也不重要（[Item-88](/Chapter-12/Chapter-12-Item-88-Write-readObject-methods-defensively.md)）。任何 readObject 方法，不管是显式的还是默认的，都会返回一个新创建的实例，这个实例与类初始化时创建的实例不同。

The readResolve feature allows you to substitute another instance for the one created by readObject [Serialization, 3.7]. If the class of an object being deserialized defines a readResolve method with the proper declaration, this method is invoked on the newly created object after it is deserialized. The object reference returned by this method is then returned in place of the newly created object. In most uses of this feature, no reference to the newly created object is retained, so it immediately becomes eligible for garbage collection.

readResolve 特性允许你用另一个实例替换 readObject[Serialization, 3.7] 创建的实例。如果正在反序列化的对象的类定义了 readResolve 方法，新创建的对象反序列化之后，将在该对象上调用该方法。该方法返回的对象引用将代替新创建的对象返回。在该特性的大多数使用中，不保留对新创建对象的引用，因此它立即就有资格进行垃圾收集。

If the Elvis class is made to implement Serializable, the following readResolve method suffices to guarantee the singleton property:

如果 Elvis 类要实现 Serializable 接口，下面的 readResolve 方法就足以保证其单例属性：

```Java
// readResolve for instance control - you can do better!
private Object readResolve() {
    // Return the one true Elvis and let the garbage collector
    // take care of the Elvis impersonator.
    return INSTANCE;
}
```

This method ignores the deserialized object, returning the distinguished Elvis instance that was created when the class was initialized. Therefore, the serialized form of an Elvis instance need not contain any real data; all instance fields should be declared transient. In fact, **if you depend on readResolve for instance control, all instance fields with object reference types must be declared transient.** Otherwise, it is possible for a determined attacker to secure a reference to the deserialized object before its readResolve method is run, using a technique that is somewhat similar to the MutablePeriod attack in Item 88.

此方法忽略反序列化对象，返回初始化类时创建的特殊 Elvis 实例。因此，Elvis 实例的序列化形式不需要包含任何实际数据；所有实例字段都应该声明为 transient。事实上，**如果你依赖 readResolve 进行实例控制，那么所有具有对象引用类型的实例字段都必须声明为 transient。** 否则，有的攻击者有可能在运行反序列化对象的 readResolve 方法之前保护对该对象的引用，使用的技术有点类似于 [Item-88](/Chapter-12/Chapter-12-Item-88-Write-readObject-methods-defensively.md) 中的 MutablePeriod 攻击。

The attack is a bit complicated, but the underlying idea is simple. If a singleton contains a nontransient object reference field, the contents of this field will be deserialized before the singleton’s readResolve method is run. This allows a carefully crafted stream to “steal” a reference to the originally deserialized singleton at the time the contents of the object reference field are deserialized.

攻击有点复杂，但其基本思想很简单。如果单例包含一个非 transient 对象引用字段，则在运行单例的 readResolve 方法之前，将对该字段的内容进行反序列化。这允许一个精心设计的流在对象引用字段的内容被反序列化时「窃取」对原来反序列化的单例对象的引用。

Here’s how it works in more detail. First, write a “stealer” class that has both a readResolve method and an instance field that refers to the serialized singleton in which the stealer “hides.” In the serialization stream, replace the singleton’s nontransient field with an instance of the stealer. You now have a circularity: the singleton contains the stealer, and the stealer refers to the singleton.

下面是它的工作原理。首先，编写一个 stealer 类，该类具有 readResolve 方法和一个实例字段，该实例字段引用序列化的单例，其中 stealer 「隐藏」在其中。在序列化流中，用一个 stealer 实例替换单例的非 transient 字段。现在你有了一个循环：单例包含了 stealer，而 stealer 引用了单例。

Because the singleton contains the stealer, the stealer’s readResolve method runs first when the singleton is deserialized. As a result, when the stealer’s readResolve method runs, its instance field still refers to the partially deserialized (and as yet unresolved) singleton.

因为单例包含 stealer，所以当反序列化单例时，窃取器的 readResolve 方法首先运行。因此，当 stealer 的 readResolve 方法运行时，它的实例字段仍然引用部分反序列化（且尚未解析）的单例。

The stealer’s readResolve method copies the reference from its instance field into a static field so that the reference can be accessed after the readResolve method runs. The method then returns a value of the correct type for the field in which it’s hiding. If it didn’t do this, the VM would throw a ClassCastException when the serialization system tried to store the stealer reference into this field.

stealer 的 readResolve 方法将引用从其实例字段复制到静态字段，以便在 readResolve 方法运行后访问引用。然后，该方法为其隐藏的字段返回正确类型的值。如果不这样做，当序列化系统试图将 stealer 引用存储到该字段时，VM 将抛出 ClassCastException。

To make this concrete, consider the following broken singleton:

要使问题具体化，请考虑以下被破坏的单例：

```Java
// Broken singleton - has nontransient object reference field!
public class Elvis implements Serializable {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { }
    private String[] favoriteSongs ={ "Hound Dog", "Heartbreak Hotel" };
    public void printFavorites() {
        System.out.println(Arrays.toString(favoriteSongs));
    }
    private Object readResolve() {
    return INSTANCE;
    }
}
```

Here is a “stealer” class, constructed as per the description above:

这里是一个 stealer 类，按照上面的描述构造：

```Java
public class ElvisStealer implements Serializable {
    static Elvis impersonator;
    private Elvis payload;

    private Object readResolve() {
        // Save a reference to the "unresolved" Elvis instance
        impersonator = payload;
        // Return object of correct type for favoriteSongs field
        return new String[] { "A Fool Such as I" };
    }

    private static final long serialVersionUID =0;
}
```

Finally, here is an ugly program that deserializes a handcrafted stream to produce two distinct instances of the flawed singleton. The deserialize method is omitted from this program because it’s identical to the one on page 354:

最后，这是一个有问题的程序，它反序列化了一个手工制作的流，以生成有缺陷的单例的两个不同实例。这个程序省略了反序列化方法，因为它与第 354 页的方法相同：

```Java
public class ElvisImpersonator {
// Byte stream couldn't have come from a real Elvis instance!
    private static final byte[] serializedForm = {
        (byte)0xac, (byte)0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x05,
        0x45, 0x6c, 0x76, 0x69, 0x73, (byte)0x84, (byte)0xe6,
        (byte)0x93, 0x33, (byte)0xc3, (byte)0xf4, (byte)0x8b,
        0x32, 0x02, 0x00, 0x01, 0x4c, 0x00, 0x0d, 0x66, 0x61, 0x76,
        0x6f, 0x72, 0x69, 0x74, 0x65, 0x53, 0x6f, 0x6e, 0x67, 0x73,
        0x74, 0x00, 0x12, 0x4c, 0x6a, 0x61, 0x76, 0x61, 0x2f, 0x6c,
        0x61, 0x6e, 0x67, 0x2f, 0x4f, 0x62, 0x6a, 0x65, 0x63, 0x74,
        0x3b, 0x78, 0x70, 0x73, 0x72, 0x00, 0x0c, 0x45, 0x6c, 0x76,
        0x69, 0x73, 0x53, 0x74, 0x65, 0x61, 0x6c, 0x65, 0x72, 0x00,
        0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x02, 0x00, 0x01,
        0x4c, 0x00, 0x07, 0x70, 0x61, 0x79, 0x6c, 0x6f, 0x61, 0x64,
        0x74, 0x00, 0x07, 0x4c, 0x45, 0x6c, 0x76, 0x69, 0x73, 0x3b,
        0x78, 0x70, 0x71, 0x00, 0x7e, 0x00, 0x02
    };

    public static void main(String[] args) {
        // Initializes ElvisStealer.impersonator and returns
        // the real Elvis (which is Elvis.INSTANCE)
        Elvis elvis = (Elvis) deserialize(serializedForm);
        Elvis impersonator = ElvisStealer.impersonator;
        elvis.printFavorites();
        impersonator.printFavorites();
    }
}
```

Running this program produces the following output, conclusively proving that it’s possible to create two distinct Elvis instances (with different tastes in music):

运行此程序将生成以下输出，最终证明可以创建两个不同的 Elvis 实例（具有不同的音乐品味）：

```Java
[Hound Dog, Heartbreak Hotel]
[A Fool Such as I]
```

You could fix the problem by declaring the favoriteSongs field transient, but you’re better off fixing it by making Elvis a single-element enum type (Item 3). As demonstrated by the ElvisStealer attack, using a readResolve method to prevent a “temporary” deserialized instance from being accessed by an attacker is fragile and demands great care.

通过将 favorites 字段声明为 transient 可以解决这个问题，但是最好把 Elvis 做成是一个单元素的枚举类型（[Item-3](/Chapter-2/Chapter-2-Item-3-Enforce-the-singleton-property-with-a-private-constructor-or-an-enum-type.md)）。ElvisStealer 所示的攻击表名，使用 readResolve 方法防止「temporary」反序列化实例被攻击者访问的方式是脆弱的，需要非常小心。

If you write your serializable instance-controlled class as an enum, Java guarantees you that there can be no instances besides the declared constants, unless an attacker abuses a privileged method such as AccessibleObject.setAccessible. Any attacker who can do that already has sufficient privileges to execute arbitrary native code, and all bets are off. Here’s how our Elvis example looks as an enum:

如果你将可序列化的实例控制类编写为枚举类型, Java 保证除了声明的常量之外不会有任何实例，除非攻击者滥用了特权方法，如 `AccessibleObject.setAccessible`。任何能够做到这一点的攻击者都已经拥有足够的特权来执行任意的本地代码，all bets are off。以下是把 Elvis 写成枚举的例子：

```Java
// Enum singleton - the preferred approach
public enum Elvis {
    INSTANCE;
    private String[] favoriteSongs ={ "Hound Dog", "Heartbreak Hotel" };
    public void printFavorites() {
        System.out.println(Arrays.toString(favoriteSongs));
    }
}
```

The use of readResolve for instance control is not obsolete. If you have to write a serializable instance-controlled class whose instances are not known at compile time, you will not be able to represent the class as an enum type.

使用 readResolve 进行实例控制并不过时。如果必须编写一个可序列化的实例控制类，而该类的实例在编译时是未知的，则不能将该类表示为枚举类型。

**The accessibility of readResolve is significant.** If you place a readResolve method on a final class, it should be private. If you place a readResolve method on a nonfinal class, you must carefully consider its accessibility. If it is private, it will not apply to any subclasses. If it is packageprivate, it will apply only to subclasses in the same package. If it is protected or public, it will apply to all subclasses that do not override it. If a readResolve method is protected or public and a subclass does not override it, deserializing a subclass instance will produce a superclass instance, which is likely to cause a ClassCastException.

**readResolve 的可访问性非常重要。** 如果你将 readResolve 方法放在 final 类上，那么它应该是私有的。如果将 readResolve 方法放在非 final 类上，必须仔细考虑其可访问性。如果它是私有的，它将不应用于任何子类。如果它是包级私有的，它将只适用于同一包中的子类。如果它是受保护的或公共的，它将应用于不覆盖它的所有子类。如果 readResolve 方法是受保护的或公共的，而子类没有覆盖它，反序列化子类实例将生成超类实例，这可能会导致 ClassCastException。

To summarize, use enum types to enforce instance control invariants wherever possible. If this is not possible and you need a class to be both serializable and instance-controlled, you must provide a readResolve method and ensure that all of the class’s instance fields are either primitive or transient.

总之，在可能的情况下，使用枚举类型强制实例控制不变量。如果这是不可能的，并且你需要一个既可序列化又实例控制的类，那么你必须提供一个 readResolve 方法，并确保该类的所有实例字段都是基本类型，或使用 transient 修饰。

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-12/Chapter-12-Introduction.md)**
- **Previous Item（上一条目）：[Item 88: Write readObject methods defensively（防御性地编写 readObject 方法）](/Chapter-12/Chapter-12-Item-88-Write-readObject-methods-defensively.md)**
- **Next Item（下一条目）：[Item 90: Consider serialization proxies instead of serialized instances（考虑以序列化代理代替序列化实例）](/Chapter-12/Chapter-12-Item-90-Consider-serialization-proxies-instead-of-serialized-instances.md)**
