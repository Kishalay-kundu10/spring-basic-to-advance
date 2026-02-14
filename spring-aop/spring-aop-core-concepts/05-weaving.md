# Weaving

## Overview

Weaving is the process of applying aspects to target objects to create advised (proxied) objects. It is the mechanism that integrates aspect code with application code at join points. Weaving can occur at different points in the application lifecycle: compile-time, load-time, or runtime.

## Definition

**Weaving**: The process of linking aspects with application code to create an advised object. This process can happen at compile time (using AspectJ compiler), load time (using class loaders), or runtime (using Spring AOP's proxy-based approach).

The weaving mechanism determines:
- **When** aspects are applied to target objects
- **How** the aspect logic is integrated
- **Performance characteristics** of the AOP implementation
- **Limitations** on which join points can be intercepted

## Key Concepts

### 1. Weaving Types

**Compile-Time Weaving (CTW)**:
- Aspects woven during compilation
- Requires AspectJ compiler (ajc)
- Modifies bytecode at compile time
- Best performance
- Can intercept all join point types

**Load-Time Weaving (LTW)**:
- Aspects woven when classes are loaded into JVM
- Uses Java agent or custom class loader
- Modifies bytecode during class loading
- Better flexibility than CTW
- Can intercept all join point types

**Runtime Weaving**:
- Aspects applied at runtime using proxies
- Spring AOP's default approach
- No bytecode modification
- Limited to method execution join points
- Easiest to configure

### 2. Spring AOP Weaving (Proxy-Based)

Spring AOP uses runtime weaving through proxies:
- **JDK Dynamic Proxies**: For interface-based proxies
- **CGLIB Proxies**: For class-based proxies

**Proxy Creation**:
```
Client → Proxy → Target Object
         ↓
      Advice
```

### 3. AspectJ Weaving

Full AspectJ supports all weaving types:
- Compile-time weaving
- Post-compile weaving (binary weaving)
- Load-time weaving

## Syntax/Configuration

### Runtime Weaving (Spring AOP Default)

```java
@Configuration
@EnableAspectJAutoProxy
public class AppConfig {
    // Runtime weaving enabled
}
```

### Load-Time Weaving Configuration

**Enable LTW**:
```java
@Configuration
@EnableLoadTimeWeaving
public class LTWConfig {
}
```

**aop.xml Configuration**:
```xml
<!DOCTYPE aspectj PUBLIC "-//AspectJ//DTD//EN" "http://www.eclipse.org/aspectj/dtd/aspectj.dtd">
<aspectj>
    <weaver options="-verbose -showWeaveInfo">
        <include within="com.example..*"/>
        <exclude within="com.example.exclude..*"/>
    </weaver>
    
    <aspects>
        <aspect name="com.example.aspects.LoggingAspect"/>
        <aspect name="com.example.aspects.SecurityAspect"/>
    </aspects>
</aspectj>
```

**JVM Argument**:
```bash
java -javaagent:path/to/aspectjweaver.jar -jar myapp.jar
```

### Compile-Time Weaving

**Maven Configuration**:
```xml
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>aspectj-maven-plugin</artifactId>
    <version>1.14.0</version>
    <configuration>
        <complianceLevel>17</complianceLevel>
        <source>17</source>
        <target>17</target>
    </configuration>
    <executions>
        <execution>
            <goals>
                <goal>compile</goal>
                <goal>test-compile</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

**Gradle Configuration**:
```gradle
plugins {
    id 'io.freefair.aspectj.post-compile-weaving' version '8.0.1'
}

dependencies {
    aspect 'org.springframework:spring-aspects:6.0.0'
}
```

## Real-World Use Cases

### 1. Choosing Weaving Strategy

```java
// For most Spring applications - Runtime Weaving
@Configuration
@EnableAspectJAutoProxy(proxyTargetClass = true)
public class RuntimeConfig {
    // Simple, no special setup required
}

// For advanced scenarios - Load-Time Weaving
@Configuration
@EnableLoadTimeWeaving(aspectjWeaving = EnableLoadTimeWeaving.AspectJWeaving.ENABLED)
public class LTWConfig {
    // Requires Java agent
    // Can intercept private methods, constructors
}
```

### 2. Performance-Critical Applications

```java
// Compile-Time Weaving for maximum performance
// Configured in build tool, not application code
// Best for high-throughput systems
```

### 3. Hybrid Approach

```java
@Configuration
@EnableAspectJAutoProxy
@EnableLoadTimeWeaving
public class HybridConfig {
    // Spring AOP for business logic
    // LTW for domain model aspects
}
```

## Interview Questions

### Question 1: What are the differences between compile-time, load-time, and runtime weaving?

**Answer:**

| Aspect | Compile-Time Weaving | Load-Time Weaving | Runtime Weaving |
|--------|---------------------|-------------------|-----------------|
| **When** | During compilation | During class loading | At runtime |
| **Tool** | AspectJ compiler (ajc) | Java agent | Spring proxy |
| **Bytecode** | Modified | Modified | Original |
| **Performance** | Best (no overhead) | Good | Slight overhead |
| **Flexibility** | Least (needs recompile) | Good | Most (dynamic) |
| **Join Points** | All types | All types | Method execution only |
| **Setup** | Complex | Moderate | Simple |
| **Use Case** | Production systems | Enterprise apps | Most Spring apps |

**Detailed Comparison:**

1. **Compile-Time Weaving (CTW)**:
- Aspects applied during compilation
- Requires AspectJ compiler
- Modified bytecode in .class files
- Cannot change aspects without recompiling
- Best performance (zero runtime overhead)
- Can intercept: constructors, field access, static methods

2. **Load-Time Weaving (LTW)**:
- Aspects applied when classes loaded into JVM
- Requires Java agent: `-javaagent:aspectjweaver.jar`
- Bytecode modified during loading
- Can change aspects without recompiling
- Small startup overhead
- Can intercept all join point types

3. **Runtime Weaving**:
- Spring AOP's default approach
- Uses proxy objects (JDK or CGLIB)
- No bytecode modification
- Most flexible configuration
- Slight method call overhead
- Limited to method execution

**Example Scenario:**
```java
// Same aspect, different weaving
@Aspect
public class PerformanceAspect {
    @Around("execution(* com.example.service.*.*(..))")
    public Object measure(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.currentTimeMillis();
        Object result = pjp.proceed();
        System.out.println("Time: " + (System.currentTimeMillis() - start));
        return result;
    }
}

// CTW: Woven at compile time, aspect code embedded in service class
// LTW: Woven at class loading, aspect code added when class loads
// Runtime: Service wrapped in proxy, aspect code in proxy
```

### Question 2: Why does Spring AOP use runtime weaving, and what are its limitations?

**Answer:**

**Why Spring AOP Uses Runtime Weaving:**

1. **Simplicity**: No special compiler or build configuration required
2. **Integration**: Works seamlessly with Spring IoC container
3. **Flexibility**: Aspects can be added/removed without recompilation
4. **Standard Java**: No custom compilation or class loading needed
5. **Suitable for most use cases**: Method interception covers 80% of needs

**Limitations of Spring AOP Runtime Weaving:**

1. **Method Execution Only**:
```java
// Works
@Before("execution(* com.example.service.*.*(..))")
public void beforeMethod() { }

// Doesn't work in Spring AOP
@Before("get(* com.example.model.User.name)") // Field access
public void beforeFieldAccess() { }

@Before("call(* com.example.service.*.*(..))")  // Method call
public void beforeMethodCall() { }
```

2. **Cannot Advise**:
- Private methods
- Final methods
- Static methods
- Constructors
- Field access
- Self-invocations within same class

3. **Self-Invocation Problem**:
```java
@Service
public class UserService {
    @Transactional
    public void saveUser(User user) {
        // Transaction starts
        validateUser(user); // Direct call, no proxy!
    }
    
    @Transactional
    public void validateUser(User user) {
        // Transaction NOT applied!
    }
}
```

4. **Proxy Creation Overhead**:
- Every advised bean gets wrapped in proxy
- Small performance cost
- Extra memory for proxy objects

**Solutions to Limitations:**

1. **Use AspectJ LTW for Advanced Scenarios**:
```java
@Configuration
@EnableLoadTimeWeaving
public class AspectJConfig {
    // Can intercept everything
}
```

2. **Self-Injection for Self-Invocation**:
```java
@Service
public class UserService {
    @Autowired
    private UserService self;
    
    public void saveUser(User user) {
        self.validateUser(user); // Goes through proxy
    }
    
    @Transactional
    public void validateUser(User user) {
        // Transaction applied!
    }
}
```

3. **Restructure Code**:
```java
// Split into separate beans
@Service
public class UserService {
    @Autowired
    private ValidationService validationService;
    
    public void saveUser(User user) {
        validationService.validate(user); // Goes through proxy
    }
}
```

## Pitfall Questions

### Pitfall 1: Self-Invocation Not Being Advised

See detailed explanation in previous question.

### Pitfall 2: Incorrect Load-Time Weaving Setup

**Scenario:**
```java
@Configuration
@EnableLoadTimeWeaving
public class Config {
    // Configuration only, no Java agent
}

// Application runs but LTW doesn't work
```

**Why it's a pitfall:**
- `@EnableLoadTimeWeaving` alone is not enough
- Must also add Java agent to JVM arguments
- Silent failure - no error message

**Correct approach:**
```bash
# Add Java agent
java -javaagent:spring-instrument.jar -jar app.jar

# Or AspectJ weaver
java -javaagent:aspectjweaver.jar -jar app.jar
```

## Related Topics

- [01-aspect.md](./01-aspect.md) - Aspects that are woven
- [06-target-object.md](./06-target-object.md) - Objects that are woven
- [../05-configuration/04-proxy-mechanisms.md](../05-configuration/04-proxy-mechanisms.md) - Proxy details

## References

- [AspectJ Weaving](https://www.eclipse.org/aspectj/doc/released/devguide/ltw.html)
- [Spring Load-Time Weaving](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-using-aspectj-ltw)
