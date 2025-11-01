# Spreedly React Native SDK - Examples

Comprehensive examples and implementation patterns for the Spreedly React Native Checkout SDK.

---

## Table of Contents

1. [Dynamic Field Configuration Examples](#dynamic-field-configuration-examples)
2. [Advanced Express Checkout Examples](#advanced-express-checkout-examples)
3. [Custom Hosted Fields Examples](#custom-hosted-fields-examples)
4. [Theme Customization Examples](#theme-customization-examples)
5. [Error Handling Examples](#error-handling-examples)
6. [Performance Optimization Examples](#performance-optimization-examples)
7. [Accessibility Examples](#accessibility-examples)

---

## ⚠️ Critical Security Requirements for All Examples

**IMPORTANT: Sensitive Payment Fields**

Before implementing any of the examples below, note that for card number (`FormFieldTypes.CARD`) and CVV (`FormFieldTypes.CVV`) fields, you **MUST** use `SPLTextField` components only. These fields cannot be implemented as custom fields or sent as additional fields during payment submission.

```typescript
// ✅ CORRECT - Use SPLTextField for sensitive payment data
<SPLTextField fieldType={FormFieldTypes.CARD} placeholder="Card Number" />
<SPLTextField fieldType={FormFieldTypes.CVV} placeholder="CVV" />

// ❌ INCORRECT - Never use custom fields for card number or CVV
<TextInput placeholder="Card Number" /> // This violates PCI compliance
<CustomField fieldType="card_number" /> // This is not allowed
```

**Why This Restriction Exists:**

- **PCI Compliance**: Card numbers and CVV codes must be handled in PCI-compliant environments
- **Security Isolation**: `SPLTextField` ensures this data never enters your application's memory space
- **Regulatory Requirements**: Payment industry regulations mandate secure handling of card data

---

## Dynamic Field Configuration Examples

Dynamic field configuration allows you to adapt your payment forms based on business logic, user preferences, or contextual requirements. This approach provides flexibility while maintaining security and validation.

**Real-World Use Cases:**

- **E-commerce**: Different fields for digital vs. physical products
- **Subscription Services**: Additional fields for recurring payments
- **B2B Payments**: Corporate billing information requirements
- **International Sales**: Country-specific field requirements
- **Guest vs. Registered Users**: Different information collection strategies

### Example 1: E-commerce Checkout Flow

```typescript
import React from 'react';
import { View } from 'react-native';
import {
  SPLTextField,
  FormFieldTypes,
  AdditionalFields,
  type FieldDescriptor
} from '@spreedly/react-native-checkout';

interface CheckoutConfig {
  productType: 'digital' | 'physical';
  userType: 'guest' | 'registered';
  requiresBilling: boolean;
  isInternational: boolean;
}

export function EcommerceCheckoutForm({ config }: { config: CheckoutConfig }) {
  const getRequiredFields = (): FieldDescriptor[] => {
    // Base payment fields (always required)
    const baseFields: FieldDescriptor[] = [
      { type: FormFieldTypes.CARD, required: true },
      { type: FormFieldTypes.EXPIRY_DATE, required: true },
      { type: FormFieldTypes.CVV, required: true },
    ];

    // Name handling based on user type
    if (config.userType === 'guest') {
      baseFields.push({ type: FormFieldTypes.NAME, required: true });
    }

    // Physical products require shipping
    if (config.productType === 'physical') {
      baseFields.push(
        { type: AdditionalFields.SHIPPING_ADDRESS_1, required: true },
        { type: AdditionalFields.SHIPPING_CITY, required: true },
        { type: AdditionalFields.SHIPPING_STATE, required: true },
        { type: AdditionalFields.SHIPPING_ZIP, required: true }
      );
    }

    // Billing address for certain scenarios
    if (config.requiresBilling || config.isInternational) {
      baseFields.push(
        { type: FormFieldTypes.ADDRESS_LINE_1, required: true },
        { type: FormFieldTypes.CITY, required: true },
        { type: FormFieldTypes.STATE, required: true },
        { type: FormFieldTypes.ZIP, required: true }
      );
    }

    // International orders need country
    if (config.isInternational) {
      baseFields.push({ type: AdditionalFields.COUNTRY, required: true });
    }

    return baseFields;
  };

  return (
    <View>
      {getRequiredFields().map((field) => (
        <SPLTextField
          key={field.type}
          fieldType={field.type}
          placeholder={getPlaceholderText(field.type)}
          style={getFieldStyle(field.type)}
        />
      ))}
    </View>
  );
}

// Helper functions
function getPlaceholderText(fieldType: string): string {
  const placeholders: Record<string, string> = {
    [FormFieldTypes.CARD]: 'Card Number',
    [FormFieldTypes.EXPIRY_DATE]: 'MM/YY',
    [FormFieldTypes.CVV]: 'CVV',
    [FormFieldTypes.NAME]: 'Cardholder Name',
    [FormFieldTypes.ADDRESS_LINE_1]: 'Address',
    [FormFieldTypes.CITY]: 'City',
    [FormFieldTypes.STATE]: 'State',
    [FormFieldTypes.ZIP]: 'ZIP Code',
    [AdditionalFields.SHIPPING_ADDRESS_1]: 'Shipping Address',
    [AdditionalFields.SHIPPING_CITY]: 'Shipping City',
    [AdditionalFields.SHIPPING_STATE]: 'Shipping State',
    [AdditionalFields.SHIPPING_ZIP]: 'Shipping ZIP',
    [AdditionalFields.COUNTRY]: 'Country',
  };
  return placeholders[fieldType] || fieldType;
}

function getFieldStyle(fieldType: string) {
  // Return appropriate styling based on field type
  return {
    height: 50,
    marginBottom: 15,
    borderWidth: 1,
    borderColor: '#E0E0E0',
    borderRadius: 8,
    paddingHorizontal: 15,
  };
}
```

### Example 2: Subscription Service Configuration

```typescript
import React, { useState } from 'react';
import { View, Switch, Text } from 'react-native';
import {
  SPLTextField,
  FormFieldTypes,
  AdditionalFields,
  type FieldDescriptor
} from '@spreedly/react-native-checkout';

interface SubscriptionTier {
  id: string;
  name: string;
  requiresBusinessInfo: boolean;
  requiresVatNumber: boolean;
  allowsTrialPeriod: boolean;
}

export function SubscriptionCheckoutForm({
  tier,
  isBusinessCustomer
}: {
  tier: SubscriptionTier;
  isBusinessCustomer: boolean;
}) {
  const [useTrialPeriod, setUseTrialPeriod] = useState(false);

  const getSubscriptionFields = (): FieldDescriptor[] => {
    const fields: FieldDescriptor[] = [
      { type: FormFieldTypes.CARD, required: true },
      { type: FormFieldTypes.EXPIRY_DATE, required: true },
      { type: FormFieldTypes.CVV, required: true },
      { type: FormFieldTypes.NAME, required: true },
      { type: AdditionalFields.EMAIL, required: true },
    ];

    // Business customers need additional information
    if (isBusinessCustomer && tier.requiresBusinessInfo) {
      fields.push(
        { type: AdditionalFields.COMPANY_NAME, required: true },
        { type: FormFieldTypes.ADDRESS_LINE_1, required: true },
        { type: FormFieldTypes.CITY, required: true },
        { type: FormFieldTypes.STATE, required: true },
        { type: FormFieldTypes.ZIP, required: true }
      );
    }

    // VAT number for EU business customers
    if (tier.requiresVatNumber && isBusinessCustomer) {
      fields.push({ type: AdditionalFields.VAT_NUMBER, required: true });
    }

    // Trial periods don't require immediate payment
    if (useTrialPeriod && tier.allowsTrialPeriod) {
      // Remove payment fields, keep identity fields
      return fields.filter(field =>
        ![FormFieldTypes.CARD, FormFieldTypes.EXPIRY_DATE, FormFieldTypes.CVV]
          .includes(field.type as FormFieldTypes)
      );
    }

    return fields;
  };

  return (
    <View>
      {tier.allowsTrialPeriod && (
        <View style={styles.trialOption}>
          <Switch
            value={useTrialPeriod}
            onValueChange={setUseTrialPeriod}
          />
          <Text>Start with free trial</Text>
        </View>
      )}

      {getSubscriptionFields().map((field) => (
        <SPLTextField
          key={field.type}
          fieldType={field.type}
          placeholder={getLocalizedPlaceholder(field.type)}
        />
      ))}
    </View>
  );
}

function getLocalizedPlaceholder(fieldType: string): string {
  // Return localized placeholder text based on field type
  // This could be integrated with i18n libraries
  return fieldType;
}

const styles = {
  trialOption: {
    flexDirection: 'row' as const,
    alignItems: 'center' as const,
    marginBottom: 20,
  },
};
```

### Example 3: Progressive Field Disclosure

```typescript
import React, { useState, useEffect } from 'react';
import { ScrollView, View, Text, Button } from 'react-native';
import {
  SPLTextField,
  FormFieldTypes,
  type FieldDescriptor
} from '@spreedly/react-native-checkout';

export function ProgressiveCheckoutForm() {
  const [completedSteps, setCompletedSteps] = useState<Set<string>>(new Set());
  const [fieldValidation, setFieldValidation] = useState<Record<string, boolean>>({});

  const isStepComplete = (stepFields: string[]) => {
    return stepFields.every(field => fieldValidation[field] === true);
  };

  const shouldShowStep = (stepId: string, dependencies: string[] = []) => {
    return dependencies.every(dep => completedSteps.has(dep));
  };

  useEffect(() => {
    // Update completed steps based on validation
    const newCompletedSteps = new Set<string>();

    if (isStepComplete([FormFieldTypes.CARD])) {
      newCompletedSteps.add('card');
    }

    if (isStepComplete([FormFieldTypes.EXPIRY_DATE, FormFieldTypes.CVV])) {
      newCompletedSteps.add('security');
    }

    if (isStepComplete([FormFieldTypes.NAME])) {
      newCompletedSteps.add('identity');
    }

    setCompletedSteps(newCompletedSteps);
  }, [fieldValidation]);

  const handleSubmit = () => {
    // Handle form submission
    console.log('Form submitted');
  };

  return (
    <ScrollView style={styles.container}>
      {/* Step 1: Card Number (Always visible) */}
      <View style={styles.step}>
        <Text style={styles.stepTitle}>Card Information</Text>
        <SPLTextField
          fieldType={FormFieldTypes.CARD}
          placeholder="Card Number"
          onValidationChange={(isValid) =>
            setFieldValidation(prev => ({ ...prev, [FormFieldTypes.CARD]: isValid }))
          }
        />
      </View>

      {/* Step 2: Security Info (Show after card is valid) */}
      {shouldShowStep('security', ['card']) && (
        <View style={styles.step}>
          <Text style={styles.stepTitle}>Security Details</Text>
          <View style={styles.row}>
            <SPLTextField
              fieldType={FormFieldTypes.EXPIRY_DATE}
              placeholder="MM/YY"
              style={styles.halfField}
              onValidationChange={(isValid) =>
                setFieldValidation(prev => ({ ...prev, [FormFieldTypes.EXPIRY_DATE]: isValid }))
              }
            />
            <SPLTextField
              fieldType={FormFieldTypes.CVV}
              placeholder="CVV"
              style={styles.halfField}
              onValidationChange={(isValid) =>
                setFieldValidation(prev => ({ ...prev, [FormFieldTypes.CVV]: isValid }))
              }
            />
          </View>
        </View>
      )}

      {/* Step 3: Cardholder Info (Show after security is complete) */}
      {shouldShowStep('identity', ['security']) && (
        <View style={styles.step}>
          <Text style={styles.stepTitle}>Cardholder Information</Text>
          <SPLTextField
            fieldType={FormFieldTypes.NAME}
            placeholder="Cardholder Name"
            onValidationChange={(isValid) =>
              setFieldValidation(prev => ({ ...prev, [FormFieldTypes.NAME]: isValid }))
            }
          />
        </View>
      )}

      {/* Submit Button (Show when all steps complete) */}
      {shouldShowStep('submit', ['card', 'security', 'identity']) && (
        <Button
          title="Complete Payment"
          onPress={handleSubmit}
        />
      )}
    </ScrollView>
  );
}

const styles = {
  container: {
    padding: 20,
  },
  step: {
    marginBottom: 30,
  },
  stepTitle: {
    fontSize: 18,
    fontWeight: 'bold' as const,
    marginBottom: 15,
  },
  row: {
    flexDirection: 'row' as const,
    justifyContent: 'space-between' as const,
  },
  halfField: {
    width: '48%',
  },
};
```

### Example 4: Multi-Region Configuration

```typescript
import React from 'react';
import { View, Text } from 'react-native';
import {
  SPLTextField,
  FormFieldTypes,
  AdditionalFields,
  type FieldDescriptor
} from '@spreedly/react-native-checkout';

interface RegionConfig {
  country: string;
  currency: string;
  requiresVAT: boolean;
  postalCodeFormat: 'zip' | 'postal';
  addressFormat: 'us' | 'international';
}

export function MultiRegionCheckoutForm({ region }: { region: RegionConfig }) {
  const getRegionSpecificFields = (): FieldDescriptor[] => {
    const baseFields: FieldDescriptor[] = [
      { type: FormFieldTypes.CARD, required: true },
      { type: FormFieldTypes.EXPIRY_DATE, required: true },
      { type: FormFieldTypes.CVV, required: true },
      { type: FormFieldTypes.NAME, required: true },
    ];

    // Address fields based on region
    if (region.addressFormat === 'us') {
      baseFields.push(
        { type: FormFieldTypes.ADDRESS_LINE_1, required: true },
        { type: FormFieldTypes.CITY, required: true },
        { type: FormFieldTypes.STATE, required: true },
        { type: FormFieldTypes.ZIP, required: true }
      );
    } else {
      baseFields.push(
        { type: FormFieldTypes.ADDRESS_LINE_1, required: true },
        { type: FormFieldTypes.CITY, required: true },
        { type: AdditionalFields.POSTAL_CODE, required: true },
        { type: AdditionalFields.COUNTRY, required: true }
      );
    }

    // VAT number for applicable regions
    if (region.requiresVAT) {
      baseFields.push({ type: AdditionalFields.VAT_NUMBER, required: false });
    }

    return baseFields;
  };

  return (
    <View>
      <Text style={styles.regionHeader}>
        Checkout for {region.country} ({region.currency})
      </Text>

      {getRegionSpecificFields().map((field) => (
        <SPLTextField
          key={field.type}
          fieldType={field.type}
          placeholder={getRegionalizedPlaceholder(field.type, region)}
          style={getRegionalFieldStyle(field.type, region)}
        />
      ))}
    </View>
  );
}

function getRegionalizedPlaceholder(fieldType: string, region: RegionConfig): string {
  const basePlaceholders: Record<string, string> = {
    [FormFieldTypes.CARD]: 'Card Number',
    [FormFieldTypes.EXPIRY_DATE]: 'MM/YY',
    [FormFieldTypes.CVV]: 'CVV',
    [FormFieldTypes.NAME]: 'Cardholder Name',
    [FormFieldTypes.ADDRESS_LINE_1]: 'Address',
    [FormFieldTypes.CITY]: 'City',
    [FormFieldTypes.STATE]: region.addressFormat === 'us' ? 'State' : 'Province/Region',
    [FormFieldTypes.ZIP]: 'ZIP Code',
    [AdditionalFields.POSTAL_CODE]: 'Postal Code',
    [AdditionalFields.COUNTRY]: 'Country',
    [AdditionalFields.VAT_NUMBER]: 'VAT Number (Optional)',
  };

  return basePlaceholders[fieldType] || fieldType;
}

function getRegionalFieldStyle(fieldType: string, region: RegionConfig) {
  // Return region-specific styling
  return {
    height: 50,
    marginBottom: 15,
    borderWidth: 1,
    borderColor: '#E0E0E0',
    borderRadius: 8,
    paddingHorizontal: 15,
  };
}

const styles = {
  regionHeader: {
    fontSize: 20,
    fontWeight: 'bold' as const,
    marginBottom: 20,
    textAlign: 'center' as const,
  },
};
```

---

## Advanced Express Checkout Examples

### Environment-Specific Configuration

```typescript
import React from 'react';
import { Button } from 'react-native';
import { SpreedlyCore, type BaseThemeConfig } from '@spreedly/react-native-checkout';

export function EnvironmentAwareExpressCheckout() {
  const getEnvironmentConfig = () => {
    const isProduction = process.env.NODE_ENV === 'production';
    const isTestEnvironment = process.env.SPREEDLY_ENVIRONMENT_KEY?.includes('test');

    if (isProduction && !isTestEnvironment) {
      // Strict validation for production
      return {
        allowBlankName: false,
        allowExpiredDate: false,
        yearFormat: '4' as const,
        nameDisplayMode: 'singleField' as const,
      };
    } else {
      // More lenient for development/testing
      return {
        allowBlankName: true,
        allowExpiredDate: true,
        yearFormat: '4' as const,
        nameDisplayMode: 'singleField' as const,
      };
    }
  };

  const handlePaymentPress = () => {
    SpreedlyCore.paymentBottomSheet(getEnvironmentConfig());
  };

  return (
    <Button title="Start Environment-Aware Payment" onPress={handlePaymentPress} />
  );
}
```

### Business Logic-Based Configuration

```typescript
import React from 'react';
import { Button } from 'react-native';
import { SpreedlyCore } from '@spreedly/react-native-checkout';

type CheckoutType = 'guest' | 'registered' | 'subscription';

export function BusinessLogicExpressCheckout({ checkoutType }: { checkoutType: CheckoutType }) {
  const configureForCheckoutType = (type: CheckoutType) => {
    switch (type) {
      case 'guest':
        return {
          allowBlankName: true, // Allow blank names for guest checkout
          allowExpiredDate: false,
          yearFormat: '4' as const,
          nameDisplayMode: 'singleField' as const,
        };

      case 'registered':
        return {
          allowBlankName: false, // Require complete information for registered users
          allowExpiredDate: false,
          yearFormat: '4' as const,
          nameDisplayMode: 'singleField' as const,
        };

      case 'subscription':
        return {
          allowBlankName: false, // Strict validation for recurring payments
          allowExpiredDate: false,
          yearFormat: '4' as const,
          nameDisplayMode: 'singleField' as const,
        };
    }
  };

  const handlePaymentPress = () => {
    SpreedlyCore.paymentBottomSheet(configureForCheckoutType(checkoutType));
  };

  return (
    <Button
      title={`Start ${checkoutType.charAt(0).toUpperCase() + checkoutType.slice(1)} Payment`}
      onPress={handlePaymentPress}
    />
  );
}
```

---

## Theme Customization Examples

### Complete Theme System

```typescript
import { type BaseThemeConfig } from '@spreedly/react-native-checkout';

// Light theme
export const lightTheme: BaseThemeConfig = {
  primaryColor: '#007AFF',
  secondaryColor: '#6B7280',
  formBorderColor: '#D1D5DB',
  formBackgroundColor: '#FFFFFF',
  fieldBackgroundColor: '#F9FAFB',
  fieldLabelColor: '#374151',
  borderRadius: 12,
  fieldShape: 'rounded',
};

// Dark theme
export const darkTheme: BaseThemeConfig = {
  primaryColor: '#0A84FF',
  secondaryColor: '#8E8E93',
  formBorderColor: '#38383A',
  formBackgroundColor: '#1C1C1E',
  fieldBackgroundColor: '#2C2C2E',
  fieldLabelColor: '#FFFFFF',
  borderRadius: 12,
  fieldShape: 'rounded',
};

// Brand theme
export const brandTheme: BaseThemeConfig = {
  primaryColor: '#6366F1', // Indigo
  secondaryColor: '#8B5CF6', // Purple
  formBorderColor: '#C7D2FE',
  formBackgroundColor: '#FFFFFF',
  fieldBackgroundColor: '#FAFAFF',
  fieldLabelColor: '#4338CA',
  borderRadius: 16,
  fieldShape: 'rounded',
};

// E-commerce theme
export const ecommerceTheme: BaseThemeConfig = {
  primaryColor: '#059669', // Emerald
  secondaryColor: '#6B7280',
  formBorderColor: '#D1FAE5',
  formBackgroundColor: '#FFFFFF',
  fieldBackgroundColor: '#F0FDF4',
  fieldLabelColor: '#065F46',
  borderRadius: 8,
  fieldShape: 'rounded',
};
```

### Dynamic Theme Selection

```typescript
import React, { useState, useEffect } from 'react';
import { useColorScheme } from 'react-native';
import { SpreedlyCore, type BaseThemeConfig } from '@spreedly/react-native-checkout';
import { lightTheme, darkTheme, brandTheme } from './themes';

export function DynamicThemedCheckout() {
  const systemColorScheme = useColorScheme();
  const [selectedTheme, setSelectedTheme] = useState<BaseThemeConfig>(lightTheme);

  useEffect(() => {
    // Auto-switch theme based on system preference
    const theme = systemColorScheme === 'dark' ? darkTheme : lightTheme;
    setSelectedTheme(theme);
    SpreedlyCore.setGlobalTheme(theme);
  }, [systemColorScheme]);

  const handleBrandedPayment = () => {
    SpreedlyCore.paymentBottomSheet({
      allowBlankName: false,
      allowExpiredDate: false,
      yearFormat: '4',
      nameDisplayMode: 'singleField',
      config: brandTheme, // Override global theme for this specific payment
    });
  };

  return (
    <Button title="Start Branded Payment" onPress={handleBrandedPayment} />
  );
}
```

---

## Error Handling Examples

### Comprehensive Error Handler

```typescript
import {
  type PaymentErrorType,
  type PaymentValidationError,
  type PaymentResultRN,
  mapPaymentResult,
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

  static handlePaymentResult(result: PaymentResultRN): {
    success: boolean;
    message: string;
    token?: string;
  } {
    const mapped = mapPaymentResult(result);

    switch (mapped.kind) {
      case 'success':
        return {
          success: true,
          message: 'Payment completed successfully!',
          token: mapped.token,
        };

      case 'validation':
        return {
          success: false,
          message: `Please correct: ${mapped.invalidFields?.join(', ') || 'Invalid fields'}`,
        };

      case 'failed':
        return {
          success: false,
          message: this.handleTypedError(mapped.errorType, mapped.message),
        };

      case 'canceled':
        return {
          success: false,
          message: 'Payment was canceled',
        };

      default:
        return {
          success: false,
          message: 'Payment processing failed',
        };
    }
  }
}
```

### Retry Logic with Exponential Backoff

```typescript
import React, { useState } from 'react';
import {
  SpreedlyCore,
  type FieldDescriptor,
  type PaymentResultRN,
} from '@spreedly/react-native-checkout';

export function usePaymentWithRetry() {
  const [retryCount, setRetryCount] = useState(0);
  const [isRetrying, setIsRetrying] = useState(false);
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
        setIsRetrying(true);
        setRetryCount((prev) => prev + 1);

        // Wait before retry (exponential backoff)
        const delay = Math.pow(2, retryCount) * 1000;
        await new Promise((resolve) => setTimeout(resolve, delay));

        setIsRetrying(false);
        return submitPayment(fields, options);
      }

      setRetryCount(0); // Reset on success or non-retryable error
      setIsRetrying(false);
      return result;
    } catch (error) {
      if (retryCount < maxRetries) {
        setIsRetrying(true);
        setRetryCount((prev) => prev + 1);

        const delay = Math.pow(2, retryCount) * 1000;
        await new Promise((resolve) => setTimeout(resolve, delay));

        setIsRetrying(false);
        return submitPayment(fields, options);
      }

      setIsRetrying(false);
      throw error;
    }
  };

  return {
    submitPayment,
    retryCount,
    isRetrying,
    canRetry: retryCount < maxRetries,
  };
}
```

---

## Performance Optimization Examples

### Debounced Validation

```typescript
import React, { useState, useCallback } from 'react';
import { useDebouncedCallback } from 'use-debounce';
import { SPLTextField, FormFieldTypes } from '@spreedly/react-native-checkout';

export function OptimizedTextField({ fieldType, ...props }) {
  const [validationState, setValidationState] = useState<'idle' | 'validating' | 'valid' | 'invalid'>('idle');

  const debouncedValidation = useDebouncedCallback(
    (isValid: boolean) => {
      setValidationState(isValid ? 'valid' : 'invalid');
      props.onValidationChange?.(isValid);
    },
    300 // 300ms delay
  );

  const handleValidationChange = useCallback((isValid: boolean) => {
    setValidationState('validating');
    debouncedValidation(isValid);
  }, [debouncedValidation]);

  return (
    <SPLTextField
      {...props}
      fieldType={fieldType}
      onValidationChange={handleValidationChange}
      style={[
        props.style,
        getValidationStyle(validationState)
      ]}
    />
  );
}

function getValidationStyle(state: string) {
  switch (state) {
    case 'validating':
      return { borderColor: '#FCD34D' }; // Yellow for validating
    case 'valid':
      return { borderColor: '#10B981' }; // Green for valid
    case 'invalid':
      return { borderColor: '#EF4444' }; // Red for invalid
    default:
      return { borderColor: '#D1D5DB' }; // Gray for idle
  }
}
```

### Memoized Components

```typescript
import React, { memo } from 'react';
import { SPLTextField, FormFieldTypes } from '@spreedly/react-native-checkout';

interface MemoizedTextFieldProps {
  fieldType: string;
  placeholder: string;
  style?: any;
  onValidationChange?: (isValid: boolean) => void;
}

export const MemoizedTextField = memo<MemoizedTextFieldProps>(({
  fieldType,
  placeholder,
  style,
  onValidationChange
}) => {
  return (
    <SPLTextField
      fieldType={fieldType}
      placeholder={placeholder}
      style={style}
      onValidationChange={onValidationChange}
    />
  );
}, (prevProps, nextProps) => {
  // Custom comparison function
  return (
    prevProps.fieldType === nextProps.fieldType &&
    prevProps.placeholder === nextProps.placeholder &&
    JSON.stringify(prevProps.style) === JSON.stringify(nextProps.style)
  );
});

MemoizedTextField.displayName = 'MemoizedTextField';
```

### Performance Monitoring Hook

```typescript
import { useEffect, useRef } from 'react';

export function usePerformanceMonitoring(componentName: string) {
  const startTime = useRef<number>(Date.now());
  const renderCount = useRef<number>(0);

  useEffect(() => {
    renderCount.current += 1;
  });

  useEffect(() => {
    const mountTime = Date.now() - startTime.current;
    console.log(`${componentName} mounted in ${mountTime}ms`);

    return () => {
      const totalTime = Date.now() - startTime.current;
      console.log(`${componentName} lifecycle: ${totalTime}ms, renders: ${renderCount.current}`);
    };
  }, [componentName]);

  return {
    renderCount: renderCount.current,
    elapsedTime: Date.now() - startTime.current,
  };
}

// Usage
export function MonitoredPaymentForm() {
  const { renderCount, elapsedTime } = usePerformanceMonitoring('PaymentForm');

  return (
    <View>
      {__DEV__ && (
        <Text>Renders: {renderCount}, Time: {elapsedTime}ms</Text>
      )}
      {/* Your payment form components */}
    </View>
  );
}
```

---

## Accessibility Examples

### Comprehensive Accessibility Implementation

```typescript
import React from 'react';
import { View, Text } from 'react-native';
import { SPLTextField, FormFieldTypes } from '@spreedly/react-native-checkout';

export function AccessiblePaymentForm() {
  return (
    <View accessibilityRole="form" accessibilityLabel="Payment information form">
      <Text style={styles.sectionHeader} accessibilityRole="header">
        Payment Details
      </Text>

      <SPLTextField
        fieldType={FormFieldTypes.CARD}
        placeholder="Card Number"
        accessibilityLabel="Credit card number"
        accessibilityHint="Enter your 16-digit credit card number"
        accessibilityRequired={true}
        testID="card-number-field"
        style={styles.field}
      />

      <View style={styles.row}>
        <SPLTextField
          fieldType={FormFieldTypes.EXPIRY_DATE}
          placeholder="MM/YY"
          accessibilityLabel="Expiration date"
          accessibilityHint="Enter card expiration date in MM/YY format"
          accessibilityRequired={true}
          testID="expiry-date-field"
          style={[styles.field, styles.halfField]}
        />

        <SPLTextField
          fieldType={FormFieldTypes.CVV}
          placeholder="CVV"
          accessibilityLabel="Security code"
          accessibilityHint="Enter the 3 or 4 digit security code on your card"
          accessibilityRequired={true}
          testID="cvv-field"
          style={[styles.field, styles.halfField]}
        />
      </View>

      <SPLTextField
        fieldType={FormFieldTypes.NAME}
        placeholder="Cardholder Name"
        accessibilityLabel="Cardholder name"
        accessibilityHint="Enter the name as it appears on your card"
        accessibilityRequired={true}
        testID="cardholder-name-field"
        style={styles.field}
      />
    </View>
  );
}

const styles = {
  sectionHeader: {
    fontSize: 20,
    fontWeight: 'bold' as const,
    marginBottom: 16,
    color: '#1F2937',
  },
  field: {
    height: 56,
    marginBottom: 16,
    borderWidth: 2,
    borderColor: '#E5E7EB',
    borderRadius: 8,
    paddingHorizontal: 16,
    fontSize: 16,
  },
  row: {
    flexDirection: 'row' as const,
    justifyContent: 'space-between' as const,
  },
  halfField: {
    width: '48%',
  },
};
```

### Screen Reader Announcements

```typescript
import React, { useState } from 'react';
import { View, Text, AccessibilityInfo } from 'react-native';
import { SPLTextField, FormFieldTypes } from '@spreedly/react-native-checkout';

export function AccessibleFormWithAnnouncements() {
  const [validationMessages, setValidationMessages] = useState<Record<string, string>>({});

  const handleValidationChange = (fieldType: string) => (isValid: boolean) => {
    const message = isValid
      ? `${getFieldName(fieldType)} is valid`
      : `${getFieldName(fieldType)} needs attention`;

    setValidationMessages(prev => ({ ...prev, [fieldType]: message }));

    // Announce to screen readers
    AccessibilityInfo.announceForAccessibility(message);
  };

  return (
    <View>
      <SPLTextField
        fieldType={FormFieldTypes.CARD}
        placeholder="Card Number"
        accessibilityLabel="Credit card number"
        onValidationChange={handleValidationChange(FormFieldTypes.CARD)}
      />

      {validationMessages[FormFieldTypes.CARD] && (
        <Text
          accessibilityRole="alert"
          accessibilityLiveRegion="polite"
          style={styles.validationMessage}
        >
          {validationMessages[FormFieldTypes.CARD]}
        </Text>
      )}
    </View>
  );
}

function getFieldName(fieldType: string): string {
  const names: Record<string, string> = {
    [FormFieldTypes.CARD]: 'Card number',
    [FormFieldTypes.EXPIRY_DATE]: 'Expiration date',
    [FormFieldTypes.CVV]: 'Security code',
    [FormFieldTypes.NAME]: 'Cardholder name',
  };
  return names[fieldType] || fieldType;
}

const styles = {
  validationMessage: {
    fontSize: 14,
    color: '#6B7280',
    marginTop: 4,
    marginBottom: 8,
  },
};
```

---

This examples document provides comprehensive implementation patterns that can be referenced from the main Integration Guide. Each example includes detailed code with proper TypeScript typing, error handling, and best practices for real-world usage.
