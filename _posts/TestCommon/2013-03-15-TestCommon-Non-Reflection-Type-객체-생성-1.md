---
layout: post
title: TestCommon - Non Reflection Type 객체 생성(1)
tags : [TestCommon]
ccl: ko
date : 2013-03-15 09:42:00 UTC
---
{% include JB/setup %}

### 서론
TestCommon에서 생성하는 객체 타입은 크게 [Non Reflection Type과 Reflection Type]으로 나뉜다.
Non Reflection Type 객체에는 다음과 같은 세 종류가 있으며,
이번 포스트에서는 먼저 Simple Type에 대해 자세히 알아보려 한다.

-   Simple Type
-   Many Type
-   Special Type

Simple Type은 아래에서 보듯이 총 16가지 타입들이 있는데(볼드체는 분류이름) 이중 Id에 의해 값이 결정되는 타입 14가지와 그렇지 않은 타입(datetime, string) 2가지가 있다.

-   **Id에 의해 값이 결정**
    -   **Numerics**
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
    -   char
    -   bool
    -   enum
-   **Id와 상관없음**
    -   string  
    -   datetime

<!-- break -->

<a id="IdGenerator"></a>
### IdGenerator
*'Id에 의해서 값이 결정된다'*는 의미는 예를들어 `char`라는 타입을 TestCommon에서 생성한다면
일단 `IdGenerator` 객체로부터 `Int32` 타입의 값을 넘겨 받는다. 넘겨 받은 정수를 가지고 해당하는 `char`가 매치되는데
이때 Id 0은 'a'가 매치되며, 1은 'b', 2는 'c', ...., 24는 'y', 그리고 25는 'z', 26은 다시 'a' 이런식의 반복이 되며,
만약 음수의 정수가 입력되면 절대값처리된 후, `char`가 매치가 된다.

`IdGenerator` 객체는 타입당 하나의 개체가 할당됨으로 Id에 의해 값이 결정되는 타입들은 서로다른 `IdGenerator` 객체를 가진다.
Id에 의해 값이 결정되는 모든 14가지 타입들에 대해서는 **디폴트 `IdGenerator`는 0보다 크거나 같고, 100보다 작은 범위의 랜덤 `Int32`값이다**.
디폴트 `IdGenerator`는 `Fixture` 클래스의 `SetDefaultIdGeneratorFactory` 메소드를 통해 변경할 수 있다.

### Numerics
Simple Type에 해당하는 숫자타입은 총 11가지인데 이들 모두 Id에 의해서 값이 결정된다.
`int`타입 또는 `int`타입보다 큰 범위를 가지는 수의 경우는 그냥 단순히 Id값을 리턴하게 되지만,
`int`타입보다 작은 범위가지는 타입의 경우, 그 범위를 넘어가는 경계지점의 수를 다시 0으로 시작하여 Id를 매치시키게 된다.
아래 코드에서 자세히 살펴보자.

아래에서 코드에서 보듯이, 
우선 디폴트로 `int`, `byte`타입은 앞서 말한바와 같이 0보다 크거나 같고,
100보다 작은 범위의 랜덤수를 리턴한다.

그러나, `int`타입과 `byte`타입의 `IdGenerator`를 랜덤이 아닌
255보다 크거나 같고 300보다 작은 `SequentialIdGenerator`를 설정하게 되면,
`int`는 차례대로 255, 256의 수를 리턴하게 되지만,
`byte`의 경우는 255를 리턴하고 난 후, 256은 범위를 벗어나게 되므로 다시 0을 리턴하게 된다.

```c#
[Fact]
public void TestCommon_CreatesNumericTypes()
{
    var fixture = new Fixture();

    // Default IdGenerator
    int intResult = fixture.Create<int>();
    double doubleResult = fixture.Create<byte>();

    Assert.True(0 <= intResult && intResult < 100, "int");
    Assert.True(0d <= doubleResult && doubleResult < 100d, "byte");

    // Specifying IdGenerator
    fixture.SetIdGenerator(typeof(int), new SequentialIdGenerator(255, 300));
    Assert.Equal(255, fixture.Create<int>());
    Assert.Equal(256, fixture.Create<int>());

    fixture.SetIdGenerator(typeof(byte), new SequentialIdGenerator(255, 300));
    Assert.Equal(255, fixture.Create<byte>());
    Assert.Equal(0, fixture.Create<byte>());
    Assert.Equal(1, fixture.Create<byte>());
    Assert.Equal(2, fixture.Create<byte>());
}
```

### char
`char`타입은 앞서 [IdGenerator](#IdGenerator)에서 살펴 것과 같이 'a'부터 'z'까지 일련의 문자를 0부터 차례로 매치시키고
26부터 다시 'a'를 매치시키게 된다.

```c#
[Fact]
public void TestCommon_CreatesCharType()
{
    var fixture = new Fixture();

    fixture.SetIdGenerator(typeof(char), new SequentialIdGenerator(0, 3));
    Assert.Equal('a', fixture.Create<char>());
    Assert.Equal('b', fixture.Create<char>());
    Assert.Equal('c', fixture.Create<char>());
    Assert.Equal('a', fixture.Create<char>());

    fixture.SetIdGenerator(typeof(char), new SequentialIdGenerator(24, 27));
    Assert.Equal('y', fixture.Create<char>());
    Assert.Equal('z', fixture.Create<char>());
    Assert.Equal('a', fixture.Create<char>());
    Assert.Equal('y', fixture.Create<char>());
}
```
### bool
Id가 음수일때는 절대값을 취하게 되며, 0과 짝수는 true가 매치되고, 홀수에 대해서는 false가 매치된다.

```c#
[Fact]
public void TestCommon_CreatesBoolType()
{
    var fixture = new Fixture();

    fixture.SetIdGenerator(typeof(bool), new SequentialIdGenerator(0, 3));
    Assert.False(fixture.Create<bool>(), "bool"); // 0
    Assert.True(fixture.Create<bool>(), "bool"); // 1
    Assert.False(fixture.Create<bool>(), "bool"); // 2
    Assert.False(fixture.Create<bool>(), "bool"); // 0
    Assert.True(fixture.Create<bool>(), "bool"); // 1
}
```
### enum
특정 enum 타입의 값 순서에 Id가 매치되며, Id가 값 순서 범위를 벗어나면 다시 처음의 enum값이 매치된다.

```c#
public enum TestEnum
{
    First = 100,
    Second = 122,
    Thrid = 167
}

[Fact]
public void TestCommon_CreatesEnumType()
{
    var fixture = new Fixture();

    fixture.SetIdGenerator(typeof(TestEnum), new SequentialIdGenerator(0, 3));
    Assert.Equal(TestEnum.First, fixture.Create<TestEnum>());
    Assert.Equal(TestEnum.Second, fixture.Create<TestEnum>());
    Assert.Equal(TestEnum.Thrid, fixture.Create<TestEnum>());
    Assert.Equal(TestEnum.First, fixture.Create<TestEnum>());
}
```
### Id와 상관없는 타입
Simple Type 들 중 `string`과 `DateTime`은 `IdGenerator`와 상관없이 non-deterministic 값을 리턴하게 된다.
`string`은 guid값을, `DateTime`은 현재시간을 나타내는 `DateTime.Now`를 리턴한다.

[Non Reflection Type과 Reflection Type]: /TestCommon's-Non-Reflection-Type-and-Reflection-Type

```c#
[Fact]
public void TestCommon_CreatesStringType()
{
    var fixture = new Fixture();

    string result1 = fixture.Create<string>();
    string result2 = fixture.Create<string>();

    ////Console.WriteLine(result1); // ba401fe2-3481-4605-b2bf-803529ea8bba
    ////Console.WriteLine(result2); // aa602846-ad5d-4a8c-b9e8-d8dfd8bba265
}

[Fact]
public void TestCommon_CreatesDateTimeType()
{
    var fixture = new Fixture();

    var result = fixture.Create<DateTime>();

    ////Console.WriteLine(result); // 03/14/2012 PM 08:11:15
}
```







