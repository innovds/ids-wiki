# Naming rules

## Package names

Package names are always written with lowercase letters. Each package level is separated by a dot. A `_` can be used for cases where several words make up a level.
Examples:
- `com.ids.contact.dto`
- `com.ids.remind_hub.entity`

## Class names

The first character of a class name must be a capital letter. Then it is written in lowercase with capitals to mark the beginning of a new word. Examples : 
- `public class ContactType`
- `public Class Contact`

## Method names

Method names should be written in lowercase with capitals to mark the beginning of a new word. Method names usually start with a verb and must explain what the method does.
Examples :
- `public void computeValue()`
- `protected void fillTarget(Object target, Object source)`

That means a method called 

When the method returns a `boolean`, the method name should be a short phrase that can be answered with yes or no (true or false). That means `check` is not a valid name for a method that return a `boolean`.
Here, we can see some examples of correct names:
- `public boolean hasSize(int value)`
- `public boolean contains(Object object)`
- `public static boolean isOrganization(Contact contact)`


This rule is also valid for accessors:

```java
    public boolean isActive() {
        return active;
    }
```

## Variable names
Variable names should be written in lowercase with uppercase to mark the start of a new word.

Examples:
- `private String name;`
- `private Long previousId;`
- `private boolean active;`

The name of the variable must clearly identify what it is stored on it. So, a variable can't be called `variable`, `var`, `myVar`, `myString`, `theBoolean` or something like this. We also avoid to use a single letter except in some specific cases like `i`, `j`, `k` for counters or `x`, `y`, `z` when we are drawing a graph.

**<u>Note</u> :** We do not start a `boolean` variable with the prefix "is" because a variable does not generally start with a verb. Moreover, this would result in an accessor whose signature would start with "isIs" which is not good for readability.

## Constant names
Constant names should be capitalized with `_` to mark the beginning of a new word.

Examples:
- `public static final String DEFAULT_VALUE = "default value";`
- `public static final int MAX_VALUE = 100_000;`

## Enums

Enums are constants and follow the same naming rules:

```java
     public enum Status {
         DRAFT, IN_PROGRESS, ACTIVE, ARCHIVED;
     }
```
# Class organization

After the declaration of the class we found (in this order) :
1. constants
1. instance fields
1. constructors
1. methods

Each of these elements are optional but for lisibility reasons they must not be mixed.
So a class must look like this :

```java
    public class MyClass {

        private static final String DEFAULT_NAME = "Name by default";

        private String name;
        
        /**
         * Constructor of MyCLass
         *
         * @param name Name of the instance
         */
        public MyClass(String name) {
            this.name = name;
        }

        /**
         * Constructor of MyCLass. Name will be set to the default value.
         */        
        public MyClass() {
            this(DEFAULT_NAME);
        }

        /**
         * @return the name
         */        
        public String getName() {
            return this.name;
        }
    }
```
# Code format

To increase code lisibility and easy comparison between different versions, it is necessary that everyone uses the same code format. Some basic rules :

- **Always use braces for code blocks**, even if there is only one line in the block.
- Never more than **one statement per line**. <u>Notice</u> : `if`/`else`, `for`/`do`/`while` are statement . That means after a `if` or a `for` there must have nothing else
- ident the code on each new block of statements (if, for...)
- don't write a very long line

The following code **is not correct**
```java
    if (myObject != null) x = myObject .getValue();
    for (int i = 0; i < getParamSize(); i++)
    x += getParam(i);
    myObject.getReferencedElement().getEmbeddedStringBuilder().append(CONSTANT_STRING_A).append(CONSTANT_STRING_B).append(CONSTANT_STRING_D).append(CONSTANT_STRING_D).append(CONSTANT_STRING_E).append(CONSTANT_STRING_F).append(CONSTANT_STRING_G).append(CONSTANT_STRING_H);
```

**This code is really ugly** and after a quick read, we can miss `x = myObject.getValue();` statement and suppose the `for` instruction is directly after the `if`. Big misunderstanding 😱.
And it is not possible to read the end of the last line without scrolling.

So, this code must be replaced by
```java
    if (myObject != null) {
        x = myObject.getValue();
    }

    for (int i = 0; i < getParamSize(); i++) {
        x += getParam(i);
    }   

    myObject.getReferencedElement().getEmbeddedStringBuilder()
                      .append(CONSTANT_STRING_A).append(CONSTANT_STRING_B).append(CONSTANT_STRING_C).append(CONSTANT_STRING_D)
                      .append(CONSTANT_STRING_E).append(CONSTANT_STRING_F).append(CONSTANT_STRING_G).append(CONSTANT_STRING_H);
```

Now, `if` and `for` blocks can be clearly identified and it is possible to read the last statement

## Blank line
For readability reason, it is necessary to add one blank line in some cases :

- after the class declaration
- after constants declaration
- after class variables declaration
- between the local variables in a method and its first statement
- between methods

The following code **is not correct**
```java
public class Person {
    private static final String DEFAULT_VALUE = "Unknown";
    private static final String ANOTHER_CONSTANT = "Not used";
    private String name;
    private String firstname;
    public String computeSomething() {
        StringBuilder builder = new StringBuilder();
        builder.append("One value for ");
        if (StringUtils.isBlanck(this.name) {
            builder.append(DEFAULT_VALUE);
        } else {
            builder.append(this.name);
        }
        builder.append(" is correct.");
        return builder.toString();
    }
    public String getName() {
        return this.name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public String getFirstname() {
        return this.firstname;
    }
    public void setFirstname(String firstname) {
        this.firstname = firstname;
    }
}
```
It must be replaced by
```java
public class Person {

    private static final String DEFAULT_VALUE = "Unknown";
    private static final String ANOTHER_CONSTANT = "Not used";

    private String name;
    private String firstname;

    public String computeSomething() {
        StringBuilder builder = new StringBuilder();

        builder.append("One value for ");

        if (StringUtils.isBlanck(this.name) {
            builder.append(DEFAULT_VALUE);
        } else {
            builder.append(this.name);
        }
        builder.append(" is correct.");

        return builder.toString();
    }

    public String getName() {
        return this.name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getFirstname() {
        return this.firstname;
    }

    public void setFirstname(String firstname) {
        this.firstname = firstname;
    }
}
```

If there is a lot of constants/variables defined, it is possible (and recommanded) to had blank lines into the declarations block to group them by concept

Example:
```java
    public static final String SUB_TYPE_CONSULTANT = "sub_type.consultant";
    public static final String SUB_TYPE_CANDIDATE = "sub_type.candidate";
    public static final String SUB_TYPE_TRAINEE = "sub_type.trainee";
    public static final String SUB_TYPE_TRAINEE_CANDIDATE = "sub_type.trainee_candidate";
    public static final String SUB_TYPE_BUSINESS_MANAGER = "sub_type.business_manager";
    public static final String SUB_TYPE_FUNCTIONAL = "sub_type.functional";

    public static final String POLE_IDS_FRANCE_CODE = "pole.idsfrance";
    public static final String POLE_ODC_MAROC = "pole.odc-maroc";
    public static final String POLE_IDS_TECHNO = "pole.idsTechnology";
    public static final String POLE_SANTE = "pole.sante";
    public static final String POLE_SOLUTION = "pole.solutions";
    public static final String POLE_SIRT = "pole.sirt";
    [...]
```

## Blank character

For lisibility reason, it is necessary to add one blank character in many cases :

- after a coma
- before opening bracket and after closing one. This rule must be applied in statements but not for a method declaration/use
- before opening brace and after closing one
- between operands of an operation. 
- between test elements. 

The following code **is not correct**
```java
    public void compute(MyObject objectA,MyObject objectB){
        if(objectA.isOk()){
           x=objectA.getValue();
        }else if(objectB.isOk()){
           x=objectB.getValue();
        }else if(objectA.isActive()&&objectB.isActive()){
           x=objectA.getValue()+objectB.getValue();
        }else{
           x=0;
        }
    }
```
and must be replaced by
```java
    public void compute(MyObject objectA, MyObject objectB) {
        if (objectA.isOk()) {
           x = objectA.getValue();
        } else if (objectB.isOk()) {
           x = objectB.getValue();
        } else if (objectA.isActive() && objectB.isActive()) {
           x = objectA.getValue() + objectB.getValue();
        } else {
           x = 0;
        }
    }
```

IntelliJ provide an easy way to **auto format code : CTRL + ALT + L**. It won't fix all rules previously listed (it won't add add braces for example) but it will add missing spaces and can force some carriage return. Don't forget to do it on every updated classes before pushing your changes.

# Methods

## size

For readability it's important to avoid too long methods. Everybody agrees on this point but it is more complicated when we have to determine the value of the max size. After some research on internet I found a lot of answers for this question: between 5 to 10, 15, 20, 30. In theses cases, the count is done <b>without blanck line, closing braces, Javadoc, nor comments</b>.
In my opinion, it's difficult to apply a very strict rule for this point. I like the rule "not more than one screen" which allow us to have a complete view of the method without scrolling. Than means the method must not have more than 40 lines <b>with blanck lines, comments and closing braces</b>.

## Parameters

### Reassignment

To improve the readability, it is advised to no reassign a parameter.

### Max number
For readability, it's also necessary to limit the number of parameters. Here too, we can find a lot of answers to this question on internet from 3 to 7. 
I think 5 is a good answer and we can consider it is possible in very very rare cases to go beyond this limit. 
If you used to write method with more than 4 parameters, I have an advise : don't forget that Java is an Object-Oriented Programming language and group your parameters into an object. With this approach, it's also easier to evolve if you need to add a new value.

Example:
```java
public boolean compute(String streetAddress, String city, String zipCode, String state, String country) {
   // Some code lines
}
```

// This can be refactored as below to increase readability
```java
public boolean setCustomerAddress(Address address) {
   // Some code lines
}
```
If you need to add a `streetAddressComplement` parameter, you'll just need to add a field into the `Address` object instead of change potentially many method's signatures
