## Chapter 9. General Programming（通用程序设计）

### Item 64: Refer to objects by their interfaces（通过接口引用对象）

Item 51 says that you should use interfaces rather than classes as parameter types. More generally, you should favor the use of interfaces over classes to refer to objects. **If appropriate interface types exist, then parameters, return values, variables, and fields should all be declared using interface types.** The only time you really need to refer to an object’s class is when you’re creating it with a constructor. To make this concrete, consider the case of LinkedHashSet, which is an implementation of the Set interface. Get in the habit of typing this:

[Item-51](/Chapter-8/Chapter-8-Item-51-Design-method-signatures-carefully.md) 指出，应该使用接口而不是类作为参数类型。更一般地说，你应该优先使用接口而不是类来引用对象。**如果存在合适的接口类型，那么应该使用接口类型声明参数、返回值、变量和字段。** 惟一真正需要引用对象的类的时候是使用构造函数创建它的时候。为了具体说明这一点，考虑 LinkedHashSet 的情况，它是 Set 接口的一个实现。声明时应养成这样的习惯：

```Java
// Good - uses interface as type
Set<Son> sonSet = new LinkedHashSet<>();
```

not this:

而不是这样：

```Java
// Bad - uses class as type!
LinkedHashSet<Son> sonSet = new LinkedHashSet<>();
```

**If you get into the habit of using interfaces as types, your program will be much more flexible.** If you decide that you want to switch implementations, all you have to do is change the class name in the constructor (or use a different static factory). For example, the first declaration could be changed to read:

**如果你养成了使用接口作为类型的习惯，那么你的程序将更加灵活。** 如果你决定要切换实现，只需在构造函数中更改类名（或使用不同的静态工厂）。例如，第一个声明可以改为：

```Java
Set<Son> sonSet = new HashSet<>();
```

and all of the surrounding code would continue to work. The surrounding code was unaware of the old implementation type, so it would be oblivious to the change.

所有的代码都会继续工作。周围的代码不知道旧的实现类型，所以它不会在意更改。

There is one caveat: if the original implementation offered some special functionality not required by the general contract of the interface and the code depended on that functionality, then it is critical that the new implementation provide the same functionality. For example, if the code surrounding the first declaration depended on LinkedHashSet’s ordering policy, then it would be incorrect to substitute HashSet for LinkedHashSet in the declaration, because HashSet makes no guarantee concerning iteration order.

有一点值得注意：如果原实现提供了接口的通用约定不需要的一些特殊功能，并且代码依赖于该功能，那么新实现提供相同的功能就非常重要。例如，如果围绕第一个声明的代码依赖于 LinkedHashSet 的排序策略，那么在声明中将 HashSet 替换为 LinkedHashSet 将是不正确的，因为 HashSet 不保证迭代顺序。

So why would you want to change an implementation type? Because the second implementation offers better performance than the original, or because it offers desirable functionality that the original implementation lacks. For example, suppose a field contains a HashMap instance. Changing it to an EnumMap will provide better performance and iteration order consistent with the natural order of the keys, but you can only use an EnumMap if the key type is an enum type. Changing the HashMap to a LinkedHashMap will provide predictable iteration order with performance comparable to that of HashMap, without making any special demands on the key type.

那么，为什么要更改实现类型呢？因为第二个实现比原来的实现提供了更好的性能，或者因为它提供了原来的实现所缺乏的理想功能。例如，假设一个字段包含一个 HashMap 实例。将其更改为 EnumMap 将为迭代提供更好的性能和与键的自然顺序，但是你只能在键类型为 enum 类型的情况下使用 EnumMap。将 HashMap 更改为 LinkedHashMap 将提供可预测的迭代顺序，性能与 HashMap 相当，而不需要对键类型作出任何特殊要求。

You might think it’s OK to declare a variable using its implementation type, because you can change the declaration type and the implementation type at the same time, but there is no guarantee that this change will result in a program that compiles. If the client code used methods on the original implementation type that are not also present on its replacement or if the client code passed the instance to a method that requires the original implementation type, then the code will no longer compile after making this change. Declaring the variable with the interface type keeps you honest.

你可能认为使用变量的实现类型声明变量是可以的，因为你可以同时更改声明类型和实现类型，但是不能保证这种更改会正确编译程序。如果客户端代码对原实现类型使用了替换时不存在的方法，或者客户端代码将实例传递给需要原实现类型的方法，那么在进行此更改之后，代码将不再编译。使用接口类型声明变量可以保持一致。

**It is entirely appropriate to refer to an object by a class rather than an interface if no appropriate interface exists.** For example, consider value classes, such as String and BigInteger. Value classes are rarely written with multiple implementations in mind. They are often final and rarely have corresponding interfaces. It is perfectly appropriate to use such a value class as a parameter, variable, field, or return type.

**如果没有合适的接口存在，那么用类引用对象是完全合适的。** 例如，考虑值类，如 String 和 BigInteger。值类很少在编写时考虑到多个实现。它们通常是 final 的，很少有相应的接口。使用这样的值类作为参数、变量、字段或返回类型非常合适。

A second case in which there is no appropriate interface type is that of objects belonging to a framework whose fundamental types are classes rather than interfaces. If an object belongs to such a class-based framework, it is preferable to refer to it by the relevant base class, which is often abstract, rather than by its implementation class. Many java.io classes such as OutputStream fall into this category.

没有合适接口类型的第二种情况是属于框架的对象，框架的基本类型是类而不是接口。如果一个对象属于这样一个基于类的框架，那么最好使用相关的基类来引用它，这通常是抽象的，而不是使用它的实现类。在 java.io 类中许多诸如 OutputStream 之类的就属于这种情况。

A final case in which there is no appropriate interface type is that of classes that implement an interface but also provide extra methods not found in the interface—for example, PriorityQueue has a comparator method that is not present on the Queue interface. Such a class should be used to refer to its instances only if the program relies on the extra methods, and this should be very rare.

没有合适接口类型的最后一种情况是，实现接口但同时提供接口中不存在的额外方法的类，例如，PriorityQueue 有一个在 Queue 接口上不存在的比较器方法。只有当程序依赖于额外的方法时，才应该使用这样的类来引用它的实例，这种情况应该非常少见。

These three cases are not meant to be exhaustive but merely to convey the flavor of situations where it is appropriate to refer to an object by its class. In practice, it should be apparent whether a given object has an appropriate interface. If it does, your program will be more flexible and stylish if you use the interface to refer to the object. **If there is no appropriate interface, just use the least specific class in the class hierarchy that provides the required functionality.**

这三种情况并不是面面俱到的，而仅仅是为了传达适合通过类引用对象的情况。在实际应用中，给定对象是否具有适当的接口应该是显而易见的。如果是这样，如果使用接口引用对象，程序将更加灵活和流行。**如果没有合适的接口，就使用类层次结构中提供所需功能的最底层的类**

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-9/Chapter-9-Introduction.md)**
- **Previous Item（上一条目）：[Item 63: Beware the performance of string concatenation（当心字符串连接引起的性能问题）](/Chapter-9/Chapter-9-Item-63-Beware-the-performance-of-string-concatenation.md)**
- **Next Item（下一条目）：[Item 65: Prefer interfaces to reflection（接口优于反射）](/Chapter-9/Chapter-9-Item-65-Prefer-interfaces-to-reflection.md)**
