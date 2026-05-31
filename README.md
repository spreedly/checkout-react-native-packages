# Spreedly Checkout React Native SDK — Distribution Repository

[![GitHub Packages](https://img.shields.io/badge/GitHub%20Packages-published-blue)](https://github.com/spreedly/checkout-react-native-packages/packages)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)

This repository hosts the compiled npm packages for the
Spreedly Checkout React Native SDK (`@spreedly/react-native-checkout`)
via [GitHub Packages](https://github.com/features/packages).

Public documentation is available at [docs.spreedly.com](https://docs.spreedly.com).

## Installation

### 1. Configure GitHub Packages authentication

Generate a Personal Access Token (PAT) with `read:packages` scope at
[github.com/settings/tokens](https://github.com/settings/tokens).

Add the following to your project-level `.npmrc` (or the global `~/.npmrc`):

```properties
//npm.pkg.github.com/:_authToken=${GITHUB_TOKEN}
@spreedly:registry=https://npm.pkg.github.com
```


This repository and its packages are **public** on GitHub — you do not need org membership or a private-repo token. GitHub Packages still requires authenticated requests to `npm.pkg.github.com`, so only `GITHUB_TOKEN` and `GITHUB_USERNAME` are needed.

### 2. Install the package

```bash
# npm
npm install @spreedly/react-native-checkout

# or yarn
yarn add @spreedly/react-native-checkout
```

### 3. Android setup

Update your project-level `android/build.gradle` with the Kotlin and Compose dependencies required by the SDK, and add the metadata version check workaround:

```gradle
buildscript {
    ext {
        buildToolsVersion = "36.0.0"
        minSdkVersion = 26
        compileSdkVersion = 36
        targetSdkVersion = 36
        ndkVersion = "27.1.12297006"
        kotlinVersion = "2.3.10"
        androidGradlePluginVersion = "8.12.0"
    }
    repositories {
        google()
        mavenCentral()
    }
    dependencies {
        classpath("com.android.tools.build:gradle:${androidGradlePluginVersion}")
        classpath("com.facebook.react:react-native-gradle-plugin")
        classpath("org.jetbrains.kotlin:kotlin-gradle-plugin:${kotlinVersion}")
        classpath("org.jetbrains.kotlin:kotlin-serialization:${kotlinVersion}")
        classpath("org.jetbrains.kotlin:compose-compiler-gradle-plugin:${kotlinVersion}")
    }
}

apply plugin: "com.facebook.react.rootproject"

subprojects { subproject ->
    subproject.tasks.withType(org.jetbrains.kotlin.gradle.tasks.KotlinCompile).configureEach {
        compilerOptions {
            freeCompilerArgs.add("-Xskip-metadata-version-check")
        }
    }
}
```

### 4. iOS setup

Add the Spreedly pod setup script to your `ios/Podfile` and call `init_spreedly_checkout_pods()` inside your app target:

```ruby
# Resolve react_native_pods.rb with node to allow for hoisting
require Pod::Executable.execute_command('node', ['-p',
  'require.resolve(
    "react-native/scripts/react_native_pods.rb",
    {paths: [process.argv[1]]},
  )', __dir__]).strip

require Pod::Executable.execute_command('node', ['-p',
  'require.resolve(
    "@spreedly/react-native-checkout/scripts/spreedly_pods_setup.rb",
    {paths: [process.argv[1]]},
  )', __dir__]).strip

platform :ios, min_ios_version_supported
prepare_react_native_project!

target 'YourApp' do
  config = use_native_modules!

  use_react_native!(
    :path => config[:reactNativePath],
    :app_path => "#{Pod::Config.instance.installation_root}/.."
  )

  init_spreedly_checkout_pods()

  post_install do |installer|
    react_native_post_install(
      installer,
      config[:reactNativePath],
      :mac_catalyst_enabled => false
    )
  end
end
```

Then install pods:

```bash
cd ios && pod install
```

### 5. Run the app

Start Metro and launch on a device or simulator:

```bash
# Start the Metro bundler
npm start
# or
yarn start

# In a separate terminal — Android
npm run android
# or
yarn android

# In a separate terminal — iOS
npm run ios
# or
yarn ios
```

## Compatibility

| Requirement | Version |
|-------------|---------|
| React Native | 0.79+ |
| React | 18.2+ |
| Android | minSdk 26 (Android 8.0+), targetSdk 34, compileSdk 36 |
| iOS | 15.1+, Xcode 15+ |
| Architectures | Legacy and New Architecture (Fabric / TurboModules) |

## Package Security

All published npm tarballs are **GPG-signed**. Stable releases include:

- **SHA-256 checksum manifest** (`release-manifest.json`) attached to GitHub Releases
- **GPG-signed manifest** (`release-manifest.json.asc`) for manifest integrity verification

Contact [mobile-team@spreedly.com](mailto:mobile-team@spreedly.com) for the public signing key.

### Additional verification steps

**Signed release tag** — each tag is signed by the same key:

**SHA-256 round-trip against the manifest** — once the manifest signature checks out,
validate any tarball you've downloaded from GitHub Packages against the trusted hashes:

## Distribution Strategy

**Current channel: GitHub Packages (npm)**

- This repository and all published packages are **public** on GitHub.
- Install requires `GITHUB_TOKEN` (with `read:packages`) and `GITHUB_USERNAME` — see [Installation](#installation).
- All artifacts are published under the `@spreedly` scope.
- Both release candidates (`-rc.N`) and stable versions are available.

**npm public registry:** Planned for a future release to provide access via the standard npm registry (`npmjs.com`). Until then, GitHub Packages is the primary distribution channel.

## Version History

See [CHANGELOG.md](CHANGELOG.md) for the full release history.

## Support

- **Spreedly Documentation**: [docs.spreedly.com](https://docs.spreedly.com/)
- **Support Portal**: [spreedly.com/support](https://spreedly.com/support/)
- **Example App**: [checkout-react-native-example](https://github.com/spreedly/checkout-react-native-example)
- **Changelog**: [CHANGELOG.md](CHANGELOG.md)
- **Security**: [SECURITY.md](SECURITY.md)

## License

Copyright 2025 Spreedly, Inc.

Licensed under the Apache License, Version 2.0. See [LICENSE](LICENSE) for details.

## Legal

- [Terms of Service](https://legal.spreedly.com/#terms)
- [Privacy Policy](https://legal.spreedly.com/#privacy-policy)
- [License](LICENSE) (Apache 2.0)
