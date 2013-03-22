---
layout: post
title: TestCommon - Parameterized Anonymous Creation(4)
tags : [TestCommon]
ccl: ko
date : 2013-03-22 06:50:00 UTC
---
{% include JB/setup %}

이번 포스트에서는 Parameterized Anonymous Creation기능 설정(5가지) 중 **특정메소드 실행**에 대해서 알아보고자 한다.

1. 생성자 선택
2. 생성자 파라메타값 설정
3. 프라퍼티값 설정
4. **특정메소드 실행**
5. Mock에 관한 속성 설정

Parameterized Anonymous Creation기능을 통해 임의의 객체를 생성할 때,
어떤  특정 메소드를 호출을 해야 할 경우에 이 기능을 사용할 수 있다.
아래의 코드의 예를 보자. `List<string>`타입의 객체 `actual`를 생성하고 난 후,
`Add`메소드를 호출하여 `expectedString`을 추가하는 것을 보여준다.

```c#
[Fact]
public void Do_ExecutesSomeMethodsOfTargetInstance()
{
    // Arrange
    var fixture = new Fixture();

    string expectedString = fixture.Create<string>();

    // Act
    var actual = fixture.Build<List<string>>()
                        .Do(x => x.Add(expectedString))
                        .Create();

    // Assert
    Assert.Equal(1, actual.Count);
    Assert.Equal(expectedString, actual[0]);
}
```
<!-- break -->

`Do`메소드는 익명의 파라메타 값을 가지는(5개까지) 메소드들이 오버로드되어 있다.
따라서 특정 메소드를 실행할때 익명값을 파라메타로 부터 넘겨 받을 수 있다.

```c#
[Fact]
public void Do_AnonymousParameter_ExecutesSomeMethodsOfTargetInstance()
{
    // Arrange
    var fixture = new Fixture();

    // Act
    var actual = fixture.Build<List<int>>()
                        .Do<int, int>((target, anonymous1, anonymous2) =>
                            {
                                target.Add(anonymous1);
                                target.Add(anonymous2);
                            })
                        .Create();

    // Assert
    Assert.Equal(2, actual.Count);
}
```