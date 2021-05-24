## Chapter 3. Methods Common to All Objects（对象的通用方法）

### Item 11: Always override hashCode when you override equals（当覆盖 equals 方法时，总要覆盖 hashCode 方法）

**You must override hashCode in every class that overrides equals.** If you fail to do so, your class will violate the general contract for hashCode, which will prevent it from functioning properly in collections such as HashMap and HashSet. Here is the contract, adapted from the Object specification:

**在覆盖了 equals 方法的类中，必须覆盖 hashCode 方法。** 如果你没有这样做，该类将违反 hashCode 方法的一般约定，这将阻止该类在 HashMap 和 HashSet 等集合中正常运行。以下是根据 Object 规范修改的约定：

- When the hashCode method is invoked on an object repeatedly during an execution of an application, it must consistently return the same value, provided no information used in equals comparisons is modified. This value need not remain consistent from one execution of an application to another.

应用程序执行期间对对象重复调用 hashCode 方法时，它必须一致地返回相同的值，前提是不对 equals 方法中用于比较的信息进行修改。这个值不需要在应用程序的不同执行之间保持一致。

- If two objects are equal according to the equals(Object) method, then calling hashCode on the two objects must produce the same integer result.

如果根据 `equals(Object)` 方法判断出两个对象是相等的，那么在两个对象上调用 hashCode 方法必须产生相同的整数结果。

- If two objects are unequal according to the equals(Object) method, it is not required that calling hashCode on each of the objects must produce distinct results. However, the programmer should be aware that producing distinct results for unequal objects may improve the performance of hash tables.

如果根据 `equals(Object)` 方法判断出两个对象不相等，则不需要在每个对象上调用 hashCode 方法时必须产生不同的结果。但是，程序员应该知道，为不相等的对象生成不同的结果可能会提高散列表的性能。

**The key provision that is violated when you fail to override hashCode is the second one: equal objects must have equal hash codes.** Two distinct instances may be logically equal according to a class’s equals method, but to Object’s hashCode method, they’re just two objects with nothing much in common. Therefore, Object’s hashCode method returns two seemingly random numbers instead of two equal numbers as required by the contract.For example, suppose you attempt to use instances of the PhoneNumber class from Item 10 as keys in a HashMap:

**当你无法覆盖 hashCode 方法时，将违反第二个关键条款：相等的对象必须具有相等的散列码。** 根据类的 equals 方法，两个不同的实例在逻辑上可能是相等的，但是对于对象的 hashCode 方法来说，它们只是两个没有共同之处的对象。因此，Object 的 hashCode 方法返回两个看似随机的数字，而不是约定要求的两个相等的数字。例如，假设你尝试使用[Item-10](/Chapter-3/Chapter-3-Item-10-Obey-the-general-contract-when-overriding-equals.md)中的 PhoneNumber 类实例作为 HashMap 中的键：

```Java
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707, 867, 5309), "Jenny");
```

At this point, you might expect m.get(new PhoneNumber(707, 867, 5309)) to return "Jenny", but instead, it returns null. Notice that two PhoneNumber instances are involved: one is used for insertion into the HashMap, and a second, equal instance is used for (attempted) retrieval. The PhoneNumber class’s failure to override hashCode causes the two equal instances to have unequal hash codes, in violation of the hashCode contract.Therefore, the get method is likely to look for the phone number in a different hash bucket from the one in which it was stored by the put method. Even if the two instances happen to hash to the same bucket, the get method will almost certainly return null, because HashMap has an optimization that caches the hash code associated with each entry and doesn’t bother checking for object equality if the hash codes don’t match.

此时，你可能期望 `m.get(new PhoneNumber(707, 867,5309))` 返回「Jenny」，但是它返回 null。注意，这里涉及到两个 PhoneNumber 实例：一个用于插入到 HashMap 中，另一个相等的实例（被试图）用于检索。由于 PhoneNumber 类未能覆盖 hashCode 方法，导致两个相等的实例具有不相等的散列码，这违反了 hashCode 方法约定。因此，get 方法查找电话号码的散列桶可能会与 put 方法存储电话号码的散列桶不同。即使这两个实例碰巧分配在同一个散列桶上，get 方法几乎肯定会返回 null，因为 HashMap 有一个优化，它缓存每个条目相关联的散列码，如果散列码不匹配，就不会检查对象是否相等。

Fixing this problem is as simple as writing a proper hashCode method for PhoneNumber. So what should a hashCode method look like? It’s trivial to write a bad one. This one, for example, is always legal but should never be used:

解决这个问题就像为 PhoneNumber 编写一个正确的 hashCode 方法一样简单。那么 hashCode 方法应该是什么样的呢？写一个反面例子很容易。例如，以下方法是合法的，但是不应该被使用：

```Java
// The worst possible legal hashCode implementation - never use!
@Override
public int hashCode() { return 42; }
```

It’s legal because it ensures that equal objects have the same hash code. It’s atrocious because it ensures that every object has the same hash code. Therefore,every object hashes to the same bucket, and hash tables degenerate to linked lists. Programs that should run in linear time instead run in quadratic time. For large hash tables, this is the difference between working and not working.

它是合法的，因为它确保了相等的对象具有相同的散列码。同时它也很糟糕，因为它使每个对象都有相同的散列码。因此，每个对象都分配到同一个桶中，散列表退化为链表。这样，原本应该在线性阶 `O(n)` 运行的程序将在平方阶 `O(n^2)` 运行。对于大型散列表，这是工作和不工作的区别。

A good hash function tends to produce unequal hash codes for unequal instances. This is exactly what is meant by the third part of the hashCode contract. Ideally, a hash function should distribute any reasonable collection of unequal instances uniformly across all int values. Achieving this ideal can be difficult. Luckily it’s not too hard to achieve a fair approximation. Here is a simple recipe:

一个好的散列算法倾向于为不相等的实例生成不相等的散列码。这正是 hashCode 方法约定第三部分的含义。理想情况下，一个散列算法应该在所有 int 值上均匀合理分布所有不相等实例集合。实现这个理想是很困难的。幸运的是，实现一个类似的并不太难。这里有一个简单的方式：

1、Declare an int variable named result, and initialize it to the hash code c for the first significant field in your object, as computed in step 2.a. (Recall from Item 10 that a significant field is a field that affects equals comparisons.)

声明一个名为 result 的 int 变量，并将其初始化为对象中第一个重要字段的散列码 c，如步骤 2.a 中计算的那样。（回想一下 [Item-10](/Chapter-3/Chapter-3-Item-10-Obey-the-general-contract-when-overriding-equals.md) 中的重要字段会对比较产生影响）

2、For every remaining significant field f in your object, do the following:

对象中剩余的重要字段 f，执行以下操作：

a. Compute an int hash code c for the field:

为字段计算一个整数散列码 c：

i. If the field is of a primitive type, compute Type.hashCode(f),where Type is the boxed primitive class corresponding to f’s type.

如果字段是基本数据类型，计算 `Type.hashCode(f)`，其中 type 是与 f 类型对应的包装类。

ii. If the field is an object reference and this class’s equals method compares the field by recursively invoking equals, recursively invoke hashCode on the field. If a more complex comparison is required,compute a “canonical representation” for this field and invoke hashCode on the canonical representation. If the value of the field is null, use 0 (or some other constant, but 0 is traditional).

如果字段是对象引用，并且该类的 equals 方法通过递归调用 equals 方法来比较字段，则递归调用字段上的 hashCode 方法。如果需要更复杂的比较，则为该字段计算一个「canonical representation」，并在 canonical representation 上调用 hashCode 方法。如果字段的值为空，则使用 0（或其他常数，但 0 是惯用的）。

iii. If the field is an array, treat it as if each significant element were a separate field. That is, compute a hash code for each significant element by applying these rules recursively, and combine the values per step 2.b. If the array has no significant elements, use a constant, preferably not 0. If all elements are significant, use Arrays.hashCode.

如果字段是一个数组，则将其每个重要元素都视为一个单独的字段。也就是说，通过递归地应用这些规则计算每个重要元素的散列码，并将每个步骤 2.b 的值组合起来。如果数组中没有重要元素，则使用常量，最好不是 0。如果所有元素都很重要，那么使用 `Arrays.hashCode`。

b. Combine the hash code c computed in step 2.a into result as follows:

将步骤 2.a 中计算的散列码 c 合并到 result 变量，如下所示：

```Java
result = 31 * result + c;
```

3、Return result.

返回 result 变量。

When you are finished writing the hashCode method, ask yourself whether equal instances have equal hash codes. Write unit tests to verify your intuition (unless you used AutoValue to generate your equals and hashCode methods,in which case you can safely omit these tests). If equal instances have unequal hash codes, figure out why and fix the problem.

当你完成了 hashCode 方法的编写之后，问问自己现在相同的实例是否具有相同的散列码。编写单元测试来验证你的直觉（除非你使用 AutoValue 生成你的 equals 方法和 hashCode 方法，在这种情况下你可以安全地省略这些测试）。如果相同的实例有不相等的散列码，找出原因并修复问题。

You may exclude derived fields from the hash code computation. In other words, you may ignore any field whose value can be computed from fields included in the computation. You must exclude any fields that are not used in equals comparisons, or you risk violating the second provision of the hashCode contract.

可以从散列码计算中排除派生字段。换句话说，你可以忽略任何可以从包含的字段计算其值的字段。你必须排除不用 `equals` 比较的任何字段，否则你可能会违反 hashCode 方法约定的第二个条款。

The multiplication in step 2.b makes the result depend on the order of the fields, yielding a much better hash function if the class has multiple similar fields. For example, if the multiplication were omitted from a String hash function, all anagrams would have identical hash codes. The value 31 was chosen because it is an odd prime. If it were even and the multiplication overflowed, information would be lost, because multiplication by 2 is equivalent to shifting. The advantage of using a prime is less clear, but it is traditional. A nice property of 31 is that the multiplication can be replaced by a shift and a subtraction for better performance on some architectures: 31 \* i == (i <<5) - i. Modern VMs do this sort of optimization automatically.

在步骤 2.b 中使用的乘法将使结果取决于字段的顺序，如果类有多个相似的字段，则会产生一个更好的散列算法。例如，如果字符串散列算法中省略了乘法，那么所有的字母顺序都有相同的散列码。选择 31 是因为它是奇素数。如果是偶数，乘法运算就会溢出，信息就会丢失，因为乘法运算等同于移位。使用素数的好处不太明显，但它是传统用法。31 有一个很好的特性，可以用移位和减法来代替乘法，从而在某些体系结构上获得更好的性能：`31 * i == (i <<5) – i`。现代虚拟机自动进行这种优化。

Let’s apply the previous recipe to the PhoneNumber class:

让我们将前面的方法应用到 PhoneNumber 类：

```Java
// Typical hashCode method
@Override
public int hashCode() {
    int result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashCode(lineNum);
    return result;
}
```

Because this method returns the result of a simple deterministic computation whose only inputs are the three significant fields in a PhoneNumber instance,it is clear that equal PhoneNumber instances have equal hash codes. This method is, in fact, a perfectly good hashCode implementation for PhoneNumber, on par with those in the Java platform libraries. It is simple, is reasonably fast, and does a reasonable job of dispersing unequal phone numbers into different hash buckets.

因为这个方法返回一个简单的确定的计算结果，它的唯一输入是 PhoneNumber 实例中的三个重要字段，所以很明显，相等的 PhoneNumber 实例具有相等的散列码。实际上，这个方法是 PhoneNumber 的一个非常好的 hashCode 方法实现，与 Java 库中的 hashCode 方法实现相当。它很简单，速度也相当快，并且合理地将不相等的电话号码分散到不同的散列桶中。

While the recipe in this item yields reasonably good hash functions, they are not state-of-the-art. They are comparable in quality to the hash functions found in the Java platform libraries’ value types and are adequate for most uses. If you have a bona fide need for hash functions less likely to produce collisions, see Guava’s com.google.common.hash.Hashing [Guava].

虽然本条目中的方法产生了相当不错的散列算法，但它们并不是最先进的。它们的质量可与 Java 库的值类型中的散列算法相媲美，对于大多数用途来说都是足够的。如果你确实需要不太可能产生冲突的散列算法，请参阅 Guava 的 com.google.common.hash.Hashing [Guava]。

The Objects class has a static method that takes an arbitrary number of objects and returns a hash code for them. This method, named hash, lets you write one-line hashCode methods whose quality is comparable to those written according to the recipe in this item. Unfortunately, they run more slowly because they entail array creation to pass a variable number of arguments, as well as boxing and unboxing if any of the arguments are of primitive type. This style of hash function is recommended for use only in situations where performance is not critical. Here is a hash function for PhoneNumber written using this technique:

Objects 类有一个静态方法，它接受任意数量的对象并返回它们的散列码。这个名为 `hash` 的方法允许你编写只有一行代码的 hashCode 方法，这些方法的质量可以与本条目中提供的编写方法媲美。不幸的是，它们运行得更慢，因为它们需要创建数组来传递可变数量的参数，如果任何参数是原始类型的，则需要进行装箱和拆箱。推荐只在性能不重要的情况下使用这种散列算法。下面是使用这种技术编写的 PhoneNumber 的散列算法：

```Java
// One-line hashCode method - mediocre performance
@Override
public int hashCode() {
    return Objects.hash(lineNum, prefix, areaCode);
}
```

If a class is immutable and the cost of computing the hash code is significant,you might consider caching the hash code in the object rather than recalculating it each time it is requested. If you believe that most objects of this type will be used as hash keys, then you should calculate the hash code when the instance is created. Otherwise, you might choose to lazily initialize the hash code the first time hash-Code is invoked. Some care is required to ensure that the class remains thread-safe in the presence of a lazily initialized field (Item 83). Our PhoneNumber class does not merit this treatment, but just to show you how it’s done, here it is. Note that the initial value for the hashCode field (in this case, 0) should not be the hash code of a commonly created instance:

如果一个类是不可变的，并且计算散列码的成本非常高，那么你可以考虑在对象中缓存散列码，而不是在每次请求时重新计算它。如果你认为这种类型的大多数对象都将用作散列键，那么你应该在创建实例时计算散列码。否则，你可以选择在第一次调用 hashCode 方法时延迟初始化散列码。在一个延迟初始化的字段（[Item-83](/Chapter-11/Chapter-11-Item-83-Use-lazy-initialization-judiciously.md)）的情况下，需要注意以确保该类仍然是线程安全的。我们的 PhoneNumber 类不值得进行这种处理，但只是为了向你展示它是如何实现的，如下所示。注意，散列字段的初始值（在本例中为 0）不应该是通常创建的实例的散列码：

```Java
// hashCode method with lazily initialized cached hash code
private int hashCode; // Automatically initialized to 0
@Override
public int hashCode() {
    int result = hashCode;

    if (result == 0) {
        result = Short.hashCode(areaCode);
        result = 31 * result + Short.hashCode(prefix);
        result = 31 * result + Short.hashCode(lineNum);
        hashCode = result;
    }

    return result;
}
```

**Do not be tempted to exclude significant fields from the hash code computation to improve performance.** While the resulting hash function may run faster, its poor quality may degrade hash tables’ performance to the point where they become unusable. In particular, the hash function may be confronted with a large collection of instances that differ mainly in regions you’ve chosen to ignore. If this happens, the hash function will map all these instances to a few hash codes, and programs that should run in linear time will instead run in quadratic time.

**不要试图从散列码计算中排除重要字段，以提高性能。** 虽然得到的散列算法可能运行得更快，但其糟糕的质量可能会将散列表的性能降低到无法使用的程度。特别是，散列算法可能会遇到大量实例，这些实例在你选择忽略的不同区域。如果发生这种情况，散列算法将把所有这些实例映射很少一部分散列码，使得原本应该在线性阶 `O(n)` 运行的程序将在平方阶 `O(n^2)` 运行。

This is not just a theoretical problem. Prior to Java 2, the String hash function used at most sixteen characters evenly spaced throughout the string,starting with the first character. For large collections of hierarchical names, such as URLs, this function displayed exactly the pathological behavior described earlier.

这不仅仅是一个理论问题。在 Java 2 之前，字符串散列算法在字符串中，以第一个字符开始，最多使用 16 个字符。对于大量且分层次的集合（如 url），该函数完全展示了前面描述的病态行为。

**Don’t provide a detailed specification for the value returned by hashCode, so clients can’t reasonably depend on it; this gives you the flexibility to change it.** Many classes in the Java libraries, such as String and Integer, specify the exact value returned by their hashCode method as a function of the instance value. This is not a good idea but a mistake that we’re forced to live with: It impedes the ability to improve the hash function in future releases. If you leave the details unspecified and a flaw is found in the hash function or a better hash function is discovered, you can change it in a subsequent release.

**不要为 hashCode 返回的值提供详细的规范，这样客户端就不能理所应当的依赖它。这（也）给了你更改它的余地。** Java 库中的许多类，例如 String 和 Integer，都将 hashCode 方法返回的确切值指定为实例值的函数。这不是一个好主意，而是一个我们不得不面对的错误：它阻碍了在未来版本中提高散列算法的能力。如果你保留了未指定的细节，并且在散列算法中发现了缺陷，或者发现了更好的散列算法，那么你可以在后续版本中更改它。

In summary, you must override hashCode every time you override equals,or your program will not run correctly. Your hashCode method must obey the general contract specified in Object and must do a reasonable job assigning unequal hash codes to unequal instances. This is easy to achieve, if slightly tedious, using the recipe on page 51. As mentioned in Item 10, the AutoValue framework provides a fine alternative to writing equals and hashCode methods manually, and IDEs also provide some of this functionality.

总之，每次覆盖 equals 方法时都必须覆盖 hashCode 方法，否则程序将无法正确运行。你的 hashCode 方法必须遵守 Object 中指定的通用约定，并且必须合理地将不相等的散列码分配给不相等的实例。这很容易实现，如果有点枯燥，可使用第 51 页的方法。如 [Item-10](/Chapter-3/Chapter-3-Item-10-Obey-the-general-contract-when-overriding-equals.md) 所述，AutoValue 框架提供了一种能很好的替代手动编写 equals 方法和 hashCode 方法的功能，IDE 也提供了这种功能。

---

**[Back to contents of the chapter（返回章节目录）](/Chapter-3/Chapter-3-Introduction.md)**

- **Previous Item（上一条目）：[Item 10: Obey the general contract when overriding equals（覆盖 equals 方法时应遵守的约定）](/Chapter-3/Chapter-3-Item-10-Obey-the-general-contract-when-overriding-equals.md)**
- **Next Item（下一条目）：[Item 12: Always override toString（始终覆盖 toString 方法）](/Chapter-3/Chapter-3-Item-12-Always-override-toString.md)**
