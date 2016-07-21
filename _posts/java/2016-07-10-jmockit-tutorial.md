---
layout: post
title: JMockIt Tutorial Translation
category : java
tags: [test, jmockit]
---
{% include JB/setup %}

---
## Mocked types and instances

在测试中，依赖项的被调用的方法和构造函数是mock的目标。Mock给我们提供一种使我们可以将测试代码与其依赖分离的机制。在一个或多个测试中我们通过声明适当的mock成员以及/或者mock参数来指明将会被mock的依赖项。Mock成员会被声明为测试类的标注实例，而mock参数将会被声明为测试函数中的标注参数。被mock的依赖项的类型就是mock成员或mock参数的类型。这些类型可以是interface, class, 包括abstract class和final class, 标注，或者enum。

默认情况下，在测试期间mock类型的所有的非私有方法（包括任何static, final或native）都会被mock。如果mock类型是一个class，那么它的所有超类，直到但是不包含`java.lang.Object`都会被递归地mock。因此，继承的方法也会自动的被mock。同时，在一个类中，它的所有非私有的构造函数也会被mock。

当一个方法或者构造函数被mock时，她的原本的实现代码不会在调用时执行。而是将该调用重定向到JMockit，这样就可以按照测试中显式或隐式指明的方式来处理。

下面例子中的测试作为一个基本的说明，来解释mock成员和mock参数的声明，以及它们在测试代码中的使用。

```java
// "Dependency" is mocked for all tests in this test class.
// The "mockInstance" field holds a mocked instance automatically created for use in each test.
@Mocked Dependency mockInstance;

@Test
public void doBusinessOperationXyz(@Mocked final AnotherDependency anotherMock)
{
   ...
   new Expectations() {{ // an "expectation block"
      ...
      // Record an expectation, with a given value to be returned:
      mockInstance.mockedMethod(...); result = 123;
      ...
   }};
   ...
   // Call the code under test.
   ...
   new Verifications() {{ // a "verification block"
      // Verifies an expected invocation:
      anotherMock.save(any); times = 1;
   }};
   ...
}
```

在执行测试方法时，对于测试方法中声明的mock参数，JMockit将会自动生成一个指定类型的对象，并传递给JUnit/TestNG测试引擎。因此，参数的值绝不会是`null`。对于一个mock成员，如果它不是`final`的，JMockit将会自动生成一个对应类型的实例，并将其赋值给该成员。

这里有几种不同的标注用来声明mock成员和mock参数，其各自拥有不同的默认行为，从而满足不同测试的需求。`@Mocked`是中心标注，它有几种属性可选；`@Injectable`是另外一个mock标注，它只能mock某一个mock实例的方法；
