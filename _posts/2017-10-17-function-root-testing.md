---
layout: post
title: Function Root Testing
tags : [testing, unit-testing, tdd]
---
{% include JB/setup %}

[프로그램에서 각 기능 진입점 역할을 하는 함수 또는 메소드를 Function Root 란 용어로 소개한 바 있다.](/how-to-write-more-testable-code#function-root) [Function Root 는 대체로 외부세상과 소통하는 IO 작업 담고 있어 테스트하기 힘든 코드로 분류된다.](/how-to-write-more-testable-code#테스트하기-어려운-코드는-가장-바깥-쪽에-위치) 이 글에서는 Function Root 를 어떻게 테스트할 것인가에 대해 알아보려 한다. 먼저 [Function Root 를 테스트하기 어렵다는 시각에서 겸손하게 만드는 방법이 있다.](/test-humility) 겸손하게 된 Function Root 는 수동테스트로 검증된다. 자동화테스트할 수 있는 방법은 두 가지로 나뉜다. [Structual Inspection 을 통한 단위테스트로 검증하거나](http://blog.ploeh.dk/2013/04/04/structural-inspection/), Function Root 가 담고 있는 IO 로직을 직접 가로지르는 통합테스트로 검증할 수 있다.

<!-- break -->

Funtion Root 는 Web API 의 Endpoint 역할을 하는 액션메소드를 생각해보자. 각각의 테스트를 설명하기 위해 사용할 예제는 [이전 포스팅 사용한 회원가입 시나리오다.](/how-to-write-more-testable-code) 여기에 마지막에 인증메일 전송 단계를 추가로 고려해보자. 단계별 내용을 살펴보면 다음과 같다.

1. 입력받은 이메일이 유효한 형식인지 검증한다. 유효하지 않으면 402(Bad Request)를 반환한다.
2. 입력받은 비밀번호가 유효한 형식인지 검증한다. 유효하지 않으면 402(Bad Request)를 반환한다.
3. _이메일, 비밀번호를 데이터베이스에 저장한다._
4. _입력받은 이메일 주소로 이메일 인증메일을 전송한다._
5. 200(OK)을 반환한다.



### Humility 를 통한 수동테스트

### Structual Inspection 을 통한 단위테스트

### 통합테스트

### Summary
