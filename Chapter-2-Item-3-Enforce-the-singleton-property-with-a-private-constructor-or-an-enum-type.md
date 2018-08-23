## Chapter 2. Creating and Destroying Objects（创建和销毁对象）

### Item 3: Enforce the singleton property with a private constructor or an enum type（使用私有构造函数或枚举类型执行单例属性）

A singleton is simply a class that is instantiated（v.实例化） exactly once [Gamma95].Singletons typically represent either a stateless object such as a function (Item24) or a system component that is intrinsically unique. **Making a class a singleton can make it difficult to test its clients** because it’s impossible to substitute a mock implementation for a singleton unless it implements an interface that serves as its type.

单例对象只是一个只实例化一次的类[Gamma95]。单例对象通常表示无状态对象，比如函数（[Item-24](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-4-Item-24-Favor-static-member-classes-over-nonstatic.md)）或系统组件，它们在本质上是唯一的。**将一个类作为单例对象会使测试它的客户端变得困难，** 因为除非它实现了作为其类型的接口，否则无法用模拟实现来代替单例对象。

There are two common ways to implement singletons. Both are based on keeping the constructor private and exporting a public static member to provide access to the sole instance. In one approach, the member is a final field:

实现单例有两种常见的方法。两者都基于保持构造函数私有和导出公共静态成员以提供对唯一实例的访问。在一种方法中，成员是最后一个字段：

```
// Singleton with public final field
public class Elvis {
public static final Elvis INSTANCE = new Elvis();
private Elvis() { ... }
public void leaveTheBuilding() { ... }
}
```

The private constructor is called only once, to initialize the public static final field Elvis.INSTANCE. The lack of a public or protected constructor guarantees a “monoelvistic” universe: exactly one Elvis instance will exist once the Elvis class is initialized—no more, no less. Nothing that a client does can change this, with one caveat: a privileged client can invoke the private constructor reflectively (Item 65) with the aid of the AccessibleObject.setAccessible method. If you need to defend against this attack, modify the constructor to make it throw an exception if it’s asked to create a second instance.

私有构造函数只调用一次，用于初始化公共静态终态字段Elvis.INSTANCE。缺少公共或受保护的构造函数保证了“monoelvistic”世界：一旦初始化了Elvis类，就只会存在一个Elvis实例——不多也不少。客户机所做的任何事情都不能改变这一点，但有一点需要注意：特权客户机可以借助AccessibleObject.setAccessible方法反射地调用私有构造函数（[Item-65](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-9-Item-65-Prefer-interfaces-to-reflection.md)）如果需要防范这种攻击，请修改构造函数，使其在请求创建第二个实例时抛出异常。

In the second approach to implementing singletons, the public member is a static factory method:

在实现单例的第二种方法中，公共成员是一种静态工厂方法：

```
// Singleton with static factory
public class Elvis {
private static final Elvis INSTANCE = new Elvis();
private Elvis() { ... }
public static Elvis getInstance() { return INSTANCE; }
public void leaveTheBuilding() { ... }
}
```

All calls to Elvis.getInstance return the same object reference, and no other Elvis instance will ever be created (with the same caveat（n.警告） mentioned（v.提到） earlier).

所有对Elvis.getInstance的调用都返回相同的对象引用，并且不会创建其他Elvis实例（与前面提到的警告相同）。

**译注：这里的警告指“特权客户机可以借助AccessibleObject.setAccessible方法反射地调用私有构造函数”**

The main advantage of the public field approach is that the API makes it clear that the class is a singleton: the public static field is final, so it will always contain the same object reference. The second advantage is that it’s simpler.

公共字段方法的主要优点是API明确了类是单例的：公共静态字段是final的，因此它总是包含相同的对象引用。第二个优点是更简单。

One advantage of the static factory approach is that it gives you the flexibility to change your mind about whether the class is a singleton without changing its API. The factory method returns the sole instance, but it could be modified to return, say, a separate instance for each thread that invokes it. A second advantage is that you can write a generic singleton factory if your application requires it (Item 30). A final advantage of using a static factory is that a method reference can be used as a supplier, for example Elvis::instance is a Supplier<Elvis>. Unless one of these advantages is relevant, the public field approach is preferable.

静态工厂方法的一个优点是，它使您能够在不更改其API的情况下更改类是否是单例对象的想法。工厂方法返回唯一的实例，但是可以对其进行修改，为调用它的每个线程返回一个单独的实例。第二个优点是，如果应用程序需要的话，可以编写通用的单例工厂（[Item-30](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-5-Item-30-Favor-generic-methods.md)）。使用静态工厂的最后一个优点是方法引用可以用作供应商，例如Elvis::instance是Supplier<Elvis>。除非这些优点中有一项是相关的，否则公共字段方法更可取。

To make a singleton class that uses either of these approaches serializable (Chapter 12), it is not sufficient merely to add implements Serializable to its declaration. To maintain（vt.维持） the singleton guarantee（n.保证）, declare all instance fields transient and provide a readResolve method (Item 89). Otherwise, each time a serialized instance is deserialized, a new instance will be created, leading,in the case of our example, to spurious（adj.虚假的） Elvis sightings. To prevent this from happening, add this readResolve method to the Elvis class:

要使单例类使用这两种方法中的任何一种（[12-Serialization]( https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-12-Introduction.md)），仅仅在其声明中添加实现serializable是不够的。要维护单例保证，声明所有实例字段为transient，并提供readResolve方法（[Item-89](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-12-Item-89-For-instance-control-prefer-enum-types-to-readResolve.md)）。否则，每次反序列化实例时，都会创建一个新实例，在我们的示例中，这会导致出现虚假的Elvis。为了防止这种情况发生，将这个readResolve方法添加到Elvis类中：

```
// readResolve method to preserve singleton property
private Object readResolve() {
// Return the one true Elvis and let the garbage collector
// take care of the Elvis impersonator.
return INSTANCE;
}
```

A third way to implement a singleton is to declare a single-element enum:

实现单例的第三种方法是声明一个单元素枚举：

```
// Enum singleton - the preferred approach
public enum Elvis {
INSTANCE;
public void leaveTheBuilding() { ... }
}
```

This approach（n.方法，途径；vt.接近） is similar to the public field approach, but it is more concise,provides the serialization machinery for free, and provides an ironclad guarantee against multiple instantiation, even in the face of sophisticated serialization or reflection attacks. This approach may feel a bit unnatural, but **a single-element enum type is often the best way to implement a singleton.** Note that you can’t use this approach if your singleton must extend a superclass other than Enum(though you can declare an enum to implement interfaces).

这种方法类似于公共字段方法，但是它更简洁，免费提供了序列化机制，并提供了对多次实例化的铁证，即使面对复杂的序列化或反射攻击也是如此。这种方法可能有点不自然，但是**单元素枚举类型通常是实现单元素的最佳方法。** 注意，如果你的单例对象必须扩展一个超类而不是Enum（尽管你可以声明一个Enum来实现接口），你就不能使用这种方法。
