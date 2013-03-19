---
layout: post
title: TestCommon - Non Reflection Type 객체 생성(2)
tags : [TestCommon]
ccl: ko
date : 2013-03-18 04:20:00 UTC
---
{% include JB/setup %}

앞서 [Non Reflection Type 객체 생성(1)](#2013-03-15-TestCommon-Non-Reflection-Type-객체-생성-1) 포스트에서는 Simple Type에 알아보았다.
이번 포스트에서는 Non Reflection Type에서 Many Type과 Special Type에 대해 알아보자.

### Many Type

TestCommon은 아래의 컬렉션에 대해서는 객체 생성 후 컬렉션의 아이템을 추가로 생성하여 `Add`한다.
이 때 `Add`하게 될 아이템의 개수는 `Fixure.ManyCount'를 설정할 수 있으며 기본 값은 3이다.

*   Array
*   List\<T\>
*   Dictionary\<TKey, TValue\>

아래와 같은 실제 사용예제들을 보면, 특별한 설명 필요없이 쉽게 이해할 수 있다.

```c#
[Fact]
public void TestCommon_CreatesArrayAndListType()
{
    var fixture = new Fixture();

    // Default ManyCount as 3
    var array = fixture.Create<int[]>();
    Assert.Equal(3, array.Length);

    // Specifying ManyCount
    fixture.ManyCount = 5;
    var list = fixture.Create<List<string>>();
    Assert.Equal(5, list.Count);
    Assert.True(list.All(x => x != null));
}
```

`Dictionary<TKey, TValue>`객체 생성 시는 `TKey`에 주의를 기울일 필요가 있다.
아래 예제와 같이 `TKey`가 `int`타입일 경우에 `SequentialIdGenerator` 타입 객체를 쓰지 않고
디폴트로 설정된 `RandomIdGenerator` 타입 객체를 사용할 경우, `TKey`의 값이 중복이 일어날 수 있다.
이경우 TestCommon에서는 예외를 발생시키므로 주의가 필요하다.

```c#
[Fact]
public void TestCommon_CreatesDictionaryType()
{
    var fixture = new Fixture();
    fixture.SetIdGenerator(typeof(int), new SequentialIdGenerator(0, 3));

    var dictionary = fixture.Create<Dictionary<int, string>>();

    Assert.Equal(3, dictionary.Count);
}
```

<!-- break -->


### Special Type
**Special Type**은 아래와 같이 **Simple Type**에도 불류되지 않고 **Many Type**에도 분류되지 않는 특별한 타입들이다.
**Special Type** 범주의 타입들은 차후, 추가되거나 삭제될 수 있다.

*   Delegate
*   Lazy\<T\>
*   Type

`Delegate`타입은 `void`인 경우와 리턴타입을 가지는 것으로 나누어 생각해 볼 수 있는데,
`void`인 `Delegate`타입의 경우는 empty body이다.
아래코드에서 `Action`타입 객체 생성을 예제를 보면 `voidDelegate`객체는 `null`이 아닌 델리게이트 객체를 리턴하지만
`voidDelegate`객체는 실행할 코드를 가지고 있지 않다.(empty body)

```c#
[Fact]
public void TestCommon_CreatesVoidDelegateType()
{
    var fixture = new Fixture();
    var voidDelegate = fixture.Create<Action>();
    Assert.NotNull(voidDelegate);
    voidDelegate(); // nothing to do...

    fixture.SetIdGenerator(typeof(int), new SequentialIdGenerator(1, 10));
    var returnDelegate = fixture.Create<Func<int>>();
    Assert.Equal(1, returnDelegate());
    Assert.Equal(1, returnDelegate());
}
```

리턴타입을 가지는 `Delegate`타입의 경우는 리턴타입에 해당하는 익명이 객체를 반환하는데,
델리게이트 객체를 실행할 때 마다 새로운 객체를 리턴하는 것이 아니라 매 호출 시마다 동일한 객체를 리턴하도록 되어 있다.

```c#
[Fact]
public void TestCommon_CreatesReturnDelegateType()
{
    var fixture = new Fixture();
    var objectFactory = fixture.Create<Func<object>>();
    Assert.Same(objectFactory(), objectFactory());
}
```

TestCommon은 [Lazy\<T\>]타입에 대해 아래코드와 같이 `T`타입 익명 객체를 설정하여 `Value` 프라퍼티를 통해 접근할 수 있도록한다.

```c#
[Fact]
public void TestCommon_CreatesLazyType()
{
    var fixture = new Fixture();
    fixture.SetIdGenerator(typeof(int), new SequentialIdGenerator(100, 1000));
    var lazy = fixture.Create<Lazy<int>>();
    Assert.Equal(100, lazy.Value);
}
```

TestCommon은 `Type`객체에 대해 가장 일반적이라 할 수 있는 `typeof(object)`결과를 리턴한다.

```c#
[Fact]
public void TestCommon_CreatesType()
{
    var fixture = new Fixture();

    var actual = fixture.Create<Type>();
    Assert.Equal(typeof(object), actual);
}
```

[Lazy\<T\>]: http://msdn.microsoft.com/en-us/library/dd642331.aspx