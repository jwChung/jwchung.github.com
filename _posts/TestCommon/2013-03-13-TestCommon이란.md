---
layout: post
title: TestCommon이란?
tags : [TestCommon]
published: true
ccl: ko
date : 2013-03-13 04:00:00 UTC
---
{% include JB/setup %}

**Test-Driven Development에서 Test에 관계되지 않는 객체를 자동으로 생성함으로써,
개발자가 테스트하고자 하는 케이스에만 집중할 수 있게 한다.**

예를들어, 아래의 `BankAccount`클래스에서 `Deposite`메소드를 테스트한다고 가정해 보자.

```c#
public class BankAccount
{
    public BankAccount(string owner, decimal balance)
    {
        Owner = owner;
        Balance = balance;
    }

    public string Owner { get; private set; }

    public decimal Balance { get; private set; }

    public void Deposite(decimal amount)
    {
        Balance += amount;
    }
}
```

그럼 다음과 같은 테스트 코드를 생각해 볼 수 있다.
문제는 `BankAccount`객체를 생성하기 위해서 생성자 파라메타 `owner`에 대해 `string`값을 넘겨줘야 하는데
아래에서와 같이 `"owner value"`는 `DepositTest`에 아무런 영향을 미치지 않고 있다.
오히려 `"owner value"`는 테스트내에서 어떤 역할을 하고 있는 것처럼 보여
테스트를 작성한 사람 뿐 아니라 다른 개발자에게까지 `DepositTest`를 이해하는데 지장을 줄 수 있다.

```c#
[Fact]
public void DepositTest()
{
    // Arrange
    decimal initialAmount = 10;
    var owner = "owner value";
    var sut = new BankAccount(owner, initialAmount);

    decimal depositedAmount = 100m;

    // Act
    sut.Deposit(depositedAmount);

    // Assert
    decimal expected = depositedAmount + initialAmount;
    Assert.Equal(expected, sut.Balance);
}
```
주) `FactAttribute`는 [Xunit] Test Framework에서 제공하는 클래스입니다.

<!-- break -->

그럼 [TestCommon]의 `Fixture` class를 도입하면 아래 코드에서 보듯이 테스트케이스와
관계되지 않는 생성자 파라메타 `owner` 값을 지정하지 않아도 될 뿐아니라,
`initialAmount`와 `depositedAmount`의 값을 상수로 지정해 주지않아도 된다.
이 값은 `Fixture` 객체에서 자동으로 값을 생성하게 되며,
값을 어떤 방식(deterministic, non-deterministic)으로 부여할지에 대한 메카니즘 또한 제공하고 있다.

```c#
[Fact]
public void DepositTestUsingFixtureClass()
{
    // Arrange
    var fixture = new Fixture();

    decimal initialAmount = fixture.Create<decimal>();
    var sut = fixture.Build<BankAccount>().ToCtor(initialAmount).Create();

    decimal depositedAmount = fixture.Create<decimal>();

    // Act
    sut.Deposit(depositedAmount);

    // Assert
    decimal expected = depositedAmount + initialAmount;
    Assert.Equal(expected, sut.Balance);
}
```

그럼 [TestCommon]을 이용하여 테스트케이스를 작성한다면,
위 코드처럼 매번 `Fixture`객체를 생성하고 테스트에 필요한 객체를 생성하기 위해서 `Fixture.Create<T>()`메서드를 호출해야 할까?
그렇지 않다.

[TestCommon]에서는 `AutoTheoryAttribute`를 통해 테스트에 필요한 객체를 아래와 같이 테스트 메소드의 파라메타로 넘겨받을 수 있다.
또한, `initialAmount`과 같이 특정객체(`sut`)의 생성자 파라메타가 될 경우 `ToCtorAttribute`와 `BuildAttribute`를 통해 이 관계를 설정할 수 있게 한다.

```c#
[AutoTheory]
public void DepositTestUsingAutoTheory(
    [ToCtor] decimal initialAmount,
    [Build] BankAccount sut,
    decimal depositedAmount)
{
    // Arrange
    // Act
    sut.Deposit(depositedAmount);

    // Assert
    decimal expected = depositedAmount + initialAmount;
    Assert.Equal(expected, sut.Balance);
}
```


[Xunit]: <http://xunit.codeplex.com/>
[TestCommon]: <https://github.com/jwChung/TestCommon>
