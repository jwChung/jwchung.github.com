---
layout: post
title: Isolating Static Method Call
tags : [Unit Testing, TestCommon]
date : 2013-04-04 13:49:00 UTC
published: false
---
{% include JB/setup %}

아래와 같이 `string` 타입의 message를 Log.txt 파일에다 기록하는 
`FileLogger` 클래스가 있다고 가정하자.

```c#
public class FileLogger
{
    public virtual void Log(string message)
    {
        using (var sw = new StreamWriter("log.txt", append: true))
        {
            sw.WriteLine(message);
        }
    }
}
```

`FileLogger` 클래스를 사용하여 Logging을 하는
`FileLoggerClient`가 아래와 같이 있다고 하자.

```c#

public class FileLoggerClient
{
    private readonly FileLogger _fileLogger;

    public FileLoggerClient(FileLogger fileLogger)
    {
        _fileLogger = fileLogger;
    }

    public void DoSomething()
    {
        _fileLogger.Log("Did something...");
    }
}
```

만일, `DoSomething` 메소드에 대한 단위테스트가 필요하고,
이를 위해 실제 `FileLogger` 타입의 객체가 사용된다면,
단위테스트 실행 시마다 Log.txt 파일에 로그메세지를 기록하게 될 것이다.
만약 Log.txt 파일을 만들 수 없거나 파일에 메세지를 기록할 수 없는 등의 파일관련 애러가 발생되면
`DoSomething` 메소드의 옳고 그름을 떠나 테스트는 실패하게 된다.

더욱이, `FileLogger` 클래스는 설명의 이해를 높이기 위해 파일에다 로그메세지를 작성하였지만,
만약 데이터베이스, 웹서비스 또는 메일을 통해 로그메세지를 작성하는 경우라면,
하나의 단위테스트 실행하기 위해 많은 시간이 소요되게 됨으로 문제가 된다.
이를 해결하기 위해서는 `FileLogger` 클래스를 [Test Double]로 대체하여
실제 `FileLogger.Log(string)` 메소드를 호출을 고립(isolation)시켜야 한다.

TestCommon의 [AutoTheory 기능]을 사용하면
자동으로 [Moq] 프레임워크를 이용하여 `FileLogger` 타입의 Mocked 인스턴스를 생성해주는데
아래와 같이 실제 `FileLogger.Log` 메소드를 호출하지 않는 단위테스트가 가능하다.


```c#
public class LoggerClientTest
{
    [AutoTheory]
    public void DoSomething_LogsCorrectMessage(
        [ToCtor] FileLogger fileLogger,
        [Build] FileLoggerClient sut)
    {
        // Arrange
        // Act
        sut.DoSomething();

        // Assert
        fileLogger.ToMock().Verify(x => x.Log("Did something..."));
    }
}
```

<!-- break -->

`FileLogger.Log` 메소드와 같이 인스턴스의 메소드는 해당 인스턴스의 타입에 대한 Test Double을 작성함으로써 고립이 가능하다.
이는 OOP의 다형성을 기초한 것으로, 만약 특정 단위테스트에서 아래와 같은 `StaticFileLogger.Log` **static** 메소드 호출이 고립의 대상이 된다면,
위와 같은 방법으로는 static 메소드 호출을 고립시키는 것은 불가능하다.

_주) static 메소드는 일반적으로 스레드 안정성을 보장해야 하지만 `StaticFileLogger.Log` 메소드는 그렇지 못하다.
스레드 안정성 문제는 이 글을 범어나는 범위이므로 이 글에서는 고려치 않기로 한다._



```c#
public class StaticFileLogger
{
    public static void Log(string message)
    {
        using (var sw = new StreamWriter("log.txt", append: true))
        {
            sw.WriteLine(message);
        }
    }
}
```

OOP의 다형성을 기반으로 작성된 Test Double 인스턴스를 통해서는 static 메소드 호출을 고립하는 것은 원칙적으로는 불가능하다.

_주) static 메소드는 특정 Test Double 프레임워크(Moles, Microsoft Fakes 등)를 사용하면 가능하나,
일반적인 OOP개념이 아니므로, 이 글에서는 이에 대한 내용은 다루지 않을 것이다._

그럼 어떻게 해야할까? 사실 static 메소드는 "클래스명.메소드명(파라메타)"의 형식으로 호출되는 대상이 하나가 된다.
만약 static 메소드처럼 호출대상을 하나의 인스턴스의 메소드로 결집시킨다면,
인스턴스의 메소드가 static 메소드 처럼 동작하게 할 수 있을 것이다.
즉, 실제 static 메소드 호출을 고립시키는 것이 아니라 인스턴스 메소드를 고립시키는 방법이다.

위 `StaticFileLogger` 클래스를 아래와 같이 수정해 보자.


```c#
public class StaticFileLogger
{
    static StaticFileLogger()
    {
        Static = new StaticClass();
    }

    internal static StaticClass Static { private get; set; }

    public static void Log(string message)
    {
        Static.Log(message);
    }

    internal class StaticClass
    {
        internal virtual void Log(string message)
        {
            using (var sw = new StreamWriter("log.txt", append: true))
            {
                sw.WriteLine(message);
            }
        }
    }
}
```

`StaticFileLogger.Log` 메소드 호출은 아래 코드와 같이 일반 static 메소드 호출과 다르지 않다.
그러나 `StaticFileLogger.Static` 프라퍼티의 Setter를 통해 `StaticFileLogger.StaticClass` 타입의 Test Double 인스턴스 설정이 가능함으로,
static 메소드 호출을 고립시키는 효과를 가져오는 것이다.

```c#
public class StaticFileLoggerClient
{
    public void DoSomething()
    {
        StaticFileLogger.Log("Did something...");
    }
}
```

즉, `StaticFileLoggerClient.DoSomething` 메소드에 대한 단위테스트를 작성하면 아래와 같이 된다.
`StaticFileLogger.Log` 메소드 호출을 고립시키기 위해 `StaticFileLogger.StaticClass` 타입의 Test Double 인스턴스
`staticClass`가 `StaticFileLogger.Static` 프라퍼티로 설정되어 단위테스트를 가능하게 하는 것이다.

여기서 주의할 점은 `StaticFileLogger.Static`는 static 프라퍼티로 이 테스트 뿐만 아니라 다른 곳에서도 사용된다.
그러므로, 아래 Teardown 부분에서와 같이 **사용하기 전의 상태로 꼭 돌려 놓아야 한다**.

```c#
public class StaticFileLoggerClientTest
{
    [AutoTheory]
    internal void DoSomething_LogsCorrectMessage(
        [Build(CallBase = false)] StaticFileLogger.StaticClass staticClass,
        StaticFileLoggerClient sut)
    {
        // Arrange
        StaticFileLogger.Static = staticClass;

        // Act
        sut.DoSomething();

        // Assert
        staticClass.ToMock().Verify(x => x.Log("Did something..."), Times.Once());

        // Teardown
        StaticFileLogger.Static = new StaticFileLogger.StaticClass();
    }
}
```

### 마치며...
이 글에서 위 `StaticFileLoggerClientTest`를 통해 보여준 방법이 static 메소드 호출 고립에 대한 유일한 해는 아니다.
내가 모르는 더 좋은 방법이 있을 수 있고,
경우에 따라선 실제 static 메소드를 직접호출(앞서의 예: Log.txt 파일에 로그메세지 기록)하는 테스트 방식을 더 좋아하는 사람도 있을 것이다.
따라서 여기서 말하는 static 메소드 호출 고립 방법은 하나의 대안으로 말씀 드리는 바이며,
여기에 대한 이견이나 코멘트가 있으신 분은 댓글을 남겨 주시기 바란다.

TestCommon에서 [Moq] 프레임워크를 통해 특정 타입의 Test Double 인스턴스를 생성하는데,
위에서 나온 `StaticFileLogger.StaticClass` 클래스와 같이 접근한정자가 internal인 타입에 대한 인스턴스를 생성하기 위해서는
아래와 같은 어셈블리 어트리뷰트를 작성해 주어야한다.

```c#
[assembly: InternalsVisibleTo("DynamicProxyGenAssembly2")]
```

해당 어셈블리가 강력한 이름으로 서명이 되어있다면, 아래와 같이 PublicKey 또한 같이 적어 주어야 한다.

```c#
[assembly: InternalsVisibleTo("DynamicProxyGenAssembly2, PublicKey=0024000004800000940000000602000000240000525341310004000001000100c547cac37abd99c8db225ef2f6c8a3602f3b3606cc9891605d02baa56104f4cfc0734aa39b93bf7852f7d9266654753cc297e7d2edfe0bac1cdcf9f717241550e0a7b191195b7667bb4f64bcb8e2121380fd1d9d46ad2d92d2d15605093924cceaf74c4861eff62abf69b9291ed0a340e113be11e6a7d3113e92484cf7045cc7")]
```

[Test Double]: http://xunitpatterns.com/Test%20Double.html
[AutoTheory 기능]: /TestCommon-AutoTheory-1
[Moq]: https://github.com/Moq/moq4