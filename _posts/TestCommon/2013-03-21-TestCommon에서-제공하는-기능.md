---
layout: post
title: TestCommon에서 제공하는 기능
tags : [TestCommon]
ccl: ko
date : 2013-03-20 15:03:00 UTC
---
{% include JB/setup %}

[TestCommon]에서 제공하는 기능을 크게 나누면 다음과 같이 5가지로 분류할 수 있다.

*   Anonymous Creation
*   Parameterized Anonymous Creation
*   Customization
*   AutoTheory
*   SpecimenMock

#### Anonymous Creation
[TestCommon과 관련한 이전 포스트들]\(이 포스트 이전)은 Anonymous Creation 기능을 설명하기 위한 포스트들이였다.
Anonymous Creation 기능은 `Fixture` 클래스의 `Create<T>` 메소드에 의해 구현이 되는데
이전 포스트들에서 언급한 바와 같이 특정 `T` 타입에 대해서 익명의 객체(또는 값)를 생성하여 반환하는 것이다.
이 기능에 관한 내용은 이전 포스트들을 참고하기 바라며, 질문사항이 있으면 언제든 댓글을 남겨 주시기 바란다.

#### Parameterized Anonymous Creation
이후 포스트들은 Anonymous Creation 외 나머지 4가지 기능에 관한 내용이 될 텐데,
이번 포스트에서 각각의 기능에 대해 간략하게나마 살펴보도록 하자.

이전 포스트([TestCommon이란])에서 살펴 본 [DepositTestUsingFixtureClass 테스트]에서
아래와 같은 코드가 나오는데, `BankAccount` 타입 객체 생성시 `initialAmount` 값을 `BankAccount`의 생성자 파라메타로 넘겨
의존객체([DOC])를 사용자가 설정할 수 있다는 것을 보여주고 있다.
이 기능을 가리켜 Parameterized Anonymous Creation이라 하는데, 생성자 파라메타를 통해 의존객체(DOC)를
설정하는 것 외에 객체 생성을 위한 생성자 선택, 프라퍼티 Setter통한 의존객체 설정 그리고 객체 생성시
필요한 모든 제반사항을 사용자가 설정할 수 있는 것들이 이 기능에 속한다.
Parameterized Anonymous Creation 기능은 `Fixture` 클래스의 `Build<T>` 메소드를 통해 구현되며,
이후에 포스팅 되는 글에서 자세한 사항을 확인하기 바란다.

```c#
var sut = fixture.Build<BankAccount>().ToCtor(initialAmount).Create();
```
<!-- break -->

#### Customization
**Customization**기능이란 특정 타입(`T`)의 객체 생성에 대해 사용자가 customization할 수 있는 방법을 제공해,
`Create<T>` 메소드 실행시 그 설정된 customization을 통해 객체가 생성되는 기능을 말한다.
이 기능은 `Fixture` 클래스의 `Customize<T>` 메소드를 통해 구현된다.
아래의 코드에서는 `string` 타입에 대한 간단한 customization의 예를 보여주는데, 이렇게 customization되면
`fixture`의 `Create<string>` 메소드는 항상 `"anonymous string"`이란 값을 리턴하게 된다.

```c#
var fixture = new Fixture();
fixture.Customize(() => "anonymous string");
```

#### AutoTheory
AutoTheory 기능은 [TestCommon이란] 포스트에서 잠깐 언급이 된 적있다.
이 기능은 매 테스트마다 필요한 객체를 생성하기 위해 `Fixture` 클래스의 `Create<T>` 혹은
`Build<T>` 메소드를 실행할  필요없이 `AutoTheoryAttribute` 클래스를 사용하여,
테스트 메소드 파라메타로 필요한 객체를 넘겨 받을 수 있는 기능을 말한다.

[DepositTestUsingFixtureClass 테스트]에서 `AutoTheory`를 사용하게 되면
[DepositTestUsingAutoTheory 테스트]와 같이 테스트 코드가 간결해 질 수 있다.

#### SpecimenMock
TestCommon에서의 Mock객체생성은 [Moq]프레임워크를 이용하게 된다.
Moq프레임워크에서는 `Mock<T>` 클래스의 `DefaultValue`라는 속성을 통해
Mock Behavior를 Customizing할 수 있는 방법을 제공하는데, 이에 대한 자세한 내용은
[여기](https://code.google.com/p/moq/wiki/QuickStart#Customizing_Mock_Behavior)에서 확인할 수 있다.
Moq프레임워크에서는 `DefaultValue` enum에 대해 `Empty`와 `Mock`값 만을 제공하는데
TestCommon에서는 `DefaultValue` enum값에 `Specimen`값을 추가하여 생성되는 모든 Mock객체의
디폴트 `DefaultValue`를 `Specimen`로 설정한다.

아래의 `IPerson`타입의 `actual`객체는 Mocked Intance인데 `DefaultValue` 속성이 `Specimen`으로 설정되어
`actual.Name` 값이 `"get_Name: anonymous string"`이 된다. 만약 `DefaultValue` 속성이 `Empty`또는 `Mock`이 되다면
`actual.Name` 값은 `null`이 된다.

```c#
public interface IPerson
{
    string Name { get; }
}

[Fact]
public void TestCommon_ReturnsSpecimenMockForReflectionType()
{
    // Arrange
    var fixture = new Fixture();
    string expectedName = "anonymous string";
    fixture.Customize(() => expectedName);

    // Act
    var actual = fixture.Create<IPerson>();

    // Assert
    Assert.Equal("get_Name: " + expectedName, actual.Name);

    actual.ToMock().DefaultValue = DefaultValue.Mock;
    Assert.Null(actual.Name);
}
```

[TestCommon]: https://github.com/jwChung/TestCommon
[TestCommon과 관련한 이전 포스트들]: /tags.html#TestCommon-ref
[TestCommon이란]: /TestCommon이란
[DepositTestUsingFixtureClass 테스트]: /TestCommon이란#DepositTestUsingFixtureClass
[DOC]: http://xunitpatterns.com/DOC.html
[DepositTestUsingAutoTheory 테스트]: /TestCommon이란#DepositTestUsingAutoTheory
[Moq]: https://github.com/Moq/moq4