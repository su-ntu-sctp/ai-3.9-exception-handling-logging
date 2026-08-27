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

Let's look at a more realistic example — accessing a customer from a list by position. Say we have a list of customer names, and we try to read a position that does not exist:

```java
List<String> customers = new ArrayList<>();
customers.add("Alice");
customers.add("Bob");
customers.add("Charlie");

int index = 5; // out of range — valid positions are 0, 1, 2
System.out.println(customers.get(index)); // throws IndexOutOfBoundsException
```

LBYL:

```java
if (index >= 0 && index < customers.size()) {
  System.out.println(customers.get(index));
} else {
  System.out.println("No customer at that position");
}
```

EAFP:

```java
try {
  System.out.println(customers.get(index));
} catch (IndexOutOfBoundsException exception) {
  System.out.println("No customer at that position");
}
```

> **Note:** for a `List` we check the size with `customers.size()` (an array would use `.length`), and an out-of-range position on a list throws `IndexOutOfBoundsException`.

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

> **Note on the catch type:** catch `IOException`, not `FileNotFoundException`. The automatic close on `FileInputStream` can throw `IOException`, so catching only the narrower `FileNotFoundException` will fail to compile. `IOException` covers both the missing-file case (from the constructor) and any failure during the auto-close.

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

Let's use a list of customer scores. We want to divide a fixed total by a score, but a score might be zero, and we might ask for a position that doesn't exist.

```java
List<Integer> scores = new ArrayList<>();
scores.add(90);
scores.add(0);
scores.add(75);
```

**Specific catch — preferred:**

```java
try {
  int result = 100 / scores.get(1); // scores.get(1) is 0 → ArithmeticException
  System.out.println(result);
} catch (ArithmeticException exception) {
  System.out.println("Cannot divide by zero: " + exception.getMessage());
}
```

**General catch — avoid:**

```java
try {
  int result = 100 / scores.get(1);
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
  int value = scores.get(5);      // no position 5 → IndexOutOfBoundsException
  int result = 100 / value;       // could also be a divide-by-zero
  System.out.println(result);
} catch (IndexOutOfBoundsException e) {
  System.out.println("Invalid position: " + e.getMessage());
} catch (ArithmeticException e) {
  System.out.println("Cannot divide by zero: " + e.getMessage());
}
```

> **Rule of thumb:** Only catch `Exception` broadly at the very top level of your application (e.g. in `main`) as a last resort safety net — never deep inside your business logic.

---

## Part 2: Custom Exceptions

We can also create our own custom exceptions. To create a **checked** exception, extend `Exception`. To create an **unchecked** exception, extend `RuntimeException`.

Custom exceptions let us signal problems in the language of our own application. Instead of a generic message, we can say exactly what went wrong in *our* domain — for example, a customer that could not be found.

Let's work with a simple `Customer` class.

```java
public class Customer {
  private int id;
  private String name;

  public Customer(int id, String name) {
    this.id = id;
    this.name = name;
  }

  public int getId() {
    return id;
  }

  public String getName() {
    return name;
  }
}
```

Now create a **checked** custom exception for the case where a customer is not found. Because we extend `Exception` (and not `RuntimeException`), this is a **checked** exception — the compiler will force callers to deal with it.

```java
public class CustomerNotFoundException extends Exception {
  public CustomerNotFoundException(String message) {
    super(message);
  }
}
```

Here is a method that looks up a customer in a list by id, and throws our custom exception if there is no match. Since the exception is checked, the method **must** declare `throws CustomerNotFoundException`.

```java
public static Customer findCustomerById(List<Customer> customers, int id) throws CustomerNotFoundException {
  for (Customer customer : customers) {
    if (customer.getId() == id) {
      return customer;
    }
  }
  throw new CustomerNotFoundException("No customer found with id: " + id);
}
```

The method does not handle the problem itself — it propagates it up to whoever called it. So the caller in `main` is now **forced** by the compiler to handle it with a `try-catch`:

```java
List<Customer> customers = new ArrayList<>();
customers.add(new Customer(1, "Alice"));
customers.add(new Customer(2, "Bob"));

try {
  Customer customer = findCustomerById(customers, 5);
  System.out.println(customer.getName());
} catch (CustomerNotFoundException exception) {
  System.out.println(exception.getMessage());
}
```

Looking something up that isn't there — a "not found" — is one of the most common real-world exceptions you will ever write. Notice how much clearer `CustomerNotFoundException` is than a generic error: the message even carries the exact id that failed.

> **Try this:** remove the `throws CustomerNotFoundException` from the method signature and see what the compiler says. Then put it back and remove the `try-catch` from `main` instead. Both give compile errors — that is exactly what "checked" means: the compiler will not let you ignore it.

Note the two keywords are different:
- **`throw`** (no *s*) — an action. It fires the exception right now: `throw new CustomerNotFoundException(...)`
- **`throws`** (with *s*) — a declaration on the method signature saying "this exception may come out of me, and I am not handling it"

### 👨‍💻 Activity: Custom Exceptions **(10 minutes)**

In the demo above we built a **checked** exception. Now you will build an **unchecked** one.

Create an unchecked exception `InvalidCustomerException` and use it to reject a customer with an empty name. Because it extends `RuntimeException`, it is **unchecked** — no `throws` needed on the method, and the caller is not forced to catch it.

Copy the snippets below and fill in the blanks.

**Step 1 — the exception class.** Create `InvalidCustomerException.java`:

```java
public class InvalidCustomerException extends _____________ {
  public InvalidCustomerException(String message) {
    _____________;
  }
}
```

**Step 2 — the method.** Add this to your class, alongside `main`. It should throw `InvalidCustomerException` if the new customer's name is empty; otherwise add the customer to the list and return the new size of the list.

```java
public static int addCustomer(List<Customer> customers, Customer newCustomer) {

  if (newCustomer.getName() == null || newCustomer.getName().isBlank()) {
    throw new _____________("Customer name cannot be empty");
  }

  customers._____________(newCustomer);
  return customers._____________;
}
```

**Step 3 — call it from `main`.** Test both cases — a valid customer, and one with an empty name:

```java
List<Customer> customers = new ArrayList<>();

// valid customer — should be added
int size = addCustomer(customers, new Customer(1, "Alice"));
System.out.println("Customers in list: " + size);

// invalid customer — should throw
try {
  addCustomer(customers, new Customer(2, ""));
} catch (_____________ exception) {
  System.out.println(exception.getMessage());
}
```

> **Question to think about:** the `try-catch` in Step 3 is optional here — the code compiles fine without it. Why? (Compare this with the checked `CustomerNotFoundException` in the demo, where the compiler *forced* you to catch it.)

---

## Part 3: Maven Project Setup

> ✅ **This is an in-class verification step, not a setup step.** Both the installation and the project setup were completed before this lesson:
> - **Maven installation** was done as **pre-work in Lesson 1** (see Lesson 1's `setup.md`).
> - **Maven project creation** (via `maven-archetype-quickstart`, setting the `pom.xml` compiler release to 21, and verifying `Hello World!`) was done as **self-study for this lesson** (see `studies.md` Task 2).
>
> If you have not completed either, refer to those materials now and complete them before continuing.

In class, we simply verify the pre-class work is in place before we start adding logging dependencies.

**Step 1** — Verify Maven is installed:

```bash
mvn -v
```

**Step 2** — Open your Maven project (created in `studies.md` Task 2) in VSCode.

**Step 3** — Open `App.java` and run it. You should see `Hello World!` in the terminal.

You are now ready to add SLF4J and Logback dependencies in Part 4.

---

## Part 4: Logging Using SLF4J and Logback

### What is Logging?

Logging is the practice of recording information about your program's execution to an output destination — such as the console, a file, or an external monitoring system.

You might be used to using `System.out.println()` to debug your code. In a real production application that is not enough: there are no severity levels, no timestamps, you cannot filter or switch it off, and it always goes to the console — never to a file or a monitoring system.

A logging framework solves all of that. It lets you:
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
Logback / Log4j / others  (the actual implementation — swappable)
```

The benefit is that your application code **never changes** when you switch logging frameworks. You only swap one dependency in `pom.xml`. This is the same principle as coding to a `List` interface instead of `ArrayList` — your code stays stable, only the implementation changes.

> **Note:** we are going straight to Logback as our implementation. If you ever wanted Log4j instead, you would change that **one dependency line** in `pom.xml` and not a single line of your Java code. That is the whole point of the facade.

### What is Logback?

**Logback** is a mature, production-grade logging framework for Java. It is the **natural successor to Log4j** and is the most widely used SLF4J implementation in the Java ecosystem. It is also what Spring Boot uses by default, so this is the stack you will meet in real projects.

Logback gives you:
- Full control over **log levels** per class or package
- **Multiple appenders** — write to console, file, database, or remote systems simultaneously
- **Rolling file appenders** — automatically rotate log files by size or date
- **Pattern-based formatting** — customise exactly what each log line looks like
- Configuration via `logback.xml` — **no code changes needed** to change logging behaviour

### Adding Dependencies

Add these two dependencies to your `pom.xml` — the SLF4J API (what you code against) and Logback (the implementation that does the work):

```xml
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-api</artifactId>
  <version>2.0.12</version>
</dependency>

<dependency>
  <groupId>ch.qos.logback</groupId>
  <artifactId>logback-classic</artifactId>
  <version>1.5.6</version>
</dependency>
```

### Using SLF4J in Your Code

Notice the log messages describe real application events — the kind of messages you would actually write in a customer-facing app.

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class App {

  private static final Logger logger = LoggerFactory.getLogger(App.class);

  public static void main(String[] args) {
    logger.info("Application started");
    logger.info("Fetching customer list");
    logger.warn("Customer email is missing for id: 2");
    logger.error("Failed to save customer: Alice");
    logger.info("Application ended");
  }
}
```

Two things to notice:

- **`LoggerFactory.getLogger(App.class)`** — we pass in the class itself, so the logger knows which class it belongs to. That class name then appears in every log line automatically, so you always know where a message came from.
- **`private static final`** — one logger per class, created once, shared. You will write this exact line at the top of nearly every class in a real project.

**Parameterised messages.** Instead of joining strings together, use `{}` placeholders and pass the values:

```java
logger.info("Customer {} created with id {}", "Alice", 1);
```

This is the production habit: it is cleaner to read, and the message is only assembled if the log is actually going to be written. With string concatenation the text is built every time, even when the level is switched off.

### Configure Logback

Logback is configured using a file called `logback.xml` placed in `src/main/resources`.

> **Note:** If you do not see a `resources` folder under `src/main`, create it manually: right-click `src/main` → New Folder → name it `resources`. It sits beside the `java` folder (they are siblings), not inside it.

We will configure **console and file output together** in one go. Create `src/main/resources/logback.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <!-- Console Appender -->
  <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
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

Let's decode this:

- An **appender** is *where* the logs go. Here we have two — one for the console, one for a file.
- The **encoder** and its **pattern** control *what each line looks like*: `%d` is the timestamp, `%thread` is the thread name, `%-5level` is the severity padded to 5 characters so the output lines up, `%logger{36}` is the class name, `%msg` is your message, and `%n` is a newline.
- The **root** element ties it together — it sets the base log level and lists which appenders to write to.

The `logs/application.log` file is created automatically by Logback when the application runs — you do not need to create it manually.

Run the application, check the console, then open `logs/application.log` and confirm the same lines are there.

### Logging Levels in Logback

| Level   | Description                                                                  |
| ------- | ---------------------------------------------------------------------------- |
| `TRACE` | Finer-grained informational events than DEBUG.                               |
| `DEBUG` | Fine-grained informational events useful for debugging.                      |
| `INFO`  | Informational messages that highlight application progress.                  |
| `WARN`  | Potentially harmful situations.                                              |
| `ERROR` | Error events that might still allow the application to continue.             |

Levels are hierarchical — setting the root level to `INFO` means only `INFO`, `WARN`, and `ERROR` messages appear. `DEBUG` and `TRACE` are suppressed. This is how you control verbosity between development and production environments — **just change the level in `logback.xml`, no code changes needed**.

### Why File Logging Matters in Production

In production, your application runs on a server — there is no console for you to watch. **File logging** is essential because:

- It creates a **persistent record** of everything that happened — even after the server restarts
- You can **review logs after the fact** when investigating a bug or incident
- Multiple team members can access the same log file
- Log files can be shipped to centralised monitoring tools (e.g. Datadog, Splunk, ELK Stack)

Console logs disappear the moment a process stops. File logs stay. In real applications you typically configure **both** — console for development convenience, file for production persistence.

For further reading, refer to the [logback documentation](http://logback.qos.ch/documentation.html).

### 👨‍💻 Activity: SLF4J + Logback **(5 minutes)**

In your Maven project `App.java`, log a small customer workflow:

- Add an `INFO` log for application start and end
- Log an `INFO` message such as `"Processing customer: Alice"`
- Calculate a customer's average order value as `totalSpent / orderCount`. With `orderCount = 0` this throws an `ArithmeticException` — trigger it inside a `try-catch` and log it with `ERROR`
- Run the app and verify output appears in both the console and `logs/application.log`
- Note: SLF4J/Logback's most severe level is `ERROR` — there is no level above it.