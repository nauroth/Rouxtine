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

- **Repo:** `nauroth/rouxtine`, public, not a fork.
- **Name:** Rouxtine. **Package id:** `io.github.nauroth.rouxtine`, with debug builds on
  `io.github.nauroth.rouxtine.debug` so both can be installed at once. No domain.
- **Money:** a Ko-fi link in the app, nothing more. No paywalled features, no ads, no in-app
  purchases, so no Play Billing library. Whether a donation link is allowed in a Play build is
  being checked (see "Open").
- **Licence:** GPL-3.0, required because the app ports onionhoney's GPL solver and data.
- **minSdk:** as low as the stack allows. Owner's test phone: **Samsung Galaxy S25, Android 16
  (One UI 8)**; use it as the Roborazzi device profile.
- **Audience and curriculum:** drills and training for every Roux level. The owner's own level
  doesn't matter. Content research covers rouxcubing.com, resources.rouxers.com and everything
  they link (onionhoney's trainers are already known).
- **Smart cubes:** support many Bluetooth protocols. The owner has none yet and will buy a cheap
  one for testing.
- **Languages:** English at launch. All UI strings in resources from day one, ready for a
  community translation platform (Weblate or Crowdin, both free for open source).
- **Builds onto the phone:** the owner prefers whatever is safest and most widely used.
  Proposed: CI publishes each `main` build to a rolling GitHub pre-release, and Obtainium on the
  phone tracks it. Plain download from the release page is the fallback, with no login and no
  zip. Debug builds are signed with a fixed key. Verification pending (see "Open").
- **Stack:** Kotlin + Jetpack Compose + Material 3, accepted as researched in
  [android-ui-stack.md](research/android-ui-stack.md): AGP 9.5.0, Kotlin 2.4.21, Compose BOM
  2026.09.00, Navigation 3, Hilt + KSP, Room 3, DataStore, Vico, Kable later for BLE.
- **Material 3 Expressive:** required. Pin `androidx.compose.material3:material3:1.5.0-beta02`
  over the BOM; stable 1.4.0 lacks `MaterialExpressiveTheme`, `MotionScheme.expressive()` and
  the new components. Some APIs (`MaterialShapes`, `LoadingIndicator`) need an experimental
  opt-in.
- **Feature map:** accepted as proposed in
  [roux-training-features.md §2](research/roux-training-features.md) (MVP / v1 / later per
  Roux stage). It will be extended with the community-site research.
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

Research for these is in progress (notes will land in `docs/research/`):

1. **Licence wording:** GPL-3.0-only (recommended, matches upstream exactly) or -or-later.
2. **Donation link on Play:** whether Google Play allows a Ko-fi link, by region. If not, the
   Play build drops it and other builds keep it.
3. **minSdk number:** the binding floor of the pinned libraries (probably 23), and what each
   lower level costs.
4. **Signing and delivery details:** fixed debug key committed vs kept as a GitHub secret,
   Obtainium status, Google's developer-verification rules for sideloaded apps, Samsung Auto
   Blocker.
5. **Which smart cube to buy first,** and the order to implement protocols.
6. **Curriculum and content scope:** from the community-site analysis, e.g. teaching from zero,
   OH, embedding vs linking videos, content that needs permission.
7. **Translation platform:** Weblate or Crowdin, once UI strings settle.

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

1. **Setup:** finish the open decisions above; Gradle skeleton; CI workflow that builds the APK
   and publishes the rolling pre-release; SessionStart hook running `setup-android-sdk.sh`.
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
