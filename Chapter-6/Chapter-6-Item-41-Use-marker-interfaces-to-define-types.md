## Chapter 6. Enums and Annotations（枚举和注解）

### Item 41: Use marker interfaces to define types（使用标记接口定义类型）

A marker interface is an interface that contains no method declarations but merely designates (or “marks”) a class that implements the interface as having some property. For example, consider the Serializable interface (Chapter 12). By implementing this interface, a class indicates that its instances can be written to an ObjectOutputStream (or “serialized”).

标记接口是一种不包含任何方法声明的接口，它只是指定（或「标记」）一个类，该类实现了具有某些属性的接口。例如，考虑 Serializable 接口（Chapter 12）。通过实现此接口，表示类的实例可以写入 ObjectOutputStream（或「序列化」）。

You may hear it said that marker annotations (Item 39) make marker interfaces obsolete. This assertion is incorrect. Marker interfaces have two advantages over marker annotations. First and foremost, **marker interfaces define a type that is implemented by instances of the marked class; marker annotations do not.** The existence of a marker interface type allows you to catch errors at compile time that you couldn’t catch until runtime if you used a marker annotation.

你可能听说过标记接口过时了，更好的方式是标记注解（[Item-39](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-6/Chapter-6-Item-39-Prefer-annotations-to-naming-patterns.md)）。这个言论是错误的。与标记注解相比，标记接口有两个优点。首先，**标记接口定义的类型由标记类的实例实现；标记注解不会。** 标记接口类型的存在允许你在编译时捕获错误，如果你使用标记注解，则在运行时才能捕获这些错误。

Java’s serialization facility (Chapter 6) uses the Serializable marker interface to indicate that a type is serializable. The ObjectOutputStream.writeObject method, which serializes the object that is passed to it, requires that its argument be serializable. Had the argument of this method been of type Serializable, an attempt to serialize an inappropriate object would have been detected at compile time (by type checking). Compile-time error detection is the intent of marker interfaces, but unfortunately, the ObjectOutputStream.write API does not take advantage of the Serializable interface: its argument is declared to be of type Object, so attempts to serialize an unserializable object won’t fail until runtime.

Java 的序列化工具（Chapter 6）使用 Serializable 标记接口来指示类是可序列化的。`ObjectOutputStream.writeObject` 方法序列化传递给它的对象，它要求其参数是可序列化的。如果该方法的参数类型是 Serializable，那么在编译时（通过类型检查）就会检测到对不合适的对象进行序列化的尝试。编译时错误检测是标记接口的目的，但不幸的是，`ObjectOutputStream.write` API 没有利用 Serializable 接口：它的参数被声明为 Object 类型，因此序列化一个不可序列化对象的尝试直到运行时才会失败。

译注：原文 `ObjectOutputStream.write` 有误，该方法的每种重载仅支持 int 类型和 byte[]，应修改为 `ObjectOutputStream.writeObject`，其源码如下：
```
public final void writeObject(Object obj) throws IOException {
    if (enableOverride) {
        writeObjectOverride(obj);
        return;
    }
    try {
        writeObject0(obj, false);
    } catch (IOException ex) {
        if (depth == 0) {
            writeFatalException(ex);
        }
        throw ex;
    }
}
```

***译注 2：使用 ObjectOutputStream.writeObject 的例子***
```
public class BaseClass implements Serializable {
    private final int id;
    private final String name;

    public BaseClass(int id, String name) {
        this.id = id;
        this.name = name;
    }

    @Override
    public String toString() {
        return "id=" + id + ", name='" + name + '\'';
    }
}

public class Main {
    private void Out() throws IOException {
        BaseClass obj = new BaseClass(1, "Mark");
        try (ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream(new File("out.txt")))) {
            out.writeObject(obj);
        }
    }

    private void In() throws IOException, ClassNotFoundException {
        try (ObjectInputStream in = new ObjectInputStream(new FileInputStream(new File("out.txt")))) {
            BaseClass obj = (BaseClass) in.readObject();
            System.out.println(obj);
        }
    }
}
```

**Another advantage of marker interfaces over marker annotations is that they can be targeted more precisely.** If an annotation type is declared with target ElementType.TYPE, it can be applied to any class or interface. Suppose you have a marker that is applicable only to implementations of a particular interface. If you define it as a marker interface, you can have it extend the sole interface to which it is applicable, guaranteeing that all marked types are also subtypes of the sole interface to which it is applicable.

**标记接口相对于标记注解的另一个优点是可以更精确地定位它们。** 如果注解类型使用 `ElementType.TYPE` 声明，它可以应用于任何类或接口。假设你有一个只适用于特定接口实现的标记。如果将其定义为标记接口，则可以让它扩展其适用的惟一接口，确保所有标记的类型也是其适用的惟一接口的子类型。

Arguably, the Set interface is just such a restricted marker interface. It is applicable only to Collection subtypes, but it adds no methods beyond those defined by Collection. It is not generally considered to be a marker interface because it refines the contracts of several Collection methods, including add, equals, and hashCode. But it is easy to imagine a marker interface that is applicable only to subtypes of some particular interface and does not refine the contracts of any of the interface’s methods. Such a marker interface might describe some invariant of the entire object or indicate that instances are eligible for processing by a method of some other class (in the way that the Serializable interface indicates that instances are eligible for processing by ObjectOutputStream).

可以说，Set 接口就是这样一个受限的标记接口。它只适用于 Collection 的子类，但是除了 Collection 定义的方法之外，它不添加任何方法。它通常不被认为是一个标记接口，因为它细化了几个 Collection 方法的约定，包括 add、equals 和 hashCode。但是很容易想象一个标记接口只适用于某些特定接口的子类，而不细化任何接口方法的约定。这样的标记接口可能描述整个对象的某个不变量，或者表明实例能够利用其他类的方法进行处理（就像 Serializable 接口能够利用 ObjectOutputStream 进行处理一样）。

**The chief advantage of marker annotations over marker interfaces is that they are part of the larger annotation facility.** Therefore, marker annotations allow for consistency in annotation-based frameworks.

**相对于标记接口，标记注解的主要优势是它们是更大的注解功能的一部分。** 因此，标记注解允许基于注解的框架中的一致性。

So when should you use a marker annotation and when should you use a marker interface? Clearly you must use an annotation if the marker applies to any program element other than a class or interface, because only classes and interfaces can be made to implement or extend an interface. If the marker applies only to classes and interfaces, ask yourself the question “Might I want to write one or more methods that accept only objects that have this marking?” If so, you should use a marker interface in preference to an annotation. This will make it possible for you to use the interface as a parameter type for the methods in question, which will result in the benefit of compile-time type checking. If you can convince yourself that you’ll never want to write a method that accepts only objects with the marking, then you’re probably better off using a marker annotation. If, additionally, the marking is part of a framework that makes heavy use of annotations, then a marker annotation is the clear choice.

那么什么时候应该使用标记注解，什么时候应该使用标记接口呢？显然，如果标记应用于类或接口之外的任何程序元素，则必须使用注解，因为只有类和接口才能实现或扩展接口。如果标记只适用于类和接口，那么可以问自己这样一个问题：「我是否可以编写一个或多个方法，只接受具有这种标记的对象？」如果是这样，你应该使用标记接口而不是注解。这将使你能够将接口用作相关方法的参数类型，这将带来编译时类型检查的好处。如果你确信自己永远不会编写只接受带有标记的对象的方法，那么最好使用标记注解。此外，如果框架大量使用注解，那么标记注解就是明确的选择。

In summary, marker interfaces and marker annotations both have their uses. If you want to define a type that does not have any new methods associated with it, a marker interface is the way to go. If you want to mark program elements other than classes and interfaces or to fit the marker into a framework that already makes heavy use of annotation types, then a marker annotation is the correct choice. **If you find yourself writing a marker annotation type whose target is ElementType.TYPE, take the time to figure out whether it really should be an annotation type or whether a marker interface would be more appropriate.**

总之，标记接口和标记注解都有各自的用途。如果你想要定义一个没有与之关联的新方法的类型，可以使用标记接口。如果你希望标记类和接口之外的程序元素，或者将标记符放入已经大量使用注解类型的框架中，那么标记符注解就是正确的选择。如果你发现自己编写的标记注解类型的目标是 `ElementType.TYPE`，那么请花时间弄清楚它究竟应该是注解类型，还是标记接口更合适。

In a sense, this item is the inverse of Item 22, which says, “If you don’t want to define a type, don’t use an interface.” To a first approximation, this item says, “If you do want to define a type, do use an interface.”

从某种意义上说，本条目与 [Item-22](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-4/Chapter-4-Item-22-Use-interfaces-only-to-define-types.md) 的说法相反，也就是说，「如果不想定义类型，就不要使用接口。」与本条目近似的说法是，「如果你确实想定义类型，那么一定要使用接口。」
