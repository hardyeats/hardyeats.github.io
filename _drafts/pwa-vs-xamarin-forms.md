# PWAs vs Xamarin.Forms

**Posted: **February 26, 2018 / **By: **Adam Pedley / **In: **[Opinion](https://xamarinhelp.com/category/opinion/)

기술은 꾸준히 발전한다는 사실 만큼은 언제나 변함이 없다. PWAs were started by Google a number of years ago and they have progressively being getting better. They allow a website to run like a native app, and this allows quicker distribution, and easier maintenance. 웹사이트가 마치 네이티브 앱처럼 돌아가고, 배포도 빨라지며, 유지보수도 편해진다. 이제 모바일 앱과 웹사이트를 따로 개발할 필요 없이 PWA 하나면 된다. 곧 모든 브라우저들이 PWA를 지원하게 되는 만큼, 2018년은 PWA가 빛을 발하는 한 해가 될 것이다.

## What Are PWAs

PWA를 도표로 나타내면 아래와 같다. PWA는 기본적으로 웹사이트이지만, 브라우저의 특정 기능을 잘 활용하여 마치 네이티브 앱처럼 작동하며, 홈 스크린에 설치할 수도 있고, 네이티브 기능을 지원하며, 인터넷 연결 없이도 사용 가능하다.

![img](https://xamarinhelp.com/wp-content/uploads/2018/02/PWAArchitecture.png)

Android and iOS provide the ability to install a PWA, and have it displayed on your home screen, for quick and easy access, just like other apps, but you don’t need an app store, or have to deal with updates as a user, which is a big advantage.

## Capabilities

PWA는 HTML5 APIPWAs are limited by what is available in the HTML5 APIs. This will change over time and changes per browser, but currently what you can do with PWAs looks something like this. I took a screenshot from [WhatWebCanDo.Today](https://whatwebcando.today/), from my Google browser. It looks different in Edge.

![img](https://xamarinhelp.com/wp-content/uploads/2018/02/HTML5Capabilities.png)

Hence, as you can see PWAs allow a lot of native abilities, and it actually would be perfectly suitable for the majority of apps. If you need access to NFC for contactless payments, Bluetooth or anything else not supported, then PWAs are a non-starter for you at the moment. For example, Banking and payment apps, are certainly not suitable for PWAs at this time.

Xamarin.Forms and native apps, have access to everything on this list, which is why they still have an advantage. Xamarin.Froms와 네이티브 앱이 여전히 ... 이유이다.

## Service Workers

While support for other aspects of PWAs has been available, Service Worker support is only just coming out. Service Workers are key to providing offline support.

The black highlighted ones are what are available as of Feb 2018. You can see that Edge and Safari currently have ServiceWorker support in their beta channels and is coming very soon. However, as of this moment, only Firefox and Chrome have support for service workers, in their stable releases, but this is all soon to change.

![img](https://xamarinhelp.com/wp-content/uploads/2018/02/ServiceWorkerSupport_Feb_2018.png)

## How Can Cross-Platform Apps Succeed?

PWAs are likely to start gaining a lot of ground, and this puts into question, should I develop a PWA or a Xamarin.Forms app? Xamarin.Forms or even native apps will succeed well in these conditions, or possible future conditions:

1. First, Xamarin.Forms needs to have a common look and feel across all platforms. Much like Flutter. Native look and feel is going out the window, and really becoming a niche requirement. Hopefully this comes about.
2. Non-browser enabled, or odd form factor devices. Think wearables, or home automation appliances.
3. Special hardware access, as described above. PWAs will always be highly sandboxed, and while areas will open up, there may always be a need to access something PWAs don’t offer.
4. In the future, AI through TensorFlow and CoreML may be areas that give native cross-platforms apps an advantage.
5. Games, using such technologies as Unity, will likely stay native and not go web based, due to performance reasons.

## 미래의 앱 시장

In 5 years time I might see the app landscape looking a little more like this. This is just speculation, but as I will explain, most apps are perfect for PWAs. Many apps in the app store, should have just been a website to begin with.

![img](https://xamarinhelp.com/wp-content/uploads/2018/02/FutureMarketShare.png)Cross-Platforms Apps are those such as Xamarin.Forms and Native Apps I am referring to apps that may be on wearables or specific form factor devices. If home automation really takes off, and many home appliance’s like a fridge and toaster all start showing a display screen, who knows, we might all want apps for them? Maybe, maybe not.

Many enterprise’s will still stick with solutions such as Xamarin.Forms or native apps for the moment, as they are still wanting to realize their investments in the technology. They are always the last to change, and this provides some inertia in keeping Xamarin.Forms relevant for a time.

Maybe existing native SDK libraries and other factors may keep the inertia of native mobile development alive for a little while, but I do except a general shift away from native mobile apps for the bulk majority.

## Summary

PWAs seems like the best approach for simple apps, and I think this is a key point. Many native apps should have been websites in the first place, but due to a flood of marketing by app development companies early on, saying customers are now all on their mobile, it seemed like every company needed an app, when it was clear they didn’t. I believe PWAs are great for many smaller businesses that just need a better mobile presence, and PWAs should help reduce the flood of template apps in various app stores. [1]

Xamarin.Forms desperately needs an identical look, across platforms solution, similar to what Flutter currently provides, as I mentioned in my [Flutter Could Be Xamarin’s Next Big Competitor](https://xamarinhelp.com/flutter-xamarins-next-big-competitor/). It is possible and can be done with Xamarin.Forms, we will just have to see if they are willing to do it.

Native apps are at best only going to be available for specific device formats, such as wearable or home automation appliances. Native apps for mobile devices, I just can’t see how it can continue, when the app you want to generate for Android and iOS will look and act almost identically, why would anyone want to build it twice?

As per a previous statement, 2018 is very much a deciding year for Xamarin.Forms, and if it can adapt to suit new requirements of the market. I just hope Microsoft don’t make the same mistakes as in the past, of using the sluggish enterprise market as an indication of what the market wants, or is going to want in the very near future.

**Additional Information**

****[1] Microsoft is going to start showing PWAs in their App Store. Initially you have to submit it there, and in the future, they will use Bing to crawl and automatically add PWAs to their App Store. I believe they are trying to preempt their disastrous mistakes and App Store numbers in the early days of the native mobile wars. I don’t know of any Google or Apple plans as of yet.

XAMARIN.FORMS MONTHLY NEWSLETTER
**JOIN 1,500+ SUBSCRIBERS**Don't miss out on updatesThe latest info from this site, Xamarin and the communityUnsubscribe at any time* * We use MailChimp, with double opt-in, and instant unsubscribe

![img](https://secure.gravatar.com/avatar/98caaf69b5d055a80aa9e56bbc06fe4c?s=100&r=g)

[Adam Pedley](https://xamarinhelp.com/author/xamarinhelp/)

[Microsoft MVP](https://mvp.microsoft.com/en-us/mvp/Adam%20%20Pedley-5002125) | Xamarin MVP | [Xamarin Certified Developer](https://university.xamarin.com/certification?q=adam.pedley@gmail.com#verify) | Xamarin Forms Developer | Melbourne, Australia