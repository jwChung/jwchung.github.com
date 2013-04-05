---
layout: post
title: TDD Kata - FizzBuzz 게임
tags : [TDD Kata, TestCommon]
date : 2013-04-03 03:05:00 UTC
---
{% include JB/setup %}

### Kata란?
[위키디피아]에서 Kata란 용어에 대해 아래와 같이 정의하고 있다. *프로그래밍 능력 향상을 위한 연습*정도로 이해할 수 있다.
제목에서 Kata 앞에 [TDD]란 용어를 붙였는데, *TDD 능력향상을 위한 연습*으로 이해하면 되겠다.

> Code Kata is a term coined by Dave Thomas, co-author of the book The Pragmatic Programmer, in a bow to the Japanese concept of kata in the martial arts.
> A code kata is an exercise in programming which helps a programmer hone their skills through practice and repetition.

### Why TDD Kata?
이 블로그에서는 TDD에 도움을 주는 [TestCommon](https://github.com/jwchung/TestCommon)에 대한 사용 설명이 주를 이루고 있지만,
이 블로그를 보시는 분들 중 TDD에 관심이 없거나 접해 보지 못한 분들이 분명 계실 것이다.
그런 분들에게 TDD에 대한 동기부여 목적으로 **TDD Kata**를 소개하고자 한다.

### TDD?
이 포스트에서 TDD의 중요성, 필요성을 논할 생각은 없다.
방대한 내용일 뿐아니라, 옳고 그름의 문제가 아니기 때문이다.
다만, 이 글을 보시는 분은 TDD에 대한 필요성을 느끼시는 분으로 생각하고 글을 이어가고자 한다.

TDD에 대한 이해는 인터넷검색 또는 서적을 통해서 하시길 바라며, 여기서는 TDD에서 가장 중요한 절차에 대해서만 알아보고자 한다.
TDD는 크게 아래와 같이 3단계로 이루어지며, 이 3단계의 사이클을 반복하면서 개발을 진행하게 된다.

*   **Red**  
**테스트를 작성하고 컴파일 가능하게만** 만든다.
이 단계에서는 구체적인 구현을 하지 않고, 클래스와 메소드 선언들만으로
테스트코드에 대해 제품코드가 컴파일이 가능하게만 코딩한다.
아마 이 단계에서 제품 코드 코딩은 비주얼 스튜디오에서 자동으로 생성해 주는 코드만으로 충분할 것이다.

*   **Green**  
이 단계에서는 해당 **테스트만 통과시킬 가장 최소한의 코딩**만 한다. 이때 최소한의 코딩은 중요한 의미가 있다.
만약 해당 테스트 범위 외에 것까지 구현하게 된다면, 테스트로 커버링 되지 않는 구현이 생겨 버리거나,
테스트 되어야 할 기능이 먼저 구현부터 되어 있어 Red 단계의 테스트를 작성할 수 없게 되는 문제점을 일으킨다.
결국, 이는 TDD가 추구하는 바를 역행하는 것이기 때문에 이를 피하기 위해 최소한의 코딩은 중요하다.


*   **Refactoring**  
이 단계에서는 테스트를 통과한다는 제약을 충족하는 범위에서 제품 코드를 좀 더 **Readable/Maintainable**하게 만든다.
더불어, 테스트 코드 자체 또한 중복코드 제거와 같은 리팩토링을 통해 Readable/Maintainable하게 만든다.

[TDD]: http://en.wikipedia.org/wiki/Test-driven_development
[위키디피아]: http://en.wikipedia.org/wiki/Kata_(programming)

<!-- break -->

### FizzBuzz 게임이란?
이번 포스트가 TDD Kata에 대해 첫 번째 포스트이다 보니 쉬운 예제가 좋겠다는 생각을 했다.
아마 알려진 TDD Kata 예제 중 FizzBuzz가 이에 가장 적합하지 않나 생각해 본다.

FizzBuzz게임이란 [위키디피아](http://en.wikipedia.org/wiki/Fizz_buzz)에서 설명하는 것 처럼,
여러명이 모여서하는 게임으로 우리나라 369게임과 비슷하다고 생각하면 된다.
원 모양으로 둘러앉아 1부터 순차적으로 증가하는 숫자에 대한 답을 한명씩 차례로 말하면 되는데 이때, 다음과 같은 규칙을 가진다.

*   3으로 나누어 떨어지면 "Fizz"라고 말한다.
*   5로 나누어 떨어지면 "Buzz"라고 말한다.
*   3과 5 모두에 대해 나누어 떨어지면 "FizzBuzz"라고 말한다.
*   그 외 숫자는 그냥 해당 숫자를 말한다.

### FizzBuzz TDD Kata
#### 준비
FizzBuzz와 FizzBuzzTest란 클래스를 각각 만든다.
FizzBuzz 클래스에 아래와 같은 메소드를 만들고 이를 TDD Kata를 통해 구현해 보자.

```c#
public string Translate(int number)
{
}
```
#### 유의사항
아래의 한 스텝은 논리적인 테스트를 의미하는 것으로
[Data-driven testing](http://en.wikipedia.org/wiki/Data-driven_testing)같이 물리적으로 여러개의 테스트들도 논리적으로 한 테스트인 것으로 간주하여 한 스텝에 포함시킨다.
여기서 주의 할 점은 **"한번에 하나씩 완성해 나가야한다"**는 것이다. 즉, 현재의 스텝에서 다음 스텝(테스트)에 대해서는 고려하지 말아야한다.
이때 테스트 완성은 위에서 설명한 Red, Green,  그리고 Refactoring 모든단계가 끝났음을 의미한다.

#### Steps
1.  3의 배수인 3, 6, 9가 입력되면 "Fizz"를 리턴한다.
2.  5의 배수인 5, 10, 20이 입력되면 "Buzz"를 리턴한다.
3.  15의 배수인 15, 30, 45가 입력되면 "FizzBuzz"를 리턴한다.
4.  3과 5의 배수가 아닌 수인 1, 2, 4, 7, 8이 입력이 되면 해당 수에 대한 문자열을 리턴한다.

#### 참고
TDD를 처음 접하시는 분이 위 스텝에 대한 테스트를 작성하기 힘들 수 있다.
그런 분들은 [TestCommon 소스코드](https://github.com/jwchung/TestCommon)에서 [FizzBuzzTest](https://github.com/jwChung/TestCommon/blob/master/demo/TDDKata/FizzBuzzKata/FizzBuzzTest.cs)
코드를 참고하시길 바란다.
물론 [FizzBuzz](https://github.com/jwChung/TestCommon/blob/master/demo/TDDKata/FizzBuzzKata/FizzBuzz.cs)에 대한 구현코드도 있는데, 비교대상으로만 이용하시기 바란다.

*   FizzBuzzTest:  
https://github.com/jwChung/TestCommon/blob/master/demo/TDDKata/FizzBuzzKata/FizzBuzzTest.cs
*   FizzBuzz:  
https://github.com/jwChung/TestCommon/blob/master/demo/TDDKata/FizzBuzzKata/FizzBuzz.cs

TestCommon 소스코드에 있는 FizzBuzzTest는 TestCommon을 이용하여 작성되었다.
TestCommon 인스톨 방법과 테스트 실행방법에 아래 포스트를 참고하기 바란다.

*   [How to Get TestCommon](/How-to-Get-TestCommon)
*   [How to Run Tests Using TestCommon](/How-to-Run-Tests-Using-TestCommon)
