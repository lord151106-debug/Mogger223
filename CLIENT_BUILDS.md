# Aviator client builds

Aviator uses one fixed manager identity across client builds:

- App display name: `Aviator`
- Android application ID: `avi.aviatorcrashgametofly.casino`
- Launcher logo: the committed Aviator logo
- Kotlin/Java namespace: `me.weishu.kernelsu`

Do not edit the app name, application ID, logo, or workflow package variables for each client. Client-specific signing is selected through GitHub Environments.

## Configure signing once per client

Create a GitHub Environment for each signing identity under **Settings → Environments**. Use `aviator` for the default production identity and a separate environment name for each additional client, such as `client-a` or `client-b`.

Add these environment secrets with the exact names below:

- `KEYSTORE`: the keystore file encoded as one-line Base64 (the workflow expects the same format as the existing repository secret).
- `KEYSTORE_PASSWORD`: keystore password.
- `KEY_ALIAS`: signing key alias.
- `KEY_PASSWORD`: signing key password.

Keep the private keystore and passwords out of Git. Never put them in `client.properties`, workflow files, build logs, or artifacts.

## Build a client

1. Open **Actions → Build Manager → Run workflow**.
2. Choose the branch to build.
3. Enter the GitHub Environment name in the `client` field.
4. Run the workflow and download the `manager` artifact.

The workflow reads the selected environment's signing secrets, calculates that key's certificate size and SHA-256 fingerprint, and passes those values into the kernel-module build. The Android manager and kernel module are therefore built against the same selected signing certificate without manually editing package variables or YAML for each client.

Normal pushes use the `aviator` environment. Pull-request builds use a temporary test key and a `.pr` package ID so they don't reuse the production signing identity.

## Important compatibility rules

- Reuse the **same signing key for every future update to a given installation**. Losing or replacing that key can prevent in-place updates.
- The package ID is intentionally fixed. APKs signed with different keys but the same package ID generally cannot be installed side-by-side for the same Android user, and one cannot update the other normally. A user may need to uninstall the old APK, which can remove its app data.
- Because KernelSU verifies the manager's package identity and signing certificate, a different client key requires a matching kernel-module build. The workflow computes the certificate values; do not disable or bypass signature verification.
- A different APK signing key does **not** isolate kernel modules, root permissions, device state, or client data. Validate the actual isolation requirements separately.
- This build still uses the upstream KernelSU release API for in-app update checks. Change it only when you have a compatible Aviator release source and matching update process.

## Local manager-only build

For a local build, the Android signing plugin accepts `KEYSTORE_FILE`, `KEYSTORE_PASSWORD`, `KEY_ALIAS`, and `KEY_PASSWORD` as Gradle properties. Keep these in your user-level `~/.gradle/gradle.properties` or another secure local secret source, not in the repository.

From `manager/`:

```sh
./gradlew :app:assembleRelease
```

A manager-only APK build does not rebuild the kernel module. Use the full **Build Manager** workflow when the signing certificate changes and the matching kernel module is required.
