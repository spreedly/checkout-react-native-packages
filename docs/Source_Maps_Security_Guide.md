# Source Maps Security Guide

Security best practices for source map handling in production builds.

## Security Risk

Source maps expose:

- Original source code structure
- Business logic and API endpoints
- Internal implementation details
- Sensitive comments

**Critical:** Never distribute source maps in production builds.

## Android Configuration

### Disable Hermes Source Maps

Edit `example/android/app/build.gradle`:

```gradle
react {
    hermesFlags = ["-O"]  // No source map in release

    // Or conditional:
    if (buildType == "release") {
        hermesFlags = ["-O"]
    } else {
        hermesFlags = ["-O", "-output-source-map"]
    }
}
```

### Exclude from APK

```gradle
android {
    buildTypes {
        release {
            packagingOptions {
                exclude "**/*.map"
                exclude "**/*.sourcemap"
            }
        }
    }
}
```

## iOS Configuration

### Xcode Build Settings

**Debug Information Format:**

- **Debug**: `DWARF with dSYM File`
- **Release**: `DWARF` (no source maps)

### Exclude from App Bundle

1. Build Phases → Copy Bundle Resources
2. Remove any `.map` or `.sourcemap` files

## Metro Bundler Configuration

Edit `example/metro.config.js`:

```javascript
module.exports = {
  transformer: {
    getTransformOptions: async () => ({
      transform: {
        experimentalImportSupport: false,
        inlineRequires: true,
      },
      sourceMapUrl:
        process.env.NODE_ENV === 'production' ? undefined : 'inline',
    }),
  },
};
```

## Verification

### Android

```bash
unzip -l app-release.apk | grep -i "\.map"
# Should return: No matches found
```

### iOS

```bash
find YourApp.app -name "*.map" -o -name "*.sourcemap"
# Should return: No matches found
```

## Best Practices

**✅ DO:**

- Generate source maps for internal debugging only
- Store source maps separately from production builds
- Use source maps in development/staging only

**❌ DON'T:**

- Include source maps in production APKs/bundles
- Distribute source maps with public releases
- Commit source maps to public repositories

## Summary

- ✅ Generate source maps for debugging
- ✅ Store separately for internal use
- ❌ Never distribute in production builds
- ✅ Verify exclusion before release

---

**Security Critical:** Always verify source maps are excluded from production distributions.
