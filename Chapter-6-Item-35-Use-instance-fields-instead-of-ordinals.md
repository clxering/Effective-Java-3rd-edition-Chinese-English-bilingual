## Chapter 6. Enums and Annotations（枚举和注解）

### Item 35: Use instance fields instead of ordinals

Many enums are naturally associated with a single int value. All enums have an ordinal method, which returns the numerical position of each enum constant in its type. You may be tempted to derive an associated int value from the ordinal:

```
// Abuse of ordinal to derive an associated value - DON'T DO THIS
public enum Ensemble {
    SOLO, DUET, TRIO, QUARTET, QUINTET,SEXTET, SEPTET, OCTET, NONET, DECTET;

    public int numberOfMusicians() { return ordinal() + 1; }
}
```

While this enum works, it is a maintenance nightmare. If the constants are reordered, the numberOfMusicians method will break. If you want to add a second enum constant associated with an int value that you’ve already used, you’re out of luck. For example, it might be nice to add a constant for double quartet, which, like an octet, consists of eight musicians, but there is no way to do it.

Also, you can’t add a constant for an int value without adding constants for all intervening int values. For example, suppose you want to add a constant representing a triple quartet, which consists of twelve musicians. There is no standard term for an ensemble consisting of eleven musicians, so you are forced to add a dummy constant for the unused int value (11). At best, this is ugly. If many int values are unused, it’s impractical. Luckily, there is a simple solution to these problems. **Never derive a value associated with an enum from its ordinal; store it in an instance field instead:**

```
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),NONET(9), DECTET(10),TRIPLE_QUARTET(12);

    private final int numberOfMusicians;

    Ensemble(int size) { this.numberOfMusicians = size; }

    public int numberOfMusicians() { return numberOfMusicians; }
}
```

The Enum specification has this to say about ordinal: “Most programmers will have no use for this method. It is designed for use by general-purpose enumbased data structures such as EnumSet and EnumMap.” Unless you are writing code with this character, you are best off avoiding the ordinal method entirely.
