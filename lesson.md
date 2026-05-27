# Lesson: Exception Handling and Logging in Java

## Lesson Overview

This lesson introduces **exception handling** and **logging** in Java (Java 21 ready). You'll learn how Java throws and handles exceptions, when to use LBYL vs EAFP, how to design specific vs general handlers, and how to propagate with `throws`. You'll also adopt the **SLF4J** façade and use **Logback** (console/file appenders) so behavior can change without touching application code.

## Lesson Objectives

By the end of this lesson, learners will be able to:

1. **Implement** exception handling using `try-catch-finally` and explain the call stack and stack trace
2. **Distinguish** checked vs unchecked exceptions and build custom exceptions for domain-specific failures
3. **Apply** logging to a Java program using SLF4J + Logback

---

## Part 1: Exceptions / Try-Catch-Finally

### What is an Exception?

An exception is an event, which occurs during the execution of a program, that disrupts the normal flow of the program's instructions.

https://docs.oracle.com/javase/tutorial/essential/exceptions/definition.html

When an error occurs within a method, the method creates an object and hands it off to the runtime system. The object, called an **exception object**, contains information about the error, including its type and the state of the program when the error occurred. Creating an exception object and handing it to the runtime system is called **throwing an exception**.

<img src="https://www3.ntu.edu.sg/home/ehchua/programming/java/images/Exception_CallStack.png" width="450">

Source: https://www3.ntu.edu.sg/home/ehchua/programming/java/j5a_exceptionassert.html

The runtime system searches the **call stack** for a method that contains a block of code that can handle the exception. This block of code is called an **exception handler**.

If no appropriate exception handler is found, then the program terminates.

**Advantages of Exceptions**

1. **Separating Error-Handling Code from "Regular" Code**
   The ability to throw exceptions makes it easy to separate error-handling code from regular code.

2. **Propagating Errors Up the Call Stack**
   The ability to propagate errors up the call stack means you can write methods for the normal case and let callers handle boundary or unexpected situations.

3. **Grouping and Differentiating Error Types**
   Each exception type indicates a particular error condition. You can create your own exceptions to signal domain-specific failures.

### Java Exception Hierarchy

All exceptions in Java are objects. They all inherit from `Throwable` at the top of the hierarchy.

```
Throwable
├── Error                          (serious system-level problems — do NOT catch these)
│   ├── OutOfMemoryError
│   ├── StackOverflowError
│   └── ...
└── Exception                      (recoverable problems — these are what we handle)
    ├── IOException                 ← checked
    ├── FileNotFoundException       ← checked
    ├── SQLException                ← checked
    ├── ClassNotFoundException      ← checked
    └── RuntimeException            ← unchecked (root of all unchecked exceptions)
        ├── ArithmeticException
        ├── NullPointerException
        ├── ArrayIndexOutOfBoundsException
        ├── IllegalArgumentException
        ├── ClassCastException
        └── ...
```

**Key distinction:**
- **`Error`** — serious problems outside your control (e.g. JVM running out of memory). You should never try to catch these.
- **`Exception`** — problems your program can anticipate and recover from. This is what we work with.
- **`RuntimeException`** — a subclass of `Exception`. These are unchecked — the compiler does not force you to handle them.

### Checked vs Unchecked Exceptions

#### What is a Checked Exception?

A **checked exception** is an exception that the Java compiler forces you to handle or declare. If you call a method that can throw a checked exception, you must either:
- Handle it with a `try-catch` block, or
- Declare it with `throws` in your method signature

Checked exceptions represent conditions that are **outside the programmer's control** but are reasonably expected — for example, a file not being found, or a network connection failing.

**Common checked exceptions:**
| Exception | When it occurs |
|-----------|---------------|
| `IOException` | General input/output failure |
| `FileNotFoundException` | File does not exist at the given path |
| `SQLException` | Database access error |
| `ClassNotFoundException` | Class cannot be found at runtime |

#### What is an Unchecked Exception?

An **unchecked exception** is an exception that the compiler does **not** force you to handle. They are subclasses of `RuntimeException` and typically represent **programming errors** — bugs in your code that should be fixed, not caught.

**Common unchecked exceptions:**
| Exception | When it occurs |
|-----------|---------------|
| `ArithmeticException` | Illegal arithmetic operation e.g. divide by zero |
| `NullPointerException` | Accessing a method or field on a `null` reference |
| `ArrayIndexOutOfBoundsException` | Accessing an array with an invalid index |
| `IllegalArgumentException` | A method receives an argument it cannot accept |
| `ClassCastException` | Illegal cast between incompatible types |

### Looking at Our First Exception

Create a file `LearnExceptions.java` and code along.

Add a method `divide`.

```java
public static int divide(int x, int y) {
  return x / y;
}
```

And call it in the `main` method.

```java
public static void main(String[] args) {
  int x = 10;
  int y = 0;
  int result = divide(x, y);
  System.out.println(result);
}
```

Run the code and see what happens.

As you might expect, the code throws an exception because you cannot divide by zero.

Java prints something known as a **stack trace**. The stack trace shows what is known as the **call stack** (the list of methods that were called). It shows the order in which methods are called and where the error occurred.

In our example here, the call stack is: `main` → `divide`.

```java
Exception in thread "main" java.lang.ArithmeticException: / by zero
        at LearnExceptions.divide(LearnExceptions.java:15)
        at LearnExceptions.main(LearnExceptions.java:5)
```

### LBYL and EAFP Approaches

There are 2 main approaches to dealing with errors when programming.

1. **Look Before You Leap (LBYL)**
2. **Easier to Ask for Forgiveness than Permission (EAFP)**

LBYL means that you check for errors before you execute the code.

EAFP means that you let the code run, throw an exception if there is an error, and then handle the exception.

LBYL vs EAFP: https://programmingduck.com/articles/lbyl-eafp

Using the LBYL approach in our `main`, we can first check if the divisor is zero before we allow the `divide` method to be called.

```java
public static void main(String[] args) {
  int x = 10;
  int y = 0;

  if (y == 0) {
    System.out.println("Cannot divide by zero");
  } else {
    int result = divide(x, y);
    System.out.println(result);
  }
}
```

As you can see, with LBYL, the developer proactively checks preconditions before executing the code, which avoids the exception. Disadvantages include time-of-check/time-of-use gaps and duplicated checks.

### Try-Catch-Finally

The other approach is EAFP using a `try-catch` block. Here is what each block does:

- **`try`** — wraps the code that might throw an exception. Java monitors this block and if an exception occurs, execution immediately jumps to the matching `catch` block.
- **`catch`** — defines how to handle a specific exception type. It only runs if the matching exception is thrown inside the `try` block. You can have multiple `catch` blocks for different exception types.
- **`finally`** — runs **always**, whether an exception was thrown or not, whether it was caught or not. Use it for cleanup code that must always execute — such as closing a file, releasing a database connection, or freeing a resource.

```java
try {
  int result = divide(x, y);
  System.out.println(result);
} catch (ArithmeticException exception) {
  System.out.println(exception);
} finally {
  System.out.println("This always runs — cleanup goes here");
}
```

Let's look at an array example.

```java
int[] numbers = { 1, 2, 3, 4, 5 };
int index = 5;
System.out.println(numbers[index]); // throws ArrayIndexOutOfBoundsException
```

LBYL:

```java
if (index >= 0 && index < numbers.length) {
  System.out.println(numbers[index]);
} else {
  System.out.println("Index is out of bounds");
}
```

EAFP:

```java
try {
  System.out.println(numbers[index]);
} catch (ArrayIndexOutOfBoundsException exception) {
  System.out.println(exception);
}
```

### Try-with-Resources (Java 7+, recommended in Java 21)

When working with resources like files, database connections, or network streams, you must always close them after use — even if an exception occurs. Forgetting to close a resource causes **resource leaks**, which can crash your application over time.

Before Java 7, developers had to close resources manually in a `finally` block:

```java
FileInputStream f = null;
try {
  f = new FileInputStream("test.txt");
  // read file...
} catch (IOException e) {
  System.out.println(e.getMessage());
} finally {
  if (f != null) {
    try {
      f.close(); // must close manually — and this can also throw!
    } catch (IOException e) {
      System.out.println(e.getMessage());
    }
  }
}
```

This is verbose and error-prone. **Try-with-resources** solves this by automatically closing any resource declared in the `try(...)` parentheses when the block ends — whether normally or due to an exception.

```java
import java.io.IOException;
import java.io.FileInputStream;
import java.util.Scanner;

try (FileInputStream f = new FileInputStream("test.txt");
     Scanner s = new Scanner(f)) {
  if (s.hasNextLine()) {
    System.out.println(s.nextLine());
  }
} catch (IOException exception) {
  System.out.println(exception.getMessage());
}
```

> Any class that implements the `AutoCloseable` interface can be used in try-with-resources. Java's built-in I/O classes, database connections, and most resource classes already implement it.

### Propagating Exceptions Up the Call Stack

We can also propagate exceptions up the call stack. Put the previous file-reading code into a method and **do not** handle it there — let callers decide.

```java
import java.io.IOException;
import java.io.FileInputStream;
import java.util.Scanner;

public static void readFileFirstLine(String filename) throws IOException {
  try (FileInputStream f = new FileInputStream(filename);
       Scanner s = new Scanner(f)) {
    if (s.hasNextLine()) {
      System.out.println(s.nextLine());
    } else {
      System.out.println("File is empty");
    }
  }
}
```

And in `main`:

```java
public static void main(String[] args) {
  try {
    readFileFirstLine("test.txt");
  } catch (IOException exception) {
    System.out.println(exception.getMessage());
  }
}
```

### General vs Specific Exception Types

When catching exceptions, always prefer **specific** exception types over general ones.

**Specific catch — preferred:**

```java
try {
  int result = 10 / 0;
  System.out.println(result);
} catch (ArithmeticException exception) {
  System.out.println("Cannot divide by zero: " + exception.getMessage());
}
```

**General catch — avoid:**

```java
try {
  int result = 10 / 0;
  System.out.println(result);
} catch (Exception exception) {
  // Bad: you have no idea what actually went wrong
  System.out.println("Something went wrong: " + exception.getMessage());
}
```

**Why specific is better:**

- It makes your intent clear — you know exactly what error you are handling
- A broad `catch (Exception e)` will silently swallow unexpected exceptions you didn't anticipate, making bugs very hard to find
- It forces you to think about each failure mode individually

**When you have multiple possible exceptions, catch each one specifically:**

```java
try {
  int[] numbers = { 1, 2, 3 };
  int result = numbers[5] / 0;
} catch (ArrayIndexOutOfBoundsException e) {
  System.out.println("Invalid index: " + e.getMessage());
} catch (ArithmeticException e) {
  System.out.println("Cannot divide by zero: " + e.getMessage());
}
```

> **Rule of thumb:** Only catch `Exception` broadly at the very top level of your application (e.g. in `main`) as a last resort safety net — never deep inside your business logic.

---

## Part 2: Custom Exceptions

We can also create our own custom exceptions. To create a **checked** exception, extend `Exception`. To create an **unchecked** exception, extend `RuntimeException`.

Unchecked example:

```java
public class InvalidArrayIndexException extends RuntimeException {
  public InvalidArrayIndexException(String message) {
    super(message);
  }
}
```

Usage:

```java
try {
  if (index < 0 || index > numbers.length - 1) {
    throw new InvalidArrayIndexException(index + " is not a valid index!");
  }
  System.out.println(numbers[index]);
} catch (InvalidArrayIndexException exception) {
  System.out.println(exception);
}
```

Another unchecked example:

```java
public class NegativeNumberException extends RuntimeException {
  public NegativeNumberException(String message) {
    super(message);
  }
}
```

```java
public static int dividePositiveNumbers(int a, int b) {
  if (a < 0 || b < 0) {
    throw new NegativeNumberException("Negative numbers are not allowed.");
  }
  return a / b;
}
```

```java
try {
  int result = dividePositiveNumbers(10, -2);
  System.out.println(result);
} catch (NegativeNumberException exception) {
  System.out.println(exception);
}
```

### 👨‍💻 Activity: Custom Exceptions **(10 minutes)**

#### Task 1: Unchecked Custom Exception

Create an unchecked exception `NoNegativeElementException`. Next, create a `sumPositiveArray` method that takes in an array of integers and throws `NoNegativeElementException` if the array contains a negative number, and returns the total sum of the array if all elements are positive.

```java
public static int sumPositiveArray(int[] numbers) {
  // your code here
}
```

#### Task 2: Checked Custom Exception

Create a checked exception `DivideByZeroElementException`. Next, create a `divideByElementAtIndex` method that takes an array of integers and an index. Throw `DivideByZeroElementException` if the divisor is zero, and return the division result otherwise.

```java
public static int divideByElementAtIndex(int[] numbers, int index) throws DivideByZeroElementException {
  // your code here
}
```

---

## Part 3: Maven Project Setup

> ✅ You should have completed Maven setup as part of your self-study (studies.md Task 2). If you have not done so, refer to studies.md now and complete the setup before continuing.

**Step 1** — Verify Maven is installed:

```bash
mvn -v
```

**Step 2** — Open your Maven project from self-study in VSCode.

**Step 3** — Open `App.java` and run it. You should see `Hello World!` in the terminal.

You are now ready to add SLF4J and Logback dependencies in Part 4.

---

## Part 4: Logging Using SLF4J

### What is Logging?

Logging is the practice of recording information about your program's execution to an output destination — such as the console, a file, or an external monitoring system.

You might be used to using `System.out.println()` to debug your code. In a real production application, this is not enough because:

- You **cannot control the output level** — everything prints regardless of severity
- There are **no timestamps** — you don't know when something happened
- You **cannot filter** — you can't say "only show me errors in production"
- You **cannot redirect output** — `System.out` always goes to the console, never to a file or monitoring system
- It **cannot be turned off** without deleting the lines from your code

A logging framework solves all of these problems. It lets you:
- Attach a **severity level** to every message (DEBUG, INFO, WARN, ERROR)
- **Filter by level** — show DEBUG in development, only ERROR in production
- **Write to multiple destinations** at once — console and file simultaneously
- Include **timestamps, thread names, and class names** automatically
- Change logging behaviour through **configuration only**, without touching your code

### What is SLF4J?

**SLF4J (Simple Logging Facade for Java)** is not a logging framework itself — it is a **facade** (an abstraction layer) that sits in front of a real logging implementation.

Think of it like this: SLF4J is the **interface**, and Logback (or Log4j, or others) is the **implementation**.

```
Your Code
    ↓
SLF4J API  (you always code against this)
    ↓
Logback / Log4j / slf4j-simple  (the actual implementation — swappable)
```

The benefit is that your application code **never changes** when you switch logging frameworks. You only swap the dependency in `pom.xml`. This is the same principle as coding to a `List` interface instead of `ArrayList` — your code stays stable, only the implementation changes.

### Adding Dependencies (SLF4J 2.x)

Search on https://mvnrepository.com/ and add:

```xml
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-api</artifactId>
  <version>2.0.12</version>
</dependency>

<!-- Simple implementation (good for demos/tests). Remove when switching to Logback. -->
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-simple</artifactId>
  <version>2.0.12</version>
  <scope>runtime</scope>
</dependency>
```

### Using SLF4J

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class App {

  private static final Logger logger = LoggerFactory.getLogger(App.class);

  public static void main(String[] args) {
    System.out.println("Hello World!");
    logger.info("Application started");
    logger.info("This is an informational message.");
    logger.error("This is an error message.");
    logger.warn("This is a warning message.");
    logger.info("Application ended");
  }
}
```

Run the application and check the console output.

---

## Part 5: Logging Using Logback

The advantage of SLF4J is easy swapping of logging frameworks. Now switch to **Logback**.

### What is Logback?

**Logback** is a mature, production-grade logging framework for Java. It is the **natural successor to Log4j** and is designed to be faster and more capable. It is the most widely used SLF4J implementation in the Java ecosystem.

Logback gives you:
- Full control over **log levels** per class or package
- **Multiple appenders** — write to console, file, database, or remote systems simultaneously
- **Rolling file appenders** — automatically rotate log files by size or date
- **Pattern-based formatting** — customise exactly what each log line looks like
- Configuration via `logback.xml` — **no code changes needed** to change logging behaviour

### Adding Dependencies (Logback 1.5.x)

Remove `slf4j-simple` from `pom.xml` and add:

```xml
<dependency>
  <groupId>ch.qos.logback</groupId>
  <artifactId>logback-classic</artifactId>
  <version>1.5.6</version>
</dependency>
```

Just by changing this one dependency, you have switched from `slf4j-simple` to `logback-classic`. Your application code using SLF4J stays exactly the same — this is the facade pattern in action.

### Configure Logback

Logback is configured using a file called `logback.xml` placed in `src/main/resources`.

> **Note:** If you do not see a `resources` folder under `src/main`, create it manually: right-click `src/main` → New Folder → name it `resources`.

Create `src/main/resources/logback.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
  </appender>

  <root level="DEBUG">
    <appender-ref ref="STDOUT" />
  </root>
</configuration>
```

Run the application and check the console output. Everything should work as before.

### Logging Levels in Logback

| Level   | Description                                                                  |
| ------- | ---------------------------------------------------------------------------- |
| `TRACE` | Finer-grained informational events than DEBUG.                               |
| `DEBUG` | Fine-grained informational events useful for debugging.                      |
| `INFO`  | Informational messages that highlight application progress.                  |
| `WARN`  | Potentially harmful situations.                                              |
| `ERROR` | Error events that might still allow the application to continue.             |

Levels are hierarchical — setting the root level to `INFO` means only `INFO`, `WARN`, and `ERROR` messages appear. `DEBUG` and `TRACE` are suppressed. This is how you control verbosity between development and production environments — **just change the level in `logback.xml`, no code changes needed**.

### Logging to a File

In production, your application runs on a server — there is no console for you to watch. **File logging** is essential because:

- It creates a **persistent record** of everything that happened — even after the server restarts
- You can **review logs after the fact** when investigating a bug or incident
- Multiple team members can access the same log file
- Log files can be shipped to centralised monitoring tools (e.g. Datadog, Splunk, ELK Stack)

Console logs disappear the moment a process stops. File logs stay. In real applications you typically configure **both** — console for development convenience, file for production persistence.

To log to both console and file, update `logback.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <!-- Console Appender -->
  <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
  </appender>

  <!-- File Appender -->
  <appender name="FILE" class="ch.qos.logback.core.FileAppender">
    <file>logs/application.log</file>
    <encoder>
      <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
  </appender>

  <root level="INFO">
    <appender-ref ref="CONSOLE"/>
    <appender-ref ref="FILE"/>
  </root>
</configuration>
```

The `logs/application.log` file will be created automatically by Logback when the application runs — you do not need to create it manually.

Check `logs/application.log` after running.

For further reading, refer to the [logback documentation](http://logback.qos.ch/documentation.html).

### 👨‍💻 Activity: SLF4J + Logback **(5 minutes)**

In your Maven project `App.java`:

- Add `INFO` logs for application start and end
- Trigger an `ArithmeticException` (divide by zero) inside a `try-catch` and log it with `ERROR`
- Run the app and verify output appears in both the console and `logs/application.log`
- Note: SLF4J/Logback uses `ERROR` — there is no `SEVERE` like in JUL

---

## 🔵 Optional: Multi-catch

Java lets you catch multiple exception types in a single `catch` block using the `|` operator. This is useful when two or more exceptions should be handled the same way.

```java
try {
  // code that may throw different exceptions
} catch (IllegalArgumentException | IllegalStateException e) {
  System.out.println("Bad input/state: " + e.getMessage());
}
```

---

END