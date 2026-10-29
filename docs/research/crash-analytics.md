# Crash reporting and usage analytics: tools, consent and store rules

Researched 2026-10-29 to pick the tools that HANDOFF leaves open ("Which tool: open."), for both the `play` and `foss` builds, before any code exists. `[n]` refers to the numbered sources at the bottom. **This is research, not legal advice.** No lawyer was consulted. EUR-Lex returned an empty bot-check page through the session proxy, so the ePrivacy Directive and the GDPR are quoted here only as the EDPB and the Article 29 Working Party quote them [40][41][42].

**Context.** HANDOFF (decided 2026-10-29): crash reporting and analytics are wanted, for fixing bugs and improving the app. Opt-in, an open-source SDK where possible, plus Play's Android vitals. No advertising ID, nothing personal, everything declared in the Play Data safety form ([HANDOFF.md](../HANDOFF.md)). Other constraints from this repo:
- R1 puts no closed Google library in either flavour ([play-billing-gpl.md](play-billing-gpl.md) §9).
- No server of our own. Opt-in crash reporting and analytics through a third-party service are the one exception ([scope.md](../scope.md):118).
- minSdk 24. The `foss` build has no proprietary libraries (scope.md:33).
- The app has no accounts and works offline, so reports must wait on the device until there is a network.

## Summary

- **Answer:**
  - **Crashes: Android vitals, plus the Sentry Android SDK (`sentry-android-core` only) sending to Sentry's EU region on the free plan.** The same code goes in both flavours. GlitchTip's hosted EU plan is the fallback: it takes the same SDK, and only the DSN changes.
  - **Usage stats: a small in-house Kotlin client that posts anonymous events to PostHog Cloud EU.** No PostHog SDK.
  - **Firebase is out.** It is closed and needs onionhoney's GPL s.7 permission. F-Droid bans it, and Firebase Analytics adds advertising-ID permissions (§3.5).
- **Android vitals costs nothing and needs no SDK.** But it only counts Play installs, on certified devices, from users who agreed to share diagnostics with Google. GitHub and Obtainium installs never appear [1][2]. The `foss` build needs the SDK.
- **Sentry is the best fit for crashes.**
  - MIT SDK, minSdk 21, about 1.8 MB of artifacts before R8 [12].
  - Offline disk cache of 30 envelopes. Auto-init can be switched off, so nothing starts before consent [13][14].
  - The EU region stores event data in Frankfurt. Account and project metadata may stay in the US [17].
  - Free Developer plan: 1 user, 5k errors a month, 30-day lookback, data residency included [18].
- **Sentry 8.62.0's defaults are close to "nothing personal", but not quite there.**
  - It sends no IP address unless Data Collection is switched on [13][14].
  - It does put a random installation ID on every event and every session. It writes that ID to a file during start-up, before the app's own options apply [13][15].
  - Remove the ID in `beforeSend` and turn session tracking off, or declare "Device or other IDs".
- **Use PostHog through its capture API, not its SDK.**
  - The SDK would add about 2.5 MB of artifacts (OkHttp, Okio, Gson) [26].
  - It also autocaptures lifecycle events, screen views and deep links by default [26].
  - The capture API needs one POST with `api_key`, `event` and `distinct_id`. Adding `"$process_person_profile": false` keeps events anonymous [29].
  - Free tier: 1M events a month, and usage stops at the limit instead of billing. EU hosting is in Frankfurt, with IP capture off by default for new EU projects [28].
- **F-Droid accepts all of this if it stays opt-in and off by default.**
  - FLOSS binaries from Maven Central are allowed [21].
  - The Tracking anti-feature is not applied when reporting is opt-in, off by default, and backed by informed consent [22].
  - Element X ships opt-in PostHog and Sentry reporting on F-Droid, and AnkiDroid and NewPipe ship ACRA there, all with no anti-feature [38][10][11]. Older Tasks.org F-Droid builds were tagged Tracking while the generic build tracked by default [39].
- **EU law: consent before anything leaves the device.**
  - Under ePrivacy art. 5(3), an app sending information off the device is "gaining access", whether or not the information is personal data [40].
  - Analytics are not "strictly necessary" [41].
  - Consent must be opt-in, separate per purpose, and as easy to withdraw as to give [42].
  - If children are in the audience and the data is personal, GDPR art. 8 adds parental consent below an age between 13 and 16, set by each country [42].
- **Play Data safety:** declare Crash logs and Diagnostics as collected, optional and not shared. Add App interactions only if usage stats ship, and Device or other IDs only if an ID is sent [3]. Google does not say whether Android vitals itself must be declared (see "Open questions").
- **The owner needs to decide** which age groups to target on Play, where they live (for the consent age), and Sentry (source-available server) versus GlitchTip (open-source server, smaller quota). There are also a few UX defaults. See "Questions for the owner".

## 1. Recommendation

### 1.1 What ships

| | `play` | `foss` (GitHub, Obtainium; F-Droid later) |
|---|---|---|
| Android vitals | Always on. No code; Google's own opt-in [1] | Not available [1] |
| Crash reports | `io.sentry:sentry-android-core` 8.62.0, opt-in, Sentry EU | Same |
| Usage stats | In-house client → PostHog Cloud EU, opt-in | Same |
| Keys | Sentry DSN and PostHog project token in the repo (public by design, §1.6) | Same |

Both flavours get one code path and the same backends. The SDK is MIT and the usage client is Rouxtine's own GPL code, so the `foss` rule "no proprietary libraries" holds [12][21].

### 1.2 Crash reports (Sentry)

- Depend on `sentry-android-core`, not `io.sentry:sentry-android`. The meta-artifact also pulls `sentry-android-ndk` (native libraries) and `sentry-android-replay` [12].
- Create a new Sentry organisation in the **EU region**. The region can't be changed later; the only way is a new organisation [17].
- Turn auto-init off with the manifest meta-data `io.sentry.auto-init=false`. Call `SentryAndroid.init` only once crash reporting is allowed [13][14].
- Options:
  - Leave every `dataCollection` field unset. Setting any one of them switches user info on, and with it IP inference, unless `userInfo=false` is also set [13][14].
  - `attachScreenshot` and `attachViewHierarchy` stay false, which is the default [14].
  - `enableAutoSessionTracking=false`. Sessions carry the installation ID [13]. Android vitals already gives crash rates for Play installs [1].
  - A `beforeSend` that removes `event.user`, which holds the installation ID [13][15].
- Project settings: turn on "prevent storing IP addresses" and add an Advanced Data Scrubbing rule that removes geo data. Sentry extracts geo from the IP even with the first setting on [16]. The SDK sends no IP with these options [13], so this is a safety net.
- Release builds: keep R8 shrinking, add `-dontobfuscate` and keep line numbers. The source is public anyway. Traces are then readable on any backend without uploading mapping files. That sidesteps GlitchTip's open deobfuscation bugs [24] and its rule that uploads count against the event quota [23].
- Debug and CI builds never send.

### 1.3 Usage stats (in-house PostHog client)

- A small Kotlin client, roughly 200 lines (estimate, not measured). Events go into a Room table. A WorkManager job with a network constraint posts them to `https://eu.i.posthog.com/batch/` [29]. Nothing is queued before the user opts in.
- Send few events, built only from fixed values: module opened, session finished (with a duration bucket), template used (bundled template id). Never send text the user typed, such as a custom Rouxtine's name.
- Every event carries `"$process_person_profile": false` [29].
- No persistent ID by default. `distinct_id` is required [29], so use a random value held in memory for one app run. Use a stored, resettable ID only if the owner wants retention numbers, and then declare "Device or other IDs" [3].
- In PostHog, keep IP capture off (the default for new EU projects [28]) and check that GeoIP enrichment is off too (see "Open questions").

### 1.4 Consent UX (both flavours)

Write all of this text with the `humanizer` skill (scope.md:16, CLAUDE.md "Writing").

- **Nothing leaves the device before consent.** No SDK starts, nothing is queued for upload, and no ID is created.
- **Crash reports have three choices:** "Ask me after a crash" (default), "Send automatically" and "Off".
  - In "Ask" mode Sentry does not start. A small Rouxtine handler, chained in front of the system's default handler, writes the crash to a local file.
  - On the next launch a sheet asks: "Rouxtine closed unexpectedly. Send a crash report?" It offers Send, Don't send, "See what's sent" (the exact payload) and "Always send".
  - Send starts Sentry and submits the stored crash. Don't send deletes the file.
  - Why not start Sentry early with sending held back: `SentryAndroid.init` writes the installation-ID file before the app's own options apply [13].
  - Information used only inside the device is not "gaining access" under art. 5(3) until it leaves (EDPB 2/2023 para 44) [40]. The same guidelines treat "storage" as a separate criterion (paras 35-36) [40]. Whether a local crash file counts was not analysed (see "Open questions").
  - To prototype: how to hand a stored crash to Sentry with proper frames, for example a serialised `Throwable` with the plain text as fallback. An uncaught-exception handler does not see ANRs. In "Send automatically" mode Sentry's default ANR integration covers those [13], and Android vitals covers them for Play installs [1].
- **Usage stats get a separate switch, off by default.** Pre-ticked boxes are invalid, and consent must be separate per purpose [42].
  - Offer it once, on a "Help improve Rouxtine" card after the first finished session, not on the first screen. "No thanks" and "Turn on" get equal weight.
  - The card says what is sent, to whom (Sentry, PostHog), where (EU, Frankfurt) and for how long, and links the privacy policy.
- **Settings › Privacy holds the same controls.** Turning one off works at once (`Sentry.close()`, cancel the worker) and deletes queued events and any ID. Withdrawal must work through the same interface, free, and without lowering the service [42].
- **Keep a local consent record:** the choices, a timestamp and the policy version. Ask again only if the purposes change.
- **Children.** If the Play target audience includes under-16s, put a neutral age question ("Are you 16 or older?") in front of the usage-stats offer, or keep usage stats strictly non-personal. That covers GDPR art. 8 and Play's mixed-audience SDK rule [42][43].
- **Privacy policy on GitHub Pages:** who the controller is, the processors, the data list, retention, and how to ask for deletion (an email address). Every Play app needs a privacy policy link [3].

### 1.5 Play Data safety form

This mapping applies Play's definitions [3] to the setup above. It is an inference, not Google's wording.

| Data type [3] | Declare? | Notes |
|---|---|---|
| Crash logs ("stack traces") | Yes | Collected, optional, not shared, purpose Analytics, encrypted in transit |
| Diagnostics | Yes | Sentry sends device and OS context (manufacturer, model, memory) [15] |
| App interactions | Only if usage stats ship | Same answers |
| Device or other IDs | Only if an ID is sent | Sentry's installation ID goes on every event unless `beforeSend` removes it [13]. A stored PostHog `distinct_id` also counts |
| Approximate location | No, with the settings above | The SDK sends no IP [13]; scrub Sentry geo [16]; PostHog IP capture off [28] |

- "Collect" means sending data off the device, including through SDKs [3].
- Passing data to a service provider that processes it for the developer is not "sharing" [3].
- Data may be marked optional only if every user, in every region, can opt in or out [3]. The consent design above does that.
- Google's FAQ: diagnostics sent with a device identifier are still declared as "Diagnostics" [3].

### 1.6 Keys and F-Droid

- A Sentry DSN only allows submitting events. It gives no read access and is safe to publish [20]. PostHog's capture endpoints are public, use the project token, and return nothing sensitive [29].
- F-Droid "does not sign up for any API keys" and includes any keys it is given in both source and binary releases [21]. A key does not have to be committed upstream, though: the fdroiddata recipe can inject it, as Element X does for a MapTiler key and Tasks.org for a Mapbox key [38][39]. Committing both keys is the simplest choice and loses nothing, since they end up public either way.
- Anyone can copy a public key and send junk events. On the free plans that can only use up the month's quota: PostHog stops at the free limit [28], and Sentry does not accept events over quota [18].
- Keep the backend URLs in build config. If an F-Droid reviewer objects to sentry.io, the `foss` DSN can point at GlitchTip's EU plan without code changes [23].

## 2. Android vitals

- **No SDK and no code.** Data comes from the Android system and includes events SDKs never see, such as crashes before an SDK starts [1].
- **Who is counted:** only users who opted in to share usage and diagnostics data with Google, only certified devices, and only installs from Google Play [1][2]. GitHub, Obtainium and F-Droid installs never appear.
- **Freshness and retention:** updated daily. Play Console shows 90 days; the Play Developer Reporting API keeps three years [2].
- **Bad-behaviour thresholds:** user-perceived crash rate 1.09 % overall and 8 % on any one phone model; user-perceived ANR rate 0.47 % overall and 8 % per model [1].
- **Data safety:** Google collects this data under the user's own setting ("Google Play collects this data") [1]. The Data safety form covers data "transmitted off a user's device from your app", including by libraries and SDKs [3]. So vitals probably needs no entry and no in-app consent. No Google page says so outright; treat it as an assumption.

## 3. The candidates

Added artifact sizes are sums of the files a POM resolver pulled from Maven Central and Google Maven on 2026-10-29, beyond what a Compose + DataStore app already ships. They are not R8 APK deltas, which will be much smaller [12][26][36][37][30][31].

| SDK | Version | Licence | minSdk | Added before R8 | Offline queue | Free backend | Verdict |
|---|---|---|---|---|---|---|---|
| ACRA core + http | 5.14.2 | Apache-2.0 | 14 | 0.37 MB | Yes, report files [6] | None hosted for free [7] | Fallback for user-sent reports (§3.4) |
| Sentry `sentry-android-core` | 8.62.0 | MIT | 21 | 1.81 MB | Yes, 30 envelopes [13][14] | Sentry EU, 5k errors/month [18]; GlitchTip EU, 1k events/month [23] | **Pick for crashes** |
| PostHog `posthog-android` | 3.73.1 | MIT | 23 | ≈2.5 MB | Yes, 1000 events [26] | PostHog EU, 1M events, 100k exceptions/month [28] | Use the capture API instead |
| Countly | 26.1.6 | MIT | 21 | 0.41 MB (0.44 MB with `multidex`) | Yes, 1000 requests [36] | None; hosted from $175/month [36] | Out |
| Bugsnag core | 6.28.0 | MIT (SaaS proprietary) | 16 | 0.65 MB | Yes, 32 events [37] | 7.5k events, 7-day retention [37] | Out |
| TelemetryDeck | 7.3.0 | MIT | 23 | ≈3.8 MB, OkHttp not counted | Yes, disk by default [33] | 50k events/month for new accounts [34] | Out |
| Aptabase | 0.1.0 (JitPack) | MIT SDK, AGPL server | 16 | not measured | **No** [35] | 20k events/month [35] | Out |
| Firebase Crashlytics | 21.0.1 | Apache-2.0 POM, closed deps | 23/24 | ≈2.8 MB | Yes [32] | Free [32] | Out (§3.5) |
| Firebase Analytics | 24.0.0 | Android SDK License | 24 | ≈6.6 MB | not checked | Free | Out (§3.5) |

The PostHog figure is a re-count from Maven Central file sizes: posthog-android 356,666 + posthog 634,957 + OkHttp 4.12.1 791,976 + okio-jvm 3.11.0 365,608 + Gson 2.10.1 283,367 + curtains 62,233 bytes = 2.49 MB [26]. The Crashlytics, Firebase Analytics and TelemetryDeck totals come from one resolver run and were not re-checked.

### 3.1 Sentry

- **Licence and size.** `sentry-android-core` 8.62.0 (2026-10-27) and its core jar `io.sentry:sentry` are MIT. It also pulls `epitaph` 0.1.1 (MIT). The AAR declares minSdk 21. The added files are the sentry jar (1.28 MB), the core AAR (0.50 MB) and epitaph (31 KB). Its lifecycle and androidx.core dependencies are already in a Compose app [12].
- **Offline.** The cache directory is `Context.getCacheDir()/sentry`, with `AndroidEnvelopeCache`. `maxCacheItems` defaults to 30, and the oldest envelope is deleted when the cache is full [13][14].
- **Consent hook.** `io.sentry.auto-init=false` in the manifest stops `SentryInitProvider` from starting the SDK. The app then calls `SentryAndroid.init` itself [13][14]. Apps can also replace the transport gate (`ITransportGate`). The SDK documents it as keeping events in the cache until sending is allowed again "or maximum number of retries to send an event has been reached" [13].
- **What it sends by default (8.62.0):**
  - **A random installation ID** whenever an event has no user ID. Data Collection options do not control it [15]. In the source, `DefaultAndroidEventProcessor.mergeUser()` sets `user.id` to `Installation.id(context)` [13].
  - **The same ID on sessions.** `options.distinctId` ("used for session tracking") defaults to the installation ID, and every session carries it [13].
  - **The ID file is written during start-up.** `readDefaultOptionValues` calls `Installation.id(context)`, which writes `filesDir/INSTALLATION`. This runs before the app's configuration callback, so app options cannot prevent it [13].
  - **Device and OS context** (manufacturer, model, memory and more) [15].
  - **No IP address by default.** The SDK adds `{{auto}}`, which asks the server to infer the IP, only when `isUserInfo()` is true. With no `dataCollection` field set, the legacy `sendDefaultPii` applies, and it defaults to false. Setting any one `dataCollection` field turns user info on unless `userInfo=false` is set too [13][14]. `sendDefaultPii` is deprecated [14].
  - `attachScreenshot` and `attachViewHierarchy` default to false [14].
- **Geo.** Sentry extracts geographic data from the user's IP "even if the setting to stop storing IP addresses is turned on". Removing it needs an Advanced Data Scrubbing rule [16].
- **Android 7.0-7.1.** For Sentry SaaS DSNs on API 25 and lower, the SDK trusts a bundled root certificate, because those versions "may lack ISRG Root X1 (Let's Encrypt)". Self-hosted DSNs keep the platform default [13]. This matters for minSdk 24 and for the in-house PostHog client (see "Open questions").
- **EU region.** Stored at rest in Frankfurt: error events, transactions, spans, logs, release health, releases, debug symbols, source maps and backups. These may be stored in the US: user accounts, organisation settings, project metadata, DSN keys, detailed usage data and audit logs, plus anything shared in a support ticket. The region can't be changed after the organisation is created [17].
- **Free Developer plan:** $0, 1 user, 5k errors, 50 replays, 30-day lookback, unlimited projects. The pricing table ticks "Data Residency" for every plan, the Developer plan included [18]. Events over quota "will not be accepted". Spike protection drops events in a sudden spike [18].
- **The server is not open source.** Sentry's server is FSL-1.1-Apache-2.0: source-available, and each version becomes Apache-2.0 two years after release. Sentry says "Fair Source is not under the OSI umbrella". Self-hosting needs 4 CPU cores, 16 GB RAM plus 16 GB swap, and 20 GB of disk [19]. That, and "no server of our own" (scope.md:118), rules self-hosting out.

### 3.2 GlitchTip and other Sentry-compatible backends

- **GlitchTip:**
  - The backend is MIT and "Sentry API compatible", so the same Sentry SDK works with it [23].
  - Hosted free "Hobby & Trial" plan: up to 1,000 events a month, 30-day retention. The cheapest paid plan is $15 a month for 100k events. Open-source discount plans start at $5 a month. Hosting is offered in the US and the EU (Germany) [23].
  - Over the limit, events are throttled step by step: about 10 % are dropped at the limit, and all of them at twice the limit [23].
  - Release and debug-file uploads count as one event per megabyte, so R8 mapping uploads eat into the free quota [23].
  - Self-hosting needs PostgreSQL 14 or newer and 512 MB RAM (256 MB minimum for the all-in-one setup) [23].
- **GlitchTip's R8 deobfuscation is unreliable today.** Three issues are open: #504 (2026-08-11, Android R8 traces not deobfuscated because of a missing `kind` field), #413 (2025-07-16, the Sentry Android Gradle plugin works only with mapping upload off) and #519 (2026-10-13, mappings over about 50 MB never deobfuscated) [24]. `-dontobfuscate` (§1.2) avoids all three.
- **Bugsink** is another Sentry-SDK-compatible backend. It runs in a single container on a small VM, with SQLite by default. Its licence is PolyForm Shield 1.0.0: source-available, not FLOSS [25].

### 3.3 PostHog

- **SDK.** `posthog-android` 3.73.1 (2026-10-27) and its core `com.posthog:posthog` 6.47.7 are MIT. The SDK supports API 23 and newer. It pulls Gson 2.10.1, OkHttp 4.12.x (through `okhttp-bom` 4.12.1), okio-jvm 3.11.0 and Square curtains 1.2.5 [26].
- **SDK behaviour.** Events are stored in a queue in device file storage and flushed when the device is online. When the queue is full, the oldest event is deleted first [26].
  - Defaults: `maxQueueSize` 1000, `flushAt` 20, `flushIntervalSeconds` 30.
  - `captureApplicationLifecycleEvents`, `captureScreenViews` and `captureDeepLinks` are all on.
  - `personProfiles` is `IDENTIFIED_ONLY`.
  - Consent: `optOut=true` in the config, plus `PostHog.optIn()` and `optOut()` [26].
- **Error tracking.** It needs `errorTrackingConfig.autoCapture=true` (default false) and supports R8 mapping upload. The docs say nothing about ANRs or native crashes [27]. That is why PostHog is not the crash tool here.
- **Cloud, free each month:** 1M analytics events, 100K error-tracking exceptions and 5K session recordings. It covers 1 project with 1-year data retention (recordings 1 month), needs no credit card, and "usage stops at the free tier limits" [28].
- **EU.** PostHog Cloud EU is hosted in Frankfurt. "IP data capture is automatically disabled by default for all new projects" there. A DPA under GDPR art. 28 is available [28].
- **Licence.** The server repository is MIT except the `ee/` directory [28].
- **Capture API.** The EU host is `https://eu.i.posthog.com`. `/i/v0/e/` takes single events and `/batch/` takes batches (request body under 20 MB). Every event needs `api_key`, `event` and `distinct_id`. `"$process_person_profile": false` processes the event without a person profile. Both endpoints are POST-only, public, and "do not return any sensitive data" [29].
- **Do not copy PostHog's age line.** Its GDPR page says "Children under 13 can only give consent with permission from their parent" [28]. GDPR art. 8 sets 16 by default, and member states can lower it to 13 at most [42].

### 3.4 ACRA

- **Licence and size.** ACRA 5.14.2 (Maven Central, updated 2026-10-01) is Apache-2.0. The ASF and FSF both say Apache-2.0 code can go into GPLv3 projects, in one direction only [4][5]. `acra-core` (294 KB) and `acra-http` (71 KB) declare minSdk 14. Their only dependencies are kotlin-stdlib, androidx.annotation and auto-service-annotations [4].
- **Offline.** Approved reports stay as files and are scheduled for sending again at each app start. `SendingConductor` sends them through every configured sender, a few per run [6].
- **Senders.** `HttpSender` needs no user action but needs a backend. `MailSender` opens an `ACTION_SEND_MULTIPLE` intent, so the user picks an email app and sends the mail. Custom `ReportSender`s are allowed [7].
- **Backends.**
  - The official ones are Acrarium (Apache-2.0; Kotlin, Spring Boot, Vaadin and MySQL, so it needs a server) and Acralyzer, which is unmaintained [7][8].
  - The only hosted one listed is Tracepot, which is commercial: EUR 9 a month for 500 reports a day with 30 days of history, and ProGuard deobfuscation marked "Coming Soon" [7][9].
  - Without a server, ACRA's free routes are email or a custom sender, such as a pre-filled GitHub issue.
- **F-Droid precedents.**
  - AnkiDroid ships ACRA 5.13.1 (http, dialog, toast, limiter) to a password-protected, self-hosted Acrarium. Its fdroiddata metadata has no anti-features [10].
  - NewPipe ships `acra-core` 5.14.2 with its own error screen. The user can copy a Markdown report or, after a privacy-policy dialog, send it by email or open a GitHub issue. NewPipe's only anti-feature is NonFreeNet, for YouTube [11].
- **Role here.** ACRA is the fallback if the owner wants nothing sent to a third party (option C in §4). Reports would then arrive only when users choose to file them.

### 3.5 Firebase Crashlytics and Google Analytics

- **Closed, so it needs onionhoney's permission.**
  - The `firebase-crashlytics` 21.0.1 POM says Apache-2.0, but it depends on `play-services-tasks` and `-basement`, which are under the "Android Software Development Kit License" [30].
  - `firebase-analytics` 24.0.0 and its `play-services-measurement-*` dependencies are under that licence themselves [30].
  - Shipping such libraries with onionhoney's GPL code needs onionhoney's written GPL s.7 permission. This is the same blocker as Play Billing [44].
- **Banned on F-Droid:** "Google Play Services and Firebase and Crashlytics" are named as strictly forbidden [21].
- **Advertising ID.** `play-services-measurement` pulls `play-services-ads-identifier`. The `-api` manifest merges `com.google.android.gms.permission.AD_ID`, `ACCESS_ADSERVICES_AD_ID` and `ACCESS_ADSERVICES_ATTRIBUTION`. The `-impl` manifest merges `AD_ID` and the install-referrer permission [31]. That conflicts with "no advertising ID" unless each one is removed with `tools:node="remove"`.
- **Crashlytics data.**
  - It keeps stack traces, Crashlytics installation UUIDs and Firebase installation IDs for 90 days [32].
  - Firebase "may process and store your data anywhere Google or its agents maintain facilities", so there is no EU residency. Google acts as processor, under SCCs and the EU-U.S. DPF [32].
  - Opt-in works through `firebase_crashlytics_collection_enabled=false` plus `setCrashlyticsCollectionEnabled(true)`. While collection is off, crashes are stored on the device and sent once it is turned on [32].
  - Crash Insights compares anonymised stack traces with those of other Firebase apps unless switched off in the console [32].
  - The service is free [32].

### 3.6 TelemetryDeck, Aptabase, Countly, Bugsnag

- **TelemetryDeck.**
  - `com.telemetrydeck:kotlin-sdk` 7.3.0 (2026-10-28) is MIT, minSdk 23. It pulls Ktor client, kotlinx-serialization and AppCompat [33].
  - It caches signals on disk by default. In-memory mode discards unsent events when the app is killed. The server adds its own salt to the user identifier and hashes it again [33].
  - Pricing changed on 2026-07-01: the free tier is 50,000 events a month for new accounts, while accounts created before that date keep 100,000 [34].
  - TelemetryDeck GmbH is in Augsburg, Germany. Its docs say the service is "all hosted within the EU". The privacy policy lists Microsoft Ireland, AWS (US) and Hetzner under website hosting [33][34].
  - It offers no self-hosting, and no source says the backend is open source [33].
  - Ruled out: the backend is not shown to be open source, the SDK is the heaviest, and the free tier is smaller than PostHog's.
- **Aptabase.**
  - The Kotlin SDK's `trackEvent` POSTs each event at once on a thread pool and only logs failures. There is no queue and no retry, and it does no crash reporting [35].
  - The SDK moved to JitPack (0.1.0, minSdk 16). The last Maven Central release is 0.0.6, from May 2023 [35].
  - The server is AGPL-3.0. The hosted free plan gives 20,000 events a month, with EU (Germany) or US (Virginia) residency, and pauses at the limit [35].
  - Ruled out: an app that works offline would lose events.
- **Countly.**
  - Android SDK 26.1.6 is MIT, minSdk 21. It keeps a request queue of up to 1000 in SharedPreferences [36].
  - The server (Countly Lite) is AGPL-3.0 with a modified s.7 that forbids removing Countly branding, and it needs your own server. The hosted Flex plan starts at $175 a month [36].
  - Ruled out: no free hosted tier.
- **Bugsnag.**
  - `bugsnag-android-core` 6.28.0 is MIT, minSdk 16. Its 650 KB AAR bundles a prebuilt native `libbugsnag-root-detection.so` for four ABIs [37].
  - It stores up to 32 events on disk, and by default it generates and stores an anonymous ID [37].
  - The SaaS is proprietary. The free plan gives 1 user, 7.5K events and 1M spans a month, with 7-day retention [37].
  - No official BugSnag page on EU residency was found [37].
  - Ruled out.

## 4. Options compared

| Option | Verdict | Why |
|---|---|---|
| **A. Vitals + Sentry EU (crashes) + PostHog EU through an in-house client (usage), same in both flavours** | **Recommended** | Mature Android crash tooling (ANR detection, R8 support, disk cache) [13][14]. Free tiers fit an opt-in app [18][28]. EU storage for event data [17][28]. FLOSS client code passes F-Droid's rules [21]. Costs: two processors, so two privacy-policy entries; Sentry's server is not FLOSS [19]; Sentry keeps project metadata in the US [17]; the usage client must be written and tested |
| B. Vitals + PostHog SDK for both errors and usage | Second choice | One vendor and the largest free tier [28]. But Android crash support is basic: no ANR or native crashes in the docs, and whether crashes survive a restart is undocumented [27]. About 2.5 MB of artifacts, and three autocapture features to turn off [26] |
| C. Vitals + ACRA, user-sent reports only (NewPipe pattern), no usage stats | Fallback if no third party is wanted | No third party, no cost, no server; nothing leaves without a tap per report [6][7][11]. But few reports arrive (GitHub needs an account; email exposes the reporter's address), there are no ANRs or crash rates, and there are no usage stats, which misses the owner's "improve the app" goal |
| D. Option A with GlitchTip EU instead of Sentry | Ready fallback | Open-source backend, so no F-Droid NonFreeNet question; only the DSN changes [23]. But 1,000 events a month, mapping uploads count against it, and deobfuscation bugs are open [23][24] |
| E. Firebase Crashlytics + Google Analytics (`play` only) | Ruled out | Closed (needs onionhoney's s.7 permission), adds AD_ID permissions, no EU residency, banned on F-Droid, leaves `foss` with nothing [21][30][31][32][44] |
| F. TelemetryDeck, Aptabase, Countly or Bugsnag | Ruled out | §3.6 |

## 5. F-Droid rules and precedents

F-Droid is "later" in scope.md:112. The `foss` build should still be ready for it from the start.

- **Libraries.** F-Droid forbids proprietary tracking, advertising and analytics libraries. Upstream must offer a FLOSS alternative or a build flavour without them. Prebuilt FLOSS binaries from trusted repositories (Maven Central, Google Maven, JitPack and others) are allowed if freely licensed [21]. ACRA (Apache-2.0) and the Sentry, PostHog, Countly, Bugsnag and TelemetryDeck SDKs (MIT) all pass that test [4][12][26][33][36][37].
- **Tracking anti-feature.**
  - The label is for apps that report activity "either without your permission or by default" [22].
  - It "would not be applied ... if the functionality is opt-in (i.e. you are asked before it happens) and disabled by default". Turning it on must also require informed consent ("a privacy policy similar to GDPR") and collect as little personal data as possible [22].
  - The one-line summary on the same page says "even when it can be turned off", so an opt-out design would be tagged [22].
  - Apps marked Tracking are hidden by default in the F-Droid client [22].
- **NonFreeNet** is for apps that "promote or depend entirely on a proprietary network service" [22].
  - Optional crash reporting to sentry.io neither promotes it nor depends on it entirely. So it probably does not trigger the label, but the label is up to the reviewer and this has not been tested (inference).
  - Pointing the DSN at GlitchTip (MIT) removes the question [23].
  - **Non-Free Dependencies** does not apply, because nothing proprietary has to be installed [22].
  - **Tethered Network Services** does not apply while reporting is optional. It is also waived when a simple setting can point the app at a self-hostable server [22].
- **Precedents.**
  - **Element X:** its non-enterprise build includes the PostHog and Sentry analytics providers in every flavour, F-Droid's included, behind an opt-in prompt. PostHog goes to Element's own `posthog.element.io`. Its fdroiddata metadata has no anti-features [38].
  - **Tasks.org** goes the other way. Firebase, Crashlytics and PostHog live only in its `googleplay` flavour. The F-Droid recipe builds `generic` and deletes Google services, Firebase and Crashlytics lines with `sed`. Recent builds carry no anti-features, but builds 4.8.x to 5.3.6 carried "Tracking" [39].
  - **AnkiDroid** and **NewPipe** ship ACRA with no Tracking label (§3.4) [10][11].

## 6. EU consent rules

- **ePrivacy art. 5(3).** EDPB Guidelines 2/2023 (v2.0, adopted 7 October 2024) [40]:
  - The provision protects "any information stored in such terminal equipment, regardless of whether or not it is personal data" (para 10, citing CJEU *Planet49*, C-673/17).
  - Software that "proactively call[s] an Application Programming Interface (API) endpoint", and SDKs that proactively send information, fall within art. 5(3) (paras 33-34).
  - Information used strictly inside the device is not "gaining access" while it "does not leave the device". Once it or anything derived from it is accessed, art. 5(3) applies (para 44).
  - "Storage" is a separate criterion. It means placing information on the device's storage, including by instructing software on the device (paras 35-36).
  - IP addresses can bring art. 5(3) into play (paras 54-55).
  - The guidelines do not analyse the exemptions. They must be assessed case by case (paras 40, 56).
- **The two exemptions.** As WP29 Opinion 04/2012 (WP194) quotes them [41], they cover storage or access:
  - "for the sole purpose of carrying out the transmission of a communication over an electronic communications network", or
  - "strictly necessary in order for the provider of an information society service explicitly requested by the subscriber or user to provide the service".
- **WP194 on analytics:** analytics are "not strictly necessary to provide a functionality explicitly requested by the user", and first-party analytics cookies "are not exempt from consent but pose limited privacy risks" [41]. WP194 is about cookies. Applying it to in-app crash and usage reporting is an inference, but a close one: Rouxtine works fully without either. So both need consent, which the owner's opt-in already gives.
- **GDPR consent (EDPB Guidelines 05/2020, v1.1)** [42]:
  - "The use of pre-ticked opt-in boxes is invalid under the GDPR."
  - Consent must be granular, given separately for separate purposes (§3.1.3).
  - Withdrawal must work "via the same electronic interface", "free of charge or without lowering service levels" (para 114, GDPR art. 7(3)).
- **Children (GDPR art. 8, EDPB 05/2020 §7.1)** [42]:
  - The rule covers processing based on consent, in an information society service offered directly to a child.
  - Under 16, consent must be "given or authorised by the holder of parental responsibility". Member states may set a lower age, "not below 13 years" (para 125).
  - A cross-border service "may need to comply with the respective national laws of each Member State" (para 131). The controller must make "reasonable efforts to verify" (para 132).
  - Art. 8 does not apply if the service is clearly offered only to people aged 18 or over (para 130).
  - Inference: speedcubing has many young users, so this matters for Rouxtine whenever the data sent is personal (an IP address, a persistent ID). Sending no ID and no IP, as in §1, keeps it as close to non-personal as the design allows.

## 7. Play rules: Data safety and Families

- **Data safety** [3]:
  - "Collect" means "transmitting data from your app off a user's device", including data sent "by libraries and/or SDKs". Data processed only on the device is not collected.
  - Passing data to a "service provider" that processes it for the developer is not "sharing".
  - Data can be declared optional "only if all users – regardless of device or region – can either optionally provide information, opt-out, or opt-in".
  - Relevant types: Crash logs ("stack traces"), Diagnostics ("battery life, loading time, latency"), Other app performance data, App interactions ("number of times they visit a page"), and Device or other IDs ("Identifiers that relate to an individual device, browser or app").
  - Every app must complete the form and link a privacy policy, even if it collects nothing.
- **Families policy** [43]:
  - Apps for both children and older audiences "must not transmit AAID, SIM Serial, Build Serial, BSSID, MAC, SSID, IMEI, and/or IMSI from children or users of unknown age".
  - Mixed-audience apps "must not implement APIs or SDKs that are not approved for use in child-directed services unless they are used behind a neutral age screen or implemented in a way that does not result in the collection of data from children".
  - The Families self-certified SDK list is required only for ads SDKs serving ads to children. With zero ads, the list does not apply.
- **What this means for Rouxtine:** the target-audience declaration decides whether the age screen in §1.4 is needed. Whether Play counts Sentry or PostHog as "approved for use in child-directed services" was not checked. If the declared audience includes children, put both behind the neutral age screen, or make sure they collect no data from children.

## Questions for the owner

1. **Target audience on Play.** Which age groups will you declare, and do they include under-13s or under-16s? This decides whether usage stats need an age screen or must be strictly non-personal (GDPR art. 8, Play Families policy).
2. **Country.** Which EU country do you live in, if any? It sets your lead data-protection authority and the national age of digital consent (13 to 16).
3. **Crash backend.** Is Sentry's EU cloud acceptable: source-available (FSL) server, 5k errors a month free, mature R8 support? Or do you want the open-source GlitchTip EU (1k events a month free, deobfuscation still buggy), for both flavours or just `foss`?
4. **Usage stats in R1.** scope.md puts both crash reports and usage stats in R1. Keep that, or ship crash reports first and usage stats later?
5. **Retention numbers.** Do you need them? They require a stored, resettable random ID, declared as "Device or other IDs". Or are anonymous events enough?
6. **Privacy policy.** Are you OK publishing one on GitHub Pages under your name with a contact email, and accepting Sentry's and PostHog's data processing terms as controller?
7. **Crash default.** Is "Ask me after a crash" the right default, so nothing is sent until the user taps? Or should crash reports stay off until switched on in Settings?
8. **Obfuscation.** Can release builds use `-dontobfuscate` (R8 shrinking kept), so stack traces are readable without uploading mapping files?

## Sources

[1]: https://developer.android.com/topic/performance/vitals (Android vitals overview, thresholds)
[2]: https://support.google.com/googleplay/android-developer/answer/9844486?hl=en (Android vitals in Play Console)
[3]: https://support.google.com/googleplay/android-developer/answer/10787469?hl=en (Data safety section)
[4]: https://repo1.maven.org/maven2/ch/acra/acra-core/5.14.2/acra-core-5.14.2.pom ; https://repo1.maven.org/maven2/ch/acra/acra-http/5.14.2/acra-http-5.14.2.pom ; https://repo1.maven.org/maven2/ch/acra/acra-core/maven-metadata.xml (AAR sizes and minSdk measured from the downloaded files)
[5]: https://www.apache.org/licenses/GPL-compatibility.html ; https://www.gnu.org/licenses/license-list.html#apache2
[6]: https://github.com/ACRA/acra/blob/master/acra-core/src/main/java/org/acra/startup/StartupProcessorExecutor.kt ; https://github.com/ACRA/acra/blob/master/acra-core/src/main/java/org/acra/sender/SendingConductor.kt
[7]: https://www.acra.ch/docs/Senders ; https://www.acra.ch/docs/Backends
[8]: https://github.com/F43nd1r/Acrarium/blob/master/README.md ; https://github.com/F43nd1r/Acrarium/blob/master/LICENSE
[9]: https://www.tracepot.com/
[10]: https://gitlab.com/fdroid/fdroiddata/-/blob/master/metadata/com.ichi2.anki.yml ; https://github.com/ankidroid/Anki-Android/wiki/Crash-Reports ; https://github.com/ankidroid/Anki-Android/blob/main/gradle/libs.versions.toml
[11]: https://gitlab.com/fdroid/fdroiddata/-/blob/master/metadata/org.schabi.newpipe.yml ; https://github.com/TeamNewPipe/NewPipe/blob/dev/app/src/main/java/org/schabi/newpipe/error/ErrorActivity.kt ; https://github.com/TeamNewPipe/NewPipe/blob/dev/gradle/libs.versions.toml
[12]: https://repo1.maven.org/maven2/io/sentry/sentry-android-core/8.62.0/sentry-android-core-8.62.0.pom ; https://repo1.maven.org/maven2/io/sentry/sentry-android-core/8.62.0/sentry-android-core-8.62.0.aar ; https://github.com/getsentry/sentry-java/blob/8.62.0/LICENSE ; https://repo1.maven.org/maven2/com/abovevacant/epitaph/0.1.1/epitaph-0.1.1.pom ; https://api.deps.dev/v3alpha/systems/maven/packages/io.sentry%3Asentry-android/versions/8.62.0:requirements
[13]: sentry-java at tag 8.62.0: https://github.com/getsentry/sentry-java/blob/8.62.0/sentry-android-core/src/main/java/io/sentry/android/core/AndroidOptionsInitializer.java (cache dir L139, L172-175; transport gate L202-204; bundled root for API ≤ 25 L205-212; ANR integration L466; distinct ID L538-540; getCacheDir L565-566) ; …/sentry-android-core/src/main/java/io/sentry/android/core/ManifestMetadataReader.java ; …/sentry-android-core/src/main/java/io/sentry/android/core/Installation.java ; …/sentry-android-core/src/main/java/io/sentry/android/core/DefaultAndroidEventProcessor.java (mergeUser L171-183) ; …/sentry-android-core/src/main/java/io/sentry/android/core/SentryAndroid.java (L128-176: defaults, then the app's configuration, then integrations) ; …/sentry/src/main/java/io/sentry/DataCollectionResolver.java ; …/sentry/src/main/java/io/sentry/SentryOptions.java ; …/sentry/src/main/java/io/sentry/Scope.java (L1048-1051) ; …/sentry/src/main/java/io/sentry/Session.java ; …/sentry/src/main/java/io/sentry/transport/ITransportGate.java
[14]: https://docs.sentry.io/platforms/android/configuration/options/ ; https://docs.sentry.io/platforms/android/manual-setup/
[15]: https://docs.sentry.io/platforms/android/data-management/data-collected/
[16]: https://docs.sentry.io/security-legal-pii/scrubbing/server-side-scrubbing/
[17]: https://docs.sentry.io/organization/data-storage-location/
[18]: https://sentry.io/pricing/ ; https://docs.sentry.io/pricing/quotas/
[19]: https://github.com/getsentry/sentry/blob/master/LICENSE.md ; https://develop.sentry.dev/self-hosted/
[20]: https://docs.sentry.io/concepts/key-terms/dsn-explainer/
[21]: https://f-droid.org/docs/Inclusion_Policy/
[22]: https://f-droid.org/docs/Anti-Features/
[23]: https://gitlab.com/glitchtip/glitchtip-backend/-/raw/master/LICENSE ; https://gitlab.com/glitchtip/glitchtip-backend/-/raw/master/README.md ; https://glitchtip.com/pricing ; https://glitchtip.com/documentation/install
[24]: https://gitlab.com/glitchtip/glitchtip-backend/-/issues/504 ; https://gitlab.com/glitchtip/glitchtip-backend/-/issues/413 ; https://gitlab.com/glitchtip/glitchtip-backend/-/issues/519
[25]: https://github.com/bugsink/bugsink/blob/main/LICENSE ; https://github.com/bugsink/bugsink/blob/main/README.md ; https://www.bugsink.com/built-to-self-host/
[26]: https://repo1.maven.org/maven2/com/posthog/posthog-android/3.73.1/posthog-android-3.73.1.pom ; https://repo1.maven.org/maven2/com/posthog/posthog/6.47.7/posthog-6.47.7.pom ; https://posthog.com/docs/libraries/android ; file sizes re-checked 2026-10-29: https://repo1.maven.org/maven2/com/posthog/posthog/6.47.7/posthog-6.47.7.jar , https://repo1.maven.org/maven2/com/squareup/okhttp3/okhttp/4.12.1/okhttp-4.12.1.jar , https://repo1.maven.org/maven2/com/squareup/okio/okio-jvm/3.11.0/okio-jvm-3.11.0.jar , https://repo1.maven.org/maven2/com/google/code/gson/gson/2.10.1/gson-2.10.1.jar
[27]: https://posthog.com/docs/error-tracking/installation/android
[28]: https://posthog.com/pricing ; https://posthog.com/docs/privacy/gdpr-compliance ; https://github.com/PostHog/posthog/blob/master/LICENSE
[29]: https://posthog.com/docs/api/capture
[30]: https://dl.google.com/android/maven2/com/google/firebase/firebase-crashlytics/21.0.1/firebase-crashlytics-21.0.1.pom ; https://dl.google.com/android/maven2/com/google/android/gms/play-services-tasks/18.4.0/play-services-tasks-18.4.0.pom ; https://dl.google.com/android/maven2/com/google/android/gms/play-services-basement/18.4.0/play-services-basement-18.4.0.pom ; https://dl.google.com/android/maven2/com/google/firebase/firebase-analytics/24.0.0/firebase-analytics-24.0.0.pom ; https://dl.google.com/android/maven2/com/google/android/gms/play-services-measurement/24.0.0/play-services-measurement-24.0.0.pom
[31]: https://dl.google.com/android/maven2/com/google/android/gms/play-services-measurement-api/24.0.0/play-services-measurement-api-24.0.0.aar ; https://dl.google.com/android/maven2/com/google/android/gms/play-services-measurement-impl/24.0.0/play-services-measurement-impl-24.0.0.aar ; https://dl.google.com/android/maven2/com/google/firebase/firebase-crashlytics/21.0.1/firebase-crashlytics-21.0.1.aar
[32]: https://firebase.google.com/docs/android/play-data-disclosure ; https://firebase.google.com/support/privacy ; https://firebase.google.com/docs/crashlytics/android/customize-crash-reports
[33]: https://repo1.maven.org/maven2/com/telemetrydeck/kotlin-sdk/7.3.0/kotlin-sdk-7.3.0.pom ; https://github.com/TelemetryDeck/KotlinSDK/blob/main/README.md ; https://telemetrydeck.com/docs/guides/privacy-faq/ ; https://telemetrydeck.com/docs/articles/hosting-solutions/
[34]: https://telemetrydeck.com/blog/pricing-update-2026/ ; https://telemetrydeck.com/privacy/
[35]: https://github.com/aptabase/aptabase-kotlin/blob/main/aptabase/src/main/java/com/aptabase/Aptabase.kt ; https://github.com/aptabase/aptabase-kotlin/blob/main/README.md ; https://repo1.maven.org/maven2/com/aptabase/aptabase/maven-metadata.xml ; https://github.com/aptabase/aptabase/blob/main/LICENSE ; https://aptabase.com/pricing
[36]: https://repo1.maven.org/maven2/ly/count/android/sdk/26.1.6/sdk-26.1.6.pom ; https://github.com/Countly/countly-sdk-android/blob/master/sdk/src/main/java/ly/count/android/sdk/CountlyStore.java#L85 ; https://github.com/Countly/countly-server/blob/master/LICENSE.md ; https://countly.com/pricing
[37]: https://repo1.maven.org/maven2/com/bugsnag/bugsnag-android-core/6.28.0/bugsnag-android-core-6.28.0.aar ; https://docs.bugsnag.com/platforms/android/configuration-options/ ; https://www.bugsnag.com/pricing/
[38]: https://github.com/element-hq/element-x-android/blob/18bb4632f2448b21464cea77d38fb7b6a2d0d1bf/plugins/src/main/kotlin/ModulesConfig.kt ; https://github.com/element-hq/element-x-android/blob/18bb4632f2448b21464cea77d38fb7b6a2d0d1bf/plugins/src/main/kotlin/extension/DependencyHandleScope.kt#L129-L143 ; https://github.com/element-hq/element-x-android/blob/18bb4632f2448b21464cea77d38fb7b6a2d0d1bf/features/analytics/impl/src/main/kotlin/io/element/android/features/analytics/impl/AnalyticsOptInPresenter.kt ; `PosthogEndpointConfigProvider.kt` in the same commit ; https://gitlab.com/fdroid/fdroiddata/-/blob/master/metadata/io.element.android.x.yml
[39]: https://gitlab.com/fdroid/fdroiddata/-/blob/master/metadata/org.tasks.yml ; https://github.com/tasks/tasks/blob/1211fc46/app/build.gradle.kts
[40]: https://www.edpb.europa.eu/system/files/2024-10/edpb_guidelines_202302_technical_scope_art_53_eprivacydirective_v2_en_0.pdf (EDPB Guidelines 2/2023, v2.0)
[41]: https://ec.europa.eu/justice/article-29/documentation/opinion-recommendation/files/2012/wp194_en.pdf (WP29 Opinion 04/2012, WP194)
[42]: https://www.edpb.europa.eu/system/files/documents/files/file1/edpb_guidelines_202005_consent_en.pdf (EDPB Guidelines 05/2020 on consent, v1.1)
[43]: https://support.google.com/googleplay/android-developer/answer/9893335?hl=en (Families policies) ; https://support.google.com/googleplay/android-developer/answer/9900633?hl=en (Families self-certified ads SDKs)
[44]: [licensing-monetization.md](licensing-monetization.md) and [play-billing-gpl.md](play-billing-gpl.md) (this folder)
[45]: https://reports.exodus-privacy.eu.org/en/reports/528771 (not reachable: connection reset, HTTP 503)

## Open questions / unverified

- **Android vitals and Data safety.** No Google page says whether data that Android and Play collect for vitals must be declared. "Not declared, no in-app consent" is an inference from the "transmitted ... from your app" definition [1][3].
- **Local crash file before consent.** EDPB 2/2023 says on-device use is not "access" until data leaves (para 44), but it treats "storage" as a separate criterion (paras 35-36) [40]. Whether Rouxtine writing its own crash file before any choice needs consent was not analysed. Neither were national transpositions of art. 5(3), or DPA guidance on exemptions for audience measurement or crash reporting (for example the CNIL's, or the owner's home DPA).
- **Sentry's transport gate as a consent switch.** `ITransportGate` keeps events cached until sending is allowed "or maximum number of retries ... has been reached" [13]. How many retries that is, and whether envelopes survive a long closed gate, was not tested. §1.4 avoids relying on it.
- **Sending a stored crash through Sentry later.** Serialising a `Throwable` and passing it to `Sentry.captureException` on the next launch is an untested idea. So is whether Sentry's ANR reporting (from `ApplicationExitInfo`) picks up exits that happened before a late `SentryAndroid.init`.
- **Android 7.0 and the PostHog host.** Sentry bundles a root certificate for its own SaaS on API ≤ 25, because those versions may lack ISRG Root X1 [13]. Which CA signs `eu.i.posthog.com`, and whether a plain `HttpURLConnection` or OkHttp call validates it on Android 7.0, was not checked. Test on an API 24 emulator.
- **PostHog GeoIP.** Whether PostHog's GeoIP enrichment still runs when IP capture is off, and how to turn it off (project setting or a per-event property), was not checked.
- **PostHog `distinct_id`.** Whether a random value held in memory for one app run counts as "Device or other IDs" in Data safety is unclear. Play's definition covers identifiers that relate to an individual app [3]. Also unchecked: whether PostHog's ingestion treats many events with an ephemeral or shared ID differently.
- **Sentry DPA.** PostHog's DPA is confirmed [28]. Sentry's DPA and how it is accepted on the free plan were not checked.
- **Sentry over quota.** The quota page says events over quota "will not be accepted" [18]. Whether the free Developer plan has any pay-as-you-go option switched on by default was not checked.
- **Bugsink's role.** No source says GlitchTip is the only FLOSS Sentry-compatible backend. Bugsink is source-available, not FLOSS [25]. Others were not surveyed.
- **NonFreeNet for sentry.io.** "Optional reporting to an FSL server is not labelled" is a reading of F-Droid's definitions. It is untested with a reviewer [22].
- **Exodus Privacy.** A search snippet says Exodus report 528771 (AnkiDroid 2.19.1, 2024-11-22) lists ACRA as crash reporting. Whether that was the F-Droid or the Play build is unknown, and the host was unreachable [45]. Expect scanners like Exodus to list any crash or analytics SDK, even an opt-in one, and say so up front in the README and in-app text. Whether Exodus lists Sentry, or would detect an in-house PostHog client, was not checked.
- **BugSnag EU residency.** No official page was found [37].
- **Sizes not re-measured.** The Crashlytics (≈2.8 MB), Firebase Analytics (≈6.6 MB) and TelemetryDeck (≈3.8 MB) totals come from one resolver run. All sizes are artifact sums, not R8 APK deltas.
- **The usage client's size.** "About 200 lines" is an estimate.
