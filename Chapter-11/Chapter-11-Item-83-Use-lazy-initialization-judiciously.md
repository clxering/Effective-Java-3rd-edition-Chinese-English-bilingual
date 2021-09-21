## Chapter 11. Concurrency（并发）

### Item 83: Use lazy initialization judiciously（明智地使用延迟初始化）

Lazy initialization is the act of delaying the initialization of a field until its value is needed. If the value is never needed, the field is never initialized. This technique is applicable to both static and instance fields. While lazy initialization is primarily an optimization, it can also be used to break harmful circularities in class and instance initialization [Bloch05, Puzzle 51].

延迟初始化是延迟字段的初始化，直到需要它的值。如果不需要该值，则不会初始化字段。这种技术既适用于静态字段，也适用于实例字段。虽然延迟初始化主要作为一种优化手段，它还可用于避免类与实例初始化中循环依赖的问题 [Bloch05, Puzzle 51]。

As is the case for most optimizations, the best advice for lazy initialization is “don’t do it unless you need to” (Item 67). Lazy initialization is a double-edged sword. It decreases the cost of initializing a class or creating an instance, at the expense of increasing the cost of accessing the lazily initialized field. Depending on what fraction of these fields eventually require initialization, how expensive it is to initialize them, and how often each one is accessed once initialized, lazy initialization can (like many “optimizations”) actually harm performance.

与大多数优化一样，延迟初始化的最佳建议是「除非需要，否则不要这样做」(第67项)。延迟初始化是一把双刃剑。它降低了初始化类或创建实例的成本，代价是增加了访问延迟初始化字段的成本。根据这些字段中最终需要初始化的部分、初始化它们的开销以及初始化后访问每个字段的频率，延迟初始化实际上会损害性能（就像许多「优化」一样）。

That said, lazy initialization has its uses. If a field is accessed only on a fraction of the instances of a class and it is costly to initialize the field, then lazy initialization may be worthwhile. The only way to know for sure is to measure the performance of the class with and without lazy initialization.

延迟初始化也有它的用途。如果一个字段只在类的一小部分实例上访问，并且初始化该字段的代价很高，那么延迟初始化可能是值得的。唯一确定的方法是以使用和不使用延迟初始化的效果对比来度量类的性能。

In the presence of multiple threads, lazy initialization is tricky. If two or more threads share a lazily initialized field, it is critical that some form of synchronization be employed, or severe bugs can result (Item 78). All of the initialization techniques discussed in this item are thread-safe.

在存在多个线程的情况下，使用延迟初始化很棘手。如果两个或多个线程共享一个延迟初始化的字段，那么必须使用某种形式的同步，否则会导致严重的错误（[Item-78](/Chapter-11/Chapter-11-Item-78-Synchronize-access-to-shared-mutable-data.md)）。本条目讨论的所有初始化技术都是线程安全的。

**Under most circumstances, normal initialization is preferable to lazy initialization.** Here is a typical declaration for a normally initialized instance field. Note the use of the final modifier (Item 17):

**在大多数情况下，常规初始化优于延迟初始化。** 下面是一个使用常规初始化的实例字段的典型声明。注意 final 修饰符的使用（[Item-17](/Chapter-4/Chapter-4-Item-17-Minimize-mutability.md)）：

```
// Normal initialization of an instance field
private final FieldType field = computeFieldValue();
```

**If you use lazy initialization to break an initialization circularity, use a synchronized accessor** because it is the simplest, clearest alternative:

**如果您使用延迟初始化来取代初始化 circularity，请使用同步访问器**，因为它是最简单、最清晰的替代方法：

```
// Lazy initialization of instance field - synchronized accessor
private FieldType field;
private synchronized FieldType getField() {
    if (field == null)
        field = computeFieldValue();
    return field;
}
```

Both of these idioms (normal initialization and lazy initialization with a synchronized accessor) are unchanged when applied to static fields, except that you add the static modifier to the field and accessor declarations.

这两种习惯用法（使用同步访问器进行常规初始化和延迟初始化）在应用于静态字段时都没有改变，只是在字段和访问器声明中添加了 static 修饰符。

**If you need to use lazy initialization for performance on a static field, use the lazy initialization holder class idiom.** This idiom exploits the guarantee that a class will not be initialized until it is used [JLS, 12.4.1]. Here’s how it looks:

**如果需要在静态字段上使用延迟初始化来提高性能，use the lazy initialization holder class idiom.** 这个用法可保证一个类在使用之前不会被初始化 [JLS, 12.4.1]。它是这样的：

```
// Lazy initialization holder class idiom for static fields
private static class FieldHolder {
    static final FieldType field = computeFieldValue();
}
private static FieldType getField() { return FieldHolder.field; }
```

When getField is invoked for the first time, it reads FieldHolder.field for the first time, causing the initialization of the FieldHolder class. The beauty of this idiom is that the getField method is not synchronized and performs only a field access, so lazy initialization adds practically nothing to the cost of access. A typical VM will synchronize field access only to initialize the class. Once the class is initialized, the VM patches the code so that subsequent access to the field does not involve any testing or synchronization.

第一次调用 getField 时，它执行 FieldHolder.field，导致初始化 FieldHolder 类。这个习惯用法的优点是 getField 方法不是同步的，只执行字段访问，所以延迟初始化实际上不会增加访问成本。典型的 VM 只会同步字段访问来初始化类。初始化类之后，VM 会对代码进行修补，这样对字段的后续访问就不会涉及任何测试或同步。

**If you need to use lazy initialization for performance on an instance field, use the double-check idiom.** This idiom avoids the cost of locking when accessing the field after initialization (Item 79). The idea behind the idiom is to check the value of the field twice (hence the name double-check): once without locking and then, if the field appears to be uninitialized, a second time with locking. Only if the second check indicates that the field is uninitialized does the call initialize the field. Because there is no locking once the field is initialized, it is critical that the field be declared volatile (Item 78). Here is the idiom:

如果需要使用延迟初始化来提高实例字段的性能，请使用双重检查模式。这个模式避免了初始化后访问字段时的锁定成本（[Item-79](/Chapter-11/Chapter-11-Item-79-Avoid-excessive-synchronization.md)）。这个模式背后的思想是两次检查字段的值（因此得名 double check）：一次没有锁定，然后，如果字段没有初始化，第二次使用锁定。只有当第二次检查指示字段未初始化时，调用才初始化字段。由于初始化字段后没有锁定，因此将字段声明为 volatile 非常重要（[Item-78](/Chapter-11/Chapter-11-Item-78-Synchronize-access-to-shared-mutable-data.md)）。下面是这个模式的示例：

```
// Double-check idiom for lazy initialization of instance fields
private volatile FieldType field;
private FieldType getField() {
    FieldType result = field;
    if (result == null) { // First check (no locking)
        synchronized(this) {
            if (field == null) // Second check (with locking)
                field = result = computeFieldValue();
        }
    }
    return result;
}
```

This code may appear a bit convoluted. In particular, the need for the local variable (result) may be unclear. What this variable does is to ensure that field is read only once in the common case where it’s already initialized.

这段代码可能看起来有点复杂。特别是不清楚是否需要局部变量（result）。该变量的作用是确保 field 在已经初始化的情况下只读取一次。

While not strictly necessary, this may improve performance and is more elegant by the standards applied to low-level concurrent programming. On my machine, the method above is about 1.4 times as fast as the obvious version without a local variable. While you can apply the double-check idiom to static fields as well, there is no reason to do so: the lazy initialization holder class idiom is a better choice.

虽然不是严格必需的，但这可能会提高性能，而且与低级并发编程相比，这更优雅。在我的机器上，上述方法的速度大约是没有局部变量版本的 1.4 倍。虽然您也可以将双重检查模式应用于静态字段，但是没有理由这样做：the lazy initialization holder class idiom is a better choice.

Two variants of the double-check idiom bear noting. Occasionally, you may need to lazily initialize an instance field that can tolerate repeated initialization. If you find yourself in this situation, you can use a variant of the double-check idiom that dispenses with the second check. It is, not surprisingly, known as the single-check idiom. Here is how it looks. Note that field is still declared volatile:

双重检查模式的两个变体值得注意。有时候，您可能需要延迟初始化一个实例字段，该字段可以容忍重复初始化。如果您发现自己处于这种情况，您可以使用双重检查模式的变体来避免第二个检查。毫无疑问，这就是所谓的「单检查」模式。它是这样的。注意，field 仍然声明为 volatile：

```
// Single-check idiom - can cause repeated initialization!
private volatile FieldType field;
private FieldType getField() {
    FieldType result = field;
    if (result == null)
        field = result = computeFieldValue();
    return result;
}
```

All of the initialization techniques discussed in this item apply to primitive fields as well as object reference fields. When the double-check or single-check idiom is applied to a numerical primitive field, the field’s value is checked against 0 (the default value for numerical primitive variables) rather than null.

本条目中讨论的所有初始化技术都适用于基本字段和对象引用字段。当双检查或单检查模式应用于数值基本类型字段时，将根据 0（数值基本类型变量的默认值）而不是 null 检查字段的值。

If you don’t care whether every thread recalculates the value of a field, and the type of the field is a primitive other than long or double, then you may choose to remove the volatile modifier from the field declaration in the single-check idiom. This variant is known as the racy single-check idiom. It speeds up field access on some architectures, at the expense of additional initializations (up to one per thread that accesses the field). This is definitely an exotic technique, not for everyday use.

如果您不关心每个线程是否都会重新计算字段的值，并且字段的类型是 long 或 double 之外的基本类型，那么您可以选择在单检查模式中从字段声明中删除 volatile 修饰符。这种变体称为原生单检查模式。它加快了某些架构上的字段访问速度，代价是需要额外的初始化（每个访问该字段的线程最多需要一个初始化）。这绝对是一种奇特的技术，不是日常使用的。

In summary, you should initialize most fields normally, not lazily. If you must initialize a field lazily in order to achieve your performance goals or to break a harmful initialization circularity, then use the appropriate lazy initialization technique. For instance fields, it is the double-check idiom; for static fields, the lazy initialization holder class idiom. For instance fields that can tolerate repeated initialization, you may also consider the single-check idiom.

总之，您应该正常初始化大多数字段，而不是延迟初始化。如果必须延迟初始化字段以实现性能目标或 break a harmful initialization circularity，则使用适当的延迟初始化技术。对于字段，使用双重检查模式；对于静态字段，the lazy initialization holder class idiom. 例如，可以容忍重复初始化的实例字段，您还可以考虑单检查模式。

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-11/Chapter-11-Introduction.md)**
- **Previous Item（上一条目）：[Item 82: Document thread safety（文档应包含线程安全属性）](/Chapter-11/Chapter-11-Item-82-Document-thread-safety.md)**
- **Next Item（下一条目）：[Item 84: Don’t depend on the thread scheduler（不要依赖线程调度器）](/Chapter-11/Chapter-11-Item-84-Don’t-depend-on-the-thread-scheduler.md)**
