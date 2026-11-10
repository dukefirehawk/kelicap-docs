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

### Removed in 1.1.2

* **Live examples:** The sentences that pointed to live examples of the AngularDart site, and the tabs of full example sources, which were never part of these docs
* **HTTP client guide:** The Wikipedia (JSONP) example section, which the old site had hidden and whose code samples were already gone
* **Glossary:** Links to the glossary of the old site, which these docs don't have

## 1.1.0

* Initial documentation for Kelicap 1.1, ported from the AngularDart site: the [Tutorial](docs/tutorial/README.md), the [User Guide](docs/user-guide/README.md) and the [Deployment Guide](docs/deployment-guide/README.md)
