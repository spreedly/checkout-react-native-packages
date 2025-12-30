## [0.1.2] - 2025-12-30

### 🔄 Changed

  - Fix/release workflow

---
## [0.0.6] - 2025-12-30

### 🐛 Fixed

  - Changelog refactoring

---
# CHANGELOG

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

---

## [0.0.4] - 2025-11-04

### 🐛 Fixed

- Artifact inspection logs added
- Sourcemap not distributed
- Documentation updated to data safety section format

### 🔄 Changed

- Changelog and versioning improvements
- Using committed lock files in CI

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
