# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Guidelines

- **NEVER create .md files without asking first** - Always ask the user before creating any markdown documentation files
- **No unnecessary comments** - Avoid adding obvious or redundant comments in code. Code should be self-explanatory
- **Use Conventional Commits** - Follow the Conventional Commits specification for all commit messages (see global CLAUDE.md for format)
- **Do not sign commits as Claude** - Never add "Co-Authored-By: Claude" or similar signatures to commits

## Project Overview

Fork Syslor de [Reedyuk/blue-falcon](https://github.com/Reedyuk/blue-falcon).
Bluetooth Low Energy (BLE) Kotlin Multiplatform library supporting Android, iOS, macOS, Windows, and JavaScript.
Published privately on GitLab Packages under `com.syslor.bluefalcon`.

## Architecture

### Multiplatform Structure

The library follows Kotlin Multiplatform conventions with platform-specific implementations:

- **commonMain** (`library/src/commonMain/kotlin/dev/bluefalcon/`): Shared API definitions and interfaces
  - `BlueFalcon.kt`: Core API declared as `expect class` with platform implementations
  - `BlueFalconDelegate.kt`: Callback interface for BLE events (device discovery, connection, characteristic changes)
  - Model classes: `BluetoothPeripheral`, `BluetoothCharacteristic`, `BluetoothService`

- **androidMain** (`library/src/androidMain/kotlin/dev/bluefalcon/`): Android BLE implementation using Android Bluetooth APIs
  - Uses `BluetoothManager`, `BluetoothAdapter`, GATT callbacks
  - Requires `ApplicationContext` for system service access

- **nativeMain** (`library/src/nativeMain/kotlin/dev/bluefalcon/`): iOS/macOS implementation
  - Uses CoreBluetooth framework (`CBCentralManager`, `CBPeripheral`)
  - `BluetoothPeripheralManager.kt`: Manages peripheral state and delegates

- **jsMain** (`library/src/jsMain/kotlin/dev/bluefalcon/`): JavaScript/Web Bluetooth implementation
  - Uses Web Bluetooth API

- **windowsMain** (`library/src/windowsMain/kotlin/dev/bluefalcon/`): Windows 10 BLE implementation
  - Uses WinRT Bluetooth APIs via JNI

- **rpiMain** (`library/src/rpiMain/kotlin/dev/bluefalcon/`): Raspberry Pi implementation (disabled)

### Key Architectural Patterns

1. **expect/actual pattern**: Core `BlueFalcon` class defined as `expect class` in commonMain, with platform-specific `actual class` implementations
2. **Delegate pattern**: `BlueFalconDelegate` interface provides callbacks for BLE events across all platforms
3. **StateFlow**: Uses Kotlin Flow for reactive peripheral list updates (`peripherals: StateFlow<Set<BluetoothPeripheral>>`)
4. **Platform abstraction**: Platform-specific types wrapped in common models (e.g., `BluetoothPeripheral` wraps Android's `BluetoothDevice` and iOS's `CBPeripheral`)

## Build Commands

The project uses Gradle composite builds, allowing you to build everything from the root directory.

### Building from Root

```bash
# Build library only
./gradlew buildLibrary

# Build all examples
./gradlew buildAllExamples

# Build library and all examples
./gradlew buildAll

# Build specific examples
./gradlew buildComposeExample
./gradlew buildKotlinMPExample

# Clean all projects
./gradlew cleanAll
```

### Building the Library Directly

```bash
cd library
./gradlew build
```

This builds all platform targets (Android, iOS, macOS, JS, Windows).

### Running Tests

```bash
# Run all library tests from root
./gradlew testLibrary

# Or from library directory
cd library

# All tests across all platforms
./gradlew allTests

# Platform-specific tests
./gradlew iosSimulatorArm64Test
./gradlew iosX64Test
./gradlew macosArm64Test
./gradlew macosX64Test
./gradlew jsBrowserTest
./gradlew jsTest

# Android tests (requires connected device/emulator)
./gradlew connectedAndroidTest
./gradlew connectedDebugAndroidTest
```

### Verification

```bash
./gradlew check          # All checks
./gradlew lint           # Android lint
./gradlew lintFix        # Apply safe lint suggestions
```

### Publishing (GitLab Packages)

Publishing uses the standard `maven-publish` plugin targeting GitLab Packages.

Requires in `local.properties`:
- `gitlabProjectId` (or `CI_PROJECT_ID` env var in CI)
- `gitlabDeployUsername` / `gitlabDeployPassword` (Deploy Token with `write_package_registry`)

In CI/CD, credentials are automatic via `CI_JOB_TOKEN`.

```bash
cd library

# Publish all platforms to GitLab
./gradlew publishAllPublicationsToGitLabRepository

# Publish a specific platform
./gradlew publishAndroidReleasePublicationToGitLabRepository

# Local testing
./gradlew publishToMavenLocal
```

## Platform-Specific Notes

### Android
- Requires `ApplicationContext` passed to `BlueFalcon` constructor
- Uses Android BLE stack (`android.bluetooth.le.*`)
- Minimum SDK: 24, Target SDK: 33
- Requires runtime permissions (handled by throwing `PermissionException`)

### iOS/macOS
- Uses CoreBluetooth framework via Kotlin/Native interop
- `autoConnect` parameter ignored on iOS (not needed)
- Creates XCFramework for distribution

### JavaScript
- Compiled JS output at `library/build/js/packages/blue-falcon/`
- Uses Web Bluetooth API
- Packaged as npm module

## Common Development Patterns

### Adding a new BLE API method

1. Add method signature to `expect class BlueFalcon` in `commonMain/kotlin/dev/bluefalcon/BlueFalcon.kt`
2. Implement `actual` method in each platform's `BlueFalcon.kt` (androidMain, nativeMain, jsMain, windowsMain)
3. If callback needed, add to `BlueFalconDelegate` interface
4. Update delegate calls in platform implementations

### Handling platform differences

Use platform-specific source sets for divergent behavior. Common API stays in commonMain, platform quirks handled in actual implementations (e.g., iOS ignores `autoConnect`, Android uses `transportMethod`).

## Project Properties

Key properties in `library/gradle.properties`:
- `version`: 2.5.0
- `group`: com.syslor.bluefalcon
- `libraryName`: blue-falcon
- `kotlinx_coroutines_version`: 1.9.0

## Project Structure

The repository uses Gradle composite builds to manage the library and examples:

```
bluefalcon-root/
├── settings.gradle.kts    # Root configuration with includeBuild() for library and examples
├── build.gradle.kts       # Custom tasks for building from root
├── library/               # Main Blue-Falcon library (included build)
│   └── settings.gradle.kts
└── examples/
    ├── ComposeMultiplatform-Example/  # Compose MP example (included build)
    ├── KotlinMP-Example/              # Kotlin MP example (included build)
    ├── JS-Example/
    ├── MacOS-Example/
    └── RPI-Example/
```

Each included build maintains its own Gradle configuration while being accessible from the root.

## Git Workflow (fork)

```
master    ← clean, sync with upstream (Reedyuk/blue-falcon)
  ├── feature/*   ← for PRs to upstream
  └── syslor      ← master + GitLab publishing customizations
```

- **Contribute upstream**: branch from `master`, PR to Reedyuk
- **Publish to GitLab**: from `syslor` branch
- **Sync upstream**: `git checkout master && git pull origin master`, then `git checkout syslor && git rebase master`

## Remotes

- `origin`: https://github.com/Reedyuk/blue-falcon.git (upstream)
- `fork`: git@github.com:Vaxela/blue-falcon.git (GitHub fork)
