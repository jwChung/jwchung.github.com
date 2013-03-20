---
layout: post
title: TestCommon - Reflection Type 객체 생성(3)
tags : [TestCommon]
ccl: ko
date : 2013-03-19 07:43:00 UTC
---
{% include JB/setup %}

TestCommon의 `Fixture.Create<T>`메소드는 `T`타입의 Mocked Instance를 리턴한다고 [이전 포스트]에서 설명하였다.
그렇다면 아래와 같은 클래스(`InternalCtorTestClass`)에 대해 `Fixture.Create<InternalCtorTestClass>`메소드를 실행하면 어떻게 될까?
TestCommon에서는 [이전 포스트]에서 설명하였듯이 Private이 아닌 생성자중 파라메타 수가 가장 적은 
생성자를 택하게 되어있다.


```c#
public class InternalCtorTestClass
{
    internal InternalCtorTestClass()
    {
    }
}
```

`InternalCtorTestClass`에 대해서는 `internal InternalCtorTestClass()` 생성자가 선택이 되는데 문제는 Internal 생성자를 [Moq] 프레임워크에서 지원하기 위해서는
아래아 같은 `InternalsVisibleToAttribute` 설정이 필요하다는 것이다. 이렇게 함으로써 `DynamicProxyGenAssembly2` 어셈블리에서 
해당 어셈블리(아래 `InternalsVisibleToAttribute`설정을 가지는 어셈블리)의 Internal 멤버에 접근이 가능한 것이다.

```c#
[assembly: InternalsVisibleTo("DynamicProxyGenAssembly2, PublicKey=0024000004800000940000000602000000240000525341310004000001000100c547cac37abd99c8db225ef2f6c8a3602f3b3606cc9891605d02baa56104f4cfc0734aa39b93bf7852f7d9266654753cc297e7d2edfe0bac1cdcf9f717241550e0a7b191195b7667bb4f64bcb8e2121380fd1d9d46ad2d92d2d15605093924cceaf74c4861eff62abf69b9291ed0a340e113be11e6a7d3113e92484cf7045cc7")]
```

먼저 위의 `InternalsVisibleToAttribute` 설정을 하지 않으면 아래와 같은 결과가 나오는데, TestCommon에서는 먼저 Mocked Instance생성을 시도한 후,
실패하게 되면 생성자를 직접 호출하여 객체 생성을 시도하게 된다.

만약 위와 같은 `InternalsVisibleToAttribute` 설정을 하게 되면, TestCommon에서는 `InternalCtorTestClass` 타입에 대해 Mocked Instance를 리턴함으로
아래의 `Assert.IsType<InternalCtorTestClass>(actual)` 코드는 실패하게 된다.

```c#
[Fact]
public void TestCommon_CannotBeMocked_CreatesNotMockedInstance()
{
    // Arrange
    var fixture = new Fixture();

    // Act
    var actual = fixture.Create<InternalCtorTestClass>();

    // Assert
    Assert.IsType<InternalCtorTestClass>(actual);
}
```

<!-- break -->

다음은 설정된 `Fixture.Ctors`의 `AccessModifiers`값이 유효하지 않을 때를 살펴보자.

아래 코드에서 `Ctors`값을 `AccessModifiers.Public`으로 설정한다면,
`InternalCtorTestClass`타입에 대해 유효한 생성자가 하나도 없게 된다.
그럼 이런 경우 객체를 생성할 수 없는 것일까? 아래의 결과에서 보듯 객체가 생성됨을 알 수 있다.
설정된 `Ctors`의 값이 먼저 고려하게 되고, 만약 이를 만족하는 생성자가 하나도 없을 때는 
`AccessModifiers.All`값이 적용되어 객체 생성을 시도하게 된다. 이때에도 먼저 Mocked Instance객체 생성 시도를 먼저하게 되고,
만약 실패하면 생성자 자체를 실행하여 그 결과를 리턴하게 된다.

```c#
[Fact]
public void TestCommon_InvalidAccessModifiersForCtors_CreatesNotMockedInstance()
{
    // Arrange
    var fixture = new Fixture { Ctors = AccessModifiers.Public };

    // Act
    var actual = fixture.Create<InternalCtorTestClass>();

    // Assert
    Assert.IsType<InternalCtorTestClass>(actual);
}
```

**요약하면,`Fixture.Create<T>`메소드는 아래와 같은 우선순위를 가지고 객체를 생성하게 된다.**

1.  `Fixture`의 `Ctors`(`AccessModifiers`)값에 해당하는 생성자들을 통해 Mocked Instance를 생성한다.
2.  `Fixture`의 `Ctors`(`AccessModifiers`)값에 해당하는 생성자들을 통해 Not Mocked Instance를 생성한다.
3.  `AccessModifiers.All` 값에 해당하는 생성자들을 통해 Mocked Instance를 생성한다.
4.  `AccessModifiers.All` 값에 해당하는 생성자들을 통해 Not Mocked Instance를 생성한다.

[이전 포스트]: /TestCommon-Reflection-Type-객체-생성-2
[Moq]: https://github.com/Moq/moq4