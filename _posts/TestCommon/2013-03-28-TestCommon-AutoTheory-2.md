---
layout: post
title: TestCommon - AutoTheory(2)
tags : [TestCommon]
ccl: ko
date : 2013-03-28 13:35:00 UTC
---
{% include JB/setup %}

[이전 포스트 - AutoTheory(1)]에서는 **AutoTheory**의 전반적인 기능을 살펴보았다.
이번 AutoTheory(2) 포스트에서는 AutoTheory 사용 시 `Fixture` 클래스 인스턴스를 어떻게 **Configuration**할 것인가에 대해 살펴보고자 한다.
여기서 `Fixture`의 Configuration이라 함은 `Fixture` 클래스가 구현하고 있는 `IFixtureConfigs` 맴버 호출로
[test fixture]에 필요한 객체(specimen) 생성 방법을 전역적으로 설정하는 것,
또는 TestCommon의 [Customization] 기능을 이용하여 특정타입의 객체 생성 방법을 설정하는 것을 말한다.

만약 테스트 메소드 내에서 `Fixture` 클래스의 인스턴스를 생성하면
아래 코드와 같이 **Arrange** 파트에서 `fixture` 객체를 Configuration할 수 있게 된다.

```c#
[Fact]
public void Fixture_ConstructedInTestMethod_ConfiguresHowToCreateSpecimen()
{
    // Arrange
    var fixture = new Fixture();
    fixture.IsGreedy = false;
    fixture.ManyCount = 5;
    fixture.SetDefaultIdGeneratorFactory(() => new SequentialIdGenerator());
    fixture.Customize(() => "anonymous string");
    ....
}
```

그러나 TestCommon의 **AutoTheory** 기능을 사용하게 되면
테스트 메소드 내에서 `Fixture` 클래스의 인스턴스를 Configuration할 방법이 없다.
더욱이, 아래 코드와 같이 `Fixture` 타입의 인스턴스 `fixture`를 테스트 파라메타로 넘겨 받아
Configuration할지라도, 이는 테스트 파라메타 객체 생성 이후의 설정이 되므로,
`value`객체의 예와같이 `fixture`의 테스트 파라메타 객체 생성에 영향을 주지 못하게 된다.

```c#
[AutoTheory]
public void Fixture_AutoTheory_CannotConfigure(Fixture fixture, string value)
{
    // Arrange
    var expectedString = "anonymous string";
    fixture.Customize(() => expectedString);

    // Act
    // Assert
    Assert.NotEqual(expectedString, value);
}
```

<!-- break -->

<h3 id="AutoTheoryAttribute 상속에 의한 방법">AutoTheoryAttribute 상속에 의한 방법</h3>

`AutoTheoryAttribute` 타입의 객체생성 시에 생성자 파라메타로
`Func<IFixture>` 델리게이트를 넘겨 줄 수 있다.
이때 아래 코드와 같이
사용자의 Configuration을 거친 `Fixture` 인스턴스를 생성자 파라메타로 넘겨 주는 것이다.

```c#
public class CustomTheoryAttribute : AutoTheoryAttribute
{
    protected internal CustomTheoryAttribute() : base(() =>
        {
            var fixture = new Fixture();
            fixture.Customize(() => "anonymous string");
            return fixture;
        })
    {
    }
}

[CustomTheory]
public void Fixture_CustomTheory_CanConfigure(string value)
{
    // Arrange
    string expectedString = "anonymous string";

    // Act
    // Assert
    Assert.Equal(expectedString, value);
}
```

<h3 id="ISetupFixture">ISetupFixture</h3>
또 다른 방법으로는 테스트 클래스에서 `ISetupFixture` 인터페이스를 구현하는 방법이다.
`AutoTheoryAttribute`에서 해당 테스트 메소드를 실행할 때, 테스트 클래스가 `ISetupFixture` 인터페이스를 구현하는지 체크한다.
이때 만약 `ISetupFixture` 인터페이스를 구현한다면 테스트 메소드 실행 전에 `SetupFixture(IFixture)` 메소드를 호출하여 Configuration을 가능하게 한다.
아래 예제 코드에서 이를 확인할 수 있다.

```c#
public class SetupFixtureExample : ISetupFixture
{
    public void SetupFixture(IFixture fixture)
    {
        fixture.Customize(() => 1234);
    }

    [AutoTheory]
    public void Fixture_SetupFixture_CanConfigure(int value)
    {
        // Arrange
        // Act
        // Assert
        Assert.Equal(1234, value);
    }
}
```

<h3 id="ISetupFixtureClass">ISetupFixtureClass</h3>
마지막 방법으로 `ISetupFixtureClass` 인터페이스를 구현하는 방법이 있다.
`ISetupFixtureClass` 인터페이스를 구현한 클래스가 속하는 네임스페이스 대해
이 네임스페이스 내의 모든 테스트 메소드들은 `ISetupFixtureClass` 구현 클래스의 `SetupFixture(IFixture)` 메소드를 호출하게 된다.

`ISetupFixtureClass` 구현 클래스는 어셈블리 내에 여러 개일 수 있는데,
이때 `SetupFixture(IFixture)` 메소드 실행 순서는 상위 네임스페이스(바깥 쪽)의 `ISetupFixtureClass` 구현 클래스의 `SetupFixture(IFixture)` 메소드가 우선 순위를 갖는다.
`TestMethodA`는 아래 코드에서처럼 `SetupFixtureClassA` 클래스의 `SetupFixture(IFixture)` 메소드만을 호출하게 되는데,
`TestMethodB`는 `SetupFixtureClassA`와 `SetupFixtureClassB` 클래스 모두의 `SetupFixture(IFixture)` 메소드를 호출하게 된다.

```c#
namespace NamespaceA
{
    public class SetupFixtureClassA : ISetupFixtureClass
    {
        public void SetupFixture(IFixture fixture)
        {
            fixture.Customize(() => 1000);
            fixture.Customize(() => "NamespaceA");
        }
    }

    public class TestClassA
    {
        [AutoTheory]
        public void TestMethodA(int intValue, string stringValue)
        {
            // Arrange
            // Act
            // Assert
            Assert.Equal(1000, intValue);
            Assert.Equal("NamespaceA", stringValue);
        }
    }

    namespace NamespaceB
    {
        public class SetupFixtureClassB : ISetupFixtureClass
        {
            public void SetupFixture(IFixture fixture)
            {
                fixture.Customize(() => "NamespaceB");
            }
        }

        public class TestClassB
        {
            [AutoTheory]
            public void Fixture_SetupFixtureClassB_CanConfigure(int intValue, string stringValue)
            {
                // Arrange
                // Act
                // Assert
                Assert.Equal(1000, intValue);
                Assert.Equal("NamespaceB", stringValue);
            }
        }
    }
}
```

`ISetupFixtureClass` 인터페이스와 `ISetupFixture` 인터페이스 둘 중에는
`ISetupFixtureClass` 인터페이스의 `SetupFixture` 메소드
가 우선순위를 가져 먼저 호출된 뒤,
`ISetupFixture` 인터페이스의 `SetupFixture` 메소드가 호출된다.

만약 같은 네임스페이 내에 `ISetupFixtureClass` 인터페이스를 구현하는 클래스가 다수 존재할 때는
이에 대한 우선순위는 정해진바 없어, 어떤 것이 먼저 호출될지 단언할 수 없다.

[이전 포스트 - AutoTheory(1)]: /TestCommon-AutoTheory-1/
[test fixture]: http://xunitpatterns.com/test%20fixture%20-%20xUnit.html
[Customization]: /TestCommon-Customization/