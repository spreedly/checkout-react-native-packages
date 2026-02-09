# Gateway-Specific 3DS Integration Guide

This guide covers integration of Gateway-Specific 3DS authentication for React Native applications using the Spreedly Checkout SDK.

## Overview

Gateway-Specific 3DS is used for payment gateways that implement their own 3DS authentication mechanism instead of using the standard EMV 3DS protocol. The SDK handles the authentication flow, challenge UI presentation, and result processing.

## When Do You Need Gateway-Specific 3DS?

You need Gateway-Specific 3DS when your **purchase API response** indicates that authentication is required.

### Check Your Purchase Response

After calling your purchase endpoint, check the response:

```typescript
const purchaseResponse = await fetch('/api/purchase', {
  method: 'POST',
  body: JSON.stringify({
    payment_method_token: token,
    amount: 100,
    attempt_3dsecure: true, // ⚠️ Important: Set this to true
  }),
});

const data = await purchaseResponse.json();

// Check if Gateway-Specific 3DS is required
const requires3DS =
  data.transaction?.state === 'pending' ||
  data.transaction?.scaAuthentication?.requiredAction === 'device_fingerprint';

if (requires3DS) {
  // Use GatewaySpecific3DS!
  const transactionToken = data.transaction?.token;
  await GatewaySpecific3DS.startFlow(transactionToken);
} else if (data.transaction?.succeeded) {
  // Payment completed without 3DS
  console.log('Payment successful!');
}
```

## Prerequisites

- Spreedly Checkout React Native SDK installed and configured
- SDK initialized with valid environment key
- Payment gateway that supports Gateway-Specific 3DS

## Platform Requirements

### Android

- Minimum API Level: 21 (Android 5.0)
- Jetpack Compose support
- FragmentActivity context required

### iOS

- Minimum iOS version: 13.0
- SwiftUI support
- UIHostingController for challenge UI

## Installation

Ensure the Spreedly Checkout SDK is properly installed. The Gateway-Specific 3DS module is included automatically.

```bash
npm install @spreedly/checkout-react-native
```

## Integration

### Step 1: Import the Module

```typescript
import { GatewaySpecific3DS } from '@spreedly/checkout-react-native';
```

### Step 2: Initialize with Event Listeners

Initialize the module before starting any 3DS flow. This sets up observers for authentication events.

```typescript
GatewaySpecific3DS.initialize({
  onTriggerCompletion: (event) => {
    console.log('3DS trigger completed:', event.token);
    // Proceed to call your backend /complete.json endpoint
  },

  onChallengeReady: (event) => {
    console.log('Challenge ready:', event.token);
    // Optional: Track challenge presentation
  },

  onResult: (result) => {
    if (result.status === 'success') {
      console.log('3DS authentication successful');
      // Handle successful authentication
    } else if (result.status === 'failed') {
      console.log('3DS authentication failed:', result.message);
      // Handle failure
    } else if (result.status === 'canceled') {
      console.log('User canceled 3DS authentication');
      // Handle cancellation
    }
  },
});
```

### Step 3: Start the 3DS Flow

After receiving a transaction token from your purchase API call:

```typescript
try {
  await GatewaySpecific3DS.startFlow(transactionToken);
  console.log('3DS flow started');
} catch (error) {
  console.error('Failed to start 3DS flow:', error);
}
```

The SDK automatically:

- Presents the challenge UI when required
- Handles user interaction
- Processes the authentication result
- Dismisses the UI after completion

### Step 4: Call Complete API

When `onTriggerCompletion` is triggered, call your backend endpoint to invoke Spreedly's `/complete.json` API:

```typescript
onTriggerCompletion: async (event) => {
  try {
    const response = await fetch('https://your-backend.com/complete', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ token: event.token }),
    });

    const data = await response.json();

    // Finalize the transaction with SDK
    await GatewaySpecific3DS.finalizeTransaction(event.token);
  } catch (error) {
    console.error('Complete API failed:', error);
  }
};
```

### Step 5: Handle Results

Process the final authentication result in the `onResult` callback:

```typescript
onResult: (result) => {
  switch (result.status) {
    case 'success':
      // Transaction authenticated successfully
      // Proceed with order fulfillment
      navigateToSuccess(result.transactionId);
      break;

    case 'failed':
      // Authentication failed
      // Show error to user
      showError(result.message);
      break;

    case 'canceled':
      // User canceled authentication
      // Return to payment screen
      navigateToPayment();
      break;
  }

  // Clean up resources
  GatewaySpecific3DS.cleanupTransaction(transactionToken);
};
```

## API Reference

### `initialize(eventListeners)`

Initializes the Gateway-Specific 3DS module and registers event listeners.

**Parameters:**

- `eventListeners` (object): Event callbacks
  - `onTriggerCompletion?: (event: GatewaySpecific3DSTriggerCompletionEvent) => void`
  - `onChallengeReady?: (event: GatewaySpecific3DSChallengeReadyEvent) => void`
  - `onResult?: (result: GatewaySpecific3DSResult) => void`

**Returns:** `void`

**Throws:** Error if native module is unavailable

---

### `startFlow(transactionToken)`

Starts the Gateway-Specific 3DS authentication flow.

**Parameters:**

- `transactionToken` (string): Transaction token from purchase API

**Returns:** `Promise<void>`

**Throws:** Error if not initialized or native module unavailable

---

### `finalizeTransaction(transactionToken)`

Finalizes the transaction after calling `/complete.json` API.

**Parameters:**

- `transactionToken` (string): Transaction token

**Returns:** `Promise<void>`

**Throws:** Error if native module is unavailable

---

### `cleanupTransaction(transactionToken)`

Cleans up resources for a specific transaction.

**Parameters:**

- `transactionToken` (string): Transaction token

**Returns:** `void`

---

### `cleanup()`

Removes all event listeners and cleans up native resources. Call when component unmounts.

**Returns:** `void`

---

## Event Types

### GatewaySpecific3DSTriggerCompletionEvent

```typescript
{
  token: string; // Transaction token
}
```

### GatewaySpecific3DSChallengeReadyEvent

```typescript
{
  token: string;              // Transaction token
  challengeUrl?: string;      // Challenge URL (if applicable)
  challengeFormEmbedUrl?: string;  // Embed URL (if applicable)
}
```

### GatewaySpecific3DSResult

Success:

```typescript
{
  status: 'success';
  transactionId?: string;  // Managed order token
}
```

Failed:

```typescript
{
  status: 'failed';
  message?: string;  // Error description
}
```

Canceled:

```typescript
{
  status: 'canceled';
}
```

## Complete Example

```typescript
import React, { useEffect, useState } from 'react';
import { View, Button, Alert } from 'react-native';
import { GatewaySpecific3DS, SpreedlyCore } from '@spreedly/checkout-react-native';

function PaymentScreen() {
  const [transactionToken, setTransactionToken] = useState<string | null>(null);

  useEffect(() => {
    // Initialize Gateway-Specific 3DS
    GatewaySpecific3DS.initialize({
      onTriggerCompletion: async (event) => {
        console.log('Trigger completion:', event.token);

        try {
          // Call your backend /complete endpoint
          const response = await fetch('https://api.example.com/complete', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ token: event.token }),
          });

          if (response.ok) {
            // Finalize with SDK
            await GatewaySpecific3DS.finalizeTransaction(event.token);
          } else {
            Alert.alert('Error', 'Failed to complete transaction');
          }
        } catch (error) {
          console.error('Complete API error:', error);
          Alert.alert('Error', 'Network error');
        }
      },

      onChallengeReady: (event) => {
        console.log('Challenge ready for:', event.token);
      },

      onResult: (result) => {
        if (result.status === 'success') {
          Alert.alert('Success', 'Payment authenticated successfully');
          // Navigate to success screen
        } else if (result.status === 'failed') {
          Alert.alert('Failed', result.message || 'Authentication failed');
        } else {
          Alert.alert('Canceled', 'Authentication was canceled');
        }

        // Clean up
        if (transactionToken) {
          GatewaySpecific3DS.cleanupTransaction(transactionToken);
        }
      },
    });

    // Cleanup on unmount
    return () => {
      GatewaySpecific3DS.cleanup();
    };
  }, []);

  const handlePayment = async () => {
    try {
      // Step 1: Create payment method
      const paymentMethod = await SpreedlyCore.createPaymentMethod({
        cardNumber: '4111111111111111',
        cvv: '123',
        expirationMonth: '12',
        expirationYear: '2025',
        fullName: 'John Doe',
      });

      // Step 2: Call your backend /purchase endpoint
      const purchaseResponse = await fetch('https://api.example.com/purchase', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Accept': 'application/json',
        },
        body: JSON.stringify({
          payment_method_token: PAYMENT_METHOD_TOKEN,
          amount: 3005,
          currency_code: 'USD',
          attempt_3dsecure: true,
        }),
      });

      const purchaseData = await purchaseResponse.json();

      // Step 3: Check if Gateway-Specific 3DS is required
      const requires3DS =
        purchaseData.transaction?.state === 'pending' ||
        purchaseData.transaction?.scaAuthentication?.requiredAction === 'device_fingerprint';

      if (requires3DS) {
        const txnToken = purchaseData.transaction.token;
        setTransactionToken(txnToken);

        // Step 4: Start Gateway-Specific 3DS flow
        await GatewaySpecific3DS.startFlow(txnToken);
        // Flow continues via callbacks (onTriggerCompletion → onResult)
      } else if (purchaseData.transaction?.succeeded) {
        // Payment completed without 3DS
        Alert.alert('Success', 'Payment completed!');
      } else {
        // Payment failed
        Alert.alert('Error', purchaseData.transaction?.message || 'Payment failed');
      }

    } catch (error) {
      console.error('Payment error:', error);
      Alert.alert('Error', 'Payment failed');
    }
  };

  return (
    <View>
      <Button title="Pay with 3DS" onPress={handlePayment} />
    </View>
  );
}

export default PaymentScreen;
```

## Error Handling

### Common Errors

**Module Not Initialized**

```typescript
Error: GatewaySpecific3DS is not initialized. Call initialize() first.
```

Solution: Call `initialize()` before `startFlow()`.

**Native Module Unavailable**

```typescript
Error: GatewaySpecific3DSModule is not available.
```

Solution: Verify SDK installation and native linking.

**No Activity Available** (Android)

```typescript
Error: No current activity available
```

Solution: Ensure flow is started from an active screen.

**SDK Not Initialized**

```typescript
Error: SDK not initialized. Call initSdk first.
```

Solution: Initialize `SpreedlyCore.initialize()` before using 3DS.

### Error Recovery

```typescript
try {
  await GatewaySpecific3DS.startFlow(transactionToken);
} catch (error) {
  console.error('3DS flow error:', error);

  // Clean up and reset state
  GatewaySpecific3DS.cleanupTransaction(transactionToken);

  // Show user-friendly message
  Alert.alert(
    'Authentication Error',
    'Unable to start authentication. Please try again.'
  );
}
```

## Lifecycle Management

### Component Mount

```typescript
useEffect(() => {
  GatewaySpecific3DS.initialize({
    /* listeners */
  });

  return () => {
    GatewaySpecific3DS.cleanup();
  };
}, []);
```

### Transaction Cleanup

Always clean up after each transaction completes:

```typescript
// After success, failure, or cancellation
GatewaySpecific3DS.cleanupTransaction(transactionToken);
```

### App Background/Foreground

The SDK handles app lifecycle automatically. If the app goes to background during authentication:

- Android: Challenge UI is preserved
- iOS: Challenge UI is restored on foreground

## Platform-Specific Notes

### Android

The challenge UI is rendered using Jetpack Compose and displayed as a bottom sheet. It requires:

- Activity must be a `FragmentActivity`
- Compose dependencies must be included
- No additional configuration needed

### iOS

The challenge UI uses SwiftUI and is presented using `UIHostingController`:

- Presented as full-screen modal
- Automatically handles safe areas
- Supports dark mode

## Testing

### Test Card Numbers

Use Spreedly's test environment with test card numbers:

```typescript
// Card requiring 3DS authentication
cardNumber: '4000000000001091';

// Card with frictionless 3DS
cardNumber: '4000000000001109';
```

### Test Mode

```typescript
SpreedlyCore.initialize({
  environmentKey: 'test_env_key',
  testMode: true,
});
```

## Troubleshooting

### Challenge UI Not Showing

**Issue:** `startFlow()` resolves but no UI appears

**Solutions:**

1. Verify `onTriggerCompletion` is being called
2. Check that gateway supports Gateway-Specific 3DS
3. Ensure transaction requires authentication
4. Check console for native errors

### Events Not Firing

**Issue:** Callbacks not being invoked

**Solutions:**

1. Call `initialize()` before `startFlow()`
2. Verify event listeners are registered
3. Check that native module is linked correctly

### Cleanup Issues

**Issue:** Resources not releasing properly

**Solutions:**

1. Call `cleanupTransaction()` after each transaction
2. Call `cleanup()` on component unmount
3. Don't reuse transaction tokens

## Best Practices

1. **Initialize Once:** Call `initialize()` once per component lifecycle
2. **Clean Up Always:** Call `cleanup()` on unmount to prevent memory leaks
3. **Error Boundaries:** Wrap 3DS flow in try-catch blocks
4. **User Feedback:** Show loading indicators during authentication
5. **Backend Validation:** Always verify transaction status on your backend
6. **Token Management:** Don't reuse transaction tokens across attempts
7. **Timeout Handling:** Implement timeouts for `/complete` API calls

## Security Considerations

- Never log sensitive payment data
- Use HTTPS for all API calls
- Validate transaction status server-side
- Implement rate limiting on backend endpoints
- Store tokens securely, never in plain text
- Clear sensitive data from memory after use

## Support

For issues or questions:

- SDK Documentation: [Spreedly Docs](https://docs.spreedly.com)
- API Reference: [Spreedly API](https://docs.spreedly.com/reference/api)
- Support: support@spreedly.com
