# String Aggregation

In Java, strings are **unalterable**. This means that any concatenation operation done with the `+` or `+=` operator results in the creation of a new `String` instance. This is fine for a small number of operations, but performance degrades significantly with many concatenations.

## The `StringBuilder` Class

Take the example of the following table:

```java
String[] stringArray = new String[number];

for (int i = 0; i < number; i++) {
    stringArray[i] = "The value of the number is " + i + ".\n";
}
```

We want to concatenate all the values in the array. The following code works but should be **avoided**:

```java
for (int i = 0; i < number; i++) {
    result += stringArray[i]; // Not correct
}
```

To concatenate strings, use the `StringBuilder` class (or `StringBuffer` if there's a risk of concurrent access, which is rare). So our code becomes:

```java
for (int i = 0; i < number; i++) {
    builder.append(stringArray[i]); // Correct
}
```

Once the concatenations are complete, a call to the `toString()` method retrieves the result from the `StringBuilder`.

## Small Performance Test

I did some tests to measure the performance gap between the two methods. The values vary by machine and load, but they give an idea of the difference:

| Number of loop occurrences | Time for `+` in **µs** | Time for `append` in **µs** | Comment |
|--|--|--|--|
| 150 | 997 | 0 | With a time less than a microsecond, the `+` method times are random but remain between 0 and 1 ms, while `append` always stays at 0 |
| 1000 | 23,935 | 0 | The `+` time clearly increases, while `append` remains immeasurable |
| 5000 | 455,782 | 998.6 | The `append` time appears randomly, similar to `+` with 150 occurrences. The performance ratio is about 450 |
| 10,000 | 1,055,307 | 997 | The `+` time continues to climb, reaching a second, while `append` remains around a millisecond |
| 20,000 | 3,033,925 | 995 | 3 seconds for `+` versus less than a millisecond for `append` |
| 30,000 | 6,092,791 | 2,993 | 6 seconds for `+` while `append` is about 3 ms, a ratio of 2,000 |

## For Further

- A `StringBuilder` is initialized with a size of 16 characters. When more space is needed, it is reallocated. It's useful to specify a size to limit reallocations.
- The `StringBuilder` allows other operations on a string like:
    - Delete part of a string: `delete(int start, int end)`
    - Delete a character: `deleteCharAt(int index)`
    - Insert a string or a primitive type at a given position: `insert(int offset, String str)`, `insert(int offset, long l)`...
    - Replace: `replace​(int start, int end, String str)`
- Do not concatenate strings using `+` inside an `append`.

## Other Aggregations

- For concatenating array elements with a separator: `String result = String.join(separator, stringArray);`. This is less efficient than `StringBuilder` (about 5 times slower).
- The `String.format` method formats messages with arguments:
    - `String.format("The code of this object is '%s'", code)` results in "The code of this object is 'object.code'"
    - `String.format("The price of this object is %d €", 12500)` results in "The price of this object is 12500 €"
- The `+` operator can be used when there are a finite number of aggregations: `String result = myString + otherString + anotherString + lastString`. The compiler transforms this to:

```java
String result = new StringBuilder(myString).append(otherString).append(anotherString).append(lastString).toString();
```

- The `concat` method of `String` can concatenate 2 `String` objects: `String result = myString.concat(otherString);`. It should not be used for more than 2 `String` objects as it creates a new `String` for each `concat`.

# `string.contains()`, `string.startWith()`, `string.endWith()`

# Autoboxing/Unboxing

Autoboxing transforms a primitive type into an equivalent wrapper object, and unboxing does the opposite. For more explanation, see [here](https://docs.oracle.com/javase/tutorial/java/data/autoboxing.html).

Be cautious because objects can be null while primitive types must have a value. The following code will throw a `NullPointerException`:

```java
Long longObj = null;
long longPrimitif = longObj;
```

There is no risk with unboxing, but it can impact performance due to object instantiation:

```java
Integer varA = 7899;
int varB = 225;
int result = varB + varA;
```

Be careful when declaring method parameters:

```java
public long sum(Long a, Long b) {
    return a + b;
}
```

This method can throw a `NullPointerException` since `a` and `b` can be null. The following method does not have this risk:

```java
public long sum(long a, long b) {
    return a + b;
}
```

The following method returns a `Boolean`, but the value can never be null:

```java
public Boolean areEquals(int valueA, int valueB) {
    return valueA == valueB;
}
```

Change the return type to `boolean` to avoid null checks.

# `Lists.newArrayList` is not equal to `List.of`

## Explanation

### `Lists.newArrayList`

The class `Lists` is available in the package `com.google.common.collect`. The static method `newArrayList` instantiates a new `ArrayList` with specified values:

```java
List<String> myList = Lists.newArrayList("a", "b", "c");
```

This can replace:

```java
List<String> myList = new ArrayList<>();
myList.add("a");
myList.add("b");
myList.add("c");
```

### `List.of`

The `List` interface is available in the package `java.util`. The static method `List.of` instantiates a `List`, but not an `ArrayList`. The returned instance type depends on the number of parameters and extends `AbstractImmutableList`. The list is immutable.

**Note:**
`List.copyOf()` also returns an immutable list, like `Collections.emptyList()`.

## Basic Use Case

Sometimes it's useful to define a `List` as constant, for example, to check if a parameter is one of the expected values:

```java
private static final List<String> EXPECTED_VALUES = Lists.newArrayList("An expected value", "Another expected value");
```

This is incorrect as `EXPECTED_VALUES` is an `ArrayList` and can be modified. Use `List.of` to create an immutable list:

```java
private static final List<String> EXPECTED_VALUES = List.of("An expected value", "Another expected value");
```

# Java Assert

Even though the Java `assert` statement offers a simple way to validate assumptions, it should not be used as it is not activated on our server, rendering it ineffective.
