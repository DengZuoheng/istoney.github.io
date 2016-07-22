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

对于一个返回值是非void类型的方法，可以通过向`result`变量赋值的方式记录其返回值。当这个方法在replay阶段被调用的时候，这个指定的返回值就会被返回给调用者。该`result`赋值语句应该紧跟在`expectation`调用的后面，而不是在`expectation`块之后。

如果这个测试需要在调用该方法时抛出一个**exception**或者**error**，你仍然可以使用`result`变量：简单的将需要抛出的示例赋值给它即可。注意，这种抛出exceptions/errors的记录对于mock方法（任意返回类型）和mock构造函数都是适用的。

在同一个expectation中，可以记录多个连续的result（返回值或抛出throwable），简单的在一行中多次给result变量赋值即可。在同一个expectation中，多个返回值或exception/error可以自由的混合。在需要记录多个连续返回值的境况中，可以一次性调用`results(Object...)`方法。同时，单独一条对result变量的赋值也可以达到这种效果，只要赋予的值是一个包含连续返回值的list或者array。

下面的测试示例为mock class **DependencyAbc**的mock方法记录了上述两种类型的result，它们会在被**UnitUnderTest**调用时用到。被测试的代码实现如下：

```java
public class UnitUnderTest
{
(1)private final DependencyAbc abc = new DependencyAbc();

   public void doSomething()
   {
(2)   int n = abc.intReturningMethod();

      for (int i = 0; i < n; i++) {
         String s;

         try {
(3)         s = abc.stringReturningMethod();
         }
         catch (SomeCheckedException e) {
            // somehow handle the exception
         }

         // do some other stuff
      }
   }
}
```

对于`doSomething()`方法，一个可能的测试是在成功的执行几次后抛出一个`SomeCheckedException`。假如我们希望（无论什么原因）完整的记录这两个class交互的expectation，我们可能会按照如下编写这个测试。（通常，在一个给定测试中，并没有必要，也不重要，去记录mock方法和mock构造函数的所有调用，我们会在后面解决这个问题。）

```java
@Test
public void doSomethingHandlesSomeCheckedException(@Mocked final DependencyAbc abc) throws Exception
{
   new Expectations() {{
(1)   new DependencyAbc();

(2)   abc.intReturningMethod(); result = 3;

(3)   abc.stringReturningMethod();
      returns("str1", "str2");
      result = new SomeCheckedException();
   }};

   new UnitUnderTest().doSomething();
}
```

这个测试记录了三种不同的expectation。第一个，表示对`DependencyAbc()`的调用，仅仅表示在被测试代码中这个依赖项会被初始化，通过无参数的构造函数；不需要指定返回值，出了偶然的exception/error会被抛出（构造函数的返回值类型是void，因此没有必要为它们记录返回的结果）。第二个expectation为`intReturningMethod()`记录了调用时的返回结果为3。第三个为`stringReturningMethod()`记录了三个连续的返回值，其中最后一个result是一个可能的exception，这样就可以达到测试的目的了（注意，只有这个exception不会继续被向外传播时，这个测试才算通过）。

---

## Matching invocations to specific instances

在前面，我们解释说在一个mock实例上的expectation记录，例如"abc.someMethd();"将会匹配任何mock class **DependencyAbc**的实例中的DependencyAbc#someMethod()的调用。在多数情况下，测试代码只会使用依赖项的一个实例，无论这个实例是被传入到测试代码中或者在测试代码中被创建，所以这不会有什么影响，我们可以将其忽略。但是，如果我们需要验证测试代码中多个实例其中特定的一个实例上调用时该怎么办？同时，如果只有mock class的一个或几个实例应该被mock，而其余实例需要保持不变时该怎么办？（当使用Java标注库或其他第三方库的class时，情景2会经常出现。）JMockit提供一种mock标注，`@Injectable`，它只mock指定类型的一个实例，而其他实例不受影响。另外，JMockit还提供几种方式来约束`@Mocked`实例与expectation的匹配，当然，它还是会mock一个class的所有实例。

### Injectable mocked instances

假设我们需要测试使用了一个给定class的多个实例的代码，而我们希望mock这些实例的某一部分。如果被mock实例可以传递，或者注入到被测试代码中，那么我们可以为它声明一个`@Injectable`的mock成员或mock参数。这个被JMockit创建的`Injectable`实例将会是一个独有的mock实例。任何其他的，除非是从一个单独的mock成员/参数获得的，实例仍将是一个普通的，非mock的实例。

同时注意到，因为一个injectable mock实例只会影响自身的行为，静态方法和构造函数也不会被mock。毕竟，一个*static*方法并不是与任何类的实例绑定的，而构造函数仅仅与新创建的（因此是不同的）实例绑定。

```java
public final class ConcatenatingInputStream extends InputStream
{
   private final Queue<InputStream> sequentialInputs;
   private InputStream currentInput;

   public ConcatenatingInputStream(InputStream... sequentialInputs)
   {
      this.sequentialInputs = new LinkedList<InputStream>(Arrays.asList(sequentialInputs));
      currentInput = this.sequentialInputs.poll();
   }

   @Override
   public int read() throws IOException
   {
      if (currentInput == null) return -1;

      int nextByte = currentInput.read();

      if (nextByte >= 0) {
         return nextByte;
      }

      currentInput = sequentialInputs.poll();
      return read();
   }
}
```

这个class可以简单使用**ByteArrayInputStream**对象作为输入来测试，不使用mock。但是，让我们假设我们希望确保*InputStrean#read()*方法在构造函数传入的任何*input stream*上都正常工作。如下测试将会测试这一点。

```java
@Test
public void concatenateInputStreams(
   @Injectable final InputStream input1, @Injectable final InputStream input2)
   throws Exception
{
   new Expectations() {{
      input1.read(); returns(1, 2, -1);
      input2.read(); returns(3, -1);
   }};

   InputStream concatenatedInput = new ConcatenatingInputStream(input1, input2);
   byte[] buf = new byte[3];
   concatenatedInput.read(buf);

   assertArrayEquals(new byte[] {1, 2, 3}, buf);
}
```

注意，这里`@Injectable`的使用是非常必要的，因此测试的class继承了被mock的class，而测试**ConcatenatingInputStream**时调用的方法正是在基类**InputStream**中定义的方法。如果**InputStream**是采用普通的mock方式，那么*read(byte[])*方法将会一直被mock，无论是哪一个实例被调用。

### The `onInstance(m)` constraint

当使用`@Mocked`或`@Capturing`（同时在相同的成员或参数上不使用`@Injectable`）时，我们仍然可以将replay的某个特定mock实例的调用与expectation记录进行匹配。为了做到这点，我们在记录expectation的时候，使用`onInstance(mockObject)`方法，如下所示。

```java
@Test
public void matchOnMockInstance(@Mocked final Collaborator mock)
{
   new Expectations() {{
      onInstance(mock).getValue(); result = 12;
   }};

   // Exercise code under test with mocked instance passed from the test:
   int result = mock.getValue();
   assertEquals(12, result);

   // If another instance is created inside code under test...
   Collaborator another = new Collaborator();

   // ...we won't get the recorded result, but the default one:
   assertEquals(0, another.getValue());
}
```
上面的例子中，测试代码只有在与记录中方法调用的同一个实例上调用`getValue()`才可以通过。当被测试代码要在同一类型的两个或多个不同实例上进行调用，并且测试希望在每一个实例上验证调用的时候，这种方式特别有用。

在测试代码以不同的方式使用到同一类型的多个实例时，为了避免在每一个expectation上使用onInstance(m)，JMockit基于作用范围内mock类型的集合自动指出需要使用"onIntsance"的地方。特别地，在给定测试中对于同一类型而言，在其作用范围内，无论存在两个或多个mock成员/参数，在实例上对方法的调用会一直和这些实例的expectation记录匹配。因此，在这种通用场景下，没有必要显式的使用*onInstance(m)*方法。

### Instances created with a given constructor

特别是对于会由被测试代码生成的未来实例，JMockit提供几种机制使我们可以匹配调用。这些机制都要求在expectation中记录对mock class的指定构造函数（"new"语句）的调用。

第一种机制需要在记录实例方法的expectation时，包含一个简单的从被记录的构造函数获取实例的expectation。如下例所示。

```java
@Test
public void newCollaboratorsWithDifferentBehaviors(@Mocked Collaborator anyCollaborator)
{
   // Record different behaviors for each set of instances:
   new Expectations() {{
      // One set, instances created with "a value":
      Collaborator col1 = new Collaborator("a value");
      col1.doSomething(anyInt); result = 123;

      // Another set, instances created with "another value":
      Collaborator col2 = new Collaborator("another value");
      col2.doSomething(anyInt); result = new InvalidStateException();
   }};

   // Code under test:
   new Collaborator("a value").doSomething(5); // will return 123
   ...
   new Collaborator("another value").doSomething(0); // will throw the exception
   ...
}
```

在上述测试中，我们用`@Mocked`为期望的class声明了一个mock成员或参数。这个mock成员/参数并没有在记录expectation的时候使用；而是使用在“实例化记录”（instantiatiion recordings）中创建的实例来记录实例方法的预期行为。使用匹配的构造函数生成的未来实例，会和记录的实例对应。同时注意，这不是时一一对应关系，而是多对一的关系，从多个可能的未来实例对应到记录的expectation中的一个实例。

第二种机制让我们为那些匹配构造函数的未来实例，记录一个替代实例。采用这种方法，我们可以这样重写上述测试。

```java
@Test
public void newCollaboratorsWithDifferentBehaviors(
   @Mocked final Collaborator col1, @Mocked final Collaborator col2)
{
   new Expectations() {{
      // Map separate sets of future instances to separate mock parameters:
      new Collaborator("a value"); result = col1;
      new Collaborator("another value"); result = col2;

      // Record different behaviors for each set of instances:
      col1.doSomething(anyInt); result = 123;
      col2.doSomething(anyInt); result = new InvalidStateException();
   }};

   // Code under test:
   new Collaborator("a value").doSomething(5); // will return 123
   ...
   new Collaborator("another value").doSomething(0); // will throw the exception
   ...
}
```

该测试的上述两个版本是等价的。当结合使用部分（partial）mock时，第二种方式还可以允许一个真实实例（非mock）来作为替换。

---

## Flexible matching of argumentvalues
