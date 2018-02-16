---
layout: post
title: 번역 - ASP.NET Core와 Akka.NET으로 장바구니 서비스 만들기
tags: [aspnet-core, akka, actor]
---

나는 최근 한 전자 상거래 업체에서 마이크로서비스 플랫폼을 구축하는 일을 하고 있다. 이 서비스는 Scala와 Akka라는 강력한 조합에 기반을 두고 있다. 특히 Akka는 웹 서비스의 속도, 확장성, 회복성에 대한 내 생각을 바꿔놓았다. Akka의 액터 모델 때문이다. 물론 거기선 Scala를 썼지만, 나는 요새 Microsoft가 .NET Core를 두고 보이는 행보에 굉장히 관심이 많다. 사실 내가 정말로 바라는 것은 .NET으로 완벽한 마이크로서비스를 만들고, 이를 Docker 컨테이너에 실어 배포하는 것이다.

여기에선 ASP.NET Core 1.0.1과 [Akka.NET](http://getakka.net/)의 비공식 알파 릴리즈를 가지고 간단한 장바구니 서비스를 만들어 볼 것이다. `16년 10월 18일 현재 Akka.NET의 컨트리뷰터들은 .NET Core의 지원을 위해 열심히 노력하고 있다. 완벽 지원이 예고된 다음 릴리즈 버전은 1.5가 될 것 같은데, 내가 지금 사용하고 있는 버전도 굉장히 안정적이긴 하지만  쓸 수 있는 기능이 제한적이다. 예를 들어 .NET Core DI 지원은 아직이지만, 다음 릴리즈에는 완벽히 지원될 거라 예상하고 있다.

**덧붙임: .NET Core 2.0과 Akka.NET 1.3(.NET Core 정식 지원)이 릴리즈 됨에 따라 내 프로젝트도 수정했다.**

여기선 장바구니 안을 확인하고, 그 속에 물건을 넣고 빼는 정도의 간단한 기능만 구현한다. It also has an internal sample product catalog, containing a few (future) gaming consoles :-). 모든 데이터는 인메모리에서 다룰 것이며, 따라서 앱이 꺼지면 모든 것이 날아간다.

ASP.NET Core는 크로스 플랫폼이기 때문에, Windows에서만 돌아가는 개발도구를 고집할 이유가 없다. 대신 커맨드 라인 콘솔과 [Visual Studio Code](https://code.visualstudio.com)를 함께 사용할텐데, 굉장히 탁월한 조합이라 생각한다. C# 확장을 설치하는 걸 잊지 말자.

>  이 솔루션의 최종 버전은 [여기](https://gitlab.com/pnieuwenhuis/newhouse-basket-service)[^역주1]에서 확인할 수 있다. 이 글에서는 각 파일에 대해 일일이 설명하지 않을 것이다.


### 솔루션 설정

| ![img](https://cdn-images-1.medium.com/max/600/1*jO5-ZUHQ5b2AEVy2YAGBjw.jpeg) |
| :--------------------------------------: |
|                기본 솔루션 구조                 |

우선 기본 디렉토리 구조에서 시작하자. 나는 `src/BasketService`에서 `dotnet new`라는 커맨드를 써서 애플리케이션을 만들었다. `global.json`라는 파일이 있는데, 여기서 솔루션 내 프로젝트들을 정의한다. 나중에 테스트 프로젝트를 붙일 때 다시 보게 될 것이다.

*src/BasketService* 안의 *project.json*라는 파일은 해당 디렉토리가 .NET Core 프로젝트라는 것을 나타내며, 다음과 같은 내용을 담고 있다.

| ![img](https://cdn-images-1.medium.com/max/800/1*0ja2IH2mEEn5PxbuSwrwaQ.jpeg) |
| :--------------------------------------: |
|      src/BasketService/project.json      |

~~Akka.NET의 버전이 1.2.0-alpha1이라는 데 주목하자. Akka.NET의 비공식 버전은 아직 공식 NuGet 저장소에 등록되지 않았다. 따라서 이 솔루션의 최상위 디렉토리에 `nuget.config`를 추가해야 Akka.NET을 찾을 수 있다.~~[^역주2] 또한, 콘솔 로깅 라이브러리로 [*Serilog*](https://serilog.net)를 사용할 것이다.

‘*Startup.cs*’  파일은 서비스 초기화를 담당한다. 여기서 로깅 및 DI 서비스를 초기화 할 것이며, Akka 액터 시스템도 마찬가지다. 액터 시스템은 싱글턴으로 ASP.NET Core 자체 DI 컨테이너에 추가될 것이다.

![img](https://cdn-images-1.medium.com/max/800/1*cnmnfar--nhsHjFBG3bGhQ.jpeg)

### Product domain

After this set-up, I’ll first start with the ‘product’ domain. As a good micro-service manages it’s own data, it should also have a subset of the product catalog available so it isn’t dependent on other services for this. The subset only includes data needed for the basket-service to operate and enough data for an eventual UI application which is consuming this micro-service. In this example, the product catalog is hard-coded though :-).

![img](https://cdn-images-1.medium.com/max/600/1*o9JW1Aybrxp-RnHQuJN9xg.jpeg)

src/BasketService/Products/Product.cs

First, the product domain object should be created. As you can see, it contains only some basic data about the product, but not information like a full description, product type or product properties as it is not needed for this service.

The product catalog will have an API endpoint to query the full catalog:

![img](https://cdn-images-1.medium.com/max/800/1*Bp6qACDQJa1LJCX1a2AbmA.jpeg)

src/BasketService/Products/Routes/ProductApiController.cs

The file only contains the minimal code to describe each endpoint. To keep code cluttering to a minimum, all actions will be separate classes which get injected here. The real implementation of ‘*GetAllProducts’ *will be done in an other file:

![img](https://cdn-images-1.medium.com/max/800/1*G4sLdSFjp93NinMV3dEG6w.jpeg)

src/BasketService/Products/Routes/GetAllProducts.cs

Here you can see the first invocation to an actor called **ProductsActor**, where it sends the ‘*GetAllProducts*’ message and expects to receive a list of products. The usage of `async` and `await` makes this call fully asynchronous, so the application can handle other requests while waiting for the actor to return a result. This class gets the logger but also a provider injected into the constructor. The provider will create the **ProductsActor **and return a `IActorRef` reference. The provider also sets the product catalog that this actor is being initialized with. For now, to make all injections work I’ve created a file ‘*Services.cs*’ under *src/Products *and add registrations to the DI container there. With this, I keep it nicely organized without cluttering the ‘*Startup.cs*’ with all DI registrations.

![img](https://cdn-images-1.medium.com/max/800/1*oYP8H-MijB63Gs6SjGXxQw.jpeg)

src/BasketService/Products/Services.cs

As you can see, it is an extension method on `IServiceCollection`, so in ‘*Startup.cs*’ i can add this line: `services.AddProductServices()`

Now that routing is in place, it’s time to create the **ProductsActor **to handle product related requests. The **ProductsActor **has the Akka `ReceiveActor` as it’s base class and in the constructor it receives the list of products which has to be held in memory. The actor would be a singleton, so it loaded only once during the whole lifetime of the application. The actor supports a set of messages which can be send to the actor such as the ‘*GetAllProducts*’ message as you’ve seen above. These message definitions are separated to a different file from the **ProductsActor **to prevent clutter. It is still part of the **ProductsActor **class though by marking it as `partial class`.

![img](https://cdn-images-1.medium.com/max/600/1*KhmOFAINv03T5pIcAao8Vg.png)

src/BasketService/Products/ProductsActor.Messages.cs

Here you can see that the `GetAllProducts` object doesn’t even has an implementation because no other data is needed for the message. The other message `UpdateStock` does have some properties to allow the caller to give some detailed information about the action he want’s to be performed.

![img](https://cdn-images-1.medium.com/max/600/1*FiarUSoli9R8RpHTHpsgcQ.png)

src/BasketService/Products/ProductsActor.Events.cs

The same also applies to the events that are returned to the caller by the **ProductsActor**. These are also simple POCO objects, sometimes containing extra information about the event that is performed by the actor. These events are also defined in a separate file from the actual actor implementation.

All events are inheriting from `ProductEvent` which only makes it easier for the caller to expect an event instead of a generic `object`.

The **ProductsActor **is a fairly simple actor because it does not persists events or changes behaviour (i.e. state machine). The first message to process is the `GetAllProducts` message, which can be handled directly in the `Receive`method. It returns only the in-memory list of products. Note that it is an immutable copy of the list, but with references to each product in the original list. So it is not fully immutable, because in that case, I had to clone each object… If only C# has more support for immutable cloneable structures like ‘case classes’ in Scala. Maybe ‘record types’ could help us in the future: <https://github.com/dotnet/roslyn/blob/features/records/docs/features/records.md>.

| ![img](https://cdn-images-1.medium.com/max/800/1*dN4o2GhzZ6mTq5ekALDo9A.png) |
| :--------------------------------------: |
| Implementation of **GetAllProducts **in **ProductsActor** |



The second message is more complex, because there is some business logic involved here. Therefore the whole implementation is in a separate function:

![img](https://cdn-images-1.medium.com/max/800/1*JEZk9h9Dh0V0Xtm1xoGfrw.png)

**UpdateStock **message implementation in **ProductsActor**

Here you can see that it is returning the different event object instances (`StockUpdated`, `InsuffientStock` and `ProductNotFound`), based on the result. Using this, the caller can determine what happened and perform action on that (or not).

### Basket domain

The basket implementation has the same setup as the product domain. So domain objects, messages, events and API implementation are all there and implemented in the same manner. You should check GitHub for how this was implemented.

I’d like to zoom into the implementation of adding items to the basket, because the **BasketActor **teams up with the **ProductsActor **for retrieving the needed product information.

![img](https://cdn-images-1.medium.com/max/800/1*m6smKbWlxOGwHn5Tqzb-ig.png)

Here you can see it asks the **ProductsActor **to update the stock first, by using a reference to the **ProductsActor** which it received in the constructor of the actor. It then uses the result that is returned from the **ProductsActor **to do it’s own actions and return it’s own result. I’m using a large ‘if…else’ construct here, but pattern matching would be a better idea. Luckily, this feature will be introduced in C# 6! Until then, this contructs helps to determine the result that was returned from the **ProductsActor**.

Also noteworthy is that we are returning an asynchronous operation as function result, because ‘asking’ the **ProductsActor **is also asynchronous. Therefore, in the constructor the Receive method must also be asynchronous:

Using `ReceiveAsync` and the `PipeTo(Sender)` method makes sure that the asynchronous result is send directly to the sender of the message.

![img](https://cdn-images-1.medium.com/max/600/1*riqg1hZVwLGgrtdLKjse3A.png)

Another thing to note is that when creating a new basket item, the product data is copied over from the product object, instead of adding a reference to the product in the basket item. The reason of doing this, is to prevent mixing up models from different domains. Now I have a snapshot of the product at the time it was added, and it cannot be changed when the product data is changing.

OK, so we have one basket actor with an in-memory list of it’s items, but what if there are multiple customers (which is hopefully the case on a e-commerce site)? To solve this, not all messages from the API layer are directly send to the **BasketActor**, but an intermediate actor called **BasketsActor**. This actor reads the customer identifier from the message that is sent and then forwards the message to the correct **BasketActor**.

| ![img](https://cdn-images-1.medium.com/max/800/1*O9ta5S4Hb_esEtENjdUm-g.png) |
| :--------------------------------------: |
| Implementation of the message forwarder of the BasketsActor |

It will create the actor first if it does not exist in the actor system. So the **BasketsActor **has a collection of child actors, where each actor is a basket of a customer. The child actor is identified using the customer identifier, so for example the identification of a basket of customer 12 in the actor system is: `/user/baskets/12`.

### 마치며

전체 구현이 담긴 [깃허브 레포지토리](https://github.com/pnieuwenhuis/aspnetcore_basketservice)를 꼭 방문해보기 바란다.

Scala에서 본래 Akka를 다뤄본 사용자로서, Akka.NET의 기능성이 Akka와 비교해 거의 다를 바 없다는 사실이 굉장히 기쁘다. 포팅팀에게 찬사를 보낸다. 이 글은 Akka에 대해 그야말로 맛보기 수준에 그치고 있지만, 내 목표는 ASP.NET Core에서 실행되는 기능적 서비스를 만드는 일이다. ~~Akka.NET의 ASP.NET Core 지원 작업은 아직 진행 중이다.~~

다음 단계는 Docker를 사용하여 이를 마이크로서비스 플랫폼에 배치하고 다른 마이크로서비스와 함께 이용하는 것이다. 이미 [다음](https://medium.com/trafi-tech-beat/running-net-core-on-docker-c438889eb5a)과 같은 예제를 쉽게 찾아볼 수 있다.

이번 구현을 통해 나는 회복성이 높은 고성능의 서비스를 구축하고 이를 마이크로서비스 환경에 배포하는 것이 .NET 플랫폼에서 가능하다는 것을 확인했다.

[^역주1]: ASP.NET Core 2.0 기준으로 재작성된 코드임.
[^역주2]: `18년 2월 현재 NuGet에서 검색 및 설치 가능함.