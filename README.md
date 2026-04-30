# Spreedly Checkout React Native SDK — Distribution Repository

[![GitHub Packages](https://img.shields.io/badge/GitHub%20Packages-published-blue)](https://github.com/spreedly/checkout-react-native-packages/packages)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)

This repository hosts the compiled npm packages for the
Spreedly Checkout React Native SDK (`@spreedly/react-native-checkout`)
via [GitHub Packages](https://github.com/features/packages).

Source code and issue tracking live in an internal repository. Public documentation
is available at [docs.spreedly.com](https://docs.spreedly.com).

## Installation

### 1. Configure GitHub Packages authentication

> **Note:** GitHub Packages authentication is required while this repository is
> internal. Once the repository is made public, the credentials block below can
> be removed and packages will resolve without a PAT.

Generate a Personal Access Token (PAT) with `read:packages` scope at
[github.com/settings/tokens](https://github.com/settings/tokens).

Add the following to your project-level `.npmrc` (or the global `~/.npmrc`):

```properties
//npm.pkg.github.com/:_authToken=YOUR_PERSONAL_ACCESS_TOKEN
@spreedly:registry=https://npm.pkg.github.com
```

### 2. Install the package

```bash
# npm
npm install @spreedly/react-native-checkout

# or yarn
yarn add @spreedly/react-native-checkout
```

### 3. iOS setup

```bash
cd ios && pod install
```

### 4. Android setup

The React Native SDK depends on native Android artifacts published to
[checkout-android-maven](https://github.com/spreedly/checkout-android-maven).
Add the required Maven repositories to your app's `settings.gradle.kts`:

```kotlin
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()

        // Spreedly GitHub Packages repository
        maven {
            url = uri("https://maven.pkg.github.com/spreedly/checkout-android-maven")
            credentials {
                username = providers.gradleProperty("gpr.usr").orNull
                    ?: System.getenv("GITHUB_USERNAME")
                password = providers.gradleProperty("gpr.key").orNull
                    ?: System.getenv("GITHUB_TOKEN")
            }
        }

        // Forter 3DS SDK repository (required for 3DS authentication)
        maven {
            url = uri("https://mobile-sdks.forter.com/android")
            credentials {
                username = "forter-android-sdk"
                password = ""
            }
        }
    }
}
```

Add your GitHub credentials to `~/.gradle/gradle.properties` (user-level, not committed):

```properties
gpr.usr=YOUR_GITHUB_USERNAME
gpr.key=ghp_YOUR_PERSONAL_ACCESS_TOKEN
```

## Compatibility

| Requirement | Version |
|-------------|---------|
| React Native | 0.77+ (recommended 0.79+) |
| React | 18.2+ |
| Android | minSdk 26 (Android 8.0+), targetSdk 34, compileSdk 36 |
| iOS | 15.1+, Xcode 15+ |
| Architectures | Legacy and New Architecture (Fabric / TurboModules) |

## Package Security

All published npm tarballs are **GPG-signed**. Stable releases include:

- **SHA-256 checksum manifest** (`release-manifest.json`) attached to GitHub Releases
- **GPG-signed manifest** (`release-manifest.json.asc`) for manifest integrity verification

To verify a package signature:

```bash
# Import Spreedly's public signing key (published on the source repo)
gpg --import spreedly-signing-key.pub

# Confirm the imported key fingerprint matches what Support communicated
# when they shared the key. Compare both values byte-for-byte before trusting it.
gpg --fingerprint

# Verify the manifest
gpg --verify release-manifest.json.asc release-manifest.json
```

Contact [mobile-team@spreedly.com](mailto:mobile-team@spreedly.com) for the public signing key.

### Additional verification steps

**Signed release tag** — each tag is signed by the same key:

```bash
git clone https://github.com/spreedly/checkout-react-native-packages.git
cd checkout-react-native-packages
git tag -v v1.0.2    # expect "Good signature from ..." matching the fingerprint Support shared
```

**SHA-256 round-trip against the manifest** — once the manifest signature checks out,
validate any tarball you've downloaded from GitHub Packages against the trusted hashes:

```bash
TAG=v1.0.2
BASE="https://github.com/spreedly/checkout-react-native-packages/releases/download/${TAG}"

curl -L -o release-manifest.json     "${BASE}/release-manifest.json"
curl -L -o release-manifest.json.asc "${BASE}/release-manifest.json.asc"

gpg --verify release-manifest.json.asc release-manifest.json

# Verify against the manifest
jq -r '.artifacts[] | "\(.sha256)  \(.file)"' release-manifest.json | sha256sum -c
```

## Distribution Strategy

**Current channel: GitHub Packages (npm)**

- Requires a GitHub Personal Access Token with `read:packages` scope.
- All artifacts are published under the `@spreedly` scope.
- Both release candidates (`-rc.N`) and stable versions are available.
- Dev builds (`-dev.*`) are published for internal testing.

**GitHub Packages visibility:** This repository will be made public, removing
the PAT requirement for consumers.

**npm public registry:** Planned for a future release to provide unauthenticated
public access via the standard npm registry. Until then, GitHub Packages
is the primary distribution channel.

## Version History

See [CHANGELOG.md](CHANGELOG.md) for the full release history.

## Documentation

- [Changelog](CHANGELOG.md)
- [Security Policy](SECURITY.md)
- [Example App](https://github.com/spreedly/checkout-react-native-example)
- [Spreedly Docs](https://docs.spreedly.com)

## License

Copyright 2025 Spreedly, Inc.

Licensed under the Apache License, Version 2.0. See [LICENSE](LICENSE) for details.
