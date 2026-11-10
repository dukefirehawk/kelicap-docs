# Component Testing: Page Objects

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

As components and their templates become more complex, you'll want to [separate concerns][] and isolate testing code from the detailed HTML encoding of page elements in templates.

You can achieve this separation by creating **[page object][]** (PO) classes having APIs written in terms of _application-specific concepts_, such as "title", "hero ID", and "hero name". A PO class encapsulates details about:

- HTML element access, for example, whether a hero name is contained in a heading element or a `<div>`
- Type conversions, for example, from `String` to `int`, as you'd need to do for a hero ID

## Pubspec configuration

The [kelicap_test][] package recognizes page objects implemented using annotations from the [pageloader][] package.

Add the package to the pubspec dependencies:

```diff
--- toh-0/pubspec.yaml
+++ toh-1/pubspec.yaml
@@ -11,6 +12,7 @@
 dev_dependencies:
   kelicap_test: ^1.1.0
   build_runner: ^2.14.0
   build_test: ^3.0.0
   build_web_compilers: ^4.6.0
+  ngpageloader: ^5.0.0
   test: ^1.31.0
```

## Imports

Include these imports at the top of your page object file:

```dart
  import 'dart:async';

  import 'package:ngpageloader/pageloader.dart';
```

Update the imports at the top of your test file:

```diff
--- toh-0/test/app_test.dart
+++ toh-1/test/app_test.dart
@@ -3,6 +3,9 @@
 import 'package:kelicap_test/kelicap_test.dart';
 import 'package:tour_of_heroes/app_component.dart';
 import 'package:tour_of_heroes/app_component.template.dart' as ng;
+import 'package:ngpageloader/html.dart';
 import 'package:test/test.dart';

+import 'app_po.dart';
+
 void main() {
```

## Running example

As running examples, this page uses the [Hero Editor][toh-pt1] and [Heroes List][toh-pt2] apps from parts 1 and 2 of the [tutorial][], respectively.

You'll first see POs and tests for the [tutorial][]'s simple [Hero Editor][toh-pt1]. Before proceeding, review the [final code][] of the [tutorial part 1][toh-pt1] and take note of the app component template:

[final code]: ../../../tutorial/toh-pt1.md#the-road-youve-travelled

<a id="toh-1-template"></a>

```dart
  template: '''
    <h1>{{title}}</h1>
    <h2>{{hero.name}}</h2>
    <div><label>id: </label>{{hero.id}}</div>
    <div>
      <label>name: </label>
      <input [(ngModel)]="hero.name" placeholder="name">
    </div>
  ''',
```

You can use a single page object for an entire app when it is as simple as the Hero Editor. You might use such a page object to test the title like this:

```dart
  test('title', () {
    expect(appPO.title, 'Tour of Heroes');
  });
```

## PO class

Pageloader recognizes POs that satisfy the following conditions.

- Source file:
  - The file contains a `part` statement referring to the filename but with a `.g.dart` suffix.
  - The file doesn't contain any Kelicap annotations (like `@Component` or `@GenerateInjector`) that would trigger the Kelicap builder. This is a temporary limitation; for details, see [pageloader issue #134](https://github.com/google/pageloader/issues/134).
- PO class (declared in the source file):
  - `@PageObject()` annotates the class.
  - The class is `abstract`.
  - The class has these constructors:
    - A default constructor.
    - A factory constructor defined as shown below.

Here's an example of a valid page object implementation:

```dart
  part 'app_po.g.dart';

  @PageObject()
  abstract class AppPO {

    AppPO();
    factory AppPO.create(PageLoaderElement context) = $AppPO.create;
    // ···
  }
```

During the build process, pageloader generates an implementation for your abstract PO class based on the fields you declare, and saves the implementation to the `*.g.dart` file. The generated code contains factory methods like `$AppPO.create`.

## PO field annotation basics

You can declaratively identify HTML elements that occur in a component's template by adorning PO class getters with [pageloader][] annotations like `@ByTagName('h1')`. For example, an initial version of `AppPO` might look like this:

```dart
  @PageObject()
  abstract class AppPO {

    AppPO();
    factory AppPO.create(PageLoaderElement context) = $AppPO.create;

    @ByTagName('h1')
    PageLoaderElement get _title;
    // ···
    String get title => _title.visibleText;
    // ···
  }
```

Because of its [@ByTagName()](https://pub.dev/documentation/ngpageloader/latest/webdriver/ByTagName-class.html) annotation, the `_h1` field will get bound to the app component [template's `<h1>` element](#toh-1-template).

Other basic tags, which you'll soon see examples of, include:
- [@ByClass()](https://pub.dev/documentation/ngpageloader/latest/webdriver/ByClass-class.html)
- [@ByCss()](https://pub.dev/documentation/ngpageloader/latest/webdriver/ByCss-class.html)
- [@ById()](https://pub.dev/documentation/ngpageloader/latest/webdriver/ById-class.html)
- [@First()](https://pub.dev/documentation/ngpageloader/latest/webdriver/First-class.html)
- [@WithVisibleText()](https://pub.dev/documentation/ngpageloader/latest/webdriver/WithVisibleText-class.html)

The PO `title` field returns the heading element's text.

## PO instantiation

Create PO instances using the PO factory constructor. PO fields are **lazily initialized** from a _context_ passed as an argument to the constructor. Create a context from the fixture's [rootElement][] as shown below. Since most page objects are shared across tests, they are generally initialized during setup:

```dart
  final testBed = NgTestBed<AppComponent>(ng.AppComponentNgFactory);
  late NgTestFixture<AppComponent> fixture;
  late AppPO appPO;

  setUp(() async {
    fixture = await testBed.create();
    final context =
        HtmlPageLoaderElement.createFromElement(fixture.rootElement);
    appPO = AppPO.create(context);
  });
```

## Using POs in tests

When the [Hero Editor][toh-pt1] app loads, it displays data for a hero named _Windstorm_ having id 1. Here's how you might test for this:

```dart
  const windstormData = <String, dynamic>{'id': 1, 'name': 'Windstorm'};

  test('initial hero properties', () {
    expect(appPO.heroId, windstormData['id']);
    expect(appPO.heroName, windstormData['name']);
  });
```

After looking at the app component's [template](#toh-1-template), you might define the PO `heroId` and `heroName` fields like this:

```dart
  abstract class AppPO {
    // ···
    @First(ByCss('div'))
    PageLoaderElement get _id; // e.g. 'id: 1'

    @ByTagName('h2')
    PageLoaderElement get _heroName;
    // ···
    int get heroId {
      final idAsString = _id.visibleText.split(':')[1];
      return int.tryParse(idAsString) ?? -1;
    }

    String get heroName => _heroName.visibleText;
    // ···
  }
```

The page object extracts the id from text that follows the "id:" label in the first `<div>`, and the hero name from the `<h2>` text.

> [!WARNING]
> <h4>PO field bindings are lazily initialized and final</h4>
>
> PO fields are bound when the field is first accessed, based on the state of the fixture's root element. Once bound, they do not change.

## PO _List_ fields

The app from [part 2][toh-pt2] of the [tutorial][] displays a list of heros, generated using an `ngFor` applied to an `<li>` element:

```html
  <h2>Heroes</h2>
  <ul class="heroes">
    <li *ngFor="let hero of heroes"
        [class.selected]="hero == selected"
        (click)="onSelect(hero)">
      <span class="badge">{{hero.id}}</span> {{hero.name}}
    </li>
  </ul>
    <h2>{{selected!.name}}</h2>
```

To define a PO field that collects all generated `<li>` elements, use the annotations introduced [earlier](#po-field-annotation-basics), but declare the field to be of type `List<PageLoaderElement>`:

```dart
  @ByTagName('li')
  List<PageLoaderElement> get _heroes;
```

When bound, the `_heroes` list will contain an element for each `<li>` in the view. If the displayed heroes list is empty, then `_heroes` will be an empty list &mdash; `List<PageLoaderElement>` PO fields are never `null`.

You might render hero data (as a map) from the text of the `<li>` elements like this:

```dart
  Iterable<Map> get heroes =>
      _heroes.map((el) => _heroDataFromLi(el.visibleText));

  // ···
  Map<String, dynamic> _heroDataFromLi(String liText) {
    final matches = RegExp((r'^(\d+) (.*)$')).firstMatch(liText);
    return _heroData(matches?[1], matches?[2]);
  }
```

## PO optional fields

Only once a hero is selected from the [Heroes List][toh-pt2], are the selected hero's details displayed using this template fragment:

```html
  <div *ngIf="selected != null">
    <h2>{{selected!.name}}</h2>
    <div><label>id: </label>{{selected!.id}}</div>
    <div>
      <label>name: </label>
      <input [(ngModel)]="selected!.name" placeholder="name">
    </div>
  </div>
```

Declare an optional field like any other field:

```dart
  @First(ByCss('div div'))
  PageLoaderElement get _heroDetailId;
```

To determine whether an optionally displayed page element is present, test its `exists` property:

```dart
  Map<String, dynamic>? get heroFromDetails {
    if (!_heroDetailId.exists) return null;
    // ···
  }
```

[kelicap_test]: https://pub.dev/packages/kelicap_test
[page object]: https://martinfowler.com/bliki/PageObject.html
[pageloader]: https://pub.dev/packages/pageloader
[rootElement]: https://pub.dev/documentation/kelicap_test/latest/kelicap_test/NgTestFixture/rootElement.html
[separate concerns]: https://en.wikipedia.org/wiki/Separation_of_concerns
[toh-pt1]: ../../../tutorial/toh-pt1.md
[toh-pt2]: ../../../tutorial/toh-pt2.md
[tutorial]: ../../../tutorial/README.md
[simulating a user click]: simulating-user-action.md#click
