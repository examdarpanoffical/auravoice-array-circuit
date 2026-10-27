# AuraVoice Android APK

This project packages the existing published AuraVoice web preview inside an Android WebView. It does not convert the preview's simulated actions into native device features. An internet connection and a published HTTPS app URL are required. The live database continues to be accessed by the website through its existing SDK; no credentials or server secrets are bundled into the APK.

## Build a debug APK on your computer

Prerequisites: JDK 17, Gradle 8.13, and the Android SDK with platform 35 and its build tools installed. Configure `ANDROID_HOME` to your SDK directory, or set `sdk.dir` in the untracked `android/local.properties`. Accept the Android SDK licenses. Android Studio can manage the SDK installation.

Publish the website first. Use its actual HTTPS URL, not the editor URL or a local development server. From the repository root, run:

```sh
cd android
read -r -p 'Published AuraVoice HTTPS URL: ' AURAVOICE_URL
gradle -PauraVoiceUrl="$AURAVOICE_URL" :app:assembleDebug
```

The installable, debug-signed APK is written to `android/app/build/outputs/apk/debug/app-debug.apk` relative to the repository root. With USB debugging enabled on your phone, install it from inside the `android` directory:

```sh
adb install -r app/build/outputs/apk/debug/app-debug.apk
```

There is no checked-in Gradle wrapper. Use the specified installed Gradle version, or generate a wrapper locally with that version before importing this folder into Android Studio. This repository's editing environment cannot run Android builds; no APK binary has been generated or verified here.

## Release distribution

The current application ID, `auravoice.preview`, identifies this preview shell. Choose your permanent application ID before publishing, then retain it for updates. Increment `versionCode` for subsequent releases.

For a signed APK, open the `android` folder in Android Studio, configure the `auraVoiceUrl` Gradle property in your local Gradle user properties, and choose **Build → Generate Signed App Bundle or APK → APK**. Create or select your own signing key and keep it outside source control. A debug APK is for testing, not a production release. For Google Play, generate a signed app bundle instead and review current target-SDK and store requirements.

## Scope and device checks

- Android 8.0 or newer; internet permission only. No microphone, background listening, accessibility, overlay, camera, or device-admin permissions are requested.
- App navigation stays on the configured HTTPS origin. User-tapped HTTPS, email, and telephone links open in other apps. Insecure HTTP links are blocked. SSL errors are never bypassed.
- Retry is available after a main-page network or HTTP failure. Android Back navigates web history before exiting. Web debugging is enabled only in debug builds.
- Test startup, internal routes, legal links, rotation, system Back, offline/retry, and database access on a physical device before distributing.
- External-provider sign-in and automatic cross-origin redirects are not implemented in this shell. Test your authentication flow; production OAuth requires a deliberate browser-based authentication and app-link callback integration, not relaxing WebView origin checks.

## Native implementation still required

1. Voice commands: user-triggered speech recognition, runtime microphone consent, supported-intent routing, and unavailable-app handling.
2. Clap finder: audio detection, explicit opt-in microphone foreground service, persistent notification, battery testing, and optional flashlight/alarm controls.
3. Voice Lock: threat-model review and secure fallback. The preview PIN is not a device credential, and this WebView cannot replace Android's secure lock screen.

These features remain clearly labeled previews in the packaged website. No native capability is claimed by this APK shell.