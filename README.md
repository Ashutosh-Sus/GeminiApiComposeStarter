# Gemini API Compose Starter

A Jetpack Compose chat UI backed by the Gemini API, rendering the conversation as a
scrolling list of Material 3 bubbles.

## Setup

### 1. Get a Gemini API key

Create a key in [Google AI Studio](https://aistudio.google.com/app/apikey).

### 2. Provide the key to the build

Pick one of the two options below — the build checks `local.properties` first, then
falls back to the `GEMINI_API_KEY` environment variable.

**Local development:** create `local.properties` in the project root (already
git-ignored; see `local.properties.example`) with:

```
GEMINI_API_KEY=your_api_key_here
```

**CI:** set a `GEMINI_API_KEY` repository secret / environment variable. No
`local.properties` file is needed on CI runners.

The key is never hardcoded in Kotlin, XML or a committed Gradle file — `app/build.gradle.kts`
reads it into `BuildConfig.GEMINI_API_KEY` at build time only.

## How the key is protected at rest

`BuildConfig.GEMINI_API_KEY` is the key's *build-time* source, but the app doesn't call
the Gemini API with that value directly. On first launch, `SecureApiKeyStore`
(`app/src/main/java/.../data/SecureApiKeyStore.kt`) seeds an `EncryptedSharedPreferences`
store whose master key is an AES-256-GCM key generated and held inside the Android
Keystore (`MasterKey.KeyScheme.AES256_GCM`). From then on, only ciphertext is persisted
on disk — both the preference value and its key name are encrypted. The key is decrypted
back into memory only at the moment `MainActivity` constructs the `GenerativeModel`; it
is never logged, toasted or displayed.

Release builds also set `isMinifyEnabled = true`, so R8 obfuscates the shipped APK and
the key isn't trivially readable from decompiled code (see `app/proguard-rules.pro` for
the keep rules this required for the Gemini SDK and Tink).

### Known limits

Client-side encryption raises the bar but can't fully hide a key from a determined
attacker with a rooted device or a debugger attached to the app's process — the key has
to exist in plaintext in memory at the moment it's used. A production app should instead
keep Gemini calls behind a backend proxy that holds the real key, and/or restrict the
client-side key with [Firebase App Check](https://firebase.google.com/docs/app-check)
so it's only usable from an attested build of this app.

## Running the app

```
./gradlew installDebug
```

(or open the project in Android Studio and run the `app` configuration).

## Tests

No automated unit or Compose UI tests are included in this submission.
