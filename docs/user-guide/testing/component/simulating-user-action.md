# Component Testing: Simulating user action

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

## Click

Use the [PageLoaderElement.click()][] method to simulate a user click on a given PO element. Here is an example for a _back_ button:

```dart
  @ByTagName('button')
  PageLoaderElement get _button;
  // ···
  Future<void> back() => _button.click();
```

Similarly, you might define a PO method for selecting a hero from a list as follows:

```dart
  @ByTagName('li')
  List<PageLoaderElement> get _heroes;
  // ···
  Future<void> selectHero(int index) => _heroes[index].click();
```

## Input: add text

The [Hero Editor][toh-pt1] allows a user to edit a hero's name by means of an `<input>` element. Use the [PageLoaderElement.type()][] method to simulate adding text to the input element:

```dart
  @ByTagName('input')
  PageLoaderElement get _input;
  // ···
  Future<void> type(String s) => _input.type(s);
```

Here is an example of how the `type()` method might be used to update a hero's name:

```dart
  const nameSuffix = 'X';

  test('update hero name', () async {
    await appPO.type(nameSuffix);
    expect(appPO.heroId, windstormData['id']);
    expect(appPO.heroName, windstormData['name'] + nameSuffix);
  });
```

## Input: clear

You can clear an input using the [PageLoaderElement.clear()][] method:

```dart
  Future<void> clear() => _input.clear();
```

Here is an example of a PO method for adding a new hero. It makes use of both `clear()` and `type()`:

```dart
  Future<void> addHero(String name) async {
    await _input.clear();
    await _input.type(name);
    await _add.click();
  }
```

[PageLoaderElement.clear()]: https://pub.dev/documentation/pageloader/latest/webdriver/PageLoaderElement/clear.html
[PageLoaderElement.click()]: https://pub.dev/documentation/pageloader/latest/webdriver/PageLoaderElement/click.html
[PageLoaderElement]: https://pub.dev/documentation/pageloader/latest/webdriver/PageLoaderElement-class.html
[PageLoaderElement.type()]: https://pub.dev/documentation/pageloader/latest/webdriver/PageLoaderElement/type.html
[toh-pt1]: ../../../tutorial/toh-pt1.md
