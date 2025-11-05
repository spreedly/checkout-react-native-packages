# Privacy Policy - Spreedly Checkout React Native SDK

**Version:** 1.0  
**Last Updated:** October 31, 2025

## Overview

The Spreedly Checkout React Native SDK is designed with privacy as a core principle. This document outlines how the SDK handles sensitive payment information and what data practices developers and end-users can expect when integrating this SDK into their applications.

## Privacy Principles

The Spreedly Checkout React Native SDK adheres to the following privacy principles:

- **Data Minimization**: We collect and process only the minimum data necessary to facilitate payment tokenization
- **Security by Design**: All data transmission is encrypted and card data is never persisted
- **Transparency**: Clear documentation of all data handling practices
- **No Tracking**: The SDK does not track user behavior or collect analytics

## Data Handling Practices

### 1. Payment Card Data

**Processing**:

- Payment card data (card number, CVV, expiration date) is processed **in memory only**
- Card data exists transiently during the tokenization process
- No payment card data is ever written to disk or persistent storage

**Storage**:

- **None** - The SDK does not store any payment card information
- All card data is immediately cleared from memory after tokenization
- No caching of sensitive payment information occurs

### 2. Data Transmission

**Secure Transmission**:

- All card data is transmitted to Spreedly's tokenization service over **TLS (Transport Layer Security)**
- Communication occurs directly between the SDK and Spreedly's secure endpoints
- No intermediary servers or services have access to card data in transit

**Encryption**:

- Industry-standard encryption protocols are used for all network communications
- Certificate pinning ensures connections are only made to verified Spreedly endpoints

### 3. Data Collection

**No Data Collection**:

- The SDK does **not collect** any user data, analytics, or telemetry
- The SDK serves purely as a facilitator for payment processing
- No personally identifiable information (PII) is collected or stored by the SDK

**Developer Responsibility**:

- Developers integrating this SDK are responsible for their own data collection practices
- Any data collected by the host application is separate from SDK operations

### 4. Persistent Storage

**No Persistent Storage**:

- The SDK does **not** use any form of persistent storage
- No data is written to:
  - Device file system
  - Shared preferences / User defaults
  - Databases
  - Cache directories
  - Keychain / Keystore

**Memory Only**:

- All SDK operations occur in volatile memory
- Data is automatically cleared when the SDK session ends

### 5. User Tracking

**No Tracking**:

- The SDK does **not** track user behavior
- No analytics or usage statistics are collected
- No session tracking or user profiling occurs
- No advertising identifiers are accessed or used

### 6. Third-Party Data Sharing

**Spreedly Only**:

- Card data is shared **exclusively** with Spreedly for tokenization purposes
- Spreedly is PCI DSS Level 1 certified
- No other third-party services receive any data from the SDK

**Tokenization Process**:

- Card data is sent to Spreedly's secure tokenization service
- Spreedly returns a payment method token
- The token (not card data) is returned to the host application

**Spreedly's Data Practices**:

- For information about how Spreedly handles payment data, please refer to [Spreedly's Privacy Policy](https://www.spreedly.com/privacy)

## SDK Permissions

### iOS

The SDK does **not** require any special iOS permissions or entitlements.

**iOS Privacy Requirements:**

- The SDK includes a `PrivacyInfo.xcprivacy` file for Apple's privacy manifest compliance
- For complete iOS privacy requirements (Privacy Manifest + App Store Connect), see [iOS Privacy Guide](iOS_Privacy_Guide.md)
- The privacy manifest is automatically included in the SDK bundle via CocoaPods

### Android

The SDK does **not** require any special Android permissions in the manifest.

**Google Play Console Data Safety Requirements:**

- For format-specific requirements when completing Google Play Console's Data Safety questionnaire, see [Android Data Safety Guide](Android_Data_Safety_Guide.md)
- This guide provides step-by-step instructions for accurately reporting the SDK's data practices in Google Play Console

## Network Activity

**Connections**:

- The SDK only connects to Spreedly's API endpoints
- All connections are made over HTTPS with TLS encryption
- No connections are made to any other external services

## Developer Responsibilities

Developers integrating this SDK should:

1. **Implement Their Own Privacy Policy**: Cover their application's data practices
2. **Secure Token Storage**: If storing payment tokens, use secure storage mechanisms
3. **Comply with PCI DSS**: Follow PCI DSS requirements appropriate to their integration
4. **User Consent**: Obtain necessary user consent for payment processing
5. **Data Protection Laws**: Comply with applicable data protection regulations (GDPR, CCPA, etc.)

## Compliance

### PCI DSS

The SDK is designed to minimize PCI DSS scope for developers:

- Card data never touches persistent storage
- Direct transmission to PCI-certified payment processor (Spreedly)
- Minimal card data exposure in the application

### Regional Regulations

**GDPR (General Data Protection Regulation)**:

- The SDK processes payment data as a data processor on behalf of the integrating application
- No personal data is stored or tracked by the SDK
- Data minimization principles are followed

**Other Regulations**:

- The SDK's design facilitates compliance with various regional data protection laws

## Security Measures

### Data Protection

- TLS encryption for all network transmission
- No persistent storage of sensitive data
- Memory is cleared after processing
- Certificate pinning to prevent man-in-the-middle attacks

### Code Security

- Regular security audits
- Input validation and sanitization
- Secure coding practices following OWASP guidelines
- No logging of sensitive payment information

## Updates to This Policy

This privacy policy may be updated periodically to reflect:

- Changes in SDK functionality
- Updates to privacy regulations
- Security improvements

**Notification**:

- Policy updates will be included in SDK release notes
- Version number and last updated date will be incremented
- Material changes will be clearly communicated to developers

## Contact Information

For questions or concerns about privacy practices in the Spreedly Checkout React Native SDK:

**Spreedly Support**: https://www.spreedly.com/support  
**Documentation**: https://docs.spreedly.com

## Summary

The Spreedly Checkout React Native SDK is a privacy-focused solution that:

✅ Processes card data in memory only  
✅ Transmits data securely via TLS  
✅ Collects no user data  
✅ Uses no persistent storage  
✅ Performs no user tracking  
✅ Shares data only with Spreedly for tokenization

By design, this SDK minimizes privacy risks and helps developers build secure payment experiences while maintaining user privacy.

---

**Note**: This document describes the privacy practices of the Spreedly Checkout React Native SDK itself. Applications integrating this SDK must maintain their own privacy policies covering their specific data practices and use cases.
