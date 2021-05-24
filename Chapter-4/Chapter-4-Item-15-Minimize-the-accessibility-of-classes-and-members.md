## Chapter 4. Classes and Interfaces（类和接口）

### Item 15: Minimize the accessibility of classes and members（尽量减少类和成员的可访问性）

The single most important factor that distinguishes a well-designed component from a poorly designed one is the degree to which the component hides its internal data and other implementation details from other components. A welldesigned component hides all its implementation details, cleanly separating its API from its implementation. Components then communicate only through their APIs and are oblivious to each others’ inner workings. This concept, known as information hiding or encapsulation, is a fundamental tenet of software design [Parnas72].

要将设计良好的组件与设计糟糕的组件区别开来，最重要的因素是：隐藏内部数据和其他实现细节的程度。设计良好的组件隐藏了所有实现细节，将 API 与实现完全分离。组件之间只通过它们的 API 进行通信，而不知道彼此的内部工作方式。这个概念被称为信息隐藏或封装，是软件设计的基本原则 [Parnas72]。

Information hiding is important for many reasons, most of which stem from the fact that it decouples the components that comprise a system, allowing them to be developed, tested, optimized, used, understood, and modified in isolation.This speeds up system development because components can be developed in parallel. It eases the burden of maintenance because components can be understood more quickly and debugged or replaced with little fear of harming other components. While information hiding does not, in and of itself, cause good performance, it enables effective performance tuning: once a system is complete and profiling has determined which components are causing performance problems (Item 67), those components can be optimized without affecting the correctness of others. Information hiding increases software reuse because components that aren’t tightly coupled often prove useful in other contexts besides the ones for which they were developed. Finally, information hiding decreases the risk in building large systems because individual components may prove successful even if the system does not.

由于许多原因，信息隐藏是重要的，其中大部分原因源于这样一个事实：它解耦了组成系统的组件，允许它们被独立开发、测试、优化、使用、理解和修改。这加快了系统开发进度，因为组件可以并行开发。也减轻了维护的负担，因为组件可以被更快地理解、调试或替换，而不必担心会损害其他组件。虽然信息隐藏本身不会获得良好的性能，但它可以实现有效的性能调优：一旦系统完成，概要分析确定了哪些组件会导致性能问题（[Item-67](/Chapter-9/Chapter-9-Item-67-Optimize-judiciously.md)），就可以在不影响其他组件正确性的情况下对这些组件进行优化。信息隐藏增加了软件的复用性，因为没有紧密耦合的组件在其他场景中通常被证明是有用的，除了开发它们时所在的场景之外。最后，信息隐藏降低了构建大型系统的风险，因为即使系统没有成功，单个组件也可能被证明是成功的。

Java has many facilities to aid in information hiding. The access control mechanism [JLS, 6.6] specifies the accessibility of classes, interfaces, and members. The accessibility of an entity is determined by the location of its declaration and by which, if any, of the access modifiers (private,protected, and public) is present on the declaration. Proper use of these modifiers is essential to information hiding.

Java 有许多工具来帮助隐藏信息。访问控制机制 [JLS, 6.6] 指定了类、接口和成员的可访问性。实体的可访问性由其声明的位置以及声明中出现的访问修饰符（private、protected 和 public）决定。正确使用这些修饰符是信息隐藏的关键。

The rule of thumb is simple: make each class or member as inaccessible as possible. In other words, use the lowest possible access level consistent with the proper functioning of the software that you are writing.

经验法则很简单：让每个类或成员尽可能不可访问。换句话说，在不影响软件正常功能时，使用尽可能低的访问级别。

For top-level (non-nested) classes and interfaces, there are only two possible access levels: package-private and public. If you declare a top-level class or interface with the public modifier, it will be public; otherwise, it will be package-private. If a top-level class or interface can be made package-private, it should be. By making it package-private, you make it part of the implementation rather than the exported API, and you can modify it, replace it, or eliminate it in a subsequent release without fear of harming existing clients. If you make it public, you are obligated to support it forever to maintain compatibility.

对于顶级（非嵌套）类和接口，只有两个可能的访问级别：包私有和公共。如果用 public 修饰符声明一个顶级类或接口，它将是公共的；否则，它将是包私有的。如果顶级类或接口可以设置为包私有，那么就应该这么做。通过将其设置为包私有，可以使其成为实现的一部分，而不是导出的 API 的一部分，并且可以在后续版本中修改、替换或删除它，而不必担心损害现有的客户端。如果将其公开，就有义务永远提供支持，以保持兼容性。

If a package-private top-level class or interface is used by only one class,consider making the top-level class a private static nested class of the sole class that uses it (Item 24). This reduces its accessibility from all the classes in its package to the one class that uses it. But it is far more important to reduce the accessibility of a gratuitously public class than of a package-private top-level class: the public class is part of the package’s API, while the package-private top-level class is already part of its implementation.

如果包级私有顶级类或接口只被一个类使用，那么可以考虑：在使用它的这个类中，将顶级类设置为私有静态嵌套类（[Item-24](/Chapter-4/Chapter-4-Item-24-Favor-static-member-classes-over-nonstatic.md)）。对于包中的所有类以及使用它的类来说，这降低了它的可访问性。但是，降低公共类的可访问性比减少包级私有顶级类的可访问性重要得多：公共类是包 API 的一部分，而包级私有顶级类已经是实现的一部分。

For members (fields, methods, nested classes, and nested interfaces), there are four possible access levels, listed here in order of increasing accessibility:

对于成员（字段、方法、嵌套类和嵌套接口），有四个可能的访问级别，这里按可访问性依次递增的顺序列出：

- **private** —The member is accessible only from the top-level class where it is declared.

**私有**，成员只能从声明它的顶级类内部访问。

- **package-private** —The member is accessible from any class in the package where it is declared. Technically known as default access, this is the access level you get if no access modifier is specified (except for interface members,which are public by default).

**包级私有**，成员可以从包中声明它的任何类访问。技术上称为默认访问，即如果没有指定访问修饰符（接口成员除外，默认情况下，接口成员是公共的），就会得到这个访问级别。

- **protected** —The member is accessible from subclasses of the class where it is declared (subject to a few restrictions [JLS, 6.6.2]) and from any class in the package where it is declared.

**保护**，成员可以通过声明它的类的子类（会受一些限制 [JLS, 6.6.2]）和声明它的包中的任何类访问。

- **public** —The member is accessible from anywhere.

**公共**，该成员可以从任何地方访问。

After carefully designing your class’s public API, your reflex should be to make all other members private. Only if another class in the same package really needs to access a member should you remove the private modifier, making the member package-private. If you find yourself doing this often, you should reexamine the design of your system to see if another decomposition might yield classes that are better decoupled from one another. That said, both private and package-private members are part of a class’s implementation and do not normally impact its exported API. These fields can, however, “leak” into the exported API if the class implements Serializable (Items 86 and 87).

在仔细设计了类的公共 API 之后，你应该本能的使所有成员都是私有的。只有当同一包中的另一个类确实需要访问一个成员时，你才应该删除 private 修饰符，使成员变为包级私有。如果你发现自己经常这样做，那么你应该重新确认系统的设计，看看是否有其他方式能产生更好地相互解耦的类。也就是说，私有成员和包级私有成员都是类实现的一部分，通常不会影响其导出的 API。但是，如果类实现了 Serializable（[Item-86](/Chapter-12/Chapter-12-Item-86-Implement-Serializable-with-great-caution.md) 和 [Item-87](/Chapter-12/Chapter-12-Item-87-Consider-using-a-custom-serialized-form.md)），这些字段可能会「泄漏」到导出的 API 中。

For members of public classes, a huge increase in accessibility occurs when the access level goes from package-private to protected. A protected member is part of the class’s exported API and must be supported forever. Also, a protected member of an exported class represents a public commitment to an implementation detail (Item 19). The need for protected members should be relatively rare.

对于公共类的成员来说，当访问级别从包级私有变为保护时，可访问性会有很大的提高。保护成员是类导出 API 的一部分，必须永远支持。此外，导出类的保护成员表示对实现细节的公开承诺（[Item-19](/Chapter-4/Chapter-4-Item-19-Design-and-document-for-inheritance-or-else-prohibit-it.md)）。需要保护成员的场景应该相对少见。

There is a key rule that restricts your ability to reduce the accessibility of methods. If a method overrides a superclass method, it cannot have a more restrictive access level in the subclass than in the superclass [JLS, 8.4.8.3]. This is necessary to ensure that an instance of the subclass is usable anywhere that an instance of the superclass is usable (the Liskov substitution principle, see Item15). If you violate this rule, the compiler will generate an error message when you try to compile the subclass. A special case of this rule is that if a class implements an interface, all of the class methods that are in the interface must be declared public in the class.

有一个关键规则限制了你降低方法的可访问性。如果一个方法覆盖了超类方法，那么它在子类中的访问级别就不能比超类 [JLS, 8.4.8.3] 更严格。这对于确保子类的实例在超类的实例可用的任何地方都同样可用是必要的（Liskov 替换原则，请参阅 [Item-15](/Chapter-4/Chapter-4-Item-15-Minimize-the-accessibility-of-classes-and-members.md)）。如果违反此规则，编译器将在尝试编译子类时生成错误消息。这个规则的一个特例是，如果一个类实现了一个接口，那么该接口中的所有类方法都必须在类中声明为 public。

To facilitate testing your code, you may be tempted to make a class, interface,or member more accessible than otherwise necessary. This is fine up to a point. It is acceptable to make a private member of a public class package-private in order to test it, but it is not acceptable to raise the accessibility any higher. In other words, it is not acceptable to make a class, interface, or member a part of a package’s exported API to facilitate testing. Luckily, it isn’t necessary either because tests can be made to run as part of the package being tested, thus gaining access to its package-private elements.

为了便于测试代码，你可能会试图使类、接口或成员更容易访问。这在一定程度上是好的。为了测试，将公共类成员由私有变为包私有是可以接受的，但是进一步提高可访问性是不可接受的。换句话说，将类、接口或成员作为包导出 API 的一部分以方便测试是不可接受的。幸运的是，也没有必要这样做，因为测试可以作为包的一部分运行，从而获得对包私有元素的访问权。

**Instance fields of public classes should rarely be public** (Item 16). If an instance field is nonfinal or is a reference to a mutable object, then by making it public, you give up the ability to limit the values that can be stored in the field. This means you give up the ability to enforce invariants involving the field. Also, you give up the ability to take any action when the field is modified, so **classes with public mutable fields are not generally thread-safe.** Even if a field is final and refers to an immutable object, by making it public you give up the flexibility to switch to a new internal data representation in which the field does not exist.

**公共类的实例字段很少采用 public 修饰**（[Item-16](/Chapter-4/Chapter-4-Item-16-In-public-classes-use-accessor-methods-not-public-fields.md)）。如果实例字段不是 final 的，或者是对可变对象的引用，那么将其公开，你就放弃了限制字段中可以存储的值的能力。这意味着你放弃了强制包含字段的不可变的能力。此外，你还放弃了在修改字段时采取任何操作的能力，因此 **带有公共可变字段的类通常不是线程安全的。** 即使一个字段是 final 的，并且引用了一个不可变的对象，通过将其公开，你放弃了切换到一个新的内部数据表示的灵活性，而该字段并不存在。

The same advice applies to static fields, with one exception. You can expose constants via public static final fields, assuming the constants form an integral part of the abstraction provided by the class. By convention, such fields have names consisting of capital letters, with words separated by underscores (Item 68). It is critical that these fields contain either primitive values or references to immutable objects (Item 17). a field containing a reference to a mutable object has all the disadvantages of a nonfinal field. While the reference cannot be modified, the referenced object can be modified—with disastrous results.

同样的建议也适用于静态字段，只有一个例外。你可以通过公共静态 final 字段公开常量，假设这些常量是类提供的抽象的组成部分。按照惯例，这些字段的名称由大写字母组成，单词以下划线分隔（[Item-68](/Chapter-9/Chapter-9-Item-68-Adhere-to-generally-accepted-naming-conventions.md)）。重要的是，这些字段要么包含基本数据类型，要么包含对不可变对象的引用（[Item-17](/Chapter-4/Chapter-4-Item-17-Minimize-mutability.md)）。包含对可变对象的引用的字段具有非 final 字段的所有缺点。虽然引用不能被修改，但是引用的对象可以被修改会导致灾难性的后果。

Note that a nonzero-length array is always mutable, so it is wrong for a class to have a public static final array field, or an accessor that returns such a field. If a class has such a field or accessor, clients will be able to modify the contents of the array. This is a frequent source of security holes:

请注意，非零长度的数组总是可变的，因此对于类来说，拥有一个公共静态 final 数组字段或返回该字段的访问器是错误的。如果一个类具有这样的字段或访问器，客户端将能够修改数组的内容。这是一个常见的安全漏洞来源：

```Java
// Potential security hole!
public static final Thing[] VALUES = { ... };
```

Beware of the fact that some IDEs generate accessors that return references to private array fields, resulting in exactly this problem. There are two ways to fix the problem. You can make the public array private and add a public immutable list:

要注意的是，一些 IDE 生成了返回私有数组字段引用的访问器，这恰恰会导致这个问题。有两种方法可以解决这个问题。你可以将公共数组设置为私有，并添加一个公共不可变 List：

```Java
private static final Thing[] PRIVATE_VALUES = { ... };
public static final List<Thing> VALUES = Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
```

Alternatively, you can make the array private and add a public method that returns a copy of a private array:

或者，你可以将数组设置为私有，并添加一个返回私有数组副本的公共方法：

```Java
private static final Thing[] PRIVATE_VALUES = { ... };
public static final Thing[] values() {
    return PRIVATE_VALUES.clone();
}
```

To choose between these alternatives, think about what the client is likely to do with the result. Which return type will be more convenient? Which will give better performance?

如何在这些备选方案中进行选择，请考虑客户可能会如何处理结果。哪种返回类型更方便？哪种表现会更好？

As of Java 9, there are two additional, implicit access levels introduced as part of the module system. A module is a grouping of packages, like a package is a grouping of classes. A module may explicitly export some of its packages via export declarations in its module declaration (which is by convention contained in a source file named module-info.java). Public and protected members of unexported packages in a module are inaccessible outside the module; within the module, accessibility is unaffected by export declarations. Using the module system allows you to share classes among packages within a module without making them visible to the entire world. Public and protected members of public classes in unexported packages give rise to the two implicit access levels, which are intramodular analogues of the normal public and protected levels. The need for this kind of sharing is relatively rare and can often be eliminated by rearranging the classes within your packages.

对于 Java 9，作为模块系统的一部分，还引入了另外两个隐式访问级别。模块是包的分组单位，就像包是类的分组单位一样。模块可以通过模块声明中的导出声明显式地导出它的一些包（按照约定包含在名为 `module-info.java` 的源文件中）。模块中未导出包的公共成员和保护成员在模块外不可访问；在模块中，可访问性不受导出声明的影响。通过使用模块系统，你可以在模块内的包之间共享类，而不会让整个世界看到它们。未导出包中的公共类和保护成员产生了两个隐式访问级别，它们是正常公共级别和保护级别的类似物。这种共享的需求相对较少，通常可以通过重新安排包中的类来解决。

Unlike the four main access levels, the two module-based levels are largely advisory. If you place a module’s JAR file on your application’s class path instead of its module path, the packages in the module revert to their nonmodular behavior: all of the public and protected members of the packages’ public classes have their normal accessibility, regardless of whether the packages are exported by the module [Reinhold, 1.2]. The one place where the newly introduced access levels are strictly enforced is the JDK itself: the unexported packages in the Java libraries are truly inaccessible outside of their modules.

与四个主要的访问级别不同，这两个基于模块的级别在很大程度上是建议级别。如果将模块的 JAR 文件放在应用程序的类路径上，而不是模块路径上，模块中的包将恢复它们的非模块行为：包的公共类的所有公共成员和保护成员都具有正常的可访问性，而不管模块是否导出包 [Reinhold,1.2]。严格执行新引入的访问级别的一个地方是 JDK 本身：Java 库中未导出的包在其模块之外确实不可访问。

Not only is the access protection afforded by modules of limited utility to the typical Java programmer, and largely advisory in nature; in order to take advantage of it, you must group your packages into modules, make all of their dependencies explicit in module declarations, rearrange your source tree, and take special actions to accommodate any access to non-modularized packages from within your modules [Reinhold, 3]. It is too early to say whether modules will achieve widespread use outside of the JDK itself. In the meantime, it seems best to avoid them unless you have a compelling need.

对于典型的 Java 程序员来说，访问保护不仅是有限实用的模块所提供的，而且本质上是建议性的；为了利用它，你必须将包以模块分组，在模块声明中显式地声明它们的所有依赖项，重新安排源代码树，并采取特殊操作以适应从模块中对非模块化包的任何访问 [Reinhold, 3]。现在说模块能否在 JDK 之外得到广泛使用还为时过早。与此同时，除非你有迫切的需求，否则最好还是不使用它们。

To summarize, you should reduce accessibility of program elements as much as possible (within reason). After carefully designing a minimal public API, you should prevent any stray classes, interfaces, or members from becoming part of the API. With the exception of public static final fields, which serve as constants,public classes should have no public fields. Ensure that objects referenced by public static final fields are immutable.

总之，你应该尽可能减少程序元素的可访问性（在合理的范围内）。在仔细设计了一个最小的公共 API 之后，你应该防止任何游离的类、接口或成员成为 API 的一部分。除了作为常量的公共静态 final 字段外，public 类应该没有公共字段。确保公共静态 final 字段引用的对象是不可变的。

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-4/Chapter-4-Introduction.md)**
- **Next Item（下一条目）：[Item 16: In public classes use accessor methods not public fields（在公共类中，使用访问器方法，而不是公共字段）](/Chapter-4/Chapter-4-Item-16-In-public-classes-use-accessor-methods-not-public-fields.md)**
