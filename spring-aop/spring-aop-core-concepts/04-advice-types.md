# Advice Types

## Overview

Advice is the action taken by an aspect at a particular join point. It represents the "what" and "when" of AOP - what code should execute and when it should execute relative to the join point. Spring AOP supports five types of advice that determine when the advice code runs: before, after returning, after throwing, after (finally), and around the method execution.

## Definition

**Advice**: The code that runs at a join point selected by a pointcut. Different advice types determine when the advice executes:
- **Before**: Runs before the join point
- **After Returning**: Runs after successful method completion
- **After Throwing**: Runs if method throws an exception
- **After (Finally)**: Runs after method completion (success or failure)
- **Around**: Wraps the join point, controlling execution

## Key Concepts

### 1. Advice Execution Order

When multiple advice types apply to the same join point:

```
@Before advice
    @Around advice (before part)
        Actual method execution
    @Around advice (after part)
@AfterReturning or @AfterThrowing
@After (finally)
```

### 2. Advice Types Comparison

| Advice Type | Timing | Can Skip Execution | Can Modify Return Value | Can Handle Exceptions | Use Case |
|-------------|--------|-------------------|------------------------|----------------------|----------|
| @Before | Before method | No | No | No | Validation, logging |
| @AfterReturning | After success | No | No | No | Post-processing |
| @AfterThrowing | After exception | No | No | Yes | Error handling |
| @After | After completion | No | No | No | Cleanup |
| @Around | Wraps method | Yes | Yes | Yes | Full control |

### 3. Common Patterns

- **@Before**: Input validation, security checks, logging
- **@AfterReturning**: Result transformation, caching, auditing
- **@AfterThrowing**: Error logging, notification, recovery
- **@After**: Resource cleanup, final logging
- **@Around**: Performance monitoring, transactions, retry logic

## Syntax/Configuration

### @Before Advice

```java
@Aspect
@Component
public class BeforeAdviceExamples {
    
    // Basic before advice
    @Before("execution(* com.example.service.*.*(..))")
    public void logBefore(JoinPoint joinPoint) {
        System.out.println("Before: " + joinPoint.getSignature().getName());
    }
    
    // Access method arguments
    @Before("execution(* com.example.service.*.save(..)) && args(entity)")
    public void validateEntity(Object entity) {
        if (entity == null) {
            throw new IllegalArgumentException("Entity cannot be null");
        }
    }
    
    // With custom annotation
    @Before("@annotation(validated)")
    public void performValidation(JoinPoint joinPoint, Validated validated) {
        System.out.println("Validating: " + validated.value());
    }
}
```

### @AfterReturning Advice

```java
@Aspect
@Component
public class AfterReturningAdviceExamples {
    
    // Basic after returning
    @AfterReturning("execution(* com.example.service.*.*(..))")
    public void logAfterReturn(JoinPoint joinPoint) {
        System.out.println("After returning: " + joinPoint.getSignature().getName());
    }
    
    // Access return value
    @AfterReturning(
        pointcut = "execution(* com.example.service.*.find*(..))",
        returning = "result"
    )
    public void logResult(JoinPoint joinPoint, Object result) {
        System.out.println("Method " + joinPoint.getSignature().getName() + 
                         " returned: " + result);
    }
    
    // Type-specific return value
    @AfterReturning(
        pointcut = "execution(* com.example.service.UserService.*(..))",
        returning = "user"
    )
    public void processUser(User user) {
        if (user != null) {
            System.out.println("User processed: " + user.getId());
        }
    }
}
```

### @AfterThrowing Advice

```java
@Aspect
@Component
public class AfterThrowingAdviceExamples {
    
    // Basic after throwing
    @AfterThrowing("execution(* com.example.service.*.*(..))")
    public void logException(JoinPoint joinPoint) {
        System.out.println("Exception in: " + joinPoint.getSignature().getName());
    }
    
    // Access exception
    @AfterThrowing(
        pointcut = "execution(* com.example.service.*.*(..))",
        throwing = "ex"
    )
    public void handleException(JoinPoint joinPoint, Exception ex) {
        System.out.println("Exception: " + ex.getMessage());
        // Send notification, log to external system
    }
    
    // Specific exception type
    @AfterThrowing(
        pointcut = "execution(* com.example.service.*.*(..))",
        throwing = "ex"
    )
    public void handleDataAccessException(DataAccessException ex) {
        System.out.println("Database error: " + ex.getMessage());
    }
}
```

### @After (Finally) Advice

```java
@Aspect
@Component
public class AfterAdviceExamples {
    
    // Basic after (finally)
    @After("execution(* com.example.service.*.*(..))")
    public void cleanup(JoinPoint joinPoint) {
        System.out.println("Cleanup after: " + joinPoint.getSignature().getName());
    }
    
    // Resource cleanup
    @After("execution(* com.example.service.*.process*(..))")
    public void releaseResources(JoinPoint joinPoint) {
        // Clean up resources regardless of success or failure
        System.out.println("Releasing resources");
    }
}
```

### @Around Advice

```java
@Aspect
@Component
public class AroundAdviceExamples {
    
    // Basic around advice
    @Around("execution(* com.example.service.*.*(..))")
    public Object measureTime(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.currentTimeMillis();
        Object result = pjp.proceed();
        long duration = System.currentTimeMillis() - start;
        System.out.println("Execution time: " + duration + "ms");
        return result;
    }
    
    // Modify arguments
    @Around("execution(* com.example.service.*.process(..))")
    public Object modifyArgs(ProceedingJoinPoint pjp) throws Throwable {
        Object[] args = pjp.getArgs();
        // Modify arguments
        return pjp.proceed(args);
    }
    
    // Skip execution conditionally
    @Around("execution(* com.example.service.*.*(..)) && @annotation(cacheable)")
    public Object cache(ProceedingJoinPoint pjp, Cacheable cacheable) throws Throwable {
        String key = createKey(pjp);
        Object cached = getFromCache(key);
        if (cached != null) {
            return cached; // Skip execution
        }
        Object result = pjp.proceed();
        putInCache(key, result);
        return result;
    }
}
```

## Real-World Use Cases

See individual advice type files in [../03-advice-types/](../03-advice-types/) for detailed examples.

## Interview Questions

### Question 1: What is the difference between @After and @AfterReturning advice?

**Answer:**

**@After (Finally) Advice:**
- Runs after method completion, regardless of outcome (success or exception)
- Similar to finally block in try-catch-finally
- Cannot access return value
- Cannot access exception
- Used for cleanup operations

**@AfterReturning Advice:**
- Runs only after successful method completion
- Does not run if exception is thrown
- Can access return value
- Cannot access exceptions
- Used for post-processing successful results

**Comparison Example:**
```java
@Aspect
@Component
public class ComparisonAspect {
    
    @After("execution(* com.example.service.*.*(..))")
    public void afterFinally(JoinPoint joinPoint) {
        // Runs always (success or exception)
        System.out.println("After: " + joinPoint.getSignature().getName());
        // Cannot access return value or exception
    }
    
    @AfterReturning(
        pointcut = "execution(* com.example.service.*.*(..))",
        returning = "result"
    )
    public void afterSuccess(JoinPoint joinPoint, Object result) {
        // Runs only on success
        System.out.println("Success: " + result);
        // Can access return value
    }
    
    @AfterThrowing(
        pointcut = "execution(* com.example.service.*.*(..))",
        throwing = "ex"
    )
    public void afterException(JoinPoint joinPoint, Exception ex) {
        // Runs only on exception
        System.out.println("Exception: " + ex.getMessage());
    }
}
```

**Execution Flow:**
```java
@Service
public class UserService {
    public User createUser(String name) {
        // @Before runs
        try {
            User user = new User(name);
            // @AfterReturning runs
            return user;
        } catch (Exception e) {
            // @AfterThrowing runs
            throw e;
        } finally {
            // @After runs
        }
    }
}
```

**When to use each:**
- Use **@After**: Resource cleanup, logging completion, metrics
- Use **@AfterReturning**: Result processing, caching, success auditing

### Question 2: Why would you choose @Around advice over other advice types?

**Answer:**

**@Around advice should be used when you need:**

1. **Full Control Over Execution**:
   - Decide whether to proceed with method execution
   - Modify arguments before execution
   - Modify return value after execution
   - Handle exceptions and decide whether to propagate

2. **Performance Monitoring**:
```java
@Around("execution(* com.example.service.*.*(..))")
public Object measurePerformance(ProceedingJoinPoint pjp) throws Throwable {
    long start = System.currentTimeMillis();
    Object result = pjp.proceed();
    long duration = System.currentTimeMillis() - start;
    log.info("{} took {}ms", pjp.getSignature().getName(), duration);
    return result;
}
```

3. **Caching**:
```java
@Around("@annotation(Cacheable)")
public Object cache(ProceedingJoinPoint pjp) throws Throwable {
    String key = generateKey(pjp);
    Object cached = cache.get(key);
    if (cached != null) {
        return cached; // Skip execution
    }
    Object result = pjp.proceed();
    cache.put(key, result);
    return result;
}
```

4. **Retry Logic**:
```java
@Around("@annotation(Retryable)")
public Object retry(ProceedingJoinPoint pjp) throws Throwable {
    int attempts = 0;
    while (attempts < 3) {
        try {
            return pjp.proceed();
        } catch (Exception e) {
            attempts++;
            if (attempts >= 3) throw e;
            Thread.sleep(1000);
        }
    }
    throw new RuntimeException("Max retries exceeded");
}
```

**When NOT to use @Around:**

Use simpler advice types when you don't need full control:
- **@Before**: Just need to run code before (logging, validation)
- **@AfterReturning**: Just need to process results
- **@AfterThrowing**: Just need to handle exceptions
- **@After**: Just need cleanup

**Performance Consideration:**
@Around is slightly more expensive than other advice types because it requires ProceedingJoinPoint.

## Pitfall Questions

### Pitfall 1: Forgetting to Call proceed() in @Around Advice

**Scenario:**
```java
@Aspect
@Component
public class ProblematicAspect {
    
    @Around("execution(* com.example.service.*.*(..))")
    public Object aroundAdvice(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("Before method");
        // FORGOT to call pjp.proceed()!
        System.out.println("After method");
        return null; // Returns null instead of actual result
    }
}
```

**Why it's a pitfall:**
- Method never executes
- Returns null (or wrong value)
- Silent failure - no exception thrown
- Difficult to debug

**Correct approach:**
```java
@Around("execution(* com.example.service.*.*(..))")
public Object aroundAdvice(ProceedingJoinPoint pjp) throws Throwable {
    System.out.println("Before method");
    Object result = pjp.proceed(); // Always call proceed()
    System.out.println("After method");
    return result; // Return the actual result
}
```

### Pitfall 2: Modifying Return Value in @AfterReturning

**Scenario:**
```java
@AfterReturning(
    pointcut = "execution(* com.example.service.*.*(..))",
    returning = "result"
)
public void modifyResult(Object result) {
    if (result instanceof User) {
        ((User) result).setName("Modified"); // Changes object state
        result = new User(); // DOESN'T change returned value
    }
}
```

**Why it's a pitfall:**
- Cannot change what's returned to caller
- Can only modify mutable object's internal state
- Use @Around if you need to change return value

**Correct approach:**
```java
@Around("execution(* com.example.service.*.*(..))")
public Object modifyReturn(ProceedingJoinPoint pjp) throws Throwable {
    Object result = pjp.proceed();
    if (result instanceof User) {
        User modifiedUser = new User();
        // copy and modify
        return modifiedUser; // Can return different object
    }
    return result;
}
```

## Related Topics

- [../03-advice-types/](../03-advice-types/) - Detailed advice type examples
- [01-aspect.md](./01-aspect.md) - Aspects that contain advice
- [03-pointcut.md](./03-pointcut.md) - Selecting where advice applies

## References

- [Spring AOP Advice Documentation](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-advice)
- [AspectJ Advice](https://www.eclipse.org/aspectj/doc/released/progguide/semantics-advice.html)
