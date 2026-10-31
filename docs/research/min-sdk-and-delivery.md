# minSdk floor, Galaxy S25 test profile, and getting CI builds onto the phone

Researched 2026-10-28, with the distribution data, Android 17 status and Samsung pages re-checked 2026-10-29. Purpose: set Rouxtine's minSdk as low as the stack allows (the owner wants as many Android versions as possible), define a Galaxy S25 profile for Roborazzi, and choose a safe CI-to-phone pipeline with signing, Android developer verification and Samsung install rules covered. `[n]` refers to the numbered sources at the bottom. "Measured" means one of the throwaway Gradle or Robolectric probes in the cloud container [1]. A line marked "inference" or UNVERIFIED was not confirmed from a primary source.

## Summary

**Part A: minSdk and test device**

- **Set minSdk 24 (Android 7.0), compileSdk 37, targetSdk 37.** 24 is the hard floor of the pinned stack. material3 1.5.0-beta02 (M3 Expressive), navigation3 1.2.0, navigationevent 1.2.0 and DataStore 1.3.0 declare 24, and no library declares more [1][2][3][4][5]. The manifest merge fails at 23 and a debug APK builds at 24 [1].
- **24 reaches 99.4% of devices that use Google Play**, on Google's feed republished on 29 Oct 2026. 23 would add 0.3 points; 26 would lose 0.4 points [6].
- **23 is possible only by freezing on older pre-releases** (material3 1.5.0-alpha25 from July 2026, navigation3 1.2.0-alpha07) and never taking another AndroidX update, because every new release now declares 24 [1][7][8][9]. That is not worth 0.3 points. 21 and 22 are impossible: Compose 1.12.1 itself declares 23 [10].
- **AndroidX's default minSdk is 24, "updated at a yearly cadence"** to cover 99% of users [9]. Expect to raise Rouxtine's minSdk when the libraries move again (inference).
- **Supporting 24–25 costs little:** legacy launcher PNGs, the compat notification-channel API, core library desugaring, location permission plus the location toggle for smart-cube scans on Android 7–11, and no Robolectric screenshots below SDK 26 (§A4, §A6).
- **Galaxy S25:** 1080 × 2340 px [11], most likely 480 dpi, which gives 360 × 780 dp at the default display size (confirm with `adb shell wm density`) [12][13]. Roborazzi qualifier: `w360dp-h780dp-normal-long-notround-any-xxhdpi-keyshidden-nonav`, measured in Robolectric [1]. The S25 has been getting One UI 9 (Android 17, API 37) since 28 Sep 2026, so the owner's phone may already run API 37 [14][15].

**Part B: delivery**

- **Confirmed: a rolling GitHub pre-release tracked by Obtainium, with plain download as the fallback.** Release assets of a public repo download without login or zip. Actions artifacts need a signed-in user, and their REST download needs a token [16][17][18]. Obtainium v1.7.3 (25 Oct 2026) reads GitHub releases and updates apps silently when it did the first install [19][20][21]. Lawnchair's nightly uses this pattern [22][23].
- **Three settings make it work.** In Obtainium, turn on "Include prereleases" and "Use release date as version string", because a fixed `dev` tag never changes [20][24]. In CI, never cancel a running publish step [25]. Keep GitHub immutable releases off; they are off on nauroth/rouxtine (checked 2026-10-29) [26][27].
- **Confirmed: package id `io.github.nauroth.rouxtine`, debug builds on `io.github.nauroth.rouxtine.debug`.** Individuals need no domain or website for developer verification, and the id must never change once published [28][29].
- **Corrected: do not commit the debug keystore; keep it in Actions secrets.** With a public key, anyone can sign an APK that installs as an update of the owner's `.debug` app. From 2027, registering `.debug` with that key would tie anyone's APK to the owner's verified identity [30][31]. Setting it up takes four repo secrets, and fork builds fall back to the default debug key [32][33]. Committing is common (Element X, Organic Maps) [34][35], so it stays defensible if `.debug` is never registered with Google.
- **Confirmed, with hardening: the release key in Actions secrets.** Store it as *environment* secrets in a `release` environment that needs the owner's approval and only deploys `v*` tags [36]. Keep an offline backup: a lost key cannot sign updates or register the package [30][37].
- **Developer verification does not touch sideloading yet (October 2026).** Since 30 Sep 2026 it applies only to installs from seven participating stores in Brazil, Indonesia, Singapore and Thailand. The global rollout is set for 2027 with no date. ADB is exempt [37][38].
- **Before 2027, register the package names** in the Android Developer Console. A free limited-distribution account covers the owner's own phones; public distribution needs a full account ($25, photo ID, proof of address) or Play Console [39][40][41]. Register early: below 50 installs names go first come, first served, and a new name needs only a SHA-256 fingerprint [31][42].
- **Samsung Auto Blocker must be off.** It blocks every install and update from outside Play and Galaxy Store, including Obtainium's and adb over USB, and it is on by default on phones released with One UI 6.1.1 or later, which includes the S25 [43][44][45]. On One UI 8.5 and later, its option to turn itself back on after 30 minutes must stay off [46][47].

## Part A. minSdk

### A1. The floor of the pinned stack

The probe read the `AndroidManifest.xml` of all 105 AARs on the release runtime classpath of the stack pinned in android-ui-stack.md [1][48]. Ten declare 24, 69 declare 23, 8 declare 21, one declares 16 and 17 declare 14. Nothing is above 24.

| minSdk | Libraries at the pinned versions |
|---|---|
| 26 | `androidx.javascriptengine` 1.1.1, which roux-training-features.md mentions for sandboxed JS. Not in the pinned set; it would need a runtime gate [49] |
| **24** | material3 and material3-ripple 1.5.0-beta02; navigation3-runtime and -ui 1.2.0; navigationevent and navigationevent-compose 1.2.0; datastore, datastore-core, datastore-preferences and datastore-preferences-core 1.3.0 [1][2][3][4][5] |
| 23 | Compose ui, ui-graphics, ui-text, foundation, foundation-layout, runtime and animation 1.12.1; material3-adaptive 1.3.0; lifecycle 2.11.0; activity and activity-compose 1.13.0; core and core-ktx 1.19.1; graphics-shapes 1.1.0; room3-runtime 3.0.3 with androidx.sqlite 2.7.1; hilt-lifecycle-viewmodel-compose and hilt-navigation-compose 1.4.0; appcompat 1.8.0; savedstate 1.4.0; window 1.5.0; Vico compose and compose-m3 3.3.1 [1][10][50][51] |
| 21 or lower | core-splashscreen 1.2.0, profileinstaller 1.4.x, Kable 0.45.0 (kable-core and kable-default-permissions) and Roborazzi 1.77.0 declare 21; hilt-android 2.60.1 declares 14. kotlinx-coroutines-android 1.11.0 and kotlinx-serialization-json 1.12.0 are plain JARs and set no floor [1][52] |

Details:

- **navigationevent is at 24 only because of a pin.** android-ui-stack.md pins `navigationevent-compose:1.2.0` [48]. navigation3 1.2.0 itself asks only for navigationevent 1.1.1, which declares 23 [1][53]. The floor stays 24 without the pin, because of material3, navigation3 and DataStore.
- **The 24 floor is recent and unannounced.** The AAR manifests show it; the release notes of these libraries do not [1]. material3-android declared 21 at 1.5.0-alpha01, 23 from alpha02 to alpha25 (29 Jul 2026), and 24 from alpha26 (12 Aug 2026) onward [1][7]. navigation3 moved to 24 at 1.2.0-beta01, navigationevent at 1.2.0-alpha04 and datastore-core at 1.3.0-alpha11 [1]. The only release note found that announces such a change is emoji2's: "minSdk is now 24" [54].
- **Every next release is already 24.** Compose ui 1.13.0-beta02, lifecycle 2.12.0-alpha05, activity 1.14.0-alpha03, room3 3.1.0-alpha03 and navigation3 1.3.0-alpha03 all declare 24 [1][8]. AndroidX documents the policy: "AndroidX libraries have the default minSdk of 24", meant to cover "99% of Android users based on the Google Play Store check-in information", and "updated at a yearly cadence" [9].
- **compileSdk 37 is required.** The AAR metadata of Compose ui, foundation and animation 1.12.1, material3 1.5.0-beta02, navigation3 1.2.0, the lifecycle Compose artifacts 2.11.0, core 1.19.1, adaptive 1.3.0, the hilt Compose artifacts and Vico 3.3.1 says `minCompileSdk=37` and `minAndroidGradlePluginVersion=9.1.0` [1][2][10]. AGP 9.5 supports up to API 37 [55].
- **Probe result.** `processDebugMainManifest` fails at minSdk 21 and 23, for example "minSdkVersion 23 cannot be smaller than version 24 declared in library […]". Which library the error names depends on dependency order. At minSdk 24, `assembleDebug` with core library desugaring builds a 17.4 MB unminified APK with minSdkVersion 24, targetSdkVersion 37 and compileSdkVersion 37 [1].
- **Vico 3.3.1 is Compose Multiplatform.** On Android it depends on JetBrains Compose 1.12.0 artifacts (`org.jetbrains.compose.ui`, `foundation`, `runtime`) [56]. The probe found 13 extra JetBrains AARs on the classpath, all minSdk 23 [1]. Check APK size and duplicate classes when adopting it.

### A2. What each level costs

Reach is from the feed of 29 Oct 2026 (§A3).

| minSdk | Reach | What it takes | Verdict |
|---|---|---|---|
| 21–22 | 99.9% | Impossible: Compose 1.12.1 declares 23 [10] | No |
| 23 (Android 6.0) | 99.7% | Freeze material3 at 1.5.0-alpha25, navigation3 at 1.2.0-alpha07 or lower, navigationevent at 1.2.0-alpha03 or 1.1.x, and DataStore at 1.3.0-alpha10 or 1.2.1 [1][7]. alpha25 already has a public `MaterialExpressiveTheme` plus `ButtonGroup`, floating toolbars and `LoadingIndicator`, and needs Compose 1.12.0-beta01 or later [7]. With these versions the manifest merge passes at 23 (merge only; no APK was built) [1]. No later Compose, lifecycle, activity, room3 or navigation3 release can be taken [1][8]. Android 5–6 compile fully ahead of time at install, so baseline profiles do nothing there [57] | Not worth 0.3 points |
| **24 (Android 7.0)** | **99.4%** | The pinned stack as is, plus the gates in §A4. Robolectric screenshots only at SDK 26 and up (§A6) | **Recommended** |
| 26 (Android 8.0) | 99.0% | Simpler: java.time is native, adaptive icons and notification channels exist everywhere, Robolectric goldens can run at the minSdk, and `javascriptengine` works without a gate [49][58][59][60] | Loses Android 7.0 and 7.1 (0.4%). Against the owner's goal |

Two notes:

- `tools:overrideLibrary` makes the manifest merger accept a library with a higher minSdk and keep the app's lower value [61]. Nothing then checks at build time that the library's code avoids newer APIs, so it can crash below its floor (inference). Do not use it for core libraries such as Compose or material3.
- The stable material3 1.4.0 declares 21, but its `MaterialExpressiveTheme` is `internal`, and it has only the token files for ButtonGroup, FloatingToolbar and LoadingIndicator, not the components [62]. M3 Expressive therefore needs 1.5.x, which means 23 (frozen at alpha25) or 24.

### A3. Distribution data

The only official per-version data is the feed behind Android Studio's New Project wizard. It counts only devices that access Google Play. Google replaced it on 29 Oct 2026 (Last-Modified 06:05:07 GMT, ETag "7093ede"). The file has no collection date, and no press write-up of it was found yet [6].

| Android | API | Share | Reach of minSdk = this API |
|---|---|---|---|
| 5.1 | 22 | 0.2% | 99.9% |
| 6.0 | 23 | 0.3% | 99.7% |
| **7.0** | **24** | 0.2% | **99.4%** |
| 7.1 | 25 | 0.2% | 99.2% |
| 8.0 | 26 | 0.5% | 99.0% |
| 8.1 | 27 | 1.5% | 98.5% |
| 9 | 28 | 3.1% | 97.0% |
| 10 | 29 | 5.7% | 93.9% |
| 11 | 30 | 10.5% | 88.2% |
| 12 | 31 | 9.6% | 77.7% |
| 13 | 33 | 11.8% | 68.1% |
| 14 | 34 | 15.3% | 56.3% |
| 15 | 35 | 14.7% | 41.0% |
| 16 | 36 | 25.4% | 26.3% |
| 17 | 37 | 0.9% | 0.9% |

- The shares add up to 99.9%. Android 5.0 (API 21) is no longer listed, and there is no separate row for API 32 [6].
- The previous feed (Last-Modified 10 Dec 2025) was based on data from 1 Dec 2025, after an April 2025 update [63]. It gave 99.1% for minSdk 24, 98.3% for 26 and 69.2% for 31 [6]. Old versions keep shrinking.
- Dynamic colour (API 31+) now reaches 77.7% of devices [6].

### A4. Building for Android 7.0–7.1 and other gated features

| Feature | Needs | What Rouxtine does |
|---|---|---|
| Launcher icon | Adaptive icons from API 26 (`res/mipmap-anydpi-v26`); themed icons from 33; Android 16 QPR2 auto-themes apps without one [59][64] | Ship legacy PNG mipmaps beside the adaptive icon, plus a monochrome layer |
| Notifications | Channels from API 26; `setPriority()` for 7.1 and lower; `POST_NOTIFICATIONS` is a runtime permission from 33 [60][65] | Use `NotificationManagerCompat.createNotificationChannel`, which "doesn't do anything on older SDKs" [66]. Request the permission only on 33+ |
| Dynamic colour | API 31 (`Build.VERSION_CODES.S`); "If not, you should fall back to using a custom light or dark ColorScheme" [67] | Static light and dark schemes below 31. material3 maps the neutral palette differently on 31–33 than on 34+ [68] |
| Ripple | material3 draws ripples with the platform `RippleDrawable`; the sparkle ripple "is available on Android 12 and above" [67][68] | Accept a plain ripple below 31. Stretch overscroll is the same on every API level [67] |
| Blur and shaders | `Modifier.blur` is ignored below Android 12. The progressive blur in Compose 1.13 renders a uniform radius on 31+ and varying radii on 33+. AGSL `RuntimeShader` needs 33 [69][70] | Plain fallback for any blur or shader cube effect |
| Variable fonts | Compose ignores font variation axes below API 26; below 28 weight synthesis gives only normal or bold [71] | Expressive type falls back to the font's default instance on 7.x |
| Haptics | Compose maps feedback types through `HapticFeedbackConstantsCompat`, which substitutes older constants below 34, 30 and 27, or `NO_HAPTICS` [72] | Nothing to do |
| java.time | Added in API 26 [58]. kotlinx-datetime also needs desugaring below 26 [73] | Turn on desugaring from day one: `isCoreLibraryDesugaringEnabled = true` and `coreLibraryDesugaring("com.android.tools:desugar_jdk_libs:2.1.5")` (latest on Google Maven, 25 Feb 2025; the official snippet still shows 2.0.3) [74][75]. No AAR in the graph asks for it, but kotlinx-datetime-jvm 0.8.0 arrives transitively through Kable and Vico [1]. The probe builds with it [1]. `android.icu` formatting is also available from API 24 [76] |
| Desugaring limit | D8 cannot desugar `MethodHandle.invoke` or `invokeExact`; code that uses them needs minSdk 26 [74] | Only matters if a future dependency uses them; none does today [1] |
| Predictive back | navigationevent 1.2.0 registers a completion-only callback on API 33 and an animated callback on 34+; below 33 there is only the classic back press [77]. For apps targeting 36 on Android 16+, system back animations are on by default, `onBackPressed` is not called and `KEYCODE_BACK` is not dispatched [78][79] | Handle back through Navigation 3 and navigationevent only |
| Edge-to-edge | Targeting 35 makes apps edge-to-edge on Android 15; targeting 36 disables the opt-out [78][80]. `enableEdgeToEdge()` covers older versions; with 3-button navigation it draws a translucent scrim [81]. In activity 1.13.0, API 23–25 can switch status-bar icons but always paint a dark navigation-bar scrim; light navigation-bar icons need 26, cutout short-edges mode 28, contrast enforcement 29 [82] | Call `enableEdgeToEdge()` from day one. Test 3-button and gesture navigation |
| Per-app language | Platform support from API 33; below that, AppCompat 1.6.0+ `setApplicationLocales()`, and "the activity hosting your Compose UI must extend AppCompatActivity" [83]. appcompat 1.8.0 pulls in fragment, emoji2-views-helper and other View-era libraries [1][51] | If an in-app language picker is wanted before Android 13, use `AppCompatActivity`; otherwise `ComponentActivity` plus `generateLocaleConfig` |
| Splash screen | core-splashscreen 1.2.0 declares 21 [1]. "The system displays the same splash screen on all versions of Android" [84], except that below API 31 the icon cannot animate and an adaptive icon is cropped; the icon background needs `Theme.SplashScreen.IconBackground` [85] | Static icon below 31; put an animated vector in `res/drawable-v31` if wanted |
| Smart-cube BLE | API 31+: `BLUETOOTH_SCAN` (with `neverForLocation`, some beacons are filtered) and `BLUETOOTH_CONNECT` at runtime. API 30 and lower: `BLUETOOTH` and `BLUETOOTH_ADMIN` capped at 30, plus location as a runtime permission; scanning from a service on 29–30 also needs background location [86]. Kable checks coarse location up to 28, fine location on 29–30 and `BLUETOOTH_SCAN` on 31+ [87]. kable-default-permissions ships that manifest [52]. On API 30 and lower Kable refuses to scan when the system location toggle is off [88] | On Android 7–11, the pairing screen asks for location permission and prompts the user to turn location on. Companion device pairing (API 26+) avoids the permission but "requires Location Services to be enabled", and does not exist on 7.x [89] |
| Baseline profiles | On 7.0–8.1, `profileinstaller` installs them on first run; on 9+, Play also uses them at install, plus Cloud Profiles [57] | Ship a baseline profile; it helps from API 24 up |

### A5. targetSdk 37 and Android 17

- **Play rule.** From 31 Aug 2026, "New apps and app updates must target Android 16 (API level 36) or higher", with an optional extension to 1 Nov 2026. No 2027 requirement is published yet [90][91]. targetSdk 37 is ahead of the rule.
- **Android 17 is released.** source.android.com's June 2026 entry says "Android 17 has been released" [92]. It reached platform stability at Beta 3 (26 Mar 2026) [93]. QPR1 (API 37.1) and a QPR2 beta (37.2, Beta 7.1 on 16 Oct 2026) exist [94]. Minor versions cannot be targeted or required [95]; new minor APIs are checked with `Build.VERSION.SDK_INT_FULL` [96].
- **Changes for apps targeting 37 that touch Rouxtine** [97]:
  - Orientation, resizability and aspect-ratio restrictions are ignored on screens 600dp and wider, and the Android 16 opt-out is gone. Rouxtine must lay out on tablets, foldables and DeX windows.
  - `ACCESS_LOCAL_NETWORK` is a runtime permission (only matters for LAN features).
  - Background audio is hardened; audio from the background needs a while-in-use foreground service or the alarm path.
  - `MessageQueue` is lock-free, which breaks code that reflects on it. Static final fields can no longer be changed through reflection or JNI.
  - Certificate transparency is on by default. RFCOMM socket `read()` returns -1 on close.
  - Home-screen widgets get a RemoteViews bitmap memory budget and throw `IllegalArgumentException` over it.
- **Changes for all apps on Android 17** [98]: memory limits based on device RAM (look for "MemoryLimiter:AnonSwap" in `ApplicationExitInfo`; relevant to solver pruning tables), and automatic Bluetooth re-pairing after bond loss (`EXTRA_PAIRING_CONTEXT`; only matters for cubes that bond).

### A6. Testing across API levels

- **Robolectric range.** Robolectric 4.17 knows SDK 23 to 37 [99] and runs tests against the targetSdk by default [100]. A test cannot run below the app's minSdk: `@Config(sdk = [23])` on a minSdk 24 app fails with "Requires newer sdk version #24" [1]. Rouxtine's range is 24–37. SDK 34–35 need Java 17 and SDK 36–37 Java 21 [99]. Each SDK is a separate download of about 88 MB (7.0), 95 MB (8.0), 213 MB (16) and 239 MB (17) [1].
- **No screenshots below SDK 26.** Robolectric's native-graphics `Bitmap` shadow starts at API 26, and hardware-accelerated capture with real shadows needs 28 [101][102]. Measured: with `@GraphicsMode(NATIVE)`, drawing red on a bitmap gives a red pixel at SDK 26 and 37 but nothing at 24 and 25 [1]. Run Roborazzi goldens at 26 or later (28 or later for elevation and shadows).
- **Qualifier gotcha.** A full qualifier string containing `-notround-any-` crashes Robolectric at SDK 24–25 with `NoSuchFieldError ... colorMode`, because the parser reads that `any` as the wide-colour-gamut wildcard and `Configuration.colorMode` exists only from 26 [1][103]. 43 of the 52 Roborazzi device constants contain it, including every phone and tablet [1][104]. Use the short form at 24–25 (§A7).
- **JDK and network.** SDK 37 on JDK 21 needs `--add-opens=java.base/jdk.internal.access=ALL-UNNAMED` on the test JVM [1][105][106]. Maven Central returns HTTP 429 to the container, so point `robolectric.dependency.repo.url` and Gradle at Google's Maven Central mirror [100][106].
- **Plan.** Roborazzi goldens at SDK 37 (optionally 36) with the GalaxyS25, MediumPhone and one tablet qualifier, light and dark. Behaviour tests at `@Config(sdk = [24])` with the short qualifier. Before a release, run one real Android 7.x device or an API 24 emulator on the owner's PC; the container has no KVM [106].

### A7. Galaxy S25 device profile

- **Hardware.** 6.2-inch FHD+ Dynamic AMOLED 2X (6.0 inches with the rounded corners), "Super Smooth 120Hz refresh rate (1~120Hz)", Bluetooth 5.4; launched on Android 15 / One UI 7 [45]. 1080 × 2340 px, about 416 ppi [11].
- **Density.** Most likely 480 dpi (xxhdpi, factor 3.0), which gives 360 × 780 dp at the default display size. Two non-Samsung sources agree: yesviz lists a 360 × 780 viewport at "3 xxhdpi" [12], and Playwright's "Galaxy S24" (same 6.2-inch 1080 × 2340 panel) is 360 × 780 at scale factor 3 [13]. Samsung's own emulator skin needs a Samsung account login and was not read [11]. Confirm with `adb shell wm size` and `adb shell wm density`. Changing Screen zoom changes the dp size.
- **Software.** One UI 8 (Android 16, API 36) reached the S25 from 15 Sep 2025 [107]. Stable One UI 9 (Android 17, API 37) has been rolling out since 28 Sep 2026 in Korea and 5 Oct 2026 in the US [14][15]. The owner's phone may be on either.
- **Navigation.** Samsung ships "Buttons enabled by default"; gestures are under Settings > Display > Navigation bar [108][109]. With buttons, `enableEdgeToEdge()` draws a translucent scrim [81]. Test both modes on the phone.
- **Roborazzi qualifiers.** Roborazzi 1.77.0 has no Samsung constant; the closest built-in, MediumPhone (= Pixel7), is 411 × 914 dp at 420 dpi [104]. Define:

  ```kotlin
  const val GalaxyS25 = "w360dp-h780dp-normal-long-notround-any-xxhdpi-keyshidden-nonav"
  const val GalaxyS25Api24 = "w360dp-h780dp-normal-long-xxhdpi-keyshidden-nonav" // SDK 24-25 only
  ```

  Measured with Robolectric 4.17: both give 1080 × 2340 px, 480 dpi, 360 × 780 dp and smallest width 360 (the first at SDK 26 and 37, the second at 24 and 25) [1]. For dark mode, add `@Config(qualifiers = "+night")` on the test; a leading `+` overlays the class-level qualifiers [110].

### A8. Recommendation (Part A)

1. Set `minSdk = 24`, `compileSdk = 37`, `targetSdk = 37`. Update the "26" in android-ui-stack.md, cloud-toolchain.md and HANDOFF.md [48][111].
2. Turn on core library desugaring with `desugar_jdk_libs` 2.1.5 [75].
3. From the first screen: legacy launcher PNGs, `NotificationManagerCompat` channels, gates for dynamic colour, blur and AGSL with static fallbacks, and `enableEdgeToEdge()`.
4. For smart cubes on Android 7–11: location permission plus a "turn on location" prompt (§A4).
5. Add the GalaxyS25 qualifiers (§A7), goldens at SDK 37, behaviour tests at SDK 24, and one Android 7.x check before each public release.
6. Re-check `distributions.json` and the AndroidX default minSdk each year [6][9].

## Part B. Getting builds onto the phone

### B1. The owner's decisions, checked

| Decision | Verdict | Evidence |
|---|---|---|
| Package id `io.github.nauroth.rouxtine` | **Confirmed** | Only organizations must give a website for developer verification; individuals verify with ID and address [28][40]. Google: "Once you publish your app, you should never change the application ID" [29]. Registration links the name to a key, and a lost key cannot register it [31][37]. Not checked: whether the name is free (needs the Status API, §B5) |
| Debug builds on `io.github.nauroth.rouxtine.debug` | **Confirmed** | Google's own example uses `applicationIdSuffix = ".debug"`, so "both the debug and the release build" install side by side [112]. It is a separate package, so from 2027 it needs its own registration, the advanced flow or ADB (inference from [31][37]) |
| Rolling GitHub pre-release tracked by Obtainium | **Confirmed, with settings** | Obtainium reads the Releases API and skips pre-releases unless "Include prereleases" is on [20]. It takes the tag as the version, so a fixed `dev` tag needs "Use release date as version string" [20][24]. Lawnchair Nightly is the working precedent: rolling `nightly` pre-release in CI, crowdsourced Obtainium config with exactly these two options [22][23]. Immutable releases must stay off [26] |
| Plain download as fallback | **Confirmed** | "Anyone with read access to a repository can view and compare releases" [16], which on a public repo means everyone. A pre-release cannot be "latest", so `/releases/latest/download/…` does not reach it [113], but `/releases/download/dev/<asset>` does if the asset name is constant [113] |
| Debug keystore committed to the repo | **Corrected: keep it in Actions secrets** | See §B4. The risks follow from documented rules: updates are allowed when certificates match [30], and registration ties a package and SHA-256 fingerprint to the owner's identity [31] |
| Release key in Actions secrets when public releases start | **Confirmed, with hardening** | Use environment secrets behind a required reviewer and a `v*` tag rule [36]; keep an offline backup [37]. See §B4 |

### B2. Options compared

| Option | How | Pros | Cons |
|---|---|---|---|
| A. Actions artifact, by hand | `upload-artifact@v7`; `archive: false` uploads a bare file (since v7.0.0) [17][114] | No setup. Fine for PR builds | Needs a GitHub login: "Users must be logged-in in order for this URL to work" [17][115]; the REST download needs a token even on public repos [18]. Kept 90 days by default, at most 90 on public repos, and since 1 Oct 2026 old run pages are deleted too [116]. nightly.link serves only zips [117]. No update notice |
| **B. Rolling `dev` pre-release + Obtainium (recommended)** | Each `main` build deletes and recreates the `dev` pre-release with one APK; Obtainium tracks it | No login, stable URL, update notices and silent background installs, free, all on GitHub [21] | Needs a stable signing key. No `dev` release for the short time between delete and publish, upload included; `gh release create` drafts, uploads, then publishes, not atomically [118]. Incompatible with immutable releases [26]. Auto Blocker must stay off (§B6). From 2027, registration, the advanced flow or ADB (§B5) |
| C. Unique pre-release tag per build + Obtainium | e.g. `v0.1.0-dev.123`, plus a job that prunes old ones | Tag is a real version; no tag moving; history kept | Clutters tags and releases; needs a pruning job. Obtainium still makes one API call per check, so it costs no extra rate limit [20] |
| D. Firebase App Distribution | CLI or Gradle plugin uploads; testers use the App Tester app [119][120] | Tester groups, emails, release notes | Testers sign in with Google; App Tester is installed from the browser [120]. Builds expire after 150 days; at most 500 testers per project [119][121]. The in-app alerts SDK is Beta [122]. APK builds are sideloads, so Auto Blocker and the 2027 rollout apply. AAB builds go through Play internal app sharing, which needs an app published on a Play track and re-signs with a test key [123]. The GitHub action wzieba/Firebase-Distribution-Github-Action has had no commit since 28 Mar 2025 [124] |
| E. Play internal testing | AAB to the internal track, up to 100 testers, available "within minutes" [125]. CI upload with r0adkll/upload-google-play (v1.1.5, maintained) [126] | Play Store installs and updates; Auto Blocker allows Play [43]. Can start "before completing app setup" [125]. New Play apps are registered for verification automatically [41] | $25 and ID; personal accounts must verify a device with the Play Console app [127]. New apps must upload AABs [128]. Production for a new personal account needs a closed test with 12 testers for 14 days [129]. Play App Signing can use your own key, so Play and GitHub builds can share one certificate [30] |
| F. Play internal app sharing | Upload an APK or AAB, share a link [130] | Any key, debuggable builds and reused version codes are allowed; up to 100 downloaders per link [130]. Installed by the Play Store, so it should pass Auto Blocker (inference from [43]; untested) | Needs Play Console. Re-signed with an Internal App Sharing key, so it cannot update GitHub-signed builds or the reverse. Links expire after 60 days. Testers turn on internal app sharing in the Play Store [130] |
| G. adb (USB or wireless) | `adb install -r` from a computer [131] | Exempt from developer verification, now and after 2027 [37] | Needs a computer. Auto Blocker turns USB debugging off [132]. No auto-update |

### B3. The pipeline in detail

> With the `play` and `foss` flavours (HANDOFF), CI builds both and publishes `fossDebug` from `app/build/outputs/apk/foss/debug/app-foss-debug.apk`. The workflow sketch below predates the flavours, and the owner generates the dev key as in §B8.

**Build type.** CI ships the `debug` build type (`.debug`), as the owner decided. Google advises judging Compose performance on a release build with R8, because debug mode "imposes a performance cost" [133]. When R8 keep rules exist, add a release-like build type for the phone, as Element X does with its `nightly` type: `initWith(release)` plus its own suffix and signing [34]. A different suffix installs as a second app.

**Signing (sketch, not built).** Gradle reads the dev key from environment variables and falls back to the default debug key when they are missing, as Lawnchair does [33]:

```kotlin
val devKeystore: String? = System.getenv("DEV_KEYSTORE")
android {
    defaultConfig {
        versionCode = (findProperty("versionCode") as String?)?.toInt() ?: 1
    }
    signingConfigs {
        if (devKeystore != null) create("dev") {
            storeFile = file(devKeystore)
            storePassword = System.getenv("DEV_STORE_PASSWORD")
            keyAlias = System.getenv("DEV_KEY_ALIAS")
            keyPassword = System.getenv("DEV_KEY_PASSWORD")
        }
    }
    buildTypes {
        getByName("debug") {
            applicationIdSuffix = ".debug"
            if (devKeystore != null) signingConfig = signingConfigs.getByName("dev")
        }
    }
}
```

**Version codes.** Android refuses only a *lower* versionCode (`INSTALL_FAILED_VERSION_DOWNGRADE`); an equal one installs [134]. Play rejects reused codes and caps them at 2100000000 [135]. `github.run_number` starts at 1, rises with each run and does not change on re-run [136], so pass it as `-PversionCode=…`. It is counted per workflow; keep an offset in the build script in case the workflow is ever replaced (inference). Use a versionName such as `0.x.y-dev.<run>+<sha7>`.

**Workflow sketch (`.github/workflows/dev.yml`, not run).**

```yaml
name: dev
on:
  push: { branches: [main] }
  workflow_dispatch:
jobs:
  dev:
    runs-on: ubuntu-latest
    permissions: { contents: write }
    concurrency: { group: dev-release, cancel-in-progress: false }
    steps:
      - uses: actions/checkout@v7
      - uses: actions/setup-java@v6
        with: { distribution: temurin, java-version: 21 }
      - uses: gradle/actions/setup-gradle@v6
      - name: Build, test, lint
        env:
          DEV_KEYSTORE_B64: ${{ secrets.DEV_KEYSTORE_B64 }}
          DEV_STORE_PASSWORD: ${{ secrets.DEV_STORE_PASSWORD }}
          DEV_KEY_ALIAS: ${{ secrets.DEV_KEY_ALIAS }}
          DEV_KEY_PASSWORD: ${{ secrets.DEV_KEY_PASSWORD }}
        run: |
          echo "$DEV_KEYSTORE_B64" | base64 --decode > "$RUNNER_TEMP/dev.jks"
          DEV_KEYSTORE="$RUNNER_TEMP/dev.jks" ./gradlew assembleDebug testDebugUnitTest lintDebug \
            -PversionCode=${{ github.run_number }}
          cp app/build/outputs/apk/debug/app-debug.apk rouxtine-dev.apk
      - name: Publish rolling pre-release
        env: { GH_TOKEN: ${{ github.token }} }
        run: |
          gh release delete dev --cleanup-tag --yes || true
          gh release create dev rouxtine-dev.apk --prerelease --target "$GITHUB_SHA" \
            --title "Rouxtine dev #${{ github.run_number }} (${GITHUB_SHA::7})" \
            --notes "$(git log -1 --format='%h %s')"
```

Why each piece:

- `gh release delete --cleanup-tag --yes` removes the release and its tag without a prompt [137]. `gh release create` would otherwise tag "the latest state of the default branch"; `--target "$GITHUB_SHA"` pins the build's commit [118]. Lawnchair's nightly runs the same delete-then-create pair [22].
- `concurrency` allows one running and one pending run per group; a newer push replaces the pending one [25]. `cancel-in-progress: true` could kill a run between delete and create and leave no `dev` release, so keep it `false` [25]. Lawnchair's nightly job has no concurrency group [22].
- A constant asset name gives the stable fallback URL `https://github.com/nauroth/rouxtine/releases/download/dev/rouxtine-dev.apk` [113]. GitHub renames asset names with special characters [138], and in a `gh` asset argument `#` starts a display label [139], so keep names plain. Run number and SHA go in the title.
- Write the notes yourself. Generated notes list merged pull requests and contributors [140], so direct pushes to `main` give near-empty notes, and GitHub does not document how it picks the previous tag once the rolling tag is deleted [140].
- Use the `gh` CLI. softprops/action-gh-release v3 and ncipollo/release-action v1.21.0 are maintained but do not move an existing tag (`target_commitish` / `commit` only apply when they create the tag) [113][141][142]. marvinpinto/action-automatic-releases has not changed since 29 Jan 2022 [143].
- The publish step needs `contents: write` [22][142].
- PR workflow: build and test only, then `upload-artifact@v7` with `archive: false` and `retention-days: 7` [17]. PRs from forks get no secrets, so their APKs carry a throwaway debug key and cannot update the owner's install [32][144].

**Obtainium config for the `dev` channel** (labels as in Obtainium's English strings [145]):

| Setting | Value | Why |
|---|---|---|
| App source URL | `https://github.com/nauroth/rouxtine` | GitHub source |
| Include prereleases | On | The only release is a pre-release [20] |
| Use release date as version string | On | The tag never changes. The release date becomes a pseudo-version [146]; it is the publish date, which changes on each delete-and-create [20][24] |
| Filter release titles by regex | `^Rouxtine dev` | Ignores future stable releases in the same repo |
| Filter links by regex | `rouxtine-dev\.apk$` | Picks the APK [145] |
| Verify the 'latest' tag | Off | It calls `/releases/latest` and fails on a pre-release-only repo [20] |

- Alternative: title each release with the versionName and turn on "Use release title as version string" instead [20]. Obtainium can then match the installed versionName.
- Only 7 of 378 crowdsourced Obtainium configs set `releaseDateAsVersion: true`; Lawnchair Nightly is the clear rolling-pre-release example [23].
- A README badge can import this config in one tap with `obtainium://app/<url-encoded JSON>`, using the same JSON shape as the crowdsourced configs; Obtainium shows a confirmation first [23][147].
- Silent updates: Obtainium's stock installer updates silently only if it is the installer of record, the phone runs Android 12+, and the app's targetSdk is at least the OS API minus 3 [21]. The platform rule is stricter: target 34+ on Android 16 and 35+ on Android 17 [148]. Rouxtine targets 37, so this holds. The first install must therefore come from Obtainium. Obtainium's Shizuku installer can install silently whoever installed the app [149].
- Unauthenticated GitHub API calls are limited to 60 per hour per IP; a token gets 5,000. The Obtainium wiki suggests a fine-grained token if limits hit [150][151].

### B4. Signing

Facts:

- Android allows an update only when the certificates match; otherwise the install fails with `INSTALL_FAILED_UPDATE_INCOMPATIBLE` [30][152]. APK Signature Scheme v3 key rotation (Android 9+) is the documented escape hatch for changing a key [153].
- AGP creates `~/.android/debug.keystore` if it is missing [30][144], and each GitHub-hosted runner is a new VM [154]. Without a fixed key, every CI build has a new key and cannot update the last one.
- The debug certificate is "insecure by design", Play refuses it, and it expires 30 years after creation. Google recommends 25 years or more for app keys; Play needs validity past 22 Oct 2033 [30].
- Secrets are not passed to workflows triggered from forks, except `GITHUB_TOKEN` [32]. Base64 is fine for small binary blobs but "is not a substitute for actual encryption" [32]. Secrets are limited to 48 KB [155]; a 4096-bit RSA PKCS12 keystore measured 4.3 KB (5.7 KB as base64) [1].
- Registering a new package name needs only the package name and the key's SHA-256 fingerprint; proof of the private key is needed only for existing names [31][42].

Options for the dev (`.debug`) key:

| Option | Precedent | Pros | Cons |
|---|---|---|---|
| Commit the keystore | Element X (`app/signature/debug.keystore`, password "android") [34]; Organic Maps (`android/app/debug.keystore`) [35] | Zero setup; local, CI and agent builds all update each other | Anyone can sign an APK that installs as an update of the owner's `.debug` app and reads its data (inference from [30]). From 2027, registering `.debug` with this key would vouch for anyone's APK under the owner's identity (inference from [31]), so `.debug` would have to stay unregistered (advanced flow or ADB only) |
| **Keep it in repo secrets (recommended)** | Lawnchair: base64 secret, fallback to the debug key when absent [22][33] | Only CI builds from this repo can update the installed app. `.debug` can be registered safely | Four secrets to set once. Local and agent builds use a different key, which does not matter because the phone only installs CI builds |

Release key (for `io.github.nauroth.rouxtine`, once public releases start):

- **Recommended:** a separate keystore, generated by the owner, stored as environment secrets of a `release` environment. Environment secrets reach only jobs that reference the environment, and with required reviewers "a job cannot access environment secrets until one of the required reviewers approves it". Deployment branch and tag rules can limit it to `v*` tags. On GitHub Free these features are available for public repositories [36].
- Keep an offline backup of the keystore and passwords. "If you lose your signing key you won't be able to register your packages" [37], and no update can be signed without it [30].
- Alternative: sign locally, as Obtainium's maintainer does; CI builds unsigned APKs and `sign.sh` signs them on the maintainer's machine, so the key never reaches CI [156]. Safer, but manual for every release.
- If Rouxtine goes to Play later, Play App Signing lets a new app upload its own app signing key instead of having Google generate one, so Play and GitHub builds can share a certificate [30].

### B5. Android developer verification

- **What it is.** "Android requires all apps to be registered by verified developers for users to install them on certified Android devices." Developers verify their identity, then register each package name with its signing key's SHA-256 [28][31].
- **Timeline.** August 2026: developer APIs, limited distribution accounts and the advanced flow launched. 30 Sep 2026: protections began for installs from participating stores (Google Play, HONOR App Market, OPPO App Market, Galaxy Store, Palm Store, V-Appstore, GetApps) in Brazil, Indonesia, Singapore and Thailand. 2027: "expanded globally for all apps on certified Android devices", no date [28][38][157].
- **Sideloading today.** "If users sideload your app directly, these new verification requirements won't apply to your app yet", but Google recommends verifying before the 2027 rollout [37]. Outside Play, enforcement "will only apply to mobile and tablet form factors in the selected regions" [37].
- **Which devices.** The developer FAQ says certified devices on Android 7+; the user Help Center says Android 8 and up and exempts AOSP and non-certified devices [37][158]. The S25 is covered either way; Samsung is a certified partner [159].
- **ADB.** "You are free to install apps without verification with ADB", and the 24-hour wait does not apply to ADB installs [37].
- **Advanced flow** (for unregistered apps): Settings > System > Developer options > "Allow apps from unverified developers"; a one-time 24-hour security delay; then on for 7 days or indefinitely; each install still warns and needs "Install anyway" [160]. No ADB command skips the wait [37]. Turning it off briefly gives a 10-minute grace period, and it governs updates too: with it off, updates to unregistered apps fail [37][161]. Samsung's menu path may differ (UNVERIFIED).
- **Accounts.**
  - *Limited distribution:* free, no government ID; needs 2-Step Verification, a Google payments profile and a contact email. It can register apps and share them with devices that users explicitly authorize. Google's pages disagree on the cap: the limited-distribution guide (updated 28 Oct 2026) says 50 devices, while the guides table and the FAQ say 20 [28][37][39]. Accounts can move from limited to full, not back [37].
  - *Full distribution:* $25; individuals give a government photo ID, proof of address, and an email and phone verified by one-time password [37][40].
  - *Play Console:* can also register apps distributed outside Play; new Play apps are registered automatically [41].
- **Name contention.** A key with over 50% of known installs wins; otherwise any key with 50 or more installs; below that, first come, first served [31].
- **Tools.** The Android Developer ID Status API (`androiddeveloperidstatus.googleapis.com`, API key, 1,000 requests a day) says whether a name is `REGISTERED`, `NOT_REGISTERED` or `REGISTERED_WITH_ANOTHER_CERTIFICATE_FINGERPRINT` [162]. The Android Developer Console API registers names and keys from CI [31].
- **Later stores.** If F-Droid or another repository re-signs the app, add its key to the registration; if its build passes 50 installs first, ownership must be proven with an `assets/adi-registration.properties` file [42].

What Rouxtine should do:

1. Now: nothing is required. Optionally check `io.github.nauroth.rouxtine` with the Status API.
2. Before the 2027 rollout: create an Android Developer Console account. For the owner's own phones, a limited-distribution account can register `io.github.nauroth.rouxtine.debug` with the dev key's SHA-256 and authorize the S25.
3. Before the first public release: upgrade to a full account (or use Play Console) and register `io.github.nauroth.rouxtine` with the release key's SHA-256. If community testers will track the `dev` channel after 2027, register `.debug` there too, or they will need the advanced flow.
4. Fallbacks at any time: the advanced flow or `adb install -r`.

### B6. Samsung gotchas

- **Auto Blocker** (One UI 6.0+) allows installs only "from official stores such as the Google Play Store and Galaxy Store" and "Blocks commands by USB cable" [43]. Installs from other sources are blocked "unless the feature is turned off" [163], and sideloaded apps "cannot be updated while the Auto Blocker feature is on" [44]. Samsung documents no per-installer exception, so it blocks installs and updates by Obtainium and by the browser alike.
- **It is likely on.** It was opt-in when announced in October 2023 [164]. Now, "For Galaxy devices released with One UI 6.1.1, the Auto Blocker feature is set to On by default", and devices that updated from older versions keep their setting [44]. The S25 launched on One UI 7 [45].
- **Turning it off:** Settings > Security and privacy > Auto Blocker [44]. A secondary source says it asks for biometrics or PIN [165].
- **One UI 8.5** added an optional "Turn on Auto blocker automatically 30 minutes after it is turned off" [46][47]. Leave it off, or background updates will fail again.
- **One UI 9** adds a Security Report of blocked installs, and its Maximum restrictions mode blocks USB connections entirely (seen in beta) [47].
- **USB debugging** is greyed out while Auto Blocker is on, per user reports [132]. No source says whether wireless debugging is affected.
- **Install unknown apps** is a per-app permission since Android 8.0 [166]. The browser used for the first APK and Obtainium each need it.

### B7. Steps on the Galaxy S25 (once)

1. Check which One UI and Android version the phone runs, under Settings > About phone > Software information (the screen that also holds Build number [167]).
2. Turn Auto Blocker off: Settings > Security and privacy > Auto Blocker > off, confirm with PIN or biometrics. On One UI 8.5 or 9, make sure the option that turns it back on after 30 minutes is off [44][46]. This lowers the phone's protection; keep "Install unknown apps" granted only to the browser and Obtainium.
3. In Chrome or Samsung Internet, open `https://github.com/ImranR98/Obtainium/releases`, download `app-arm64-v8a-release.apk` from v1.7.3 or later [19][156]. Open it; when Android asks, allow the browser to install unknown apps; install [166]. Obtainium needs Android 8.0+ [168]; it is also on IzzyOnDroid and F-Droid [169].
4. Open Obtainium. Allow notifications. Following Obtainium's own note, turn off battery optimisation for it so background checks run [145].
5. In Obtainium, tap Add app, paste `https://github.com/nauroth/rouxtine`, set the options in the §B3 table, tap Add, then Install. Allow Obtainium to install unknown apps when asked. (Once the README has an Obtainium badge, tapping it fills all of this in [147].)
6. Do the first Rouxtine install from Obtainium, not from the browser, so later updates install silently [21].
7. Optional: if Obtainium reports rate-limit errors, add a fine-grained GitHub token in its settings ("GitHub access token") [145][151].
8. Fallback: bookmark `https://github.com/nauroth/rouxtine/releases/download/dev/rouxtine-dev.apk` [113].
9. Optional, for adb: Settings > About phone > Software information > tap Build number seven times [167]; then Developer options > Wireless debugging, `adb pair`, `adb connect`, `adb install -r rouxtine-dev.apk` (Android 11+, same Wi-Fi as the PC) [131].
10. Measure the screen once with `adb shell wm size` and `adb shell wm density`, and update the GalaxyS25 qualifier if it is not 1080x2340 / 480.
11. In 2027, before enforcement reaches the owner's region: authorize the S25 in the limited-distribution account (§B5), or set up the advanced flow.

### B8. Steps for the owner on GitHub and a PC (once)

1. Generate the dev key on a PC with any JDK (tested 2026-10-29 with keytool [1]):

   ```sh
   keytool -genkeypair -v -keystore rouxtine-dev.jks -storetype PKCS12 -alias rouxtine-dev \
     -keyalg RSA -keysize 4096 -validity 10000 -dname "CN=Rouxtine dev"
   base64 -w 0 rouxtine-dev.jks > rouxtine-dev.jks.b64   # macOS: base64 -i rouxtine-dev.jks -o rouxtine-dev.jks.b64
   ```

   10,000 days is about 27 years, above Google's 25-year advice [30]. The base64 commands are from GitHub's docs [32].
2. In nauroth/rouxtine > Settings > Secrets and variables > Actions, add `DEV_KEYSTORE_B64`, `DEV_STORE_PASSWORD`, `DEV_KEY_ALIAS` (`rouxtine-dev`) and `DEV_KEY_PASSWORD` [32]. With PKCS12, keytool ignores a separate key password, so `DEV_KEY_PASSWORD` equals `DEV_STORE_PASSWORD` (tested [1]). Keep the `.jks` and passwords in a password manager. Do not commit them. (The session proxy blocks the GitHub Actions API, so an agent cannot set these.)
3. Leave immutable releases off: repo Settings, "Releases" section, "Enable release immutability" unticked [27][170].
4. When public releases start: generate a separate release keystore the same way; create a `release` environment with yourself as required reviewer and a `v*` tag rule; store the release key there as environment secrets [36].

## Sources

[1]: Probes run in the cloud container on 2026-10-28 and 2026-10-29: two independent throwaway Gradle 9.8.2 + AGP 9.5.0 projects with the pinned stack (manifest reads of the 105 AARs on releaseRuntimeClasspath, manifest merges at minSdk 21/23/24, assembleDebug at 24), and Robolectric 4.17 tests (SDK range, native-graphics pixel test, qualifier test). Scripts and logs lived in the session scratchpad and were not committed.
[2]: https://dl.google.com/dl/android/maven2/androidx/compose/material3/material3-android/1.5.0-beta02/material3-android-1.5.0-beta02.aar
[3]: https://dl.google.com/dl/android/maven2/androidx/navigation3/navigation3-runtime-android/1.2.0/navigation3-runtime-android-1.2.0.aar
[4]: https://dl.google.com/dl/android/maven2/androidx/navigationevent/navigationevent-android/1.2.0/navigationevent-android-1.2.0.aar
[5]: https://dl.google.com/dl/android/maven2/androidx/datastore/datastore-core-android/1.3.0/datastore-core-android-1.3.0.aar
[6]: https://dl.google.com/android/studio/metadata/distributions.json (fetched 2026-10-29; Last-Modified 29 Oct 2026 06:05:07 GMT, ETag "7093ede")
[7]: https://dl.google.com/dl/android/maven2/androidx/compose/material3/material3-android/1.5.0-alpha25/ (AAR, -sources.jar and .pom)
[8]: https://dl.google.com/dl/android/maven2/androidx/compose/ui/ui-android/1.13.0-beta02/ui-android-1.13.0-beta02.aar
[9]: https://developer.android.com/jetpack/androidx/versions ("Minimum SDK Version")
[10]: https://dl.google.com/dl/android/maven2/androidx/compose/ui/ui-android/1.12.1/ui-android-1.12.1.aar
[11]: https://developer.samsung.com/galaxy-emulator-skin/galaxy-s.html
[12]: https://yesviz.com/devices/samsung-s25/
[13]: https://github.com/microsoft/playwright/blob/main/packages/playwright-core/src/server/deviceDescriptorsSource.json ("Galaxy S24")
[14]: https://www.androidauthority.com/samsung-one-ui-9-galaxy-s25-fold-7-flip-7-stable-3716021/
[15]: https://www.androidauthority.com/samsung-one-ui-9-galaxy-s25-fold-7-flip-7-us-rollout-3719221/
[16]: https://docs.github.com/en/repositories/releasing-projects-on-github/about-releases
[17]: https://github.com/actions/upload-artifact/blob/cf430e030ddbb5b0abf93d22962f4752f3646cd9/README.md
[18]: https://docs.github.com/en/rest/actions/artifacts#download-an-artifact
[19]: https://github.com/ImranR98/Obtainium/releases (tags v1.7.0 to v1.7.3 read with git ls-remote on 2026-10-29)
[20]: https://github.com/ImranR98/Obtainium/blob/fcd1807c8f8d2b8a5e27d4214ac4d472ed045598/lib/app_sources/github.dart
[21]: https://github.com/ImranR98/Obtainium/blob/fcd1807c8f8d2b8a5e27d4214ac4d472ed045598/lib/installers/stock_installer.dart
[22]: https://github.com/LawnchairLauncher/lawnchair/blob/2d072998677ff198a0abc3f19c22af9ad32b5574/.github/workflows/ci.yml
[23]: https://github.com/ImranR98/apps.obtainium.imranr.dev/blob/f989b98cffe77f34230caac23f17144f45cfde8d/public/data/apps/complex/app.lawnchair.nightly.json
[24]: https://github.com/ImranR98/Obtainium/blob/fcd1807c8f8d2b8a5e27d4214ac4d472ed045598/lib/providers/source_provider.dart
[25]: https://docs.github.com/en/actions/how-tos/write-workflows/choose-when-workflows-run/control-workflow-concurrency
[26]: https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/immutable-releases
[27]: https://api.github.com/repos/nauroth/rouxtine/immutable-releases (read 2026-10-29: {"enabled":false}; the repo had 0 releases)
[28]: https://developer.android.com/developer-verification/guides
[29]: https://developer.android.com/build/configure-app-module
[30]: https://developer.android.com/studio/publish/app-signing
[31]: https://developer.android.com/developer-verification/guides/android-developer-console
[32]: https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/use-secrets
[33]: https://github.com/LawnchairLauncher/lawnchair/blob/2d072998677ff198a0abc3f19c22af9ad32b5574/build.gradle
[34]: https://github.com/element-hq/element-x-android/blob/046e1bce78c01af042dcc8bda93e0980fb984de3/app/build.gradle.kts
[35]: https://github.com/organicmaps/organicmaps/blob/1b9a011b4272d69b002ee78b9d30d2483e50688d/android/groovy/application-signing.gradle
[36]: https://docs.github.com/en/actions/reference/workflows-and-actions/deployments-and-environments
[37]: https://developer.android.com/developer-verification/guides/faq
[38]: https://developer.android.com/developer-verification
[39]: https://developer.android.com/developer-verification/guides/limited-distribution
[40]: https://developer.android.com/developer-verification/guides/full-distribution
[41]: https://developer.android.com/developer-verification/guides/google-play-console
[42]: https://developer.android.com/developer-verification/guides/open-source-app-registration
[43]: https://docs.samsungknox.com/admin/fundamentals/whitepaper/samsung-knox-mobile-security/system-security/samsung-auto-blocker/
[44]: https://www.samsung.com/au/support/mobile-devices/protect-your-galaxy-device-with-the-new-auto-blocker-feature/
[45]: https://news.samsung.com/global/samsung-galaxy-s25-series-sets-the-standard-of-ai-phone-as-a-true-ai-companion
[46]: https://www.androidauthority.com/one-ui-8-5-auto-blocker-auto-turn-on-apk-teardown-3599873/
[47]: https://www.androidauthority.com/samsung-one-ui-9-auto-blocker-update-3666893/
[48]: android-ui-stack.md (this folder), "Versions to pin"
[49]: https://dl.google.com/dl/android/maven2/androidx/javascriptengine/javascriptengine/1.1.1/javascriptengine-1.1.1.aar
[50]: https://dl.google.com/dl/android/maven2/androidx/hilt/hilt-lifecycle-viewmodel-compose/1.4.0/hilt-lifecycle-viewmodel-compose-1.4.0.aar
[51]: https://dl.google.com/dl/android/maven2/androidx/appcompat/appcompat/1.8.0/appcompat-1.8.0.aar
[52]: https://maven-central.storage-download.googleapis.com/maven2/com/juul/kable/kable-default-permissions/0.45.0/kable-default-permissions-0.45.0.aar
[53]: https://dl.google.com/dl/android/maven2/androidx/navigation3/navigation3-runtime-android/1.2.0/navigation3-runtime-android-1.2.0.module
[54]: https://developer.android.com/jetpack/androidx/releases/emoji2
[55]: https://developer.android.com/build/releases/agp-9-5-0-release-notes
[56]: https://maven-central.storage-download.googleapis.com/maven2/com/patrykandpatrick/vico/compose-android/3.3.1/compose-android-3.3.1.module
[57]: https://developer.android.com/topic/performance/baselineprofiles/overview
[58]: https://developer.android.com/reference/java/time/Instant
[59]: https://developer.android.com/about/versions/oreo/android-8.0
[60]: https://developer.android.com/develop/ui/views/notifications/channels
[61]: https://developer.android.com/build/manage-manifests ("Override uses-sdk for imported libraries")
[62]: https://dl.google.com/dl/android/maven2/androidx/compose/material3/material3-android/1.4.0/material3-android-1.4.0-sources.jar
[63]: https://9to5google.com/2026/01/30/android-16-is-on-7-5-of-devices-in-latest-distribution-numbers-update/
[64]: https://developer.android.com/develop/ui/views/launch/icon_design_adaptive
[65]: https://developer.android.com/develop/ui/views/notifications/notification-permission
[66]: https://developer.android.com/reference/androidx/core/app/NotificationManagerCompat
[67]: https://developer.android.com/develop/ui/compose/designsystems/material3
[68]: https://dl.google.com/dl/android/maven2/androidx/compose/material3/material3-android/1.5.0-beta02/material3-android-1.5.0-beta02-sources.jar (and material3-ripple-android-1.5.0-beta02-sources.jar)
[69]: https://developer.android.com/reference/kotlin/androidx/compose/ui/draw/blur.modifier
[70]: https://developer.android.com/develop/ui/views/graphics/agsl
[71]: https://dl.google.com/dl/android/maven2/androidx/compose/ui/ui-text-android/1.12.1/ui-text-android-1.12.1-sources.jar
[72]: https://dl.google.com/dl/android/maven2/androidx/core/core/1.19.1/core-1.19.1-sources.jar (HapticFeedbackConstantsCompat)
[73]: https://github.com/Kotlin/kotlinx-datetime/blob/master/README.md
[74]: https://developer.android.com/studio/write/java8-support
[75]: https://dl.google.com/dl/android/maven2/com/android/tools/desugar_jdk_libs/maven-metadata.xml
[76]: https://developer.android.com/guide/topics/resources/internationalization
[77]: https://dl.google.com/dl/android/maven2/androidx/navigationevent/navigationevent-android/1.2.0/navigationevent-android-1.2.0-sources.jar
[78]: https://developer.android.com/about/versions/16/behavior-changes-16
[79]: https://developer.android.com/guide/navigation/custom-back/predictive-back-gesture
[80]: https://developer.android.com/about/versions/15/behavior-changes-15
[81]: https://developer.android.com/develop/ui/compose/system/setup-e2e
[82]: https://dl.google.com/dl/android/maven2/androidx/activity/activity/1.13.0/activity-1.13.0-sources.jar (EdgeToEdge.kt)
[83]: https://developer.android.com/guide/topics/resources/app-languages
[84]: https://developer.android.com/develop/ui/views/launch/splash-screen/migrate
[85]: https://developer.android.com/reference/kotlin/androidx/core/splashscreen/SplashScreen
[86]: https://developer.android.com/develop/connectivity/bluetooth/bt-permissions
[87]: https://github.com/JuulLabs/kable/blob/0.45.0/kable-core/src/androidMain/kotlin/scan/requirements/ScanPermissions.kt
[88]: https://github.com/JuulLabs/kable/blob/0.45.0/kable-core/src/androidMain/kotlin/scan/requirements/LocationServicesEnabled.kt
[89]: https://developer.android.com/develop/connectivity/bluetooth/companion-device-pairing
[90]: https://developer.android.com/google/play/requirements/target-sdk
[91]: https://support.google.com/googleplay/android-developer/answer/11926878
[92]: https://source.android.com/docs/whatsnew/site-updates (June 2026, "Android 17 release")
[93]: https://developer.android.com/about/versions/17/release-notes
[94]: https://developer.android.com/about/versions/17/qpr2/release-notes
[95]: https://developer.android.com/guide/topics/manifest/uses-sdk-element
[96]: https://developer.android.com/reference/android/os/Build.VERSION
[97]: https://developer.android.com/about/versions/17/behavior-changes-17
[98]: https://developer.android.com/about/versions/17/behavior-changes-all
[99]: https://github.com/robolectric/robolectric/blob/robolectric-4.17/robolectric/src/main/java/org/robolectric/plugins/DefaultSdkProvider.java
[100]: https://robolectric.org/configuring/
[101]: https://github.com/robolectric/robolectric/blob/robolectric-4.17/shadows/framework/src/main/java/org/robolectric/shadows/ShadowNativeBitmap.java
[102]: https://github.com/robolectric/robolectric/blob/robolectric-4.17/shadows/framework/src/main/java/org/robolectric/shadows/HardwareRenderingScreenshot.java
[103]: https://github.com/robolectric/robolectric/blob/robolectric-4.17/shadows/framework/src/main/java/org/robolectric/android/QualifierParser.java
[104]: https://github.com/takahirom/roborazzi/blob/1.77.0/roborazzi/src/main/java/com/github/takahirom/roborazzi/RobolectricDeviceQualifiers.kt
[105]: https://github.com/robolectric/robolectric/blob/robolectric-4.17/sandbox/src/main/java/org/robolectric/interceptors/AndroidInterceptors.java
[106]: cloud-toolchain.md (this folder), "Gotchas and fixes" and "Limits and testing on the user's phone"
[107]: https://9to5google.com/2025/09/15/samsung-one-ui-8-galaxy-s25-stable-update/
[108]: https://www.samsung.com/uk/support/mobile-devices/optimised-navigation-bar-functions-on-galaxy-devices/
[109]: https://9to5google.com/2025/08/04/samsung-android-gesture-navigation-one-ui-8/
[110]: https://robolectric.org/device-configuration/
[111]: https://github.com/nauroth/rouxtine/blob/main/docs/HANDOFF.md ("Decided", "Open")
[112]: https://developer.android.com/build/build-variants
[113]: https://docs.github.com/en/rest/releases/releases
[114]: https://github.com/actions/upload-artifact/releases
[115]: https://docs.github.com/en/actions/how-tos/manage-workflow-runs/download-workflow-artifacts
[116]: https://docs.github.com/en/organizations/managing-organization-settings/configuring-the-retention-period-for-github-actions-artifacts-and-logs-in-your-organization
[117]: https://github.com/oprypin/nightly.link/blob/40f52e005caa271f22286b7479acb74d93ac3e46/src/nightly_link.cr
[118]: https://cli.github.com/manual/gh_release_create
[119]: https://firebase.google.com/docs/app-distribution/android/distribute-cli
[120]: https://firebase.google.com/docs/app-distribution/get-set-up-as-a-tester?platform=android
[121]: https://firebase.google.com/docs/app-distribution/troubleshooting
[122]: https://firebase.google.com/docs/app-distribution/android/set-up-alerts
[123]: https://firebase.google.com/docs/app-distribution/android/distribute-console?apptype=aab
[124]: https://github.com/wzieba/Firebase-Distribution-Github-Action/blob/master/README.md
[125]: https://support.google.com/googleplay/android-developer/answer/9845334?hl=en
[126]: https://github.com/r0adkll/upload-google-play
[127]: https://support.google.com/googleplay/android-developer/answer/6112435?hl=en
[128]: https://developer.android.com/guide/app-bundle
[129]: https://support.google.com/googleplay/android-developer/answer/14151465?hl=en
[130]: https://support.google.com/googleplay/android-developer/answer/9844679?hl=en
[131]: https://developer.android.com/tools/adb
[132]: https://forum.developer.samsung.com/t/s24-ultra-developer-option-usb-debugging-greyed-out/29526
[133]: https://developer.android.com/develop/ui/compose/performance
[134]: https://android.googlesource.com/platform/frameworks/base/+/refs/heads/main/services/core/java/com/android/server/pm/PackageManagerServiceUtils.java
[135]: https://developer.android.com/studio/publish/versioning
[136]: https://docs.github.com/en/actions/reference/workflows-and-actions/contexts
[137]: https://cli.github.com/manual/gh_release_delete
[138]: https://docs.github.com/en/rest/releases/assets#upload-a-release-asset
[139]: https://cli.github.com/manual/gh_release_upload
[140]: https://docs.github.com/en/repositories/releasing-projects-on-github/automatically-generated-release-notes
[141]: https://github.com/softprops/action-gh-release/blob/6680cc089e59314c4460b839412c25fb6a71ce3b/README.md
[142]: https://github.com/ncipollo/release-action/blob/1b9acf6ded5a3b65414f069bb0c2c847ac83a145/README.md
[143]: https://github.com/marvinpinto/action-automatic-releases
[144]: https://android.googlesource.com/platform/tools/base/+/refs/heads/mirror-goog-studio-main/build-system/gradle-core/src/main/java/com/android/build/gradle/internal/tasks/ValidateSigningTask.kt
[145]: https://github.com/ImranR98/Obtainium/blob/fcd1807c8f8d2b8a5e27d4214ac4d472ed045598/assets/translations/en.json
[146]: https://github.com/ImranR98/Obtainium-Wiki/blob/536cea30d538205c582223dc7ea1eaea9b0a3a84/pages/app_tracking.en.md
[147]: https://github.com/ImranR98/Obtainium-Wiki/blob/536cea30d538205c582223dc7ea1eaea9b0a3a84/pages/deep_links.en.md
[148]: https://developer.android.com/reference/android/content/pm/PackageInstaller.SessionParams#setRequireUserAction(int)
[149]: https://github.com/ImranR98/Obtainium/blob/fcd1807c8f8d2b8a5e27d4214ac4d472ed045598/lib/installers/shizuku_installer.dart
[150]: https://docs.github.com/en/rest/using-the-rest-api/rate-limits-for-the-rest-api
[151]: https://github.com/ImranR98/Obtainium-Wiki/blob/536cea30d538205c582223dc7ea1eaea9b0a3a84/pages/sources.en.md
[152]: https://android.googlesource.com/platform/frameworks/base/+/refs/heads/main/core/java/android/content/pm/PackageManager.java
[153]: https://source.android.com/docs/security/features/apksigning/v3
[154]: https://docs.github.com/en/actions/concepts/runners/github-hosted-runners
[155]: https://docs.github.com/en/actions/reference/security/secrets
[156]: https://github.com/ImranR98/Obtainium/blob/fcd1807c8f8d2b8a5e27d4214ac4d472ed045598/.github/workflows/release.yml (and sign.sh)
[157]: https://android-developers.googleblog.com/2026/06/android-developer-verification.html
[158]: https://support.google.com/android/answer/17065026?hl=en
[159]: https://www.android.com/certified/partners/
[160]: https://support.google.com/android/answer/17588095?hl=en
[161]: https://www.androidauthority.com/google-android-advanced-flow-sideloading-rollout-begins-3700073/
[162]: https://developer.android.com/developer-verification/guides/check-registration-status
[163]: https://www.samsung.com/ca/support/mobile-devices/protect-your-galaxy-device-with-the-new-auto-blocker-feature/
[164]: https://news.samsung.com/uk/protect-your-device-your-way-with-samsung-auto-blocker
[165]: https://innovation-village.com/how-to-turn-off-auto-blocker-on-samsung-galaxy-phones/
[166]: https://developer.android.com/about/versions/oreo/android-8.0-changes
[167]: https://developer.android.com/studio/debug/dev-options
[168]: https://github.com/ImranR98/Obtainium/blob/fcd1807c8f8d2b8a5e27d4214ac4d472ed045598/android/app/build.gradle.kts
[169]: https://github.com/ImranR98/Obtainium/blob/fcd1807c8f8d2b8a5e27d4214ac4d472ed045598/README.md
[170]: https://docs.github.com/en/code-security/how-tos/secure-your-supply-chain/establish-provenance-and-integrity/prevent-release-changes

## Open questions / unverified

- **S25 density.** The default 480 dpi / 360 × 780 dp comes from non-Samsung sources. Run `adb shell wm size` and `adb shell wm density` on the owner's phone.
- **S25 software.** Is the owner's phone on One UI 8 or 8.5 (Android 16, API 36) or One UI 9 (Android 17, API 37) [14]? Does One UI show Android's predictive-back animations, and how do Samsung's gesture settings interact with in-app back handling?
- **Robolectric and insets.** Robolectric screenshots probably do not draw system bars or real insets (not verified). Edge-to-edge on the S25 needs a device check.
- **AndroidX's next default minSdk.** The 29 Oct 2026 feed puts 26 at 99.0%, which would still meet the "99%" rule, so a yearly bump to 25 or 26 is plausible (inference). The timing is unknown.
- **Distribution feed.** The 29 Oct 2026 file has no collection date. Re-check it before each release.
- **Play's 2027 target-API rule** is not announced. Requiring API 37 by about August 2027 follows the usual pattern but is not confirmed.
- **Language picker before Android 13?** That decides `AppCompatActivity` versus `ComponentActivity`.
- **Android 7.x test device.** A used 7.0/7.1 phone, or an API 24 emulator on the owner's PC. Firebase Test Lab's device list was not readable here.
- **Verification dates and regions.** The exact 2027 date and regional order of the global rollout are unknown. The limited-distribution device cap is 50 or 20 depending on the Google page.
- **Advanced flow and Obtainium.** With the advanced flow on, can Obtainium still update an unregistered app silently, or does every update need "Install anyway"? Registering `.debug` avoids the question.
- **Obtainium's own registration.** If ImranR98 does not register `dev.imranr.obtainium` before 2027, updating Obtainium itself will need the advanced flow or ADB.
- **Advanced flow on Samsung.** Google gives the path Settings > System > Developer options; One UI may place it elsewhere. Not checked on a Samsung phone.
- **Auto Blocker.** No per-installer exception is documented. Does it affect wireless debugging? Does One UI 9 set it up differently at first boot?
- **Mixed installers.** If the owner installs a build from the browser fallback over an Obtainium install, does Obtainium stay the installer of record (and keep silent updates)? Not tested.
- **Play internal app sharing and Auto Blocker.** Expected to pass because the Play Store installs it, but not tested.
- **Package name free?** `io.github.nauroth.rouxtine` was not checked with the Status API (needs a Google Cloud API key).
- **Release-like dev build.** Whether to move the phone channel from the debuggable `.debug` build to an R8 build type once keep rules exist.
- **Access limits in this session.** The proxy blocked the GitHub Actions API (repo Actions settings were not read), and api.github.com and github.com pages for repos outside the session, so release `published_at` dates of third-party actions and Obtainium were taken from git tag commits.
