# Spreedly React Native SDK - Integration Guide

A comprehensive guide for integrating the Spreedly React Native Checkout SDK into your mobile application.

---

## 🔐 Security Notice

**IMPORTANT**: Payment applications handle sensitive financial data and are high-value targets for attackers. Before integrating the Spreedly SDK, please review our comprehensive [Security Best Practices](#security-best-practices) section, which covers:

- ✅ **API Key Security**: Never hardcode credentials, implement rotation policies
- ✅ **Mobile Security**: Screenshot prevention, screen recording risks, and mitigation strategies
- ✅ **Token Storage**: Never store payment tokens in AsyncStorage, UserDefaults, or SharedPreferences
- ✅ **PCI Compliance**: Mandatory use of SPLTextField for all sensitive payment fields

**Quick Security Checklist:**

- [ ] Environment keys in `.env` file (not hardcoded)
- [ ] Payment tokens sent to backend immediately (not stored locally)
- [ ] Screenshot prevention considered and implemented where appropriate
- [ ] Card number, CVV, and expiry date use `SPLTextField` components only

See the complete [Security Integration Checklist](#-security-integration-checklist) for production readiness.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Installation](#installation)
3. [Quick Start](#quick-start)
4. [Express Checkout Integration](#express-checkout-integration)
5. [3DS Challenge Integration](#3ds-challenge-integration)
6. [Hosted Fields Integration](#hosted-fields-integration)
7. [Advanced Configuration](#advanced-configuration)
8. [Payment Result Handling](#payment-result-handling)
9. [Error Handling](#error-handling)
10. [Customization](#customization)
11. [API Reference](#api-reference)
12. [Best Practices](#best-practices)
    - [Security Best Practices](#security-best-practices)
      - [API Key & Environment Key Security](#1-api-key-and-environment-key-security)
      - [Mobile App Security](#2-mobile-app-security) (Screenshot Prevention, Screen Recording)
      - [Token Storage Security](#3-token-storage-security) (Never use AsyncStorage/UserDefaults/SharedPreferences)
      - [Security Integration Checklist](#-security-integration-checklist)
    - [Performance Best Practices](#performance-best-practices)
    - [UX Best Practices](#ux-best-practices)
13. [Troubleshooting](#troubleshooting)
14. [Comprehensive Troubleshooting Checklist](#comprehensive-troubleshooting-checklist)
15. [Examples Documentation](./Examples.md)

---

## Prerequisites

### Repository Access Requirements

⚠️ **IMPORTANT**: Before integrating the Spreedly SDK, your team must have access to these private repositories:

1. **Main SDK Repository**: `https://github.com/spreedly/checkout-react-native`
2. **iOS Native Package**: `https://github.com/spreedly/checkout-ios-package`
3. **Android Native Package**: `https://github.com/spreedly/checkout-android-maven`

**To Request Access:**

- Contact your Spreedly representative or support team
- Provide GitHub usernames for team members who need access
- Ensure access is granted to all three repositories

### Authentication Requirements

After receiving repository access, you'll need:

- **GitHub Account**: With access to private Spreedly repositories
- **GitHub Personal Access Token**: With required permissions (see below)
- **Spreedly Account**: Valid Spreedly environment and API credentials
- **Spreedly Environment Key**: Provided by Spreedly (see [Getting Your Environment Key](#getting-your-environment-key))
- **Authentication Endpoint**: Backend endpoint to generate auth params (see [Authentication Parameters](#authentication-parameters))

**Required Token Permissions:**

- ✅ `read:packages` - Access GitHub Packages
- ✅ `repo` - Access private repositories
- ✅ `read:org` - Read organization membership (if applicable)

### System Requirements

**Why These Specific Versions?**

The Spreedly SDK requires modern React Native versions to leverage the latest security features, performance improvements, and native module architecture enhancements essential for payment processing.

- **React Native**: 0.76+ (recommended 0.79+)
  - **Required for**: New Architecture support, improved native module performance, and security patches
  - **0.76+**: Minimum version with stable Fabric/TurboModules support
  - **0.79+**: Recommended for latest security updates and performance optimizations
- **React**: 19.x
  - **Required for**: React Native 0.76+ compatibility and modern hook implementations
  - **Concurrent Features**: Enables better performance for payment form interactions
- **Node.js**: 18+
  - **Required for**: Modern JavaScript features, better package resolution, and build tooling
  - **LTS Support**: Ensures stable development environment
- **TypeScript**: 4.5+ (optional but recommended)
  - **Type Safety**: Prevents common integration errors with strongly-typed payment APIs
  - **Developer Experience**: Enhanced IDE support and auto-completion for SDK methods
  - **Runtime Safety**: Catches payment-related type mismatches at compile time

### Platform Support

**Android:**

- Minimum SDK: 24 (Android 7.0 Nougat)
- Target SDK: 34
- Compile SDK: 35
- NDK: 27.1.12297006
- **Kotlin**: 2.0.21+ (required for compatibility)
- **Android Gradle Plugin**: 8.7.2+ (required for Kotlin 2.0.21+)
- **Gradle**: 8.9+ (automatically managed by wrapper)

**iOS:**

- iOS: 15.1+
- Xcode: 15+
- CocoaPods: Latest

### Architecture Support

- ✅ **Legacy Architecture**: Full support
- ✅ **New Architecture** (Fabric/TurboModules): Full support

---

## Authentication Parameters

### Getting Your Environment Key

**What is an Environment Key?**

The Environment Key is a unique identifier for your Spreedly environment that's required to initialize the SDK. This key determines which Spreedly environment (test or production) your app will connect to.

**How to Get Your Environment Key:**

1. **Contact Spreedly**: Your Environment Key will be provided by your Spreedly representative or support team
2. **Environment Types**:
   - **Test Environment**: For development and testing (e.g., `test_abc123def456`)
   - **Production Environment**: For live transactions (e.g., `prod_xyz789ghi012`)
3. **Security**: Store this key securely and never commit it to version control

**Example Configuration:**

```bash
# .env file (add to .gitignore)
SPREEDLY_ENVIRONMENT_KEY=test_your_environment_key_here
FORTER_SITE_ID=your_forter_site_id_here  # Leave empty or omit if not using Forter
```

### Forter Integration (Optional)

**What is Forter?**

Forter is an AI-powered fraud prevention platform that helps protect against payment fraud. The Spreedly SDK supports optional Forter integration for enhanced fraud detection.

**How to Get Your Forter Site ID:**

1. **Contact Forter**: If you're a Forter customer, your Site ID will be provided by your Forter representative
2. **Sandbox vs. Production**: Use sandbox Site ID for testing, production Site ID for live transactions
3. **Configuration**: Add the Site ID to your `.env` file

**When to Use Forter Integration:**

- ✅ If you have an existing Forter account
- ✅ When you need advanced fraud detection
- ✅ For high-value transactions requiring additional protection

**If Not Using Forter:**

Pass an empty string for `forterSiteId`:

```typescript
forterSiteId: '', // Required parameter, use empty string if not using Forter
```

### Authentication Parameters from Backend

**Critical Security Requirement**: The SDK requires dynamic authentication parameters that **must be generated by your secure backend** for each session. Never hardcode these values in your mobile app.

**Required Parameters:**

- `nonce`: Unique random string for each request
- `signature`: HMAC signature of the request
- `certificateToken`: Certificate token for the session
- `timestamp`: Unix timestamp of when the params were generated

**Backend Implementation Required:**

Your backend must implement an endpoint (commonly `/api/v1/auth/params`) that:

1. **Generates a unique nonce** for each request
2. **Creates an HMAC signature** using your Spreedly secret key
3. **Issues a certificate token** for the session
4. **Returns current timestamp** for request validation

**Example Backend Response:**

```json
{
  "nonce": "abc123def456ghi789",
  "signature": "hmac_sha256_signature_here",
  "certificateToken": "cert_token_abc123",
  "timestamp": 1640995200
}
```

**Frontend Implementation:**

```typescript
// Fetch fresh auth params from your backend
const fetchAuthParams = async () => {
  const response = await fetch('https://your-backend.com/api/v1/auth/params', {
    method: 'GET',
    headers: {
      'Authorization': 'Bearer your_user_token',
      'Content-Type': 'application/json',
    },
  });

  if (!response.ok) {
    throw new Error('Failed to fetch auth params');
  }

  return await response.json();
};

// Use in SDK initialization
const initializeSpreedly = async () => {
  const authParams = await fetchAuthParams();

  await SpreedlyCore.initSdk({
    token: authParams.certificateToken,
    nonce: authParams.nonce,
    signature: authParams.signature,
    certificateToken: authParams.certificateToken,
    timestamp: authParams.timestamp.toString(),
    environmentKey: process.env.SPREEDLY_ENVIRONMENT_KEY, // From Spreedly
    forterSiteId: process.env.FORTER_SITE_ID || '', // Empty string if not using Forter
  });
};
```

**🔒 Security Best Practices:**

**DO:**

- ✅ Always fetch fresh params for each SDK initialization
- ✅ Validate user authentication before generating auth params
- ✅ Use HTTPS for all auth param requests
- ✅ Implement rate limiting on your auth endpoint

**DON'T:**

- ❌ Never hardcode nonce, signature, or certificate tokens
- ❌ Never store auth params in local storage or app state

**🎯 Why This Architecture?**

This backend-first approach ensures **security**, **compliance**, and **fraud prevention** by keeping sensitive operations on your secure server with full audit trails.

---

## Installation

### Step 1: Configure Private Repository Access

Since the Spreedly SDK is distributed via GitHub Packages, you need to configure access to the private repository.

#### GitHub Token Setup (Required)

1. **Create GitHub Personal Access Token**:
   - Go to [GitHub Settings > Developer settings > Personal access tokens > Tokens (classic)](https://github.com/settings/tokens)
   - Click "Generate new token (classic)"
   - Give it a descriptive name (e.g., "Spreedly SDK Access")
   - Select scopes:
     - ✅ `read:packages` - Access GitHub Packages
     - ✅ `repo` - Access private repositories
     - ✅ `read:org` - Read organization membership (if applicable)
   - Click "Generate token"
   - **Important**: Copy and save the token immediately (you won't see it again)

2. **Save Your Credentials**:
   Keep these handy for the setup process:
   - **GITHUB_USERNAME**: Your GitHub username
   - **GITHUB_TOKEN**: The personal access token you just created

#### Configure Package Manager for GitHub Packages

**Option 1: Yarn Configuration**

For **Yarn v2+ (Berry)**, create a `.yarnrc.yml` file in your project root:

```yaml
npmScopes:
  spreedly:
    npmRegistryServer: 'https://npm.pkg.github.com'
    npmAuthToken: 'YOUR_GITHUB_TOKEN'

npmRegistryServer: 'https://registry.npmjs.org'
```

For **Yarn v1 (Classic)**, create a `.yarnrc` file in your project root:

```
"@spreedly:registry" "https://npm.pkg.github.com"
"//npm.pkg.github.com/:_authToken" "YOUR_GITHUB_TOKEN"
registry "https://registry.npmjs.org"
```

**Option 2: Project-specific NPM configuration**

Create a `.npmrc` file in your project root:

```bash
@spreedly:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken=${GITHUB_TOKEN}
```

Then set the environment variable:

```bash
export GITHUB_TOKEN=your_github_personal_access_token
```

**Configuration Templates:**
You can find configuration templates in the SDK package after installation:

- `node_modules/@spreedly/react-native-checkout/templates/.npmrc.template`
- `node_modules/@spreedly/react-native-checkout/templates/.yarnrc.yml.template`
- `node_modules/@spreedly/react-native-checkout/templates/.yarnrc.template`

**Option 3: NPM Configuration**

```bash
# Configure npm to use GitHub Packages for @spreedly scope
npm config set @spreedly:registry https://npm.pkg.github.com

# Authenticate with your GitHub token
npm config set //npm.pkg.github.com/:_authToken YOUR_GITHUB_TOKEN
```

### Step 2: Set Environment Variables

⚠️ **IMPORTANT**: Set these environment variables **before** installing the package, as the installation process needs access to private native packages.

**Create `.env` file in your project root:**

```bash
# .env
GITHUB_USERNAME=your_github_username
GITHUB_TOKEN=your_github_personal_access_token
```

**Or set environment variables directly:**

```bash
export GITHUB_USERNAME=your_github_username
export GITHUB_TOKEN=your_github_personal_access_token
```

**Add to `.gitignore`:**

```bash
echo ".env" >> .gitignore
```

### Step 3: Install the Package

```bash
# Using yarn
yarn add @spreedly/react-native-checkout

# Using npm
npm install @spreedly/react-native-checkout
```

### Step 4: iOS Setup

#### Configure Private iOS Dependencies

The Spreedly SDK uses private iOS packages that require additional configuration.

**Update your `ios/Podfile`:**

You need to add **two specific lines** to your existing Podfile:

1. **Add the require statement** at the top (after any existing requires)
2. **Add the init function call** inside your target block

```ruby
# ios/Podfile

# 1. ADD THIS REQUIRE STATEMENT (at the top, after existing requires)
require Pod::Executable.execute_command('node', ['-p',
  'require.resolve(
    "@spreedly/react-native-checkout/scripts/spreedly_pods_setup.rb",
    {paths: [process.argv[1]]},
  )', __dir__]).strip

# Your existing Podfile content remains the same...
platform :ios, min_ios_version_supported
prepare_react_native_project!

target 'YourAppName' do
  config = use_native_modules!

  use_react_native!(
    :path => config[:reactNativePath],
    # Your existing React Native configurations
  )

  # 2. ADD THIS FUNCTION CALL (inside your target block)
  init_spreedly_checkout_pods()

  # Your existing pods remain unchanged...
end

# Your existing post_install block remains the same...
post_install do |installer|
  react_native_post_install(
    installer,
    config[:reactNativePath],
    :mac_catalyst_enabled => false
  )
end
```

**What These Changes Do:**

- **Line 1**: Loads the Spreedly setup script that configures access to private iOS dependencies
- **Line 2**: Initializes Spreedly-specific CocoaPods that are required for the SDK to function

**⚠️ Important**: Don't replace your entire Podfile - just add these two lines to your existing configuration.

#### Install Pods

**After updating your Podfile with the two required lines above:**

**Option 1: Standard Installation (Credentials in Podfile.lock)**

```bash
cd ios
bundle install
bundle exec pod install
cd ..
```

**Option 2: Secure Installation (Recommended for Teams)**

To prevent GitHub credentials from being saved in `Podfile.lock`, use the setup script:

**Prerequisites**: Ensure you have a `.env` file in your project root with your GitHub credentials:

```bash
# Create .env file in your project root (not in node_modules)
echo "GITHUB_USERNAME=your_github_username" > .env
echo "GITHUB_TOKEN=your_github_personal_access_token" >> .env
echo ".env" >> .gitignore
```

**Run the setup script**:

```bash
# Run the secure setup script from your project root
# The script will automatically find your .env file
./node_modules/@spreedly/react-native-checkout/scripts/setup_local_dev_ios.sh

# Then install pods
cd ios
bundle install
bundle exec pod install
cd ..
```

**What the setup script does:**

- Looks for `.env` file in your project root (recommended location)
- Configures Git URL rewriting for private repositories
- Sets up environment variables securely in your shell configuration
- Prevents credentials from appearing in `Podfile.lock`
- Creates a backup of your shell configuration

**Script search order for `.env` file:**

1. `./.env` (your project root - recommended)
2. `./example/.env` (your project's example directory)
3. SDK's example directory (fallback)
4. SDK's root directory (fallback)

⚠️ **Security Note**: The setup script modifies your shell configuration (`.zshrc`, `.bashrc`, etc.) to add environment variables. A backup is created automatically.

### Step 5: Android Setup

#### Version Compatibility Check

⚠️ **IMPORTANT**: Before configuring Android dependencies, ensure your project uses compatible versions:

**Check your `android/build.gradle` file and update if necessary:**

```gradle
// android/build.gradle
buildscript {
    ext {
        buildToolsVersion = "35.0.0"
        minSdkVersion = 26
        compileSdkVersion = 35
        targetSdkVersion = 34
        ndkVersion = "27.1.12297006"

        // REQUIRED: Kotlin version must be 2.0.21+ for Spreedly SDK compatibility
        kotlinVersion = "2.0.21"
    }
    dependencies {
        // REQUIRED: Android Gradle Plugin 8.7.2+ for Kotlin 2.0.21+ support
        classpath("com.android.tools.build:gradle:8.7.2")
        classpath("org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlinVersion")
    }
}
```

**Why these versions are required:**

- **Kotlin 2.0.21+**: The Spreedly SDK uses modern Kotlin features and Compose integration
- **Android Gradle Plugin 8.7.2+**: Required for Kotlin 2.0.21+ compatibility
- **NDK 27.1.12297006**: Required for React Native 0.76+ native module compilation

**Version Compatibility Matrix:**

| React Native | Kotlin  | Android Gradle Plugin | Gradle | Status                 |
| ------------ | ------- | --------------------- | ------ | ---------------------- |
| 0.76+        | 2.0.21+ | 8.7.2+                | 8.9+   | ✅ **Supported**       |
| 0.75         | 1.9.25  | 8.5.2                 | 8.8    | ⚠️ **Not Recommended** |
| 0.72-0.74    | 1.8.x   | 8.x                   | 8.x    | ❌ **Not Supported**   |

⚠️ **Important**: If you're upgrading from an older React Native version, you must update all these versions together to avoid compatibility issues.

#### Configure Private Android Dependencies

The Spreedly SDK uses private Android packages that require additional configuration.

**Update your `android/build.gradle`:**

```gradle
// android/build.gradle
apply from: "../node_modules/@spreedly/react-native-checkout/scripts/spreedly_github_setup.gradle"

buildscript {
    // your existing buildscript configuration
}

allprojects {
    // Repository configuration is handled by spreedly_github_setup.gradle
    // No additional repository configuration needed
}
```

**What the Gradle setup does:**

- Configures access to GitHub Packages Maven repository
- Reads GitHub credentials from environment variables or `.env` file
- Automatically handles authentication for private Android dependencies
- Maintains fallback to standard repositories (Google, Maven Central)

#### Verify Android Setup

Test that the Android build can access private dependencies:

```bash
cd android
# Use the correct command that works with the project structure
./gradlew :app:dependencies --configuration implementation | grep -i spreedly
```

**Alternative verification commands:**

```bash
# If the above doesn't work, try these alternatives:
./gradlew dependencies | grep -i spreedly
./gradlew :app:dependencies | grep -i spreedly

# To see all available project modules:
./gradlew projects
```

You should see Spreedly dependencies listed if the setup is correct. Look for entries like:

```
+--- com.spreedly:checkout-android:x.x.x
```

### Step 6: Configuration

For monorepo projects, ensure your `metro.config.js` includes the SDK:

```javascript
// metro.config.js
const { getDefaultConfig } = require('@react-native/metro-config');
const defaultConfig = getDefaultConfig(__dirname);

module.exports = {
  ...defaultConfig,
  resolver: {
    ...defaultConfig.resolver,
    assetExts: [...defaultConfig.resolver.assetExts, 'bin'],
  },
};
```

---

## Quick Setup Checklist

Before proceeding with integration, ensure you have completed these steps:

### ✅ Pre-Integration Checklist

- [ ] **Repository Access**: Confirmed access to all three private repositories
- [ ] **GitHub Token**: Created with `read:packages`, `repo`, and `read:org` permissions
- [ ] **Credentials Ready**: `GITHUB_USERNAME` and `GITHUB_TOKEN` available
- [ ] **Package Manager**: Configured for GitHub Packages (NPM/Yarn)
- [ ] **Version Compatibility**: Verified Kotlin 2.0.21+ and Android Gradle Plugin 8.7.2+
- [ ] **iOS Configuration**: Updated Podfile with `spreedly_pods_setup.rb`
- [ ] **Android Configuration**: Updated `build.gradle` with `spreedly_github_setup.gradle`
- [ ] **Environment Variables**: Set `GITHUB_USERNAME` and `GITHUB_TOKEN`
- [ ] **Security Setup**: Used secure iOS setup script (optional but recommended)

### 🚀 Quick Installation Commands

```bash
# 1. Check version compatibility (IMPORTANT for Android)
# Ensure your android/build.gradle has:
# - kotlinVersion = "2.0.21" (or higher)
# - Android Gradle Plugin 8.7.2+

# 2. Configure package manager (choose one)
npm config set @spreedly:registry https://npm.pkg.github.com
npm config set //npm.pkg.github.com/:_authToken YOUR_GITHUB_TOKEN

# 3. Set environment variables (REQUIRED before installation)
export GITHUB_USERNAME=your_username
export GITHUB_TOKEN=your_token

# 4. Install SDK
npm install @spreedly/react-native-checkout

# 5. iOS setup (secure method)
sh ./node_modules/@spreedly/react-native-checkout/scripts/setup_local_dev_ios.sh
cd ios && pod install && cd ..

# 6. Verify Android setup and build
cd android && ./gradlew dependencies --configuration implementation | grep spreedly
./gradlew build  # Verify build works with correct Kotlin version
```

---

## 🚀 Quick Reference

### Most Common Integration Patterns

**1. Basic Payment Form (Recommended for most apps):**

```typescript
// Three essential fields for card payments
<SPLTextField formFieldType={FormFieldTypes.CARD} label="Card Number" />
<SPLTextField formFieldType={FormFieldTypes.EXPIRY_DATE} label="MM/YY" />
<SPLTextField formFieldType={FormFieldTypes.CVV} label="CVV" />
```

**2. Express Checkout (Fastest integration):**

```typescript
// One-line payment solution
SpreedlyCore.paymentBottomSheet({ yearFormat: '4' });
```

**3. SDK Initialization (Required first step):**

```typescript
// Always fetch fresh auth params from your backend
const authParams = await fetchAuthParams();
await SpreedlyCore.initSdk({
  ...authParams,
  environmentKey: process.env.SPREEDLY_ENVIRONMENT_KEY,
});
```

**4. Handle Payment Results:**

```typescript
// Use SDK's result mapping for consistent handling
const mapped = mapPaymentResult(result);
if (mapped.kind === 'success') {
  // Send mapped.token to your backend
}
```

### ⚡ Common Gotchas to Avoid

- ❌ **Don't use** `fieldType` → ✅ **Use** `formFieldType`
- ❌ **Don't use** `placeholder` → ✅ **Use** `label`
- ❌ **Don't hardcode** auth params → ✅ **Fetch from backend**
- ❌ **Don't use** `TextInput` for card data → ✅ **Use** `SPLTextField`

---

## Quick Start

### ⚠️ Critical: SDK Initialization Timing

**IMPORTANT**: Always wait for SDK initialization to complete before rendering SPLTextField or ExpressCheckout components. Rendering these components before the SDK is ready can cause errors on Android and iOS.

```typescript
// ✅ CORRECT: Wait for SDK initialization
const { isLoading } = useSpreedlyInit();

if (isLoading) {
  return <ActivityIndicator />;
}

return <SPLTextField formFieldType={FormFieldTypes.CARD} />;
```

```typescript
// ❌ WRONG: Renders immediately, can cause windowRecomposer error
return <SPLTextField formFieldType={FormFieldTypes.CARD} />;
```

For detailed troubleshooting of React Navigation integration issues, see the [React Navigation Integration Guide](./React_Navigation_Integration_Guide.md).

### Basic SDK Initialization

```typescript
import React, { useEffect, useState } from 'react';
import { ActivityIndicator } from 'react-native';
import { SpreedlyCore, type SpreedlySDKInitOptions } from '@spreedly/react-native-checkout';

export function App() {
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    const initializeSpreedly = async () => {
      try {
        setIsLoading(true);
        // Fetch fresh auth params from your backend (see Authentication Parameters section)
        const authParams = await fetchAuthParams();

        const options: SpreedlySDKInitOptions = {
          token: authParams.certificateToken,
          nonce: authParams.nonce,
          signature: authParams.signature,
          certificateToken: authParams.certificateToken,
          timestamp: authParams.timestamp.toString(),
          environmentKey: process.env.SPREEDLY_ENVIRONMENT_KEY, // From Spreedly
          forterSiteId: process.env.FORTER_SITE_ID || '', // Empty string if not using Forter
        };

        await SpreedlyCore.initSdk(options);
        console.log('✅ Spreedly SDK initialized successfully');
      } catch (error) {
        console.error('❌ Failed to initialize Spreedly SDK:', error);
      } finally {
        setIsLoading(false);
      }
    };

    initializeSpreedly();
  }, []);

  // Don't render Spreedly components until SDK is ready
  if (isLoading) {
    return <ActivityIndicator style={{ flex: 1 }} />;
  }

  return (
    // Your app components with Spreedly fields
  );
}

// Helper function to fetch auth params from your backend
const fetchAuthParams = async () => {
  const response = await fetch('https://your-backend.com/api/v1/auth/params');
  if (!response.ok) {
    throw new Error('Failed to fetch auth params');
  }
  return await response.json();
};
```

---

## Express Checkout Integration

The Express Checkout provides a pre-built payment form with minimal integration effort, designed for rapid deployment and consistent user experience across different apps.

**When to Use Express Checkout:**

- **Rapid Prototyping**: Get a payment flow up and running quickly for demos or MVPs
- **Standard Checkout Flow**: When your payment requirements align with common e-commerce patterns
- **Minimal Customization Needed**: When the default UI and behavior meet your needs
- **Team Efficiency**: Reduce development time and focus on core business logic
- **Consistent UX**: Leverage Spreedly's tested and optimized payment experience

**Express Checkout Features:**

- ✅ **Pre-built UI Components**: Card number, expiry, CVV, and name fields with built-in validation
- ✅ **Responsive Design**: Automatically adapts to different screen sizes and orientations
- ✅ **Accessibility Support**: Full VoiceOver/TalkBack support and keyboard navigation
- ✅ **Error Handling**: Built-in validation messages and error states
- ✅ **Theme Customization**: Apply your brand colors and styling
- ✅ **Security Compliance**: PCI-compliant hosted fields that never expose card data
- ✅ **Cross-Platform**: Identical behavior on iOS and Android

**Comparison with Hosted Fields:**

| Feature            | Express Checkout | Hosted Fields            |
| ------------------ | ---------------- | ------------------------ |
| **Setup Time**     | ⚡ Minutes       | 🔧 Hours                 |
| **Customization**  | 🎨 Theme-based   | 🎨 Full control          |
| **Validation**     | ✅ Built-in      | 🔧 Manual setup          |
| **Error Handling** | ✅ Automatic     | 🔧 Custom implementation |
| **Best For**       | Standard flows   | Custom experiences       |

### Basic Payment Bottom Sheet

```typescript
import React, { useEffect, useState } from 'react';
import { View, Button, Alert, Text } from 'react-native';
import {
  SpreedlyCore,
  SpreedlyEventEmitter,
  SpreedlyEventTypes,
  type PaymentResultRN,
  mapPaymentResult
} from '@spreedly/react-native-checkout';

export function ExpressCheckout() {
  const [paymentResult, setPaymentResult] = useState<string | null>(null);

  useEffect(() => {
    // Listen for payment results
    const handlePaymentResult = (result: PaymentResultRN) => {
      const mapped = mapPaymentResult(result);

      switch (mapped.kind) {
        case 'success':
          setPaymentResult(mapped.token);
          Alert.alert('Success', 'Payment completed successfully!');
          break;
        case 'failed':
          Alert.alert('Error', mapped.message);
          break;
        case 'canceled':
          Alert.alert('Canceled', 'Payment was canceled');
          break;
        case 'validation':
          Alert.alert('Validation Error', mapped.message);
          break;
      }
    };

    // Add event listener for payment results using the SDK's event emitter
    const subscription = SpreedlyEventEmitter.addListener(
      SpreedlyEventTypes.PAYMENT_BOTTOM_SHEET_RESULT,
      handlePaymentResult
    );

    return () => {
      subscription.remove();
    };
  }, []);

  const handlePaymentPress = () => {
    SpreedlyCore.paymentBottomSheet({
      allowBlankName: false,
      allowExpiredDate: false,
      yearFormat: '4',
      nameDisplayMode: 'singleField',
    });
  };

  return (
    <View style={{ padding: 20 }}>
      <Button
        title="Start Payment"
        onPress={handlePaymentPress}
      />
      {paymentResult && (
        <Text>Payment Token: {paymentResult}</Text>
      )}
    </View>
  );
}
```

### With Custom Theme

You can customize the appearance of payment components to match your brand. For comprehensive theming documentation including dark mode support, see the [Theme Customization Guide](./Theme_Guide.md).

**Quick Example:**

```typescript
import { SpreedlyCore } from '@spreedly/react-native-checkout';

SpreedlyCore.paymentBottomSheet({
  allowBlankName: false,
  allowExpiredDate: false,
  yearFormat: '4',
  nameDisplayMode: 'singleField',
  theme: {
    primaryColor: '#6366F1',
    secondaryColor: '#8B5CF6',
    formBorderColor: '#D1D5DB',
    formBackgroundColor: '#FFFFFF',
    fieldBackgroundColor: '#F9FAFB',
    fieldLabelColor: '#6B7280',
    borderRadius: 8,
    fieldShape: 'rounded',
  },
});
```

**📖 For complete theming documentation**: See [Theme_Guide.md](./Theme_Guide.md) for:

- Dark mode support
- Global theme configuration
- Component-level theming
- Pre-built theme examples
- Platform-specific behavior

---

## 3DS Challenge Integration

The Spreedly SDK provides built-in support for 3D Secure (3DS) authentication, which adds an additional layer of security for card-not-present transactions. The SDK handles the entire 3DS challenge UI and flow - your application only needs to initiate the purchase on your backend and trigger the SDK's challenge presentation.

**When to Use 3DS:**

- **Regulatory Compliance**: SCA (Strong Customer Authentication) requirements in Europe
- **Fraud Reduction**: Shift liability for fraudulent transactions to the card issuer
- **Higher Authorization Rates**: Banks may approve more transactions with 3DS verification
- **Customer Trust**: Visual security indicators increase checkout confidence

**3DS Flow Overview:**

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Your App       │     │  Your Backend   │     │  Spreedly API   │
│                 │     │                 │     │                 │
│  1. User taps   │────>│  2. Process     │────>│  3. Returns     │
│     "Pay"       │     │     Purchase    │     │     tokens      │
│                 │<────│                 │<────│                 │
│                 │     │                 │     │                 │
│  4. Call SDK    │     │                 │     │                 │
│     showThreeDS │     │                 │     │                 │
│     Challenge() │     │                 │     │                 │
│                 │     │                 │     │                 │
│  ┌───────────┐  │     │                 │     │                 │
│  │ SDK 3DS   │  │     │                 │     │                 │
│  │ Challenge │  │     │                 │     │                 │
│  │ UI        │  │     │                 │     │                 │
│  └───────────┘  │     │                 │     │                 │
│                 │     │                 │     │                 │
│  5. Listen for  │     │                 │     │                 │
│     result      │     │                 │     │                 │
│     event       │     │                 │     │                 │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

**Key Responsibilities:**

| Component        | Responsibility                                                 |
| ---------------- | -------------------------------------------------------------- |
| **Your Backend** | Process purchase via Spreedly API, return tokens               |
| **Your App**     | Initiate purchase, trigger 3DS challenge, handle result        |
| **Spreedly SDK** | Display 3DS challenge UI, handle user interaction, emit result |

### Basic 3DS Integration

```typescript
import React, { useEffect, useState } from 'react';
import { View, Button, Alert, Text, ActivityIndicator } from 'react-native';
import {
  SpreedlyCore,
  SpreedlyEventEmitter,
  SpreedlyEventTypes,
  type ThreeDSChallengeResult,
} from '@spreedly/react-native-checkout';

export function ThreeDSCheckout() {
  const [isProcessing, setIsProcessing] = useState(false);
  const [paymentResult, setPaymentResult] = useState<string | null>(null);
  const [errorMessage, setErrorMessage] = useState<string | null>(null);

  useEffect(() => {
    // Listen for 3DS challenge results from the SDK
    const subscription = SpreedlyEventEmitter.addListener(
      SpreedlyEventTypes.THREE_DS_CHALLENGE_RESULT,
      (result: ThreeDSChallengeResult) => {
        switch (result.status) {
          case 'success':
            console.log('3DS Challenge successful', result.transactionId);
            setPaymentResult('Payment completed successfully!');
            setErrorMessage(null);
            // Optionally notify your backend of successful authentication
            break;

          case 'failed':
            console.log('3DS Challenge failed:', result.message);
            setPaymentResult(null);
            setErrorMessage(result.message || '3DS verification failed');
            break;
        }
        setIsProcessing(false);
      }
    );

    // Cleanup subscription on unmount
    return () => {
      subscription.remove();
    };
  }, []);

  const handleCheckout = async () => {
    setIsProcessing(true);
    setErrorMessage(null);
    setPaymentResult(null);

    try {
      // Step 1: Call YOUR backend to process the purchase
      // This is the ONLY part that happens on your server
      const purchaseResult = await processPurchaseOnBackend({
        paymentMethodToken: 'pm_xxx', // Token from createCreditCard or saved payment method
        amount: 9999, // Amount in cents
        currencyCode: 'USD',
      });

      // Step 2: Extract tokens from your backend response
      const { managedOrderToken, transactionToken } = purchaseResult;

      if (managedOrderToken && transactionToken) {
        // Step 3: Show 3DS challenge using the SDK
        // The SDK handles all the 3DS UI and flow automatically
        SpreedlyCore.showThreeDSChallenge(managedOrderToken, transactionToken);
      } else {
        setErrorMessage('Missing required tokens for 3DS challenge');
        setIsProcessing(false);
      }
    } catch (error) {
      console.error('Error processing purchase:', error);
      setErrorMessage((error as Error).message || 'Failed to process purchase');
      setIsProcessing(false);
    }
  };

  return (
    <View style={{ padding: 20 }}>
      <Button
        title={isProcessing ? 'Processing...' : 'Pay Now'}
        onPress={handleCheckout}
        disabled={isProcessing}
      />

      {isProcessing && <ActivityIndicator style={{ marginTop: 20 }} />}

      {paymentResult && (
        <Text style={{ marginTop: 20, color: 'green' }}>{paymentResult}</Text>
      )}

      {errorMessage && (
        <Text style={{ marginTop: 20, color: 'red' }}>{errorMessage}</Text>
      )}
    </View>
  );
}

// Example backend API call - implement according to your backend
async function processPurchaseOnBackend(params: {
    paymentMethodToken: string;
    amount: number;
    currencyCode: string;
  }): Promise<{ managedOrderToken: string | null; transactionToken: string | null }> {
    const { paymentMethodToken, amount, currencyCode } = params;

    // Build request body with snake_case field names as expected by API
    const requestBody = {
      amount: amount,
      currency_code: currencyCode,
      payment_method_token: paymentMethodToken,
    };

    try {
      const response = await fetch(
        'https://your-backend.com/api/purchase',
        {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            Accept: 'application/json',
          },
          body: JSON.stringify(requestBody),
        }
      );

      const data = await response.json();

      if (!response.ok) {
        const error = new Error(
          data.errors?.[0]?.message || 'Purchase request failed'
        );
        throw error;
      }

      // Extract tokens from the response
      const managedOrderToken =
        data?.transaction?.sca_authentication?.managed_order_token ?? null;
      const transactionToken = data?.transaction?.token ?? null;

      return { managedOrderToken, transactionToken };
    } catch (error) {
      if (
        error instanceof Error &&
        !error.message.includes('Purchase request failed')
      ) {
        const enhancedError = new Error(
          `Network error during purchase: ${error.message}`
        );
        throw enhancedError;
      }
      throw error;
    }
  }
```

### 3DS API Reference

#### `SpreedlyCore.showThreeDSChallenge(managedOrderToken, transactionToken)`

Displays the 3DS challenge UI to the user. The SDK handles all user interaction and emits a result event when complete.

**Parameters:**

| Parameter           | Type     | Description                                                           |
| ------------------- | -------- | --------------------------------------------------------------------- |
| `managedOrderToken` | `string` | The managed order token from your backend's Spreedly API response     |
| `transactionToken`  | `string` | The transaction token from your backend's purchase/authorize response |

**Example:**

```typescript
SpreedlyCore.showThreeDSChallenge(
  'mot_abc123xyz789', // managedOrderToken from backend
  'txn_def456uvw012' // transactionToken from backend
);
```

#### `SpreedlyCore.hideThreeDSChallenge()`

Programmatically dismisses the 3DS challenge UI. Use this for edge cases where you need to close the challenge (e.g., session timeout).

**Example:**

```typescript
// Close challenge programmatically (use sparingly)
SpreedlyCore.hideThreeDSChallenge();
```

#### `SpreedlyEventTypes.THREE_DS_CHALLENGE_RESULT`

Event emitted when the 3DS challenge completes or fails.

**Event Data Type: `ThreeDSChallengeResult`**

```typescript
type ThreeDSChallengeResult =
  | { status: 'success'; transactionId?: string }
  | { status: 'failed'; message?: string };
```

**Result Status Values:**

| Status    | Description                             | Action                                         |
| --------- | --------------------------------------- | ---------------------------------------------- |
| `success` | 3DS verification completed successfully | Proceed with order fulfillment                 |
| `failed`  | 3DS verification failed                 | Show error, allow retry or alternative payment |

### Backend Integration Requirements

Your backend must implement the purchase API endpoint that calls Spreedly's API and returns the required tokens:

**Required Backend Response:**

```json
{
  "managedOrderToken": "mot_abc123xyz789",
  "transactionToken": "txn_def456uvw012"
}
```

**Backend Implementation Notes:**

1. **Use Spreedly's Purchase/Authorize API** with 3DS enabled
2. **Extract `managed_order_token`** from the API response
3. **Extract `transaction.token`** from the API response
4. **Return both tokens** to your mobile app
5. **Handle backend errors** gracefully and return appropriate error messages

### Error Handling Best Practices

```typescript
useEffect(() => {
  const subscription = SpreedlyEventEmitter.addListener(
    SpreedlyEventTypes.THREE_DS_CHALLENGE_RESULT,
    (result: ThreeDSChallengeResult) => {
      switch (result.status) {
        case 'success':
          // ✅ Success - proceed with order
          handlePaymentSuccess(result.transactionId);
          break;

        case 'failed':
          // ❌ Failed - show user-friendly error
          if (result.message?.includes('timeout')) {
            showError('Verification timed out. Please try again.');
          } else if (result.message?.includes('declined')) {
            showError('Card verification failed. Please try a different card.');
          } else {
            showError(
              result.message || 'Verification failed. Please try again.'
            );
          }
          // Allow user to retry or choose different payment method
          break;
      }
    }
  );

  return () => subscription.remove();
}, []);
```

### Security Considerations for 3DS

- **Never expose** Spreedly API credentials in your mobile app
- **Always process purchases** through your secure backend
- **Validate tokens** on your backend before using them
- **Implement session timeouts** for incomplete 3DS flows
- **Log 3DS events** for compliance and debugging (without sensitive data)

---

## Hosted Fields Integration

For custom checkout flows, use individual hosted field components that give you complete control over the user experience while maintaining PCI compliance.

**When to Use Hosted Fields:**

- **Custom UI Requirements**: When you need specific layouts, animations, or design patterns
- **Complex Validation Logic**: When you need custom validation rules beyond standard card validation
- **Multi-Step Flows**: For wizard-style checkouts or progressive disclosure patterns
- **Advanced Integration**: When integrating with existing form libraries or state management
- **Brand-Specific Experience**: When Express Checkout doesn't match your design system

**Hosted Fields Architecture:**

Hosted Fields use a secure iframe-like approach where sensitive payment data never touches your application code:

```
┌─────────────────────────────────────────┐
│ Your React Native App                   │
│ ┌─────────────────────────────────────┐ │
│ │ SPLTextField (Secure Container)     │ │
│ │ ┌─────────────────────────────────┐ │ │
│ │ │ Native Payment Field            │ │ │
│ │ │ (PCI Compliant - Isolated)      │ │ │
│ │ └─────────────────────────────────┘ │ │
│ └─────────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

**Key Benefits:**

- 🔒 **PCI Compliance**: Card data never enters your app's memory or storage
- 🎨 **Full Customization**: Complete control over styling, layout, and behavior
- ⚡ **Performance**: Native implementation for smooth user interactions
- 🛡️ **Security**: Encrypted communication between fields and Spreedly servers
- 🔧 **Flexibility**: Mix and match fields based on your requirements

**Security Model:**

- **Data Isolation**: Payment data is processed in isolated native components
- **Token-Based**: Your app only receives secure payment tokens, never raw card data
- **Encrypted Transit**: All communication uses TLS encryption
- **No Storage**: Card data is never persisted on the device

### 🔒 Critical Security Requirements

**MANDATORY: Sensitive Payment Fields**

The following fields contain sensitive payment data and **MUST ONLY** be implemented using `SPLTextField` components:

- 💳 **Card Number** (`FormFieldTypes.CARD`)
- 🔐 **CVV/Security Code** (`FormFieldTypes.CVV`)
- 📅 **Expiry Date** (`FormFieldTypes.EXPIRY_DATE`)

**⚠️ PCI Compliance Requirement**: These fields cannot be implemented as custom TextInput components or sent as additional fields during payment submission.

```typescript
// ✅ CORRECT - Use SPLTextField for ALL sensitive payment data
<SPLTextField formFieldType={FormFieldTypes.CARD} label="Card Number" />
<SPLTextField formFieldType={FormFieldTypes.CVV} label="CVV" />
<SPLTextField formFieldType={FormFieldTypes.EXPIRY_DATE} label="MM/YY" />

// ❌ INCORRECT - Never use custom fields for sensitive data
<TextInput placeholder="Card Number" />           // Violates PCI compliance
<CustomField fieldType="card_number" />          // Not allowed
<View><Text>Enter CVV:</Text><TextInput /></View> // Security violation
```

**Why This Restriction Exists:**

- 🔒 **PCI DSS Compliance**: Card data must be handled in PCI-compliant environments
- 🚫 **Data Isolation**: Sensitive data never enters your application's memory space
- 📜 **Regulatory Requirements**: Payment industry regulations mandate secure handling
- 🔍 **Fraud Prevention**: Prevents accidental exposure or logging of payment information
- 🛡️ **Liability Protection**: Reduces your PCI compliance scope and liability

**Non-Sensitive Fields (Can Use Custom Implementation):**

These fields can be implemented using either `SPLTextField` or custom `TextInput` components:

- 📝 **Cardholder Name** (`FormFieldTypes.NAME`)
- 🏠 **Billing Address** fields (address, city, state, zip)
- 📧 **Email Address**
- 📞 **Phone Number**
- 🏷️ **Custom Metadata** (order ID, customer ID, etc.)

**Flexibility for Non-Sensitive Data:**

```typescript
// Option 1: Use SPLTextField (recommended for consistency)
<SPLTextField formFieldType={FormFieldTypes.NAME} label="Cardholder Name" />

// Option 2: Use custom TextInput (allowed for non-sensitive fields)
<TextInput
  placeholder="Cardholder Name"
  value={cardholderName}
  onChangeText={setCardholderName}
  // Your custom styling and validation
/>
```

**Example of Proper Implementation:**

```typescript
// Sensitive fields - MUST use SPLTextField
<SPLTextField formFieldType={FormFieldTypes.CARD} label="Card Number" />
<SPLTextField formFieldType={FormFieldTypes.CVV} label="CVV" />
<SPLTextField formFieldType={FormFieldTypes.EXPIRY_DATE} label="MM/YY" />

// Non-sensitive fields - Can use SPLTextField or custom implementation
<SPLTextField formFieldType={FormFieldTypes.NAME} label="Cardholder Name" />
// OR
<TextInput
  placeholder="Cardholder Name"
  value={cardholderName}
  onChangeText={setCardholderName}
/>
```

### Basic Card Form

```typescript
import React, { useState } from 'react';
import { View, Button, StyleSheet, Alert } from 'react-native';
import {
  SPLTextField,
  FormFieldTypes,
  ValidationManager,
  SpreedlyCore,
  AdditionalFields,
  mapPaymentResult,
  type FieldDescriptor,
  type CreateCreditCardResult
} from '@spreedly/react-native-checkout';

/**
 * ValidationManager is a utility class that helps manage form validation state.
 * It tracks which fields are valid/invalid and determines overall form validity.
 *
 * Key methods:
 * - isFormValid(fields, validationState): Returns true if all required fields are valid
 * - createValidationChangeHandler(): Creates handlers for field validation callbacks (deprecated)
 *
 * Modern approach: Handle validation directly in onValidationChange callbacks
 */

export function CustomCardForm() {
  const [fieldValidation, setFieldValidation] = useState<Record<string, boolean>>({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  const fields: FieldDescriptor[] = [
    // Core payment fields
    { type: FormFieldTypes.CARD, required: true },
    { type: FormFieldTypes.EXPIRY_DATE, required: true },
    { type: FormFieldTypes.CVV, required: true },
    { type: FormFieldTypes.NAME, required: true },

    // More core fields can be added here.
  ];

  // ValidationManager helps track form validity across multiple fields
  const isFormValid = ValidationManager.isFormValid(fields, fieldValidation);

  const handleSubmit = async () => {
    if (!isFormValid || isSubmitting) return;

    setIsSubmitting(true);

    try {
      const result: CreateCreditCardResult = await SpreedlyCore.createCreditCard({
        fields: fields,
        formFieldTypes: fields.map((f) => f.type),
        metadata: {
          orderId: '12345'
        }
      });

      // Use the SDK's result mapping for consistent handling
      const mapped = mapPaymentResult(result as PaymentResultRN);

      switch (mapped.kind) {
        case 'success':
          console.log('Payment token:', mapped.token);
          Alert.alert('Success', 'Payment completed successfully!');
          // Send token to your backend for processing
          break;

        case 'validation':
          console.log('Validation errors:', mapped.invalidFields);
          Alert.alert('Validation Error', mapped.message);
          // Highlight invalid fields in your UI
          break;

        case 'failed':
          console.error('Payment failed:', mapped.errorType, mapped.message);
          Alert.alert('Payment Failed', mapped.message);
          // Handle different error types appropriately
          break;

        case 'canceled':
          console.log('Payment was canceled');
          Alert.alert('Canceled', 'Payment was canceled');
          break;

        default:
          console.log('Unknown result:', result);
          Alert.alert('Error', 'An unexpected error occurred');
      }
    } catch (error) {
      console.error('Payment error:', error);
      Alert.alert('Error', 'Failed to process payment. Please try again.');
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <View style={styles.container}>
      <SPLTextField
        formFieldType={FormFieldTypes.CARD}
        label="Card Number"
        style={styles.field}
        onValidationChange={(isValid) => {
          setFieldValidation(prev => ({ ...prev, [FormFieldTypes.CARD]: isValid }));
        }}
      />

      <View style={styles.row}>
        <SPLTextField
          formFieldType={FormFieldTypes.EXPIRY_DATE}
          label="MM/YY"
          style={[styles.field, styles.halfField]}
          onValidationChange={(isValid) => {
            setFieldValidation(prev => ({ ...prev, [FormFieldTypes.EXPIRY_DATE]: isValid }));
          }}
        />

        <SPLTextField
          formFieldType={FormFieldTypes.CVV}
          label="CVV"
          style={[styles.field, styles.halfField]}
          onValidationChange={(isValid) => {
            setFieldValidation(prev => ({ ...prev, [FormFieldTypes.CVV]: isValid }));
          }}
        />
      </View>

      <SPLTextField
        formFieldType={FormFieldTypes.NAME}
        label="Cardholder Name"
        style={styles.field}
        onValidationChange={(isValid) => {
          setFieldValidation(prev => ({ ...prev, [FormFieldTypes.NAME]: isValid }));
        }}
      />

      <Button
        title={isSubmitting ? "Processing..." : "Submit Payment"}
        onPress={handleSubmit}
        disabled={!isFormValid || isSubmitting}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    padding: 20,
  },
  field: {
    height: 50,
    marginBottom: 15,
    borderWidth: 1,
    borderColor: '#E0E0E0',
    borderRadius: 8,
    paddingHorizontal: 15,
  },
  row: {
    flexDirection: 'row',
    justifyContent: 'space-between',
  },
  halfField: {
    width: '48%',
  },
});
```

### With Additional Fields

```typescript
import React, { useState } from 'react';
import { View, Button, StyleSheet, TextInput } from 'react-native';
import {
  SPLTextField,
  FormFieldTypes,
  ValidationManager,
  SpreedlyCore,
  AdditionalFields,
  mapPaymentResult,
  type FieldDescriptor,
  type CreateCreditCardResult
} from '@spreedly/react-native-checkout';

export function ExtendedCardForm() {
  const [fieldValidation, setFieldValidation] = useState<Record<string, boolean>>({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  // State for additional fields data
  const [additionalData, setAdditionalData] = useState({
    name: '',
    email: '',
    phoneNumber: '',
    addressLine1: '',
    city: ''
  });

  const fields: FieldDescriptor[] = [
    // Core payment fields
    { type: FormFieldTypes.CARD, required: true },
    { type: FormFieldTypes.EXPIRY_DATE, required: true },
    { type: FormFieldTypes.CVV, required: true },
  ];

  const isFormValid = ValidationManager.isFormValid(fields, fieldValidation);

  const handleSubmit = async () => {
    if (!isFormValid || isSubmitting) return;

    setIsSubmitting(true);

    try {
      const result: CreateCreditCardResult = await SpreedlyCore.createCreditCard({
        fields: fields,
        formFieldTypes: fields.map((f) => f.type),
        metadata: {
          orderId: '12345'
        },
        additionalFields: {
          [AdditionalFields.NAME]: additionalData.name,
          [AdditionalFields.EMAIL]: additionalData.email,
          [AdditionalFields.PHONE_NUMBER]: additionalData.phoneNumber,
          [AdditionalFields.ADDRESS_LINE_1]: additionalData.addressLine1,
          [AdditionalFields.CITY]: additionalData.city,
        }
      });

      // Use the SDK's result mapping for consistent handling
      const mapped = mapPaymentResult(result as PaymentResultRN);

      switch (mapped.kind) {
        case 'success':
          console.log('Payment token:', mapped.token);
          // Send token to your backend for processing
          break;

        case 'validation':
          console.log('Validation errors:', mapped.invalidFields);
          // Highlight invalid fields in your UI
          break;

        case 'failed':
          console.error('Payment failed:', mapped.errorType, mapped.message);
          // Handle different error types appropriately
          break;

        case 'canceled':
          console.log('Payment was canceled');
          break;

        default:
          console.log('Unknown result:', result);
      }
    } catch (error) {
      console.error('Payment error:', error);
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <View>
      {/* Core payment fields - MUST use SPLTextField for sensitive data */}
      <SPLTextField formFieldType={FormFieldTypes.CARD} label="Card Number" />
      <SPLTextField formFieldType={FormFieldTypes.EXPIRY_DATE} label="MM/YY" />
      <SPLTextField formFieldType={FormFieldTypes.CVV} label="CVV" />

      {/* Additional fields - Can use regular TextInput for non-sensitive data */}
      <TextInput
        style={styles.input}
        placeholder="Cardholder Name"
        value={additionalData.name}
        onChangeText={(text) => setAdditionalData(prev => ({ ...prev, name: text }))}
      />

      <TextInput
        style={styles.input}
        placeholder="Email Address"
        value={additionalData.email}
        onChangeText={(text) => setAdditionalData(prev => ({ ...prev, email: text }))}
        keyboardType="email-address"
        autoCapitalize="none"
      />

      <TextInput
        style={styles.input}
        placeholder="Phone Number"
        value={additionalData.phoneNumber}
        onChangeText={(text) => setAdditionalData(prev => ({ ...prev, phoneNumber: text }))}
        keyboardType="phone-pad"
      />

      <TextInput
        style={styles.input}
        placeholder="Address Line 1"
        value={additionalData.addressLine1}
        onChangeText={(text) => setAdditionalData(prev => ({ ...prev, addressLine1: text }))}
      />

      <TextInput
        style={styles.input}
        placeholder="City"
        value={additionalData.city}
        onChangeText={(text) => setAdditionalData(prev => ({ ...prev, city: text }))}
      />

      <Button
        title={isSubmitting ? "Processing..." : "Submit Payment"}
        onPress={handleSubmit}
        disabled={!isFormValid || isSubmitting}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  input: {
    height: 50,
    marginBottom: 15,
    borderWidth: 1,
    borderColor: '#E0E0E0',
    borderRadius: 8,
    paddingHorizontal: 15,
    fontSize: 16,
  },
});
```

---

## Advanced Configuration

### Global SDK Configuration

```typescript
import { SpreedlyCore } from '@spreedly/react-native-checkout';

// Set global validation parameters
SpreedlyCore.setParam('ALLOW_BLANK_NAME', false);
SpreedlyCore.setParam('ALLOW_EXPIRED_DATE', false);

// Set global theme (see Theme_Guide.md for complete documentation)
SpreedlyCore.setGlobalTheme({
  theme: {
    primaryColor: '#0077C8',
    secondaryColor: '#AFB4B5',
    formBorderColor: '#D1D5DB',
    formBackgroundColor: '#FFFFFF',
    fieldBackgroundColor: '#FFFFFF',
    fieldLabelColor: '#6B7280',
    borderRadius: 8,
    fieldShape: 'rounded',
  },
  darkTheme: {
    primaryColor: '#60A5FA',
    secondaryColor: '#9CA3AF',
    formBorderColor: '#374151',
    formBackgroundColor: '#1F2937',
    fieldBackgroundColor: '#111827',
    fieldLabelColor: '#9CA3AF',
    borderRadius: 8,
    fieldShape: 'rounded',
  },
});
```

**📖 For complete theming documentation**: See [Theme_Guide.md](./Theme_Guide.md)

#### Understanding Validation Parameters

The `setParam` method allows you to configure global validation behavior across all SDK components. These settings affect both Express Checkout and Hosted Fields implementations.

**`ALLOW_BLANK_NAME` Parameter**

Controls whether the cardholder name field can be left empty during payment processing.

```typescript
// Strict validation - name is required (recommended for most use cases)
SpreedlyCore.setParam('ALLOW_BLANK_NAME', false);

// Lenient validation - name can be empty
SpreedlyCore.setParam('ALLOW_BLANK_NAME', true);
```

**When to use `ALLOW_BLANK_NAME: false` (Default - Recommended):**

- **Fraud Prevention**: Cardholder names help verify the legitimacy of transactions
- **Compliance Requirements**: Many payment processors and regulations require cardholder names
- **Chargeback Protection**: Having complete cardholder information strengthens dispute resolution
- **Business Requirements**: When your business model requires complete customer information

**When to use `ALLOW_BLANK_NAME: true` (Use with caution):**

- **Guest Checkout**: For simplified anonymous transactions
- **Specific Processor Requirements**: Some processors don't require names for certain transaction types
- **Legacy System Compatibility**: When integrating with systems that don't collect names

⚠️ **Important Considerations:**

- Even when allowing blank names, the field validation will still check format if a name is provided
- Some payment processors may reject transactions without cardholder names
- Consider your fraud prevention strategy when allowing blank names

**`ALLOW_EXPIRED_DATE` Parameter**

Controls whether expired credit cards are accepted during validation.

```typescript
// Strict validation - expired cards are rejected (recommended for live transactions)
SpreedlyCore.setParam('ALLOW_EXPIRED_DATE', false);

// Lenient validation - expired cards are accepted
SpreedlyCore.setParam('ALLOW_EXPIRED_DATE', true);
```

**When to use `ALLOW_EXPIRED_DATE: false` (Default - Recommended):**

- **Live Payment Processing**: Expired cards will be declined by processors anyway
- **User Experience**: Prevents users from submitting forms that will fail
- **Fraud Prevention**: Expired cards are often associated with fraudulent activity
- **Data Quality**: Ensures only valid, current payment methods are collected

**When to use `ALLOW_EXPIRED_DATE: true` (Special cases only):**

- **Testing Environments**: When using test cards that may be expired
- **Card Storage**: When storing cards for future use (though not recommended)
- **Legacy Data Migration**: When importing existing card data that may include expired cards
- **Specific Business Logic**: When your application handles expiration validation separately

⚠️ **Security Implications:**

- Allowing expired dates in production can lead to failed transactions
- Users may become frustrated if they can submit expired cards that later fail
- Consider implementing client-side warnings even when allowing expired dates

**Example: Environment-Specific Configuration**

```typescript
// Configure validation based on environment
const isProduction = process.env.NODE_ENV === 'production';
const isTestEnvironment =
  process.env.SPREEDLY_ENVIRONMENT_KEY?.includes('test');

// Strict validation for production
if (isProduction && !isTestEnvironment) {
  SpreedlyCore.setParam('ALLOW_BLANK_NAME', false);
  SpreedlyCore.setParam('ALLOW_EXPIRED_DATE', false);
} else {
  // More lenient for development/testing
  SpreedlyCore.setParam('ALLOW_BLANK_NAME', true);
  SpreedlyCore.setParam('ALLOW_EXPIRED_DATE', true);
}
```

**Example: Business Logic-Based Configuration**

```typescript
// Configure based on checkout type
const configureForCheckoutType = (
  checkoutType: 'guest' | 'registered' | 'subscription'
) => {
  switch (checkoutType) {
    case 'guest':
      // Allow blank names for guest checkout
      SpreedlyCore.setParam('ALLOW_BLANK_NAME', true);
      SpreedlyCore.setParam('ALLOW_EXPIRED_DATE', false);
      break;

    case 'registered':
      // Require complete information for registered users
      SpreedlyCore.setParam('ALLOW_BLANK_NAME', false);
      SpreedlyCore.setParam('ALLOW_EXPIRED_DATE', false);
      break;

    case 'subscription':
      // Strict validation for recurring payments
      SpreedlyCore.setParam('ALLOW_BLANK_NAME', false);
      SpreedlyCore.setParam('ALLOW_EXPIRED_DATE', false);
      break;
  }
};
```

### Dynamic Field Configuration

Dynamic field configuration allows you to adapt your payment forms based on business logic, user preferences, or contextual requirements. This approach provides flexibility while maintaining security and validation.

**Real-World Use Cases:**

- **E-commerce**: Different fields for digital vs. physical products
- **Subscription Services**: Additional fields for recurring payments
- **B2B Payments**: Corporate billing information requirements
- **International Sales**: Country-specific field requirements
- **Guest vs. Registered Users**: Different information collection strategies

#### Basic Dynamic Configuration Example

```typescript
export function DynamicForm() {
  const [includeShipping, setIncludeShipping] = useState(false);

  const getFields = (): FieldDescriptor[] => {
    const baseFields = [
      { type: FormFieldTypes.CARD, required: true },
      { type: FormFieldTypes.EXPIRY_DATE, required: true },
      { type: FormFieldTypes.CVV, required: true },
    ];

    if (includeShipping) {
      return [
        ...baseFields,
        { type: AdditionalFields.SHIPPING_ADDRESS_1, required: true },
        { type: AdditionalFields.SHIPPING_CITY, required: true },
        { type: AdditionalFields.SHIPPING_STATE, required: true },
        { type: AdditionalFields.SHIPPING_ZIP, required: true },
      ];
    }

    return baseFields;
  };

  return (
    <View>
      <Switch
        value={includeShipping}
        onValueChange={setIncludeShipping}
      />
      <Text>Include Shipping Address</Text>

      {getFields().map((field) => (
        <SPLTextField
          key={field.type}
          formFieldType={field.type}
          label={field.type.replace('_', ' ').toLowerCase()}
        />
      ))}
    </View>
  );
}
```

📖 **For More Advanced Examples**: See [Examples.md](./Examples.md) for comprehensive implementation patterns including:

- E-commerce checkout flows with product-specific fields
- Subscription service configurations with trial periods
- Progressive field disclosure patterns
- Multi-region configurations with localization
- Performance optimization techniques
- Advanced error handling patterns

---

## Payment Result Handling

### Understanding Payment Results

Payment result handling is a critical part of the integration that determines how your app responds to different payment outcomes. The SDK provides structured result objects that help you implement robust payment flows.

**Payment Flow Overview:**

```
User Submits Payment → SDK Processes → Result Generated → Your App Handles Result
                                           ↓
                                   ┌─────────────────┐
                                   │ PaymentResultRN │
                                   └─────────────────┘
                                           ↓
                                   ┌─────────────────┐
                                   │ MappedOutcome   │ ← Simplified handling
                                   └─────────────────┘
```

**Why Result Mapping Matters:**

- **Consistency**: Provides uniform handling across Express Checkout and Hosted Fields
- **Error Categorization**: Groups similar errors for appropriate user messaging
- **Business Logic**: Enables different handling based on failure type (retry vs. abort)
- **User Experience**: Helps provide meaningful feedback to users

The SDK returns `PaymentResultRN` objects that can be mapped to simplified outcomes:

```typescript
import {
  mapPaymentResult,
  type PaymentResultRN,
  type MappedOutcome,
} from '@spreedly/react-native-checkout';

export function handlePaymentResult(result: PaymentResultRN): void {
  const mapped: MappedOutcome = mapPaymentResult(result);

  switch (mapped.kind) {
    case 'success':
      // Payment completed successfully
      console.log('Payment token:', mapped.token);
      // Send token to your backend for processing
      break;

    case 'validation':
      // Validation errors occurred
      console.log('Invalid fields:', mapped.invalidFields);
      console.log('Error message:', mapped.message);
      // Highlight invalid fields in your UI
      break;

    case 'failed':
      // Payment processing failed
      console.log('Error type:', mapped.errorType);
      console.log('Error message:', mapped.message);
      // Show appropriate error message to user
      break;

    case 'canceled':
      // User canceled the payment
      console.log('Payment was canceled by user');
      break;

    case 'initial':
      // Initial state, no action taken yet
      break;
  }
}
```

### Custom Result Mapping

```typescript
import { type PaymentResultRN } from '@spreedly/react-native-checkout';

export function customPaymentHandler(result: PaymentResultRN) {
  switch (result.status) {
    case 'completed':
      if (result.token) {
        // Send to your payment processor
        processPayment(result.token);
      }
      break;

    case 'failed':
      const errorType = result.failureDetails?.errorType;
      const message = result.failureDetails?.message;

      switch (errorType) {
        case 'API_ERROR':
          showRetryableError(message);
          break;
        case 'NETWORK_ERROR':
          showNetworkError();
          break;
        case 'VALIDATION_ERROR':
          showValidationError(message);
          break;
        default:
          showGenericError();
      }
      break;

    case 'validation_failed':
      if (result.invalidFields) {
        highlightInvalidFields(result.invalidFields);
      }
      break;
  }
}
```

---

## Error Handling

### Comprehensive Error Handling

```typescript
import {
  type PaymentErrorType,
  type PaymentValidationError,
} from '@spreedly/react-native-checkout';

export class PaymentErrorHandler {
  static handleError(error: any): string {
    if (error?.failureDetails?.errorType) {
      return this.handleTypedError(
        error.failureDetails.errorType,
        error.failureDetails.message
      );
    }

    // Handle network errors
    if (error?.code === 'NETWORK_ERROR') {
      return 'Please check your internet connection and try again.';
    }

    // Handle validation errors
    if (error?.invalidFields) {
      return `Please correct the following fields: ${error.invalidFields.join(', ')}`;
    }

    return 'An unexpected error occurred. Please try again.';
  }

  static handleTypedError(
    errorType: PaymentErrorType,
    message?: string
  ): string {
    switch (errorType) {
      case 'API_ERROR':
        return message || 'Payment processing failed. Please try again.';

      case 'NETWORK_ERROR':
        return 'Connection failed. Please check your internet and retry.';

      case 'UNKNOWN_ERROR':
        return 'An unexpected error occurred. Please contact support if this continues.';

      default:
        return message || 'Payment failed. Please try again.';
    }
  }
}
```

### Retry Logic

```typescript
export function usePaymentWithRetry() {
  const [retryCount, setRetryCount] = useState(0);
  const maxRetries = 3;

  const submitPayment = async (
    fields: FieldDescriptor[],
    options?: any
  ): Promise<PaymentResultRN> => {
    try {
      const result = await SpreedlyCore.createCreditCard({
        fields,
        ...options,
      });

      if (
        result.status === 'failed' &&
        result.failureDetails?.errorType === 'NETWORK_ERROR' &&
        retryCount < maxRetries
      ) {
        setRetryCount((prev) => prev + 1);
        // Wait before retry (exponential backoff)
        await new Promise((resolve) =>
          setTimeout(resolve, Math.pow(2, retryCount) * 1000)
        );
        return submitPayment(fields, options);
      }

      setRetryCount(0); // Reset on success or non-retryable error
      return result;
    } catch (error) {
      if (retryCount < maxRetries) {
        setRetryCount((prev) => prev + 1);
        await new Promise((resolve) =>
          setTimeout(resolve, Math.pow(2, retryCount) * 1000)
        );
        return submitPayment(fields, options);
      }
      throw error;
    }
  };

  return { submitPayment, retryCount, canRetry: retryCount < maxRetries };
}
```

---

## Customization

### Theme Configuration

The Spreedly SDK supports comprehensive theming with automatic dark mode support. For complete theming documentation, see the [Theme Customization Guide](./Theme_Guide.md).

**Quick Theme Example:**

```typescript
import { SpreedlyCore, SPLTextField, FormFieldTypes } from '@spreedly/react-native-checkout';

// Set global theme with dark mode support
SpreedlyCore.setGlobalTheme({
  theme: {
    primaryColor: '#6366F1',
    secondaryColor: '#8B5CF6',
    formBorderColor: '#D1D5DB',
    formBackgroundColor: '#FFFFFF',
    fieldBackgroundColor: '#F9FAFB',
    fieldLabelColor: '#6B7280',
    borderRadius: 12,
    fieldShape: 'rounded',
  },
  darkTheme: {
    primaryColor: '#818CF8',
    secondaryColor: '#A78BFA',
    formBorderColor: '#374151',
    formBackgroundColor: '#1F2937',
    fieldBackgroundColor: '#111827',
    fieldLabelColor: '#9CA3AF',
    borderRadius: 12,
    fieldShape: 'rounded',
  },
});

// Or apply theme to individual components
<SPLTextField
  formFieldType={FormFieldTypes.CARD}
  label="Card Number"
  theme={lightTheme}
  darkTheme={darkTheme}
/>
```

**📖 Complete Theming Documentation**: See [Theme_Guide.md](./Theme_Guide.md) for:

- Global theme configuration
- Component-level theming
- Dark mode implementation
- Pre-built theme examples
- Platform-specific behavior
- Accessibility guidelines

### Custom Field Styling

For styling the wrapper containers around SDK components:

```typescript
import { StyleSheet } from 'react-native';
import { SPLTextField, FormFieldTypes } from '@spreedly/react-native-checkout';

export const fieldStyles = StyleSheet.create({
  container: {
    marginBottom: 16,
  },
  fieldWrapper: {
    borderWidth: 2,
    borderColor: '#007AFF',
    borderRadius: 12,
    padding: 4,
  },
});

export function StyledPaymentField() {
  return (
    <View style={fieldStyles.container}>
      <View style={fieldStyles.fieldWrapper}>
        <SPLTextField
          formFieldType={FormFieldTypes.CARD}
          label="Card Number"
        />
      </View>
    </View>
  );
}
```

**Note**: The `style` prop on `SPLTextField` applies to the native wrapper view, not the internal field styling. Use `theme` and `darkTheme` props to customize the field appearance itself.

---

## API Reference

### Core Methods

#### `SpreedlyCore.initSdk(options: SpreedlySDKInitOptions): void`

Initialize the SDK with authentication parameters.

**Parameters:**

- `options.token: string` - Certificate token
- `options.nonce: string` - Unique nonce for request
- `options.signature: string` - Request signature
- `options.certificateToken: string` - Certificate token (duplicate for compatibility)
- `options.timestamp: string` - Request timestamp
- `options.environmentKey: string` - Spreedly environment key
- `options.forterSiteId: string` - Forter site ID for fraud prevention integration (pass empty string `''` if not using Forter)

#### `SpreedlyCore.createCreditCard(options: CreateCreditCardOptions): Promise<CreateCreditCardResult>`

Create a payment method using hosted fields.

**Parameters:**

- `options.fields: FieldDescriptor[]` - Array of field configurations
- `options.metadata?: object` - Additional metadata
- `options.additionalFields?: object` - Additional field values

**Returns:** Promise resolving to payment result

#### `SpreedlyCore.paymentBottomSheet(options?: PaymentBottomSheetOptions): void`

Show the express checkout bottom sheet.

**Parameters:**

- `options.allowBlankName?: boolean` - Allow empty name field
- `options.allowExpiredDate?: boolean` - Allow expired dates
- `options.yearFormat?: '2' | '4'` - Year format display
- `options.nameDisplayMode?: 'singleField' | 'separateFields'` - Name input mode
- `options.theme?: BaseThemeConfig` - Custom theme for light mode
- `options.darkTheme?: BaseThemeConfig` - Custom theme for dark mode

**For theming documentation**, see [Theme_Guide.md](./Theme_Guide.md)

**🚧 SDK Enhancement Recommendations:**

1. **Type Safety for `nameDisplayMode`**: Currently accepts string literals, but should use SDK-defined constants:

   ```typescript
   // Current (prone to typos)
   nameDisplayMode: 'singleField'; // Could be mistyped as 'single-field'

   // Recommended SDK improvement
   nameDisplayMode: NameDisplayMode.SINGLE_FIELD; // Type-safe constant
   ```

2. **Consistency with `yearFormat`**: The `yearFormat` parameter uses string literals but should also use constants:

   ```typescript
   // Current
   yearFormat: '4';

   // Recommended
   yearFormat: YearFormat.FOUR_DIGIT;
   ```

3. **Runtime Validation**: The SDK should validate these parameters and provide clear error messages for invalid values rather than failing silently.

#### `SpreedlyCore.setGlobalTheme(options: GlobalThemeOptions | BaseThemeConfig): void`

Set global theme for all SDK components with optional dark mode support.

**For complete theming documentation**, see [Theme_Guide.md](./Theme_Guide.md)

#### `SpreedlyCore.setParam(parameter: ValidationParameter, value: boolean): void`

Set global validation parameters.

#### `SpreedlyCore.showThreeDSChallenge(managedOrderToken: string, transactionToken: string): void`

Display the 3DS challenge UI to authenticate a transaction.

**Parameters:**

- `managedOrderToken: string` - Managed order token from your backend's Spreedly API response
- `transactionToken: string` - Transaction token from your backend's purchase/authorize response

**Usage:** See [3DS Challenge Integration](#3ds-challenge-integration) for complete implementation details.

#### `SpreedlyCore.hideThreeDSChallenge(): void`

Programmatically dismiss the 3DS challenge UI. Use sparingly for edge cases like session timeouts.

### Components

#### `<SPLTextField>`

Secure hosted field component for payment data entry.

**Props:**

- `formFieldType: string` - **Required**. Field type from FormFieldTypes or AdditionalFields
- `label: string` - **Required**. Field label or placeholder text shown within the field
- `errorMessage?: string` - Optional error message to display beneath the field
- `theme?: CustomThemeConfig` - Visual theme for light mode (colors, shapes) applied to the field
- `darkTheme?: CustomThemeConfig` - Visual theme for dark mode. If not provided, uses `theme` for both modes
- `isRequired?: boolean` - Whether this field is required for validation. Defaults to `true`
- `yearFormat?: string` - Expiry year format: `'2'` for YY, `'4'` for YYYY. Affects expiry fields only
- `style?: StyleProp<ViewStyle>` - Style applied to the native wrapper view
- `onValidationChange?: (isValid: boolean) => void` - Called when the native field validation state changes
- `onContentSizeChange?: (size: { width: number; height: number }) => void` - Called when the native content size changes

**For theming documentation**, see [Theme_Guide.md](./Theme_Guide.md)

**Example Usage:**

```typescript
<SPLTextField
  formFieldType={FormFieldTypes.CARD}
  label="Card Number"
  errorMessage={cardError}
  theme={lightTheme}
  darkTheme={darkTheme}
  isRequired={true}
  style={{ marginBottom: 16 }}
  onValidationChange={(isValid) => setCardValid(isValid)}
  onContentSizeChange={(size) => console.log('Field size:', size)}
/>
```

**Important Notes:**

- Use `formFieldType` (not `fieldType`) for the field type
- Use `label` (not `placeholder`) for the field text
- The component automatically handles minimum height and responsive sizing
- Validation callbacks provide boolean results, not detailed validation objects

### Type Definitions

#### `FormFieldTypes`

**Core Payment Fields:**

- `CARD` - Credit card number (🔒 Sensitive)
- `CVV` - Security code (🔒 Sensitive)
- `EXPIRY_DATE` - Expiration date (🔒 Sensitive)
- `MONTH` - Expiration month
- `YEAR` - Expiration year
- `YEAR_SECONDARY` - Secondary year field
- `NAME` - Cardholder name

**Address Fields:**

- `ADDRESS_LINE_1` - Address line 1
- `ADDRESS_LINE_2` - Address line 2
- `CITY` - City
- `STATE` - State/Province
- `ZIP` - Postal code

**⚠️ Important**: Few fields are available in both `FormFieldTypes` and `AdditionalFields`. Use them like this `FormFieldTypes.EXPIRY_DATE` for basic forms or `AdditionalFields.EXPIRY_DATE` for extended forms.

#### `AdditionalFields`

**Includes all FormFieldTypes plus:**

**Contact Information:**

- `COUNTRY` - Country
- `PHONE_NUMBER` - Phone number
- `EMAIL` - Email address

**Shipping Address:**

- `SHIPPING_ADDRESS_1` - Shipping address line 1
- `SHIPPING_ADDRESS_2` - Shipping address line 2
- `SHIPPING_CITY` - Shipping city
- `SHIPPING_STATE` - Shipping state
- `SHIPPING_ZIP` - Shipping postal code
- `SHIPPING_COUNTRY` - Shipping country
- `SHIPPING_PHONE_NUMBER` - Shipping phone

**📝 Note**: `AdditionalFields` contains all `FormFieldTypes` plus the additional fields listed above. Use `AdditionalFields` when you need extended field options.

#### Theme Configuration Types

For complete theme configuration documentation including dark mode support, see [Theme_Guide.md](./Theme_Guide.md).

**Quick Reference:**

```typescript
interface BaseThemeConfig {
  primaryColor: string;
  secondaryColor: string;
  formBorderColor: string;
  formBackgroundColor: string;
  fieldBackgroundColor: string;
  fieldLabelColor: string;
  borderRadius: number;
  fieldShape: string;
}

interface GlobalThemeOptions {
  theme?: BaseThemeConfig;
  darkTheme?: BaseThemeConfig;
}
```

#### 3DS Challenge Types

```typescript
/**
 * Status of a 3DS challenge operation
 */
type ThreeDSChallengeStatus = 'success' | 'failed';

/**
 * Result of a 3DS challenge operation.
 * Emitted via the THREE_DS_CHALLENGE_RESULT event.
 */
type ThreeDSChallengeResult =
  | { status: 'success'; transactionId?: string }
  | { status: 'failed'; message?: string };
```

#### Event Types

```typescript
/**
 * Available Spreedly event types for event listeners
 */
const SpreedlyEventTypes = {
  PAYMENT_BOTTOM_SHEET_RESULT: 'onPaymentBottomSheetResult',
  RECACHE_RESULT: 'onRecacheResult',
  THREE_DS_CHALLENGE_RESULT: 'onThreeDSChallengeResult',
} as const;
```

---

## Best Practices

Following these best practices ensures secure, performant, and maintainable payment integrations that provide excellent user experiences while meeting compliance requirements.

### Security Best Practices

**Why Security Matters in Payment Processing:**

Payment applications are high-value targets for attackers. Following security best practices protects both your users' sensitive data and your business from fraud, compliance violations, and reputation damage. The Spreedly SDK is designed with security-first principles, but proper implementation is crucial.

**Security Principles:**

- **Defense in Depth**: Multiple layers of security controls
- **Least Privilege**: Minimal access rights for components and users
- **Data Minimization**: Collect and store only necessary information
- **Secure by Default**: Safe configurations out of the box

#### 1. API Key and Environment Key Security

**Critical: Never Hardcode Credentials**

```typescript
// ❌ NEVER DO THIS - Hardcoded credentials
const BAD_EXAMPLE = {
  environmentKey: 'test_abc123def456', // NEVER hardcode
  apiKey: 'sk_live_123456789', // NEVER hardcode
};

// ✅ CORRECT - Use environment variables
const CORRECT_EXAMPLE = {
  environmentKey: process.env.SPREEDLY_ENVIRONMENT_KEY,
  // Environment key should be in .env file, never committed
};
```

**Environment Variable Management:**

```bash
# Create .env file in project root
echo "SPREEDLY_ENVIRONMENT_KEY=your_key_here" > .env
echo ".env" >> .gitignore

# Load environment variables in your app
import Config from 'react-native-config';
const environmentKey = Config.SPREEDLY_ENVIRONMENT_KEY;
```

**Key Rotation Policy:**

- **Immediate Rotation Required If:**
  - Credentials are accidentally committed to version control
  - Credentials are exposed in logs or error messages
  - Team member with access leaves the organization
  - Suspicious activity detected in your Spreedly account

- **Regular Rotation Schedule:**
  - Rotate environment keys every 90 days (quarterly)
  - Rotate GitHub tokens every 90 days
  - Document rotation dates and maintain audit trail

#### 2. Mobile App Security

**Screenshot and Screen Recording Prevention**

Payment screens should implement additional security measures to prevent unauthorized capture of sensitive information. The Spreedly SDK provides built-in `ScreenSecurity` module to help protect payment flows from screenshots and screen recording.

**Security Concerns:**

- Screenshots can capture sensitive payment data
- Screen recording apps may operate without visible indicators
- App switcher thumbnails can expose payment information
- Screen sharing during video calls poses data exposure risks

#### Built-in Screenshot Prevention

The Spreedly SDK provides screenshot and screen recording protection on both iOS and Android platforms. Both platforms now require explicit activation of the `ScreenSecurity` module.

**Manual Activation Required (iOS & Android)**

You need to explicitly activate the `ScreenSecurity` module for screenshot protection on both platforms:

```typescript
import React, { useEffect } from 'react';
import { ScreenSecurity } from '@spreedly/react-native-checkout';

const App = () => {
  useEffect(() => {
    // Activate screenshot protection on both iOS and Android
    ScreenSecurity.activateProtection({
      backgroundColor: '#FFFFFF', // Color shown in iOS screenshots (Android shows black)
    }).catch(console.error);

    // Cleanup: deactivate protection when app unmounts
    return () => {
      ScreenSecurity.deactivateProtection().catch(console.error);
    };
  }, []);

  return <AppNavigator />;
};

export default App;
```

**Platform-Specific Behavior:**

**iOS Implementation:**

- ✅ Shows specified background color (e.g., white) in screenshots
- ✅ Provides event listeners for screenshot detection
- ✅ Provides event listeners for screen recording detection
- ✅ Can check if screen is being captured in real-time

**Android Implementation:**

- ✅ Uses `FLAG_SECURE` to completely prevent screenshots
- ✅ Prevents screen recording entirely
- ✅ Screenshots/recordings show black screen (system behavior)
- ⚠️ No detection events available (Android platform limitation)

**Platform Comparison:**

| Feature                    | iOS                                                            | Android                                                        |
| -------------------------- | -------------------------------------------------------------- | -------------------------------------------------------------- |
| Screenshot Prevention      | ✅ Manual activation via `ScreenSecurity.activateProtection()` | ✅ Manual activation via `ScreenSecurity.activateProtection()` |
| Screenshot Behavior        | Shows custom background color                                  | Shows black screen (system behavior)                           |
| Screen Recording Detection | ✅ Available via event listeners                               | ❌ Not available (Android limitation)                          |
| Configuration Required     | Yes - must call activate/deactivate                            | Yes - must call activate/deactivate                            |
| Implementation Method      | UITextField overlay technique                                  | FLAG_SECURE window flag                                        |

**Important Notes:**

- **Both Platforms**: You must explicitly activate and deactivate screenshot protection using the `ScreenSecurity` module.
- **Activation Timing**: Call `activateProtection()` after app initialization to avoid interfering with React Native bridge initialization.
- **App Store Compliance**: Screenshot protection is compliant with both App Store and Play Store guidelines when used for security purposes.
- **User Experience**: Consider informing users why screenshot protection is active to avoid confusion.
- **Background Color**: On iOS, choose a background color that matches your app's theme. On Android, the system always shows black.
- **Android Limitation**: Android's FLAG_SECURE provides prevention but no detection events. iOS provides both prevention and detection.

**Additional Security Measures:**

Beyond the built-in `ScreenSecurity` module, consider these additional protections:

- Implement session timeouts for payment screens
- Display security warnings before payment entry
- Consider biometric authentication for payment actions
- Clear sensitive data when app enters background

**Note**: The Spreedly SDK's `SPLTextField` components automatically implement security measures including:

- Preventing text selection and copying
- Disabling clipboard access for sensitive fields
- Secure text entry for payment data

**Further Reading:**

- React Native Security Best Practices
- Mobile Payment Security Guidelines (PCI Mobile Payment Acceptance Security Guidelines)
- Platform-specific security documentation for iOS and Android

#### 3. Token Storage Security

**CRITICAL: Never Store Payment Tokens Insecurely**

Payment tokens received from the Spreedly SDK should **NEVER** be stored in insecure local storage. They should be immediately sent to your backend for processing.

**❌ NEVER Store Tokens locally:**
**✅ CORRECT Approach: Immediate Server Transmission**

**Token Storage Security Checklist:**

- [ ] **Never use AsyncStorage** for payment tokens
- [ ] **Never use UserDefaults/SharedPreferences** for payment tokens
- [ ] **Send tokens to backend immediately** after receiving from SDK
- [ ] **Clear tokens from memory** after processing
- [ ] **Implement token expiration** if temporary storage is unavoidable
- [ ] **Use hardware-backed encryption** (Keychain/Keystore) only if absolutely necessary
- [ ] **Log security events** (token access, processing attempts)
- [ ] **Implement retry logic** for backend communication failures
- [ ] **Never log token values** in console or crash reports
- [ ] **Validate backend response** before considering payment complete

#### 4. Repository Access Security

**Protect GitHub Credentials:**

```bash
# ❌ DON'T commit credentials to version control
echo ".env" >> .gitignore
echo "Podfile.lock" >> .gitignore  # If using secure iOS setup

# ✅ DO use environment variables
export GITHUB_TOKEN=ghp_xxxxxxxxxxxx
```

**Use Secure iOS Setup:**

```bash
# ✅ Prevent credentials in Podfile.lock
./node_modules/@spreedly/react-native-checkout/scripts/setup_local_dev_ios.sh

# ❌ DON'T commit Podfile.lock with embedded credentials
```

**Rotate Tokens Regularly:**

```bash
# Generate new tokens quarterly
# Update tokens in all environments (local, CI/CD)
# Revoke old tokens after updating
```

#### 5. Validation Security

**Validate on Both Client and Server:**

```typescript
// Client-side validation using ValidationManager
const isValid = ValidationManager.isFormValid(fields, fieldValidation);

// Always validate on server after receiving token
fetch('/api/process-payment', {
  method: 'POST',
  body: JSON.stringify({ token: paymentToken }),
});
```

**Secure Authentication:**

```typescript
// Fetch fresh auth params for each session
const authParams = await fetchAuthParams(); // From your secure backend
```

---

### 🔐 Security Integration Checklist

Use this comprehensive checklist to ensure your integration follows all security best practices:

#### API Key & Credentials Security

- [ ] Environment keys stored in `.env` file (never hardcoded)
- [ ] `.env` file added to `.gitignore`
- [ ] Using environment variables or secure config management
- [ ] GitHub tokens have minimal required permissions
- [ ] Quarterly rotation schedule established for all credentials
- [ ] Process defined for emergency credential rotation
- [ ] No credentials in version control history
- [ ] No credentials in log files or error messages

#### Mobile App Security

- [ ] Screenshot prevention considered for payment screens
- [ ] Screen recording risks assessed and mitigated
- [ ] Security warnings shown to users before payment
- [ ] Payment screens timeout after inactivity
- [ ] Clipboard access disabled for sensitive fields (SPLTextField handles this automatically)
- [ ] Sensitive data cleared when app backgrounds

#### Token Storage Security

- [ ] Tokens sent to backend immediately after receipt
- [ ] No tokens stored in AsyncStorage
- [ ] No tokens stored in UserDefaults (iOS)
- [ ] No tokens stored in SharedPreferences (Android)
- [ ] No tokens in Redux persist or similar
- [ ] Token cleared from memory after processing
- [ ] Only transaction IDs and status stored locally
- [ ] Keychain/Keystore used ONLY if temporary storage absolutely required

#### Payment Field Security

- [ ] Card number field uses `SPLTextField` (never custom TextInput)
- [ ] CVV field uses `SPLTextField` (never custom TextInput)
- [ ] Expiry date uses `SPLTextField` (never custom TextInput)
- [ ] No sensitive fields sent as additional fields
- [ ] PCI-compliant field handling verified

#### Validation & Error Handling

- [ ] Client-side validation implemented
- [ ] Server-side validation implemented
- [ ] Fresh auth params fetched for each session
- [ ] Error messages don't expose sensitive data
- [ ] Failed attempts logged (without sensitive data)
- [ ] Rate limiting considered for payment submissions

#### Development & Deployment

- [ ] Separate test and production environment keys
- [ ] CI/CD pipelines use secure credential management
- [ ] Production builds use release signing
- [ ] Security audit performed before production release
- [ ] Team trained on security best practices
- [ ] Security incident response plan documented

#### Monitoring & Compliance

- [ ] Payment transaction logging implemented (without card data)
- [ ] Security event monitoring in place
- [ ] Regular security audits scheduled
- [ ] PCI DSS compliance requirements reviewed
- [ ] Privacy policy updated for payment processing
- [ ] User data handling complies with regulations (GDPR, CCPA, etc.)

**Security Review Schedule:**

- **Weekly**: Review payment transaction logs for anomalies
- **Monthly**: Review security event logs and failed authentication attempts
- **Quarterly**: Rotate credentials (API keys, GitHub tokens)
- **Annually**: Complete security audit and penetration testing
- **After Changes**: Security review for any payment flow modifications

**🚨 Security Incident Response:**

If you discover a security issue:

1. **Immediate Actions:**
   - Rotate all compromised credentials immediately
   - Disable affected payment methods temporarily
   - Notify your security team and Spreedly support
   - Document the incident with timestamps

2. **Investigation:**
   - Determine scope of potential data exposure
   - Review access logs and transaction history
   - Identify root cause and attack vector

3. **Resolution:**
   - Implement fixes for identified vulnerabilities
   - Test fixes in staging environment
   - Deploy to production with monitoring
   - Update security documentation

4. **Post-Incident:**
   - Conduct post-mortem review
   - Update security procedures
   - Train team on lessons learned
   - Notify affected users if required by regulations

---

### Performance Best Practices

**Why Performance Matters in Payment Flows:**

Payment forms are critical conversion points where even small delays can lead to cart abandonment. Users expect instant feedback and smooth interactions. Poor performance in payment flows directly impacts revenue and user satisfaction.

**📈 Performance Impact:**

Slow payment forms = lost revenue. Key metrics to monitor:

- **Time to Interactive**: How quickly users can start entering payment data
- **Validation Response**: Delay between input and validation feedback
- **Form Submission**: Duration from submit to result
- **Memory Usage**: RAM consumption during payment flows

1. **Initialize SDK early**:

```typescript
// Initialize in App component or root
useEffect(() => {
  SpreedlyCore.initSdk(options);
}, []); // Only once
```

2. **Debounce validation**:

```typescript
import { useDebouncedCallback } from 'use-debounce';

const debouncedValidation = useDebouncedCallback(
  (isValid: boolean) => {
    setFieldValidation((prev) => ({ ...prev, [fieldType]: isValid }));
  },
  300 // 300ms delay
);
```

3. **Minimize re-renders**:

```typescript
const MemoizedTextField = React.memo(SPLTextField);
```

### UX Best Practices

**Why UX Matters in Payment Forms:**

Payment forms are often the final step in a user's journey. Poor UX at this critical moment can cause users to abandon their purchase, even after investing time in product selection. Great payment UX builds trust and confidence.

**UX Principles for Payment Forms:**

- **Clarity**: Users should always understand what's happening and what's expected
- **Feedback**: Immediate response to user actions builds confidence
- **Error Prevention**: Guide users toward success rather than just catching errors
- **Accessibility**: Ensure all users can complete payments regardless of abilities
- **Trust**: Visual and interaction design should reinforce security and reliability

**Common UX Pitfalls to Avoid:**

- ❌ Unclear error messages ("Invalid input")
- ❌ No loading states during processing
- ❌ Sudden layout shifts during validation
- ❌ Inaccessible form controls
- ❌ Overwhelming users with too many fields at once

1. **Provide clear feedback**:

```typescript
const [validationState, setValidationState] = useState<{
  [key: string]: 'valid' | 'invalid' | 'pending'
}>({});

<SPLTextField
  formFieldType={FormFieldTypes.CARD}
  label="Card Number"
  onValidationChange={(isValid) => {
    setValidationState(prev => ({
      ...prev,
      [FormFieldTypes.CARD]: isValid ? 'valid' : 'invalid'
    }));
  }}
/>
```

2. **Progressive disclosure**:

```typescript
// Show fields progressively as previous fields are completed
const shouldShowCVV = fieldValidation[FormFieldTypes.CARD] === true;
const shouldShowName =
  shouldShowCVV && fieldValidation[FormFieldTypes.CVV] === true;
```

3. **Accessibility support**:

```typescript
<SPLTextField
  formFieldType={FormFieldTypes.CARD}
  label="Card Number"
  accessibilityLabel="Credit card number"
  accessibilityHint="Enter your 16-digit card number"
  testID="card-number-field"
/>
```

---

## Troubleshooting

This section covers the most frequently encountered issues during Spreedly SDK integration, along with their root causes and step-by-step solutions. Understanding these common problems can save significant development time and help you implement robust error handling.

**How to Use This Section:**

1. **Identify Symptoms**: Match error messages or behaviors with the problem descriptions
2. **Understand Root Causes**: Learn why these issues occur to prevent recurrence
3. **Apply Solutions**: Follow step-by-step instructions for your specific platform
4. **Verify Fixes**: Use the provided verification commands to confirm resolution
5. **Implement Prevention**: Apply best practices to avoid similar issues

**When to Seek Additional Help:**

- If solutions don't resolve your specific issue
- When encountering errors not covered in this guide
- For environment-specific problems (CI/CD, corporate networks, etc.)
- When integrating with custom build systems or monorepos

**Debugging Strategy:**

1. **Start with Environment**: Verify all prerequisites are met
2. **Check Credentials**: Ensure GitHub access and tokens are valid
3. **Isolate the Problem**: Test with minimal reproduction cases
4. **Review Logs**: Examine build logs for specific error details
5. **Test Incrementally**: Add complexity gradually after basic setup works

### Common Issues

**Issue Categories:**

- 🔐 **Authentication & Access**: GitHub credentials and repository access
- 📦 **Package Resolution**: Dependency installation and version conflicts
- 🛠️ **Build Configuration**: Platform-specific build setup issues
- 🚀 **Runtime Errors**: SDK initialization and usage problems
- 🔧 **Development Environment**: Local setup and tooling issues

#### 1. **Private Repository Access Issues** 🔐

**Problem**: Build fails with "Could not resolve dependency" or "Authentication failed"

**Root Cause Analysis:**

This is the most common issue when integrating the Spreedly SDK. It occurs because:

- **Missing Repository Access**: Your GitHub account doesn't have access to Spreedly's private repositories
- **Invalid Credentials**: GitHub token is expired, has insufficient permissions, or is incorrectly configured
- **Network Issues**: Corporate firewalls or proxy settings blocking GitHub Packages access
- **Configuration Errors**: Incorrect package manager setup for private repositories

**Impact**: Without proper access, your build process cannot download the SDK dependencies, preventing compilation and deployment.

**When This Occurs:**

- During `npm install` or `yarn install`
- During Android Gradle sync (`./gradlew build`)
- During iOS pod installation (`pod install`)
- In CI/CD pipelines when credentials aren't properly configured

**Android Solutions**:

```bash
# Verify GitHub token has correct permissions
# Token needs: read:packages, repo, read:org permissions

# Check environment variables
echo $GITHUB_USERNAME
echo $GITHUB_TOKEN

# Verify .env file (if using)
cat .env | grep GITHUB

# Test Gradle access
cd android
./gradlew dependencies --info | grep github
```

**iOS Solutions**:

```bash
# Clear CocoaPods cache
rm -rf ~/Library/Caches/CocoaPods
rm -rf Pods/ Podfile.lock

# Use secure setup script
./node_modules/@spreedly/react-native-checkout/scripts/setup_local_dev_ios.sh

# Verify spreedly_pods_setup.rb is loaded
cd ios
pod install --verbose

# Check if Spreedly pods are accessible
pod search SpreedlyCore --silent
```

#### 2. **Pod Setup Script Not Found**

**Problem**: `cannot load such file -- ./scripts/spreedly_pods_setup.rb`

**Solution**: Update your `ios/Podfile` with the correct require statement:

```ruby
# Correct approach (recommended)
require Pod::Executable.execute_command('node', ['-p',
  'require.resolve(
    "@spreedly/react-native-checkout/scripts/spreedly_pods_setup.rb",
    {paths: [process.argv[1]]},
  )', __dir__]).strip
```

**Alternative approaches** (if the above doesn't work):

```ruby
# Fallback method using path construction
require Pod::Executable.execute_command('node', ['-p',
  'require("path").join(
    require("path").dirname(require.resolve("@spreedly/react-native-checkout/package.json")),
    "scripts/spreedly_pods_setup.rb"
  )', __dir__]).strip
```

**Troubleshooting steps**:

```bash
# Verify the script exists in node_modules
ls -la node_modules/@spreedly/react-native-checkout/scripts/

# Test the require.resolve command
node -p "require.resolve('@spreedly/react-native-checkout/scripts/spreedly_pods_setup.rb')"

# If the above fails, try the fallback
node -p "require('path').join(require('path').dirname(require.resolve('@spreedly/react-native-checkout/package.json')), 'scripts/spreedly_pods_setup.rb')"
```

#### 3. **Setup Script Can't Find .env File**

**Problem**: `Error: .env file not found` when running the setup script

**Solution**: Ensure your `.env` file is in the correct location:

```bash
# Check if .env file exists in your project root
ls -la .env

# If not, create it in your project root (not in node_modules)
echo "GITHUB_USERNAME=your_github_username" > .env
echo "GITHUB_TOKEN=your_github_personal_access_token" >> .env

# Add to .gitignore
echo ".env" >> .gitignore

# Run the script from your project root
./node_modules/@spreedly/react-native-checkout/scripts/setup_local_dev_ios.sh
```

**The script searches for `.env` in this order:**

1. Your project root (`./.env`) - **recommended location**
2. Your project's example directory (`./example/.env`)
3. SDK's directories (fallback)

**Common locations where users mistakenly put .env:**

- ❌ `node_modules/@spreedly/react-native-checkout/.env`
- ❌ `ios/.env`
- ✅ `./.env` (project root - correct location)

**Common Error Messages**:

- `fatal: could not read Username for 'https://github.com'` → Missing GitHub credentials
- `HTTP 401 Unauthorized` → Invalid or expired GitHub token
- `Package not found` → Missing repository access or incorrect token permissions
- `Unable to load Maven metadata` → Android Gradle configuration issue

#### 4. **SDK Initialization Fails** 🚀

**Problem**: `SpreedlyCore.initSdk()` throws an error

**Root Cause Analysis:**

SDK initialization failures typically occur due to:

- **Missing Authentication Parameters**: Required fields like `token`, `nonce`, or `signature` are undefined or empty
- **Invalid Credentials**: Authentication parameters are malformed or expired
- **Network Connectivity**: Unable to reach Spreedly servers for validation
- **Timing Issues**: Attempting to initialize before React Native bridge is ready
- **Environment Mismatch**: Using test credentials in production or vice versa

**Common Error Messages:**

- `"Environment key is required"` → Missing or empty environment key
- `"Invalid signature"` → Signature doesn't match expected value
- `"Network request failed"` → Connectivity or server issues
- `"SDK already initialized"` → Attempting to initialize multiple times

**Impact**: Without proper initialization, no SDK methods will work, preventing payment processing entirely.

**Prevention Strategy:**

- Always validate authentication parameters before calling `initSdk()`
- Implement proper error handling and retry logic
- Use environment-specific configuration management
- Initialize early in your app lifecycle but after React Native is ready

**Solutions**:

```typescript
// Check all required parameters
const options = {
  token: authParams.certificateToken, // Must not be empty
  nonce: authParams.nonce, // Must not be empty
  signature: authParams.signature, // Must not be empty
  certificateToken: authParams.certificateToken,
  timestamp: authParams.timestamp.toString(), // Must be string
  environmentKey: process.env.SPREEDLY_ENVIRONMENT_KEY, // Must not be empty
  forterSiteId: process.env.FORTER_SITE_ID || '', // Can be empty string if not using Forter
};

// Validate before init
if (!options.environmentKey) {
  throw new Error('Environment key is required');
}
```

#### 5. **Fields Not Validating**

**Problem**: `onValidationChange` not firing

**Solutions**:

```typescript
// Ensure proper event handling
<SPLTextField
  formFieldType={FormFieldTypes.CARD}
  label="Card Number"
  onValidationChange={(isValid) => {
    console.log('Validation:', isValid); // Debug log
    setFieldValidation(prev => ({ ...prev, [FormFieldTypes.CARD]: isValid }));
  }}
/>
```

#### 6. **Metro Bundle Issues**

**Problem**: Package not found or bundle errors

**Solutions**:

```bash
# Clear Metro cache
npx react-native start --reset-cache

# Clean and reinstall
rm -rf node_modules
npm install
cd ios && pod install
```

#### 7. **iOS Build Errors**

**Problem**: CocoaPods or Xcode build failures

**Solutions**:

```bash
# Update CocoaPods
cd ios
bundle install
bundle exec pod install --repo-update

# Clean Xcode build
rm -rf ios/build
```

#### 8. **Android Build Errors**

**Problem**: Gradle or NDK issues

**Solutions**:

```bash
# Clean Android build
cd android
./gradlew clean

# Verify NDK version
android {
  ndkVersion "27.1.12297006"
}
```

#### 9. **Kotlin Version Compatibility Issues**

**Problem**: Build fails with Kotlin-related errors like:

- `Could not find org.jetbrains.kotlin:kotlin-compose-compiler-plugin-embeddable:X.X.X`
- `Kotlin version mismatch` errors
- Compose compilation errors

**Root Cause**: Version mismatch between your project's Kotlin version and the Spreedly SDK's requirements.

**Solution**: Update your `android/build.gradle` to use compatible versions:

```gradle
// android/build.gradle
buildscript {
    ext {
        // Update these versions to match Spreedly SDK requirements
        kotlinVersion = "2.0.21"  // Must be 2.0.21 or higher
    }
    dependencies {
        // Update Android Gradle Plugin for Kotlin 2.0.21+ compatibility
        classpath("com.android.tools.build:gradle:8.7.2")
        classpath("org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlinVersion")
    }
}
```

**Verification steps**:

```bash
# Clean and rebuild after version updates
cd android
./gradlew clean
./gradlew build

# Check if Kotlin versions are aligned
./gradlew dependencies | grep kotlin
```

**Common version combinations that work:**

- **React Native 0.76+**: Kotlin 2.0.21 + AGP 8.7.2
- **React Native 0.75**: Kotlin 1.9.25 + AGP 8.5.2 (not recommended)
- **React Native 0.72-0.74**: Not supported by Spreedly SDK

#### 10. **Android Lint Failures with Kotlin 2.0.21+**

**Problem**: Build fails during lint analysis with errors like:

- `Unexpected failure during lint analysis`
- `Found class org.jetbrains.kotlin.analysis.api.resolution.KaCallableMemberCall, but interface was expected`
- `The crash seems to involve the detector androidx.lifecycle.lint.NonNullableMutableLiveDataDetector`

**Root Cause**: Android Lint has compatibility issues with Kotlin 2.0.21+ and newer Compose versions.

**Solution**: The Spreedly SDK automatically handles these lint compatibility issues. The SDK's `android/build.gradle` already includes the necessary lint configuration to prevent these failures.

If you're still experiencing lint issues, verify that you're using the correct Kotlin version:

```gradle
// android/build.gradle
buildscript {
    ext {
        kotlinVersion = "2.0.21"  // Must be 2.0.21 or higher
    }
}
```

**Verification**:

```bash
cd android
./gradlew lintDebug  # Test lint analysis
./gradlew build      # Full build with lint
```

### Debug Mode

Enable debug logging for troubleshooting:

```typescript
// Add to your initialization
if (__DEV__) {
  console.log('Spreedly SDK Debug Mode Enabled');

  // Log all payment events
  SpreedlyCore.addListener('onPaymentResult');
  // Add custom debugging here
}
```

### Performance Monitoring

```typescript
export function usePerformanceMonitoring() {
  useEffect(() => {
    const startTime = Date.now();

    return () => {
      const endTime = Date.now();
      console.log(`Component lifecycle: ${endTime - startTime}ms`);
    };
  }, []);
}
```

## Comprehensive Troubleshooting Checklist

Use this checklist to systematically diagnose and resolve integration issues:

### 🔍 **Pre-Integration Verification**

- [ ] **Repository Access Confirmed**
  - [ ] Access granted to all three private repositories
  - [ ] GitHub username added to Spreedly organization
  - [ ] Repository access tested (can view repositories in browser)

- [ ] **GitHub Token Setup**
  - [ ] Personal Access Token created with correct permissions
  - [ ] Token includes `read:packages`, `repo`, and `read:org` scopes
  - [ ] Token is not expired (check expiration date)
  - [ ] Token tested with GitHub API calls

- [ ] **Environment Variables**
  - [ ] `GITHUB_USERNAME` set correctly
  - [ ] `GITHUB_TOKEN` set correctly
  - [ ] Variables accessible in build environment
  - [ ] `.env` file in correct location (project root)
  - [ ] `.env` file added to `.gitignore`

### 📦 **Package Installation Verification**

- [ ] **Package Manager Configuration**
  - [ ] `.npmrc` or `.yarnrc` configured for GitHub Packages
  - [ ] Registry configuration points to correct GitHub Packages URL
  - [ ] Authentication token properly referenced
  - [ ] No conflicting registry configurations

- [ ] **Installation Process**
  - [ ] `npm install` or `yarn install` completes without errors
  - [ ] SDK package appears in `node_modules/@spreedly/react-native-checkout`
  - [ ] Package version matches expected version
  - [ ] Dependencies resolved correctly

### 🛠️ **Platform-Specific Setup**

**Android Checklist:**

- [ ] **Version Compatibility**
  - [ ] Kotlin version 2.0.21 or higher
  - [ ] Android Gradle Plugin 8.7.2 or higher
  - [ ] Gradle wrapper 8.9 or higher
  - [ ] React Native 0.76 or higher

- [ ] **Gradle Configuration**
  - [ ] `spreedly_github_setup.gradle` applied in `android/build.gradle`
  - [ ] Environment variables accessible during Gradle sync
  - [ ] No conflicting repository configurations
  - [ ] Gradle sync completes successfully

- [ ] **Build Verification**
  - [ ] `./gradlew dependencies` shows Spreedly dependencies
  - [ ] `./gradlew build` completes without errors
  - [ ] No lint failures related to Kotlin compatibility

**iOS Checklist:**

- [ ] **CocoaPods Setup**
  - [ ] `spreedly_pods_setup.rb` required in Podfile
  - [ ] `init_spreedly_checkout_pods()` called in Podfile
  - [ ] CocoaPods version is latest
  - [ ] Xcode version 15 or higher

- [ ] **Pod Installation**
  - [ ] `pod install` completes without errors
  - [ ] Spreedly pods appear in `Pods/` directory
  - [ ] No credential-related errors during installation
  - [ ] Podfile.lock contains Spreedly dependencies

- [ ] **Xcode Build**
  - [ ] Project opens in Xcode without errors
  - [ ] Build succeeds for both simulator and device
  - [ ] No missing framework errors

### 🚀 **Runtime Verification**

- [ ] **SDK Initialization**
  - [ ] All authentication parameters provided
  - [ ] Parameters are valid and not expired
  - [ ] `initSdk()` completes without errors
  - [ ] Network connectivity available
  - [ ] Correct environment (test vs. production)

- [ ] **Component Integration**
  - [ ] `SPLTextField` components render correctly
  - [ ] Field validation callbacks fire properly
  - [ ] No console errors during field interactions
  - [ ] Payment submission works end-to-end

### 🔧 **Development Environment**

- [ ] **Metro/Bundler**
  - [ ] Metro cache cleared if needed
  - [ ] No bundle resolution errors
  - [ ] TypeScript compilation succeeds (if using TypeScript)
  - [ ] No import/export errors

- [ ] **Device/Simulator Testing**
  - [ ] Tested on physical devices
  - [ ] Tested on simulators/emulators
  - [ ] Both debug and release builds work
  - [ ] Performance acceptable on target devices

### 🌐 **Network and Security**

- [ ] **Corporate Environment**
  - [ ] Proxy settings configured if needed
  - [ ] Firewall allows GitHub Packages access
  - [ ] SSL/TLS certificates trusted
  - [ ] VPN doesn't interfere with package resolution

- [ ] **CI/CD Pipeline**
  - [ ] Environment variables available in CI
  - [ ] Build agents have necessary permissions
  - [ ] Caching strategies don't interfere with private packages
  - [ ] Deployment process includes all necessary dependencies

### 🐛 **Common Issue Quick Checks**

- [ ] **"Package not found" errors**
  - [ ] Verify repository access
  - [ ] Check token permissions
  - [ ] Confirm package manager configuration

- [ ] **"Authentication failed" errors**
  - [ ] Verify token is not expired
  - [ ] Check token has correct scopes
  - [ ] Ensure environment variables are set

- [ ] **Build failures**
  - [ ] Check version compatibility matrix
  - [ ] Verify all setup scripts ran successfully
  - [ ] Clear caches and retry clean build

- [ ] **Runtime errors**
  - [ ] Validate SDK initialization parameters
  - [ ] Check network connectivity
  - [ ] Verify environment configuration

### 📞 **When to Contact Support**

Contact Spreedly support if:

- [ ] All checklist items pass but issues persist
- [ ] Encountering errors not covered in documentation
- [ ] Need assistance with enterprise/corporate environment setup
- [ ] Require help with custom build configurations
- [ ] Experience issues specific to your payment processor integration

**Support Information to Provide:**

- Complete error messages and stack traces
- Platform and version information (React Native, iOS, Android)
- Build logs (with sensitive information redacted)
- Steps to reproduce the issue
- Environment details (development, staging, production)

---

## Support

For additional support and questions:

- **Documentation**: [Spreedly Documentation](https://docs.spreedly.com)
- **GitHub Issues**: [React Native SDK Issues](https://github.com/spreedly/checkout-react-native/issues)
- **Support**: Contact Spreedly Support

---

_This guide covers the essential integration patterns for the Spreedly React Native SDK. For the most up-to-date information, please refer to the official documentation and changelog._
