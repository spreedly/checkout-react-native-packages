# 3DS Challenge Integration Guide

**A step-by-step guide for integrating 3D Secure (3DS) authentication into your React Native app**

## What You'll Learn

This guide covers:

- Understanding 3D Secure and when to use it
- Setting up 3DS challenge flow in your React Native app
- Handling all challenge events (success, failed, canceled)
- Integrating with your backend for purchase/authorize flows
- Best practices for error handling and user experience

## Overview

3D Secure (3DS) is an authentication protocol that adds an extra layer of security for card-not-present transactions. It helps protect against fraudulent transactions by requiring the cardholder to verify their identity with their bank.

**When to Use 3DS:**

- **Regulatory Compliance** - SCA (Strong Customer Authentication) requirements in Europe (PSD2)
- **Fraud Reduction** - Shift liability for fraudulent transactions to the card issuer
- **Higher Authorization Rates** - Banks may approve more transactions with 3DS verification
- **Customer Trust** - Visual security indicators increase checkout confidence

**What the SDK does for you:**

- Displays secure, native 3DS challenge UI
- Handles all user interaction with the bank's authentication
- Communicates results back to your app via events
- Manages the full challenge lifecycle automatically

**What you need to do:**

- Process purchase/authorize on your backend via Spreedly API
- Retrieve `managedOrderToken` and `transactionToken` from the response
- Call `showThreeDSChallenge()` with those tokens
- Listen for challenge results and update your UI accordingly

## Minimal Integration

Here's the minimal code needed to integrate 3DS:

```typescript
import { useEffect, useState } from 'react';
import {
  SpreedlyCore,
  SpreedlyEventEmitter,
  SpreedlyEventTypes,
  type ThreeDSChallengeResult,
} from '@spreedly/react-native-checkout';

// 1. Set up event listener for 3DS results
useEffect(() => {
  const subscription = SpreedlyEventEmitter.addListener(
    SpreedlyEventTypes.THREE_DS_CHALLENGE_RESULT,
    (result: ThreeDSChallengeResult) => {
      if (result.status === 'success') {
        // Payment authenticated successfully!
        handlePaymentSuccess(result.transactionId);
      } else {
        // Challenge failed
        handlePaymentFailed(result.message);
      }
    }
  );

  return () => subscription.remove();
}, []);

// 2. Process purchase and trigger 3DS challenge
const handleCheckout = async () => {
  // Call your backend to process purchase via Spreedly API
  const { managedOrderToken, transactionToken } =
    await processBackendPurchase();

  // Show 3DS challenge UI
  SpreedlyCore.showThreeDSChallenge(managedOrderToken, transactionToken);
};
```

That's it! Read on for complete implementation details and best practices.

---

## How It Works - Visual Flow

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           Your React Native App                          │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    │ 1. User taps "Pay Now"
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                            Your Backend                                  │
│                                                                          │
│  • Receives payment request from app                                     │
│  • Calls Spreedly Purchase/Authorize API with 3DS enabled               │
│  • Returns managedOrderToken + transactionToken to app                  │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    │ 2. Tokens returned to app
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  SpreedlyCore.showThreeDSChallenge(                                     │
│    managedOrderToken,                                                    │
│    transactionToken                                                      │
│  )                                                                       │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    │ 3. SDK presents 3DS challenge
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    Native 3DS Challenge UI                               │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                                                                   │   │
│  │    🏦 Your Bank                                                  │   │
│  │                                                                   │   │
│  │    Please verify your identity to complete this purchase         │   │
│  │                                                                   │   │
│  │    Amount: $99.99                                                │   │
│  │    Merchant: Your Store                                          │   │
│  │                                                                   │   │
│  │    ┌─────────────────────────────────────────────┐              │   │
│  │    │  Enter verification code: [______]          │              │   │
│  │    └─────────────────────────────────────────────┘              │   │
│  │                                                                   │   │
│  │    [Cancel]                              [Verify]               │   │
│  │                                                                   │   │
│  └─────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ├─→ User cancels
                                    │   └→ Event: status = 'canceled' (if supported)
                                    │
                                    ├─→ Verification fails
                                    │   └→ Event: status = 'failed', message = '...'
                                    │
                                    └─→ Verification succeeds ✓
                                        │
                                        ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  Event: THREE_DS_CHALLENGE_RESULT                                       │
│         status = 'success'                                              │
│         transactionId = 'txn_abc123...'                                 │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
                              Done! ✓
                    Proceed with order fulfillment
```

### Responsibility Summary

| Component        | Responsibility                                                 |
| ---------------- | -------------------------------------------------------------- |
| **Your Backend** | Process purchase via Spreedly API, return tokens               |
| **Your App**     | Initiate purchase, trigger 3DS challenge, handle result        |
| **Spreedly SDK** | Display 3DS challenge UI, handle user interaction, emit result |

---

## Quick Start

Follow these steps to integrate 3DS into your React Native app:

### Step 1: Import Required Modules

```typescript
import React, { useState, useEffect } from 'react';
import { View, Button, Alert, Text, ActivityIndicator } from 'react-native';
import {
  SpreedlyCore,
  SpreedlyEventEmitter,
  SpreedlyEventTypes,
  type ThreeDSChallengeResult,
} from '@spreedly/react-native-checkout';
```

**Key imports:**

- `SpreedlyCore`: Main module to call 3DS methods
- `SpreedlyEventEmitter`: To listen for challenge result events
- `SpreedlyEventTypes`: Event type constants
- `ThreeDSChallengeResult`: TypeScript type for results

### Step 2: Set Up State Variables

```typescript
const [isProcessing, setIsProcessing] = useState(false);
const [paymentResult, setPaymentResult] = useState<string | null>(null);
const [errorMessage, setErrorMessage] = useState<string | null>(null);
```

**State variables:**

- `isProcessing`: To show loading indicator during payment
- `paymentResult`: Stores success message or transaction ID
- `errorMessage`: To display error messages to the user

### Step 3: Set Up Event Listener

Add this in your component to listen for 3DS challenge results:

```typescript
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
          notifyBackendSuccess(result.transactionId);
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

  // Clean up listener when component unmounts
  return () => {
    subscription.remove();
  };
}, []);
```

**Event handler breakdown:**

- `result.status` tells you what happened: 'success' or 'failed'
- `result.transactionId` is available on success
- `result.message` provides error details on failure
- Don't forget the cleanup function!

### Step 4: Create the Payment Function

```typescript
const handlePayment = async () => {
  setIsProcessing(true);
  setErrorMessage(null);
  setPaymentResult(null);

  try {
    // Step 1: Call your backend to process the purchase
    const response = await fetch('https://your-api.com/purchase', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        paymentMethodToken: selectedCard.token,
        amount: cartTotal,
        currency: 'USD',
      }),
    });

    const data = await response.json();

    // Step 2: Extract tokens from backend response
    const { managedOrderToken, transactionToken } = data;

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
```

**What this does:**

1. Calls your backend to initiate the purchase via Spreedly API
2. Extracts `managedOrderToken` and `transactionToken` from the response
3. Calls `showThreeDSChallenge()` to present the 3DS challenge UI

### Step 5: Add UI Components

```typescript
return (
  <View style={styles.container}>
    <Text style={styles.title}>Checkout</Text>

    <View style={styles.cartSummary}>
      <Text>Total: ${cartTotal.toFixed(2)}</Text>
    </View>

    <Button
      title={isProcessing ? 'Processing...' : 'Pay Now'}
      onPress={handlePayment}
      disabled={isProcessing}
    />

    {isProcessing && <ActivityIndicator size="large" />}

    {paymentResult && (
      <View style={styles.successContainer}>
        <Text style={styles.successText}> {paymentResult}</Text>
      </View>
    )}

    {errorMessage && (
      <View style={styles.errorContainer}>
        <Text style={styles.errorText}> {errorMessage}</Text>
        <Button title="Try Again" onPress={handlePayment} />
      </View>
    )}
  </View>
);
```

---

## API Reference

### `SpreedlyCore.showThreeDSChallenge(managedOrderToken, transactionToken)`

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

### `SpreedlyCore.hideThreeDSChallenge()`

Programmatically dismisses the 3DS challenge UI. Use this for edge cases where you need to close the challenge (e.g., session timeout).

**Example:**

```typescript
// Close challenge programmatically (use sparingly)
SpreedlyCore.hideThreeDSChallenge();
```

### `SpreedlyEventTypes.THREE_DS_CHALLENGE_RESULT`

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

---

## Backend Integration Requirements

Your backend must implement the purchase API endpoint that calls Spreedly's API and returns the required tokens:

```
Backend receives: payment_method_token, amount, currency
Backend returns: managed_order_token, transaction_token
```

**Backend Implementation Notes:**

1. **Use Spreedly's Purchase/Authorize API** with 3DS enabled
2. **Extract `managed_order_token`** from the API response
3. **Extract `transaction.token`** from the API response
4. **Return both tokens** to your mobile app
5. **Handle backend errors** gracefully and return appropriate error messages

**Example Backend Response:**

```json
{
  "success": true,
  "managedOrderToken": "mot_abc123xyz789",
  "transactionToken": "txn_def456uvw012"
}
```

---

## Event Handling

### Complete Event Handler Example

```typescript
useEffect(() => {
  const subscription = SpreedlyEventEmitter.addListener(
    SpreedlyEventTypes.THREE_DS_CHALLENGE_RESULT,
    (result: ThreeDSChallengeResult) => {
      switch (result.status) {
        case 'success':
          //  Success - proceed with order
          handlePaymentSuccess(result.transactionId);
          break;

        case 'failed':
          //  Failed - show error and offer retry
          handlePaymentFailed(result.message);
          break;
      }
    }
  );

  return () => subscription.remove();
}, []);
```

### Success Handler

```typescript
const handlePaymentSuccess = (transactionId?: string) => {
  setIsProcessing(false);
  setPaymentResult('Payment successful!');
  setErrorMessage(null);

  // Navigate to success screen or show confirmation
  Alert.alert('Payment Complete', 'Your order has been placed successfully!', [
    {
      text: 'View Order',
      onPress: () =>
        navigation.navigate('OrderConfirmation', { transactionId }),
    },
  ]);

  // Optionally notify your backend
  if (transactionId) {
    notifyBackendOfSuccess(transactionId);
  }
};
```

### Failure Handler

```typescript
const handlePaymentFailed = (message?: string) => {
  setIsProcessing(false);
  setPaymentResult(null);

  const errorMsg = message || '3DS verification failed. Please try again.';
  setErrorMessage(errorMsg);

  Alert.alert('Payment Failed', errorMsg, [
    { text: 'Try Again', onPress: handlePayment },
    { text: 'Use Different Card', onPress: () => setShowCardSelector(true) },
    { text: 'Cancel', style: 'cancel' },
  ]);
};
```

---

## Error Handling

### Common Error Scenarios

| Error Type            | Cause                                   | User Action                     |
| --------------------- | --------------------------------------- | ------------------------------- |
| Challenge timeout     | User took too long to complete          | Retry the payment               |
| Authentication failed | Wrong OTP/verification code             | Retry with correct code         |
| Bank declined         | Issuing bank rejected authentication    | Contact bank or use other card  |
| Network error         | Connection lost during challenge        | Check connection and retry      |
| Invalid tokens        | Backend returned invalid/expired tokens | Retry from beginning            |
| SDK not initialized   | `initSdk()` not called before challenge | Ensure SDK is initialized first |

### Error Handling Best Practices

#### 1. Always Show User-Friendly Messages

```typescript
case 'failed':
  //  Bad: Show technical error
  setErrorMessage(result.message);

  //  Good: Show friendly message with action
  let friendlyMessage = 'Unable to verify your payment. Please try again.';

  if (result.message?.includes('timeout')) {
    friendlyMessage = 'Verification timed out. Please try again.';
  } else if (result.message?.includes('canceled')) {
    friendlyMessage = 'Verification was cancelled.';
  } else if (result.message?.includes('network')) {
    friendlyMessage = 'Connection lost. Please check your internet and try again.';
  }

  setErrorMessage(friendlyMessage);
  break;
```

#### 2. Provide Clear Retry Options

```typescript
{
  errorMessage && (
    <View style={styles.errorContainer}>
      <Text style={styles.errorText}>{errorMessage}</Text>

      <View style={styles.buttonRow}>
        <Button title="Try Again" onPress={handlePayment} />
        <Button
          title="Use Different Card"
          onPress={() => navigation.navigate('SelectCard')}
        />
      </View>
    </View>
  );
}
```

#### 3. Log Errors for Debugging

```typescript
case 'failed':
  // Log technical details for debugging
  console.error('3DS Challenge Failed:', {
    message: result.message,
    timestamp: new Date().toISOString(),
    transactionAttempt: currentTransactionId,
  });

  // Show user-friendly message
  setErrorMessage('Payment verification failed. Please try again.');
  break;
```

#### 4. Handle Session Timeouts

```typescript
// Set a timeout to auto-hide challenge after extended period
const CHALLENGE_TIMEOUT = 5 * 60 * 1000; // 5 minutes

useEffect(() => {
  let timeoutId: NodeJS.Timeout;

  if (isProcessing) {
    timeoutId = setTimeout(() => {
      SpreedlyCore.hideThreeDSChallenge();
      setIsProcessing(false);
      setErrorMessage('Session timed out. Please try again.');
    }, CHALLENGE_TIMEOUT);
  }

  return () => clearTimeout(timeoutId);
}, [isProcessing]);
```

---

## Security Considerations

### Critical Security Rules

1. **Never expose Spreedly API credentials** in your mobile app
2. **Always process purchases** through your secure backend
3. **Validate tokens** on your backend before using them
4. **Implement session timeouts** for incomplete 3DS flows
5. **Log 3DS events** for compliance and debugging (without sensitive data)

### Secure Implementation Checklist

- [ ] API keys are only on your backend, never in the mobile app
- [ ] All purchase requests go through your backend
- [ ] Backend validates all incoming requests
- [ ] Tokens are validated before use
- [ ] Session timeouts are implemented
- [ ] Logging doesn't include sensitive card data
- [ ] HTTPS is used for all network requests

---

## Best Practices

### 1. Always Clean Up Event Listeners

```typescript
useEffect(() => {
  const subscription = SpreedlyEventEmitter.addListener(
    SpreedlyEventTypes.THREE_DS_CHALLENGE_RESULT,
    handleChallengeResult
  );

  // This cleanup function runs when component unmounts
  return () => subscription.remove();
}, []);
```

### 2. Handle All Event States

```typescript
const handleChallengeResult = (result: ThreeDSChallengeResult) => {
  switch (result.status) {
    case 'success':
      // Handle success
      handlePaymentSuccess(result.transactionId);
      break;

    case 'failed':
      // Handle failure
      handlePaymentFailed(result.message);
      break;

    default:
      // Handle unexpected status
      console.warn('Unexpected 3DS result status:', result);
      setErrorMessage('Unexpected error occurred');
      setIsProcessing(false);
  }
};
```

### 3. Show Loading State During Challenge

```typescript
const [isProcessing, setIsProcessing] = useState(false);

// Set loading when starting
const handlePayment = async () => {
  setIsProcessing(true);
  // ... process payment
};

// Clear loading when result received
useEffect(() => {
  const subscription = SpreedlyEventEmitter.addListener(
    SpreedlyEventTypes.THREE_DS_CHALLENGE_RESULT,
    (result) => {
      setIsProcessing(false); // Always clear loading
      // ... handle result
    }
  );
  return () => subscription.remove();
}, []);
```

### 4. Prevent Double Submissions

```typescript
const handlePayment = async () => {
  // Prevent double-tap
  if (isProcessing) return;

  setIsProcessing(true);
  // ... rest of payment logic
};
```

### 5. Provide Clear UI Feedback

```typescript
{
  isProcessing && (
    <View style={styles.processingOverlay}>
      <ActivityIndicator size="large" color="#007AFF" />
      <Text style={styles.processingText}>Verifying your payment...</Text>
      <Text style={styles.processingSubtext}>
        Please complete the verification in the popup
      </Text>
    </View>
  );
}
```

---

## Complete Example

Here's a complete example component integrating 3DS:

```typescript
import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  Button,
  Alert,
  ActivityIndicator,
  StyleSheet,
} from 'react-native';
import {
  SpreedlyCore,
  SpreedlyEventEmitter,
  SpreedlyEventTypes,
  type ThreeDSChallengeResult,
} from '@spreedly/react-native-checkout';

interface CheckoutScreenProps {
  cartTotal: number;
  paymentMethodToken: string;
  onSuccess: (transactionId: string) => void;
}

export function CheckoutScreen({
  cartTotal,
  paymentMethodToken,
  onSuccess,
}: CheckoutScreenProps) {
  const [isProcessing, setIsProcessing] = useState(false);
  const [errorMessage, setErrorMessage] = useState<string | null>(null);

  // Set up 3DS result listener
  useEffect(() => {
    const subscription = SpreedlyEventEmitter.addListener(
      SpreedlyEventTypes.THREE_DS_CHALLENGE_RESULT,
      (result: ThreeDSChallengeResult) => {
        setIsProcessing(false);

        switch (result.status) {
          case 'success':
            setErrorMessage(null);
            Alert.alert('Success', 'Payment completed successfully!');
            if (result.transactionId) {
              onSuccess(result.transactionId);
            }
            break;

          case 'failed':
            setErrorMessage(result.message || 'Payment verification failed');
            break;
        }
      }
    );

    return () => subscription.remove();
  }, [onSuccess]);

  const handlePayment = async () => {
    if (isProcessing) return;

    setIsProcessing(true);
    setErrorMessage(null);

    try {
      // Call your backend to process the purchase
      const response = await fetch('https://your-api.com/purchase', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          paymentMethodToken,
          amount: cartTotal,
          currency: 'USD',
        }),
      });

      if (!response.ok) {
        throw new Error('Failed to process payment');
      }

      const data = await response.json();
      const { managedOrderToken, transactionToken } = data;

      if (managedOrderToken && transactionToken) {
        // Show 3DS challenge
        SpreedlyCore.showThreeDSChallenge(managedOrderToken, transactionToken);
      } else {
        throw new Error('Missing tokens for 3DS verification');
      }
    } catch (error) {
      setIsProcessing(false);
      setErrorMessage(
        error instanceof Error ? error.message : 'Payment failed'
      );
    }
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Complete Your Purchase</Text>

      <View style={styles.summary}>
        <Text style={styles.totalLabel}>Total:</Text>
        <Text style={styles.totalAmount}>${cartTotal.toFixed(2)}</Text>
      </View>

      {errorMessage && (
        <View style={styles.errorContainer}>
          <Text style={styles.errorText}>{errorMessage}</Text>
        </View>
      )}

      <Button
        title={isProcessing ? 'Processing...' : `Pay $${cartTotal.toFixed(2)}`}
        onPress={handlePayment}
        disabled={isProcessing}
      />

      {isProcessing && (
        <View style={styles.loadingContainer}>
          <ActivityIndicator size="large" />
          <Text style={styles.loadingText}>
            Please complete verification...
          </Text>
        </View>
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 20,
  },
  summary: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    marginBottom: 20,
    padding: 15,
    backgroundColor: '#f5f5f5',
    borderRadius: 8,
  },
  totalLabel: {
    fontSize: 18,
  },
  totalAmount: {
    fontSize: 18,
    fontWeight: 'bold',
  },
  errorContainer: {
    backgroundColor: '#ffebee',
    padding: 15,
    borderRadius: 8,
    marginBottom: 20,
  },
  errorText: {
    color: '#c62828',
  },
  loadingContainer: {
    alignItems: 'center',
    marginTop: 20,
  },
  loadingText: {
    marginTop: 10,
    color: '#666',
  },
});
```

---

## Troubleshooting

### Common Issues

| Issue                         | Cause                          | Solution                                         |
| ----------------------------- | ------------------------------ | ------------------------------------------------ |
| Challenge not appearing       | SDK not initialized            | Call `SpreedlyCore.initSdk()` before showing 3DS |
| Events not firing             | Listener not set up            | Ensure `useEffect` runs before payment           |
| Challenge shows twice         | Duplicate calls                | Add `isProcessing` guard to prevent double calls |
| "No activity available" error | App backgrounded               | Keep app in foreground during checkout           |
| Tokens undefined              | Backend not returning properly | Check backend API response structure             |

## Summary

Integrating 3DS requires these key steps:

1. **Import modules** - Get `SpreedlyCore`, `SpreedlyEventEmitter`, and types
2. **Set up listener** - Use `useEffect` to add event listener before payment
3. **Call backend** - Process purchase and get tokens
4. **Show challenge** - Call `showThreeDSChallenge()` with tokens
5. **Handle results** - Switch on `result.status` to handle success/failure
6. **Clean up** - Remove listener when component unmounts

## Support & Resources

**Need Help?**

- 📖 [Spreedly API Documentation](https://docs.spreedly.com/)
- 💻 Review the example app in `example/src/screens/threeDsScreen/ThreeDsScreen.tsx`
- 🔍 Check console for error messages and warnings
- ⚙️ Ensure React Native version 0.77+ is installed

**Related Guides:**

- [Integration Guide](./integration_guide.md) - Full SDK integration
- [CVV Recaching Guide](./cvv_recaching_guide.md) - Update saved card CVV
- [Theme Guide](./theme_guide.md) - Customize SDK appearance
- [Security Guide](./security.md) - Security best practices

---

**Last Updated**: March 2026