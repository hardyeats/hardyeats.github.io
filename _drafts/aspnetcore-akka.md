---
layout: post
title: 번역 - ASP.NET Core와 Akka.NET으로 장바구니 서비스 만들기
tags: [aspnet-core, akka, actor]
---

나는 요새 전자 상거래 업체에서 마이크로서비스 플랫폼을 구축하는 일을 하고 있다. 이 서비스는 Scala와 Akka라는 강력한 조합에 기반을 두고 있다. 특히 Akka는 웹 서비스의 속도, 확장성, 회복성에 대한 내 생각을 바꿔놓았다. Akka의 액터 모델 때문이다. 물론 거기선 Scala를 썼지만, 나는 요새 Microsoft가 .NET Core를 두고 보이는 행보에 굉장히 관심이 많다. 사실 내가 정말로 바라는 것은 .NET으로 완벽한 마이크로서비스를 만들고, 이를 Docker 컨테이너에 실어 배포하는 것이다.

이제 ASP.NET Core 1.0.1과 [Akka.NET](http://getakka.net/)의 비공식 알파 릴리즈를 가지고 간단한 장바구니 서비스를 만들어 볼 것이다. ~~`16년 10월 18일 현재 Akka.NET의 컨트리뷰터들은 .NET Core 지원을 위해 열심히 노력하고 있다. 완벽 지원이 예고된 다음 릴리즈 버전은 1.5가 될 것 같은데, 내가 지금 사용하고 있는 버전도 굉장히 안정적이긴 하지만  쓸 수 있는 기능이 제한적이다. 예를 들어 .NET Core DI 지원은 아직 미완성이지만, 다음 릴리즈에는 완벽히 지원될 거라 예상하고 있다.~~

**덧붙임: .NET Core 2.0과 Akka.NET 1.3(.NET Core 정식 지원)이 릴리즈 됨에 따라 내 프로젝트도 수정했다.**

여기선 장바구니 안을 확인하고, 그 속에 물건을 넣고 빼는 정도의 간단한 기능만 구현한다. 모든 데이터는 인메모리에서 다룰 것이며, 따라서 앱이 꺼지면 모든 것이 날아간다.

ASP.NET Core는 크로스 플랫폼이기 때문에, Windows에서만 돌아가는 개발도구를 고집할 이유가 없다. 대신 커맨드 라인 콘솔과 [Visual Studio Code](https://code.visualstudio.com)를 함께 사용할텐데, 굉장히 탁월한 조합이라 생각한다. C# 확장을 설치하는 걸 잊지 말자.

>  이 솔루션의 최종 버전은 [여기](https://gitlab.com/pnieuwenhuis/newhouse-basket-service)[^역주1]에서 확인할 수 있다. 이 글에서는 각 파일에 대해 일일이 설명하지 않을 것이다.


### 솔루션 설정

| ![img](https://cdn-images-1.medium.com/max/600/1*jO5-ZUHQ5b2AEVy2YAGBjw.jpeg) |
| :--------------------------------------: |
|                기본 솔루션 구조                 |

우선 기본 디렉토리 구조를 보자. 나는 `src/BasketService`에서 `dotnet new`라는 커맨드를 써서 애플리케이션을 만들었다. `global.json`라는 파일이 있는데, 여기서 솔루션 내 프로젝트들을 정의한다. 나중에 테스트 프로젝트를 붙일 때 다시 보게 될 것이다.

*src/BasketService* 안의 *project.json*라는 파일은 해당 디렉토리가 .NET Core 프로젝트라는 것을 알려주며, 다음과 같은 내용을 담고 있다.

| ![img](https://cdn-images-1.medium.com/max/800/1*0ja2IH2mEEn5PxbuSwrwaQ.jpeg) |
| :--------------------------------------: |
|      src/BasketService/project.json      |

~~Akka.NET의 버전이 1.2.0-alpha1이라는 데 주목하자. Akka.NET의 비공식 버전은 아직 공식 NuGet 저장소에 등록되지 않았다. 따라서 이 솔루션의 최상위 디렉토리에 `nuget.config`를 추가해야 Akka.NET을 찾을 수 있다.~~[^역주2] 또한, 콘솔 로깅 라이브러리로 [*Serilog*](https://serilog.net)를 사용할 것이다.

‘*Startup.cs*’  파일은 서비스 초기화를 담당한다. 여기서 로깅 및 DI 서비스를 초기화 할 것이며, Akka 액터 시스템도 마찬가지다. 액터 시스템은 싱글턴으로 ASP.NET Core 자체 DI 컨테이너에 추가될 것이다.

![img](https://cdn-images-1.medium.com/max/800/1*cnmnfar--nhsHjFBG3bGhQ.jpeg)

### 상품(Product) 도메인

설정을 마쳤으니, 이제 '상품(product)' 도메인을 시작하자. 좋은 마이크로서비스라면 자신의 데이터를 직접 관리해야 하며, 이때문에 다른 서비스에 의존하지 않아야 한다. 장바구니 서비스가 사용할 상품 카탈로그의 하위 집합을 정의하자. 여기에는 장바구니 서비스가 돌아가는 데 필요한 데이터, 그리고 이 서비스를 소비하는 최종 UI 애플리케이션을 위한 데이터만 포함한다. 물론 이 예제에서는 하드 코딩된 상품 카탈로그를 사용한다. ^^; 

| ![img](https://cdn-images-1.medium.com/max/600/1*o9JW1Aybrxp-RnHQuJN9xg.jpeg) |
| :--------------------------------------: |
|  src/BasketService/Products/Product.cs   |

우선 상품 도메인 객체를 만들어야 한다. 보다시피 상품에 대한 기본적인 데이터만 포함되어 있다. 상세한 설명, 상품 유형, 상품 속성과 같은 정보는 이 서비스에 필요하지 않으므로 포함되어 있지 않다.

이제 전체 상품 카탈로그를 조회할 수 있는 API 끝점을 만들어야 한다.

| ![img](https://cdn-images-1.medium.com/max/800/1*Bp6qACDQJa1LJCX1a2AbmA.jpeg) |
| :--------------------------------------: |
| src/BasketService/Products/Routes/ProductApiController.cs |

이 파일은 끝점에 대한 최소한의 코드만을 담고 있다. 코드가 복잡해지는 걸 피하기 위해, 모든 액션은 별도 클래스로 작성 후 여기에 주입할 것이다. `*GetAllProducts’*의 실제 구현은 다른 파일에서 이뤄진다.

| ![img](https://cdn-images-1.medium.com/max/800/1*G4sLdSFjp93NinMV3dEG6w.jpeg) |
| :--------------------------------------: |
| src/BasketService/Products/Routes/GetAllProducts.cs |

여기서 액터를 처음으로 호출하게 된다. **ProductsActor**라는 이름의 액터는 ‘*GetAllProducts*’라는 메시지를 보낸 후 제품의 리스트를 받는 것을 기다린다. `async`와 `await`을 쓰면 이 호출은 완전히 비동기적이 된다. 액터가 결과를 기다리는 동안 앱이 다른 요청들을 처리할 수 있다는 말이다. 이 클래스는 로거 뿐만 아니라 한 프로바이더를 생성자를 통해 주입 받았다. 이 프로바이더는 상품 카탈로그와 **ProductsActor **를 생성한 후 `IActorRef` 참조를 반환할 것이다. 이제 *src/Products*에 ‘*Services.cs*’를 만들어 거기에서 DI 등록 작업을 한다. 이런 테크닉을 사용하면, DI 등록을 ‘*Startup.cs*’에서만 할 때보다 코드를 더 깔끔하게 정돈할 수 있다.

| ![img](https://cdn-images-1.medium.com/max/800/1*oYP8H-MijB63Gs6SjGXxQw.jpeg) |
| :--------------------------------------: |
|  src/BasketService/Products/Services.cs  |

보다시피 `IServiceCollection`의 확장 메서드이기 때문에 ‘*Startup.cs*’에서  `services.AddProductServices()`라고 추가할 수 있다.

라우팅 설정이 끝났으니, 상품과 관련된 요청을 처리할 **ProductsActor**를 만들 차례다. **ProductsActor**는 Akka의 `ReceiveActor`를 상속하고 있으며, 생성자로부터 메모리에 있는 상품의 리스트를 받아 온다. 이 액터는 싱글턴이므로 애플리케이션의 전체 생명주기 내내 단 한 번 로드된다. 그리고 앞서 본 ‘*GetAllProducts*’와 같은 액터에게 보낼 수 있는 메시지들을 지원한다. 코드가 난잡해지는 것을 피하기 위해 이러한 메시지들을 정의하는 일은 **ProductsActor**와는 다른 파일에서 수행한다. 이 파일 역시 **ProductsActor** 클래스의 일부분이기 때문에 `partial class`로 만든다.

| ![img](https://cdn-images-1.medium.com/max/600/1*KhmOFAINv03T5pIcAao8Vg.png) |
| :--------------------------------------: |
| src/BasketService/Products/ProductsActor.Messages.cs |

보다시피 `GetAllProducts` 객체는 구현이 없다. 이 메시지에 대해서 다른 정보가 필요 없기 때문이다. 다른 메시지인 `UpdateStock`은 호출자가 원하는 작업이 무엇인지 규정하는 몇 가지 프로퍼티를 가지고 있다.

| ![img](https://cdn-images-1.medium.com/max/600/1*FiarUSoli9R8RpHTHpsgcQ.png) |
| :--------------------------------------: |
| src/BasketService/Products/ProductsActor.Events.cs |

호출자에게 반환될 이벤트들을 정의할 때도 비슷하다. 이 이벤트는 단순한 POCO 객체일 수도 있고, 액터가 수행하는 이벤트에 대한 추가 정보가 들어있는 경우도 있다. 이러한 이벤트들 역시 별도의 파일에서 정의된다.

모든 이벤트들이 `ProductEvent`를 상속하고 있는 것은, 단지 호출자가 무슨 종류의 이벤트인지 파악하기 쉽게 만들기 위함이다.

**ProductsActor**는 굉장히 단순한 액터라 볼 수 있다. 이벤트를 지속하거나 행동을 바꾸지 않기 때문이다. 첫 번째로 처리해야 할 메시지가 `GetAllProducts` 메시지인데, 곧바로 `Receive` 메소드로 처리할 수 있다. 이는 오로지 메모리에 있는 상품의 목록을 반환할 뿐이다. 그런데 이 목록은 원래의 목록을 참조하고 있는 것이므로, 완전히 불변이라 볼 수 없다. 그래서 일일이 복제를 해야 했는데… Scala의 [케이스 클래스](https://docs.scala-lang.org/ko/tutorials/tour/case-classes.html.html)처럼, C#도 불변의 복제 가능한 구조체를 잘 지원해 줬으면 한다. 아마도 언젠가 [‘record types’](https://github.com/dotnet/roslyn/blob/features/records/docs/features/records.md)을 활용할 수도 있겠다.

| ![img](https://cdn-images-1.medium.com/max/800/1*dN4o2GhzZ6mTq5ekALDo9A.png) |
| :--------------------------------------: |
| **ProductsActor**에서 **GetAllProducts** 구현 |

두 번째 메시지는 좀 더 복잡하다. 비지니스 로직이 약간 들어가 있기 때문이다. 따라서 별도의 함수로 구현한다.

| ![img](https://cdn-images-1.medium.com/max/800/1*JEZk9h9Dh0V0Xtm1xoGfrw.png) |
| :--------------------------------------: |
| **ProductsActor**에서 **UpdateStock **메시지 구현 |

보다시피 결과에 따라 다른 이벤트 객체 인스턴스(`StockUpdated`, `InsuffientStock`, `ProductNotFound`)를 반환한다. 호출자는 이를 통해 무슨 일이 벌어졌는지 알 수 있고, 무슨 행동을 취해야 할지 혹은 아무 행동도 하지 않아야 할지를 결정할 수 있다.

### 장바구니(Basket) 도메인

장바구니 도메인 역시 상품 도메인과 같은 설정을 따른다. 따라서 도메인 객체, 메시지, 이벤트, API 구현을 모두 동일한 방식으로 하면 된다. 실제 구현 내용은 깃허브에서 확인해 보자.

여기선 장바구니에 상품 추가하는 것을 어떻게 구현했는지만 살펴보자. 필요한 상품 정보를 찾기 위해 **BasketActor**가  **ProductsActor **와 협력하는 모습을 볼 수 있다.

![img](https://cdn-images-1.medium.com/max/800/1*m6smKbWlxOGwHn5Tqzb-ig.png)

액터[^역주3]는 생성자를 통해 **ProductsActor**의 참조를 전달 받는다. 그리고 이를 통해  **ProductsActor**에게 우선 재고 수량을 갱신하라고 요청한다. 그리고  **ProductsActor**가 반환하는 결과에 따라 스스로 행동을 취한 뒤 또 한 번 자신의 결과를 반환한다. 여기서 장황한 If...else 구조를 썼는데, 패턴 매칭을 쓰는 게 더 현명할 것이다. 패턴 매팅은 C# 6에서 도입된다고 한다![^역주4] 그 전까진, 이 구조가  **ProductsActor**가 반환하는 결과를 지정하는 데 도움을 줄 것이다.

함수의 결과로 비동기 작업[^역주5]을 반환한다는 점을 눈여겨 보자. **ProductsActor**를 '물어 보는 일'이 비동기적이기 때문이다. 따라서 생성자 안의 Receive 메서드도 비동기적이어야 한다.

| ![img](https://cdn-images-1.medium.com/max/800/1*HpVaveWA-gnZ0zrMbbcvTg.png) |
| :--------------------------------------: |
|          **BasketActor**의 생성자 내          |

`ReceiveAsync`와 `PipeTo(Sender)` 메서드를 사용하면 비동기 결과가 메시지를 보낸 주체에게 직접 보내진다.

![img](https://cdn-images-1.medium.com/max/600/1*riqg1hZVwLGgrtdLKjse3A.png)

Another thing to note is that when creating a new basket item, the product data is copied over from the product object, instead of adding a reference to the product in the basket item. The reason of doing this, is to prevent mixing up models from different domains. Now I have a snapshot of the product at the time it was added, and it cannot be changed when the product data is changing.

그런데 만약 고객이 여러 명이 되면 어떡할까? API 계층에서 보내는 메시지를 전부 **BasketActor**로 보내면 직접 안 된다. 대신 **BasketsActor**라는 이름의 중계 액터를 만들어야 한다. 이 액터는 메시지로부터 고객 식별자를 읽은 다음에 적절한 **BasketActor**를 찾아 이 메시지를 전달한다.

| ![img](https://cdn-images-1.medium.com/max/800/1*O9ta5S4Hb_esEtENjdUm-g.png) |
| :--------------------------------------: |
|       **BasketsActor**의 메시지 전달자 구현       |

액터 시스템 내에 액터가 없으면 액터를 먼저 생성한다. 따라서 **BasketsActor**는 자식 액터들의 컬렉션이라 볼 수 있으며, 각각의 액터는 고객이다. 자식 액터는 고객 식별자를 통해 구별한다. 예를 들어 12번 고객의 장바구니는 `/user/baskets/12`로 찾아갈 수 있다.

### 마치며

전체 구현이 담긴 [깃허브 레포지토리](https://github.com/pnieuwenhuis/aspnetcore_basketservice)[^역주6]를 꼭 방문해보기 바란다.

Scala에서 본래 Akka를 다뤄본 사용자로서, Akka.NET의 기능성이 Akka와 비교해 거의 다를 바 없다는 사실이 굉장히 기쁘다. 포팅 팀에게 찬사를 보낸다. 이 글은 Akka를 맛보기 수준으로 다루는데 그치고 있지만, 내 목표는 ASP.NET Core에서 실행되는 기능적 서비스를 만드는 일이다. ~~Akka.NET의 ASP.NET Core 지원 작업은 아직 진행 중이다.~~

다음 단계는 Docker를 사용하여 이를 마이크로서비스 플랫폼에 배치하고 다른 마이크로서비스와 함께 이용하는 것이다. 이미 [다음](https://medium.com/trafi-tech-beat/running-net-core-on-docker-c438889eb5a)과 같은 예제를 쉽게 찾아볼 수 있다.

이번 구현을 통해 나는 회복성이 높은 고성능의 서비스를 구축하고 이를 마이크로서비스 환경에 배포하는 것이 .NET 플랫폼에서 가능하다는 것을 확인했다.

##참조

[Building a basket micro-service using ASP.NET Core and Akka.NET](https://medium.com/@FurryMogwai/building-a-basket-micro-service-using-asp-net-core-and-akka-net-ea2a32ca59d5)

[^역주1]: ASP.NET Core 2.0 기준으로 재작성된 코드임.
[^역주2]: `18년 2월 현재 NuGet에서 검색 및 설치 가능함.
[^역주3]: 장바구니 액터(BasketActor)를 가리킴.
[^역주4]: ASP.NET Core 2.0부터는 C# 7.1을 지원함.
[^역주5]: Task<T>
[^역주6]: ASP.NET Core 1.0.x를 기준으로 작성된 코드임.