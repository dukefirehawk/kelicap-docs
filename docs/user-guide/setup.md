# Setup for Development

Setting up a new Kelicap project is straightforward using common Dart tools.This page leads you through getting and running the starter app that's featured in this guide and tutorial.

## Get prerequisites

For information on how to get the Dart SDK,
see [the Dart site]({{site.dartlang}}/get-dart):

- **Dart SDK** {{site.data.pkg-vers.SDK.vers}} or a compatible version. For
  KelicapDart, SDK >= 3.13.0 is required.
- Your favorite IDE & code editor, such as WebStorm, Eclipse, or **Visual Studio Code** (recommended).

## Install Command-line Tools

### kelicap_cli (recommended)

[`kelicap_cli`]({{site.pub-pkg}}/kelicap_cli) is a community-maintained command-line tool for creating and managing KelicapDart applications. KelicapDart can still run without it, but `kelicap_cli` makes your life much easier by providing some handy commands such as `kelicap create <project_name>` to create a new project and `kelicap clean` to clean all build artifacts and cache, similar to `flutter clean` if you come from Flutter. For more info on `kelicap_cli`, see [`kelicap_cli`'s Pub page]({{site.pub-pkg}}/kelicap_cli).

Run the following to install `kelicap_cli`:

```bash
  dart pub global activate kelicap_cli
```

## Create a starter project

For generic purposes, use `kelicap create <app_name>` to create a new Kelicap application. If you want to follow the examples in this guide and tutorial, they are based on the [Quickstart](https://github.com/dukefirehawk/kelicap-templates/tree/quickstart) project on GitHub. You can get the project's files by cloning the repo.

```bash
  git clone https://github.com/dukefirehawk/kelicap-templates/quickstart
```

## Get dependencies  {#get}

```bash
  dart pub update
```

## Customize the project

1. Open **`web/index.html`**, and replace the text of the **`<title>`** element with a title suitable for your app. For example: `<title>Kelicap Tour of Heroes</title>`.

2. Open **`pubspec.yaml`**, and update the **description** to suit your project. For example: `description: Tour of Heroes`.

3. *Optional*. If you'd like to change your project's name, then do a project-wide *search-and-replace* of the current value of the **pubspec `name`** entry (**`starer_app`**) with a name suitable for your app - usually it will be the same as the directory name you chose earlier.

This project-global rename will touch: `pubspec.yaml`, `web/main.dart` and `test/app_test.dart`.

## Run the app

### Visual Studio Code

Install the [Dart VSCode extension](https://marketplace.visualstudio.com/items?itemName=Dart-Code.dart-code). If you already have the Flutter VSCode extension installed, this is not necessary.

1. Start VS Code.
2. Press **Ctrl+Shift+X** (Windows/Linux) or **Cmd+Shift+X** (Mac) to open the Extensions side panel.
3. Type `Dart` in the extensions search field, and select **Dart** in the list, and click **Install**.

<!-- kelicap will change this -->
After that, usually you will do a one-time configuration **for each project** to set things up (you won't need to do this if you use `kelicap` to create a new project):

1. Press **F5** or choose **Run > Start Debugging**.
2. VSCode will prompt you to select an environment. Choose **Dart & Flutter** or simply **Dart**. <!-- Does the Dart option even exists? It is here just in case. -->
3. After that, VSCode will create a `launch.json` file in the `.vscode` folder that contains something like this:

```json
{
 // Use IntelliSense to learn about possible attributes.
 // Hover to view descriptions of existing attributes.
 "version": "0.2.0",
 "configurations": [
  {
   "name": "quickstart",
   "request": "launch",
   "type": "dart"
  }
 ]
}
```

Add the `"program": "web"` value in "configurations":

```json
{
 // Use IntelliSense to learn about possible attributes.
 // Hover to view descriptions of existing attributes.
 "version": "0.2.0",
 "configurations": [
  {
   "name": "quickstart",
   "request": "launch",
   "type": "dart",
   "program": "web" // ADD THIS LINE
  }
 ]
}
```

You're now all set! You can close this file, and press **F5** to start running your app!

The first build usually takes a few minutes, so sit back and grab a coffee while your app is building. Builds after that are much quicker because assets are cached on disk and Dart will execute incremental builds.

### Command-Line

To run the app from the command line, use dart command line to build and serve the app:

```bash
   dart run build_runner serve
```

Then, to view your app, use the Chrome/Firefox/Edge browser to visit [localhost:8080](localhost:8080).

### Result

Regardless of your method, you should see the following app in a browser window:

![A web page with the header: Hello Kelicap](/guide/images/starter-app.png)

## Reload the app

Whenever you change the app, reload the browser window. As you save updates to the code, the dart `serve` automatically detects changes and serves the new app.

## Next step

If you're new to Kelicap, we recommend staying on the [learning path](learning-kelicap.md).
If you'd like to know more about the app you just created, see [User Guide](README.md)
