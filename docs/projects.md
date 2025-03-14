# Project

A project is an app that contains React code and has a dependency on `react-native`.

Projects can provide additional properties to alter the CLI behavior, such as custom location of React Native files (this is useful when running RNTester from source) or a non-standard location of a project (useful when working in a brownfield app).

## How does it work?

A project can define a `react-native.config.js` at the root with custom configuration to be picked up by the CLI.

For example, below configuration informs CLI about a source directory with iOS files.

```js
/** @type import("@react-native-community/cli-types").Config */
module.exports = {
  project: {
    ios: {
      sourceDir: './custom-ios-location',
    },
  },
};
```

> Note: You may find this useful in scenarios where multiple `Podfile` files are present and CLI chooses the wrong one. When that happens, CLI will print a warning asking you to verify its selection.

You can check all available options below.

## Project interface

```ts
// referenced from @react-native-community/cli-types
type ProjectConfigT = {
  reactNativePath: ?string;
  project: {
    android?: AndroidProjectParams;
    ios?: IOSProjectParams;
    // Any additional platforms would appear here
    [key: string]: any;
  };
  platforms: {
    android: AndroidPlatformConfig;
    ios: IOSPlatformConfig;
    // Any additional platforms would appear here
    [key: string]: any;
  };
  dependencies: {
    [key: string]: DependencyConfig;
  };
  commands: Command[];
  assets?: string[];
};
```

### reactNativePath

A custom path to React Native, in case `require('react-native')` would throw. Useful when running
React Native from a (custom) source.

### project

A map of specific settings that can be set per platform. The exact shape is always defined by the package that provides given platform.

In most cases, as a React Native developer, you should not need to define any of these.

The following settings are available on iOS and Android:

```ts
// referenced from @react-native-community/cli-types
type IOSProjectParams = {
  sourceDir?: string;
  watchModeCommandParams?: string[];
  automaticPodsInstallation?: boolean;
  assets?: string[];
};

type AndroidProjectParams = {
  sourceDir?: string;
  appName?: string;
  manifestPath?: string;
  packageName?: string;
  dependencyConfiguration?: string;
  watchModeCommandParams?: string[];
  assets?: string[];
};
```

#### project.ios.sourceDir

A path to a directory where iOS source files are located. In most cases, you shouldn't need to set it, unless you have
multiple `Podfile` files in your project.

#### project.ios.watchModeCommandParams

Array of strings that will be passed to the `npx react-native run-ios` command when running in watch mode.

#### project.ios.automaticPodsInstallation

> Default: `true`

A boolean value to determine if you want to automatically install CocoaPods when running `run-ios` or `build-ios` command when:

- they are not yet installed
- a new dependency visible for autolinking is installed
- a version of existing native dependency has changed

If set to `true`, you can skip running `pod install` manually whenever it's needed.

> Note: Starting from React Native 0.73, CLI's `init` command scaffolds the project with `react-native.config.js` file with this value set to `true` by default. Older projects can opt-in after migrating to 0.73. Please note that if your setup does not follow the standard React Native template, e.g. you are not using Gems to install CocoaPods, this might not work properly for you. Starting from React Native 0.79, users shouldn't install CocoaPods manually, but can still opt-out of automatic installation by setting this value to `false`.

### project.ios.assets

Array of folder paths that will be passed to the `npx react-native link-assets` command to specify the assets to be linked to iOS project.

#### project.android.appName

A name of the app in the Android `sourceDir`, equivalent to Gradle project name. By default it's `app`.

#### project.android.sourceDir

See [`dependency.platforms.android.sourceDir`](dependencies.md#platformsandroidsourcedir)

#### project.android.manifestPath

See [`dependency.platforms.android.manifestPath`](dependencies.md#platformsandroidmanifestpath)

#### project.android.packageName

See [`dependency.platforms.android.packageName`](dependencies.md#platformsandroidpackagename)

#### project.android.dependencyConfiguration

See [`dependency.platforms.android.configuration`](dependencies.md#platformsandroiddependencyconfiguration)

#### project.android.watchModeCommandParams

Array of strings that will be passed to the `npx react-native run-android` command when running in watch mode.

### project.android.assets

Array of folder paths that will be passed to the `npx react-native link-assets` command to specify the assets to be linked to Android project.

### platforms

A object with platforms defined inside a project. You can check the format and options available [`here`](platforms.md#platform-interface)

### commands

An array of commands defined inside a project. You can check the format and options available [`here`](plugins.md#command-interface)

### dependencies

Dependencies is a map where key is the name of the dependency and value is an object that can override any of the resolved settings for a particular package.

For example, you could set:

```js
/** @type import("@react-native-community/cli-types").Config */
module.exports = {
  dependencies: {
    'react-native-webview': {
      platforms: {
        ios: null,
      },
    },
  },
};
```

in order to disable linking of React Native WebView on iOS.

or you could set:

```js
/** @type import("@react-native-community/cli-types").Config */
module.exports = {
  dependencies: {
    'react-native-brownfield': {
      platforms: {
        android: {
          dependencyConfiguration:
            'embed project(path: ":react-native-brownfield-bridge", configuration: "default")',
        },
      },
    },
  },
};
```

in order to use something else than `implementation` _(default scope method)_

Another use-case would be supporting local libraries that are not discoverable for autolinking, since they're not part of your `dependencies` or `devDependencies`:

```js
/** @type import("@react-native-community/cli-types").Config */
module.exports = {
  dependencies: {
    'local-rn-library': {
      root: path.join(__dirname, '/path/to/library'),
    },
  },
};
```

The object provided here is deep merged with the dependency config. Check [`projectConfig`](platforms.md#projectconfig) and [`dependencyConfig`](platforms.md#dependencyConfig) return values for a full list of properties that you can override.

> Note: This is an advanced feature and you should not need to use it most of the time.

### assets

Array of folder paths that will be passed to the `npx react-native link-assets` command to specify the assets to be linked to Android / iOS projects.
