---
layout: post
title: TestCommon's Non-Reflectoin Type and Reflection Type
tags : [TestCommon]
ccl: ko
date : 2013-03-14 11:58:00 UTC
---
{% include JB/setup %}

### 서론
아래의 `MailAccountTest` 클래스의 `TestCommon_ConstructsInstanceOfComplexType` 테스트 메소드를 실행해 해 보자.

```c#
public class MailAccountTest
{
    [AutoTheory]
    public void TestCommon_ConstructsInstanceOfComplexType(
        MailAccount sut)
    {
        // Arrange
        // Act
        // Assert
        Assert.NotNull(sut);
        Assert.NotNull(sut.Owner);
        Assert.NotNull(sut.Owner.Name);
        Assert.NotNull(sut.EmailAddress);
    }
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

public class MailAccount
{
    public MailAccount(Person owner, string emailAddress)
    {
        EmailAddress = emailAddress;
        Owner = owner;
    }

    public Person Owner { get; set; }

    public string EmailAddress { get; private set; }
}
```

**아무런 값 지정없이 [TestCommon]은 `MailAccount` 객체를 생성하고 있다. 어떻게 가능한 것일까?**

MailAccount 객체를 생성하기 위해서 먼저 `owner`에 대한 `Person` 객체를 먼저 생성한다.
이때 `Person` 객체를 생성하기 위해 필요한 `name`과 `age`는 [TestCommon]의 설정에 따라
임의(anonymous)로 만들게 된다.
이때 임의라는 뜻은 설정값에 따라 non-deterministric 값일 수도 있고, deterministric 값일 수 있다는 것을 의미한다.
(이에 대한 자세한 내용은 차후 포스팅되는 글에서 확인할 수 있으며, 관련 글이 포스팅된 후 이 부분에 링크를 만들 것이다.)

`Person` 객체를 생성과 더불어 `emailAddress`에 대한 `string` 객체를 생성한 후,
이를 가지고 [TestCommon]은 `MailAccount`의 생성자를 실행하여 `MailAccount` 객체를 비로소 획득할 수 있게된다.

이러한 일려의 과정에서 주안점은 생성자를 통한 객체 생성이냐 아니면 [TestCommon]에서 설정에 따라 객체(혹은 값)을 생성하느냐이다.
[TestCommon]에서는 전자를 **Reflection Type**이라하고 후자를 **Non-Reflecton Type**이라하는데
이는 다시 **Simple Type**, **Many Type**, 그리고 **Special Type**으로 세분화 된다.

<!-- break -->

### Non-Reflecton Type
예를들어 `int` 같은 값은 생성자를 통해 특정 값을 생성하는 것이 불가능하다.
이러한 값은 [TestCommon]에서 0과 100사이의 수를 랜덤으로 부여(기본 설정 - non-deterministric)할 수 있고
0부터 1씩 증가하는 수를 부여(deterministric)할 수도 있다.
`int`와 같이 생성자를 통해 객체(혹은 값)를 생성할 수 없거나 혹은 `List<string>`과 같이
객체 생성후 추가적인 설정(자식객체 `Add`)이 필요한 경우의 타입들을 통칭하여 **Non-Reflecton Type**이라한다.

#### Simple Type
**Non-Reflecton Type** 중 .Net Framework에서 기본이 되는 타입을 **Simple Type**으로 분류하며 해당되는 타입들은 아래와 같다.

*   Numerics
    -   byte
    -   sbyte
    -   decimal
    -   double
    -   float
    -   int
    -   uint
    -   long
    -   ulong
    -   short
    -   ushort
*   char
*   bool
*   string
*   datetime
*   enum
 

#### Many Type
**Many Type** 는 컬렉션 객체를 말하는 것으로 아래와 같은 타입에 대해서 [TestCommon]은 해당 객체생성과 더불어
미리 설정되어 있는 `ManyCount`값에 따라 자식객체를 추가시켜 준다.
예를들어 `List<string>` 타입에 대해 [TestCommon]은 디폴트 `ManyCount`로 3개의 `string`객체를
`List<string>` 객체에 `Add`시키게 된다.

*   Array
*   List\<T\>
*   Dictionary\<TKey, TValue\>

#### Special Type

**Special Type**은 아래와 같이 **Simple Type**에도 불류되지 않고 **Many Type**에도 분류되지 않는 특별한 타입들을 일컫는다.
향후 [TestCommon]에 특정기능을 추가하거나 버그를 수정하기 위해, 또 다른 타입이 이 범주에 추가 될 수 있다.

*   Delegate
*   Lazy\<T\>
*   Type

### Reflecton Type
앞서 살펴본 **Non-Refletion Type**은 생성자를 통해 객체를 생성하는 것이 불가능하거나,
객체생성 후 **Many Type**처럼 추가적인 설정이 필요한 Type을 말한다면 **Reflecton Type**은 그외 Reflecton을 통행 객체를 생성할 수 있는 모든 타입을 말한다.

### 맺음말
이번 포스트에서는 [TestCommon]에서 제공하는 객체타입을 크게 **Non-Refletion Type**과 **Refletion Type**으로 나누어 간략히 살펴보았다.
이어지는 포스트들에서는 각 카테고리별 타입에 대해 보다 자세히 살펴볼 것이다.

[TestCommon]: <https://github.com/jwChung/TestCommon>