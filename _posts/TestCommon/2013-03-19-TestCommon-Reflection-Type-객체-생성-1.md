---
layout: post
title: TestCommon - Reflection Type 객체 생성(1)
tags : [TestCommon]
ccl: ko
date : 2013-03-19 04:50:00 UTC
---
{% include JB/setup %}

이전 포스트에서 Non Reflection Type에 대해 TestCommon이 임의의 객체를 어떻게 생성하는지 알아보았다.
Non Reflection Type 이 외의 모든 타입은 Reflection Type이며 Reflection을 통해 객체를 생성하게 된다.

TestCommon이 Reflection Type 객체를 어떻게 생성하는지를 아래와 같은 `Person` 클래스를 통해 알아보자.

```c#
public class Person
{
    public Person()
    {
    }

    public Person(string name)
    {
        Name = name;
    }

    private Person(string name, int age)
    {
        Name = "Used Private Constructor";
    }

    public string Name { get; private set; }

    public int? Age { get; set; }

    public void InjectName(string name)
    {
        Name = name;
    }
}
```

<!-- break -->

### Modest/Greedy 생성자
TestCommon은 기본적으로 `private`이 아닌 생성자들 중 가장 생성자 파라메타 수가 적은(Modest)
생성자를 통해 객체를 생성한다. `Person` 클래스의 생성자를 중에서는 아무 생성자 파라메타를 취하지 않는 기본 생성자가 이에 해당한다.
따라서 아래 코드에서 보듯이 `Name`과 `Age`프라퍼트는 모두 `null`이 된다.

```c#
[Fact]
public void TestCommon_CreatesInstanceUsingModestCtorAsDefault()
{
    var fixture = new Fixture();

    var actual = fixture.Create<Person>();

    Assert.Null(actual.Name);
    Assert.Null(actual.Age);
}
```

`Fixture.IsGreedy`값이 디폴트로 `false`이나 이 값을 `true`로 설정하게 되면, `Person`의 생성자는 기본 생성자가 아니라,
`string`타입의 `name` 값을 갖는 생성자 `public Person(string name)`를 객체생성에 이용하게 된다.

```c#
[Fact]
public void TestCommon_CanCreateInstanceUsingGreedyCtor()
{
    var fixture = new Fixture { IsGreedy = true };

    var actual = fixture.Create<Person>();

    Assert.NotNull(actual.Name);
    Assert.Null(actual.Age);
}
```

### 생성자 AccessModifiers
TestCommon은 Reflection Type의 객체를 생성할 때, `private`이 아닌 생성자들을 기초로 하지만
`AccessModifiers` `enum`타입의 `Fixture.Ctors` 프라퍼티 값 설정을 통해 이를 바꿀 수 있다.

```c#
[Fact]
public void TestCommon_CreatesInstanceUsingValueOfCtors()
{
    var fixture = new Fixture { IsGreedy = true };

    var actualForDefaultCtors = fixture.Create<Person>();
    Assert.NotNull(actualForDefaultCtors.Name);
    Assert.NotEqual("Used Private Constructor", actualForDefaultCtors.Name);

    fixture.Ctors = AccessModifiers.Public | AccessModifiers.Private;
    var actualForPublicOrPrivateCtors = fixture.Create<Person>();
    Assert.Equal("Used Private Constructor", actualForPublicOrPrivateCtors.Name);
}
```

### AutoProperties
특정 객체의 의존객체들([DOC: Depended-On Component][DOC])를 설정하는 방법은 
**Modest/Greedy**/**AccessModifiers**에 따른 생성자 선택을 통한 익명의 생성자 파라메타 값을 넘겨주는 방법뿐 아니라
프라퍼티 값 설정을 통해서도 의존객체들을 설정할 수 있다.

TestCommon에서는 이에 대한 메타니즘은 `Fixture.AutoProperties` 프라퍼티를 통해 제공하고 있는데
`AccessModifiers` 타입의 `enum` 값을 아래 예제코드처럼 설정할 수 있다.
`AutoProperties` 프라퍼티는 디폴트로 `AccessModifiers.None`값을 가지며 이 값은 객체 생성후 어떤 프라퍼티에 대한 `setter`도 호출하지 않는다는 뜻이다.
아래코드처럼 만약 이 값을 `None`이 아닌 값을 설정하게 되면, 그 값에 대응되는 프라퍼티의 `setter`들을 익명의 객체(값)을 가지고 설정하게 된다.

```c#
[Fact]
public void TestCommon_CreatesInstanceUsingAutoProperties()
{
    var fixture = new Fixture();

    // None
    var actualForDefaultAutoProperties = fixture.Create<Person>();
    Assert.Null(actualForDefaultAutoProperties.Age);

    // Specifying the certain AsscessModifiers as Public.
    fixture.AutoProperties = AccessModifiers.Public;
    var actualForPublicAutoProperties = fixture.Create<Person>();
    Assert.NotNull(actualForPublicAutoProperties.Age);
}
```

### 각각의 Reflection Type에 대한 커스터마제이션
위에서 언급한 세 가지  - Modest/Greedy 생성자, 생성자 AccessModifiers 그리고 AutoProperties - 설정은
TestCommon에서 생성하는 모든 Reflection Type 들에 영향을 주게 된다.
만약 특정한 한 타입에 대해 위와 같은 설정이 필요하면 어떻게 해야 할까?
이는 `Fixture` 클래스의 `Customize` 메소드로 설정할 수 있는데, 이 후에 포스팅되는 글을 참고하기 바란다.


[DOC]: http://xunitpatterns.com/DOC.html