# 3.9 Self Studies

**Estimated Preparation Time:** 70 minutes

---

## Task 1 — Exception Handling in Java (30 minutes)

Watch the following video on Exception Handling in Java:

- 📹 [Exception Handling in Java — https://www.youtube.com/watch?v=1XAfapkBQjk&t=11s]
  

While watching, refer to **lesson.md Part 1 and Part 2** and pay attention to:
- What a stack trace is and how to read it
- The difference between `try`, `catch`, and `finally` and when each block runs
- The difference between checked exceptions (must handle or declare) and unchecked exceptions (runtime errors)
- How to create a custom exception by extending `RuntimeException` or `Exception`

**Guiding Questions:**
1. What is the difference between a checked and an unchecked exception? Give one example of each.
2. When does the `finally` block run? Can you think of a situation where it is useful?
3. Why would you create a custom exception instead of using a built-in one like `IllegalArgumentException`?

---

## Task 2 — Maven Project Setup (15 minutes)

Watch the following video on Maven project setup:

- 📹 [Maven Project Setup in Java — [VIDEO URL NEEDED](https://www.youtube.com/watch?v=rqGRW-ocYpY)]
  

While watching, refer to **lesson.md Part 4** and pay attention to:
- What Maven is and why it is used
- What a `pom.xml` file is and what it contains
- How to add a dependency to a Maven project

**Guiding Questions:**
1. What is the purpose of the `pom.xml` file?
2. What is a dependency in Maven and how do you add one?
3. Why is it useful to use a build tool like Maven instead of managing libraries manually?

---

## Task 3 — SLF4J + Logback (25 minutes)

Watch the following video on SLF4J and Logback:

- 📹 [SLF4J + Logback Logging in Java — https://www.youtube.com/watch?v=oiaEP57nsmI&t=922s]
  

While watching, refer to **lesson.md Parts 5 and 6** and pay attention to:
- What SLF4J is and why it is used as a facade rather than a direct logging framework
- How to add SLF4J and Logback dependencies to `pom.xml`
- How to configure `logback.xml` for console and file output
- The difference between logging levels: TRACE, DEBUG, INFO, WARN, ERROR

**Guiding Questions:**
1. What is the difference between SLF4J and Logback — what does each one do?
2. What does the `logback.xml` configuration file control?
3. If you wanted to stop DEBUG logs from appearing in production, what would you change in `logback.xml`?

---

## Active Engagement Strategies

- Pause and code along with each video in VS Code
- After each video, close it and try to recreate the example from memory
- Use the guiding questions to check your understanding before class

---

## Additional Reading Material

- [Java Exception Handling — W3Schools](https://www.w3schools.com/java/java_try_catch.asp)
- [Java Custom Exceptions — Baeldung](https://www.baeldung.com/java-new-custom-exception)
- [Maven in 5 Minutes — Apache Maven](https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html)
- [SLF4J Documentation](https://www.slf4j.org/manual.html)
- [Logback Documentation](http://logback.qos.ch/documentation.html)