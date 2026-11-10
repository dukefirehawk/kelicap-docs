# AGENTS.md — Kelicap Docs

Guidance for AI coding agents working in this repository.

## What this repo is

User-facing documentation for the [Kelicap](https://github.com/dukefirehawk/kelicap) web framework. Kelicap is a fork of AngularDart (ngdart) for Dart 3.13+ that uses `package:web` for all DOM access. The framework source lives in a separate repo (`../kelicap` in a typical workspace). Check API names, package names and behavior against that source, not against Angular or AngularDart docs.

This repo has no build, no site generator and no Dart package. The docs are plain GitHub-flavored Markdown, read on GitHub.

## Layout

```text
README.md                  Top-level table of contents and docs version
AUTHORS.md                 Maintainers
assets/branding/           Logos and banners
docs/
  tutorial/                Tour of Heroes tutorial (toh-pt0.md … toh-pt6.md)
    assets/toh/            Tutorial screenshots
  user-guide/              Framework guide, one topic per file
    router/                Routing guide, split into numbered parts (1.md … 6.md)
    testing/               component/ and e2e/ testing guides
    assets/<topic>/        Images for the page of the same name
  deployment-guide/        Deployment docs
```

Every section has a `README.md` holding its table of contents. When you add, rename or remove a page, update that section's README (and the root README if the section list changes).

## Markdown format

The content was ported from the old AngularDart Jekyll site (`site-angulardart`) and has been converted to plain GitHub-flavored Markdown. Don't reintroduce the old site's syntax, and don't copy it in from Angular or AngularDart sources:

| Don't use | Use |
| --- | --- |
| `<?code-excerpt "…"?>`, `<code-tabs>`/`<?code-pane?>` | The code itself, in a fenced block |
| Bare ` ``` ` fences | Fences with a language: `dart`, `html`, `css`, `yaml`, `bash`, `text` |
| `[!…!]` highlight markers in code | Plain code; point out the important lines in the prose |
| `{!{name}!}` | `{{name}}` (fenced code needs no escaping) |
| Liquid tags: `{% … %}`, `{{site.…}}` variables | Plain Markdown and full URLs |
| `<img src="{% asset … %}">` | `![Alt text](assets/<topic>/x.png)`, with the image committed under `assets/` |
| `<div class="l-sub-section">`, `<div class="alert …">` | GitHub callouts: `> [!NOTE]` or `> [!WARNING]` (plain `>` blockquotes inside lists) |
| Site-absolute links (`/guide/setup`), links without `.md` | Relative links to the file: `[setup](setup.md)`, `[forms](../user-guide/forms.md#anchor)` |
| `## Heading {#id}` and `<a id>` anchors | GitHub's generated heading anchors (lowercase, spaces to `-`, punctuation dropped) |
| Jekyll front matter (`title:`, `prevpage:`) | A `# Title` heading |
| Prose hard-wrapped across many lines | One line per paragraph |

API links point to the latest docs on pub.dev, e.g. `https://pub.dev/documentation/kelicap/latest/kelicap/NgFor-class.html` or `https://pub.dev/documentation/kelicap_router/latest/kelicap_router/Router-class.html`. Package and library names in these URLs are lowercase.

## Content conventions

- **Product name:** "Kelicap", capitalized in prose. Replace "Angular"/"AngularDart" with "Kelicap", except where the text is really about Angular's history or the TypeScript framework (e.g. `intro.md`, `learning-kelicap.md`).
- **Package names in code are lowercase.** Use the real packages from the framework repo:

  | Use | Not |
  | --- | --- |
  | `package:kelicap/kelicap.dart` | `package:Kelicap/Kelicap.dart`, `package:kelicap/Kelicap.dart`, `package:ngdart/…` |
  | `package:kelicap_router/kelicap_router.dart` | `package:ngrouter/…` |
  | `package:kelicap_forms/kelicap_forms.dart` | `package:ngforms/…` |
  | `package:kelicap_test/kelicap_test.dart` | `package:ngtest/…` |

  `package:ngpageloader` (e2e testing) has no Kelicap equivalent yet. Leave it unless the maintainer says otherwise. Example app package names should be lowercase snake_case (`tour_of_heroes`, not `Kelicap_tour_of_heroes`).
- **Code samples** should be valid for Dart 3 (null safety, `package:web` rather than `dart:html`) and match the current framework API. If you're unsure whether an API still exists, grep the framework repo.
- **Images** go in `assets/<topic>/` next to the page that uses them. Use descriptive alt text. Before referencing an image, check the file is there.
- **Tone:** second person ("you"), present tense, short and direct. Keep headings in Title Case to match the existing pages.

## Checking your changes

There's no build or CI. Before finishing:

- Check that every relative link and image path you added or changed points to a file that exists.
- Search the pages you edited for old site syntax and wrong imports: `grep -nE '\{%|\{\{ ?(site|page)\.|\{ ?#|<\?code|\{!\{|l-sub-section|dart:html|package:(ng|Kelicap)' <files>`.
- Preview the Markdown (GitHub, or your editor's preview) when you change tables, nested lists or code fences.

## Git

- Work on a topic branch (`docs/<name>`, `feature/<name>`, `fix/<name>`) and merge it into `main` with a pull request.
- Commit messages use the conventional prefix `docs:`, e.g. `docs: convert pipes guide to plain markdown`.
- Update the version in `README.md` only when the maintainer asks.
