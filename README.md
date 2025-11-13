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

- **[Integration Guide](docs/Integration_Guide.md)** - Complete integration walkthrough
- **[Examples](docs/Examples.md)** - Code examples and use cases
- **[Theme Guide](docs/Theme_Guide.md)** - Theme customization and dark mode support
- **[React Native 0.76 Requirements](docs/RN_076_Requirement.md)** - Version-specific requirements

### Privacy & Security

- **[Security Policy](docs/Security.md)** - Security vulnerability reporting and policy
- **[Unified Privacy](docs/Unified_Privacy.md)** - Comprehensive privacy documentation
- **[Platform Privacy Requirements](docs/Platform_Privacy_Requirements.md)** - Privacy requirements across platforms
- **[iOS Privacy Guide](docs/iOS_Privacy_Guide.md)** - iOS-specific privacy setup
- **[Android Data Safety Guide](docs/Android_Data_Safety_Guide.md)** - Android Data Safety form requirements
- **[Source Maps Security Guide](docs/Source_Maps_Security_Guide.md)** - Secure source maps configuration

### Platform-Specific Guides

- **[Android Lint Troubleshooting](docs/Android_Lint_Troubleshooting.md)** - Android-specific issues and troubleshooting
- **[Xcode Cloud](docs/Xcode_Cloud.md)** - Xcode Cloud integration guide

### Development & Testing

- **[Unit Testing](docs/Unit_Testing.md)** - Testing guide and best practices

### Distribution

- **[App Distribution Guide](docs/App_Distribution_Guide.md)** - App distribution and release process
- **[GitHub Private Release Guide](docs/GitHub_Private_Release_Guide.md)** - GitHub private release setup

### Troubleshooting

- **[Common Issues](docs/Integration_Guide.md#troubleshooting)** - General troubleshooting guide

### Version History

- **[CHANGELOG.md](CHANGELOG.md)** - Detailed version history and release notes

## 🔧 Compatibility

- **React Native**: 0.76.x
- **React**: 19.x
- **Android**: minSdk 24 (Android 7.0+), targetSdk 34, compileSdk 35
- **iOS**: 15.1+, Xcode 15+
- **Architectures**: Legacy and New Architecture (Fabric/TurboModules)

## 📋 Version History

For a detailed list of changes, new features, bug fixes, and updates between versions, please refer to our [CHANGELOG.md](CHANGELOG.md). The changelog provides comprehensive information about:

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

## 🚀 Getting Started

### 1. Installation and Setup

Follow our comprehensive [Integration Guide](docs/Integration_Guide.md) for step-by-step setup instructions.

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
          Alert.alert('Success', `Payment token: ${mapped.token}`);
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
      {paymentToken && <Text>Payment Token: {paymentToken}</Text>}
    </View>
  );
}
```
