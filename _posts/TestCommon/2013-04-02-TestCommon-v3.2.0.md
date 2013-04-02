---
layout: post
title: TestCommon(v3.2.0) - Deterministic Specimen For Simple Type
tags : [TestCommon]
ccl: ko
date : 2013-04-02 05:59:00 UTC
---
{% include JB/setup %}

### String과 DateTime 타입 IdGenerator에 기초

이전 TestCommon 버전(v3.1.3)에서는 [Non Reflection Type 객체 생성](/TestCommon-Non-Reflection-Type-객체-생성-1/)에서 언급한 바와 같이
`string`에 대해서는 GUID 값을, `DateTime`에 대해서는 `DateTime.Now` 값으로
`IdGenerator`에 기초하지 않은 값을 리턴하였다.

**TestCommon(v3.2.0)**에서는 아래 예제코드처럼 `string`과 `DateTime` 타입을 `IdGenerator`에 기초한 값을 리턴하는 것으로 수정되었는데,
`string` 타입에 대해서는 "Anonymous"와 `Id`를 결합한 문자열을 리턴하고,
`DateTime` 타입에 대해서는 현재의 날짜(시간제외)에서 `Id`만큼의 날짜를 더한 값을 리턴하게 된다.

```c#
[Fact]
public void StringType_IsBasedOnIdGenerator()
{
    // Arrange
    var fixture = new Fixture();
    fixture.SetDefaultIdGeneratorFactory(() => new SequentialIdGenerator(0, 100));

    // Act
    // Asserte
    Assert.Equal("Anonymous00", fixture.Create<string>());
    Assert.Equal("Anonymous01", fixture.Create<string>());
    Assert.Equal("Anonymous02", fixture.Create<string>());
}

[Fact]
public void DateTimeType_IsBasedOnIdGenerator()
{
    // Arrange
    var fixture = new Fixture();
    fixture.SetDefaultIdGeneratorFactory(() => new SequentialIdGenerator(0, 100));

    var date = DateTime.Now.Date;

    // Act
    // Asserte
    Assert.Equal(date, fixture.Create<DateTime>());
    Assert.Equal(date.AddDays(1), fixture.Create<DateTime>());
    Assert.Equal(date.AddDays(2), fixture.Create<DateTime>());
}
```
<!-- break -->

### Deterministic vs Nondeterministic
일반적으로 테스트에 사용되는 값들은 Deterministic해야 한다.
왜냐하면 테스트가 실패하게 될 경우 실패의 원인을 찾기 위해서이다.
그런데 만약 아래 코드처럼 `value > 50` 코드를 테스트할 경우,
TestCommon의 Anonymous Creation으로 만들어진 `int` 값으로 테스트하게 된다면,
`int` 타입에 대해 TestCommon(v3.1.3)은 0이상 100미만의 값을 리턴하기 때문에
테스트가 50%확률로 Fail 또는 Pass가 된다. 이러한 현상을 가지는 테스트를 [NondeterministicTest]이라 하는데,
물론 예제 테스트의 NondeterministicTest로 인한 테스트 실패의 원인은 금방 알 수 있으나,
일반적으로 NondeterministicTest 실패의 원인은 찾기가 까다롭다.
따라서 테스트에 사용되는 값들은 원칙적으로 Deterministic해야 하는 것이다.

```c#
[Fact]
public void NonDeterministricValue_CausesProblemOfNondeterministicTest()
{
    // Arrange
    var fixture = new Fixture();

    // Act
    // Assert
    var value = fixture.Create<int>();
    if (value > 50)
    {
        Assert.True(false);
    }
}
```

그러나 테스트에 사용되는 값을 Nondeterministic을 사용한다면 **test coverage**를 높일 수 있는 장점이 있다.
아래 예제 코드와 같이 만약 `int` 값에 대한 더하기(`+`)를 테스트한다고 가정해 보자.
아래 코드는 하나의 테스트를 작성했으나 테스트 실행 시마다 `val1`와 `val2`에 다른 값이 입력 됨으로
하나의 테스트로 여러 가지 경우의 테스트하는 효과를 가져온다.

```c#
[Fact]
public void NonDeterministricValue_ImprovesTestCoverage()
{
    // Arrange
    var fixture = new Fixture();

    int val1 = fixture.Create<int>();
    int val2 = fixture.Create<int>();
    int expected = val1 + val2;

    // Act
    int actual = val1 + val2;

    // Assert
    Assert.Equal(expected, actual);
}
```
Nondeterministic 값은 NondeterministicTest 문제점을 가지지만 test coverage 높이는 장점이 있다.
또한, NondeterministicTest 문제는 TestCommon의 [Parameterized Anonymous Creation] 혹은 [Customization] 기능으로 해결할 수 있다.
이러한 이유에서 TestCommon(v3.1.3)에서는 [Simple Type][Non Reflection Type 객체 생성]들에 대해 디폴트로 Nondeterministic 값을 설정하였다.

그러나 사용자가 만약 이를 인지하지 못하고 위 `value > 50` 테스트 예제처럼 NondeterministicTest 문제에 봉착할 가능성이 열려있다.

### 날짜에 기초한 Random Seed 도입
**TestCommon(v3.2.0)**에서는 test coverage 높이면서 NondeterministicTest문제를 해결하기 위해 Random Seed를 도입하였다.
그런데 만약 Random Seed 숫자가 고정이 되어 있으면, 항상 정해진 순서로 값을 리턴하기 때문에
NondeterministicTest문제는 해결이 되나, test coverage를 높일 수는 없다.
따라서 TestCommon(v3.2.0)는 현재의 날짜가 1년 365일 중 몇일 째인가를 따져 그 수를 Random Seed로 이용한다.
`static` 프라퍼티인 `RandomIdGenerator.Seed`를 이용하면 테스트에 활용되는 Random Seed 넘버를 확인할 수 있다.

TestCommon(v3.2.0)에서 Simple Type 값 생성을 위해 사용되는 `RandomIdGenerator`는
날짜를 기초로한 Random Seed와 더불어 최소값 0(inclusive), 최대값 100(exclusive)의 랜덤수를 사용한다.(최소/최대값은 이전 v3.1.3과 동일)
또한 TestCommon에서는 타입별 `IdGenerator`를 지원하는데
TestCommon(v3.2.0)의 디폴트 `RandomIdGenerator`객체는 아래 코드처럼 모든 Simple Type들에 대해 공유된다.
물론 `SetDefaultIdGeneratorFactory` 메소드로 각각 Simple Type들에 대해 `RandomIdGenerator` 객체를 따로 사용하도록 설정할 수 있다.

```c#
[Fact]
public void RandomIdGenerator_IsSharedWithAllSimpleTypes()
{
    // Arrange
    var fixture = new Fixture();

    var random = new Random(RandomIdGenerator.Seed);

    int intExpected = random.Next(0, 100);
    byte byteExpected = (byte)random.Next(0, 100);

    // Act
    // Assert
    Assert.Equal(intExpected, fixture.Create<int>());
    Assert.Equal(byteExpected, fixture.Create<byte>());
}

[Fact]
public void RandomIdGenerator_SetDefaultIdGeneratorFactory_CannotBeSharedWithAllSimpleTypes()
{
    // Arrange
    var fixture = new Fixture();

    // Using one instance of the RandomIdGenerator type is default for all the simpple types as the following.
    ////var randomIdGenerator = new RandomIdGenerator(seed: RandomIdGenerator.Seed);
    ////fixture.SetDefaultIdGeneratorFactory(() => randomIdGenerator);

    fixture.SetDefaultIdGeneratorFactory(
        () => new RandomIdGenerator(seed: RandomIdGenerator.Seed));

    int intExpected = new Random(RandomIdGenerator.Seed).Next(0, 100);
    byte byteExpected = (byte)new Random(RandomIdGenerator.Seed).Next(0, 100);

    // Act
    // Assert
    Assert.Equal(intExpected, fixture.Create<int>());
    Assert.Equal(byteExpected, fixture.Create<byte>());
}
```

### Sementic Versioning For v3.2.0
위에서 설명한 바와 같이 TestCommon 버전 v3.1.3에서 **v3.2.0**로의 업데이트 내용은
"*String과 DateTime 타입 IdGenerator에 기초*"와 "*날짜를 기초한 Random Seed 도입*"이다.
"날짜를 기초한 Random Seed 도입"한 것은 어차피 기존 v3.1.2에서 Simple Type들에 대해 Nondeterministic 값이었기 때문에
하위 호환성을 보장하는 기능추가로 [Sementic Verioning]에 의해 minor 버전번호 증가가 맞다.

하지만 "String과 DateTime 타입 IdGenerator에 기초"한 것은 리턴값의 변화,
예를들어 GUID 형태의 `string`값에서 "AnonymousXX"형태로 변했기 때문에
하위 호환성을 보장 못하는 변경이므로 Sementic Verioning에 의해서는 major 버전번호 증가에 해당한다.

그러나 "*String과 DateTime 타입 IdGenerator에 기초*"만으로 major 버전번호 증가 업데이트는 무리가 있으며,
v3으로의 major 버전번호 증가한 시기가 얼마지나지 않은 시점이라는 점,
그리고 향후에 이루어질 major 버전증가(v4)에 "*String과 DateTime 타입 IdGenerator에 기초*"를 포함시키기엔
Simple Type 값에 영향을 준다는 시급성 때문에 이번 minor 업데이트에 포함시키게 되었다.

[Non Reflection Type 객체 생성]: </TestCommon-Non-Reflection-Type-객체-생성-1/>
[NondeterministicTest]: <http://xunitpatterns.com/Erratic%20Test.html#Nondeterministic%20Test>
[Parameterized Anonymous Creation]: </TestCommon-Parameterized-Anonymous-Creation-1>
[Customization]: </TestCommon-Customization>
[Sementic Verioning]: http://semver.org/
