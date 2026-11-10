# Component Testing: Basics

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

Ready to write tests for your own project? This page explains how to setup your project, and it illustrates how to write basic component tests.

## Pubspec configuration

Projects with component tests depend on the [kelicap_test][], [build_test][], and [test][] packages. Because these packages are used only during development, and not shipped with the app, they don't belong under `dependencies` in the [pubspec][]. Instead add them under **`dev_dependencies`**, for example:

```yaml
  dev_dependencies:
    kelicap_test: ^1.1.0
    build_runner: ^2.14.0
    build_test: ^3.0.0
    build_web_compilers: ^4.6.0
    test: ^1.31.0
```

## API basics: _test_() and _expect_()

Write component tests using the [test][] package API. For example, define tests using `test()` functions containing `expect()` test assertions:

```dart
  test('Default greeting', () {
    expect(fixture.text, 'Hello Kelicap');
  });
```

You can also use other [test][] package features, like [group()][], to write your component tests.

## Test fixture, setup and teardown

Component tests must explicitly define the **component under test**. You define it by passing the component class name as a generic type argument to the [NgTestBed][] and [NgTestFixture][] classes:

```dart
  import 'package:tour_of_heroes/app_component.template.dart' as ng;
  // ···
  void main() {
    final testBed = NgTestBed<AppComponent>(ng.AppComponentNgFactory);
    late NgTestFixture<AppComponent> fixture;
    // ···
  }
```

You'll generally initialize the fixture in a `setUp()` function. Since component tests are often asynchronous, the `tearDown()` function generally instructs the test framework to dispose of any running tests before it moves on to the next test group, if any. Here is an example:

```dart
  @TestOn('browser')

  import 'package:kelicap_test/kelicap_test.dart';
  import 'package:tour_of_heroes/app_component.dart';
  import 'package:tour_of_heroes/app_component.template.dart' as ng;
  import 'package:test/test.dart';

  void main() {
    final testBed = NgTestBed<AppComponent>(ng.AppComponentNgFactory);
    late NgTestFixture<AppComponent> fixture;

    setUp(() async {
      fixture = await testBed.create();
    });

    tearDown(disposeAnyRunningTest);

    test('Default greeting', () {
      expect(fixture.text, 'Hello Kelicap');
    });
    // ···
  }
```

Use [setUpAll()][] and [tearDownAll()][] for setup/teardown that should encompass all tests and be performed only once for the entire test suite.

## More _package:test_ features

You've just worked through the elementary features of `package:test`. For complete coverage of the package capabilities, see the [package test documentation.][test]

[kelicap_test]: https://pub.dev/packages/kelicap_test
[build_test]: https://pub.dev/packages/build_test
[group()]: https://pub.dev/packages/test#writing-tests
[group API]: https://pub.dev/documentation/test/latest/test/group.html
[NgTestBed]: https://pub.dev/documentation/kelicap_test/latest/kelicap_test/NgTestBed-class.html
[NgTestFixture]: https://pub.dev/documentation/kelicap_test/latest/kelicap_test/NgTestFixture-class.html
[pubspec]: https://dart.dev/tools/pub/pubspec
[setUpAll()]: https://pub.dev/documentation/test_core/latest/test_core/setUpAll.html
[tearDownAll()]: https://pub.dev/documentation/test/latest/test/tearDownAll.html
[test]: https://pub.dev/packages/test
