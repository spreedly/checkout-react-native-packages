# Braintree APM Payment Guide

Integrate PayPal and Venmo payments into your React Native application using the Spreedly Checkout SDK's Braintree Alternative Payment Methods (APM) module.

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Dependencies](#dependencies)
- [Platform Setup](#platform-setup)
  - [iOS Setup](#ios-setup)
  - [Android Setup](#android-setup)
- [API Reference](#api-reference)
  - [BraintreeAPM](#braintreeapm)
  - [BraintreeAPMCheckoutConfig](#braintreeapmcheckoutconfig)
  - [BraintreeAPMResult](#braintreeapmresult)
- [Integration Guide](#integration-guide)
  - [Step 1: Backend Purchase Request](#step-1-backend-purchase-request)
  - [Step 2: Initialize the Observer](#step-2-initialize-the-observer)
  - [Step 3: Present Checkout](#step-3-present-checkout)
  - [Step 4: Handle Payment Results](#step-4-handle-payment-results)
  - [Step 5: Confirm the Transaction](#step-5-confirm-the-transaction)
  - [Step 6: Cleanup](#step-6-cleanup)
- [Complete Example](#complete-example)
- [Payment Flow Diagram](#payment-flow-diagram)
- [Error Handling](#error-handling)
- [URL Scheme Handling](#url-scheme-handling)
- [Troubleshooting](#troubleshooting)

---

## Overview

The Braintree APM module enables PayPal and Venmo payment flows through Spreedly's unified checkout experience. The SDK wraps Braintree's native SDKs on both iOS and Android, presenting the native PayPal/Venmo authorization flows and returning results to your React Native layer via event-based communication.

**Supported Payment Types:**

| Payment Type | iOS | Android |
| ------------ | --- | ------- |
| PayPal       | Yes | Yes     |
| Venmo        | Yes | Yes     |

---

## Prerequisites

1. **Spreedly account** with a Braintree gateway configured
2. **Braintree merchant account** with PayPal and/or Venmo enabled
3. **Spreedly Checkout SDK** initialized in your React Native app (`initSdk` called with your environment key)
4. **Backend service** capable of calling Spreedly's purchase and confirm APIs

---

## Dependencies

### npm / React Native

```json
{
  "@spreedly/react-native-checkout": ">=0.2.5"
}
```

### iOS (CocoaPods)

The SDK's podspec (`SpreedlyCheckout.podspec`) declares these dependencies automatically:

| Pod                 | Version     | Purpose                                           |
| ------------------- | ----------- | ------------------------------------------------- |
| `SpreedlyCore`      | Private SDK | Core Spreedly payment engine                      |
| `SpreedlySecurity`  | Private SDK | Security and encryption                           |
| `SpreedlyUI`        | Private SDK | Shared UI components                              |
| `SpreedlyBraintree` | Private SDK | Braintree APM wrapper                             |
| `Braintree`         | `~> 7.0`    | Braintree iOS SDK (PayPal, Venmo, data collector) |
| `Forter3DS`         | latest      | 3D Secure support                                 |

**Podfile setup:**

```ruby
# Load Spreedly pod setup script
load '../../scripts/spreedly_pods_setup.rb'

target 'YourApp' do
  # ... React Native config ...

  # Adds all required Spreedly pods including Braintree
  init_spreedly_checkout_pods()
end
```

The `init_spreedly_checkout_pods()` function (from `scripts/spreedly_pods_setup.rb`) installs all required pods from the Spreedly private repository:

```ruby
def init_spreedly_checkout_pods
  private_repo = get_private_repo_url()

  pod 'Forter3DS', :git => 'https://bitbucket.org/forter-mobile/forter-ios.git'
  pod 'SpreedlySecurity', :git => private_repo, :tag => "1.0.29"
  pod 'SpreedlyCore', :git => private_repo, :tag => "1.0.29"
  pod 'SpreedlyUI', :git => private_repo, :tag => "1.0.29"
  pod 'SpreedlyStripeAPM', :git => private_repo, :tag => "1.0.29"
  pod 'SpreedlyBraintree', :git => private_repo, :tag => "1.0.29"
  pod 'StripePaymentSheet', '~> 25.0'
  pod 'Braintree', '~> 7.0'
end
```

### Android (Gradle)

The SDK module's `build.gradle` declares these dependencies:

| Artifact                                           | Version  | Purpose                        |
| -------------------------------------------------- | -------- | ------------------------------ |
| `com.spreedly:checkout-payments-core`              | `0.10.2` | Core Spreedly payment engine   |
| `com.spreedly:checkout-hostedfields`               | `0.10.2` | Hosted fields support          |
| `com.spreedly:checkout-paymentsheet`               | `0.10.2` | Payment sheet UI               |
| `com.spreedly:checkout-braintree-apm`              | `0.10.2` | Braintree APM integration      |
| `com.forter.mobile:forter3ds`                      | `2.0.4`  | 3D Secure support              |
| `io.ktor:ktor-client-*`                            | `2.3.12` | HTTP client for SDK networking |
| `org.jetbrains.kotlinx:kotlinx-serialization-json` | `1.6.3`  | JSON serialization             |

**Gradle repository setup** (`scripts/spreedly_github_setup.gradle`):

```groovy
gradle.allprojects { project ->
  project.repositories {
    maven {
      url = uri("https://maven.pkg.github.com/spreedly/checkout-android-maven")
      credentials {
        username = env.GITHUB_USERNAME ?: System.getenv('GITHUB_USERNAME')
        password = env.GITHUB_TOKEN ?: System.getenv('GITHUB_TOKEN')
      }
    }
    maven {
      url = uri("https://mobile-sdks.forter.com/android")
      credentials {
        username = "forter-android-sdk"
        password = "<forter-token>"
      }
    }
    google()
    mavenCentral()
  }
}
```

> **Note:** Android dependencies require a GitHub Personal Access Token with `read:packages` scope to access the Spreedly Maven repository. Set `GITHUB_USERNAME` and `GITHUB_TOKEN` in your `.env` file or as environment variables.

---

## Platform Setup

### iOS Setup

#### 1. URL Scheme Registration

Register a URL scheme in your `Info.plist` so PayPal/Venmo can redirect back to your app:

```xml
<key>CFBundleURLTypes</key>
<array>
  <dict>
    <key>CFBundleTypeRole</key>
    <string>Editor</string>
    <key>CFBundleURLName</key>
    <string>com.yourcompany.yourapp.braintree</string>
    <key>CFBundleURLSchemes</key>
    <array>
      <string>$(PRODUCT_BUNDLE_IDENTIFIER).spreedly.braintree</string>
    </array>
  </dict>
</array>
```

#### 2. AppDelegate URL Handling

Handle the return URL in your `AppDelegate.swift`:

```swift
import SpreedlyCore
import SpreedlyBraintree

class AppDelegate: UIResponder, UIApplicationDelegate {
  // ...

  func application(
    _ app: UIApplication,
    open url: URL,
    options: [UIApplication.OpenURLOptionsKey : Any] = [:]
  ) -> Bool {
    // Try Braintree URL handler first (PayPal/Venmo return)
    if BraintreeURLHandler.handleOpen(url: url) {
      return true
    }
    // Fall back to offsite payment return handler
    Spreedly.shared().handleOffsiteReturn(url: url)
    return true
  }
}
```

### Android Setup

#### 1. AndroidManifest Activity Declaration

Register the Braintree APM activity in your `AndroidManifest.xml` to handle PayPal/Venmo redirects:

```xml
<application>
  <!-- Your main activity -->

  <!-- Braintree APM return handler -->
  <activity
    android:name="com.spreedly.braintree.BraintreeAPMActivity"
    android:exported="true"
    android:launchMode="singleTop"
    android:theme="@style/Theme.Spreedly.Transparent">
    <intent-filter>
      <action android:name="android.intent.action.VIEW" />
      <category android:name="android.intent.category.DEFAULT" />
      <category android:name="android.intent.category.BROWSABLE" />
      <data android:scheme="${applicationId}.braintree" />
    </intent-filter>
  </activity>
</application>
```

> **Note:** The `${applicationId}` placeholder is automatically replaced by your app's package name at build time.

---

## API Reference

### BraintreeAPM

The main module exported from `@spreedly/react-native-checkout`.

```typescript
import { BraintreeAPM } from '@spreedly/react-native-checkout';
```

| Method                    | Signature                                               | Description                                                              |
| ------------------------- | ------------------------------------------------------- | ------------------------------------------------------------------------ |
| `initialize()`            | `() => void`                                            | Initialize the payment result observer. Call before presenting checkout. |
| `presentCheckout(config)` | `(config: BraintreeAPMCheckoutConfig) => Promise<void>` | Present the native PayPal/Venmo authorization flow.                      |
| `cleanup()`               | `() => void`                                            | Release resources and cancel subscriptions. Call on unmount.             |

### BraintreeAPMCheckoutConfig

Configuration passed to `BraintreeAPM.presentCheckout()`.

```typescript
interface BraintreeAPMCheckoutConfig {
  /** Spreedly transaction token from purchase response */
  transactionToken: string;

  /** Payment method type */
  paymentType: 'paypal' | 'venmo';

  /** Merchant name displayed during authorization */
  merchantDisplayName: string;

  /** Braintree client token from gateway_specific_response_fields */
  clientToken?: string;

  /** Payment amount (formatted as string, e.g., "75.00") */
  amount?: string;

  /** ISO 4217 currency code (e.g., "USD") */
  currencyCode?: string;
}
```

> **Important:** `clientToken` is marked optional in the TypeScript interface for flexibility, but it is **required at runtime**. The native layer validates its presence and will return a failure result if missing. The client token is obtained from the purchase response at `gateway_specific_response_fields.braintree.client_token`.

### BraintreeAPMResult

Result emitted via the `BRAINTREE_APM_PAYMENT_RESULT` event.

```typescript
type BraintreeAPMResult =
  | {
      status: 'success';
      transactionToken: string;
      nonce?: string;
      deviceData?: string;
      state?: string;
    }
  | {
      status: 'failed';
      message?: string;
      state?: string;
      failureDetails?: { message: string };
    }
  | {
      status: 'canceled';
    };
```

| Status     | Fields                                             | Description                                                                                    |
| ---------- | -------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| `success`  | `transactionToken`, `nonce`, `deviceData`, `state` | Payment authorized. Use `nonce` to confirm the transaction.                                    |
| `failed`   | `message`, `state`, `failureDetails`               | Payment failed. `state: "pending"` indicates a timeout — check transaction status server-side. |
| `canceled` | —                                                  | User dismissed the payment flow.                                                               |

---

## Integration Guide

### Step 1: Backend Purchase Request

Create a pending purchase transaction via your backend. Your backend calls the Spreedly Purchase API with the Braintree gateway:

```typescript
import { purchaseBraintreeAPM } from './network/purchaseBraintree';

const response = await purchaseBraintreeAPM({
  amount: 7500, // Amount in cents
  currency_code: 'USD',
  redirect_url: 'yourapp://com.yourcompany.yourapp/braintree/checkout',
  callback_url: 'https://yourbackend.com/api/braintree/callback',
});

// response contains:
// - transaction_token: string  (Spreedly transaction token)
// - client_token: string       (Braintree client token for SDK auth)
// - state: string              (transaction state, typically "processing")
```

The `client_token` is extracted from the Spreedly response at:

```
transaction.gateway_specific_response_fields.braintree.client_token
```

### Step 2: Initialize the Observer

Initialize the Braintree APM observer when your payment component mounts. This sets up native-side subscriptions to receive payment results:

```typescript
import {
  BraintreeAPM,
  SpreedlyEventEmitter,
  SpreedlyEventTypes,
  type BraintreeAPMResult,
} from '@spreedly/react-native-checkout';

useEffect(() => {
  // Initialize the native observer
  BraintreeAPM.initialize();

  // Subscribe to payment results
  const subscription = SpreedlyEventEmitter.addListener(
    SpreedlyEventTypes.BRAINTREE_APM_PAYMENT_RESULT,
    (result: BraintreeAPMResult) => {
      handlePaymentResult(result);
    }
  );

  return () => {
    subscription.remove();
    BraintreeAPM.cleanup();
  };
}, []);
```

### Step 3: Present Checkout

After obtaining the `transaction_token` and `client_token` from your backend, present the native checkout:

```typescript
await BraintreeAPM.presentCheckout({
  transactionToken: response.transaction_token,
  paymentType: 'paypal', // or 'venmo'
  merchantDisplayName: 'Your Store Name',
  clientToken: response.client_token,
  amount: '75.00',
  currencyCode: 'USD',
});
```

This opens the native PayPal or Venmo authorization flow (browser or app).

### Step 4: Handle Payment Results

Process the result event in your handler:

```typescript
const handlePaymentResult = async (result: BraintreeAPMResult) => {
  switch (result.status) {
    case 'success':
      if (result.nonce) {
        // Nonce received — confirm the transaction on your backend
        await confirmPayment(result.transactionToken, result.nonce);
      } else {
        // No nonce — transaction may have been completed server-side
        showSuccess();
      }
      break;

    case 'failed':
      if (result.state === 'pending') {
        // Timeout — check transaction status via your backend
        showWarning('Payment status unknown. Please check your order.');
      } else {
        showError(result.message || 'Payment failed');
      }
      break;

    case 'canceled':
      showInfo('Payment canceled');
      break;
  }
};
```

### Step 5: Confirm the Transaction

When you receive a `success` result with a `nonce`, send it to your backend to call Spreedly's confirm endpoint:

```typescript
const confirmPayment = async (transactionToken: string, nonce: string) => {
  const response = await fetch(
    `https://yourbackend.com/api/transactions/${transactionToken}/confirm`,
    {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        state: 'Successful',
        nonce: nonce,
        payment_method_type: 'paypal', // or 'venmo'
      }),
    }
  );

  const data = await response.json();
  // Handle confirmation result
};
```

### Step 6: Cleanup

Always clean up when your payment component unmounts:

```typescript
useEffect(() => {
  return () => {
    BraintreeAPM.cleanup();
  };
}, []);
```

---

## Complete Example

```typescript
import React, { useState, useCallback, useEffect } from 'react';
import { View, Text, Button, Alert } from 'react-native';
import {
  BraintreeAPM,
  SpreedlyEventEmitter,
  SpreedlyEventTypes,
  type BraintreeAPMResult,
} from '@spreedly/react-native-checkout';

const PaymentScreen: React.FC = () => {
  const [isProcessing, setIsProcessing] = useState(false);
  const transactionTokenRef = React.useRef<string | null>(null);

  const handleResult = useCallback(async (result: BraintreeAPMResult) => {
    if (result.status === 'success' && result.nonce) {
      try {
        await confirmBraintreeAPM({
          transaction_token: result.transactionToken,
          state: 'Successful',
          nonce: result.nonce,
          payment_method_type: 'paypal',
        });
        Alert.alert('Success', 'Payment completed!');
      } catch (error) {
        Alert.alert('Error', 'Failed to confirm payment');
      }
    } else if (result.status === 'failed') {
      Alert.alert('Failed', result.message || 'Payment failed');
    } else if (result.status === 'canceled') {
      Alert.alert('Canceled', 'Payment was canceled');
    }
    setIsProcessing(false);
  }, []);

  useEffect(() => {
    BraintreeAPM.initialize();

    const subscription = SpreedlyEventEmitter.addListener(
      SpreedlyEventTypes.BRAINTREE_APM_PAYMENT_RESULT,
      handleResult
    );

    return () => {
      subscription.remove();
      BraintreeAPM.cleanup();
    };
  }, [handleResult]);

  const handlePayment = async () => {
    setIsProcessing(true);
    try {
      const response = await purchaseBraintreeAPM({
        amount: 7500,
        currency_code: 'USD',
        redirect_url: 'yourapp://braintree/checkout',
        callback_url: 'https://yourbackend.com/callback',
      });

      transactionTokenRef.current = response.transaction_token;

      await BraintreeAPM.presentCheckout({
        transactionToken: response.transaction_token,
        paymentType: 'paypal',
        merchantDisplayName: 'My Store',
        clientToken: response.client_token,
        amount: '75.00',
        currencyCode: 'USD',
      });
    } catch (error) {
      setIsProcessing(false);
      Alert.alert('Error', (error as Error).message);
    }
  };

  return (
    <View style={{ flex: 1, justifyContent: 'center', padding: 20 }}>
      <Button
        title={isProcessing ? 'Processing...' : 'Pay $75.00 with PayPal'}
        onPress={handlePayment}
        disabled={isProcessing}
      />
    </View>
  );
};
```

---

## Payment Flow Diagram

```
┌──────────────┐       ┌──────────────┐       ┌──────────────┐
│  React Native │       │   Backend    │       │   Spreedly   │
│     App       │       │   Server     │       │     API      │
└──────┬───────┘       └──────┬───────┘       └──────┬───────┘
       │                      │                      │
       │  1. Purchase request │                      │
       │─────────────────────>│                      │
       │                      │  2. POST /purchase   │
       │                      │─────────────────────>│
       │                      │                      │
       │                      │  3. transaction_token│
       │                      │     + client_token   │
       │                      │<─────────────────────│
       │  4. Response         │                      │
       │<─────────────────────│                      │
       │                      │                      │
       │  5. BraintreeAPM.presentCheckout()          │
       │──────────────────────────────────────────>  │
       │                      │                      │
       │  6. User authorizes in PayPal/Venmo app     │
       │  ←─── (browser/app redirect) ───>           │
       │                      │                      │
       │  7. BraintreeAPMResult (nonce)              │
       │<─────────────── (event) ────────────────    │
       │                      │                      │
       │  8. Confirm with nonce                      │
       │─────────────────────>│                      │
       │                      │  9. POST /confirm    │
       │                      │─────────────────────>│
       │                      │                      │
       │                      │  10. Confirmation    │
       │                      │<─────────────────────│
       │  11. Final result    │                      │
       │<─────────────────────│                      │
```

---

## Error Handling

### Validation Errors

The native layer validates configuration before presenting checkout:

| Error                              | Cause                                                         |
| ---------------------------------- | ------------------------------------------------------------- |
| `transactionToken is required`     | Missing or empty `transactionToken`                           |
| `paymentType is required`          | Missing or empty `paymentType`                                |
| `clientToken is required`          | Missing `clientToken` from purchase response                  |
| `Unsupported payment type: <type>` | `paymentType` is not `paypal` or `venmo`                      |
| `SDK not initialized`              | (Android) `initSdk` was not called before presenting checkout |

### Timeout Handling

The Android platform implements a timeout mechanism to prevent the UI from being stuck in a processing state:

- **Android:** 3-second timeout after the host activity resumes. If no payment result is received, a `failed` result with `state: "pending"` is emitted.

When you receive a `pending` state, you should query your backend for the actual transaction status, as the payment may still have succeeded server-side.

### Network / Gateway Errors

Payment failures from Braintree are surfaced via the `failed` status with details in the `message` and `failureDetails` fields.

---

## URL Scheme Handling

PayPal and Venmo redirect the user back to your app via a URL scheme after authorization. Proper URL scheme configuration is critical for the payment flow to complete.

### iOS

- **URL Scheme:** `$(PRODUCT_BUNDLE_IDENTIFIER).spreedly.braintree`
- **Handler:** `BraintreeURLHandler.handleOpen(url:)` in `AppDelegate`
- The SDK listens for `UIApplication.didBecomeActiveNotification` after checkout to detect app foreground return

### Android

- **URL Scheme:** `${applicationId}.braintree`
- **Handler:** `BraintreeAPMActivity` declared in `AndroidManifest.xml`
- The SDK uses `ActivityLifecycleCallbacks.onActivityResumed` to detect return and calls `SpreedlyBraintreeAPMCheckout.finalizeIfActive()` to trigger result emission

---

## Troubleshooting

### Pay button stuck in "Processing..."

**Cause:** The app returned from the PayPal/Venmo flow but no result event was emitted.

**Solutions:**

1. Verify URL schemes are correctly configured in `Info.plist` (iOS) or `AndroidManifest.xml` (Android)
2. Ensure `BraintreeURLHandler.handleOpen(url:)` is called in your `AppDelegate` (iOS)
3. Ensure `BraintreeAPMActivity` is declared in `AndroidManifest.xml` (Android)
4. The SDK includes built-in timeout fallbacks, so the button should eventually reset with a `pending` failure

### "clientToken is required" error

**Cause:** The purchase response did not include a Braintree client token.

**Solution:** Ensure your Braintree gateway is properly configured in Spreedly and that the purchase response includes `gateway_specific_response_fields.braintree.client_token`.

### Venmo redirects but payment doesn't complete

**Cause:** Venmo app is not installed, or the URL scheme is misconfigured.

**Solutions:**

1. Venmo requires the Venmo app to be installed on the device
2. Verify the URL scheme matches your bundle identifier
3. On the Braintree dashboard, ensure Venmo is enabled for your merchant account

### Android: "SDK not initialized"

**Cause:** `initSdk` was not called before attempting to present checkout.

**Solution:** Call `initSdk` with your Spreedly environment key before initializing the Braintree APM observer.

### iOS: Build errors with Braintree pods

**Cause:** Missing access to the Spreedly private pod repository.

**Solution:** Ensure SSH key access to `github.com:spreedly/checkout-ios-package.git` is configured. Run `pod install --repo-update` after configuration changes.