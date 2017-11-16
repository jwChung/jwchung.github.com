---
layout: post
title: DI vs DIP vs IoC
tags : [dependency-injection, solid]
---
{% include JB/setup %}

Dependency Injection(DI), Inversion of Control(IoC) 그리고 Dependency Inversion Principle(DIP)은 비슷한 듯하면서 다르다. 많은 사람이 혼란해 한다. 특별히 IoC는 좀 넓은 의미이고 DI는 IoC의 하위개념으로 좀 더 좁은 의미로 해석하는 것이다. IoC라는 전체집합에 DI는 부분집합이라고 생각하는 글을 많이 본다. 이렇게 생각하는 이유에 대해 내 나름데로 찾은 답은 [Martin Folwer의 글이다.](https://martinfowler.com/articles/injection.html) 혼란을 일으키는 주된 내용을 인용하면 다음과 같다.

> The approach that these containers use is to ensure that any user of a plugin follows some convention that allows a separate assembler module to inject the implementation into the lister. As a result I think we need a more specific name for this pattern. Inversion of Control is too generic a term, and thus people find it confusing. As a result with a lot of discussion with various IoC advocates we settled on the name Dependency Injection.

> 컨테이너를 사용하는 접근법은 어떤 플러그인 사용자든지 별도의 어셈블리 모듈에서 의존성(본문에서 `MovieFinder`를 말함)을 의존자(`MovieLister`)에게 주입하는 규칙을 따르도록 한다. 결과적으로 IoC는 너무 일반적인 용어라서 이런 패턴에 대한 좀 더 구체적인 이름이 필요하다고 생각한다. IoC를 사용하는 많은 사람과 토론을 거친 후, Dependency Injection이란 이름을 정했다.

IoC는 일반적인 의미라서 좀 더 구체적인 용어가 필요해 DI라는 용어가 생겼다고 이해된다. IoC는 좀 넓은 의미이고 DI는 IoC의 하위개념으로 좀 더 좁은 의미로 해석할 수 있다.  이런 이해를 바탕으로 DI가 IoC의 한 종류라 많이들 얘기한다. 하지만 나는 이 주장에 반대한다. **DI는 IoC의 부분집합이라고 생각하지 않고 이 둘은 별개라고 생각한다.** 그래서 난 Martin Folwer의 내용 또는 표현에 문제가 있다고 생각한다. 그가 DI는 IoC의 부분집합이다고 생각한다면 내용의 오류를 지적하고 싶고, 그렇지 않고 이 둘을 별개라고 생각한다면 오해를 불러 일으키는 표현의 문제점을 지적하고 싶다. 왜 그런지 내용을 살펴보자.

<!-- break -->

### Dependency Injection

아래코드를 보면, 피자가게(`PizzaStore`)에서 피자(`Pizza`)를 팔고 있다. 피자가게 예제는 단순해서 실제 프로젝트에서 사용되는 코드와 동 떨어져 있지만, 개념을 집중해서 설명하는 데는 효과적이라 생각한다. 

아래 `PizzaStore` 인스턴스는 정해진 개수의 `Pizza` 인스턴스를 직접 가지고 있다. 만약 3개롤 초과하는 피자를 팔고 싶다면 [`PizzaStore` 클래스를 수정해야 한다. OCP를 위배이다.](https://en.wikipedia.org/wiki/Open/closed_principle)

```c#
// C# 코드로 작성되었다.
// "..." 는 코드가 생략되었다 의미다.

public class PizzaStore
{
    private readonly Pizza[] pizzas =
        new Pizza[] { new Pizza(), new Pizza(), new Pizza() };

    public void Sell(int count)
    {
        // pizzas 필드를 사용해 피자를 판매한다.

        ...
    }
}
```

이 문제는 `PizzaStore` 클래스가 `Pizza` 배열을 직접참조하고 있기 때문에 발생하고 있다. `Pizza` 배열 의존성을 인수(argument) 넘겨 받으면 문제가 해결된다. 이제 `PizzaStore` 생성할 때 필요한 만큼 `Pizza`를 넘겨주면 된다. **인수를 취하는 것이 DI의 개념의 대부분이다**고 해도 과언이 아니다.

```c#
public class PizzaStore
{
    private readonly Pizza[] pizzas;

    public PizzaStore(Pizza[] pizzas)
    {
        this.pizzas = pizzas;
    }

    public void Sell(int count)
    {
        // pizzas 필드를 사용해 피자를 판매한다.

        ...
    }
}
``` 

 [Rúnar Bjarnason는 한 세미나에서 아래와 같이 DI는 인수를 취하는 것을 허세적으로 표현한 것이고 말했다.](https://www.youtube.com/watch?v=ZasXwtTRkio)

> Dependency Injection is really just a pretentious way to say 'taking an argument'.

[DI 주제로 500 페이지 넘게 책을 쓴 Mark Seemann](https://www.amazon.com/Dependency-Injection-NET-Mark-Seemann/dp/1935182501)은 적어도 이 표현에 반대할 거라 예상하지만, [그는  Rúnar Bjarnason 표현이 틀렸다기보다 추가될 내용이 있다고 했다.](http://blog.ploeh.dk/2017/01/27/dependency-injection-is-passing-an-argument/) Mark Seemann의 추가 내용을 설명하기 위해 아래 `IPizzaStore`라는 인터페이스를 만들자. 위에서 언급된 `PizzaStore` 클래스가 이 인터페이스를 구현한다고 해보자. `Pizza` 배열이 인수로 넘겨지지만, `IPizzaStore` 인터페이스(추상화)를 전혀 해치지 않고 있다는 것에 주목하자. 그는 이 점을 강조했다. **추상화를 해치지 않고 의존성을 인수로 넘겨주는 방법이 DI**라고 말이다. 이것이 DI 개념의 전부이다. 여기에 DIP 또는 IoC를 넣어 DI 개념을 혼란시킬 필요가 없다.

```c#
public interface IPizzaStore
{
    void Sell(int count);
}
```

### Dedendency Inversion Principle

피자가게에서 한 종류의 피자(`Pizza`)가 아니라 여러 종류의 피자를 팔고 싶어한다. 위에서 살펴 본 코드로는 불가능하다. 문제 원인은 피자가게(`PizzaStore`)가 피자(`Pizza`)에 의존하고 있기 때문이다. 아래 그림과 같이 더 중요한 `PizzaStore` 클래스가 덜 중요한 `Pizza` 클래스에 의존한 것이다. DIP 위배이다.

![DIP](../images/DIP-violation.png)

덜 중요한 `Pizza` 클래스가 더 중요한 `PizzaStore`에 의존하게 만들어야 한다. 아래 그림과 같이 `IPizza` 인터페이스(추상화)를 도입해 DIP 위배 문제를 해결해보자. 위 그림에서 `PizzaStore` 클래스가 `Pizza` 클래스를 의존(참조, 실선으로 표현)하는 것에서, `Pizza` 클래스가 `IPizza` 인터페이스를 의존(인터페이스 구현, 점선으로 표현)하는 것으로 **화살표 방향이 뒤집힌 것(역전)**을 확인할 수 있다. DIP 원칙을 지킨 것이다.

![DIP](../images/DIP-resolved.png)

아래 DIP 원칙을 나타내는 문장 둘을 보며 DIP를 다시 한번 정리하고 넘어가는 것이 좋겠다. 더 중요한 모듈이 덜 중요한 모듈에 의존하면 안된다. 이 관계를 뒤집기 위해 추상화가 필요하다. 아울러 추상화가 실체에 의존해서는 안된다.

> A. High-level modules should not depend on low-level modules. Both should depend on abstractions.  
B. Abstractions should not depend on details. Details should depend on abstractions.

DI는 으레 DIP와 연결되었다고 생각한다. 하지만 이 둘은 별개다. DI를 사용하면서 DIP 개념이 필요하지 않는 경우도 많다란 뜻이다. 예를들어 아래 `CachedUserStore` 클래스를 보자. 사용자를 `userId`를 통해 매번 조회하는 것이 아니라 일정 기간(`duration`) 동안 캐쉬하는 기능을 제공하는 `IUserStore`의 [Decorator](http://www.dofactory.com/net/decorator-design-pattern) 역할을 한다. 이때 `duration` 값은 DI를 통해 주입되지만, DIP 개념이 필요한 것은 아니다. 

```c#
public interface IUserStore
{
    User Find(string userId);
}

public class CachedUserStore : IUserStore
{
    ...

    public CachedUserStore(IUserStore innerStore, TimeSpan duration)
    {
        ...
    }

    public User Find(string userId)
    {
        ...
    }
}
```

### Inversion of Control

IoC 개념은 프레임워크와 라이브라리 사용 차이에서 설명될 수 있다. 라이브러리 사용은 필요한 것을 직접 가져와서 쓰는 반면, 프레임워크에서는 규칙에 따라 구성요소를 등록해 놓으면 프레임워크에서 이 구성요소를 가져다 쓰게 된다. 이 관계에서 제어가 역전되었다고 표현한다. [같은 맥락에서 템플릿메소드 패턴 역시 IoC를 설명하기에 좋은 예다.](http://www.dofactory.com/net/template-method-design-pattern) IoC를 좀 더 외우기 쉽게 Hollywood Principle이라고 한다. "내가 전에도 얘기했잖아, 나한테 먼저 연락하지마, 필요하면 내가 연락할께"라고 말이다.

피자가게 문제로 돌아가보자. DI와 DIP 개념을 사용하여 피자가게에서는 아래코드에서 보듯 원하는 종류의 피자를 원하는 개수만큼 만들어 팔 수 있게 되었다.

```c#
public static void Main()
{
    var pizzaStore = new PizzaStore(
        new IPizza[]{
            new CheezePizza(),
            new CheezePizza(),
            ...
            new ShrimpPizza(),
            new BulgogiPizza(),
            ...
        });

    ...
}
```

여기서 피자의 종류가 아주 많다면 어떻게 될까? 손으로 많은 종류의 피자를 나열해서 생성하는 것은 많은 비용을 요구한다. 이 번거로운 작업을 프레임워크가 해주면 좋겠다. IoC 개념이 담긴 IoC 컨테이너가 이 역할을 해줄 수 있다. 아래 코드에서는 특정 모듈(어셈블리)에 속하면서 `IPizza` 인터페이서를 구현하는 모든 타입을 IoC 컨테이너로 등록하고 사용하는 것을 보여주고 있다. 특정 피자 클래스 생성자는 사용자에 의해 직접호출되는 것이 아니라, IoC 컨테이너에 의해 호출된다. 인스턴스 생성방향이 역전되어, 헐리우드 배우와 같은 모습이 된 것이다.

```c#
public static void Main()
{
    Type[] pizzaTypes = Assembly.GetEntryAssembly()
        .GetExportedTypes()
        .Where(typeof(IDisposable).IsAssignableFrom);

    var builder = new IoCContainerBuilder();
    buildr.RegisterTypes(pizzaTypes);

    IoCContainer container = builder.Build();

    IPizza[] allKindOfPizza = container.Resolve<IPizza[]>();

    var pizzaStore = new PizzaStore(allKindOfPizza);

    ...
}
```

### Pure DI

DI 개념이 DIP의 개념과 별개이 듯, IoC 개념도 그렇다. DI를 사용한다고 IoC 컨테이너가 무조건 필요한 것이 아니다. 얼마든지 피자가게 생성자에 피자를 직접 나열하여 인스턴스를 생성할 수 있다. 이것이 DI의 개념에 반하는 것이 아니다. 별개의 문제이다.

Mark Seemann은 IoC 컨테이너를 사용하지 않는 DI에 대해 Pure DI라는 용어를 사용했다. 그의 책 "Dependency Injection in .NET" 에서는 이 용어 대신 Poor Man's DI라는 용어로 기술되어 있다. [하미만 Poor Man라는 뜻에서 오는 부정적인 의미 때문에  IoC 컨테이너를 사용하지 않는 DI의 강점이 퇴색되는 것이 안타까웠던 모양이다.](http://blog.ploeh.dk/2014/06/10/pure-di/) 책에서 사용한 용어를 바꾸면 혼란이 있음에도 불구하고 그는 Poor Man's DI 대신 Pure DI라고 불러주길 희망하고 있다.

Pure Man DI의 강점이 뭐길래 용어를 바꾸는 모험을 단행한 것일까? [When to use a DI Container라는 글에서 그의 생각을 엿볼 수 있다.](http://blog.ploeh.dk/2012/11/06/WhentouseaDIContainer/)