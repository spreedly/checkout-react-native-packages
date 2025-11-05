# Platform-Specific Privacy Requirements

**Version:** 1.0  
**Last Updated:** November 2025

## Overview

This document provides a comprehensive reference for platform-specific privacy requirements and clarifies what is **implemented in code** versus what requires **documentation/guides** for developers. It includes coverage analysis and requirements for both iOS and Android platforms.

## Quick Reference

| Platform    | Requirement Type          | Implementation             | Documentation      |
| ----------- | ------------------------- | -------------------------- | ------------------ |
| **iOS**     | Privacy Manifest (plist)  | ✅ **Code Implementation** | ✅ Guide Available |
| **Android** | Data Safety Questionnaire | ❌ Manual Entry            | ✅ Guide Available |

---

## iOS: Privacy Manifest (PrivacyInfo.xcprivacy)

### Type: **CODE IMPLEMENTATION** ✅

The iOS privacy manifest is a **required file** that must be included in the SDK bundle.

#### Implementation Status

✅ **Implemented:**

- File location: `ios/PrivacyInfo.xcprivacy`
- Bundled via: CocoaPods podspec (`s.resources`)
- Format: XML Property List (plist)
- Compliance: Declares API usage, no data collection, no tracking

#### What's Included

The SDK includes a `PrivacyInfo.xcprivacy` file that:

- Declares API usage (UserDefaults, File Timestamp, System Boot Time)
- States no data collection
- States no tracking
- Uses Apple's required reason codes

#### Developer Action Required

**For SDK Developers:**

- ✅ File is already implemented in `ios/PrivacyInfo.xcprivacy`
- ✅ File is automatically bundled via `SpreedlyCheckout.podspec`
- ✅ No additional action needed

**For App Developers:**

- ✅ Privacy manifest is automatically included when using the SDK
- ⚠️ App may need its own `PrivacyInfo.xcprivacy` for app-specific APIs/data

#### Documentation

- **Complete Guide:** [iOS Privacy Guide](iOS_Privacy_Guide.md) - Privacy Manifest + App Store Connect Privacy Labels
- **Format Reference:** [Apple Privacy Manifest Documentation](https://developer.apple.com/documentation/bundleresources/privacy_manifests)

---

## Android: Data Safety Questionnaire

### Type: **DOCUMENTATION/GUIDE** ✅

The Android Data Safety section is a **manual questionnaire** filled out in Google Play Console.

#### Implementation Status

❌ **Not a Code Implementation:**

- This is a manual form in Google Play Console
- No code or manifest files required
- Completed by app developers when publishing to Google Play

#### What's Provided

The SDK provides:

- ✅ **Comprehensive Guide:** Step-by-step instructions for completing the questionnaire
- ✅ **Format Mapping:** Exact answers for each question
- ✅ **Verification Checklist:** Ensures all requirements are met

#### Developer Action Required

**For SDK Developers:**

- ✅ Documentation guide is provided
- ✅ No code implementation needed

**For App Developers:**

- ⚠️ **Must complete Google Play Console Data Safety section manually**
- ✅ Use the provided guide to ensure accurate responses
- ✅ Combine SDK data practices with app-specific practices

#### Documentation

- **Format Guide:** [Android Data Safety Guide](Android_Data_Safety_Guide.md)
- **Reference:** [Google Play Data Safety Policy](https://support.google.com/googleplay/android-developer/answer/10787469)

---

## Comparison Table

| Aspect               | iOS Privacy Manifest        | Android Data Safety    |
| -------------------- | --------------------------- | ---------------------- |
| **Type**             | Code file (plist)           | Manual questionnaire   |
| **Location**         | `ios/PrivacyInfo.xcprivacy` | Google Play Console    |
| **Included in SDK?** | ✅ Yes (automatically)      | ❌ No (manual entry)   |
| **Format**           | XML Property List           | Web form               |
| **Required By**      | Apple App Store             | Google Play Store      |
| **Developer Action** | None (automatic)            | Complete form manually |
| **Documentation**    | Implementation guide        | Questionnaire guide    |

---

## Implementation Checklist

### iOS Privacy Manifest

- [x] ✅ `PrivacyInfo.xcprivacy` file created in `ios/` directory
- [x] ✅ File included in `SpreedlyCheckout.podspec` resources
- [x] ✅ XML format validated
- [x] ✅ API usage declared (UserDefaults, File Timestamp, System Boot Time)
- [x] ✅ Data collection declared (empty - no collection)
- [x] ✅ Tracking declared (false - no tracking)
- [x] ✅ Documentation guide created

### Android Data Safety

- [x] ✅ Documentation guide created
- [x] ✅ Format-specific requirements documented
- [x] ✅ Questionnaire mapping provided
- [x] ✅ Verification checklist included
- [x] ✅ Reference to guide in Unified Privacy doc

---

## Key Differences

### iOS (Implementation Required)

```
SDK Code
  └── ios/PrivacyInfo.xcprivacy ✅ (File exists)
       └── Bundled automatically via CocoaPods
            └── Included in app bundle
                 └── Apple validates automatically
```

### Android (Documentation Required)

```
SDK Documentation
  └── docs/Android_Data_Safety_Guide.md ✅ (Guide exists)
       └── App Developer reads guide
            └── Completes Google Play Console form manually
                 └── Google validates submission
```

---

## For Developers Using This SDK

### iOS Developers

**What You Get:**

- ✅ Privacy manifest automatically included
- ✅ No action required
- ✅ App Store compliance handled

**What You May Need:**

- ⚠️ Your own `PrivacyInfo.xcprivacy` if your app uses additional APIs or collects data

### Android Developers

**What You Get:**

- ✅ Comprehensive guide for completing Data Safety section
- ✅ Exact answers for SDK-related questions
- ✅ Format-specific requirements documented

**What You Must Do:**

- ⚠️ Complete Google Play Console Data Safety section manually
- ⚠️ Use the guide to ensure accurate responses
- ⚠️ Combine SDK practices with your app's practices

---

## Related Documentation

- [Unified Privacy Policy](Unified_Privacy.md) - Complete privacy practices
- [iOS Privacy Guide](iOS_Privacy_Guide.md) - Complete iOS guide (Privacy Manifest + App Store Connect)
- [Android Data Safety Guide](Android_Data_Safety_Guide.md) - Android questionnaire guide

---

## Summary

**iOS Privacy Manifest:**

- ✅ **Code Implementation** - File exists in SDK
- ✅ Automatically bundled and included
- ✅ No developer action required
- ✅ Guide available for reference

**Android Data Safety:**

- ✅ **Documentation/Guide** - Manual questionnaire
- ❌ No code implementation needed
- ⚠️ Developer must complete form manually
- ✅ Comprehensive guide provided

---

**Note:** Both platforms require format-specific compliance, but the implementation approach differs:

- **iOS:** Requires actual code file (plist) in SDK ✅
- **Android:** Requires documentation guide for manual form ✅
