## Chapter 8. Methods（方法）

### Item 52: Use overloading judiciously（明智地使用重载）

The following program is a well-intentioned attempt to classify collections according to whether they are sets, lists, or some other kind of collection:

下面的程序是一个善意的尝试，根据一个 Collection 是 Set、List 还是其他的集合类型来进行分类：

```Java
// Broken! - What does this program print?
public class CollectionClassifier {
    public static String classify(Set<?> s) {
        return "Set";
    }

    public static String classify(List<?> lst) {
        return "List";
    }

    public static String classify(Collection<?> c) {
        return "Unknown Collection";
    }

    public static void main(String[] args) {
        Collection<?>[] collections = {
            new HashSet<String>(),new ArrayList<BigInteger>(),new HashMap<String, String>().values()
        };
        for (Collection<?> c : collections)
            System.out.println(classify(c));
    }
}
```

You might expect this program to print Set, followed by List and Unknown Collection, but it doesn’t. It prints Unknown Collection three times. Why does this happen? Because the classify method is overloaded, and **the choice of which overloading to invoke is made at compile time.** For all three iterations of the loop, the compile-time type of the parameter is the same: `Collection<?>`. The runtime type is different in each iteration, but this does not affect the choice of overloading. Because the compile-time type of the parameter is `Collection<?>`, the only applicable overloading is the third one, `classify(Collection<?>)`, and this overloading is invoked in each iteration of the loop.

你可能期望这个程序打印 Set，然后是 List 和 Unknown Collection，但是它没有这样做。它打印 Unknown Collection 三次。为什么会这样？因为 classify 方法被重载，并且 **在编译时就决定了要调用哪个重载。** 对于循环的三个迭代过程，参数的编译时类型是相同的：`Collection<?>`。运行时类型在每个迭代中是不同的，但这并不影响重载的选择。因为参数的编译时类型是 `Collection<?>`，唯一适用的重载是第三个，`classify(Collection<?>)`，这个重载在循环的每个迭代过程中都会调用。

The behavior of this program is counterintuitive because **selection among overloaded methods is static, while selection among overridden methods is dynamic.** The correct version of an overridden method is chosen at runtime, based on the runtime type of the object on which the method is invoked. As a reminder, a method is overridden when a subclass contains a method declaration with the same signature as a method declaration in an ancestor. If an instance method is overridden in a subclass and this method is invoked on an instance of the subclass, the subclass’s overriding method executes, regardless of the compile-time type of the subclass instance. To make this concrete, consider the following program:

这个程序的行为违反常规，因为 **重载方法的选择是静态的，而覆盖法的选择是动态的。** 在运行时根据调用方法的对象的运行时类型选择覆盖方法的正确版本。提醒一下，当子类包含与祖先中的方法声明具有相同签名的方法声明时，方法将被覆盖。如果在子类中覆盖实例方法，并且在子类的实例上调用此方法，则执行子类的覆盖方法，而不管子类实例的编译时类型如何。为了更具体的说明，考虑以下程序:

```Java
class Wine {
    String name() { return "wine"; }
}

class SparklingWine extends Wine {
    @Override
    String name() { return "sparkling wine"; }
}

class Champagne extends SparklingWine {
    @Override
    String name() { return "champagne"; }
}

public class Overriding {
    public static void main(String[] args) {
        List<Wine> wineList = List.of(new Wine(), new SparklingWine(), new Champagne());
    for (Wine wine : wineList)
        System.out.println(wine.name());
    }
}
```

The name method is declared in class Wine and overridden in subclasses SparklingWine and Champagne. As you would expect, this program prints out wine, sparkling wine, and champagne, even though the compiletime type of the instance is Wine in each iteration of the loop. The compile-time type of an object has no effect on which method is executed when an overridden method is invoked; the “most specific” overriding method always gets executed. Compare this to overloading, where the runtime type of an object has no effect on which overloading is executed; the selection is made at compile time, based entirely on the compile-time types of the parameters.

name 方法在 Wine 类中声明，并在 SparklingWine 和 Champagne 子类中重写。正如你所期望的，这个程序打印出 wine、sparkling 和 champagne，即使实例的编译时类型是循环每次迭代中的 wine。对象的编译时类型对调用覆盖方法时执行的方法没有影响；「最特定的」覆盖方法总是被执行。将此与重载进行比较，在重载中，对象的运行时类型对执行重载没有影响；选择是在编译时进行的，完全基于参数的编译时类型。

In the CollectionClassifier example, the intent of the program was to discern the type of the parameter by dispatching automatically to the appropriate method overloading based on the runtime type of the parameter, just as the name method did in the Wine example. Method overloading simply does not provide this functionality. Assuming a static method is required, the best way to fix the CollectionClassifier program is to replace all three overloadings of classify with a single method that does explicit instanceof tests:

在 CollectionClassifier 示例中，程序的目的是通过根据参数的运行时类型自动分派到适当的方法重载来识别参数的类型，就像 Wine 示例中的 name 方法所做的那样。方法重载不提供此功能。假设需要一个静态方法，修复 CollectionClassifier 程序的最佳方法是用一个执行显式 instanceof 测试的方法替换 classification 的所有三个重载：

```Java
public static String classify(Collection<?> c) {
    return c instanceof Set ? "Set" :c instanceof List ? "List" : "Unknown Collection";
}
```

Because overriding is the norm and overloading is the exception, overriding sets people’s expectations for the behavior of method invocation. As demonstrated by the CollectionClassifier example, overloading can easily confound these expectations. It is bad practice to write code whose behavior is likely to confuse programmers. This is especially true for APIs. If the typical user of an API does not know which of several method overloadings will get invoked for a given set of parameters, use of the API is likely to result in errors. These errors will likely manifest themselves as erratic behavior at runtime, and many programmers will have a hard time diagnosing them. Therefore you should **avoid confusing uses of overloading.**

因为覆盖是规范，而重载是例外，所以覆盖满足了人们对方法调用行为的期望。正如 CollectionClassifier 示例所示，重载很容易混淆这些期望。编写可能使程序员感到困惑的代码是不好的行为。对于 API 尤其如此。如果 API 的用户不知道一组参数应该调用哪一种方法重载，那么使用 API 时很可能会导致错误。这些错误很可能在运行时表现为不稳定的行为，许多程序员将很难诊断它们。因此，**应该避免混淆重载的用法。**

Exactly what constitutes a confusing use of overloading is open to some debate. **A safe, conservative policy is never to export two overloadings with the same number of parameters.** If a method uses varargs, a conservative policy is not to overload it at all, except as described in Item 53. If you adhere to these restrictions, programmers will never be in doubt as to which overloading applies to any set of actual parameters. These restrictions are not terribly onerous because **you can always give methods different names instead of overloading them.**

究竟是什么构成了混淆重载的用法还有待商榷。**安全、保守的策略是永远不导出具有相同数量参数的两个重载。** 如果一个方法使用了可变参数，保守策略是根本不重载它，除非如 [Item-53](/Chapter-8/Chapter-8-Item-53-Use-varargs-judiciously.md) 所述。如果遵守这些限制，程序员就不会怀疑一组参数应该调用哪一种方法重载。这些限制并不十分繁琐，因为 **你总是可以为方法提供不同的名称，而不是重载它们。**

For example, consider the ObjectOutputStream class. It has a variant of its write method for every primitive type and for several reference types. Rather than overloading the write method, these variants all have different names, such as writeBoolean(boolean), writeInt(int), and writeLong(long). An added benefit of this naming pattern, when compared to overloading, is that it is possible to provide read methods with corresponding names, for example, readBoolean(), readInt(), and readLong(). The ObjectInputStream class does, in fact, provide such read methods.

例如，考虑 ObjectOutputStream 类。对于每个基本类型和几种引用类型，其 write 方法都有变体。这些变体都有不同的名称，而不是重载 write 方法，例如 `writeBoolean(boolean)`、`writeInt(int)` 和 `writeLong(long)`。与重载相比，这种命名模式的另一个好处是，可以为 read 方法提供相应的名称，例如 `readBoolean()`、`readInt()` 和 `readLong()`。ObjectInputStream 类实际上也提供了这样的读方法。

For constructors, you don’t have the option of using different names: multiple constructors for a class are always overloaded. You do, in many cases, have the option of exporting static factories instead of constructors (Item 1). Also, with constructors you don’t have to worry about interactions between overloading and overriding, because constructors can’t be overridden. You will probably have occasion to export multiple constructors with the same number of parameters, so it pays to know how to do it safely.

对于构造函数，你没有使用不同名称的机会：一个类的多个构造函数只能重载。在很多情况下，你可以选择导出静态工厂而不是构造函数（[Item-1](/Chapter-2/Chapter-2-Item-1-Consider-static-factory-methods-instead-of-constructors.md)）。你可能会有机会导出具有相同数量参数的多个构造函数，因此知道如何安全地执行是有必要的。

Exporting multiple overloadings with the same number of parameters is unlikely to confuse programmers if it is always clear which overloading will apply to any given set of actual parameters. This is the case when at least one corresponding formal parameter in each pair of overloadings has a “radically different” type in the two overloadings. Two types are radically different if it is clearly impossible to cast any non-null expression to both types. Under these circumstances, which overloading applies to a given set of actual parameters is fully determined by the runtime types of the parameters and cannot be affected by their compile-time types, so a major source of confusion goes away. For example, ArrayList has one constructor that takes an int and a second constructor that takes a Collection. It is hard to imagine any confusion over which of these two constructors will be invoked under any circumstances.

如果总是清楚一组参数应该调用哪一种方法重载，那么用相同数量的参数导出多个重载不太可能让程序员感到困惑。在这种情况下，每对重载中至少有一个对应的形式参数在这两个重载中具有「完全不同的」类型。如果不可能将任何非空表达式强制转换为这两种类型，那么这两种类型是完全不同的。在这些情况下，应用于给定实际参数集的重载完全由参数的运行时类型决定，且不受其编译时类型的影响，因此消除了一个主要的混淆源。例如，ArrayList 有一个接受 int 的构造函数和第二个接受 Collection 的构造函数。很难想象在什么情况下会不知道这两个构造函数中哪个会被调用。

Prior to Java 5, all primitive types were radically different from all reference types, but this is not true in the presence of autoboxing, and it has caused real trouble. Consider the following program:

在 Java 5 之前，所有原始类型都与所有引用类型完全不同，但在自动装箱时并非如此，这造成了真正的麻烦。考虑以下方案：

```Java
public class SetList {
public static void main(String[] args) {
    Set<Integer> set = new TreeSet<>();
    List<Integer> list = new ArrayList<>();
    for (int i = -3; i < 3; i++) {
        set.add(i);
        list.add(i);
    }
    for (int i = 0; i < 3; i++) {
        set.remove(i);
        list.remove(i);
    }
    System.out.println(set +""+list);
    }
}
```

First, the program adds the integers from −3 to 2, inclusive, to a sorted set and a list. Then, it makes three identical calls to remove on the set and the list. If you’re like most people, you’d expect the program to remove the non-negative values (0, 1, and 2) from the set and the list and to print [-3, -2, -1] [-3, -2, -1]. In fact, the program removes the non-negative values from the set and the odd values from the list and prints [-3, -2, -1] [-2, 0, 2]. It is an understatement to call this behavior confusing.

首先，程序将从 -3 到 2 的整数（包括）添加到已排序的 Set 和 List 中。然后，它执行三个相同的调用来删除集合和列表。如果你和大多数人一样，你希望程序从集合和列表中删除非负值（0、1 和 2），并打印 `[-3,2,1][-3,2,1]`。实际上，程序从 Set 中删除非负值，从 List 中删除奇数值，并输出 `[-3,2,1][-2,0,2]`。把这种行为称为混乱，只是一种保守的说法。

Here’s what’s happening: The call to set.remove(i) selects the overloading remove(E), where E is the element type of the set (Integer), and autoboxes i from int to Integer. This is the behavior you’d expect, so the program ends up removing the positive values from the set. The call to list.remove(i), on the other hand, selects the overloading remove(int i), which removes the element at the specified position in the list. If you start with the list [-3, -2, -1, 0, 1, 2] and remove the zeroth element, then the first, and then the second, you’re left with [-2, 0, 2], and the mystery is solved. To fix the problem, cast list.remove’s argument to Integer, forcing the correct overloading to be selected. Alternatively, you could invoke Integer.valueOf on i and pass the result to list.remove. Either way, the program prints [-3, -2, -1] [-3, -2, -1], as expected:

实际情况如下：调用 `set.remove(i)` 选择重载 `remove(E)`，其中 E 是 set （Integer）的元素类型，而将从 int 自动装箱到 Integer 中。这是你期望的行为，因此程序最终会从 Set 中删除正值。另一方面，对 `list.remove(i)` 的调用选择重载 `remove(int i)`，它将删除 List 中指定位置的元素。如果从 List `[-3，-2，-1,0,1,2]` 开始，移除第 0 个元素，然后是第 1 个，然后是第 2 个，就只剩下 `[-2,0,2]`，谜底就解开了。若要修复此问题，要将 `list.remove` 的参数转换成 Integer，强制选择正确的重载。或者，你可以调用 `Integer.valuef()`，然后将结果传递给 `list.remove`。无论哪种方式，程序都会按预期打印 `[-3, -2, -1] [-3, -2, -1]`:

```Java
for (int i = 0; i < 3; i++) {
    set.remove(i);
    list.remove((Integer) i); // or remove(Integer.valueOf(i))
}
```

The confusing behavior demonstrated by the previous example came about because the List<E> interface has two overloadings of the remove method: remove(E) and remove(int). Prior to Java 5 when the List interface was “generified,” it had a remove(Object) method in place of remove(E), and the corresponding parameter types, Object and int, were radically different. But in the presence of generics and autoboxing, the two parameter types are no longer radically different. In other words, adding generics and autoboxing to the language damaged the List interface. Luckily, few if any other APIs in the Java libraries were similarly damaged, but this tale makes it clear that autoboxing and generics increased the importance of caution when overloading. The addition of lambdas and method references in Java 8 further increased the potential for confusion in overloading. For example, consider these two snippets:

前一个示例所演示的令人困惑的行为是由于 List<E> 接口对 remove 方法有两个重载：`remove(E)` 和 `remove(int)`。在 Java 5 之前，当 List 接口被「泛化」时，它有一个 `remove(Object)` 方法代替 `remove(E)`，而相应的参数类型 Object 和 int 则完全不同。但是，在泛型和自动装箱的存在下，这两种参数类型不再完全不同。换句话说，在语言中添加泛型和自动装箱破坏了 List 接口。幸运的是，Java 库中的其他 API 几乎没有受到类似的破坏，但是这个故事清楚地表明，自动装箱和泛型出现后，在重载时就应更加谨慎。Java 8 中添加的 lambda 表达式和方法引用进一步增加了重载中混淆的可能性。例如，考虑以下两个片段：

```Java
new Thread(System.out::println).start();
ExecutorService exec = Executors.newCachedThreadPool();
exec.submit(System.out::println);
```

While the Thread constructor invocation and the submit method invocation look similar, the former compiles while the latter does not. The arguments are identical (System.out::println), and both the constructor and the method have an overloading that takes a Runnable. What’s going on here? The surprising answer is that the submit method has an overloading that takes a `Callable<T>`, while the Thread constructor does not. You might think that this shouldn’t make any difference because all overloadings of println return void, so the method reference couldn’t possibly be a Callable. This makes perfect sense, but it’s not the way the overload resolution algorithm works. Perhaps equally surprising is that the submit method invocation would be legal if the println method weren’t also overloaded. It is the combination of the overloading of the referenced method (println) and the invoked method (submit) that prevents the overload resolution algorithm from behaving as you’d expect.

虽然 Thread 构造函数调用和 submit 方法调用看起来很相似，但是前者编译而后者不编译。参数是相同的 `System.out::println`，构造函数和方法都有一个重载，该重载接受 Runnable。这是怎么回事？令人惊讶的答案是，submit 方法有一个重载，它接受一个 `Callable<T>`，而线程构造函数没有。你可能认为这不会有什么区别，因为 println 的所有重载都会返回 void，所以方法引用不可能是 Callable。这很有道理，但重载解析算法不是这样工作的。也许同样令人惊讶的是，如果 println 方法没有被重载，那么 submit 方法调用将是合法的。正是被引用的方法 println 和被调用的方法 submit 的重载相结合，阻止了重载解析算法按照你所期望的那样运行。

Technically speaking, the problem is that System.out::println is an inexact method reference [JLS, 15.13.1] and that “certain argument expressions that contain implicitly typed lambda expressions or inexact method references are ignored by the applicability tests, because their meaning cannot be determined until a target type is selected [JLS, 15.12.2].” Don’t worry if you don’t understand this passage; it is aimed at compiler writers. The key point is that overloading methods or constructors with different functional interfaces in the same argument position causes confusion. Therefore, **do not overload methods to take different functional interfaces in the same argument position.** In the parlance of this item, different functional interfaces are not radically different. The Java compiler will warn you about this sort of problematic overload if you pass the command line switch - Xlint:overloads.

从技术上讲，问题出在 System.out::println 上，它是一个不准确的方法引用 [JLS, 15.13.1]，并且「某些包含隐式类型化 lambda 表达式或不准确的方法引用的参数表达式会被适用性测试忽略，因为它们的含义在选择目标类型之前无法确定 [JLS, 15.12.2]。」如果你不明白这段话，不要担心；它的目标是编译器编写器。关键是在相同的参数位置上重载具有不同功能接口的方法或构造函数会导致混淆。因此，**不要重载方法来在相同的参数位置上使用不同的函数式接口。** 用本项目的话说，不同的函数式接口并没有根本的不同。如果你通过命令行开关 `Xlint:overloads`, Java 编译器将对这种有问题的重载发出警告。

Array types and class types other than Object are radically different. Also, array types and interface types other than Serializable and Cloneable are radically different. Two distinct classes are said to be unrelated if neither class is a descendant of the other [JLS, 5.5]. For example, String and Throwable are unrelated. It is impossible for any object to be an instance of two unrelated classes, so unrelated classes are radically different, too.

数组类型和 Object 以外的类类型是完全不同的。此外，数组类型和 Serializable 和 Cloneable 之外的接口类型也完全不同。如果两个不同的类都不是另一个类的后代 [JLS, 5.5]，则称它们是不相关的。例如，String 和 Throwable 是不相关的。任何对象都不可能是两个不相关类的实例，所以不相关的类也是完全不同的。

There are other pairs of types that can’t be converted in either direction [JLS, 5.1.12], but once you go beyond the simple cases described above, it becomes very difficult for most programmers to discern which, if any, overloading applies to a set of actual parameters. The rules that determine which overloading is selected are extremely complex and grow more complex with every release. Few programmers understand all of their subtleties.

还有其他成对的类不能在任何方向相互转换 [JLS, 5.1.12]，但是一旦超出上面描述的简单情况，大多数程序员就很难辨别一组参数应该调用哪一种方法重载。决定选择哪个重载的规则非常复杂，并且随着每个版本的发布而变得越来越复杂。很少有程序员能理解它们所有的微妙之处。

There may be times when you feel the need to violate the guidelines in this item, especially when evolving existing classes. For example, consider String, which has had a contentEquals(StringBuffer) method since Java 4. In Java 5, CharSequence was added to provide a common interface for StringBuffer, StringBuilder, String, CharBuffer, and other similar types. At the same time that CharSequence was added, String was outfitted with an overloading of the contentEquals method that takes a CharSequence.

有时候，你可能觉得会被迫违反本条目中的指导原则，特别是在更新现有类时。例如，考虑 String，它从 Java 4 开始就有一个 `contentEquals(StringBuffer)` 方法。在 Java 5 中，添加了 CharSequence 来为 StringBuffer、StringBuilder、String、CharBuffer 和其他类似类型提供公共接口。在添加 CharSequence 的同时，String 还配备了一个重载的 contentEquals 方法，该方法接受 CharSequence。

While the resulting overloading clearly violates the guidelines in this item, it causes no harm because both overloaded methods do exactly the same thing when they are invoked on the same object reference. The programmer may not know which overloading will be invoked, but it is of no consequence so long as they behave identically. The standard way to ensure this behavior is to have the more specific overloading forward to the more general:

虽然这样的重载明显违反了此项中的指导原则，但它不会造成任何危害，因为当在同一个对象引用上调用这两个重载方法时，它们做的是完全相同的事情。程序员可能不知道将调用哪个重载，但只要它们的行为相同，就没有什么不良后果。确保这种行为的标准方法是将更具体的重载转发给更一般的重载：

```Java
// Ensuring that 2 methods have identical behavior by forwarding
public boolean contentEquals(StringBuffer sb) {
    return contentEquals((CharSequence) sb);
}
```

While the Java libraries largely adhere to the spirit of the advice in this item, there are a number of classes that violate it. For example, String exports two overloaded static factory methods, valueOf(char[]) and valueOf(Object), that do completely different things when passed the same object reference. There is no real justification for this, and it should be regarded as an anomaly with the potential for real confusion.

虽然 Java 库在很大程度上遵循了本条目中的主旨精神，但是有一些类违反了它。例如，String 导出两个重载的静态工厂方法 `valueOf(char[])` 和 `valueOf(Object)`，它们在传递相同的对象引用时执行完全不同的操作。这样做没有真正的理由，它应该被视为一种异常行为，有可能造成真正的混乱。

To summarize, just because you can overload methods doesn’t mean you should. It is generally best to refrain from overloading methods with multiple signatures that have the same number of parameters. In some cases, especially where constructors are involved, it may be impossible to follow this advice. In these cases, you should at least avoid situations where the same set of parameters can be passed to different overloadings by the addition of casts. If this cannot be avoided, for example, because you are retrofitting an existing class to implement a new interface, you should ensure that all overloadings behave identically when passed the same parameters. If you fail to do this, programmers will be hard pressed to make effective use of the overloaded method or constructor, and they won’t understand why it doesn’t work.

总而言之，方法可以重载，但并不意味着就应该这样做。通常，最好避免重载具有相同数量参数的多个签名的方法。在某些情况下，特别是涉及构造函数的情况下，可能难以遵循这个建议。在这些情况下，你至少应该避免同一组参数只需经过类型转换就可以被传递给不同的重载方法。如果这是无法避免的，例如，因为要对现有类进行改造以实现新接口，那么应该确保在传递相同的参数时，所有重载的行为都是相同的。如果你做不到这一点，程序员将很难有效地使用重载方法或构造函数，他们将无法理解为什么它不能工作。

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-8/Chapter-8-Introduction.md)**
- **Previous Item（上一条目）：[Item 51: Design method signatures carefully（仔细设计方法签名）](/Chapter-8/Chapter-8-Item-51-Design-method-signatures-carefully.md)**
- **Next Item（下一条目）：[Item 53: Use varargs judiciously（明智地使用可变参数）](/Chapter-8/Chapter-8-Item-53-Use-varargs-judiciously.md)**
