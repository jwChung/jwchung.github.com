---
layout: post
title: TestCommon - Parameterized Anonymous Creation(1)
tags : [TestCommon]
ccl: ko
date : 2013-03-21 07:56:00 UTC
---
{% include JB/setup %}

Parameterized Anonymous Creation 기능은 특정 타입의 객체 생성시
필요한 제반사항을 사용자가 설정할 수 있도록 하는 기능으로 `Build<T>` 메소드를 통해 구현된다.
앞서 살펴본 Anonymous Creation 기능을 통해서는 사용자가 생성자 파라메타를 넘긴다던가
프라퍼티에 특정값을 설정하는 것과 같이, 사용자가 의존객체([DOC])를 설정을 할 수가 없었다.

TestCommon의 Parameterized Anonymous Creation 기능을 통해서 사용자는 의존객체(DOC)를 설정할 수 있는데,
이 것은 [Test Double]을 테스트 객체에 설정하는 것을 가능하게 한다.
여기서 __Test Double__은 _테스트를 수행하기 위해서 실제 컴포넌트 역할을 대체하는 기능을 가진 객체나 컴포넌트_정도로 이해할 수 있다.
[단위테스트]에서 실제 의존객체를 대신하는 Test Double을 테스트 객체에 설정한다는 것은 중요한 의미를 갖는다.
다음의 `LoggerClass` 예제를 보자.

`Log_ShouldLogCorrectMessage`는 `Log`메소드에 대한 단위테스트인데
`TextWriter`타입에 대해서 __Test Double__(`textWriter`)객체를 사용하고 있다.
따라서 `Log_ShouldLogCorrectMessage` 단위테스트에서는 `Wirte`메소드가 어떻게 구현되었는지에 구애받지 않고
`"message"`인자로 호출되었는지만 테스트하게 된다. 단위테스트의 이러한 특성을 가리켜 테스트 고립(Isolation)이라한다.
사실 _Parameterized Anonymous Creation 기능의 주 목적이 이 Test Double설정에 있다_고해도 과언이 아니다.

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
public void Log_ShouldLogCorrectMessage()
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

<!-- break -->

객체 생성시 필요한 제반사항은, 아래와 같이 5가지로 나누어 살펴볼 수 있는데,
[지난 포스트\(TestCommon - Reflection Type 객체 생성(1))]에서 설명한
_세 가지(Modest/Greedy 생성자, 생성자 AccessModifiers 그리고 AutoProperties)설정은
여기 Parameterized Anonymous Creation 기능에도 그대로 적용이 된다_.

Parameterized Anonymous Creation 기능에서의 사용자 설정은 아래와 같은 순서로 적용이 되며
예를들어 프라퍼티 값 설정 후, 생성자 파라메타값 설정은 할 수가 없다. 그러나 불필요한 설정은 건너뛰어
생성자 파라메타값 설정후, 특정메소드 실행은 유효한다.

1. 생성자 선택
2. 생성자 파라메타값 설정
3. 프라퍼티값 설정
4. 특정메소드 실행
5. Mock에 관한 속성 설정

아래 `SpecimenBuilderTestType`는 Parameterized Anonymous Creation 기능 설명을 위한
테스트 클래스이다.

<a id="SpecimenBuilderTestType"></a>

```c#
public class SpecimenBuilderTestType
{
    public SpecimenBuilderTestType(string string1, int int1, string string2)
    {
        String1 = string1;
        Int1 = int1;
        String2 = string2;
    }

    public SpecimenBuilderTestType(string string1, int int1)
    {
        String1 = string1;
        Int1 = int1;
    }

    protected SpecimenBuilderTestType(byte byte1)
    {
        Byte1 = byte1;
    }

    public byte Byte1 { get; private set; }

    public string String1 { get; private set; }

    public int Int1 { get; private set; }

    public string String2 { get; private set; }

    public string String3 { get; set; }

    public string String4 { get; internal set; }
}
```

<h3>생성자 선택</h3>

Parameterized Anonymous Creation을 통해 객체를 생성하기 위해서는 우선
생성자를 선택하여야 한다.

#### WithCtors
`WithCtors`메소드는 생성자의 액세스 한정자 범위를 정하는 역할을 한다.
생성자의 액세스 한정자 범위는 디폴트로 `Fixture.Ctors`값(`NotPrivate`)을 따르게 된다.
그러나 `WithCtors`메소드를 호출하게 되면 이 디폴트값을 무시된다.

아래 코드에서 보듯 `WithCtors(AccessModifiers.Public)`을 호출함으로써 TestCommon은
생성자를 `public SpecimenBuilderTestType(string string1, int int1)`를 선택하게 되는 것이다. 따라서
`String1`의 프라퍼티는 `null`이 아니게 된다.

```c#
[Fact]
public void WithCtors_DelimitCtorAccessModifier()
{
    // Arrange
    var fixture = new Fixture();

    // Act
    var actual = fixture.Build<SpecimenBuilderTestType>()
                        .WithCtors(AccessModifiers.Public)
                        .Create();

    // Assert
    Assert.NotNull(actual.String1);
}
```

<h4>WithModestCtor/WithGreedyCtor</h4>
`WithModestCtor/WithGreedyCtor` 메소드는 파라메타 수가 가장 적은 생성자를 선택할 것인가(`WithModestCtor`),
아니면 반대로 파라메타 수가 가장 많은 생성자를 선택할 것인가(`WithGreedyCtor`)를 결정하게 해준다.
이 설정은 디폴트로 `Fixture.IsGreedy`값(false)를 따르게되며 `WithModestCtor/WithGreedyCtor` 메소드가 호출이 되면,
디폴트 값은 무시된다.

아래 코드를 보면 TestCommon은 디폴트로 `Fixture.IsGreedy`값이 false이고
`Fixture.Ctors`값이 `NotPrivate`이기 때문에 `public SpecimenBuilderTestType(string string1, int int1)` 생성자가
선택이 된다. 그러나 `.WithGreedyCtor()` 메소드 호출로 TestCommon은 
`public SpecimenBuilderTestType(string string1, int int1, string string2)`생성자를 이용하게 된다.


```c#
[Fact]
public void WithGreedyCtor_LetUseGreedyCtor()
{
    // Arrange
    var fixture = new Fixture();

    // Act
    var actual = fixture.Build<SpecimenBuilderTestType>()
                        .WithGreedyCtor()
                        .Create();

    // Assert
    Assert.NotNull(actual.String2);
}
```

#### WithCtor
앞서 `WithModestCtor/WithGreedyCtor` 메소드는 파라메타 수가 많으냐 적으냐에 따라 생성자가 결정이 되었으나,
`WithCtor`메소드를 사용하게 되면 특정 메소드를 선택할 수 있게 해 준다.
`WithCtor`메소드를 사용시에는 `WithCtors`메소드 또는 `Fixture.Ctors`값에 따른 생성자 액세스 한정자 제약을 받게 된다.
따라서 생성자 한정자에 의해 한번 필터링 된 결과가 `WithCtor`메소드에 전달되게 된다.

아래 코드에서 `Fixture.Ctors`값이 `NotPrivate`이기 때문에 `SpecimenBuilderTestType`타입의 모든 생성자가 `WithCtor`메소드로 전달이 되고,
생성자 파라메타 수가 1인 생성자가 선택되어 결과적으로 `protected SpecimenBuilderTestType(byte byte1)` 생성자가 객체생성에 이용이 된다.

```c#
[Fact]
public void WithCtor_LetSelectCertainCtor()
{
    // Arrange
    var fixture = new Fixture();

    // Act
    var actual = fixture.Build<SpecimenBuilderTestType>()
                        .WithCtor(cs => cs.Single(c => c.GetParameters().Length == 1))
                        .Create();

    // Assert
    Assert.Null(actual.String1);
    Assert.Null(actual.String2);
}
```

### What's Next?
이번 포스트에서는 Parameterized Anonymous Creation 기능 설정 5가지 중
첫 번째 **생성자 선택**에 대해 알아보았다.
이후 포스트들에서 나머지 네 가지 설정에 대해 자세히 알아보도록 하자.


[단위테스트]: http://ko.wikipedia.org/wiki/%EC%9C%A0%EB%8B%9B_%ED%85%8C%EC%8A%A4%ED%8A%B8
[Test Double]: http://xunitpatterns.com/Test%20Double.html
[지난 포스트\(TestCommon - Reflection Type 객체 생성(1))]: /TestCommon-Reflection-Type-객체-생성-1/
[DOC]: http://xunitpatterns.com/DOC.html
