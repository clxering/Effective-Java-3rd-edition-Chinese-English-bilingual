## Chapter 6. Enums and Annotations（枚举和注释）

### Item 36: Use EnumSet instead of bit fields

If the elements of an enumerated type are used primarily in sets, it is traditional to use the int enum pattern (Item 34), assigning a different power of 2 to each constant:

```
// Bit field enumeration constants - OBSOLETE!
public class Text {
    public static final int STYLE_BOLD = 1 << 0; // 1
    public static final int STYLE_ITALIC = 1 << 1; // 2
    public static final int STYLE_UNDERLINE = 1 << 2; // 4
    public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8
    // Parameter is bitwise OR of zero or more STYLE_ constants
    public void applyStyles(int styles) { ... }
}
```

This representation lets you use the bitwise OR operation to combine several constants into a set, known as a bit field:

```
text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```

The bit field representation also lets you perform set operations such as union and intersection efficiently using bitwise arithmetic. But bit fields have all the disadvantages of int enum constants and more. It is even harder to interpret a bit field than a simple int enum constant when it is printed as a number. There is no easy way to iterate over all of the elements represented by a bit field. Finally, you have to predict the maximum number of bits you’ll ever need at the time you’re writing the API and choose a type for the bit field (typically int or long) accordingly. Once you’ve picked a type, you can’t exceed its width (32 or 64 bits) without changing the API.

Some programmers who use enums in preference to int constants still cling to the use of bit fields when they need to pass around sets of constants. There is no reason to do this, because a better alternative exists. The java.util package provides the EnumSet class to efficiently represent sets of values drawn from a single enum type. This class implements the Set interface, providing all of the richness, type safety, and interoperability you get with any other Set implementation. But internally, each EnumSet is represented as a bit vector. If the underlying enum type has sixty-four or fewer elements—and most do—the entire EnumSet is represented with a single long, so its performance is comparable to that of a bit field. Bulk operations, such as removeAll and retainAll, are implemented using bitwise arithmetic, just as you’d do manually for bit fields. But you are insulated from the ugliness and errorproneness of manual bit twiddling: the EnumSet does the hard work for you.

Here is how the previous example looks when modified to use enums and enum sets instead of bit fields. It is shorter, clearer, and safer:

```
// EnumSet - a modern replacement for bit fields
public class Text {
    public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }
    // Any Set could be passed in, but EnumSet is clearly best
    public void applyStyles(Set<Style> styles) { ... }
}
```

Here is client code that passes an EnumSet instance to the applyStyles method. The EnumSet class provides a rich set of static factories for easy set creation, one of which is illustrated in this code:

```
text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```

Note that the applyStyles method takes a Set<Style> rather than an EnumSet<Style>. While it seems likely that all clients would pass an EnumSet to the method, it is generally good practice to accept the interface type rather than the implementation type (Item 64). This allows for the possibility of an unusual client to pass in some other Set implementation.

In summary, **just because an enumerated type will be used in sets, there is no reason to represent it with bit fields.** The EnumSet class combines the conciseness and performance of bit fields with all the many advantages of enum types described in Item 34. The one real disadvantage of EnumSet is that it is not, as of Java 9, possible to create an immutable EnumSet, but this will likely be remedied in an upcoming release. In the meantime, you can wrap an EnumSet with Collections.unmodifiableSet, but conciseness and performance will suffer.

