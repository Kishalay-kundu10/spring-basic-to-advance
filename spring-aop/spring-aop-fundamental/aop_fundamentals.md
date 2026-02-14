# Fundamentals of Spring AOP

This document covers the fundamental concepts you need to understand Spring AOP: terminology, comparison with AspectJ, and proxy mechanisms.

---

## Table of Contents
- [AOP Terminology](#aop-terminology)
- [Spring AOP vs AspectJ](#spring-aop-vs-aspectj)
- [AOP Proxies](#aop-proxies)

---

# AOP Terminology

## 📖 Definition

AOP has its own terminology that describes the key concepts of the paradigm. Understanding these terms is essential for working with Spring AOP effectively. These terms form the vocabulary used to discuss and implement aspect-oriented solutions.

---

## 🎯 Core AOP Terms

### 1. Aspect

**Definition:** A modularization of a concern that cuts across multiple classes. It's a class annotated with `@Aspect` that contains advice and pointcuts.

**Think of it as:** A module that encapsulates cross-cutting behavior (like logging, security).

```java
@Aspect
@Component
public class LoggingAspect {
    // This entire class is an Aspect
    // It contains advice methods and pointcuts
}
```

---

### 2. Join Point

**Definition:** A point during the execution of a program, such as method execution, exception handling, or field access. In Spring AOP, a join point always represents a method execution.

**Think of it as:** Any place in your code where an aspect can be applied.

```java
public class UserService {
    
    public User getUser(Long id) {  // <- Join Point (method execution)
        return userRepository.findById(id);
    }
    
    public void deleteUser(Long id) {  // <- Join Point (method execution)
        userRepository.deleteById(id);
    }
}
```

---

### 3. Advice

**Definition:** Action taken by an aspect at a particular join point. It's the code that runs when a pointcut matches.

**Types of Advice:**
- **@Before** - Runs before the method
- **@After** - Runs after the method (finally)
- **@AfterReturning** - Runs after successful method completion
- **@AfterThrowing** - Runs if method throws exception
- **@Around** - Wraps the method execution

**Think of it as:** The actual code that executes when your aspect is triggered.

```java
@Aspect
@Component
public class SecurityAspect {
    
    // This method is an "advice"
    @Before("execution(* com.example.service.*.*(..))")
    public void checkSecurity(JoinPoint joinPoint) {
        System.out.println("Security check for: " + joinPoint.getSignature());
    }
}
```

---

### 4. Pointcut

**Definition:** A predicate that matches join points. It's an expression that selects which methods the advice should apply to.

**Think of it as:** A filter that decides where your aspect applies.

```java
@Aspect
@Component
public class LoggingAspect {
    
    // This is a Pointcut expression
    @Before("execution(* com.example.service.*.*(..))") 
    //       ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    //       This expression is the Pointcut
    public void logBefore(JoinPoint joinPoint) {
        System.out.println("Executing: " + joinPoint.getSignature());
    }
}
```

**Common Pointcut Expressions:**
```java
// All methods in service package
execution(* com.example.service.*.*(..))

// All methods starting with 'get'
execution(* get*(..))

// All methods with @Transactional annotation
@annotation(org.springframework.transaction.annotation.Transactional)

// All methods in classes annotated with @Service
@within(org.springframework.stereotype.Service)
```

---

### 5. Target Object

**Definition:** The object being advised by one or more aspects. Also called the "advised object."

**Think of it as:** The actual object whose methods are being intercepted.

```java
@Service
public class UserService {  // <- This is the Target Object
    public User getUser(Long id) {
        return userRepository.findById(id);
    }
}
```

---

### 6. AOP Proxy

**Definition:** An object created by the AOP framework to implement the aspect contracts. In Spring AOP, it's either a JDK dynamic proxy or a CGLIB proxy.

**Think of it as:** A wrapper around your target object that intercepts method calls.

```
Client -> Proxy (with aspects) -> Target Object

When you call:     userService.getUser(1)
Actually calling:  Proxy.getUser() -> Logging Aspect -> Target.getUser()
```

---

### 7. Weaving

**Definition:** The process of linking aspects with other application types or objects to create an advised object. 

**When it happens:**
- **Compile-time** - AspectJ compiler
- **Load-time** - AspectJ load-time weaving
- **Runtime** - Spring AOP (creates proxies at runtime)

**Think of it as:** The process of applying aspects to your code.

---

### 8. Introduction

**Definition:** Declaring additional methods or fields on behalf of a type. Allows you to add new interfaces (and implementations) to any advised object.

**Think of it as:** Adding new methods to existing classes without modifying them.

```java
@Aspect
@Component
public class IntroductionAspect {
    
    @DeclareParents(
        value = "com.example.service.*+",
        defaultImpl = UsageTrackedImpl.class
    )
    public static UsageTracked mixin;
}

// Now all service classes automatically have UsageTracked methods!
```

---

## 💡 Visual Representation

```
┌─────────────────────────────────────────────────────────────┐
│                         ASPECT                               │
│  ┌────────────────────────────────────────────────────────┐ │
│  │                     ADVICE                              │ │
│  │  (What to do: logging, security check, etc.)           │ │
│  └────────────────────────────────────────────────────────┘ │
│                           +                                  │
│  ┌────────────────────────────────────────────────────────┐ │
│  │                    POINTCUT                             │ │
│  │  (Where to do it: which methods to intercept)          │ │
│  └────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
                            ↓
                    Applied to
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                    TARGET OBJECT                             │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐           │
│  │ JOIN POINT │  │ JOIN POINT │  │ JOIN POINT │           │
│  │ (method1)  │  │ (method2)  │  │ (method3)  │           │
│  └────────────┘  └────────────┘  └────────────┘           │
└─────────────────────────────────────────────────────────────┘
                            ↓
                    Accessed through
                            ↓
                    ┌─────────────┐
                    │  AOP PROXY  │
                    └─────────────┘
```

---

## 📝 Complete Example with All Terms

```java
// ASPECT: Module containing cross-cutting concerns
@Aspect
@Component
public class PerformanceAspect {
    
    // POINTCUT: Named reusable pointcut expression
    @Pointcut("execution(* com.example.service.*.*(..))")
    public void serviceMethods() {}
    
    // ADVICE: Code that runs at join points
    @Around("serviceMethods()")  // Uses the pointcut
    public Object measurePerformance(ProceedingJoinPoint joinPoint) throws Throwable {
        // JoinPoint parameter gives access to join point information
        
        long start = System.currentTimeMillis();
        
        // Proceed to the TARGET OBJECT method (join point)
        Object result = joinPoint.proceed();
        
        long executionTime = System.currentTimeMillis() - start;
        
        System.out.println(joinPoint.getSignature() + " executed in " + executionTime + "ms");
        
        return result;
    }
}

// TARGET OBJECT: The object being advised
@Service
public class UserService {
    
    // JOIN POINT: Method execution that can be intercepted
    public User getUserById(Long id) {
        return userRepository.findById(id);
    }
    
    // Another JOIN POINT
    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }
}

// When Spring creates UserService bean, it actually creates:
// AOP PROXY -> wraps -> TARGET OBJECT (UserService)
```

---

## 🎓 Interview Questions

**Q1: Explain the relationship between Aspect, Advice, and Pointcut.**

**Answer:**
- **Aspect** = Container (the class with @Aspect)
- **Pointcut** = Filter (which methods to intercept)
- **Advice** = Action (what to do at those methods)

**Relationship:**
```java
@Aspect  // This is an Aspect
@Component
public class LoggingAspect {
    
    // Advice = What to do (log method execution)
    // Pointcut = Where to do it (all service methods)
    @Before("execution(* com.example.service.*.*(..))")
    //      ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    //      This is the Pointcut expression
    public void logMethodExecution(JoinPoint joinPoint) {
        // This method body is the Advice
        System.out.println("Executing: " + joinPoint.getSignature().getName());
    }
}
```

**Think of it as:**
- **Aspect** is a module (like a service class)
- **Pointcut** is a condition (like an if statement deciding which methods)
- **Advice** is the behavior (like a method that gets executed)

---

**Q2: What is the difference between Target Object and Proxy in Spring AOP?**

**Answer:**
- **Target Object**: Your actual business object (e.g., UserService)
- **Proxy**: A wrapper created by Spring that intercepts calls to the target

**How it works:**
```java
// Target Object - your actual class
@Service
public class UserService {
    public User getUser(Long id) {
        return userRepository.findById(id);
    }
}

// At runtime, Spring creates:
Client Code -> Proxy (with aspects) -> Target Object

// When you call:
userService.getUser(1);

// Actually executes:
1. Proxy intercepts the call
2. Proxy executes @Before advice
3. Proxy calls target.getUser(1)
4. Target executes business logic
5. Proxy executes @After advice
6. Proxy returns result to client
```

**Key Point:** When you inject `UserService`, you actually get a proxy, not the real object. The proxy delegates to the real object after applying aspects.

---

## ⚠️ Common Pitfalls

**PITFALL 1: Confusing Join Point with Pointcut**

**Problem:**
```java
// WRONG understanding:
"Join Point is the expression that selects methods"
```

**Correction:**
- **Join Point** = An actual point in execution (like a specific method call)
- **Pointcut** = An expression that selects which join points to advise

```java
// Join Points (actual method executions in your code)
public void getUserById(Long id) { }  // <- Join Point 1
public void deleteUser(Long id) { }   // <- Join Point 2

// Pointcut (expression selecting which join points)
@Before("execution(* com.example.service.*.*(..))")
//      ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
//      This expression (pointcut) selects both join points above
```

---

**PITFALL 2: Thinking Target Object and Proxy are the same**

**Problem:**
```java
@Service
public class UserService {
    @Autowired
    private UserService userService; // Gets the proxy
    
    public void method1() {
        this.method2();  // Internal call - bypasses proxy!
    }
    
    @Transactional
    public void method2() {
        // Transaction won't work when called via this.method2()
    }
}
```

**Why it fails:**
- `this` refers to the target object, not the proxy
- Internal calls don't go through the proxy
- Aspects don't apply to internal calls

**Solution:**
```java
@Service
public class UserService {
    @Autowired
    private UserService self; // Inject self (gets proxy)
    
    public void method1() {
        self.method2();  // Goes through proxy - aspects work!
    }
    
    @Transactional
    public void method2() {
        // Transaction works correctly now
    }
}
```

---

# Spring AOP vs AspectJ

## 📖 Definition

**Spring AOP** and **AspectJ** are both frameworks for implementing Aspect-Oriented Programming, but they differ significantly in their implementation approach, capabilities, and use cases. Spring AOP is a proxy-based, Spring-container-dependent solution focused on method execution, while AspectJ is a full-featured AOP framework that can weave aspects at compile-time, load-time, or runtime, and supports field access, constructor calls, and more.

---

## 🎯 Detailed Comparison

### Key Differences Table

| Feature | Spring AOP | AspectJ |
|---------|------------|---------|
| **Implementation** | Proxy-based (JDK/CGLIB) | Bytecode weaving |
| **Weaving Time** | Runtime only | Compile-time, Post-compile, Load-time |
| **Join Points** | Method execution only | Method, constructor, field access, etc. |
| **Performance** | Slower (proxy overhead) | Faster (direct bytecode) |
| **Complexity** | Simple, easy to use | More complex, powerful |
| **Dependencies** | Requires Spring container | Standalone, no Spring needed |
| **Target** | Spring beans only | Any Java object |
| **Learning Curve** | Easy | Steeper |

---

## 💡 Spring AOP

### How It Works

Spring AOP creates proxies around your beans:

```
Client -> Proxy (contains aspects) -> Target Object

@Service
public class UserService { }

// Spring creates:
UserServiceProxy -> wraps -> UserService instance
```

### Characteristics

✅ **Pros:**
- Easy to use with Spring applications
- No special compiler needed
- Works with existing Spring projects
- Good for most common use cases

❌ **Cons:**
- Only method-level interception
- Only works on Spring beans
- Cannot intercept internal method calls
- Performance overhead from proxies
- Cannot advise final methods/classes (with JDK proxy)

### Example

```java
@Aspect
@Component
public class LoggingAspect {
    
    // Spring AOP - Only method execution
    @Before("execution(* com.example.service.*.*(..))")
    public void logMethod(JoinPoint joinPoint) {
        System.out.println("Method called: " + joinPoint.getSignature());
    }
}
```

---

## 💡 AspectJ

### How It Works

AspectJ modifies bytecode directly:

```
Source Code -> AspectJ Compiler -> Modified Bytecode (with aspects woven in)

OR

Regular Bytecode -> AspectJ Load-Time Weaver -> Modified Bytecode
```

### Characteristics

✅ **Pros:**
- Full AOP support (fields, constructors, static methods)
- Better performance (no proxy overhead)
- Works on any Java object
- Can intercept internal method calls
- More powerful pointcut expressions

❌ **Cons:**
- Requires AspectJ compiler or load-time weaver
- More complex setup
- Steeper learning curve
- Compilation takes longer

### Example

```java
@Aspect
public class AspectJExample {
    
    // Can intercept field access (not possible in Spring AOP)
    @Before("get(* com.example.model.User.email)")
    public void beforeFieldAccess(JoinPoint joinPoint) {
        System.out.println("Accessing email field");
    }
    
    // Can intercept constructor calls (not possible in Spring AOP)
    @Before("call(com.example.model.User.new(..))")
    public void beforeConstructor(JoinPoint joinPoint) {
        System.out.println("Creating new User");
    }
}
```

---

## 📝 Side-by-Side Code Comparison

### Spring AOP Example (Proxy-Based)

```java
// Configuration
@Configuration
@EnableAspectJAutoProxy  // Enable Spring AOP
public class AopConfig {
}

// Aspect
@Aspect
@Component
public class SpringAopExample {
    
    // Only method execution join points
    @Around("execution(* com.example.service.*.*(..))")
    public Object logAround(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("Before method: " + pjp.getSignature());
        Object result = pjp.proceed();
        System.out.println("After method");
        return result;
    }
}

// Must be a Spring bean
@Service
public class UserService {
    public User getUser(Long id) {
        return userRepository.findById(id);
    }
}
```

### AspectJ Example (Bytecode Weaving)

```java
// No Spring needed - standalone AspectJ
@Aspect
public class AspectJExample {
    
    // Can intercept anything
    @Around("execution(* com.example.service.*.*(..))")
    public Object logAround(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("Before method: " + pjp.getSignature());
        Object result = pjp.proceed();
        System.out.println("After method");
        return result;
    }
    
    // Can intercept field access (Spring AOP cannot)
    @Before("set(* com.example.User.email)")
    public void beforeEmailSet(JoinPoint jp) {
        System.out.println("Setting email field");
    }
    
    // Can intercept constructors (Spring AOP cannot)
    @Before("execution(com.example.User.new(..))")
    public void beforeUserCreation() {
        System.out.println("Creating User object");
    }
}

// Doesn't need to be a Spring bean
public class UserService {
    public User getUser(Long id) {
        return userRepository.findById(id);
    }
}
```

---

## 🎓 Interview Questions

**Q1: When would you choose Spring AOP over AspectJ?**

**Answer:**
**Choose Spring AOP when:**
- You're already using Spring Framework
- You only need method-level interception
- You want simple setup without special compilers
- You're advising Spring beans
- Performance overhead is acceptable
- You need quick implementation

**Example use cases:**
```java
// Logging method calls - Spring AOP is perfect
@Before("execution(* com.example.service.*.*(..))")
public void logMethod(JoinPoint jp) {
    log.info("Method: " + jp.getSignature());
}

// Transaction management - Spring AOP works well
@Around("@annotation(transactional)")
public Object handleTransaction(ProceedingJoinPoint pjp) {
    // Transaction logic
}
```

**Choose AspectJ when:**
- Need field-level interception
- Need constructor interception
- Need to advise non-Spring objects
- Performance is critical
- Need to intercept internal method calls

**Example use cases:**
```java
// Field access logging - requires AspectJ
@Before("get(* com.example.User.password)")
public void logPasswordAccess() {
    log.warn("Password field accessed!");
}

// Constructor interception - requires AspectJ
@Before("execution(com.example.User.new(..))")
public void validateUserCreation() {
    // Validate before object creation
}
```

---

**Q2: What are the limitations of Spring AOP compared to AspectJ?**

**Answer:**
**Spring AOP Limitations:**

1. **Only Method Execution Join Points**
```java
// Spring AOP - CAN do
@Before("execution(* getUserById(..))")

// Spring AOP - CANNOT do
@Before("get(* User.email)")        // Field access
@Before("call(User.new(..))")       // Constructor
@Before("staticinitialization(User)") // Static initialization
```

2. **Only Spring Beans**
```java
// Spring AOP - Works
@Service
public class UserService { }  // Managed by Spring

// Spring AOP - Doesn't work
public class RegularClass { }  // Not a Spring bean
```

3. **Cannot Intercept Internal Calls**
```java
@Service
public class UserService {
    public void method1() {
        this.method2();  // Spring AOP won't intercept this!
    }
    
    @Transactional
    public void method2() {
        // Transaction won't work when called internally
    }
}
```

4. **Cannot Advise Final Methods/Classes**
```java
// Spring AOP with JDK Proxy - Cannot advise
public final class FinalClass { }  // Cannot proxy

public class UserService {
    public final void finalMethod() { }  // Cannot proxy
}
```

**AspectJ has none of these limitations** but requires more complex setup.

---

## ⚠️ Common Pitfalls

**PITFALL 1: Using Spring AOP annotations with AspectJ expectations**

**Problem:**
```java
@Aspect
@Component
public class MyAspect {
    
    // This WON'T work with Spring AOP - field access not supported
    @Before("get(* com.example.User.password)")
    public void logPasswordAccess() {
        System.out.println("Password accessed");
    }
}
```

**Why it fails:**
- Spring AOP only supports method execution
- Field access requires AspectJ

**Solution:**
Either switch to AspectJ or use method-level interception:
```java
// Option 1: Use AspectJ (requires AspectJ weaver)

// Option 2: Intercept getter method instead (Spring AOP)
@Before("execution(* getPassword())")
public void logPasswordAccess() {
    System.out.println("Password accessed via getter");
}
```

---

**PITFALL 2: Expecting Spring AOP to work without Spring container**

**Problem:**
```java
// Creating object manually
UserService service = new UserService();
service.getUser(1L);  // Spring AOP won't work!
```

**Why it fails:**
- Spring AOP requires Spring to create proxies
- Manually created objects aren't proxied

**Solution:**
```java
// Let Spring create the bean
@Service
public class UserService {
    public User getUser(Long id) {
        return userRepository.findById(id);
    }
}

// Inject it (Spring creates proxy automatically)
@RestController
public class UserController {
    @Autowired  // Gets proxy, not direct instance
    private UserService userService;
    
    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.getUser(id);  // Goes through proxy
    }
}
```

---

# AOP Proxies

## 📖 Definition

**AOP Proxies** are objects created by Spring to implement aspect behavior. A proxy is a wrapper around your target object that intercepts method calls, applies advice, and then delegates to the real object. Spring AOP uses two types of proxies: **JDK Dynamic Proxies** (interface-based) and **CGLIB Proxies** (class-based).

---

## 🎯 How Proxies Work

### The Proxy Pattern

```
                 ┌──────────────┐
                 │    Client    │
                 └───────┬──────┘
                         │ calls method
                         ▼
                 ┌──────────────┐
                 │   Proxy      │ (created by Spring)
                 │              │
                 │ - Intercepts │
                 │ - Applies    │
                 │   Aspects    │
                 │ - Delegates  │
                 └───────┬──────┘
                         │ delegates to
                         ▼
                 ┌──────────────┐
                 │Target Object │ (your actual object)
                 │              │
                 │ Business     │
                 │ Logic        │
                 └──────────────┘
```

---

## 💡 JDK Dynamic Proxies

### When Used
- Target object implements **at least one interface**
- Default choice when interfaces are present

### How It Works
```java
// Your interface
public interface UserService {
    User getUser(Long id);
    void deleteUser(Long id);
}

// Your implementation
@Service
public class UserServiceImpl implements UserService {
    public User getUser(Long id) {
        return userRepository.findById(id);
    }
    
    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }
}

// Spring creates a JDK Dynamic Proxy that implements UserService
// Proxy implements same interface -> intercepts -> delegates to target
```

### Characteristics

✅ **Advantages:**
- Built into JDK (no external dependencies)
- Standard Java mechanism
- Lightweight

❌ **Limitations:**
- **Requires interface** - cannot proxy classes without interfaces
- Can only proxy methods defined in interface
- Cannot proxy final methods

---

## 💡 CGLIB Proxies

### When Used
- Target object **doesn't implement any interface**
- Explicitly configured to use CGLIB even with interfaces
- Enabled with `@EnableAspectJAutoProxy(proxyTargetClass = true)`

### How It Works
```java
// No interface - just a class
@Service
public class UserService {
    public User getUser(Long id) {
        return userRepository.findById(id);
    }
    
    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }
}

// Spring creates a CGLIB proxy by creating a subclass
// UserService$$EnhancedBySpringCGLIB extends UserService
```

### Characteristics

✅ **Advantages:**
- Works without interfaces
- Can proxy concrete classes
- More flexible

❌ **Limitations:**
- **Cannot proxy final classes** - cannot extend final class
- **Cannot proxy final methods** - cannot override final methods
- **Cannot proxy private methods** - not visible to subclass
- Slightly slower than JDK proxies
- Requires CGLIB library (included in Spring)

---

## 📝 Code Examples

### JDK Dynamic Proxy Example

```java
// 1. Define interface
public interface PaymentService {
    void processPayment(BigDecimal amount);
    void refundPayment(Long paymentId);
}

// 2. Implementation
@Service
public class PaymentServiceImpl implements PaymentService {
    
    @Override
    public void processPayment(BigDecimal amount) {
        System.out.println("Processing payment: " + amount);
    }
    
    @Override
    public void refundPayment(Long paymentId) {
        System.out.println("Refunding payment: " + paymentId);
    }
}

// 3. Aspect
@Aspect
@Component
public class LoggingAspect {
    @Before("execution(* com.example.service.PaymentService.*(..))")
    public void logBefore(JoinPoint jp) {
        System.out.println("Before: " + jp.getSignature());
    }
}

// 4. Usage
@RestController
public class PaymentController {
    @Autowired
    private PaymentService paymentService;  // Actually gets a JDK proxy
    
    @PostMapping("/pay")
    public void pay(@RequestParam BigDecimal amount) {
        paymentService.processPayment(amount);  // Goes through proxy
    }
}
```

### CGLIB Proxy Example

```java
// 1. No interface - just a class
@Service
public class NotificationService {
    
    public void sendEmail(String to, String message) {
        System.out.println("Sending email to: " + to);
    }
    
    public void sendSMS(String phone, String message) {
        System.out.println("Sending SMS to: " + phone);
    }
}

// 2. Aspect
@Aspect
@Component
public class LoggingAspect {
    @Before("execution(* com.example.service.NotificationService.*(..))")
    public void logBefore(JoinPoint jp) {
        System.out.println("Before: " + jp.getSignature());
    }
}

// 3. Usage
@RestController
public class NotificationController {
    @Autowired
    private NotificationService notificationService;  // Gets CGLIB proxy
    
    @PostMapping("/notify")
    public void notify(@RequestParam String email) {
        notificationService.sendEmail(email, "Hello");  // Through proxy
    }
}
```

### Forcing CGLIB Proxies

```java
// Force CGLIB even when interfaces exist
@Configuration
@EnableAspectJAutoProxy(proxyTargetClass = true)  // Force CGLIB
public class AopConfig {
}
```

---

## 🎓 Interview Questions

**Q1: What is the difference between JDK Dynamic Proxy and CGLIB Proxy?**

**Answer:**

| Feature | JDK Dynamic Proxy | CGLIB Proxy |
|---------|-------------------|-------------|
| **Requirement** | Requires interface | Works with classes |
| **Mechanism** | Implements interface | Extends class (subclass) |
| **Performance** | Slightly faster | Slightly slower |
| **Limitations** | Needs interface | Cannot proxy final classes/methods |
| **When used** | Target implements interface | No interface or explicitly configured |

**Example:**

```java
// JDK Dynamic Proxy scenario
public interface UserService { }
@Service
public class UserServiceImpl implements UserService { }
// Spring creates: Proxy implements UserService -> delegates to UserServiceImpl

// CGLIB Proxy scenario
@Service
public class OrderService { }  // No interface
// Spring creates: OrderService$$CGLIB extends OrderService
```

**Key Difference:**
- JDK proxy **implements** the same interface
- CGLIB proxy **extends** the target class

---

**Q2: Why can't CGLIB proxy final classes or methods?**

**Answer:**
CGLIB creates proxies by **creating a subclass** of your target class. In Java:
- You cannot extend a `final` class
- You cannot override a `final` method

```java
// This WON'T work with CGLIB
@Service
public final class FinalService {  // Cannot be subclassed
    public void doSomething() { }
}

// This partially works with CGLIB
@Service
public class UserService {
    
    public void method1() { }  // Can be proxied
    
    public final void method2() { }  // CANNOT be proxied (cannot override)
    
    private void method3() { }  // CANNOT be proxied (not visible)
}
```

**Why it matters:**
- Aspects won't apply to final methods
- Can lead to unexpected behavior

**Solution:**
```java
// Remove final keyword
@Service
public class UserService {
    public void method2() { }  // Now can be proxied
}

// OR use interface with JDK proxy
public interface UserService {
    void method2();
}

@Service
public class UserServiceImpl implements UserService {
    @Override
    public void method2() { }  // JDK proxy works with final
}
```

---

## ⚠️ Common Pitfalls

**PITFALL 1: Internal method calls bypass proxy**

**Problem:**
```java
@Service
public class UserService {
    
    public void method1() {
        this.method2();  // Internal call - bypasses proxy!
    }
    
    @Transactional
    public void method2() {
        // Transaction won't work!
    }
}
```

**Why it fails:**
```
Client -> Proxy -> method1()
                    └─> this.method2()  // 'this' = target, not proxy!
```

**Solution:**
```java
// Option 1: Self-injection
@Service
public class UserService {
    @Autowired
    private UserService self;  // Gets proxy
    
    public void method1() {
        self.method2();  // Goes through proxy - works!
    }
    
    @Transactional
    public void method2() { }
}

// Option 2: Separate beans (BETTER)
@Service
public class UserService {
    @Autowired
    private TransactionalService transactionalService;
    
    public void method1() {
        transactionalService.method2();  // Through proxy
    }
}

@Service
public class TransactionalService {
    @Transactional
    public void method2() { }
}
```

---

**PITFALL 2: Casting to implementation class**

**Problem:**
```java
@Service
public class UserServiceImpl implements UserService {
    public void specialMethod() { }  // Not in interface
}

@RestController
public class UserController {
    @Autowired
    private UserService userService;
    
    public void doSomething() {
        // Trying to cast to implementation
        UserServiceImpl impl = (UserServiceImpl) userService;  // ClassCastException!
        impl.specialMethod();
    }
}
```

**Why it fails:**
With JDK proxy, `userService` is actually a proxy that implements `UserService`, not `UserServiceImpl`.

**Solution:**
```java
// Option 1: Add method to interface
public interface UserService {
    void specialMethod();
}

// Option 2: Inject implementation directly (forces CGLIB)
@Autowired
private UserServiceImpl userServiceImpl;

// Option 3: Use CGLIB proxies globally
@EnableAspectJAutoProxy(proxyTargetClass = true)
```

---

## 🔗 Related Topics

- [What is AOP?](#what-is-aop-aspect-oriented-programming) - AOP basics
- [AOP Terminology](#aop-terminology) - Core concepts
- [Join Points](../02-core-concepts/01-joinpoints.md) - Interception points

---

## 📚 Key Takeaways

### AOP Terminology
✅ **Aspect** = Module containing advice and pointcuts  
✅ **Join Point** = Point in execution (method call)  
✅ **Advice** = Code that runs at join points  
✅ **Pointcut** = Expression selecting join points  
✅ **Target** = Object being advised  
✅ **Proxy** = Wrapper created by Spring

### Spring AOP vs AspectJ
✅ **Spring AOP** = Proxy-based, runtime weaving, method-only, Spring beans  
✅ **AspectJ** = Bytecode weaving, compile/load-time, full AOP support  
✅ **Use Spring AOP** for most common cases (logging, transactions)  
✅ **Use AspectJ** when needing field access, constructors, or non-Spring objects

### AOP Proxies
✅ **JDK Proxy** = Interface-based, implements interface  
✅ **CGLIB Proxy** = Class-based, extends class  
✅ **Internal calls** bypass proxy (major gotcha!)  
✅ **Final classes/methods** cannot be proxied with CGLIB  
✅ **Always inject through Spring** to get proxied instances

---

**Next Steps:**  
Now that you understand the fundamentals, proceed to [Core Concepts](../02-core-concepts/01-joinpoints.md) to dive deeper into join points, pointcuts, and advice types.