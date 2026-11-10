# Appendices

The balance of this guide is a set of appendices that elaborate some of the points covered quickly above.

The appendix material isn't essential. Continued reading is for the curious.

## Appendix: Link parameters list

A link parameters list holds the ingredients for router navigation:

* the *path* of the route to the destination component
* required and optional route parameters that go into the route URL

You can bind the `RouterLink` directive to such an list like this:

```html
  <a [routerLink]="RoutePaths.heroes.toUrl()"
     [routerLinkActive]="'active-route'">Heroes</a>
```

You've written a two element list when specifying a route parameter to the `navigate` method:

```dart
  Future<NavigationResult> _gotoDetail(int id) =>
      _router.navigate(_heroUrl(id));
```

These examples cover the need for an app with one level routing. The moment you add a child router, such as the crisis center, you create new link parameters list possibilities.

Recall that you specified a default child route for crisis center so this simple `RouterLink` is fine.

```dart
  [routerLinkActive]="'active-route'">Crisis Center</a>
```

Parse it out:

* The first item in the list identifies the parent route (`CrisisCenter`) whose path is `/crises/...`.
* There are no parameters for this parent route so you're done with it.
* The default child route is `Crises`, which has path `/`.
* The resulting path is `/crises/`.

Take it a step further. Consider the following router link that navigates from the root of the app down to the *Dragon Crisis*:

```dart
  // FIXME: This is out-of-date
  <a [routerLink]="Routes.crises.toUrl()"
     [routerLinkActive]="'active-route'">Crisis Center</a>
  <a [routerLink]="['CrisisCenter', 'Crisis Center', 'CrisisDetail', {'id': '1'}]">Dragon Crisis</a>
```

* The first item in the list identifies the parent route (`CrisisCenter`) whose path is `/crises`.
* There are no parameters for this parent route so we're done with it.
* The second item identifies the child route for details about a particular crisis ('CrisisDetail') whose path is `/:id`.
* The third and final item provides the `id` of the *Dragon Crisis* (`{'id': '1'}`).
* The resulting path is `/crises/1`.

In summary, you can write apps with one, two or more levels of routing. The link parameters list affords the flexibility to represent any routing depth and any legal sequence of route names and route parameter values.

## Appendix: *LocationStrategy* and browser URL styles

When the router navigates to a new component view, it updates the browser's location and history with a URL for that view. This is a strictly local URL. The browser shouldn't send this URL to the server and should not reload the page.

Modern HTML 5 browsers support [history.pushState](https://developer.mozilla.org/en-US/docs/Web/API/History_API), a technique that changes a browser's location and history without triggering a server page request. The router can compose a "natural" URL that is indistinguishable from one that would otherwise require a page load.

Here's the *Crisis Center* URL in this "HTML 5 pushState" style:

```text
localhost:3002/crises/
```

Older browsers send page requests to the server when the location URL changes ... unless the change occurs after a "#" (called the "hash"). Routers can take advantage of this exception by composing in-app route URLs with hashes.  Here's a "hash URL" that routes to the *Crisis Center*

```text
localhost:3002/src/#/crises/
```

The router supports both styles with two `LocationStrategy` providers:

1. `PathLocationStrategy` - the default "HTML 5 pushState" style.
1. `HashLocationStrategy` - the "hash URL" style.

The [routerProviders][] list sets the `LocationStrategy` to the `PathLocationStrategy`. Use [routerProvidersHash][] instead if you want your app to use hash URLs.

> [!NOTE]
> Learn about providers and the app-launching process in the [Dependency Injection guide](../dependency-injection.md#root-injector-providers)

### Which strategy is best?

You must choose a strategy and you need to make the right call early in the project. It won't be easy to change later once the app is in production and there are lots of app URL references in the wild.

Almost all kelicap projects should use the default HTML 5 style. It produces URLs that are easier for users to understand. And it preserves the option to do _server-side rendering_ later.

Rendering critical pages on the server is a technique that can greatly improve perceived responsiveness when the app first loads. An app that would otherwise take ten or more seconds to start could be rendered on the server and delivered to the user's device in less than a second.

This option is only available if app URLs look like normal web URLs without hashes (#) in the middle.

Stick with the default unless you have a compelling reason to resort to hash routes.

### HTML 5 URLs and the  *&lt;base href>*

While the router uses the [HTML 5 pushState](https://developer.mozilla.org/en-US/docs/Web/API/History_API) style by default, you *must* configure that strategy with a **base href**

The preferred way to configure the strategy is to add a [&lt;base href> element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/base) tag in the `<head>` of the `index.html`.

```html
  <script>
    // WARNING: DO NOT set the <base href> like this in production!
    // Details: https://github.com/dukefirehawk/kelicap-docs/blob/main/docs/user-guide/router/appendices.md
    (function () {
      var m = document.location.pathname.match(/^(\/[-\w]+)+\/web($|\/)/);
      document.write('<base href="' + (m ? m[0] : '/') + '" />');
    }());
  </script>
```

Without that tag, the browser may not be able to load resources (images, css, scripts) when "deep linking" into the app. Bad things could happen when someone pastes an app link into the browser's address bar or clicks such a link in an email link.

Some developers may not be able to add the `<base>` element, perhaps because they don't have access to `<head>` or the `index.html`.

Those developers may still use HTML 5 URLs by taking two remedial steps:

1. Provide the router with an appropriate [appBaseHref][] value.
1. Use _root URLs_ for all web resources: css, images, scripts, and template html files.

[appBaseHref]: https://pub.dev/documentation/kelicap_router/latest/kelicap_router/appBaseHref-constant.html
[routerProviders]: https://pub.dev/documentation/kelicap_router/latest/kelicap_router/routerProviders-constant.html
[routerProvidersHash]: https://pub.dev/documentation/kelicap_router/latest/kelicap_router/routerProvidersHash-constant.html
