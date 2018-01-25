---
layout: post
title:  "Dagger 2를 이용한 의존성 주입"
---


[원문링크](https://github.com/codepath/android_guides/wiki/Dependency-Injection-with-Dagger-2)

## 개요

대부분의 안드로이드 앱들은 다른 의존성을 필요로 하는 객체를 인스턴스화 한다. 예를 들어, 트위터 API 클라이언트를 만들 때 [Retrofit](https://github.com/codepath/android_guides/wiki/Consuming-APIs-with-Retrofit) 같은 네트워킹 라이브러리를 활용할 수 있다. 이 라이브러리를 사용하려면, [Gson](https://github.com/codepath/android_guides/wiki/Leveraging-the-Gson-Library) 같은 파싱 라이브러리를 추가해야 할지도 모른다.또한, 인증이나 캐싱을 구현하는 클래스들은 [shared preferences](https://github.com/codepath/android_guides/wiki/Storing-and-Accessing-SharedPreferences) 같은 공용 저장소에 접근하는 게 필요할 수 있으므로, 우선 이들을 인스턴스화 한 뒤 내재된 의존성 체인을 만들어야 한다.

의존성 주입(Dependency Injection)에 익숙하지 않다면, [이 영상](https://www.youtube.com/watch?v=IKD2-MAkXyQ)을 보고 오자.

Dagger 2는 이러한 의존성들을 분석해 서로를 연결해주는 코드를 생성한다. 다른 의존성 프레임워크들은 XML에 의존하는 한계에 시달리거나, 런타임에 의존성의 유효성을 확인해야 하거나, 구동 시 성능 저하를 겪어야 했다. [Dagger 2](http://google.github.io/dagger/)는 오로지 자바 [annotation processors](https://www.youtube.com/watch?v=dOcs-NKK-RA)에 의존하며, 컴파일 타임에 의존성을 분석하고 유효한지 검사한다. 때문에 지금까지 개발된  가장 효율적인 의존성 주입 프레임워크 중 하나로 간주된다.

### 장점

Dagger 2의 또 다른 장점들을 알아보자.

- **공유 인스턴스에 대한 접근이 간편해진다**. [ButterKnife](https://github.com/codepath/android_guides/wiki/Reducing-View-Boilerplate-with-Butterknife)가 View 객체, 이벤트 핸들러, 리소스에 대한 참조를 정의하는 걸 손쉽게 만들어주듯이, Dagger 2는 공유 인스턴스에 대한 참조 획득을 간편하게 만들어준다. 예를 들어, 일단 Dagger에서 `MyTwitterApiClient`나  `SharedPreferences` 같은 싱글턴 인스턴스를 선언해두면,  다음과 같이 `@Inject` 어노테이션을 사용해 간편하게 필드를 선언할 수 있다.

```java
public class MainActivity extends Activity {
   @Inject MyTwitterApiClient mTwitterApiClient;
   @Inject SharedPreferences sharedPreferences;

   public void onCreate(Bundle savedInstance) {
       // 싱글턴 인스턴스를 필드에 할당
       InjectorClass.inject(this);
   } 
```

- **Easy configuration of complex dependencies**. There is an implicit order in which your objects are often created. Dagger 2 walks through the dependency graph and [generates code](https://github.com/codepath/android_guides/wiki/Dependency-Injection-with-Dagger-2#code-generation) that is both easy to understand and trace, while also saving you from writing the large amount of boilerplate code you would normally need to write by hand to obtain references and pass them to other objects as dependencies. 또한 리팩토링을 단순화하는 데 도움이 된다. 왜냐하면 모듈을 생성하는 순서 보다는 어떤 모듈을 만들어야 하느냐에 집중할 수 있기 때문이다.
- **단위 테스트, 통합 테스트가 더 쉬워진다.** Because the dependency graph is created for us, we can easily swap out modules that make network responses and mock out this behavior.
- **인스턴스의 범위(Scope)를 지정할 수 있다** Dagger 2를 활용하면 애플리케이션의 전체 생명주기 내내 지속되는 인스턴스를 쉽게 관리할 수 있을뿐만 아니라, 수명이 짧은 인스턴스를 정의할 수 있다(예를 들어 액티비티 생명주기나 사용자 세션).

### Setup

Android Studio by default will not allow you to navigate to generated Dagger 2 code as legitimate classes because they are not normally added to the source path. Adding the `annotationProcessor`plugin will add these files into the IDE classpath and enable you to have more visibility.

Make sure to [upgrade](https://github.com/codepath/android_guides/wiki/Getting-Started-with-Gradle#upgrading-gradle) to the latest Gradle version to use the `annotationProcessor` syntax:

```groovy
dependencies {
  compile 'com.google.dagger:dagger-android:2.11'
  compile 'com.google.dagger:dagger-android-support:2.11' // if you use the support libraries
  annotationProcessor 'com.google.dagger:dagger-android-processor:2.11'
  annotationProcessor 'com.google.dagger:dagger-compiler:2.11'
}
```

Note that the `provided` keyword refers to dependencies that are only needed at compilation. The Dagger compiler generates code that is used to create the dependency graph of the classes defined in your source code. These classes are added to the IDE class path during compilation. The `annotationProcessor` keyword, which is understood by the Android Gradle plugin, does not add these classes to the class path, they are used only for annotation processing, which prevents accidentally referencing them.

### 싱글턴 만들기

![Dagger Injections Overview](https://raw.githubusercontent.com/codepath/android_guides/master/images/dagger_general.png)

다음은 Dagger 2를 이용해 모든 싱글턴 생성을 중앙집중화 하는 간단한 예제다. 우선 아무런 의존성 주입 프레임워크를 쓰지 않고 트위터 클라이언트 코드를 다음과 같이 작성했다고 가정하자.

```java
OkHttpClient client = new OkHttpClient();

// OkHttp의 캐싱을 허용
int cacheSize = 10 * 1024 * 1024; // 10 MiB
Cache cache = new Cache(getApplication().getCacheDir(), cacheSize);
client.setCache(cache);

// Used for caching authentication tokens
SharedPreferences sharedPreferences = PreferenceManager.getDefaultSharedPreferences(this);

// Instantiate Gson
Gson gson = new GsonBuilder().create();
GsonConverterFactory converterFactory = GsonConverterFactory.create(gson);

// Build Retrofit
Retrofit retrofit = new Retrofit.Builder()
                                .baseUrl("https://api.github.com")
                                .addConverterFactory(converterFactory)
                                .client(client)  // custom client
                                .build();
```

#### Declare your singletons

어떤 객체가 의존성 체인에 포함돼야 하는지 정해야 한다면 Dagger 2 **모듈(module)**을 만들어야 한다. 예를 들어,  애플리케이션 생명주기와 연동되며 모든 액티비티와 프래그먼트에서 사용할 수 있는 단일 `Retrofit` 인스턴스를 만들고자 한다면, 우선 `Retrofit` 인스턴스가 제공될 수 있다는 걸 Dagger에게 알려줘야 한다.

캐싱을 설정하려면 Application context가 필요하다. 우리가 만들 첫번째 Dagger 모듈인 `AppModule.java`는 이에 대한 참조를 제공하는데 쓰인다. 이제 `@Provides` 어노테이션을 사용해 메소드를 만들어 Dagger에게 이 메소드가 `Application` 반환형의 생성자(즉, Application 클래스의 인스턴스를 제공하는 메소드)라는 것을 알려줄 것이다. 

```java
@Module
public class AppModule {

    Application mApplication;

    public AppModule(Application application) {
        mApplication = application;
    }

    @Provides
    @Singleton
    Application providesApplication() {
        return mApplication;
    }
}
```

`NetModule.java`라는 클래스를 만들고 `@Module`이라는 어노테이션을 사용함으로써, Dagger가 적절한 인스턴스를 제공하는 메소드를 찾게끔 만든다.

The methods ( that will actually expose available return types ) should also be annotated with the `@Provides` annotation. `@Singleton` 어노테이션은 이 인스턴스가 애플리케이션이 실행되는 내내 오직 한 번 생성되어야 한다는 것을 Dagger에게 알려준다. In the following example, we are specifying `SharedPreferences`, `Gson`, `Cache`, `OkHttpClient`, and `Retrofit` as the return types that can be used as part of the dependency list.

```java
@Module
public class NetModule {

    String mBaseUrl;
    
    // Constructor needs one parameter to instantiate.  
    public NetModule(String baseUrl) {
        this.mBaseUrl = baseUrl;
    }

    // Dagger는 오로지 @Provides 어노테이션을 사용한 메소드들만 찾는다.
    @Provides
    @Singleton
    // Application reference must come from AppModule.class
    SharedPreferences providesSharedPreferences(Application application) {
        return PreferenceManager.getDefaultSharedPreferences(application);
    }

    @Provides
    @Singleton
    Cache provideOkHttpCache(Application application) { 
        int cacheSize = 10 * 1024 * 1024; // 10 MiB
        Cache cache = new Cache(application.getCacheDir(), cacheSize);
        return cache;
    }

   @Provides 
   @Singleton
   Gson provideGson() {  
       GsonBuilder gsonBuilder = new GsonBuilder();
       gsonBuilder.setFieldNamingPolicy(FieldNamingPolicy.LOWER_CASE_WITH_UNDERSCORES);
       return gsonBuilder.create();
   }

   @Provides
   @Singleton
   OkHttpClient provideOkHttpClient(Cache cache) {
      OkHttpClient client = new OkHttpClient.Builder();
      client.cache(cache);
      return client.build();
   }

   @Provides
   @Singleton
   Retrofit provideRetrofit(Gson gson, OkHttpClient okHttpClient) {
      Retrofit retrofit = new Retrofit.Builder()
                .addConverterFactory(GsonConverterFactory.create(gson))
                .baseUrl(mBaseUrl)
                .client(okHttpClient)
                .build();
        return retrofit;
    }
}
```

사실 메소드의 이름(`provideGson()`, `provideRetrofit()` 등등)은 중요하지 않으며 아무렇게나 지어도 된다. The return type annotated with a `@Provides` annotation is used to associate this instantiation with any other modules of the same type. `@Singleton` 어노테이션은 애플리케이션 전체 생명주기를 통틀어 단 한 번 초기화될 것이라는 것을 Dagger에게 선언하기 위해 사용한다.

`Retrofit` 인스턴스는 `Gson` 인스턴스와 `OkHttpClient` 인스턴스를 모두 의존하고 있기 때문에, 같은 클래스에 이 두 타입을 취하는 또다른 메서드를 정의한다. 이 메서드는 `@Provides` 어노테이션과 2개의 매개 변수를 가지고 있기 때문에, Dagger는 `Retrofit` 인스턴스를 만들 때 `Gson`과 `OkHttpClient`에 의존해야 한다는 것을 알아차리게 된다.

#### Define injection targets

Dagger provides a way for the fields in your activities, fragments, or services to be assigned references simply by annotating the fields with an `@Inject` annotation and calling an `inject()`method. Calling `inject()` will cause Dagger 2 to locate the singletons in the dependency graph to try to find a matching return type. If it finds one, it assigns the references to the respective fields. 예를 들어, 아래의 예제에서 Dagger는 `MyTwitterApiClient` 타입과 `SharedPreferences` 타입을 반환하는 provider를 찾으려고 할 것이다.

```java
public class MainActivity extends Activity {
   @Inject MyTwitterApiClient mTwitterApiClient;
   @Inject SharedPreferences sharedPreferences;

  public void onCreate(Bundle savedInstance) {
       // assign singleton instances to fields
       InjectorClass.inject(this);
   } 
```

Dagger 2에서 쓰이는 주입 클래스를 **컴포넌트(component)**라고 부른다. 컴포넌트는 액티비티, 서비스, 프래그먼트에 참조를 할당하여, 이들이 좀 전에 정의한 싱글턴에 접근할 수 있게 만들어 준다. 이제 `@Component` 어노테이션을 사용한 클래스가 필요해진다. Note that the activities, services, or fragments that are allowed to request the dependencies declared by the modules (by means of the `@Inject` annotation) should be declared in this class with individual `inject()` methods:

```java
@Singleton
@Component(modules={AppModule.class, NetModule.class})
public interface NetComponent {
   void inject(MainActivity activity);
   // void inject(MyFragment fragment);
   // void inject(MyService service);
}
```

**Note** that base classes are not sufficient as injection targets.Dagger 2 relies on strongly typed classes, so you must specify explicitly which ones should be defined. (There are [suggestions](https://blog.gouline.net/2015/05/04/dagger-2-even-sharper-less-square/) to workaround the issue, but the code to do so may be more complicated to trace than simply defining them.)

#### 코드 생성

An important aspect of Dagger 2 is that the library generates code for classes annotated with the `@Component` interface. You can use a class prefixed with `Dagger` (i.e. `DaggerTwitterApiComponent.java`) that will be responsible for instantiating an instance of our dependency graph and using it to perform the injection work for fields annotated with `@Inject`. See the [setup guide](https://github.com/codepath/android_guides/wiki/Dependency-Injection-with-Dagger-2#setup).

 `Dagger`라는 접두사가 붙은 클래스를  [설정 가이드](https://github.com/codepath/android_guides/wiki/Dependency-Injection-with-Dagger-2#setup)를 보자.

### 컴포넌트 인스턴스화 하기

앞서 말한 작업들은 모두 `Application`을 상속한 클래스 내에서 이루어져야 한다. 왜냐하면 이러한 인스턴스들은 애플리케이션의 전체 생명주기를 통틀어 오직 한 번만 선언되어야 하기 때문이다.

```java
public class MyApp extends Application {

    private NetComponent mNetComponent;

    @Override
    public void onCreate() {
        super.onCreate();
        
        // Dagger%COMPONENT_NAME%
        mNetComponent = DaggerNetComponent.builder()
                // list of modules that are part of this component need to be created here too
                .appModule(new AppModule(this)) // This also corresponds to the name of your module: %component_name%Module
                .netModule(new NetModule("https://api.github.com"))
                .build();

        // If a Dagger 2 component does not have any constructor arguments for any of its modules,
        // then we can use .create() as a shortcut instead:
        //  mNetComponent = com.codepath.dagger.components.DaggerNetComponent.create();
    }

    public NetComponent getNetComponent() {
       return mNetComponent;
    }
}
```

Dagger 컴포넌트를 참조할 수 없다면, 반드시 프로젝트를 리빌드 해야 한다(안드로이드 스튜디오에선, *Build > Rebuild Project* 선택).

Because we are extending the default `Application` class with the class `MyApp`, we have to specify `MyApp` as the application `name` in the AndroidManifest.xml in order for it to be instantiated. This way your app will launch `MyApp` to handle the initial instantiation.

```xml
<application
      android:allowBackup="true"
      android:name=".MyApp">
```

이제 액티비티에서 컴포넌트에 접근해 `inject()`를 호출하기만 하면 된다.

```java
public class MyActivity extends Activity {
  @Inject OkHttpClient mOkHttpClient;
  @Inject SharedPreferences sharedPreferences;

  public void onCreate(Bundle savedInstance) {
        // assign singleton instances to fields
        // We need to cast to `MyApp` in order to get the right method
        ((MyApp) getApplication()).getNetComponent().inject(this);
    } 
```

### Qualified types

![Dagger Qualifiers](https://raw.githubusercontent.com/codepath/android_guides/master/images/dagger_qualifiers.png)

If we need two different objects of the same return type, we can use the `@Named` qualifier annotation. You will define it both where you provide the singletons (`@Provides` annotation), and where you inject them (`@Inject` annotations):

반환형은 같지만 서로 다른 두 객체가 필요하다면, `@Named` 어노테이션을 사용하자. 싱글턴을 제공하는 곳, 두 곳에서 다 써야 한다.

```java
@Provides @Named("cached")
@Singleton
OkHttpClient provideOkHttpClient(Cache cache) {
    OkHttpClient client = new OkHttpClient();
    client.setCache(cache);
    return client;
}

@Provides @Named("non_cached") @Singleton
OkHttpClient provideOkHttpClient() {
    OkHttpClient client = new OkHttpClient();
    return client;
}
```

Injection will also require these named annotations too:

```
@Inject @Named("cached") OkHttpClient client;
@Inject @Named("non_cached") OkHttpClient client2;
```

`@Named` is a qualifier that is pre-defined by dagger, but you can create your own qualifier annotations as well:

```java
@Qualifier
@Documented
@Retention(RUNTIME)
public @interface DefaultPreferences {
}
```

### Scopes

![Dagger Scopes](https://raw.githubusercontent.com/codepath/android_guides/master/images/dagger_scopes.png)

In Dagger 2, you can define how components should be encapsulated by defining custom scopes. For instance, you can create a scope that only lasts the duration of an activity or fragment lifecycle. You can create a scope that maps only to a user authenticated session. You can define any number of custom scope annotations in your application by declaring them as a public `@interface`:

```
@Scope
@Documented
@Retention(value=RetentionPolicy.RUNTIME)
public @interface MyActivityScope
{
}
```

Even though Dagger 2 does not rely on the annotation at runtime, keeping the `RetentionPolicy`at RUNTIME is useful in allowing you to inspect your modules later.

### Dependent Components vs. Subcomponents

Leveraging scopes allows us to create either **dependent components** or **subcomponents**. The example above showed that we used the `@Singleton` annotation that lasted the entire lifecycle of the application. We also relied on one major Dagger component.

If we wish to have multiple components that do not need to remain in memory all the time (i.e. components that are tied to the lifecycle of an activity or fragment, or even tied to when a user is signed-in), we can create dependent components or subcomponents. In either case, each provide a way of encapsulating your code. We'll see how to use both in the next section.

There are several considerations when using these approaches:

- **Dependent components require the parent component to explicitly list out what dependencies can be injected downstream, while subcomponents do not.** For parent components, you would need to expose to the downstream component by specifying the type and a method:

```
// parent component
@Singleton
@Component(modules={AppModule.class, NetModule.class})
public interface NetComponent {
    // remove injection methods if downstream modules will perform injection

    // downstream components need these exposed
    // the method name does not matter, only the return type
    Retrofit retrofit(); 
    OkHttpClient okHttpClient();
    SharedPreferences sharedPreferences();
}
```

If you forget to add this line, you will likely to see an error about an injection target missing. Similar to how private/public variables are managed, using a parent component allows more explicit control and better encapsulation, but using subcomponents makes dependency injection easier to manage at the expense of less encapsulation.

- **Two dependent components cannot share the same scope.** For instance, two components cannot both be scoped as `@Singleton`. This restriction is imposed because of reasons described [here](https://github.com/google/dagger/issues/107#issuecomment-71073298). Dependent components need to define their own scope.
- **While Dagger 2 also enables the ability to create scoped instances, the responsibility rests on you to create and delete references that are consistent with the intended behavior.**Dagger 2 does not know anything about the underlying implementation. See this Stack Overflow [discussion](http://stackoverflow.com/questions/28411352/what-determines-the-lifecycle-of-a-component-object-graph-in-dagger-2) for more details.

#### Dependent Components

![Dagger Component Dependencies](https://raw.githubusercontent.com/codepath/android_guides/master/images/dagger_dependency.png)

For instance, if we wish to use a component created for the entire lifecycle of a user session signed into the application, we can define our own `UserScope` interface:

```java
import java.lang.annotation.Retention;
import javax.inject.Scope;

@Scope
public @interface UserScope {
}
```

Next, we define the parent component:

```
  @Singleton
  @Component(modules={AppModule.class, NetModule.class})
  public interface NetComponent {
      // downstream components need these exposed with the return type
      // method name does not really matter
      Retrofit retrofit();
  }
```

We can then define a child component:

```
@UserScope // using the previously defined scope, note that @Singleton will not work
@Component(dependencies = NetComponent.class, modules = GitHubModule.class)
public interface GitHubComponent {
    void inject(MainActivity activity);
}
```

Let's assume this GitHub module simply returns back an API interface to the GitHub API:

```
@Module
public class GitHubModule {

    public interface GitHubApiInterface {
      @GET("/org/{orgName}/repos")
      Call<ArrayList<Repository>> getRepository(@Path("orgName") String orgName);
    }

    @Provides
    @UserScope // needs to be consistent with the component scope
    public GitHubApiInterface providesGitHubInterface(Retrofit retrofit) {
        return retrofit.create(GitHubApiInterface.class);
    }
}
```

In order for this `GitHubModule.java` to get access to the `Retrofit` instance, we need explicitly define them in the upstream component. If the downstream modules will be performing the injection, they should also be removed from the upstream components too:

```
@Singleton
@Component(modules={AppModule.class, NetModule.class})
public interface NetComponent {
    // remove injection methods if downstream modules will perform injection

    // downstream components need these exposed
    Retrofit retrofit();
    OkHttpClient okHttpClient();
    SharedPreferences sharedPreferences();
}
```

The final step is to use the `GitHubComponent` to perform the instantiation. This time, we first need to build the `NetComponent` and pass it into the constructor of the `DaggerGitHubComponent` builder:

```
NetComponent mNetComponent = DaggerNetComponent.builder()
                .appModule(new AppModule(this))
                .netModule(new NetModule("https://api.github.com"))
                .build();

GitHubComponent gitHubComponent = DaggerGitHubComponent.builder()
                .netComponent(mNetComponent)
                .gitHubModule(new GitHubModule())
                .build();
```

See [this example code](https://github.com/codepath/dagger2-example) for a working example.

#### Subcomponents

![Dagger subcomponents](https://raw.githubusercontent.com/codepath/android_guides/master/images/dagger_subcomponent.png)

Using subcomponents is another way to extend the object graph of a component. Like components with dependencies, subcomponents have their own life-cycle and can be garbage collected when all references to the subcomponent are gone, and have the same scope restrictions. One advantage in using this approach is that you do not need to define all the downstream components.

Another major difference is that subcomponents simply need to be declared in the parent component.

Here's an example of using a subcomponent for an activity. We annotate the class with a custom scope and the `@Subcomponent` annotation:

```
@MyActivityScope
@Subcomponent(modules={ MyActivityModule.class })
public interface MyActivitySubComponent {
    @Named("my_list") ArrayAdapter myListAdapter();
}
```

The module that will be used is defined below:

```
@Module
public class MyActivityModule {
    private final MyActivity activity;

    // must be instantiated with an activity
    public MyActivityModule(MyActivity activity) { this.activity = activity; }
   
    @Provides @MyActivityScope @Named("my_list")
    public ArrayAdapter providesMyListAdapter() {
        return new ArrayAdapter<String>(activity, android.R.layout.my_list);
    }
    ...
}
```

Finally, in the **parent component**, we will define a factory method with the return value of the component and the dependencies needed to instantiate it:

```
@Singleton
@Component(modules={ ... })
public interface MyApplicationComponent {
    // injection targets here

    // factory method to instantiate the subcomponent defined here (passing in the module instance)
    MyActivitySubComponent newMyActivitySubcomponent(MyActivityModule activityModule);
}
```

In the above example, a new instance of the subcomponent will be created every time that the `newMyActivitySubcomponent()` is called. To use the submodule to inject an activity:

```
public class MyActivity extends Activity {
  @Inject ArrayAdapter arrayAdapter;

  public void onCreate(Bundle savedInstance) {
        // assign singleton instances to fields
        // We need to cast to `MyApp` in order to get the right method
        ((MyApp) getApplication()).getApplicationComponent())
            .newMyActivitySubcomponent(new MyActivityModule(this))
            .inject(this);
    } 
}
```

#### Subcomponent Builders

*Available starting in v2.7*

![Dagger subcomponent builders](https://raw.githubusercontent.com/codepath/android_guides/master/images/subcomponent_builders.png)

Subcomponent builders allow the creator of the subcomponent to be de-coupled from the parent component, by removing the need to have a subcomponent factory method declared on that parent component.

```
@MyActivityScope
@Subcomponent(modules={ MyActivityModule.class })
public interface MyActivitySubComponent {
    ...
    @Subcomponent.Builder
    interface Builder extends SubcomponentBuilder<MyActivitySubComponent> {
        Builder activityModule(MyActivityModule module);
    }
}

public interface SubcomponentBuilder<V> {
    V build();
}
```

The subcomponent is declared as an inner interface in the subcomponent interface and it must include a `build()` method which the return type matching the subcomponent. It's convenient to declare a base interface with this method, like `SubcomponentBuilder` above. This new **builder must be added to the parent component graph** using a "binder" module with a "subcomponents" parameter:

```
@Module(subcomponents={ MyActivitySubComponent.class })
public abstract class ApplicationBinders {
    // Provide the builder to be included in a mapping used for creating the builders.
    @Binds @IntoMap @SubcomponentKey(MyActivitySubComponent.Builder.class)
    public abstract SubcomponentBuilder myActivity(MyActivitySubComponent.Builder impl);
}

@Component(modules={..., ApplicationBinders.class})
public interface ApplicationComponent {
    // Returns a map with all the builders mapped by their class.
    Map<Class<?>, Provider<SubcomponentBuilder>> subcomponentBuilders();
}

// Needed only to create the above mapping
@MapKey @Target({ElementType.METHOD}) @Retention(RetentionPolicy.RUNTIME)
public @interface SubcomponentKey {
    Class<?> value();
}
```

Once the builders are made available in the component graph, the activity can use it to create its subcomponent:

```
public class MyActivity extends Activity {
  @Inject ArrayAdapter arrayAdapter;

  public void onCreate(Bundle savedInstance) {
        // assign singleton instances to fields
        // We need to cast to `MyApp` in order to get the right method
        MyActivitySubcomponent.Builder builder = (MyActivitySubcomponent.Builder)
            ((MyApp) getApplication()).getApplicationComponent())
            .subcomponentBuilders()
            .get(MyActivitySubcomponent.Builder.class)
            .get();
        builder.activityModule(new MyActivityModule(this)).build().inject(this);
    } 
}
```

## ProGuard

Dagger 2 should work out of box without ProGuard, but if you start seeing `library class dagger.producers.monitoring.internal.Monitors$1 extends or implements program class javax.inject.Provider`, make sure your Gradle configuration uses the `annotationProcessor`declaration instead of `provided`.

## Troubleshooting

- If you are upgrading Dagger 2 versions (i.e. from v2.0 to v2.5), some of the generated code has changed. If you are incorporating Dagger code that was generated with older versions, you may see `MemberInjector` and `actual and former argument lists different in length` errors. Make sure to clean the entire project and verify that you have upgraded all versions to use the consistent version of Dagger 2.

## References

- [Dagger 2 Github Page](http://google.github.io/dagger/)
- [Sample project using Dagger 2](https://github.com/vinc3m1/nowdothis)
- [Vince Mi's Codepath Meetup Dagger 2 Slides](https://docs.google.com/presentation/d/1bkctcKjbLlpiI0Nj9v0QpCcNIiZBhVsJsJp1dgU5n98/)
- <http://code.tutsplus.com/tutorials/dependency-injection-with-dagger-2-on-android--cms-23345>
- [Jake Wharton's Devoxx Dagger 2 Slides](https://speakerdeck.com/jakewharton/dependency-injection-with-dagger-2-devoxx-2014)
- [Jake Wharton's Devoxx Dagger 2 Talk](https://www.parleys.com/tutorial/5471cdd1e4b065ebcfa1d557/)
- [Dagger 2 Google Developers Talk](https://www.youtube.com/watch?v=oK_XtfXPkqw)
- [Dagger 1 to Dagger 2](http://frogermcs.github.io/dagger-1-to-2-migration/)
- [Tasting Dagger 2 on Android](http://fernandocejas.com/2015/04/11/tasting-dagger-2-on-android/)
- [Dagger 2 Testing with Mockito](http://blog.sqisland.com/2015/04/dagger-2-espresso-2-mockito.html#sthash.IMzjLiVu.dpuf)
- [Snorkeling with Dagger 2](https://github.com/konmik/konmik.github.io/wiki/Snorkeling-with-Dagger-2)
- [Dependency Injection in Java](https://www.objc.io/issues/11-android/dependency-injection-in-java/)
- [Component Dependency vs. Submodules in Dagger 2](http://jellybeanssir.blogspot.de/2015/05/component-dependency-vs-submodules-in.html)
- [Dagger 2 Component Scopes Test](https://github.com/joesteele/dagger2-component-scopes-test)
- [Advanced Dagger Talk](http://www.slideshare.net/nakhimovich/advanced-dagger-talk-from-360anDev)

Created by [CodePath](http://codepath.com/) with much help from the community. Contributed content licensed under [cc-wiki](https://creativecommons.org/licenses/by-sa/3.0/) with [attribution required](http://blog.stackoverflow.com/2009/06/attribution-required/). You are free to remix and reuse, as long as you attribute and use a similar license.
