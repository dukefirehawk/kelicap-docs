# Routing Overview

Overview of core router features

The Kelicap **router** enables navigation from one view to the next as users perform app tasks. This guide covers the router's primary features, illustrating them through the evolution of a small app that you can run live.

## Overview

The browser is a familiar model of app navigation:

- Enter a URL in the address bar and the browser navigates to a corresponding page.
- Click links on the page and the browser navigates to a new page.
- Click the browser's back and forward buttons and the browser navigates backward and forward through the history of pages you've seen.

The Kelicap router borrows from this model. It can interpret a browser URL as an instruction to navigate to a client-generated view. It can pass optional parameters along to the supporting view component that help it decide what specific content to present. You can bind the router to links on a page and it will navigate to the appropriate app view when the user clicks a link. You can navigate imperatively when the user clicks a button, selects from a drop box, or in response to some other stimulus from any source. And the router logs activity in the browser's history journal so the back and forward buttons work as well.

You'll learn many router details in this guide, which covers:

- Setting the [base href](1.md#set-the-base-href)
- Importing from the [router library](#add-kelicap_router)
- [Configuring the router](1.md#routes)
- Handling unmatched URLs with a [wildcard route](2.md#wildcard-route)
- The [link parameters list](appendices.md#appendix-link-parameters-list) that propels router navigation
- Setting the [default route](2.md#default-route) where the app navigates at launch
- [Redirecting](2.md#redirect-route) from one route to another
- Navigating when the user clicks a data-bound [RouterLink](1.md#routerlinks)
- Navigating under [program control](3.md#navigating-imperatively)
- Embedding critical information in the URL with [route parameters](3.md#route-definition-with-a-parameter)
- Creating [child and relative routes](4.md)
- Toggling CSS classes for the [active router link](4.md#show-selected-crisis-as-active)
- Setting a [default child route](4.md#crisis-home-route)
- Confirming or canceling navigation with [router lifecycle hooks](5.md)
- Providing optional information across routes with [query parameters](5.md#query-parameters-and-fragments)
- Loading feature areas [asynchronously](6.md)
- Choosing the "HTML5" or "hash" [URL style](appendices.md#appendix-locationstrategy-and-browser-url-styles)

## Setup overview

### Add kelicap_router

Router functionality is in the [kelicap_router][] library, which comes in [its own package.][kelicap_router@pub] Add the package to the pubspec dependencies:

```yaml
  dependencies:
    kelicap: ^1.1.0
    # ···
    kelicap_router: ^1.1.0
```

In any Dart file that makes use of router features, import the router library:

```dart
  import 'package:kelicap_router/kelicap_router.dart';
```

### Register providers and list directives

If you're already familiar with Kelicap routing, here's a reminder of what you need to do:

- Choose a [location strategy](appendices.md#which-strategy-is-best).
- [Register appropriate router providers][router providers] when launching your app.
- Ensure that each routing component has metadata listing the [router directives](1.md#appcomponent-navigation) used by the component.

[router providers]: 1.md#add-router-providers

## Basic feature overview

This guide proceeds in phases, marked by milestones, starting from a skeletal app and building toward a modular, multi-view design with child routes. This overview of core router concepts will help orient you to the details that follow.

### \<base href>

Most routing apps have a `<base href="...">` element in the `index.html` `<head>` to tell the router how to compose navigation URLs. For details, see [Set the *base href*](1.md#set-the-base-href).

### Routes

[Routes](1.md#routes) tell the router which views to display when a user clicks a link or pastes a URL into the browser address bar. To configure routes you'll need to do the following:

Define [route paths](1.md#route-paths):

```dart
  import 'package:kelicap_router/kelicap_router.dart';

  class RoutePaths {
    static final heroes = RoutePath(path: 'heroes');
  }
```

Define [route definitions](1.md#route-definitions):

```dart
  import 'package:kelicap_router/kelicap_router.dart';

  import 'route_paths.dart';
  import 'hero_list_component.template.dart' as hero_list_template;

  export 'route_paths.dart';

  class Routes {
    static final heroes = RouteDefinition(
      routePath: RoutePaths.heroes,
      component: hero_list_template.HeroListComponentNgFactory,
    );

    static final all = <RouteDefinition>[
      heroes,
    ];

    static final heroRoute = <RouteDefinition>[hero];
  }
```

Bind the route definitions to a *router outlet*, as illustrated next.

### Router outlet

When you visit [localhost:8080/#/heroes](http://localhost:8080/#/heroes){:.no-automatic-external}, the router matches the URL with the heroes route path, and displays a `HeroListComponent` immediately below the `<router-outlet>` in the routing component template:

```dart
  import 'src/routes.dart';

  @Component(
    template: '''
      ...
      <router-outlet [routes]="Routes.all"></router-outlet>
    ''',
    directives: [routerDirectives],
    exports: [RoutePaths, Routes],
  )
  class AppComponent {
  }
```

For details, see [RouterOutlet](1.md#routeroutlet).

### Router links

A [RouterLink][] directive on an anchor tag gives the router control over the anchor. Bind each `RouterLink` directive to a template expression that evaluates to a URL.

```dart
  template: '''
    <h1>Kelicap Router</h1>
    <nav>
      <a [routerLink]="RoutePaths.crises.toUrl()"
         [routerLinkActive]="'active-route'">Crisis Center</a>
      <a [routerLink]="RoutePaths.heroes.toUrl()"
         [routerLinkActive]="'active-route'">Heroes</a>
    </nav>
    <router-outlet [routes]="Routes.all"></router-outlet>
  ''',
  styles: ['.active-route {color: #039be5}'],
```

A [RouterLinkActive][] will apply the named CSS class to the anchor whose link is active. This helps visually distinguish the active links.

For details, see [RouterLinks](1.md#routerlinks).

## Summary

Here are the key router terms and their meanings.

| **Router Part** | **Meaning** |
| [Router][] | Displays the app component for the active URL. Manages navigation from one component to the next. |
| Routing component | An Kelicap component with a `<router-outlet>` that displays views based on router navigations. |
| Route | A kind of `RouteDefinition`. Defines how the router should navigate to a component based on a URL pattern. Most routes consist of a path, a route name, and a component type. |
| [RouteDefinition][] | Defines how the router should navigate to a component based on a URL pattern. |
| [RouterOutlet][] | The directive (`<router-outlet>`) that marks where the router should display a views. |
| [RouterLink][] | The directive for binding a clickable HTML element to a route. Clicking an element with a `routerLink` directive triggers a navigation. |

## What next?

The rest of this guide describes the development of a multi-page routed app through a sequence of milestones. Each milestone highlights specific design decisions and introduces new key features of the router.

[kelicap_router]: https://pub.dev/documentation/kelicap_router/latest/kelicap_router/kelicap_router-library.html
[kelicap_router@pub]: https://pub.dev/packages/kelicap_router
[RouteDefinition]: https://pub.dev/documentation/kelicap_router/latest/kelicap_router/RouteDefinition-class.html
[Router]: https://pub.dev/documentation/kelicap_router/latest/kelicap_router/Router-class.html
[RouterLink]: https://pub.dev/documentation/kelicap_router/latest/kelicap_router/RouterLink-class.html
[RouterLinkActive]: https://pub.dev/documentation/kelicap_router/latest/kelicap_router/RouterLinkActive-class.html
[RouterOutlet]: https://pub.dev/documentation/kelicap_router/latest/kelicap_router/RouterOutlet-class.html
