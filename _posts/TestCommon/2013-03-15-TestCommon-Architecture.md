---
layout: post
title: TestCommon Architecture
tags : [TestCommon]
ccl: ko
date : 2013-03-15 02:34:00 UTC
---
{% include JB/setup %}

이번 포스트는 TestCommon의 내부 Architecture를 살펴보려 한다.
일단 아래 그림에서 `|-- ` 기호는 `HAS`관계이다.
예를 들어 아래 첫 번째 줄과 두 번째 줄에서 "`Fixture` 객체는 `SpecimenContext` 객체를 가진다"라고 이해할 수 있고,
`:` 표시는 `IS`관계(또는 상속관계)를 나타내는 것으로 두번 째 줄에서 `SpecimenContext` 클래스는 `ISpecimenProvider` 타입을 상속한다라고 이해할 수 있다.

TestCommon의 `Fixture` 객체는 `SpecimenContext`라는 객체를 가지고,
`SpecimenContext`객체는 하나의 `CompositeSpecimenProvider` 객체를 가진다.

```
|-- Fixture
    |-- SpecimenContext: ISpecimenContext
        |-- CompositeSpecimenProvider: ISpecimenProvider
            |-- CompositeSpecimenProvider: ISpecimenProvider (For Simple Type)
                |-- IntSpecimenProvider: ISpecimenProvider
                |-- StringSpecimenProvider: ISpecimenProvider
                |-- ...
            |-- CompositeSpecimenProvider: ISpecimenProvider (For Many Type)
                |-- ArraySpecimenProvider: ISpecimenProvider
                |-- ListSpecimenProvider: ISpecimenProvider
                |-- ...
            |-- CompositeSpecimenProvider: ISpecimenProvider (For Special Type)
                |-- DelegateSpecimenProvider: ISpecimenProvider
                |-- LazySpecimenProvider: ISpecimenProvider
                |-- ...
            |-- CompositeSpecimenProvider: ISpecimenProvider (For Reflection Type)
                |-- ReflectionSpecimenProvider: ISpecimenProvider
                |-- ...
            |-- ThrowingNullSpecimenException: ISpecimenProvider
```

<!-- break -->

`CompositeSpecimenProvider` 객체는 이름에서 알 수 있듯이 `ISpecimenProvider`에 대해 [Composite 패턴]을 이루고 있다.
테스트 객체를 생성하기 위해 `Fixture.Create<T>` 메소드를 호출하면
`Type`객체(`typeof(T)`)가 최상위 `CompositeSpecimenProvider`객체 `Create`메소드의 `source`로 전달된다.
`ISpecimenContext`타입 `context`파마메타 객체는 위 그림에서 `SpecimenContext` 타입의 객체를 나타내며 `Fixture`객체안에서 하나의 객체만 존재한다.

```c#
internal interface ISpecimenProvider
{
    object Create(object source, ISpecimenContext context);
}
```
`source`(`Type`)객체는 `ISpecimenProvider`타입 객체들로 이루어진 트리를 타고 아래로 전파되는데
만약 주어진 `source`에 대한 책임을 가지는 `ISpecimenProvider`를 만나면 `Create` 메소드 실행하고 그 결과 값을 리턴하는 것으로 트리검색을 종료하게 된다.
`ISpecimenProvider`는 [Composite 패턴]을 이루고 있지만 그것 보다 [Chain of responsibility 패턴]으로 구현되었다는 점에 더욱 주목할 필요가 있다.

예를 들어, `Fixture.Create<int[]>()`라는 메소드를 실행하게 된다면, Simple Type에 대한 모든 `ISpecimenProvider` 객체를 탐색하고 난 후
`ArraySpecimenProvider` 객체를 만나면 비로소 배열 객체를 생성하고 트리 탐색을 끝내게 되는 것이다.
따라서 Special Type과 Reflection Type은 트리검색 할 필요가 없다.
만약 트리탐색을 끝까지 하였는데도 특정 `source`가 처리되지 않으면 최후 `ThrowingNullSpecimenException` 객체를 만나 예외를 발생시키는 것으로 끝을 맺게된다.

TestCommon에서는 [Chain of responsibility 패턴]에 대한 ISpecimenProvider 순서가 중요하다. 위 그림에서 `ThrowingNullSpecimenException` 객체를 최상위로 배치한다면
모든 `Fixture.Create<T>()`결과는 예외를 발생시키게 될 것이며, 만약 Reflection Type에 대한 `CompositeSpecimenProvider` 객체를 최상위로 배치한다면 `string`같은 Simple Type에 대해서도
생성자를 실행을 통해 객체생성을 시도하게되어 결국 임의 문자열을 생성할 수 없게 될 것이다.

*이상에서의 TestCommon Architecture설명은 실제 구현된 것과 정확히 일치하지는 않는다.
설명을 보다 쉽게하기 위해서 디자인(위 그림)을 간단하게 나타내어 설명하였다.*


[Composite 패턴]: <http://ko.wikipedia.org/wiki/%EC%BB%B4%ED%8F%AC%EC%A7%80%ED%8A%B8_%ED%8C%A8%ED%84%B4>
[Chain of responsibility 패턴]: <http://ko.wikipedia.org/wiki/Chain_of_responsibility_%ED%8C%A8%ED%84%B4>