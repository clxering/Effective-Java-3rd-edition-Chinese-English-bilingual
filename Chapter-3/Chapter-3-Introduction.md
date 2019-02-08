## Chapter 3. Methods Common to All Objects（对象的通用方法）

### Chapter 3 Introduction（章节介绍）

ALTHOUGH Object is a concrete class, it is designed primarily for extension. All of its nonfinal methods (equals, hashCode, toString, clone, and finalize) have explicit general contracts because they are designed to be overridden. It is the responsibility of any class overriding these methods to obey their general contracts; failure to do so will prevent other classes that depend on the contracts (such as HashMap and HashSet) from functioning properly in conjunction with the class.

虽然 Object 是一个具体的类，但它主要是为扩展而设计的。它的所有非 final 方法（equals、hashCode、toString、clone 和 finalize）都有显式的通用约定，因为它们的设计目的是被覆盖。任何类都有责任覆盖这些方法并将之作为一般约定；如果不这样做，将阻止依赖于约定的其他类（如 HashMap 和 HashSet）与之一起正常工作。

This chapter tells you when and how to override the nonfinal Object methods. The finalize method is omitted from this chapter because it was discussed in Item 8. While not an Object method,Comparable.compareTo is discussed in this chapter because it has a similar character.

本章将告诉你何时以及如何覆盖 Object 类的非 final 方法。finalize 方法在本章中被省略，因为它在 [Item-8](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-2/Chapter-2-Item-8-Avoid-finalizers-and-cleaners.md) 中讨论过。虽然 Comparable.compareTo 不是 Object 类的方法，但是由于具有相似的特性，所以本章也对它进行讨论。
