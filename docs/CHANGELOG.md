## [0.3.1-beta.0] - 2026-04-02

### ✨ Added

  - Workflow and integration config refactoring
  - Documentation updates
  - Modular dependency approch added

### 🐛 Fixed

  - Update app distribution workflow to match mono repo architecture
  - Release docs updated
  - Cursor folder removal
  - Dependabot workflow fix
  - Dependabot workflow fix
  - Packages export fix

### 🔄 Changed

  - Feat: Added gitleaks config to CI Workflow
  - Fix: 3ds gateway race condition fix

### 📦 Native SDK Versions

| Platform | Component / SDK | Version |
| -------- | --------------- | ------- |
| Android  | checkout-payments-core (core) | 0.12.1 |
| Android  | checkout-braintree-apm | 0.12.1 |
| Android  | checkout-stripe-apm | 0.12.1 |
| iOS      | Spreedly private pods (tag) | 1.2.2 |
| iOS      | Braintree (CocoaPods) |   s.dependency SpreedlyBraintree |
| iOS      | StripePaymentSheet (CocoaPods) | ~> 25.0 |

---
# CHANGELOG

## [0.3.0] - 2026-03-19

### 🐛 Fixed

- Dependabot codeql vulnerablities fix
- DataDog sdkPlatform logging added
- Workflows optimized
- PCI Compliant audit and related fixes

### 🔄 Changed

- latest react native version support added
- documentations refactoring
- braintree payment integration
- stripe apm
- ebanx payments
- Offsite payments

### 📦 Native SDK Versions

| Platform | SDK                  | Version |
| -------- | -------------------- | ------- |
| Android  | checkout-android     | 0.12.1  |
| iOS      | checkout-ios-package | 1.2.2   |

---

## [0.2.5] - 2026-02-09

### 🐛 Fixed

- Dependabot fix

### 🔄 Changed

- Feat/hc 1030 3ds gateway
- Fix/documents review

### 📦 Native SDK Versions

| Platform | SDK                  | Version |
| -------- | -------------------- | ------- |
| Android  | checkout-android     | 0.9.1   |
| iOS      | checkout-ios-package | 1.0.15  |

---

## [0.2.1] - 2026-01-28

### ✨ Added

- Empty expiry date switch

### 🐛 Fixed

- Test coverage improved
- Dependabot alerts fixed
- Integration workflow detox dependency fix
- Archieve xcode fix
- Console and debugger restriction to production
- Dark theme android fixed and testcases updated

### 🔄 Changed

- 3DS Implementation

### 📦 Native SDK Versions

| Platform | SDK                  | Version |
| -------- | -------------------- | ------- |
| Android  | checkout-android     | 0.8.3   |
| iOS      | checkout-ios-package | 1.0.11  |

---

## [0.2.0] - 2025-12-31

### ✨ Added

- Recaching CVV
- Retaining CVV

### 📦 Native SDK Versions

| Platform | SDK                  | Version |
| -------- | -------------------- | ------- |
| Android  | checkout-android     | 0.4.8   |
| iOS      | checkout-ios-package | 0.0.67  |

---

## [0.0.6] - 2025-12-30

### 🐛 Fixed

- Bug fixes and improvements

### 📦 Native SDK Versions

| Platform | SDK                  | Version |
| -------- | -------------------- | ------- |
| Android  | checkout-android     | 0.4.8   |
| iOS      | checkout-ios-package | 0.0.67  |

---

## [0.0.5] - 2025-11-13

### ✨ Added

- Central Logging to Datadog
- DAST scan workflow
- Security documentation

### 🐛 Fixed

- Runners updated, CodeQL issue fix
- OSS dependency reviewed and turbo library removed

### 🔄 Changed

- Dark theme support
- Screenshot/Screen recording prevention
- Security checklist fixes

### 📦 Native SDK Versions

| Platform | SDK                  | Version |
| -------- | -------------------- | ------- |
| Android  | checkout-android     | 0.3.8   |
| iOS      | checkout-ios-package | 0.0.51  |

---

## [0.0.4] - 2025-11-04

### 🐛 Fixed

- Artifact inspection logs added
- Sourcemap not distributed
- Documentation updated to data safety section format

### 🔄 Changed

- Changelog and versioning improvements
- Using committed lock files in CI

### 📦 Native SDK Versions

| Platform | SDK                  | Version |
| -------- | -------------------- | ------- |
| Android  | checkout-android     | 0.3.8   |
| iOS      | checkout-ios-package | 0.0.51  |

---

## [0.0.3] - 2025-11-03

### ✨ Added

- Changelog to release process
- SDK release setup
- Font size accessibility handling
- Keyboard next button functionality for both platforms
- Integration tests

### 🐛 Fixed

- Bug fixes and stability improvements
- Theme colors fix
- Integration documentation updates

### 🔄 Changed

- Version bump for both platforms
- Theming fixes and improvements

### 📦 Native SDK Versions

| Platform | SDK                  | Version |
| -------- | -------------------- | ------- |
| Android  | checkout-android     | 0.3.8   |
| iOS      | checkout-ios-package | 0.0.38  |

---

## [0.0.2] - 2025-10-01

### ✨ Added

- Initial React Native SDK setup
- Android SDK initialization with TextField and Checkout button
- iOS SDK initialization with TextField
- Checkout functionality on SDK and example app
- Payment bottom sheet with additional fields
- Form validations and onValidationChange exposure
- Custom textfield support
- Allow blank name and allow expired support
- Year format handling for both platforms
- Unit test setup for Android, iOS, and src modules
- E2E setup with Detox for both platforms
- Theme selection on payment bottom sheet
- Navigation setup
- App distribution workflow using Fastlane, Firebase, and TestFlight
- Nightly build workflow added
- CI/CD for lint checks
- CodeQL workflow added

### 🐛 Fixed

- Year format fix for Android
- Android and iOS logs fix
- Payment sheet crash support for missing props

### 🔄 Changed

- Package name to com.spreedly.rn
- Setup documentation improvements

### 📦 Native SDK Versions

| Platform | SDK                  | Version |
| -------- | -------------------- | ------- |
| Android  | checkout-android     | 0.3.2   |
| iOS      | checkout-ios-package | 0.0.20  |

---

## [0.0.1] - 2025-09-01

### ✨ Added

- Initial project setup
- React Native SDK foundation

---

## Version History Guidelines

### Semantic Versioning

This project follows [Semantic Versioning](https://semver.org/):

- **MAJOR** version for incompatible API changes
- **MINOR** version for backwards-compatible functionality additions
- **PATCH** version for backwards-compatible bug fixes
- **Beta/Alpha** versions for pre-release testing

### Change Categories

- **Added** - New features and capabilities
- **Changed** - Changes in existing functionality
- **Deprecated** - Soon-to-be removed features
- **Removed** - Removed features
- **Fixed** - Bug fixes
- **Security** - Vulnerability fixes

### PCI DSS Compliance

Each release must document:

- Change request identifier
- Description of changes
- Security impact assessment (if applicable)
- Date of release

### Example Entry Format

```markdown
## [1.0.0] - YYYY-MM-DD

### Added

- New payment method support
- Enhanced validation for card numbers

### Fixed

- Memory leak in payment form component

### Security

- Updated dependencies to address CVE-XXXX-XXXXX
```
