## Chapter 3. Methods Common to All Objects（对象的通用方法）

### Item 14: Consider implementing Comparable（考虑实现Comparable）

Unlike the other methods discussed in this chapter, the compareTo method is not declared in Object. Rather, it is the sole method in the Comparable interface. It is similar in character to Object’s equals method, except that it permits order comparisons in addition to simple equality comparisons, and it is generic. By implementing Comparable, a class indicates that its instances have a natural ordering. Sorting an array of objects that implement Comparable is as simple as this:

与本章中讨论的其他方法不同，compareTo 方法未在 Object 中声明。相反，它是 Comparable 接口中的唯一方法。它的特征与 Object 的 equals 方法类似，只不过它允许除了简单的相等比较之外的顺序比较，并且它是通用的。通过实现 Comparable ，意味着类的实例具有自然顺序。对实现 Comparable 的对象数组进行排序就像这样简单：

```java
Arrays.sort(a);
```

It is similarly easy to search, compute extreme values, and maintain automatically sorted collections of Comparable objects. For example, the following program, which relies on the fact that String implements Comparable, prints an alphabetized list of its command-line arguments with duplicates eliminated:
它同样易于搜索，计算极值，并维护自动排序的 Comparable 对象集合。例如，由于 String 实现了 Comparable 接口，实现打印其命令行参数的按字母顺序排列的列表，并删除重复项的功能只需这样做：

```java
public class WordList {
public static void main(String[] args) {
Set<String> s = new TreeSet<>();
Collections.addAll(s, args);
System.out.println(s);
}}
```

By implementing Comparable, you allow your class to interoperate with all of the many generic algorithms and collection implementations that depend on this interface. You gain a tremendous amount of power for a small amount of effort. Virtually all of the value classes in the Java platform libraries, as well as all enum types (Item 34), implement Comparable. If you are writing a value class with an obvious natural ordering, such as alphabetical order, numerical order, or chronological order, you should implement the Comparable interface:

通过实现 Comparable ，你可以让你写的类与依赖于此接口的所有许多通用算法和集合实现进行相互操作，事半功倍。 实际上，Java平台库中的所有值类以及所有枚举类型  (Item 34) 都实现了 Comparable 。 如果您正在编写具有明显自然顺序的值类，例如字母顺序，数字顺序或时间顺序，则应 实现Comparable 接口：

```java
public interface Comparable<T> {
int compareTo(T t);
}
```

The general contract of the compareTo method is similar to that of equals:

compareTo 方法的一般约定类似于 equals 方法：

Compares this object with the specified object for order. Returns a negative integer, zero, or a positive integer as this object is less than, equal to, or greater than the specified object. Throws ClassCastException if the specified object’s type prevents it from being compared to this object.

将此对象与指定的对象进行比较以获得顺序。 返回负整数，零或正整数表明此对象小于，等于或大于指定对象。 如果指定对象的类型阻止将其与此对象进行比较，则抛出 ClassCastException。

In the following description, the notation sgn(expression) designates the mathematical signum function, which is defined to return -1, 0, or 1,according to whether the value of expression is negative, zero, or positive.

在以下描述中，符号sgn（表达式）指定数学符号函数，其被定义为根据表达式的值是负，零还是正而返回-1,0或1。

- The implementor must ensure that sgn(x.compareTo(y)) == -sgn(y. compareTo(x)) for all x and y. (This implies that x.compareTo(y) must throw an exception if and only if y.compareTo(x) throws an exception.)
- 实现者必须确保所有x和y的sgn（x.compareTo（y））== -sgn（y.compareTo（x））。 （这意味着当且仅当y.compareTo（x）抛出异常时，x.compareTo（y）必须抛出异常。）
- The implementor must also ensure that the relation is transitive: (x.compareTo(y) > 0 && y.compareTo(z) > 0) implies x.compareTo(z) > 0.
- 实现者还必须确保关系是传递的：（x.compareTo（y）> 0 && y.compareTo（z）> 0）意味着x.compareTo（z）> 0。
- Finally, the implementor must ensure that x.compareTo(y) == 0 implies that sgn(x.compareTo(z)) == sgn(y.compareTo(z)),for all z.
- 最后，对于所有z，实现者必须确保x.compareTo（y）== 0暗示sgn（x.compareTo（z））== sgn（y.compareTo（z））。
- It is strongly recommended, but not required, that (x.compareTo(y)== 0) == (x.equals(y)). Generally speaking, any class that implements the Comparable interface and violates this condition should clearly indicate this fact. The recommended language is “Note: This class has a natural ordering that is inconsistent with equals.”
- 强烈建议（但不要求）（x.compareTo（y）== 0）==（x.equals（y））。 一般来说，任何实现Comparable接口并且违反此条件的类都应该清楚地表明这一事实。 推荐的说法是“请注意：此类具有与 equals 不一致的自然顺序。”

Don’t be put off by the mathematical nature of this contract. Like the equals contract (Item 10), this contract isn’t as complicated as it looks. Unlike the equals method, which imposes a global equivalence relation on all objects,compareTo doesn’t have to work across objects of different types: when confronted with objects of different types, compareTo is permitted to throw ClassCastException. Usually, that is exactly what it does. The contract does permit intertype comparisons, which are typically defined in an interface implemented by the objects being compared.

不要被这份约定的数学性质所拖延。与 equals 的约定 (Item 10) 一样，这个约定并不像看起来那么复杂。 与equals 方法不同，equals 方法对所有对象强加全局等价关系，compareTo 不必跨越不同类型的对象：遇到不同类型的对象时，compareTo 允许抛出ClassCastException。 通常，这正是它的作用。 约定确实允许交互式比较，这通常在由被比较的对象实现的接口中定义。

Just as a class that violates the hashCode contract can break other classes that depend on hashing, a class that violates the compareTo contract can break other classes that depend on comparison. Classes that depend on comparison include the sorted collections TreeSet and TreeMap and the utility classes Collections and Arrays, which contain searching and sorting algorithms.

正如违反 hashCode 约定的类会破坏依赖于哈希的其他类一样，违反 compareTo 约定的类可能会破坏依赖于比较的其他类。 依赖于比较的类包括已排序的集合 TreeSet 和 TreeMap 以及包含搜索和排序算法的Collections和Arrays。

Let’s go over the provisions of the compareTo contract. The first provision says that if you reverse the direction of a comparison between two object references, the expected thing happens: if the first object is less than the second,then the second must be greater than the first; if the first object is equal to the second, then the second must be equal to the first; and if the first object is greater than the second, then the second must be less than the first. The second provision says that if one object is greater than a second and the second is greater than a third, then the first must be greater than the third. The final provision says that all objects that compare as equal must yield the same results when compared to any other object.

我们来看看 compareTo 的约定。 第一条说如果你反转两个对象引用之间的比较方向，就会发生预期的事情：如果第一个对象小于第二个对象，那么第二个对象必须大于第一个对象; 如果第一个对象等于第二个对象，则第二个对象必须等于第一个对象; 如果第一个对象大于第二个对象，那么第二个对象必须小于第一个对象。 第二条规定，如果一个对象大于第二个对象而第二个对象大于第三个，那么第一个对象必须大于第三个物体。 最后的一条说，与任何其他对象相比，所有比较相等的对象必须产生相同的结果。

One consequence of these three provisions is that the equality test imposed by a compareTo method must obey the same restrictions imposed by the equals con-tract: reflexivity, symmetry, and transitivity. Therefore, the same caveat applies: there is no way to extend an instantiable class with a new value component while preserving the compareTo contract, unless you are willing to forgo the benefits of object-oriented abstraction (Item 10). The same workaround applies, too. If you want to add a value component to a class that implements Comparable, don’t extend it; write an unrelated class containing an instance of the first class. Then provide a “view” method that returns the contained instance. This frees you to implement whatever compareTo method you like on the containing class, while allowing its client to view an instance of the containing class as an instance of the contained class when needed.

这三个规定的一个结果是，compareTo 方法所施加的相等性测试必须遵循与 equals 相同的限制：反身性，对称性和传递性。 因此，同样的警告适用：除非您愿意放弃面向对象抽象的好处  (Item 10)，否则无法在保留compareTo 约定的情况下使用新值组件继承可实例化类。 同样的解决方法也适用。 如果要将值组件添加到实现Comparable 的类中，请不要继承它; 而是编写一个包含第一个类实例的无关类。 然后提供一个返回包含实例的“view”方法。 这使您可以在包含的类上实现您喜欢的任何compareTo方法，同时允许其客户端在需要时查看包含类的实例作为包含类的实例。

The final paragraph of the compareTo contract, which is a strong suggestion rather than a true requirement, simply states that the equality test imposed by the compareTo method should generally return the same results as the equals method. If this provision is obeyed, the ordering imposed by the compareTo method is said to be consistent with equals. If it’s violated, the ordering is said to be inconsistent with equals. A class whose compareTo method imposes an order that is inconsistent with equals will still work, but sorted collections containing elements of the class may not obey the general contract of the appropriate collection interfaces (Collection, Set, or Map). This is because the general contracts for these interfaces are defined in terms of the equals method, but sorted collections use the equality test imposed by compareTo in place of equals. It is not a catastrophe if this happens, but it’s something to be aware of.

compareTo 约定的最后一段是一个强烈的建议，而不是一个真正的要求，只是说明 compareTo 方法所施加的相等测试通常应该返回与 equals 方法相同的结果。 如果遵守此约定，则 compareTo 方法强加的顺序与equals一致。 如果它被违反，则说明顺序与equals不一致。 compareTo 方法强加与 equals 不一致的顺序的类仍然有效，但包含该类元素的有序集合可能不遵守相应集合接口（Collection，Set或Map）的常规约定。 这是因为这些接口的一般契约是根据 equals 方法定义的，但是有序集合使用 compareTo 强加的相等性测试来代替 equals。 如果发生这种情况，这不是灾难，但需要注意。

For example, consider the BigDecimal class, whose compareTo method is inconsistent with equals. If you create an empty HashSet instance and then add new BigDecimal("1.0") and new BigDecimal("1.00"),the set will contain two elements because the two BigDecimal instances added to the set are unequal when compared using the equals method. If,however, you perform the same procedure using a TreeSet instead of a HashSet, the set will contain only one element because the two BigDecimal instances are equal when compared using the compareTo method. (See the BigDecimal documentation for details.)

例如，考虑BigDecimal类，其 compareTo 方法与 equals 不一致。 如果您创建一个空的 HashSet 实例，然后添加new BigDecimal（“1.0”）和new BigDecimal（“1.00”），该集将包含两个元素，因为使用 equals 方法比较时，添加到集合中的两个BigDecimal实例是不相等的。 但是，如果使用 TreeSet 而不是 HashSet 执行相同的过程，则该集将仅包含一个元素，因为使用 compareTo 方法比较时，两个 BigDecimal 实例相等。 （有关详细信息，请参阅BigDecimal文档。）

Writing a compareTo method is similar to writing an equals method, but there are a few key differences. Because the Comparable interface is parameterized, the compareTo method is statically typed, so you don’t need to type check or cast its argument. If the argument is of the wrong type, the invocation won’t even compile. If the argument is null, the invocation should throw a NullPointer-Exception, and it will, as soon as the method attempts to access its members.

编写 compareTo 方法与编写 equals 方法类似，但存在一些关键差异。 因为 Comparable 接口是参数化的，所以compareTo 方法是静态类型的，因此您不需要键入 check 或 cast 其参数。 如果参数的类型错误，则调用甚至不会编译。 如果参数为 null ，则调用应该抛出 NullPointer-Exception，并且一旦方法尝试访问其成员，它就会抛出。

In a compareTo method, fields are compared for order rather than equality.To compare object reference fields, invoke the compareTo method recursively.If a field does not implement Comparable or you need a nonstandard ordering, use a Comparator instead. You can write your own comparator or use an existing one, as in this compareTo method for CaseInsensitiveString in Item 10:

在compareTo方法中，字段的比较采用顺序比较而不是相等比较。要比较对象引用字段，请递归调用compareTo方法。如果字段未实现Comparable或您需要非标准排序，请改用Comparator。 您可以编写自己的比较器或使用现有的，如 Item 10 中 CaseInsensitiveString 的 compareTo 方法：

```
// Single-field Comparable with object reference field
public final class CaseInsensitiveString
implements Comparable<CaseInsensitiveString> {
public int compareTo(CaseInsensitiveString cis) {
return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
} ... // Remainder omitted
}
```

Note that CaseInsensitiveString implements Comparable<CaseInsensitiveString>. This means that a CaseInsensitiveString reference can be compared only to another CaseInsensitiveString reference. This is the normal pattern to follow when declaring a class to implement Comparable.

请注意， CaseInsensitiveString 实现了 Comparable <CaseInsensitiveString>  。 这意味着CaseInsensitiveString 引用只能与另一个 CaseInsensitiveString 引用进行比较。 这是声明一个类来实现 Comparable 时要遵循的正常模式。

Prior editions of this book recommended that compareTo methods compare integral primitive fields using the relational operators < and >, and floating point primitive fields using the static methods Double.compare and Float.compare. In Java 7, static compare methods were added to all of Java’s boxed primitive classes. **Use of the relational operators < and > in compareTo methods is verbose and error-prone and no longer recommended.**

本书的老版本建议 compareTo 方法使用关系运算符 < 和 > 比较整数字段，使用静态方法 Double.compare 和 Float.compare 比较浮点基元字段。 在 Java 7 中，静态比较方法被添加到所有 Java 的装箱原始类中。 **因此，在compareTo 方法中使用关系运算符 < 和 > 是冗长且容易出错的，不再推荐使用。**

If a class has multiple significant fields, the order in which you compare them is critical. Start with the most significant field and work your way down. If a comparison results in anything other than zero (which represents equality),you’re done; just return the result. If the most significant field is equal, compare the next-most-significant field, and so on, until you find an unequal field or compare the least significant field. Here is a compareTo method for the PhoneNumber class in Item 11 demonstrating this technique:

如果一个类有多个重要字段，那么比较它们的顺序至关重要。 从最重要的字段开始，逐步完成。 如果比较产生除零以外的任何东西（代表相等），那么你就完成了; 直接返回结果即可。 如果最重要的字段相等，则比较次最重要的字段，依此类推，直到找到不相等的字段或比较最不重要的字段。 这是 Item 11 中 PhoneNumber 类的 compareTo 方法，演示了这种技术：

```java
// Multiple-field Comparable with primitive fields
public int compareTo(PhoneNumber pn) {
int result = Short.compare(areaCode, pn.areaCode);
if (result == 0) {
result = Short.compare(prefix, pn.prefix);
if (result == 0)
result = Short.compare(lineNum, pn.lineNum);
} return result;
}
```

In Java 8, the Comparator interface was outfitted with a set of comparator construction methods, which enable fluent construction of comparators. These comparators can then be used to implement a compareTo method, as required by the Comparable interface. Many programmers prefer the conciseness of this approach, though it does come at a modest performance cost: sorting arrays of PhoneNumber instances is about 10% slower on my machine. When using this approach, consider using Java’s static import facility so you can refer to static comparator construction methods by their simple names for clarity and brevity. Here’s how the compareTo method for PhoneNumber looks using this approach:

在Java 8中，Comparator 接口配备了一组比较器构造方法，让使用者能流畅地构造比较器。 然后，可以使用这些比较器来实现 Comparable 接口要求的 compareTo 方法。 许多程序员更喜欢这种方法的简洁性，虽然它确实消耗了适度的性能成本：在我的机器上排序 PhoneNumber 实例的数组大约慢10％。 使用此方法时，请考虑使用 Java 的静态导入工具，以便您可以通过简单的名称清晰和简洁地引用静态比较器构造方法。 以下是使用此方法的 PhoneNumber 的 compareTo 方法：

```java
// Comparable with comparator construction methods
private static final Comparator<PhoneNumber> COMPARATOR =
comparingInt((PhoneNumber pn) -> pn.areaCode)
.thenComparingInt(pn -> pn.prefix)
.thenComparingInt(pn -> pn.lineNum);
public int compareTo(PhoneNumber pn) {
return COMPARATOR.compare(this, pn);
}
```

This implementation builds a comparator at class initialization time, using two comparator construction methods. The first is comparingInt. It is a static method that takes a key extractor function that maps an object reference to a key of type int and returns a comparator that orders instances according to that key.In the previous example, comparingInt takes a lambda () that extracts the area code from a PhoneNumber and returns a Comparator<PhoneNumber> that orders phone numbers according to their area codes. Note that the lambda explicitly specifies the type of its input parameter (PhoneNumber pn). It turns out that in this situation, Java’s type inference isn’t powerful enough to figure the type out for itself, so we’re forced to help it in order to make the program compile.

这种实现方式使用两种比较器构造方法在类初始化时构建比较器。 第一个是 comparingInt 。 它是一个接收一个属性的静态方法，它将对象引用映射到 int 类型的值，并返回一个比较器，该比较器根据该键对实例进行排序。在前面的示例中，comparisonInt 采用一个 lambda 的形式从 PhoneNumber 中提取区号，并根据区号返回一个Comparator <PhoneNumber>用于 PhoneNumber 的排序。 请注意，lambda 显式指定其输入参数的类型（PhoneNumber pn）。 事实证明，在这种情况下，Java的类型推断并不足以为自己确定类型，因此我们不得不帮助它来编译程序。

If two phone numbers have the same area code, we need to further refine the comparison, and that’s exactly what the second comparator construction method,thenComparingInt, does. It is an instance method on Comparator that takes an int key extractor function, and returns a comparator that first applies the original comparator and then uses the extracted key to break ties. You can stack up as many calls to thenComparingInt as you like, resulting in a lexicographic ordering. In the example above, we stack up two calls to thenComparingInt, resulting in an ordering whose secondary key is the prefix and whose tertiary key is the line number. Note that we did not have to specify the parameter type of the key extractor function passed to either of the calls to thenComparingInt: Java’s type inference was smart enough to figure this one out for itself.

如果两个电话号码具有相同的区号，我们需要进一步细化比较，这正是第二个比较器构造方法 thenComparingInt 所做的。 它是 Comparator 上的一个实例方法，它是一个接收 int 属性的函数，并返回一个比较器，该比较器首先应用原始比较器，然后使用提取的属性来断开关系。 您可以根据需要向 thenComparingInt 堆叠尽可能多的调用，从而产生字典顺序。 在上面的示例中，我们将两个调用堆叠到 thenComparingInt，从而产生一个排序，其第二个对比的属性是前缀，第三个是行号。 请注意，我们没有必要指定传递给 thenComparingInt 的任一调用的函数的入参的参数类型：Java 的类型推断足够聪明，可以自己解决这个问题。

The Comparator class has a full complement of construction methods.There are analogues to comparingInt and thenComparingInt for the primitive types long and double. The int versions can also be used for narrower integral types, such as short, as in our PhoneNumber example. The double versions can also be used for float. This provides coverage of all of Java’s numerical primitive types.

Comparator 类具有完整的构造方法。对于原始类型 long 和 double ，也有 compareInt 和 thenComparingIn t的类似方法。 int 版本的方法也可用于较窄的整数类型，例如short，如我们的 PhoneNumber 示例中所示。double版本的方法也可用于 float。 这提供了所有 Java 的数字基本类型的覆盖。 

There are also comparator construction methods for object reference types.The static method, named comparing, has two overloadings. One takes a key extractor and uses the keys’ natural order. The second takes both a key extractor and a comparator to be used on the extracted keys. There are three overloadings of the instance method, which is named thenComparing. One overloading takes only a comparator and uses it to provide a secondary order. A second overloading takes only a key extractor and uses the key’s natural order as a secondary order. The final overloading takes both a key extractor and a comparator to be used on the extracted keys.

对象引用类型也有比较器构造方法。名为 comapring 的静态方法，有两个重载。 一个需要一个键提取器，并使用键的自然顺序。 第二个采用键提取器和比较器，用于提取键。 实例方法有三个重载，命名为 thenComparing 。 一个重载仅使用比较器并使用它来提供二级键提取器。 第二次重载仅使用一个键提取器，并使用键的自然顺序作为二级排序。 最后的重载需要一个键提取器和一个用于提取键的比较器。

Occasionally you may see compareTo or compare methods that rely on the fact that the difference between two values is negative if the first value is less than the second, zero if the two values are equal, and positive if the first value is greater. Here is an example:

有时您可能会看到 compareTo 或 compare 方法，这些方法依赖于两个值之间的差异，如果第一个值小于第二个值则为负值，如果两个值相等则为零，如果第一个值更大则为正值。 这是一个例子：

```java
// BROKEN difference-based comparator - violates transitivity!
static Comparator<Object> hashCodeOrder = new Comparator<>() {
public int compare(Object o1, Object o2) {
return o1.hashCode() - o2.hashCode();
} };
```

Do not use this technique. It is fraught with danger from integer overflow and IEEE 754 floating point arithmetic artifacts [JLS 15.20.1, 15.21.1]. Furthermore,the resulting methods are unlikely to be significantly faster than those written using the techniques described in this item. Use either a static compare method:

不要使用这种技术。 它充满了整数溢出和 IEEE 754 浮点运算伪像的危险 [JLS 15.20.1,15.21.1] 。 此外，所得到的方法不可能比使用本项目中描述的技术编写的方法快得多。 使用静态比较方法：

```
// Comparator based on static compare method
static Comparator<Object> hashCodeOrder = new Comparator<>() {
public int compare(Object o1, Object o2) {
return Integer.compare(o1.hashCode(), o2.hashCode());
} };
```

or a comparator construction method:

或比较器构造方法：

```java
// Comparator based on Comparator construction method
static Comparator<Object> hashCodeOrder =
Comparator.comparingInt(o -> o.hashCode());

```

In summary, whenever you implement a value class that has a sensible ordering, you should have the class implement the Comparable interface so that its instances can be easily sorted, searched, and used in comparison-based collections. When comparing field values in the implementations of the compareTo methods, avoid the use of the < and > operators. Instead, use the static compare methods in the boxed primitive classes or the comparator construction methods in the Comparator interface.

总之，每当实现具有合理排序的值类时，您应该让类实现 Comparable 接口，以便可以在基于比较的集合中轻松地对其实例进行排序，搜索和使用。 比较 compareTo 方法实现中的字段值时，请避免使用 < 和 > 运算符。 而是使用包装类中的静态比较方法或 Comparator 接口中的比较器构造方法。