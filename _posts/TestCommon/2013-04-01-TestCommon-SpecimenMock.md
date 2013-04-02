---
layout: post
title: TestCommon - SpecimenMock
tags : [TestCommon]
ccl: ko
date : 2013-04-01 06:53:00 UTC
---
{% include JB/setup %}

<h3 id="MockDefaultValue">MockDefaultValue</h3>
TestCommon에서는 [Moq] 프레임워크를 사용하여 [Test Double]을 지원하고 있다.
Moq에서는 Mocking되지 않은 멤버(프라퍼티, 메소드)의 리턴 값을 `DefaultValue`라는 값(`Empty`/`Mock`)을 통해 설정할 수 있다.
Moq의 `DefaultValue`에 대해서는 [퀵스타트]를 참고하기 바란다.

한편, TestCommon에서는 Moq에서 제공하는 `DefaultValue`값과 더불어 `Specimen`이라는 기본값을 제공하기 위해
아래와 같은 `MockDefaultValue`을 사용한다.

```c#
public enum MockDefaultValue
{
    Empty = 0,
    Mock = 1,
    Specimen = 2
}
```


`MockDefaultValue`를 사용하는 Mock 객체를 생성하기 위해서 TestCommon에서는 Moq의 `Mock<T>` 클래스를 상속하는
`SpecimenMock<T>` 클래스를 정의하여 `DefaultValue` 프라퍼티를 아래와 같이 오버라이드한다.
`DefaultValue` 값이 `2`이면 TestCommon의 `MockDefaultValue.Specimen` 값으로 취급하게 된다.


```c#
public override DefaultValue DefaultValue
{
    get
    {
        return base.DefaultValue;
    }

    set
    {
        base.DefaultValue = value;
        switch (value)
        {
            case DefaultValue.Empty: // MockDefaultValue.Empty
            case DefaultValue.Mock:  // MockDefaultValue.Mock
                break;
            case (DefaultValue)2:    // MockDefaultValue.Specimen
                SetFixtureValueProvider();
                break;
            default:
                throw new ArgumentOutOfRangeException("value");
        }
    }
}
```

<!-- break -->

<h3 id="SpecimenMock&lt;T&gt; 객체 생성">SpecimenMock&lt;T&gt; 객체 생성</h3>

TestCommon에서 `SpecimenMock<T>` 타입의 객체는 생성자가 `public`으로 노출되지 않아
`new` 연산자로 객체를 직접 생성할 수 없으나,
`Fixture` 클래스의 `Create<T>` 또는 `Build<T>` 메소드를 통해 생성할 수 있다.

이때 `Fixture` 클래스의 `MockDefaultValue` 타입인 `DefaultValue` 프라퍼티를 통해
`Fixture` 클래스의 인스턴스(`fixture`)에서 생성되는 모든 `SpecimenMock<T>`
인스턴스에 대해 `MockDefaultValue` 값을 설정할 수 있다.(아래 코드에서 확인)

```c#
[Theory]
[InlineData(MockDefaultValue.Empty)]
[InlineData(MockDefaultValue.Mock)]
[InlineData(MockDefaultValue.Specimen)]
public void Fixture_AnonymousCreation_CreatesInstanceOfSpecimenMockAsTestDouble(
    MockDefaultValue expectedMockDefaultValue)
{
    // Arrange
    var fixture = new Fixture { DefaultValue = expectedMockDefaultValue };

    // Act
    var actual = fixture.Create<IDisposable>();

    // Assert
    var mock = actual.ToMock();

    string actualMockType = mock.GetType().ToString();
    Assert.Equal("TestCommon.SpecimenMock`1[System.IDisposable]", actualMockType);

    var actualMockDefaultValue = (MockDefaultValue)mock.DefaultValue;
    Assert.Equal(expectedMockDefaultValue, actualMockDefaultValue);
}
```

또는, 아래 코드와 같이 `Build<T>().WithDefaultValue(MockDefaultValue)` 메소드를 통해 특정 `T` 타입의 `SpecimenMock<T>` 인스턴스의
`MockDefaultValue` 값을 설정할 수 있다.

```c#
[Theory]
[InlineData(MockDefaultValue.Empty)]
[InlineData(MockDefaultValue.Mock)]
[InlineData(MockDefaultValue.Specimen)]
public void Fixture_ParameterizedAnonymousCreation_CreatesInstanceOfSpecimenMockAsTestDouble(
    MockDefaultValue expectedMockDefaultValue)
{
    // Arrange
    var fixture = new Fixture();

    // Act
    var actual = fixture.Build<IDisposable>()
                        .WithDefaultValue(expectedMockDefaultValue)
                        .Create();

    // Assert
    var mock = actual.ToMock();

    string actualMockType = mock.GetType().ToString();
    Assert.Equal("TestCommon.SpecimenMock`1[System.IDisposable]", actualMockType);

    var actualMockDefaultValue = (MockDefaultValue)mock.DefaultValue;
    Assert.Equal(expectedMockDefaultValue, actualMockDefaultValue);
}
```

<h3 id="MockDefaultValue.Specimen의 Anonymous Creation">MockDefaultValue.Specimen의 Anonymous Creation</h3>
TestCommon에서 생성되는 `SpecimenMock<T>` 타입 객체의 `MockDefaultValue` 값이 `Empty` 또는 `Mock`으로 설정이 되면,
생성된 `SpecimenMock<T>` 객체는 Moq의 `Mock<T>` 타입 객체와 동일하게 취급이 된다. 다음의 예제를 살펴 보자.

먼저, `MockDefaultValue` 값이 `Empty`일 때는 `ITestType` 타입 객체의 모든 프라퍼티 값 들은
각 타입에 대해 디폴트 값(`default(T)`)을 리턴하기 때문에 모두 `null`이 된다.

`MockDefaultValue` 값이 `Mock`일 때는

*   `string` 타입: `Mock<string>` 인스턴스를 생성할 수 없으므로 `default(string)` 값인 `null`을 리턴
*   `IDisposable`과 `EmptyParameterizedCtorType` 타입: 각 타입에 대한 Mocked(`Mock<T>.Object`) 인스턴스 리턴
*   `ParameterizedCtorType` 타입: Moq 프레임워크에서는 `IDisposable`과 `EmptyParameterizedCtorType` 타입과 같이 인터페이스이거나
 기본 생성자를 가지는 타입에 대해서만 Mock을 지원한다. 만약 `ParameterizedCtorType`과 같이 생성자 파라메타를 가지게 되면,
어떤 값을 넘겨야 할지 결정할 수 없기 때문에 `ArgumentException` 예외를 발생시키게 된다.

`MockDefaultValue` 값이 `Specimen`일 때는 TestCommon의 **Anonymous Creation** 기능을 통해 
아래 코드 결과와 같이 모든 타입에 대해 익명 객체를 생성할 수 있게 된다.
이러한 TestCommon의 SpecimenMock 기능은 앞서 `MockDefaultValue` 값이 `Mock`일 때 `ArgumentException` 예외를 발생시키는 경우처럼
테스트 목적으로 특정 타입(`ParameterizedCtorType`)에 대해 기본 생성자를 만들지 않고도 Mocked 객체생성을 가능하게 한다.

```c#
public interface ITestType
{
    string GetValue { get; }

    IDisposable Disposable { get; }

    EmptyParameterizedCtorType EmptyParameterizedCtorType { get; }

    ParameterizedCtorType ParameterizedCtorType { get; }
}

public class EmptyParameterizedCtorType
{
}

public class ParameterizedCtorType
{
    public ParameterizedCtorType(int intValue)
    {
    }
}

[Fact]
public void SpecimenMock_CanReturnAnonymousObjectByAnonymousCreation()
{
    // Arrange
    var fixture = new Fixture();

    var mockedForEmpty = fixture.Build<ITestType>()
                                .WithDefaultValue(MockDefaultValue.Empty)
                                .Create();

    var mockedForMock = fixture.Build<ITestType>()
                               .WithDefaultValue(MockDefaultValue.Mock)
                               .Create();

    var mockedForSpecimen = fixture.Build<ITestType>()
                                   .WithDefaultValue(MockDefaultValue.Specimen)
                                   .Create();

    // Act
    // Assert
    Assert.Null(mockedForEmpty.GetValue);
    Assert.Null(mockedForEmpty.Disposable);
    Assert.Null(mockedForEmpty.EmptyParameterizedCtorType);
    Assert.Null(mockedForEmpty.ParameterizedCtorType);

    Assert.Null(mockedForMock.GetValue);
    Assert.NotNull(mockedForMock.Disposable);
    Assert.NotNull(mockedForMock.EmptyParameterizedCtorType);
    Assert.Throws<ArgumentException>(() => mockedForMock.ParameterizedCtorType);

    Assert.NotNull(mockedForSpecimen.GetValue);
    Assert.NotNull(mockedForSpecimen.Disposable);
    Assert.NotNull(mockedForSpecimen.EmptyParameterizedCtorType);
    Assert.NotNull(mockedForSpecimen.ParameterizedCtorType);
}
```
또한 아래 코드와 같이 `Fixture` 타입 객체(`fixture`)의 **Cutomization**된 객체(`expected`) 또한
Mocked 객체(`actual`)의 멤버로 이용될 수 있다.

```c#
[Fact]
public void SpeicmenMock_CanReturnCustomizedValue()
{
    // Arrange
    var fixure = new Fixture();
    var expected = fixure.Create<IDisposable>();
    fixure.Customize(() => expected);

    // Act
    var actual = fixure.Create<ITestType>();

    // Assert
    Assert.Same(expected, actual.Disposable);
}
```

[Moq]: https://github.com/Moq/moq4
[Test Double]: http://xunitpatterns.com/Test%20Double.html
[퀵스타트]: https://code.google.com/p/moq/wiki/QuickStart