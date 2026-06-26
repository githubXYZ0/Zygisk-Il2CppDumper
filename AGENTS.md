# AGENTS.md

## Cursor Cloud specific instructions

This repo is **Zygisk-Il2CppDumper**, a single Android Magisk/Zygisk native module
(C/C++ compiled via Gradle + NDK). It is **build-only**: there are no servers,
databases, or automated test suites, and no UI. "Running" the product means flashing
the built Magisk module ZIP onto a rooted Android device — which is not possible in
this VM. End-to-end verification here means producing the build artifact.

### Toolchain (already installed in the VM snapshot)
- **JDK 11** at `/usr/lib/jvm/java-11-openjdk-amd64` — required because Gradle 7.5 / AGP
  7.4.2 cannot run on the system default Java 21.
- **Android SDK** at `/opt/android-sdk` (platform `android-32`, `build-tools;32.0.0`,
  `ndk;25.2.9519653`, `cmake;3.22.1`).

### Non-obvious gotchas
- Gradle is pinned to Java 11 via `~/.gradle/gradle.properties`
  (`org.gradle.java.home=/usr/lib/jvm/java-11-openjdk-amd64`). Do **not** rely on
  `JAVA_HOME`; if you run Gradle and it picks Java 21 it will fail. This file lives in
  `$HOME` (snapshot), not the repo.
- The SDK path is provided to Gradle via `local.properties` (`sdk.dir=/opt/android-sdk`),
  which is gitignored. The update script recreates it on startup, so you normally don't
  need to touch it.
- The Android **`sdkmanager`** CLI itself requires Java 17+ (class file v61). If you ever
  need to install more SDK packages, run `sdkmanager` with Java 21
  (`JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64`), even though the Gradle build uses Java 11.

### Common commands (run from repo root)
- Build the module: `./gradlew :module:assembleRelease`
  - Output: flashable ZIP at `out/zygisk-il2cppdumper-v1.2.0-release.zip` plus the
    per-ABI `.so` files under `out/magisk_module_release/zygisk/`.
- Lint: `./gradlew :module:lintRelease`
- Clean: `./gradlew clean`
- To target a specific game (real usage), edit `GamePackageName` in
  `module/src/main/cpp/game.h` before building (CI does this via `sed`). This is a source
  edit — revert it if you don't intend to commit it.
