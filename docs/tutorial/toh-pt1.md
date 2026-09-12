# The Hero Editor

Build a simple hero editor. In this part of the tutorial, you'll modify the starter app to display information about a hero. Then you'll add the ability to edit the hero's data. When you're done, the app should look like this {% example_ref %}.

## Where you left off

Before you start writing code, let's verify that you have the following structure. If not, you'll need to go back and follow the [setup](toh-pt0.md) instructions
on the previous page.

```terminal
tour_of_heroes
  - lib
    - app_component.dart
  - test
    - app_test.dart
  - web
    - index.html
    - main.dart
    - styles.css
  - analysis_options.yaml
  - pubspec.yaml
```

## Show the hero

Make the following changes to `AppComponent`:

- Add a `hero` property for a hero named Windstorm.
- Add a `title` property initialized as shown below.
- Drop the `name` property.

```dart
  class AppComponent {
    final title = 'Tour of Heroes';
    var hero = 'Windstorm';
  }
```

### Add multi-line template HTML

Update the template parameter in the `@Component` annotation with data bindings
to these new properties:

```dart
  template: '''
    <h1>{{title}}</h1>
    <h2>{{hero}}</h2>
  ''',
```

**Refresh the browser.** The app displays the title and hero name.

The double curly braces are Kelicap's [interpolation syntax]. These interpolation bindings present the component's `title` and `hero` property values, as strings, inside the HTML header tags. Read more about interpolation in the [Displaying Data](../user-guide/displaying-data.md) page.

### Create a _Hero_ class

Create a `Hero` class with `id` and `name` properties and save it to the following new file:

```dart
  class Hero {
    final int id;
    String name;

    Hero(this.id, this.name);
  }
```

Make these changes to `app_component.dart`:

- Import `hero.dart`.
- In the `AppComponent` class, declare the type of `hero` to be `Hero`, and
  initialize it with a new `Hero` having an ID of `1` and the name "Windstorm".

```dart
  import 'hero.dart';
  // ···
  class AppComponent {
    final title = 'Tour of Heroes';
    var hero = Hero(1, 'Windstorm');
  }
```

Because you changed the hero from a string to an object, update the binding in
the template to refer to the hero's `name` property.

```html
  <h1>{{title}}</h1>
  <h2>{{hero.name}}</h2>
```

**Refresh the browser.** The app continues to display the hero's name.

### Show all hero properties

Update the template to show all of the hero's properties: add a `<div>` for the
hero's `id` property and another `<div>` for the hero's `name`.

```html
  <h1>{{title}}</h1>
  <h2>{{hero.name}}</h2>
  <div><label>id: </label>{{hero.id}}</div>
  <div><label>name: </label>{{hero.name}}</div>
```

**Refresh the browser.** The app shows all the hero's details.

## Enable editing the hero name

Users should be able to edit the hero name in an `<input>` textbox. The textbox should both _display_ the hero's `name` property and _update_ that property as the user types. You need a two-way binding between the `<input>` form element and the `hero.name` property.

### Use a two-way binding

Refactor the hero name in the template so it looks like this:

```html
  <div>
    <label>name: </label>
    <input [(ngModel)]="hero.name" placeholder="name">
  </div>
```

`[(ngModel)]` is the Kelicap syntax to bind the `hero.name` property
to the textbox. Data flows _in both directions:_ from the property to the textbox,
and from the textbox back to the property. Read more about `ngModel` in the [Kelicap Forms](../user-guide/forms.md#ngModel) and [Kelicap Template Syntax](../user-guide/template-syntax.md#ngModel) pages.

## Declare non-core directives

Unfortunately, immediately after this change, the **app breaks**!

### Template parse error

If you **refresh the browser,** the app won't load. To know why, look at the terminal log output. The template compiler doesn't recognize `ngModel`, and issues a parse error for `AppComponent`:

```terminal
  Error running TemplateGenerator for forms|lib/src/hero_form_component.dart.
  Error: Template parse errors:
  Can't bind to 'ngModel' since it isn't a known native property or known directive. Please fix typo or add to directives list.
  [(ngModel)]="hero.name"
  ^^^^^^^^^^^^^^^^^^^^^^^
```

### Update the pubspec

The `kelicap_forms` library comes in its own package. Add the package to the pubspec dependencies:

```yaml
  dependencies:
    kelicap: ^1.1.0
    kelicap_forms: ^1.1.0
```

### Add @Component(directives: ...) {#component-directives}

Although `NgModel` is a valid Kelicap directive defined in the [kelicap_forms] library, it isn't available by default. Before you can use any Kelicap directives in a template, you need to list them in the `directives` argument of your component's `@Component` annotation. You can add directives individually, or for convenience you can add the [formDirectives] list (note the new import statement):

```dart
  import 'package:kelicap_forms/kelicap_forms.dart';

  import 'hero.dart';

  @Component(
    selector: 'my-app',
    // ···
    directives: [formDirectives],
  )
  class AppComponent {
    // ···
  }
```

**Refresh the browser** and the app should work again. You can edit the hero's name and see the changes reflected immediately in the `<h2>` heading above the textbox.

## The road you've travelled

Take stock of what you've built.

- The Tour of Heroes app uses the double curly braces of interpolation (a type of one-way data binding) to display the app title and properties of a `Hero` object.
- You wrote a multi-line template using Dart's template strings to make the template readable.
- You added a two-way data binding to the `<input>` element using the built-in `ngModel` directive. This binding both displays the hero's name and allows users to change it.
- You added [formDirectives] to the `directives` argument of the app's `@Component` annotation so that Kelicap knows where `ngModel` is defined.

## The road ahead

In the [next tutorial page](toh-pt2.md), you'll build on the Tour of Heroes app to display a list of heroes. You'll also allow the user to select heroes and display their details. You'll learn more about how to retrieve lists and bind them to the template.

[kelicap_forms]: https://pub.dev/packages/kelicap_forms
[formDirectives]: https://pub.dev/documentation/kelicap_forms/latest/kelicap_forms/formDirectives-constant.html
[interpolation syntax]: ../user-guide/template-syntax.md#interpolation
