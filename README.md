# Scribe App

A writing platform for creators.

## EAS Build and Submit Setup

### Prerequisites

1. **Expo Account**
   - Create an account at [expo.dev](https://expo.dev)
   - Install EAS CLI: `npm install -g eas-cli`
   - Login to EAS: `eas login`

2. **EXPO_TOKEN for CI/CD**
   - Create an access token in your Expo account:
     - Go to [expo.dev](https://expo.dev)
     - Navigate to your account settings > Access Tokens
     - Create a new access token with a descriptive name (e.g., "GitHub CI")
   - Add it as a secret in your GitHub repository settings:
     - Go to Settings > Secrets > Actions
     - Add a new repository secret named `EXPO_TOKEN`
     - Paste your Expo token as the value
   - This token will be used in GitHub Actions workflow to authenticate with Expo services

3. **Google Play Console Account**
   - You need a Google Play Console account to publish your app to Google Play Store
   - Create an app in the Google Play Console
   - Set up an internal testing track

4. **Apple Developer Account**
   - Required for iOS app submission
   - Create an app in App Store Connect
   - Note your Apple Team ID and App Store Connect App ID

5. **Service Account Key (Android)**
   - Create a service account in Google Play Console with the following permissions:
     - Edit releases
     - Manage production releases
     - Manage testing track releases
   - Download the JSON key file and save it as `service-account-file.json` in the project root
   - Make sure to add this file to `.gitignore` to keep it secure

6. **App Signing (Android)**
   - EAS handles app signing by default
   - If you need to use your own keystore:
     ```bash
     # Generate a keystore
     keytool -genkeypair -v -keystore upload-keystore.jks -alias upload -keyalg RSA -keysize 2048 -validity 10000
     
     # Export certificate
     keytool -export -rfc -keystore upload-keystore.jks -alias upload -file upload_certificate.pem
     
     # For Google Play encryption (if needed)
     java -jar pepk.jar --keystore=upload-keystore.jks --alias=upload --output=encrypted-key.zip --include-cert --rsa-aes-encryption --encryption-key-path=/path/to/encryption_public_key.pem
     ```
   - Add keystore configuration to eas.json if using custom keystore:
     ```json
     "production": {
       "android": {
         "buildType": "app-bundle",
         "credentialsSource": "local",
         "keystorePath": "./upload-keystore.jks",
         "keystoreAlias": "upload"
       }
     }
     ```

### Building and Submitting

#### Manual Build and Submit

```bash
# Configure your project (first time only)
eas build:configure

# Build for Android (creates AAB file for Google Play)
eas build --platform android --profile production

# Build for iOS
eas build --platform ios --profile production

# Submit to Google Play
eas submit --platform android --latest

# Submit to App Store
eas submit --platform ios --latest
```

#### Automated via GitHub Actions

The GitHub workflow will automatically build and submit your app when you push to the main branch or when manually triggered.

To manually trigger a build:
1. Go to the "Actions" tab in your GitHub repository
2. Select the "Build and Submit" workflow
3. Click "Run workflow"
4. Choose the branch and click "Run workflow"

### Configuration Files

#### eas.json

This file contains the configuration for EAS Build and Submit. Key sections:

- `build`: Defines build profiles (development, preview, production)
  - The `production` profile for Android uses `buildType: "app-bundle"` to generate AAB files
- `submit`: Configures app submission settings for each platform
  - For Android, it specifies the service account key path and track

#### GitHub Workflow

The `.github/workflows/build-and-submit.yml` file defines the CI/CD pipeline that:
1. Builds the Android app as AAB
2. Builds the iOS app
3. Submits the Android app to Google Play
4. Submits the iOS app to App Store

Example workflow file:
```yaml
name: Build and Submit
on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  build-and-submit:
    runs-on: ubuntu-latest
    steps:
      - name: 🏗 Setup repo
        uses: actions/checkout@v3

      - name: 🏗 Setup Node
        uses: actions/setup-node@v3
        with:
          node-version: 18.x
          cache: npm

      - name: 🏗 Setup EAS
        uses: expo/expo-github-action@v8
        with:
          eas-version: latest
          token: ${{ secrets.EXPO_TOKEN }}

      - name: 📦 Install dependencies
        run: npm install

      - name: 🚀 Build Android app
        run: eas build --platform android --profile production --non-interactive

      - name: 🚀 Build iOS app
        run: eas build --platform ios --profile production --non-interactive

      - name: 📱 Submit to Google Play
        run: eas submit --platform android --latest --non-interactive

      - name: 📱 Submit to App Store
        run: eas submit --platform ios --latest --non-interactive
```

### Troubleshooting

- **Build Failures**: Check the EAS build logs for detailed error messages
- **Submission Failures**: Verify your service account key and app store credentials
- **Android App Bundle**: Make sure your app meets Google Play requirements
  - AAB is the required format for new apps on Google Play
  - EAS automatically generates AAB when using `buildType: "app-bundle"` in eas.json
- **iOS Submission**: Ensure your app meets App Store guidelines and has proper provisioning profiles
- **EXPO_TOKEN Issues**: If your CI/CD pipeline fails with authentication errors, check that your EXPO_TOKEN is valid and has the necessary permissions

For more detailed information, refer to the [EAS Build documentation](https://docs.expo.dev/build/introduction/).