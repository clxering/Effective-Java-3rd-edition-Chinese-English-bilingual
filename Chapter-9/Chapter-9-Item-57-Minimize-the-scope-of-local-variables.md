## Chapter 9. General Programming（通用程序设计）

### Item 57: Minimize the scope of local variables

This item is similar in nature to Item 15, “Minimize the accessibility of classes and members.” By minimizing the scope of local variables, you increase the readability and maintainability of your code and reduce the likelihood of error.

Older programming languages, such as C, mandated that local variables must be declared at the head of a block, and some programmers continue to do this out of habit. It’s a habit worth breaking. As a gentle reminder, Java lets you declare variables anywhere a statement is legal (as does C, since C99).

**The most powerful technique for minimizing the scope of a local variable is to declare it where it is first used.** If a variable is declared before it is used, it’s just clutter—one more thing to distract the reader who is trying to figure out what the program does. By the time the variable is used, the reader might not remember the variable’s type or initial value.

Declaring a local variable prematurely can cause its scope not only to begin too early but also to end too late. The scope of a local variable extends from the point where it is declared to the end of the enclosing block. If a variable is declared outside of the block in which it is used, it remains visible after the program exits that block. If a variable is used accidentally before or after its region of intended use, the consequences can be disastrous.

**Nearly every local variable declaration should contain an initializer.** If you don’t yet have enough information to initialize a variable sensibly, you should postpone the declaration until you do. One exception to this rule concerns try-catch statements. If a variable is initialized to an expression whose evaluation can throw a checked exception, the variable must be initialized inside a try block (unless the enclosing method can propagate the exception). If the value must be used outside of the try block, then it must be declared before the try block, where it cannot yet be “sensibly initialized.” For an example, see page 283.

Loops present a special opportunity to minimize the scope of variables. The for loop, in both its traditional and for-each forms, allows you to declare loop variables, limiting their scope to the exact region where they’re needed. (This region consists of the body of the loop and the code in parentheses between the for keyword and the body.) Therefore, prefer for loops to while loops, assuming the contents of the loop variable aren’t needed after the loop terminates.

For example, here is the preferred idiom for iterating over a collection (Item 58):

```
// Preferred idiom for iterating over a collection or array
for (Element e : c) {
    ... // Do Something with e
}
```

If you need access to the iterator, perhaps to call its remove method, the preferred idiom uses a traditional for loop in place of the for-each loop:

```
// Idiom for iterating when you need the iterator
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
    Element e = i.next();
    ... // Do something with e and i
}
```

To see why these for loops are preferable to a while loop, consider the following code fragment, which contains two while loops and one bug:

```
Iterator<Element> i = c.iterator();
while (i.hasNext()) {
    doSomething(i.next());
}
...
Iterator<Element> i2 = c2.iterator();
while (i.hasNext()) { // BUG!
    doSomethingElse(i2.next());
}
```

The second loop contains a copy-and-paste error: it initializes a new loop variable, i2, but uses the old one, i, which is, unfortunately, still in scope. The resulting code compiles without error and runs without throwing an exception, but it does the wrong thing. Instead of iterating over c2, the second loop terminates immediately, giving the false impression that c2 is empty. Because the program errs silently, the error can remain undetected for a long time.

If a similar copy-and-paste error were made in conjunction with either of the for loops (for-each or traditional), the resulting code wouldn’t even compile. The element (or iterator) variable from the first loop would not be in scope in the second loop. Here’s how it looks with the traditional for loop:

```
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
Element e = i.next();
... // Do something with e and i
}
...
// Compile-time error - cannot find symbol i
for (Iterator<Element> i2 = c2.iterator(); i.hasNext(); ) {
Element e2 = i2.next();
... // Do something with e2 and i2
}
```

Moreover, if you use a for loop, it’s much less likely that you’ll make the copy-and-paste error because there’s no incentive to use different variable names in the two loops. The loops are completely independent, so there’s no harm in reusing the element (or iterator) variable name. In fact, it’s often stylish to do so. The for loop has one more advantage over the while loop: it is shorter, which enhances readability. Here is another loop idiom that minimizes the scope of local variables:

```
for (int i = 0, n = expensiveComputation(); i < n; i++) {
    ... // Do something with i;
}
```

The important thing to notice about this idiom is that it has two loop variables, i and n, both of which have exactly the right scope. The second variable, n, is used to store the limit of the first, thus avoiding the cost of a redundant computation in every iteration. As a rule, you should use this idiom if the loop test involves a method invocation that is guaranteed to return the same result on each iteration.

A final technique to minimize the scope of local variables is to keep methods small and focused. If you combine two activities in the same method, local variables relevant to one activity may be in the scope of the code performing the other activity. To prevent this from happening, simply separate the method into two: one for each activity.

---
**[Back to contents of the chapter（返回章节目录）](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-9/Chapter-9-Introduction.md)**
- **Next Item（下一条目）：[Item 58: Prefer for each loops to traditional for loops](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-9/Chapter-9-Item-58-Prefer-for-each-loops-to-traditional-for-loops.md)**
