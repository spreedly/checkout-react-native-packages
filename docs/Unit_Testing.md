# Unit Testing Guide

This document describes how we organize tests, our coverage goals, and patterns to follow when adding tests for both the JS/TS SDK layer and the native Android module.

## Goals

- Stability first: catch regressions in core logic (validation, mapping)
- Developer experience: fast, deterministic tests that are easy to run locally and in CI
- Coverage targets:
  - JS/TS (core logic and wrappers): 90%+ per-file
  - Android Kotlin (unit): 80%+ (push higher where feasible)

## Directory Structure

- JS/TS tests live next to the SDK source:
  - Source: `src/**/*.{ts,tsx}`
  - Tests: `src/__tests__/*.test.{ts,tsx}`
- Android unit tests:
  - Source: `android/src/main/java/...`
  - Tests: `android/src/test/java/...`

## Running Tests

- JS/TS:

```sh
yarn test               # quick run
yarn test:coverage      # with coverage
# Equivalent:
# jest --coverage --collectCoverageFrom="src/**/*.{ts,tsx}"
```

- Android (library module):

```sh
yarn android:test            # debug unit tests
yarn android:test:release    # release unit tests
```

## JS/TS Testing Patterns

- Pure logic modules: cover all branches
  - Example: `src/utils/PaymentResultMapper.ts`
  - Test: `src/__tests__/PaymentResultMapper.test.ts` enumerates all statuses and error types
- Validation helpers: assert required vs optional field behavior
  - Example: `src/ValidationManager.ts`
  - Test: `src/__tests__/ValidationManager.test.ts`
- Component wrappers: assert props and event mapping with react-test-renderer
  - Examples:
    - `src/headlessComponent/SPLTextField.tsx`
    - `src/PaymentBottomSheet/PaymentBottomSheet.tsx`
  - Use minimal `react-native` mocks and wrap renders in `act()`
  - Example tests:
    - `src/__tests__/SPLTextField.test.tsx`
    - `src/__tests__/PaymentBottomSheet.test.tsx`
- Codegen-based native component wrappers:
  - Mock `react-native/Libraries/Utilities/codegenNativeComponent` to a simple passthrough renderer
  - Example test: `src/__tests__/SpreedlyCheckoutViewNativeComponent.test.tsx`

### Coverage Tips

- Use `collectCoverageFrom` to include files that don’t import in tests by default
- If a wrapper is extremely thin, prefer testing the consuming logic rather than duplicating RN internals
- Avoid over-mocking; keep mocks minimal and purposeful

## Android (Kotlin) Testing Patterns

- Mapping helpers and utility classes:
  - Example: `android/src/main/java/com/spreedlycheckout/utils/Utils.kt`
  - Test: `android/src/test/java/com/spreedlycheckout/utils/UtilsPaymentResultTest.kt`
- Module classes that depend on coroutines/flows:
  - Use `kotlinx-coroutines-test` and a `MainDispatcherRule`
  - Mock dependencies (e.g., `Spreedly`) with Mockito

### Configuration

- Gradle (already configured):
  - `testOptions.unitTests.returnDefaultValues = true`
  - `testOptions.unitTests.includeAndroidResources = true`
  - Dependencies: JUnit 4, Mockito core, Kotlin test, coroutines-test

### Running

```sh
yarn android:test            # debug unit tests
```

## Adding New Tests

1. Identify the module (JS/TS or Android) and the behaviors to verify
2. Choose the right pattern:
   - Pure logic → direct unit tests
   - Component wrapper → renderer with minimal RN mocks and `act()`
   - Android utility → plain JUnit with Mockito
3. Place tests under the correct `__tests__` folder (JS/TS) or `src/test/java` (Android)
4. Ensure coverage remains above targets; break large scenarios into focused cases

## FAQ

- Why not snapshot tests? We focus on behavioral tests to reduce brittleness.
- Should we mock native modules? Only as needed to keep tests deterministic.
- Do we require Detox? Detox is for E2E in the example app; keep unit tests fast and isolated.

## CI Notes

- Ensure Android SDK/NDK and credentials (if needed) are available in your CI job when running `yarn android:test`.
- For JS/TS, the default `jest` preset for React Native is already configured in `package.json`.
