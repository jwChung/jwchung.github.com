---
layout: post
title: Function Root Testing
tags : [testing, unit-testing, tdd]
---
{% include JB/setup %}

[프로그램에서 각 기능 진입점 역할을 하는 함수 또는 메소드를 Function Root 란 용어로 소개한 바 있다.](/how-to-write-more-testable-code#function-root) Function Root는 대체로 외부세상과 소통하는 IO 작업 담고 있어 테스트하기 힘든 코드로 분류된다. 테스트 비용을 줄이기 위해 테스트 가능한 코드는 Function Root로 부터 분리된다. 따라서 Function Root는 테스트하기 쉬운코드와 그렇지 않은 코드가 만나는 장소를 제공하며, 이들을 어떻게 구성하지가 주된 임무가 된다. Function Root는 순수함수코드와 비순수함수코드 전체를 가로지르는 통합테스트로 검증할 수 있다. 하지만 자동화테스트 비용이 높으면 이를 포기하고 수동테스트로 검증할 수도 있다. 이때 [Function Root는 겸손해야 한다.](/test-humility)

<!-- break -->

### Structure Inspection

Function Root 내 IO 작업을 Test Double로 고립(isolation)시킨다면 Function Root를 단위테스트(unit testing)로도 검증하는 것이 가능하다. 단위테스트는 다른 의존성으로 부터 고립된 최소 단위 API 테스트를 의미한다. 이때 고립이란, [Jessica Kerr](http://jessitron.com/) 의 표현을 빌리자면, [함수 파라메타를 통해서만 외부세상과 소통할 때 그 함수는 고립되었다고 한다.](http://blog.ploeh.dk/2015/05/07/functional-design-is-intrinsically-testable/)

> A function has the property of Isolation when the only information it has about the external word is passed into it via arguments.

[이전 글에서 사용한 회원가입 예제](/test-humility#humble-object)를 Web API Endpoint로 바꿔보자. 이를 단위테스트가 가능한 C# 코드로 작성해보면 다음과 같다. 좀 더 복잡한 시나리오를 위해 사용자 이메일을 확인하는 단계를 마지막에 추가했다.(아래 `EmailConfirmation.SendAsync` 메소드 호출 참고) IO 작업 코드를 고립시키기 위해 의존성을 직접 사용하지 않고 생성자를 통해 넘겨받고 있는 것을 확인할 수 있다.

```c#
public class AccountController : ApiController
{
    public AccountController(
        IUserStore userStore, IEmailConfirmation emailConfirmation)
    {
        this.UserStore = userStore
            ?? throw new ArgumentNullException(nameof(userStore));

        this.EmailConfirmation = emailConfirmation
            ?? throw new ArgumentNullException(nameof(emailConfirmation));
    }

    public IUserStore UserStore { get; }

    public IEmailConfirmation EmailConfirmation { get; }

    [HttpPost]
    public async Task<IHttpActionResult> SignUpAsync(
        string email, string password)
    {
        try
        {
            var emailObj = new Email(email);
            var passwordObj = new Password(password);

            await this.UserStore.AddAsync(emailObj, passwordObj);
            await this.EmailConfirmation.SendAsync(emailObj);

            return this.Ok();
        }
        catch (ArgumentException exception)
        {
            return this.BadRequest(exception.Message);
        }
    }
}
```
유효한 이메일과 비밀번호로 `SignUpAsync` 메소드 테스트를 C# 코드로 작성하면 아래와 같다. `IUserStore` 와 `IEmailConfirmation` 은 각각 `UserStoreSpy` 와 `EmailConfirmationSpy` 라는 Test Double로 대체되어 단위테스트를 가능하게 한다.

```c#
[Fact]
public async Task SignUpAsyncWithValidEmailAndPassowordReturnsOkResult()
{
    // Fixture setup
    var userStoreSpy = new UserStoreSpy();
    var emailConfirmationSpy = new EmailConfirmationSpy();

    var sut = new AccountController(userStoreSpy, emailConfirmationSpy);

    string email = "jwchung@hotmail.com";
    string password = "P@assW0rd";

    // Exercise
    await sut.SignUpAsync(email, password);

    // Verify outcome
    Assert.Equal(new Email(email), userStoreSpy.Email);
    Assert.Equals(new Password(password), userStoreSpy.Password);

    Assert.Equal(new Email(email), emailConfirmationSpy.Email);
}

private class UserStoreSpy : IUserStore
{
    public Email Email { get; set; }

    public Password Password { get; set; }

    public Task AddAsync(Email email, Password password)
    {
        this.Email = email;
        this.Password = password;

        return Task.FromResult<object>(null); // 빈 Task 반환
    }
}

private class EmailConfirmationSpy : IEmailConfirmation
{
    public Email Email { get; set; }
            
    public Task SendAsync(Email email)
    {
        this.Email = email;
        return Task.FromResult<object>(null); // 빈 Task 반환
    }
}
```

이같이 Function Root를 단위테스트로 접근하는 방식을 [Mark Seemann은 Structure Inspection이라고 했다.](http://blog.ploeh.dk/2013/04/04/structural-inspection/) Function Root를 이루는 각 단계가 테스트된 후 남은 일은 이들이 맞물려(composition) 전체가 잘 돌아가는지를 테스트하는 것이다. Structure Inspection은 복잡하게 구성된 시스템을 피드백이 빠른 단위테스트로 검증한다는 면에서 장점을 가진다. 하지만 다른 각도에서 생각해볼 문제가 있다.

### Reused Abstractions Principle(RAP)

추상화란 본디 공통점을 바탕으로 한다. 공통점이란 최소 두 가지 대상물이 존재해야 발견된다. 만약 단위테스트를 위한 고립 목적으로만 추상화가 사용되었다면 [이는 RAP 위배에 해당한다.](http://www.codemanship.co.uk/parlezuml/blog/?postid=934) RAP 관점에서 올바른 추상화란 구현체가 최소 2개 이상일 때를 말한다. 위 회원가입 구현은 RAP를 위반한 것인가? 그럴 수도 있고 아닐 수도 있다. 사용된 추상화(`IUserStore`, `IEmailConfirmation`)의 구현체가 얼마나 되는지 위 코드만으로는 확인될 수 없기 때문이다. 그럼 RAP를 위배한 코드는 나쁜가? 사용되지 않는 추상화를 도입했다는 것은 분명 비용 부담이다. 그러나 Function Root의 구성(composition)을 Structure Inspection으로 검증할 수 있는 것은 장점이다.

### Test-induced design damage

David Heinemeier Hansson(DHH)이 쓴 [TDD is dead](http://david.heinemeierhansson.com/2014/tdd-is-dead-long-live-testing.html) 라는 유명한 글이 있다. 이 글에서 그는 TDD가 죽은 주된 이유로 테스트에서 유발된 디자인 손상을 꼽는다. 주제에서 조금 벗어난 얘기를 하자면, DHH는 TDD를 디자인 도구로 생각하는 모양이다. [나는 개인적으로는 TDD를 디자인 도구로 생각하지 않는다.](https://www.facebook.com/jinwook.chung.167/posts/1890555361179897) 오히려 디자인이 좋을수록 TDD에 드는 비용이 절감되고, 잘못된 디자인은 TDD를 망친다고 생각한다.

다시 본론으로 돌아가 DHH 글에서 다음 내용을 살펴보자. 

> Test-first units leads to an overly complex web of intermediary objects and indirection in order to avoid doing anything that's "slow". Like hitting the database. Or file IO. Or going through the browser to test the whole system. It's given birth to some truly horrendous monstrosities of architecture. A dense jungle of service objects, command patterns, and worse.

> 테스트 우선개발 단위테스트는 느린 작업을 하지않기 위해 지나치게 복잡한 중간 객체와 간접 참조를 낳게 된다. 느린 작업 예로, 데이터베이스를 사용하는 작업, 파일 작업, 또는 전체시스템을 브라우져로 테스트하는 것을 들 수 있다. 아울러 테스트 우선개발 단위테스트로 인해 서비스 오브젝트, 커멘드 패턴 그리고 더 나쁜 것들이 모여 정글을 만드는데 이는 실로 끔직한 괴물과도 같은 아키텍처이다.

끔직한 괴물같은 아키텍처에 대해 그는 [Test-induced design damage라는 글에서 Hexagonal design damage이라는 내용으로 좀 더 자세히 얘기한다.](http://david.heinemeierhansson.com/2014/test-induced-design-damage.html) 단지 단위테스트의 빠른 실행을 위해 도입되는 복잡한 [Hexagonal 디자인](http://blog.ploeh.dk/2013/12/03/layers-onions-ports-adapters-its-all-the-same/)은 금지하고, 레일즈의 컨트롤러 같은 Function Root 검증은 단위테스트가 아니라 통합테스트가 더 적합하다라고 말이다.

DHH가 말하는 것 처럼 위 회원가입 코드에서 사용된 `IUserStore`, `IEmailConfirmation` 추상화는 테스트에서 유발된 디자인 손상(Hexagonal design damage)으로 봐야할까? 단위테스트의 빠른 피드백을 위한 것도 하나의 이유가 되지만 이들 추상화는 RAP를 준수하면서 의미있는 인터페이스가 될 수 있다는 게 나의 입장이다. 일례로 `IUserStore` 추상화를 통해 우리는 다양한 데이터 저장소에서를 사용할 수 있다. 이것이 비지니스에 중요한 요구사항이라면 `IUserStore`는 결코 디자인 손상이라 할 수 없는 것이다.

### Summary


 Function Root는 테스트하기 쉬운 코드와 그렇지 않은 IO관련 코드가 구성(composition)되는 곳이다. 이것에 포함되는 IO관련 코드를 고립시키면, Structure Inspection을 통해 전체 코드가 잘 구성되어 돌아가는지 단위테스트로 검증이 가능하다. 고립의 수단으로 도입된 추상화는 빠른 실행을 위해서만 존재하기 보다 추상화 본래 목적에 충실해야 한다. 추상화의 구현체가 하나일 경우는 RAP 위배이며, 추상화 도입의 설득력이 약해진다. 다만 RAP 위배에서 오는 단점보다, 구성이 잘 되었는가 단위테스트하여 얻는 빠른 피드백 장점이 더 큰지는 따져봐야 한다. 이 경우 현재 고려되지 못한 구현체가 있을 수 있는 점은 추상화 도입의 긍정적 요소로 평가될 수 있다.

Function Root를 단위테스트하는데 많은 비용이 든다면 통합테스트로 검증할 수도 있다. 이마저도 여의치 않으면 자동화테스트를 포기하고 Function Root를 [겸손하게 만들도록 하자.](/test-humility)
