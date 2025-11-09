# android-pjsip — SIP JNI API

This document describes the **Java-side API** exposed by the JNI layer  
(backed by an optimized PJSIP core).  
All blocking or complex work is hidden behind a simple, robust interface.

> The compiled JNI libraries are exported to:  
> `./pjsip_android/dist/jni/<variant>/<abi>/libsip_jni.so`  
> `./pjsip_android/dist/jni/<variant>/<abi>/libc++_shared.so`

---

## ⚠️ Package Name Constraint

**Keep the Java class path unchanged:**  
`com.zhanglihow.sipnewtest.SipNative`

If you rename or move this class, JNI symbol binding will fail.  
(You may later switch to dynamic registration via `JNI_OnLoad` if needed.)

---

## Quick Start (Java/Kotlin)

```java
public final class SipNative {
    static { System.loadLibrary("sip_jni"); }

    // ===== Lifecycle =====
    public static native void myJniInit();    // optional: cache VM/ClassLoader, init logging/anti-debug
    public static native void init();         // async: initialize PJSIP stack
    public static native void NetworkCheck(); // async: re-create transports after network changes

    // ===== Configuration (sync, fast) =====
    public static native void setAccountConfig(
        String user,
        String domain,
        String registrar,
        String realm,
        String password
    );

    public static native void setTransportConfig(
        int enable_stun,       // 0/1
        String stunServer,     // e.g. "stun.l.google.com:19302"
        int enable_ip_rewrite, // 0/1
        int sig_trans_proto,   // Protocol.UDP/TCP/TLS
        int sig_trans_port,    // e.g. 5060
        int enable_ice,        // 0/1
        String turnServer,     // "turn:host:port" (optional)
        String turnUser,       // optional
        String turnPass,       // optional
        int enable_webrtc      // 0/1
    );

    // ===== Call Control (async) =====
    public static native int  makeCall(String dest); // returns DIAL_STARTED if enqueued
    public static native void answerCall();
    public static native void hangupCall();

    // ===== Diagnostics (sync) =====
    public static native String sipRtpIsAlive();
    public static native void   internal_check();

    // ===== PCM Bridging (optional) =====
    public static void onPcmFromSIP(byte[] pcm, int len) { /* app-side */ }
    public static void onPcmFromMic(byte[] pcm, int len) { /* app-side */ }

    // ===== Constants for readability =====
    public static final class Protocol {
        public static final int UDP = 1;
        public static final int TCP = 2;
        public static final int TLS = 3;
    }

    public static final class DialResult {
        public static final int DIAL_STARTED        = 0;
        public static final int DIAL_AFTER_HANGUP   = 1;
        public static final int DIAL_INVALID_DEST   = 2;
        public static final int DIAL_INTERNAL_ERROR = 3;
    }
}
```

---

## Minimal Usage

```java
SipNative.myJniInit();   // once per process
SipNative.init();        // async start

SipNative.setAccountConfig(
    "1001","192.168.1.10","192.168.1.10","*","1234"
);

SipNative.setTransportConfig(
    1, "stun.l.google.com:19302",
    1, 1, 5060,
    0, "", "", "",
    0
);

// A -> B
SipNative.makeCall("1002");
```

---

## Threading & Performance

- `init()`, `NetworkCheck()`, `makeCall()`, `answerCall()`, and `hangupCall()`  
  run on **background threads** (non-blocking for UI).  
- JNI internally registers threads to **PJLIB** as needed; you do not need to manage it.  
- Fast setters (`setAccountConfig`, `setTransportConfig`) are **synchronous** and lightweight.  
- PCM callbacks (`onPcmFromSIP`, `onPcmFromMic`) are invoked from background threads —  
  if you need to update UI, use `Handler` / `runOnUiThread()`.

---

## Network Changes & Recovery

The SIP core continuously monitors Android network connectivity changes  
(e.g. **Wi-Fi ↔ 4G**).  

Whenever a transition is detected, the stack will  
**automatically rebuild all transports and re-REGISTER**  
within **~30 seconds** after the network becomes available again.

This ensures seamless call recovery even under frequent network switches or short outages.

Additionally, upper-layer apps may **manually trigger** the same recovery process at any time  
by calling:

```java
SipNative.NetworkCheck();
```

This explicit call forces immediate re-initialization and registration refresh,  
useful in cases where connectivity changes are known in advance  
or when the app wants deterministic control over reconnection timing.

---

## Error & Return Values

- `makeCall()` returns an enum-like integer (`DIAL_STARTED`, `DIAL_INVALID_DEST`, …).  
  Detailed outcomes are delivered via Java-side call state callbacks.  
- `sipRtpIsAlive()` returns `"NO CALL"`, `"CALL MEDIA NOT READY"`, "Rx stream ACTIVE"`, `"Rx stream INACTIVE"`,or `"UNKNOWN"`  
  depending on RTP reception state — useful for detecting remote media interruption.

---

## Permissions

Ensure your `AndroidManifest.xml` includes the following:

```xml
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.RECORD_AUDIO"/>
<uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS"/>
<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>
```

Optionally, for Bluetooth or background audio:

```xml
<uses-permission android:name="android.permission.BLUETOOTH"/>
<uses-permission android:name="android.permission.FOREGROUND_SERVICE"/>
```

---

## ABI & Artifacts

Currently built and tested for **arm64-v8a**.  
The compiled native libraries are automatically copied to:

```
./pjsip_android/dist/jni/<variant>/<abi>/
```

Typical outputs:
```
dist/jni/debug/arm64-v8a/libsip_jni.so
dist/jni/debug/arm64-v8a/libc++_shared.so
```

To integrate into another app, copy the `.so` files to your own project’s  
`app/src/main/jniLibs/<abi>/` directory.

---

## Logging

- PJSIP logs are redirected to:  
  `/data/data/<your.package.name>/files/pjsip.log`
- Only accessible by the app itself; useful for runtime diagnostics.  
- Current version always enables file logging and anti-debug hardening.  
  (A runtime switch may be added in future releases.)

---

## ProGuard / R8 Configuration

To prevent JNI entry points from being obfuscated, add:

```pro
-keep class com.zhanglihow.sipnewtest.SipNative {
    public static *;
    native <methods>;
}
```

---

## Development Environment

- **Android NDK:** r25 or later  
- **Gradle:** 7.6 or later  
- **minSdk:** 24  
- **targetSdk:** 34  
- **JDK:** 11+ recommended  
- If you encounter Lint errors in CI, you may skip them via:  
  `./gradlew assembleDebug -x lintVitalDebug`

---

## Versioning & Compatibility

- Supports **STUN / TURN / ICE / Trickle-ICE**.  
- Optional **P2P media** mode (requires FreeSWITCH coordination).  
- Validated with 30×24-hour continuous stress and high-traffic tests.  
- Auto-recovery within ~30 s after connectivity restoration.  
- Compatible with Android 10 – 14.

---

## Example Initialization Order

1. `myJniInit()` — once per process  
2. `init()` — start PJSIP stack asynchronously  
3. `setAccountConfig()` + `setTransportConfig()`  
4. Wait until REGISTER success  
5. Use `makeCall()` / `answerCall()` / `hangupCall()` as needed  

---

## Notes

- All network and media recovery are handled inside JNI;  
  the app may still call `NetworkCheck()` manually if needed.  
- PCM bridging callbacks are optional — implement them only if you need  
  raw audio access or external recorder integration.  
- PJSIP and cURL binaries in `third_party/` are distributed  
  under their original open-source licenses (GPL / commercial).

---

**End of API.md**
