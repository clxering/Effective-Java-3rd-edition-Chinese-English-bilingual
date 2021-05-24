## Chapter 8. Methods（方法）

### Item 50: Make defensive copies when needed（在需要时制作防御性副本）

One thing that makes Java a pleasure to use is that it is a safe language. This means that in the absence of native methods it is immune to buffer overruns, array overruns, wild pointers, and other memory corruption errors that plague unsafe languages such as C and C++. In a safe language, it is possible to write classes and to know with certainty that their invariants will hold, no matter what happens in any other part of the system. This is not possible in languages that treat all of memory as one giant array.

Java 是一种安全的语言，这是它的一大优点。这意味着在没有本地方法的情况下，它不受缓冲区溢出、数组溢出、非法指针和其他内存损坏错误的影响，这些错误困扰着 C 和 c++ 等不安全语言。在一种安全的语言中，可以编写一个类并确定它们的不变量将保持不变，而不管在系统的任何其他部分发生了什么。在将所有内存视为一个巨大数组的语言中，这是不可能的。

Even in a safe language, you aren’t insulated from other classes without some effort on your part. **You must program defensively, with the assumption that clients of your class will do their best to destroy its invariants.** This is increasingly true as people try harder to break the security of systems, but more commonly, your class will have to cope with unexpected behavior resulting from the honest mistakes of well-intentioned programmers. Either way, it is worth taking the time to write classes that are robust in the face of ill-behaved clients.

即使使用一种安全的语言，如果你不付出一些努力，也无法与其他类隔离。**你必须进行防御性的设计，并假定你的类的客户端会尽最大努力破坏它的不变量。** 随着人们越来越多地尝试破坏系统的安全性，这个观点越来越正确，但更常见的情况是，你的类将不得不处理程序员的无意错误所导致的意外行为。无论哪种方式，都值得花时间编写一个健壮的类来面对行为不轨的客户端。

While it is impossible for another class to modify an object’s internal state without some assistance from the object, it is surprisingly easy to provide such assistance without meaning to do so. For example, consider the following class, which purports to represent an immutable time period:

虽然如果没有对象的帮助，另一个类是不可能修改对象的内部状态的，但是要提供这样的帮助却出奇地容易。例如，考虑下面的类，它表示一个不可变的时间段：

```Java
// Broken "immutable" time period class
public final class Period {
    private final Date start;
    private final Date end;

    /**
    * @param start the beginning of the period
    * @param end the end of the period; must not precede start
    * @throws IllegalArgumentException if start is after end
    * @throws NullPointerException if start or end is null
    */
    public Period(Date start, Date end) {
        if (start.compareTo(end) > 0)
            throw new IllegalArgumentException(start + " after " + end);
        this.start = start;
        this.end = end;
    }

    public Date start() {
        return start;
    }

    public Date end() {
        return end;
    }
    ... // Remainder omitted
}
```

At first glance, this class may appear to be immutable and to enforce the invariant that the start of a period does not follow its end. It is, however, easy to violate this invariant by exploiting the fact that Date is mutable:

乍一看，这个类似乎是不可变的，并且要求一个时间段的开始时间不能在结束时间之后。然而，利用 Date 是可变的这一事实很容易绕过这个约束：

```Java
// Attack the internals of a Period instance
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
end.setYear(78); // Modifies internals of p!
```

As of Java 8, the obvious way to fix this problem is to use Instant (or Local-DateTime or ZonedDateTime) in place of a Date because Instant (and the other java.time classes) are immutable (Item 17). **Date is obsolete and should no longer be used in new code.** That said, the problem still exists: there are times when you’ll have to use mutable value types in your APIs and internal representations, and the techniques discussed in this item are appropriate for those times.

从 Java 8 开始，解决这个问题的典型方法就是使用 Instant（或 Local-DateTime 或 ZonedDateTime）来代替 Date，因为 Instant（和其他时间类）类是不可变的（[Item-17](/Chapter-4/Chapter-4-Item-17-Minimize-mutability.md)）。**Date 已过时，不应在新代码中使用。** 尽管如此，问题仍然存在：有时你必须在 API 和内部表示中使用可变值类型，本项目中讨论的技术适用于这些情形。

To protect the internals of a Period instance from this sort of attack, **it is essential to make a defensive copy of each mutable parameter to the constructor** and to use the copies as components of the Period instance in place of the originals:

为了保护 Period 实例的内部不受此类攻击，**必须将每个可变参数的防御性副本复制给构造函数**，并将副本用作 Period 实例的组件，而不是原始组件：

```Java
// Repaired constructor - makes defensive copies of parameters
public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());
    if (this.start.compareTo(this.end) > 0)
        throw new IllegalArgumentException(this.start + " after " + this.end);
}
```

With the new constructor in place, the previous attack will have no effect on the Period instance. Note that **defensive copies are made before checking the validity of the parameters (Item 49), and the validity check is performed on the copies rather than on the originals.** While this may seem unnatural, it is necessary. It protects the class against changes to the parameters from another thread during the window of vulnerability between the time the parameters are checked and the time they are copied. In the computer security community, this is known as a time-of-check/time-of-use or TOCTOU attack [Viega01].

有了新的构造函数，之前的攻击将不会对 Period 实例产生影响。注意，**防御性副本是在检查参数的有效性之前制作的（[Item-49](/Chapter-8/Chapter-8-Item-49-Check-parameters-for-validity.md)），有效性检查是在副本上而不是在正本上执行的。** 虽然这看起来不自然，但却是必要的。在检查参数和复制参数之间的空窗期，它保护类不受来自其他线程的参数更改的影响。在计算机安全社区里，这被称为 time-of-check/time-of-use 或 TOCTOU 攻击 [Viega01]。

Note also that we did not use Date’s clone method to make the defensive copies. Because Date is nonfinal, the clone method is not guaranteed to return an object whose class is java.util.Date: it could return an instance of an untrusted subclass that is specifically designed for malicious mischief. Such a subclass could, for example, record a reference to each instance in a private static list at the time of its creation and allow the attacker to access this list. This would give the attacker free rein over all instances. To prevent this sort of attack, **do not use the clone method to make a defensive copy of a parameter whose type is subclassable by untrusted parties.**

还要注意，我们没有使用 Date 的 clone 方法来创建防御性副本。因为 Date 不是 final 的，所以不能保证 clone 方法返回一个 java.util.Date 的实例对象：它可以返回一个不受信任子类的实例，这个子类是专门为恶意破坏而设计的。例如，这样的子类可以在创建时在私有静态列表中记录对每个实例的引用，并允许攻击者访问这个列表。这将使攻击者可以自由控制所有实例。为防止此类攻击，**对可被不受信任方子类化的参数类型，不要使用 clone 方法进行防御性复制。**

While the replacement constructor successfully defends against the previous attack, it is still possible to mutate a Period instance, because its accessors offer access to its mutable internals:

虽然替换构造函数成功地防御了之前的攻击，但是仍然可以修改 Period 实例，因为它的访问器提供了对其可变内部结构的访问：

```Java
// Second attack on the internals of a Period instance
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
p.end().setYear(78); // Modifies internals of p!
```

To defend against the second attack, merely modify the accessors to return defensive copies of mutable internal fields:

要防御第二次攻击，只需修改访问器，返回可变内部字段的防御副本：

```Java
// Repaired accessors - make defensive copies of internal fields
public Date start() {
    return new Date(start.getTime());
}

public Date end() {
    return new Date(end.getTime());
}
```

With the new constructor and the new accessors in place, Period is truly immutable. No matter how malicious or incompetent a programmer, there is simply no way to violate the invariant that the start of a period does not follow its end (without resorting to extralinguistic means such as native methods and reflection). This is true because there is no way for any class other than Period itself to gain access to either of the mutable fields in a Period instance. These fields are truly encapsulated within the object.

有了新的构造函数和新的访问器，Period 实际上是不可变的。无论程序员多么恶毒或无能，都不可能违背一个时间段的开始时间不能在结束时间之后这一不变条件（除非使用诸如本地方法和反射之类的外部语言手段）。这是真的，因为除了 Period 本身之外，任何类都无法访问 Period 实例中的任何可变字段。这些字段真正封装在对象中。

In the accessors, unlike the constructor, it would be permissible to use the clone method to make the defensive copies. This is so because we know that the class of Period’s internal Date objects is java.util.Date, and not some untrusted subclass. That said, you are generally better off using a constructor or static factory to copy an instance, for reasons outlined in Item 13.

在访问器中，与构造函数不同，可以使用 clone 方法进行防御性复制。这是因为我们知道 Period 的内部 Date 对象的类是 `java.util.Date`，而不是某个不可信的子类。也就是说，基于 [Item-13](/Chapter-3/Chapter-3-Item-13-Override-clone-judiciously.md) 中列出的原因，一般情况下，最好使用构造函数或静态工厂来复制实例。

Defensive copying of parameters is not just for immutable classes. Any time you write a method or constructor that stores a reference to a client-provided object in an internal data structure, think about whether the client-provided object is potentially mutable. If it is, think about whether your class could tolerate a change in the object after it was entered into the data structure. If the answer is no, you must defensively copy the object and enter the copy into the data structure in place of the original. For example, if you are considering using a client-provided object reference as an element in an internal Set instance or as a key in an internal Map instance, you should be aware that the invariants of the set or map would be corrupted if the object were modified after it is inserted.

参数的防御性复制不仅适用于不可变类。在编写方法或构造函数时，如果要在内部数据结构中存储对客户端提供的对象的引用，请考虑客户端提供的对象是否可能是可变的。如果是，请考虑该对象进入数据结构之后，你的类是否能够容忍该对象发生更改。如果答案是否定的，则必须防御性地复制对象，并将副本输入到数据结构中，而不是原始正本。举个例子，如果你正在考虑使用由客户提供的对象引用作为内部 Set 实例的元素，或者作为内部 Map 实例的键, 就应该意识到如果这个对象在插入之后发生改变，Set 或者 Map 的约束条件就会遭到破坏。

The same is true for defensive copying of internal components prior to returning them to clients. Whether or not your class is immutable, you should think twice before returning a reference to an internal component that is mutable. Chances are, you should return a defensive copy. Remember that nonzero-length arrays are always mutable. Therefore, you should always make a defensive copy of an internal array before returning it to a client. Alternatively, you could return an immutable view of the array. Both of these techniques are shown in Item 15.

在将内部组件返回给客户端之前应对其进行防御性复制也是如此。无论你的类是否是不可变的，在返回对可变内部组件的引用之前，你都应该三思。很有可能，你应该返回一个防御性副本。记住，非零长度数组总是可变的。因此，在将内部数组返回给客户端之前，应该始终创建一个防御性的副本。或者，你可以返回数组的不可变视图。这两种技术都已经在 [Item-15](/Chapter-4/Chapter-4-Item-15-Minimize-the-accessibility-of-classes-and-members.md) 中演示过。

Arguably, the real lesson in all of this is that you should, where possible, use immutable objects as components of your objects so that you that don’t have to worry about defensive copying (Item 17). In the case of our Period example, use Instant (or LocalDateTime or ZonedDateTime), unless you’re using a release prior to Java 8. If you are using an earlier release, one option is to store the primitive long returned by Date.getTime() in place of a Date reference.

可以说，所有这些教训体现了，在可能的情况下，应该使用不可变对象作为对象的组件，这样就不必操心防御性复制（[Item-17](/Chapter-4/Chapter-4-Item-17-Minimize-mutability.md)）。在我们的 Period 示例中，使用 Instant（或 LocalDateTime 或 ZonedDateTime），除非你使用的是 Java 8 之前的版本。如果使用较早的版本，一个选项是存储 `Date.getTime()` 返回的 long 基本数据类型，而不是 Date 引用。

There may be a performance penalty associated with defensive copying and it isn’t always justified. If a class trusts its caller not to modify an internal component, perhaps because the class and its client are both part of the same package, then it may be appropriate to dispense with defensive copying. Under these circumstances, the class documentation should make it clear that the caller must not modify the affected parameters or return values.

防御性复制可能会带来性能损失，而且并不总是合理的。如果一个类信任它的调用者不会去修改内部组件，可能是因为类和它的客户端都是同一个包的一部分，那么就应该避免防御性复制。在这种情况下，类文档应该表明调用者不能修改受影响的参数或返回值。

Even across package boundaries, it is not always appropriate to make a defensive copy of a mutable parameter before integrating it into an object. There are some methods and constructors whose invocation indicates an explicit handoff of the object referenced by a parameter. When invoking such a method, the client promises that it will no longer modify the object directly. A method or constructor that expects to take ownership of a client-provided mutable object must make this clear in its documentation.

即使跨越包边界，在将可变参数集成到对象之前对其进行防御性复制也并不总是合适的。有一些方法和构造函数，它们的调用要求参数引用的对象要进行显式切换。当调用这样一个方法时，客户端承诺不再直接修改对象。希望拥有客户端提供的可变对象所有权的方法或构造函数必须在其文档中明确说明这一点。

Classes containing methods or constructors whose invocation indicates a transfer of control cannot defend themselves against malicious clients. Such classes are acceptable only when there is mutual trust between a class and its client or when damage to the class’s invariants would harm no one but the client. An example of the latter situation is the wrapper class pattern (Item 18). Depending on the nature of the wrapper class, the client could destroy the class’s invariants by directly accessing an object after it has been wrapped, but this typically would harm only the client.

包含方法或构造函数的类，如果其方法或构造函数的调用需要移交对象的控制权，就不能保护自己免受恶意客户端的攻击。只有当一个类和它的客户端之间相互信任，或者对类的不变量的破坏只会对客户端造成伤害时，这样的类才是可接受的。后一种情况的一个例子是包装类模式（[Item-18](/Chapter-4/Chapter-4-Item-18-Favor-composition-over-inheritance.md)）。根据包装类的性质，客户端可以在包装对象之后直接访问对象，从而破坏类的不变量，但这通常只会损害客户端。

In summary, if a class has mutable components that it gets from or returns to its clients, the class must defensively copy these components. If the cost of the copy would be prohibitive and the class trusts its clients not to modify the components inappropriately, then the defensive copy may be replaced by documentation outlining the client’s responsibility not to modify the affected components.

总而言之，如果一个类具有从客户端获取或返回给客户端的可变组件，则该类必须防御性地复制这些组件。如果复制的成本过高，并且类信任它的客户端不会不适当地修改组件，那么可以不进行防御性的复制，取而代之的是在文档中指明客户端的职责是不得修改受到影响的组件。

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-8/Chapter-8-Introduction.md)**
- **Previous Item（上一条目）：[Item 49: Check parameters for validity（检查参数的有效性）](/Chapter-8/Chapter-8-Item-49-Check-parameters-for-validity.md)**
- **Next Item（下一条目）：[Item 51: Design method signatures carefully（仔细设计方法签名）](/Chapter-8/Chapter-8-Item-51-Design-method-signatures-carefully.md)**
