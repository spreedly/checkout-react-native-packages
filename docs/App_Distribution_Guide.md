# App Distribution Guide

A comprehensive guide for distributing the Spreedly React Native SDK example app using GitHub Actions and Fastlane.

---

## Table of Contents

1. [Overview](#overview)
2. [GitHub Actions Setup](#github-actions-setup)
3. [Local Fastlane Setup](#local-fastlane-setup)
4. [Required Secrets & Configuration](#required-secrets--configuration)
5. [Running Distributions](#running-distributions)
6. [Troubleshooting](#troubleshooting)

---

## Overview

The app distribution system supports:

- **iOS**: TestFlight distribution via App Store Connect
- **Android**: Firebase App Distribution
- **Platforms**: Both GitHub Actions (CI/CD) and local development

### Distribution Methods

| Platform    | CI/CD (GitHub Actions)    | Local Development         |
| ----------- | ------------------------- | ------------------------- |
| **iOS**     | TestFlight                | TestFlight                |
| **Android** | Firebase App Distribution | Firebase App Distribution |

---

## GitHub Actions Setup

### Workflow File Location

`.github/workflows/app-distribution.yml`

### Required GitHub Secrets

Navigate to: **Repository → Settings → Secrets and variables → Actions**

#### iOS Secrets

```
ASC_KEY_ID=ABCD1234EF           # App Store Connect API Key ID
ASC_ISSUER_ID=12345678-1234-... # App Store Connect Issuer ID
ASC_KEY_CONTENT=-----BEGIN...   # App Store Connect API Key (full .p8 content)
```

#### Android Secrets

```
ANDROID_FIREBASE_APP_ID=1:123456789:android:abcdef123456
FIREBASE_SERVICE_ACCOUNT_KEY={"type":"service_account",...}  # Full JSON content
FIREBASE_TOKEN=1//abc123...     # Firebase CLI token (fallback)
FIREBASE_TESTER_GROUPS=qa-internal,developers
```

### Running GitHub Actions Workflow

1. **Navigate to Actions tab** in your GitHub repository
2. **Select "app-distribution" workflow**
3. **Click "Run workflow"**
4. **Configure inputs:**

#### Required Inputs

- **platform**: `ios` or `android`
- **branch**: Git branch/tag/SHA to build (default: `main`)

#### Optional Inputs

- **build_type** (Android only): `apk` or `aab` (default: `apk`)
- **release_notes**: Custom release notes
- **tester_groups**: Firebase tester groups (default: `qa-internal`)
- **testers**: Individual tester emails (comma-separated)

#### Example Workflow Run

```yaml
platform: android
build_type: apk
release_notes: 'Bug fixes and performance improvements'
tester_groups: 'qa-internal,beta-testers'
testers: 'john@example.com,jane@example.com'
branch: main
```

---

## Local Fastlane Setup

### Prerequisites

1. **Install dependencies:**

```bash
cd example
bundle install  # Install Ruby gems including Fastlane
```

2. **Platform-specific setup:**

**iOS:**

```bash
cd example/ios
pod install --repo-update
```

**Android:**

```bash
# Ensure Android SDK is installed and configured
# Verify with: android/gradlew assembleRelease
```

### Required Local Configuration

#### iOS Configuration

Create App Store Connect API key file:

```bash
# Save your .p8 key file securely
mkdir -p ~/secrets
# Place your AuthKey_ABCD1234EF.p8 file in ~/secrets/
```

#### Android Configuration

Create Firebase service account file:

```bash
# Save your service account JSON securely
mkdir -p ~/secrets
# Place your firebase-service-account.json file in ~/secrets/
```

### Environment Variables Setup

#### iOS Environment Variables

```bash
export ASC_KEY_ID="ABCD1234EF"
export ASC_ISSUER_ID="12345678-1234-5678-9abc-123456789abc"
export ASC_KEY_CONTENT="$(cat ~/secrets/AuthKey_ABCD1234EF.p8)"
export IOS_CHANGELOG="Local iOS build - $(date)"
```

#### Android Environment Variables

**Option 1: Using Service Account File Path**

```bash
export GOOGLE_APPLICATION_CREDENTIALS="$HOME/secrets/firebase-service-account.json"
export ANDROID_FIREBASE_APP_ID="1:123456789:android:abcdef123456"
export FIREBASE_TESTER_GROUPS="qa-internal"
export ANDROID_CHANGELOG="Local Android build - $(date)"
export ANDROID_DIST_TYPE="apk"  # or "aab"
```

**Option 2: Using Service Account JSON Content**

```bash
export FIREBASE_SERVICE_ACCOUNT_KEY="$(cat ~/secrets/firebase-service-account.json)"
export ANDROID_FIREBASE_APP_ID="1:123456789:android:abcdef123456"
export FIREBASE_TESTER_GROUPS="qa-internal"
export ANDROID_CHANGELOG="Local Android build - $(date)"
export ANDROID_DIST_TYPE="apk"
```

**Option 3: Using Firebase CLI Token (Fallback)**

```bash
# First, get Firebase CLI token
firebase login:ci  # Copy the token

export FIREBASE_TOKEN="1//abc123def456..."
export ANDROID_FIREBASE_APP_ID="1:123456789:android:abcdef123456"
export FIREBASE_TESTER_GROUPS="qa-internal"
export ANDROID_CHANGELOG="Local Android build - $(date)"
export ANDROID_DIST_TYPE="apk"
```

### Running Fastlane Locally

#### iOS Distribution

```bash
cd example

# Set iOS environment variables (see above)
export ASC_KEY_ID="ABCD1234EF"
export ASC_ISSUER_ID="12345678-1234-5678-9abc-123456789abc"
export ASC_KEY_CONTENT="$(cat ~/secrets/AuthKey_ABCD1234EF.p8)"
export IOS_CHANGELOG="Local iOS build - $(date)"

# Run Fastlane
bundle exec fastlane ios beta
```

#### Android Distribution

```bash
cd example

# Set Android environment variables (choose one option from above)
export GOOGLE_APPLICATION_CREDENTIALS="$HOME/secrets/firebase-service-account.json"
export ANDROID_FIREBASE_APP_ID="1:123456789:android:abcdef123456"
export FIREBASE_TESTER_GROUPS="qa-internal"
export ANDROID_CHANGELOG="Local Android build - $(date)"
export ANDROID_DIST_TYPE="apk"

# Run Fastlane
bundle exec fastlane android beta
```

---

## Required Secrets & Configuration

### Complete Configuration Checklist

#### GitHub Repository Secrets

- [ ] `ASC_KEY_ID` - App Store Connect API Key ID
- [ ] `ASC_ISSUER_ID` - App Store Connect Issuer ID
- [ ] `ASC_KEY_CONTENT` - App Store Connect API Key (.p8 file content)
- [ ] `ANDROID_FIREBASE_APP_ID` - Firebase Android App ID
- [ ] `FIREBASE_SERVICE_ACCOUNT_KEY` - Firebase service account JSON
- [ ] `FIREBASE_TOKEN` - Firebase CLI token (optional fallback)
- [ ] `FIREBASE_TESTER_GROUPS` - Default Firebase tester groups

#### Local Development Files

- [ ] `~/secrets/AuthKey_ABCD1234EF.p8` - App Store Connect API key
- [ ] `~/secrets/firebase-service-account.json` - Firebase service account

#### App Store Connect Setup

1. **Create API Key:**
   - App Store Connect → Users and Access → Keys → App Store Connect API
   - Download `.p8` file and note Key ID and Issuer ID

2. **Configure App:**
   - Ensure app exists in App Store Connect
   - TestFlight access configured

#### Firebase Setup

1. **Create Service Account:**
   - Firebase Console → Project Settings → Service accounts
   - Generate new private key (downloads JSON file)

2. **Configure App:**
   - Firebase Console → App Distribution
   - Add tester groups and individual testers
   - Note the Firebase App ID

---

## Running Distributions

### Quick Start Scripts

#### Create Local Distribution Scripts

**iOS Distribution Script** (`scripts/distribute_ios.sh`):

```bash
#!/bin/bash
set -e

echo "🍎 Starting iOS distribution..."

# Configuration
export ASC_KEY_ID="ABCD1234EF"
export ASC_ISSUER_ID="12345678-1234-5678-9abc-123456789abc"
export ASC_KEY_CONTENT="$(cat ~/secrets/AuthKey_ABCD1234EF.p8)"
export IOS_CHANGELOG="Local iOS build - $(date '+%Y-%m-%d %H:%M')"

cd example
echo "📦 Building and uploading to TestFlight..."
bundle exec fastlane ios beta

echo "✅ iOS distribution complete!"
echo "📱 Check TestFlight for the new build"
```

**Android Distribution Script** (`scripts/distribute_android.sh`):

```bash
#!/bin/bash
set -e

echo "🤖 Starting Android distribution..."

# Configuration
export GOOGLE_APPLICATION_CREDENTIALS="$HOME/secrets/firebase-service-account.json"
export ANDROID_FIREBASE_APP_ID="1:123456789:android:abcdef123456"
export FIREBASE_TESTER_GROUPS="qa-internal"
export ANDROID_CHANGELOG="Local Android build - $(date '+%Y-%m-%d %H:%M')"
export ANDROID_DIST_TYPE="apk"

cd example
echo "📦 Building and uploading to Firebase App Distribution..."
bundle exec fastlane android beta

echo "✅ Android distribution complete!"
echo "📱 Check Firebase App Distribution for the new build"
```

**Make scripts executable:**

```bash
chmod +x scripts/distribute_ios.sh
chmod +x scripts/distribute_android.sh
```

### Usage Examples

#### GitHub Actions

```bash
# Trigger via GitHub UI or API
# Replace OWNER and REPO with your repository details
curl -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/repos/OWNER/REPO/actions/workflows/app-distribution.yml/dispatches \
  -d '{
    "ref": "main",
    "inputs": {
      "platform": "android",
      "build_type": "apk",
      "release_notes": "Bug fixes and improvements",
      "tester_groups": "qa-internal,beta-testers"
    }
  }'
```

#### Local Development

```bash
# iOS
./scripts/distribute_ios.sh

# Android
./scripts/distribute_android.sh

# Or run Fastlane directly
cd example
bundle exec fastlane ios beta      # iOS
bundle exec fastlane android beta  # Android
```

---

## Troubleshooting

### Common Issues

#### 1. **Authentication Failures**

**Problem**: "Authentication failed" or "Invalid credentials"

**iOS Solutions:**

```bash
# Verify App Store Connect API key
openssl pkcs8 -nocrypt -in ~/secrets/AuthKey_ABCD1234EF.p8

# Check key permissions in App Store Connect
# Ensure key has "Developer" role or higher
```

**Android Solutions:**

```bash
# Verify service account JSON
cat ~/secrets/firebase-service-account.json | jq .

# Test Firebase authentication
firebase projects:list --token="$FIREBASE_TOKEN"

# Check service account permissions in Firebase Console
# Ensure account has "Firebase App Distribution Admin" role
```

#### 2. **Build Failures**

**Problem**: Build fails during compilation

**Solutions:**

```bash
# Clean builds
cd example

# iOS
cd ios && rm -rf build/ && pod install && cd ..

# Android
cd android && ./gradlew clean && cd ..

# Verify dependencies
bundle install
```

#### 3. **Missing Environment Variables**

**Problem**: "Environment variable not set" errors

**Solutions:**

```bash
# Check current environment
env | grep -E "(ASC_|FIREBASE_|ANDROID_)"

# Verify required variables are set
echo "iOS: $ASC_KEY_ID"
echo "Android: $ANDROID_FIREBASE_APP_ID"
```

#### 4. **Firebase App Distribution Issues**

**Problem**: "App not found" or "Invalid app ID"

**Solutions:**

```bash
# Verify Firebase App ID format
# Should be: 1:123456789:android:abcdef123456

# Check Firebase project access
firebase projects:list

# Verify app exists in Firebase Console
# Firebase Console → App Distribution → Apps
```

#### 5. **TestFlight Upload Issues**

**Problem**: "Invalid bundle" or "Upload failed"

**Solutions:**

```bash
# Verify app configuration
# App Store Connect → My Apps → [Your App]

# Check bundle identifier matches
# iOS project settings vs App Store Connect

# Verify provisioning profiles
# Xcode → Preferences → Accounts → Download Manual Profiles
```

### Debug Mode

Enable verbose logging for troubleshooting:

```bash
# Fastlane verbose mode
bundle exec fastlane ios beta --verbose
bundle exec fastlane android beta --verbose

# Firebase CLI debug mode
export FIREBASE_DEBUG=true
```

### Getting Help

**Log Locations:**

- **Fastlane logs**: `example/fastlane/logs/`
- **GitHub Actions logs**: Actions tab → Workflow run → Job logs
- **Xcode logs**: Xcode → Window → Devices and Simulators → View Device Logs

**Support Channels:**

- **Internal**: #mobile-sdk Slack channel
- **Fastlane**: [Fastlane Documentation](https://docs.fastlane.tools/)
- **Firebase**: [Firebase App Distribution Docs](https://firebase.google.com/docs/app-distribution)

---

## Security Best Practices

### Secret Management

- ✅ **Never commit secrets** to version control
- ✅ **Use GitHub Secrets** for CI/CD
- ✅ **Store local secrets** in `~/secrets/` with proper permissions
- ✅ **Rotate secrets regularly** (quarterly recommended)
- ✅ **Use service accounts** instead of personal accounts

### File Permissions

```bash
# Secure local secret files
chmod 600 ~/secrets/firebase-service-account.json
chmod 600 ~/secrets/AuthKey_*.p8
```

### Access Control

- **App Store Connect**: Use API keys with minimal required permissions
- **Firebase**: Use service accounts with "Firebase App Distribution Admin" role only
- **GitHub**: Limit repository access to necessary team members

---

_This guide covers the complete setup and usage of the app distribution system. For additional support, contact the mobile SDK team._
