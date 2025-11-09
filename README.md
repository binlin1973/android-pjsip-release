# android-pjsip

[![CI](https://github.com/binlin1973/android-pjsip/actions/workflows/ci.yml/badge.svg)](https://github.com/binlin1973/android-pjsip/actions/workflows/ci.yml)

A **commercial-ready SIP JNI library** with an integrated **Android test app**,  
built on an optimized version of **PJSIP** and designed for production-grade reliability.

---

##  Highlights

- **Full SIP feature set** – covers all core signaling flows (`REGISTER`, `INVITE`, `BYE`, `RE-INVITE`, etc.)
- **Optimized PJSIP core** – improved threading, media session handling, and registration logic
- **Proven stability** – passed continuous 30×24-hour stress and high-traffic tests
- **Adaptive network recovery** – automatically detects connectivity changes and quickly re-registers
- **Media stream watchdog** – detects remote media interruption and handles it gracefully (e.g., hangup)
- **STUN / TURN / ICE / Trickle-ICE** support – ensures NAT traversal in complex network environments
- **Aggressive nomination & ICE parameter tuning** for faster peer-to-peer connectivity
- **P2P media support** (with optional FreeSWITCH backend coordination)

---

##  Build Instructions (Local)

### Prerequisites
- Android SDK + NDK (tested with NDK r25+)
- Gradle 7.6 or newer
- Linux or macOS build environment

### Steps
cd ./pjsip_android/

./gradlew clean

./gradlew assembleDebug | tee build_debug.log

### The resulting APK can be found under:

./pjsip_android/app/build/outputs/apk/debug/app-debug.apk

You can install it directly to a device for testing:

### Exported SIP JNI Library

After each successful build, the compiled SIP JNI libraries are automatically exported to:

./pjsip_android/dist/jni/

For example:

./pjsip_android/dist/jni/debug/arm64-v8a/libsip_jni.so

./pjsip_android/dist/jni/debug/arm64-v8a/libc++_shared.so

or

./pjsip_android/dist/jni/release/arm64-v8a/libsip_jni.so

./pjsip_android/dist/jni/release/arm64-v8a/libc++_shared.so


You can copy these .so libraries into your own Android project’s JNI folder.

- **Developer API Guide:** See [API.md](./API.md) for a concise, production-ready programming interface:
  initialization, configuration, call control, network recovery, diagnostics, logging & anti-debug, and packaging.

---

##  Distribution

Only the built APK and this README are publicly released.
All source code and JNI logic remain private and proprietary.

If you wish to publish an open-source demo, create a public repo (e.g., android-pjsip-demo)
and upload only:

the APK binary

usage instructions

screenshots or test results

##  License

This repository and its JNI code are proprietary.  
Redistribution or modification of the JNI source code is **not permitted** without explicit written authorization from the copyright holder.

The prebuilt **PJSIP** and **cURL** libraries included in `third_party/`  
are distributed under their respective **open-source licenses** (GPL or commercial).  
Their copyright and licensing remain with their original authors.

Only the compiled **Demo APK** and documentation may be freely used for evaluation and testing purposes.
