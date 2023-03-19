---
title: "Junit 5 Parameter Resolver Extensions"
date: 2020-11-29T22:57:00-07:00
tags: [java, junit, testing]
draft: false 
---

When writing tests in [Junit 5](https://junit.org/junit5/docs/current/user-guide/), you should always strive to 
 leverage the extension system to improve the easy composition of your tests. Extensions make your test classes more 
 declarative, and make writing more tests easy. After [lifecycle](https://junit.org/junit5/docs/current/user-guide/#extensions-lifecycle-callbacks)
 extensions, [parameter resolver](https://junit.org/junit5/docs/current/user-guide/#extensions-parameter-resolution) 
 extensions are essential to developing a good test framework.
 
Parameter resolvers are exactly that, an interface for resolving test parameters. These parameters can be test method 
 parameters (`@Test`), test class constructor parameters, or test lifecycle method parameters (`@BeforeEach`, 
 `@AfterAll`, etc). Let me provide an example of a parameter resolver implemenation:
 
```java
...
public class MyParameterResolver implements ParameterResolver {
        
        @Override
        public boolean supportsParameter(ParameterContext parameterContext, ExtensionContext extensionContext) {
            return MyParameter.class.isAssignableFrom(parameterContext.getParameter().getType());
        }

        @Override
        public MyParameter resolveParameter(ParameterContext parameterContext, ExtensionContext extensionContext) { 
            return new MyParameter();
        }

}
```

For simple parameter resolvers like the one provided above, Junit 5 (since version 5.6.0) has provided an abstract class
 [TypeBasedParameterResolver](https://github.com/junit-team/junit5/blob/r5.7.0/junit-jupiter-api/src/main/java/org/junit/jupiter/api/extension/support/TypeBasedParameterResolver.java),
 which can be extended to simplify a safe parameter resolver where only the Type matters. This feature has an 
 experimental flag at the time of this writing. In the following example you can see it simplifies the boiler plate of 
 the extension:
```java
public class MyParameterResolver extends TypeBasedParameterResolver<MyParameter> {
        
        @Override
        public MyParameter resolveParameter(ParameterContext parameterContext, ExtensionContext extensionContext) { 
            return new MyParameter();
        }

}

```

With the extension demonstrated above, here is an associated test class:
```java
...
@ExtendWith(MyParameterResolver.class)
class MyClassTest {
    
    @Test
    void myTestMethod(MyParameter myParam) {
        org.junit.jupiter.api.Assertions.assertNotNull(myParam);
    }
}
```

It's just as easy to use the parameter in other supported cases:

```java
...
@ExtendWith(MyParameterResolver.class)
class MyClassTest {

    private final MyParameter classField;

    MyClassTest(MyParameter myConstructorParam) {
        this.classField = myConstructorParam;
    }

    @BeforeEach
    void setup(MyParameter myLifeCycleMethodParameter) {
        org.junit.jupiter.api.Assumptions.assumeTrue(myLifeCycleMethodParameter != null);
    }
    
    @Test
    void myTestMethod() {
        org.junit.jupiter.api.Assertions.assertNotNull(classField);
    }
}
```

These are contrived examples, but they should give a clear idea of how to get a parameter resolver to work for you. In 
 my day to day work, I like to go a bit further to write declarative test classes rapidly, I leverage 
 [Project Lombok](https://projectlombok.org/) in my test classes to eliminate my boilerplate. Specifically, I tend to
 use the [@RequiredArgsConstructor](https://projectlombok.org/features/constructor) annotation to cut the constructor 
 from the test class.

```java
...
@ExtendWith(MyParameterResolver.class)
@RequiredArgsConstructor
class MyClassTest {

    private final MyParameter classField;
    
    @Test
    void myTestMethod() {
        org.junit.jupiter.api.Assertions.assertNotNull(classField);
    }
}
```

When working with Junit 5, and particularly when developing a testing framework, working with parameter resolvers 
 should be second nature.
