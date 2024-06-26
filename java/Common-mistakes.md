# Date convertion

The JDK provides the class `java.text.SimpleDateFormat` to convert `java.util.Date` from/to `java.lang.String`. **This class must not be used as it is slow and not thread safe**. Instead of, you must use the class `org.apache.commons.lang3.time.FastDateFormat` which offer the same methods, but it is quicker and thread safe. As this class is thread safe, only one instance of it must be created for a given date format.

# enum

## No setter !!

It is possible to specify a value to refine the behavior of its `enum`. In order to access this value, a getter is necessary but you should never define a setter for this variable. **An `enum` is unalterable** and must remain so

    public enum Status {

        DRAFT("status.draft"),
        IN_PROGESS("status.inProgress"),
        ACTIVE("status.active"),
        ARCHIVED("status.archived"),
    
        private String value;
    
        private Status(String value) {
            this.value = value;
        }

        public String getValue() {
            return this.value;
        }
        // Never define a setter for field value
    }

## Equality

As `enum` are constants, there is no need to use `equals` method to compare them. You must directly use `==` which is quicker.

    if (myObject.status == Status.IN_PROGRESS) {

# Log an Exception

The way to log an exception is really simple. There is no need to call getMessage() or other methods from the `Exception` caught:

```java
try {
   deleteObject(id);
} catch (Exception e) {
   log.error("An error occurred while trying to delete the object {}", id, e);
}
```
With this trace we gonna have:
- a precise information about what we were doing when the exception is thrown
- the exception message with a complete stack trace 

# Choose the right import

Be careful about the import you choose. During reviews, I saw a lot of wrong classes imported, especially on `Lists` class.

![image.png](/.attachments/image-420c0b46-13d2-460c-8eca-8ad79601e631.png)

In this case, the right import is the first one as assertj packages should only be used for unit tests.

