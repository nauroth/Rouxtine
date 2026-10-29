# Handoff: Rouxtine, a native Android Roux trainer

Started 2026-10-28 in `nauroth/roux-trainers` (research session), moved to this repo the same
day. Read this first, then the research notes it links to. The notes hold the evidence; this
file holds the decisions and the plan.

Paths like `src/lib/Pruner.tsx:471` in these docs refer to
[nauroth/roux-trainers](https://github.com/nauroth/roux-trainers) at `0129460`, the fork of
onionhoney's web Roux Trainers (GPL-3.0) that this app ports from. Attach that repo read-only
when porting.

## Goal (owner's words, condensed)

A **native Android app** with a **great-looking, modern UI** (Material 3 Expressive motion and
shape morphing) that contains every tool needed to get good at **Roux, and only Roux**: no
CFOP, ZZ or other methods anywhere in the app. It is for the **whole Roux community, every
level**, from first-time solvers to advanced. It grows out of onionhoney's web Roux Trainers,
but it is a new Kotlin app, not a wrapper of the web app.

## Decided (confirmed by the owner)

- **Core concept: modules and Rouxtines.** Every training tool is a toggleable **module**.
  Users tie modules together into their own training regimen, a **Rouxtine**, focused on what
  they want to progress on and what they need to get better at. Full design:
  **[rouxtine-design.md](rouxtine-design.md)**. In short: custom Rouxtines, level/goal
  templates (community ones via GitHub pull requests) and app-generated Rouxtines; automatic
  adaptation inside blocks, suggestions with reasons across blocks; share by code, QR or file;
  daily streak with freezes plus a weekly goal. All of it in the first release.
- **Scope:** **[scope.md](scope.md)** says what ships in R1, R2+, later, and never. It
  supersedes the priorities in the feature-map notes.
- **Repo:** `nauroth/rouxtine`, public, not a fork.
- **Name:** Rouxtine. **Package id:** `io.github.nauroth.rouxtine`, with debug builds on
  `io.github.nauroth.rouxtine.debug` so both can be installed at once. No domain.
- **Money (decided 2026-10-29 in a parallel session, replacing "Ko-fi only"):**
  - The app is **free** with **zero ads**. Every training function is free: never paywall a
    trainer, the solver, stats or analysis.
  - Paid extras are only **cosmetic** (a one-time **Supporter** unlock for themes) or a
    **specific add-on** (possibly a paid smart-cube module), sold through **Play Billing** in
    the `play` flavour, with the 15 % fee tier enabled.
  - **Ko-fi lives off Play only:** GitHub README, `foss` build, F-Droid metadata. No donate link
    or wording in the Play build, and no link to a page that shows one.
  - **Blocker before the first paid feature:** Play Billing is a closed library. Shipping it in
    one APK with onionhoney's GPL code needs onionhoney's written permission (a GPL s.7
    additional permission); see [licensing-monetization.md](research/licensing-monetization.md)
    and [monetization.md](research/monetization.md). Being verified.
  - **Play account:** personal. Once anything is sold, Google shows the developer's full address
    publicly; the owner accepts this. Local business, tax and social-security setup is the
    owner's to arrange before the first paid feature ships.
- **Crash reporting and analytics (decided 2026-10-29):** wanted, for fixing bugs and improving
  the app. Opt-in, open-source SDK where possible, plus Play's Android vitals. No advertising
  ID, nothing personal, everything declared in the Play Data safety form. Which tool: open.
- **Licence:** GPL-3.0-only (SPDX `GPL-3.0-only`), matching onionhoney's upstream exactly. GPL
  is required because the app ports their GPL solver and data.
- **minSdk 24** (Android 7.0), compileSdk/targetSdk 37. The owner asked for "as low as the stack
  allows", and 24 is that floor: material3 1.5.0-beta02, navigation3, navigationevent and
  DataStore declare 24, and a merge at 23 fails (measured). It reaches 99.4 % of Play devices.
  Evidence: [min-sdk-and-delivery.md](research/min-sdk-and-delivery.md) Part A.
- **Test phone:** Samsung Galaxy S25, Android 16 (One UI 8; One UI 9 / Android 17 has been
  rolling out since 2026-09-28). Roborazzi qualifier
  `w360dp-h780dp-normal-long-notround-any-xxhdpi-keyshidden-nonav` (confirm the density with
  `adb shell wm density`).
- **Audience and curriculum:** drills and training for every Roux level. The owner's own level
  doesn't matter. Content research covers rouxcubing.com, resources.rouxers.com and everything
  they link (onionhoney's trainers are already known).
- **Smart cubes:** support many Bluetooth protocols, starting with QiYi and GAN Gen4 (see
  "Smart cubes to buy" below).
- **Languages:** English at launch. All UI strings in resources from day one. Community
  translations through **Weblate** (Hosted Weblate's free plan for libre projects), set up once
  the first screens exist.
- **Distribution:** **Google Play is the main release** (`play` flavour, no donation link),
  plus a signed APK on GitHub releases (`foss` flavour: no proprietary libraries, everything
  unlocked, Ko-fi link), which also serves Obtainium users. One app signing key for both, uploaded to Play App Signing, so users can
  switch between them. F-Droid later, if wanted. A new personal Play account needs $25, ID,
  and a closed test with 12 testers for 14 days before production.
- **Signing:** keys live in GitHub Actions secrets, never in the repo. The debug key goes in
  four repo secrets. The release key goes in environment secrets behind the owner's approval,
  is generated on the owner's computer, and gets an offline backup.
- **Smart cubes to buy:** a QiYi AI 3x3 and a GAN i4, so QiYi and GAN Gen4 can both be tested.
  Smart-cube support ships after R1 ([scope.md](scope.md) R2+).
- **Content scope:** teach from zero (own text); one neutral lesson for cubers switching from
  other methods; Roux and everything that applies to Roux, nothing else (no glossary entries
  for other methods); videos linked, not embedded, low priority; third-party material only with
  consent; adapt Roux Reader sections (MIT) with notice; U-sticker + pattern CMLL recognition
  with standard names; M-hand as a setting (right by default), OH hand only with OH later;
  community links maybe later; no mindset content.
- **Builds onto the phone:** the owner prefers whatever is safest and most widely used.
  Confirmed by research: CI publishes each `main` build to a rolling GitHub pre-release, and
  Obtainium on the phone tracks it ("Include prereleases" and "Use release date as version
  string" on). Plain download from the release page is the fallback, with no login and no zip.
  The owner must turn **Samsung Auto Blocker** off (and keep its auto-re-enable off), or every
  install outside Play/Galaxy Store is blocked. Details:
  [min-sdk-and-delivery.md](research/min-sdk-and-delivery.md) Part B.
- **Stack:** Kotlin + Jetpack Compose + Material 3, accepted as researched in
  [android-ui-stack.md](research/android-ui-stack.md): AGP 9.5.0, Kotlin 2.4.21, Compose BOM
  2026.09.00, Navigation 3, Hilt + KSP, Room 3, DataStore, Vico, Kable later for BLE.
- **Material 3 Expressive:** required. Pin `androidx.compose.material3:material3:1.5.0-beta02`
  over the BOM; stable 1.4.0 lacks `MaterialExpressiveTheme`, `MotionScheme.expressive()` and
  the new components. Some APIs (`MaterialShapes`, `LoadingIndicator`) need an experimental
  opt-in.
- **Feature map:** superseded by [scope.md](scope.md). The feature-map notes
  ([roux-training-features.md §2](research/roux-training-features.md),
  [roux-community-resources.md §4, §9](research/roux-community-resources.md)) keep the
  evidence and the code to reuse for each feature.
- **Timer:** the app keeps **its own timer and stats** (the source of truth) and has an
  **"Export to Twisty/Cubic Timer"** action. It writes Twisty's *backup* `.txt` format, which
  both apps import, with categories like "Roux FB" / "CMLL". Format, gotchas and dedupe rules
  are in [timer-integration.md](research/timer-integration.md) (sections A, "Generator
  gotchas"). Zero-tap insertion is impossible: neither app exposes an entry point.
- **Virtual cube:** custom renderer on Compose `Canvas` with pure-Kotlin 3D math, ported from
  the web app's `src/components/CubeSim.tsx`. It is the only option that shows up in JVM
  screenshot tests. No OpenGL, Filament or WebView in R1.
- **Solver:** port the TS solver to Kotlin with sparse/hashed pruning tables. Do not embed the
  JS: its dense tables allocate 191–382 MB. Evidence:
  [roux-training-features.md §5](research/roux-training-features.md).

## Open

Grilling finished 2026-10-29. The design decisions are settled.

**Owner to-dos (no rush, none block the first code):**
- Buy a QiYi AI 3x3 and a GAN i4.
- Turn off Samsung Auto Blocker on the S25 when the first APK is ready.
- When CI is set up: paste the four debug-key secrets into GitHub (the agent generates them
  and hands them over in a file that is deleted afterwards).
- Before going public:
  - create the Google Play developer account ($25, ID);
  - line up 12 testers for the 14-day closed test (the Roux Discord is a good place);
  - generate the release key on your own computer;
  - register `io.github.nauroth.rouxtine` for developer verification before the 2027 rollout.
- Optional: write to onionhoney about Rouxtine and ask for an explicit licence statement
  ([licensing-monetization.md §6](research/licensing-monetization.md)).
- Later: ask content authors for consent ([roux-community-resources.md §10](research/roux-community-resources.md) item 7).

**Defaults the agent chose (the owner may override):**
- Smart cubes in R2, not R1, to keep R1 shippable.
- Sharing (code, QR, file) in R1, because templates use the same format.
- Community templates are bundled per release; no online fetch in R1.
- App links (tappable URLs) wait until there is a hosted verification file.

## What exists now

- `docs/research/`: research notes, each with cited sources and an "open questions /
  unverified" list.
  - `android-ui-stack.md`: stack, cube-rendering options, screenshot tools, versions to pin.
  - `cloud-toolchain.md`: **verified in the cloud container**. SDK install, build, Roborazzi
    screenshots and lint all work; exact commands, timings and five required fixes.
    `img/` holds the spike's light and dark screenshots.
  - `roux-training-features.md`: Roux learning path, feature map, existing tools, reuse and
    port assessment with `path:line` refs into roux-trainers, licensing.
  - `timer-integration.md`: Twisty Timer / Cubic Timer analysis and the export format spec.
  - `design-plugins.md`: which design skills cover Compose/M3, and which to skip and why.
  - `roux-community-resources.md`: full analysis of rouxcubing.com, resources.rouxers.com,
    the Roux Reader, the rouxers tutorial and everything they link: curriculum by level,
    drill and learning-content catalogues, alg sets with licences, tools, video index,
    feature-map delta, scope questions.
  - `training-regimens.md`: prior art and design space for modules and Rouxtines, per-attempt
    log schema, adaptive algorithms (FSRS-6, log-time EMAs).
  - `licensing-monetization.md`: Ko-fi on Play/F-Droid/GitHub, GPL duties for every build.
  - `min-sdk-and-delivery.md`: minSdk floor, S25 profile, CI-to-phone pipeline, signing,
    developer verification, Samsung Auto Blocker.
  - `smart-cubes.md`: cube to buy, protocol matrix and order, Android BLE, timing, code to port.
  - `monetization.md`: Play options (donations, Supporter unlock, tips, paid, ads), account
    setup, GPL precedents. Written in a parallel session.
  - `extra-skills.md`: nine more skills reviewed; only Humanizer installed.
- `docs/scope.md` and `docs/rouxtine-design.md`: the decided scope and Rouxtine design.
- Branch `wip/research-data`: the structured intermediate outputs (claims, verifier verdicts,
  crawl summaries, YouTube index) and workflow scripts behind those notes. Raw page texts were
  not kept (most sources have no licence).
- `scripts/setup-android-sdk.sh`: idempotent SDK + Gradle-mirror setup (52 s fresh, 0 s warm).
  Not wired up yet: hook it into a SessionStart hook or the environment setup script.
- `.claude/settings.json`: plugins `ponytail`, `android-skills` (Google, official),
  `chrisbanes-skills`, `humanizer`. They load at session start.
- `.claude/skills/`: vendored `md3e`, `material-design-3-ui`, `design-critique` and
  `accessibility-review`. Sources, commits and licences are in `.claude/skills/VENDORED.md`.
- `CLAUDE.md` and `docs/agents/`: the mattpocock-skills setup (GitHub Issues tracker on this
  repo, default triage labels, single-context domain docs).

## Environment facts that will bite

From [cloud-toolchain.md](research/cloud-toolchain.md); read its "Gotchas and fixes" before
the first build.

- No `/dev/kvm`, so **no emulator**. You see the UI through **Roborazzi PNGs**: record, then
  open them with the Read tool. `verifyRoborazziDebug` produces a red-pixel diff. Anything
  GPU/OpenGL renders blank on the JVM.
- **Maven Central returns HTTP 429** from the cloud container's IP. The setup script installs a
  Gradle init script that redirects to Google's Maven Central mirror. Keep `mavenCentral()` in
  the build files.
- Robolectric 4.17 + SDK 37 + JDK 21 needs `--add-opens=java.base/jdk.internal.access=ALL-UNNAMED`.
  Pin `espresso-core` 3.7.0 and `androidx.test:core` 1.7.0, and set
  `buildToolsVersion = "37.0.0"`.
- AGP 9 compiles Kotlin itself: don't apply `kotlin-android` in modules.
- Untestable here: touch feel, frame timing, BLE, haptics, audio. Those need the owner's phone
  via the CI APK.

## Suggested plan

1. **Setup:** finish the open decisions above; Gradle skeleton (minSdk 24); CI workflow that
   builds the APK and publishes the rolling pre-release; SessionStart hook running
   `setup-android-sdk.sh`.
2. **Domain model:** `CONTEXT.md` with the Roux glossary (FB, SB, FS, SS, DR, LP, CMLL, NMCLL,
   LSE, EO, EOLR(b), MC, 4c, x2y/CN, AUF), via `domain-modeling`.
3. **Spec and tickets:** turn [scope.md](scope.md) R1 and [rouxtine-design.md](rouxtine-design.md)
   into a spec, then into tickets in GitHub Issues.
4. **Core, test-first:** port `CubieCube` / `Move` / `MoveSeq` with the parser fix noted in
   the feature map, the masks and the random-state generator. Port the existing TS tests
   (`src/lib/*.test.ts`) as the spec.
5. **Look and feel early:** theme from a seed colour (`md3e`'s `scripts/generate_theme.py`),
   `MaterialExpressiveTheme`, navigation skeleton, Canvas cube with masking and turn
   animation. Roborazzi baselines in light and dark from day one, and critique each new screen
   PNG.
6. **Module contract, attempt log and skill model** (pure Kotlin, JVM-tested), then the
   Rouxtine engine: blocks, selectors, runs, Today, templates, sharing.
7. **R1 modules** in this order: TIM, CMX/CMR, EO/EOLR/L4C/LSX, FBT/INS/FLP, SS/SLP, BEG.
   Then analysis, suggestions, generation, onboarding, streak/goal, Learn screens.
8. **R2+:** smart cubes (QiYi, then GAN Gen4), slow solves, reconstruction study, more
   modules ([scope.md](scope.md)).

## Suggested skills (call via the Skill tool)

- **Deciding and planning:** `mattpocock-skills:grilling` (open decisions),
  `mattpocock-skills:domain-modeling` (CONTEXT.md), `to-spec` / `to-tickets` /
  `wayfinder` (spec → issues), `mattpocock-skills:codebase-design` (module seams: cube core,
  solver, trainers, UI).
- **Building:** `mattpocock-skills:tdd` (solver/cube port), `mattpocock-skills:diagnosing-bugs`.
  From `android-skills`: `navigation-3`, `edge-to-edge`, `adaptive`, `testing-setup`,
  `agp-9-upgrade`. From `chrisbanes-skills`: `compose-component-design`, `compose-animations`,
  `compose-state-and-effects`, `compose-ui-testing-patterns`.
- **Design:** `md3e` (M3 Expressive API and theme generator), `material-design-3-ui` (design
  judgment), then `design-critique` and `accessibility-review` on Roborazzi PNGs, read with
  Android rules (see VENDORED.md).
- **Writing:** `humanizer` for every piece of user-facing text: in-app strings, lessons,
  store listing, release notes.
- **Environment:** `session-start-hook` to wire `scripts/setup-android-sdk.sh`;
  `mattpocock-skills:research` for any new research question.

## Don't

- Don't add non-Roux content: no CFOP/ZZ algorithms, methods or scramble types.
- Don't vendor skills or code without a licence. `charliesbot/android-design` and
  `gdlbo/material-skill` have no licence, so all rights are reserved.
- Don't use TNoodle (AGPL) or unlicensed alg data (SpeedCubeDB, Kian's sheets) without
  permission. Use onionhoney's GPL alg data, solver-generated algs or user-entered algs.
- Don't add ads. Never paywall a trainer, the solver, stats or analysis. No donate link or
  wording in the `play` build. No Play Billing until onionhoney's written permission is in hand.
- Don't portrait-lock the timer. targetSdk 36+ ignores orientation locks on screens 600dp
  and wider.
