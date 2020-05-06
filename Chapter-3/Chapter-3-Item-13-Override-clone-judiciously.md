## Chapter 3. Methods Common to All Objects（对象的通用方法）

### Item 13: Override clone judiciously（明智地覆盖 clone 方法）

The Cloneable interface was intended（目的） as a mixin interface (Item 20) for classes to advertise that they permit cloning. Unfortunately, it fails to serve this purpose. Its primary flaw（n. 瑕疵，缺点） is that it lacks a clone method, and Object’s clone method is protected. You cannot, without resorting（求助） to reflection (Item 65), invoke clone on an object merely（adv. 仅仅，只是） because it implements Cloneable.Even a reflective invocation may fail, because there is no guarantee（n. 保证；担保） that the object has an accessible clone method. Despite this flaw and many others, the facility（n. 设施；设备） is in reasonably wide use, so it pays to understand it. This item tells you how to implement a well-behaved clone method, discusses when it is appropriate to do so, and presents alternatives.

Cloneable 接口的目的是作为 mixin 接口（[Item-20](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-4/Chapter-4-Item-20-Prefer-interfaces-to-abstract-classes.md)），用于让类来宣称它们允许克隆。不幸的是，它没有达到这个目的。它的主要缺点是缺少 clone 方法，并且 Object 类的 clone 方法是受保护的。如果不求助于反射（[Item-65](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-9/Chapter-9-Item-65-Prefer-interfaces-to-reflection.md)），就不能仅仅因为对象实现了 Cloneable 就能调用 clone 方法。即使反射调用也可能失败，因为不能保证对象具有可访问的 clone 方法。尽管存在这样那样的缺陷，但该设施的使用范围相当广泛，因此理解它是值得的。本项目将告诉你如何实现行为良好的 clone 方法，讨论什么时候应该这样做，并提供替代方法。

**译注：mixin 是掺合，混合，糅合的意思，即可以将任意一个对象的全部或部分属性拷贝到另一个对象上。**

So what does Cloneable do, given that it contains no methods? It determines the behavior of Object’s protected clone implementation: if a class implements Cloneable, Object’s clone method returns a field-byfield copy of the object; otherwise it throws CloneNotSupportedException. This is a highly atypical use of interfaces and not one to be emulated. Normally, implementing an interface says something about what a class can do for its clients. In this case, it modifies the behavior of a protected method on a superclass.

如果 Cloneable 不包含任何方法，它会做什么呢？它决定 Object 的受保护克隆实现的行为：如果一个类实现了 Cloneable，对象的克隆方法返回对象的逐域拷贝；否则它会抛出 CloneNotSupportedException。这是接口的一种高度非典型使用，而不是可模仿的。通常，实现接口说明了类可以为其客户做些什么。在本例中，它修改了超类上受保护的方法行为。

Though the specification（n. 规格；说明书；详述） doesn’t say it, in practice, a class implementing Cloneable is expected to provide a properly（adv. 适当地；正确地；恰当地） functioning public clone method. In order to achieve（vt. 取得；获得；实现；） this, the class and all of its superclasses must obey a complex（adj. 复杂的；合成的）, unenforceable, thinly documented protocol. The resulting mechanism is fragile, dangerous, and extralinguistic（adj. 语言以外的；语言学以外的）: it creates objects without calling a constructor.

虽然规范没有说明，但是在实践中，一个实现 Cloneable 的类应该提供一个功能正常的公共 clone 方法。为了实现这一点，类及其所有超类必须遵守复杂的、不可强制执行的、文档很少的协议。产生的机制是脆弱的、危险的和非语言的：即它创建对象而不调用构造函数。

The general contract for the clone method is weak. Here it is, copied from the Object specification（n. 规格；说明书；详述） :

clone 方法的一般约定很薄弱。这里是从 Object 规范复制过来的：

Creates and returns a copy of this object. The precise meaning of “copy” may depend on the class of the object. The general intent（n. 意图；目的；含义，adj. 专心的；急切的；坚决的） is that, for any object x,the expression

创建并返回此对象的副本。「复制」的确切含义可能取决于对象的类别。一般的目的是，对于任何对象 x，表达式

```
x.clone() != x
```

will be true, and the expression

值将为 true，并且这个表达式

```
x.clone().getClass() == x.getClass()
```

will be true, but these are not absolute requirements（n. 要求；必要条件；）. While it is typically the case that

值将为 true，但这些不是绝对的必要条件。通常情况下

```
x.clone().equals(x)
```

will be true, this is not an absolute requirement.

值将为 true，但这些不是绝对的必要条件。

By convention（n. 大会；惯例；约定；协定；习俗）, the object returned by this method should be obtained（v. 获得） by calling super.clone. If a class and all of its superclasses (except Object) obey this convention, it will be the case that

按照惯例，这个方法返回的对象应该通过调用 super.clone 来获得。如果一个类和它的所有超类（对象除外）都遵守这个约定，那么情况就是这样

```
x.clone().getClass() == x.getClass().
```

By convention, the returned object should be independent of the object being cloned. To achieve this independence, it may be necessary to modify one or more fields of the object returned by super.clone before returning it.

按照惯例，返回的对象应该独立于被克隆的对象。为了实现这种独立性，可能需要在 super.clone 返回对象前修改其一个或多个字段。

This mechanism is vaguely similar to constructor chaining, except that it isn’t enforced: if a class’s clone method returns an instance that is not obtained by calling super.clone but by calling a constructor, the compiler won’t complain, but if a subclass of that class calls super.clone, the resulting object will have the wrong class, preventing the subclass from clone method from working properly. If a class that overrides clone is final, this convention may be safely ignored, as there are no subclasses to worry about. But if a final class has a clone method that does not invoke super.clone, there is no reason for the class to implement Cloneable, as it doesn’t rely on the behavior of Object’s clone implementation.

这种机制有点类似于构造函数链接，只是没有强制执行：如果一个类的克隆方法返回的实例不是通过调用 super.clone 而是通过调用构造函数获得的，编译器不会抱怨，但是如果这个类的一个子类调用 super.clone,由此产生的对象将有错误的类,防止子类克隆方法从正常工作。如果覆盖克隆的类是 final 的，那么可以安全地忽略这个约定，因为不需要担心子类。但是如果 final 类有一个不调用 super 的克隆方法。类没有理由实现 Cloneable，因为它不依赖于对象克隆实现的行为。

Suppose you want to implement Cloneable in a class whose superclass provides a well-behaved clone method. First call super.clone. The object you get back will be a fully functional replica of the original. Any fields declared in your class will have values identical to those of the original. If every field contains a primitive value or a reference to an immutable object, the returned object may be exactly what you need, in which case no further processing is necessary. This is the case, for example, for the PhoneNumber class in Item 11, but note that **immutable classes should never provide a clone method** because it would merely encourage wasteful copying. With that caveat, here’s how a clone method for PhoneNumber would look:

假设你希望在一个类中实现 Cloneable，该类的超类提供了一个表现良好的克隆方法。第一个叫 super.clone。返回的对象将是原始对象的完整功能副本。类中声明的任何字段都具有与原始字段相同的值。如果每个字段都包含一个基本类型或对不可变对象的引用，那么返回的对象可能正是你所需要的，在这种情况下不需要进一步的处理。例如，对于[Item-11](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-3/Chapter-3-Item-11-Always-override-hashCode-when-you-override-equals.md)中的 PhoneNumber 类就是这样，但是要注意，**不可变类永远不应该提供克隆方法**，因为它只会鼓励浪费复制。有了这个警告，以下是 PhoneNumber 的克隆方法的外观：

```
// Clone method for class with no references to mutable state
@Override public PhoneNumber clone() {
    try {
        return (PhoneNumber) super.clone();
    } catch (CloneNotSupportedException e) {
        throw new AssertionError(); // Can't happen
    }
}
```

In order for this method to work, the class declaration for PhoneNumber would have to be modified to indicate that it implements Cloneable. Though Object’s clone method returns Object, this clone method returns PhoneNumber. It is legal and desirable to do this because Java supports covariant return types. In other words, an overriding method’s return type can be a subclass of the overridden method’s return type. This eliminates the need for casting in the client. We must cast the result of super.clone from Object to PhoneNumber before returning it, but the cast is guaranteed to succeed.

为了让这个方法工作，必须修改 PhoneNumber 的类声明，以表明它实现了 Cloneable。虽然 Object 的 clone 方法返回 Object，但是这个 clone 方法返回 PhoneNumber。这样做是合法的，也是可取的，因为 Java 支持协变返回类型。换句话说，覆盖方法的返回类型可以是被覆盖方法的返回类型的子类。这样就不需要在客户端中进行强制转换。我们必须在 super.clone 返回前将结果转换为 PhoneNumber 类型，这类强制转换肯定会成功。

The call to super.clone is contained in a try-catch block. This is because Object declares its clone method to throw CloneNotSupportedException, which is a checked exception. Because PhoneNumber implements Cloneable, we know the call to super.clone will succeed. The need for this boilerplate indicates that CloneNotSupportedException should have been unchecked (Item 71).

对 super.clone 的调用包含在 try-catch 块中。这是因为 Object 声明其克隆方法来抛出 CloneNotSupportedException。因为 PhoneNumber 实现了 Cloneable，所以我们知道对 super.clone 的调用将会成功。这个样板文件的需要表明 CloneNotSupportedException 应该是被选中的（[Item-71](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-10/Chapter-10-Item-71-Avoid-unnecessary-use-of-checked-exceptions.md)）。

If an object contains fields that refer to mutable objects, the simple clone implementation shown earlier can be disastrous. For example, consider the Stack class in Item 7:

如果对象包含引用可变对象的字段，前面所示的简单克隆实现可能是灾难性的。例如，考虑 [Item-7](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-2/Chapter-2-Item-7-Eliminate-obsolete-object-references.md) 中的堆栈类：

```
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
      this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // Eliminate obsolete reference
        return result;
    }

    // Ensure space for at least one more element.
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

Suppose you want to make this class cloneable. If the clone method merely（adv. 仅仅，只不过；只是） returns super.clone(), the resulting Stack instance will have the correct value in its size field, but its elements field will refer to the same array as the original Stack instance. Modifying the original will destroy the invariants in the clone and vice versa. You will quickly find that your program produces nonsensical results or throws a NullPointerException.

假设你想让这个类是可克隆的。如果克隆方法只返回 super.clone()，则结果堆栈实例在其大小字段中将有正确的值，但其元素字段将引用与原始堆栈实例相同的数组。修改初始值将破坏克隆中的不变量，反之亦然。你将很快发现你的程序产生了无意义的结果或抛出 NullPointerException。

This situation could never occur as a result of calling the sole constructor in the Stack class. In effect, the clone method functions as a constructor;you must ensure that it does no harm to the original object and that it properly establishes invariants on the clone. In order for the clone method on Stack to work properly, it must copy the internals of the stack. The easiest way to do this is to call clone recursively on the elements array:

由于调用堆栈类中的唯一构造函数，这种情况永远不会发生。实际上，clone 方法充当构造函数;你必须确保它不会对原始对象造成伤害，并且在克隆上正确地建立不变量。为了使堆栈上的克隆方法正常工作，它必须复制堆栈的内部。最简单的方法是在元素数组上递归地调用 clone：

```
// Clone method for class with references to mutable state
@Override
public Stack clone() {
    try {
        Stack result = (Stack) super.clone();
        result.elements = elements.clone();
        return result;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

Note that we do not have to cast the result of elements.clone to Object[]. Calling clone on an array returns an array whose runtime and compile-time types are identical to those of the array being cloned. This is the preferred idiom to duplicate an array. In fact, arrays are the sole compelling use of the clone facility（n. 设施；设备；容易；灵巧）.

注意，我们不需要将 `elements.clone` 的结果强制转换到 `Object[]`。在数组上调用 clone 将返回一个数组，该数组的运行时和编译时类型与被克隆的数组相同。这是复制数组的首选习惯用法。实际上，数组是 clone 工具唯一引人注目的用途。

Note also that the earlier solution would not work if the elements field were final because clone would be prohibited from assigning a new value to the field. This is a fundamental problem: like serialization, the Cloneable architecture is incompatible with normal use of final fields referring to mutable objects, except in cases where the mutable objects may be safely shared between an object and its clone. In order to make a class cloneable, it may be necessary to remove final modifiers from some fields.

还要注意，如果元素字段是 final 的，早期的解决方案就无法工作，因为克隆将被禁止为字段分配新值。这是一个基本问题：与序列化一样，可克隆体系结构与正常使用引用可变对象的 final 字段不兼容，除非在对象与其克隆对象之间可以安全地共享可变对象。为了使类可克隆，可能需要从某些字段中删除最终修饰符。

It is not always sufficient（adj. 足够的；充分的） merely to call clone recursively. For example,suppose you are writing a clone method for a hash table whose internals consist of an array of buckets, each of which references the first entry in a linked list of key-value pairs. For performance, the class implements its own lightweight singly linked list instead of using java.util.LinkedList internally:

仅仅递归地调用克隆并不总是足够的。例如，假设你正在为 hash 表编写一个克隆方法， hash 表的内部由一组 bucket 组成，每个 bucket 引用键-值对链表中的第一个条目。为了提高性能，类实现了自己的轻量级单链表，而不是在内部使用 `java.util.LinkedList`：

```
public class HashTable implements Cloneable {
    private Entry[] buckets = ...;

    private static class Entry {
        final Object key;
        Object value;
        Entry next;

        Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }
    } ... // Remainder omitted
}
```

Suppose you merely clone the bucket array recursively, as we did for Stack:

假设你只是递归地克隆 bucket 数组，就像我们对 Stack 所做的那样：

```
// Broken clone method - results in shared mutable state!
@Override
public HashTable clone() {
    try {
        HashTable result = (HashTable) super.clone();
        result.buckets = buckets.clone();
        return result;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

Though the clone has its own bucket array, this array references the same linked lists as the original, which can easily cause nondeterministic behavior in both the clone and the original. To fix this problem, you’ll have to copy the linked list that comprises each bucket. Here is one common approach:

尽管克隆具有自己的 bucket 数组，但该数组引用的链接列表与原始链表相同，这很容易导致克隆和原始的不确定性行为。要解决这个问题，你必须复制包含每个 bucket 的链表。这里有一个常见的方法：

```
// Recursive clone method for class with complex mutable state
public class HashTable implements Cloneable {

private Entry[] buckets = ...;

    private static class Entry {
        final Object key;
        Object value;
        Entry next;

        Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }

        // Recursively copy the linked list headed by this Entry
        Entry deepCopy() {
            return new Entry(key, value,next == null ? null : next.deepCopy());
        }
    }

    @Override
    public HashTable clone() {
        try {
            HashTable result = (HashTable) super.clone();
            result.buckets = new Entry[buckets.length];

            for (int i = 0; i < buckets.length; i++)
                if (buckets[i] != null)
                    result.buckets[i] = buckets[i].deepCopy();

            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    } ... // Remainder omitted
}
```

The private class HashTable.Entry has been augmented to support a “deep copy” method. The clone method on HashTable allocates a new buckets array of the proper size and iterates over the original buckets array,deep-copying each nonempty bucket. The deepCopy method on Entry invokes itself recursively to copy the entire linked list headed by the entry. While this technique is cute and works fine if the buckets aren’t too long, it is not a good way to clone a linked list because it consumes one stack frame for each element in the list. If the list is long, this could easily cause a stack overflow. To prevent this from happening, you can replace the recursion in deepCopy with iteration:

私有类 HashTable.Entry 已经被增强为支持「深度复制」方法。HashTable 上的 clone 方法分配一个大小合适的新 bucket 数组，并遍历原始 bucket 数组，深度复制每个非空 bucket。条目上的 deepCopy 方法会递归地调用自己来复制以条目开头的整个链表。虽然这种技术很可爱，而且如果 bucket 不太长也可以很好地工作，但是克隆链表并不是一个好方法，因为它为链表中的每个元素消耗一个堆栈帧。如果列表很长，很容易导致堆栈溢出。为了防止这种情况的发生，你可以用迭代替换 deepCopy 中的递归：

```
// Iteratively copy the linked list headed by this Entry
Entry deepCopy() {
    Entry result = new Entry(key, value, next);
    for (Entry p = result; p.next != null; p = p.next)
        p.next = new Entry(p.next.key, p.next.value, p.next.next);
    return result;
}
```

A final approach to cloning complex mutable objects is to call super.clone, set all of the fields in the resulting object to their initial state,and then call higher-level methods to regenerate the state of the original object.In the case of our HashTable example, the buckets field would be initialized to a new bucket array, and the put(key, value) method (not shown) would be invoked for each key-value mapping in the hash table being cloned. This approach typically yields a simple, reasonably elegant clone method that does not run as quickly as one that directly manipulates the innards of the clone. While this approach is clean, it is antithetical to the whole Cloneable architecture because it blindly overwrites the field-by-field object copy that forms the basis of the architecture.

克隆复杂可变对象的最后一种方法是调用 super.clone，将结果对象中的所有字段设置为初始状态，然后调用更高级别的方法重新生成原始对象的状态。在我们的 HashTable 示例中，bucket 字段将初始化为一个新的 bucket 数组，并且对于克隆的 hash 表中的每个键值映射将调用 put(key, value)方法（未显示）。这种方法通常产生一个简单、相当优雅的克隆方法，它的运行速度不如直接操作克隆的内部的方法快。虽然这种方法很简洁，但它与整个可克隆体系结构是对立的，因为它盲目地覆盖了构成体系结构基础的逐字段对象副本。

Like a constructor, a clone method must never invoke an overridable method on the clone under construction (Item 19). If clone invokes a method that is overridden in a subclass, this method will execute before the subclass has had a chance to fix its state in the clone, quite possibly leading to corruption in the clone and the original. Therefore, the put(key, value) method discussed in the previous paragraph should be either final or private. (If it is private, it is presumably the “helper method” for a nonfinal public method.)

与构造函数一样，克隆方法决不能在正在构建的克隆上调用可覆盖方法（[Item-19](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-4/Chapter-4-Item-19-Design-and-document-for-inheritance-or-else-prohibit-it.md)）。如果 clone 调用一个在子类中被重写的方法，这个方法将在子类有机会修复其在克隆中的状态之前执行，很可能导致克隆和原始的破坏。因此，前一段中讨论的 put(key, value)方法应该是 final 或 private 方法。（如果它是私有的，那么它可能是非最终公共方法的「助手方法」。)

Object’s clone method is declared to throw CloneNotSupportedException, but overriding methods need not. **Public clone methods should omit the throws clause,** as methods that don’t throw checked exceptions are easier to use (Item 71).

对象的 clone 方法被声明为抛出 CloneNotSupportedException，但是重写方法不需要。**公共克隆方法应该省略 throw 子句，** 作为不抛出受控异常的方法更容易使用（[Item-71](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-10/Chapter-10-Item-71-Avoid-unnecessary-use-of-checked-exceptions.md)）。

You have two choices when designing a class for inheritance (Item 19), but whichever one you choose, the class should not implement Cloneable. You may choose to mimic the behavior of Object by implementing a properly functioning protected clone method that is declared to throw CloneNotSupportedException. This gives subclasses the freedom to implement Cloneable or not, just as if they extended Object directly.Alternatively, you may choose not to implement a working clone method, and to prevent subclasses from implementing one, by providing the following degenerate clone implementation:

在为继承设计类时，你有两种选择（[Item-19](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-4/Chapter-4-Item-19-Design-and-document-for-inheritance-or-else-prohibit-it.md)），但是无论你选择哪一种，类都不应该实现 Cloneable。你可以选择通过实现一个功能正常的受保护克隆方法来模拟对象的行为，该方法声明为抛出 CloneNotSupportedException。这给子类实现 Cloneable 或不实现 Cloneable 的自由，就像它们直接扩展对象一样。或者，你可以选择不实现工作克隆方法，并通过提供以下简并克隆实现来防止子类实现一个工作克隆方法：

```
// clone method for extendable class not supporting Cloneable
@Override
protected final Object clone() throws CloneNotSupportedException {
    throw new CloneNotSupportedException();
}
```

There is one more detail that bears noting. If you write a thread-safe class that implements Cloneable, remember that its clone method must be properly synchronized, just like any other method (Item 78). Object’s clone method is not synchronized, so even if its implementation is otherwise satisfactory, you may have to write a synchronized clone method that returns super.clone().

还有一个细节需要注意。如果你编写了一个实现了 Cloneable 的线程安全类，请记住它的克隆方法必须正确同步，就像其他任何方法一样（[Item-78](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-11/Chapter-11-Item-78-Synchronize-access-to-shared-mutable-data.md)）。对象的克隆方法不是同步的，因此即使它的实现在其他方面是令人满意的，你也可能需要编写一个返回 super.clone()的同步克隆方法。

To recap, all classes that implement Cloneable should override clone with a public method whose return type is the class itself. This method should first call super.clone, then fix any fields that need fixing. Typically, this means copying any mutable objects that comprise the internal “deep structure” of the object and replacing the clone’s references to these objects with references to their copies. While these internal copies can usually be made by calling clone recursively, this is not always the best approach. If the class contains only primitive fields or references to immutable objects, then it is likely the case that no fields need to be fixed. There are exceptions to this rule. For example, a field representing a serial number or other unique ID will need to be fixed even if it is primitive or immutable.

回顾一下，所有实现 Cloneable 的类都应该使用一个返回类型为类本身的公共方法覆盖 clone。这个方法应该首先调用 super.clone，然后修复任何需要修复的字段。通常，这意味着复制任何包含对象内部「深层结构」的可变对象，并将克隆对象对这些对象的引用替换为对其副本的引用。虽然这些内部副本通常可以通过递归调用 clone 来实现，但这并不总是最好的方法。如果类只包含基元字段或对不可变对象的引用，那么很可能不需要修复任何字段。这条规则也有例外。例如，表示序列号或其他唯一 ID 的字段需要固定，即使它是原始的或不可变的。

Is all this complexity really necessary? Rarely. If you extend a class that already implements Cloneable, you have little choice but to implement a well-behaved clone method. Otherwise, you are usually better off providing an alternative means of object copying. A better approach to object copying is to provide a copy constructor or copy factory. A copy constructor is simply a constructor that takes a single argument whose type is the class containing the constructor, for example,

所有这些复杂性真的有必要吗？很少。如果你扩展了一个已经实现了 Cloneable 的类，那么除了实现行为良好的克隆方法之外，你别无选择。否则，最好提供对象复制的替代方法。一个更好的对象复制方法是提供一个复制构造函数或复制工厂。复制构造函数是一个简单的构造函数，它接受单个参数，其类型是包含构造函数的类，例如，

```
// Copy constructor
public Yum(Yum yum) { ... };
```

A copy factory is the static factory (Item 1) analogue of a copy constructor:

复制工厂是复制构造函数的静态工厂（[Item-1](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-2/Chapter-2-Item-1-Consider-static-factory-methods-instead-of-constructors.md)）类似物：

```
// Copy factory
public static Yum newInstance(Yum yum) { ... };
```

The copy constructor approach and its static factory variant have many advantages over Cloneable/clone: they don’t rely on a risk-prone extralinguistic object creation mechanism; they don’t demand unenforceable adherence to thinly documented conventions; they don’t conflict with the proper use of final fields; they don’t throw unnecessary checked exceptions; and they don’t require casts.

复制构造函数方法及其静态工厂变体与克隆/克隆相比有许多优点：它们不依赖于易发生风险的语言外对象创建机制；他们不要求无法强制执行的约定；它们与最终字段的正确使用不冲突；它们不会抛出不必要的检查异常；而且不需要强制类型转换。

Furthermore, a copy constructor or factory can take an argument whose type is an interface implemented by the class. For example, by convention all generalpurpose collection implementations provide a constructor whose argument is of type Collection or Map. Interface-based copy constructors and factories,more properly known as conversion constructors and conversion factories, allow the client to choose the implementation type of the copy rather than forcing the client to accept the implementation type of the original. For example, suppose you have a HashSet, s, and you want to copy it as a TreeSet. The clone method can’t offer this functionality, but it’s easy with a conversion constructor:new TreeSet<>(s).

此外，复制构造函数或工厂可以接受类型为类实现的接口的参数。例如，按照约定，所有通用集合实现都提供一个构造函数，其参数为 collection 或 Map 类型。基于接口的复制构造函数和工厂（更确切地称为转换构造函数和转换工厂）允许客户端选择副本的实现类型，而不是强迫客户端接受原始的实现类型。例如，假设你有一个 HashSet s，并且希望将它复制为 TreeSet。克隆方法不能提供这种功能，但是使用转换构造函数很容易：new TreeSet<>(s)。

Given all the problems associated（adj. 关联的；联合的） with Cloneable, new interfaces should not extend it, and new extendable classes should not implement it. While it’s less harmful for final classes to implement Cloneable, this should be viewed as a performance optimization, reserved for the rare cases where it is justified (Item 67). As a rule, copy functionality is best provided by constructors or factories. A notable exception to this rule is arrays, which are best copied with the clone method.

考虑到与 Cloneable 相关的所有问题，新的接口不应该扩展它，新的可扩展类不应该实现它。虽然 final 类实现 Cloneable 的危害要小一些，但这应该被视为一种性能优化，仅在极少数情况下（[Item-67](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-9/Chapter-9-Item-67-Optimize-judiciously.md)）是合理的。通常，复制功能最好由构造函数或工厂提供。这个规则的一个明显的例外是数组，最好使用 clone 方法来复制数组。

---
**[Back to contents of the chapter（返回章节目录）](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-3/Chapter-3-Introduction.md)**
- **Previous Item（上一条目）：[Item 12: Always override toString（始终覆盖 toString 方法）](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-3/Chapter-3-Item-12-Always-override-toString.md)**
- **Next Item（下一条目）：[Item 14: Consider implementing Comparable（考虑实现 Comparable 接口）](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-3/Chapter-3-Item-14-Consider-implementing-Comparable.md)**
