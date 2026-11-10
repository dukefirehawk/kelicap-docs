# Component Testing: @Input() and @Output()

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

This section describes how to test components with [@Input(), and @Output() properties](../../template-syntax.md#input-and-output-properties--input-and-output-).

## Running example

The app from [part 3][] of the [tutorial][] will be used as a running example to illustrate how to test a component with `@Input()` properties, specifically the `HeroDetailComponent`:

```dart
  import 'package:kelicap/kelicap.dart';
  import 'package:kelicap_forms/kelicap_forms.dart';

  import 'hero.dart';

  @Component(
    selector: 'my-hero',
    template: '''
      <div *ngIf="hero != null">
        <h2>{{hero!.name}}</h2>
        <div><label>id: </label>{{hero!.id}}</div>
        <div>
          <label>name: </label>
          <input [(ngModel)]="hero!.name" placeholder="name">
        </div>
      </div>''',
    directives: [coreDirectives, formDirectives],
  )
  class HeroComponent {
    @Input()
    Hero? hero;
  }
```

Here is the [page object][] for this component:

```dart
  import 'dart:async';

  import 'package:ngpageloader/pageloader.dart';

  part 'hero_detail_po.g.dart';

  @PageObject()
  abstract class HeroDetailPO {
    HeroDetailPO();
    factory HeroDetailPO.create(PageLoaderElement context) = $HeroDetailPO.create;

    @First(ByCss('div h2'))
    PageLoaderElement get _title;

    @First(ByCss('div div'))
    PageLoaderElement get _id;

    @ByTagName('input')
    PageLoaderElement get _input;

    Map? get heroFromDetails {
      if (!_id.exists) return null;
      final idAsString = _id.visibleText.split(':')[1];
      return _heroData(idAsString, _title.visibleText);
    }

    Future<void> clear() => _input.clear();
    Future<void> type(String s) => _input.type(s);

    Map<String, dynamic> _heroData(String idAsString, String name) =>
        {'id': int.tryParse(idAsString) ?? -1, 'name': name};
  }
```

The app component template contains a `<my-hero>` element that binds the `hero` property to the app component's `selectedHero`:

```html
  <my-hero [hero]="selected"></my-hero>
```

The tests shown below use the following target hero data:

```dart
  const targetHero = {'id': 1, 'name': 'Alice'};
```

## @Input(): No initial value

This case occurs when either of the following is true:

- The input is bound to an initial null value, such as when app component's `selectedHero` is null above.
- A component uses a `<my-hero>` element without a `hero` property:
  ```html
  <my-hero></my-hero>
    ```

When a component is created, its inputs are left uninitialized, so basic [page object][] setup is sufficient to test for this case:

```dart
  group('No initial @Input() hero:', () {
    setUp(() async {
      fixture = await testBed.create();
      final context =
          HtmlPageLoaderElement.createFromElement(fixture.rootElement);
      po = HeroDetailPO.create(context);
    });

    test('has empty view', () {
      expect(fixture.rootElement.text!.trim(), '');
      expect(po.heroFromDetails, isNull);
    });
    // ···
  });
```

## @Input(): Non-null initial value

Initialization of an input property with a non-null value must be done when the test fixture is created. Provide an initialization callback as the named parameter `beforeChangeDetection` of the `NgTestBed.create()` method:

```dart
  group('${targetHero['name']} initial @Input() hero:', () {
    // ···
    setUp(() async {
      fixture = await testBed.create(
          beforeChangeDetection: (c) => c.hero = Hero(
                targetHero['id'] as int,
                targetHero['name'] as String,
              ));
      final context =
          HtmlPageLoaderElement.createFromElement(fixture.rootElement);
      po = HeroDetailPO.create(context);
    });

    test('show hero details', () {
      expect(po.heroFromDetails, targetHero);
    });
    // ···
  });
```

## @Input(): Change value

To emulate an input binding's change in value, use the `NgTestFixture.update()` method. This applies whether or not the input property was [explicitly initialized](#input-non-null-initial-value):

```dart
  group('No initial @Input() hero:', () {
    setUp(() async {
      fixture = await testBed.create();
      final context =
          HtmlPageLoaderElement.createFromElement(fixture.rootElement);
      po = HeroDetailPO.create(context);
    });
    // ···

    test('transition to ${targetHero['name']} hero', () async {
      await fixture.update((comp) {
        comp.hero = Hero(
          targetHero['id'] as int,
          targetHero['name'] as String,
        );
      });
      expect(po.heroFromDetails, targetHero);
    });
  });
```

## @Output() properties

An `@Output()` property allows a component to raise [custom events][] in response to a timeout or input event. Output properties are visible from a component's API as public [Stream][] fields.

You can test an output property by first triggering a change. Then wait for an expected update on the output property's stream, from inside the callback passed as argument to the `NgTestFixture.update()` method.

For example, you might test the font [sizer component][], from the [Two-way binding][] section of the [Template Syntax][] page, as follows:

```dart
  group('inc:', () {
    const expectedSize = initSize + 1;

    setUp(() => po.inc());

    test('font size is $expectedSize', () async {
      // ···
    });

    test(
        '@Output $expectedSize size event',
        () => fixture.update((c) async {
              expect(await c.sizeChange.first, expectedSize);
            }));
  });
```

In this test group, the `setUp()` method initiates a font increment event, and the output test awaits for the updated font size to appear on the `sizeChange` stream.

[custom events]: ../../template-syntax.md#custom-events
[page object]: page-objects.md
[part 3]: ../../../tutorial/toh-pt3.md
[Stream]: https://api.dart.dev/stable/dart-async/Stream-class.html
[sizer component]: ../../template-syntax.md#two-way-binding---
[Two-way binding]: ../../template-syntax.md#two-way-binding---
[Template Syntax]: ../../template-syntax.md
[tutorial]: ../../../tutorial/README.md
