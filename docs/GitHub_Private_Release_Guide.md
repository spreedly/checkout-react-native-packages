# Spreedly React Native SDK - GitHub Private Release Guide

A comprehensive guide for releasing the Spreedly React Native SDK to GitHub Packages (private distribution).

---

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Setup](#setup)
4. [Release Process](#release-process)
5. [Installation Instructions](#installation-instructions)
6. [Troubleshooting](#troubleshooting)

---

## Overview

This guide covers the **private release process** for the Spreedly React Native SDK using GitHub Packages.

### **🔒 GitHub Packages (Private)**

- **Target**: Internal teams and private clients
- **Registry**: GitHub Packages (`npm.pkg.github.com`)
- **Access**: Requires GitHub authentication
- **Package Name**: `@spreedly/react-native-checkout`
- **Import**: `import { SpreedlyCore } from '@spreedly/react-native-checkout';`

---

## Prerequisites

### Required Secrets

Configure these secrets in your GitHub repository:
**Repository → Settings → Secrets and variables → Actions**

#### GitHub Packages (Required)

```
GITHUB_TOKEN=ghp_xxxxxxxxxxxx  # Automatically provided by GitHub Actions
```

_Note: `GITHUB_TOKEN` is automatically provided by GitHub Actions with the necessary permissions for GitHub Packages._

#### Slack Notifications (Recommended)

```
SLACK_WEBHOOK_URL=https://hooks.slack.com/...  # Team notifications via Slack
```

**Note**: Slack notifications are now implemented using native curl commands (enterprise-compliant). The workflow will automatically skip Slack notifications if `SLACK_WEBHOOK_URL` is not configured.

#### GPG Signing (Optional but Recommended)

```
GPG_PRIVATE_KEY=-----BEGIN PGP PRIVATE KEY BLOCK-----
...your private key...
-----END PGP PRIVATE KEY BLOCK-----

GPG_PASSPHRASE=your_gpg_passphrase
```

**Note**: GPG signing provides cryptographic authentication for Git tags and release manifests. The workflow will automatically skip GPG operations if these secrets are not configured, but will still create standard (unsigned) tags and releases.

### Repository Permissions

Ensure your repository has the correct permissions:

1. **Go to Repository Settings → Actions → General**
2. **Workflow permissions**: Select "Read and write permissions"
3. **Allow GitHub Actions to create and approve pull requests**: ✅ Enabled

### Package Permissions

Configure GitHub Packages permissions:

1. **Go to Repository Settings → Code and automation → Packages**
2. **Package creation**: Allow packages to be created
3. **Package visibility**: Set to "Private" for internal releases

---

## Setup

### Quick Setup Checklist

Before your first release, complete these setup steps:

#### ✅ GitHub Repository Setup

- [ ] **Repository Permissions**: Set to "Read and write permissions" in Actions settings
- [ ] **Package Creation**: Enabled in repository settings
- [ ] **Package Visibility**: Set to "Private" for internal releases
- [ ] **Branch Protection**: Configure main branch protection rules (optional)

#### ✅ GitHub Packages Authentication

- [ ] **GITHUB_TOKEN**: Automatically available (no setup needed)
- [ ] **Workflow Permissions**: Includes `contents: write` and `packages: write`
- [ ] **Repository Access**: Team members have appropriate access levels

#### ✅ Release Workflow Testing

- [ ] **Test Build**: Run `yarn prepare` locally
- [ ] **Test Linting**: Run `yarn lint` locally
- [ ] **Test Tests**: Run `yarn test` locally
- [ ] **Dry Run**: Test workflow with prerelease first

#### ✅ GPG Setup (Optional)

- [ ] **Generate GPG Key**: Create a dedicated key for releases
- [ ] **Export Private Key**: Add to GitHub Secrets as `GPG_PRIVATE_KEY`
- [ ] **Export Public Key**: Update `public-key.asc` in repository
- [ ] **Test Signing**: Verify GPG setup works locally

### Repository Configuration

**Step-by-step setup:**

1. **Repository Settings**:
   - Go to **Repository → Settings → General**
   - Ensure repository is **Private** (required for GitHub Packages)
   - Verify **Issues** and **Projects** are enabled (optional)

2. **Actions Permissions**:
   - Go to **Repository → Settings → Actions → General**
   - **Actions permissions**: Allow all actions and reusable workflows
   - **Workflow permissions**: Select **"Read and write permissions"**
   - **Allow GitHub Actions to create and approve pull requests**: ✅ **Enabled**

3. **Package Settings**:
   - Go to **Repository → Settings → Code and automation → Packages**
   - **Package creation**: ✅ **Allow packages to be created**
   - **Package visibility**: Set to **"Private"** for internal releases

### First-Time Setup Verification

**Verify your setup with these commands:**

```bash
# 1. Check repository permissions
# Replace OWNER and REPO with your repository details
gh api repos/OWNER/REPO --jq '.permissions'

# 2. Verify workflow file exists
ls -la .github/workflows/release.yml

# 3. Test local build
yarn install
yarn test
yarn lint
yarn prepare

# 4. Check package.json configuration
cat package.json | jq '.name, .publishConfig'
```

### GPG Setup (Optional but Recommended)

**Generate GPG Key for Releases:**

```bash
# 1. Generate a new GPG key
gpg --full-generate-key

# Follow prompts:
# - Key type: (1) RSA and RSA (default)
# - Key size: 4096 bits
# - Expiration: 2y (2 years)
# - Name: Spreedly Release Bot
# - Email: releases@spreedly.com
# - Passphrase: [secure passphrase]

# 2. List keys to get the key ID
gpg --list-secret-keys --keyid-format LONG
# Note the key ID after sec rsa4096/

# 3. Export private key for GitHub Secrets
gpg --armor --export-secret-keys YOUR_KEY_ID > private-key.asc

# 4. Export public key for repository
gpg --armor --export YOUR_KEY_ID > public-key.asc

# 5. Add to GitHub Secrets:
# - GPG_PRIVATE_KEY: Contents of private-key.asc
# - GPG_PASSPHRASE: Your GPG passphrase

# 6. Replace public-key.asc in repository with your exported public key

# 7. Test signing locally
echo "test" | gpg --armor --detach-sign --local-user YOUR_KEY_ID
```

**Security Notes:**

- Keep your GPG private key and passphrase secure
- Use a strong passphrase for the GPG key
- Consider using a dedicated key for releases only
- Regularly rotate keys (every 1-2 years)
- Store backup of private key securely offline

### Package Scope and Naming

GitHub Packages requires scoped packages:

- **Private Package Name**: `@spreedly/react-native-checkout`
- **Registry**: `https://npm.pkg.github.com`
- **Access**: Restricted (requires authentication)
- **Scope**: `@spreedly` (matches GitHub organization)

---

## Software Bill of Materials (SBOM)

Starting with the automated release workflow, every release automatically generates and includes a comprehensive Software Bill of Materials (SBOM) that documents all dependencies and components used in the build.

### SBOM Overview

The SBOM provides:

- **Dependency Transparency**: Complete list of all direct and transitive dependencies
- **Security Compliance**: Meets NTIA SBOM minimum requirements for federal software procurement
- **Vulnerability Tracking**: Enables automated security scanning and vulnerability management
- **License Compliance**: Tracks component licenses for legal compliance
- **Supply Chain Security**: Provides artifact integrity verification

### Generated SBOM Files

Each release includes three SBOM-related files:

| File             | Format         | Purpose                                       |
| ---------------- | -------------- | --------------------------------------------- |
| `sbom.json`      | CycloneDX JSON | Machine-readable format for automated tools   |
| `sbom.xml`       | CycloneDX XML  | Industry-standard format for enterprise tools |
| `npm-audit.json` | npm audit      | Vulnerability report from npm registry        |

### SBOM Standards Compliance

- **Format**: CycloneDX specification (OWASP standard)
- **Generator**: `@cyclonedx/cdxgen` - Official CycloneDX tool for Node.js projects
- **Standards**: NTIA SBOM minimum elements compliance
- **Coverage**: Direct and transitive dependencies with licensing information
- **Integrity**: SHA-256 checksums included in release manifest
- **Verification**: GPG-signed manifest for tamper detection

### Using the SBOM

#### Download SBOM Files

```bash
# Download SBOM files for a specific release
VERSION="1.2.3" # Replace with your desired version
# Replace OWNER and REPO with your repository details

curl -L "https://github.com/OWNER/REPO/releases/download/v${VERSION}/sbom.json" -o sbom.json
curl -L "https://github.com/OWNER/REPO/releases/download/v${VERSION}/sbom.xml" -o sbom.xml
curl -L "https://github.com/OWNER/REPO/releases/download/v${VERSION}/npm-audit.json" -o npm-audit.json
```

#### Verify SBOM Integrity

```bash
# Download and verify the release manifest
# Replace OWNER and REPO with your repository details
curl -L "https://github.com/OWNER/REPO/releases/download/v${VERSION}/release-manifest.json" -o manifest.json

# Extract SBOM checksums from manifest and verify
grep -A 5 "sbom.json" manifest.json | grep "sha256"
sha256sum -c <<< "$(grep -A 5 '"name": "sbom-output/sbom.json"' manifest.json | grep sha256 | cut -d'"' -f4)  sbom.json"
```

#### Analyze Dependencies

```bash
# View component count
cat sbom.json | jq '.components | length'

# List all components with versions
cat sbom.json | jq -r '.components[] | "\(.name)@\(.version)"'

# Check for known vulnerabilities (requires npm-audit.json)
cat npm-audit.json | jq '.vulnerabilities | keys | length'
```

#### Integration with Security Tools

The CycloneDX format is compatible with:

- **Dependency-Track**: Open-source vulnerability management
- **FOSSA**: Commercial license and security scanning
- **Snyk**: Developer security platform
- **JFrog Xray**: Universal artifact analysis
- **GitHub Security**: Advanced security features
- **SPDX Tools**: License compliance checking

### SBOM in CI/CD Pipelines

Organizations can integrate SBOM verification into their CI/CD:

```bash
# Example: Verify no high-severity vulnerabilities
VULN_COUNT=$(cat npm-audit.json | jq '.metadata.vulnerabilities.high // 0')
if [ "$VULN_COUNT" -gt 0 ]; then
  echo "❌ High-severity vulnerabilities found: $VULN_COUNT"
  exit 1
fi

# Example: Verify component count hasn't dramatically increased
COMPONENT_COUNT=$(cat sbom.json | jq '.components | length')
if [ "$COMPONENT_COUNT" -gt 500 ]; then
  echo "⚠️  Large number of components: $COMPONENT_COUNT"
fi
```

---

## Release Process

### Automated Release (Recommended)

#### 1. Trigger Release Workflow

**Via GitHub UI:**

1. Navigate to **Actions** tab
2. Select **"Release Package"** workflow
3. Click **"Run workflow"**
4. Configure release parameters:

```yaml
Version Type: patch|minor|major|prerelease
Custom Version: 1.2.3 (optional)
Release Notes: 'Bug fixes and improvements'
Registry: github (select this for private release)
Prerelease: false|true
```

**Via GitHub CLI:**

```bash
gh workflow run release.yml \
  -f version_type=minor \
  -f release_notes="Added new payment methods" \
  -f registry=github \
  -f prerelease=false
```

#### 2. Release Workflow Steps

The automated workflow:

1. ✅ **Validates** code (tests, linting)
2. 📦 **Builds** package (`yarn prepare`)
3. 🔢 **Bumps** version in `package.json`
4. ⚙️ **Configures** GitHub Packages registry
5. 📤 **Publishes** to GitHub Packages
6. 📋 **Generates** Software Bill of Materials (SBOM) in CycloneDX format
7. 🔐 **Generates** release manifest with SHA-256 checksums (including SBOM)
8. ✍️ **Signs** manifest with GPG (if configured)
9. 🏷️ **Creates** signed Git tag and GitHub release
10. 📎 **Attaches** manifest, signature, SBOM, and package files to release
11. 📝 **Generates** changelog
12. 🔔 **Notifies** team via Slack (if configured)
13. 📋 **Creates** summary with security and SBOM verification steps

#### 3. Pre-Release Testing

Before running a production release, test with a prerelease:

```bash
# Test GitHub Packages release
gh workflow run release.yml \
  -f version_type=prerelease \
  -f release_notes="Testing release workflow" \
  -f registry=github \
  -f prerelease=true
```

**Verify the prerelease:**

1. Check GitHub Packages: Repository → Packages
2. Test installation: `npm install @spreedly/react-native-checkout@1.0.0-beta.0`
3. Verify package contents and functionality
4. Delete prerelease if successful: Repository → Releases → Delete

#### 4. Production Release

Once testing is complete, run the production release:

```bash
# Production GitHub Packages release
gh workflow run release.yml \
  -f version_type=minor \
  -f release_notes="New features and improvements" \
  -f registry=github \
  -f prerelease=false
```

### Manual Release (Advanced)

#### GitHub Packages Release

```bash
# 1. Prepare release
yarn install
yarn test
yarn lint
yarn prepare

# 2. Version bump
npm version patch  # or minor, major

# 3. Configure GitHub Packages
echo "@spreedly:registry=https://npm.pkg.github.com" >> .npmrc
echo "//npm.pkg.github.com/:_authToken=$GITHUB_TOKEN" >> .npmrc

# 4. Update package.json for GitHub Packages
node -e "
  const pkg = require('./package.json');
  pkg.publishConfig = {
    registry: 'https://npm.pkg.github.com',
    access: 'restricted'
  };
  require('fs').writeFileSync('./package.json', JSON.stringify(pkg, null, 2));
"

# 5. Publish
npm publish

# 6. Create Git tag
git add package.json
git commit -m "chore: release v$(node -p "require('./package.json').version")"
git tag v$(node -p "require('./package.json').version")
git push origin main --tags
```

---

## Installation Instructions

### For Clients Using GitHub Packages (Private)

#### 1. Authentication Setup

**NPM Configuration:**

```bash
# Configure npm for GitHub Packages
npm config set @spreedly:registry https://npm.pkg.github.com

# Authenticate (choose one method)

# Method A: Personal Access Token
npm config set //npm.pkg.github.com/:_authToken ghp_xxxxxxxxxxxx

# Method B: .npmrc file in project
echo "@spreedly:registry=https://npm.pkg.github.com" >> .npmrc
echo "//npm.pkg.github.com/:_authToken=\${GITHUB_TOKEN}" >> .npmrc

# Method C: Environment variable
export GITHUB_TOKEN=ghp_xxxxxxxxxxxx
```

**Yarn Configuration:**

For **Yarn v2+ (Berry)**, create `.yarnrc.yml`:

```yaml
npmScopes:
  spreedly:
    npmRegistryServer: 'https://npm.pkg.github.com'
    npmAuthToken: '${GITHUB_TOKEN}'
npmRegistryServer: 'https://registry.npmjs.org'
```

For **Yarn v1 (Classic)**, create `.yarnrc`:

```
"@spreedly:registry" "https://npm.pkg.github.com"
"//npm.pkg.github.com/:_authToken" "${GITHUB_TOKEN}"
registry "https://registry.npmjs.org"
```

Then set the environment variable:

```bash
export GITHUB_TOKEN=ghp_xxxxxxxxxxxx
```

#### 2. Installation

```bash
# Install the package
npm install @spreedly/react-native-checkout

# Or with yarn
yarn add @spreedly/react-native-checkout
```

#### 3. Platform Setup

```bash
# iOS
cd ios && pod install

# Android (automatic linking)
```

### Integration Code

```typescript
import { SpreedlyCore } from '@spreedly/react-native-checkout';

// Initialize SDK
await SpreedlyCore.initSdk({
  environmentKey: 'your_environment_key',
  // ... other options
});
```

**GitHub Token Requirements:**

- **Scope**: `read:packages` (minimum)
- **Access**: Repository access or organization membership
- **Expiration**: Set appropriate expiration (90 days recommended)

---

## Troubleshooting

### Common Issues

#### 1. **GitHub Packages Authentication Failures**

**Problem**: `npm ERR! 401 Unauthorized` when publishing to GitHub Packages

**Solutions**:

```bash
# Verify GitHub token has correct permissions
curl -H "Authorization: token $GITHUB_TOKEN" https://api.github.com/user

# Check if token has packages permission
curl -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/user/packages?package_type=npm

# Verify repository access
gh auth status

# Test GitHub Packages authentication
npm whoami --registry=https://npm.pkg.github.com
```

**Required Token Permissions:**

- ✅ `read:packages` - Read packages
- ✅ `write:packages` - Publish packages
- ✅ `repo` - Repository access
- ✅ `read:org` - Organization membership (if applicable)

#### 2. **GitHub Actions Workflow Failures**

**Problem**: Release workflow fails with permissions error

**Solutions**:

```bash
# Check workflow permissions
# Replace OWNER and REPO with your repository details
gh api repos/OWNER/REPO/actions/permissions

# Verify repository settings
# Go to Settings → Actions → General → Workflow permissions
# Select "Read and write permissions"

# Check if GITHUB_TOKEN has required scopes
gh auth status --show-token
```

#### 3. **Build Failures**

**Problem**: Build fails during release

**Solutions**:

```bash
# Clean build
rm -rf lib/ node_modules/
yarn install
yarn prepare

# Check build output
ls -la lib/

# Verify TypeScript compilation
yarn tsc --noEmit
```

#### 4. **Git Tag Issues**

**Problem**: Tag already exists

**Solutions**:

```bash
# Delete local tag
git tag -d v1.2.3

# Delete remote tag
git push origin :refs/tags/v1.2.3

# Force create new tag
git tag -f v1.2.3
git push origin v1.2.3 --force
```

#### 5. **Registry Configuration**

**Problem**: Publishing to wrong registry

**Solutions**:

```bash
# Check current registry
npm config get registry

# Check scoped registry
npm config get @spreedly:registry

# Reset configuration
npm config delete registry
npm config delete @spreedly:registry
```

### Debug Commands

```bash
# Check npm configuration
npm config list

# Verify package contents
npm pack --dry-run

# Test installation locally
npm install ./spreedly-checkout-1.0.0.tgz

# Check published package
npm view @spreedly/react-native-checkout
```

### Getting Help

**Internal Support:**

- **Slack**: #mobile-sdk channel
- **Email**: mobile-team@spreedly.com

**External Resources:**

- **GitHub Packages**: [docs.github.com/packages](https://docs.github.com/packages)

---

## Release Security & Integrity

### Release Manifest

Starting with our enhanced release workflow, every release includes a cryptographic **Release Manifest** (`release-manifest.json`) that provides integrity verification for all published assets.

#### What's Included in the Manifest

The release manifest contains SHA-256 checksums for:

- **NPM Package Tarball** - The main distributable package
- **Compiled Library Files** - All files in `lib/` directory (`.js`, `.d.ts`, `.json`)
- **Key Source Files** - TypeScript source files from `src/`
- **Metadata Files** - `package.json`, `README.md`, `LICENSE`

#### Manifest Format

```json
{
  "release_info": {
    "version": "1.2.3",
    "tag": "v1.2.3",
    "timestamp": "2024-01-15T10:30:00Z",
    "commit_sha": "abc123...",
    "prerelease": false
  },
  "assets": [
    {
      "name": "spreedly-checkout-1.2.3.tgz",
      "type": "npm_package",
      "sha256": "d4f8a7b2c1e9f5a6b3c8d9e2f1a4b7c5d8e9f2a5b6c9d2e5f8a1b4c7d0e3f6a9",
      "size_bytes": 125840,
      "description": "NPM package tarball for GitHub Packages"
    },
    {
      "name": "lib/index.js",
      "type": "compiled_library",
      "sha256": "a1b2c3d4e5f6789012345678901234567890123456789012345678901234567890",
      "size_bytes": 4562,
      "description": "Compiled library file"
    }
    // ... more assets
  ]
}
```

#### Verifying Release Integrity

**Step 1: Download the Manifest**

```bash
# Download manifest for specific version
# Replace OWNER and REPO with your repository details
curl -L https://github.com/OWNER/REPO/releases/download/v1.2.3/release-manifest.json \
  -o release-manifest.json
```

**Step 2: Verify Manifest Integrity**

Each release includes the manifest's own SHA-256 checksum in the GitHub Release description:

```bash
# Verify the manifest itself hasn't been tampered with
echo "MANIFEST_SHA256_FROM_RELEASE  release-manifest.json" | sha256sum -c
```

**Step 3: Verify Individual Assets**

```bash
# Download and verify the NPM package
npm pack @spreedly/react-native-checkout@1.2.3

# Extract the filename and expected checksum from manifest
TARBALL_NAME=$(cat release-manifest.json | jq -r '.assets[] | select(.type=="npm_package") | .name')
EXPECTED_SHA=$(cat release-manifest.json | jq -r '.assets[] | select(.type=="npm_package") | .sha256')

# Verify the package checksum
echo "$EXPECTED_SHA  $TARBALL_NAME" | sha256sum -c
```

**Step 4: Automated Verification Script**

You can create an automated verification script:

```bash
#!/bin/bash
# verify-release.sh

VERSION="$1"
# Replace OWNER and REPO with your repository details
MANIFEST_URL="https://github.com/OWNER/REPO/releases/download/v${VERSION}/release-manifest.json"

# Download manifest
echo "Downloading manifest for version $VERSION..."
curl -L "$MANIFEST_URL" -o "release-manifest-${VERSION}.json"

# Verify NPM package
echo "Verifying NPM package integrity..."
npm pack "@spreedly/react-native-checkout@${VERSION}"

TARBALL=$(ls spreedly-checkout-*.tgz)
EXPECTED_SHA=$(cat "release-manifest-${VERSION}.json" | jq -r '.assets[] | select(.type=="npm_package") | .sha256')

if echo "$EXPECTED_SHA  $TARBALL" | sha256sum -c --quiet; then
    echo "✅ Package integrity verified"
else
    echo "❌ Package integrity check FAILED"
    exit 1
fi

echo "Release $VERSION integrity verification complete"
```

Usage:

```bash
chmod +x verify-release.sh
./verify-release.sh 1.2.3
```

#### Integration with CI/CD

For automated environments, you can integrate manifest verification:

```yaml
# In your CI pipeline
- name: Verify Spreedly SDK Integrity
  run: |
    # Download manifest
    # Replace OWNER and REPO with your repository details
    curl -L https://github.com/OWNER/REPO/releases/download/v${{ env.SDK_VERSION }}/release-manifest.json \
      -o manifest.json

    # Verify package before installation
    npm pack @spreedly/react-native-checkout@${{ env.SDK_VERSION }}
    EXPECTED_SHA=$(jq -r '.assets[] | select(.type=="npm_package") | .sha256' manifest.json)
    TARBALL=$(ls spreedly-checkout-*.tgz)
    echo "$EXPECTED_SHA  $TARBALL" | sha256sum -c

    # Install verified package
    npm install "./$TARBALL"
```

#### Security Benefits

- **Supply Chain Security**: Verify packages haven't been tampered with
- **Audit Trail**: Complete record of what was published in each release
- **Compliance**: Meet security requirements for asset verification
- **Trust**: Cryptographic proof of release integrity

---

## Security Considerations

### Token Security

- ✅ **Use scoped tokens** with minimal permissions
- ✅ **Rotate tokens regularly** (quarterly)
- ✅ **Store tokens securely** in GitHub Secrets
- ❌ **Never commit tokens** to version control

### Package Security

- ✅ **Use private repositories** for sensitive code
- ✅ **Audit dependencies** regularly
- ✅ **Sign releases** with GPG keys

#### GPG Signing Implementation

Every release is cryptographically signed using GPG for enhanced security:

**🏷️ Git Tags**: All release tags are signed with `git tag -s`

- Provides cryptographic proof of release authenticity
- Verifiable with `git tag -v v1.2.3`
- Links release to authorized Spreedly developer

**📄 Release Manifest**: The `release-manifest.json` file is GPG-signed

- Creates detached signature file (`release-manifest.json.sig`)
- Proves manifest integrity and authenticity
- Prevents tampering with asset checksums

**🔑 Public Key**: Available in public-key.asc

- Used to verify all Spreedly releases
- Import once: `curl -L https://raw.githubusercontent.com/OWNER/REPO/main/public-key.asc | gpg --import`
- Replace OWNER and REPO with your repository details

**🔄 Fallback Behavior**: If GPG is not configured:

- Standard annotated tags are created (`git tag -a`)
- Manifest is generated without signature
- Release process continues normally
- Users can still verify SHA-256 checksums

**GPG Verification Process:**

```bash
# 1. Download release files
# Replace OWNER and REPO with your repository details
curl -L https://github.com/OWNER/REPO/releases/download/v1.2.3/release-manifest.json -o manifest.json
curl -L https://github.com/OWNER/REPO/releases/download/v1.2.3/release-manifest.json.sig -o manifest.sig

# 2. Import public key (first time only)
curl -L https://raw.githubusercontent.com/OWNER/REPO/main/public-key.asc | gpg --import

# 3. Verify GPG signature
gpg --verify manifest.sig manifest.json

# 4. Verify Git tag signature
git tag -v v1.2.3

# 5. Use verified manifest to check asset integrity
npm pack @spreedly/react-native-checkout@1.2.3
EXPECTED_SHA=$(jq -r '.assets[] | select(.type=="npm_package") | .sha256' manifest.json)
echo "$EXPECTED_SHA  *.tgz" | sha256sum -c
```

### Access Control

- **GitHub Packages**: Repository-based access control
- **Regular audits** of package access

---

_This guide ensures smooth private releases for the Spreedly React Native SDK via GitHub Packages._
