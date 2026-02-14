# Pointcut

## Overview

A Pointcut is a predicate or expression that matches join points in your application. Pointcuts define WHERE advice should be applied by selecting specific join points based on criteria such as method names, class names, annotations, parameters, and more. They act as the "selector" in AOP, determining which methods will be intercepted by aspects.

## Definition

**Pointcut**: An expression that selects one or more join points where advice should be executed. In Spring AOP, pointcuts use AspectJ pointcut expression language to match method executions. A pointcut separates the concern of "where to apply advice" from "what advice to apply," enabling flexible and reusable aspect definitions.

Key components:
- **Pointcut Expression**: The pattern that defines matching criteria
- **Pointcut Designators**: Keywords like `execution`, `within`, `@annotation` that specify matching rules
- **Pointcut Signature**: Named pointcuts that can be reused and combined

## Key Concepts

### 1. Pointcut Designators (PCDs)

Spring AOP supports the following AspectJ pointcut designators:

**Method Execution**:
- `execution()` - Matches method execution join points (most common)

**Type Matching**:
- `within()` - Matches join points within certain types
- `this()` - Matches join points where the proxy implements a given type
- `target()` - Matches join points where the target object implements a given type

**Annotation Matching**:
- `@annotation()` - Matches join points where the method has a given annotation
- `@within()` - Matches join points within types that have a given annotation
- `@target()` - Matches join points where the target object's class has a given annotation
- `@args()` - Matches join points where arguments have given annotations

**Parameter Matching**:
- `args()` - Matches join points where arguments are instances of given types
- `bean()` - Spring-specific, matches join points within a Spring bean

### 2. Pointcut Expression Syntax

```
execution(modifiers? return-type declaring-type? method-name(parameters) throws?)
```

- `modifiers`: public, private, protected (optional)
- `return-type`: void, String, *, etc. (required)
- `declaring-type`: package and class name (optional)
- `method-name`: method name or pattern (required)
- `parameters`: parameter types or patterns (required)
- `throws`: exception types (optional)

### 3. Wildcards and Patterns

- `*` - Matches any number of characters (except dot)
- `..` - Matches any number of parameters or packages
- `+` - Matches subclasses/implementations

### 4. Named Pointcuts

Reusable pointcut definitions that can be referenced by name:

```java
@Pointcut("execution(* com.example.service.*.*(..))")
public void serviceMethods() {}
```

## Syntax/Configuration

### Basic Pointcut Expressions

```java
@Aspect
@Component
public class BasicPointcutExamples {
    
    // Match all methods in a package
    @Before("execution(* com.example.service.*.*(..))")
    public void allServiceMethods() { }
    
    // Match methods with specific name
    @Before("execution(* com.example.service.*.findById(..))")
    public void findByIdMethods() { }
    
    // Match methods with specific return type
    @Before("execution(com.example.model.User com.example.service.*.*(..))")
    public void methodsReturningUser() { }
    
    // Match methods with specific parameters
    @Before("execution(* com.example.service.*.save(com.example.model.User))")
    public void saveUserMethods() { }
    
    // Match public methods only
    @Before("execution(public * com.example.service.*.*(..))")
    public void publicMethods() { }
}
```

### Named Pointcuts

```java
@Aspect
@Component
public class NamedPointcutExamples {
    
    // Define named pointcuts
    @Pointcut("execution(* com.example.service.*.*(..))")
    public void serviceMethods() {}
    
    @Pointcut("execution(* com.example.repository.*.*(..))")
    public void repositoryMethods() {}
    
    @Pointcut("@annotation(org.springframework.transaction.annotation.Transactional)")
    public void transactionalMethods() {}
    
    // Use named pointcuts
    @Before("serviceMethods()")
    public void beforeService() { }
    
    @Before("repositoryMethods()")
    public void beforeRepository() { }
    
    @Before("transactionalMethods()")
    public void beforeTransactional() { }
}
```

### Combining Pointcuts

```java
@Aspect
@Component
public class CombinedPointcutExamples {
    
    @Pointcut("execution(* com.example.service.*.*(..))")
    public void serviceMethods() {}
    
    @Pointcut("execution(public * *(..))")
    public void publicMethods() {}
    
    @Pointcut("@annotation(com.example.annotations.Loggable)")
    public void loggableMethods() {}
    
    // AND combination
    @Before("serviceMethods() && publicMethods()")
    public void publicServiceMethods() { }
    
    // OR combination
    @Before("serviceMethods() || repositoryMethods()")
    public void serviceOrRepositoryMethods() { }
    
    // NOT combination
    @Before("serviceMethods() && !loggableMethods()")
    public void nonLoggableServiceMethods() { }
    
    // Complex combination
    @Before("(serviceMethods() || repositoryMethods()) && publicMethods()")
    public void complexCombination() { }
}
```

### Annotation-Based Pointcuts

```java
// Custom annotation
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Auditable {
    String value() default "";
}

@Aspect
@Component
public class AnnotationPointcutExamples {
    
    // Match methods with @Auditable annotation
    @Before("@annotation(com.example.annotations.Auditable)")
    public void auditableMethods() { }
    
    // Access annotation attributes
    @Before("@annotation(auditable)")
    public void auditableMethodsWithParam(Auditable auditable) {
        System.out.println("Audit: " + auditable.value());
    }
    
    // Match classes with annotation
    @Before("@within(org.springframework.stereotype.Service)")
    public void serviceAnnotatedClasses() { }
    
    // Match target objects with annotation
    @Before("@target(org.springframework.stereotype.Repository)")
    public void repositoryAnnotatedTargets() { }
}
```

### Parameter-Based Pointcuts

```java
@Aspect
@Component
public class ParameterPointcutExamples {
    
    // Match methods with specific parameter types
    @Before("execution(* com.example.service.*.*(String, int))")
    public void methodsWithStringAndInt() { }
    
    // Match methods with any number of parameters
    @Before("execution(* com.example.service.*.*(..))")
    public void methodsWithAnyParameters() { }
    
    // Match methods with at least one parameter
    @Before("execution(* com.example.service.*.*(*, ..))")
    public void methodsWithAtLeastOneParam() { }
    
    // Access parameter values
    @Before("execution(* com.example.service.*.save(..)) && args(entity)")
    public void saveMethodsWithParam(Object entity) {
        System.out.println("Saving: " + entity);
    }
    
    // Match specific parameter types
    @Before("args(com.example.model.User)")
    public void methodsAcceptingUser(User user) {
        System.out.println("User parameter: " + user.getName());
    }
}
```

### Bean-Based Pointcuts (Spring-Specific)

```java
@Aspect
@Component
public class BeanPointcutExamples {
    
    // Match specific bean by name
    @Before("bean(userService)")
    public void userServiceBean() { }
    
    // Match beans with name pattern
    @Before("bean(*Service)")
    public void allServiceBeans() { }
    
    // Exclude specific beans
    @Before("execution(* com.example.service.*.*(..)) && !bean(excludedService)")
    public void allServiceExceptExcluded() { }
}
```

## Real-World Use Cases

### 1. Layered Architecture Pointcuts

```java
@Aspect
@Component
public class LayeredArchitectureAspect {
    
    // Controller layer
    @Pointcut("within(com.example.controller..*)")
    public void controllerLayer() {}
    
    // Service layer
    @Pointcut("within(com.example.service..*)")
    public void serviceLayer() {}
    
    // Repository layer
    @Pointcut("within(com.example.repository..*)")
    public void repositoryLayer() {}
    
    // All application layers
    @Pointcut("controllerLayer() || serviceLayer() || repositoryLayer()")
    public void applicationLayers() {}
    
    // Log all layer interactions
    @Before("applicationLayers()")
    public void logLayerAccess(JoinPoint joinPoint) {
        String layer = determineLayer(joinPoint);
        log.info("Accessing {} layer: {}", layer, joinPoint.getSignature().getName());
    }
    
    private String determineLayer(JoinPoint joinPoint) {
        String className = joinPoint.getTarget().getClass().getName();
        if (className.contains(".controller.")) return "CONTROLLER";
        if (className.contains(".service.")) return "SERVICE";
        if (className.contains(".repository.")) return "REPOSITORY";
        return "UNKNOWN";
    }
}
```

### 2. CRUD Operations Pointcuts

```java
@Aspect
@Component
public class CrudOperationsAspect {
    
    // Create operations
    @Pointcut("execution(* com.example..*.save*(..)) || " +
              "execution(* com.example..*.create*(..)) || " +
              "execution(* com.example..*.insert*(..))")
    public void createOperations() {}
    
    // Read operations
    @Pointcut("execution(* com.example..*.find*(..)) || " +
              "execution(* com.example..*.get*(..)) || " +
              "execution(* com.example..*.read*(..))")
    public void readOperations() {}
    
    // Update operations
    @Pointcut("execution(* com.example..*.update*(..)) || " +
              "execution(* com.example..*.modify*(..))")
    public void updateOperations() {}
    
    // Delete operations
    @Pointcut("execution(* com.example..*.delete*(..)) || " +
              "execution(* com.example..*.remove*(..))")
    public void deleteOperations() {}
    
    // All write operations
    @Pointcut("createOperations() || updateOperations() || deleteOperations()")
    public void writeOperations() {}
    
    // Audit write operations
    @AfterReturning("writeOperations()")
    public void auditWrite(JoinPoint joinPoint) {
        log.info("Write operation: {} with args: {}", 
            joinPoint.getSignature().getName(), 
            Arrays.toString(joinPoint.getArgs()));
    }
}
```

### 3. Security Pointcuts

```java
// Security annotations
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD, ElementType.TYPE})
public @interface RequiresRole {
    String[] value();
}

@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD, ElementType.TYPE})
public @interface RequiresPermission {
    String value();
}

@Aspect
@Component
public class SecurityPointcutAspect {
    
    // Methods requiring authentication
    @Pointcut("@annotation(org.springframework.security.access.annotation.Secured) || " +
              "@within(org.springframework.security.access.annotation.Secured)")
    public void securedMethods() {}
    
    // Methods requiring specific roles
    @Pointcut("@annotation(RequiresRole)")
    public void roleBasedMethods() {}
    
    // Methods requiring permissions
    @Pointcut("@annotation(RequiresPermission)")
    public void permissionBasedMethods() {}
    
    // Admin-only operations
    @Pointcut("execution(* com.example.admin..*.*(..))")
    public void adminOperations() {}
    
    // Public endpoints (no security)
    @Pointcut("@annotation(PublicEndpoint)")
    public void publicEndpoints() {}
    
    // Protected endpoints (all except public)
    @Pointcut("within(com.example.controller..*) && !publicEndpoints()")
    public void protectedEndpoints() {}
    
    // Enforce security
    @Before("protectedEndpoints()")
    public void checkAuthentication(JoinPoint joinPoint) {
        // Security validation logic
        if (!SecurityContextHolder.getContext().getAuthentication().isAuthenticated()) {
            throw new SecurityException("Authentication required");
        }
    }
    
    @Before("roleBasedMethods() && @annotation(requiresRole)")
    public void checkRoles(RequiresRole requiresRole) {
        // Role validation logic
    }
}
```

### 4. Performance Monitoring Pointcuts

```java
@Aspect
@Component
public class PerformanceMonitoringAspect {
    
    // Monitor slow operations (database, external APIs)
    @Pointcut("execution(* com.example.repository..*.*(..)) || " +
              "execution(* com.example.external..*.*(..))")
    public void slowOperations() {}
    
    // Monitor public service methods
    @Pointcut("execution(public * com.example.service..*.*(..))")
    public void publicServiceMethods() {}
    
    // Monitor methods with @Timed annotation
    @Pointcut("@annotation(org.springframework.boot.actuate.metrics.annotation.Timed)")
    public void timedMethods() {}
    
    // All monitored operations
    @Pointcut("slowOperations() || timedMethods()")
    public void monitoredOperations() {}
    
    @Around("monitoredOperations()")
    public Object measurePerformance(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.currentTimeMillis();
        try {
            return pjp.proceed();
        } finally {
            long duration = System.currentTimeMillis() - start;
            if (duration > 1000) { // Log if > 1 second
                log.warn("Slow operation: {} took {} ms", 
                    pjp.getSignature().getName(), duration);
            }
        }
    }
}
```

## Common Patterns

### 1. Centralized Pointcut Library

```java
@Aspect
public class PointcutLibrary {
    
    // Package-level pointcuts
    @Pointcut("within(com.example.web..*)")
    public void webLayer() {}
    
    @Pointcut("within(com.example.service..*)")
    public void serviceLayer() {}
    
    @Pointcut("within(com.example.repository..*)")
    public void dataAccessLayer() {}
    
    // Annotation-based pointcuts
    @Pointcut("@annotation(org.springframework.transaction.annotation.Transactional)")
    public void transactionalOperation() {}
    
    @Pointcut("@annotation(org.springframework.cache.annotation.Cacheable)")
    public void cacheableOperation() {}
    
    // Method pattern pointcuts
    @Pointcut("execution(* com.example..*.find*(..))")
    public void queryMethods() {}
    
    @Pointcut("execution(* com.example..*.save*(..))")
    public void saveMethods() {}
    
    // Combined pointcuts
    @Pointcut("serviceLayer() && transactionalOperation()")
    public void transactionalServiceOperation() {}
}

// Using the library
@Aspect
@Component
public class LoggingAspect {
    
    @Before("com.example.aspects.PointcutLibrary.webLayer()")
    public void logWebAccess() { }
    
    @Before("com.example.aspects.PointcutLibrary.transactionalServiceOperation()")
    public void logTransactionalService() { }
}
```

### 2. Hierarchical Pointcut Organization

```java
@Aspect
public class HierarchicalPointcuts {
    
    // Base pointcuts
    @Pointcut("execution(* com.example..*.*(..))")
    public void anyApplicationMethod() {}
    
    @Pointcut("execution(public * *(..))")
    public void anyPublicMethod() {}
    
    // Refined pointcuts
    @Pointcut("anyApplicationMethod() && anyPublicMethod()")
    public void publicApplicationMethods() {}
    
    @Pointcut("publicApplicationMethods() && within(com.example.service..*)")
    public void publicServiceMethods() {}
    
    @Pointcut("publicServiceMethods() && @annotation(Validated)")
    public void validatedPublicServiceMethods() {}
}
```

### 3. Dynamic Pointcut Selection

```java
@Aspect
@Component
public class DynamicPointcutAspect {
    
    @Value("${monitoring.enabled:true}")
    private boolean monitoringEnabled;
    
    @Value("${monitoring.packages}")
    private List<String> monitoredPackages;
    
    @Pointcut("execution(* com.example.service.*.*(..))")
    public void serviceMethods() {}
    
    @Around("serviceMethods()")
    public Object conditionalMonitoring(ProceedingJoinPoint pjp) throws Throwable {
        if (!monitoringEnabled) {
            return pjp.proceed();
        }
        
        String packageName = pjp.getSignature().getDeclaringTypeName();
        boolean shouldMonitor = monitoredPackages.stream()
            .anyMatch(packageName::startsWith);
        
        if (!shouldMonitor) {
            return pjp.proceed();
        }
        
        // Monitoring logic
        long start = System.currentTimeMillis();
        Object result = pjp.proceed();
        long duration = System.currentTimeMillis() - start;
        log.info("Monitored: {} - {}ms", pjp.getSignature().getName(), duration);
        
        return result;
    }
}
```

## Interview Questions

### Question 1: Explain the difference between execution, within, and this pointcut designators in Spring AOP.

**Answer:**

These three pointcut designators serve different purposes in selecting join points:

**1. execution()**
- **Purpose**: Matches method execution join points based on method signature
- **Most common designator** in Spring AOP
- **Syntax**: `execution(modifiers? return-type declaring-type? method-name(params) throws?)`
- **Matches**: The actual method execution

**Example**:
```java
// Matches all methods in UserService class
@Before("execution(* com.example.service.UserService.*(..))")

// Matches methods returning User type
@Before("execution(com.example.model.User com.example.service.*.*(..))")

// Matches save methods with User parameter
@Before("execution(* com.example.service.*.save(com.example.model.User))")
```

**2. within()**
- **Purpose**: Matches join points within certain types/packages
- **More coarse-grained** than execution
- **Syntax**: `within(type-pattern)`
- **Matches**: All methods within specified types or packages

**Example**:
```java
// Matches all methods in service package
@Before("within(com.example.service.*)")

// Matches all methods in service package and sub-packages
@Before("within(com.example.service..*)")

// Matches all methods in UserService class
@Before("within(com.example.service.UserService)")
```

**3. this()**
- **Purpose**: Matches where the **proxy** object is an instance of the given type
- **Works with interfaces** for JDK proxies
- **Syntax**: `this(type)`
- **Matches**: Based on proxy type

**Example**:
```java
// Matches when proxy implements UserService interface
@Before("this(com.example.service.UserService)")

// Useful for checking proxy type
@Before("this(org.springframework.aop.SpringProxy)")
```

**Comparison Table**:

| Aspect | execution() | within() | this() |
|--------|------------|----------|--------|
| Granularity | Method-level | Type/Package-level | Proxy-level |
| Flexibility | Most flexible | Less flexible | Limited |
| Performance | Slower (checks each method) | Faster (checks type once) | Fast |
| Common Use | Specific method patterns | Package-wide aspects | Proxy type checking |

**Practical Example**:
```java
@Aspect
@Component
public class ComparisonAspect {
    
    // execution: Specific method pattern
    @Before("execution(public * com.example.service.*.find*(..))")
    public void beforeFind() {
        // Matches only find* methods
    }
    
    // within: All methods in package
    @Before("within(com.example.service..*)")
    public void beforeService() {
        // Matches ALL methods in service package
    }
    
    // this: Proxy type check
    @Before("this(com.example.service.UserService)")
    public void beforeUserServiceProxy() {
        // Matches when proxy implements UserService
    }
}
```

**When to use each**:
- Use **execution()** when you need fine-grained method selection
- Use **within()** when applying aspects to entire packages/types
- Use **this()** when you need to verify proxy type (rare in practice)
- **Combine them** for precise control:
  ```java
  @Before("within(com.example.service..*) && execution(public * find*(..))")
  // Public find* methods in service package
  ```

### Question 2: How would you write a pointcut to match all methods in a service layer that return a specific type and have at least one parameter?

**Answer:**

Here's a comprehensive approach with multiple solutions:

**Solution 1: Basic Execution Pointcut**
```java
@Aspect
@Component
public class ServiceLayerAspect {
    
    // Match methods returning User with at least one parameter
    @Pointcut("execution(com.example.model.User com.example.service.*.*(*, ..))")
    public void userReturningMethodsWithParams() {}
    
    @Before("userReturningMethodsWithParams()")
    public void beforeUserMethods(JoinPoint joinPoint) {
        log.info("Executing: {}", joinPoint.getSignature());
    }
}
```

**Solution 2: Combining Multiple Conditions**
```java
@Aspect
@Component
public class RefinedServiceLayerAspect {
    
    // Define base pointcuts
    @Pointcut("within(com.example.service..*)")
    public void serviceLayer() {}
    
    @Pointcut("execution(com.example.model.User *.*(..))")
    public void returnsUser() {}
    
    @Pointcut("execution(* *.*(*, ..))")
    public void hasAtLeastOneParameter() {}
    
    // Combine conditions
    @Pointcut("serviceLayer() && returnsUser() && hasAtLeastOneParameter()")
    public void targetMethods() {}
    
    @Before("targetMethods()")
    public void beforeTarget(JoinPoint joinPoint) {
        log.info("Method: {}, Args: {}", 
            joinPoint.getSignature().getName(),
            joinPoint.getArgs().length);
    }
}
```

**Solution 3: With Parameter Type Constraints**
```java
@Aspect
@Component
public class ParameterConstrainedAspect {
    
    // User-returning methods with String parameter
    @Pointcut("execution(com.example.model.User com.example.service.*.*(String, ..))")
    public void userMethodsWithStringParam() {}
    
    // User-returning methods with Long parameter
    @Pointcut("execution(com.example.model.User com.example.service.*.*(Long, ..))")
    public void userMethodsWithLongParam() {}
    
    // Either String or Long as first parameter
    @Pointcut("userMethodsWithStringParam() || userMethodsWithLongParam()")
    public void userMethodsWithSpecificParams() {}
    
    // Access the parameter value
    @Before("execution(com.example.model.User com.example.service.*.*(String, ..)) && args(param, ..)")
    public void beforeWithStringParam(String param) {
        log.info("Called with String parameter: {}", param);
    }
}
```

**Solution 4: Generic Type Handling**
```java
@Aspect
@Component
public class GenericTypeAspect {
    
    // Match methods returning any List<User>
    @Pointcut("execution(java.util.List<com.example.model.User> com.example.service..*.*(*, ..))")
    public void listUserReturningMethods() {}
    
    // Match methods returning Optional<User>
    @Pointcut("execution(java.util.Optional<com.example.model.User> com.example.service..*.*(*, ..))")
    public void optionalUserReturningMethods() {}
    
    // Match any generic User return type
    @Pointcut("listUserReturningMethods() || optionalUserReturningMethods()")
    public void genericUserReturningMethods() {}
}
```

**Solution 5: Complete Example with Validation**
```java
// Custom annotation for documentation
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ReturnsUser {
    String description() default "";
}

@Aspect
@Component
@Slf4j
public class CompleteServiceAspect {
    
    // Main pointcut
    @Pointcut("execution(com.example.model.User com.example.service..*.*(*, ..)) && " +
              "!execution(* *..*Test*.*(..))")  // Exclude test classes
    public void serviceMethodsReturningUser() {}
    
    // With annotation
    @Pointcut("@annotation(ReturnsUser)")
    public void annotatedUserMethods() {}
    
    // Combined
    @Pointcut("serviceMethodsReturningUser() || annotatedUserMethods()")
    public void allUserReturningMethods() {}
    
    // Before advice with parameter validation
    @Before("allUserReturningMethods()")
    public void validateParameters(JoinPoint joinPoint) {
        Object[] args = joinPoint.getArgs();
        
        if (args.length == 0) {
            log.warn("Method {} called without parameters", 
                joinPoint.getSignature().getName());
            return;
        }
        
        log.info("Method: {} with {} parameters", 
            joinPoint.getSignature().getName(), args.length);
        
        // Validate parameters
        for (int i = 0; i < args.length; i++) {
            if (args[i] == null) {
                log.warn("Parameter {} is null", i);
            }
        }
    }
    
    // After returning advice with result validation
    @AfterReturning(
        pointcut = "allUserReturningMethods()", 
        returning = "user"
    )
    public void validateResult(JoinPoint joinPoint, User user) {
        if (user == null) {
            log.warn("Method {} returned null User", 
                joinPoint.getSignature().getName());
        } else {
            log.info("Method {} returned User: {}", 
                joinPoint.getSignature().getName(), user.getId());
        }
    }
}

// Usage in service
@Service
public class UserService {
    
    @ReturnsUser(description = "Finds user by ID")
    public User findById(Long id) {
        // Implementation
        return new User();
    }
    
    public User findByEmail(String email, boolean active) {
        // Implementation
        return new User();
    }
    
    // This won't match (no parameters)
    public User getCurrentUser() {
        return new User();
    }
    
    // This won't match (wrong return type)
    public List<User> findAll(String filter) {
        return Collections.emptyList();
    }
}
```

**Key Points**:
1. **Return type**: Must be exact match (use fully qualified name)
2. **At least one parameter**: Use `(*, ..)` pattern
3. **Service layer**: Use `within()` or package pattern
4. **Exclusions**: Use `!` to exclude patterns
5. **Parameter access**: Use `args()` to capture parameter values
6. **Flexibility**: Combine multiple pointcuts for maintainability

## Pitfall Questions

### Pitfall 1: Overly Broad Pointcut Expressions

**Scenario**:
```java
@Aspect
@Component
public class BroadPointcutAspect {
    
    // TOO BROAD - matches everything!
    @Before("execution(* *(..))")
    public void logEverything(JoinPoint joinPoint) {
        log.info("Method: {}", joinPoint.getSignature().getName());
    }
}
```

**Why it's a pitfall:**

1. **Performance Impact**: 
   - Matches EVERY method in your application
   - Includes Spring framework methods, library methods
   - Creates proxies for all beans
   - Massive performance overhead

2. **Infinite Recursion Risk**:
   - If the aspect uses any Spring beans (like loggers that use Spring)
   - The logging itself triggers the aspect
   - Causes stack overflow

3. **Unexpected Behavior**:
   - Advises getter/setters, toString(), equals()
   - Affects framework internals
   - Hard to debug issues

4. **Example of the problem**:
```java
@Aspect
@Component
public class ProblematicAspect {
    
    @Autowired
    private LogService logService;  // This is a Spring bean
    
    @Before("execution(* *(..))")
    public void logAll(JoinPoint joinPoint) {
        // This calls logService.log()
        // Which triggers this aspect again
        // Which calls logService.log()
        // INFINITE RECURSION!
        logService.log(joinPoint.getSignature().getName());
    }
}
```

**Correct approach:**

**1. Be Specific with Package Patterns**:
```java
@Aspect
@Component
public class SpecificPointcutAspect {
    
    // Only match your application packages
    @Before("execution(* com.example.service..*.*(..))")
    public void logServiceMethods(JoinPoint joinPoint) {
        log.info("Service method: {}", joinPoint.getSignature().getName());
    }
    
    // Multiple specific packages
    @Before("execution(* com.example.service..*.*(..)) || " +
            "execution(* com.example.repository..*.*(..))")
    public void logApplicationMethods(JoinPoint joinPoint) {
        log.info("Application method: {}", joinPoint.getSignature().getName());
    }
}
```

**2. Exclude Framework and Library Code**:
```java
@Aspect
@Component
public class SafePointcutAspect {
    
    @Pointcut("within(com.example..*)")
    public void applicationPackage() {}
    
    @Pointcut("within(org.springframework..*) || " +
              "within(java..*) || " +
              "within(javax..*)")
    public void frameworkPackages() {}
    
    // Only application code, exclude frameworks
    @Pointcut("applicationPackage() && !frameworkPackages()")
    public void safeApplicationCode() {}
    
    @Before("safeApplicationCode()")
    public void logSafely(JoinPoint joinPoint) {
        // Safe to log
        System.out.println("Method: " + joinPoint.getSignature().getName());
    }
}
```

**3. Use Bean Designator to Avoid Infrastructure Beans**:
```java
@Aspect
@Component
public class BeanBasedAspect {
    
    // Only specific beans
    @Before("bean(*Service) || bean(*Repository)")
    public void logBusinessBeans(JoinPoint joinPoint) {
        log.info("Business method: {}", joinPoint.getSignature().getName());
    }
    
    // Exclude infrastructure beans
    @Before("bean(*Service) && !bean(loggingService)")
    public void avoidRecursion(JoinPoint joinPoint) {
        // Won't advise loggingService itself
        log.info("Method: {}", joinPoint.getSignature().getName());
    }
}
```

**4. Use Annotations for Explicit Control**:
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Monitored {
}

@Aspect
@Component
public class AnnotationBasedAspect {
    
    // Only methods explicitly marked as @Monitored
    @Before("@annotation(Monitored)")
    public void monitorMarkedMethods(JoinPoint joinPoint) {
        log.info("Monitored: {}", joinPoint.getSignature().getName());
    }
}

// Usage
@Service
public class UserService {
    
    @Monitored  // This will be advised
    public User createUser(User user) {
        // ...
    }
    
    // This won't be advised
    public User findUser(Long id) {
        // ...
    }
}
```

**Best Practices**:
1. **Always scope to your packages**: Never use `execution(* *(..))`
2. **Exclude framework code**: Explicitly exclude Spring, JDK packages
3. **Test impact**: Monitor performance impact of pointcuts
4. **Use bean() designator**: For better control over which beans are advised
5. **Prefer annotations**: For explicit, opt-in behavior
6. **Start narrow, expand carefully**: Begin with specific pointcuts, broaden only if needed

### Pitfall 2: Incorrect Wildcard Usage in Pointcut Expressions

**Scenario**:
```java
@Aspect
@Component
public class WildcardMistakesAspect {
    
    // MISTAKE 1: Single * for packages
    @Before("execution(* com.example.service.*.*(..))")
    public void mistake1() {
        // Only matches ONE level: com.example.service.UserService
        // Does NOT match: com.example.service.impl.UserServiceImpl
    }
    
    // MISTAKE 2: Wrong parameter pattern
    @Before("execution(* com.example.service.*.*(*,..))")
    public void mistake2() {
        // Matches methods with at least ONE parameter
        // Does NOT match methods with zero parameters
    }
    
    // MISTAKE 3: Missing return type wildcard
    @Before("execution(com.example.service.*.*(..))")
    public void mistake3() {
        // COMPILE ERROR: Missing return type
        // Should be: execution(* com.example.service.*.*(..))
    }
    
    // MISTAKE 4: Confusing * with ..
    @Before("execution(* com.example.*.service.*.*(..))")
    public void mistake4() {
        // Matches: com.example.web.service.UserService
        // Does NOT match: com.example.api.v1.service.UserService
    }
}
```

**Why it's a pitfall:**

1. **Single vs Double Dot**:
   - `*` matches exactly one package level or one word
   - `..` matches zero or more package levels
   - Common mistake: Using `*` when you need `..`

2. **Parameter Patterns**:
   - `()` - no parameters
   - `(..)` - any number of parameters (including zero)
   - `(*)` - exactly one parameter
   - `(*,..)` - at least one parameter
   - `(*,*)` - exactly two parameters

3. **Return Type Confusion**:
   - Return type is REQUIRED in execution expressions
   - `*` matches any return type
   - Specific types must be fully qualified

4. **Class Name Patterns**:
   - `*Service` matches classes ending with "Service"
   - `Service*` matches classes starting with "Service"
   - `*Service*` matches classes containing "Service"

**Correct approach:**

**1. Proper Package Wildcards**:
```java
@Aspect
@Component
public class CorrectPackagePatterns {
    
    // Single level - direct children only
    @Pointcut("within(com.example.service.*)")
    public void singleLevel() {
        // Matches: com.example.service.UserService
        // Does NOT match: com.example.service.impl.UserServiceImpl
    }
    
    // Multiple levels - all descendants
    @Pointcut("within(com.example.service..*)")
    public void multipleLevel() {
        // Matches: com.example.service.UserService
        // Matches: com.example.service.impl.UserServiceImpl
        // Matches: com.example.service.impl.admin.AdminServiceImpl
    }
    
    // Specific sub-package
    @Pointcut("within(com.example.service.impl.*)")
    public void implPackageOnly() {
        // Matches: com.example.service.impl.UserServiceImpl
        // Does NOT match: com.example.service.impl.admin.AdminServiceImpl
    }
}
```

**2. Correct Parameter Patterns**:
```java
@Aspect
@Component
public class CorrectParameterPatterns {
    
    // No parameters
    @Pointcut("execution(* com.example.service.*.*())")
    public void noParameters() {}
    
    // Any number of parameters (including zero)
    @Pointcut("execution(* com.example.service.*.*(..))")
    public void anyParameters() {}
    
    // Exactly one parameter of any type
    @Pointcut("execution(* com.example.service.*.*(*))")
    public void oneParameter() {}
    
    // At least one parameter
    @Pointcut("execution(* com.example.service.*.*(*,..))")
    public void atLeastOneParameter() {}
    
    // Exactly two parameters
    @Pointcut("execution(* com.example.service.*.*(*,*))")
    public void twoParameters() {}
    
    // First parameter is String, rest can be anything
    @Pointcut("execution(* com.example.service.*.*(String,..))")
    public void firstParamString() {}
    
    // Specific parameter types
    @Pointcut("execution(* com.example.service.*.*(String,Long))")
    public void stringAndLong() {}
}
```

**3. Return Type Patterns**:
```java
@Aspect
@Component
public class CorrectReturnTypePatterns {
    
    // Any return type (including void)
    @Pointcut("execution(* com.example.service.*.*(..))")
    public void anyReturnType() {}
    
    // Void return type
    @Pointcut("execution(void com.example.service.*.*(..))")
    public void voidReturnType() {}
    
    // Specific return type (fully qualified)
    @Pointcut("execution(com.example.model.User com.example.service.*.*(..))")
    public void userReturnType() {}
    
    // Specific return type (with import - if in same file)
    @Pointcut("execution(User com.example.service.*.*(..))")
    public void userReturnTypeShort() {}
    
    // Non-void return type (any except void)
    @Pointcut("execution(!void com.example.service.*.*(..))")
    public void nonVoidReturnType() {}
}
```

**4. Class Name Patterns**:
```java
@Aspect
@Component
public class CorrectClassNamePatterns {
    
    // Classes ending with "Service"
    @Pointcut("within(com.example.service..*Service)")
    public void serviceClasses() {
        // Matches: UserService, OrderService
        // Does NOT match: ServiceUtil, MyServiceImpl
    }
    
    // Classes starting with "Abstract"
    @Pointcut("within(com.example..Abstract*)")
    public void abstractClasses() {}
    
    // Classes containing "Service"
    @Pointcut("within(com.example..*Service*)")
    public void serviceRelated() {
        // Matches: UserService, ServiceUtil, MyServiceImpl
    }
    
    // Specific class and all subclasses
    @Pointcut("within(com.example.service.BaseService+)")
    public void baseServiceAndSubclasses() {}
}
```

**5. Complex Patterns with Validation**:
```java
@Aspect
@Component
@Slf4j
public class ValidatedComplexPatterns {
    
    // Correct: All public methods in service layer with at least one parameter
    @Pointcut("execution(public * com.example.service..*.*(*, ..)) && " +
              "!within(com.example.service..*Test*)")
    public void publicServiceMethodsWithParams() {}
    
    // Correct: All save/update/delete methods in repositories
    @Pointcut("execution(* com.example.repository..*+.save*(..)) || " +
              "execution(* com.example.repository..*+.update*(..)) || " +
              "execution(* com.example.repository..*+.delete*(..))")
    public void repositoryWriteMethods() {}
    
    // Correct: Methods returning collections
    @Pointcut("execution(java.util.List com.example.service..*.*(..))" +
              "execution(java.util.Set com.example.service..*.*(..))" +
              "execution(java.util.Collection com.example.service..*.*(..))")
    public void collectionReturningMethods() {}
    
    @Before("publicServiceMethodsWithParams()")
    public void validatePattern(JoinPoint joinPoint) {
        log.info("Matched: {}", joinPoint.getSignature().toLongString());
        // Verify the pattern is matching correctly
    }
}
```

**Quick Reference Table**:

| Pattern | Meaning | Example Match | Example No Match |
|---------|---------|---------------|------------------|
| `*` | One element | `UserService` | `user.Service` |
| `..` | Zero or more | `com.example`, `com.example.service.impl` | N/A |
| `*Service` | Ends with | `UserService` | `ServiceImpl` |
| `Service*` | Starts with | `ServiceImpl` | `UserService` |
| `*Service*` | Contains | `UserService`, `ServiceImpl` | `MyImpl` |
| `()` | No params | `getAll()` | `getAll(String)` |
| `(..)` | Any params | `save()`, `save(User)` | N/A |
| `(*)` | One param | `save(User)` | `save()`, `save(User, String)` |
| `(*,..)` | At least one | `save(User)`, `save(User, String)` | `save()` |
| `+` | Subclasses | All implementing classes | N/A |

**Testing Your Pointcuts**:
```java
@SpringBootTest
public class PointcutTest {
    
    @Autowired
    private ApplicationContext context;
    
    @Test
    public void testPointcutMatches() {
        // Get all advised beans
        String[] beanNames = context.getBeanDefinitionNames();
        
        for (String beanName : beanNames) {
            Object bean = context.getBean(beanName);
            boolean isProxy = AopUtils.isAopProxy(bean);
            
            if (isProxy) {
                System.out.println("Advised bean: " + beanName);
                // Verify expected beans are advised
            }
        }
    }
}
```

**Best Practices**:
1. **Test your pointcuts**: Always verify they match what you expect
2. **Use `..` for packages**: Unless you specifically need single-level
3. **Be explicit with parameters**: Use `(..)` for any, `()` for none
4. **Fully qualify return types**: Avoid ambiguity
5. **Use `within()` for package patterns**: More readable than execution patterns
6. **Document complex expressions**: Explain what they're meant to match
7. **Start specific, expand carefully**: Begin with narrow patterns

## Code Examples

### Complete Working Example: Pointcut Library

```java
// Centralized pointcut definitions
@Aspect
public class ApplicationPointcuts {
    
    // ==== Layer Pointcuts ====
    @Pointcut("within(com.example.controller..*)")
    public void controllerLayer() {}
    
    @Pointcut("within(com.example.service..*)")
    public void serviceLayer() {}
    
    @Pointcut("within(com.example.repository..*)")
    public void repositoryLayer() {}
    
    // ==== Method Type Pointcuts ====
    @Pointcut("execution(* com.example..*.find*(..))")
    public void queryMethods() {}
    
    @Pointcut("execution(* com.example..*.save*(..)) || " +
              "execution(* com.example..*.create*(..)) || " +
              "execution(* com.example..*.insert*(..))")
    public void createMethods() {}
    
    @Pointcut("execution(* com.example..*.update*(..)) || " +
              "execution(* com.example..*.modify*(..))")
    public void updateMethods() {}
    
    @Pointcut("execution(* com.example..*.delete*(..)) || " +
              "execution(* com.example..*.remove*(..))")
    public void deleteMethods() {}
    
    // ==== Annotation Pointcuts ====
    @Pointcut("@annotation(org.springframework.transaction.annotation.Transactional)")
    public void transactional() {}
    
    @Pointcut("@annotation(org.springframework.cache.annotation.Cacheable)")
    public void cacheable() {}
    
    @Pointcut("@annotation(org.springframework.security.access.prepost.PreAuthorize)")
    public void secured() {}
    
    // ==== Combined Pointcuts ====
    @Pointcut("createMethods() || updateMethods() || deleteMethods()")
    public void writeMethods() {}
    
    @Pointcut("serviceLayer() && writeMethods()")
    public void serviceWriteMethods() {}
    
    @Pointcut("serviceLayer() && transactional()")
    public void transactionalServiceMethods() {}
}

// Logging aspect using the library
@Aspect
@Component
@Slf4j
public class LoggingAspect {
    
    @Before("com.example.aspects.ApplicationPointcuts.controllerLayer()")
    public void logController(JoinPoint joinPoint) {
        log.info("Controller: {}", joinPoint.getSignature().getName());
    }
    
    @Before("com.example.aspects.ApplicationPointcuts.serviceWriteMethods()")
    public void logServiceWrites(JoinPoint joinPoint) {
        log.info("Service Write: {} with args: {}", 
            joinPoint.getSignature().getName(),
            Arrays.toString(joinPoint.getArgs()));
    }
}

// Performance monitoring aspect
@Aspect
@Component
@Slf4j
public class PerformanceAspect {
    
    @Around("com.example.aspects.ApplicationPointcuts.repositoryLayer()")
    public Object monitorRepository(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.currentTimeMillis();
        try {
            return pjp.proceed();
        } finally {
            long duration = System.currentTimeMillis() - start;
            log.info("Repository method {} took {} ms", 
                pjp.getSignature().getName(), duration);
        }
    }
}

// Security aspect
@Aspect
@Component
public class SecurityAspect {
    
    @Before("com.example.aspects.ApplicationPointcuts.secured()")
    public void checkSecurity(JoinPoint joinPoint) {
        log.info("Security check for: {}", joinPoint.getSignature().getName());
        // Security validation
    }
}
```

## Related Topics

- [01-aspect.md](./01-aspect.md) - Understanding aspects that use pointcuts
- [02-join-point.md](./02-join-point.md) - Join points selected by pointcuts
- [04-advice-types.md](./04-advice-types.md) - Advice applied at pointcuts
- [../04-pointcut-expressions/](../04-pointcut-expressions/) - Detailed pointcut expressions

## References

- [Spring AOP Pointcut API](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/aop/Pointcut.html)
- [AspectJ Pointcut Language](https://www.eclipse.org/aspectj/doc/released/progguide/semantics-pointcuts.html)
- [Spring Framework AOP Documentation](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-pointcuts)
