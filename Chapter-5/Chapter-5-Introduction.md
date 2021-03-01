## Chapter 5. Generics（泛型）

### Chapter 5 Introduction（章节介绍）

SINCE Java 5, generics have been a part of the language. Before generics, you had to cast every object you read from a collection. If someone accidentally inserted an object of the wrong type, casts could fail at runtime. With generics,you tell the compiler what types of objects are permitted in each collection. The compiler inserts casts for you automatically and tells you at compile time if you try to insert an object of the wrong type. This results in programs that are both safer and clearer, but these benefits, which are not limited to collections, come at a price. This chapter tells you how to maximize the benefits and minimize the complications.

自 Java 5 以来，泛型一直是 Java 语言的一部分。在泛型出现之前，从集合中读取的每个对象都必须进行强制转换。如果有人不小心插入了错误类型的对象，强制类型转换可能在运行时失败。对于泛型，你可以告知编译器在每个集合中允许哪些类型的对象。编译器会自动为你进行强制转换与插入的操作，如果你试图插入类型错误的对象，编译器会在编译时告诉你。这就产生了更安全、更清晰的程序，但是这些好处不仅仅局限于集合，而且也是有代价的。这一章会告诉你如何最大限度地扬长避短。

### Contents of the chapter（章节目录）
- [Item 26: Do not use raw types（不要使用原始类型）](/Chapter-5/Chapter-5-Item-26-Do-not-use-raw-types.md)
- [Item 27: Eliminate unchecked warnings（消除 unchecked 警告）](/Chapter-5/Chapter-5-Item-27-Eliminate-unchecked-warnings.md)
- [Item 28: Prefer lists to arrays（list 优于数组）](/Chapter-5/Chapter-5-Item-28-Prefer-lists-to-arrays.md)
- [Item 29: Favor generic types（优先使用泛型）](/Chapter-5/Chapter-5-Item-29-Favor-generic-types.md)
- [Item 30: Favor generic methods（优先使用泛型方法）](/Chapter-5/Chapter-5-Item-30-Favor-generic-methods.md)
- [Item 31: Use bounded wildcards to increase API flexibility（使用有界通配符增加 API 的灵活性）](/Chapter-5/Chapter-5-Item-31-Use-bounded-wildcards-to-increase-API-flexibility.md)
- [Item 32: Combine generics and varargs judiciously（明智地合用泛型和可变参数）](/Chapter-5/Chapter-5-Item-32-Combine-generics-and-varargs-judiciously.md)
- [Item 33: Consider typesafe heterogeneous containers（考虑类型安全的异构容器）](/Chapter-5/Chapter-5-Item-33-Consider-typesafe-heterogeneous-containers.md)
