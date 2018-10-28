## Chapter 4. Classes and Interfaces（类和接口）

### Item 21: Design interfaces for posterity（为后代设计接口）

Prior to Java 8, it was impossible to add methods to interfaces without breaking existing implementations. If you added a new method to an interface, existing implementations would, in general, lack the method, resulting in a compile-time error. In Java 8, the default method construct was added [JLS 9.4], with the intent of allowing the addition of methods to existing interfaces. But adding new methods to existing interfaces is fraught with risk.

在Java 8之前，在不破坏现有实现的情况下向接口添加方法是不可能的。如果在接口中添加新方法，现有的实现通常会缺少该方法，从而导致编译时错误。在Java 8中，添加了默认的方法构造[JLS 9.4]，目的是允许向现有接口添加方法。但是向现有接口添加新方法充满了风险。

The declaration for a default method includes a default implementation that is used by all classes that implement the interface but do not implement the default method. While the addition of default methods to Java makes it possible to add methods to an existing interface, there is no guarantee that these methods will work in all preexisting implementations. Default methods are “injected” into existing implementations without the knowledge or consent of their implementors. Before Java 8, these implementations were written with the tacit understanding that their interfaces would never acquire any new methods.

默认方法的声明包括一个默认实现，所有实现接口但不实现默认方法的类都使用这个默认实现。虽然向Java添加默认方法使向现有接口添加方法成为可能，但不能保证这些方法在所有现有实现中都能工作。默认方法被“注入”到现有的实现中，而无需实现者的知情或同意。在Java 8之前，编写这些实现时都默认它们的接口永远不会获得任何新方法。

Many new default methods were added to the core collection interfaces in Java 8, primarily to facilitate the use of lambdas (Chapter 6). The Java libraries’ default methods are high-quality general-purpose implementations, and in most cases, they work fine. **But it is not always possible to write a default method that maintains（vt. 维持；继续；维修；主张；供养） all invariants（n. [数] 不变量，invariant的复数；不变性） of every conceivable（adj. 可能的；想得到的，可想像的） implementation.**

Java 8的核心集合接口增加了许多新的默认方法，主要是为了方便lambdas的使用（第6章）。**但是，并不总是能够编写一个默认方法来维护每个可想到的实现的所有不变量**

For example, consider the removeIf method, which was added to the Collection interface in Java 8. This method removes all elements for which a given boolean function (or predicate) returns true. The default implementation is specified to traverse the collection using its iterator, invoking the predicate on each element, and using the iterator’s remove method to remove the elements for which the predicate returns true. Presumably the declaration looks something like this:

例如，考虑removeIf方法，它被添加到Java 8中的集合接口中。该方法删除了给定的布尔函数（或predicate）返回true的所有元素。指定默认实现，以使用迭代器遍历集合，在每个元素上调用predicate，并使用迭代器的remove方法删除谓词返回true的元素。大概声明是这样的：

```
// Default method added to the Collection interface in Java 8
default boolean removeif(predicate<? super e> filter) {
    objects.requirenonnull(filter);
    boolean result = false;
    for (iterator<e> it = iterator(); it.hasnext(); ) {
        if (filter.test(it.next())) {
            it.remove();
            result = true;
        }
    }
    return result;
}
```

This is the best general-purpose implementation one could possibly write for the removeIf method, but sadly, it fails on some real-world Collection implementations. For example, consider org.apache.commons.collections4.collection.SynchronizedCollection. This class, from the Apache Commons library, is similar to the one returned by the static factory Collections.-synchronizedCollection in java.util. The Apache version additionally provides the ability to use a client-supplied object for locking, in place of the collection. In other words, it is a wrapper class (Item 18), all of whose methods synchronize on a locking object before delegating to the wrapped collection.

这是为removeIf方法编写的最好的通用实现，但遗憾的是，它在一些实际的集合实现中失败了。例如，考虑org.apache.commons.collections4.collection.SynchronizedCollection。这个类来自Apache Commons库，类似于静态工厂集合返回的类。-synchronizedCollection java.util。Apache版本还提供了使用客户机提供的对象进行锁定的功能，以代替集合。换句话说，它是一个包装器类（[Item-18](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-4-Item-18-Favor-composition-over-inheritance.md)），其所有方法在委托给包装集合之前同步锁定对象。

The Apache SynchronizedCollection class is still being actively maintained, but as of this writing, it does not override the removeIf method. If this class is used in conjunction with Java 8, it will therefore inherit the default implementation of removeIf, which does not, indeed cannot, maintain the class’s fundamental promise: to automatically synchronize around each method invocation. The default implementation knows nothing about synchronization and has no access to the field that contains the locking object. If a client calls the removeIf method on a SynchronizedCollection instance in the presence of concurrent modification of the collection by another thread, a ConcurrentModificationException or other unspecified behavior may result.

Apache SynchronizedCollection类仍然得到了积极的维护，但是在编写本文时，它没有覆盖removeIf方法。如果这个类与Java 8一起使用，那么它将继承removeIf的默认实现，而removeIf并不能维护类的基本承诺：自动同步每个方法调用。默认实现对同步一无所知，也无法访问包含锁定对象的字段。如果客户端在SynchronizedCollection实例上调用removeIf方法，而另一个线程同时修改了集合，那么可能会导致ConcurrentModificationException或其他未指定的行为。

In order to prevent this from happening in similar Java platform libraries implementations, such as the package-private class returned by Collections.synchronizedCollection, the JDK maintainers had to override the default removeIf implementation and other methods like it to perform the necessary synchronization before invoking the default implementation. Preexisting collection implementations that were not part of the Java platform did not have the opportunity to make analogous changes in lockstep with the interface change, and some have yet to do so.

为了防止类似的Java平台库实现中发生这种情况，例如集合返回的包私有类。synchronizedCollection, JDK维护人员必须覆盖默认的removeIf实现和其他类似的方法，以便在调用默认实现之前执行必要的同步。不属于Java平台的现有集合实现没有机会与接口更改同步进行类似的更改，有些还没有这样做。

**In the presence of default methods, existing implementations of an interface may compile without error or warning but fail at runtime.** While not terribly common, this problem is not an isolated incident either. A handful of the methods added to the collections interfaces in Java 8 are known to be susceptible, and a handful of existing implementations are known to be affected.

在有默认方法的情况下，接口的现有实现可以在没有错误或警告的情况下编译，但是在运行时失败。虽然这个问题并不常见，但也不是孤立的事件。已知Java 8中添加到集合接口的少数方法是易受影响的，已知会影响到现有的少数实现。

Using default methods to add new methods to existing interfaces should be avoided unless the need is critical, in which case you should think long and hard about whether an existing interface implementation might be broken by your default method implementation. Default methods are, however, extremely useful for providing standard method implementations when an interface is created, to ease the task of implementing the interface (Item 20).

除非必要，否则应该避免使用默认方法向现有接口添加新方法，在这种情况下，您应该仔细考虑现有接口实现是否可能被默认方法破坏。然而，在创建接口时，默认方法对于提供标准方法实现非常有用，以减轻实现接口的任务（[Item-20](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-4-Item-20-Prefer-interfaces-to-abstract-classes.md)）。

It is also worth noting that default methods were not designed to support removing methods from interfaces or changing the signatures of existing methods. Neither of these interface changes is possible without breaking existing clients.

同样值得注意的是，默认方法的设计并不支持从接口中删除方法或更改现有方法的签名。在不破坏现有客户端的情况下，这些更改都是不可能的。

The moral is clear. Even though default methods are now a part of the Java platform, **it is still of the utmost importance to design interfaces with great care.** While default methods make it possible to add methods to existing interfaces, there is great risk in doing so. If an interface contains a minor flaw, it may irritate its users forever; if an interface is severely deficient, it may doom the API that contains it.

这个教训很清楚。尽管默认方法现在已经是Java平台的一部分，但是谨慎地设计接口仍然是非常重要的。**虽然默认方法使向现有接口添加方法成为可能，但这样做存在很大风险。** 如果一个接口包含一个小缺陷，它可能会永远激怒它的使用者；如果接口有严重缺陷，它可能会毁掉包含它的API。

Therefore, it is critically important to test each new interface before you release it. Multiple programmers should implement each interface in different ways. At a minimum, you should aim for three diverse implementations. Equally important is to write multiple client programs that use instances of each new interface to perform various tasks. This will go a long way toward ensuring that each interface satisfies all of its intended uses. These steps will allow you to discover flaws in interfaces before they are released, when you can still correct them easily. **While it may be possible to correct some interface flaws after an interface is released, you cannot count on it.**

因此，在发布每个新接口之前对其进行测试非常重要。多个程序员应该以不同的方式实现每个接口。至少，您应该以三种不同的实现为目标。同样重要的是编写多个客户端程序，这些程序使用每个新接口的实例来执行各种任务。这将大大有助于确保每个接口满足其所有预期用途。这些步骤将允许您在接口被发布之前发现它们的缺陷，而您仍然可以轻松地纠正它们。**虽然在接口被发布之后可以纠正一些接口缺陷，但是您不能指望（这种方式）。**
