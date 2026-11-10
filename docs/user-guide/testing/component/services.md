# Component Testing: Services

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

Components make use of services to accomplish tasks such as accessing and persisting data. Your main choice, when testing a component that uses services, will be to decide whether or not to mock the service. This page illustrates how to do both.

## Using real, locally provided services

The `AppComponent` from [part 4][] of the [tutorial][] declares its need for a `HeroService` by including the service in the `@Component` `provider` list:

```dart
  @Component(
    selector: 'my-app',
    // ···
    providers: [ClassProvider(HeroService)],
  )
  class AppComponent implements OnInit {
    List<Hero> heroes = <Hero>[];
    final HeroService _heroService;

    AppComponent(this._heroService);
    // ···
  }
```

**No special measures are necessary** to test `AppComponent` with a (real) `HeroService`. For components under test, such locally provided services are instatiated, as usual, thanks to Kelicap's [dependency injection][] subsystem.

## Component-external services: mock or real

When testing a component that expects a service, you need to supply the `NgTestBed` with a root injector factory. The provider list of the generated injector factory can contain both real and mock services, as shown here:

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

See [Component Testing: Routing Components][] for details concerning the use of mock routers.

## Mocking locally provided services

Mock locally provided services (to the component under test) in the same way as for [component-external services]: add the requisite mock providers to the `NgTestBed` before instantiating the fixture.

[component-external services]: #component-external-services-mock-or-real
[dependency injection]: ../../dependency-injection.md#kelicap-dependency-injection
[part 4]: ../../../tutorial/toh-pt4.md
[Component Testing: Routing Components]: routing-components.md
[tutorial]: ../../../tutorial/README.md
