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

这里有几种不同的标注用来声明mock成员和mock参数，其各自拥有不同的默认行为，从而满足不同测试的需求。`@Mocked`是中心标注，它有几种适用与不同场景的属性可选；`@Injectable`是另外一种mock标注，它只能mock某一个mock实例的方法；`@Capturing`又是另外一种标注，它扩展了mock，使得一个class去实现一个mock接口，或者让一个子类去继承一个mock class。当在一个成员变量或参数上使用`@Injectable`和`@Capturing`时，`@Mock`是被隐含在其中的，因此不需要（不是不可以）再次使用`@Mock`。

JMockit创建的mock实例可以在测试代码中正常使用，或者在测试代码中传递，或者不使用。和其他的mock API不同的是，这些mock对象并不一定是测试代码在调用方法是用到的那些。默认情况下（例如，在不使用`@Injectable`时）,JMockit并不关心测试代码是在哪一个mock实例上调用方法。这样，在测试代码中使用`new`调用构造函数时，mock实例的创建就是透明的。仅仅要求被初始化的class属于某个mock类型，仅此而已。

---

## Expectations

一个`expectation`是一系列与测试相关的指定的mock方法/构造函数的集合。一个`expectation`可能包含多次对同一方法或构造函数的调用，但是并不需要包含测试执行中所有对该方法的调用。一次函数调用是否属于一个给定`expectation`，不仅仅取决于该方法/构造函数的签名，还包括一些运行时的因素，例如方法是在哪一个实例上调用的，参数值，以及/或者**调用次数是否已经匹配**。因此，对于一个给定`expectation`，我们有几个可选的匹配约束可以设置。

当调用设计到一个或多个参数时，需要给每一个参数指定确切的参数值。例如，可以指定一个`String`类型参数的值为`"test string"`，这就使得`expectation`只匹配在对应参数上值为该字符串的方法调用。我们在后面可以看到，除了给出一个特定参数值外，我们还可以通过指定一个更加宽松的约束来匹配所有不同参数的集合。

下面的例子展示了一个`Dependency#someMethod(int, String)`的`expectation`，它将匹配参数值与指定值一致的该方法的调用。注意到，一个`expectation`它本身是通过对mock方法的一次独立的调用来定义的。该过程并不涉及其他特殊的API（像其他mock API做的那样^_^）。然而，这次调用并不是测试中真正的调用，它仅仅是用来定义`expectation`。

```java
@Test
public void doBusinessOperationXyz(@Mocked final Dependency mockInstance)
{
   ...
   new Expectations() {{
      ...
      // An expectation for an instance method:
      mockInstance.someMethod(1, "test"); result = "mocked";
      ...
   }};

   // A call to code under test occurs here, leading to mock invocations
   // that may or may not match specified expectations.
}
```

在我们理解了`record`、`replay`和`verify`的调用之间的区别之后我们会了解更多关于`expectation`的东西。

---

## The record-replay-verify model

任何一个开发测试都可以分成至少三个独立的执行阶段。这些阶段顺序执行，一次一个，如下所示。

```java
@Test
public void someTestMethod()
{
   // 1. Preparation: whatever is required before the code under test can be exercised.
   ...
   // 2. The code under test is exercised, usually by calling a public method.
   ...
   // 3. Verification: whatever needs to be checked to make sure the code exercised by
   //    the test did its job.
   ...
}
```

首先，我们有一个准备阶段，在这里测试需要的对象和数据项被创建，或从其他地方获得。然后执行被测试代码。最后，将被测试代码的执行结果和预期结果进行对比。

这种三阶段模型被称为`Arrange`、`Act`和`Assert`语法，或者简称为`AAA`。或者其他称法，但是意义相同。

在基于行为的（`behavior-based`）的测试中（即使用mock类型，以及其对应mock实例），我们可以找到如下三个阶段，她们和上述描述的传统测试的三个阶段紧密相关：

1. **record**阶段，方法调用被记录的阶段。这发生在测试准备阶段，在被测试的方法调用执行之前。
2. **replay**阶段，在该阶段被测试代码被执行，以及我们感兴趣的mock调用。之前被mock的方法/构造函数的调用将在这里`replay`。通常在录制的调用和播放之间并没有一一对应的关系。
3. **verify**阶段，验证调用按照预期的发生。该阶段发生在测试验证期间，在被测试代码执行之后。

采用JMockit写出的基于行为的测试将按照如下的模板：

```java
import mockit.*;
... other imports ...

public class SomeTest
{
   // Zero or more "mock fields" common to all test methods in the class:
   @Mocked Collaborator mockCollaborator;
   @Mocked AnotherDependency anotherDependency;
   ...

   @Test
   public void testWithRecordAndReplayOnly(mock parameters)
   {
      // Preparation code not specific to JMockit, if any.

      new Expectations() {{ // an "expectation block"
         // One or more invocations to mocked types, causing expectations to be recorded.
         // Invocations to non-mocked types are also allowed anywhere inside this block
         // (though not recommended).
      }};

      // Unit under test is exercised.

      // Verification code (JUnit/TestNG assertions), if any.
   }

   @Test
   public void testWithReplayAndVerifyOnly(mock parameters)
   {
      // Preparation code not specific to JMockit, if any.

      // Unit under test is exercised.

      new Verifications() {{ // a "verification block"
         // One or more invocations to mocked types, causing expectations to be verified.
         // Invocations to non-mocked types are also allowed anywhere inside this block
         // (though not recommended).
      }};

      // Additional verification code, if any, either here or before the verification block.
   }

   @Test
   public void testWithBothRecordAndVerify(mock parameters)
   {
      // Preparation code not specific to JMockit, if any.

      new Expectations() {{
         // One or more invocations to mocked types, causing expectations to be recorded.
      }};

      // Unit under test is exercised.

      new VerificationsInOrder() {{ // an ordered verification block
         // One or more invocations to mocked types, causing expectations to be verified
         // in the specified order.
      }};

      // Additional verification code, if any, either here or before the verification block.
   }
}
```

上述模板还有一些变形，但其精髓是expectation块属于record阶段，并且出现在被测试代码执行之前，而verification块属于verify阶段。一个测试方法可以包含任意数目的expectation块和verification块。

---

## Regular versus strict expectations

在`new Expectation() {...}`中记录的expectations是普通的。这意味着这些调用预期会在replay阶段至少出现一次；也可以以与其他expectations不同的相对顺序出现多次。此外，不匹配任何记录的expectation的调用允许以任意顺序出现任意多次。如果，没有调用与给定expectation`中的记录匹配，那么在测试的最后会有一个*missing invocation*的错误会被抛出，导致测试失败（这只是默认行为，而且可以被重写）。

该API还支持strict expectation的概念：在replay中只允许与记录匹配的调用执行（需要时，可以显式指明允许），调用次数（默认情况下一次）和顺序都要匹配。replay期间如果出现匹配失败的调用，将会视为*unexpected*，立即触发一个*unexpected invocation*的错误，进而导致测试失败。上述这些都可以通过`StrictExpectation`子类实现。

注意到，在strict expectation中，所有在replay中出现的与expectation匹配的调用，都会被隐式的验证。任何其他不匹配的调用都会被视为*unexpected*，将导致测试失败。如果任何strict expectation中的记录缺少匹配，即在replay中没有出现调用，也会导致测试失败。

我们可以在同一个测试中编写多个expectation块，一些普通的（使用`Expectation`）和一些strict的（使用`StrictExpectation`），来混合使用不同严格等级的expectation。尽管通常情况下，一个给定的mock成员或mock参数将会出现在一种类型的expectation块中。

大部分测试会简单的采用“普通”的expectation。strict expectation的使用更可能是一种个人偏好。

### Strict and non-strict mocks

注意，我们不会指明一个给定的mock类型/实例是strict或不是。一个mock成员或参数的严格性是由它在测试的使用情况来决定的。一旦第一条strict expectation在`new StrictExpectation() {...}`块中记录，那么相关的mock类型/实例将会在整个测试中认为是strict的，否则，将会被视为不是strict的。

---

## Recoding results for and expectation

对于一个返回值是非void类型的方法，可以通过向`result`变量赋值的方式记录其返回值。当在
