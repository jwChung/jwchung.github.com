---
layout: post
title: TDD Kata - ConsoleStringCalculator
tags : [TDD Kata, TestCommon]
date : 2013-04-05 07:41:00 UTC
published: false
---
{% include JB/setup %}

TDD Kata에 대한 세 번째 포스팅이다.
첫 번째 포스트에서는 아주 간단한 [FizzBuzz] 게임을 예로 들었었고,
두 번째 포스트에서는 조금 더 어려운 예제로 [StringCalculator]를 소개했다.
FizzBuzz와 StringCalculator는 TDD를 접하지 못한 분들에 대해 TDD 소개를 위해서는 좋은 예제임이 틀림이 없다.
그러나, TDD에서 중요한 요소인 테스트 대상 클래스([SUT])와 의존객체([DOC])사이에서 일어나는 **상호작용(interaction)**을 설명하기에는 적합하지 않다.

이번 포스트는 두 번째 포스트의 연장선에 있으며, 
`StringCalculator` 클래스를 활용하여 실제 콘솔어플리케이션인 `ConsoleStringCalculator` 클래스를 작성해 보는 것이다.
`ConsoleStringCalculator` 클래스의 `Main(string[])` 메소드에 `string` 타입으로 일련의 숫자들을 파라메타로 넘기면,
`Main(string[])` 메소드 내에서는 다시 `StringCalculator` 객체의 `int Add(string)` 메소드에 그 일련의 숫자 `string` 값을 넘겨 그 결과로 받은 합계를
화면에 출력하는 시나리오이다.
이를 통해, 우리는 SUT와 DOC 사이에 일어나는 interaction을 배울 수 있는데,
이때, DOC(`StringCalculator` 타입 객체)에 대해 실제 구현 객체를 사용하는 것이 아니라 테스트 목적의 [Test Double]로
대체하여 테스트하게 된다. 이 때문에 이번 포스트 예제가 이전의 예제들보다 훨씬 어려운게 사실이다.

이번 포스트의 예제 역시 StringCalculator 예제와 같이 Roy Osherove의 블로그에 나오는 [예제]를 기초로 하는데,
이와는 조금 다른 스텝으로 TDD Kata를 구성하려한다.
왜냐하면 Roy Osherove의 TDD Kata 예제에서는 `ConsoleStringCalculator` 타입의 DOC인 `StringCalculator` 타입에 대한 [Test Double] 언급이 없어,
단위테스트(unit testing)를 위한 예제인지, 통합테스트(integration testing)를 위한 예제인지 혼돈이 있기 때문이다.

<!-- break -->

### ConsoleStringCalculator TDD Kata

#### 준비사항
먼저 아래와 같은 인터페이스 `IStringCalculator`와 콘솔프로그램 진입점이 되는 `ConsoleStringCalculator` 클래스를 아래와 같이 만든다.

```c#
public interface IStringCalculator
{
	int Add(string numbers);
}
```

```c#
public class ConsoleStringCalculator
{
	public static void Main(string[] args)
	{
	}
}
```

여기서 한 가지 생각해 볼 문제가 있다. `Main` 메소드 내에서 `IStringCalculator` 타입의 Test Double 인스턴스를 사용하게 될텐데
어떻게 이를 `Main` 메소드 내부로 넘겨줄 수 있을까?

사실 바로 이전 포스트([Isolating Static Method Call]) 작성 이유가 이 문제에 대한 대안을 제시하기 위해 작성이 되었다.
이전 포스트에서 제시한 방법을 활용하면 **static factory method**를 통해 아래와 같은 방법으로 `Main` 메소드 내에
단위테스트를 위한 `IStringCalculator` 타입의 Test Double 인스턴스(`stringCalculator`) 설정이 가능하다.

```c#
public static void Main(string[] args)
{
	IStringCalculator stringCalculator = StringCalculatorFactory.Create();

	...
}
```

위의 방법을 활용해도 되지만 또 다른 방법을 생각해 본다면,
아래 코드와 같이 `ConsoleStringCalculator` 타입 내에 `IStringCalculator` 타입의 **static 프라퍼티 setter**를 선언하는 것이다.
이것으로 단위테스트 내에서 Test Double을 설정할 수 있는 것이다.

```c#
public class ConsoleStringCalculator
{
	internal IStringCalculator StringCalculator { get; set;}
	
	public static void Main(string[] args)
	{
	}
}
```

위 두가지 방법 중 어느 방법을 사용해도 무방하지만 중요한 것은
두 가지 방법 모두 Static 멤버를 활용하는 것으로, 특정 테스트의 Test Double 설정이 다른 테스트에 영향을 주게 된다.
따라서 테스트를 마치기 전에 꼭 사용이전으로 돌려 놓는 것(**teardown**)을 잊지 말아야 하겠다.

#### 유의사항
지난 포스트에서 TDD Kata 유의사항을 명시하였는데 여기에서 다시 상기해 보자.

한 스텝은 논리적인 테스트를 의미하는 것으로
[Data-driven testing](http://en.wikipedia.org/wiki/Data-driven_testing)같이 물리적으로 여러개의 테스트들도 논리적으로 한 테스트인 것으로 간주하여 한 스텝에 포함시킨다.
여기서 주의 할 점은 **"한번에 하나씩 완성해 나가야한다"**는 것이다. 즉, 현재의 스텝에서 다음 스텝(테스트)에 대해서는 고려하지 말아야한다.
이때 테스트 완성은 Red, Green,  그리고 Refactoring 모든단계가 끝났음을 의미한다.

#### Steps
*   `ConsoleStringCalculatorTest` 클래스를 만들고, Main(string[] args) 메소드가
`IStringCalculator`의 `Add` 메소드 결과를 아래와 같은 형식으로 콘솔에 출력하는 것을 테스트하라.
꼭 `IStringCalculator` 타입의 Test Double을 활용해야 됨을 명심하라.

    __"The result is 6"__

    콘솔출력힌트: `Console.Set(TextWriter)` 메소드를 통해 콘솔 출력에 대한 Test Double을 설정할 수 있다.

*   첫 번째 결과 후, 프로그램을 바로 종료시키지 말고,
__"another input please"__라는 메세지로 또 다른 numbers 값을 입력 할 것인가를 물어라.
만약 사용자가 또 다른 값을 입력하면 그 결과를 출력하고 다시 "another input please" 메세지로 새로운 값을 입력할 것인가를 물어라.
이것을 사용자가 새로운 numbers 값 입력 없이 엔터를 입력할 때까지 반복하라.

*   [지난 포스트][StringCalculator]의 `StringCalculator` 클래스가 `IStringCalculator`를 구현한다는 것을 테스트하라.
이때 테스트는 `StringCalculatorTest` 클래스에 작성하라.

*   `IStringCalculator` 타입의 `ConsoleStringCalculator.StringCalculator` static 프라퍼티가 디폴트로
`StringCalculator` 타입 객체를 리턴하라.

#### 참고
*   [ConsoleStringCalculatorTest](https://github.com/jwChung/TestCommon/blob/master/demo/TDDKata/ConsoleStringCalculatorKata/UnitTests/ConsoleStringCalculatorTest.cs):
테스트 작성이 힘드신 분만 이 파일을 참고해서 보세요. 스스로 해결하는 것이 가장 좋은 방법입니다.  
https://github.com/jwChung/TestCommon/blob/master/demo/TDDKata/ConsoleStringCalculatorKata/UnitTests/ConsoleStringCalculatorTest.cs

*   [ConsoleStringCalculator](https://github.com/jwChung/TestCommon/blob/master/demo/TDDKata/ConsoleStringCalculatorKata/ConsoleStringCalculator.cs):
단위테스트로 `ConsoleStringCalculator` 클래스 구현하기 전에는 이 파일을 보지 않으시길 바랍니다. 단지 자신이 한 것과 비교(참고)용으로만 사용하세요.  
https://github.com/jwChung/TestCommon/blob/master/demo/TDDKata/ConsoleStringCalculatorKata/ConsoleStringCalculator.cs

*   [IntegrationTest](https://github.com/jwChung/TestCommon/blob/master/demo/TDDKata/ConsoleStringCalculatorKata/IntegrationTests/ConsoleStringCalculatorTest.cs): 위에서 언급하지는 않았지만 `StringCalculator` 클래스 Test Double로 처리 되었기 때문에 `ConsoleStringCalculator` 클래스와 실제 상호작용 하는 것을
테스트할 필요가 있다. 이를 우리는 단위테스트와 다르게 통합테스트([Integration Testing])라 부르는데 이 파일을 참고하시길 바란다.  
https://github.com/jwChung/TestCommon/blob/master/demo/TDDKata/ConsoleStringCalculatorKata/IntegrationTests/ConsoleStringCalculatorTest.cs



[FizzBuzz]: /TDD-Kata-FizzBuzz-%EA%B2%8C%EC%9E%84
[StringCalculator]: /TDD-Kata-StringCalculator
[StringCalculator TDD Kata]: http://osherove.com/tdd-kata-1/
[SUT]: http://xunitpatterns.com/SUT.html
[DOC]: http://xunitpatterns.com/DOC.html
[Test Double]: http://xunitpatterns.com/Test%20Double.html
[예제]: http://osherove.com/tdd-kata-2/
[Isolating Static Method Call]: /Isolating-Static-Method-Call
[Integration Testing]: http://en.wikipedia.org/wiki/Integration_testing