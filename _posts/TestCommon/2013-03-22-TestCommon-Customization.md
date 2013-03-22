---
layout: post
title: TestCommon - Customizatoin
tags : [TestCommon]
ccl: ko
date : 2013-03-22 14:52:00 UTC
---
{% include JB/setup %}

TestCommon에서의 **Customization**기능이란 특정 타입에 대한 객체생성 방법을 설정하는 것을 말한다.
이 기능은 아래와 같은 `Fixture`클래스의 `Customize`메소드들로 구현된다.
1, 2번 메소드는 Customization의 대상 타입에 대해 제네릭 파라메타를 사용하고, 3, 4번 메소드는 `Type` 파라메타를 사용한다.
따라서 3, 4번 메소드는 1, 2번 메소드 사용과 별반 차이가 없으므로 본 글에서는 제네릭 파라메타를 사용하는
1, 2번 메소드에 대해서만 설명하도록 하겠다.

1.  `Customize<T>(Func<T>)`
2.  `Customize<T>(Func<ISpecimenBuilder<T>, T>)`
3.  `Customize(Type, Func<object>)`
4.  `Customize(Type, Func<ISpecimenBuilder<object>, object>)`

<!-- break -->


<h4 id="Customize&lt;T&gt;(Func&lt;T&gt;)">Customize&lt;T&gt;(Func&lt;T&gt;)</h4>
아래 코드는 `Customize<int>(Func<int>)`메소드를 이용하여
`int`타입에 대해 100부터 1씩 증가하는 값을 설정하고 있다.
이를 통해 `fixture` 객체에서 생성되는 `int` 값은 모두 이 설정을 따르게 되어있다.
아래 코드에서, 직접적으로 `Create<int>()`메소드를 통해 생성되는 `int` 값도 앞서의 설정을 따르게 되지만,
`person.Age`값처럼 생성자 파라메타도 이 설정을 따르게 된다.

```c#
[Fact]
public void Customize_Func_CustomizesConstructingInstanceOfCertainType()
{
    // Arrange
    var fixture = new Fixture();

    int initial = 100;

    // Act
    fixture.Customize(() => initial++);

    // Assert
    Assert.Equal(100, fixture.Create<int>());

    var person = fixture.Create<Person>();
    Assert.Equal(101, person.Age);
}

public class Person
{
    public Person(string name, int age)
    {
        Name = name;
        Age = age;
    }

    public string Name { get; private set; }

    public int Age { get; private set; }
}
```

그럼 동일한 타입에 대하여 다수의 Customization을 설정을 하면 어떻게 될까?
아래 코드를 보자. 그렇다, 먼저 설정된 Customization은 무시되고 나중에 설정된
Customization이 사용된다.

```c#
[Fact]
public void Customize_MoreThanOne_LetsUseLastCustomization()
{
    // Arrange
    var fixture = new Fixture();

    int initial = 100;
    int constant = 10000;

    // Act
    fixture.Customize(() => initial++);
    fixture.Customize(() => constant);

    // Assert
    Assert.Equal(10000, fixture.Create<int>());

    var person = fixture.Create<Person>();
    Assert.Equal(10000, person.Age);
}
```

<h4 id="Customize&lt;T&gt;(Func&lt;ISpecimenBuilder&lt;T&gt;, T&gt;)">Customize&lt;T&gt;(Func&lt;ISpecimenBuilder&lt;T&gt;, T&gt;)</h4>
`Fixture`클래스의 `Build<T>`메소드를 통한 Parameterized Anonymous Creation은 한번 객체 생성 후, 그 과정을 다시 이용할 수 없게 된다.
그러나, `Customize<T>(Func<ISpecimenBuilder<T>, T>)`메소드를 이용하면 Parameterized Anonymous Creation과정을 Customization으로 등록하여
계속 이용할 수 있게 된다.

아래 코드에서 `person1`과 `person2`는 서로 다른 객체이지만, 같은 Parameterized Anonymous Creation 과정을 밟게 된다.

```c#
[Fact]
public void Customize_SpecimenBuilder_CustomizesUsingParameterizedAnonymousCreation()
{
    // Arrange
    var fixture = new Fixture();

    string expectedName = fixture.Create<string>();
    int expectedAge = fixture.Create<int>();

    // Act
    fixture.Customize<Person>(
        sb => sb.ToCtor(expectedName)
                .ToCtor(expectedAge)
                .Create());

    // Assert
    var person1 = fixture.Create<Person>();
    var person2 = fixture.Create<Person>();

    Assert.NotSame(person1, person2);

    Assert.Equal(expectedName, person1.Name);
    Assert.Equal(expectedAge, person1.Age);

    Assert.Equal(expectedName, person2.Name);
    Assert.Equal(expectedAge, person2.Age);
}
```