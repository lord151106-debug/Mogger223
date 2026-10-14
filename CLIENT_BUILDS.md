# Client builds (aviator)

This repository can build separately branded APKs from one source tree. Root-manager behavior, signature verification, and kernel/userspace compatibility logic are not weakened by branding.

Default Aviator identity:

- Display name (`KSU_NAME`): `aviator`
- Android applicationId (`KSU_PACKAGE_NAME`): `avi.aviatorcrashgametofly.casino`

The Kotlin/Java namespace remains `me.weishu.kernelsu`. Do not rename source packages unless you also update userspace component class names such as `me.weishu.kernelsu.ui.MainActivity`.

## One-time setup

1. Copy `manager/client.properties.example` to `manager/client.properties`.
2. Edit only these values for the client:
   - `KSU_NAME`: launcher/app display name.
   - `KSU_PACKAGE_NAME`: unique Android application ID in reverse-DNS format. This must match the kernel `KSU_PACKAGE_NAME` and ksud `KSU_PACKAGE_NAME` used on the same device.
3. Place the client's approved launcher icon assets in the app's existing Android launcher/mipmap resources. The config file does not change image files automatically; keep a copy of the original assets before replacing them.
4. Keep `manager/client.properties`, `manager/sign.properties`, signing keys, and passwords out of Git.

The local file is ignored by Git. Gradle reads it automatically. You can override either value for a one-off build with Gradle project properties, for example:

```sh
./gradlew :app:assembleRelease -PKSU_NAME="aviator" -PKSU_PACKAGE_NAME="avi.aviatorcrashgametofly.casino"
```

Run Gradle from the `manager` directory. Confirm the resulting APK name and install it on a test device before delivery.

## Kernel and ksud must match the applicationId

Changing only the Android `applicationId` is not enough. The kernel crowns the manager by package name **and** APK v2 certificate hash:

- `kernel/Kbuild` `KSU_PACKAGE_NAME` (default now `avi.aviatorcrashgametofly.casino`)
- `userspace/ksud` `KSU_PACKAGE_NAME` (build.rs / `KSU_PACKAGE_NAME` env)
- `EXPECTED_SIZE` / `EXPECTED_HASH` in `kernel/Kbuild` remain the certificate check. They are **not** the applicationId. A custom-signed Aviator APK will not be recognized until you rebuild the kernel with **your** certificate size and SHA-256. Do not disable that check.

This APK cannot manage an unmodified upstream KernelSU kernel that still looks for `me.weishu.kernelsu`.

## Signing per client

The Android signing plugin already reads these Gradle project properties:

- `KEYSTORE_FILE`
- `KEYSTORE_PASSWORD`
- `KEY_ALIAS`
- `KEY_PASSWORD`

Keep the keystore outside the repository. For local builds, put secrets in your user-level Gradle properties file (`~/.gradle/gradle.properties`) or pass them through a secure CI secret mechanism. Do not commit passwords, keystores, private keys, or certificates containing private key material.

If a client must receive future updates as an in-place app update, preserve that client's signing key securely and reuse it for that client's later releases. A different key will normally prevent an update over the installed app; the user may need to uninstall first, which can remove app data. Use a separate key per client when you want separate signing ownership and can securely maintain each client's key. Never overwrite a client's key with another client's key.

Example local Gradle properties (replace paths and values locally; do not commit them):

```properties
KEYSTORE_FILE=/absolute/path/to/aviator-release.jks
KEYSTORE_PASSWORD=use-a-private-secret
KEY_ALIAS=aviator
KEY_PASSWORD=use-a-private-secret
```

## Keep client data separate

- A distinct package ID and signing key separate Android app identity and release signing; they do **not** by themselves isolate root access, kernel modules, or device data.
- This build does not add a central dashboard or telemetry. Do not add a server upload of device identifiers, root events, logs, or client data without an explicit, documented client requirement and consent.
- In-app manager updates still query `https://api.github.com/repos/tiann/KernelSU/releases/latest`. That is an upstream KernelSU URL, not an Aviator release. Point it at your own GitHub releases only after you host matching APKs there.
- Do not give a client a build signed with another client's private key. Validate each build on a spare/test device and verify the APK's package ID, signature, and launcher icon before distribution.

## Build

From `manager/`:

```sh
./gradlew :app:assembleRelease
```

The output is normally under `manager/app/build/outputs/apk/release/`. Exact output names can vary with the Gradle build configuration.
