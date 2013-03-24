---
layout: post
title: TestCommon - AutoTheory(1)
tags : [TestCommon]
ccl: ko
date : 2013-03-24 04:37:00 UTC
---
{% include JB/setup %}

이번 포스트는 TestCommon의 **AutoTheory** 기능에 대해 설명하고자 한다.

AutoTheory 기능은 `AutoTheoryAttribute`로 구현되는데,
이 어트리뷰트는 [Xunit] 프레임워크의 [TheoryAttribute] 클래스를 상속한다.
따라서 `AutoTheoryAttribute`는 `TheoryAttribute`의 모든 기능을 부여받게 되는데,
다음과 같은 기능을 가진다.

*   FactAttribute 기능
*   TheoryAttribute 기능
*   Anonymous Creation 기능
*   Parameterized Anonymous Creation 기능

<h3 id="FactAttribute 기능">FactAttribute 기능</h3>

Xunit 프레임워크의 `TheoryAttribute`의 경우 만약 `DataAttribute` 없이 아래와 같은 테스트를 작성하면,
"No data found for ..."란 예외 메세지를 만나게 된다. 즉 `TheoryAttribute`를 사용하는 테스트는 무조건 `DataAttribute`과 같이 작성되어야 함을 의미한다.

```c#
[Theory]
public void Theory_IfDataAttributeIsNotSpecified_Throws()
{
    // Arrange
    // Act
    // Assert
}
```

위와 같은 테스트는 Xunit 프레임워크의 `FactAttribute`를 통해 작성될 수 있는데,
**`AutoTheoryAttribute`는 자동적으로 `DataAttribute` 없을 때 `FactAttribute`와 같은 행동를 하게 된다.**
아래의 테스트 코드에서 확인할 수 있다.

```c#
[AutoTheory]
public void AutoTheory_EvenIfDataAttributeIsNotSpecified_DoesNotThrow()
{
    // Arrange
    // Act
    // Assert
}
```

<!-- break -->

<h3 id="TheoryAttribute 기능">TheoryAttribute 기능</h3>

`AutoTheoryAttribute`는 앞서 언급한 바와 같이 `TheoryAttribute`를 상속하였기 때문에
기본적으로 `TheoryAttribute`가 가지는 모든 기능을 갖게 된다.

아래 `AutoTheory_BehaveAsThoery` 테스트에서 보듯이 `AutoTheoryAttribute`는 `TheoryAttribute`와 같이 
`ClassDataAttribute`, `PropertyDataAttribute` 그리고 `InlineDataAttribute` 모두를 가질 수 있다.

```c#
public class ClassDataItems : IEnumerable<object[]>
{
    public IEnumerator<object[]> GetEnumerator()
    {
        yield return new object[] { "string", 100 };
    }

    IEnumerator IEnumerable.GetEnumerator()
    {
        return GetEnumerator();
    }
}

public static IEnumerable<object[]> PropertyDataItems
{
    get
    {
        yield return new object[] { "string", 100 };
    }
}

[AutoTheory]
[ClassData(typeof(ClassDataItems))]
[PropertyData("PropertyDataItems")]
[InlineData("string", 100)]
public void AutoTheory_BehaveAsThoery(string stringValue, int intValue)
{
    // Arrange
    // Act
    // Assert
    Assert.Equal("string", stringValue);
    Assert.Equal(100, intValue);
}
```

<h3 id="Anonymous Creation 기능">Anonymous Creation 기능</h3>

TestCommon에서 **Anonymous Creation** 기능은 `Fixture` 클래스의 `Create<T>` 메소드를 통해서도 구현이 되지만,
`AutoTheoryAttribute`를 통해서 더욱 간결한 코드로 구현될 수 있다.

아래 `NotByAutoTheoryAttribute` 테스트 코드는 `AutoTheoryAttribute`를 사용하지 않고
`Fixture` 클래스의 `Create<T>` 메소드를 통해 Anonymous Creation 기능을 구현하였는데,

```c#
[AutoTheory]
public void NotByAutoTheoryAttribute()
{
    // Arrange
    var fixture = new Fixture();

    // Act
    var actual = fixture.Create<string>();

    // Assert
    Assert.NotNull(actual);
}
```
위 코드에서 `AutoTheoryAttribute`를 도입하면 아래와 같이 더욱 간결한 코드로 바뀔 수 있다.
위 코드와 같은 방법을 사용하게 된다면,
매 테스트 코드에서 `Fixture` 클래스 객체를 생성하고,
`Create<T>` 메소드를 호출해야 하는 번거로움을 있는데 
`AutoTheoryAttribute` 사용으로 이를 해소할 수 있게 되는 것이다.

```c#
[AutoTheory]
public void ByAutoTheoryAttribute(string actual)
{
    // Arrange
    // Act
    // Assert
    Assert.NotNull(actual);
}
```

그러면 `DataAttribute`가 사용될 때는 `AutoTheoryAttribute`의 Anonymous Creation 기능은 어떻게 작동할까?

아래 코드에서 보듯이 `DataAttribute`(`InlineDataAttribute`)에서 명시된 값이 먼저 테스트 메소드 파라메타에
차례로 입력이 되고 난 후, 나머지 명시되지 않은 값에 대해서 Anonymous Creation 기능에 의해 값이 입력이 되는 것이다.

```c#
[AutoTheory]
[InlineData("string", 100)]
public void AutoTheory_AnonymousCreationWithDataAttribute_CreatesTestSpecimenCorrectly(
    string stringValue1,
    int intValue,
    string stringValue2)
{
    // Arrange
    // Act
    // Assert
    Assert.Equal("string", stringValue1);
    Assert.Equal(100, intValue);
    Assert.NotNull(stringValue2);
}
```

<h3 id="Parameterized Anonymous Creation 기능">Parameterized Anonymous Creation 기능</h3>
`AutoTheoryAttribute`는 Anonymous Creation 기능 뿐만 아니라 **Parameterized Anonymous Creation 기능**도 
지원하는데, 이를 위해 다음과 같은 어트리뷰트가 사용된다.

*   BuildAttribute
*   ToCtorAttribute
*   ToSetterAttribute

<h4 id="BuildAttribute">BuildAttribute</h4>
`BuildAttribute`는 Parameterized Anonymous Creation 대상이 되는 테스트 파라메타 앞에 선언되어
해당 파라메타 값이 Anonymous Creation이 아닌 Parameterized Anonymous Creation으로 생성이 된다는 것을 나타낸다.
아래 코드와 같이 `BuildAttribute`가 선언된 `manyInstance2`는 Parameterized Anonymous Creation 기능으로 
객체가 생성이 되기 때문에 아이템을 가지지 못하게 된다.

```c#
[AutoTheory]
public void ConstructingManyInstance_ParameterizedAnonymousCreation_DoesNotAddItems(
    List<string> manyInstance1,
   [Build] List<string> manyInstance2)
{
    // Arrange
    // Act
    // Assert
    Assert.Equal(3, manyInstance1.Count);
    Assert.Equal(0, manyInstance2.Count);
}
```

`BuildAttribute`의 `IsMocked` 프라퍼티는 선언된 파라메타가 Mocked Instance로 생성될지, 아닐지를
결정하게 된다. 아래 코드에서 확인하자.

```c#
[AutoTheory]
public void IsMocked_AsTrue_CreatesSpecimenAsMockedInstance(
    object anonymousCreation,
    [Build] object isMockedAsFalse,
    [Build(IsMocked = true)] object isMockedAsTrue)
{
    // Arrange
    // Act
    // Assert
    Assert.IsNotType<object>(anonymousCreation); // mocked

    Assert.IsType<object>(isMockedAsFalse); // not mocked

    Assert.IsNotType<object>(isMockedAsTrue); // mocked
}
```

`BuildAttribute`에는 Mocked Instance에 대한 Mock 속성을 설정할 수 있는 프라퍼티들(`CallBase`, `DefaultValue`, `Behavior`)을 가지고 있다.
이중 어느 하나의 값을 입력하더라도 `BuildAttribute`에서는 자동으로 `IsMocked`을 `true`로 설정하여,
Mocked Instance 객체를 생성하게 된다.

아래 코드는 Mock 속성 설정 프라퍼티들 중 `CallBase` 예를 보여주는데,
TestCommon에서는 `CallBase`가 디폴트로 `true`이다.([Moq] 프레임워크에서는 `false`가 디폴트)
따라서, Anonymous Creation으로 생성된 `callBaseAsTrue` 파라메타는 `GetString()` 메소드 호출에 대해
"TestCommon's CallBase is true as default."이라는 문자열을 리턴하게 되지만,

Parameterized Anonymous Creation 기능으로 `CallBase`가 `false`로 설정된 `callBaseAsFalse`파라메타는 
익명의 문자열(GUID 형태)을 리턴하게 되어 "TestCommon's..."문자열과는 같지 않게 된다.

```c#
public class CallBaseTestType
{
    public virtual string GetString()
    {
        return "TestCommon's CallBase is true as default.";
    }
}

[AutoTheory]
public void CallBase_CanSpecifyMockedCallBase(
    CallBaseTestType callBaseAsTrue,
    [Build] CallBaseTestType notMocked,
    [Build(CallBase = false)] CallBaseTestType callBaseAsFalse)
{
    // Arrange
    var expected = "TestCommon's CallBase is true as default.";

    // Act
    // Assert
    Assert.Equal(
        expected,
        callBaseAsTrue.GetString()); // call base as false

    Assert.Equal(
        expected,
        notMocked.GetString()); // not mocked

    Assert.NotEqual(
        expected,
        callBaseAsFalse.GetString()); // call base as true
}
```

그 외 `BuildAttibute`는 `IsGreedy`와 `AutoProperties` 프라퍼티를 가지는데 `IsGreedy`는 이어서 나오는 `ToCtorAttribute`에서 살펴볼 예정이다.
`AutoProperties`는 `AccessModifiers`타입 값을 취하는데 이 프라퍼티에 값이 설정되면, 생성되는 파라메타 객체에 대해 `AutoProperties`의 액세스 한정자를
만족하는 모든 프라퍼티에 대해 임의의 값이 Setter Injection된다.

<h4 id="ToCtorAttribute">ToCtorAttribute</h4>
`ToCtorAttribute`는 `Build<T>().ToCtor()`메소드([지난 포스트 - 생성자 파라메타값 설정] 참조)와 같은 [Constructor Injection] 기능을 수행한다.
[SpecimenBuilderTestType] 타입의 파라메타 `specimenBuilderTest`는 `BuildAttribute`를 가지므로, Parameterized Anonymous Creation을 통해 객체가 생성이 되는데,
이때 `ToCtorAttribute`를 가지는 `string1`의 Constructor Injection 대상이 된다.
아래 코드는 `ToCtorAttribute`를 타입(`string`)을 통한 Constructor Injection을 보여 주는데,
`SpecimenBuilderTestType`의 생성자는 `string` 타입 파라메타를 가지는 생성자들 중 파라메타 수가 가장 적은 것이 선택이 된다.(**Modest**)


```c#
[AutoTheory]
public void ToCtorAttribute_ByType_BehaveAsConstructorInjection(
    [ToCtor] string string1,
    [Build] SpecimenBuilderTestType specimenBuilderTest)
{
    // Arrange
    // Act
    // Assert
    Assert.Same(string1, specimenBuilderTest.String1);
    Assert.Null(specimenBuilderTest.String2);
}
```

`string` 타입 파라메타를 가지는 생성자들 중 파라메타 수가 가장 많은(**Greedy**)를 선택하기 위해서
`BuildAttibute`의 `IsGreedy` 프라퍼티를 아래와 같이 `true`로 설정할 수 있다.(디폴트는 `false`)
타입에 의한 Constructor Injection은 유일성이 보장되지 않기 때문에 아래 코드와 같이
`specimenBuilderTest`객체의 `String1`과 `String2`프라퍼티 모두 생성자 파라메타로 넘겨 받은 `string1`이 설정된다.

```c#
[AutoTheory]
public void ToCtorAttribute_IsGreedyAsTrue_UsesGreedyCtor(
    [ToCtor] string string1,
    [Build(IsGreedy = true)] SpecimenBuilderTestType specimenBuilderTest)
{
    // Arrange
    // Act
    // Assert
    Assert.Same(string1, specimenBuilderTest.String1);
    Assert.Same(string1, specimenBuilderTest.String2);
}
```

`ToCtorAttribute`를 타입을 통하여 Constructor Injection 시, 때때로 아래 코드와 같이
Dervied 타입객체(`string stringValue`)를 Base 타입의 생성자 파라메타(`object value`)에 Injection해야 하는 상황이 생긴다.
이럴 경우 타입이 서로 달라서 Injetion하지 못하는데 아래와 같이 `As` 프라퍼티를 사용하면 이를 해결할 수 있다.

```c#
public class AsTestType
{
    public AsTestType(object value)
    {
        Value = value;
    }

    public object Value { get; private set; }
}

[AutoTheory]
public void ToCtorAttribute_AsProperty_InjectsValueToBaseTypedCtorParameter(
    [ToCtor(As = typeof(object))] string stringValue,
    [Build] AsTestType asTestType)
{
    // Arrange
    // Act
    // Assert
    Assert.Same(stringValue, asTestType.Value);
}
```


`BuildAttibute`는 `Build<T>().ToCtor()` 메소드와 같이 타입에 의한 Constructor Injection외에,
파라메타 이름과 포지션에 의한 방법도 아래 코드처럼 지원한다.

```c#
[AutoTheory]
public void ToCtorAttribute_ByNameAndPosition_InjectsToCertainCtorParameter(
    [ToCtor("string2")] string string2,
    [ToCtor(0)] string string1,
    [Build(IsGreedy = true)] SpecimenBuilderTestType specimenBuilderTest)
{
    // Arrange
    // Act
    // Assert
    Assert.Same(string1, specimenBuilderTest.String1);
    Assert.Same(string2, specimenBuilderTest.String2);
}
```

그럼 아래 코드와 같이 테스트 메소드 파라메타들 중에서
`BuildAttibute`를 가지는 파라메타가 하나 이상일 때,
`ToCtorAttribute`의 대상이 되는 파라메타는 어떻게 결정이 되는 것일까?

`ToCtorAttribute`는 대상 `BuildAttibute` 파라메타를 선택하는 것에 대해 다음과 같은 우선 순위를 갖는다.

1.   `Index`(`BuildAttirbute` 파라메타 인덱스)가 입력된 경우
2.   `Name`(`BuildAttirbute` 파라메타 이름)가 입력된 경우
3.   위 둘다 입력 되지 않으면, `BuildAttirbute` 파라메타 이름이 "sut"인 경우
4.   그외는 맨 처음나오는 `BuildAttirbute` 파라메타 선택(`Index`를 0으로 처리)

아래 코드를 보면 `string1`파라메타에 대해 `BuildAttirbute`를 선택하기 위한 인덱스(`0`)와 이름(`"specimenBuilderTest2"`)이 모두 입력이되어 있는데
이 경우, 위 우선순위를 따라 `string1`파라메타는 `specimenBuilderTest1`에 Constructor Injection된다.(인덱스가 이름보다 우선)

```c#
[AutoTheory]
public void ToCtorAttribute_IndexPriority_InjectsToCtorParameter(
    [ToCtor(Index = 0, Name = "specimenBuilderTest2")] string string1,
    [Build] SpecimenBuilderTestType specimenBuilderTest1,
    [Build] SpecimenBuilderTestType specimenBuilderTest2)
{
    // Arrange
    // Act
    // Assert
    Assert.Same(string1, specimenBuilderTest1.String1);
    Assert.Null(specimenBuilderTest2.String1);
}
```

만약, 위 코드에서 `string1`의 값을 `specimenBuilderTest1`과 `specimenBuilderTest2` 모두에 Constructor Injection하고 싶으면,
아래 코드와 같이 `ToCtorAttribute`를 각각에 대해 적어주면 된다. **이때 인덱스 값 `0`, `1`은 파라메타 포지션 값이 아닌,
`BuildAttribute`를 가지는 파라메타들에 대한 인덱스 값임으로 주의가 필요하다.**

```c#
[AutoTheory]
public void ToCtorAttribute_MoreThanOne_InjectsToCtorParameter(
    [ToCtor(Index = 0), ToCtor(Index = 1)] string string1,
    [Build] SpecimenBuilderTestType specimenBuilderTest1,
    [Build] SpecimenBuilderTestType specimenBuilderTest2)
{
    // Arrange
    // Act
    // Assert
    Assert.Same(string1, specimenBuilderTest1.String1);
    Assert.Same(string1, specimenBuilderTest2.String1);
}
```

아래 코드는 위 우선순위의 3번에 해당하는 경우로, `BuildAttribute`를 가지는 파라메타에 대한 인데스, 이름 모두가 입력 되지 않은 경우,
TestCommon이 `BuildAttribute`를 가지는 파라메타 중 "sut" 이름의 파라메타를 선택하는 것을 보여준다.
이때 [SUT]는 System Under Test의 약자로 무엇을 테스트하느냐를 지칭하게 되는데,
TestCommon에서는 이 무엇에 해당하는 것이 테스트 대상이되는 클래스타입의 객체가 된다.
따라서 `BuildAttribute` 대부분은 이 SUT에 해당하는 테스트 파라메타에 붙게 된다.


```c#
[AutoTheory]
public void ToCtorAttribute_ParameterNamedSut_InjectsToCtorParameter(
    [ToCtor] string string1,
    [Build] SpecimenBuilderTestType specimenBuilderTest,
    [Build] SpecimenBuilderTestType sut)
{
    // Arrange
    // Act
    // Assert
    Assert.Null(specimenBuilderTest.String1);
    Assert.Same(string1, sut.String1);
}
```

아래 코드는 위 우선순위의 4번에 해당하는 경우이다.
TestCommon에서는 위 우선순위 1, 2, 3번 어느 경우도 해당되지 않으면,
제일 먼저 나오는 `BuildAttribute`를 가지는 파라메타를 `ToCtorAttribute`의 대상 파라메타로 설정하게 된다.

```c#
[AutoTheory]
public void ToCtorAttribute_FirstBuildAttribute_InjectsToCtorParameter(
    [ToCtor] string string1,
    [Build] SpecimenBuilderTestType specimenBuilderTest,
    [Build] SpecimenBuilderTestType notSut)
{
    // Arrange
    // Act
    // Assert
    Assert.Same(string1, specimenBuilderTest.String1);
    Assert.Null(notSut.String1);
}
```

<h4 id="ToSetterAttribute">ToSetterAttribute</h4>
`ToSetterAttribute`는 `Build<T>().ToSetter()`메소드([지난 포스트 - 프라퍼티 값 설정] 참조)의 [Setter Injection] 기능을 수행한다.
`ToSetterAttribute`은 앞서 `ToCtorAttribute` 설명과 거의 같은데, 차이점인 Depedency Injection 프라퍼티 선택의 문제에 대해서만 살펴보자.

`ToSetterAttribute`의 프라퍼티 선택은 타입과 이름에 의한 두 가지 방법을 제공한다. 이때 타입에 대한 방법은 유일성을 보장하지 못하므로,
여러 프라퍼티에 값이 입력될 수 있다는 것에 주의하자.

아래 코드를 보자. `publicOrInternalValue`과 `privateValue`는 타입(`string`)에 의한 방법으로 `sut`에 Setter Injection된다.
이때 프라퍼티 setter의 액세스 한정자(`AccessModifiers`)를 지정할 수 있다.
디폴트는 `publicOrInternalValue` 프라퍼티와 같이 `AccessModifiers.PublicOrInternal`이다.
즉 타입에 의한 Setter Injection은 타입과 액세스 한정자 모두를 만족하여야 한다.
이름에 의한 방법은 유일성을 보장함으로 액세스 한정자에 상관없이 특정 프라퍼티에 값을 입력시킬 수 있다.

```c#
[AutoTheory]
public void ToSetterAttribute_InjectsValueToSetter(
    [ToSetter] string publicOrInternalValue,
    [ToSetter(AccessModifiers.Private)] string privateValue,
    [ToSetter("String2")] string namedValue,
    [Build] SpecimenBuilderTestType sut)
{
    // Arrange
    // Act
    // Assert
    Assert.Same(publicOrInternalValue, sut.String3); // pubic string property
    Assert.Same(publicOrInternalValue, sut.String4); // internal string property
    Assert.Same(privateValue, sut.String1); // private string property
    Assert.Same(namedValue, sut.String2); // "String2" named property.
}
```


[Xunit]: http://xunit.codeplex.com/
[TheoryAttribute]: http://xunit.codeplex.com/wikipage?title=Comparisons#note4
[Moq]: https://github.com/Moq/moq4
[지난 포스트 - 생성자 파라메타값 설정]: /TestCommon-Parameterized-Anonymous-Creation-2/
[Constructor Injection]: http://xunitpatterns.com/Dependency%20Injection.html#Constructor%20Injection
[SpecimenBuilderTestType]: /TestCommon-Parameterized-Anonymous-Creation-1#SpecimenBuilderTestType
[SUT]: http://xunitpatterns.com/SUT.html
[지난 포스트 - 프라퍼티 값 설정]: /TestCommon-Parameterized-Anonymous-Creation-3/
[Setter Injection]: http://xunitpatterns.com/Dependency%20Injection.html#Setter%20Injection