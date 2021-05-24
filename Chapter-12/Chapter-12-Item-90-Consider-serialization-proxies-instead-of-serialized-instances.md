## Chapter 12. Serialization（序列化）

### Item 90: Consider serialization proxies instead of serialized instances（考虑以序列化代理代替序列化实例）

As mentioned in Items 85 and 86 and discussed throughout this chapter, the decision to implement Serializable increases the likelihood of bugs and security problems as it allows instances to be created using an extralinguistic mechanism in place of ordinary constructors. There is, however, a technique that greatly reduces these risks. This technique is known as the serialization proxy pattern.

正如在 [Item-85](/Chapter-12/Chapter-12-Item-85-Prefer-alternatives-to-Java-serialization.md) 和 [Item-86](/Chapter-12/Chapter-12-Item-86-Implement-Serializable-with-great-caution.md) 中提到的贯穿本章的问题：实现 Serializable 接口的决定增加了出现 bug 和安全问题的可能性，因为它允许使用一种超语言机制来创建实例，而不是使用普通的构造函数。然而，有一种技术可以大大降低这些风险。这种技术称为序列化代理模式。

The serialization proxy pattern is reasonably straightforward. First, design a private static nested class that concisely represents the logical state of an instance of the enclosing class. This nested class is known as the serialization proxy of the enclosing class. It should have a single constructor, whose parameter type is the enclosing class. This constructor merely copies the data from its argument: it need not do any consistency checking or defensive copying. By design, the default serialized form of the serialization proxy is the perfect serialized form of the enclosing class. Both the enclosing class and its serialization proxy must be declared to implement Serializable.

序列化代理模式相当简单。首先，设计一个私有静态嵌套类，它简洁地表示外围类实例的逻辑状态。这个嵌套类称为外围类的序列化代理。它应该有一个构造函数，其参数类型是外围类。这个构造函数只是从它的参数复制数据：它不需要做任何一致性检查或防御性复制。按照设计，序列化代理的默认序列化形式是外围类的完美序列化形式。外围类及其序列代理都必须声明实现 Serializable 接口。

For example, consider the immutable Period class written in Item 50 and made serializable in Item 88. Here is a serialization proxy for this class. Period is so simple that its serialization proxy has exactly the same fields as the class:

例如，考虑 [Item-50](/Chapter-8/Chapter-8-Item-50-Make-defensive-copies-when-needed.md) 中编写的不可变 Period 类，并在 [Item-88](/Chapter-12/Chapter-12-Item-88-Write-readObject-methods-defensively.md) 中使其可序列化。这是该类的序列化代理。Period 非常简单，它的序列化代理具有与类完全相同的字段：

```Java
// Serialization proxy for Period class
private static class SerializationProxy implements Serializable {
    private final Date start;
    private final Date end;
    SerializationProxy(Period p) {
        this.start = p.start;
        this.end = p.end;
    }
    private static final long serialVersionUID =234098243823485285L; // Any number will do (Item 87)
}
```

Next, add the following writeReplace method to the enclosing class. This method can be copied verbatim into any class with a serialization proxy:

接下来，将以下 writeReplace 方法添加到外围类中。通过序列化代理，这个方法可以被逐字地复制到任何类中：

```Java
// writeReplace method for the serialization proxy pattern
private Object writeReplace() {
    return new SerializationProxy(this);
}
```

The presence of this method on the enclosing class causes the serialization system to emit a SerializationProxy instance instead of an instance of the enclosing class. In other words, the writeReplace method translates an instance of the enclosing class to its serialization proxy prior to serialization.

该方法存在于外围类，导致序列化系统产生 SerializationProxy 实例，而不是外围类的实例。换句话说，writeReplace 方法在序列化之前将外围类的实例转换为其序列化代理。

With this writeReplace method in place, the serialization system will never generate a serialized instance of the enclosing class, but an attacker might fabricate one in an attempt to violate the class’s invariants. To guarantee that such an attack would fail, merely add this readObject method to the enclosing class:

有了这个 writeReplace 方法，序列化系统将永远不会生成外围类的序列化实例，但是攻击者可能会创建一个实例，试图违反类的不变性。为了保证这样的攻击会失败，只需将这个 readObject 方法添加到外围类中：

```Java
// readObject method for the serialization proxy pattern
private void readObject(ObjectInputStream stream) throws InvalidObjectException {
    throw new InvalidObjectException("Proxy required");
}
```

Finally, provide a readResolve method on the SerializationProxy class that returns a logically equivalent instance of the enclosing class. The presence of this method causes the serialization system to translate the serialization proxy back into an instance of the enclosing class upon deserialization.

最后，在 SerializationProxy 类上提供一个 readResolve 方法，该方法返回外围类的逻辑等效实例。此方法的存在导致序列化系统在反序列化时将序列化代理转换回外围类的实例。

This readResolve method creates an instance of the enclosing class using only its public API and therein lies the beauty of the pattern. It largely eliminates the extralinguistic character of serialization, because the deserialized instance is created using the same constructors, static factories, and methods as any other instance. This frees you from having to separately ensure that deserialized instances obey the class’s invariants. If the class’s static factories or constructors establish these invariants and its instance methods maintain them, you’ve ensured that the invariants will be maintained by serialization as well.

这个 readResolve 方法仅使用其公共 API 创建了一个外围类的实例，这就是该模式的美妙之处。它在很大程度上消除了序列化的语言外特性，因为反序列化实例是使用与任何其他实例相同的构造函数、静态工厂和方法创建的。这使你不必单独确保反序列化的实例遵从类的不变性。如果类的静态工厂或构造函数建立了这些不变性，而它的实例方法维护它们，那么你就确保了这些不变性也将通过序列化来维护。

Here is the readResolve method for Period.SerializationProxy above:

以下是上述 `Period.SerializationProxy` 的 readResolve 方法：

```Java
// readResolve method for Period.SerializationProxy
private Object readResolve() {
    return new Period(start, end); // Uses public constructor
}
```

Like the defensive copying approach (page 357), the serialization proxy approach stops the bogus byte-stream attack (page 354) and the internal field theft attack (page 356) dead in their tracks. Unlike the two previous approaches, this one allows the fields of Period to be final, which is required in order for the Period class to be truly immutable (Item 17). And unlike the two previous approaches, this one doesn’t involve a great deal of thought. You don’t have to figure out which fields might be compromised by devious serialization attacks, nor do you have to explicitly perform validity checking as part of deserialization.

与防御性复制方法（第 357 页）类似，序列化代理方法阻止伪字节流攻击（第 354 页）和内部字段盗窃攻击（第 356 页）。与前两种方法不同，这种方法允许 Period 的字段为 final，这是 Period 类真正不可变所必需的（[Item-17](/Chapter-4/Chapter-4-Item-17-Minimize-mutability.md)）。与前两种方法不同，这一种方法不需要太多的思考。你不必指出哪些字段可能会受到狡猾的序列化攻击的危害，也不必显式地执行有效性检查作为反序列化的一部分。

There is another way in which the serialization proxy pattern is more powerful than defensive copying in readObject. The serialization proxy pattern allows the deserialized instance to have a different class from the originally serialized instance. You might not think that this would be useful in practice, but it is.

序列化代理模式还有另一种方式比 readObject 中的防御性复制更强大。序列化代理模式允许反序列化实例具有与初始序列化实例不同的类。你可能不认为这在实践中有用，但它确实有用。

Consider the case of EnumSet (Item 36). This class has no public constructors, only static factories. From the client’s perspective, they return EnumSet instances, but in the current OpenJDK implementation, they return one of two subclasses, depending on the size of the underlying enum type. If the underlying enum type has sixty-four or fewer elements, the static factories return a RegularEnumSet; otherwise, they return a JumboEnumSet.

考虑 EnumSet 的情况（[Item-36](/Chapter-6/Chapter-6-Item-36-Use-EnumSet-instead-of-bit-fields.md)）。该类没有公共构造函数，只有静态工厂。从客户端的角度来看，它们返回 EnumSet 实例，但是在当前 OpenJDK 实现中，它们返回两个子类中的一个，具体取决于底层枚举类型的大小。如果底层枚举类型有 64 个或更少的元素，则静态工厂返回一个 RegularEnumSet；否则，它们返回一个 JumboEnumSet。

Now consider what happens if you serialize an enum set whose enum type has sixty elements, then add five more elements to the enum type, and then deserialize the enum set. It was a RegularEnumSet instance when it was serialized, but it had better be a JumboEnumSet instance once it is deserialized. In fact that’s exactly what happens, because EnumSet uses the serialization proxy pattern. In case you’re curious, here is EnumSet’s serialization proxy. It really is this simple:

现在考虑，如果序列化一个枚举集合，它的枚举类型有 60 个元素，然后给这个枚举类型再增加 5 个元素，之后反序列化这个枚举集合。当它被序列化的时候，返回 RegularEnumSet 实例，但最好是 JumboEnumSet 实例。事实上正是这样，因为 EnumSet 使用序列化代理模式。如果你好奇，这里是 EnumSet 的序列化代理。其实很简单：

```Java
// EnumSet's serialization proxy
private static class SerializationProxy <E extends Enum<E>> implements Serializable {
    // The element type of this enum set.
    private final Class<E> elementType;

    // The elements contained in this enum set.
    private final Enum<?>[] elements;

    SerializationProxy(EnumSet<E> set) {
        elementType = set.elementType;
        elements = set.toArray(new Enum<?>[0]);
    }

    private Object readResolve() {
        EnumSet<E> result = EnumSet.noneOf(elementType);
        for (Enum<?> e : elements)
            result.add((E)e);
        return result;
    }

    private static final long serialVersionUID =362491234563181265L;
}
```

The serialization proxy pattern has two limitations. It is not compatible with classes that are extendable by their users (Item 19). Also, it is not compatible with some classes whose object graphs contain circularities: if you attempt to invoke a method on such an object from within its serialization proxy’s readResolve method, you’ll get a ClassCastException because you don’t have the object yet, only its serialization proxy.

序列化代理模式有两个限制。它与客户端可扩展的类不兼容（[Item-19](/Chapter-4/Chapter-4-Item-19-Design-and-document-for-inheritance-or-else-prohibit-it.md)）。而且，它也不能与对象图中包含循环的某些类兼容：如果你试图从对象的序列化代理的 readResolve 方法中调用对象上的方法，你将得到一个 ClassCastException，因为你还没有对象，只有对象的序列化代理。

Finally, the added power and safety of the serialization proxy pattern are not free. On my machine, it is 14 percent more expensive to serialize and deserialize Period instances with serialization proxies than it is with defensive copying.

最后，序列化代理模式所增强的功能和安全性并不是没有代价的。在我的机器上，通过序列化代理来序列化和反序列化 Period 实例的开销，比用保护性拷贝进行的开销增加了14%。

In summary, consider the serialization proxy pattern whenever you find yourself having to write a readObject or writeObject method on a class that is not extendable by its clients. This pattern is perhaps the easiest way to robustly serialize objects with nontrivial invariants.

总之，当你发现必须在客户端不可扩展的类上编写 readObject 或 writeObject 方法时，请考虑序列化代理模式。要想稳健地将带有重要约束条件的对象序列化时，这种模式可能是最容易的方法。

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-12/Chapter-12-Introduction.md)**
- **Previous Item（上一条目）：[Item 89: For instance control prefer enum types to readResolve（对于实例控制，枚举类型优于 readResolve）](/Chapter-12/Chapter-12-Item-89-For-instance-control-prefer-enum-types-to-readResolve.md)**
