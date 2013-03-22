---
layout: post
title: TestCommon - Parameterized Anonymous Creation(3)
tags : [TestCommon]
ccl: ko
date : 2013-03-22 06:32:00 UTC
---
{% include JB/setup %}

이번 포스트에서는 Parameterized Anonymous Creation기능 설정(5가지) 중 **프라퍼티값 설정**에 대해서 알아보고자 한다.

1. 생성자 선택
2. 생성자 파라메타값 설정
3. **프라퍼티값 설정**
4. 특정메소드 실행
5. Mock에 관한 속성 설정


[Test Double]을 테스트 대상 객체에 설정하는 방법은 [이전 포스트]에서 언급한 생성자 파라메타값 설정을
통해서도 가능하지만 이번 포스트에서 소개할 **프라퍼티값 설정**으로도 가능하다.
전자를 가리켜 [Constructor Injection]이라하고 후자를 가리켜 [Setter Injection]이라고 한다.

TestCommon에서의 Constructor Injection은 `ToCtor`메소드를 통해 구현되었는데
**Setter Injection**은 `ToSetter`메소드를 통해 구현이 된다.
[지난 포스트]에서 언급한 `Fixture` 클래스의 `AutoProperties`로 설정된 프라퍼티 값들은
**Setter Injection**설정으로 특정 프라퍼티 값이 변경될 수 있다.

<!-- break -->

<h4 id="ToAllSetters">ToAllSetters()/ToAllSetters(AccessModifiers)</h4>
`ToAllSetters(AccessModifiers)`메소드는 `Fixture` 클래스의 `AutoProperties`와 같은 기능을 한다.
주어진 `AccessModifiers`의 액세스한정자와 일치하는 모든 프라퍼티에 대해 임의의 값을 설정하게 된다.
`ToAllSetters()`메소드는 `ToAllSetters(AccessModifiers)`의 오버로드 메소드로 `AccessModifiers`값이
`PublicOrInternal`로 설정이 된다.
[SpecimenBuilderTestType] 클래스는 지난 포스트를 참고하도록 하자.

```c#
[Fact]
public void ToAllSetters_SetsValueToAllPulicOrInternalProperties()
{
    // Arrange
    var fixture = new Fixture();

    // Act
    var actualForDefault = fixture.Build<SpecimenBuilderTestType>()
                                  .Create();

    var actualForToAllSetters = fixture.Build<SpecimenBuilderTestType>()
                                       .ToAllSetters()
                                       .Create();

    // Assert
    Assert.Null(actualForDefault.String4);
    Assert.NotNull(actualForToAllSetters.String4);
}
```

<h4 id="NotToAllSetters">NotToAllSetters()</h4>
`NotToAllSetters()`메소드는 `ToAllSetters()`메소드의 반대 개념이다.
즉 설정된 모든 Setter Injection을 무시한다.
아래 코드에서 `actualForToAllSetters`에서는 모든 프라퍼티에 대해 임의 값이 설정되지만,
`NotToAllSetters()`메소드가 호출된 `actualForNotToAllSetters` 객체는 모든 프라퍼티에 대해 
Setter Injection이 적용되지 않는다.

```c#
[Fact]
public void NotToAllSetters_DeletesAllSetterInjections()
{
    // Arrange
    var fixture = new Fixture();

    // Act
    var actualForToAllSetters = fixture.Build<SpecimenBuilderTestType>()
                                       .ToAllSetters(AccessModifiers.All)
                                       .Create();

    var actualForNotToAllSetters = fixture.Build<SpecimenBuilderTestType>()
                                       .ToAllSetters(AccessModifiers.All)
                                       .NotToAllSetters()
                                       .Create();

    // Assert
    Assert.NotNull(actualForToAllSetters.String1);
    Assert.NotNull(actualForToAllSetters.String2);
    Assert.NotNull(actualForToAllSetters.String3);
    Assert.NotNull(actualForToAllSetters.String4);

    Assert.Null(actualForNotToAllSetters.String1);
    Assert.Null(actualForNotToAllSetters.String2);
    Assert.Null(actualForNotToAllSetters.String3);
    Assert.Null(actualForNotToAllSetters.String4);
}
```

<h4 id="ToSetter">ToSetter()</h4>
**Setter Injection**을 특정 프라퍼티에 매치시키는 방법에 따라
TestCommon은 `ToSetter`에 대한 오버로드 메소드들이 많은데,
이름에 의한 방법, Expressoin에 의한 방법 그리고 타입에 의한 방법과 같이 크게 3가지로 나눌 수 있다.
이름에 의한 방법과 Expression에 의한 방법은 특정 프라퍼티 선택에 있어서
유일성을 보장하지만 타입에 의한 방법은 그렇지 못하다.

아래 코드는 **이름에 의한 방법**으로 Setter Injection 설정을 보여주는데,
`Int1` 프라퍼티에 특정 값 `expectedInt1`이 입력됨을 알 수 있다.

```c#
[Fact]
public void ToSetter_ByName_SetsValueToCertainProperty()
{
    // Arrange
    var fixture = new Fixture();

    var expectedInt1 = fixture.Create<int>();

    // Act
    var actual = fixture.Build<SpecimenBuilderTestType>()
                        .ToSetter("Int1", expectedInt1)
                        .Create();

    // Assert
    Assert.Equal(expectedInt1, actual.Int1);
}
```

**Expressino에 의한 방법** 또한 이름에 의한 방법처럼 유일성을 보장하기 때문에, 아래 코드와 같이
특정 프라퍼티에 값을 입력할 수 있다.

```c#
[Fact]
public void ToSetter_ByExpression_SetsValueToCertainProperty()
{
    // Arrange
    var fixture = new Fixture();

    string expectedString = fixture.Create<string>();

    // Act
    var actual = fixture.Build<SpecimenBuilderTestType>()
                        .ToSetter(x => x.String1, expectedString)
                        .Create();

    // Assert
    Assert.Equal(expectedString, actual.String1);
}
```

**타입에 의한 방법**은 앞서 두 가지의 경우와는 다르게 유일성을 보장하지 못하기 때문에
아래 코드와 같이 주어진 타입과 같은 모든 프라퍼티에 대해 Setter Injection이 설정된다.
이때 액세스 한정자 `AccessModifiers`를 통해 Setter Injection 범위를 제한할 수 있는데
디폴트 값은 값은 `PublicOrInternal`이다.

```c#
[Fact]
public void ToSetter_ByType_SetsValueToAllPropertiesMatchedWithType()
{
    // Arrange
    var fixture = new Fixture();

    string expectedString = fixture.Create<string>();

    // Act
    var actual = fixture.Build<SpecimenBuilderTestType>()
                        .ToSetter(expectedString)  // AccessModifiers as PublicOrInternal
                        .Create();

    // Assert
    Assert.NotEqual(expectedString, actual.String1); // private
    Assert.NotEqual(expectedString, actual.String2); // private
    Assert.Equal(expectedString, actual.String3); // public
    Assert.Equal(expectedString, actual.String4); // internal
}
```

<h4 id="NotToSetter">NotToSetter()</h4>
`NotToAllSetters()`메소드는 `ToSetter()`메소드의 반대 개념이다.
특정 프라퍼티에 대한 Settter Injetion설정을 제거한다.

```c#
[Fact]
public void NotToSetter_DeletesCertainSetterInjection()
{
    // Arrange
    var fixture = new Fixture();

    // Act
    var actual = fixture.Build<SpecimenBuilderTestType>()
                        .ToAllSetters()
                        .NotToSetter(x => x.String3)
                        .Create();

    // Assert
    Assert.Null(actual.String3); // public
    Assert.NotNull(actual.String4); // internal
}
```

[Test Double]: http://xunitpatterns.com/Test%20Double.html
[이전 포스트]: /TestCommon-Parameterized-Anonymous-Creation-2/
[지난 포스트]: /TestCommon-Reflection-Type-객체-생성-1#AutoProperties
[Constructor Injection]: http://xunitpatterns.com/Dependency%20Injection.html#Constructor%20Injection
[Setter Injection]: http://xunitpatterns.com/Dependency%20Injection.html#Setter%20Injection
[SpecimenBuilderTestType]: /TestCommon-Parameterized-Anonymous-Creation-1#SpecimenBuilderTestType
