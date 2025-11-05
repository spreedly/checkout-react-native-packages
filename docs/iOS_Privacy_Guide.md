# iOS Privacy Requirements Guide

**Version:** 2.0  
**Last Updated:** November 2025  
**Purpose:** Complete guide for iOS privacy requirements including Privacy Manifest and App Store Connect Privacy Labels

## Overview

This document provides comprehensive guidance for iOS privacy requirements when integrating the Spreedly Checkout React Native SDK. It covers both:

1. **Privacy Manifest** (`PrivacyInfo.xcprivacy`) - Code file included in SDK
2. **App Store Connect Privacy Labels** - Questionnaire completed during app submission

---

## Part 1: Privacy Manifest (PrivacyInfo.xcprivacy)

### Overview

The iOS privacy manifest is a **required file** that must be included in the SDK bundle. Apple requires this file for all SDKs and frameworks that use certain APIs or collect data.

### Implementation Status

✅ **Implemented:** The SDK includes a `PrivacyInfo.xcprivacy` file at `ios/PrivacyInfo.xcprivacy`

This file is automatically included in the SDK bundle when distributed via CocoaPods.

### File Location

- **SDK Source:** `ios/PrivacyInfo.xcprivacy`
- **Bundled With:** Included in CocoaPods podspec (`s.resources`)

### Current Implementation

The SDK's `PrivacyInfo.xcprivacy` file declares:

1. **API Usage Declarations** (`NSPrivacyAccessedAPITypes`):
   - **UserDefaults API** (CA92.1) - Used by underlying Spreedly SDK dependencies
   - **File Timestamp API** (C617.1) - Used by underlying Spreedly SDK dependencies
   - **System Boot Time API** (35F9.1) - Used by underlying Spreedly SDK dependencies

2. **Data Collection** (`NSPrivacyCollectedDataTypes`):
   - **Empty array** - The SDK does not collect any data types

3. **Tracking** (`NSPrivacyTracking`):
   - **False** - The SDK does not track users

### API Usage Reason Codes

- **CA92.1** - UserDefaults: Used for functionality that is not user-facing
- **C617.1** - File Timestamp: Used for functionality that is not user-facing
- **35F9.1** - System Boot Time: Used for functionality that is not user-facing

**Note:** These API usages come from the underlying Spreedly SDK dependencies (`SpreedlySecurity`, `SpreedlyCore`, `SpreedlyUI`), not from the React Native SDK wrapper itself.

### Format Requirements

#### XML Structure

The `PrivacyInfo.xcprivacy` file must be valid XML Property List format:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <!-- Required keys -->
</dict>
</plist>
```

#### Required Keys

1. **NSPrivacyAccessedAPITypes** (array)
   - Required if SDK uses any of Apple's designated APIs
   - Each entry must include:
     - `NSPrivacyAccessedAPIType` (string)
     - `NSPrivacyAccessedAPITypeReasons` (array of reason codes)

2. **NSPrivacyCollectedDataTypes** (array)
   - Required to declare data collection (empty if none)
   - Each entry includes data type and purpose

3. **NSPrivacyTracking** (boolean)
   - Required to declare tracking behavior
   - Set to `false` if no tracking occurs

4. **NSPrivacyTrackingDomains** (array, optional)
   - Required only if `NSPrivacyTracking` is `true`
   - Not included in SDK manifest

### Integration Requirements

**For SDK Developers:**

- ✅ File is already implemented in `ios/PrivacyInfo.xcprivacy`
- ✅ File is automatically bundled via `SpreedlyCheckout.podspec`
- ✅ No additional action needed

**For App Developers:**

- ✅ Privacy manifest is automatically included when using the SDK
- ⚠️ App may need its own `PrivacyInfo.xcprivacy` for app-specific APIs/data

### Verification Checklist

- [ ] ✅ `PrivacyInfo.xcprivacy` file exists in `ios/` directory
- [ ] ✅ File is included in `SpreedlyCheckout.podspec` resources
- [ ] ✅ XML format is valid
- [ ] ✅ All required keys are present
- [ ] ✅ API usage reasons match actual SDK behavior
- [ ] ✅ Data collection array is empty (no data collection)
- [ ] ✅ Tracking is set to `false`

---

## Part 2: App Store Connect Privacy Labels

### Overview

**Important:** App Store Connect Privacy Labels are separate from the privacy manifest file. They are completed during app submission in App Store Connect and are user-facing privacy disclosures.

### Quick Reference Summary

| Category            | Answer                                         |
| ------------------- | ---------------------------------------------- |
| **Data Collection** | ✅ Yes (Payment information collected)         |
| **Data Sharing**    | ✅ Yes (Shared with Spreedly for tokenization) |
| **Tracking**        | ❌ No (No tracking performed)                  |
| **Purpose**         | Payment processing, App functionality          |

### Section 1: Data Collection

**Question:** "Does your app collect data?"

**Answer:** ✅ **YES**

**Note:** The SDK collects payment card information during tokenization. This data is processed in memory only and not stored.

#### Financial Information

**Category:** Financial Information  
**Type:** Payment Info  
**Collection:** ✅ **YES**

**Details:**

- **What:** Card number, CVV, expiration date
- **Linked to User:** ✅ Yes (Payment card is linked to user)
- **Used for Tracking:** ❌ No
- **Purpose:** App Functionality, Payment Processing

**Format-specific Notes for App Store Connect:**

- Select "Financial Information" category
- Subcategory: "Payment Info"
- Mark as "Linked to User"
- Mark as "Not Used for Tracking"
- Purpose: "App Functionality", "Payment Processing"

### Section 2: Data Sharing

**Question:** "Is this data shared with third parties?"

**Answer:** ✅ **YES**

#### Shared With: Spreedly

**Third-Party Name:** Spreedly  
**Third-Party Type:** Payment Processor  
**Purpose:** Payment Processing

**Data Shared:**

- Financial Information (Payment Info)
  - Card number
  - CVV
  - Expiration date

**Format-specific Notes for App Store Connect:**

- Third-party name: "Spreedly"
- Category: "Service Provider"
- Purpose: "Payment Processing"
- Data type: "Financial Information" → "Payment Info"
- Link to privacy policy: https://www.spreedly.com/privacy

**Important:** Spreedly is PCI DSS Level 1 certified and handles payment data according to their privacy policy.

### Section 3: Tracking

**Question:** "Does your app track users?"

**Answer:** ❌ **NO**

**Details:**

- The SDK does not track users across apps or websites
- No advertising identifiers are accessed
- No user profiling or tracking occurs

**Format-specific Notes for App Store Connect:**

- Select "No" for tracking
- No tracking domains need to be declared

### Section 4: Data Collection Details

**Purpose of Data Collection**

**Primary Purpose:** Payment Processing

**App Store Connect Categories:**

- ✅ **App Functionality** (Required)
- ✅ **Payment Processing** (Required)

**Format-specific Notes for App Store Connect:**

- Select "App Functionality" as purpose
- Select "Payment Processing" as purpose
- Do NOT select: Analytics, Advertising, Personalization, Developer Communications

**Is data collection optional?** ❌ **NO**

**Reason:** Payment processing requires card information to function.

**Format-specific Notes for App Store Connect:**

- Mark as "Required for functionality"
- Note: "Payment processing cannot proceed without card data"

### Complete Form Checklist

Use this checklist when filling out the App Store Connect Privacy Labels form:

#### Data Types Collected

- [ ] ✅ **Financial Information**
  - [ ] Subcategory: Payment Info
  - [ ] Linked to User: ✅ Yes
  - [ ] Used for Tracking: ❌ No
  - [ ] Purpose: Payment Processing, App Functionality

#### Data Sharing

- [ ] ✅ **Data is shared with third parties**
  - [ ] Third-party name: Spreedly
  - [ ] Third-party type: Service Provider
  - [ ] Purpose: Payment Processing
  - [ ] Data type: Financial Information (Payment Info)
  - [ ] Privacy policy link: https://www.spreedly.com/privacy

#### Tracking

- [ ] ❌ **No tracking performed**
  - [ ] Tracking: ❌ No
  - [ ] No tracking domains

#### Purpose

- [ ] ✅ **App Functionality** (Required)
- [ ] ✅ **Payment Processing** (Required)
- [ ] ❌ Not for Analytics, Advertising, or Personalization

### Verification Checklist

Before submitting your App Store Connect privacy labels, verify:

- [ ] ✅ Financial information collection is marked as "Yes"
- [ ] ✅ Payment Info subcategory is selected
- [ ] ✅ Data sharing with Spreedly is declared
- [ ] ✅ Spreedly's privacy policy link is included
- [ ] ✅ Tracking is marked as "No"
- [ ] ✅ Purpose is listed as "Payment Processing" and "App Functionality"
- [ ] ✅ Required for functionality is marked as "Yes"
- [ ] ✅ Linked to user is marked as "Yes"
- [ ] ✅ Used for tracking is marked as "No"

---

## Key Differences: Privacy Manifest vs. App Store Connect

| Aspect               | Privacy Manifest (PrivacyInfo.xcprivacy) | App Store Connect Privacy Labels |
| -------------------- | ---------------------------------------- | -------------------------------- |
| **Location**         | Code file in SDK                         | Web form in App Store Connect    |
| **Purpose**          | Declare API usage, data collection       | User-facing privacy disclosure   |
| **Data Sharing**     | ❌ Not included                          | ✅ Required                      |
| **Tracking Details** | Boolean only                             | Detailed tracking information    |
| **Third-Party Info** | ❌ Not included                          | ✅ Required                      |
| **User Visibility**  | Technical (for Apple)                    | Public (shown to users)          |

**Important:** Both are required:

- ✅ `PrivacyInfo.xcprivacy` - Declares SDK's technical privacy practices
- ✅ App Store Connect Privacy Labels - Declares user-facing privacy information

---

## Developer Responsibilities

### Host Application Data Practices

**Important:** The SDK's data practices are separate from your host application's practices.

When completing the App Store Connect privacy labels:

1. **SDK Data Practices:** Report SDK data collection as described in this document
2. **Host App Data Practices:** Report your application's own data collection separately
3. **Combined Reporting:** You may need to combine both in your privacy labels

### Token Storage

If your application stores payment tokens returned by the SDK:

- ✅ Report token storage in your privacy labels
- ✅ Implement user deletion mechanisms for stored tokens
- ✅ Document token storage practices in your privacy policy
- ✅ Use secure storage mechanisms (iOS Keychain)

---

## Related Documentation

- [Unified Privacy Policy](Unified_Privacy.md) - Complete privacy practices
- [Android Data Safety Guide](Android_Data_Safety_Guide.md) - Android questionnaire guide
- [Platform Privacy Requirements](Platform_Privacy_Requirements.md) - Platform comparison
- [Spreedly Privacy Policy](https://www.spreedly.com/privacy) - Third-party privacy practices

### Apple Resources

- [App Store Privacy Labels](https://developer.apple.com/app-store/app-privacy-details/)
- [Privacy Manifests Documentation](https://developer.apple.com/documentation/bundleresources/privacy_manifests)
- [App Store Connect Help](https://help.apple.com/app-store-connect/)

---

## Summary

### Privacy Manifest

✅ **Implemented** - File exists in SDK source  
✅ **Formatted** - Follows Apple's XML Property List format  
✅ **Complete** - Declares all required API usage  
✅ **Compliant** - No data collection, no tracking  
✅ **Bundled** - Automatically included via CocoaPods

### App Store Connect Privacy Labels

✅ **Data Collection:** Financial information (payment info) - Required, Linked to User  
✅ **Data Sharing:** Shared with Spreedly for payment processing  
✅ **Tracking:** No tracking performed  
✅ **Purpose:** Payment processing, App functionality

**Remember:**

- The privacy manifest is a **code implementation** requirement
- App Store Connect Privacy Labels are **user-facing** disclosures
- Both must be completed for App Store compliance

---

**Note:** Apple's privacy requirements may change. Always refer to the latest Apple documentation and ensure your implementation reflects current SDK practices.
