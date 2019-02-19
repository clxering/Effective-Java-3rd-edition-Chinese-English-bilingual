## Chapter 6. Enums and Annotations（枚举和注解）

### Item 40: Consistently use the Override annotation

The Java libraries contain several annotation types. For the typical programmer, the most important of these is @Override. This annotation can be used only on method declarations, and it indicates that the annotated method declaration overrides a declaration in a supertype. If you consistently use this annotation, it will protect you from a large class of nefarious bugs. Consider this program, in which the class Bigram represents a bigram, or ordered pair of letters:

```
// Can you spot the bug?
public class Bigram {
private final char first;
private final char second;
public Bigram(char first, char second) {
this.first = first;
this.second = second;
} public boolean equals(Bigram b) {
return b.first == first && b.second == second;
} public int hashCode() {
return 31 * first + second;
}
public static void main(String[] args) {
Set<Bigram> s = new HashSet<>();
for (int i = 0; i < 10; i++)
for (char ch = 'a'; ch <= 'z'; ch++)
s.add(new Bigram(ch, ch));
System.out.println(s.size());
}}
```

The main program repeatedly adds twenty-six bigrams, each consisting of two identical lowercase letters, to a set. Then it prints the size of the set. You might expect the program to print 26, as sets cannot contain duplicates. If you try running the program, you’ll find that it prints not 26 but 260. What is wrong with it?

Clearly, the author of the Bigram class intended to override the equals method (Item 10) and even remembered to override hashCode in tandem (Item 11). Unfortunately, our hapless programmer failed to override equals, overloading it instead (Item 52). To override Object.equals, you must define an equals method whose parameter is of type Object, but the parameter of Bigram’s equals method is not of type Object, so Bigram inherits the equals method from Object. This equals method tests for object identity, just like the == operator. Each of the ten copies of each bigram is distinct from the other nine, so they are deemed unequal by Object.equals, which explains why the program prints 260.

Luckily, the compiler can help you find this error, but only if you help it by telling it that you intend to override Object.equals. To do this, annotate Bigram.equals with @Override, as shown here:

```
@Override public boolean equals(Bigram b) {
return b.first == first && b.second == second;
}
```

If you insert this annotation and try to recompile the program, the compiler will generate an error message like this:

```
Bigram.java:10: method does not override or implement a method from a supertype
@Override public boolean equals(Bigram b) {
^
```

You will immediately realize what you did wrong, slap yourself on the forehead, and replace the broken equals implementation with a correct one (Item 10):

```
@Override public boolean equals(Object o) {
if (!(o instanceof Bigram))
return false;
Bigram b = (Bigram) o;
return b.first == first && b.second == second;
}
```

Therefore, you should **use the Override annotation on every method declaration that you believe to override a superclass declaration.** There is one minor exception to this rule. If you are writing a class that is not labeled abstract and you believe that it overrides an abstract method in its superclass, you needn’t bother putting the Override annotation on that method. In a class that is not declared abstract, the compiler will emit an error message if you fail to override an abstract superclass method. However, you might wish to draw attention to all of the methods in your class that override superclass methods, in which case you should feel free to annotate these methods too. Most IDEs can be set to insert Override annotations automatically when you elect to override a method.

Most IDEs provide another reason to use the Override annotation consistently. If you enable the appropriate check, the IDE will generate a warning if you have a method that doesn’t have an Override annotation but does override a superclass method. If you use the Override annotation consistently, these warnings will alert you to unintentional overriding. They complement the compiler’s error messages, which alert you to unintentional failure to override. Between the IDE and the compiler, you can be sure that you’re overriding methods everywhere you want to and nowhere else.

The Override annotation may be used on method declarations that override declarations from interfaces as well as classes. With the advent of default methods, it is good practice to use Override on concrete implementations of interface methods to ensure that the signature is correct. If you know that an interface does not have default methods, you may choose to omit Override annotations on concrete implementations of interface methods to reduce clutter.

In an abstract class or an interface, however, it is worth annotating all methods that you believe to override superclass or superinterface methods, whether concrete or abstract. For example, the Set interface adds no new methods to the Collection interface, so it should include Override annotations on all of its method declarations to ensure that it does not accidentally add any new methods to the Collection interface.

In summary, the compiler can protect you from a great many errors if you use the Override annotation on every method declaration that you believe to override a supertype declaration, with one exception. In concrete classes, you need not annotate methods that you believe to override abstract method declarations (though it is not harmful to do so).
