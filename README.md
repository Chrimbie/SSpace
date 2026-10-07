SSpace for Android

An initial native Android app for focusing available connectivity on any chosen app. Choose an installed app with a launcher icon, select competing apps, and start a timed session. Selected competing apps' internet traffic is discarded locally; the focused app and unselected apps continue using their normal connection.

## What this version does

- Lists installed apps with launcher activities, including system apps, in the focus picker. SSpace itself is excluded.
- Opens the selected app from its launcher, without navigating it to a website. The active session and notification show the focused app's name.
- Reset settings ends any active session, restores internet access, clears app choices and traffic counters, and restores the default 25-minute duration. The picker returns to "Choose an app…" on the same screen so you can immediately select a new focus app. Android's VPN and notification permissions are retained.
- Select all / Deselect all controls apply to the displayed competing apps. The focused app, detected browsers, the Google app, the current WebView provider, and launcher apps sharing their UIDs are excluded. Previously saved selections are cleaned up when the app opens and when the focus changes. The previous browser choice migrates to the app picker. Selection controls are disabled during a focus session.
- Offers 15, 25, 45, and 60 minute sessions.
- Uses Android `VpnService` to capture only the selected competing packages, covering IPv4 and IPv6 traffic.
- Shows a countdown, blocked traffic attempts, and an ongoing notification with an End focus action.
- Restores access by closing the local VPN interface on stop, expiry, permission revocation, or service destruction.
- Revalidates packages at start and rejects an empty selection to avoid accidentally capturing all device traffic.
- Keeps preferences locally, with no server, analytics, packet storage, or remote VPN gateway.

This version **blocks selected apps during the session**, including their foreground traffic, notifications that depend on their own network requests, and sync. It does not distinguish their foreground and background activity, rate-limit apps, or set carrier/router bandwidth priorities. System apps are excluded from the selection list unless they are updated system apps.

The benefit is reduced competition on this phone. It cannot increase the physical connection speed, control other devices sharing Wi-Fi, combine mobile data and Wi-Fi, or guarantee faster page loads. Blocked-byte counts measure intercepted outgoing traffic attempts, including retries; they are not measured data savings. Shared Android services, push delivery, and system DNS may operate outside selected packages, so this is not a comprehensive privacy firewall.

## Build

Requires a JDK 17 or newer and Android SDK platform 35. Gradle 8.9 is pinned by the included wrapper; Android Gradle Plugin is 8.7.3. Android 8.0 (API 26) and newer are supported by the declared minimum; the current target is API 35.

Set `ANDROID_HOME` to your SDK, or add `sdk.dir` to an untracked `local.properties` file. Set `JAVA_HOME` to your installed JDK.

```powershell
.\gradlew.bat :app:assembleDebug :app:testDebugUnitTest :app:lintDebug
```

Debug APK: `app/build/outputs/apk/debug/app-debug.apk`.

Open this folder in Android Studio to edit or run the app. The initial UI uses native Android views and Java, with no web wrapper.

## Device verification still required

The build and pure policy tests do not validate actual packet filtering or Android/OEM service behavior. Before relying on this app:

1. Install the debug APK on a test phone. Choose an installed app and a competing app that can make fresh network requests. Test a browser and a non-browser (for example, music or video calling) as focus targets in separate runs.
2. Establish baseline behavior in both apps. Start a focus session and accept Android's VPN permission.
3. Verify the focused app and an unselected app can load fresh network content, while the selected competing app's fresh requests fail. Cached pages are not a reliable test. Check both IPv4 and IPv6, and TCP and UDP where available. Verify Select all excludes the focused app; end the session, change focus, and verify the new target is protected. Relaunch SSpace and check that the focus choice is remembered.
   Also test Select all with pages opened from the Google app and with a second installed browser. Both should stay connected. Apps with their own embedded WebViews still lose access when selected; deselect the host app if you need its pages.
4. Verify blocked traffic attempts increase. End focus using the app and then the notification in a separate run; confirm network access returns.
   Test Reset settings both during a session and while idle: expect no selected competing apps, no chosen focus app, a 25:00 timer, cleared counters, and restored connectivity. Choose a new focus app on the same screen, then relaunch to check that reset settings remain cleared until you make new selections.
5. Check timer expiry, screen-off behavior, rotation during VPN consent, VPN permission revocation, switching Wi-Fi/mobile data, and killing the process. Expiry uses elapsed time, but CPU suspension can delay cleanup until the process runs again.
6. Compare repeated page-load measurements with active competing downloads, with and without focus. Only claim performance gains supported by measurements.

SSpace occupies Android's single VPN slot, replacing an existing VPN when activated. It is not designed for always-on VPN or lockdown mode; 'Block connections without VPN' must be off because the focused app intentionally bypasses this local filter. While running, Android displays the VPN indicator. If the focused app relies on another app, leave that helper unselected. Store release, release signing, localization, and on-device validation remain future work.

## Platform references

- [Android VPN development and per-app routing](https://developer.android.com/develop/connectivity/vpn)
- [Foreground service types](https://developer.android.com/develop/background-work/services/fgs/service-types)
- [Android Gradle Plugin 8.7 compatibility](https://developer.android.com/build/releases/agp-8-7-0-release-notes)
