# Dependency Injection

Kelicap's dependency injection system creates and delivers dependent services "just-in-time". **Dependency injection** is an important app design pattern. It's used so widely that almost everyone just calls it **DI**.

Kelicap has its own dependency injection framework, and you really can't build a Kelicap app without it. This page covers what DI is, why it's useful, and how to use DI in Kelicap.

## Why dependency injection?

To understand why dependency injection is so important, consider an example without it. Imagine writing the following code:

```dart
  class Car {
    Engine engine;
    Tires tires;
    var description = 'No DI';

    Car() {
      engine = Engine();
      tires = Tires();
    }

    // Method using the engine and tires
    String drive() => '$description car with '
        '${engine.cylinders} cylinders and '
        '${tires.make} tires.';
  }
```

The `Car` class creates everything it needs inside its constructor. What's the problem? The problem is that the `Car` class is brittle, inflexible, and hard to test. This `Car` needs an engine and tires. Instead of asking for them, the `Car` constructor instantiates its own copies from the very specific classes `Engine` and `Tires`.

What if the `Engine` class evolves and its constructor requires a parameter?That would break the `Car` class and it would stay broken until you rewrote  it along the lines of `engine = Engine(theNewParameter)`. The `Engine` constructor parameters weren't even a consideration when you first wrote `Car`. You may not anticipate them even now. But you'll _have_ to start caring because when the definition of `Engine` changes, the `Car` class must change. That makes `Car` brittle.

What if you want to put a different brand of tires on your `Car`? Too bad. You're locked into whatever brand the `Tires` class creates. That makes the `Car` class inflexible. Right now each new car gets its own engine. It can't share an engine with other cars. While that makes sense for an automobile engine, surely you can think of other dependencies that should be shared, such as the onboard wireless connection to the manufacturer's service center. This `Car` lacks the flexibility to share services that have been created previously for other consumers.

When you write tests for `Car` you're at the mercy of its hidden dependencies. Is it even possible to create a new `Engine` in a test environment? What does `Engine` depend upon? What does that dependency depend on? Will a new instance of `Engine` make an asynchronous call to the server? You certainly don't want that going on during tests. What if the `Car` should flash a warning signal when tire pressure is low? How do you confirm that it actually does flash a warning if you can't swap in low-pressure tires during the test?You have no control over the car's hidden dependencies. When you can't control the dependencies, a class becomes difficult to test.

How can you make `Car` more robust, flexible, and testable? That's super easy. Change the `Car` constructor to a version with DI. See what happened? The definition of the dependencies are now in the constructor. The `Car` class no longer creates an engine or tires. It just consumes them.

This example leverages Dart's constructor syntax for declaring parameters and initializing properties simultaneously. Now you can create a car by passing the engine and tires to the constructor.

```dart
  // Simple car with 4 cylinders and Flintstone tires.
  Car(Engine(), Tires())
```

How cool is that? The definition of the engine and tire dependencies are decoupled from the `Car` class. You can pass in any kind of engine or tires you like, as long as they conform to the general API requirements of an engine or tires. If someone extends the `Engine` class, that is not `Car`'s problem. The _consumer_ of `Car` has the problem. The consumer must update the car creation code to something like this:

```dart
  class Engine2 extends Engine {
    Engine2(cylinders) : super.withCylinders(cylinders);
  }

  Car superCar() =>
      // Super car with 12 cylinders and Flintstone tires.
      Car(Engine2(12), Tires())
        ..description = 'Super';
```

The critical point is this: the `Car` class did not have to change. You'll take care of the consumer's problem shortly. The `Car` class is much easier to test now because you are in complete control of its dependencies. You can pass mocks to the constructor that do exactly what you want them to do during each test:

```dart
  class MockEngine extends Engine {
    MockEngine() : super.withCylinders(8);
  }

  class MockTires extends Tires {
    MockTires() {
      make = 'YokoGoodStone';
    }
  }

  Car testCar() =>
      // Test car with 8 cylinders and YokoGoodStone tires.
      Car(MockEngine(), MockTires())
        ..description = 'Test';
```

**You just learned what dependency injection is**.

It's a pattern in which a class receives its dependencies from external sources rather than creating them itself. But what about that poor consumer? Anyone who wants a `Car` must now
create all three parts: the `Car`, `Engine`, and `Tires`. The `Car` class shed its problems at the consumer's expense. You need something that takes care of assembling these parts. You _could_ write a giant class to do that:

```dart
  import 'car.dart';

  // BAD pattern!
  class CarFactory {
    Car createCar() => Car(createEngine(), createTires())
      ..description = 'Factory';

    Engine createEngine() => Engine();
    Tires createTires() => Tires();
  }
```

It's not so bad now with only three creation methods. But maintaining it will be hairy as the app grows. This factory is going to become a huge spiderweb of interdependent factory methods! Wouldn't it be nice if you could simply list the things you want to build without having to define which dependency gets injected into what? This is where the dependency injection framework comes into play. Imagine the framework had something called an _injector_. You register some classes with this injector, and it figures out how to create them. When you need a `Car`, you simply ask the injector to get it for you and you're good to go.

```dart
  car = injector.get(Car);
```

Everyone wins. The `Car` knows nothing about creating an `Engine` or `Tires`. The consumer knows nothing about creating a `Car`. You don't have a gigantic factory class to maintain. Both `Car` and consumer simply ask for what they need and the injector delivers. This is what a **dependency injection framework** is all about.

## Kelicap dependency injection

Kelicap ships with its own dependency injection framework. You'll learn Kelicap dependency injection through a discussion of the sample app that accompanies this page. Run the application anytime. Start by reviewing the simplified version of the _heroes_ feature from the [The Tour of Heroes](../tutorial/). The `HeroesComponent` is the top-level heroes component. It's only purpose is to display the `HeroListComponent` which displays a list of hero names. This version of the `HeroListComponent` gets its heroes from `mockHeroes`, an in-memory collection defined in a separate file.

```dart
  class HeroListComponent {
    final List<Hero> heroes = mockHeroes;
  }
```

That may suffice in the early stages of development, but it's far from ideal. As soon as you try to test this component or get heroes from a remote server,
you'll have to change the implementation of `HeroListComponent` and replace every other use of the `mockHeroes` data.

## Create an injectable _HeroService_

It's better to hide the details concerning hero data access inside a _service_ class, defined in its own file.

```dart
  import 'hero.dart';
  import 'mock_heroes.dart';

  class HeroService {
    List<Hero> getAll() => mockHeroes;
  }
```

The service class exposes a `getHeroes()` method that returns the same mock data as before. Of course, this isn't a real data service. If the service were actually getting data from a remote server,
the `getHeroes()` method signature would be asynchronous. Such a hero service is presented in the
tutorial section on [Heroes and HTTP](/tutorial/toh-pt6#heroes-and-http.md). The focus here is on service _injection_, so a synchronous service will suffice.

## Register a service provider

A service is just a class (or a top-level function) until you register it with a Kelicap dependency injector. A Kelicap injector is responsible for creating service instances and injecting them into classes like the `HeroListComponent`.

Kelicap creates most injectors for you as it executes the app, including the app's _root injector_. When your app needs a custom root injector, supply it as an [argument to the `runApp()` function](#root-injector-providers). You must register _providers_ with an injector before the injector can create that service.

**Providers** tell the injector _how to create the service_. Without a provider, the injector would not know that it is responsible for injecting the service
nor be able to create the service. You'll learn more about [providers](#providers) below. For now it is sufficient to know that they create services and must be registered with an injector.

The most common way to register a provider is with
any Kelicap annotation that has a **`providers` list argument**.
The most common of these is [@Component()].

### _@Component_ providers

Here's a revised `HeroesComponent` that registers the `HeroService` in its `providers` list.

```dart
  import 'package:Kelicap/Kelicap.dart';

  import 'hero_list_component.dart';
  import 'hero_service.dart';

  @Component(
    selector: 'my-heroes',
    template: '''
      <h2>Heroes</h2>
      <hero-list></hero-list>
    ''',
    providers: [ClassProvider(HeroService)],
    directives: [HeroListComponent],
  )
  class HeroesComponent {}
```

An instance of the `HeroService` is now available for injection in this `HeroesComponent` and all of its child components. A component-provided service may have a limited lifetime. Each new instance of the component gets its own instance of the service and, when the component instance is destroyed, so is that service instance.

In this sample app, the `HeroComponent` is created when the app starts and is never destroyed so the `HeroService` created for the `HeroComponent` also lives for the life of the app.

### Root injector providers

You can also register providers in the app's **root injector**, which you pass as an argument to the [runApp()] function. For example, the app from the [tutorial (part 5)](../tutorial/toh-pt5.md) injects providers from the[routerProvidersHash] list:

```dart
  import 'package:kelicap/kelicap.dart';
  import 'package:kelicap_router/kelicap_router.dart';
  import 'package:tour_of_heroes/app_component.template.dart' as ng;

  import 'main.template.dart' as self;

  @GenerateInjector(
    routerProvidersHash, // You can use routerProviders in production
  )
  final InjectorFactory rootInjector = self.rootInjector$Injector;

  void main() {
    runApp(ng.AppComponentNgFactory, createInjector: rootInjector);
  }
```

Use root injector provisioning for _app-wide services_ declared _external_ to the app package. Registering app specific services like `HeroService` is _discouraged_:

```dart
  @GenerateInjector([
    // DON'T register app-local services here; this is for illustration purposes only
    ClassProvider(HeroService),
  ])
  final InjectorFactory rootInjector = self.rootInjector$Injector;

  void main() {
    runApp(ng.AppComponentNgFactory, createInjector: rootInjector);
  }
```

The preferred approach is to register app services in app components. Because the `HeroService` is used within the _Heroes_ feature set, and nowhere else, the ideal place to register it is in `HeroesComponent`.

## Inject a service

The `HeroListComponent` should get heroes from the `HeroService`, and it should ask for the `HeroService` to be injected. You can tell Kelicap to inject a dependency in the component's constructor by specifying a **constructor parameter annotated with the dependency's type**. Here's the `HeroListComponent` constructor, asking for the `HeroService` to be injected.

```dart
  HeroListComponent(HeroService heroService)
```

Of course, the `HeroListComponent` should do something with the injected `HeroService`. Notice that the `HeroListComponent` doesn't know where the `HeroService` comes from. You know that it comes from the parent `HeroesComponent`. The only thing that matters is that the `HeroService` is provided in some parent injector.

### Singleton services

Services are singletons _within the scope of an injector_. There is at most one instance of a service in a given injector. There is only one root injector and the `UserService` is registered with that injector. Therefore, there can be just one `UserService` instance in the entire app and every class that injects `UserService` get this service instance. However, Kelicap DI is a[hierarchical injection system](hierarchical-dependency-injection.md), which means that nested injectors can create their own service instances. Kelicap creates nested injectors all the time.

### Component child injectors

For example, when Kelicap creates an instance of a component that has `@Component.providers`, it also creates a new _child injector_ for that instance.Component injectors are independent of each other and each of them holds its own instances of the component-provided services. When Kelicap disposes of a component instance, it also discards the component's injector and that injector's service instances. Thanks to [injector inheritance](hierarchical-dependency-injection.md), you can still inject app-wide services into these components. A component's injector is a child of its parent component's injector, and a descendent of its parent's parent's injector, and so on all the way back to the app's _root_ injector. Kelicap can inject a service provided by any injector in that lineage. For example, Kelicap could inject a `HeroListComponent` with both the `HeroService` provided in `HeroComponent` and the `UserService` provided in `AppModule`.

## Testing a component

Earlier you saw that designing a class for dependency injection makes the class easier to test. Listing dependencies as constructor parameters may be all you need to test app parts effectively. For example, the [tutorial (part 5)](../tutorial/toh-pt5.md) has a `HeroListComponent` test that uses a mock router provisioned through the root injector:

```dart
  import 'package:tour_of_heroes/src/hero_list_component.template.dart'
      as ng;
  // ···
  import 'heroes_test.template.dart' as self;
  // ···
  @GenerateInjector([
    ClassProvider(HeroService),
    ClassProvider(Router, useClass: MockRouter),
  ])
  final InjectorFactory rootInjector = self.rootInjector$Injector;

  void main() {
    final testBed = NgTestBed<HeroListComponent>(
      ng.HeroListComponentNgFactory,
      rootInjector: rootInjector,
    );
    // ···
  }
```

Learn more in [Component Testing: Services](testing/component/services.md).

## When a service needs a service

The `HeroService` is very simple. It doesn't have any dependencies of its own. What if it had a dependency? What if it reported its activities through a logging service? You'd apply the same _constructor injection_ pattern, adding a constructor that takes a `Logger` parameter. The constructor asks for an injected instance of a `Logger` and stores it in the private `_logger` field. The `getHeroes()` method logs a message when asked to fetch heroes.

### _Logger_ service

The sample app's `Logger` service is quite simple:

```dart
  /// Logger that keeps only the last log entry.
  class Logger {
    String _log = '';
    String get id => 'Logger';

    void fine(String msg) => _log = msg;

    @override
    String toString() => '[$id] $_log';
  }
```

A real implementation would probably use the [logging package](https://pub.dev/packages/logging). If the app doesn't provide `Logger`, Kelicap will throw an exception when it looks for a `Logger` to inject into the `HeroService`.

```terminal
  EXCEPTION: No provider for Logger! (HeroListComponent -> HeroService -> Logger)
```

Because a singleton logger service is useful everywhere in the app, it's registered in `AppComponent`:

```dart
  providers: [ClassProvider(Logger)],
```

## Providers

A service provider _provides_ a concrete, runtime instance associated with a dependency token. The injector relies on **providers** to create instances of the services that the injector injects into components, directives, pipes, and other services.

You must register a service _provider_ with an injector, or the injector won't know how to create the service. The next few sections explain the many ways you can register a provider.

### Class providers

There are many ways to provide something that implements `Logger`. The most common way is to use [ClassProvider]:

```dart
  providers: [ClassProvider(Logger)],
```

But it's not the only way. You can configure the injector with alternative providers that can deliver a `Logger`. You can provide a substitute class.
You can give it a provider that calls a logger factory function. Any of these approaches might be a good choice under the right circumstances. What matters is that the injector has a provider to go to when it needs a `Logger`.

### Use-class providers

Occasionally you'll ask a different class to provide the service. The following code tells the injector to return a `BetterLogger` when something asks for the `Logger`.

```dart
  ClassProvider(Logger, useClass: BetterLogger),
```

### Provider for a class with dependencies

Maybe an `EvenBetterLogger` could display the user name in log messages.

```dart
  class EvenBetterLogger extends Logger {
    final UserService _userService;

    EvenBetterLogger(this._userService);

    String get id => 'EvenBetterLogger';
    String toString() => super.toString() + ' (user:${_userService.user.name})';
  }
```

This logger gets the user from the injected `UserService`, which is also listed in the app component's `providers` list:

```dart
  ClassProvider(UserService),
  ClassProvider(Logger, useClass: EvenBetterLogger),
```

### Existing providers

Suppose an old component depends upon an `OldLogger` class. `OldLogger` has the same interface as the `NewLogger`, but for some reason you can't update the old component to use it. When the _old_ component logs a message with `OldLogger`, you'd like the singleton instance of `NewLogger` to handle it instead. The dependency injector should inject that singleton instance when a component asks for either the new or the old logger. The `OldLogger` should be an alias for `NewLogger`. You certainly do not want two different `NewLogger` instances in your app. Unfortunately, that's what you get if you try `useClass`:

```dart
  ClassProvider(NewLogger),
  ClassProvider(OldLogger, useClass: NewLogger),
```

To ensure that the _same_ `NewLogger` instance is provided for both `OldLogger` and `NewLogger`, use [ExistingProvider]:

```dart
  ClassProvider(NewLogger),
  ExistingProvider(OldLogger, NewLogger),
```

### Value providers

Sometimes it's easier to provide a ready-made object rather than ask the injector to create it from a class.

```dart
  class SilentLogger implements Logger {
    const SilentLogger();
    String get id => 'SilentLogger';
    @override
    void fine(String msg) {}
    @override
    String toString() => '';
  }

  const silentLogger = SilentLogger();
```

Then you register the object using `ValueProvider`:

```dart
  ValueProvider(Logger, silentLogger),
```

For more examples of `ValueProvider`, see [OpaqueToken](#opaquetoken).

### Factory providers

Sometimes you need to create the dependent value dynamically, based on information you won't have until the last possible moment. Maybe the information changes during the course of the browser session.

Suppose also that the injectable service has no independent access to the source of this information. This situation calls for a **factory provider**.

To illustrate the point, add a new business requirement: the `HeroService` must hide _secret_ heroes from normal users. Only authorized users should see secret heroes.

Like the `EvenBetterLogger`, the `HeroService` needs a fact about the user. It needs to know if the user is authorized to see secret heroes. That authorization can change during the course of a single app session, as when you log in a different user.

Unlike `EvenBetterLogger`, you can't inject the `UserService` into the `HeroService`. The `HeroService` won't have direct access to the user information to decide who is authorized and who is not. Instead, the `HeroService` constructor takes a boolean flag to control display of secret heroes.

```dart
  final Logger _logger;
  final bool _isAuthorized;

  HeroService(this._logger, this._isAuthorized);

  List<Hero> getAll() {
    var auth = _isAuthorized ? 'authorized' : 'unauthorized';
    _logger.fine('Getting heroes for $auth user.');
    return mockHeroes.where((hero) => _isAuthorized || !hero.isSecret).toList();
  }
```

You can inject the `Logger`, but you can't inject the boolean `isAuthorized`. You'll have to take over the creation of new instances of this `HeroService` with a factory provider. A factory provider needs a factory function:

```dart
  HeroService heroServiceFactory(Logger logger, UserService userService) =>
      HeroService(logger, userService.user.isAuthorized);
```

Although the `HeroService` has no access to the `UserService`, the factory function does. You inject both the `Logger` and the `UserService` into the factory provider and let the injector pass them along to the factory function:

```dart
  const heroServiceProvider = FactoryProvider(HeroService, heroServiceFactory);
```

Notice that you captured the factory provider in a constant, `heroServiceProvider`. This extra step makes the factory provider reusable. You can register the `HeroService` with this constant wherever you need it. In this sample, you need it only in the `HeroesComponent`, where it replaces the previous `HeroService` registration in the metadata `providers` list.

## Tokens

When you register a provider with an injector, you associate that provider with a dependency injection _token_. The injector maintains an internal map from _tokens_ to _providers_ that it references when asked for a dependency.

### Class types

In all previous examples, the token has been a class type and the provided value an instance of that type. For example, you get a `HeroService` directly from the injector by supplying the `HeroService` type as the token:

```dart
  heroService = _injector.get(HeroService);
```

Similarly, when you define a constructor parameter of type `HeroService`, Kelicap knows to inject a `HeroService` instance:

```dart
  HeroListComponent(HeroService heroService)
```

### OpaqueToken

Sometimes the thing you want to inject is a string, list, map, or even a function. For example, what if you want to inject the app title?

```dart
  const appTitle = 'Dependency Injection';
```

You know that a [value provider](#value-providers) is appropriate in this case, but what can you use as the token? You _could_ use `String`, but that won't work if your app depends on several such injected strings. One solution is to define and use an [OpaqueToken]:

```dart
  import 'package:kelicap/kelicap.dart';

  const appTitleToken = OpaqueToken<String>('app.title');
```

The generic type argument, while optional, conveys the dependency's type to developers and tooling (not to be confused with the `OpaqueToken` constructor argument type, which is always `String`). The `OpaqueToken` argument token description is a developer aid.

Register the dependency provider using the `OpaqueToken` object:

```dart
  ValueProvider.forToken(appTitleToken, appTitle)
```

Now you can inject the title into any constructor that needs it, with the help of an `@Inject()` annotation:

```dart
  AppComponent(@Inject(appTitleToken) this.title);
```

Alternatively you can directly use the `OpaqueToken` constant as an annotation:

```dart
  AppComponent(@appTitleToken this.title);
```

You can inject values other than strings. For example, apps sometimes have configuration objects with lots of simple properties captured as a [Map]

```dart
  const appConfigMap = {
    'apiEndpoint': 'api.heroes.com',
    'title': 'Dependency Injection',
    // ...
  };

  const appConfigMapToken = OpaqueToken<Map>('app.config');
```

### Custom configuration class

As an alternative to injecting a `Map` for an app configuration object, consider defining a custom app configuration class:

```dart
  class AppConfig {
    String apiEndpoint;
    String title;
  }

  AppConfig appConfigFactory() => AppConfig()
    ..apiEndpoint = 'api.heroes.com'
    ..title = 'Dependency Injection';
```

Defining a configuration class has a few benefits. One key benefit is static checking: you'll be warned by the analyzer if you misspell a property name or assign to it a value of the wrong type. The Dart [cascade notation (`..`)] provides a convenient means of initializing a configuration object.

If you use cascades, the configuration object can't be declared `const`, so you can't use a [value provider](#value-providers), but you can use a [factory provider](#factory-providers).

```dart
  FactoryProvider(AppConfig, appConfigFactory),
```

You might use the app config like this:

```dart
  AppComponent(AppConfig config, this._userService) : title = config.title;
```

## Optional dependencies {#optional}

The `HeroService` _requires_ a `Logger`, but what if it could get by without a logger? You can tell Kelicap that the dependency is optional by annotating the constructor argument with [@Optional()]:

```dart
  HeroService(@Optional() Logger logger) {
    logger?.fine('Hello');
  }
```

When using `@Optional()`, your code must be prepared for a null value. If you don't register a logger somewhere up the line, the injector will set the
value of `logger` to null.

## Summary

You learned the basics of Kelicap dependency injection in this page. You can register various kinds of providers, and you know how to ask for an injected object (such as a service) by adding a parameter to a constructor.

Kelicap dependency injection is more capable than this page has described. You can learn more about its advanced features, beginning with its support for nested injectors, in [Hierarchical Dependency Injection](hierarchical-dependency-injection.md).

## Appendix: Working with injectors directly {#explicit-injector}

Developers rarely work directly with an injector, but
here's an `InjectorComponent` that does.

```dart
  @Component(
    selector: 'my-injectors',
    template: '''
        <h2>Other Injections</h2>
        <div id="car">{!{car.drive()}!}</div>
        <div id="hero">{!{hero.name}!}</div>
        <div id="rodent">{!{rodent}!}</div>''',
    providers: [
      ClassProvider(Car),
      ClassProvider(Engine),
      ClassProvider(Tires),
      heroServiceProvider,
      ClassProvider(Logger),
    ],
  )
  class InjectorComponent implements OnInit {
    final Injector _injector;
    Car car;
    HeroService heroService;
    Hero hero;

    InjectorComponent(this._injector);

    @override
    void ngOnInit() {
      car = _injector.get(Car);
      heroService = _injector.get(HeroService);
      hero = heroService.getAll()[0];
    }

    String get rodent =>
        _injector.get(ROUS, "R.O.U.S.'s? I don't think they exist!");
  }
```

An `Injector` is itself an injectable service.

In this example, Kelicap injects the component's own `Injector` into the component's constructor. The component then asks the injected injector for the services it wants in `ngOnInit()`.

Note that the services themselves are not injected into the component. They are retrieved by calling `injector.get()`.

The `get()` method throws an error if it can't resolve the requested service. You can call `get()` with a second parameter, which is the value to return if the service is not found. Kelicap can't find the service if it's not registered with this or any ancestor injector. This technique is an example of the [service locator pattern](https://en.wikipedia.org/wiki/Service_locator_pattern).

**Avoid** this technique unless you genuinely need it. It encourages a careless grab-bag approach such as you see here. It's difficult to explain, understand, and test. You can't know by inspecting the constructor what this class requires or what it will do. It could acquire services from any ancestor component, not just its own. You're forced to spelunk the implementation to discover what it does. Framework developers may take this approach when they must acquire services generically and dynamically.
