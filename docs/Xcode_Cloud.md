# Xcode Cloud Setup for Example App (iOS)

Follow these steps to fix missing Pods `.xcconfig` / `XCFileList` errors and ship builds from Xcode Cloud.

## Workflow Configuration

- Workspace: `example/ios/SpreedlyCheckoutExample.xcworkspace`
- Scheme: `SpreedlyCheckoutExample`
- Platform: iOS
- Configuration: Release (or as needed)
- Post-clone script: `ci_scripts/ci_post_clone.sh`

## Environment Variables (Required)

Add as encrypted variables in the workflow:

- `GITHUB_USERNAME` – GitHub username with access to private pods
- `GITHUB_TOKEN` – Personal access token with `repo` scope

## What the Post-clone Script Does

`ci_scripts/ci_post_clone.sh` will:

1. Configure Git to authenticate to `https://github.com/spreedly/checkout-ios-package.git` using the above env vars (if provided).
2. Install Node dependencies at repo root and inside `example/`.
3. Install Bundler gems from `example/Gemfile`.
4. Run `bundle exec pod install` in `example/ios` and verify the workspace exists.

## Notes

- Always select the `.xcworkspace` in the build step. Using the `.xcodeproj` can cause the “Unable to open base configuration reference file Pods-\*.xcconfig” error.
- If switching branches or Pod versions, enable a Clean build in the workflow to clear Derived Data.
- For distribution, enable “Upload to App Store Connect” or integrate Fastlane from the `example` directory.
