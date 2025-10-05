# Android 16KB Page Size Support Upgrade Guide

## Overview

This guide documents the changes made to support Android 15's 16KB page size requirement. Starting November 1, 2025, Google Play requires all apps to support 16KB page sizes.

## What Was Changed

### 1. Android Native Libraries Updated
- **Old version**: ONNX Runtime 1.15.1
- **New version**: ONNX Runtime 1.23.0
- **Location**: `android/src/main/jniLibs/`
- **Architectures supported**:
  - `arm64-v8a` (64-bit ARM)
  - `armeabi-v7a` (32-bit ARM)
  - `x86` (32-bit Intel)
  - `x86_64` (64-bit Intel)

All libraries have been verified to have **2^14 (16KB) alignment**, ensuring compatibility with Android 15+ devices.

### 2. Android Build Configuration Updated
- **Android Gradle Plugin**: Upgraded from 7.3.0 → 8.7.3
- **compileSdkVersion**: Upgraded from 33 → 35 (Android 15)
- **Gradle Wrapper**: Upgraded from 7.5 → 8.9
- **File**: `android/build.gradle`

### 3. iOS Dependency Updated
- **Old version**: onnxruntime-objc 1.15.1
- **New version**: onnxruntime-objc 1.23.0
- **File**: `ios/onnxruntime.podspec`

### 4. Plugin Version
- **New version**: 1.5.0 (from 1.4.1)
- **Files**: `pubspec.yaml`, `CHANGELOG.md`

## Verification

The libraries were verified using the included `verify_16kb_alignment.sh` script:

```bash
./verify_16kb_alignment.sh
```

All libraries show `align 2**14` for LOAD segments, confirming 16KB page size support.

## Testing Recommendations

### 1. Test on Android 15 Emulator with 16KB Page Size

Create an Android 15 emulator with 16KB page size:
```bash
# Create AVD with 16KB page size
avdmanager create avd -n android15_16kb -k "system-images;android-35;google_apis;x86_64" -d pixel_5
```

### 2. Verify Page Size on Device/Emulator

```bash
adb shell getconf PAGE_SIZE
# Should return: 16384
```

### 3. Run Example App

```bash
cd example
flutter run
```

Test all ONNX model inference functionality to ensure compatibility.

### 4. Check for Crashes

Monitor logcat for any segmentation faults or alignment-related crashes:
```bash
adb logcat | grep -E "(SIGSEGV|alignment|onnxruntime)"
```

## Breaking Changes

⚠️ **ONNX Runtime 1.23.0 may have API changes from 1.15.1**

While the Dart FFI bindings should remain mostly compatible, you should:
1. Test all model loading and inference operations
2. Check for any deprecated API usage
3. Review ONNX Runtime release notes: https://github.com/microsoft/onnxruntime/releases

## Helper Scripts

Two helper scripts have been added to assist with library management:

### `update_onnx_libs.sh`
Downloads and extracts ONNX Runtime Android libraries from Maven Central.
- Automatically backs up existing libraries
- Extracts native libraries from AAR
- Supports version configuration

### `verify_16kb_alignment.sh`
Verifies that native libraries have proper 16KB alignment.
- Checks all architectures
- Reports alignment for each library
- Identifies incompatible libraries

## Updating to Newer Versions

To update to a future version of ONNX Runtime:

1. Edit `update_onnx_libs.sh` and change `ONNX_VERSION`:
   ```bash
   ONNX_VERSION="1.24.0"  # or whatever version you need
   ```

2. Run the update script:
   ```bash
   ./update_onnx_libs.sh
   ```

3. Verify alignment:
   ```bash
   ./verify_16kb_alignment.sh
   ```

4. Update iOS podspec to match:
   ```ruby
   s.dependency 'onnxruntime-objc', '1.24.0'
   ```

5. Update version and changelog

## Additional Resources

- [Android 16KB Page Size Guide](https://developer.android.com/guide/practices/page-sizes)
- [ONNX Runtime Releases](https://github.com/microsoft/onnxruntime/releases)
- [ONNX Runtime Android Maven Repository](https://repo1.maven.org/maven2/com/microsoft/onnxruntime/onnxruntime-android/)

## Notes

- The 16KB page size requirement is **Android-specific** and primarily affects ARM64 devices
- macOS and Linux libraries remain at 1.15.1 (no 16KB requirement for these platforms)
- Windows libraries remain unchanged
- You can keep the backup libraries in `android/src/main/jniLibs_backup_*` for rollback if needed

## Troubleshooting

### Build Errors After Update

If you encounter build errors:
1. Clean the build: `flutter clean`
2. Delete build folders: `rm -rf android/build example/android/build`
3. Update dependencies: `flutter pub get`
4. Rebuild: `flutter build apk`

### Runtime Crashes on Android 15

If the app crashes on Android 15 devices:
1. Verify page size: `adb shell getconf PAGE_SIZE`
2. Check library alignment: `./verify_16kb_alignment.sh`
3. Review logcat for SIGSEGV errors
4. Ensure all dependencies also support 16KB pages

### Compatibility Issues

If you need to maintain compatibility with older ONNX Runtime versions:
1. Keep the backup libraries: `android/src/main/jniLibs_backup_*`
2. Consider maintaining separate branches for different versions
3. Test thoroughly with your specific models
