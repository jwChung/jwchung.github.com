---
layout: post
title: TestCommon - Reflection Type 객체 생성(2)
tags : [TestCommon]
ccl: ko
date : 2013-03-19 06:04:00 UTC
---
{% include JB/setup %}

*이번 포스트를 이해하기 위해서는 [Test Double]과 [Moq] 프레임워크에 대한 이해가 선행되어야한다.
[Moq]의 도움말은 [여기](https://code.google.com/p/moq/wiki/QuickStart)를 참고하기 바란다.*

<h3 id="Mocked Instance">Mocked Instance</h3>
TestCommon의 `Fixture.Creat<T>` 메소드를 통해 Reflection Type 객체를 생성하면,
디폴트로 [Moq] 프레임워크를 이용하여, Mocked Instance를 생성한다. 아래코드를 보자.
`Person` 클래스는 [이전 포스트](/TestCommon-Reflection-Type-객체-생성-1)를 참고하자.
`actual` 변수의 결과는 `Person` 타입과 같지 않고, `Castle.Proxies.PersonProxy` 타입이라는 것을 보여주는데,
이 것은 `actual`이 [Moq] 프레임워크를 이용하여 `new Mock<Person>().Object`를 통해 설정되었다는 것을 의미한다.
즉, `actual`은 **Mocked Instance**이다.


```c#
[Fact]
public void TestCommon_CreatesMockedInstanceAsDefault()
{
    var fixture = new Fixture();

    var actual = fixture.Create<Person>();

    Assert.IsNotType<Person>(actual);
    Console.WriteLine(actual.GetType().ToString());  // Castle.Proxies.PersonProxy
}
```

<!-- break -->

### Abstract 타입 객체 생성
TestCommon이 [Moq] 프레임워크를 통해 Mocked Instance를 생성하므로, 인터페이스와 추상클래스에 대해서도 아래와 같이 객체 생성이 가능하다.

```c#
[Fact]
public void TestCommon_CreatesMockedInstanceForInterface()
{
    var fixture = new Fixture();

    var actual = fixture.Create<IDisposable>();

    Assert.NotNull(actual);
}

[Fact]
public void TestCommon_CreatesMockedInstanceForAbstractClass()
{
    var fixture = new Fixture();

    var actual = fixture.Create<TextWriter>();

    Assert.NotNull(actual);
}
```

### 왜 Mocked Instance인가?
아직 **Parameterized Anonymous Creation 기능**에 대한 글을 포스트하지 않아서 아래코드가 좀 생소할 수 있지만,
**Parameterized Anonymous Creation 기능**이란 말 그대로 객체 생성시, 특정 [의존객체(DOC)][DOC]를 사용자가 설정할 수 있다는 말이다.

아래에서 `ToMock()`이라는 확장메소드는 [Moq] 프레임워크에서 제공하는 `Mock.Get<T>()`메소드와 동일하다.
즉 Mocked Instance에 대한 Mock Instance(`Mock<T>` 타입)를 반환한다.

아래 예제코드에서 보듯이 `textWriter`변수가 **Mocked Instance**로 설정됨으로,
실제 `TextWriter` 클래스를 이용하지 않고 [Test Double]을 통하여 `Log`메소드를 테스트할 수 있게 해 준다.
즉, TestCommon의 `Fixture.Creat<T>` 메소드를 통해 반환되는 Mocked Instance는
실제 `T` 타입의 객체와 동일하게 행동하지만, 사용자로 하여금 **객체 행동을 바꿀 수 있는 수단을 제공**하고 있다.

```c#
public class LoggerClass
{
    private readonly TextWriter _textWriter;

    public LoggerClass(TextWriter textWriter)
    {
        _textWriter = textWriter;
    }

    public void Log()
    {
        _textWriter.Write("message");
    }
}

[Fact]
public void TestCommon_CreatesMockedInstanceToOverideMethod()
{
    // Arrange
    var fixture = new Fixture();

    var textWriter = fixture.Create<TextWriter>();
    var sut = fixture.Build<LoggerClass>().ToCtor(textWriter).Create();

    // Act
    sut.Log();

    // Assert
    textWriter.ToMock().Verify(x => x.Write("message"), Times.Once());
}
```


[Test Double]: http://xunitpatterns.com/Using%20Test%20Doubles.html
[Moq]: https://github.com/Moq/moq4
[DOC]: http://xunitpatterns.com/DOC.html
