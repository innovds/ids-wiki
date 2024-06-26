# String aggregation
In Java, strings are **unalterable**. This means that any concatenation operation done with the `+` or `+=` operator results in the creation of a new `String` instance. No worries when it comes to doing the operation a small number of times but when the number of concatenations increases, the performance degradation is enormous.

## The classe `StringBuilder`
Take the example of the following table :

        String[] stringArray = new String[number];

        for (int i = 0; i < number; i++) {
            stringArray[i] = "The value of the number is " + i + ".\n";
        }
We want to concatenate all the values in the array. The following code works but should be **avoided**:

        for (int i = 0; i < number; i++) {
            result += stringArray[i]; // Not correct
        }
In order to concatenate character strings, you must use the `StringBuilder` class (or `StringBuffer` if there is a risk of concurrent access, which is quite rare). So our code becomes:

        for (int i = 0; i < number; i++) {
            builder.append(stringArray[i]); // Correct
        }

Once the concatenations are complete, a call to the `toString()` method allows you to retrieve the result from the `StringBuilder`

## Small performance test

I did some tests to measure the performance gap that we get from the previous code. The values are not very precise and they obviously vary according to the machine and its load at the time of the test, but they give an idea of the difference that exists between the two ways of doing things:

| Number of loop occurrences | Time for `+` in **µs** | Time for `append` in **µs** | Comment|
|--|--|--|--|
| 150 | 997 | 0 | With a time less than a microsecond, the times of the `+` method are random but remain between 0 and 1ms whereas for `append` we always remain at 0|
| 1000 | 23,935 | 0 | The `+` time has clearly increased while for the `append` it remains immeasurable |
| 5000| 455,782 | 998.6 | The `append` time appears randomly, as was the case for the `+` with 150 occurrences. We can estimate the performance ratio between the two methods at least 450 at this time of processing |
| 10,000| 1,055,307| 997 | The `+` time continues to climb and reaches the second while the `append` remains around the millisecond |
| 20,000 | 3,033,925| 995 | 3 seconds for the `+` against less than a millisecond for the `append` |
| 30,000 | 6,092,791 | 2,993 | We are now at 6 seconds for the `+` while the `append` is at about 3 ms, a ratio of 2,000 |


## For further

- A `StringBuilder` is by default initialized with a size of 16 characters. When the concatenation requires a larger size, the object is automatically reallocated with a larger size. This is why it is useful to specify a size to the constructor in order to limit these reassignments. And it is better to plan a little too big (without exaggerating) than a little too small.
- The `StringBuilder` allows other operations on a string like
   - delete part of a string: `delete(int start, int end)`
   - delete a character: `deleteCharAt(int index)`
   - insert a String or a primitive type at a given position: `insert(int offset, String str)`, `insert(int offset, long l)`...
   - replace: `replace​(int start, int end, String str)`
- Do not concatenate strings using `+` inside an `append`.


## Other aggregations
- When it comes to concatenating the elements of an array with a separator, there are other ways to proceed, in particular `String result = String.join(separator, stringArray);`. But this way remains noticeably less efficient than the `StringBuilder` (about 5 times slower).
- The `String.format` method can be used to format messages with arguments:
   - the result of `String.format("The code of this object is '%s'", code)` is "The code of this object is 'object.code'"
   - the result of `String.format("The price of this object is %d €", 12500)` is "The price of this object is 12500 €"
- `+` This operateur can perfectly be usued in some cases, for example where there is a finite number of aggregations to do. `String result = myString + otherString + anotherString + lastString` is correct and quick as the compiler transforms this code to 
``` java 
String result = new StringBuilder(myString).append(otherString).append(anotherString).append(lastString).toString()
``` 
- `concat` method of `String` object. It's possible to use this method to concatenate 2 (and only 2) `String` : `String result = myString.concat(otherString);`. But **it must never be used to concat more than 2** `String` because it will instanciate a new `String` for each `concat`

# string.contains() / string.startWith() / string.endWith()


#Autoboxing/unboxing

Autoboxing consists of transforming a primitive type into an equivalent wrapper object and inboxing does the opposite. Autoboxing transforms for example an `int` into `Integer`, a `char` into `Character`, a `boolean` into `Boolean` or a `long` into `Long`. For more explanation, see [here](https://docs.oracle.com/javase/tutorial/java/data/autoboxing.html).
It is nevertheless worth remaining careful because there is a fundamental difference between primitive types and objects: objects can be null while a primitive type must have a value. So the following code will not cause any compilation problems but will always throw a NullPointerException.

    Long longObj = null;
    long longPrimitif = longObj;
No such risk with unboxing, but keep in mind that this operation results in the instantiation of an object which, in turn, can have an impact on performance. For example, the following code works but is not relevant because it requires initializing a useless Integer.

    Integer varA = 7899;
    int varB = 225;
    int result = varB + varA;

Similarly, be careful when declaring the parameters of a method:

    public long sum(Long a, Long b) {
        return a + b;
    }
In the above method, there is a risk of `NullPointerException` since `a` and `b` are `Long` and can therefore be null, whereas there is no risk in the following method:

    public long sum(long a, long b) {
        return a + b;
    }

In the following example the method returns a `Boolean` but the value can never be null.

    public Boolean areEquals(int valueA, int valueB) {
        return valueA == valueB;
    }

This code works but it weighs down the use of the result as the null value must be checked. So, the returned type must be change to `boolean`

# Lists.newArrayList is not equal to List.of

## Explanation

### `Lists.newArrayList` 

The class `Lists` is available with package `com.google.common.collect` (never use the `Lists` class from AssertJ outside of test classes 😡). The static method `newArrayList` intanciates a new `ArrayList` with the specified values. This method is a short cut to create a new `ArrayList` and directly add one or more entries. 
For example, 

     List<String> myList = Lists.newArrayList("a", "b", "c");
can be used to replace

      List<String> myList = new ArrayList<>();
      myList.add("a");
      myList.add("b");
      myList.add("c");

### `List.of`
Here, `List` is the well known interface available with package `java.util`. It is implemented by `ArrayList`, `Vector` classes and some other ones. The static method `List.of` intanciates a `List`, but not an `ArrayList`. The returned instance type depends on the number of given parameters. But all of them have a common point : they extend `AbstractImmutableList`. As the returned list is immutable, it won't be possible to add, replace or remove any element.

**<u>Noticed:<u>**
`List.copyOf()` also returns an immutable list, like `Collections.emptyList()`.

## Basic use case

Sometime it's useful to define a `List` as constant, to check if a parameter is one of the expected values for example. In this case, the list could be defined like this :

    private static final List<String> EXPECTED_VALUES = Lists.newArrayList("An expected value", "Another expected value");

But, with this code there is something wrong : as `EXPECTED_VALUES` is an `ArrayList`, it is possible to add, replace or remove values. However, here we need an immutable list to be sure no changes will be made. There is an easy way to get what we need :

    private static final List<String> EXPECTED_VALUES = List.of("An expected value", "Another expected value");

# Java assert

Even if the Java `assert` instruction propose a simple way to validate an hypothesis, it must never be used as it is not activated on our server. So the `assert` instruction will have no effect.
