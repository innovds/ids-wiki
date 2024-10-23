# 1 - Good Practices

## Use JUnit 5

<u>Version 4 of JUnit must not be used; only version 5 is allowed.</u> <b>This means the following code</b>

```java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.junit.MockitoJUnitRunner;

@RunWith(MockitoJUnitRunner.class)
public class MyObjectTest {

    @Test
    public void myMethod_should_return_ok() {
        // Write the test here
    }
}
```

<b>must be replaced by this</b>:

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.junit.jupiter.MockitoExtension;

@ExtendWith(MockitoExtension.class)
class MyObjectTest {

    @Test
    void myMethod_should_return_ok() {
        // Write the test here
    }
}
```

Notable differences:
- Imports are updated (including `@Test` annotation).
- `@RunWith` is replaced by `@ExtendWith`.
- The `public` access modifier is removed from the class and test methods.

## Identifiers
- When using multiple object types in the same test (e.g., `TechnicalDocument`, `TechnicalDocumentDescription`, `Experience`), use distinct values for IDs. This reduces potential errors.
- Use constants for IDs or codes if they appear in multiple places to improve readability, scalability, and avoid typos.

## Test Method Naming

For clarity, the test method name should begin with the name of the method under test, followed by an explanation of what the test expects. Examples:
- Testing the `computeValue` method expecting a response of 18:  
`void computeValue_should_return_18()`
- Testing `computeValue` expecting an `IllegalArgumentException`:  
`void computeValue_should_throw_IllegalArgumentException()`

## Structuring Tests

Tests should follow the "Given-When-Then" structure:
- **Given**: Sets up conditions and mocks.
- **When**: Calls the method under test.
- **Then**: Verifies the expected results with assertions.

Every test should include these parts, even if one is empty. **The "When" part must never be empty** and, in most cases, **neither should the "Then" part**.

Example:

```java
private static final Integer OBJECT_ID = 8;

@Test
void getOne_should_return_computed_object() {
    // Given
    doReturn(getExpectedObject(OBJECT_ID)).when(myClassRepository).getOne(OBJECT_ID);

    // When
    MyClass result = objectToTest.getOne(OBJECT_ID);

    // Then
    assertThat(result).isSameAs(getExpectedObject(OBJECT_ID));
}
```

## Using AssertJ

### Introduction
JUnit natively provides assertions via `org.junit.jupiter.api.Assertions`:

```java
assertEquals(expectedValue, actualValue);
```

AssertJ offers more flexible assertions via `org.assertj.core.api.Assertions`:

```java
assertThat(actualValue).isEqualTo(expectedValue);
```

AssertJ allows more detailed and readable tests compared to basic JUnit assertions.

### Exception Assertions

To verify exceptions, use `assertThatExceptionOfType`. This enables further checks on the message and cause:

```java
assertThatExceptionOfType(ObjectNotFoundException.class).isThrownBy(() -> myMethod())
    .withMessage("Expected exception message");
```

AssertJ also provides specific assertions for common exceptions:

```java
assertThatIllegalStateException()
assertThatIllegalArgumentException()
assertThatIOException()
assertThatNullPointerException()
```

### Jupiter vs AssertJ Assertion Comparison

| Assertion | JUnit (Jupiter) | AssertJ | Comment |
|-----------|-----------------|---------|---------|
| Object not null | `assertNotNull(obj);` | `assertThat(obj).isNotNull();` | Both behave identically |
| Equal objects | `assertEquals(expected, actual);` | `assertThat(actual).isEqualTo(expected);` | Both are similar |
| Exception thrown | `assertThrows(NullPointerException.class, () -> myMethod());` | `assertThatExceptionOfType(IllegalArgumentException.class).isThrownBy(() -> myMethod())` | Differ in approach |
| Instance of a class | `assertTrue(obj instanceof ExpectedClass);` | `assertThat(obj).isInstanceOf(ExpectedClass.class);` | AssertJ provides clearer error messages |
| Compare fields | - | `assertThat(objA).usingRecursiveComparison().isEqualTo(objB)` | Allows recursive comparison without needing `equals` implementation. |

### Complete Example

Testing a `List` object with AssertJ:

```java
assertThat(myList).hasSize(5)
    .contains(expectedObject, anotherExpectedObject);
```

Key advantages:
- No explicit null checks are needed.
- Chaining methods simplifies reading.
- Error messages are more detailed.

### Best Practice

For consistency, readability, and precise error messages, **use AssertJ** for all assertions.

## Reducing Redundant Tests

Redundant checks, like `isNotNull` when using `isNotEmpty`, should be avoided:

```java
assertThat(myList).isNotEmpty(); // No need for assertThat(myList).isNotNull();
```

## Using `doXxx().when()` over `when().thenXxx()`

For consistency and flexibility, always use `doXxx().when()`. Unlike `when().thenXxx()`, it works for methods that throw exceptions or are `void`.

### Mocking Static Methods

When mocking static methods, initialize and close the mock for each test:

```java
private MockedStatic<SecurityUtils> securityMock;

@BeforeEach
public void init() {
    securityMock = mockStatic(SecurityUtils.class);
}

@AfterEach
public void close() {
    securityMock.close();
}
```

## Skipping Tests

Use `@Disabled` instead of commenting out test annotations, and include the reason:

```java
@Disabled("Unexpected NullPointerException")
@Test
void should_not_fail() {
    // test code
}
```

## Mockito `any()` Usage Guidelines

Be cautious with `any()`, `anyXxx()`, and `any(Class)`. Always use precise values where possible to avoid biased tests. Use `any()` only when:
- The mock behavior cannot determine specific values.
- Mocking or verifying behavior for all object values is required.

## Object Comparisons

- `isSameAs`: Checks if two objects are the same instance.
- `isEqualTo`: Verifies equality based on `equals()` method.
- `usingRecursiveComparison`: Compares fields of two objects, even if of different types.

## Example: Avoiding Biased Tests

Avoid:

```java
doReturn(getExpectedObject()).when(myClassRepository).save(any(MyClass.class));
```

Use:

```java
doReturn(getExpectedObject(10)).when(myClassRepository).save(getExpectedObject(null));
```

This ensures precise behavior, leading to reliable tests.

## Verifying Mock Behavior

To test interactions with mocks, use `verify`. Be specific about arguments when possible:

Incorrect:

```java
verify(sender).sendMessage(any(MessageObj.class));
```

Correct:

```java
verify(sender).sendMessage(getValidMessage(ID_A));
verify(sender, never()).sendMessage(getValidMessage(ID_B));
```

If no method call is expected, use:

```java
verify(sender, never()).sendMessage(any(MessageObj.class));
```

## Use `@Nested` Classes for Logical Grouping

Use `@Nested` classes to group related test cases, improving readability and organization.

Example:

```java
class OrderServiceTest {

    @Nested
    class CreateOrderTests {

        @Test
        void shouldCreateOrderSuccessfully() {
            // test logic
        }

        @Test
        void shouldFailIfCustomerNotFound() {
            // test logic
        }
    }

    @Nested
    class CancelOrderTests {

        @Test
        void shouldCancelOrderSuccessfully() {
            // test logic
        }
    }
}
```

## Consider `@ParameterizedTest` for Multiple Inputs

To run the same logic with different inputs, use `@ParameterizedTest`.

Example:

```java
@ParameterizedTest
@ValueSource(strings = {"", "  ", "\t", "\n"})
void isBlankShouldReturnTrueForBlankStrings(String input) {
    assertTrue(StringUtils.isBlank(input));
}
```

Or for more complex inputs:

```java
@ParameterizedTest
@CsvSource({
    "apple, 1",
    "banana, 2"
})
void fruitShouldHaveExpectedOrder(String fruit, int order) {
    assertEquals(order, getFruitOrder(fruit));
}
```

## Leveraging `@BeforeAll`, `@AfterEach`, and Other JUnit 5 Features

- Use `@BeforeAll` to initialize resources once for the entire test class:

```java
@BeforeAll
static void setUp() {
    // Initialize shared resources
}
```

- Use `@AfterEach` to clean up resources after each test:

```java
@AfterEach
void tearDown() {
    // Clean up resources or reset states
}
```
