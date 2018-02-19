---
layout: post
title: ASP.NET Core의 의존성 주입 기초
comments: true
tags: [aspnet-core, dependency-injection]
---

아래는 미리 허락을 받고 [Matthew Jones](https://exceptionnotfound.net/author/matthew-jones/)의 글([Getting Started with Dependency Injection in ASP.NET Core](https://exceptionnotfound.net/getting-started-with-dependency-injection-in-asp-net-core/))을  번역한 것입니다.

---

나는 요새 ASP.NET Core로 상용앱을 개발하며 신중하게 조금씩 공부해 나가는 중인데, 이 프레임워크의 특성 중 내가 가장 깊이 파고들었던 것이 의존성 주입(Dependency Injection; DI)의 기본 지원이다. DI는 현대 웹 앱에서 널리 쓰이고 있기 때문에, 이같은 결정은 ASP.NET Core의 커다란 진전이라고 본다.

그런데, 현 시점 기준으로, ASP.NET Core의 DI에 대한 공식 문서가 내 맘에 들지 않는다. 아마도 충분한 예제 코드를 제공하고 있지 않아서 그런 것 같다. 나는 이 글에서, 내가 직접 설계한 프로젝트와 공식 문서에서 나온 예제를 다시 만든 프로젝트, 두 가지를 소개할 것이다. 코드는 전부 [깃허브](https://github.com/exceptionnotfound/CoreDIDemo)에 공개되어 있다. 지금부터 ASP.NET Core에서 DI를 어떻게 사용하는지 배워보자!

# 의존성 주입이 뭔데?

우선, 몇 가지 기초 사항. DI의 개념은 다음과 같다. `A`라는 클래스를 만들 때, `A`가 의존하는 클래스들을 `A`로 생성하는 게 아니라, 그 클래스들을 `A`에 **주입**해야 한다는 것이다. DI는 느슨한 결합(loose coupling)과 높은 응집(high cohesion)을 가능하게 만들어 준다. 이는 많은 소프트웨어 프로그램들이 지향하는 이상이다.

ASP.NET Core 문서는 다음과 같이 설명한다:

> "[DI를 사용하면,] 어느 클래스가 자신의 동작을 수행하기 위해 필요한 객체들을 제공받기 위해, 협력 객체(collaborators)를 직접 객체화하거나 스태틱 참조(static references)를 사용하지 않는다. 대부분의 경우 클래스는 생성자를 통해 의존성을 선언하며, 이를 통해 [명시적 의존성 원칙(Explicit Dependencies Principle)](http://deviq.com/explicit-dependencies-principle/)을 따를 수 있다. 이러한 접근법을 **생성자 주입**이라고 부른다."

생성자 주입은 DI를 구현하는 방법들 중 단연코 가장 보편적인 접근법이다. ASP.NET Core 역시 생성자 주입을 사용하며, 우리도 실습해 볼 것이다.

마지막으로, 이외에도 다양한 DI 프레임워크가 있다는 것을 알아 두자. [StructureMap](http://structuremap.github.io/), [Autofac](https://autofac.org/), [Ninject](http://www.ninject.org/), [SimpleInjector](https://simpleinjector.org/index.html) 등등이 있다. 나는 StructureMap을 가장 좋아하지만, 대략적으로 보자면 모두  같은 일을 하는 셈이다.  써보고 가장 맘에 드는 걸 골라보자. 이 데모에서는 기본 ASP.NET Core DI 컨테이너를 사용한다.

## 간단한 DI 예제

ASP.NET Core 앱에서 DI가 어떻게 작동되는지 보기 위해, 간단한 MVC 데모 앱을 만들어 보자.

앱 사용자 목록을 모델링 해야한다고 가정하자. 이러한 사용자를 나타내는 클래스는 다음과 같다.

```c#
public class User
{
	public string FirstName { get; set; }  
  	public string LastName { get; set; }  
  	public DateTime DateOfBirth { get; set; }  
  	public string Username { get; set; }  
}
```

또한 사용자 검색을 위한 인터페이스와 리포지토리가 필요하다.

```c#
public interface IUserRepository
{
    List<User> GetAll();
}

public class UserRepository : IUserRepository
{
    public List<User> GetAll()
    {
        return new List<User>()
        {
            new User()
            {
                FirstName = "Ash",
                LastName = "Ketchum",
                DateOfBirth = new DateTime(1997, 12, 30),
                Username = "ichooseyou"
            },
            new User()
            {
                FirstName = "Brock",
                LastName = "Harrison",
                DateOfBirth = new DateTime(1992, 3,31),
                Username = "rockrulez"
            },
            new User()
            {
                FirstName = "Misty",
                LastName = "",
                DateOfBirth = new DateTime(1999, 5,4),
                Username = "ihearttogepi"
            }
        };
    }
}
```

우리의 MVC 컨트롤러가 참조할 것은 **인터페이스**이지, 클래스가 아니다. 컨트롤러를 하나 정의하자.

```c#
public class HomeController : Controller
{
	private readonly IUserRepository _userRepository;

	public HomeController(IUserRepository userRepository)
    {
    	_userRepository = userRepository;
    }

	[HttpGet]
	public IActionResult Index()
	{
		return View(_userRepository.GetAll());
	}
}
```

여기서 설정을 확인하자. 우리는  `IUserRepositoy`의  `private readonly` 인스턴스를 가지고 있으며, 생성자는 `IUserRepository` 타입의 인스턴스를 가져와서 우리의 private 인스턴스에 할당한다. 가장 간단한 형태의 생성자 주입이다.

하지만 아직 끝난 게 아니다. 예전에 블로그에서 다뤘듯, ASP.NET Core 프로젝트는 앱이 구동될 환경을 설정하는 [Startup.cs 파일을 가지고 있다](https://www.exceptionnotfound.net/the-startup-file-in-asp-net-core-1-0-what-does-it-do/). Startup.cs 파일은 ASP.NET Core의 서비스 계층에 서비스들을 배치하여 DI를 가능하게 만든다.

우리의 컨트롤러에 `IUserRepository`를 주입하려면 Startup.cs에 파일을 다음과 같이 고쳐야 한다.

```c#
public void ConfigureServices(IServiceCollection services)
{
  	// Add framework services.
  	services.AddMvc();
  
  	// Basic Demo
  	services.AddTransient<IUserRepository, UserRepository>();
}
```

`AddTransient<>()`가 무엇을 의미하는지는 잠시 후에 다루겠다. 지금은 우선 앱을 실행해 보자.

다음과 같은 화면이 보일 것이다.

![img](https://exceptionnotfound.net/content/images/2016/07/simple-di-demo-page.png)

일단 `IUserRepository`를 서비스 계층에 추가하니, 그것을 ASP.NET Core가 자동으로 `HomeController` 클래스에 주입했다는 것을 주목하자. 굉장히 깔끔하다!

## DI 생명주기

생명주기란 DI에서 주입된 객체가 언제 생성되고 재생성 될지를 특정한다. 다음과 같은 3가지 경우가 있다:

- **Transient** : 요청 받을 때마다 생성
- **Scoped** : 요청 당 한 번 생성
- **Singleton** : 처음으로 요청 받을 때에 생성. 이후의 요청들은 최초에 생성된 인스턴스를 사용.

ASP.NET Core [공식 문서](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection#service-lifetimes-and-registration-options)는 이러한 생명주기가 어떻게 작동하는지 설명하고 있지만, 만족스럽지 않다. 내가 보기에 중요한 부분의 코드를 생략하고 있으며, 실행 가능한 예제를 제공하지 않는다. 나는 이것을 완벽히 작동하는 코드로 만들어 [깃허브에 공개](https://github.com/exceptionnotfound/CoreDIDemo)했다.

이러한 생명주기들이 어떻게 작동하는지 보기 위해, 인터페이스를 몇 개 만들도록 하자. `IOperation`이란 이름의 인터페이스는 각각의 생명주기에서 수행되는 작업을 나타낸다. 이 작업은 `Guid.NewGuid()` 혹은 전달 받은 Guid값을 반환한다. 덧붙여 4개의 인터페이스를 더 정의할텐데, 이들은 각각 서로다른 생명주기 속에서 수행되는 작업을 나타낸다.

```c#
public interface IOperation
{
    Guid GetCurrentID();
}

public interface IOperationTransient : IOperation { }

public interface IOperationScoped : IOperation { }

public interface IOperationSingleton : IOperation { }

public interface IOperationSingletonInstance : IOperation { }
```

`IOperationSingletonInstance` 는 항상 공백 Guid를 반환하는 특별한 형태이다.

이제 `Operation` 클래스를 정의하자. 이 클래스는 4개의 생명주기 인터페이스를 구현하여 `GetCurrentID()`라는 메소드를 정의한다.

```c#
public class Operation : IOperationTransient, IOperationScoped, IOperationSingleton, IOperationSingletonInstance
{
    public Guid Guid { get; set; }

    public Operation()
    {
        Guid = Guid.NewGuid();
    }

    public Operation(Guid guid)
    {
        Guid = guid;
    }

    public Guid GetCurrentID()
    {
        return Guid;
    }
}
```

이 `Operation` 클래스는 `Guid.NewGuid()`를 반환하거나, 생성자를 통해 전달된 Guid를 사용한다( `IOperationSingletonInstance`의 경우).

그런데 Transient 작업이 어떻게 동작하는지 뚜렷하게 보여주려면 여전히 또 다른 클래스가 필요하다. `OperationService`라는 이름의 이 클래스는  앞선 생명주기 인터페이스들을 구현한 인스턴스 집합을 나타내지만, 컨트롤러에 주입되는 인스턴스들과는 다르다.

```c#
public class OperationService
{
    public IOperationTransient TransientOperation { get; }
    public IOperationScoped ScopedOperation { get; }
    public IOperationSingleton SingletonOperation { get; }
    public IOperationSingletonInstance SingletonInstanceOperation { get; }

    public OperationService(IOperationTransient transientOperation,
        IOperationScoped scopedOperation,
        IOperationSingleton singletonOperation,
        IOperationSingletonInstance instanceOperation)
    {
        TransientOperation = transientOperation;
        ScopedOperation = scopedOperation;
        SingletonOperation = singletonOperation;
        SingletonInstanceOperation = instanceOperation;
    }
}
```

이제 4개의 생명주기 인터페이스와 `OperationService` 클래스를 DI 컨테이너에 등록해야 한다. 좀 전에 `IUserRepository` 클래스를 등록할 때처럼 Startup 클래스를 수정하자.

```c#
public void ConfigureServices(IServiceCollection services)
{
    // Add framework services.
    services.AddMvc();

    //Basic demo
    services.AddTransient<IUserRepository, UserRepository>();

    //Lifetimes demo
    services.AddTransient<IOperationTransient, Operation>();
    services.AddScoped<IOperationScoped, Operation>();
    services.AddSingleton<IOperationSingleton, Operation>();
    services.AddSingleton<IOperationSingletonInstance>(new Operation(Guid.Empty));
    services.AddTransient<OperationService, OperationService>();
}
```

모든 의존성이 등록됐기 때문에, 이제 생명주기들이 어떻게 작동하는지 보여주는 `OperationsController`를 만들 수 있다.

```c#
public class OperationsController : Controller
{
    private readonly OperationService _operationService;
    private readonly IOperationTransient _transientOperation;
    private readonly IOperationScoped _scopedOperation;
    private readonly IOperationSingleton _singletonOperation;
    private readonly IOperationSingletonInstance _singletonInstanceOperation;

    public OperationsController(OperationService operationService,
        IOperationTransient transientOperation,
        IOperationScoped scopedOperation,
        IOperationSingleton singletonOperation,
        IOperationSingletonInstance singletonInstanceOperation)
    {
        _operationService = operationService;
        _transientOperation = transientOperation;
        _scopedOperation = scopedOperation;
        _singletonOperation = singletonOperation;
        _singletonInstanceOperation = singletonInstanceOperation;
    }

    public IActionResult Index()
    {
        // viewbag contains controller-requested services
        ViewBag.Transient = _transientOperation;
        ViewBag.Scoped = _scopedOperation;
        ViewBag.Singleton = _singletonOperation;
        ViewBag.SingletonInstance = _singletonInstanceOperation;

        // operation service has its own requested services
        ViewBag.Service = _operationService;
        return View();
    }
}

```

For posterity, `Index()` 액션의 결과를 표시하는 .cshtml 파일을 추가할 것이다.

```csharp
@{
    ViewData["Title"] = "ASP.NET Core Dependency Injection Demo";
}

<h2>Lifetimes</h2>

<span>Controller Operations</span>
<ul>
    <li><strong>Transient:</strong> @ViewBag.Transient.GetCurrentID()</li>
    <li><strong>Scoped:</strong> @ViewBag.Scoped.GetCurrentID()</li>
    <li><strong>Singleton:</strong> @ViewBag.Singleton.GetCurrentID()</li>
    <li><strong>Singleton Instance:</strong> @ViewBag.SingletonInstance.GetCurrentID()</li>
</ul>
<span>OperationService Operations</span>
<ul>
    <li><strong>Transient:</strong> @ViewBag.Service.TransientOperation.GetCurrentID()</li>
    <li><strong>Scoped:</strong> @ViewBag.Service.ScopedOperation.GetCurrentID()</li>
    <li><strong>Singleton:</strong> @ViewBag.Service.SingletonOperation.GetCurrentID()</li>
    <li><strong>Singleton Instance:</strong> @ViewBag.Service.SingletonInstanceOperation.GetCurrentID()</li>
</ul>
```

## 생명주기 테스트

모든 코드가 작성됐으니, 이제 테스트할 준비가 됐다. 데모 페이지의 스크린샷 2개를 살펴보자. 첫번째 스크린샷은 처음으로 페이지를 불렀을 때의 모습이다.

![img](https://exceptionnotfound.net/content/images/2016/07/di-lifetimes-demo-first.png)

두번째 스크린샷은 동일한 페이지를 새로고침한 모습이다.

![img](https://exceptionnotfound.net/content/images/2016/07/di-lifetimes-demo-second.png)

둘 사이의 차이점을 비교하며 알아차려야 할 점은 다음과 같다.

- 새로고침할 때마다 Transient Guid는 달라진다.
- Scoped Guid 역시 새로고침 하면 값이 달라지지만, 같은 스크린샷 내의 2개의 값은 같다.
- Singleton Guid는 언제나 같다.
- Singleton Instance Guid는 (예상대로) 항상 000....이다.

실시간으로 쉽게 알아볼 수 있도록 다음과 같이 gif를 만들었다.

![A short demo showing the differences between Transient, Scoped, and Singleton services in ASP.NET Core](https://exceptionnotfound.net/content/images/2016/07/dependency-injection-example.gif)

## 요약

ASP.NET Core의 가장 훌륭한 특성 중 하나는 경량의 자체 DI 컨테이너가 포함된 것이다. 우리는 이 포스트에서 컨트롤러에 간단한 서비스를 어떻게 주입하는지, 3가지 DI 생명주기 (Transient, Scoped, Singleton)가 무엇인지, 그리고 각각 어떤 식으로 서로 다르게 행동하는지 살펴보았다.

언제든 [깃허브](https://github.com/exceptionnotfound/CoreDIDemo)에 공개된 코드 베이스를 확인해 보고, 개선점이 있다면 알려주길 바란다.

Happy Coding!