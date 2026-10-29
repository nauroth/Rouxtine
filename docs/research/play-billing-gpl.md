# Play Billing and the GPL: what a paid unlock on Play needs

Researched 2026-10-29 to answer one question before any code exists: may Rouxtine's `play` build ship Google's Play Billing Library next to code ported from onionhoney's GPL-3.0 Roux Trainers? It also settles the disagreement between [licensing-monetization.md](licensing-monetization.md) and [monetization.md](monetization.md) on that point. `[n]` refers to the numbered sources at the bottom. `LICENSE:n` is a line of the GPLv3 text in this repo's `LICENSE`, which is byte-identical to upstream's [1][14]. **This is research, not legal advice.** No lawyer was consulted and no court ruling was found. Everything here rests on the GPL text and the FSF's FAQ.

**Context (HANDOFF "Money", decided 2026-10-29):** the app is free with zero ads. Paid extras are only cosmetic (a one-time Supporter unlock for themes) or a specific add-on (maybe a paid smart-cube module), sold through Play Billing in the `play` flavour. A trainer, the solver, stats and analysis are never paywalled.

## Summary

- **Answer: no Play Billing without onionhoney's written permission.** Under the GPL text, as the FSF reads it, an APK holding both the closed billing library and onionhoney's ported code is one combined work. Only onionhoney can permit that combination, with a GPL section 7 additional permission (§2). licensing-monetization.md is right. monetization.md is wrong where it files the ask under "etiquette" and cites Thunderbird as a GPL precedent (§8).
- **Why:**
  - The library is closed. Its POM declares the Android SDK License, there is no sources jar, and that licence forbids redistribution and derivative works [4][6].
  - It is not a GPL "System Library". It ships inside each app, not with the OS or a compiler (`LICENSE:123-132`) [2 #WindowsRuntimeAndGPL].
  - The FSF says modules in the same executable are "definitely combined", and a wrapper module in between is "legally irrelevant" [2].
  - Only the copyright holders can grant the exception. A permission that covers only part of the program does nothing for the whole (`LICENSE:349-352`) [2 #GPLIncompatibleLibs].
- **One person to ask.** `git blame` gives every line Rouxtine will port to onionhoney (Zhouheng "Jeffrey" Sun). The only other human author changed a file that won't be ported (§3) [14][15].
- **Upstream is quiet.** The last commit is from 2025-07-27, and the last maintainer comment found is from 2021. A reply may be slow or never come (§3) [14][15].
- **There is no way around the library for in-app purchases on Play.** Every update since 2026-08-31 must use Billing Library 8 or later [7][8]. A paid *download* needs no library, which makes a separate paid "key" app possible (§5) [8][10][11].
- **Other Google SDKs raise the same question.** Crashlytics (through `play-services-tasks`), ML Kit barcode scanning, Google's code scanner, Play Core review/update and `play-services-oss-licenses` all come under closed Google terms. ZXing is Apache-2.0 (§6) [12][13].
- **Recommendation (§9):**
  1. **R1 ships with no closed Google library in either flavour.** It then needs nobody's permission. Paid extras wait.
  2. **Ask onionhoney now, and don't block on the answer.** Use a public GitHub issue (draft in §9), and include the licence statement upstream is missing.
  3. **Set the inbound terms from the first commit**, so contributors and translators never become more people to ask.
  4. **If the answer is no, or never comes,** sell the themes as a separate paid "Supporter key" app with no billing library, and never ship Play Billing. Don't rewrite the GPL code.

## 1. What the Play Billing Library is

- `com.android.billingclient:billing` 9.1.0 is the latest release [4]. Its POM declares the "Android Software Development Kit License". There is no sources jar (HTTP 404). `billing-ktx` 9.1.0 has the same licence [4].
- The AAR's `classes.jar` holds 607 classes: 337 obfuscated (`zz*`) classes under `com/google/android/gms/internal/play_billing`, 269 under `com/android/billingclient/api`, and `BuildConfig`. They are merged into the app's dex [4].
- Its manifest needs minSdk 23, so Rouxtine's minSdk 24 is fine. It requests the `com.android.vending.BILLING` permission and declares `<queries>` for `com.android.vending.billing.InAppBillingService.BIND` and a billing test companion. The library binds to the Play Store app at runtime [4].
- It pulls in `play-services-base`, `-basement`, `-location` and `-tasks`, each under the Android SDK License, plus Apache-2.0 `datatransport` and AndroidX artifacts [4][5].
- The Android SDK License Agreement [6]:
  - s.3.1 grants a "limited, worldwide, royalty-free, non-assignable, non-exclusive, and non-sublicensable license to use the SDK solely to develop applications for compatible implementations of Android".
  - s.3.3: "Google reserves all rights not expressly granted to you."
  - s.3.4: "Except to the extent required by applicable third party licenses, you may not copy (except for backup purposes), modify, adapt, redistribute, decompile, reverse engineer, disassemble, or create derivative works of the SDK or any part of the SDK."
- So people who receive the APK get no source and no right to modify the library. That makes it incompatible with the GPL.

## 2. Why the GPL needs a permission for it

### 2.1 Not a System Library

- GPL s.1 defines System Libraries as things that are "included in the normal form of packaging a Major Component" (kernel, window system, compiler, interpreter) and serve "only to enable use of the work with that Major Component, or to implement a Standard Interface" with a public-source implementation (`LICENSE:123-132`). Only those are left out of the Corresponding Source (`LICENSE:137-139`, `LICENSE:293-295`).
- The billing client is a Maven artifact that each app bundles. It ships with neither the OS nor a compiler, and its job is to talk to the Play Store app. It fails the first test.
- FAQ #WindowsRuntimeAndGPL: libraries "can only qualify as System Libraries as long as they're not distributed with the program itself" [2].
- One nuance: FAQ #SystemLibraryException says source isn't required for real system libraries "even if you distribute a linked executable containing them" [2]. That doesn't help here, because billing is not packaged with a Major Component in the first place.

### 2.2 One combined work

- FAQ #GPLStaticVsDynamic: "Linking [name of your program] statically or dynamically with other modules is making a combined work" [2].
- FAQ #MereAggregation: "If the modules are included in the same executable file, they are definitely combined in one program" [2]. The s.5 "aggregate" clause covers works "not combined with it such as to form a larger program" (`LICENSE:235-243`), so it doesn't apply.
- s.5c: "You must license the entire work, as a whole, under this License", and the licence applies "to the whole of the work, and all its parts, regardless of how they are packaged" (`LICENSE:222-226`).
- s.1: the Corresponding Source includes "dynamically linked subprograms that the work is specifically designed to require" (`LICENSE:141-145`). s.6 requires the distributor to provide it (`LICENSE:247-250`). Nobody can provide the billing source.
- FAQ #GPLWrapper: talking to the GPL code only through a permissively licensed module in between "is legally irrelevant" [2]. An Apache-2.0 wrapper module doesn't help.
- This is the FSF's interpretation. No court ruling on Android linking was found.

### 2.3 Only the copyright holders can grant it

- FAQ #GPLIncompatibleLibs: "if you want to use parts of other GPL-covered programs by other authors in your code, you cannot authorize the exception for them. You have to get the approval of the copyright holders of those programs" [2]. #LinkingOverControlledInterface says the same [2].
- s.7: if permissions "apply only to part of the Program ... the entire Program remains governed by this License without regard to the additional permissions" (`LICENSE:349-352`). The owner's permission on their own files does nothing for an APK that also holds onionhoney's code.
- Anyone who conveys the work "may at your option remove any additional permissions" (`LICENSE:354-356`). Forks can drop it.
- **Later contributors.** "When other people modify the program, they do not have to make the same exception for their code" [2]. The owner may place permissions on "material, added by you" (`LICENSE:357-359`). So the inbound terms must carry the permission from the first commit. Otherwise every contributor and translator becomes another person to ask. It costs nothing now, although it has no effect on the combined APK until onionhoney agrees.

### 2.4 How the permission can be given

- s.5c ends: "This License gives no permission to license the work in any other way, but it does not invalidate such permission if you have separately received it" (`LICENSE:226-228`). Reading this to cover a section 7 permission given separately, for example in an issue reply or an email, is reasonable. It is still an interpretation.
- The FAQ says "you should put this text in each file to which you are granting this permission" [2]. So the cleanest record is the text committed to onionhoney's own repo, or a signed email. A one-line issue reply is weaker.
- Once Rouxtine applies it, s.7 requires "a statement of the additional terms that apply to those files, or a notice indicating where to find the applicable terms" in the relevant source files (`LICENSE:398-401`). In practice: a `LICENSE-EXCEPTION` file, and an SPDX header such as `GPL-3.0-only WITH GPL-3.0-linking-exception` with a line pointing to that file.
- **Model text.** FAQ #GPLIncompatibleLibs gives a GPLv3 template [2]: "If you modify this Program, or any covered work, by linking or combining it with [name of library] (or a modified version of that library), containing parts covered by the terms of [name of library's license], the licensors of this Program grant you additional permission to convey the resulting work." A braced sentence about library source follows. The FAQ says to remove it when "not everybody can distribute source for the libraries", which is the case for Play Billing [2]. SPDX publishes the same wording, without the braced sentence, as `GPL-3.0-linking-exception` [3].
- **Filled in for Rouxtine (draft):**

  > Additional permission under GNU GPL version 3 section 7
  >
  > If you modify this Program, or any covered work, by linking or combining it with the Google Play Billing Library (com.android.billingclient:billing and com.android.billingclient:billing-ktx) and the Google Play services client libraries it depends on (or modified versions of those libraries), containing parts covered by the terms of the Android Software Development Kit License Agreement, the licensors of this Program grant you additional permission to convey the resulting work.

- **A wider version** could cover "any client library published by Google LLC for use with the Google Play Store app, Google Play services or Firebase". That would also allow Crashlytics, ML Kit and in-app review later, but it is a bigger ask. ML Kit and Play Core come under their own terms (§6), so the "containing parts covered by the terms of ..." clause would have to name those too.

### 2.5 If billing shipped without permission

- s.8: propagating outside the licence "is void, and will automatically terminate your rights" (`LICENSE:409-413`).
- After the violation stops, rights come back "provisionally, unless and until the copyright holder explicitly and finally terminates your license", and permanently if the holder doesn't notify "prior to 60 days after the cessation". They also come back permanently if it is the first notice from that holder and the violation is cured "prior to 30 days after your receipt of the notice" (`LICENSE:415-427`).
- This decides whether the licence survives. It does not cap liability, and distribution outside the licence is still infringement for that period.
- HANDOFF's "Don't" list already forbids it: "No Play Billing until onionhoney's written permission is in hand" (HANDOFF.md:244-245).

## 3. Who holds the copyright in the code Rouxtine ports

**onionhoney alone, for everything Rouxtine will port.**
- A full clone of onionhoney/roux-trainers has 84 commits on master. The head is `0129460` (2025-07-27), and it was still the head on 2026-10-29 [14].
- `git blame -w` over `src/lib` (CubeLib, Defs, Math, Pruner, Solver, CachedSolver, Analyzer, Evaluator, Algs, Selector), `src/reducers/*`, `src/components/CubeSim.tsx`, `src/Config.tsx` and `src/Types.tsx` gives 4,603 lines to "Jeffrey Sun" and 5,305 to "Zhouheng Sun", and no other author [14].
- Those identities use several emails, one of them at Stanford. The GitHub web merge of PR #21 is authored "Zhouheng Sun" with the Stanford address, which ties the onionhoney account to them [14][15].
- The only other human author is Eric Xu: 3 commits on 2023-08-19, +14/−5 lines, only in `src/components/TopBarView.tsx`, which won't be ported. PR #21 was merged [14][15].
- Dependabot's 13 commits sit only on unmerged branches. The history merged from onionhoney/cmll-trainer (`a8e61e7`, 2020-03-09) adds no other author on master [14].

**Sources that would add copyright holders. Don't copy from them:**
- **PR #31**, "Add support to 'lefty handed' Roux" by GitHub user Pekito (git author Pedro Luiz). One commit on 2026-08-19, 8 files, +295/−9. It is open, closes #14, and has no reviews or maintainer comments [15]. Write the M-hand setting independently.
- **AJGDsn.** On 2026-10-29 this account had 8 public repos, none of them roux-trainers and none a fork. `AJGDsn/roux-trainers` asks for credentials, so it is missing or private [16]. Their GPL-3.0 smartcube-gym (Petrus, Roux, LEOR, APB, ZZ) is not a fork, and copying from it would add a holder [16].
- **csTimer** is GPL-3.0 ([licensing-monetization.md §2.3](licensing-monetization.md)).
- The nauroth/roux-trainers fork changes no source: `git diff 0129460 origin/master -- src` is empty [14].

**Vendored code that `git blame` credits to onionhoney but isn't theirs:**
- `src/lib/three/OrbitControls.js` is three.js code (MIT) and differs from three@0.125.2 by two diff lines [19]. It won't be ported, because Rouxtine draws on a Canvas.
- `src/lib/min2phase/min2phase.js` is cubing/min2phase.js@a465b5e plus an `/* eslint-disable */` line and five one-token fixes (adds `let`, `var` and `this.`, drops a Java `(char)` cast). `index.ts` and `min2phase.d.ts` are byte-identical to that repo. `min2phase-solver.ts`, `-wrapper.ts`, `.worker.ts`, `off-thread.ts` and `on-thread.ts` are onionhoney's edits of Lucas Garron's files [18]. No file in upstream's `src` carries a licence or copyright header [14].
- `Defs.tsx:48-57` holds the csTimer facelet-order tables (8 corners, 12 edges). They follow Kociemba's standard cubie order (URF…DRB, UR…BR) [14]. Whether a table like that is protectable at all is a legal question (Open questions).
- The README says the app was "Inspired by http://cubegrass.appspot.com/". It doesn't say whether code was taken [14].

**min2phase can be used under MIT.**
- In cs0x7f/min2phase, commit `4d183b9` (2023-02-21, Chen Shuang) added a "License MIT" section with "Copyright (c) 2023 Chen Shuang" after the existing "License GPLv3" section [17].
- Only `src/Search.java` has a header: GPL-3.0-or-later, "Copyright (C) 2015 Shuang Chen". `CoordCube`, `CubieCube`, `Tools` and `Util` have none. Every code commit is by Chen Shuang. The one other author changed one README line [17].
- cubing/min2phase.js says "dual-licensed under: MIT, GPLv3 or later" (Lucas Garron, 2018) in LICENSE.md, but its `package.json` says `GPL-3.0-or-later` [18]. The JS was made in 2018, when Chen's code was GPL-only, so Garron's MIT grant can only cover Garron's own part. Exactly how the JS was produced is not proven.
- So port Chen's Java under Chen's 2023 MIT grant. The README doesn't say "either/or" in so many words, and Search.java's header still says GPL, so ask Chen to confirm if certainty matters.

**Stanford is unlikely to own any of it.**
- 32 upstream commits across all refs came from Stanford identities: 6 in 2020, 25 in 2021, 1 in 2023. On master it is 22 (4, 17, 1) [14].
- Stanford's Copyright Policy (RPH 9.2, current version 2018-03-18) claims no ownership of "unpatentable software" that is not an institutional work and "did not make significant use of University resources". It adds: "Such works include those of students created in the course of their education". "Ordinary use of desktop computers" is given as non-significant use. Works for hire and works under "contractual obligations" are exceptions [20].
- Whether onionhoney was a student, an employee (for example an RA or TA) or neither at the time is unknown. The FAQ warns that "the employer or the school might claim" the copyright [2 #HowIGetCopyright]. The message to onionhoney asks.

**Upstream never states that its code is under the GPL.**
- `LICENSE` is the plain GPLv3 text, byte-identical to Rouxtine's (sha256 `8b1ba204…`). `package.json` has no licence field, no file in `src` has a header, and the README doesn't mention a licence [14].
- FAQ #LicenseCopyOnly: a copy of the GPL alone "does not explicitly state that the code in the same repository may be used under the GNU GPL", and "An explicit statement saying that eliminates all doubt" [2].
- s.14: "If the Program does not specify a version number of the GNU General Public License, you may choose any version ever published by the Free Software Foundation" (`LICENSE:575-577`). Whether shipping the v3 text counts as specifying a version is debatable.
- Ask for the statement in the same message as the permission.

**Upstream responsiveness** (GitHub pages read on 2026-10-29) [15]:
- The most recent onionhoney comment found is on issue #4, 2021-10-14: "This seems like a very interesting proposal." Only issues #4, #11, #26 and #27 were read in full.
- PR #21 was merged on 2023-08-20. The last commit is from 2025-07-27.
- PR #31 (2026-08-19) has no review or comment. 14 issues are open; the newest, #32, was opened by another user on 2026-10-28. Issues #26 and #27 (2025) were closed by their own authors.

## 4. The precedents, re-checked

| App | Licence | Billing | Other copyright holders | Linking exception | What it shows |
|---|---|---|---|---|---|
| Twisty Timer [21] | GPL-3.0-or-later ("Copyright (C) 2016 Ariovaldo Neto") | `com.anjlab.android.iab.v3:library:2.0.3` (Apache-2.0), which pulls `com.android.billingclient:billing` 4.0.0 at runtime | Yes. 13 author names (four of them spellings of Ari Neto). D Gardner wrote +12,243/−5,506 lines in 2016. `contributors.txt` lists 9 people | None. At `2fcd6f7`, section 7 wording appears only inside `LICENSE.txt` | Ships billing with other people's GPL code and no exception. Play shows "In-app purchases", last updated 2023-06-23, so it is a stale example |
| Tasks.org [22] | GPLv3. README: "The source code from Astrid's open source Android app serves as the basis of Tasks." | `googleplay` flavour adds `play.billing.ktx`, Firebase Crashlytics, Play services and more (`app/build.gradle.kts:288-298`) | Yes. 25 of 58 files under `com/todoroo` carry "Copyright (c) 2012 Todoroo Inc". 442 author names, including bots and translators | None found in LICENSE, README, CONTRIBUTING, `com/todoroo`, or the `googleplay` and `generic` source sets. The rest of the tree was not searched | Same as Twisty Timer |
| Thunderbird for Android [23] | Apache-2.0 | `full` flavour uses `billing` and `billing-ktx` 9.1.0; `foss` uses a funding link | n/a | n/a | No copyleft, so no conflict to resolve. Not a GPL precedent |
| AnkiDroid [24] | GPL-3.0, with an AGPL-3.0 part of the back end and an LGPL-3.0 API | None anywhere in its Gradle files at `9015ffb`. The `play` flavour only sets `SHOW_DONATE_LINKS=false`, because Google "no longer considers Open Collective to be acceptable for a tax-exempt link" | n/a | n/a | Not a billing precedent |
| OsmAnd [25] | Code GPLv3 ("Copyright © 2010–2026 OsmAnd BV"), artwork CC-BY-NC-ND 4.0 | Billing 8.0.0 in every flavour ([licensing-monetization.md §4](licensing-monetization.md)) | Not checked | None in LICENSE | Not checked further |

The two GPL apps that ship billing with other people's code did it without permission. That shows tolerance, not permission, and it is practice, not legal authority. Whether anyone ever objected to them is unknown.

## 5. No way around the library on Play

- **In-app purchases need Play's billing system.** The Payments policy says apps "requiring or accepting payment for access to in-app features or services, including any app functionality, digital content" must use it [8].
- **And that means Google's library.** The deprecation FAQ (updated 2026-10-23) says: "By Aug 31, 2026, all new apps and updates to existing apps must use Billing Library version 8 or later", with an extension possible until Nov 1, 2026 [7]. A home-made client for the old interface would not be "Billing Library version 8 or later" (an inference from that wording).
- **Yearly upgrades.** Version 8 is accepted until 2027-08-31 and version 9 until 2028-08-31, each extendable to Nov 1 [7]. Any billing integration needs roughly one major upgrade a year.
- **A paid download needs no library.** "Developers charging for app downloads from Google Play must use Google Play's billing system as the method of payment for those transactions" [8], and Play's store checkout handles that payment. Paid unlocker apps are listed today: Poweramp Full Version Unlocker ($19.99, updated 2025-07-02) and Nova Launcher Lifetime Prime ($29.99, updated 2023-09-05) [10][11].
- **A key app is separate from the GPL app.** If the main app only checks that a key app is installed and signed with the same certificate, the two are not linked. That rests on FAQ #MereAggregation and is an interpretation [2]. It matters little here: if the key app holds only the owner's code, neither APK contains closed code.

## 6. Other Google libraries raise the same question

| Artifact | Version | Licence in its POM | Typical use |
|---|---|---|---|
| `com.google.android.play:review` | 2.0.2 | Play Core Software Development Kit Terms of Service | In-app review prompt |
| `com.google.android.play:app-update` | 2.1.0 | Play Core Software Development Kit Terms of Service | In-app update prompt |
| `com.google.mlkit:barcode-scanning` | 17.3.0 | ML Kit Terms of Service | QR scanning |
| `com.google.android.gms:play-services-code-scanner` | 16.1.0 | ML Kit Terms of Service | QR scanning through Play services |
| `com.google.firebase:firebase-crashlytics` | 21.0.1 | Apache-2.0, but it depends on `play-services-tasks` 18.4.0 (Android SDK License), and `firebase-common` 23.0.0 pulls `-basement` and `-tasks` | Crash reports |
| `com.google.android.gms:play-services-oss-licenses` | 17.5.3 | Android Software Development Kit License | Licences screen |
| `com.google.zxing:core` | 3.5.4 | Apache-2.0 (from `zxing-parent`) | QR encoding and decoding |

Sources: [12][13].

- R1 includes opt-in crash reporting and analytics with an "open-source SDK" (scope.md:34) and sharing by QR code (scope.md:48). Picking Crashlytics, ML Kit, the code scanner or the oss-licenses plugin would need the same permission as billing. ZXing and an open-source crash tool avoid it.
- Android vitals needs nothing in the app. Its data "comes from the Android system", only from users who agreed to share it, and "only counts issues from certified devices and apps installed from Google Play" [26]. It doesn't cover the GitHub APK.

## 7. Options

**(a) Written section 7 permission from onionhoney.** Here that means onionhoney alone (§3), plus the owner for their own code. Use the text in §2.4.
- For: one person to ask. It keeps the decided plan (a Supporter unlock and maybe a smart-cube add-on through Play Billing). It can be bundled with the licence statement upstream lacks (§3). A written reply in an issue or an email probably counts (§2.4).
- Against: upstream has been quiet since 2025-07-27, and onionhoney may say no or never reply. Every later contributor and translator must grant the same permission (§2.3). Forks may remove it. Code from PR #31, smartcube-gym, csTimer or any other GPL source would add a holder who must also agree.

**(b) Keep closed Google libraries out of R1.** HANDOFF's current plan, made stricter. Neither flavour contains a closed Google library: Android vitals, an open-source opt-in crash and analytics tool, ZXing for QR, and no Play Core, ML Kit or Firebase. Paid extras wait until (a) or (d1) is in place.
- For: needs nobody's permission and no rewrite. `play` and `foss` differ only in links and wording, so the build is simpler and clean for F-Droid. Nothing in R1 depends on onionhoney's answer.
- Against: no revenue on Play in R1. It rules out Crashlytics and ML Kit for good unless (a) uses the wider wording.

**(c) Don't port the GPL code; reimplement it independently.** Rewrite the cube model, block solvers, analyzer, evaluator, renderer, trainer flows and alg data without using onionhoney's source.
- Size of what would be replaced, from upstream at `0129460`: 6,215 TypeScript lines [14]:
  - core lib 3,836 (CubeLib 1,188, Pruner 1,013, Algs 423, Evaluator 280, Solver 279, Analyzer 258, Defs 221, CachedSolver 69, Math 66, Selector 39);
  - CubeSim 485;
  - trainer state machines 1,423 (`src/reducers/*.tsx` without the test file);
  - Config and Types 471.
- Plus 814 lines of `src/lib` tests, which HANDOFF step 4 plans to port as the spec (HANDOFF.md:205-207), and the data: the CMLL alg list in `Algs.tsx` and `two_gram_v1.json` (9,112 bytes of move-pair ergonomics weights, imported by `Evaluator.tsx`) [14].
- **ESTIMATE:** 3–5k Kotlin lines and roughly 4–8 weeks of agent-assisted work. onionhoney's tuned solver behaviour and ergonomics data would be lost.
- For: removes onionhoney from the licensing question for good, and lets the owner pick any licence or exception.
- Against: weeks spent on code that already exists. A clean room is hard to defend here, because the owner, these docs (with `path:line` references) and the agents have all read the code. It goes against the project's aim of building openly on onionhoney's work.

**(d1) A paid "Supporter key" app instead of Play Billing.** Sell the cosmetic unlock as a separate paid download, for example "Rouxtine Supporter". The main app checks that the key package is installed and signed with the same certificate. The key app is the owner's own code, contains nothing from onionhoney, and can itself be GPL. A paid smart-cube add-on could work the same way.
- For: needs nobody's permission and no closed library in any APK. Works whether onionhoney says yes, no or nothing. Precedents exist on Play (§5).
- Against: a second Play listing, which probably needs its own 12-tester closed test (Open questions), and a clunkier purchase than a button in the app. Whether a key-only app passes Play's minimum-functionality and spam rules is unverified. Like any unlock under the GPL, it is easy to bypass, which the owner already accepts. Don't put Play Billing into an add-on that exchanges complex internal data with the main app: under FAQ #MereAggregation that could make them one program [2].

**(d2) A broader relicence from onionhoney.** Ask them to offer their code under MIT or Apache-2.0 as well as the GPL (FAQ #ReleaseUnderGPLAndNF) [2].
- For: also settles the Play Terms of Service question (Open questions) and every future SDK choice.
- Against: a much bigger request than a narrow linking permission. Earlier releases stay GPL anyway.

**(d3) Ship billing without permission, as Twisty Timer and Tasks.org do.**
- For: no waiting.
- Against: a GPL violation under the text and the FSF's reading (§2). The only holder affected is the person who could simply say yes. It contradicts HANDOFF's "Don't" list. **Not an option.**

## 8. Corrections to other notes

- **monetization.md:15-19** (Recommendation 2) says "Twisty Timer (cubing, GPL) and Thunderbird already sell through Play Billing this way". Thunderbird is Apache-2.0 [23]. Twisty Timer ships billing with no exception despite other copyright holders [21]. Neither shows that billing is allowed next to someone else's GPL code.
- **monetization.md:17 and :100** list "advanced stage analytics" and "advanced per-stage stats and 'what to work on'" as Supporter candidates. HANDOFF "Money" rules these out: never paywall stats or analysis. Line 100's "deeper smart-cube analytics" sits between that rule and the allowed paid smart-cube add-on, so the owner should decide it.
- **monetization.md:25-26** says "Tasks.org, Thunderbird and AnkiDroid use the same split". That holds for the flavour split, but AnkiDroid's `play` flavour has no billing library [24].
- **monetization.md:159-161** files asking onionhoney under "Community etiquette". Under the GPL text as the FSF reads it, it is a precondition for billing (§2). The same line calls min2phase "GPL-3.0-or-later". It can be used under Chen Shuang's MIT grant (§3).
- **licensing-monetization.md:248** says of Tasks.org, OsmAnd, Twisty Timer and Mullvad: "Each relies on its main copyright holder doing the distributing." That is wrong for Twisty Timer and Tasks.org, which carry code from other holders (§4). They rely on nobody objecting.
- **HANDOFF.md:42-45** says the billing blocker is "Being verified". This note is that check, and the blocker stands as written. **HANDOFF.md:130-131** lists writing to onionhoney as optional. If the owner wants paid extras on Play, it becomes a to-do that includes the permission request.

## 9. Recommendation

1. **Option (b) for R1.** Put no closed Google library in either flavour. Use Android vitals plus an open-source opt-in crash and analytics tool, ZXing for QR, and no Play Core, ML Kit, Firebase or oss-licenses plugin. R1 then needs nobody's permission. Keep HANDOFF's blocker as written.
2. **Option (a), asked now, without waiting on it.** Open the message below as a public GitHub issue on onionhoney/roux-trainers, so there is a written public record. Optionally point to it by email at the address on their commits. Offer a pull request that adds the licence statement and the permission to their README, so that merging it becomes the record (§2.4).
3. **Set the terms from the first commit:**
   - Add `LICENSE-EXCEPTION` with the §2.4 text, applied to the owner's own code.
   - In CONTRIBUTING, say that contributions are `GPL-3.0-only WITH` that exception, with a DCO sign-off. Use the same terms for Weblate translations.
   - Port min2phase from Chen Shuang's Java under the MIT grant.
   - Copy nothing from PR #31, smartcube-gym or csTimer.
4. **If the answer is no, or never comes,** sell the Supporter themes as a paid key app (d1), and never ship Play Billing. Skip (c): its cost is far out of proportion, and the code has already been read.

**Draft message.** The owner sends it under their own name and fills in the brackets.

> **Title:** Rouxtine, an Android Roux trainer built on Roux Trainers: licence question
>
> Hi onionhoney,
>
> I'm [name], a Roux solver. I'm building Rouxtine (https://github.com/nauroth/rouxtine), a free, open-source Android app for training Roux. Its cube model, solver and trainers will be a Kotlin port of your Roux Trainers, under GPL-3.0. You'll be credited in the app, on the store page and in every ported file.
>
> I have three questions.
>
> 1. Licence. The repo includes the GPL v3 text, but no line says the code is under it. Could you confirm that roux-trainers is licensed under GPL-3.0? "Only" or "or later" are both fine. A sentence in the README would settle it for everyone.
>
> 2. Ownership. Apart from the vendored min2phase and three.js files, are you the only copyright holder of src/lib, src/reducers and src/components/CubeSim.tsx? Did a school or employer have any claim on that code when you wrote it?
>
> 3. One permission, only if you're happy with it. Every trainer, the solver and the stats will stay free in every version of Rouxtine. Later I'd like to sell an optional one-time "Supporter" unlock on Google Play that only changes colour themes, and maybe a paid add-on for smart cubes, which would be new code. The GitHub build will have everything unlocked. Google requires its closed-source Play Billing Library for any purchase inside an app, and under the GPL I can only ship that library next to your code with your permission. Would you grant this?
>
> "Additional permission under GNU GPL version 3 section 7: If you modify this Program, or any covered work, by linking or combining it with the Google Play Billing Library (com.android.billingclient:billing and com.android.billingclient:billing-ktx) and the Google Play services client libraries it depends on (or modified versions of those libraries), containing parts covered by the terms of the Android Software Development Kit License Agreement, the licensors of this Program grant you additional permission to convey the resulting work."
>
> If you agree, I can open a pull request that adds the licence line and this text to your README, so you only need to merge it. A reply here quoting the text works too.
>
> "No" is a perfectly good answer, and Rouxtine stays free either way.
>
> Thanks for Roux Trainers. This app couldn't exist without it.
>
> [name]

## Questions for the owner

1. Send the message to onionhoney now? As a public GitHub issue, an email to the address on their commits, or both? Under which name?
2. How wide should the permission be: Play Billing only, as drafted, or every Google Play, Play services and Firebase client library? The wider version keeps Crashlytics, ML Kit and in-app review open for later, but it is a bigger ask. Should it also include an app-store (Play Terms of Service) permission?
3. If onionhoney says no or never answers, do you accept a paid "Supporter key" app (no billing library) as the way to sell themes, or no paid extras on Play at all?
4. Do you want to offer onionhoney anything beyond credit, such as a revenue share or a free Supporter unlock?
5. OK to set contributor terms from the first commit: GPL-3.0-only with the Play exception plus a DCO sign-off, for code and Weblate translations? Which licence should community Rouxtine templates use (for example CC0)?
6. For R1, confirm open-source SDKs only: an open-source opt-in crash and analytics tool, ZXing for QR, and no Firebase, ML Kit or Play Core libraries. Then R1 needs nobody's permission.
7. min2phase: rely on Chen Shuang's MIT section in the README, or also ask Chen to confirm that it covers the GPL-headed `Search.java`?

## Sources

[1]: https://www.gnu.org/licenses/gpl-3.0.txt (this repo's `LICENSE`; sha256 `8b1ba204…`, the same as upstream's `LICENSE`)
[2]: https://www.gnu.org/licenses/gpl-faq.html (anchors as cited; page dated "$Date: 2026/01/13 15:26:29 $"; quotes taken from a copy fetched 2026-10-28 and matched against the live page on 2026-10-29)
[3]: https://spdx.org/licenses/GPL-3.0-linking-exception.html ; https://spdx.org/licenses/GPL-3.0-linking-exception.json
[4]: https://dl.google.com/android/maven2/com/android/billingclient/billing/maven-metadata.xml ; https://dl.google.com/android/maven2/com/android/billingclient/billing/9.1.0/billing-9.1.0.pom ; https://dl.google.com/android/maven2/com/android/billingclient/billing/9.1.0/billing-9.1.0.aar (`classes.jar`, `AndroidManifest.xml`, `third_party_licenses.txt`) ; https://dl.google.com/android/maven2/com/android/billingclient/billing/9.1.0/billing-9.1.0-sources.jar (HTTP 404) ; https://dl.google.com/android/maven2/com/android/billingclient/billing-ktx/9.1.0/billing-ktx-9.1.0.pom
[5]: https://dl.google.com/android/maven2/com/google/android/gms/play-services-basement/18.9.0/play-services-basement-18.9.0.pom ; https://dl.google.com/android/maven2/com/google/android/gms/play-services-base/18.5.0/play-services-base-18.5.0.pom ; https://dl.google.com/android/maven2/com/google/android/gms/play-services-location/19.0.0/play-services-location-19.0.0.pom ; https://dl.google.com/android/maven2/com/google/android/gms/play-services-tasks/18.2.0/play-services-tasks-18.2.0.pom ; https://dl.google.com/android/maven2/com/google/android/datatransport/transport-runtime/3.1.8/transport-runtime-3.1.8.pom
[6]: https://developer.android.com/studio/terms (Android SDK License Agreement, fetched 2026-10-29, ss.3.1, 3.3, 3.4)
[7]: https://developer.android.com/google/play/billing/deprecation-faq (Play Billing Library deprecation FAQ, last updated 2026-10-23)
[8]: https://support.google.com/googleplay/android-developer/answer/9858738 (Play Payments policy, fetched 2026-10-29)
[9]: https://support.google.com/googleplay/android-developer/answer/14151465 (testing requirements for new personal accounts, fetched 2026-10-29)
[10]: https://play.google.com/store/apps/details?id=com.maxmpz.audioplayer.unlock (Poweramp Full Version Unlocker, fetched 2026-10-29)
[11]: https://play.google.com/store/apps/details?id=com.teslacoilsw.launcher.prime (Nova Launcher Lifetime Prime, fetched 2026-10-29)
[12]: https://dl.google.com/android/maven2/com/google/android/play/review/2.0.2/review-2.0.2.pom ; https://dl.google.com/android/maven2/com/google/android/play/app-update/2.1.0/app-update-2.1.0.pom ; https://dl.google.com/android/maven2/com/google/mlkit/barcode-scanning/17.3.0/barcode-scanning-17.3.0.pom ; https://dl.google.com/android/maven2/com/google/android/gms/play-services-code-scanner/16.1.0/play-services-code-scanner-16.1.0.pom ; https://dl.google.com/android/maven2/com/google/firebase/firebase-crashlytics/21.0.1/firebase-crashlytics-21.0.1.pom ; https://dl.google.com/android/maven2/com/google/firebase/firebase-common/23.0.0/firebase-common-23.0.0.pom ; https://dl.google.com/android/maven2/com/google/android/gms/play-services-oss-licenses/17.5.3/play-services-oss-licenses-17.5.3.pom
[13]: https://repo1.maven.org/maven2/com/google/zxing/zxing-parent/3.5.4/zxing-parent-3.5.4.pom ; https://repo1.maven.org/maven2/com/google/zxing/core/3.5.4/core-3.5.4.pom (read through Google's Maven Central mirror)
[14]: https://github.com/onionhoney/roux-trainers/tree/0129460a5c30082ea0e29b4489623889118078ec (full clone with `refs/pull/*`; `git log --all`, `git blame -w`, `git ls-remote` on 2026-10-29; `wc -l` on an export of master) ; https://github.com/nauroth/roux-trainers (`git diff 0129460 origin/master -- src`)
[15]: https://github.com/onionhoney/roux-trainers/pull/21 ; https://github.com/onionhoney/roux-trainers/pull/31 ; https://github.com/onionhoney/roux-trainers/issues ; https://github.com/onionhoney/roux-trainers/issues/4 ; https://github.com/onionhoney/roux-trainers/issues/11 ; https://github.com/onionhoney/roux-trainers/issues/26 ; https://github.com/onionhoney/roux-trainers/issues/27 (read on 2026-10-29)
[16]: https://github.com/AJGDsn (repository search `user:AJGDsn`, 2026-10-29) ; https://github.com/AJGDsn/smartcube-gym
[17]: https://github.com/cs0x7f/min2phase/blob/4d183b9eff8119cac72bc50ef35a7d8990740e06/README.md ; https://github.com/cs0x7f/min2phase/blob/4d183b9eff8119cac72bc50ef35a7d8990740e06/src/Search.java
[18]: https://github.com/cubing/min2phase.js/blob/a465b5e458fc1850299cab3c6254973c7f4c76ae/LICENSE.md ; https://github.com/cubing/min2phase.js/blob/a465b5e458fc1850299cab3c6254973c7f4c76ae/package.json ; https://github.com/cubing/min2phase.js/tree/a465b5e458fc1850299cab3c6254973c7f4c76ae/src (diffed against upstream `src/lib/min2phase`)
[19]: https://unpkg.com/three@0.125.2/LICENSE ; https://unpkg.com/three@0.125.2/examples/jsm/controls/OrbitControls.js
[20]: https://doresearch.stanford.edu/policies/research-policy-handbook/intellectual-property/copyright-policy (Stanford RPH 9.2, fetched 2026-10-29)
[21]: https://github.com/aricneto/TwistyTimer/blob/2fcd6f7d2697a0810f37245ac033f1da12448382/README.md ; https://github.com/aricneto/TwistyTimer/blob/2fcd6f7d2697a0810f37245ac033f1da12448382/app/build.gradle#L87-L88 ; https://repo1.maven.org/maven2/com/anjlab/android/iab/v3/library/2.0.3/library-2.0.3.pom ; https://play.google.com/store/apps/details?id=com.aricneto.twistytimer (git log on a blobless clone)
[22]: https://github.com/tasks/tasks/blob/4d7792727f1a96c92410fa00665a5e657681600f/README.md ; https://github.com/tasks/tasks/blob/4d7792727f1a96c92410fa00665a5e657681600f/app/build.gradle.kts#L288-L298 ; https://github.com/tasks/tasks/blob/4d7792727f1a96c92410fa00665a5e657681600f/app/src/main/java/com/todoroo/andlib/utility/AndroidUtilities.kt#L1-L5 (git log and grep on a blobless clone)
[23]: https://github.com/thunderbird/thunderbird-android/blob/1d3437a/LICENSE ; https://github.com/thunderbird/thunderbird-android/blob/1d3437a/README.md#L98-L100 ; https://github.com/thunderbird/thunderbird-android/blob/1d3437a/app-thunderbird/build.gradle.kts#L175-L264 ; https://github.com/thunderbird/thunderbird-android/blob/1d3437a/gradle/libs.versions.toml#L14
[24]: https://github.com/ankidroid/Anki-Android/blob/9015ffb/AnkiDroid/build.gradle.kts#L104-L266 ; https://github.com/ankidroid/Anki-Android/blob/9015ffb/README.md#L116-L120
[25]: https://github.com/osmandapp/OsmAnd/blob/master/LICENSE
[26]: https://developer.android.com/topic/performance/vitals (Android vitals, last updated 2026-10-19)

## Open questions / unverified

- **No legal counsel and no court ruling.** Everything rests on the GPL text and the FSF's FAQ [1][2].
- **A permission given separately.** That s.5c's "separately received" covers a section 7 permission given in an issue or an email is an interpretation (§2.4).
- **Play Terms of Service.** Play's user terms ("personal, non-commercial use only") against the GPL's ban on further restrictions affect every Play build of onionhoney's code, including options (b) and (d1). [licensing-monetization.md §2.2](licensing-monetization.md) finds it probably fine because of Play's carve-outs, but untested. Only an app-store permission or (d2) settles it. Those primary sources were not re-read for this note.
- **The key app (d1).** Play's testing page says personal accounts created after 2023-11-13 "must run a closed test for their app with a minimum of 12 testers who have been opted in continuously for at least 14 days" [9]. "For their app" suggests a second listing needs its own test; not confirmed. Whether a key-only app passes the minimum-functionality and spam policies is unverified. Per [monetization.md §3](monetization.md), a free app can never become paid, so the key app would have to be paid from its first release (not re-checked here). How the main app should detect the key on Android 11+ was not researched.
- **Key app as a separate program.** That an app which is only detected is not combined with the main app is an interpretation of FAQ #MereAggregation [2].
- **onionhoney at Stanford.** Whether they were a student, an employee or neither in 2020–2021 is unknown, and Stanford's policy treats these cases differently [20].
- **min2phase.** Chen Shuang's MIT section and the GPL header in `Search.java` disagree, and how the 2018 JS port was made is not proven [17][18].
- **Defs.tsx tables.** Whether a standard facelet ordering is protectable at all is a legal opinion, not checked [14].
- **SPDX identifier.** Whether `GPL-3.0-linking-exception` is the right SPDX id for a text that names specific libraries, rather than a custom `LicenseRef-`/`AdditionRef-` id, was not checked [3].
- **Exception wording.** The §2.4 text follows the FSF template but is not lawyer-vetted. Neither is the app-store permission in Terpoir ([licensing-monetization.md §4](licensing-monetization.md)).
- **Rewrite estimate.** The 3–5k Kotlin lines and 4–8 weeks for (c) are an ESTIMATE. Only the line and byte counts are measured [14].
- **Precedents.** Whether anyone ever objected to Twisty Timer or Tasks.org is unknown. Tasks.org's tree was searched for exception wording only in the places listed in §4 [21][22].
- **Tooling.** On 2026-10-29, curl to www.gnu.org and web.archive.org got connection resets, and curl or `gh` to github.com for onionhoney's repo got proxy 403s. WebFetch reached both. The live FAQ shows the same `$Date` as the cached copy, so the quotes are current [2].
