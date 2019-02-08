## Chapter 8. Methods（方法）

### Item 55: Return optionals judiciously

Prior to Java 8, there were two approaches you could take when writing a method that was unable to return a value under certain circumstances. Either you could throw an exception, or you could return null (assuming the return type was an object reference type). Neither of these approaches is perfect. Exceptions should be reserved for exceptional conditions (Item 69), and throwing an exception is expensive because the entire stack trace is captured when an exception is created. Returning null doesn’t have these shortcomings, but it has its own. If a method returns null, clients must contain special-case code to deal with the possibility of a null return, unless the programmer can prove that a null return is impossible. If a client neglects to check for a null return and stores a null return value away in some data structure, a NullPointerException may result at some arbitrary time in the future, at some place in the code that has nothing to do with the problem.

In Java 8, there is a third approach to writing methods that may not be able to return a value. The Optional<T> class represents an immutable container that can hold either a single non-null T reference or nothing at all. An optional that contains nothing is said to be empty. A value is said to be present in an optional that is not empty. An optional is essentially an immutable collection that can hold at most one element. Optional<T> does not implement Collection<T>, but it could in principle.

A method that conceptually returns a T but may be unable to do so under certain circumstances can instead be declared to return an Optional<T>. This allows the method to return an empty result to indicate that it couldn’t return a valid result. An Optional-returning method is more flexible and easier to use than one that throws an exception, and it is less error-prone than one that returns null.

In Item 30, we showed this method to calculate the maximum value in a collection, according to its elements’ natural order.

```
// Returns maximum value in collection - throws exception if empty
public static <E extends Comparable<E>> E max(Collection<E> c) {
    if (c.isEmpty())
        throw new IllegalArgumentException("Empty collection");
    E result = null;
    for (E e : c)
        if (result == null || e.compareTo(result) > 0)
    result = Objects.requireNonNull(e);
    return result;
}
```

This method throws an IllegalArgumentException if the given collection is empty. We mentioned in Item 30 that a better alternative would be to return Optional<E>. Here’s how the method looks when it is modified to do so:

```
// Returns maximum value in collection as an Optional<E>
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
    if (c.isEmpty())
        return Optional.empty();
    E result = null;
    for (E e : c)
        if (result == null || e.compareTo(result) > 0)
    result = Objects.requireNonNull(e);
    return Optional.of(result);
}
```

As you can see, it is straightforward to return an optional. All you have to do is to create the optional with the appropriate static factory. In this program, we use two: Optional.empty() returns an empty optional, and Optional.of(value) returns an optional containing the given non-null value. It is a programming error to pass null to Optional.of(value). If you do this, the method responds by throwing a NullPointerException. The Optional.ofNullable(value) method accepts a possibly null value and returns an empty optional if null is passed in. **Never return a null value from an Optional-returning method:** it defeats the entire purpose of the facility.

Many terminal operations on streams return optionals. If we rewrite the max method to use a stream, Stream’s max operation does the work of generating an optional for us (though we do have to pass in an explicit comparator):

```
// Returns max val in collection as Optional<E> - uses stream
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
    return c.stream().max(Comparator.naturalOrder());
}
```

So how do you choose to return an optional instead of returning a null or throwing an exception? Optionals are similar in spirit to checked exceptions (Item 71), in that they force the user of an API to confront the fact that there may be no value returned. Throwing an unchecked exception or returning a null allows the user to ignore this eventuality, with potentially dire consequences. However, throwing a checked exception requires additional boilerplate code in the client.

If a method returns an optional, the client gets to choose what action to take if the method can’t return a value. You can specify a default value:

```
// Using an optional to provide a chosen default value
String lastWordInLexicon = max(words).orElse("No words...");
```

or you can throw any exception that is appropriate. Note that we pass in an exception factory rather than an actual exception. This avoids the expense of creating the exception unless it will actually be thrown:

```
// Using an optional to throw a chosen exception
Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);
```

If you can prove that an optional is nonempty, you can get the value from the optional without specifying an action to take if the optional is empty, but if you’re wrong, your code will throw a NoSuchElementException:

```
// Using optional when you know there’s a return value
Element lastNobleGas = max(Elements.NOBLE_GASES).get();
```

Occasionally you may be faced with a situation where it’s expensive to get the default value, and you want to avoid that cost unless it’s necessary. For these situations, Optional provides a method that takes a Supplier<T> and invokes it only when necessary. This method is called orElseGet, but perhaps it should have been called orElseCompute because it is closely related to the three Map methods whose names begin with compute. There are several Optional methods for dealing with more specialized use cases: filter, map, flatMap, and ifPresent. In Java 9, two more of these methods were added: or and ifPresentOrElse. If the basic methods described above aren’t a good match for your use case, look at the documentation for these more advanced methods and see if they do the job.

In case none of these methods meets your needs, Optional provides the isPresent() method, which may be viewed as a safety valve. It returns true if the optional contains a value, false if it’s empty. You can use this method to perform any processing you like on an optional result, but make sure to use it wisely. Many uses of isPresent can profitably be replaced by one of the methods mentioned above. The resulting code will typically be shorter, clearer, and more idiomatic.

For example, consider this code snippet, which prints the process ID of the parent of a process, or N/A if the process has no parent. The snippet uses the ProcessHandle class, introduced in Java 9:

```
Optional<ProcessHandle> parentProcess = ph.parent();
System.out.println("Parent PID: " + (parentProcess.isPresent() ?
String.valueOf(parentProcess.get().pid()) : "N/A"));
```

The code snippet above can be replaced by this one, which uses Optional’s map function:

```
System.out.println("Parent PID: " + ph.parent().map(h -> String.valueOf(h.pid())).orElse("N/A"));
```

When programming with streams, it is not uncommon to find yourself with a Stream<Optional<T>> and to require a Stream<T> containing all the elements in the nonempty optionals in order to proceed. If you’re using Java 8, here’s how to bridge the gap:

```
streamOfOptionals.filter(Optional::isPresent).map(Optional::get)
```

In Java 9, Optional was outfitted with a stream() method. This method is an adapter that turns an Optional into a Stream containing an element if one is present in the optional, or none if it is empty. In conjunction with Stream’s flatMap method (Item 45), this method provides a concise replacement for the code snippet above:

```
streamOfOptionals..flatMap(Optional::stream)
```

Not all return types benefit from the optional treatment. **Container types, including collections, maps, streams, arrays, and optionals should not be wrapped in optionals.** Rather than returning an empty Optional<List<T>>, you should simply return an empty List<T> (Item 54). Returning the empty container will eliminate the need for client code to process an optional. The ProcessHandle class does have the arguments method, which returns Optional<String[]>, but this method should be regarded as an anomaly that is not to be emulated.

So when should you declare a method to return Optional<T> rather than T? As a rule, **you should declare a method to return Optional<T> if it might not be able to return a result and clients will have to perform special processing if no result is returned.** That said, returning an Optional<T> is not without cost. An Optional is an object that has to be allocated and initialized, and reading the value out of the optional requires an extra indirection. This makes optionals inappropriate for use in some performance-critical situations. Whether a particular method falls into this category can only be determined by careful measurement (Item 67).

Returning an optional that contains a boxed primitive type is prohibitively expensive compared to returning a primitive type because the optional has two levels of boxing instead of zero. Therefore, the library designers saw fit to provide analogues of Optional<T> for the primitive types int, long, and double. These optional types are OptionalInt, OptionalLong, and OptionalDouble. They contain most, but not all, of the methods on Optional<T>. Therefore, **you should never return an optional of a boxed primitive type,** with the possible exception of the “minor primitive types,” Boolean, Byte, Character, Short, and Float.

Thus far, we have discussed returning optionals and processing them after they are returned. We have not discussed other possible uses, and that is because most other uses of optionals are suspect. For example, you should never use optionals as map values. If you do, you have two ways of expressing a key’s logical absence from the map: either the key can be absent from the map, or it can be present and map to an empty optional. This represents needless complexity with great potential for confusion and errors. More generally, **it is almost never appropriate to use an optional as a key, value, or element in a collection or array.** 

This leaves a big question unanswered. Is it ever appropriate to store an optional in an instance field? Often it’s a “bad smell”: it suggests that perhaps you should have a subclass containing the optional fields. But sometimes it may be justified. Consider the case of our NutritionFacts class in Item 2. A NutritionFacts instance contains many fields that are not required. You can’t have a subclass for every possible combination of these fields. Also, the fields have primitive types, which make it awkward to express absence directly. The best API for NutritionFacts would return an optional from the getter for each optional field, so it makes good sense to simply store those optionals as fields in the object.

In summary, if you find yourself writing a method that can’t always return a value and you believe it is important that users of the method consider this possibility every time they call it, then you should probably return an optional. You should, however, be aware that there are real performance consequences associated with returning optionals; for performance-critical methods, it may be better to return a null or throw an exception. Finally, you should rarely use an optional in any other capacity than as a return value.
