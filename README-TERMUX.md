# Proximity Call Bypass — build instructions

## What this app actually does

- While a call is ringing or active, a foreground service tries to hold a
  `PowerManager` wake lock to keep the screen from turning off.
- It does **not** disable, override, or gain any access to the physical
  proximity sensor. There is no public Android API that lets a third-party
  app do that.
- The stock ColorOS Phone app manages its own proximity-based screen
  behavior at the system level during real calls, independent of this app.
  A normal, non-system app has no supported way to override that. In
  practice this means the bypass may have little or no effect on your Oppo
  A53 during an actual phone call — this is a real platform limitation, not
  a bug you can code around. It has the best chance of doing something
  useful in situations where nothing else (i.e. no system dialer) is
  already holding a competing proximity wake lock, e.g. certain VoIP flows.

## Bugs fixed in this version

1. `CallMonitorService` was calling the `registerReceiver(receiver, filter, flags:Int)`
   overload, which only exists on API 33+ (Android 13+). Your target device
   (Oppo A53, Android 12 / API 31) does not have that method — the
   original code would throw at runtime the moment the service started.
   Fixed to use `ContextCompat.registerReceiver(...)`, which works correctly
   all the way back to API 26.
2. Missing `POST_NOTIFICATIONS` runtime permission request/manifest entry,
   required on Android 13+ for the foreground-service notification to show.
   Added (harmless no-op on Android 12, but makes the app forward-compatible).

## Building the APK

This requires an actual Android SDK + Gradle + JDK 17+ environment. That is
not something that can be produced inside a text chat — you need to run the
build yourself, either:

**On a PC (fastest/most reliable):**
1. Install Android Studio (bundles SDK, Gradle, JDK).
2. Open this project folder, let it sync.
3. `Build > Build APK(s)`, or from a terminal in the project root:
   ```
   ./gradlew assembleDebug
   ```
4. Output: `app/build/outputs/apk/debug/app-debug.apk`
5. Copy that file to your phone (USB, cloud drive, email) and tap it to
   install (you'll need to allow "install unknown apps" for whichever app
   you use to open it).

**In Termux on an Android device with the SDK already set up:**
```
cd ProximityCallBypass
gradle assembleDebug
```
Same output path as above.

## Installing on the Oppo A53

- Settings > Apps > [the app you used to open the APK] > Install unknown apps > allow.
- After install, open the app, flip the switch, grant the phone-state
  (and, if prompted, notification) permission.
- ColorOS is aggressive about killing background services — you'll likely
  need to disable battery optimization for this app in
  Settings > Battery > App Battery Management, or it may get killed shortly
  after you leave the app.
