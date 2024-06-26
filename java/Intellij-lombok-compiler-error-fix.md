# Lombok Compiler Error Fix

Recently, we encountered an error at service launch related to Lombok:

```
java: You aren't using a compiler supported by lombok, so lombok will not work and has been disabled.
  Your processor is: com.sun.proxy.$Proxy17
  Lombok supports: OpenJDK javac, ECJ
```

# Temporary Solution

To temporarily fix this issue, follow these steps:

1. **Go to Settings:**
    - Navigate to `Build, Execution, Deployment` -> `Compiler` -> `Shared build process VM options`.

2. **Add the following command:**
    - `-Djps.track.ap.dependencies=false`

Source: [Stack Overflow](https://stackoverflow.com/questions/65128763/java-you-arent-using-a-compiler-supported-by-lombok-so-lombok-will-not-work-a)

