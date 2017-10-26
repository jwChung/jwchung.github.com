---
layout: post
title: Sometimes tell, sometimes ask
tags : [oop]
---
{% include JB/setup %}

필요한 데이터를 묻기보다, 하고 싶은 걸 말해라는 Tell, Don't Ask(이하 TDA) 원칙이 있다. 필요한 데이터 요청없이 행위를 실행되려면, 행위와 데이터가 함께 있어야 한다. Oriented Object Programming(이하 OOP)은 데이터(필드)와 관련 행위(메소드)를 클래스로 한데 묶는 것이 특징인데, TDA는 이를 잘 살려주는 도구라 할만하다.

아래 `UserAccount` 클래스는 `email`과 `password` 데이터로 표현되며, 데이터베이스에 사용자 계정을 저장하기 위해 `Save`라는 행위를 가지고 있다. 데이터를 표현하는 필드, 행위를 의미하는 메소드가 동일 클래스에 위치하기 때문에 필요한 데이터를 따로 요청할 필요가 없다. TDA를 따른 것이다.

TDA가 OOP의 특징을 살려주는 도구이긴 하지만, 다른 시각에서 보면 문제가 있다.

<!-- break -->

```c#
// C#으로 작성된 코드이며, ...는 코드가 생략되었다는 뜻이다.

public class UserAcccount
{
    private string email;
    private string password;

    // 필드 초기화
    ...

    public void Save()
    {
        // 데이터베이스에 저장
        ...
    }
}
```

### Open/Closed Principle(OCP)

사용자 계정을 데이터베이스에 저장하는 대신, 파일로 저장해야 하는 수정사항이 생겼다고 하자. 이 요구사항을 반영하려면 `Save` 메소드 내부 코드를 수정해야만 한다. [확장에는 열려있어야 하고, 변경에는 닫혀 있어야 한다는 OCP 원칙을 위반한 것이다.](http://blog.ploeh.dk/2012/01/03/SOLIDisAppend-only/)

그럼 아래 코드를 보자. `DbStore` 클래스가 `Save` 메소드를 가지고 있고, 데이터베이스에 저장할 데이터를 `UserAcccount` 인스턴스에 요청하고 있다. 하고 싶은 걸 말하지 않고, 필요한 데이터를 묻는 경우다. TDA를 따르지 않는다. 그러나, 이것은 사용자 계정을 파일로 저장할 새로운 `FileStore` 클래스 작성을 가능하게 한다. 코드를 수정하거나 삭제하는 일 없이, 파일로 저장하는 새로운 기능을 추가할 수 있게 되었다. OCP 원칙을 위배하지 않은 것이다.

```c#
public class UserAcccount
{
    // 속성 초기화
    ...

    public string Email { get; }

    public string Password { get; }
}

public class DbStore
{
    public void Save(UserAccount account)
    {
        string email = acccount.Email; // TDA 위배
        string password = account.Password;

        // 데이터베이스에 저장
        ...
    }
}
```

### Single Responsibility Principle(SRP)

이제 TDA 원칙를 지키면서 사용자 이메일 주소로 메일을 전송하는 기능을 추가해보자. 이를 구현하면 아래 코드와 같은 모습이 된다. 추가된 `SendEmail` 메소드는 앞서 언급한 OCP 위배와 같은 문제를 가진다. 여기서 `UserAcccount` 클래스는 계정 저장과 메일 전송이라는 두 가지 책임을 가기게 된다.

_메소드가 두 개이기 때문에 책임이 두 개가 생긴 것이 아니다. 만약 `Save` 메소드에 `Remove` 메소드를 추가했다면, 책임이 추가 되었다고 얘기하기 보다 기능이 되었다고 했을 것이다._

둘 중 하나의 기능이 변경되면, 코드 수정이 필요하다. 클래스를 수정하는 이유는 단 하나여야 한다는 SRP 원칙을 위배한 것이다.

```c#
public class UserAcccount
{
    private string email;
    private string password;

    ...

    public void Save()
    {
        ...
    }

    public void SendEmail()
    {
        ...
    }
}
```

SRP 원칙을 지키는 방법은 간단하다. 위 OCP 경우에서 `FileStore` 클래스를 새로 만든 것처럼, 메일 전송을 위한 새 `EmailService` 클래스를 만들면 된다. 이 클래스는 메일 전송을 위해 `UserAcccount` 인스턴스에 사용자 이메일 데이터를 요청할 것이다. SRP 원칙을 따르기 위해 TSA 원칙은 포기해야 한다.

### Why OCP and SRP matter?

데이터가 관련 행위를 갖는다는 OOP 특징을 포기하면서 OCP 와 SRP 를 따르는 것이 맞는것인가? 
OCP와 SRP가 왜 중요한지를 얘기하는 것은 이 글을 벗어난 주제이다. 하지만, 아무런 설명없이 OCP 또는 SRP 위배는 문제다고 얘기할 순 없는 노릇이다.