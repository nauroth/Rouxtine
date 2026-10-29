# Licensing and monetization: the Ko-fi link, in-app purchases and GPL duties

Researched 2026-10-28 (Play, F-Droid and Android pages re-read 2026-10-29) to settle how Rouxtine may ask for money, and what the GPL requires of every build. `[n]` refers to the numbered sources at the bottom. `LICENSE:n` is a line of the GPLv3 text [44] in this repo's `LICENSE`, which is byte-identical to upstream's [48]. **This is research, not legal advice.**

**Owner's decision (given):** the only monetization is a Ko-fi link in the app. No paywalled features, no ads, no in-app purchases, so no Play Billing library. Licence: GPL-3.0-only.

## Summary

- **Google Play build: no Ko-fi link, in any region.**
  - Play's Payments policy says apps "may not lead users to a payment method other than Google Play's billing system", through the app or its store listing [1].
  - Google has enforced this against in-app donation links in free open-source apps: WireGuard (2019), StreetComplete (2022) and AnkiDroid (2026) [4][3][5].
  - Two exceptions exist on paper, and neither fits. Tax-exempt donations need a recipient like a 501(c)(3) charity [1][5]. Google's "creator tips" FAQ has never, in any case found, been accepted for donations to an app's own developer [2].
  - The regional programs (US, EEA, UK, AU, JP, later everywhere) are for selling digital content or services. Each needs enrolment, API integration and fee reporting. Most also need a registered business or Play Billing running alongside [7][9][10][12].
- **F-Droid build: yes, in the app and in the metadata.** F-Droid's rules do not forbid donation links [31][32]. The current F-Droid builds of Tasks.org and Shattered Pixel Dungeon open donation pages from inside the app and carry no anti-feature [34][35][56][63].
- **GitHub-release build: yes.** No store policy applies, only the GPL.
- **What to do: two build flavours.**
  - `play` has no link and no "donate" wording, and its store listing does not mention Ko-fi.
  - `full` has the Ko-fi link and is used for F-Droid and GitHub releases. Tasks.org splits its builds the same way [56].
  - Google allows this: "We do not require parity across platforms. You can create different versions of your app" [2].
- **Watch the source link in the Play build.** The GPL requires directions to the source next to every binary (`LICENSE:275-286`). Google also rejected StreetComplete for a plain link to its GitHub page that pointed on to donations [3]. Give the source location as plain text or as a link to the release's source archive, not as a link to a page with a donate button.
- **The original question** ("donations, a premium UI, or rewrite everything?"):
  - Donations are fine.
  - The GPL also allows selling the app or premium features. But the premium code ships as public GPL source, so anyone may rebuild and share an unlocked copy [45].
  - On Play, paid features need the closed Play Billing library [1][27][29]. Shipping it with onionhoney's GPL code needs onionhoney's written s.7 permission, because onionhoney is the sole author of the code to port [45][49].
  - **No rewrite is needed** for donations, a paid app, or (with onionhoney's permission) Play in-app purchases.
- **The chosen model needs nobody's permission and no rewrite.** GPL-3.0-only matches what upstream ships: the plain GPLv3 text with no "or later" [48][45].
- **GPL duties for every build:**
  - a licence notice in every file, and a modification notice on every ported file;
  - an About/Licences screen with the "Appropriate Legal Notices";
  - source directions next to every binary;
  - MIT notices for min2phase and the smart-cube code. Upstream's vendored copies carry none.
- **Recommendation:** ship `play` without the link and `full` with it. Put Ko-fi in `FUNDING.yml`, the README, the website and F-Droid's `Donate:` field. Tell onionhoney about Rouxtine and ask them for an explicit licence statement, but don't block on the reply. Details in §6.

## 1. The Ko-fi link, channel by channel

### 1.1 Google Play

**The rule.** The Payments policy's anti-steering clause applies "Other than the conditions described in Section 3, Section 8, and Section 9" [1]. It bans leading users to another payment method via:
- "An app's listing in Google Play";
- "In-app webviews, buttons, links, messaging, advertisements, or other calls to action" [1].

The FAQ adds: "This includes directly linking to a webpage that could lead to an alternate payment method." Links to "administrative information" are allowed "as long as the webpage does not eventually lead to an alternate payment method" [2]. Outside the app, "you are free to communicate with your users about alternative purchase options" [2].

**Why neither exception fits.**
- *Tax-exempt donations* (Section 3). Play Billing "must not be used" for "tax exempt donations" [1]. Google reads this narrowly. It told AnkiDroid that "donations can only be accepted from organizations that meet the criteria of being tax-exempt (for example, 501(c)(3) organizations, etc.)", and it did not accept Open Collective's 501(c)(6) status [5]. A personal Ko-fi page is not tax-exempt.
- *Creator tips.* The FAQ says a tip where "100% of the tip or contribution from a user goes to the creator and the payment does not grant access to any digital content or services" is a peer-to-peer payment that needs no Play Billing [2]. Google does not define "creator". No case was found in which Google accepted this for donations to an app's own developer. The three cases below went the other way.

**Enforcement record.**

| App | When | What happened |
|---|---|---|
| WireGuard | Oct 2019 | Removed and delisted, older versions included, over an in-app donation link to the WireGuard website. The appeal was rejected [4] |
| StreetComplete | Feb–Mar 2022 | Rejected for Liberapay, GitHub Sponsors and Patreon links on its About page. Then rejected again for a link to its GitHub homepage with a note that donation links are there. Resolved by removing the link and showing a dialog explaining that the app can't link to any donation system [3] |
| AnkiDroid | reported 2026-09-02 | Flagged for an Open Collective link. Google said the app "is not eligible to solicit donations within the app", and updates were blocked while the appeal ran [5] |

**By region.** No region gives a personal developer without Play Billing a working route to a Ko-fi link.

| Region | Route that exists | Fits a personal Ko-fi link? |
|---|---|---|
| All regions (default) | None: the anti-steering rule [1] | No |
| Any region, tax-exempt recipient | Payments policy s.3 [1] | No. Google wants a 501(c)(3)-type recipient [5] |
| United States | The Epic injunction, effective 2025-10-29: for US users Google "will not prohibit a developer from ... link to transactions". The same page also says "Our current policies and programs remain in effect", and it routes link-outs through the external content links program [6]. That program needs a declaration form, the external links APIs and transaction reporting. Fees are owed on "any transactions" within 24 hours of a link, plus $2.85 per linked app install. Reporting started 2026-10-01; download reporting is due 2027-02-01 [7]. US alternative billing also needs its APIs [8] | Unclear, and not built for donations (see Open questions) |
| EEA | External offers: "Developer must be registered as a business", APIs, and no Play Billing in the app [10]. Billing choice program (since 2026-06-30): users must get "a choice of Google Play Billing alongside" the alternative, and links "can only direct users to your owned offers" [9]. EEA alternative billing and user choice billing need a registered business [11][12] | No. Needs a business or the Play Billing library |
| UK | Billing choice since 2026-06-30 (needs Play Billing alongside) [9]. User choice billing pilot (registered business, non-gaming apps) [12] | No |
| Australia, Japan | Billing choice since 2026-09-30 [9]. Japan's external payments program has accepted no new enrolments since 2026-09-30 and ends after 2027-03-31 [14] | No |
| South Korea, India | User choice billing: an alternative offered next to Play Billing, fee "Reduced by 4%" [13]. Billing choice reaches South Korea on 2026-12-31 [15] | No |
| Rest of world | Billing choice from 2027-09-30 [15] | No |
| Countries where Play Billing is unavailable | "the Payments policy's requirement to use Google Play's billing system does not apply in that country" [2] | Unclear whether the anti-steering rule still applies. Not worth geo-gating |

**What the Play build may still say.** The FAQ lets "consumption only" apps (no in-app purchases at all) "provide additional information about purchasing options without direct links" [2]. Rouxtine's Play build would be consumption-only. StreetComplete's accepted fix was a dialog with no link [3]. So a plain sentence such as "Rouxtine is free and funded by donations; see the project website", with no link and no URL, is probably the most Play allows. The FAQ's examples are about purchases, not donations, so this still carries review risk. Launch without it.

**Links the Play build must keep.** The GPL requires "clear directions next to the object code saying where to find the Corresponding Source" (`LICENSE:280-284`). A GitHub repo with a `FUNDING.yml` shows a Sponsor button [40], and a README with a Ko-fi link leads straight to Ko-fi. StreetComplete's GitHub link was rejected, although it came with a note pointing to donations [3]. To meet the GPL with the least review risk:
- In the About screen, show the source location as plain text (repo URL and release tag). Link only to the tag's source archive.
- Say the same in the store listing description.
- Point any privacy-policy link at a page with no donate button.

### 1.2 F-Droid

- **An in-app link is allowed.** The Inclusion Policy forbids proprietary tracking, ads and analytics libraries and requires FLOSS. It says nothing against donation links [31]. No anti-feature covers donations [32].
  - `NonFreeNet` is for apps that "promote or depend entirely on a proprietary network service" [32]. Ko-fi is proprietary, but the precedents show donation links don't draw this label:
    - Tasks.org's non-Play builds open `https://tasks.org/donate` from an in-app banner (`TaskListFragment.kt:633`) [56]. Its latest F-Droid builds (15.10–15.12, flavour `generic`) carry no anti-feature [34].
    - Shattered Pixel Dungeon's shared core opens `https://www.patreon.com/ShatteredPixel` (`SupporterScene.java:86`) [63]. Its F-Droid entry lists that Patreon URL as `Donate` and has no anti-features [35].
- **Metadata.**
  - `Donate:` takes a list of `https://` (or `taler://`, `bitcoin:`, `litecoin:`) URLs [33].
  - F-Droid advises "a link that the developer explicitly makes public, rather than something that is auto-generated 'button code' (e.g. PayPal, Stripe, Ko-Fi, Patreon, etc.)" [33]. So use the plain profile URL, `https://ko-fi.com/<name>`.
  - Donation links "need verification by upstream developers". F-Droid "can consider funding.json, FUNDING.yml or README files", or the app's main website [31].
  - Curators do edit these fields. AntennaPod's entry notes: "Upstream devs don't need more donation so donation links are removed." [36]
- **No recipe surgery needed.** With no Play Billing and no Google libraries, the `full` flavour builds as is. Other apps' recipes delete Play-only code even when a FOSS flavour exists, because the source scan covers the whole tree. Mullvad's recipe uses `rm: android/lib/billing` plus `sed` edits [37]. Tasks.org's recipe deletes Gradle lines and modules [34]. If Play-only code is ever added, keep it in its own source set or module.
- **Other F-Droid gates:**
  - Active development "for at least 3 months", unless the app has proven users or similar [31].
  - A unique application ID, "advised" to stem from a domain the developer owns. `io.github.nauroth.rouxtine` reverses `nauroth.github.io`.
  - Forks need a new ID, name and icon.
  - No downloading of executables or auto-updates without opt-in consent.
  - "All assets need to have valid legal licenses" [31].
- **Reproducible builds.** With the `Binaries:` field, "F-Droid will use upstream binaries if the verification succeeded" [33]. F-Droid then ships the owner-signed APK, so GitHub and F-Droid installs share one key and users can switch between them.

### 1.3 GitHub releases

- **Allowed everywhere.** No store rules apply. The Ko-fi link can go in the app, the README, the release notes and `.github/FUNDING.yml` (`ko_fi: USERNAME`, plus up to four custom URLs) [40].
- **Android developer verification:**
  - From 2026-09-30, apps from unregistered developers can't be installed "from participating stores in Brazil, Indonesia, Singapore, and Thailand on certified devices running Android 7+" [41].
  - Sideloads and other stores are not affected yet. Verification expands "globally for all apps on certified Android devices in 2027" [41][42].
  - Unregistered apps will then need the "advanced flow": a one-time setup with "a one-time, one-day wait". ADB installs keep working [42].
  - Register the package name and signing key(s). If F-Droid re-signs the app, its key's fingerprint can be added too [43].
  - A full Android Developer Console account costs $25. A free "limited distribution" account covers up to 20 devices [42].
- **Updater.** If the `full` build ever checks GitHub for updates, make the check opt-in. F-Droid lists "Checking for updates without your knowledge or permission" under Tracking [32], and forbids auto-update downloads without consent [31]. The Play build may not update itself "using any method other than Google Play's update mechanism" [20].

### 1.4 What to build

```kotlin
// app/build.gradle.kts (sketch)
android {
    flavorDimensions += "store"
    productFlavors {
        create("full") { dimension = "store"; isDefault = true } // F-Droid + GitHub releases
        create("play") { dimension = "store" }                   // Google Play
    }
}
```

- Put the Support screen and the Ko-fi URL in `src/full/`. Put a no-op stub in `src/play/`. The Play APK then contains no Ko-fi URL at all.
- F-Droid recipe: `gradle: [full]`. CI builds `fullRelease` (APK) for GitHub and `playRelease` (AAB) for Play.
- Play listing: no Ko-fi link and no donation wording. Take its screenshots from the `play` flavour.
- `play` About screen: plain-text source location plus the source-archive link (§1.1), and the legal notices (§5).

### 1.5 Other Play launch items checked along the way

- **Testing gate.** Personal accounts created after 2023-11-13 must run "a closed test for their app with a minimum of 12 testers who have been opted in continuously for at least 14 days" before production [18].
- **Target API.** From 2026-08-31, new apps and updates must target Android 16 (API 36), with an extension possible to 2026-11-01 [19].
- **Privacy bonus.** Play shows the developer's legal name, country and email. "If you decide to monetize on Google Play then Google will display your full address." [17] A free app with no Play payments avoids that.
- **No way back to paid.** "Once your app has been offered for free, the app can't be changed to paid" [23]. The DDA says free products "will remain free of charge" [24 s.3.7]. A paid version would need a new package.
- **Families policy.** If the declared target audience includes children, the Play Console answers must be accurate. Apps for children and older users "must not transmit AAID ... from children or users of unknown age". Monetization rules also apply [21].

## 2. The original question: donations, a premium UI, or a rewrite?

> "Can I ask for donations or sell in-app purchases like a premium UI, or do we need to rewrite everything ourselves?"

**Short answer.** You can ask for donations without rewriting anything. The GPL also allows selling, but the code stays public. On Play, in-app sales need the closed Play Billing library, so they need onionhoney's written permission. A rewrite is only needed to leave the GPL entirely.

### 2.1 What the GPL allows

- **Donations: yes.** s.10 forbids "any further restrictions" on GPL rights. Its example is "a license fee, royalty, or other charge for exercise of rights granted under this License" (`LICENSE:463-466`). A voluntary tip that unlocks nothing is neither. FAQ: "You can charge people a fee to get a copy from you. You can't require people to pay you when they get a copy from someone else." [45 #DoesTheGPLAllowRequireFee]
- **Selling copies: yes.**
  - s.4: "You may charge any price or no price for each copy that you convey, and you may offer support or warranty protection for a fee" (`LICENSE:205-206`).
  - FAQ: "Except in one special situation, there is no limit on what price you can charge." The exception is the s.6b written offer for source [45 #DoesTheGPLAllowMoney].
  - FSF: "we encourage people who redistribute free software to charge as much as they wish or can" [46].
- **But buyers can share.** "if someone pays your fee and gets a copy, the GPL gives them the freedom to release it to the public, with or without a fee" [45 #DoesTheGPLRequireAvailabilityToPublic]. The source must be offered "at no further charge" (`LICENSE:275-278`).
- **Premium features: legal, but not protected.**
  - Code in the APK is part of the Corresponding Source (`LICENSE:134-137`). The GPL covers "the whole of the work, and all its parts, regardless of how they are packaged" (`LICENSE:222-226`).
  - A GPL work can't count as an "effective technological measure", and whoever conveys it waives the power to forbid circumvention (`LICENSE:181-193`; [45 #DRMProhibited]). Anyone may patch out an unlock check and redistribute the result.
  - No GPL text says outright that premium in-app purchases are allowed; that follows from s.4 and from the absence of a ban.
  - The FSF calls a required ongoing fee to keep using GPL software "an additional requirement on top of the GPL, and the license prohibits it" [45 #SubscriptionFee]. So any gate must stay purely technical, with no EULA or ToS term against removing it.
- **A "premium UI" is code.** In Compose, colour schemes, shapes, motion and layouts are Kotlin, so they fall under `LICENSE:222-226`. Only standalone files (bitmaps, fonts, sounds) can plausibly carry their own licence. The FAQ notes that "some programs, particularly video games, can have artwork/audio that is licensed separately" [45 #WhatCaseIsOutputGPL]. OsmAnd does this: its UI/UX resources are CC-BY-NC-ND, and F-Droid flags the app `NonFreeAssets` [58][38].
- **Server-side premium (sync, backup).** "Mere interaction with a user through a computer network, with no transfer of a copy, is not conveying" (`LICENSE:99-101`). New server code that is never distributed has no source duty [45 #GPLRequireSourcePostedPublic, #UnreleasedMods]. Server code that reuses the ported GPL code is still a GPL-covered modification, so it can't be licensed differently.
- **Paid help.** s.4 allows "support or warranty protection for a fee" (`LICENSE:206`). Play's FAQ exempts "1:1 online paid services" between two individuals that can't be replayed, such as lessons or coaching, from Play Billing [2].

### 2.2 Why in-app purchases on Play are the hard part

- **The library is mandatory.**
  - In-app purchases must use Play Billing [1].
  - "By Aug 31, 2026, all new apps and updates to existing apps must use Billing Library version 8 or later", with an extension possible to Nov 1, 2026. Version 8 is accepted until 2027-08-31 and version 9 until 2028-08-31 [27].
- **The library is proprietary.**
  - Its POM (9.1.0, also 5.1.0) declares the "Android Software Development Kit License" [28]. A verifier found no sources jar and 337 obfuscated classes under `com/google/android/gms/internal/play_billing`.
  - It pulls in `play-services-*` artifacts under the same licence. Its `datatransport` dependencies are Apache-2.0 [28].
  - Unity's notice labels Billing 5.1.0 "Apache 2.0" but pastes MIT text and links the SDK terms [30]. Google's own POM contradicts it [28].
- **Its terms clash with the GPL.**
  - The SDK licence is "limited ... non-sublicensable", to "use the SDK solely to develop applications" (s.3.1). "You may not copy (except for backup purposes), modify, adapt, redistribute, decompile, reverse engineer, disassemble, or create derivative works of the SDK or any part of the SDK" (s.3.4) [29].
  - The licence contains no express grant to ship SDK parts inside an app; that permission is only implied [29]. Recipients get no source and no right to modify, so the library is GPL-incompatible.
- **The System Library exception doesn't cover it.**
  - It applies to code packaged with a "Major Component", meaning the kernel, window system, compiler or interpreter (`LICENSE:123-132`). The billing AAR is a Maven library compiled into each APK.
  - FAQ: linking "is making a combined work" [45 #GPLStaticVsDynamic].
  - This is an interpretation. No court ruling was found.
- **The fix is a s.7 permission, and onionhoney must give it.**
  - FAQ #GPLIncompatibleLibs offers wording; SPDX publishes it as `GPL-3.0-linking-exception` [45][47]. For a library without public source, drop the bracketed source sentence.
  - "Only the copyright holders for the program can legally release their software under these terms ... you cannot authorize the exception for them" [45].
  - s.7: if permissions cover only part of the program, "the entire Program remains governed by this License without regard to the additional permissions" (`LICENSE:349-352`). The owner's permission on Rouxtine's own files therefore does nothing for an APK that also contains onionhoney's ported code.
  - Anyone who conveys the work may remove such permissions (`LICENSE:354-356`).
- **The Play Terms of Service are a separate, smaller issue.**
  - Users get "personal, non-commercial use only", and "may not: sell, rent, lease, redistribute, ... modify, sublicense ... any Content ... except as specifically permitted" [25].
  - The FSF called Apple's similar store rules "exactly the kind of 'further restrictions' that are prohibited by the GPL", and said its analysis "would apply to all versions of the GNU GPL and AGPL" [26].
  - Play's carve-outs probably let the GPL govern: the Provider EULA clause, and "except as specifically permitted" [25]. The DDA grants users a "nonexclusive, worldwide, and perpetual license" to use the product and allows a developer EULA [24 s.5.3].
  - Many GPL apps ship on Play (§4). For code the distributor doesn't own, an explicit app-store permission removes the doubt; Terpoir has one [65].

### 2.3 Who holds the copyright to the code Rouxtine ports

- **Upstream licence.**
  - `LICENSE` is the unmodified GPLv3 text. The only difference from gnu.org's copy is a missing final newline.
  - Nothing says "only" or "or later". `package.json` has no licence field, and no source file has a header [48].
  - s.14: if no version is specified, "you may choose any version ever published by the Free Software Foundation" (`LICENSE:575-577`).
  - FAQ #LicenseCopyOnly: a copy of the GPL in the repo "does not explicitly state that the code in the same repository may be used under the GNU GPL", while "An explicit statement saying that eliminates all doubt" [45]. If the permissions did not apply, the result would be no licence at all, not a permissive one. Upstream is GPL-3.0 in practice, but only an explicit statement settles it.
- **Ports keep the same version.** FAQ #TranslateCode: translation is modification, and "the translated program must be covered by the same versions of the GNU GPL" [45]. GPL-3.0-only for the Kotlin ports is consistent with what upstream ships.
- **Authors.**
  - A `git blame` of master `0129460` gives every line of `Solver`, `Pruner`, `CubeLib`, `Algs`, `Analyzer`, `Evaluator` and `CubeSim` (also `Defs`, `Math`, `CachedSolver`) to Zhouheng "Jeffrey" Sun, GitHub onionhoney [49].
  - The 84 commits have one other human author: Eric Xu, with 3 commits (+14/−5) in `src/components/TopBarView.tsx` via PR #21 on 2023-08-19. That file is not planned for porting [49][50].
  - Open PR #31 ("support left handed FB/SB", Pedro Luiz, 2026-08-19, +295/−9, including +71 in `src/lib/CubeLib.tsx`) was not merged on 2026-10-28. Reusing its lefty code would add a copyright holder [50].
  - Upstream last changed on 2025-07-27 (master `0129460`, gh-pages `a75a5bb`) [49].
- **Vendored third-party code in upstream.**
  - `src/lib/min2phase/*` is a lightly modified copy of cubing/min2phase.js. Lucas Garron's 2018 licence says it is "dual-licensed under: MIT, GPLv3 or later" [51]. The package calls itself "A JS package for Chen Shuang's ... min2phase" [51].
  - Chen Shuang's own cs0x7f/min2phase has had a "License MIT" section in its README since commit `4d183b9` (2023-02-21). `src/Search.java` still has a GPL-3.0-or-later-only header [52]. This corrects roux-training-features.md §4, which lists min2phase as GPL-3.0-or-later only [67].
  - `src/lib/three/OrbitControls.js` comes from three.js (MIT) [53].
  - Upstream's copies carry no licence or attribution header, and the repo ships neither MIT notice (checked in `src/lib/min2phase/` and `src/lib/three/`). Rouxtine must add them itself.
- **Smart-cube and solver libraries:**
  - MIT: smartcube-web-bluetooth (Pau Oliva and Andy Fedotov; not on npm), gan-web-bluetooth, cube.js and kewb [54][55].
  - cubing.js and twsearch are "MPL-2.0 OR GPL-3.0-or-later" (twsearch's `Cargo.toml` says MPL-2.0 only; its README gives both). MPL-2.0 is file-level weak copyleft, not permissive [55].
  - GPL-3.0: csTimer, tnoodle-lib, open-gan, RubiksCube-TwophaseSolver and rob-twophase [55].
  - Under the chosen model, GPL-3.0 code may be combined with Rouxtine. MIT sources still keep later options open, because every GPL author would have to agree to an exception.

### 2.4 Do we need a rewrite?

- **No.** Donations, a paid app, or Play purchases with onionhoney's permission all work with a straight port.
- A Kotlin port is a modification. s.0 defines "modify" as to "copy from or adapt all or part of the work in a fashion requiring copyright permission" (`LICENSE:84-86`; [45 #TranslateCode]).
- Only an independent reimplementation would free the app from the GPL. Its size estimate is in Open questions. That matters only for a closed module, which the owner has ruled out.
- The GPL-derived core, by upstream line count [48]:
  - `CubeLib` 1,188, `Defs` 221, `Math` 66
  - `Pruner` 1,013, `Solver` 279, `CachedSolver` 69
  - `Analyzer` 258, `Evaluator` 280, `Algs` 423
  - `src/reducers` 1,448, `CubeSim` 485

## 3. Options

| Option | What | Pros | Cons | Verdict |
|---|---|---|---|---|
| **A. Donations outside Play** (Ko-fi) | Ko-fi in the `full` app, `FUNDING.yml`, README, website and F-Droid `Donate:`. No link in `play` | No permission or rewrite. No proprietary library, so a clean F-Droid build. Simplest Play compliance. Developer address stays off Play [17] | Play users, likely the biggest group, see no prompt. Donation income is usually small | **Chosen** |
| B. Play Billing tip jar | One-time "support" products that unlock nothing, like Twisty Timer's consumed `donation_tier1..5` [57] | Play users can give in the app. Features stay identical | Needs onionhoney's s.7 permission plus inbound terms for contributors. Fee 10% + 5% billing fee on the first $1M in AU/EEA/JP/UK/US [16]. Play then shows the full address [17]. Pulls in proprietary `play-services-*` [28]. Billing 8.3+ raises minSdk to 23 (8.0.0 allows 21, usable to 2027-08-31) [27] | Ruled out by the decision |
| C. Premium cosmetics via Play Billing, unlocked on F-Droid | The Tasks.org model: premium themes on Play, all unlocked in `generic` (`hasPro` returns `IS_GENERIC`) [56] | Proven at 500K+ Play installs. Learning features stay free | Same issues as B. Compose theme code is GPL, so anyone can share it unlocked. Gate must stay purely technical [45 #SubscriptionFee]. Conflicts with "no paywalled features" | Ruled out |
| D. Paid app on Play, free elsewhere | The DAVx5 model ($6.49 on Play, free on F-Droid) [60] | No library in the APK. Play handles one SKU [1] | Paid downloads also pay 10% + 5% billing fee on the first $1M in AU/EEA/JP/UK/US [16][15]. A free app can never turn paid, so it needs a separate package from day one [23][24]. A free twin risks the "Repetitive Content" spam policy [22] (OsmAnd and OsmAnd+ coexist [58]). Buyers may share the APK [45]. Beginners on Play hit a price | Ruled out |
| E. Relicence or broad permission from onionhoney | MIT/Apache-2.0 for their code, or GPL plus a Play-libraries and app-store permission [45 #ReleaseUnderGPLAndNF] | One author to ask. Removes the linking and store-terms questions | Upstream dormant since 2025-07-27 [49]. They may say no or not reply. Earlier releases stay GPL [45 #CanDeveloperThirdParty] | Optional: ask for an explicit licence statement (§6) |
| F. Clean reimplementation of the core | Write the solver and cube model from algorithm descriptions, not from the TS files | Full licensing freedom without anyone's consent | Large effort (unverified estimate below). Contamination risk: the owner has read the code. Gives up building openly on onionhoney's work | Not needed |
| G. Server-side premium | A paid hosted sync or backup service; the app stays GPL and free | Charges for a real running cost | Running a service. F-Droid `NonFreeNet` if the service is proprietary [32]. Selling it in-app still needs Play Billing or a regional program. Tasks.org sells sync via Play in `googleplay` and asks for a GitHub sponsorship in `generic` [56] | Possible later, outside the app |
| H. Paid coaching or support | Sold outside the app; GPL s.4 allows it (`LICENSE:206`) | No code or store impact. Play exempts 1:1 services from Play Billing [2] | Owner's time | Possible later |

## 4. Precedents

| App | Licence | Play | F-Droid | Money |
|---|---|---|---|---|
| Tasks.org [56][34] | GPL-3.0 | Free + in-app purchases, 500K+ | `generic` flavour, no anti-features in current builds | `googleplay` adds Play Billing, Firebase, PostHog and Google's oss-licenses library. `generic` stubs billing, unlocks pro, opens tasks.org/donate. Hosted sync needs a GitHub sponsorship in `generic` |
| Twisty Timer [57] | GPL-3.0-or-later | Free + in-app purchases, 1M+ | Not listed (metadata 404) | `donation_tier1..5` via anjlab 2.0.3, which depends on proprietary Billing 4.0.0. Purchases are consumed and unlock nothing. Last commit `2fcd6f7` (authored 2023-01-06, committed 2023-06-23) |
| OsmAnd [58][38] | Code GPLv3+, UI/UX CC-BY-NC-ND | OsmAnd+ $69.99; a free OsmAnd listing too | "OsmAnd~", `NonFreeAssets`, `NonFreeNet`, build-level Tracking | Billing 8.0.0 in every flavour. F-Droid's `prebuild.sh` deletes it with `sed` |
| DAVx5 [60][39] | GPLv3 | Paid, $6.49 | Free (`ose` flavour) | Donate, Liberapay, Bitcoin. Contributors sign a CLA |
| Conversations [59][39] | GPL-3.0 | Free on the US listing on 2026-10-28, though the README still says "If you bought the App on Google Play" | `free` flavour | Liberapay |
| AntennaPod [61][36] | GPL-3.0 | Free, no in-app purchases, 1M+ | `free` flavour | `play` adds Wear OS and Cast. F-Droid removed its donation links |
| Mullvad [62][37] | GPL-3.0-or-later | Free + in-app purchases | `oss` flavour, `NonFreeNet` | Billing lives in `lib/billing`, which F-Droid's recipe deletes. README: the App Store build "is not GPL licensed" but under Apple's EULA, relying on copyright ownership |
| Shattered Pixel Dungeon [63][35] | GPL-3.0 | Free; listing shows "In-app purchases" although the public Android build has no billing library | `Donate:` Patreon, no anti-features | In-app "Support the Game" opens Patreon. The Play binary may differ from the source, so it is no proof for Play |
| linphone [64] | GPLv3 or proprietary (dual) | n/a | n/a | Dual licensing works because Belledonne collects a contributor agreement for every patch |
| Terpoir [65] | `GPL-3.0-or-later WITH AdditionRef-Terpoir-AppStore-exception` | n/a | `foss` build fails if a Play library appears | s.7 "Google Play libraries" and "Application stores" permissions, plus DCO inbound terms. The seed commit (2026-10-10) is authored "Claude", so the wording is not lawyer-vetted |
| ics-openvpn [66] | GPLv2 + special exceptions | n/a | n/a | Linking exceptions by Arne Schwabe. A CLA "to retain my ability to relicense" |

Tasks.org, OsmAnd, Twisty Timer and Mullvad ship Play Billing without any written Play-library exception. Each relies on its main copyright holder doing the distributing [56][57][58][62].

## 5. GPL obligations Rouxtine must meet

These apply to every build that leaves the owner's hands: Play, F-Droid and GitHub.

1. **Licence text.** Give "all recipients a copy of this License along with the Program" (s.4). Keep `LICENSE` in the repo and make the text viewable in the app.
2. **Notices in each file.** "You should put a notice at the start of each source file, stating what license it carries" [45 #NoticeInSourceFile]. Use `SPDX-License-Identifier: GPL-3.0-only` plus copyright lines. Ported files keep onionhoney's copyright line next to the owner's.
3. **Modification notices** (s.5a). A modified work "must carry prominent notices stating that you modified it, and giving a relevant date" (`LICENSE:214-215`). Example: "Ported to Kotlin from onionhoney/roux-trainers `src/lib/Solver.tsx` @0129460 on <date>."
4. **Licence statement** (s.5b): "prominent notices stating that it is released under this License" (`LICENSE:217-219`).
5. **Appropriate Legal Notices in the UI** (s.5d, s.0). Each interactive UI must offer "a convenient and prominently visible feature" (`LICENSE:103-110`, `LICENSE:230-232`) that shows:
   - a copyright notice;
   - that there is no warranty;
   - that licensees may convey the work under the GPL;
   - how to view the licence.

   "If the interface presents a list of user commands or options, such as a menu, a prominent item in the list meets this criterion." So *Settings → About & licences* is enough.
6. **Source directions** (s.6d). Source may sit on another server "provided you maintain clear directions next to the object code saying where to find the Corresponding Source", and it must stay available "for as long as needed" (`LICENSE:280-286`; [45 #SourceAndBinaryOnDifferentSites]). In practice:
   - Tag every release, and attach the source archive to the GitHub release.
   - Name the tag in the Play listing and in the About screen (see §1.1 for the Play wording).
   - F-Droid builds from the tag anyway.
7. **What the source must include** (s.1). It is "all the source code needed to generate, install, and ... run the object code and to modify the work, including scripts to control those activities" (`LICENSE:134-137`). That means the Gradle files, the CI workflow, and the generator of any precomputed pruning tables.
   - Signing keys are not included. "Installation Information" applies only to User Products (`LICENSE:297-328`; [45 #GiveUpKeys]).
   - The build need not reproduce the exact binary hash [45 #MustSourceBuildToMatchExactHashOfBinary].
8. **Third-party notices.** MIT requires that "The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software" [51]. List these in the licences screen:
   - min2phase (Chen Shuang's MIT grant [52], or Lucas Garron's for the JS port [51]);
   - any three.js-derived camera code [53];
   - smart-cube protocol code (Pau Oliva, Andy Fedotov [54]);
   - AndroidX and other Apache-2.0 libraries.

   Build the screen from FLOSS code. Tasks.org keeps Google's `play-services-oss-licenses` in its Play-only flavour [56].
9. **No further restrictions** (s.10, `LICENSE:463`). No EULA, ToS or Ko-fi perk terms may limit copying, modifying or sharing.
10. **Identity.** F-Droid requires a fork to have its own ID, name and icon [31]. Rouxtine already has its own name and package ID.

## 6. Recommendation

1. **Flavours.** Ship `full` (Ko-fi link; F-Droid and GitHub) and `play` (no link, no donation wording). Keep the Ko-fi URL in `src/full/` only.
2. **Where Ko-fi goes.** Use the plain profile URL in `.github/FUNDING.yml` (`ko_fi:`), the README, the website, the GitHub release notes and F-Droid's `Donate:` field. That also gives F-Droid its verification [31][33][40].
3. **Play build and listing.**
   - No Ko-fi anywhere.
   - Source directions as plain text plus a source-archive link.
   - A privacy-policy page without a donate button.
   - Screenshots taken from `play`.
4. **Don't chase Play exceptions.** Skip the regional programs and don't ask Play support to bless a "creator tip" link. Revisit only if Google publishes donation-specific rules or the US injunction changes. Optional later: a StreetComplete-style sentence with no link, accepting some review risk [3][2].
5. **GPL hygiene from the first commit.**
   - `GPL-3.0-only` SPDX headers on every file, and modification notices on ported files.
   - The licences screen, with MIT notices for vendored code.
   - A CONTRIBUTING file stating that contributions are GPL-3.0-only with a DCO sign-off. If a Play-library exception is ever wanted, the inbound terms must include it from then on, as Terpoir does [65]. Otherwise every contributor becomes another person to ask.
6. **Write to onionhoney once.** Tell them about Rouxtine. Ask them to confirm the licence explicitly, which per FAQ #LicenseCopyOnly "eliminates all doubt" [45]; GPL-3.0-only is the natural reading. Optionally ask in the same message for a Terpoir-style app-store permission. Don't block on the reply: upstream has been quiet since 2025-07-27 [49].
7. **Prefer MIT sources for new code**: min2phase under Chen's MIT grant, smartcube-web-bluetooth and gan-web-bluetooth. The GPL doesn't require this, but it keeps future options open.
8. **Before the 2027 rollout,** register the package and signing key for Android developer verification [41][43]. Consider reproducible builds, so F-Droid ships the owner-signed APK [33].

## Sources

[1]: https://support.google.com/googleplay/android-developer/answer/9858738 (Play Payments policy)
[2]: https://support.google.com/googleplay/android-developer/answer/10281818 (Understanding Google Play's Payments policy, FAQ)
[3]: https://www.theregister.com/2022/03/02/google_blocks_android_tool/ (StreetComplete, 2022-03-02)
[4]: https://www.phoronix.com/news/WireGuard-Ejected-Play-Store (WireGuard, 2019-10-16)
[5]: https://gigazine.net/gsc_news/en/20260902-ankidroid-google-play-policy-violation/ (AnkiDroid, 2026-09-02; primary https://github.com/ankidroid/Anki-Android/issues/21656 blocked by the session proxy, HTTP 403)
[6]: https://support.google.com/googleplay/android-developer/answer/15582165 (Play policy update for US users)
[7]: https://support.google.com/googleplay/android-developer/answer/16470497 (US external content links program)
[8]: https://support.google.com/googleplay/android-developer/answer/16497028 (US alternative billing)
[9]: https://support.google.com/googleplay/android-developer/answer/17161464 (billing choice program)
[10]: https://support.google.com/googleplay/android-developer/answer/14372887 (EEA external offers program)
[11]: https://support.google.com/googleplay/android-developer/answer/12348241 (EEA alternative billing)
[12]: https://support.google.com/googleplay/android-developer/answer/12570971 (user choice billing pilot)
[13]: https://support.google.com/googleplay/android-developer/answer/13821247 (billing alternatives by country)
[14]: https://support.google.com/googleplay/android-developer/answer/16787536 (Japan external payments program)
[15]: https://support.google.com/googleplay/android-developer/answer/16954621 (new service fees and rollout dates)
[16]: https://support.google.com/googleplay/android-developer/answer/112622 (service fees)
[17]: https://support.google.com/googleplay/android-developer/answer/13628312 (developer information shown on Play)
[18]: https://support.google.com/googleplay/android-developer/answer/14151465 (testing requirements for new personal accounts)
[19]: https://support.google.com/googleplay/android-developer/answer/11926878 (target API level requirements)
[20]: https://support.google.com/googleplay/android-developer/answer/9888379 (Device and Network Abuse policy)
[21]: https://support.google.com/googleplay/android-developer/answer/9893335 (Families policies)
[22]: https://support.google.com/googleplay/android-developer/answer/9899034 (Spam policy)
[23]: https://support.google.com/googleplay/android-developer/answer/6334373 (set your app's price)
[24]: https://play.google/developer-distribution-agreement.html (Developer Distribution Agreement, effective 2025-09-15)
[25]: https://play.google/play-terms/ (Google Play Terms of Service, 2024-07-01)
[26]: https://www.fsf.org/blogs/licensing/more-about-the-app-store-gpl-enforcement (FSF, 2010-05-26)
[27]: https://developer.android.com/google/play/billing/deprecation-faq (Billing Library deprecation FAQ)
[28]: https://dl.google.com/android/maven2/com/android/billingclient/billing/9.1.0/billing-9.1.0.pom ; https://dl.google.com/android/maven2/com/android/billingclient/billing/5.1.0/billing-5.1.0.pom ; https://dl.google.com/android/maven2/com/google/android/datatransport/transport-runtime/3.1.8/transport-runtime-3.1.8.pom ; https://dl.google.com/android/maven2/com/google/android/gms/play-services-location/19.0.0/play-services-location-19.0.0.pom
[29]: https://developer.android.com/studio/terms (Android SDK License Agreement)
[30]: https://docs.unity3d.com/Packages/com.unity.purchasing@4.9/license/Third%20Party%20Notices.html
[31]: https://f-droid.org/docs/Inclusion_Policy/
[32]: https://f-droid.org/docs/Anti-Features/
[33]: https://f-droid.org/docs/Build_Metadata_Reference/
[34]: https://gitlab.com/fdroid/fdroiddata/-/blob/master/metadata/org.tasks.yml
[35]: https://gitlab.com/fdroid/fdroiddata/-/blob/master/metadata/com.shatteredpixel.shatteredpixeldungeon.yml
[36]: https://gitlab.com/fdroid/fdroiddata/-/blob/master/metadata/de.danoeh.antennapod.yml
[37]: https://gitlab.com/fdroid/fdroiddata/-/blob/master/metadata/net.mullvad.mullvadvpn.yml
[38]: https://gitlab.com/fdroid/fdroiddata/-/blob/master/metadata/net.osmand.plus.yml ; https://gitlab.com/f-droid-mirrors/OsmAnd-submodules/-/blob/master/prebuild.sh
[39]: https://gitlab.com/fdroid/fdroiddata/-/blob/master/metadata/at.bitfire.davdroid.yml ; https://gitlab.com/fdroid/fdroiddata/-/blob/master/metadata/eu.siacs.conversations.yml
[40]: https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/displaying-a-sponsor-button-in-your-repository
[41]: https://developer.android.com/developer-verification
[42]: https://developer.android.com/developer-verification/guides/faq
[43]: https://developer.android.com/developer-verification/guides/open-source-app-registration
[44]: https://www.gnu.org/licenses/gpl-3.0.txt (identical to this repo's `LICENSE` except the final newline)
[45]: https://www.gnu.org/licenses/gpl-faq.html (anchors as cited; snapshot https://web.archive.org/web/20261023105423/https://www.gnu.org/licenses/gpl-faq.html)
[46]: https://www.gnu.org/philosophy/selling.html
[47]: https://spdx.org/licenses/GPL-3.0-linking-exception.html
[48]: https://github.com/onionhoney/roux-trainers/tree/0129460a5c30082ea0e29b4489623889118078ec (LICENSE, package.json, README, src/lib, src/reducers)
[49]: https://github.com/onionhoney/roux-trainers/commits/master (read from a full clone; GitHub pages and API were blocked, HTTP 403)
[50]: https://github.com/onionhoney/roux-trainers/pull/21 ; https://github.com/onionhoney/roux-trainers/pull/31 (read as git refs `refs/pull/*`)
[51]: https://github.com/cubing/min2phase.js/blob/a465b5e458fc1850299cab3c6254973c7f4c76ae/LICENSE.md ; https://github.com/cubing/min2phase.js/blob/a465b5e458fc1850299cab3c6254973c7f4c76ae/README.md
[52]: https://github.com/cs0x7f/min2phase/blob/4d183b9eff8119cac72bc50ef35a7d8990740e06/README.md ; https://github.com/cs0x7f/min2phase/blob/master/src/Search.java
[53]: https://registry.npmjs.org/three/latest
[54]: https://github.com/poliva/smartcube-web-bluetooth/blob/master/LICENSE.txt ; https://registry.npmjs.org/smartcube-web-bluetooth (404) ; https://github.com/afedotov/gan-web-bluetooth/blob/master/package.json
[55]: https://github.com/ldez/cubejs/blob/master/LICENSE ; https://github.com/luckasRanarison/kewb/blob/master/LICENSE ; https://github.com/cubing/cubing.js/blob/main/package.json ; https://github.com/cubing/twsearch/blob/main/README.md ; https://github.com/cs0x7f/cstimer/blob/master/LICENSE ; https://github.com/thewca/tnoodle-lib/blob/master/LICENSE ; https://github.com/turnerglenn-cmyk/open-gan/blob/main/NOTICE ; https://github.com/hkociemba/RubiksCube-TwophaseSolver/blob/master/LICENSE ; https://github.com/efrantar/rob-twophase/blob/master/LICENSE
[56]: https://github.com/tasks/tasks/blob/4d7792727f1a96c92410fa00665a5e657681600f/app/build.gradle.kts#L125-L133 ; …/app/build.gradle.kts#L288-L306 ; …/app/src/main/java/org/tasks/billing/Inventory.kt#L60-L77 ; …/app/src/main/java/com/todoroo/astrid/activity/TaskListFragment.kt#L617-L636 ; …/app/src/main/java/org/tasks/auth/SignInActivity.kt ; …/app/src/generic/java/org/tasks/billing/BillingClientImpl.kt ; https://play.google.com/store/apps/details?id=org.tasks
[57]: https://github.com/aricneto/TwistyTimer/blob/2fcd6f7d2697a0810f37245ac033f1da12448382/app/src/main/java/com/aricneto/twistytimer/fragment/dialog/DonateDialog.java#L50-L62 ; …/app/src/main/java/com/aricneto/twistytimer/activity/MainActivity.java#L543-L547 ; …/app/build.gradle#L88 ; https://repo1.maven.org/maven2/com/anjlab/android/iab/v3/library/2.0.3/library-2.0.3.pom ; https://play.google.com/store/apps/details?id=com.aricneto.twistytimer
[58]: https://github.com/osmandapp/OsmAnd/blob/master/LICENSE ; https://github.com/osmandapp/OsmAnd/blob/master/OsmAnd/build-common.gradle ; https://play.google.com/store/apps/details?id=net.osmand.plus ; https://play.google.com/store/apps/details?id=net.osmand
[59]: https://codeberg.org/iNPUTmice/Conversations/src/branch/master/README.md ; https://play.google.com/store/apps/details?id=eu.siacs.conversations
[60]: https://github.com/bitfireAT/davx5-ose/blob/main/README.md ; https://github.com/bitfireAT/davx5-ose/blob/main/CONTRIBUTING.md ; https://play.google.com/store/apps/details?id=at.bitfire.davdroid
[61]: https://github.com/AntennaPod/AntennaPod/blob/develop/playFlavor.gradle ; https://github.com/AntennaPod/AntennaPod/blob/develop/playback/cast/build.gradle
[62]: https://github.com/mullvad/mullvadvpn-app/blob/main/android/app/build.gradle.kts ; https://github.com/mullvad/mullvadvpn-app/blob/main/README.md ; https://github.com/mullvad/mullvadvpn-app/blob/main/CONTRIBUTING.md
[63]: https://github.com/00-Evan/shattered-pixel-dungeon/blob/master/core/src/main/java/com/shatteredpixel/shatteredpixeldungeon/scenes/SupporterScene.java ; https://play.google.com/store/apps/details?id=com.shatteredpixel.shatteredpixeldungeon
[64]: https://github.com/BelledonneCommunications/linphone-android/blob/master/README.md
[65]: https://github.com/nolsen311/Terpoir/blob/main/LICENSING.md
[66]: https://github.com/schwabe/ics-openvpn/blob/master/doc/LICENSE.txt ; https://github.com/schwabe/ics-openvpn/blob/master/README.md
[67]: roux-training-features.md (this folder), §4 and §6

## Open questions / unverified

- **US link-outs.** For US users the injunction says Google "will not prohibit ... link to transactions". The same page keeps "current policies and programs" and routes links through the external content links program [6][7]. Whether a Ko-fi link shown only to US users is allowed without enrolment, and what a donation would owe under that program, is unclear.
- **Creator tips.** Would Google accept the "creator tips" FAQ [2] for a developer's own no-perks donation link? No case was found either way, apart from the three rejections. AnkiDroid's outcome comes from Gigazine only; its GitHub issue was blocked by the session proxy (HTTP 403) [5].
- **Pages that lead on to Ko-fi.** Does a Play listing's website field, or an in-app source link, count as "leading users" to payment when the target page carries a Ko-fi or Sponsor button? StreetComplete's case suggests yes, at least when combined with donation wording [3].
- **Countries without Play Billing.** Does the anti-steering rule apply where Play Billing is unavailable? The FAQ only lifts the duty to use Play Billing [2].
- **F-Droid labels.** Could F-Droid tag an in-app Ko-fi link `NonFreeNet` ("promote ... a proprietary network service") [32]? Current precedents say no [34][35].
- **Ko-fi's own terms.** Fees and payout terms on ko-fi.com and help.ko-fi.com were behind a Cloudflare bot check and were not read.
- **EU trader status.** Whether an EU-based owner taking outside donations must declare trader status on Play, which can show an address and phone number, was not checked.
- **Play ToS and onionhoney's code.** Play's user terms versus the GPL is probably fine thanks to its carve-outs, but untested [25][26]. An app-store permission from onionhoney would settle it.
- **Upstream's GPL version.** Technically "any version ever published" (s.14). GPL-3.0-only is the chosen and natural reading; an explicit statement from onionhoney would settle it [45 #LicenseCopyOnly].
- **PR #31.** Whether it got any reply was not checked; GitHub pages and API for repositories outside the session were blocked (HTTP 403). The git refs show it unmerged [50].
- **Stanford.** Some upstream commits came from Stanford-network identities. Whether Stanford's IP policy claims anything was not checked.
- **min2phase MIT grant.** Chen Shuang's MIT grant is in the README, while the file headers still say GPL-3.0-or-later. Ask Chen Shuang, or rely on the README grant [52].
- **csTimer influence.** smartcube-web-bluetooth credits csTimer (GPL) for the timestamp linear-fit idea. Confirm no csTimer code was copied before porting that function under MIT.
- **Alg lists.** Does copyright cover alg lists? The US *Feist* and 37 CFR 202.1 reasoning and the EU Database Directive (96/9/EC) argument were not verified. Verified fact: upstream `src/lib/Algs.tsx` credits no source for its 42 CMLL algs, and line 73 reads `// my 2h algs` [48].
- **Rewrite estimate.** A clean rewrite of about 3–5k Kotlin lines (3–6 developer-weeks) is an unverified estimate. So is the reliance on 17 U.S.C. §102(b) (ideas and methods are not protected).
- **Psysonic.** Its contributor-consent drive (issue #1772, about 28 agreements) was not verified (GitHub blocked).
- **Shattered Pixel Dungeon on Play.** Whether its Play build contains Play Billing, or shows the Patreon button, is unknown [63].
- **Conversations' Play price.** It may vary by region [59].
- **Verification stores.** Which stores count as "participating" for developer verification is not listed on the pages read [41][42].
- **Registered business.** Is the owner one, or will Rouxtine be? The EEA external offers and alternative billing programs and the user choice billing pilot need one [10][11][12]; none of them fits donations anyway.
- **Tooling gaps.** gnu.org reset connections on 2026-10-29. FAQ quotes come from a copy fetched on 2026-10-28 and the archive.org snapshot [45]. A verifier's count of 337 obfuscated billing classes was not repeated here.
