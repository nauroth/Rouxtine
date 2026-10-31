# Handoff: Rouxtine, a native Android Roux trainer

Started 2026-10-28 in `nauroth/roux-trainers` (research session), moved to this repo the same
day. Read this first, then the research notes it links to. The notes hold the evidence; this
file holds the decisions and the plan.

Paths like `src/lib/Pruner.tsx:471` in these docs refer to
[nauroth/roux-trainers](https://github.com/nauroth/roux-trainers) at `0129460`, the fork of
onionhoney's web Roux Trainers (GPL-3.0) that this app ports from. Attach that repo read-only
when porting.

## Next session: start here (paused 2026-10-29, owner out of usage)

1. **Review branch `wip/audit-fixes`** (`git diff main...wip/audit-fixes`). It holds a pre-code
   audit's fixes to HANDOFF, scope, design and research notes, applied by an agent and **not
   reviewed**; the run was stopped before its check pass. Raw findings are in
   `audit-data/audit-results.json` on that branch. Keep what is right, confirm no owner
   decision changed, merge into `main`.
2. **Apply the note corrections** listed in
   [related-projects.md "What changes in our plans"](research/related-projects.md) (min2phase
   licence wording, CaseVisualizer, a few line fixes), unless the audit branch already did.
3. **Bring the owner the final questions** from the audit, plus these from the new notes:
   R1 with no closed Google library and the onionhoney permission request
   ([play-billing-gpl.md](research/play-billing-gpl.md) §9); crash/analytics setup and its
   owner questions ([crash-analytics.md](research/crash-analytics.md)). Then start coding
   (plan below).

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
  adaptation inside blocks, suggestions with reasons across blocks; share by link, code, QR or
  file; daily streak with freezes plus a weekly goal. All of it in the first release.
- **Scope:** **[scope.md](scope.md)** says what ships in R1, R2+, later, and never. It
  supersedes the priorities in the feature-map notes.
- **Offline-first:** every trainer, stats and Rouxtine feature works with no network. No
  accounts and no server of our own. The only network use is opt-in crash and usage reports
  (and, Later, an opt-in template refresh from GitHub).
- **Repo:** `nauroth/rouxtine`, public, not a fork.
- **Name:** Rouxtine. **Package id:** `io.github.nauroth.rouxtine`, with debug builds on
  `io.github.nauroth.rouxtine.debug` so both can be installed at once. No domain.
- **Money (decided 2026-10-29 in a parallel session, replacing "Ko-fi only"):**
  - The app is **free** with **zero ads**. Every training function is free: never paywall a
    trainer, the solver, stats or analysis.
  - Paid extras are only **cosmetic** (a one-time **Supporter** unlock for themes) or a
    **specific add-on** (possibly a paid smart-cube module), sold through **Play Billing** in
    the `play` flavour, with the 15 % fee tier enabled. What a smart-cube add-on may contain
    is open (Open, question 5).
  - **Ko-fi lives off Play only:** GitHub README, `foss` build, F-Droid metadata. No donate link
    or wording in the Play build, and no link to a page that shows one.
  - **Blocker before the first paid feature:** Play Billing is a closed library. Shipping it in
    one APK with onionhoney's GPL code needs onionhoney's written permission (a GPL s.7
    additional permission). Verified in [play-billing-gpl.md](research/play-billing-gpl.md):
    the blocker stands, and onionhoney is the only copyright holder of the code to port. The
    same holds for every closed Google library (Firebase and Crashlytics, ML Kit and the GMS
    code scanner, Play Core review/update, `play-services-oss-licenses`): none goes into any
    APK until a permission covers it, and CI enforces that from plan step 1. Background:
    [licensing-monetization.md](research/licensing-monetization.md) and
    [monetization.md](research/monetization.md).
  - **Contributors are copyright holders too.** Before the first outside pull request is
    merged, templates included, CONTRIBUTING.md must state the inbound terms (Open,
    question 6). Otherwise each contributor becomes one more person to ask.
  - **Play account:** personal. Once anything is sold, Google shows the developer's full address
    publicly; the owner accepts this. Every account also declares trader or non-trader status
    when it is created, and from 2027-03-01 Play shows an individual trader's phone number too
    ([Play help](https://support.google.com/googleplay/android-developer/answer/17594752);
    Open, question 10). Local business, tax and social-security setup, and a check of the EU
    Cyber Resilience Act (Open, question 9), are the owner's to arrange before the first paid
    feature ships.
- **Crash reporting and analytics (decided 2026-10-29):** wanted, for fixing bugs and improving
  the app. Opt-in, open-source SDK where possible, plus Play's Android vitals. No advertising
  ID, nothing personal, everything declared in the Play Data safety form. Which tool: open; the
  recommendation is in [crash-analytics.md](research/crash-analytics.md) (Open, question 12).
  What follows from these decisions:
  - Nothing leaves the phone before the user agrees: no SDK starts (auto-init off), nothing is
    queued for upload, no ID is created. Crash reports and usage stats are separate choices,
    both changeable in Settings. Turning one off stops sending and deletes queued data and any
    ID.
  - Reports never carry user-entered text (Rouxtine names, notes, share codes) or attempt
    times. Debug and CI builds never send.
  - "Nothing personal" in practice: no account, no advertising ID, no user-entered content. The
    receiving server still sees IP addresses, so the owner acts as data controller: a privacy
    policy, the vendor's processing agreement, EU hosting, IP storage off.
  - Only FLOSS SDKs, since closed Google libraries are barred (see "Money"). The `foss` build can
    then carry the same code and keep "no proprietary libraries".
- **Licence:** GPL-3.0-only (SPDX `GPL-3.0-only`), matching onionhoney's upstream exactly. GPL
  is required because the app ports their GPL solver and data. **Duties in every published
  build** ([licensing-monetization.md §5](research/licensing-monetization.md)):
  - an SPDX header and copyright lines on every file, keeping onionhoney's line on ported files;
  - a dated port notice on each ported file, e.g. "Ported to Kotlin from onionhoney/roux-trainers
    `src/lib/Solver.tsx` @0129460 on <date>";
  - the licence text viewable in the app, and an About screen with the no-warranty notice and
    the MIT and Apache-2.0 notices (min2phase, AndroidX, Material Symbols);
  - a tag and a source archive for every published build; the About screen shows the
    versionName, plus the commit SHA in `dev` builds;
  - the Corresponding Source includes the CI workflow and any pruning-table generator;
  - no EULA or terms that restrict what the GPL allows.
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
  the first screens exist. (scope.md says "once R1 strings settle"; Open, question 6.)
- **Distribution:** **Google Play is the main release** (`play` flavour, no donation link),
  plus a signed APK on GitHub releases (`foss` flavour: no proprietary libraries, everything
  unlocked, Ko-fi link), which also serves Obtainium users. Obtainium is not a store: it reads
  GitHub releases, so there is nothing to publish there. One app signing key for both, uploaded
  to Play App Signing, so users can switch between Play and GitHub installs. (The F-Droid main
  repo signs with its own key unless the build is reproducible.) A new personal Play account
  needs $25, ID, and a closed test with 12 testers for 14 days before production.
- **Signing:** keys live in GitHub Actions secrets, never in the repo. The debug key goes in
  four repo secrets. The release key goes in environment secrets behind the owner's approval,
  is generated on the owner's computer, and gets an offline backup.
- **Smart cubes to buy:** a QiYi AI 3x3 and a GAN i4, so QiYi and GAN Gen4 can both be tested.
  Smart-cube support ships after R1 ([scope.md](scope.md) R2+).
- **Content scope:** teach from zero (own text); one neutral lesson for cubers switching from
  other methods; Roux and everything that applies to Roux, nothing else (no glossary entries
  for other methods); videos linked, not embedded, low priority; third-party material only with
  consent, and inventors credited where their technique is taught; adapt Roux Reader sections
  (MIT) with notice; U-sticker + pattern CMLL recognition with standard names; OH hand only
  with OH later; community links maybe later; no mindset content. M-hand is an agent default
  for now (see "Defaults" and Open, question 21).
- **Builds onto the phone:** the owner prefers whatever is safest and most widely used.
  Confirmed by research: CI builds both flavours of each `main` build and publishes the
  `foss` debug build to a rolling GitHub pre-release (see "Defaults"), and Obtainium on the
  phone tracks it ("Include prereleases" and "Use release date as version string" on). Plain
  download from the release page is the fallback, with no login and no zip. The owner must turn
  **Samsung Auto Blocker** off (and keep its auto-re-enable off), or every install outside
  Play/Galaxy Store is blocked. Details:
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

Grilling finished 2026-10-29. The pre-code audit the same day checked the docs against the
research and raised the questions below. Each has a recommendation after the arrow. None blocks
the first code (plan steps 1–5) unless it says so.

**Questions for the owner**

*Licensing and money* (evidence: [play-billing-gpl.md](research/play-billing-gpl.md)):
1. **Write to onionhoney now?** As a public GitHub issue, an email to the address on their
   commits, or both, and under which name? → Now, as a public issue on onionhoney/roux-trainers
   (a written public record), optionally pointed to by email, under your own name. The draft is
   in play-billing-gpl.md §9. Offer a pull request that adds the licence line and the
   permission to their README, so merging it is the record. Don't wait for the answer.
2. **What to ask for.** Play Billing only, every Google Play, Play services and Firebase
   library, an app-store permission, the right to name them in the listing? → The Play Billing
   permission as drafted, plus a Terpoir-style app-store permission (so Play's terms don't count
   as further restrictions) and consent to name "Roux Trainers"/onionhoney in the store listing.
   Skip the wider library permission: R1 needs none of those libraries (question 7).
3. **If onionhoney says no, or never answers?** → Sell the Supporter themes as a separate paid
   "Supporter key" app with no billing library (play-billing-gpl.md §7, d1), and never ship
   Play Billing.
4. **Offer onionhoney anything beyond credit?** → Credit in the app, the listing and every
   ported file, and the Supporter unlock for free. Leave a revenue share for them to raise.
5. **What may the paid smart-cube add-on contain?** Every smart-cube feature in scope is a
   trainer input, a module, stats or analysis, which "never paywall" rules out, and
   monetization.md's candidates were all analytics. → Keep protocols, the SC module, automatic
   splits, smart-cube input and all smart-cube stats free in both builds. Drop the add-on, or
   limit it to cosmetic extras such as cube skins or replay themes, so the Supporter themes stay
   the paid item.
6. **Contributor and template terms; Weblate timing.** → Code and translations: GPL-3.0-only
   with a DCO sign-off, plus the same s.7 permission asked of onionhoney, from the first commit
   (play-billing-gpl.md §9 step 3). Templates: CC0-1.0, so share codes and template files can
   travel anywhere and template authors never need to be asked. Weblate: start once R1 strings
   settle, as scope.md says, rather than when the first screens exist.
7. **R1 without closed Google libraries, even if onionhoney says yes in time?** → Yes: FLOSS
   reporting tools, ZXing for QR, AboutLibraries for the licences screen, no Play Core review
   or update prompts. The build stays simple and F-Droid-clean, and R1 doesn't wait on anyone.
8. **min2phase.** → Port Chen Shuang's Java under its 2023 MIT grant, and send Chen a one-line
   request to confirm it also covers the GPL-headed `Search.java`. Don't wait on the reply.
9. **EU Cyber Resilience Act.** Selling an unlock probably makes Rouxtine a product supplied in
   a commercial activity: manufacturer duties from 2027-12-11, and reporting of actively
   exploited vulnerabilities since 2026-09-11. Donations alone don't count. → Keep R1 and the
   `foss` build free of any sale. Before scope.md R2+ item 11 starts, get a short legal opinion.
   If the CRA applies, budget the work (an SBOM in CI, SECURITY.md with a disclosure address, a
   five-year update commitment, the declaration of conformity) or keep money to off-Play
   donations.
10. **Trader status and phone number.** You accepted that the address is shown. From
    2027-03-01 Play also shows an individual trader's phone number. → Declare non-trader at
    account creation while nothing is sold on Play, if your own assessment agrees. Switch
    before the first paid product, once you are OK with the phone number being public.
11. **Ko-fi scope.** The parallel session said "Ko-fi on GitHub only"; "Money" reads it as the
    README, the `foss` build and the F-Droid metadata. → Keep that reading: all three are off
    Play. Add the README line once the Ko-fi page exists. A `.github/FUNDING.yml` Sponsor button
    is optional; it changes nothing for Play, because the `play` build never links to the repo
    page.

*Privacy, crash reports and analytics* (evidence: [crash-analytics.md](research/crash-analytics.md)):

12. **Tools.** → Option A: Android vitals; crash reports through `sentry-android-core` (MIT) to a
    new Sentry organisation in the EU region, on the free plan; usage stats through a small
    in-house client that posts anonymous events to PostHog Cloud EU. The same code in both
    flavours. GlitchTip's hosted EU plan is the fallback (only the DSN changes), for example if
    an F-Droid reviewer objects to Sentry's source-available server. Firebase is out.
13. **Play target audience, age and country.** → Declare 13–15, 16–17 and 18+, not under 13.
    Put a neutral age picker (not a leading "Are you 16 or older?") in front of the reporting
    offers. Offer crash reports and usage stats only from 16, or your country's age of digital
    consent; younger users get Android vitals only. Which EU country do you live in? It sets that
    age (13–16) and your data-protection authority. Outside the EEA, a GDPR art. 27
    representative may be needed. Before R1, also check whether app-store age-signal laws
    (Texas SB 2420) apply; Play's Age Signals library is closed.
14. **Crash-report default.** → "Ask me after a crash": the crash goes to a local file, and on
    the next launch a sheet asks before anything is sent. Sentry doesn't start in this mode.
15. **Usage stats in R1?** → Yes, as scoped, built after crash reports.
16. **Retention numbers?** They need a stored, resettable random ID, declared as "Device or
    other IDs". → No ID: anonymous events only.
17. **A privacy policy under your name.** Play needs one for every app, and you become the data
    controller (accepting Sentry's and PostHog's processing terms). → Yes, on
    `nauroth.github.io`, with a contact email and no Ko-fi.
18. **`-dontobfuscate` in release builds** (shrinking kept), so stack traces read without
    uploading mapping files? → Yes. The source is public anyway.

*Distribution and release*:

19. **".apk on GitHub should be enough no? Or publish on Obtainium and F-Droid also?"**
    Obtainium needs nothing: it reads GitHub releases (a README "Add to Obtainium" badge is
    optional). → R1: Play plus the GitHub APK. After R1, IzzyOnDroid first, because it ships our
    own signed APK, so users can still switch. The F-Droid main repo only with reproducible
    builds, so it ships our signature too. Keep the `foss` flavour F-Droid-clean from day one.
20. **When to open the closed test.** → At milestone M5 (plan step 7), with 15–20 testers, so
    the 14 days and the review overlap the last milestone. No public release before R1.

*Content and design*:

21. **M-hand (your Q19.10: "Only necessary for OH module later on no?").** No: M-hand matters
    in two-handed LSE too. EOLR sheets exist for both M hands, and the EO flowchart has lefty
    and righty halves. → Keep M-hand as an R1 setting, right by default. Don't ask it in
    onboarding; ask once, in context, the first time the user opens an LSE lesson or module.
    OH hand waits for the OH module (Later), as you suggested.
22. **Lore (your Q19.12 "No").** scope.md still lists "Lore and timeline cards" under Later. →
    Read "No" as covering lore too: move them to Out, with no mantra field and no Midori joke.
    Plain inventor credits in lessons and the glossary stay (Q19.7).
23. **Streak plus weekly goal ("Or do both?").** With a 3-day goal, a user who meets it misses
    four days a week, so the two clash unless rest days are defined. → Flexible rest days
    ([rouxtine-design.md §8](rouxtine-design.md)): a missed day is a rest day while the week
    can still reach its goal; freezes cover the rest; one freeze earned per week the goal is
    met, at most 2 banked.
24. **Backup and moving to a new phone.** With no accounts, all history lives on the phone. →
    Both in R1: Android Auto Backup, set to keep only the source-of-truth tables (under its
    25 MB cap) and rebuild caches after a restore; and a manual "Export all data" / "Import
    data" zip through the Storage Access Framework. No cloud sync.

**Owner to-dos (none block the first code):**
- Buy a QiYi AI 3x3 and a GAN i4.
- Turn off Samsung Auto Blocker on the S25 when the first APK is ready.
- When CI is set up: generate the dev key on your own computer with the `keytool` command in
  [min-sdk-and-delivery.md §B8](research/min-sdk-and-delivery.md), then paste the four secrets
  into GitHub.
- Create the repo `nauroth/nauroth.github.io` with Pages on. It hosts the privacy policy, the
  share-link page and the App Links file. No Ko-fi or donate wording there.
- Write to onionhoney (question 1). It is no longer optional while paid extras on Play are
  planned, and it should happen early, because a "no" changes the money plan. Record the reply
  in the repo (e.g. `LICENSING.md`).
- Create the Ko-fi page, then add the README line (question 11).
- Before going public, in this order:
  1. Create the Google Play developer account ($25, ID), declaring trader or non-trader status
     at creation (question 10).
  2. Publish the privacy policy on `nauroth.github.io`; link it in Play Console and in
     Settings > About in both builds.
  3. Complete Play's App content: privacy policy URL, Data safety, IARC content rating, target
     audience (question 13), ads = none, app access.
  4. Start the closed test with 15–20 testers (the Roux Discord is a good place), so drop-outs
     don't take it below 12. The 14 days must be consecutive; a tester who leaves and rejoins
     starts over.
  5. Allow 14 days of closed test plus up to 7 days of production review before R1.
  6. Generate the release key on your own computer.
  7. Register `io.github.nauroth.rouxtine` for developer verification before the 2027 rollout,
     and `.debug` with the dev key's SHA-256 (a limited-distribution account), or plan on the
     advanced flow for the dev channel.

  Until steps 2–3 are done, the `play` flavour can be tried on Play's internal testing track,
  which doesn't need the Data safety form.
- Before the first paid feature: onionhoney's written permission, trader status (question 10),
  the CRA check (question 9), and your business, tax and social-security setup.
- Later: ask content authors for consent ([roux-community-resources.md §10](research/roux-community-resources.md) item 7).

**Defaults the agent chose (the owner may override):**
- Smart cubes in R2, not R1, to keep R1 shippable.
- Sharing (link, code, QR, file) in R1, because templates use the same format.
- Community templates are bundled per release; no online fetch in R1. Suggestions come in
  through a GitHub issue form, and maintainers turn them into pull requests.
- M-hand as an R1 setting (right by default, mirrors LSE solutions, not asked in onboarding),
  until question 21 is answered.
- F-Droid after R1 (scope.md Later), until question 19 is answered.
- CI publishes `fossDebug` to the `dev` pre-release, because GitHub carries the `foss` flavour;
  `playDebug` is built and tested but not published. Both use the `.debug` id, so only one can
  be installed at a time.
- Both flavours of a release get the same versionCode, since they share the package id and key.
- The dev key is generated by the owner, not by an agent, so it exists only on the owner's
  computer and in GitHub secrets.
- The R1 placement test leaves out EOLR recognition, because the R1 EOLR module has no
  recognition mode (two-step EOLR recognition is R2+).
- Everything marked *(default)* in [rouxtine-design.md](rouxtine-design.md): data model, run
  lifecycle, ids and versions, wire format, selector, suggestion, generator, Today, day and
  accessibility rules.

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
  - `licensing-monetization.md`: Ko-fi on Play/F-Droid/GitHub, GPL duties for every build. Its
    "Ko-fi only, no billing" decision is superseded by "Money".
  - `min-sdk-and-delivery.md`: minSdk floor, S25 profile, CI-to-phone pipeline, signing,
    developer verification, Samsung Auto Blocker.
  - `smart-cubes.md`: cube to buy, protocol matrix and order, Android BLE, timing, code to port.
  - `monetization.md`: Play options (donations, Supporter unlock, tips, paid, ads), account
    setup, GPL precedents. Written in a parallel session; corrected in part by
    `play-billing-gpl.md` §8.
  - `play-billing-gpl.md`: whether Play Billing may ship next to onionhoney's GPL code (not
    without their s.7 permission), who holds the copyright, other closed Google SDKs, the
    options, and the draft message to onionhoney.
  - `crash-analytics.md`: crash and usage reporting for both flavours (Sentry EU and an
    in-house PostHog client recommended), consent UX, Data safety, F-Droid rules, EU consent
    law.
  - `extra-skills.md`: nine more skills reviewed; only Humanizer installed.
  - `play-billing-gpl.md`: Play Billing needs onionhoney's written s.7 permission; R1 ships with
    no closed Google library; draft request; paid "Supporter key" app as the fallback.
  - `crash-analytics.md`: Android vitals + Sentry EU for crashes, in-house client to PostHog EU
    for usage stats, opt-in and off by default; F-Droid, consent and Data safety.
  - `related-projects.md`: eleven GitHub repos the owner found, each with a verdict
    (min2phase: port; Roux Reader: reuse; five: borrow ideas; two: reference; two: skip).
- `docs/scope.md` and `docs/rouxtine-design.md`: the decided scope and Rouxtine design.
- Branch `wip/research-data`: the structured intermediate outputs (claims, verifier verdicts,
  crawl summaries, YouTube index) and workflow scripts behind those notes. Raw page texts were
  not kept (most sources have no licence).
- `scripts/setup-android-sdk.sh`: idempotent SDK + Gradle-mirror setup (52 s fresh, 0 s warm).
  Not wired up yet: hook it into a SessionStart hook or the environment setup script.
- `.claude/settings.json`: enables the plugins `ponytail`, `android-skills` (Google, official),
  `chrisbanes-skills` and `humanizer`. Loading is not verified yet. In the first session rooted
  at this repo, check that `/plugin` lists `humanizer@humanizer` and note the date here. If it
  doesn't load, vendor blader/humanizer (MIT) into `.claude/skills/humanizer` with an entry in
  VENDORED.md, as was done for md3e.
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

Each milestone (M0–M6) ends with a `dev` build the owner checks on the S25 through Obtainium.
R1 stays the first public release.

1. **Setup (M0):** Gradle skeleton (minSdk 24) with the `play` and `foss` flavours. CI runs
   `assembleFossDebug assemblePlayDebug`, tests and lint, and publishes
   `app/build/outputs/apk/foss/debug/app-foss-debug.apk` as `rouxtine-dev.apk` to the `dev`
   pre-release. CI guards: fail if any runtime classpath of either flavour holds an artifact
   from `com.google.android.gms`, `com.google.firebase`, `com.google.android.play`,
   `com.google.mlkit` or `com.android.billingclient`; fail if the `play` bundle contains
   "ko-fi" or "donat"; an SPDX check (e.g. REUSE lint); a DCO check. An About/Licences stub.
   CONTRIBUTING.md with the inbound terms once question 6 is answered, and in any case before
   the first outside pull request is merged. SessionStart hook running
   `setup-android-sdk.sh`.
2. **Domain model:** `CONTEXT.md` with the Roux glossary (FB, SB, FS, SS, DR, LP, CMLL, NMCLL,
   LSE, EO, EOLR(b), MC, 4c, x2y/CN, AUF), via `domain-modeling`. Define every case key by cube
   state and record the counts there ([scope.md](scope.md), canonical case ids) before any
   table uses them.
3. **Spec and tickets:** turn [scope.md](scope.md) R1 and [rouxtine-design.md](rouxtine-design.md)
   into a spec, then into tickets in GitHub Issues.
4. **Core and solver, test-first (M1):** port `CubieCube` / `Move` / `MoveSeq` with the parser
   fix noted in the feature map, the masks and the random-state generator; min2phase from
   Chen Shuang's Java (MIT) for scrambles; the Roux solver with sparse tables. Port the existing
   TS tests (`src/lib/*.test.ts`) as the spec, and add golden tests against the TS code. Add a
   benchmark build for the S25 (table build time, memory) that also times the INS searches:
   `analyze_fb` over x2y and CN, and FB+DR from scratch with the sparse `fbdr` table. Porting
   notes: generate LSE/EO/EOLR/4c cases (MC included) as target states, then derive the
   scramble, never by inverting truncated solutions; pad min2phase scrambles with M-slice moves
   for offset centres; check the `lse` pruner's `edge_encode` (`src/lib/Pruner.tsx:883`) with a
   golden test.
5. **Look and feel early (M2):** theme from a seed colour (`md3e`'s `scripts/generate_theme.py`),
   `MaterialExpressiveTheme`, navigation skeleton, Canvas cube with masking and turn
   animation, then TIM with sessions and export as the first touch-feel check on the phone.
   Roborazzi baselines in light, dark and font scale 2.0 from day one, and critique each new
   screen PNG.
6. **Module contract, attempt log and skill model (M3)** (pure Kotlin, JVM-tested), with a
   `SimulatedLearner` that writes realistic attempt logs (per-case log-normal times with
   learning curves, error rates, forgetting on the FSRS default curve, schedules with breaks,
   time-zone changes, penalty amendments). Golden scenario tests on top (e.g. 30 days of slow
   EOLR recognition → finding → suggestion → generated Rouxtine) and property tests (no finding
   below thresholds, suggestion limits kept, generation deterministic, streak ledger stable under
   amendments). A debug-only "Load demo history" uses the same generator for screenshots. Then
   the `:format` module (template schema, loader, share codec, CI validator; no template content
   yet), Quick sessions, CMX/CMR, and the Recap, Equal, Weakest and SRS selectors.
7. **The rest of R1:**
   - **M4:** the other R1 modules (EO/EOLR/L4C/LSX, FBT/INS/FLP, SS/SLP), Ladder, builder,
     runs, Today, sharing.
   - **M5:** Learn infrastructure first (lesson renderer, glossary generated from `CONTEXT.md`),
     then BEG and the other lessons; onboarding and placement; the 9 R1 templates;
     streak/goal/reminders; consent with crash and usage reports; the privacy policy.
   - **M6:** analysis, suggestions, generation, polish.
   - Release: the Play steps under "Before going public". R1 ships once the closed test has run
     14 days and production access is granted.
8. **R2+:** smart cubes (QiYi, then GAN Gen4), slow solves, reconstruction study, more
   modules ([scope.md](scope.md)). Before TRK, check that upstream's hidden `TrackerView`
   still works.

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
  store listing, release notes. Licence texts, copyright and permission notices, and quoted
  legal wording are copied verbatim, never humanized. Privacy-policy and consent text may be
  humanized, then checked against the Data safety answers.
- **Environment:** `session-start-hook` to wire `scripts/setup-android-sdk.sh`;
  `mattpocock-skills:research` for any new research question.

## Don't

- Don't add non-Roux content: no steps, whole alg sets, lessons, comparisons, glossary entries
  or scramble types from other methods (CFOP, ZZ…). Single algs used inside a Roux step are
  fine, named for their Roux use: Sune in 2-look corners, a corner swap a switcher may know as
  a T- or J-perm, OLLCP algs for Pinkie Pie. Roux-based big-cube methods (Meyer, Stadler, CR4)
  belong only in the Later big-cube pack.
- Don't vendor skills or code without a licence. `charliesbot/android-design` and
  `gdlbo/material-skill` have no licence, so all rights are reserved.
- Don't use TNoodle (AGPL) or unlicensed alg data (SpeedCubeDB, Kian's sheets) without
  permission. Use onionhoney's GPL alg data, solver-generated algs or user-entered algs.
- Don't copy code from sources that would add copyright holders to the Play question:
  onionhoney's open PR #31 (write the M-hand setting independently), AJGDsn's smartcube-gym,
  csTimer. Use them for ideas only.
- Don't add ads. Never paywall a trainer, the solver, stats or analysis. No donate link or
  wording in the `play` build. No Play Billing, or any other closed Google library, until
  onionhoney's written permission covers it.
- Don't link the repo page from the `play` build or its listing (the README shows Ko-fi). Give
  the source as plain text plus the tag's source-archive link. Never say there that the GitHub
  build is free or unlocked.
- Don't put "Rubik" or "Rubik's" in the store title, short description, keywords, icon or
  screenshots; say "3x3 cube" or "speedcube". Name other products (Twisty Timer, Cubic Timer,
  GAN, QiYi, WCA) only in factual compatibility sentences in the full description. Credit
  onionhoney in the About screen, and by name in the listing once they agree (question 2).
- Don't portrait-lock the timer. targetSdk 36+ ignores orientation locks on screens 600dp
  and wider.
