---
layout: post
title: TestCommon - Parameterized Anonymous Creation(5)
tags : [TestCommon]
ccl: ko
date : 2013-03-22 07:50:00 UTC
---
{% include JB/setup %}

이번 포스트에서는 Parameterized Anonymous Creation기능 설정(5가지) 중 마지막인 **Mock에 관한 속성 설정**에 대해서 알아보고자 한다.

1. 생성자 선택
2. 생성자 파라메타값 설정
3. 프라퍼티값 설정
4. 특정메소드 실행
5. **Mock에 관한 속성 설정**

[지난 포스트]에서 `Fixture.Create<T>`메소드를 통해 만들어진 임의의 객체는
[Moq] 프레임워크를 이용해 생성된 Mocked Instance라는 것을 보여 주었다.
그러나 아래 코드에서 보듯이 TestCommon의 `Fixture.Build<T>`메소드를 통해 생성된 임의의 객체는
기본적으로 Mocked Instance가 아니다.

```c#
[Fact]
public void Build_CreatesNotMockedInstance()
{
    // Arrange
    var fixture = new Fixture();

    // Act
    var actualForCreate = fixture.Create<object>();
    var actualForBuild = fixture.Build<object>().Create();

    // Assert
    Assert.IsNotType<object>(actualForCreate);
    ////This line shows that actualForCreate is assibnable to object.
    Assert.IsAssignableFrom<object>(actualForCreate);

    Assert.IsType<object>(actualForBuild);
}
```

<h4 id="AsMocked">AsMocked</h4>
`Fixture.Build<T>`메소드를 통해 생성된 임의의 객체를 Mocked Instance로 만들기는 어렵지 않다.
다음 코드를 보자. `AsMocked`메소드를 통해 Mocked Instance로 객체를 생성할 수 있다는 것을 보여준다.

```c#
[Fact]
public void AsMocked_CreatesMockedInstance()
{
    // Arrange
    var fixture = new Fixture();

    // Act
    var actual = fixture.Build<object>().AsMocked().Create();

    // Assert
    Assert.IsNotType<object>(actual);
    ////This line shows that actual is assibnable to object.
    Assert.IsAssignableFrom<object>(actual);
}
```
<!-- break -->

<h4 id="WithBehavior">Mock에 관한 속성</h4>
TestCommon에서 `Create<T>`/`Build<T>().AsMocked()`로 생성되는 Mocked Instace들은
기본적으로 다음과 같은 `Fixture`클래스의 **Mock에 관한 속성**들을 따른다.
(Mock에 관한 속성에 대한 사항은 [Moq 도움말]을 참고하기 바란다.)
"=>" 기호 다음에 나오는 값이 디폴트 설정이다.

`DefaultValue`에 대한 디폴트 값이 `Specimen`인데
이 값은 [Moq]에서 제공하는 것이 아니라 TestCommon에서 제공하는 것이다.
자세한 내용은 차후 포스팅 될 SpecimenMock 기능에 대한 글을 보기 바란다.


*   `Fixture.CallBase` => `false`
*   `Fixture.DefaultValue` => `Specimen`
*   `Fixture.Behavior` => `Loose`

Parameterized Anonymous Creation(`Build<T>`)은
특정 타입 `T`에 대해서 각각의 Mock 속성 값을 설정할 수 있는 메소드들을 아래와 같이 제공한다.
아래의 메소드로 설정되는 Mock 속성은 위 `Fixture`의 값을 무시하게 된다.

*   `WithCallBase(bool)`
*   `WithDefaultValue(MockDefaultValue)`
*   `WithBehavior(MockBehavior)`

아래 코드는 `WithBehavior`메소드 사용 예를 보여주는데, `actualForStrict`은 `AsMocked()`메소드를 호출하지 않았는데도
불구하고, Mocked Instace로 생성이 된다.
왜냐하면, 위 세 메소드 중 어느 하나만 호출을 하더라도 자동적으로 
`AsMocked()`메소드가 호출되기 때문이다. 따라서 명시적으로 `AsMocked()`메소드를 호출하여도 되지만 생략하여도 무방하다.

`WithCallBase`와 `WithDefaultValue`메소드의 테스트는 각자가 해 보길 바란다.

```c#
[Fact]
public void WithBehavior_SetsMockBehaviorToMockedInstance()
{
    // Arrange
    var fixture = new Fixture();

    // Act
    var actualForLoose = fixture.Build<IDisposable>().AsMocked().Create();
    var actualForStrict = fixture.Build<IDisposable>().WithBehavior(MockBehavior.Strict).Create();

    // Assert
    Assert.DoesNotThrow(actualForLoose.Dispose);
    Assert.Throws<MockException>(() => actualForStrict.Dispose());
}
```


<h4>What's Next?</h4>
이번 포스트를 끝으로 Parameterized Anonymous Creation 기능 설명은 마치고자 한다.
질문이 있으시면 언제든 댓글을 남겨주시기 바란다.
이후 이어질 포스트 내용은 TestCommon의 **Customization** 기능이다.


[지난 포스트]: /TestCommon-Reflection-Type-객체-생성-2/#Mocked%20Instance
[Moq]: https://github.com/Moq/moq4
[Moq 도움말]: http://code.google.com/p/moq/wiki/QuickStart#Customizing_Mock_Behavior
