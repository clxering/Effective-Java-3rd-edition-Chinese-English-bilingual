## Chapter 8. Methods（方法）

### Item 51: Design method signatures carefully（仔细设计方法签名）

This item is a grab bag of API design hints that don’t quite deserve items of their own. Taken together, they’ll help make your API easier to learn and use and less prone to errors.

本条目是一个 API 设计提示的大杂烩，它们还不完全值得拥有独立的条目。总之，它们将帮助你使 API 更容易学习和使用，并且更不容易出错。

**Choose method names carefully.** Names should always obey the standard naming conventions (Item 68). Your primary goal should be to choose names that are understandable and consistent with other names in the same package. Your secondary goal should be to choose names consistent with the broader consensus, where it exists. Avoid long method names. When in doubt, look to the Java library APIs for guidance. While there are plenty of inconsistencies— inevitable, given the size and scope of these libraries—there is also a fair amount of consensus.

**仔细选择方法名称。** 名称应始终遵守标准的命名约定（[Item-68](/Chapter-9/Chapter-9-Item-68-Adhere-to-generally-accepted-naming-conventions.md)）。你的主要目标应该是选择可理解的、与同一包中的其他名称风格一致的名称。你的第二个目标应该是选择被广泛认可的名字。避免长方法名。如果有疑问，可以参考 Java 库 API，尽管其中也存在大量的矛盾（考虑到这些库的规模和范围，这是不可避免的）但也达成了相当多的共识。

**Don’t go overboard in providing convenience methods.** Every method should “pull its weight.” Too many methods make a class difficult to learn, use, document, test, and maintain. This is doubly true for interfaces, where too many methods complicate life for implementors as well as users. For each action supported by your class or interface, provide a fully functional method. Consider providing a “shorthand” only if it will be used often. **When in doubt, leave it out.**

**不要提供过于便利的方法。** 每种方法都应该各司其职。太多的方法使得类难以学习、使用、记录、测试和维护。对于接口来说更是如此，在接口中，太多的方法使实现者和用户的工作变得复杂。对于类或接口支持的每个操作，请提供一个功能齐全的方法。只有在经常使用时才考虑提供便捷方式。**但如果有疑问，就不要提供。**

**Avoid long parameter lists.** Aim for four parameters or fewer. Most programmers can’t remember longer parameter lists. If many of your methods exceed this limit, your API won’t be usable without constant reference to its documentation. Modern IDEs help, but you are still much better off with short parameter lists. **Long sequences of identically typed parameters are especially harmful**. Not only won’t users be able to remember the order of the parameters, but when they transpose parameters accidentally, their programs will still compile and run. They just won’t do what their authors intended.

**避免长参数列表。** 设定四个或更少的参数。大多数程序员记不住更长的参数列表。如果你的许多方法超过了这个限制，而用户没有对文档的不断查看，你的 API 将无法使用。现代 IDE 会有所帮助，但是使用简短的参数列表仍然会让情况好得多。**长序列的同类型参数尤其有害**。用户不仅不能记住参数的顺序，而且当他们不小心转置参数时，他们的程序仍然会编译和运行。它们只是不会按照作者的意图去做。

There are three techniques for shortening overly long parameter lists. One is to break the method up into multiple methods, each of which requires only a subset of the parameters. If done carelessly, this can lead to too many methods, but it can also help reduce the method count by increasing orthogonality. For example, consider the java.util.List interface. It does not provide methods to find the first or last index of an element in a sublist, both of which would require three parameters. Instead it provides the subList method, which takes two parameters and returns a view of a sublist. This method can be combined with the indexOf or lastIndexOf method, each of which has a single parameter, to yield the desired functionality. Moreover, the subList method can be combined with any method that operates on a List instance to perform arbitrary computations on sublists. The resulting API has a very high power-to-weight ratio.

有三种方法可以缩短过长的参数列表。一种方法是将方法分解为多个方法，每个方法只需要参数的一个子集。如果操作不当，这可能导致产生太多的方法，但它也可以通过增加正交性来帮助减少方法数量。例如，考虑 `java.util.List` 接口。它不提供查找子列表中元素的第一个或最后一个索引的方法，这两个方法都需要三个参数。相反，它提供了 subList 方法，该方法接受两个参数并返回子列表的视图。此方法可以与 indexOf 或 lastIndexOf 方法组合使用以达到所需的功能，其中每个方法都有一个参数。此外，subList 方法可以与操作 List 实例的任何方法组合使用，以执行子列表上的任意操作。这样的 API 就具有非常高的 power-to-weight 比。

A second technique for shortening long parameter lists is to create helper classes to hold groups of parameters. Typically these helper classes are static member classes (Item 24). This technique is recommended if a frequently occurring sequence of parameters is seen to represent some distinct entity. For example, suppose you are writing a class representing a card game, and you find yourself constantly passing a sequence of two parameters representing a card’s rank and its suit. Your API, as well as the internals of your class, would probably benefit if you added a helper class to represent a card and replaced every occurrence of the parameter sequence with a single parameter of the helper class.

缩短长参数列表的第二种技术是创建 helper 类来保存参数组。通常，这些 helper 类是静态成员类（[Item-42](/Chapter-7/Chapter-7-Item-42-Prefer-lambdas-to-anonymous-classes.md)）。如果经常出现的参数序列表示一些不同的实体，则推荐使用这种技术。例如，假设你正在编写一个表示纸牌游戏的类，你发现会不断地传递一个序列，其中包含两个参数，分别表示纸牌的等级和花色。如果你添加一个 helper 类来表示一张卡片，并用 helper 类的一个参数替换参数序列中的每个出现的参数，那么你的 API 以及类的内部结构都可能受益。

A third technique that combines aspects of the first two is to adapt the Builder pattern (Item 2) from object construction to method invocation. If you have a method with many parameters, especially if some of them are optional, it can be beneficial to define an object that represents all of the parameters and to allow the client to make multiple “setter” calls on this object, each of which sets a single parameter or a small, related group. Once the desired parameters have been set, the client invokes the object’s “execute” method, which does any final validity checks on the parameters and performs the actual computation.

结合前两个方面讨论的第三种技术是，从对象构建到方法调用都采用建造者模式（[Item-2](/Chapter-2/Chapter-2-Item-2-Consider-a-builder-when-faced-with-many-constructor-parameters.md)）。如果你有一个方法带有许多参数，特别是其中一些参数是可选的，最好定义一个对象来表示所有参数，并允许客户端多次调用「setter」来使用这个对象，每一次都设置一个参数或较小相关的组。一旦设置了所需的参数，客户机就调用对象的「execute」方法，该方法对参数进行最终有效性检查并执行实际操作。

**For parameter types, favor interfaces over classes** (Item 64). If there is an appropriate interface to define a parameter, use it in favor of a class that implements the interface. For example, there is no reason to ever write a method that takes HashMap on input—use Map instead. This lets you pass in a HashMap, a TreeMap, a ConcurrentHashMap, a submap of a TreeMap, or any Map implementation yet to be written. By using a class instead of an interface, you restrict your client to a particular implementation and force an unnecessary and potentially expensive copy operation if the input data happens to exist in some other form.

**对于参数类型，优先选择接口而不是类**（[Item-64](/Chapter-9/Chapter-9-Item-64-Refer-to-objects-by-their-interfaces.md)）。如果有合适的接口来定义参数，那么使用它来支持实现该接口的类。例如，没有理由编写一个输入使用 HashMap 的方法，而应该使用 Map。这允许你传入 HashMap、TreeMap、ConcurrentHashMap、TreeMap 的子映射或任何尚未编写的 Map 实现。通过使用类而不是接口，你可以将客户端限制在特定的实现中，如果输入数据碰巧以某种其他形式存在，则会强制执行不必要的、可能代价很高的复制操作。

**Prefer two-element enum types to boolean parameters,** unless the meaning of the boolean is clear from the method name. Enums make your code easier to read and to write. Also, they make it easy to add more options later. For example, you might have a Thermometer type with a static factory that takes this enum:

**双元素枚举类型优于 boolean 参数，** 除非布尔值的含义在方法名中明确。枚举使代码更容易读和写。此外，它们使以后添加更多选项变得更加容易。例如，你可能有一个 Thermometer 类型与静态工厂，采用枚举：

```Java
public enum TemperatureScale { FAHRENHEIT, CELSIUS }
```

Not only does Thermometer.newInstance(TemperatureScale.CELSIUS) make a lot more sense than Thermometer.newInstance(true), but you can add KELVIN to TemperatureScale in a future release without having to add a new static factory to Thermometer. Also, you can refactor temperaturescale dependencies into methods on the enum constants (Item 34). For example, each scale constant could have a method that took a double value and converted it to Celsius.

`Thermometer.newInstance(TemperatureScale.CELSIUS)` 不仅比 `Thermometer.newInstance(true)` 更有意义，而且你可以在将来的版本中向 TemperatureScale 添加 KELVIN，而不必向 Thermometer 添加新的静态工厂。此外，你还可以将 TemperatureScale 依赖项重构为 enum 常量（[Item-34](/Chapter-6/Chapter-6-Item-34-Use-enums-instead-of-int-constants.md)）上的方法。例如，每个刻度单位都有一个方法，该方法带有 double 值并将其转换为摄氏度。

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-8/Chapter-8-Introduction.md)**
- **Previous Item（上一条目）：[Item 50: Make defensive copies when needed（在需要时制作防御性副本）](/Chapter-8/Chapter-8-Item-50-Make-defensive-copies-when-needed.md)**
- **Next Item（下一条目）：[Item 52: Use overloading judiciously（明智地使用重载）](/Chapter-8/Chapter-8-Item-52-Use-overloading-judiciously.md)**
