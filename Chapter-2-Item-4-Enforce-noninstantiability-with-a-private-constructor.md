## Chapter 2. Creating and Destroying Objects（创建和销毁对象）

### Item 4: Enforce noninstantiability with a private constructor（用私有构造函数执行非实例化）

Occasionally（adv.偶尔） you’ll want to write a class that is just a grouping of static methods and static fields. Such classes have acquired（v.取得） a bad reputation（n.名声） because some people abuse them to avoid thinking in terms of objects, but they do have valid uses. They can be used to group related methods on primitive values or arrays, in the manner of java.lang.Math or java.util.Arrays. They can also be used to group static methods, including factories (Item 1), for objects that implement some interface, in the manner of java.util.Collections. (As of Java 8, you can also put such methods in the interface, assuming it’s yours to modify.) Lastly, such classes can be used to group methods on a final class, since you can’t put them in a subclass.

有时你会想要写一个类，它只是一个静态方法和静态字段的组合。这样的类已经获得了坏名声，因为有些人滥用它们来避免用对象来思考，但是它们确实有有效的用途。他们可以用java.lang.Math或java.util.Arrays的方式，对原始的值或数组进行分组。它们还可以用于对以java.util.Collections的方式实现某些接口的对象分组静态方法，包括工厂（[Item-1](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-2-Item-1-Consider-static-factory-methods-instead-of-constructors.md)）。（对于Java 8，您也可以将这些方法放入接口中，假设您可以进行修改。）最后，这些类可以用来在终态类上进行分组，因为你不能把它们放在子类里。

Such utility classes were not designed to be instantiated: an instance would be nonsensical. In the absence of explicit constructors, however, the compiler provides a public, parameterless default constructor. To a user, this constructor is indistinguishable from any other. It is not uncommon to see unintentionally instantiable classes in published APIs.

这样的实用程序类不是为实例化而设计的:实例是无意义的。然而，在没有显式构造函数的情况下，编译器提供了一个公共的、无参数的默认构造函数。对于用户来说，这个构造函数与其他构造函数没有区别。在已发布的api中看到无意中实例化的类是很常见的。

**Attempting to enforce noninstantiability by making a class abstract does not work.** The class can be subclassed and the subclass instantiated.Furthermore, it misleads the user into thinking the class was designed for inheritance (Item 19). There is, however, a simple idiom to ensure noninstantiability. A default constructor is generated only if a class contains no explicit constructors, so **a class can be made noninstantiable by including a private constructor:** 

**试图通过使类抽象来增强非实例化性是行不通的。** 可以对类进行子类化，并实例化子类。此外，它误导用户认为类是为继承而设计的（[Item-19](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-4-Item-19-Design-and-document-for-inheritance-or-else-prohibit-it.md)）。然而，有一个简单的习惯用法来确保非实例化。只有当类不包含显式构造函数时，才会生成默认构造函数，因此**可以通过包含私有构造函数使类不可实例化：** 

```
// Noninstantiable utility class
public class UtilityClass {
// Suppress default constructor for noninstantiability
private UtilityClass() {
throw new AssertionError();
} ... // Remainder omitted
}
```

Because the explicit constructor is private, it is inaccessible outside the class.The AssertionError isn’t strictly required, but it provides insurance in case the constructor is accidentally invoked from within the class. It guarantees the class will never be instantiated under any circumstances. This idiom is mildly counterintuitive because the constructor is provided expressly so that it cannot be invoked. It is therefore wise to include a comment, as shown earlier.

因为显式构造函数是私有的，所以在类之外是不可访问的。AssertionError不是严格要求的，但是它提供了保险，以防构造函数意外地从类中调用。它保证类在任何情况下都不会被实例化。这个习惯用法有点违反常规，因为构造函数是明确提供的，但不能调用它。因此，如前面所示，包含注释是明智的。

As a side effect, this idiom also prevents the class from being subclassed. All constructors must invoke a superclass constructor, explicitly or implicitly, and a subclass would have no accessible superclass constructor to invoke.

作为一个副作用，这个习惯用法也防止了类被子类化。所有构造函数都必须调用超类构造函数，无论是显式的还是隐式的，子类都没有可访问的超类构造函数可调用。
