---
layout: post
title: 학습테스트(Learning Test)란?
tags : [Miscellaneous]
date : 2013-03-20 07:12:00 UTC
---
{% include JB/setup %}

아마도 이 글을 읽으시는 프로그래머 분께서는 프로그래밍 언어를 배우면서 수없이 많은 콘솔어플리케이션을 작성했을 것이다.
물론 특정 목적에 맞는 어플리케이션을 만들기 위해서 콘솔을 사용했을 수 있지만,
자신이 알고싶어 하는 코드를 확인하기 위해서도 콘솔을 많이 이용했을 것이다.

나도 메인함수에다 테스트하고자 하는 코드를 작성하고 콘솔에다 그 결과를 찍어서 눈으로 확인하는 것이 제일 편했었다.
그러나 매번 콘솔어플리케이션을 만드는 것, 나중에 다시 보려고 콘솔어플리케이션에 이름을 붙이고
폴더에 정리한다는 것이 여간 귀찮은 작업이 아닐 수 없었다.
콘솔어플리케이션 이름 붙이는게 귀찮아서 대충 붙이고 난후, 나중에 코드를 다시 보려했을때 뭐가 뭔지 이해하지 못 했던 일이 비일비재하였다.
이런 일을 수없이 겪고 난 후, 한 때는 아예 테스트목적의 콘솔어플리케이션은 저장하지 않는다는 생각까지 했었다.

그러던 중 [단위테스트]란 것을 알게 되었다.
단위테스트를 통해 메소드단위로 바로 코드를 실행할 수 있다는 것을 알고 난 후로는
이제 더 이상 콘솔어플리케이션에 테스트 목적의 코드를 작성하지 않았다.

<!-- break -->

### 학습테스트(Learning Test)
그럼 학습테스트(Learning Test)란 무엇일까?(이 용어를 접한건 어느 책에서였다. 그런데 지금은 어느 책이었는지 생각나질 않는다.)
문자 그대로 배우기 위한 테스트 정도로 이해할 수 있다. 학습을 위한 테스트 결과를 콘솔에다 결과를 찍어서 눈으로 확인하는 것이 아니라
[단위테스트]의 `Assertion`으로 바로 확인하는 것이다.

예를들어 1더하기 2가 3이 되는지를 테스트한다고 가정해 보자. 물론 이런 테스트를 하는 사람은 없을 것이다.
(어디까지나 학습테스트 설명을 위한 예제이다.)
아마 예전같으면 아래와 같은 콘솔어플리케이션을 만들어 눈으로 3을 확인했을 것이다.

```c#
public void Main()
{
    int a = 1;
    int b = 2;
    int result = a + b;
    Console.WriteLine(result);  // 3
}
```

그러나 이 예제를 Learning Test로 작성해 보면 아래와 같다. 눈으로 3을 확인하는 것이 아니라 코드실행에서 바로 결과를 입증하는 형태이다.
만약 결과가 3과 같지 않으면 예외를 발생시키게 될 것이며 예외를 발생시키더라도 다른 테스트들 실행은 계속 이루어진다.
메소드단위로 바로 코드를 실행할 수 있기 때문에 콘솔어플리케이션을 만들어야하는 번거로움이 없을 뿐더러, 테스트 메소드를 클래스, 네임스페이스 혹은 폴더단위로
구분지어 이해하기 좋게 분류할 수 있는 장점까지 가진다.

```c#
[Fact]
public void OnePlusTwoShouldBeThree()
{
    int a = 1;
    int b = 2;
    int result = a + b;
    Assert.Equal(3, result);
}
```

아래 그림은 내가 [Pro ASP.NET MVC 3 Framework]라는 책을 공부하면서 작성한 Learning Test코드를
Visual Studio 2012(솔루션 탐색기)를 통해 본 모습니다.
프로젝트 네임을 **ProAspNetMvc3Framework.LearningTest**으로 정해 [Pro ASP.NET MVC 3 Framework]의 Learning Test에
대한 프로젝트임을 알 수 있게 하였고, 책에서 나오는 챕터 당 하나의 폴더를 만들어 챕터에 대한 Learning Test 코드들을
해당 폴더에 두도록 하였다. 물론 이 것은 하나의 예시이며 자신 편한 방법으로 프로젝트와 폴더를 구성할 수 있을 것이다.

![Alt LearningTestExample][LearningTestExample]

### 테스트 프레임워크
Learning Test는 테스트 프레임워크를 이용하는 것이 됨으로 먼저 테스트 프레임워크를 이해하여야 한다.
Visual Studio에서는 다음과 같은 3가지 테스트 프레임워크가 주로 사용된다.

*   MSTest: Visual Studio에서 제공되는 Test Framework
*   [NUnit]: 오픈소스 Test Framework
*   [Xunit]: 오픈소스 Test Framework

아래에 나오는 **학습테스트(Learning Test) 예제**는 [Xunit]을 이용하는 [TestCommon]을 사용하였다.
[TestCommon]에 설치 및실행방법은 아래 글에서 확인할 수 있다.

*    [How to Get TestCommon]
*    [How to Run Tests Using TestCommon]

### 학습테스트(Learning Test) 예제
[Pro ASP.NET MVC 3 Framework]에서 설명하는 라우팅에 대한 Learning Test코드를 아래와 같이 작성하였다.
네임스페이스, 클래스 네임 그리고 메소드 네임을 이용하여 어떤 Learning Test코드를 작성하였는지를 쉽게 알아볼 수 있도록 이름을 붙였다.

아래에서 `InlineDataAttribute`에 나열된 `string`값은 각 테스트 메소드 파라메타에 순차적으로 입력이 되며,
나머지 `InlineDataAttribute`에 의해 값이 부여 되지 않은 메소드 파라메타값은 [TestCommon]에서 임의의 값으로 설정해준다.
ASP.NET MVC 3의 라우팅에 대해 이해하고 계시는 분은 아마 아래 Learning Test코드가 무엇을 의미하는지 아실 것이다.
이 글은 ASP.NET MVC 3설명하려는게 목적이 아니므로 이에 대한 설명은 생략하기로 한다.

아래와 같이 라우팅 공부하면서 Learning Test코드를 작성해 놓으면,
이후에 라우팅에 대해 조금이라도 석연치 않은 부분이 있으면 이 Learning Test코드를 찾아서 빨리 이해할 수 있을 것이다.


```c#
public class RoutingTest
{
    [AutoTheory]
    [InlineData("~/", "Home", "Index")]
    [InlineData("~/Customer", "Customer", "Index")]
    [InlineData("~/Customer/List", "Customer", "List")]
    public void routing_uses_the_default_segment_value_if_url_does_not_contain_a_value_for_the_default_segment(
        string url,
        string expectedController,
        string expectedAction,
        HttpContextBase httpContext,
        RouteCollection routes)
    {
        // Fixture setup
        routes.MapRoute(
            string.Empty,
            "{controller}/{action}",
            new { controller = "Home", action = "Index" });

        httpContext.ToMock().Setup(x => x.Request.AppRelativeCurrentExecutionFilePath).Returns(url);
        httpContext.ToMock().Setup(x => x.Request.PathInfo).Returns(string.Empty);

        // Exercise system
        var resultRouteData = routes.GetRouteData(httpContext);

        // Verify outcome
        Assert.Equal(expectedController, resultRouteData.Values["controller"]);
        Assert.Equal(expectedAction, resultRouteData.Values["action"]);
    }

    [AutoTheory]
    [InlineData("{controller}/{action}", "~/Customer/List/All")]
    [InlineData("{controller}/{action}/{id}", "~/Customer/List/All/Delete")]
    public void routing_returns_null_route_data_if_segment_lenght_is_not_matched(
        string pattern,
        string url,
        HttpContextBase httpContext,
        RouteCollection routes)
    {
        // Fixture setup
        routes.MapRoute(
            string.Empty,
            pattern,
            new { controller = "Home", action = "Index" });

        httpContext.ToMock().Setup(x => x.Request.AppRelativeCurrentExecutionFilePath).Returns(url);

        // Exercise system
        var resultRouteData = routes.GetRouteData(httpContext);

        // Verify outcome
        Assert.Null(resultRouteData);
    }

    [AutoTheory]
    [InlineData("~/Shop/Index", "Home", "Index")]
    public void routing_can_use_static_segment(
        string url,
        string expectedController,
        string expectedAction,
        HttpContextBase httpContext,
        RouteCollection routes)
    {
        // Fixture setup
        routes.MapRoute("ShopSchema", "Shop/{action}", new { controller = "Home" });

        httpContext.ToMock().Setup(x => x.Request.AppRelativeCurrentExecutionFilePath).Returns(url);

        // Exercise system
        var routeData = routes.GetRouteData(httpContext);

        // Verify outcome
        Assert.Equal(expectedController, routeData.Values["controller"]);
        Assert.Equal(expectedAction, routeData.Values["action"]);
    }

    [AutoTheory]
    [InlineData("~/", "Home", "Index", "DefaultId")]
    [InlineData("~/Customer", "Customer", "Index", "DefaultId")]
    [InlineData("~/Customer/List", "Customer", "List", "DefaultId")]
    [InlineData("~/Customer/List/All", "Customer", "List", "All")]
    public void routing_can_use_custom_segment(
        string url,
        string expectedController,
        string expectedAction,
        string expectedDefaultId,
        HttpContextBase httpContext,
        RouteCollection routes)
    {
        // Fixture setup
        routes.MapRoute(
            "MyRoute",
            "{controller}/{action}/{id}",
            new { controller = "Home", action = "Index", id = "DefaultId" });

        httpContext.ToMock().Setup(x => x.Request.AppRelativeCurrentExecutionFilePath).Returns(url);

        // Exercise system
        var resultRouteData = routes.GetRouteData(httpContext);

        // Verify outcome
        Assert.Equal(expectedController, resultRouteData.Values["controller"]);
        Assert.Equal(expectedAction, resultRouteData.Values["action"]);
        Assert.Equal(expectedDefaultId, resultRouteData.Values["id"]);
    }

    [AutoTheory]
    [InlineData("~/", "Home", "Index", "")]
    [InlineData("~/Customer/List/All", "Customer", "List", "All")]
    public void routing_can_use_optional_url_segment(
        string url,
        string expectedController,
        string expectedAction,
        string expectedDefaultId,
        HttpContextBase httpContext,
        RouteCollection routes)
    {
        // Fixture setup
        routes.MapRoute(
            "MyRoute",
            "{controller}/{action}/{id}",
            new { controller = "Home", action = "Index", id = UrlParameter.Optional });

        httpContext.ToMock().Setup(x => x.Request.AppRelativeCurrentExecutionFilePath).Returns(url);

        // Exercise system
        var routeData = routes.GetRouteData(httpContext);

        // Verify outcome
        Assert.Equal(expectedController, routeData.Values["controller"]);
        Assert.Equal(expectedAction, routeData.Values["action"]);
        Assert.Equal(expectedDefaultId, routeData.Values["id"].ToString());
    }

    [AutoTheory]
    [InlineData("~/Customer/List", "Customer", "List", "", null)]
    [InlineData("~/Customer/List/All/Delete", "Customer", "List", "All", "Delete")]
    [InlineData("~/Customer/List/All/Delete/Perm", "Customer", "List", "All", "Delete/Perm")]
    public void routing_can_catch_all_url_segments(
        string url,
        string expectedController,
        string expectedAction,
        string expectedDefaultId,
        string expectedCatchAll,
        HttpContextBase httpContext,
        RouteCollection routes)
    {
        // Fixture setup
        routes.MapRoute(
            "MyRoute",
            "{controller}/{action}/{id}/{*catchall}",
            new { controller = "Home", action = "Index", id = UrlParameter.Optional });

        httpContext.ToMock().Setup(x => x.Request.AppRelativeCurrentExecutionFilePath).Returns(url);

        // Exercise system
        var resultRouteData = routes.GetRouteData(httpContext);

        // Verify outcome
        Assert.Equal(expectedController, resultRouteData.Values["controller"]);
        Assert.Equal(expectedAction, resultRouteData.Values["action"]);
        Assert.Equal(expectedDefaultId, resultRouteData.Values["id"].ToString());
        Assert.Equal(expectedCatchAll, resultRouteData.Values["catchall"]);
    }
}
```

### 서드파티 라이브러리에 대한 Learning Test
어떤 프로젝트를 진행하면서 특정 서드파티 라이브러리를 사용한다면,
공개된 API와 도움말을 통해 그 라이브러리를 이해하고 사용법을 익히게 될 것이다.
이 경우 아마 앞서 말한 방식으로 Learning Test를 작성할 수 있을 것이다.

근데 이때 **Learning Test**작성은 다음과 같은 사항을 체크할 수 있게 해준다.

*   특정 서드파티 라이브러리를 사용하면서 그 라이브러리가 자신이 원하는데로
올바르게 작동하고 있다고 확신할 수 있는가?
*   버전업되면서 기존버전과 같은 결과물을 보장하는가?

위와 같은 사항이 전제되지 않으면 우리는 섣불리 서드파티 라이브러리를 참조할 수 없다.
Learning Test작성은 위의 사항을 체크할 수 있는 기회를 제공함으로, 제품으로 빌드되는 코드와 같이 관리되어야 할 것이다.

[단위테스트]: http://ko.wikipedia.org/wiki/%EC%9C%A0%EB%8B%9B_%ED%85%8C%EC%8A%A4%ED%8A%B8
[Pro ASP.NET MVC 3 Framework]: http://www.apress.com/9781430234043
[LearningTestExample]: /assets/images/LearningTestExample.jpg
[NUnit]: <http://www.nunit.org/>
[Xunit]: <http://xunit.codeplex.com/>
[TestCommon]: /TestCommon이란/
[How to Get TestCommon]: /How-to-Get-TestCommon
[How to Run Tests Using TestCommon]: /How-to-Run-Tests-Using-TestCommon