I gave ChatGPT-4o a summary of the clean code book written by [wojteklu](https://gist.github.com/wojteklu/73c6914cc446146b8b533c0988cf8d29), and then asked it to support the summary with examples and explanations. Here is the result:

### General Rules

#### Follow Standard Conventions
Using standard conventions makes code predictable and easier to understand.

**Java Example:**
```java
// Correct
public class CustomerService {
    public void addCustomer(Customer customer) {
        // Implementation
    }
}

// Incorrect
public class customerService {
    public void AddCustomer(Customer customer) {
        // Implementation
    }
}
```

#### Keep It Simple, Stupid (KISS)
Simplify code by breaking down complex logic into smaller methods.

**Java Example:**
```java
// Complex
public void processOrder(Order order) {
    // Complex logic
}

// Simplified
public void processOrder(Order order) {
    validateOrder(order);
    calculateTotal(order);
    saveOrder(order);
}

private void validateOrder(Order order) {
    // Validation logic
}

private void calculateTotal(Order order) {
    // Calculation logic
}

private void saveOrder(Order order) {
    // Saving logic
}
```

#### Boy Scout Rule
Always leave the codebase cleaner than you found it.

**Example:**
```java
// Before
public void processOrder(Order order) {
    if (order == null) {
        // Handle error
    }
    // Process order
}

// After (with null check method)
public void processOrder(Order order) {
    if (isValidOrder(order)) {
        // Process order
    }
}

private boolean isValidOrder(Order order) {
    return order != null;
}
```

#### Always Find Root Cause
Investigate and fix the root cause of problems rather than just addressing symptoms.

**Example:**
```java
// Symptom fix
public void processPayment(Payment payment) {
    if (payment.getAmount() > 0) {
        // Process payment
    } else {
        // Log error
    }
}

// Root cause fix
public void processPayment(Payment payment) {
    if (payment == null || payment.getAmount() <= 0) {
        throw new IllegalArgumentException("Invalid payment");
    }
    // Process payment
}
```

### Design Rules

#### Keep Configurable Data at High Levels
Store configuration settings in a configuration file or environment variables.

**Java Example:**
```java
// Configuration file (config.properties)
db.url=jdbc:mysql://localhost:3306/mydb
db.user=root
db.password=root

// Java code
public class DatabaseConfig {
    private Properties config;

    public DatabaseConfig() {
        config = new Properties();
        try (InputStream input = new FileInputStream("config.properties")) {
            config.load(input);
        } catch (IOException ex) {
            ex.printStackTrace();
        }
    }

    public String getDbUrl() {
        return config.getProperty("db.url");
    }

    // Other methods...
}
```

#### Prefer Polymorphism to if/else or switch/case
Use polymorphism to handle different behaviors.

**Java Example:**
```java
// Before
public double calculateArea(Shape shape) {
    if (shape instanceof Circle) {
        Circle circle = (Circle) shape;
        return Math.PI * circle.getRadius() * circle.getRadius();
    } else if (shape instanceof Square) {
        Square square = (Square) shape;
        return square.getSide() * square.getSide();
    }
    return 0;
}

// After
public abstract class Shape {
    public abstract double calculateArea();
}

public class Circle extends Shape {
    private double radius;

    @Override
    public double calculateArea() {
        return Math.PI * radius * radius;
    }
}

public class Square extends Shape {
    private double side;

    @Override
    public double calculateArea() {
        return side * side;
    }
}
```

#### Separate Multi-threading Code
Keep multi-threading code separate from business logic.

**Java Example:**
```java
// Multi-threading logic
public class TaskExecutor {
    private ExecutorService executor = Executors.newFixedThreadPool(10);

    public void executeTask(Runnable task) {
        executor.submit(task);
    }
}

// Business logic
public class OrderService {
    private TaskExecutor taskExecutor;

    public OrderService(TaskExecutor taskExecutor) {
        this.taskExecutor = taskExecutor;
    }

    public void processOrder(Order order) {
        taskExecutor.executeTask(() -> {
            // Process order
        });
    }
}
```

#### Prevent Over-configurability
Focus on essential configurations only.

**Example:**
```java
// Over-configurable
public class ApplicationConfig {
    private Map<String, String> config = new HashMap<>();

    public void setConfig(String key, String value) {
        config.put(key, value);
    }

    public String getConfig(String key) {
        return config.get(key);
    }
}

// Simplified
public class ApplicationConfig {
    private String dbUrl;
    private String dbUser;
    private String dbPassword;

    // Getters and setters
}
```

#### Use Dependency Injection
Inject dependencies rather than instantiating them within classes.

**Java Example:**
```java
// Without DI
public class OrderService {
    private Database database = new Database();

    public void saveOrder(Order order) {
        database.save(order);
    }
}

// With DI
public class OrderService {
    private Database database;

    public OrderService(Database database) {
        this.database = database;
    }

    public void saveOrder(Order order) {
        database.save(order);
    }
}
```

#### Follow Law of Demeter
A class should only communicate with its immediate dependencies.

**Java Example:**
```java
// Violates Law of Demeter
public class OrderService {
    public void processOrder(Order order) {
        order.getCustomer().getAddress().getZipCode();
        // Process order
    }
}

// Follows Law of Demeter
public class OrderService {
    public void processOrder(Order order) {
        String zipCode = order.getCustomerAddressZipCode();
        // Process order
    }
}
```

### Understandability Tips

#### Be Consistent
Consistency in coding practices helps in understanding the code better.

**Java Example:**
```java
// Consistent naming
public class CustomerService {
    private Database database;

    public void addCustomer(Customer customer) {
        database.save(customer);
    }

    public void deleteCustomer(Customer customer) {
        database.delete(customer);
    }
}

// Inconsistent naming
public class CustomerService {
    private Database db;

    public void addCustomer(Customer cust) {
        db.save(cust);
    }

    public void removeCustomer(Customer c) {
        db.delete(c);
    }
}
```

#### Use Explanatory Variables
Instead of using cryptic variable names, use meaningful names.

**Java Example:**
```java
// Cryptic
int s = 0;
for (int i = 0; i < 10; i++) {
    s += i;
}

// Explanatory
int sum = 0;
for (int number = 0; number < 10; number++) {
    sum += number;
}
```

#### Encapsulate Boundary Conditions
Handle boundary conditions in a single place to avoid errors.

**Java Example:**
```java
// Scattered boundary checks
public void processOrder(Order order) {
    if (order == null) {
        throw new IllegalArgumentException("Order cannot be null");
    }
    // Process order
}

// Encapsulated boundary check
public void processOrder(Order order) {
    validateOrder(order);
    // Process order
}

private void validateOrder(Order order) {
    if (order == null) {
        throw new IllegalArgumentException("Order cannot be null");
    }
}
```

#### Prefer Dedicated Value Objects to Primitive Types
Use value objects to represent complex data rather than primitive types.

**Java Example:**
```java
// Using primitive types
public void createUser(String firstName, String lastName, int age) {
    // Create user
}

// Using value objects
public void createUser(UserName userName, Age age) {
    // Create user
}

public class UserName {
    private String firstName;
    private String lastName;
    // Constructors, getters, setters
}

public class Age {
    private int age;
    // Constructors, getters, setters
}
```

#### Avoid Logical Dependency
Ensure methods do not depend on the state set by other methods.

**Java Example:**
```java
// With logical dependency
public class OrderService {
    private boolean isOrderValid;

    public void validateOrder(Order order) {
        isOrderValid = (order != null);
    }

    public void processOrder(Order order) {
        if (isOrderValid) {
            // Process order
        }
    }
}

// Without logical dependency
public class OrderService {
    public void processOrder(Order order) {
        if (isValidOrder(order)) {
            // Process order
        }
    }

    private boolean isValidOrder(Order order) {
        return order != null;
    }
}
```

#### Avoid Negative Conditionals
Rewrite conditions positively for better readability.

**Java Example:**
```java
// Negative conditionals
if (!order.isShipped()) {
    // Process order
}

// Positive conditionals
if (order.isPending()) {
    // Process order
}
```

### Naming Rules

#### Choose Descriptive and Unambiguous Names
Names should clearly describe the purpose.

**Java Example:**
```java
// Descriptive
public void calculateInterest(double principal, double rate, int time) {
    // Calculate interest
}

// Non-descriptive
public void calcInt(double p, double r, int t) {
    // Calculate interest
}
```

#### Make Meaningful Distinctions
Differentiate names meaningfully.

**Java Example:**
```java
// Meaningful distinctions
public void setStartDate(Date startDate) {
    this.startDate =

 startDate;
}

public void setEndDate(Date endDate) {
    this.endDate = endDate;
}

// Ambiguous distinctions
public void setDate1(Date date1) {
    this.date1 = date1;
}

public void setDate2(Date date2) {
    this.date2 = date2;
}
```

#### Use Pronounceable Names
Names should be easy to read and pronounce.

**Java Example:**
```java
// Pronounceable
public void processEmployee(Employee employee) {
    // Process employee
}

// Not pronounceable
public void processEmp(Employee emp) {
    // Process employee
}
```

#### Use Searchable Names
Names should be easy to search.

**Java Example:**
```java
// Searchable
public void updateCustomer(Customer customer) {
    // Update customer
}

// Not searchable
public void updCust(Customer cust) {
    // Update customer
}
```

#### Replace Magic Numbers with Named Constants
Define constants for magic numbers.

**Java Example:**
```java
// Magic numbers
public void setAgeLimit(int age) {
    if (age < 18) {
        // Underage
    }
}

// Named constants
public static final int AGE_LIMIT = 18;

public void setAgeLimit(int age) {
    if (age < AGE_LIMIT) {
        // Underage
    }
}
```

#### Avoid Encodings
Don’t include type information or prefixes in names.

**Java Example:**
```java
// Without encoding
public class CustomerService {
    // Implementation
}

// With encoding
public class CustomerServiceClass {
    // Implementation
}
```

### Functions Rules

#### Small
Functions should be small and do one thing.

**Java Example:**
```java
// Large function
public void processOrder(Order order) {
    validateOrder(order);
    calculateTotal(order);
    saveOrder(order);
    sendConfirmation(order);
}

// Small functions
public void processOrder(Order order) {
    validateOrder(order);
    calculateTotal(order);
    saveOrder(order);
    sendConfirmation(order);
}

private void validateOrder(Order order) {
    // Validation logic
}

private void calculateTotal(Order order) {
    // Calculation logic
}

private void saveOrder(Order order) {
    // Saving logic
}

private void sendConfirmation(Order order) {
    // Sending confirmation logic
}
```

#### Do One Thing
Each function should perform a single task.

**Java Example:**
```java
// Does multiple things
public void validateAndSaveOrder(Order order) {
    validateOrder(order);
    saveOrder(order);
}

// Does one thing
public void validateOrder(Order order) {
    // Validation logic
}

public void saveOrder(Order order) {
    // Saving logic
}
```

#### Use Descriptive Names
Function names should clearly indicate their purpose.

**Java Example:**
```java
// Descriptive
public void sendEmailConfirmation(Order order) {
    // Send email confirmation
}

// Non-descriptive
public void sendEmail(Order order) {
    // Send email confirmation
}
```

#### Prefer Fewer Arguments
Limit the number of arguments a function takes.

**Java Example:**
```java
// Too many arguments
public void createUser(String firstName, String lastName, String email, String phone, String address) {
    // Create user
}

// Fewer arguments with object
public void createUser(User user) {
    // Create user
}

public class User {
    private String firstName;
    private String lastName;
    private String email;
    private String phone;
    private String address;
    // Constructors, getters, setters
}
```

#### Have No Side Effects
Functions should not change the state of the system.

**Java Example:**
```java
// With side effects
public boolean isOrderValid(Order order) {
    boolean valid = (order != null);
    if (valid) {
        // Update order status
        order.setStatus("VALID");
    }
    return valid;
}

// Without side effects
public boolean isOrderValid(Order order) {
    return order != null;
}
```

#### Don’t Use Flag Arguments
Instead of using a flag to control function behavior, split the function into multiple functions.

**Java Example:**
```java
// With flag argument
public void processOrder(Order order, boolean sendConfirmation) {
    // Process order
    if (sendConfirmation) {
        sendConfirmation(order);
    }
}

// Without flag argument
public void processOrder(Order order) {
    // Process order
}

public void processOrderWithConfirmation(Order order) {
    processOrder(order);
    sendConfirmation(order);
}
```

### Comments Rules

#### Always Try to Explain Yourself in Code
Write code that explains itself.

**Java Example:**
```java
// Self-explanatory code
public void processOrder(Order order) {
    validateOrder(order);
    calculateTotal(order);
    saveOrder(order);
}

// Comment explaining the obvious
public void processOrder(Order order) {
    // Validate the order
    validateOrder(order);
    // Calculate the total
    calculateTotal(order);
    // Save the order
    saveOrder(order);
}
```

#### Don’t Be Redundant
Avoid unnecessary comments.

**Java Example:**
```java
// Redundant comment
int age = 25; // Set age to 25

// No comment needed
int age = 25;
```

#### Don’t Add Obvious Noise
Avoid comments stating the obvious.

**Java Example:**
```java
// Obvious comment
int i = 0; // Initialize i to 0

// No comment needed
int i = 0;
```

#### Don’t Use Closing Brace Comments
Proper indentation should make the code structure clear.

**Java Example:**
```java
// With closing brace comments
if (order.isValid()) {
    processOrder(order);
} // end if

// Without closing brace comments
if (order.isValid()) {
    processOrder(order);
}
```

#### Don’t Comment Out Code. Just Remove
Remove unused code instead of commenting it out.

**Java Example:**
```java
// Commented-out code
public void processOrder(Order order) {
    // validateOrder(order);
    // calculateTotal(order);
    saveOrder(order);
}

// Remove unused code
public void processOrder(Order order) {
    saveOrder(order);
}
```

#### Use as an Explanation of Intent
Use comments to explain why something is done, not what is done.

**Java Example:**
```java
// Explain intent
public void processOrder(Order order) {
    // Validate order to ensure all required fields are filled
    validateOrder(order);
    saveOrder(order);
}
```

#### Use as Clarification of Code
Use comments to clarify complex code logic.

**Java Example:**
```java
// Clarify complex logic
public void calculateTotal(Order order) {
    double total = 0;
    for (Item item : order.getItems()) {
        total += item.getPrice() * item.getQuantity();
    }
    // Apply discount if order contains more than 10 items
    if (order.getItems().size() > 10) {
        total *= 0.9;
    }
    order.setTotal(total);
}
```

#### Use as a Warning of Consequences
Warn about potential side effects or important implications in the code.

**Java Example:**
```java
// Warning of consequences
public void deleteOrder(Order order) {
    // This will permanently remove the order from the database
    database.delete(order);
}
```

### Source Code Structure

#### Separate Concepts Vertically
Keep related code together and unrelated code separate.

**Java Example:**
```java
// Related code together
public class OrderService {
    public void processOrder(Order order) {
        validateOrder(order);
        calculateTotal(order);
        saveOrder(order);
    }

    private void validateOrder(Order order) {
        // Validation logic
    }

    private void calculateTotal(Order order) {
        // Calculation logic
    }

    private void saveOrder(Order order) {
        // Saving logic
    }
}
```

#### Related Code Should Appear Vertically Dense
Code that works together should be close together in the file.

**Java Example:**
```java
// Vertically dense related code
public class OrderService {
    public void processOrder(Order order) {
        validateOrder(order);
        calculateTotal(order);
        saveOrder(order);
    }

    private void validateOrder(Order order) {
        // Validation logic
    }

    private void calculateTotal(Order order) {
        // Calculation logic
    }

    private void saveOrder(Order order) {
        // Saving logic
    }
}
```

#### Declare Variables Close to Their Usage
Declare variables near where they are used to make the code easier to read.

**Java Example:**
```java
// Declare variables close to usage
public void calculateTotal(Order order) {
    double total = 0;
    for (Item item : order.getItems()) {
        double itemTotal = item.getPrice() * item.getQuantity();
        total += itemTotal;
    }
    order.setTotal(total);
}
```

#### Dependent Functions Should Be Close
Functions that depend on each other should be placed close to each other.

**Java Example:**
```java
// Dependent functions close together
public class OrderService {
    public void processOrder(Order order) {
        validateOrder(order);
        saveOrder(order);
    }

    private void validateOrder(Order order) {
        // Validation logic
    }

    private void saveOrder(Order order) {
        // Saving logic
    }
}
```

#### Similar Functions Should Be Close
Functions that perform similar tasks should be placed together.

**Java Example:**
```java
// Similar functions together
public class OrderService {
    public void saveOrder(Order order) {
        // Saving logic
    }

    public void deleteOrder(Order order) {
        // Deleting logic
    }
}


```

#### Place Functions in the Downward Direction
Call functions should appear above callee functions.

**Java Example:**
```java
// Downward direction
public class OrderService {
    public void processOrder(Order order) {
        validateOrder(order);
    }

    private void validateOrder(Order order) {
        // Validation logic
    }
}
```

#### Keep Lines Short
Avoid long lines of code that are hard to read.

**Java Example:**
```java
// Short lines
public void processOrder(Order order) {
    if (order == null || order.getItems().isEmpty()) {
        throw new IllegalArgumentException("Invalid order");
    }
    // Process order
}
```

#### Don’t Use Horizontal Alignment
Avoid aligning code horizontally for readability.

**Java Example:**
```java
// Without horizontal alignment
public class OrderService {
    public void processOrder(Order order) {
        // Process order
    }

    public void validateOrder(Order order) {
        // Validate order
    }

    public void saveOrder(Order order) {
        // Save order
    }
}
```

#### Use White Space to Associate Related Things and Disassociate Weakly Related
Use white space to make the code easier to read.

**Java Example:**
```java
// Using white space
public class OrderService {
    public void processOrder(Order order) {
        validateOrder(order);
        calculateTotal(order);
        saveOrder(order);
    }

    private void validateOrder(Order order) {
        // Validation logic
    }

    private void calculateTotal(Order order) {
        // Calculation logic
    }

    private void saveOrder(Order order) {
        // Saving logic
    }
}
```

#### Don’t Break Indentation
Keep consistent indentation to maintain readability.

**Java Example:**
```java
// Consistent indentation
public class OrderService {
    public void processOrder(Order order) {
        if (order == null) {
            throw new IllegalArgumentException("Order cannot be null");
        }
        // Process order
    }
}
```

### Objects and Data Structures

#### Hide Internal Structure
Encapsulate internal data and expose only necessary methods.

**Java Example:**
```java
// Hide internal structure
public class Order {
    private List<Item> items;

    public List<Item> getItems() {
        return new ArrayList<>(items);
    }
}

// Expose internal structure
public class Order {
    public List<Item> items;
}
```

#### Prefer Data Structures
Use data structures instead of complex objects when appropriate.

**Java Example:**
```java
// Using data structures
public class Order {
    private List<Item> items;

    public List<Item> getItems() {
        return new ArrayList<>(items);
    }
}

// Using complex objects
public class Order {
    private List<Item> items;

    public List<Item> getItems() {
        return items;
    }

    public void addItem(Item item) {
        items.add(item);
    }

    public void removeItem(Item item) {
        items.remove(item);
    }
}
```

#### Avoid Hybrid Structures (Half Object and Half Data)
Don’t mix object-oriented and data-oriented styles.

**Java Example:**
```java
// Hybrid structure
public class Order {
    public List<Item> items;

    public void addItem(Item item) {
        items.add(item);
    }

    public void removeItem(Item item) {
        items.remove(item);
    }
}

// Pure data structure
public class Order {
    private List<Item> items;

    public List<Item> getItems() {
        return new ArrayList<>(items);
    }
}
```

#### Should Be Small
Classes should be small and focused.

**Java Example:**
```java
// Small class
public class OrderService {
    public void processOrder(Order order) {
        validateOrder(order);
        saveOrder(order);
    }
}

// Large class
public class OrderService {
    public void processOrder(Order order) {
        validateOrder(order);
        saveOrder(order);
        sendEmailConfirmation(order);
        updateInventory(order);
    }
}
```

#### Do One Thing
Classes should have a single responsibility.

**Java Example:**
```java
// Single responsibility
public class OrderValidator {
    public void validateOrder(Order order) {
        // Validation logic
    }
}

public class OrderSaver {
    public void saveOrder(Order order) {
        // Saving logic
    }
}

// Multiple responsibilities
public class OrderService {
    public void processOrder(Order order) {
        validateOrder(order);
        saveOrder(order);
    }

    private void validateOrder(Order order) {
        // Validation logic
    }

    private void saveOrder(Order order) {
        // Saving logic
    }
}
```

#### A Small Number of Instance Variables
Keep the number of instance variables low.

**Java Example:**
```java
// Few instance variables
public class Order {
    private List<Item> items;
    private Customer customer;
}

// Many instance variables
public class Order {
    private List<Item> items;
    private Customer customer;
    private Date orderDate;
    private double totalAmount;
    private String status;
}
```

#### Base Class Should Know Nothing About Their Derivatives
Base classes should be unaware of their subclasses.

**Java Example:**
```java
// Base class unaware of derivatives
public abstract class Animal {
    public abstract void makeSound();
}

public class Dog extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Woof");
    }
}

public class Cat extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Meow");
    }
}
```

#### Better to Have Many Functions Than to Pass Some Code into a Function to Select a Behavior
Prefer multiple functions over passing behavior as parameters.

**Java Example:**
```java
// Passing behavior as parameter
public void processOrder(Order order, boolean sendConfirmation) {
    if (sendConfirmation) {
        sendConfirmation(order);
    }
}

// Multiple functions
public void processOrder(Order order) {
    // Process order
}

public void processOrderWithConfirmation(Order order) {
    processOrder(order);
    sendConfirmation(order);
}
```

#### Prefer Non-Static Methods to Static Methods
Use instance methods over static methods where appropriate.

**Java Example:**
```java
// Instance methods
public class OrderService {
    public void processOrder(Order order) {
        // Process order
    }
}

// Static methods
public class OrderService {
    public static void processOrder(Order order) {
        // Process order
    }
}
```

### Tests

#### One Assert Per Test
Ensure each test checks a single condition.

**Java Example:**
```java
// One assert per test
public class OrderServiceTest {
    @Test
    public void testProcessOrder() {
        Order order = new Order();
        orderService.processOrder(order);
        assertTrue(order.isProcessed());
    }
}

// Multiple asserts per test
public class OrderServiceTest {
    @Test
    public void testProcessOrder() {
        Order order = new Order();
        orderService.processOrder(order);
        assertTrue(order.isProcessed());
        assertEquals("PROCESSED", order.getStatus());
    }
}
```

#### Readable
Write tests that are easy to read and understand.

**Java Example:**
```java
// Readable test
public class OrderServiceTest {
    @Test
    public void testProcessOrder() {
        Order order = new Order();
        orderService.processOrder(order);
        assertTrue(order.isProcessed());
    }
}

// Unreadable test
public class OrderServiceTest {
    @Test
    public void t1() {
        Order o = new Order();
        os.processOrder(o);
        assertTrue(o.isProcessed());
    }
}
```

#### Fast
Tests should run quickly.

**Java Example:**
```java
// Fast test
public class OrderServiceTest {
    @Test
    public void testProcessOrder() {
        Order order = new Order();
        orderService.processOrder(order);
        assertTrue(order.isProcessed());
    }
}

// Slow test
public class OrderServiceTest {
    @Test
    public void testProcessOrder() {
        Order order = new Order();
        orderService.processOrder(order);
        Thread.sleep(1000); // Simulate delay
        assertTrue(order.isProcessed());
    }
}
```

#### Independent
Tests should not depend on each other.

**Java Example:**
```java
// Independent tests
public class OrderServiceTest {
    @Test
    public void testProcessOrder() {
        Order order = new Order();
        orderService.processOrder(order);
        assertTrue(order.isProcessed());
    }

    @Test
    public void testCancelOrder() {
        Order order = new Order();
        orderService.cancelOrder(order);
        assertFalse(order.isProcessed());
    }
}

// Dependent tests
public class OrderServiceTest {
    @Test
    public void testProcessOrder() {
        Order order = new Order();
        orderService.processOrder(order);
        assertTrue(order.isProcessed());
    }

    @Test
    public void testCancelOrder() {
        testProcessOrder();
        Order order = new Order();
        orderService.cancelOrder(order);
        assertFalse(order.isProcessed());
    }
}
```

#### Repeatable
Tests should yield the same result every time they run.

**Java Example:**
```java
// Repeatable test
public class OrderServiceTest {
    @Test
    public void testProcessOrder() {
        Order order = new Order();
        orderService.processOrder(order);
        assertTrue(order.isProcessed());
    }
}

// Non-repeatable test
public class OrderServiceTest {
    @Test
    public void testProcessOrder() {
        Order order = new Order();
        orderService.processOrder(order);
        assertTrue(order.isProcessed());
    }

    @Test
    public void testProcessOrderAgain() {
        Order order = new Order();
       

 orderService.processOrder(order);
        orderService.processOrder(order);
        assertTrue(order.isProcessed());
    }
}
```

#### Given, When, Then
Follow the Given-When-Then format for writing tests.

**Java Example:**
```java
// Given-When-Then format
public class OrderServiceTest {
    @Test
    public void testProcessOrder() {
        // Given
        Order order = new Order();

        // When
        orderService.processOrder(order);

        // Then
        assertTrue(order.isProcessed());
    }
}
```

## Conclusion

These clean code practices help improve code readability, maintainability, and reduce errors. By following these principles, developers can create code that is easier to understand, modify, and test.