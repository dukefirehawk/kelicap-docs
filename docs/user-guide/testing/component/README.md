# Component Testing

This part of the [Testing](../README.md) how-to guide covers [component testing][] of KelicapDart apps using the following packages:

[package:test][] : Dart's standard [testing][] package, which is similar to most [unit testing][] frameworks, including [Jasmine][] for JavaScript. If you've used a modern testing framework before, then writing tests using `package:test` will have a familiar feel. Testing basics is covered in [Writing component tests: Basics](basics.md). For in-depth coverage of the package capabilities, see the `package:test` [documentation][package:test].

[package:kelicap_test][] : A complementary package that provides a means of interacting with Kelicap components created as test fixtures.

[package:mockito][] : A library for creating [mock objects](https://en.wikipedia.org/wiki/Mock_object). Not all component tests require mock objects, but in later sections, you'll learn the fundamentals of using `package:mockito` as needed. For further details, see the package [documentation][package:mockito].

## Scope

The [kelicap_test][] package is suitable for testing either a single component, or a _small_ group of related components. This package is not meant to test an entire app. For that, you need to write [end-to-end tests](../e2e/README.md).

## Topics

This guide covers the following component testing topics:

- [Component testing](README.md)
- [Running component tests](running-tests.md)
- Writing component tests
  - [Basics](basics.md): pubspec config, test API fundamentals
  - [Page objects](page-objects.md): field annotation, initialization, and more
  - [Simulating user action](simulating-user-action.md): click, type, clear
  - [Services](services.md): local, external, mock, real
  - [`@Input()` and `@Output()`](input-and-output.md)
  - [Routing components](routing-components.md)

[Jasmine]: https://jasmine.github.io
[kelicap_test]: https://pub.dev/packages/kelicap_test
[component testing]: https://en.wikipedia.org/wiki/Software_testing
[package:kelicap_test]: https://pub.dev/packages/kelicap_test
[package:mockito]: https://pub.dev/packages/mockito
[package:test]: https://pub.dev/packages/test
[testing]: https://dart.dev/guides/testing
[unit testing]: https://en.wikipedia.org/wiki/Unit_testing
