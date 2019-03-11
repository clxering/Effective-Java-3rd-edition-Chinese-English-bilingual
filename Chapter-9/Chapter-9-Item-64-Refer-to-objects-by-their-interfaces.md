## Chapter 9. General Programming（通用程序设计）

### Item 64: Refer to objects by their interfaces

Item 51 says that you should use interfaces rather than classes as parameter types. More generally, you should favor the use of interfaces over classes to refer to objects. **If appropriate interface types exist, then parameters, return values, variables, and fields should all be declared using interface types.** The only time you really need to refer to an object’s class is when you’re creating it with a constructor. To make this concrete, consider the case of LinkedHashSet, which is an implementation of the Set interface. Get in the habit of typing this:

```
// Good - uses interface as type
Set<Son> sonSet = new LinkedHashSet<>();
```

not this:

```
// Bad - uses class as type!
LinkedHashSet<Son> sonSet = new LinkedHashSet<>();
```

**If you get into the habit of using interfaces as types, your program will be much more flexible.** If you decide that you want to switch implementations, all you have to do is change the class name in the constructor (or use a different static factory). For example, the first declaration could be changed to read:

```
Set<Son> sonSet = new HashSet<>();
```

and all of the surrounding code would continue to work. The surrounding code was unaware of the old implementation type, so it would be oblivious to the change.

There is one caveat: if the original implementation offered some special functionality not required by the general contract of the interface and the code depended on that functionality, then it is critical that the new implementation provide the same functionality. For example, if the code surrounding the first declaration depended on LinkedHashSet’s ordering policy, then it would be incorrect to substitute HashSet for LinkedHashSet in the declaration, because HashSet makes no guarantee concerning iteration order.

So why would you want to change an implementation type? Because the second implementation offers better performance than the original, or because it offers desirable functionality that the original implementation lacks. For example, suppose a field contains a HashMap instance. Changing it to an EnumMap will provide better performance and iteration order consistent with the natural order of the keys, but you can only use an EnumMap if the key type is an enum type. Changing the HashMap to a LinkedHashMap will provide predictable iteration order with performance comparable to that of HashMap, without making any special demands on the key type.

You might think it’s OK to declare a variable using its implementation type, because you can change the declaration type and the implementation type at the same time, but there is no guarantee that this change will result in a program that compiles. If the client code used methods on the original implementation type that are not also present on its replacement or if the client code passed the instance to a method that requires the original implementation type, then the code will no longer compile after making this change. Declaring the variable with the interface type keeps you honest.

**It is entirely appropriate to refer to an object by a class rather than an interface if no appropriate interface exists.** For example, consider value classes, such as String and BigInteger. Value classes are rarely written with multiple implementations in mind. They are often final and rarely have corresponding interfaces. It is perfectly appropriate to use such a value class as a parameter, variable, field, or return type.

A second case in which there is no appropriate interface type is that of objects belonging to a framework whose fundamental types are classes rather than interfaces. If an object belongs to such a class-based framework, it is preferable to refer to it by the relevant base class, which is often abstract, rather than by its implementation class. Many java.io classes such as OutputStream fall into this category.

A final case in which there is no appropriate interface type is that of classes that implement an interface but also provide extra methods not found in the interface—for example, PriorityQueue has a comparator method that is not present on the Queue interface. Such a class should be used to refer to its instances only if the program relies on the extra methods, and this should be very rare.

These three cases are not meant to be exhaustive but merely to convey the flavor of situations where it is appropriate to refer to an object by its class. In practice, it should be apparent whether a given object has an appropriate interface. If it does, your program will be more flexible and stylish if you use the interface to refer to the object. **If there is no appropriate interface, just use the least specific class in the class hierarchy that provides the required functionality.**

---
**[Back to contents of the chapter（返回章节目录）](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-9/Chapter-9-Introduction.md)**
- **Previous Item（上一条目）：[Item 63: Beware the performance of string concatenation](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-9/Chapter-9-Item-63-Beware-the-performance-of-string-concatenation.md)**
- **Next Item（下一条目）：[Item 65: Prefer interfaces to reflection](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-9/Chapter-9-Item-65-Prefer-interfaces-to-reflection.md)**
