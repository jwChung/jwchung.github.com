---
layout: post
title: TDD Kata - StringCalculator
tags : [TDD Kata, TestCommon]
date : 2013-04-03 07:35:00 UTC
---
{% include JB/setup %}

이번 글이 TDD Kata에 대한 두 번째 포스팅이다.
이전 [FizzBuzz TDD Kata](/TDD-Kata-FizzBuzz-게임/)는 쉬운 TDD Kata 예제 중 하나다.
이번에 소개할 [StringCalculator TDD Kata] 예제는 The Art of Unit Testing 저자인
Roy Osherove가 만든 예제로 유명한 예제 중 하나다.

그럼 시작해 보자.

### StringCalculator TDD Kata

#### 유의사항
StringCalculator TDD Kata에서 Roy Osherove는 아래와 같은 사항에 유의하라고 한다.([블로그][StringCalculator TDD Kata] 참고)

*   다음단계의 스텝을 미리 읽지 않는다.
*   한번에 하나의 스텝만 수행한다. 이것은 점진적으로 프로그래밍되는 것을 배우기 위한 것이다.
*   유효한 메소드 파라메타만 고려한다. 유효하지 않는 파라메타를 테스트할 필요가 없다.



#### Steps
*   StringCalculator 클래스를 만들고 `int Add(string numbers)`메소드를 추가한다.  
이 메소드는 빈문자열("")에 대해서 0을 리턴한다.
오직 테스트를 패스하기 위한 목적으로 가능한 간단한 코드를 작성해야한다.
이는 생각하지 못한 테스트를 작성할 수 있게끔 도와준다.
*   1개의 숫자를 받아(예 "1") 해당 숫자를 리턴하라.
*   2개의 숫자를 받아(예 "1,2") 해당 숫자들의 합을 리턴하라.
*   n개의 숫자를 받아(예 "1,2,3,4...") 그들의 합을 리턴하라.
*   콤마(",")외에 개행문자("\n")도 숫자 구분자로 사용 가능하게 하라.  
예를 들어, "1\n2,3"은 6을 리턴한다.
그러나 "1,\n"와 같이 연속된 구분자는 고려할 필요가 없다.
*   "//[delimiter]\n[numbers…]" 형태로 사용자가 지정한 char를 구분자로 사용 가능하게 하라.  
예를 들어, "//;\n1;2"는 3을 리턴한다.
*   만약 마이너스 정수가 넘어오면 해당 정수와 "negatives not allowed"라는 메세지를 가지는 예외를 발생시킨다.
마이너스 숫자가 여러개일 경우 모두 예외메세지에 포함시킨다.
*   **TDD 초급자의 경우는 여기서 잠깐**  
현 단계까지 걸리는 시간이 30분 미만이면 다음 스텝으로 이동하고 그렇지 못한 경우는 다시 연습한다.
*   1000보다 큰 수는 합계에서 제외시킨다.  
예를 들어,  2 + 1001  = 2
*   사용자 구분자가 다음과 같은 형태로 문자열도 가능하게 하라. "//[delimiter]\n"  
예를 들어, "//[---]\n1---2---3"는 6을 리턴한다.
*   사용자 구분자가 다음과 같은 형태로 여러 개 입력할 수 있도록 해라. "//[delimiter1][delimiter2]\n"  
예를 들어, "//[--][%%]\n1--2%%3"는 6을 리턴한다.

<!-- break -->

#### 참고
*   [StringCalculatorTest](https://github.com/jwChung/TestCommon/blob/master/demo/TDDKata/StringCalculatorKata/StringCalculatorTest.cs):  
https://github.com/jwChung/TestCommon/blob/master/demo/TDDKata/StringCalculatorKata/StringCalculatorTest.cs
*   [StringCalculator](https://github.com/jwChung/TestCommon/blob/master/demo/TDDKata/StringCalculatorKata/StringCalculator.cs):  
https://github.com/jwChung/TestCommon/blob/master/demo/TDDKata/StringCalculatorKata/StringCalculator.cs

[StringCalculator TDD Kata]: http://osherove.com/tdd-kata-1/