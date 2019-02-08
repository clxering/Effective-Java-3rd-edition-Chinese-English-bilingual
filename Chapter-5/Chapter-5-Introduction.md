## Chapter 5. Generics（泛型）

### Chapter 5 Introduction（章节介绍）

SINCE Java 5, generics have been a part of the language. Before generics, you had to cast every object you read from a collection. If someone accidentally inserted an object of the wrong type, casts could fail at runtime. With generics,you tell the compiler what types of objects are permitted in each collection. The compiler inserts casts for you automatically and tells you at compile time if you try to insert an object of the wrong type. This results in programs that are both safer and clearer, but these benefits, which are not limited to collections, come at a price. This chapter tells you how to maximize the benefits and minimize the complications.

自 Java 5 以来，泛型一直是 Java 语言的一部分。在泛型出现之前，从集合中读取的每个对象都必须进行强制转换。如果有人不小心插入了错误类型的对象，强制类型转换可能在运行时失败。对于泛型，你可以告知编译器在每个集合中允许哪些类型的对象。编译器会自动为你进行强制转换与插入的操作，如果你试图插入类型错误的对象，编译器会在编译时告诉你。这就产生了更安全、更清晰的程序，但是这些好处不仅仅局限于集合，而且也是有代价的。这一章会告诉你如何最大限度地扬长避短。
