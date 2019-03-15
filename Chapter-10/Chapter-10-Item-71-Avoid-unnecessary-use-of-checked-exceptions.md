## Chapter 10. Exceptions（异常）

### Item 71: Avoid unnecessary use of checked exceptions

Many Java programmers dislike checked exceptions, but used properly, they can improve APIs and programs. Unlike return codes and unchecked exceptions, they force programmers to deal with problems, enhancing reliability. That said, overuse of checked exceptions in APIs can make them far less pleasant to use. If a method throws checked exceptions, the code that invokes it must handle them in one or more catch blocks, or declare that it throws them and let them propagate outward. Either way, it places a burden on the user of the API. The burden increased in Java 8, as methods throwing checked exceptions can’t be used directly in streams (Items 45–48).

许多 Java 程序员不喜欢 checked 异常，但是如果使用得当，它们可以有利于 API 和程序。与返回代码和 unchecked 异常不同，它们强制程序员处理问题，提高了可靠性。相反，在 API 中过度使用 checked 异常会变得不那么令人愉快。如果一个方法抛出 checked 异常，调用它的代码必须在一个或多个 catch 块中处理它们；或者通过声明抛出，让它们向外传播。无论哪种方式，它都给 API 的用户带来了负担。Java 8 中的负担增加了，因为抛出 checked 异常的方法不能直接在流中使用 **(项目45-48)。**

This burden may be justified if the exceptional condition cannot be prevented by proper use of the API and the programmer using the API can take some useful action once confronted with the exception. Unless both of these conditions are met, an unchecked exception is appropriate. As a litmus test, ask yourself how the programmer will handle the exception. Is this the best that can be done?

如果（1）正确使用 API 也不能防止异常情况，（2）并且使用 API 的程序员在遇到异常时可以采取一些有用的操作，那么这种负担是合理的。除非满足这两个条件，否则可以使用 unchecked 异常。作为一个 **试金石，** 程序员应该问问自己将如何处理异常。这是最好的办法吗？

```
} catch (TheCheckedException e) {
    throw new AssertionError(); // Can't happen!
}
```

Or this?

或者这样？

```
} catch (TheCheckedException e) {
    e.printStackTrace(); // Oh well, we lose.
    System.exit(1);
}
```

If the programmer can do no better, an unchecked exception is called for.

如果程序员不能做得更好，则需要一个 unchecked 异常。

The additional burden on the programmer caused by a checked exception is substantially higher if it is the sole checked exception thrown by a method. If there are others, the method must already appear in a try block, and this exception requires, at most, another catch block. If a method throws a single checked exception, this exception is the sole reason the method must appear in a try block and can’t be used directly in streams. Under these circumstances, it pays to ask yourself if there is a way to avoid the checked exception.

如果 checked 异常是方法抛出的唯一 checked 异常，那么 checked 异常给程序员带来的额外负担就会大得多。如果还有其他方法，则该方法必须已经出现在 try 块中，并且此异常最多需要另一个 catch 块。如果一个方法抛出一个 checked 异常，那么这个异常就是该方法必须出现在 try 块中而不能直接在流中使用的唯一原因。在这种情况下，有必要问问自己是否有办法避免 checked 异常。

被一个方法单独拋出的受检异常，会给程序员带来非常高的额外负担。如果这个方法还有其他的受检异常，它被调用的时候一定已经出现在一个try块中，所以这个异常只需要另外一个catch块。如果方法只抛出单个受检的异常，仅仅一个异常就会导致该方法不得不外于try块中。在这些情况下，应该问自己，是否有别的途径来避免使用受检的异常。

这种例子就是CloneNotSupportedException。它是被Object.clone抛出来的，而Object.clone应该 只是在实现了Cloneable的对象上才可以被调用(见第11条)。 在实践中，catch块几乎总是具有断言(assertion)失败的特征。异常受检的本质并没有为程序员提供任何好处，它反而需要付出努力，还使程序更为复杂。

The easiest way to eliminate a checked exception is to return an optional of the desired result type (Item 55). Instead of throwing a checked exception, the method simply returns an empty optional. The disadvantage of this technique is that the method can’t return any additional information detailing its inability to perform the desired computation. Exceptions, by contrast, have descriptive types, and can export methods to provide additional information (Item 70).

消除 checked 异常的最简单方法是返回所需结果类型的 Optional 对象(第55项)。该方法只返回一个空的可选异常，而不是抛出一个已检查的异常。这种技术的缺点是，该方法不能返回任何详细说明其无法执行所需计算的附加信息。相反，异常具有描述性类型，并且可以导出方法来提供附加信息(第70项)。

把受检的异常变成未受检的异常”的-种方法是，把这个抛出异常的方法分成两个方法，其中第一个方法返回一个boolean,表明是否应该抛出异常。这种API重构，把下面的调用序列:

You can also turn a checked exception into an unchecked exception by breaking the method that throws the exception into two methods, the first of which returns a boolean indicating whether the exception would be thrown. This API refactoring transforms the calling sequence from this:

```
// Invocation with checked exception
try {
    obj.action(args);
}
catch (TheCheckedException e) {
    ... // Handle exceptional condition
}
```

into this:

```
// Invocation with state-testing method and unchecked exception
if (obj.actionPermitted(args)) {
    obj.action(args);
}
else {
    ... // Handle exceptional condition
}
```

This refactoring is not always appropriate, but where it is, it can make an API more pleasant to use. While the latter calling sequence is no prettier than the former, the refactored API is more flexible. If the programmer knows the call will succeed, or is content to let the thread terminate if it fails, the refactoring also allows this trivial calling sequence:

```
obj.action(args);
```

If you suspect that the trivial calling sequence will be the norm, then the API refactoring may be appropriate. The resulting API is essentially the state-testing method API in Item 69 and the same caveats apply: if an object is to be accessed concurrently without external synchronization or it is subject to externally induced state transitions, this refactoring is inappropriate because the object’s state may change between the calls to actionPermitted and action. If a separate actionPermitted method would duplicate the work of the action method, the refactoring may be ruled out on performance grounds.

In summary, when used sparingly, checked exceptions can increase the reliability of programs; when overused, they make APIs painful to use. If callers won’t be able to recover from failures, throw unchecked exceptions. If recovery may be possible and you want to force callers to handle exceptional conditions, first consider returning an optional. Only if this would provide insufficient information in the case of failure should you throw a checked exception.

---
**[Back to contents of the chapter（返回章节目录）](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-10/Chapter-10-Introduction.md)**
- **Previous Item（上一条目）：[Item 70: Use checked exceptions for recoverable conditions and runtime exceptions for programming errors](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-10/Chapter-10-Item-70-Use-checked-exceptions-for-recoverable-conditions-and-runtime-exceptions-for-programming-errors.md)**
- **Next Item（下一条目）：[Item 72: Favor the use of standard exceptions](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-10/Chapter-10-Item-72-Favor-the-use-of-standard-exceptions.md)**
