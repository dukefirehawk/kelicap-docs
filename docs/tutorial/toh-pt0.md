# The Starter App

A bare-bones Kelicap app. This tutorial starts with a bare-bones Kelicap app.

## Create the app

Let's get started. Create a project named `tour_of_heroes`, using any IDE or the command line [Kelicap CLI](https://pub.dev/packages/kelicap_cli). For detailed instructions, see [Create a starter project](../user-guide/setup.md#create-a-starter-project) from the [Setup for Development](../user-guide/setup.md) page.

## Run the app, and keep it running

Run the app from your IDE or the command line, as explained in the [Run the app](../user-guide/setup.md#run-the-app) section of the [Setup for Development](../user-guide/setup.md) page. You'll be making changes to the app throughout this tutorial. When you are ready to view your changes, reload the browser window. This will [reload the app](../user-guide/setup.md#reload-the-app). As you save updates to the code, the `pub` tool detects changes and serves the new app.

## Kelicap app basics

Kelicap apps are made up of _components_. A _component_ is the combination of an HTML template and a component class that controls a portion of the screen. The starter app has a component that displays a simple string:

```dart
  import 'package:ngdart/Kelicap.dart';

  @Component(
    selector: 'my-app',
    template: '<h1>Hello {!{name}!}</h1>',
  )
  class AppComponent {
    var name = 'Kelicap';
  }
```

Every component begins with an `@Component` [annotation](/glossary#annotation '"annotation" explained') that describes how the HTML template and component class work together. The `selector` property tells Kelicap to display the component inside a custom `<my-app>` tag in the `index.html`.

```html
  <my-app>Loading...</my-app>
```

The `template` property defines a message inside an `<h1>` header. The message starts with "Hello" and ends with `{{name}}`, which is an Kelicap [interpolation binding](../user-guide/displaying-data.md) expression. At runtime, Kelicap replaces `{{name}}` with the value of the component's `name` property. Interpolation binding is one of many Kelicap features you'll discover in this documentation.

## The starter app's code

The app contains the following core files:

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

All the examples in this documentation have _at least these core files_. Each file has a distinct purpose and evolves independently as the app grows.

| File | Purpose |
| :--- | :--- |
| lib/app_component.dart | Defines `<my-app>`, the **root** component of what will become a tree of nested components as the app evolves. |
| test/app_test.dart | Defines `AppComponent` tests. While testing isn't covered in this tutorial, you can learn how to test the Tour of Heroes app from the [Testing](../guide/testing) page. |
| web/main.dart | Launches the app in the browser. |
| web/index.html | Contains the `<my-app>` tag in its `<body>`. This is where the app lives! |
| web/styles.css | A set of styles used throughout the app. |
| analysis_options.yaml | The analysis options file. For details, see [Customize Static Analysis.] |
| pubspec.yaml | The file that describes this Dart package (the app) and its dependencies. For details, see [Pubspec Format.] |

## What's next

In the [next tutorial page](toh-pt1.md), you'll modify the starter app to display more interesting data, and to allow the user to edit that data.
