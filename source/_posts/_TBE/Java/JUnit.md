---
title: JUnit
date: 2017-01-01 09:00:00
tags: [Java]
---

# org.junit

## **Classes**

### Assert
> 断言。不符条件的话会中断测试方法并报错。

`assertArrayEquals()`
`assertEquals()`
`assertFalse()`
`assertNotNull()`
`assertNotSame()`
`assertNull()`
`assertSame()`
`assertThat()`
`assertTrue()`
`fail()`

### Asssume
> 假设。不符条件的话会中断测试方法但不会报错。

`assumeNoException()`
`assumeNotNull()`
`assumeThat()`
`assumeTrue()`

### Test.None
> 缺省的报空异常。

## **Errors**

### ComparisonFailure
> 当 assertEquals(String, String) 方法失败时抛出，用来展示两个复杂字符串之间具体的不同之处。

`getActual()`
`getExpected()`
`getMessage()`

## **Annotation Types**

### After
> @After 方法在 @Test 方法之后运行（即使 @Before 和 @Test 方法抛出了异常），针对每个测试方法都会执行一次。父类中的 @After 方法会在当前类的 @After 方法之后运行。

### AfterClass
> @AfterClass 方法在本测试类的所有 @Test 方法运行结束之后运行（即使 @BeforeClass 方法抛出了异常），针对所有测试方法，只执行一次。父类中的 @AfterClass 方法会在当前类的 @AfterClass 方法之后运行。

### Before
> @Before 方法在 @Test 方法之前运行，针对每个测试方法都会执行一次。父类中的 @Before 方法会在当前类的 @Before 方法之前运行。

### BeforeClass
> @BeforeClass 方法在本测试类的所有 @Test 方法之前运行，针对所有测试方法，只执行一次。父类中的 @BeforeClass 方法会在当前类的 @BeforeClass 方法之前运行。

### Ignore
> @Ignore 可标记类或方法。被标记的类或方法不会在测试过程中被运行。可自定义参数 @Ignore("why the test is being ignored") 来备注忽略的原因。

### Test
> 将一个 public void 方法标记为测试方法，该方法中抛出的任何异常都会被视为测试失败。有两个可选参数：@Test(expected=SomeException.class) 除了参数中声明的异常外，抛出任何其他异常都会被视为测试失败，缺省为 org.junit.Test.None.class；@Test(timeout=100) 若测试方法运行时间大于参数声明的时间（毫秒），则视为测试失败，缺省为0L。

---
# org.harmcrest.core

## **Classes**

### AllOf
> 对多个 Matcher 进行逻辑与短路运算。

`allOf()`
`describeTo()`
`matches()`

### AnyOf
> 对多个 Matcher 进行逻辑或短路运算。

`anyOf()`
`describeTo()`
`matches()`

### DescribeAs
> 提供针对另一个 Matcher 的自定义描述。

`describedAs()`
`describeTo()`
`matches()`

### Is
> 包装另一个 Matcher，保留其行为但使其更具可读性。

`describeTo()`
`is()`
`matches()`

### IsAnything
> 一个永远返回 true 的 Matcher。

`any()`
`anything()`
`describeTo()`
`matches()`

### IsEqual
> 通过 Object.equals() 方法计算值是否相等。

`describeTo()`
`equalTo()`
`matches()`

### IsInstanceOf
> 测试是否为某个类的实例。

`decribeTo()`
`instanceOf()`
`matches()`

### IsNot
> 对一个 Matcher 的结果进行逻辑非运算。

`describeTo()`
`matches()`
`not()`

### IsNull
> 判断是否为 null。

`describeTo()`
`matches()`
`notNullValue()`
`nullValue()`

### IsSame
> 判断是否为同一个对象。

`describeTo()`
`matches()`
`sameInstance()`

---
# org.junit.matchers

## **Classes**

### JUnitMatchers
> 包含了一些 hamcrest 的基础 CoreMatchers 类中没有的 match 方法，供 assertThat() 方法使用。

`both()`
`containsString()`
`either()`
`everyItem()`
`hasItem()`
`hasItems()`

---
# org.junit.runner

## **Interfaces**

### Desribable
> 包含了一个 getDescription() 方法，提供了被测试方法的描述信息。

## **Classes**

### Description
> 包含了将运行或已运行的测试类的类名和类信息。以前是通过 TestCase 和 TestSuite 类来容纳和展示测试类信息，而在 JUnit4 中由于单元测试没有除 Object 之外的父类，所以需要 Description 类来提供类的描述信息。

`addChild()`
`childlessCopy()`
`createSuiteDescription()`
`createTestDescription()`
`equals()`
`getAnnotation()`
`getAnnotations()`
`getChildren()`
`getDisplayName()`
`hashCode()`
`isEmpty()`
`isSuite()`
`isTest()`
`testCount()`
`toString()`

### JUnitCore
> JUnitCore 是运行测试方法的入口。支持运行 JUnit4、JUnit3.8.x 以及混合方法。可在命令行中执行 `java org.junit.runner.JUnitCore TestClass1 TestClass2 ...`来运行测试。也可使用静态方法 runClasses(Class[]) 来批量运行测试。可以通过创建 JUnitCore 的实例来添加 listener。

### Request
> 一个 Request 对象是所有待运行测试的抽象描述。之前版本的 JUnit 没有这个概念，后来为了支持过滤、排序，并提供更抽象、更丰富的类型（而不只是待测试的那些类），才有了 Request 类。
JUnit 运行测试的流程是：一个指定了待运行测试的 Request -> 为 Request 中指明的每个类创建一个 Runner -> 该 Runner 返回一个 Description （包含了待运行测试的树状结构）。

`aClass()`
`classes()`
`classWithoutSuiteMethod()`
`errorReport()`
`filterWith()`
`getRunner()`
`method()`
`runner()`
`sortWith()`

### Result
> Result 对象收集所有被运行的测试的信息。

`createListener()`
`getFailureCount()`
`getFailures()`
`getIgnoreCount()`
`getRunCount()`
`getRunTime()`
`wasSuccessful()`

### Runner
> Runner 负责运行测试并将重要事件通知给一个 RunNotifier。当使用 RunWith 类调用一个自定义 runner 时需要继承 Runner 类。当创建自定义 runner 时，除了要实现其抽象方法外，还要提供一个构造方法接受包含了所有测试的类作为构造参数。
缺省的 runner 实现保证了测试类会在测试运行前立刻被实例化并且 runner 不会保留测试实例的引用，留给 gc 去清理。

`getDescription()`
`run()`
`testCount()`

## **Annotation Types**
<span id='returnFromRunWith'></span>
### RunWith
> 若一个类被 @RunWith 注解或继承了一个被 @RunWith 注解的类，则 JUnit 会调用在注解中引用的 runner 类来运行测试，而不是 JUnit 中的 runner 类（缺省为 BlockJUnit4ClassRunner.class），从而实现不同的测试行为。详见 [@RunWith](#runWith)

---
# org.junit.runner.manipulation

## **Interfaces**

### Filterable
> runner 可通过实现此接口来实现过滤测试用例的功能。

`filter()`

### Sortable
> runner 可通过实现此接口来实现对测试用例的排序功能。

## **Classes**

### Filter
> 一种典型的应用 Filter 的场景是运行一个测试类中的个别方法。在运行测试之前，可以通过继承 Filter 并将其作为 Request.filterWith() 的入参，或作为 Runner.filter() 的入参（该方法实现自 Filterable 接口）。

`apply()`
`describe()`
`shouldRun()`

Sorter
> 用来对测试排序。通常不是直接使用 Sorter，而是使用 Request.sortWith(Comparator)。

`apply()`
`compare()`

## **Exceptions**

### NoTestsRemainException
> 当 Filter 过滤掉 Runner 中所有测试时抛出。

---
# org.junit.runner.notification

## **Classes**

### Failure
> Failure 对象保存着失败的测试的 Description 和运行时抛出的异常。通常这个 Description 是单个测试的，但是如果在构建测试时发生问题（比如在 @BeforeClass 方法中），则不只包含单个测试的信息。

`getDescription()`
`getException()`
`getMessage()`
`getTestHeader()`
`getTrace()`
`toString()`

### RunListener
> 通过继承 RunListener 并重写适当的方法，来在测试运行期间对某些事件做出响应。如果 Listener 抛出异常，那么为了余下的测试正常运行，该 Listener 会被移除。

`testAssumptionFailure()`
`testFailure()`
`testFinished()`
`testIgnored()`
`testRunFinished()`
`testRunStarted()`
`testStarted()`

### RunNotifier
> 如果自定义了 Runner，就需要向 JUnit 通知测试的运行过程，办法就是向实现的 Runner.runWith(RunNotifier) 传入 RunNotifier 实例。

`addFirstListener()`
`addListener()`
`fireTestAssumptionFailed()`
`fireTestFailure()`
`fireTestFinished()`
`fireTestIgnored()`
`fireTestRunFinished()`
`fireTestRunStarted()`
`fireTestStarted()`
`pleaseStop()`
`removeListener()`

## **Exceptions**

### StoppedByUserException
> 当用户请求停止测试时抛出。

---
# org.junit.runners

## **Classes**

### AllTests
> 用来运行 JUnit3.8.x 风格的 AllTests 类（即只实现了一个静态方法 suite() 的类）。

### BlockJUnit4ClassRunner
> JUnit4 缺省的 Runner，跟上个版本的缺省 Runner（JUnit4ClassRunner）运行效果相同，但具有以一些额外的优点：基于 Statement 的更简单的实现，从而可以在执行流程中插入新的操作；更适合扩展和复用。

`collectInitializationErrors()`
`computeTestMethods()`
`createTest()`
`describeChild()`
`getChildren()`
`methodBlock()`
`methodInvoker()`
`possiblyExpectingExceptions()`
`runChild()`
`testName()`
`validateInstanceMethods()`
`validateTestMethods()`
`validateZeroArgConstructor()`
`withAfters()`
`withBefores()`
`withPotentialTimeout()`

### JUnit4
> 当前版本缺省 Runner 的“别名”，用来实现向后兼容。若缺省 Runner 随着版本更新而改变时，JUnit4 就会改成新缺省 Runner 的“别名”。在 @RunWith() 注解中使用 JUnit4.class 就可以始终使用缺省的 Runner 而不用在意 JUnit 的版本。

### Parameterized
> 为每个测试方法都用每组参数构造一个实例，即构造 测试方法m*参数数量n 个实例。

`getChildren()`

### ParentRunner
> ParentRunner 在整个测试树中处于父节点的位置，而其子节点由某种数据对象 T 定义（对于 BlockJUnit4ClassRunner 是 Method，对于 Suite 是 Class）。ParentRunner 的子类必须实现寻找子节点、描述子节点和运行子节点的功能。ParentRunner 会过滤和排序子节点，处理 @BeforeClass 和 @AfterClass 方法，创建一个复合的 Description，以及按顺序运行子节点。

`childrenInvoker()`
`classBlock()`
`collectInitializationErrors()`
`describeChild()`
`filter()`
`getChildren()`
`getDescription()`
`getName()`
`getTestClass()`
`run()`
`runChild()`
`sort()`
`validatePublicVoidNoArgMethods()`
`withAfterClasses()`
`withBeforeClasses()`


### Suite
> Suite 可以将多个测试类中的指定方法组合在一起运行。相当于 JUnit3.8.x 中的 static Test suite() 方法。

`describeChild()`
`getChildren()`
`runChild()`

## **Annotation Types**

### Parameterized.Parameters
> 从被注解的方法中获得参数数组，每组参数都会作为测试类构造函数的入参来依次实例化一个测试对象。

### Suite.SuiteClasses
> 此注解参数中传入的类的所有测试方法将被组合到一起依次运行。

---
<span id='runWith'></span>
# @RunWith 详述

**BlockJUnit4ClassRunner**
JUnit 的缺省 Runner。不添加 @RunWith 注解时使用的都是这个 Runner。

**Suite**
用来执行分布在多个类中的测试用例。
```
@RunWith(Suite.class)
@SuiteClasses({TestOne.class, TestTwo.class})
public class TestSuitMain {
}
```
JUnit 会依次运行 TestOne 和 TestTwo 中的测试用例，而 TestSuitMain 中的内容（包括其构造方法）不会被运行。

**Parameterized**
Parameterized 继承自 Suite，从构造参数的角度实现了 Suite 的功能——同一个测试类通过多组构造参数来测试不同场景。
```
@RunWith(Parameterized.class)
public class TestGenerateParams {
	private String greeting;

	public TestGenerateParams(String greeting) {
		super();
		this.greeting = greeting;
	}

	@Test
	public void testParams() {
		System.out.println(greeting);
	}
	
	/**
	 * @return 至少是二维数组
	 */
	@Parameters
	public static List<String[]> getParams() {
		return Arrays.asList(new String[][]{
        		                {"hello"}, 
        						{"hi"}, 
        						{"good morning"},
        						{"how are you"}});
	}
}
```
首先需要用 @Parameters 注解一个静态方法，该方法返回一个至少是二维的数组，即若干组需要用到的构造参数。然后需要一个具有入参的构造方法。Parameterized runner 会依次在上述静态方法返回的数组中取用一组作为构造参数来创建测试类的实例，然后运行。

**Categories**
Categories 继承自 Suite，它不仅可以如 Suite 一样将多个测试用例组合到一起，还可以将这些测试用例自定义分类，并只运行其中特定的某些类。
首先需要定义一些用来对测试用例进行分组的接口
```
public interface NormalTests {
}
public interface SpecialTests {
}
```
然后用 @Category 注解对测试类或测试方法进行分类
```
public class TestA {
    @Category(NormalTests.class)
    @Test
    public void normalTestInTestA() {
    }
    
    @Category(SpecialTests.class)
    @Test
    public void specialTestInTestA() {
    }
}

public class TestB {
    @Category(NormalTests.class)
    @Test
    public void normalTestInTestB() {
    }
    
    @Category(SpecialTests.class)
    @Test
    public void specialTestInTestB() {
    }
}
```
最后假如需要将 TestA 和 TestB 组合到一起的话
```
@RunWith(Categories.class)
@SuiteClasses({TestA.class, TestB.class})
@IncludeCategory(NormalTests.class)//添加若干个此注解来包含若干个指定的测试用例
@ExcludeCategory(SpecialTests.class)//添加若干个此注解来排除若干个指定的测试用例
public class TestCategories {
}
```

**Theories**
Theories 继承自 BlockJUnit4ClassRunner，提供了除 Parameterized 之外的另一种参数测试解决方案。Theories 不再需要使用带有参数的 Constructor 而是接受有参的测试方法，修饰的注解也从 @Test 变成了 @Theory，而参数的提供则变成了使用 @DataPoint 或者 @Datapoints 来修饰的变量，两者的唯一不同是前者代表一个数据后者代表一组数据。Theories 会尝试所有类型匹配的参数作为测试方法的入参（有点排列组合的意思）。
```
@RunWith(Theories.class)
public class TheoriesTest {
	@DataPoint
	public static String nameValue1 = "Tony";
	
	@DataPoint 
	public static String nameValue2 = "Jim";
	
	@DataPoint
	public static int ageValue1 = 10;
	
	@DataPoint
	public static int ageValue2 = 20;
	
	@Theory
	public void testMethod(String name, int age) {
		System.out.println(String.format("%s's age is %s", name, age));
	}
}
```
结果如下：
>   Tony's age is 10
    Tony's age is 20
    Jim's age is 10
    Jim's age is 20
    
同样使用 @DataPoints 可以获得一样的效果
```
@RunWith(Theories.class)
public class TheoriesTest {
	@DataPoints
	public static String[] names = {"Tony", "Jim"};
	
	@DataPoints
	public static int[] ageValue1 = {10, 20};
	
	@Theory
	public void testMethod(String name, int age) {
		System.out.println(String.format("%s's age is %s", name, age));
	}
}
```
除此以外 Theories 还可以支持自定义数据提供的方式，需要继承 JUnit 的 ParameterSupplier 类。下面的代码使用 ParameterSupplier 实现上述例子： 
```
public class NameSupplier extends ParameterSupplier {
	@Override
	public List<PotentialAssignment> getValueSources(ParameterSignature sig) {
		PotentialAssignment nameAssignment1 = PotentialAssignment.forValue("name", "Tony");
		PotentialAssignment nameAssignment2 = PotentialAssignment.forValue("name", "Jim");
		return Arrays.asList(new PotentialAssignment[]{nameAssignment1, nameAssignment2});
	}
}

public class AgeSupplier extends ParameterSupplier {
	@Override
	public List<PotentialAssignment> getValueSources(ParameterSignature sig) {
		PotentialAssignment ageAssignment1 = PotentialAssignment.forValue("age", 10);
		PotentialAssignment ageAssignment2 = PotentialAssignment.forValue("age", 20);
		return Arrays.asList(new PotentialAssignment[]{ageAssignment1, ageAssignment2});
	}
}

@RunWith(Theories.class)
public class TheoriesTest {
	@Theory
	public void testMethod(@ParametersSuppliedBy(NameSupplier.class)String name, @ParametersSuppliedBy(AgeSupplier.class)int age) {
		System.out.println(String.format("%s's age is %s", name, age));
	}
}
```
[return](#returnFromRunWith)

---
