---
layout: post
title: JMockIt Usage Tips
category : java
tags: [test, jmockit]
---
{% include JB/setup %}

### Mock the Dependencies of Mocked Fields

在一个测试类中声明的`@Mocked`成员变量（`@Mocked`参数，`@Injectable`变量），在执行测试类中的测试方法之前，JMockit会调用适当的构造函数初始化这些mock变量。如果这些变量的构造函数依赖于其他无法获得的类时，就会造成初始化失败，进而造成测试失败。该问题的原因在于，JMockit在mock成员变量时，也要通过选取适合的constructor进行构造，因此如果在constructor中存在外部依赖，就会引起初始化问题。

因此，在这种情况下需要在声明mock变量的时候，同时也要声明其依赖项对应的mock变量，用于构造该mock变量。

### Mocked vs Injectable

两者的区别，引用[Bowen's blog](http://phoenixjiangnan.github.io/2016/04/06/test/jmockit/Unit-Test-JMockit-What-are-the-differences-between-Mocked-and-Injectable-in-JMockit-and-when-to-use-Injectable-rather-than-Mocked/)的话就是：

> In one sentence, `@Mocked` will mock everything and all instances of that class, and `@Injectable` will only mock a specific method/field of one instance of that class.

因此，对于使用场景的选择准则是：

> - Mark an instance `@Injectable` when you CAN pass that instance to the class that you want to actually test
- Mark an instance/class `@Mocked` when you CANNOT pass that instance to the class that you want to actually test

