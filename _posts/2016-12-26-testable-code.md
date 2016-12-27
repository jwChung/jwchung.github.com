---
layout: post
title: Testable Code
tags : [unit-testing, tdd]
---
{% include JB/setup %}

테스트 작성을 어려워 하는 분들에게 제가 처음 드리는 조언은 테스트 작성을 너무 어렵게 생각하지 말고, 메소드 검증을 위한 콘솔 응용프로그램을 가볍게 작성하라는 것입니다. 테스트 잘 작성하려면 테스트 관련 지식과 경험이 필요하지만 시작 단계에서 부터 무겁게 접근하는 것은 바람직하지 않기 때문입니다.

시작에 앞서 이 글에서 사용된 테스트라는 용어는 다음과 같은 의미로 사용됩니다. 이 글에서 예제로 보여지는 코드는 C#으로 작성되었습니다. 하지만 Java 또는 다른 OO언어를 경험하셨다면 이 글을 이해하는데는 문제 되지 않는다고 생각합니다.

> 이 글에서 테스트라는 용어는 코드로 작성된 자동화 단위테스트를 의미합니다. 프로덕션 코드보다 선행되어 테스트가 작성되는 TDD의미는 포함되지 않습니다.

 최근 사내 후배가 테스트를 작성하고 싶은데 테스트를 잘 알지 못하고, 경험이 적어서 어려움이 있다는 얘기를 했습니다. 후배는 프로덕션 코드를 먼저 작성 후 테스트로 작성된 코드를 검증하고자 했습니다. 그런데 테스트 대상 프로덕션 코드는 테스트가 거의 불가능했습니다. 후배가 테스트를 작성하기 힘든 이유는 테스트에 대한 경험 혹은 관련 지식이 부족해서가 아니라 엄밀히 말하면, 대상 프로덕션 코드 자체가 테스트할 수 없는 상태였던 것입니다.  이 글을 통해 후배의 프로덕션 코드가 왜 테스트가 불가능했던 것인지, 어떻게 하면 **테스트하기 쉬운 코드(testable code)** 를 작성할 수 있는지에 대해 얘기해보려 합니다.
 
 테스트 관련하여 [어쭙지 않은 글](/Unit-Test의-물리적-이해)을 써 놓고 꾀나 긴 시간이 흘렀습니다. 그 동안 저는 어떻게 하면 테스트를 잘 작성할 수 있는지에만 온통 관심이 있었습니다. 그런데 테스트 작성은 테스트 자체 지식과 경험 뿐아니라, 프로덕션 코드가 얼마나 테스트하기 쉽게 작성되는지에도 밀접한 관련이 있다는 사실을 새삼 일깨우게 되었습니다.  Sergey Kolodiy는 [Unit Tests, How to Write Testable Code and Why it Matters](https://www.toptal.com/qa/how-to-write-testable-code-and-why-it-matters) 라는 글을 통해 테스트하기 쉬운 코드 작성을 위해 필요한 사항 및 방법을 잘 설명하고 있습니다. 이 글과 연관된 글로써 읽어볼만한 글이라 생각합니다.

<!-- break -->

### non-deterministic
테스트를 어렵게 만드는 요인 중 하나가 **불확실성(non-deterministic)** 입니다. 아래 `GetAMOrPM` 메소드는 현재 시간에 따라 AM / PM을 반환하고 있습니다. 이 메소드는 테스트하는 시각에 따라 변하기 때문에 테스트가 불가능합니다. 
```c#
public string GetAMOrPM()
{
    var now = DateTime.Now;
    if (now.Hour < 12)
    {
        return "AM";
    }
    else
    {
        return "PM";
    }
}
```
시간 외에도 테스트 대상 코드가 Random 수를 사용하거나 또는 IO에 의존한다면 불확실성을 유발합니다. 다음의 `GetUserNameBy` 메소드는 `userId`를 가지고 해당 사용자 이름을 반환합니다. 이 메소드는 네트워크 IO에 의존하며 테스트되려면 데이터베이스가 사용되어야 합니다. 데이터베이스는 테스트 외부 환경에 놓여져 테스트에 필요한 조건이 미리 설정되어야 합니다. 이런 설정은 어렵거나 불가능한 경우도 있습니다. 설정이 되었다 하더라도 다른 외부요인에 의해 설정한 테스트 조건이 흩트러질 수 있습니다. 불확실성은 항상 존재합니다.

```c#
public string GetUserNameBy(string userId)
{
    using(var context = new EnvicaseDbContext())
    {
        var query = from user in context.Users
                    where user.Id == userId
                    select user;

        return query.SingleOrDefault();
    }
}
```

### side effect
불확실성은 테스트를 어렵게 만드는 첫 번째 요인입니다. **부수효과(side effect)** 는 그 뒤를 이어 테스트를 어렵게 만드는 두 번째 요인입니다. 데이터베이스에 데이터를 입력 또는 수정하는 것과 같이 관측가능한 시스템 상태를 변경하는 행위를 부수효과라 합니다.

회원가입 로직 처리를 위해 `email`과 `password`를 입력 받아 `SignUp` 이라는 메소드 구현을 생각해보겠습니다. `email`과 `password`가 유효한 값이면 새로운 사용자 계정이 데이터베이스에 생성됩니다. 사용자 계정 생성이 끝나면 입력 받은 이메일로 본인 인증 메일이 발송되는 시나리오를 생각해볼 수 있습니다. 이때 새로운 사용자 계정이 데이터베이스 생성되는 것, 그리고 본인 인증 메일이 전송되는 것은 부수효과입니다. 데이터베이스 부수효과를 테스트하려면 데이터베이스를 미리 테스트 조건에 맞게 설정해야 하고 생성된 데이터 행을 테스트가 직접 접근해서 읽어와야 하는 어려움이 따릅니다. 더욱이 인증 메일이 잘 전송되었는지 확인하는 부수효과 테스트는 검증 로직 작성이 매우 어렵습니다. 부수효과는 테스트를 어렵게 만들거나 불가능하게 만드는 요인인 것입니다.

### pure fuction
테스트를 어렵게 만드는 요인을 제거하면 당연히 테스트하기 쉬운 코드가 됩니다. 테스트를 방해하는 요인이 앞서 살펴 본 불확실성과 부수효과입니다. 함수형 패러다임에서 불확실성과 부수효과가 없는 함수를 가리켜 **순수함수(pure fuction)** 라 정의합니다. 테스트하기 쉬운 코드란 불확실성과 부수효과를 가지 않는 코드 또는 순수함수 형태의 코드라 할 수 있습니다.

## isolation
소프트웨어는 테스트 하기 쉬운 코드만으로 이루어질 수 없습니다. 우리는 데이터베이스로 부터 사용자 정보를 조회하고, 메일을 전송해야하거나 사용자 계정을 데이터베이스에 저장해야 합니다. 이렇게 불확실성과 부수효과 유발시키는 시나리오는 테스트하기 쉬운코드와 어려운(또는 불가능한) 코드가 함께 공존하게 됩니다. 가령 아래 `SignUp` 는 회원가입을 처리하는 메소드입니다. 세부코드를 구현을 생략하고 메소드가 구현해야될 4가지 요구사항을 주석을 설명했습니다. 1번과 2번 요구사항은 불확실성과 부수효과가 없기때문에 테스트하기 쉽지만, 3번과 4번 요구사항은 테스트하기 어렵습니다. 테스트하기 어려운 코드를 포함하는 `SignUp` 는 어떻게 하면 테스트하기 쉽게 만들까요? 그 해답은 **격리(isolation)** 에 있습니다. 

```c#
public void SignUp(string email, string password)
{
    // 1. 이메일 형식이 올바른지 확인 후 올바르지 않으면 예외발생
    ...

    // 2. 비밀번호 형식이 올바른지 확인 후 올바르지 않으면 예외발생
    ...

    // 3. 데이터베이스에 새로운 계정 생성
    ...

    // 4. 이메일로 본인 인증메일 전송
    ...
}
```
격리를 잘 설명한 문장을 Mark Seemann의 [Functional design is intrinsically testable](http://blog.ploeh.dk/2015/05/07/functional-design-is-intrinsically-testable/) 라는 글에서 읽었습니다. 글 본문에 나온 [Jessica Kerr](http://blog.jessitron.com/) 의 정의는 아래와 같습니다.

> A function has the property of Isolation when the only information it has about the external world is passed into it via arguments.

3번과 4번 구현코드는 데이터베이스와 이메일전송 시스템과 같은 외부환경에 의존하게 되므로 테스트가 어렵습니다. 이 외부환경을 메소드 인자(argument)로 격리시킨다면 `SignUp` 메소드를 테스트하기 쉬운 형태로 변경시킬 수 있습니다. 아래 코드는 테스트를 어렵게 만드는 3번과 4번 요구사항 구현코드를 격리시키고 있는 예를 보여주고 있습니다. 생성자에서 넘겨 받는 `IUserStore` 와 `IEmailSender` 인스턴스를 [테스트더블](http://xunitpatterns.com/Test%20Double.html) 로 처리할 수 있습니다. `SignUp` 메소드는 외부환경에 구애받지 않고 테스트될 수 있습니다.

```c#
public class UserService
{
    public UserService(IUserStore userStore, IEmailSender emailSender)
    {
        this.UserStore = userStore;
        this.EmailSender = emailSender;
    }

    public IUserStore UserStore { get; }

    public IEmailSender EmailSender { get; }

    public void SignUp(string email, string password)
    {
        // 1. 이메일 형식이 올바른지 확인 후 올바르지 않으면 예외발생
        if(!new EmailVaildator().Validate(email))
            throw new ArgumentException("이메일 형식이 올바르지 않습니다.");

        // 2. 비밀번호 형식이 올바른지 확인 후 올바르지 않으면 예외발생
        ...

        // 3. 데이터베이스에 새로운 계정 생성
        this.UserStore.Save(new User(email, password));

        // 4. 이메일로 본인 인증메일 전송
        this.EmailSender.Send(email, "인증메일 제목", "인증메일 내용");
    }

    private class EmailVaildator
    {
        public bool Validate(string email)
        {
            ...
        }
    }
}

public interface IUserStore
{
    void Save(User user);
}

public interface IEmailSender
{
    void Send(string toMail, string subject, string body);
}
``` 