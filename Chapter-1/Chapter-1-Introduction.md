## Chapter 1. Introduction

This book is designed to help you make effective use of the Java programming language and its fundamental libraries: `java.lang`, `java.util`, and `java.io`, and subpackages such as `java.util.concurrent` and `java.util.function`. Other libraries are discussed from time to time.

This book consists of ninety items, each of which conveys one rule. The rules capture practices generally held to be beneficial by the best and most experienced programmers. The items are loosely grouped into eleven chapters, each covering one broad aspect of software design. The book is not intended to be read from cover to cover: each item stands on its own, more or less. The items are heavily cross-referenced so you can easily plot your own course through the book.

Many new features were added to the platform since the last edition of this book was published. Most of the items in this book use these features in some way. This table shows you where to go for primary coverage of key features:

| Feature                       | Items       | Release |
|-------------------------------|-------------|---------|
| Lambdas                       | Items [42](../Chapter-7/Chapter-7-Item-42-Prefer-lambdas-to-anonymous-classes.md)-[44](Chapter-7/Chapter-7-Item-44-Favor-the-use-of-standard-functional-interfaces.md) | Java 8  |
| Streams                       | Items 45-48 | Java 8  |
| Optionals                     | Item 55     | Java 8  |
| Default methods in interfaces | Item 21     | Java 8  |
| `try`-with-resources          | Item 9      | Java 7  |
| `@SafeVarargs`                | Item 32     | Java 7  |
| Modules                       | Item 15     | Java 7  |

Most items are illustrated with program examples. A key feature of this book is that it contains code examples illustrating many design patterns and idioms. Where appropriate, they are cross-referenced to the standard reference work in this area [Gamma95].

Many items contain one or more program examples illustrating some practice to be avoided. Such examples, sometimes known as antipatterns, are clearly labeled with a comment such as // Never do this!. In each case, the item explains why the example is bad and suggests an alternative approach. 

This book is not for beginners: it assumes that you are already comfortable with Java. If you are not, consider one of the many fine introductory texts, such as Peter Sestoft’s Java Precisely [Sestoft16]. While Effective Java is designed to be accessible to anyone with a working knowledge of the language, it should provide food for thought even for advanced programmers.

Most of the rules in this book derive from a few fundamental principles. Clarity and simplicity are of paramount importance. The user of a component should never be surprised by its behavior. Components should be as small as possible but no smaller. (As used in this book, the term component refers to any reusable software element, from an individual method to a complex framework consisting of multiple packages.) Code should be reused rather than copied. The dependencies between components should be kept to a minimum. Errors should be detected as soon as possible after they are made, ideally at compile time.

While the rules in this book do not apply 100 percent of the time, they do characterize best programming practices in the great majority of cases. You should not slavishly follow these rules, but violate them only occasionally and with good reason. Learning the art of programming, like most other disciplines, consists of first learning the rules and then learning when to break them.

For the most part, this book is not about performance. It is about writing programs that are clear, correct, usable, robust, flexible, and maintainable. If you can do that, it’s usually a relatively simple matter to get the performance you need (Item 67). Some items do discuss performance concerns, and a few of these items provide performance numbers. These numbers, which are introduced with the phrase “On my machine,” should be regarded as approximate at best.

Bloch, Joshua. Effective Java (p. 2). Pearson Education. Kindle Edition. 

---
**[Back to contents of the chapter（返回章节目录）](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-2/Chapter-2-Introduction.md)**
- **Next Item（下一条目）：[Chapter 2 Introduction（章节介绍）](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-2/Chapter-2-Introduction.md)**
