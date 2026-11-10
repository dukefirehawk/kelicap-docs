# Kelicap — Documentation

[![Kelicap Web Framework](assets/branding/kelicap_1_banner.jpeg)](https://github.com/dukefirehawk/kelicap)

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](LICENSE)

Documentation, tutorial and user guides for the [Kelicap](https://github.com/dukefirehawk/kelicap) web framework.

Kelicap is a lightweight web framework for modern Dart (3.13+) that renders native HTML. It is a fork of AngularDart, uses [`package:web`](https://pub.dev/packages/web) for all DOM access, and keeps a syntax close to Angular's.

**Version:** 1.1.2

## 📖 Table of Contents

### [Tutorial: Tour of Heroes](docs/tutorial/README.md)

Build a small app step by step, from a starter project to a routed app that talks to a server.

1. [The Starter App](docs/tutorial/toh-pt0.md)
2. [The Hero Editor](docs/tutorial/toh-pt1.md)
3. [Master/Detail](docs/tutorial/toh-pt2.md)
4. [Multiple Components](docs/tutorial/toh-pt3.md)
5. [Services](docs/tutorial/toh-pt4.md)
6. [Routing](docs/tutorial/toh-pt5.md)
7. [HTTP](docs/tutorial/toh-pt6.md)

### [User Guide](docs/user-guide/README.md)

How each part of the framework works, from installation and architecture to templates, dependency injection, forms, routing and testing. 

### [Deployment Guide](docs/deployment-guide/README.md)

How to build a Kelicap app for production and deploy it.

## Related projects

- [Kelicap](https://github.com/dukefirehawk/kelicap): the framework, published on pub.dev as [`kelicap`](https://pub.dev/packages/kelicap), [`kelicap_router`](https://pub.dev/packages/kelicap_router), [`kelicap_forms`](https://pub.dev/packages/kelicap_forms) and [`kelicap_test`](https://pub.dev/packages/kelicap_test)
- [Kelicap Components](https://github.com/dukefirehawk/kelicap_components): Material Design 2 components for Kelicap, published as [`kelicap_components`](https://pub.dev/packages/kelicap_components)

## Contributing

The docs are plain GitHub-flavored Markdown with no build step. Work on a topic branch and open a pull request against `main`. [AGENTS.md](AGENTS.md) describes the layout, the Markdown format and the conventions for code samples. Record user-facing changes in [CHANGELOG.md](CHANGELOG.md).

## License

This documentation is licensed under the [Creative Commons Attribution 4.0 International License](LICENSE) (CC BY 4.0). Maintainers are listed in [AUTHORS.md](AUTHORS.md).
