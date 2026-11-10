# Component Testing: Routing Components

**Component testing topics:**

- [Component testing](README.md)
- [Running component tests](running-tests.md)
- Writing component tests
  - [Basics](basics.md): pubspec config, test API fundamentals
  - [Page objects](page-objects.md): field annotation, initialization, and more
  - [Simulating user action](simulating-user-action.md): click, type, clear
  - [Services](services.md): local, external, mock, real
  - [`@Input()` and `@Output()`](input-and-output.md)
  - [Routing components](routing-components.md)

This page describes how to test routing components using real or mock routers. Whether or not you mock the router will, among other reasons,  depend on the following:

- The degree to which you wish to test your component in isolation
- The effort you are willing to invest in coding mock router behavior for your particular tests

## Running example

This page uses the heroes app from [part 5][] of the [tutorial][] as a running example.

## Using a mock router

To test using a mock router, you must add the mock router class to the providers list of the generated root injector, as described in the section on [Component-external services: mock or real](services.md#component-external-services-mock-or-real):

```dart
  import 'package:tour_of_heroes/src/hero_list_component.template.dart'
      as ng;
  import 'package:tour_of_heroes/src/hero_service.dart';
  import 'package:tour_of_heroes/src/route_paths.dart';
  import 'package:mockito/mockito.dart';
  import 'package:ngpageloader/html.dart';
  import 'package:test/test.dart';

  import 'heroes_test.template.dart' as self;
  import 'heroes_po.dart';
  import 'utils.dart';

  late NgTestFixture<HeroListComponent> fixture;
  late HeroesPO po;

  @GenerateInjector([
    ClassProvider(HeroService),
    ClassProvider(Router, useClass: MockRouter),
  ])
  final InjectorFactory rootInjector = self.rootInjector$Injector;

  void main() {
    final injector = InjectorProbe(rootInjector);
    final testBed = NgTestBed<HeroListComponent>(
      ng.HeroListComponentNgFactory,
      rootInjector: injector.factory,
    );

    setUp(() async {
      fixture = await testBed.create();
      final context =
          HtmlPageLoaderElement.createFromElement(fixture.rootElement);
      po = HeroesPO.create(context);
    });

    tearDown(disposeAnyRunningTest);
    // ···
  }
```

The `InjectorProbe` will allow individual tests to access the test context injector. You'll see an example soon.

```dart
  class InjectorProbe {
    InjectorFactory _parent;
    Injector? _injector;

    InjectorProbe(this._parent);

    InjectorFactory get factory => _factory;
    Injector? get injector => _injector;

    Injector _factory(Injector parent) => _injector = _parent(parent);
    T get<T>(dynamic token) => injector?.get(token);
  }
```

The following one-line class definition of `MockRouter` illustrates how easy it is to use the [Mockito][package:mockito] package to define a mock class with the appropriate API:

```dart
  @GenerateNiceMocks([MockSpec<Router>()])
  export 'utils.mocks.dart';
```

### Testing programmatic link navigation

Selecting a hero from the heroes list causes a ["mini detail" view to appear](../../../tutorial/toh-pt5.md#add-the-mini-detail):

![Mini Hero Detail](../../../tutorial/assets/toh/mini-hero-detail.png)

Clicking the "View Details" button should cause a [request to navigate to the corresponding hero's detail view](../../../tutorial/toh-pt5.md#update-the-herolistcomponent-class). The button's click event is bound to the `gotoDetail()` method which is defined as follows:

```dart
  Future<NavigationResult> gotoDetail() =>
      _router.navigate(_heroUrl(selected!.id));
```

The heroes test works as follows:

- The `setUp()` method selects a hero.
- The test expects a _single_ call to the mock router's `navigate()` method, with an appropriate path a parameter.

### Testing routerLink attribute navigation

The app [dashboard][], from [part 5][] of the [tutorial][], supports direct navigation to hero details using router links:

```html
  <a *ngFor="let hero of heroes" class="col-1-4"
     [routerLink]="heroUrl(hero.id)">
    <div class="module hero">
      <h4>{{hero.name}}</h4>
    </div>
  </a>
```

Setting up the initial providers list is a bit more involved. Because the dashboard uses the [RouterLink][] directive, you need to add all the usual router providers ([routerProviders][]). Since you'll be testing outside of the context of the app's `index.html` file, which sets the [\<base href>][base href], you also need to provide a value for [appBaseHref][]:

```dart
  final testBed = NgTestBed<DashboardComponent>(ng.DashboardComponentNgFactory,
      rootInjector: injector.factory);
```

The test itself is similar to the one used for heroes, with the exception that navigating a `routerLink` causes the router's `navigate()` method to be called with two arguments. The test checks for expected values for both arguments.

How might you write the tests shown in this section using a real router? That's covered next.

## Using a real router

### Testing the app root

Provisioning and setup for use of a real router is similar to what you've seen already:

```dart
  final injector = InjectorProbe(rootInjector);
  final testBed = NgTestBed<AppComponent>(ng.AppComponentNgFactory,
      rootInjector: injector.factory);

  setUp(() async {
    fixture = await testBed.create();
    router = injector.get<Router>(Router);
    await router.navigate('/');
    await fixture.update();
    final context =
        HtmlPageLoaderElement.createFromElement(fixture.rootElement);
    appPO = AppPO.create(context);
  });
```

Where `routerProvidersForTesting` is defined as follows:

```dart
  const /* List<Provider|List<Provider>> */ routerProvidersForTesting = [
    ValueProvider.forToken(appBaseHref, '/'),
    routerProviders,
    // Mock platform location even with real router, otherwise sometimes tests hang.
    ClassProvider(PlatformLocation, useClass: MockPlatformLocation),
  ];
```

Among other things, testing the app root using a real router allows you to exercise features like [deep linking][]:

```dart
  group('Deep linking:', () {
    test('navigate to hero details', () async {
      await router.navigate('/heroes/11');
      await fixture.update();
      expect(fixture.rootElement.querySelector('my-hero'), isNotNull);
    });

    test('navigate to heroes', () async {
      await router.navigate('/heroes');
      await fixture.update();
      expect(fixture.rootElement.querySelector('my-heroes'), isNotNull);
    });
  });
```

Not only are you using the real router, but the tests shown above are also testing out the use of app routes.

Testing using a real router for a non-app-root component requires more test infrastructure, as you'll see in the next section.

### Testing a non-root component

Consider testing the dashboard component using a real router. Remember that the dashboard is navigated to, and its view is displayed in the app root's [RouterOutlet][], which gets initialized with the full set of app routes:

```html
  template: '''
    <h1>{{title}}</h1>
    <nav>
      <a [routerLink]="RoutePaths.dashboard.toUrl()"
         [routerLinkActive]="'active'">Dashboard</a>
      <a [routerLink]="RoutePaths.heroes.toUrl()"
         [routerLinkActive]="'active'">Heroes</a>
    </nav>
    <router-outlet [routes]="Routes.all"></router-outlet>
  ''',
```

Before you can test a dashboard, you need to create a test component with a router outlet and a suitably (restricted) set of routes, something like this:

```dart
  @Component(
    selector: 'test',
    template: '''
      <my-dashboard></my-dashboard>
      <router-outlet [routes]="Routes.heroRoute"></router-outlet>
    ''',
    directives: [RouterOutlet, DashboardComponent],
    exports: [Routes],
  )
  class TestComponent {
    final Router router;

    TestComponent(this.router);
  }
```

The test bed and test fixture are then parameterized over `TestComponent` rather than `DashboardComponent`:

```dart
  late NgTestFixture<TestComponent> fixture;
  late DashboardPO po;
  late Router router;

  @GenerateInjector([
    ClassProvider(HeroService),
    routerProvidersForTesting,
  ])
  final InjectorFactory rootInjector = self.rootInjector$Injector;

  void main() {
    final injector = InjectorProbe(rootInjector);
    final testBed = NgTestBed<TestComponent>(
      self.TestComponentNgFactory as ComponentFactory<TestComponent>,
      rootInjector: injector.factory,
    );
    // ···
  }
```

One way to test navigation, is to log the real router's change in navigation state. You can achieve this by registering a listener:

```dart
  late List<RouterState> navHistory;

  setUp(() async {
    fixture = await testBed.create();
    router = fixture.assertOnlyInstance.router;
    navHistory = [];
    router.onRouteActivated.listen((newState) => navHistory.add(newState));
    final context =
        HtmlPageLoaderElement.createFromElement(fixture.rootElement);
    po = DashboardPO.create(context);
  });
```

Using this navigation history, the go-to-detail test illustrated previously when using a mock router, can be written as follows:

```dart
  test('select hero and navigate to detail + navHistory', () async {
    await po.selectHero(3);
    await fixture.update();
    expect(navHistory.length, 1);
    expect(navHistory[0].path, '/heroes/15');
    // Or, using a custom matcher:
    expect(navHistory[0], isRouterState('/heroes/15'));
  });
```

Contrast this with the [heroes "go to details" test](#testing-routerlink-attribute-navigation) shown earlier. While the dashboard test requires more testing infrastructure, the test has the advantage of ensuring that [route configurations][] are declared as expected.

Alternatively, for a simple test scenario like go-to-detail, you can simply test the last URL cached by the mock platform location:

```dart
  test('select hero and navigate to detail + mock platform location', () async {
    await po.selectHero(3);
    await fixture.update();
    final mockLocation = injector.get<MockPlatformLocation>(PlatformLocation);
    expect(mockLocation.pathname, '/heroes/15');
  });
```

[appBaseHref]: https://pub.dev/documentation/kelicap_router/latest/kelicap_router/appBaseHref-constant.html
[app shell]: ../../../tutorial/toh-pt5.md#create-appcomponent
[base href]: ../../router/1.md#set-the-base-href
[dashboard]: ../../../tutorial/toh-pt5.md#add-heroes-to-the-dashboard
[deep linking]: https://en.wikipedia.org/wiki/Deep_linking
[link parameter list]: ../../router/appendices.md#appendix-link-parameters-list
[package:mockito]: https://pub.dev/packages/mockito
[part 5]: ../../../tutorial/toh-pt5.md
[PlatformLocation]: https://pub.dev/documentation/kelicap_router/latest/kelicap_router/PlatformLocation-class.html
[route configurations]: ../../../tutorial/toh-pt5.md#configure-routes
[routerProviders]: https://pub.dev/documentation/kelicap_router/latest/kelicap_router/routerProviders-constant.html
[RouterLink]: https://pub.dev/documentation/kelicap_router/latest/kelicap_router/RouterLink-class.html
[RouterOutlet]: https://pub.dev/documentation/kelicap_router/latest/kelicap_router/RouterOutlet-class.html
[tutorial]: ../../../tutorial/README.md
