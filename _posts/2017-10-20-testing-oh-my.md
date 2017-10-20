---
layout: post
title: Testing, Oh my!
tags : [testing-oh-my]
---
{% include JB/setup %}

_이 글에서 별도 설명없이 사용되는 테스트라는 용어는 코드로 수행되는 자동화테스트를 의미한다._

예전에 회사 후배가 어떤 코드를 테스트하고 싶은데, 아직 경험과 스킬이 부족해서 힘들다란 말을 했다. 그 후배가 테스트하고자 하는 코드(System Under Test, 이하 SUT)는 테스트하기 어려운 IO 로직에 직접 의존하고 있어 테스트가 거의 불가능했다. 수동테스트 밖에 길이 없었던 것이었다. 

테스트에 어려움을 느끼는 초심자의 큰 오해 중 하나는, 그 어려움이 테스트 스킬 부족 때문이라고 생각하는 것이다. 테스트 스킬이 중요하지 않은 것은 아니지만, 그보다 SUT가 얼마나 테스트하기 쉽게 디자인되었는지가 더 중요한 요소이다. 후배의 예처럼 IO 로직 같은 테스트하기 어려운 코드를 직접 참조해버리면 아무리 뛰어난 스킬이 있더라도 손과 눈으로 수동테스트 하는 방법 밖엔 없는 것이다.

_[나는 TDD(Test-Driven Development)가 디자인 도구라 생각하지 않는다.](https://www.facebook.com/jinwook.chung.167/posts/1890555361179897) 테스트가 디자인을 이끄는 것이 아니라 좋은 디자인이 테스트하기 쉬운 코드를 만든다고 생각한다. 여기서 다뤄지는 내용은 테스트를 먼저 작성하든(TDD), 테스트를 나중에 작성하든 디자인은 그것들보다 먼저 계획된다는 관점에서 기술되었다._

그 후배에게 들려줬던 테스트 방법에 대한 내용을 글로 써봤다. 어떤 코드가 테스트하기 쉬운지 또는 어려운지([Testable Code](/testable-code)), 테스트가 쉬운 코드는 어떻게 작성하는지([How to write more testable code](/how-to-write-more-testable-code)), 테스트가 어려운 코드는 어떻게 다뤄야 하는지([Test Humility](/test-humility)), 그리고 테스트하기 쉬운 코드와 어려운 코드가 만나게 되는 지점은 어떤 테스트 방법이 있는지([Function Root Testing](/function-root-testing))에 대한 내용들이다.

<!-- break -->

글의 순서는 다음과 같다. 작성 편이를 위해 경어는 생략했다.

1. [Testable Code](/testable-code)
2. [How to write more testable code](/how-to-write-more-testable-code)
3. [Test Humility](/test-humility)
4. [Function Root Testing](/function-root-testing)