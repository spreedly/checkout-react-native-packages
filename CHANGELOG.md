## [0.0.3-beta.16] - 2025-11-03

### ✨ Added

  - Adding changelog to release and improved changelog messages (f3a148c)

---
## [0.0.3-beta.15] - 2025-11-03

---
## [0.0.3-beta.14] - 2025-11-03

### ✨ Added

  - Changelog pushed to package repo (7abbcb6)

---
## [0.0.3-beta.13] - 2025-11-03

### ✨ Added

  - Changelog updates (4a70a78)

---
## [0.0.3-beta.12] - 2025-11-01

### 🐛 Fixed

  - Integration guide updating fixed ([823c1fd](https://github.com/spreedly/checkout-react-native/commit/823c1fd4eb4a98dfedf7a3cc319f070e80410aac))

---
## [0.0.3-beta.11] - 2025-11-01

### ✨ Added

  - Pull before push to publish repo ([dfcd007](https://github.com/spreedly/checkout-react-native/commit/dfcd007bae0b3cb67db1fb4e7553cd1d104ea99c))

---
## [0.0.3-beta.9] - 2025-11-01

### ✨ Added

  - Humanized changelog fix ([4a093fe](https://github.com/spreedly/checkout-react-native/commit/4a093feff80393f282e18f8e8d3796000d3ed6bf))
  - Humanized changelog and integration guide changes push to new repo automatically ([541333e](https://github.com/spreedly/checkout-react-native/commit/541333e50060c735ad71c210fc0820818171a414))

---
## [0.0.3-beta.7] - 2025-11-01

### Changes
- feat: hc-447: new repo version handling and fixes (984694f)

---
## [0.0.3-beta.6] - 2025-10-31

### Changes
- feat: hc-447: changelog added and new repo url update (08336c7)
- Feat/checklist points (#57) (a274d21)
- HC-443: Release workflow updates (#59) (a587d14)
- Fix/bugs (#56) (7e9ffe0)
- HC-377: Keyboard next button functionality for both platforms (#55) (79c9adb)
- Fix/bugs with codeql seperation (#54) (bcb5abb)
- fix: e2e tests fix (#52) (900e0a4)
- fix: integration doc updates (#51) (7f03adb)
- fix: update integration doc according to new changes (#50) (b3565e6)
- HC-361: Bug Fixes (#49) (1e7211a)
- Feature/xcode build fix (#48) (fcc228d)
- Fix: HC-378: CI, Lint-checks, CodeQL merged together (#47) (43ad5bd)
- HC-359: Font size accessibility handling (#46) (5262d1d)
- Revert "Fix codeql script to run swift analysis everytime (#43)" (#45) (872ac72)
- fix: hc-360: colors fix for theme (#44) (22be8b4)
- Fix codeql script to run swift analysis everytime (#43) (991efee)
- feat: standardize on iPhone 16 simulators across all workflows (#42) (2d3bfdf)
- HC-352: Version bump both platforms and theming fixes (#41) (fd790ab)
- Feat/sdk release setup (#40) (5d70813)
- HC-347: Nightly build (#39) (eb272a0)
- HC-194: updated codeql work flow to run code ql on rn sdk and example (#4) (0e5e62d)
- HC-341: CI (#37) (9f17ddc)
- fix: android app distribution build workflow with service key json (#38) (5743784)
- HC-323: Integration Tests (#36) (e261994)
- HC-323: CICD for lint checks (#33) (bfdbb82)
- feat: add app icon, build updates (#35) (d13e614)
- HC-294: Automation tests (#34) (42a2a90)
- HC: 322: Config Changes (#32) (1583e45)
- HC-311: FormFieldType Difference (#31) (3d35416)
- Spl Text field height adjustment issue, SDK docs and code cleanup (#30) (86999a2)
- feat: hc-305: setup sample app distribution workflow using fastlane Firebase n TestFlight (#29) (5e864a2)
- feat: hc-298: change package name to com.spreddly.rn. app and app name to spreedly rn sdk sample (#28) (a1d4ad9)
- feat HC-291,  HC-292, HC-293, HC-296: unit andorid, ios, src tests cases and pre-commit check (#27) (415fc9a)
- HC-295: Theme selection on payment bottom sheet (#26) (3d9638d)
- HC-279: Navigation added (#24) (24e3962)
- feat: hc-266: unit test cases setup for ios (#25) (b04f3f8)
- HC-261: Theme addition for both platforms (#23) (a6b2365)
- feat: hc-269: setup e2e setup for detox (#22) (525634b)
- fix: hc-270: yearformat fix android (#20) (418bb53)
- feat: HC-265 and hc-266: Unit test case setup for android and src module (#21) (c826503)
- feat: hc-268: added e2e setup with detox for android (#19) (e625645)
- HC:250: Customfield added, allowblankname allowwexpired support, Year format for iOS (#18) (4b407c3)
- fix: hc-250: native sdk diff for checkout flow, drop in and spltextfield component (#17) (2dbe0db)
- HC-151: Exposed setparam (#16) (01bee39)
- fix: hc-151: android and ios logs fix (#15) (d48d34e)
- HC-155: Exposed onvalidationchange for ios, added form validations and hidebottomsheet seperation (#14) (62d75b1)
- HC-216: Payment bottom sheet with additional fields for iOS (#13) (b83ac79)
- HC-180: Android payment sheet crash support for missing props (#12) (68068c7)
- fix: hc-153 hc-156: remove rn checkout button from rn sdk (#11) (d7734ae)
- HC-181: iOS payment bottom sheet (#9) (1dc3320)

---
# CHANGELOG

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.0.3-beta.5] - 2024-XX-XX

### Changed

- Current development version
- Refer to Git commit history for detailed changes

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

- Change request identifier (e.g., Jira ticket: HC-XXX)
- Description of changes
- Security impact assessment (if applicable)
- Date of release

### Example Entry Format

```markdown
## [1.0.0] - YYYY-MM-DD

**Change Request:** [HC-XXX](JIRA_LINK)

### Added

- New payment method support (#123)
- Enhanced validation for card numbers (#124)

### Fixed

- Memory leak in payment form component (#125)

### Security

- Updated dependencies to address CVE-XXXX-XXXXX
```

---

## Template for New Release

Copy and customize this template when creating a new release:

```markdown
## [X.Y.Z] - YYYY-MM-DD

**Change Request:** [TICKET-ID](JIRA_LINK)
**Author:** Developer Name
**Reviewers:** Reviewer Names

### Added

- Description of new features

### Changed

- Description of changes to existing features

### Deprecated

- Features marked for future removal

### Removed

- Features that were removed

### Fixed

- Bug fixes with issue references

### Security

- Security updates and vulnerability fixes
```

---

[0.0.3-beta.5]: https://github.com/spreedly/checkout-react-native/releases/tag/v0.0.3-beta.5
