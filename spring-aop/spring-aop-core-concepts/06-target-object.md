# Target Object

## Overview

A Target Object is the actual business object that is being advised by one or more aspects. It is the real object that contains the business logic and is wrapped by a proxy when aspects are applied. The target object is also called the "advised object" or "proxied object."

## Definition

**Target Object**: The object that is the subject of advice from one or more aspects. In Spring AOP, this is the actual object instance that contains business logic, and it is wrapped by a proxy object that applies aspect advice before delegating calls to the target.

Key characteristics:
- Contains actual business logic
- Wrapped by proxy when advised
- Unaware of aspects applied to it
- Accessible through `JoinPoint.getTarget()`

## Key Concepts

### 1. Proxy vs Target

```
Client Code
    ↓
  Proxy Object (Spring-generated)
    ↓ (delegates to)
  Target Object (Your class)
```

**Proxy**: Spring-generated wrapper that intercepts method calls
**Target**: Your actual business object

### 2. Proxy Types

Spring creates proxies in two ways:

**JDK Dynamic Proxy**:
- Target implements interface
- Proxy implements same interface
- Uses `java.lang.reflect.Proxy`

**CGLIB Proxy**:
- Target is a class (no interface)
- Proxy extends target class
- Uses bytecode generation

### 3. Target Object Access

```java
@Before("execution(* com.example.service.*.*(..))")
public void beforeAdvice(JoinPoint joinPoint) {
    Object target = joinPoint.getTarget();       // Actual target
    Object proxy = joinPoint.getThis();          // Proxy object
    Class<?> targetClass = target.getClass();    // Target class
}
```

## Syntax/Configuration

### Defining Target Objects

```java
// Simple target object
@Service
public class UserService {
    public void createUser(User user) {
        // Business logic
    }
}

// Interface-based target (JDK proxy)
public interface PaymentService {
    void processPayment(Payment payment);
}

@Service
public class PaymentServiceImpl implements PaymentService {
    @Override
    public void processPayment(Payment payment) {
        // Business logic
    }
}
```

### Accessing Target in Aspects

```java
@Aspect
@Component
public class TargetAccessAspect {
    
    @Before("execution(* com.example.service.*.*(..))")
    public void inspectTarget(JoinPoint joinPoint) {
        Object target = joinPoint.getTarget();
        Object proxy = joinPoint.getThis();
        
        System.out.println("Target class: " + target.getClass().getName());
        System.out.println("Proxy class: " + proxy.getClass().getName());
        System.out.println("Is proxy: " + AopUtils.isAopProxy(proxy));
        System.out.println("Is JDK proxy: " + AopUtils.isJdkDynamicProxy(proxy));
        System.out.println("Is CGLIB proxy: " + AopUtils.isCglibProxy(proxy));
    }
}
```

### Forcing Proxy Type

```java
// Force CGLIB proxies
@Configuration
@EnableAspectJAutoProxy(proxyTargetClass = true)
public class CglibProxyConfig {
}

// Allow JDK proxies (default)
@Configuration
@EnableAspectJAutoProxy(proxyTargetClass = false)
public class JdkProxyConfig {
}
```

## Real-World Use Cases

### 1. Target Validation

```java
@Aspect
@Component
public class TargetValidationAspect {
    
    @Before("execution(* com.example.service.*.*(..))")
    public void validateTarget(JoinPoint joinPoint) {
        Object target = joinPoint.getTarget();
        
        // Check if target is in valid state
        if (target instanceof Lifecycle) {
            Lifecycle lifecycle = (Lifecycle) target;
            if (!lifecycle.isRunning()) {
                throw new IllegalStateException("Service not initialized");
            }
        }
    }
}
```

### 2. Target Metadata Inspection

```java
@Aspect
@Component
public class MetadataInspectionAspect {
    
    @Before("execution(* com.example.service.*.*(..))")
    public void logTargetMetadata(JoinPoint joinPoint) {
        Object target = joinPoint.getTarget();
        Class<?> targetClass = target.getClass();
        
        // Get annotations
        Service serviceAnnotation = targetClass.getAnnotation(Service.class);
        if (serviceAnnotation != null) {
            log.info("Service: {}", serviceAnnotation.value());
        }
        
        // Get interfaces
        Class<?>[] interfaces = targetClass.getInterfaces();
        log.info("Implements: {}", Arrays.toString(interfaces));
    }
}
```

### 3. Target State Management

```java
@Aspect
@Component
public class StateManagementAspect {
    
    private Map<Object, AtomicInteger> callCounts = new ConcurrentHashMap<>();
    
    @Before("execution(* com.example.service.*.*(..))")
    public void trackCalls(JoinPoint joinPoint) {
        Object target = joinPoint.getTarget();
        
        callCounts.computeIfAbsent(target, k -> new AtomicInteger(0))
                  .incrementAndGet();
        
        log.info("Target {} called {} times", 
            target.getClass().getSimpleName(),
            callCounts.get(target).get());
    }
}
```

## Interview Questions

### Question 1: What is the difference between the target object and the proxy object in Spring AOP?

**Answer:**

**Target Object**:
- Your actual business object
- Contains real business logic
- Unaware of aspects
- Not generated by Spring
- Example: `UserServiceImpl` instance

**Proxy Object**:
- Wrapper around target
- Generated by Spring
- Intercepts method calls
- Applies aspects
- Delegates to target
- Example: `UserServiceImpl$$EnhancerBySpringCGLIB$$12345`

**Visual Representation**:
```
Client → Proxy → Target
         ↓
      Aspects
```

**Code Example**:
```java
@Aspect
@Component
public class ProxyVsTargetAspect {
    
    @Before("execution(* com.example.service.*.*(..))")
    public void showDifference(JoinPoint joinPoint) {
        Object proxy = joinPoint.getThis();
        Object target = joinPoint.getTarget();
        
        System.out.println("Proxy: " + proxy.getClass().getName());
        // Output: com.example.service.UserServiceImpl$$EnhancerBySpringCGLIB$$1234
        
        System.out.println("Target: " + target.getClass().getName());
        // Output: com.example.service.UserServiceImpl
        
        System.out.println("Same object: " + (proxy == target));
        // Output: false
    }
}
```

**Key Differences**:

| Aspect | Target Object | Proxy Object |
|--------|---------------|--------------|
| **Created By** | Developer | Spring AOP |
| **Purpose** | Business logic | Intercept calls |
| **Aware of AOP** | No | Yes |
| **Class Type** | Original class | Subclass/Interface impl |
| **Performance** | No overhead | Slight overhead |
| **Access** | `getTarget()` | `getThis()` |

**When you need each**:
- **Target**: Access actual object state, call methods directly
- **Proxy**: Understand proxy type, check proxy status

### Question 2: Why can't Spring AOP advise final methods or classes?

**Answer:**

Spring AOP cannot advise final methods or classes because of how proxy generation works:

**CGLIB Proxy Limitation**:
- CGLIB creates proxies by **extending** the target class
- Extended class overrides methods to add advice
- Final methods **cannot be overridden**
- Final classes **cannot be extended**

**Example of the Problem**:
```java
@Service
public final class UserService {  // PROBLEM: Final class
    public void saveUser(User user) {
        // Cannot be proxied
    }
}

@Service
public class OrderService {
    public final void processOrder(Order order) {  // PROBLEM: Final method
        // Cannot be advised
    }
}
```

**What Happens**:
```java
// Spring tries to create proxy
public class OrderService$$CGLIB extends OrderService {
    @Override
    public final void processOrder(Order order) {  // COMPILATION ERROR!
        // Cannot override final method
    }
}
```

**Solution 1: Remove final modifier**:
```java
@Service
public class UserService {  // No longer final
    public void saveUser(User user) {
        // Can be proxied
    }
}
```

**Solution 2: Use interface + JDK Proxy**:
```java
public interface UserService {
    void saveUser(User user);
}

@Service
public final class UserServiceImpl implements UserService {  // Can be final
    @Override
    public void saveUser(User user) {
        // JDK proxy implements interface, not extends class
    }
}
```

**Solution 3: Use AspectJ instead of Spring AOP**:
```java
@Configuration
@EnableLoadTimeWeaving
public class AspectJConfig {
    // AspectJ modifies bytecode directly
    // Can handle final methods/classes
}
```

**Additional Considerations**:

1. **Private Methods**: Also cannot be advised (not visible to proxy)
2. **Static Methods**: Cannot be advised (belong to class, not instance)
3. **Constructors**: Cannot be advised in Spring AOP (use AspectJ)

**Detection Code**:
```java
@Component
public class ProxyChecker implements CommandLineRunner {
    
    @Autowired
    private ApplicationContext context;
    
    @Override
    public void run(String... args) {
        String[] beanNames = context.getBeanDefinitionNames();
        
        for (String beanName : beanNames) {
            Object bean = context.getBean(beanName);
            
            if (AopUtils.isAopProxy(bean)) {
                System.out.println("Proxied: " + beanName);
            } else if (isFinalClass(bean)) {
                System.out.println("WARNING: Final class not proxied: " + beanName);
            }
        }
    }
    
    private boolean isFinalClass(Object bean) {
        return Modifier.isFinal(bean.getClass().getModifiers());
    }
}
```

## Pitfall Questions

### Pitfall 1: Exposing Target Object Breaks Encapsulation

**Scenario:**
```java
@Aspect
@Component
public class TargetExposingAspect {
    
    private Map<String, Object> exposedTargets = new HashMap<>();
    
    @Before("execution(* com.example.service.*.*(..))")
    public void exposeTarget(JoinPoint joinPoint) {
        Object target = joinPoint.getTarget();
        String key = target.getClass().getSimpleName();
        
        // PROBLEM: Exposing target to outside world
        exposedTargets.put(key, target);
    }
    
    // PROBLEM: Public access to targets
    public Object getTarget(String key) {
        return exposedTargets.get(key);
    }
}

// Elsewhere in code
Object target = aspect.getTarget("UserService");
// Direct access bypasses proxy and aspects!
```

**Why it's a pitfall:**
- Bypasses proxy and all aspects
- Breaks AOP guarantees
- Security/transaction aspects not applied
- Violates encapsulation

**Correct approach:**
```java
@Aspect
@Component
public class SafeAspect {
    
    // Store metadata, not targets
    private Map<String, TargetMetadata> metadata = new ConcurrentHashMap<>();
    
    @Before("execution(* com.example.service.*.*(..))")
    public void collectMetadata(JoinPoint joinPoint) {
        Object target = joinPoint.getTarget();
        String key = target.getClass().getSimpleName();
        
        // Store metadata, not reference
        metadata.computeIfAbsent(key, k -> new TargetMetadata(
            target.getClass().getName(),
            System.currentTimeMillis()
        ));
    }
    
    // Safe: Returns metadata, not target
    public TargetMetadata getMetadata(String key) {
        return metadata.get(key);
    }
}
```

### Pitfall 2: Modifying Target State Without Understanding Scope

**Scenario:**
```java
@Aspect
@Component
public class StatefulAspect {
    
    @Before("execution(* com.example.service.*.*(..))")
    public void modifyTarget(JoinPoint joinPoint) {
        Object target = joinPoint.getTarget();
        
        if (target instanceof ConfigurableService) {
            ConfigurableService service = (ConfigurableService) target;
            // PROBLEM: Modifying singleton target state
            service.setConfig(new Config());
        }
    }
}

@Service  // Default scope: singleton
public class UserService implements ConfigurableService {
    private Config config;  // Shared across all calls!
    
    @Override
    public void setConfig(Config config) {
        this.config = config;
    }
}
```

**Why it's a pitfall:**
- Target is typically singleton
- State modified by one thread affects all threads
- Concurrent modification issues
- Unexpected behavior

**Correct approach:**

**Option 1: Don't modify target state**:
```java
@Aspect
@Component
public class StatelessAspect {
    
    @Around("execution(* com.example.service.*.*(..))")
    public Object wrapWithConfig(ProceedingJoinPoint pjp) throws Throwable {
        // Don't modify target, pass config as parameter
        Config config = new Config();
        Object[] newArgs = addConfig(pjp.getArgs(), config);
        return pjp.proceed(newArgs);
    }
}
```

**Option 2: Use thread-local storage**:
```java
@Service
public class UserService {
    private ThreadLocal<Config> configHolder = new ThreadLocal<>();
    
    public void setConfig(Config config) {
        configHolder.set(config);  // Thread-safe
    }
    
    public Config getConfig() {
        return configHolder.get();
    }
}
```

**Option 3: Prototype scope if needed**:
```java
@Service
@Scope("prototype")  // New instance per request
public class UserService {
    private Config config;  // Safe to modify
}
```

## Related Topics

- [01-aspect.md](./01-aspect.md) - Aspects that advise targets
- [02-join-point.md](./02-join-point.md) - Access to target at join points
- [05-weaving.md](./05-weaving.md) - How targets are woven
- [../05-configuration/04-proxy-mechanisms.md](../05-configuration/04-proxy-mechanisms.md)

## References

- [Spring AOP Proxies](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-understanding-aop-proxies)
- [AopUtils Documentation](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/aop/support/AopUtils.html)
