---
title: "Junit 5 Composed Annotations"
date: 2020-12-01T17:16:39-07:00
tags: [java, junit, testing]
draft: false
---

Junit 5 uses an extension system to support code reuse and clean code patterns. An important part of this is being able
 to register test extensions 
 [declaratively](https://junit.org/junit5/docs/current/user-guide/#extensions-registration-declarative). Extension 
 declarations can become quite cumbersome if they need to be repeated over and over again. For example:
 
```java
...
@ExtendWith(MyFirstTestExtension.class)
@ExtendWith(MySecondTestExtension.class)
@ExtendWith(MyRequiredTestExtension.class)
@ExtendWith(MyParameterResolver.class)
class MyClassTest {

  @Test
  void myTestMethod(MyParameter myParam) {
    assertNotNull(myParam);
  }
}
```
Or:
```java
...
@ExtendWith({
  MyFirstTestExtension.class,
  MySecondTestExtension.class,
  MyRequiredTestExtension.class,
  MyParameterResolver.class
})
class MyClassTest {

  @Test
  void myTestMethod(MyParameter myParam) {
    assertNotNull(myParam);
  }
}
```

If you're working with many test classes, making sure all tests have all the right extensions can become an 
 unwelcome chore -- especially if you're adding and extending a framework and providing that framework as a consumable
 library. Composed annotations are the solution to this issue. 

Composed annotations are annotations which are themselves annotated. Allowing a single annotation to be used where many 
 were required. Let's look at one in action:
 
```java 
...
@Target({ ElementType.TYPE, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@ExtendWith({
  MyFirstTestExtension.class,
  MySecondTestExtension.class,
  MyRequiredTestExtension.class,
  MyParameterResolver.class
})
 public @interface MyComposedAnnotation {
 }
 ```

Using this annotation on a test class:

```java
...
@MyComposedAnnotation
class MyClassTest {

  @Test
  void myTestMethod(MyParameter myParam) {
    assertNotNull(myParam);
  }
}
```

This cleans up the test boilerplate and eases the maintenance burden, but there's more to it. It also allows you to 
 explicitly and easily manage the extension execution order of test extensions which depend on other test extensions. 
 Extensions in the class reference array in a supplied `@ExtendWith()` are executed first to last for a given callback
 in the JUnit 5 [lifecycle](https://junit.org/junit5/docs/current/user-guide/#extensions-execution-order-diagram).
