---
layout: post
title: How to Run Tests Using TestCommon
tags : [TestCommon, Tools]
ccl: ko
date : 2013-03-14 05:45:00 UTC
---
{% include JB/setup %}

<style>
.nuget-badge code
{
	-moz-border-radius: 5px;
	-webkit-border-radius: 5px;
	background-color: #202020;
	border: 4px solid #c0c0c0;
	border-radius: 5px;
	box-shadow: 2px 2px 3px #6e6e6e;
	color: #e2e2e2;
	display: block;
	font: 1.5em 'andale mono', 'lucida console', monospace;
	line-height: 1.5em;
	overflow: auto;
	padding: 15px;
}
</style>


[TestCommon]을 이용하여 작성된 테스트를 어떻게 실행하는지를 살펴볼 필요가 있다.
이 사항을 알아야 앞으로 나오는 테스트를 실행해볼 수 있기 때문이다.

[TestCommon]을 이용하여 작성된 테스트를 어떻게 실행하느냐는 것은 곧 어떤 Test Framework를 사용하여 어떤 Test Runner로 실행하느냐로 바꿔말할 수 있다.
.NET에서 사용가능한 테스트 프레임워크는 아래 3가지 정도인데 각 테스트 프레임워크는 자체
Test Runner를 포함하고 있다.

*   MSTest: Visual Studio에서 제공되는 Test Framework
*   [NUnit]: 오픈소스 Test Framework
*   [Xunit]: 오픈소스 Test Framework

위 중 특정 Test Framework를 사용하여야지만 [TestCommon]을 이용할 수 있는 것은 아니지만
[Xunit]에 대해서는 테스트 메소드 파라메터에 대해 자동으로 값을 입력하는 기능을 제공하고 있다.
`AutoTheoryAttribute`는 [Xunit]의 `TheoryAttribute`를 상속하여 구현되어 있으므로 [DepositTestUsingAutoTheory] 테스트와 같이 메소드 파라메터로 값을 넘겨 받을려면
[Xunit]에 대한 Test Runner를 사용하여야 한다. 만약 [DepositTestUsingAutoTheory] 테스트를 [Xunit]을 사용하지 않고 MSTest 또는 [NUnit]을 사용하게 되면
`Fixture` 클래스를 이용하여 [DepositTestUsingFixtureClass] 테스트와 같이 구현하여야 한다.

`AutoTheoryAttribute`를 사용하면 `Fixture` 클래스를 사용할 때보다 코드가 간결해지지만
MSTest 또는 [NUnit]을 사용하기 원하는 개발자는 이를 활용하지 못하게 된다.
따라서 **[TestCommon]은 [Xunit] Test Framework 사용을 권장한다.**

사실 [TestCommon]은 [Xunit]에 대한 참조를 가지고 있어 [TestCommon]을 [NuGet]을 통해 설치하게 되면 [Xunit] 라이브러리도 같이 설치되게 된다.
MSTest 또는 [NUnit] 사용자 입장에서 보면 [Xunit] 라이브러리는 불필요 한 것이 된다.
최초 [TestCommon] 라이브러리 설계시 이 점을 고려해 공통라이브러리(`Fixture` class 지원)와
[Xunit]를 필요로하는 라이브러리(`AutoTheoryAttribute` 지원)로 나눠 개발하려고 하였으나,
[TestCommon]을 사용하면서 `AutoTheoryAttribute`를 사용하지 않을 이유가 없다고 생각되었고
또한 라이브러리를 여러개로 나누면 설치과정이 복잡해진다는 이유에서 그렇게 하지 않았다.

<!-- break -->

### Xunit Test Runner
그럼 Xunit Test Framework에서 작성된 테스트들은 어떤 Test Runner를 사용할 수 있는지에 대한 얘기를 해 보자.
[Xunit] 테스들을 실행하기 위해서 Xunit 프로젝트에 포함되어 있는 [xunit.runners]만을 사용해야하는 것은 아니며 아래와 같이 여러가지 방법이 있다.

*   [xunit.runners]
*   [xUnit.net runner for Visual Studio 2012]
*   [Resharper Test Runner]
*   [TestDriven.Net]

각각의 Test Runner는 장단점을 가지지만 나는 아래와 같은 이유에서 [TestDriven.Net]을 좋아한다.

*   상업적 목적이 아닌 개인이 사용, 오픈소스 개발자에게 무료이다. 자세한 내용은 해당 [라이센스][TestDriven.Net License]를 확인하기 바란다.
*   시각적인 효과가 적어서 다른 Test Runner들과 비교했을때 빠른 편이다.
*   솔루션 탐색기에서 하위 테스트들을 포함하여 파일, 폴더 형태로 바로 실행가능하다.
*   텍스트 편집기에서 테스트 메소드, 클래스에 포함된 테스트 메소드들, 또는 네임스페이스에 포함된 테스트 메소드들을 바로 실행가능하다.

### TestCommon 소스의 테스트를 실행하려면?
[TestCommon]의 모든 어셈블리들은 지연서명이 되어있다.
따라서 [GitHub][TestCommon]에서 다운받은 소스코드를 컴파일하고 테스트들을 실행하기위해서는 다음과 같은 명령실행으로
강력한 어셈블리 검사를 수행하지 않도록 하여야 한다. Windows가 64비트 환경일때는 x86환경과 x64환경 모두 실행해 줄 필요가 있다.

```
sn -Vr *,db4e4bc25029966c
```



[TestCommon]: <https://github.com/jwChung/TestCommon>
[NuGet]: <http://nuget.org/>
[NUnit]: <http://www.nunit.org/>
[Xunit]: <http://xunit.codeplex.com/>
[DepositTestUsingAutoTheory]: </TestCommon이란#DepositTestUsingAutoTheory>
[DepositTestUsingFixtureClass]: </TestCommon이란#DepositTestUsingFixtureClass>
[xunit.runners]: http://nuget.org/packages/xunit.runners/1.9.1
[xUnit.net runner for Visual Studio 2012]: http://visualstudiogallery.msdn.microsoft.com/463c5987-f82b-46c8-a97e-b1cde42b9099
[Resharper Test Runner]: http://www.jetbrains.com/resharper/features/unit_testing.html
[TestDriven.Net]: http://testdriven.net/
[TestDriven.Net License]: http://testdriven.net/purchase_licenses.aspx