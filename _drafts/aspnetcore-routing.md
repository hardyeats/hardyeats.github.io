

# ASP.NET Core Demystified - Routing in MVC

ASP.NET Core MVC는 초보 웹 개발자가 생소해 할 수 있는 개념들을 상당수 도입했다. 나는 이런 개발자들이 본격적인 ASP.NET Core 애플리케이션을 만드는데 도움을 주기 위해  ASP.NET Core Demystified 시리즈를 연재하고 있다.

이 글에서는 라우팅의 개념에 대해 살펴보고, 그것을 이용해 URL과 액션을 매치시키는 법을 배울 것이다. 항상 그랬듯이 [깃허브](https://github.com/exceptionnotfound/AspNetCoreRoutingDemo)에 샘플 프로젝트를 공개해 뒀으므로, 코드를 참조해가면서 ASP.NET Core의 라우팅을 공부하길 바란다.

![A superposition of helicopter routes over New York City.](https://exceptionnotfound.net/content/images/2017/06/helicopter-routes.png)

## 기초

*라우팅(Routing)*이란 ASP.NET Core에서 URL을 가지고 컨트롤러 액션이나 파일 등에 매핑시키는 시스템을 가리키는 일반적인 용어다. 이 시스템은 *라우트 핸들러(Route Handler)*라고 부르는 클래스들을 사용하며, 이들은 실제로 매핑을 수행한다. Most of the time, you won't be creating routes at such a low level (e.g. creating route handlers),. rather you will be defining routes and telling ASP.NET Core where those routes map to.

라우트를 정의하는 방법은 주로 2가지다.

- **컨벤션 기반 라우팅**은 ASP.NET Core Startup.cs 파일에 정의된 일련의 규칙에 따라 라우트를 만든다. 
- **애트리뷰트 라우팅**은 컨트롤러 액션에 사용된 애트리뷰트에 기반하여 라우트를 만든다.

이 2가지 방법은 함께 쓰일 수도 있다. 우선 컨벤션 기반 라우팅을 살펴보자.

## 컨벤션 기반 라우팅

In convention-based routing, you define a series of route conventions that are meant to represent all the possible routes in your system. 이러한 정의들은 ASP.NET Core 프로젝트 내의 Startup.cs 파일에 위치한다.

 For example, let's look what might be the simplest possible convention-based route for an ASP.NET Core MVC application:

```c#
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
{
    ...
    app.UseMvc(routes =>
    {
        routes.MapRoute(
            template: "{controller}/{action}");
    });
}

```

This route expects, and will map, URLs like the following:

- <http://yoursite.com/home/index>
- <http://yoursite.com/blogs/all>
- <http://yoursite.com/recipes/index>

However, what if we wanted to have more specific routes? Say, like these:

- <http://yoursite.com/blogpost/1/5>
- <http://yoursite.com/home/index/17>

In order to get our actions to match those URLs, we need to add a few features to our convention-based routes.

#### 라우트 제약 조건(Route Constraints)

Let's say we want to match the following URL:

- <http://yoursite.com/home/index/17>

One simple way of doing this might be to define the following route:

```
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
{
    ...
    app.UseMvc(routes =>
    {
        routes.MapRoute(
            template: "{controller}/{action}/{id}");
    });
}

```

This will, in fact, match the desired route, but it will also match the following:

- <http://yoursite.com/home/index/all>
- <http://yoursite.com/recipes/index/first>
- <http://yoursite.com/samples/costco/many>

This probably isn't what we want. In the above URLs, the values "all", "first", and "many" will be mapped to an action's `id` parameter.

만약 `id`가 정수여야만 한다면, 다음과 같이 *라우트 제약 조건*을 걸 수 있다.

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
{
    ...
    app.UseMvc(routes =>
    {
        routes.MapRoute(
            template: "{controller}/{action}/{id:int}");
    });
}

```

`{id:int}`는 URL의 해당 부분이 반드시 정수여야 함을 명시하며, 그렇지 않다면 URL은 이 라우트와 매핑되지 않는다. 

개발자가 정할 수 있는 라우트 제약 조건은 셀 수 없이 많다.

- `:int`
- `:bool`
- `:datetime`
- `:decimal`
- `:guid`
- `:length(min,max)`
- `:alpha`
- `:range(min,max)`

그 밖에 모든 조건을 알아보려면 [ASP.NET Core 라우팅 문서](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-constraint-reference)를 참조하자.

#### Optional Parameters

It's also possible that we want to map the following URLs to the same action:

- <http://yoursite.com/blog/posts>
- <http://yoursite.com/blog/posts/4>

To do this, we can use *optional parameters* in our convention-based routes by adding a `?` to the optional parameter's constraint, like so:

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
{
    ...
    app.UseMvc(routes =>
    {
        routes.MapRoute(
            template: "{controller}/{action}/{id:int?}");
    });
}

```

NOTE: You may only have one optional parameter per route, and that optional parameter must be the last parameter.

#### 기본값

In addition to route constraints and optional parameters, we can also specify what happens if parts of the route are not provided. Say we encounter this URL:

- <http://yoursite.com/>

We want to map this URL to the controller `home` and action `index`, so we introduce default route values by defining the following route:

```
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
{
    ...
    app.UseMvc(routes =>
    {
        routes.MapRoute(
            template: "{controller=Home}/{action=Index}/{id:int?}");
    });
}

```

This will now properly route the URL to our Home/Index action. We could also map default values by using a `defaults` property:

```
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
{
    ...
    app.UseMvc(routes =>
    {
        routes.MapRoute(
            template: "{controller}/{action}/{id:int?}"),
            defaults: new { controller = "Home", action = "Index" }
    });
}

```

#### Named Routes

We can also provide a name for any given route. Naming a route allows us to have multiple routes with similar parameters but that can be used differently. The names must be unique across the project.

```
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
{
    ...
    app.UseMvc(routes =>
    {
        routes.MapRoute(
            name: "default",
            template: "{controller}/{action}/{id:int?}"),
            defaults: new { controller = "Home", action = "Index" }
    });
}

```

#### Sample Routes

As always with my tutorials posts, I've included a sample project [over on GitHub](https://github.com/exceptionnotfound/AspNetCoreRoutingDemo). In that project, we define the following convention-based routes:

```
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
{
    ...
    app.UseMvc(routes =>
    {
        routes.MapRoute(
            name: "blogPosts",
            template: "convention/{action}/{blogID:int}/{postID:int}",
            defaults: new { controller = "Convention", action = "BlogPost" }
            );

        routes.MapRoute(
            name: "allBlogs",
            template: "convention/{action}/{blogID:int?}",
            defaults: new { controller = "Convention", action = "AllBlogsAndIndex" }
            );

        routes.MapRoute(
            name: "default",
            template: "{controller=Home}/{action=Index}/{id:int?}");
    });
}

```

## 애트리뷰트 라우팅

In contrast to convention-based routing, attribute routing allows us to define which route goes to each action by using attributes on said actions. If you are just using convention-based routing, there's no need to employ attribute routing at all. However, many people (myself included) believe that attribute routing provides a better relationship between actions and routes and more fine-grained control over routes.

Here's the simplest possible use of attribute routing:

```c#
public class HomeController : Controller
{
    [HttpGet("home/index")] //We generally want to use the more specific Http[Verb] attributes over the generic Route attribute
    public IActionResult Index()
    {
        return View();
    }
}

```

*NOTE: In ASP.NET Core MVC, the attributes [HttpGet], [HttpPost] and similar attributes can be used to assign routes.*

#### Token Replacement

One of the things ASP.NET Core MVC does to make routing a bit more flexible is provide tokens for `[area]`, `[controller]`, and `[action]`. These tokens get replaced by their action values in the route table. For example, if we have the following controller:

```
[Route("[controller]")]
public class UsersController : Controller { }

```

If there are two actions, index and view, in this controller, their routes are now:

- /users/index
- /users/view

Thus, token replacement provides a short way of including the controller, area, and action names into the route.

#### Multiple Routes

Attribute routing, because it focuses on the individual actions, allows for multiple routes to be mapped to the same action or controller. One of the most common ways of doing this is to define default routes, like so:

```
public class UsersController : Controller
{
    [HttpGet("~/")]
    [HttpGet("")]
    [HttpGet("index")]
    public IActionResult Index()
    {
        return View();
    }
}

```

The three `[HttpGet]` attributes define that the action matches the following routes:

- `[HttpGet("index")]` matches <http://yoursite.com/users/index>.
- `[HttpGet("")]` matches <http://yoursite.com/users> (AKA the controller default action).
- `[HttpGet("~/")]` matches <http://yoursite.com/> (AKA the application's default action).

You can define as many routes as you like for each action or controller.

## Mixed Routing

It is perfectly valid to use convention-based routing for some controllers and actions and attribute routing for others. However, ASP.NET Core MVC does not allow for convention-based routes and attribute routing to exist on the same action. If an action uses attribute routing, no convention-based routes can map to that action. See the [ASP.NET Core docs](https://docs.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#mixed-routing-attribute-routing-vs-conventional-routing) for more info.

## Opinion

If you don't need my blithe little opinion on routing, skip to the next section.

Are they gone? Good.

Attribute route all the things! Seriously, attribute routing is much more powerful than convention-based routing. Convention-based routing is good for static files and the like, but if you need to map an action to a route, use attribute routing!

## 요약

ASP.NET Core MVC에서 라우팅이란 URL을 컨트롤러 액션이나 기타 리소스에 매핑시키는 시스템이다. There are two primary methods of creating routes: convention-based routing, which defines a few routes in the Startup.cs file, and attribute routing, which defines a route per action. You can mix the two styles, but if an action has an attribute route, it cannot be reached by using a convention-based route.

As always, there's a sample project [over on GitHub](https://github.com/exceptionnotfound/AspNetCoreRoutingDemo) that demonstrates many of the routing features talked about in this post. Check it out! Also, please feel free to share in the comments anything useful from the Routing features of ASP.NET Core that I didn't already cover.

This post is meant to be an introductory-level demo to the concept of Routing, while eliminating the cruft in the official Microsoft documentation. It's meant to show the most common uses, not all of them. That said, if you think I missed something that should be here, please let me know in the comments!

Happy Coding!