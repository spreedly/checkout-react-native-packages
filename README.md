**⚠️ BETA - Not for Production Use ⚠️**

# @spreedly/react-native-checkout

A React Native SDK that provides secure payment processing through Spreedly's platform, supporting both native UI components and headless primitives for Android and iOS.

## 🚀 Quick Start

### Installation

```sh
# Install the package
yarn add @spreedly/react-native-checkout

# iOS setup
cd ios && pod install
```

### Android Setup

Add the required Maven repositories to your React Native app's `android/build.gradle` file:

```groovy
allprojects {
    repositories {
        // ... existing repositories (google(), mavenCentral(), etc.)

        // Spreedly GitHub Packages repository
        maven {
            url = uri("https://maven.pkg.github.com/spreedly/checkout-android-maven")
            credentials {
                username = System.getenv('GITHUB_USERNAME') ?: project.findProperty('GITHUB_USERNAME')
                password = System.getenv('GITHUB_TOKEN') ?: project.findProperty('GITHUB_TOKEN')
            }
        }

        // Forter 3DS SDK repository (required for 3DS authentication)
        maven {
            url = uri("https://mobile-sdks.forter.com/android")
            credentials {
                username = System.getenv('FORTER_USERNAME') ?: "forter-android-sdk"
                password = System.getenv('FORTER_PASSWORD') ?: ""
            }
        }
    }
}
```

**Environment Variables**: Set your GitHub credentials as environment variables or in `gradle.properties`:

```bash
# Option 1: Environment variables
export GITHUB_USERNAME=your_github_username
export GITHUB_TOKEN=your_github_personal_access_token

# Option 2: Add to ~/.gradle/gradle.properties
GITHUB_USERNAME=your_github_username
GITHUB_TOKEN=your_github_personal_access_token
```

> **Note**: Your GitHub token needs `read:packages` scope to download the Spreedly SDK packages.

### Basic Usage

Initialize the SDK once at app startup:

```ts
import { useEffect } from 'react';
import { SpreedlyCore } from '@spreedly/react-native-checkout';

export function App() {
  useEffect(() => {
    SpreedlyCore.initSdk({
      token: '<TOKEN>',
      nonce: '<NONCE>',
      signature: '<SIGNATURE>',
      certificateToken: '<CERTIFICATE_TOKEN>',
      timestamp: '<TIMESTAMP>',
      environmentKey: '<ENVIRONMENT_KEY>',
    });
  }, []);

  return <YourAppContent />;
}
```

Create secure payment forms using headless components:

```tsx
import { SPLTextField, FormFieldTypes } from '@spreedly/react-native-checkout';

export function PaymentForm() {
  return (
    <>
      <SPLTextField formFieldType={FormFieldTypes.CARD} label="Card Number" />
      <SPLTextField formFieldType={FormFieldTypes.MONTH} label="MM" />
      <SPLTextField formFieldType={FormFieldTypes.YEAR} label="YY" />
      <SPLTextField formFieldType={FormFieldTypes.CVV} label="CVV" />
    </>
  );
}
```

## 📚 Documentation

### Integration Guides

- **[Integration Guide](docs/guides/integration_guide.md)** - Complete integration walkthrough
- **[React Native 0.77+ Requirements](docs/guides/rn_077_requirement.md)** - Version-specific requirements

### SDK Reference

- **[API Reference](docs/guides/integration_guide.md#api-reference)** - Complete API documentation
- **[Component Reference](docs/guides/integration_guide.md#components)** - UI component documentation
- **[Event Types](docs/guides/integration_guide.md#event-types)** - Event system documentation

### Security & Privacy

- **[SECURITY.md](SECURITY.md)** - Spreedly security policy (vulnerability reporting and disclosure)
- **[Security Policy](docs/guides/security.md)** - Vulnerability reporting, security best practices, and compliance information
- **[Android Data Safety Guide](docs/guides/android_data_safety_guide.md)** - Android data safety declarations and requirements
- **[iOS Privacy Guide](docs/guides/ios_privacy_guide.md)** - iOS privacy manifest and App Store requirements
- **[Unified Privacy Guide](docs/guides/unified_privacy.md)** - Unified privacy documentation across platforms

### Troubleshooting

- **[Common Issues](docs/guides/integration_guide.md#troubleshooting)** - General troubleshooting guide

### Version History

- **[CHANGELOG.md](docs/CHANGELOG.md)** - Detailed version history and release notes

## 🔧 Compatibility

- **React Native**: 0.77+ (recommended 0.79+)
- **React**: 18.2+
- **Android**: minSdk 26 (Android 8.0+), targetSdk 34, compileSdk 36
- **iOS**: 15.1+, Xcode 15+
- **Architectures**: Legacy and New Architecture (Fabric/TurboModules)

## 📋 Version History

For a detailed list of changes, new features, bug fixes, and updates between versions, please refer to our [CHANGELOG.md](docs/CHANGELOG.md). The changelog provides comprehensive information about:

- **New Features**: Added functionality and enhancements
- **Bug Fixes**: Resolved issues and improvements
- **Breaking Changes**: API changes that may affect existing integrations
- **Security Updates**: Security patches and vulnerability fixes
- **Deprecations**: Features scheduled for removal

Use the changelog to understand what's changed between versions and to identify which version includes the features or fixes you need.

## 🏗️ Architecture Support

This SDK supports both React Native architectures:

- **Legacy Architecture**: Full compatibility with existing React Native apps
- **New Architecture (Fabric)**: Optimized performance with the latest React Native features

The SDK automatically detects and adapts to your app's architecture.

## 📦 Components

### Headless Components

Secure, PCI-compliant input components that handle sensitive payment data:

- `SPLTextField` - Secure text input for card details
- `PaymentBottomSheet` - Modal payment form
- Event emitters for payment lifecycle management

### Core Module

- `SpreedlyCore` - Main SDK initialization and configuration
- Payment method tokenization
- Secure data handling and transmission

## 🔐 Security

- **PCI DSS Compliant**: Secure handling of sensitive payment data
- **End-to-End Encryption**: All payment data is encrypted in transit
- **Tokenization**: Card data is tokenized and never stored locally
- **Secure Communication**: TLS 1.2+ for all network communications
- **Security Testing**: Comprehensive security testing including:
  - Static Application Security Testing (SAST) via CodeQL
  - Dependency vulnerability scanning
  - **Dynamic Application Security Testing (DAST)** - Runtime security scanning

## 🚀 Getting Started

### 1. Installation and Setup

Follow our comprehensive [Integration Guide](docs/guides/integration_guide.md) for step-by-step setup instructions.

### 2. Configuration

Configure your Spreedly environment and obtain the necessary credentials:

- Environment Key
- Authentication tokens
- Certificate tokens

### 3. Implementation

Choose your integration approach:

- **Headless Components**: Build custom UI with secure payment inputs
- **Pre-built Components**: Use ready-made payment forms
- **Hybrid Approach**: Combine both for maximum flexibility

### 4. Testing

Test your integration using Spreedly's test environment and test card numbers.

## 📖 Examples

### Basic Card Form

```tsx
import React, { useState } from 'react';
import { View, Button, Alert } from 'react-native';
import {
  SPLTextField,
  FormFieldTypes,
  SpreedlyCore,
  ValidationManager,
  mapPaymentResult,
  type FieldDescriptor,
  type CreateCreditCardResult,
} from '@spreedly/react-native-checkout';

export function BasicCardForm() {
  const [isProcessing, setIsProcessing] = useState(false);
  const [fieldValidation, setFieldValidation] = useState<
    Record<string, boolean>
  >({});

  const fields: FieldDescriptor[] = [
    { type: FormFieldTypes.CARD, required: true },
    { type: FormFieldTypes.EXPIRY_DATE, required: true },
    { type: FormFieldTypes.CVV, required: true },
  ];

  const isFormValid = ValidationManager.isFormValid(fields, fieldValidation);

  const handleSubmit = async () => {
    if (!isFormValid || isProcessing) return;

    setIsProcessing(true);
    try {
      const result: CreateCreditCardResult =
        await SpreedlyCore.createCreditCard({
          fields: fields,
          formFieldTypes: fields.map((f) => f.type),
        });

      const mapped = mapPaymentResult(result as any);

      switch (mapped.kind) {
        case 'success':
          // Send mapped.token to your backend over HTTPS
          Alert.alert('Success', 'Payment method saved');
          break;
        case 'validation':
          Alert.alert(
            'Validation Error',
            mapped.message || 'Please correct invalid fields'
          );
          break;
        case 'failed':
          Alert.alert('Error', mapped.message || 'Payment failed');
          break;
        case 'canceled':
          Alert.alert('Canceled', 'Payment was canceled');
          break;
      }
    } catch (error: any) {
      Alert.alert('Error', error.message || 'Payment processing failed');
    } finally {
      setIsProcessing(false);
    }
  };

  return (
    <View style={{ padding: 20 }}>
      <SPLTextField
        formFieldType={FormFieldTypes.CARD}
        label="Card Number"
        onValidationChange={(isValid) =>
          setFieldValidation((prev) => ({ ...prev, CARD: isValid }))
        }
      />
      <SPLTextField
        formFieldType={FormFieldTypes.EXPIRY_DATE}
        label="MM/YY"
        onValidationChange={(isValid) =>
          setFieldValidation((prev) => ({ ...prev, EXPIRY_DATE: isValid }))
        }
      />
      <SPLTextField
        formFieldType={FormFieldTypes.CVV}
        label="CVV"
        onValidationChange={(isValid) =>
          setFieldValidation((prev) => ({ ...prev, CVV: isValid }))
        }
      />

      <Button
        title={isProcessing ? 'Processing...' : 'Submit Payment'}
        onPress={handleSubmit}
        disabled={!isFormValid || isProcessing}
      />
    </View>
  );
}
```

### Payment Bottom Sheet

```tsx
import React, { useState, useEffect } from 'react';
import { View, Button, Alert, Text } from 'react-native';
import {
  SpreedlyCore,
  SpreedlyEventEmitter,
  SpreedlyEventTypes,
  mapPaymentResult,
  type PaymentResultRN,
} from '@spreedly/react-native-checkout';

export function PaymentScreen() {
  const [paymentToken, setPaymentToken] = useState<string | null>(null);

  useEffect(() => {
    // Listen for payment results
    const subscription = SpreedlyEventEmitter.addListener(
      SpreedlyEventTypes.PAYMENT_BOTTOM_SHEET_RESULT,
      (result: PaymentResultRN) => {
        const mapped = mapPaymentResult(result);

        switch (mapped.kind) {
          case 'success':
            setPaymentToken(mapped.token || null);
            Alert.alert('Success', 'Payment completed successfully!');
            break;
          case 'failed':
            Alert.alert('Error', mapped.message || 'Payment failed');
            break;
          case 'canceled':
            Alert.alert('Canceled', 'Payment was canceled');
            break;
          case 'validation':
            Alert.alert(
              'Validation Error',
              mapped.message || 'Please correct invalid fields'
            );
            break;
        }
      }
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
      <Button title="Pay Now" onPress={handlePaymentPress} />
      {paymentToken && <Text>Payment method saved</Text>}
    </View>
  );
}
```

## 🛠️ Development

### For SDK Users

If you're integrating this SDK into your app, you only need the documentation above. The SDK is distributed as a compiled package and doesn't require any build steps.

## 📋 Release Notes

### Latest Release

See [GitHub Releases](https://github.com/spreedly/checkout-react-native/releases) for the latest version and changelog.

### Distribution

This package is distributed through:

- **GitHub Packages** (Private): `@spreedly/react-native-checkout`

## 🔐 Package Security Verification

Every release includes SHA-256 checksums and GPG signatures for package integrity verification. This package is published by the verified **Spreedly** organization on GitHub.

## 🤝 Support

### Documentation

- [Integration Guide](docs/guides/integration_guide.md) - Complete setup and usage guide
- [API Reference](docs/guides/integration_guide.md#api-reference) - Detailed API documentation
- [Go-live / ops checklist](docs/GO_LIVE_INDEX.md) - Production readiness and CI runbook map

### Spreedly support and security

- **Support portal / Help Center**: [spreedly.com/support](https://spreedly.com/support/)
- **Spreedly Trust Center (security)**: [security@spreedly.com](mailto:security@spreedly.com) — see [SECURITY.md](SECURITY.md) for reporting options
- **Security policy (this repository)**: [SECURITY.md](SECURITY.md)

### Getting Help

- **Issues**: [GitHub Issues](https://github.com/spreedly/checkout-react-native/issues)
- **Internal Support**: #mobile-sdk Slack channel
- **Email**: mobile-team@spreedly.com

## License

Copyright 2025 Spreedly, Inc.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

See [LICENSE](LICENSE) file for details.