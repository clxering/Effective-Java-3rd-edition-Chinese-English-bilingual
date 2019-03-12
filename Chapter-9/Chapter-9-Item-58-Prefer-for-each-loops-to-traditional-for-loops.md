## Chapter 9. General Programming（通用程序设计）

### Item 58: Prefer for-each loops to traditional for loops（for-each 循环优于传统的 for 循环）

As discussed in Item 45, some tasks are best accomplished with streams, others with iteration. Here is a traditional for loop to iterate over a collection:

```
// Not the best way to iterate over a collection!
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
    Element e = i.next();
    ... // Do something with e
}
```

and here is a traditional for loop to iterate over an array:

```
// Not the best way to iterate over an array!
for (int i = 0; i < a.length; i++) {
    ... // Do something with a[i]
}
```

These idioms are better than while loops (Item 57), but they aren’t perfect. The iterator and the index variables are both just clutter—all you need are the elements. Furthermore, they represent opportunities for error. The iterator occurs three times in each loop and the index variable four, which gives you many chances to use the wrong variable. If you do, there is no guarantee that the compiler will catch the problem. Finally, the two loops are quite different, drawing unnecessary attention to the type of the container and adding a (minor) hassle to changing that type.

The for-each loop (officially known as the “enhanced for statement”) solves all of these problems. It gets rid of the clutter and the opportunity for error by hiding the iterator or index variable. The resulting idiom applies equally to collections and arrays, easing the process of switching the implementation type of a container from one to the other:

```
// The preferred idiom for iterating over collections and arrays
for (Element e : elements) {
    ... // Do something with e
}
```

When you see the colon (:), read it as “in.” Thus, the loop above reads as “for each element e in elements.” There is no performance penalty for using for-each loops, even for arrays: the code they generate is essentially identical to the code you would write by hand.

The advantages of the for-each loop over the traditional for loop are even greater when it comes to nested iteration. Here is a common mistake that people make when doing nested iteration:

```
// Can you spot the bug?
enum Suit { CLUB, DIAMOND, HEART, SPADE }
enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT,NINE, TEN, JACK, QUEEN, KING }
...
static Collection<Suit> suits = Arrays.asList(Suit.values());
static Collection<Rank> ranks = Arrays.asList(Rank.values());
List<Card> deck = new ArrayList<>();
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
deck.add(new Card(i.next(), j.next()));
```

Don’t feel bad if you didn’t spot the bug. Many expert programmers have made this mistake at one time or another. The problem is that the next method is called too many times on the iterator for the outer collection (suits). It should be called from the outer loop so that it is called once per suit, but instead it is called from the inner loop, so it is called once per card. After you run out of suits, the loop throws a NoSuchElementException.

If you’re really unlucky and the size of the outer collection is a multiple of the size of the inner collection—perhaps because they’re the same collection—the loop will terminate normally, but it won’t do what you want. For example, consider this ill-conceived attempt to print all the possible rolls of a pair of dice:

```
// Same bug, different symptom!
enum Face { ONE, TWO, THREE, FOUR, FIVE, SIX }
...
Collection<Face> faces = EnumSet.allOf(Face.class);
for (Iterator<Face> i = faces.iterator(); i.hasNext(); )
for (Iterator<Face> j = faces.iterator(); j.hasNext(); )
System.out.println(i.next() + " " + j.next());
```

The program doesn’t throw an exception, but it prints only the six “doubles” (from “ONE ONE” to “SIX SIX”), instead of the expected thirty-six combinations.

To fix the bugs in these examples, you must add a variable in the scope of the outer loop to hold the outer element:

```
// Fixed, but ugly - you can do better!
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); ) {
    Suit suit = i.next();
    for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
        deck.add(new Card(suit, j.next()));
}
```

If instead you use a nested for-each loop, the problem simply disappears. The resulting code is as succinct as you could wish for:

```
// Preferred idiom for nested iteration on collections and arrays
for (Suit suit : suits)
for (Rank rank : ranks)
deck.add(new Card(suit, rank));
```

Unfortunately, there are three common situations where you can’t use foreach:

- **Destructive filtering** —If you need to traverse a collection removing selected elements, then you need to use an explicit iterator so that you can call its remove method. You can often avoid explicit traversal by using Collection’s removeIf method, added in Java 8.

- **Transforming** —If you need to traverse a list or array and replace some or all of the values of its elements, then you need the list iterator or array index in order to replace the value of an element.

- **Parallel iteration** —If you need to traverse multiple collections in parallel, then you need explicit control over the iterator or index variable so that all iterators or index variables can be advanced in lockstep (as demonstrated unintentionally in the buggy card and dice examples above). If you find yourself in any of these situations, use an ordinary for loop and be wary of the traps mentioned in this item.

Not only does the for-each loop let you iterate over collections and arrays, it lets you iterate over any object that implements the Iterable interface, which consists of a single method. Here is how the interface looks:

```
public interface Iterable<E> {
// Returns an iterator over the elements in this iterable
Iterator<E> iterator();
}
```

It is a bit tricky to implement Iterable if you have to write your own Iterator implementation from scratch, but if you are writing a type that represents a group of elements, you should strongly consider having it implement Iterable, even if you choose not to have it implement Collection. This will allow your users to iterate over your type using the foreach loop, and they will be forever grateful.

In summary, the for-each loop provides compelling advantages over the traditional for loop in clarity, flexibility, and bug prevention, with no performance penalty. Use for-each loops in preference to for loops wherever you can.

---
**[Back to contents of the chapter（返回章节目录）](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-9/Chapter-9-Introduction.md)**
- **Previous Item（上一条目）：[Item 57: Minimize the scope of local variables（将局部变量的作用域最小化）](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-9/Chapter-9-Item-57-Minimize-the-scope-of-local-variables.md)**
- **Next Item（下一条目）：[Item 59: Know and use the libraries（了解并使用库）](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-9/Chapter-9-Item-59-Know-and-use-the-libraries.md)**
