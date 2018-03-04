# PWAs vs Xamarin.Forms

Posted: February 26, 2018 By **[Adam Pedley](https://xamarinhelp.com/author/xamarinhelp/)** In [Opinion](https://xamarinhelp.com/category/opinion/)

기술이 꾸준히 발전한다는 사실 만큼은 언제나 변함이 없다. 구글이 몇 해 전부터 시작한 PWA 역시 꾸준히 발전하고 있다. 이제 웹사이트가 마치 네이티브 앱처럼 돌아가고, 배포도 빨라지며, 유지보수도 편해진다. 모바일 앱과 웹사이트를 따로 개발할 필요 없이 PWA 하나면 된다. 곧 모든 브라우저들이 PWA를 지원하게 되는 만큼, 2018년은 PWA가 빛을 발하는 한 해가 될 것이다.

## PWA란 무엇인가

PWA의 개념을 도표로 나타내면 아래와 같다. PWA는 기본적으로 웹사이트이지만, 브라우저의 특정 기능을 잘 활용하여 마치 네이티브 앱처럼 작동하며, 홈 스크린에 설치할 수도 있고, 네이티브 기능을 지원하며, 인터넷 연결 없이도 사용 가능하다.

![img](https://xamarinhelp.com/wp-content/uploads/2018/02/PWAArchitecture.png)

Android and iOS provide the ability to install a PWA, and have it displayed on your home screen, for quick and easy access, just like other apps, but you don’t need an app store, or have to deal with updates as a user, which is a big advantage.

## Capabilities

PWA는 HTML5 APIPWAs are limited by what is available in the HTML5 APIs. This will change over time and changes per browser, but currently what you can do with PWAs looks something like this. I took a screenshot from [WhatWebCanDo.Today](https://whatwebcando.today/), from my Google browser. It looks different in Edge.

![img](https://xamarinhelp.com/wp-content/uploads/2018/02/HTML5Capabilities.png)

보다시피 PWA는 굉장히 많은 네이티브 기능을 수행할 수 있기 때문에, 시중의 앱들 대부분은 PWA로 바꿔도 무방하다. 하지만 NFC나 블루투스처럼 아직 지원되지 않는 기능이 필요하다면 PWA는 이 시점에서 좋은 선택이 아니다. 예를 들어 은행 앱이나 결제 앱은 확실히 PWA에 적합하지 않다.

반면 Xamarin.Forms와 네이티브 앱은 위에 나열된 모든 기능이 가능하니, 여전히 강점을 가지고 있다고 할 수 있다.

## 서비스 워커(Service Workers)

또한 서비스 워커는 오프라인 사용을 가능케 해주는 핵심이지만, 대부분의 브라우저들이 아직 잘 지원해주지 않는다.

The black highlighted ones are what are available as of Feb 2018. You can see that Edge and Safari currently have ServiceWorker support in their beta channels and is coming very soon. However, as of this moment, only Firefox and Chrome have support for service workers, in their stable releases, but this is all soon to change.

![img](https://xamarinhelp.com/wp-content/uploads/2018/02/ServiceWorkerSupport_Feb_2018.png)

## 언제 크로스플랫폼 앱으로 개발해야 할까?

이제 PWA와 Xamarin.Forms 중 뭘로 개발해야 하는가, 라는 질문과 마주하게 된다. Xamarin.Forms나 네이티브 앱으로 개발해서 이득을 보는 경우는 다음과 같다.

1. First, Xamarin.Forms needs to have a common look and feel across all platforms. Much like Flutter. Native look and feel is going out the window, and really becoming a niche requirement. Hopefully this comes about.
2. 웨어러블이나 가정용 자동화 기기 같이, 브라우저 지원이 안되는 장치들.
3. Special hardware access, as described above. PWAs will always be highly sandboxed, and while areas will open up, there may always be a need to access something PWAs don’t offer.
4. 앞으로 텐서플로우나 CoreML을 통한 AI는 네이티브 크로스플랫폼 앱에게 유리한 영역이 될 것이다.
5. Unity 같은 기술을 사용하는 게임은 성능 이슈 때문에 웹 기반이 아니라 네이티브로 남을 가능성이 높다.

## 미래의 앱 시장

앞으로 5년 후에, 미래의 앱 시장은 다음과 같이 되지 않을까 한다. 그저 예측일 뿐이지만, 위에서 설명했듯이, 대부분의 앱들은 PWA로 개발해도 무방하다. Many apps in the app store, should have just been a website to begin with.

![img](https://xamarinhelp.com/wp-content/uploads/2018/02/FutureMarketShare.png)

Cross-Platforms Apps are those such as Xamarin.Forms and Native Apps I am referring to apps that may be on wearables or specific form factor devices. If home automation really takes off, and many home appliance’s like a fridge and toaster all start showing a display screen, who knows, we might all want apps for them? Maybe, maybe not.

Many enterprise’s will still stick with solutions such as Xamarin.Forms or native apps for the moment, as they are still wanting to realize their investments in the technology. They are always the last to change, and this provides some inertia in keeping Xamarin.Forms relevant for a time. 

아마도 기존의 네이티브 SDK 등의 이유로 네이티브 모바일 개발이 잠시 동안은 수명을 유지하겠지만, 그 이후엔 대규모의 전환이 일어날 것이다.

## 요약

PWA는 간단한 앱 개발을 위한 최선의 선택으로 보인다. Many native apps should have been websites in the first place, but due to a flood of marketing by app development companies early on, saying customers are now all on their mobile, it seemed like every company needed an app, when it was clear they didn’t. I believe PWAs are great for many smaller businesses that just need a better mobile presence, and PWAs should help reduce the flood of template apps in various app stores. [1]

Xamarin.Forms desperately needs an identical look, across platforms solution, similar to what Flutter currently provides, as I mentioned in my [Flutter Could Be Xamarin’s Next Big Competitor](https://xamarinhelp.com/flutter-xamarins-next-big-competitor/). It is possible and can be done with Xamarin.Forms, we will just have to see if they are willing to do it.

네이티브 앱은 기껏해야 웨어러블이나 가정용 자동화 기기에나 쓰일 것이다. 스마트폰이나 태블릿을 위해서 네이티브 앱을 개발하는 건 더 이상 보기 힘들게 될 것이다. 어차피 안드로이드에서나 iOS에서나 똑같이 작동 하길 바랄텐데, 뭐하러 2번 빌드를 하려고 하는가?

As per a previous statement, 2018 is very much a deciding year for Xamarin.Forms, and if it can adapt to suit new requirements of the market. I just hope Microsoft don’t make the same mistakes as in the past, of using the sluggish enterprise market as an indication of what the market wants, or is going to want in the very near future.

**Additional Information**

****[1] Microsoft is going to start showing PWAs in their App Store. Initially you have to submit it there, and in the future, they will use Bing to crawl and automatically add PWAs to their App Store. I believe they are trying to preempt their disastrous mistakes and App Store numbers in the early days of the native mobile wars. I don’t know of any Google or Apple plans as of yet.

![img](https://secure.gravatar.com/avatar/98caaf69b5d055a80aa9e56bbc06fe4c?s=100&r=g)

[Adam Pedley](https://xamarinhelp.com/author/xamarinhelp/)

[Microsoft MVP](https://mvp.microsoft.com/en-us/mvp/Adam%20%20Pedley-5002125) | Xamarin MVP | [Xamarin Certified Developer](https://university.xamarin.com/certification?q=adam.pedley@gmail.com#verify) | Xamarin Forms Developer | Melbourne, Australia