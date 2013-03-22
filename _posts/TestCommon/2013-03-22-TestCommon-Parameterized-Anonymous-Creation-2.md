---
layout: post
title: TestCommon - Parameterized Anonymous Creation(2)
tags : [TestCommon]
ccl: ko
date : 2013-03-22 02:41:00 UTC
---
{% include JB/setup %}

[이전 포스트]에서는 아래와 같은 Parameterized Anonymous Creation기능 설정(5가지) 중
첫 번째 **생성자 선택**에 대해 알아보았다. 이번 포스트에서는 **생성자 파라메타값 설정**에 대해서 알아보고자 한다.

1. 생성자 선택
2. **생성자 파라메타값 설정**
2. 프라퍼티 값 설정
3. 특정메소드 실행
4. Mock에 관한 속성 설정

생성자 파라메타값 설정은 생성자에게 특정한 값을 전달하는 것 외에
**생성자 선택**에도 영향을 준다.
TestCommon에서는 디폴트로 `NonPrivate` 생성자들 중 파타메타 개수가 가장 적은 것을 선택하게 되어있다.
따라서, [SpecimenBuilderTestType] 타입 생성자는 `protected SpecimenBuilderTestType(byte byte1)`가 선택되어야 한다.
그런데 생성자에 대해 특정 파라메타값을 설정하게 되면, 그 **특정파라메타를 가지는
생성자가 우선순위**를 가지게 된다.

아래 코드에서 `int`타입의 `expectedInt1`값이 생성자 파라메타로 설정이 되었는데, 이 설정으로
생성자 선택이

*   "NonPrivate 생성자들 중 파타메타 개수가 가장 적은 것"에서 
*   "NonPrivate이고 int타입의 파라메타를 가지는 생성자들 중 파타메타 개수가 가장 적은 것"으로 바뀌게 된다.

결과적으로 [SpecimenBuilderTestType] 타입 생성자는
`public SpecimenBuilderTestType(string string1, int int1)`가 선택이 되어
`String1`은 `null`이 아니고, `String2`는 `null`이 되는 것이다.

```c#
[Fact]
public void ConstructorInjection_LetSelectCertainCtor()
{
    // Arrange
    var fixture = new Fixture();

    int expectedInt1 = fixture.Create<int>();

    // Act
    var actual = fixture.Build<SpecimenBuilderTestType>()
                        .ToCtor(expectedInt1)
                        .Create();

    // Assert
    Assert.NotNull(actual.String1);
    Assert.Null(actual.String2);
}
```

생성자 파라메타 설정 방법은 파라메타의 **타입에 의한 방법**, **이름에 의한 방법** 그리고 **포지션에 의한 방법**으로 3가지가 있다.
파라메타의 이름과 포지션은 유일성을 보장하기 때문에 파라메타 값 설정에 문제가 되지 않지만,
파라메타 타입의 경우는 유일성이 보장되지 못하기 때문에 타입이 같은 모든 생성자 파라메타에 적용이 된다.

<!-- break -->

#### 타입에 의한 방법
위에서 보여준 예제 코드 역시 타입에 의한 방법이었는데, `int`타입이 하나라서 유일성에 관한 문제를
알아보지 못했다. 그러면 다음 코드는 어떨까? 아래 코드에서 TestCommon은
"NonPrivate이고 string타입의 파라메타를 가지는 생성자들 중 파타메타 개수가 가장 많은 것"이라는 조건으로
`public SpecimenBuilderTestType(string string1, int int1, string string2)` 생성자를 선택하게 된다.
Assert결과에서 알 수 있듯이
`expectedString`값은 타입이 같은 `string1`과 `string2` 생성자 파라메타 모두에 적용이 된다는 것을 보여준다.

```c#
[Fact]
public void ConstructorInjection_ByType_PassesValueToAllSameTypedParameters()
{
    // Arrange
    var fixture = new Fixture();

    string expectedString = fixture.Create<string>();

    // Act
    var actual = fixture.Build<SpecimenBuilderTestType>()
                        .WithGreedyCtor()
                        .ToCtor(expectedString)
                        .Create();

    // Assert
    Assert.Same(expectedString, actual.String1);
    Assert.Same(expectedString, actual.String2);
}
```

#### 이름에 의한 방법
파라메타 이름은 유일성이 보장되기 때문에 특정 파라메타에 대한 값 설정이 가능하다.
아래 코드에서 확인 가능하자.

```c#
[Fact]
public void ConstructorInjection_ByName_PassesValueToSameNamedParameter()
{
    // Arrange
    var fixture = new Fixture();

    string expectedString = fixture.Create<string>();

    // Act
    var actual = fixture.Build<SpecimenBuilderTestType>()
                        .ToCtor("string2", expectedString)
                        .Create();

    // Assert
    Assert.NotEqual(expectedString, actual.String1);
    Assert.Same(expectedString, actual.String2);
}
```

#### 포지션에 의한 방법
파라메타 포지션 역시 유일성이 보장되기 때문에 특정 파라메타에 대한 값 설정이 가능하다.
아래 코드에서 확인 가능하자.

```c#
[Fact]
public void ConstructorInjection_ByPosition_PassesValueToSamePositionedParameter()
{
    // Arrange
    var fixture = new Fixture();

    string expectedString = fixture.Create<string>();

    // Act
    var actual = fixture.Build<SpecimenBuilderTestType>()
                        .ToCtor(0, expectedString)
                        .Create();

    // Assert
    Assert.Same(expectedString, actual.String1);
    Assert.NotEqual(expectedString, actual.String2);
}
```

[이전 포스트]: /TestCommon-Parameterized-Anonymous-Creation-1/
[SpecimenBuilderTestType]: /TestCommon-Parameterized-Anonymous-Creation-1#SpecimenBuilderTestType
