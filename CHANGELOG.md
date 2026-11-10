# Change Log

## 1.1.2

### Changes in 1.1.2

* **Format:** All pages are plain GitHub-flavored Markdown. The leftovers of the old AngularDart Jekyll site are gone: `<?code-excerpt?>` lines, Liquid tags and `{{site.*}}` variables, `{% asset %}` images, front matter, and `{#id}` heading IDs. Notes are GitHub callouts (`> [!NOTE]`, `> [!WARNING]`), every code block names its language, and paragraphs are no longer hard-wrapped
* **Code samples:** Imports use the Kelicap packages: `package:kelicap/kelicap.dart`, `package:kelicap_forms/kelicap_forms.dart`, `package:kelicap_router/kelicap_router.dart` and `package:kelicap_test/kelicap_test.dart`, where they used `package:Kelicap/Kelicap.dart` or the `ngdart`, `ngforms`, `ngrouter` and `ngtest` packages. The tutorial's app is imported as `package:tour_of_heroes`. Template interpolation is written `{{name}}`, where it was `{!{name}!}`
* **Testing guide:** The pubspec snippets depend on `kelicap_test: ^1.1.0`, where they used `ngtest: ^4.1.1`, and on the versions of `build_runner`, `build_test`, `build_web_compilers` and `test` that `kelicap_test` uses. The table of contents is a plain list at the top of each page
* **Routing guide:** The overview lists the router features this guide covers, each linked to its page, where it listed the sections of the Angular TypeScript guide
* **AI agents:** Added `AGENTS.md`, with the layout, Markdown format and conventions of these docs, and a `CLAUDE.md` that loads it

### Fixes in 1.1.2

* **Links:** Links between pages are relative links to the `.md` files, where many were links to the old site (`/guide/...`) or had no extension. API links go to the latest docs on pub.dev. Links to a section go to its heading: about 80 pointed to anchors that did not exist
* **Images:** Images of the user guide and tutorial show from the `assets/` folders of these docs
* **Headings:** Headings that had an anchor line such as `<a id="…"></a>` or `<div id="…"></div>` right above them show as headings, where GitHub showed them as plain text. Links to them go to the heading's own anchor
* **Anchors:** Links to a heading with emphasis, such as `## Inside _*ngFor_`, go to its anchor (`#inside-ngfor`). Links whose anchor ended in `.md`, such as `#event-binding-event.md`, go to the section
* **Structural directives guide:** The page names the `<ng-container>` element, where it showed the variable `{{page.ng_container}}`
* **Attribute directives guide:** The "Why not call it highlight?" and "Your directive isn't working?" asides are notes, and the stray `</div>` tags and Material icon tags are gone
* **Code samples:** The samples use the Kelicap API and are null safe:
  * Directives inject the host element as a `package:web` `HTMLElement`, and the dialog service of the routing guide uses `package:web`, where they imported `dart:html`. The fetch pipe of the pipes guide loads data with `package:http`
  * Pipes are plain classes with a `transform` method, where they extended `PipeTransform`, which Kelicap doesn't have
  * Fields that are set after construction are `late`, fields that can be unset (such as an `@Input()` with no default) are nullable, and lists start empty, where they were non-nullable fields with no value. Optional parameters have defaults
  * The `AppComponent` of the attribute directives guide lists only `HighlightDirective` at first, and the page says when to import and add `AutoIdDirective`
  * The component styles guide uses `ViewEncapsulation.none` and `.emulated`, where it used `.None` and `.Emulated`
  * The displaying data guide names `coreDirectives`, where it named `CORE_DIRECTIVES`
  * Code blocks name their language correctly: Dart samples had `html`, and HTML templates had `dart`
* **API links:** Links to `Directive`, `Component.encapsulation`, `ViewEncapsulation`, `coreDirectives` and the `ngpageloader` classes go to their pages on pub.dev, where they returned 404. The testing guide links to the `ngpageloader` package, where it linked to `pageloader`. The `dart:html` links go to `package:web` and `package:http`
* **Setup:** The starter project is the `starter_app` template of `kelicap-templates`, and the clone command works, where it named a `quickstart` branch that does not exist. The page adds `web` as a direct dependency, which the samples import, and gets the dependencies with `dart pub get`, where it ran `dart pub update`, which is not a command
* **Other links:** The security guide links to the Kelicap changelog, and the deployment guide to `ComponentLoader` and the current Dart docs on deferred loading, where they linked to pages that do not exist

### Removed in 1.1.2

* **Live examples:** The sentences that pointed to live examples of the AngularDart site, and the tabs of full example sources, which were never part of these docs
* **HTTP client guide:** The Wikipedia (JSONP) example section, which the old site had hidden and whose code samples were already gone
* **Glossary:** Links to the glossary of the old site, which these docs don't have
* **`json` pipe:** The `JsonPipe` section of the pipes guide, the `| json` examples of the pipes and template syntax guides, and the screenshot of their output, as Kelicap has no `json` pipe
* **Functional directives:** The attribute directives guide shows a stateless directive as a class, `AutoIdDirective`, where it showed a functional directive (a top-level function annotated with `@Directive`), which Kelicap doesn't support

## 1.1.0

* Initial documentation for Kelicap 1.1, ported from the AngularDart site: the [Tutorial](docs/tutorial/README.md), the [User Guide](docs/user-guide/README.md) and the [Deployment Guide](docs/deployment-guide/README.md)
