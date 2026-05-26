# Assignment (Optional)

## Brief

Create a Maven project called ExceptionsLoggingAssignment and solve the following problems using exception handling, custom exceptions, and logging with SLF4J + Logback.

1. **Bank Account with Custom Exceptions and Logging**
   - Create a `BankAccount` class with:
     - `private double balance` attribute
     - Constructor that initializes balance
     - `deposit(double amount)` method
     - `withdraw(double amount)` method
   - Create two custom unchecked exceptions:
     - `InsufficientFundsException` - thrown when withdrawal amount exceeds balance
     - `InvalidAmountException` - thrown when deposit or withdrawal amount is negative or zero
   - Implement exception handling:
     - In the `withdraw()` method, check if withdrawal amount exceeds balance and throw `InsufficientFundsException`
     - In both methods, check if amount is <= 0 and throw `InvalidAmountException`
   - Add SLF4J logging:
     - Log INFO messages for successful deposits and withdrawals
     - Log ERROR messages when exceptions are thrown
     - Log WARN messages for suspicious activities (e.g., transactions exceeding $10,000)
   - Test your implementation with multiple scenarios including valid and invalid operations

2. **File Reader with Exception Handling and Logging Configuration**
   - Create a method `readFileContent(String filename)` that:
     - Uses try-with-resources to read a file
     - Handles `FileNotFoundException` (checked exception)
     - Returns the file content as a String
     - Propagates the exception using `throws` in the method signature
   - In the `main` method:
     - Call `readFileContent()` with both an existing file and a non-existing file
     - Use try-catch to handle the FileNotFoundException
     - Add appropriate SLF4J logging at INFO and ERROR levels
   - Configure Logback (`logback.xml`):
     - Set up both console and file appenders
     - Configure log pattern to show timestamp, thread, level, and message
     - Set root logging level to DEBUG
     - Logs should be written to `logs/application.log`
   - Test and verify logs appear in both console and file

## Submission (Optional)

- Submit the URL of the GitHub Repository that contains your work to NTU black board.
- Should you reference the work of your classmate(s) or online resources, give them credit by adding either the name of your classmate or URL.

## References
- Java: https://docs.oracle.com/javase/