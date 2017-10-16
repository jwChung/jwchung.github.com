---
layout: post
title: Test Humility
tags : [testing, unit-testing, tdd]
---
{% include JB/setup %}

_이 글에서 별도 설명없이 사용되는 테스트라는 용어는 코드로 수행되는 자동화테스트를 의미한다._

[이전 글에서 테스트하기 쉬운코드가 무엇인지 알아보았다.](/testable-code) IO 작업과 관련되지 않은 순수함수 형태가 테스트하기 쉬운 코드였다. 순수함수 코드는 입력 값이 같으면 항상 동일한 결과값을 리턴한다. 외부세상을 변경시키는 부수효과 또한 없다. 코드로 수행되는 자동화테스트 검증에 적합한 경우다. 반면 IO 작업과 관련된 비순수함수 테스트는 항상 고비용이 요구되는 것은 아니다. 테스트 비용을 줄일 수 있는 도구들이 속속 등장하고 있기 때문이다. [원격 저장소와 동일한 기능을 로컬 머신에서 실행하고 테스트할 수 있는 에뮬레이터](https://docs.microsoft.com/en-us/azure/storage/common/storage-use-emulator), [설정과 관리비용이 적은 로컬 데이터베이스](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/sql-server-2016-express-localdb), 그리고 [메모리를 데이터베이스 저장소 활용하여 테스트할 수 있는 도구도 있다.](https://docs.microsoft.com/en-us/ef/core/providers/in-memory/) 자동화테스트 검증 비용이 낮아짐으로 기존에 비용이 높아 자동화테스트 범주에 속하지 못했던 코드들이 점차 그 테두리 안으로 들어오고 있다.

테스트 용이성은 상대적인 것이기 때문에 한 편에서 느끼는 테스트 비용부담은 다른 편에선 그렇지 않게 느낄 수 있다. 예를들어 비밀번호 재설정을 위한 메일 발송 시나리오를 생각해보자. 이메일이 잘 발송되었는지 자동으로 테스트하려면 실제 메일서비스에 접속하여 메일이 잘 도착되었는지 코드를 통해 검증해야 한다. 이 검증에 대해 비용이 높다는 사람이 있는 반면, 자동화테스트에서 오는 회귀검증의 실이익이 더 크다고 생각하는 사람도 있다. 

<!-- break -->

### Humble Object

중요한 사실은 비용이 높은 경우이건 또는 테스트가 불가능한 경우이건, 우리는 자동화테스트를 이행할 수 없는 경우에 놓일 수 있다는 것이다. 내가 테스트 초심자였을 때 테스트하기 어려운 코드를 만나면, 어떻게 테스트 비용을 낮춰 자동화할 수 있을지만 고민했었다. 테스트하지 말아야 한다는 생각을 못했다. 그러던 중 만난 [xUnit Patterns 책](https://www.amazon.com/xUnit-Test-Patterns-Refactoring-Code/dp/0131495054)에선 [다음과 같은 사실을 일깨워 주었다. 목마른 사막에서 만난 오아시스 같았다.](http://xunitpatterns.com/Humble%20Object.html)

> 테스트가 어려우면 테스트하지마라. 단, 그 코드는 최대한 겸손(humility)해야 한다.

위 문장에서 테스트하지 말라는 뜻은 자동화테스트에 한정된다. 눈으로 확인하는 수동테스트까지 하지말라는 뜻은 아니다. 겸손의 의미는 테스트할 수 없는 코드는 최대한 테스트가 필요없을 만큼 로직이 단순해야 한다는 뜻이다. 사용자 정보(이메일과 비밀번호)를 가지고 회원가입하는 아래 `SignUp` 메소드를 살펴보자. C#으로 작성된 코드이며, `...` 표시는 코드가 생략되었음을 의미한다. 여기서 사용자 정보 저장(`UserStore.AddAsync`)이 테스트 비용이 높아 테스트할 수 없는 경우라 가정해보자. `UserStore.AddAsync`를 테스트할 수 없기 때문에 `SignUp`도 테스트하기 어렵다. 테스트하기 어려우면 테스트를 포기하고 대신 `SignUp` 메소드를 겸손하게 만들어 보자.

```c#
public async Task SignUp(string email, string password)
{
    // 이메일이 유효한지 검사합니다.
    if (!email.Contains("@"))
        throw new ArgumentException("유효한 이메일 형식이 아닙니다.");
    ...

    // 비밀번호가 유효한지 검사합니다.
    if (password.Length < 8)
        throw new ArgumentException("비밀번호는 최소 8자리 이상입니다.");
    ...

    await UserStore.AddAsync(email, passwod);
}
```

테스트 가능한 코드를 최대한 `SingUp`에서 분리시켰다. [이전 글에서 테스트 가능한 코드를 작성하려면, 테스트하기 어려운 코드와 쉬운 코드 분리하자고 한 것과 연결되는 내용이다.](/how-to-write-more-testable-code#테스트하기-어려운-코드와-쉬운-코드-분리) [`Email` 클래스](/how-to-write-more-testable-code#테스트하기-어려운-코드와-쉬운-코드-분리)와 같은 방법(Primitive Obsession)으로 구현된 `Password` 클래스를 사용해보자. 이 두 클래스를 적용해 아래와 같은 코드를 작성할 수 있다. `SignUp` 자체를 테스트할 순 없지만, `Email` 클래스와 `Password` 클래스는 이와 분리시켜 테스트할 수 있다. `SignUp` 메소드를 겸손하게 만든 결과다.

```c#
public async Task SignUp(string email, string password)
{
    var emailObj = new Email(email);
    var passwordObj = new Password(password);

    await UserStore.AddAsync(emailObj.Value, passwordObj.Value);
}
```

### Summary

모든 코드를 테스트할 순 없다. 비용이 높아서 그럴 수도 있고, 테스트가 불가능할 수도 있다. 테스트할 수 없는 코드는 말 그대로 자동화된 검증을 수행하지 못하므로 겸손할 필요가 있다. 테스트 가능한 코드는 최대한 분리되어야 하며 테스트 불가능한 코드의 로직은 단순해야 한다. 테스트가 어려운 UI 코드(View)에서 테스트 가능한 코드를 최대한 분리하여 Controller, Presenter 또는 ViewModel 에 위치 시키는 것도 같은 이치다.