# Multiple Components

Refactor the master/detail view into separate components. The `AppComponent` is doing _everything_ at the moment. In the beginning, it showed details of a single hero. Then it became a master/detail form with both a list of heroes and the hero detail. Soon there will be new requirements and capabilities. You can't keep piling features on top of features in one component; that's not maintainable.

You'll need to break it up into sub-components, each focused on a specific task or workflow.
Eventually, the `AppComponent` could become a simple shell that hosts those sub-components.

In this page, you'll take the first step in that direction by carving out the hero details into a separate, reusable component. When you're done, the app should look like this {% example_ref %}.

## Where you left off

Before getting started on this page, verify that you have the following structure from earlier in the Tour of Heroes. If not, go back to the previous pages.

```terminal
tour_of_heroes
  - lib
    - app_component.{css,dart,html}
    - src
      - hero.dart
      - mock_heroes.dart
  - test
    - ...
  - web
    - index.html
    - main.dart
    - styles.css
  - analysis_options.yaml
  - pubspec.yaml
```

{% include_relative _keep-app-running.md %}

## Make a hero detail component

Create a file named `hero_component.dart`. This file will hold the new `HeroComponent`.

**Kelicap conventions**:

* The component _class_ name should be written in [upper camel case](/glossary#pascalcase) and end in the word "Component".  The hero detail component class is `HeroComponent`.

* The component _file_ name should be in [snake case](/glossary#snake_case) - lowercase with underscore separation - and end in `_component.dart`. The `HeroComponent` class goes in the `hero_component.dart` file.

* Internal implementation files should be placed under `lib/src`. See the [pub package layout conventions]({{site.dartlang}}/tools/pub/package-layout) for details.

Start writing the `HeroComponent` as follows:

```dart
  import 'package:ngdart/Kelicap.dart';
  import 'package:ngforms/ngforms.dart';

  @Component(
    selector: 'my-hero',
    directives: [coreDirectives, formDirectives],
  )
  class HeroComponent {
  }
```

To define a component, you always import the main Kelicap library.

The `@Component` annotation provides the Kelicap metadata for the component. The CSS selector name, `my-hero`, will match the element tag that identifies this component within a parent component's template. Near the end of this tutorial page you'll add a `<my-hero>` element to the `AppComponent` template.

### Hero detail template

To move the hero detail view to the `HeroComponent`, cut the hero detail _content_ from the bottom of the `AppComponent` template and paste it into a new `template` argument of the `@Component` annotation. The `HeroComponent` has a _hero_, not a _selected hero_. Replace `selected` by `hero`, everywhere in the template. When you're done, the new template should look like this:

```html
  template: '''
    <div *ngIf="hero != null">
      <h2>{!{hero!.name}!}</h2>
      <div><label>id: </label>{!{hero!.id}!}</div>
      <div>
        <label>name: </label>
        <input [(ngModel)]="hero!.name" placeholder="name">
      </div>
    </div>''',
```

In case you're wondering, the  `!` symbol after `hero` is a null-check operator for Dart, one of the language features that help you write more error-proof code. For details, checkout the [null-safety introduction page](https://dart.dev/null-safety).

You also might be confused as to why we still need to declare `hero` as non-null even though it's already wrapped inside the `ngIf` condition that checks whether it's null or not (try removing one of the `!` and recompile!). The answer is that `hero` might be changed outside of the `ngIf` condition. This is very unlikely to occur (and definitely impossible to occur in this situation), but when it does, the little `!` will save you a ton of time debugging.

### Add the _hero_ property

The `HeroComponent` template binds to the component's `hero` property. Add that property, along with the requisite import, to the `HeroComponent` class.

```dart
  import 'hero.dart';

  class HeroComponent {
    Hero? hero;
  }
```

### The _hero_ property is an _input_ property

Later in this page, the parent `AppComponent` will tell the child `HeroComponent` which hero to display by binding its `selected` to the `hero` property of the `HeroComponent`. The binding will look like this:

```dart
  <my-hero [hero]="selected"></my-hero>
```

Putting square brackets around the `hero` property, to the left of the equal sign (=), makes it the _target_ of a property binding expression. You must declare a _target_ binding property to be an _input_ property. Otherwise, Kelicap rejects the binding and throws an error. Declare that `hero` is an _input_ property by annotating it with `@Input()`:

```dart
  @Input()
  Hero? hero;
```

Read more about _input_ properties in the [Attribute Directives](../user-guide/attribute-directives.md#why-input) page.

That's it. The `hero` property is the only thing in the `HeroComponent` class. All it does is receive a hero object through its `hero` input property and then bind to that property with its template.
Here's the complete `HeroComponent`.

```dart
  import 'package:ngdart/Kelicap.dart';
  import 'package:ngforms/ngforms.dart';

  import 'hero.dart';

  @Component(
    selector: 'my-hero',
    template: '''
      <div *ngIf="hero != null">
        <h2>{!{hero!.name}!}</h2>
        <div><label>id: </label>{!{hero!.id}!}</div>
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

## Add _HeroComponent_ to the _AppComponent_

The `AppComponent` is still a master/detail view. It used to display the hero details on its own, before you cut out that portion of the template. Now it will delegate to the `HeroComponent`. Start by importing the `HeroComponent` so `AppComponent` can refer to it.

```dart
  import 'src/hero_component.dart';
```

Recall that `my-hero` is the CSS `selector` in the `HeroComponent` metadata. That's the tag name of the element that represents the `HeroComponent`. Add a `<my-hero>` element near the bottom of the `AppComponent` template, where the hero detail view used to be. Coordinate the master `AppComponent` with the `HeroComponent` by binding the `selected` property of the `AppComponent` to the `hero` property of the `HeroComponent`.

```dart
  <my-hero [hero]="selected"></my-hero>
```

Now every time the `selected` changes, the `HeroComponent` gets a new hero to display. The revised `AppComponent` template should look like this:

```html
  <h1>{!{title}!}</h1>
  <h2>Heroes</h2>
  <ul class="heroes">
    <li *ngFor="let hero of heroes"
        [class.selected]="hero == selected"
        (click)="onSelect(hero)">
      <span class="badge">{!{hero.id}!}</span> {!{hero.name}!}
    </li>
  </ul>
  <my-hero [hero]="selected"></my-hero>
```

The detail _should_ update every time the user picks a new hero.  It's not happening yet!  Click a hero. No details. If you look for an error in the console of the browser development tools. No error. It is as if Kelicap were ignoring the new tag. That's because _it is ignoring the new tag_.

### The _directives_ list

A browser ignores HTML tags and attributes that it doesn't recognize. So does Kelicap. You've imported `HeroComponent`, and you've used `<my-hero>` in the template, but you haven't told Kelicap about it.

Just as you've done for the built-in Kelicap directives, tell Kelicap about the hero detail component by listing it in the metadata `directives` list. You don't need `formDirectives` anymore, so delete it and the `Kelicap_forms` import at the top of the file:

```dart
  directives: [coreDirectives, HeroComponent],
```

 **Refresh the browser.** The app works!

## App design changes

As [before](toh-pt2.md), whenever a user clicks on a hero name, the hero detail appears below the hero list. But now the `HeroComponent` is presenting those details. Refactoring the original `AppComponent` into two components yields benefits, both now and in the future:

1. You simplified the `AppComponent` by reducing its responsibilities.
2. You can evolve the `HeroComponent` into a rich hero editor
   without touching the parent `AppComponent`.
3. You can evolve the `AppComponent` without touching the hero detail view.
4. You can reuse the `HeroComponent` in the template of some future parent component.

### Review the app structure

Verify that you have the following structure:

```terminal
tour_of_heroes
  - lib
    - app_component.{css,dart,html}
    - src
      - hero.dart
      - hero_component.dart
      - mock_heroes.dart
  - test
    - ...
  - web
    - index.html
    - main.dart
    - styles.css
  - analysis_options.yaml
  - pubspec.yaml
```

## The road you’ve travelled

Here's what you achieved in this page:

* You created a reusable component.
* You learned how to make a component accept input.
* You learned to declare the app directives in a `directives` list.
* You learned to bind a parent component to a child component.

Your app should look like this {% example_ref %}.

## The road ahead

The Tour of Heroes app is more reusable with shared components, but its (mock) data is still hard coded within the `AppComponent`. That's not sustainable. Data access should be refactored to a separate service and shared among the components that need data.

You’ll learn to create services in the [next tutorial](toh-pt4.md) page.
