# Android Lint Troubleshooting Guide

This guide helps resolve common Android lint and build issues when integrating the Spreedly React Native SDK.

## Common Issues and Solutions

### 1. Lint Analysis Failure with Kotlin 2.0.21+

**Problem**: Build fails with lint analysis errors like:

```
Execution failed for task ':app:lintVitalAnalyzeRelease'.
> Unexpected failure during lint analysis (this is a bug in lint or one of the libraries it depends on)
  Message: Found class org.jetbrains.kotlin.analysis.api.resolution.KaCallableMemberCall, but interface was expected
```

**Solution**: Add comprehensive lint configuration to your app's `android/build.gradle`:

```gradle
android {
    lint {
        // Disable problematic lint checks that cause failures with Kotlin 2.0.21+
        disable 'NullSafeMutableLiveData'

        // Disable other checks that may cause issues with React Native and Kotlin 2.0+
        disable 'UnusedMaterialScaffoldPaddingParameter'
        disable 'UnusedMaterial3ScaffoldPaddingParameter'
        disable 'GradleCompatible'

        // Disable checks that may interfere with React Native SDK integration
        disable 'InvalidPackage'
        disable 'MissingPermission'

        // Ensure lint doesn't fail the build on warnings
        abortOnError false
        warningsAsErrors false

        // Skip lint analysis for release builds if needed
        checkReleaseBuilds false
    }
}
```

**Alternative Solution** (if the above doesn't work):
Add this to your app's `android/build.gradle` to completely skip lint for release builds:

```gradle
android {
    buildTypes {
        release {
            // Skip lint analysis for release builds
            lintOptions {
                checkReleaseBuilds false
                abortOnError false
            }
        }
    }
}
```

### 2. Deprecated RCTEventEmitter Warnings

**Problem**: Warnings about deprecated `RCTEventEmitter` usage:

```
'interface RCTEventEmitter : Any, JavaScriptModule' is deprecated. gseprecated in Java.
```

**Solution**: These warnings are from the SDK's internal implementation and have been fixed in recent versions. If you're still seeing them:

1. Update to the latest SDK version
2. The warnings don't affect functionality and can be safely ignored
3. Add to your `android/build.gradle` to suppress warnings:

```gradle
android {
    lint {
        warningsAsErrors false
    }
}
```

### 3. "Condition is always 'true'" Warning

**Problem**: Build shows warnings like:

```
w: Condition is always 'true'.
```

**Solution**: This warning has been fixed in recent SDK versions. If you're still seeing it:

1. Update to the latest SDK version
2. Add to your `android/build.gradle` to suppress Kotlin warnings:

```gradle
android {
    kotlinOptions {
        suppressWarnings = true
    }

    lint {
        disable 'KotlinConstantConditions'
        warningsAsErrors false
    }
}
```

### 4. React Native 0.76+ Compatibility

**Problem**: Build fails with Kotlin-related errors or version mismatches.

**Solution**: Ensure your app uses compatible versions:

```gradle
// In your app's android/build.gradle
android {
    compileSdkVersion 34

    defaultConfig {
        minSdkVersion 24
        targetSdkVersion 34
    }
}

// In your project's android/build.gradle
buildscript {
    ext {
        kotlinVersion = "2.0.21"
    }
    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlinVersion"
        classpath "com.android.tools.build:gradle:8.7.2"
    }
}
```

### 4. React Native 0.76+ Compatibility

**Problem**: Build issues when using React Native 0.76 or newer.

**Solution**: The SDK is fully compatible with React Native 0.76+. Ensure you have:

1. **Correct Kotlin version**: 2.0.21+
2. **Correct Android Gradle Plugin**: 8.7.2+
3. **Updated lint configuration** (see solution #1 above)

### 5. Detekt Configuration Not Found

**Problem**: Build fails with detekt configuration errors like:

```
Execution failed for task ':spreedly_react-native-checkout:detekt'.
> Provided path 'detekt-config.yml' does not exist!
```

**Solution**: This has been fixed in recent SDK versions. The detekt configuration is now properly included in the package at `config/detekt-config.yml`. If you're still experiencing issues:

1. **Update to the latest SDK version**
2. **Clear node_modules and reinstall**:
   ```bash
   rm -rf node_modules
   npm install
   # or
   yarn install
   ```
3. **Verify the config file exists**:
   ```bash
   ls node_modules/@spreedly/react-native-checkout/config/detekt-config.yml
   ```

### 6. Gradle Build Cache Issues

**Problem**: Inconsistent build failures or cached dependency issues.

**Solution**: Clean and rebuild:

```bash
cd android
./gradlew clean
./gradlew build
```

Or for a complete clean:

```bash
cd android
rm -rf build
rm -rf app/build
./gradlew clean
./gradlew build
```

## Prevention Tips

### 1. Keep Dependencies Updated

Regularly update your dependencies to avoid compatibility issues:

```bash
npx react-native upgrade
```

### 2. Use Consistent Versions

Ensure all team members use the same versions by committing:

- `android/gradle/wrapper/gradle-wrapper.properties`
- `package-lock.json` or `yarn.lock`

### 3. Monitor Build Warnings

Address build warnings promptly before they become errors in newer versions.

## Getting Help

If you continue experiencing issues:

1. **Check the version compatibility matrix** in the main documentation
2. **Search existing GitHub issues** for similar problems
3. **Create a new issue** with:
   - Your React Native version
   - Your Kotlin version
   - Your Android Gradle Plugin version
   - Complete error logs
   - Steps to reproduce

## Version Compatibility Matrix

| React Native | Kotlin  | Android Gradle Plugin | Status             |
| ------------ | ------- | --------------------- | ------------------ |
| 0.76+        | 2.0.21+ | 8.7.2+                | ✅ Supported       |
| 0.75         | 1.9.25+ | 8.5.2+                | ⚠️ Limited Support |
| 0.72-0.74    | 1.8.x   | 8.x                   | ❌ Not Supported   |

For the most up-to-date compatibility information, see the main [Integration Guide](./Integration_Guide.md).
