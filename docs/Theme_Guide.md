# Spreedly React Native SDK - Theme Customization Guide

Complete guide for customizing the visual appearance of Spreedly payment components with light and dark mode support.

---

## Table of Contents

1. [Theme Structure](#theme-structure)
2. [Global Theme](#global-theme)
3. [Component Theming](#component-theming)
4. [Dark Mode](#dark-mode)
5. [Pre-Built Themes](#pre-built-themes)
6. [Nullability Reference](#nullability-reference)
7. [Troubleshooting](#troubleshooting)

---

## Theme Structure

### BaseThemeConfig Interface

```typescript
interface BaseThemeConfig {
  primaryColor: string; // Buttons, focus states, accents (e.g., '#6366F1')
  secondaryColor: string; // Subtle highlights, secondary elements
  formBorderColor: string; // Form container borders
  formBackgroundColor: string; // Form container background
  fieldBackgroundColor: string; // Individual field backgrounds
  fieldLabelColor: string; // Labels and placeholder text
  borderRadius: number; // Corner radius in pixels (e.g., 8, 12, 16)
  fieldShape: string; // 'rounded' or 'square'
}
```

---

## Global Theme

### setGlobalTheme Method

Applies theme to all SDK components.

```typescript
SpreedlyCore.setGlobalTheme(options: GlobalThemeOptions | BaseThemeConfig): void
```

### GlobalThemeOptions

```typescript
interface GlobalThemeOptions {
  theme?: BaseThemeConfig; // Light mode theme (optional)
  darkTheme?: BaseThemeConfig; // Dark mode theme (optional)
}
```

**Rules:**

- At least one of `theme` or `darkTheme` must be provided
- If only `theme`: Used for both light and dark modes
- If only `darkTheme`: Used for both modes
- If both: Automatically switches based on system appearance

### Basic Usage

**Single theme for both modes:**

```typescript
import { SpreedlyCore } from '@spreedly/react-native-checkout';

const myTheme = {
  primaryColor: '#6366F1',
  secondaryColor: '#8B5CF6',
  formBorderColor: '#D1D5DB',
  formBackgroundColor: '#FFFFFF',
  fieldBackgroundColor: '#F9FAFB',
  fieldLabelColor: '#6B7280',
  borderRadius: 12,
  fieldShape: 'rounded',
};

SpreedlyCore.setGlobalTheme(myTheme);
```

**Separate light and dark themes:**

```typescript
SpreedlyCore.setGlobalTheme({
  theme: {
    primaryColor: '#0077C8',
    secondaryColor: '#6B7280',
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

---

## Component Theming

### SPLTextField

Override global theme for individual fields.

```typescript
<SPLTextField
  formFieldType={FormFieldTypes.CARD}
  label="Card Number"
  theme={lightTheme}      // Optional - light mode
  darkTheme={darkTheme}   // Optional - dark mode
/>
```

**Nullability:**

- `theme`: Optional - uses global theme if not provided
- `darkTheme`: Optional - uses `theme` for both modes if not provided

### PaymentBottomSheet (Method)

```typescript
SpreedlyCore.paymentBottomSheet({
  theme: lightTheme, // Optional
  darkTheme: darkTheme, // Optional
  // ... other options
});
```

### PaymentBottomSheet (Component)

```typescript
<PaymentBottomSheet
  theme={lightTheme}      // Optional
  darkTheme={darkTheme}   // Optional
  // ... other props
/>
```

---

## Dark Mode

### Automatic Detection

The SDK automatically detects and responds to system appearance:

- **iOS**: Uses `UIUserInterfaceStyle`
- **Android**: Uses `Configuration.UI_MODE_NIGHT_MASK`

### Creating Dark Themes

**Design principles:**

- Invert luminosity (dark backgrounds, light text)
- Use lighter accent colors for visibility
- Maintain proper contrast

**Example:**

```typescript
// Light mode
const lightTheme = {
  primaryColor: '#0077C8', // Standard blue
  formBackgroundColor: '#FFFFFF',
  fieldBackgroundColor: '#FFFFFF',
  fieldLabelColor: '#6B7280',
  // ...
};

// Dark mode (adapted)
const darkTheme = {
  primaryColor: '#60A5FA', // Lighter blue (more visible on dark)
  formBackgroundColor: '#1F2937', // Dark gray
  fieldBackgroundColor: '#111827', // Darker gray
  fieldLabelColor: '#9CA3AF', // Light gray text
  // ...
};
```

---

## Pre-Built Themes

### Default Theme

```typescript
export const DefaultThemeConfig = {
  primaryColor: '#0077C8',
  secondaryColor: '#AFB4B5',
  formBorderColor: '#D9D9D9',
  formBackgroundColor: '#FFFFFF',
  fieldBackgroundColor: '#FFFFFF',
  fieldLabelColor: '#AFB4B5',
  borderRadius: 8,
  fieldShape: 'rounded',
};

export const DarkThemeConfig = {
  primaryColor: '#00A0FF',
  secondaryColor: '#6C757D',
  formBorderColor: '#3A3A3C',
  formBackgroundColor: '#1F2937',
  fieldBackgroundColor: '#111827',
  fieldLabelColor: '#8E8E93',
  borderRadius: 8,
  fieldShape: 'rounded',
};
```

### Color Variants

**Blue:**

```typescript
const BlueThemeConfig = {
  primaryColor: '#0077C8',
  formBorderColor: '#0077C8',
  // ... standard light theme properties
};

const BlueDarkThemeConfig = {
  primaryColor: '#60A5FA',
  formBorderColor: '#60A5FA',
  formBackgroundColor: '#1F2937',
  fieldBackgroundColor: '#111827',
  // ... standard dark theme properties
};
```

**Green:**

```typescript
const GreenThemeConfig = {
  primaryColor: '#32a852',
  formBorderColor: '#32a852',
  // ... standard light theme properties
};

const GreenDarkThemeConfig = {
  primaryColor: '#34D399',
  formBorderColor: '#34D399',
  formBackgroundColor: '#1F2937',
  fieldBackgroundColor: '#111827',
  // ... standard dark theme properties
};
```

**Purple:**

```typescript
const PurpleThemeConfig = {
  primaryColor: '#8B5CF6',
  formBorderColor: '#8B5CF6',
  // ... standard light theme properties
};

const PurpleDarkThemeConfig = {
  primaryColor: '#A78BFA',
  formBorderColor: '#A78BFA',
  formBackgroundColor: '#1F2937',
  fieldBackgroundColor: '#111827',
  // ... standard dark theme properties
};
```

---

## Nullability Reference

### Required vs Optional

| Parameter   | Method/Component       | Required | Default Behavior            |
| ----------- | ---------------------- | -------- | --------------------------- |
| `theme`     | `setGlobalTheme()`     | No\*     | SDK defaults                |
| `darkTheme` | `setGlobalTheme()`     | No       | Uses `theme` for both modes |
| `theme`     | `SPLTextField`         | No       | Uses global theme           |
| `darkTheme` | `SPLTextField`         | No       | Uses `theme` for both modes |
| `theme`     | `paymentBottomSheet()` | No       | SDK defaults                |
| `darkTheme` | `paymentBottomSheet()` | No       | SDK defaults                |

**\*Note**: For `setGlobalTheme()`, at least one of `theme` or `darkTheme` must be provided.

### Behavior Matrix

| `theme` | `darkTheme` | Light Mode       | Dark Mode        |
| ------- | ----------- | ---------------- | ---------------- |
| ✅      | ✅          | Uses `theme`     | Uses `darkTheme` |
| ✅      | ❌          | Uses `theme`     | Uses `theme`     |
| ❌      | ✅          | Uses `darkTheme` | Uses `darkTheme` |
| ❌      | ❌          | SDK defaults     | SDK defaults     |

---

## Troubleshooting

### Theme Not Applying

**Check:**

1. SDK initialized before setting theme
2. Valid hex color format (e.g., `#FFFFFF`)
3. All required properties present

**Checklist:**

- [ ] `darkTheme` provided
- [ ] Device in dark mode
- [ ] Colors appropriate for dark backgrounds
- [ ] React Native detecting dark mode correctly

---

## Complete Example

```typescript
import React, { useEffect, useState } from 'react';
import { useColorScheme } from 'react-native';
import {
  SpreedlyCore,
  SPLTextField,
  FormFieldTypes,
  ValidationManager,
} from '@spreedly/react-native-checkout';

const lightTheme = {
  primaryColor: '#6366F1',
  secondaryColor: '#8B5CF6',
  formBorderColor: '#D1D5DB',
  formBackgroundColor: '#FFFFFF',
  fieldBackgroundColor: '#F9FAFB',
  fieldLabelColor: '#6B7280',
  borderRadius: 12,
  fieldShape: 'rounded',
};

const darkTheme = {
  primaryColor: '#818CF8',
  secondaryColor: '#A78BFA',
  formBorderColor: '#374151',
  formBackgroundColor: '#1F2937',
  fieldBackgroundColor: '#111827',
  fieldLabelColor: '#9CA3AF',
  borderRadius: 12,
  fieldShape: 'rounded',
};

export function ThemedPaymentForm() {
  const [fieldValidation, setFieldValidation] = useState({});
  const isDark = useColorScheme() === 'dark';

  useEffect(() => {
    const init = async () => {
      await SpreedlyCore.initSdk({ /* auth params */ });

      // Set global theme
      SpreedlyCore.setGlobalTheme({
        theme: lightTheme,
        darkTheme: darkTheme,
      });
    };
    init();
  }, []);

  const handlePaymentBottomSheet = () => {
    SpreedlyCore.paymentBottomSheet({
      theme: lightTheme,
      darkTheme: darkTheme,
    });
  };

  return (
    <>
      <SPLTextField
        formFieldType={FormFieldTypes.CARD}
        label="Card Number"
        theme={lightTheme}
        darkTheme={darkTheme}
        onValidationChange={(isValid) => {
          setFieldValidation(prev => ({ ...prev, card: isValid }));
        }}
      />

      <Button title="Pay with Bottom Sheet" onPress={handlePaymentBottomSheet} />
    </>
  );
}
```

---

## API Reference

### Types

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

### Methods

```typescript
// Both signatures supported
SpreedlyCore.setGlobalTheme(theme: BaseThemeConfig): void
SpreedlyCore.setGlobalTheme(options: GlobalThemeOptions): void

SpreedlyCore.paymentBottomSheet(options?: {
  theme?: BaseThemeConfig;
  darkTheme?: BaseThemeConfig;
  // ... other options
}): void
```

### Component Props

```typescript
<SPLTextField
  formFieldType={string}          // Required
  label={string}                  // Required
  theme={BaseThemeConfig}         // Optional
  darkTheme={BaseThemeConfig}     // Optional
/>

<PaymentBottomSheet
  theme={BaseThemeConfig}         // Optional
  darkTheme={BaseThemeConfig}     // Optional
/>
```

---

_For general SDK integration, see the [Integration Guide](./Integration_Guide.md). For implementation examples, see [Examples.md](./Examples.md)._
