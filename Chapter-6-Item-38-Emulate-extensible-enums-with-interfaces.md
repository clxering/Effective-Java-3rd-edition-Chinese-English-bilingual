## Chapter 6. Enums and Annotations（枚举和注释）

### Item 38: Emulate extensible enums with interfaces

In almost all respects, enum types are superior to the typesafe enum pattern described in the first edition of this book [Bloch01]. On the face of it, one exception concerns extensibility, which was possible under the original pattern but is not supported by the language construct. In other words, using the pattern, it was possible to have one enumerated type extend another; using the language feature, it is not. This is no accident. For the most part, extensibility of enums turns out to be a bad idea. It is confusing that elements of an extension type are instances of the base type and not vice versa. There is no good way to enumerate over all of the elements of a base type and its extensions. Finally, extensibility would complicate many aspects of the design and implementation.

That said, there is at least one compelling use case for extensible enumerated types, which is operation codes, also known as opcodes. An opcode is an enumerated type whose elements represent operations on some machine, such as the Operation type in Item 34, which represents the functions on a simple calculator. Sometimes it is desirable to let the users of an API provide their own operations, effectively extending the set of operations provided by the API.

Luckily, there is a nice way to achieve this effect using enum types. The basic idea is to take advantage of the fact that enum types can implement arbitrary interfaces by defining an interface for the opcode type and an enum that is the standard implementation of the interface. For example, here is an extensible version of the Operation type from Item 34:

```
// Emulated extensible enum using an interface
public interface Operation {
    double apply(double x, double y);
}

public enum BasicOperation implements Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };
    
    private final String symbol;
    
    BasicOperation(String symbol) {
        this.symbol = symbol;
    }
    
    @Override public String toString() {
        return symbol;
    }
}
```

While the enum type (BasicOperation) is not extensible, the interface type (Operation) is, and it is the interface type that is used to represent operations in APIs. You can define another enum type that implements this interface and use instances of this new type in place of the base type. For example, suppose you want to define an extension to the operation type shown earlier, consisting of the exponentiation and remainder operations. All you have to do is write an enum type that implements the Operation interface:

```
// Emulated extension enum
public enum ExtendedOperation implements Operation {
    EXP("^") {
        public double apply(double x, double y) {
            return Math.pow(x, y);
        } 
    },
    REMAINDER("%") {
        public double apply(double x, double y) {
            return x % y;
    } 
};

    private final String symbol;
    
    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }
    
    @Override public String toString() {
        return symbol;
    }
}
```

You can now use your new operations anywhere you could use the basic operations, provided that APIs are written to take the interface type (Operation), not the implementation (BasicOperation). Note that you don’t have to declare the abstract apply method in the enum as you do in a nonextensible enum with instance-specific method implementations (page 162). This is because the abstract method (apply) is a member of the interface (Operation).

Not only is it possible to pass a single instance of an “extension enum” anywhere a “base enum” is expected, but it is possible to pass in an entire extension enum type and use its elements in addition to or instead of those of the base type. For example, here is a version of the test program on page 163 that exercises all of the extended operations defined previously:

```
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(ExtendedOperation.class, x, y);
}

private static <T extends Enum<T> & Operation> void test(Class<T> opEnumType, double x, double y) {
        for (Operation op : opEnumType.getEnumConstants())
            System.out.printf("%f %s %f = %f%n",x, op, y, op.apply(x, y));
}
```

Note that the class literal for the extended operation type (ExtendedOperation.class) is passed from main to test to describe the set of extended operations. The class literal serves as a bounded type token (Item 33). The admittedly complex declaration for the opEnumType parameter (<T extends Enum<T> & Operation> Class<T>) ensures that the Class object represents both an enum and a subtype of Operation, which is exactly what is required to iterate over the elements and perform the operation associated with each one.

A second alternative is to pass a Collection<? extends Operation>, which is a bounded wildcard type (Item 31), instead of passing a class object:

```
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(Arrays.asList(ExtendedOperation.values()), x, y);
}

private static void test(Collection<? extends Operation> opSet,double x, double y) {
    for (Operation op : opSet)
        System.out.printf("%f %s %f = %f%n",x, op, y, op.apply(x, y));
}
```

The resulting code is a bit less complex, and the test method is a bit more flexible: it allows the caller to combine operations from multiple implementation types. On the other hand, you forgo the ability to use EnumSet (Item 36) and EnumMap (Item 37) on the specified operations.

Both programs shown previously will produce this output when run with command line arguments 4 and 2:

```
4.000000 ^ 2.000000 = 16.000000
4.000000 % 2.000000 = 0.000000
```

A minor disadvantage of the use of interfaces to emulate extensible enums is that implementations cannot be inherited from one enum type to another. If the implementation code does not rely on any state, it can be placed in the interface, using default implementations (Item 20). In the case of our Operation example, the logic to store and retrieve the symbol associated with an operation must be duplicated in BasicOperation and ExtendedOperation. In this case it doesn’t matter because very little code is duplicated. If there were a larger amount of shared functionality, you could encapsulate it in a helper class or a static helper method to eliminate the code duplication.

The pattern described in this item is used in the Java libraries. For example, the java.nio.file.LinkOption enum type implements the CopyOption and OpenOption interfaces.

In summary, **while you cannot write an extensible enum type, you can emulate it by writing an interface to accompany a basic enum type that implements the interface.** This allows clients to write their own enums (or other types) that implement the interface. Instances of these types can then be used wherever instances of the basic enum type can be used, assuming APIs are written in terms of the interface.

