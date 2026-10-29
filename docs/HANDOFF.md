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
  they want to progress on and what they need to get better at. All training hangs off this;
  the details are being settled (see "Open").
- **Repo:** `nauroth/rouxtine`, public, not a fork.
- **Name:** Rouxtine. **Package id:** `io.github.nauroth.rouxtine`, with debug builds on
  `io.github.nauroth.rouxtine.debug` so both can be installed at once. No domain.
- **Money:** a Ko-fi link in the app, nothing more. No paywalled features, no ads, no in-app
  purchases, so no Play Billing library. Research: Google Play bans in-app donation links to
  the developer in every region, so a Play build would need a `play` flavour without the link;
  F-Droid and GitHub builds may keep it
  ([licensing-monetization.md](research/licensing-monetization.md)).
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
- **Smart cubes:** support many Bluetooth protocols. The owner has none yet and will buy a cheap
  one for testing.
- **Languages:** English at launch. All UI strings in resources from day one. Community
  translations through **Weblate** (Hosted Weblate's free plan for libre projects), set up once
  the first screens exist.
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
- **Feature map:** accepted as proposed in
  [roux-training-features.md §2](research/roux-training-features.md) (MVP / v1 / later per
  Roux stage). The community-site research proposes changes and new rows in
  [roux-community-resources.md §9](research/roux-community-resources.md); not yet accepted.
- **Timer:** the app keeps **its own timer and stats** (the source of truth) and has an
  **"Export to Twisty/Cubic Timer"** action. It writes Twisty's *backup* `.txt` format, which
  both apps import, with categories like "Roux FB" / "CMLL". Format, gotchas and dedupe rules
  are in [timer-integration.md](research/timer-integration.md) (sections A, "Generator
  gotchas"). Zero-tap insertion is impossible: neither app exposes an entry point.
- **Virtual cube:** custom renderer on Compose `Canvas` with pure-Kotlin 3D math, ported from
  the web app's `src/components/CubeSim.tsx`. It is the only option that shows up in JVM
  screenshot tests. No OpenGL, Filament or WebView for the MVP.
- **Solver:** port the TS solver to Kotlin with sparse/hashed pruning tables. Do not embed the
  JS: its dense tables allocate 191–382 MB. Evidence:
  [roux-training-features.md §5](research/roux-training-features.md).

## Open: settle with the owner before building

Grilling is in progress (`mattpocock-skills:grilling`). Research for every item below has
landed in `docs/research/`. Question numbers continue the session's numbering.

**Asked, waiting for the owner's answer** (the recommendation is in brackets; the regimen
research in [training-regimens.md §5](research/training-regimens.md) backs each one):

- **Q11 What "toggleable" means** [both: an app-wide toggle hides a module everywhere;
  Rouxtines compose modules; any visible module also runs alone as a Quick session].
- **Q12 Who builds a Rouxtine** [templates and a hand builder in the first release, picked by a
  short placement test; app-generated Rouxtines later, opt-in only].
- **Q13 How much the app adapts** [automatically inside a block (weak cases more often, SRS);
  across blocks only suggestions with a reason, which the user accepts; a saved Rouxtine never
  changes silently].
- **Q14 Rouxtines in the first release?** [yes, simple: run as a guided session, edit blocks, a
  few templates; every trainer built as a module from day one].
- **Q15 Sharing** [v1: versioned text code with checksum, also as link, file and QR; structure
  only, no stats; no server].

**Next round, not yet asked:**

1. **Signing, reopened.** Research corrects the earlier "commit the debug keystore" choice:
   with a public key anyone can sign an APK that updates the owner's `.debug` app, and from
   2027 registering `.debug` would tie such APKs to the owner's identity. Recommended: debug
   key in four GitHub Actions secrets, the release key as environment secrets behind owner
   approval, plus an offline backup.
2. **Distribution channels:** Play, F-Droid, GitHub releases, or a mix. Play needs a `play`
   flavour without the Ko-fi link, and the GPL source directions must not link to a page with
   a donate button. Also: register the package names in the Android Developer Console before
   developer verification goes global in 2027 (free limited-distribution account for own
   devices; $25 full account for public distribution).
3. **Which smart cube to buy:** QiYi AI 3x3 first (about EUR 20–23; best-documented protocol,
   cube timestamps). GAN i4 second, or first if only one cube will ever be bought. Protocol
   order and code to port: [smart-cubes.md](research/smart-cubes.md).
4. **Content scope:** the 12 questions in
   [roux-community-resources.md §10](research/roux-community-resources.md) (teach from zero,
   switchers, OH as a profile, big cubes later, the Roux-variant boundary, link vs embed videos,
   permissions to ask, adapting the MIT Roux Reader, recognition naming, M-hand/OH-hand
   defaults, community links, mindset content).
5. **Feature-map delta:** accept or trim the changed and new rows in
   [roux-community-resources.md §9](research/roux-community-resources.md) and the proposed
   module catalogue in §4.1 and [training-regimens.md §5.1](research/training-regimens.md).
6. **Rouxtine internals** (after Q11–Q15): one active Rouxtine driving "Today"; block stop
   rules; a weekly practice-days goal (default 3) instead of a daily streak; day variants for
   30–60 min budgets; placement test shape ([training-regimens.md §5.3–5.9](research/training-regimens.md)).
7. **Write to onionhoney** once (optional, non-blocking): tell them about Rouxtine and ask for
   an explicit licence statement ([licensing-monetization.md §6](research/licensing-monetization.md)).

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
- Branch `wip/research-data`: the structured intermediate outputs (claims, verifier verdicts,
  crawl summaries, YouTube index) and workflow scripts behind those notes. Raw page texts were
  not kept (most sources have no licence).
- `scripts/setup-android-sdk.sh`: idempotent SDK + Gradle-mirror setup (52 s fresh, 0 s warm).
  Not wired up yet: hook it into a SessionStart hook or the environment setup script.
- `.claude/settings.json`: plugins `ponytail`, `android-skills` (Google, official),
  `chrisbanes-skills`. They load at session start.
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
3. **Spec and tickets:** turn the feature map's MVP rows into a spec, then into tickets in
   GitHub Issues.
4. **Core, test-first:** port `CubieCube` / `Move` / `MoveSeq` with the parser fix noted in
   the feature map, the masks and the random-state generator. Port the existing TS tests
   (`src/lib/*.test.ts`) as the spec.
5. **Look and feel early:** theme from a seed colour (`md3e`'s `scripts/generate_theme.py`),
   `MaterialExpressiveTheme`, navigation skeleton, Canvas cube with masking and turn
   animation. Roborazzi baselines in light and dark from day one, and critique each new screen
   PNG.
6. **MVP trainers** in feature-map order: FB + inspection planner, FBLP(+DR), SS/SBLP, CMLL
   (2-look, full 42, recognition drill), LSE (EO, EOLR/EOLRb, 4c). Then the timer, sessions,
   stats and Twisty/Cubic export.
7. **v1:** spaced repetition, curriculum gates, smart-cube BLE with automatic splits,
   reconstruction analyzer, chain flow.

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
- **Environment:** `session-start-hook` to wire `scripts/setup-android-sdk.sh`;
  `mattpocock-skills:research` for any new research question.

## Don't

- Don't add non-Roux content: no CFOP/ZZ algorithms, methods or scramble types.
- Don't vendor skills or code without a licence. `charliesbot/android-design` and
  `gdlbo/material-skill` have no licence, so all rights are reserved.
- Don't use TNoodle (AGPL) or unlicensed alg data (SpeedCubeDB, Kian's sheets) without
  permission. Use onionhoney's GPL alg data, solver-generated algs or user-entered algs.
- Don't add paywalls, ads or in-app purchases. The only monetization is a donation link.
- Don't portrait-lock the timer. targetSdk 36+ ignores orientation locks on screens 600dp
  and wider.
