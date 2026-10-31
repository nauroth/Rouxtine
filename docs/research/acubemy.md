# acubemy: ideas worth adapting for Roux

Researched 2026-10-31 from public sources only (no APK inspection): the Google Play and App Store listings (description, version history, data safety, public reviews), the developer's website, FAQ, blog, imprint, privacy policy and terms, one public solve page, YouTube video descriptions and chapters, public GitHub READMEs and one Reddit thread snapshot. No app binary was downloaded, decompiled or disassembled, and no code was read or copied. `[n]` refers to the numbered sources at the bottom. Rouxtine references such as `rouxtine-design.md §5` point to `docs/` in `nauroth/rouxtine`; research notes are in `docs/research/`. A line marked *inference* is my reading, not a stated fact.

## Summary

- **What it is.** acubemy is a closed-source smart-cube timer and trainer for Android, iOS and the web, made by acubemy GmbH in Karlsruhe, Germany [1][4][15]. It is free with no ads. Premium costs €4.99/month or €49.99/year [1][7].
- **CFOP first, Roux second.** Its Smart Training drill cards target OLL and PLL, with cross and F2L trainers "coming next" [21]. Roux support is real but thin: CMLL cases (v2.5.0), CMLL statistics (v2.6.0), and Roux reconstructions improved with gyro data (v2.7.0) [4]. No public source mentions EO, EOLR, 4c or block-building trainers.
- **The best ideas for Rouxtine** all need a smart cube, so they land in R2+ with the SC module (`scope.md` R2+ item 1):
  1. A per-solve step table: recognition, execution, step time, turns and TPS for every step [9].
  2. A wrong-case alert after a solve ("you did the wrong OLL; this was the case") [3][5]. For Roux this means CMLL and EOLR misrecognition feeding the CMR confusables queue.
  3. **Cube state sync:** scrambles are computed from the cube's current state, so the user never resets it between drill reps [21].
  4. Gyro-assisted reconstruction [4][10]. For Roux, this can tell M from L+R′ and r from L plus a rotation.
  5. Hands-free control by "no-op" cube gestures such as R4 and L4 (*inference*, [21][5]).
- **Ideas that fit R1** need no hardware: a "prove you know it" card per case, a per-case "too hard" switch, an "N in a row" clear rule with goal times from the user's own data, longer averages (ao50/ao100/ao1000), scramble history, keep-screen-on, and a free full data export [21][4][5][7].
- **Skip:** accounts, cloud sync, PvP, prize tournaments, a global daily-scramble leaderboard, crowd benchmarks, and the plugin/games system. Each needs a server, or is not Roux training, or both (the rules at the top of `scope.md`; `scope.md` "Out").
- **Avoid what reviews complain about:** lost data after save errors (a 1-star review reports 5,000 solves gone), UI clipped on small phones, too many taps to reach learning, and training behind a subscription [5][3][28].
- **About inspecting the APK:** not done, and not worth doing. acubemy's terms say users "may not copy, redistribute, reverse engineer, or attempt to extract the source code" [17]. The app has no open licence, so none of its code could go into a GPL-3.0 app anyway. The public material above already shows the features. For Bluetooth protocols, the MIT and GPL drivers chosen in `smart-cubes.md` are the right sources.

## 1. What the app is

### Developer

- The store listings name the developer account "Tan Grumser", based in Germany [1][4].
- The website's operator is acubemy GmbH, Karlsruhe, registered at the Mannheim local court [15]. Tournaments are run by the same company [7].
- Replies to reviews are signed "the acubemy team" or by a named team member, so more than one person seems to work on it (*inference*) [3][5].

### Platforms

- **Android:** package `com.acubemy.acubemy` on Google Play. It is listed under "Entertainment" with 5K+ downloads, 4.9 stars from 108 ratings, and was last updated on 2026-10-24 [1].
- **iOS, iPadOS, Apple silicon Macs and visionOS:** iOS 15 or later, English only, 75.6 MB, 4.8 stars from 29 ratings, age rating 13+ [4].
- **Web:** runs in Chrome, Edge and Opera through Web Bluetooth. Firefox and Safari are not supported. On iOS the FAQ points to the Bluefy browser [7].
- **Release pace:** the App Store history runs from v1.11.0 (2026-04-10) to v2.7.1 (about 2026-10-25), with 18 listed releases in six and a half months [4].

### Price model

- The app is free, with no ads [1][4].
- **Basic (free):** solving analytics, cloud sync and backup, all cubes, local and online matches [7].
- **Premium:** €4.99/month or €49.99/year, or $4.99/$49.99 in the US App Store. It adds OLL/PLL statistics, learning new algorithms, alg drills, Personalized Training, data exports and priority support [7][4].
- **Supporter:** €14.99/month or €149.99/year. It adds "Enable further Development for acubemy" and a "Special Supporter Badge and Role" [7][4].
- Premium has a 7-day free trial and a 3-day refund window on the first payment. Upgrading applies Premium analysis to past solves too [7].
- Web payments go through Stripe, and app payments through Google Play or Apple, managed with RevenueCat [16].
- The FAQ says iOS in-app purchases are "not yet available" [7], but the App Store already lists them [4]. The FAQ is out of date.
- They use creator marketing: a sponsored video carries a referral link and a discount code [24].

### Supported smart cubes and timers

- The FAQ table lists these models as supported [7]:
  - GAN: 16 models, from the GAN 356 i (2019) to the GAN 16 UI, i4 and i Carry 4 (2026).
  - MoYu: 7 models, including the WeiLong V10 AI, V11 AI and Super AoLong AI.
  - QiYi: the X-Man Tornado V4 AI and the QiYi Smart Cube.
  - GoCube: 2 models. Rubik's Connected: 2 models. Giiker: 3 models.
- Both cubes the owner plans to buy, the QiYi AI and the GAN i4, are on that list [7].
- Since v2.6.0 it also supports two 2x2 smart cubes, GAN and MoYu WeiPo [4].
- On the web, some encrypted cubes need the MAC address typed in once. The native apps read it automatically [7]. This matches `smart-cubes.md` §4.
- **Timers:** a space-bar and touch timer, and "pick up and put down times for smart cubes" (v2.1.1) [4]. A manual inspection timer came in v2.4.0 [4]. No public source mentions the GAN Halo, the QiYi timer or a Stackmat.
- Scrambles are WCA random-state, from "a custom implementation of Kociemba's two-phase solver" [7].
- Turns are counted in STM, and rotations are not counted [7].

### CFOP features (and other methods)

- Each step is reconstructed: cross, each F2L slot with its F2L case number, OLL case and PLL case, each with its own split [9].
- After a solve, it tells you which OLL or PLL you got wrong and what the right one was [3][5].
- F2L support arrived in v1.12.0 [4]. ZBLL (472 cases, detected in ZZ solves) came in v2.7.0 [4].
- In v2.5.0 it added all WCA events (2x2 to 7x7, Megaminx, Pyraminx, Skewb, Square-1, Clock) [4].
- The blog has an OLL fingertricks Shorts series and a CFOP "three pillars" article [8].

### Roux support

- The Play listing says solves are reconstructed for "CFOP, Roux and ZZ", with recognition and execution timed separately [1].
- The iOS listing says the algorithm trainer "Covers CFOP, Roux and ZZ" [4].
- Roux items in the changelog: "CMLL cases for Roux solvers" (v2.5.0), "CMLL statistics … with cases grouped by subset" (v2.6.0), and "More accurate Roux reconstructions using your cube's gyro data" (v2.7.0) [4].
- Our earlier note found Roux insights after 12 Roux solves [30].
- A sponsored creator thanked them for "Roux support" in July 2026 [24].
- Not found in any public source: EO, EOLR or 4c recognition, LSE efficiency, FB or SB trainers, an inspection trainer for Roux, or a public Roux solve page.

### Training and plans

- **The pitch:** "Set your target solve time and acubemy tells you exactly which split is holding you back, based on 25M+ analyzed solves. Your entire training plan is built around reaching that goal" [1].
- **Smart Training 2.0** (v2.4.0, Premium) [4][21]:
  - It unlocks after 12 solves and builds "an infinite queue of cards from your weakest cases".
  - **Drill cards** show your slowest OLL and PLL cases. "Beat the goal time three times in a row to clear the card."
  - **Adaptive goal times** come from your target average and your recent executions. Marking a case "too hard" (L4) relaxes its goal.
  - **"Forgot the algorithm?" (R4):** the app sets the case up on your physical cube and shows the algorithm while you execute it.
  - **Check cards** are spaced repetition for algorithms you are learning.
  - **Show cards:** "prove you already solve a case in one look and its memorization jumps to 100%".
  - **Custom trainings**, e.g. a PLL-only drill; a **learning stack** for any case from the alg sets; a **virtual-only mode** that turns off the physical setup.
- **Cube state sync** (v2.4.0): "acubemy computes the scramble from your physical cube's current state", so no reset is needed [21].
- **Sessions** (v2.4.0): a new creation flow, an incognito mode and a "solving-method lock" [4].
- **Coming next:** "smart trainers for cross, F2L and more" [21], plus cross and metronome training [8].
- The training page needs a login, so the plan itself was not inspected [30].

### Analysis

- **Per solve:** time, TPS, turns and "fluency", the share of the solve spent turning [9][8].
- **Step table** [9]: for each step, recognition, execution, step time, cumulative time, turns and TPS. It also shows totals, e.g. recognition 5.60 s (34 %) and execution 10.93 s (66 %) for a 16.53 s solve.
- **Statistics:** averages, PBs, TPS and movecount over time [1]. The timer screen shows single, ao5, ao12, ao50, ao100 and ao1000 [31]. ao50 and ao1000 can be shared (v2.6.0) [4].
- **"Insights":** strengths and weaknesses across the whole solve [8]. The site markets "OLL/PLL/F2L weakness detection" and benchmarking "against 3 million+ solves" [6], while the store listings say "25M+" [1][4].
- **Step evaluations:** "smarter step-by-step solve evaluations" (v2.4.0) [4]. One reviewer likes the labels "optimal solve, brilliant, blunder, spam" [5].
- **Three pillars blog:** time = movecount ÷ TPS, with fluency on top. It works through a counterfactual by hand for one solve [8]. Our `training-regimens.md` already adapts this per Roux stage [30].

### Reconstructions

- A full move list per step, with rotations taken from the gyro [9][12][13].
- A replay at 0.1× to 2× speed with a gyro toggle [9].
- A public share page per solve, showing the cube model and a "Use this scramble" button [9].
- **Gyro notes from their own cube reviews** [10][11]:
  - The GAN i4 gyro updates at about 10 Hz, and its drift "accumulates" and needs "a software measure on our side" between solves.
  - Cubes without a gyro (e.g. the GAN iCarry 4) cannot report orientation or rotations, which limits the analysis.

### Social features

- Local and online 1v1 matches since v2.2.0 (July 2026) [4]. Real-time PvP on the same scramble [1].
- Online tournaments every two weeks, with elimination rounds, leaderboard prizes and raffles. Prizes are partner-shop vouchers or Premium time [1][7]. Under-18s need a parent's consent [7].
- A daily scramble, the same for everyone, with a global leaderboard [1].
- A Discord server, where "every development update" is posted first and users submit solves for critique videos [21][25]. Also YouTube, TikTok and Instagram accounts [6].
- **Games & Plugins** (Sept 2026): sandboxed web plugins loaded from public GitHub repos, built with an MIT JavaScript SDK [18][19][23]. Examples include a rhythm game, a gyro "cube roller", and a community metronome plugin [19][20].

### Data practices (for contrast)

- Accounts use Firebase Authentication. Data is stored on Google Cloud in Europe [16].
- On the web, the site uses PostHog, Google Analytics, Meta Pixel and Reddit Pixel, only after cookie consent [16]. It also uses Sentry, Stripe, RevenueCat, Resend and Vercel [16].
- **Play Data safety:** email addresses may be shared for "Advertising or marketing", device IDs for analytics, and app interactions for advertising [2].
- A guest mode with no account arrived in v2.5.0 [4].

## 2. Ideas for Rouxtine

"In Rouxtine" cites where the idea already is, or says **new**. Releases follow scope.md: R1, R2+, Later, or skip.

| # | acubemy feature | Roux equivalent | In Rouxtine | Release | Why |
|---|---|---|---|---|---|
| 1 | Per-solve step table: recognition, execution, step time, cumulative, turns, TPS [9] | Rows for FB, SB, CMLL, then EO, LR and 4c inside LSE | Partly. SC module with automatic Roux splits (`scope.md` R2+ item 1); per-attempt recognition and execution fields (`rouxtine-design.md §9`). Column set and "Use this scramble" are **new** | R2+ (with SC) | Per-move timing needs a smart cube. R1's manual splits give only step times |
| 2 | Fluency %, TPS while turning and movecount on every solve [8][9] | Same three numbers per Roux stage, movecount against the solver optimum | Partly. Recognition / execution / efficiency findings (`rouxtine-design.md §5`); counterfactual per stage (`training-regimens.md` §6.5, counterfactual seconds). "Fluency" as a user-facing number is **new** | R2+ (with SC) | Needs move timestamps. Show STM and say which metric is used, as acubemy's FAQ does [7] |
| 3 | Wrong-case alert after a solve: "which OLL you messed up" and the right one [3][5] | Executed CMLL alg does not match the CMLL case; wrong EOLR choice | **New** for real solves. Drills have the CMR confusables queue (`scope.md` "Modules in R1", CMR row) | R2+ (with SC) | CMLL misrecognition is a common Roux leak. Each detected error can feed the confusables queue automatically |
| 4 | "Jump from any case directly into targeted practice" [1]; post-solve alerts on time you could have saved, with a drill [30] | From a reconstruction: "Drill this CMLL case", "Plan this FB in INS", "Compare SB with the solver" | Partly. "Drill missed / Drill slow" after a block (`rouxtine-design.md §5`); SLOW and REC (`scope.md` R2+ item 2). Solve-to-drill link is **new** | R2+ | Closes the loop from timed solves to drills, which the design keeps separate for stats (`rouxtine-design.md §3`) |
| 5 | Step labels such as "optimal", "brilliant", "blunder" [5][4] | Label FB, SB and LSE by moves over solver optimum | Partly. Efficiency vs optimum (`rouxtine-design.md §5, §9`). Labels are **new** | R2+ (with SLOW/SC) | A reviewer finds labels encouraging [5]. Use neutral words ("optimal", "+3 moves"): the design rules out pressure copy (`rouxtine-design.md §8`) |
| 6 | Drill cards: clear a case by beating its goal time 3 times in a row; goals adapt [21] | CMX execution, EOLR and 4c drills | Partly. Weakest selector and `target(...)` / `untilMiss` stop rules (`rouxtine-design.md §3, §5`). "N in a row" clear rule and per-case adaptive goal are **new** | R1 | Pure selector logic, no hardware. Base goals on the user's own recent times, not on a split table: Roux tables are provisional (`rouxtine-design.md §5`) |
| 7 | "Too hard" marks a case and relaxes its goal [21] | Per-case flag in any case module | **New** | R1 | Small. Keeps one hard case from stalling a block |
| 8 | Show cards: prove one-look knowledge and the case jumps to "known" [21] | "I know this" one-shot test per CMLL, EOLR or 4c case | Partly. Placement test covers CMLL and EOLR recognition (`rouxtine-design.md §6`). Per-case version is **new** | R1 | Lets switchers and experienced Rouxers skip learning queues quickly |
| 9 | Check cards (SRS), learning stack, suggestions of what to learn next [21][5][30] | FSRS-6 for CMLL, EOLR, 4c; module gates | Yes. SRS selector (`rouxtine-design.md §5`); recommendations (`rouxtine-design.md §6`) | R1 (already) | Already planned. Keep it to one tap from a case to its learn view (see §3) |
| 10 | Cube state sync: scramble computed from the cube's current state, no reset [21] | Case scrambles for CMLL, EOLR, SB-LP from wherever the cube is | Partly. "Smart-cube input in every case module" (`scope.md` R2+ item 1). State sync is **new** | R2 (with SC) | Removes the reset between reps, the main friction of smart-cube drills. Needs a full-cube solver (the min2phase port) |
| 11 | "Forgot the algorithm?": the app guides the case onto the physical cube and shows the alg [21] | Same for CMLL and EOLR | **New** | R2+ | Depends on #10 |
| 12 | Control by cube gestures: "too hard" (L4), "forgot" (R4); usable "without taking the hand off the cube" [21][5]. *Inference:* L4 and R4 are four quarter turns, which leave the state unchanged | No-op gestures to skip, show the alg, or mark too hard | **New** | R2+ (with SC) | Fits drills well. Avoid U and M gestures: LSE uses them all the time |
| 13 | Gyro data for "more accurate Roux reconstructions" [4]; i4 gyro about 10 Hz with drift, reset between solves [10]; the plugin SDK streams a quaternion at 20–60 Hz with a "white up, green front" calibration [19] | Tell M from L+R′, and r from L plus a rotation; count rotations | Partly. Pair merging and modulo-rotation detection (`smart-cubes.md §3.5`); gyro stored in the log (`training-regimens.md` §6.3, smart-cube solves row). Gyro-assisted detection is **new** | R2 (test on the GAN i4) | M and r are core Roux moves. The QiYi AI has no gyro (`smart-cubes.md` §3.1), so time-window merging stays the fallback. Re-zero the gyro at each solve start |
| 14 | "Pick up and put down times for smart cubes" [4] | Start and stop SC solve timing from cube events | Partly. SC cube-clock timing (`scope.md` R2+ item 1, SC module) | R2+ | Exact meaning is not public (see §5) |
| 15 | ao50/ao100/ao1000 shown and shareable [31][4]; a review asks for ao250+ and best TPS per step [5] | Same averages in TIM; best TPS per Roux stage | Partly. ao5/ao12/Mo10Ao5 (`scope.md` R1 "Platform and foundations", Timer) | R1 (averages), R2+ (per-stage TPS) | Longer averages are trivial. Per-stage TPS needs SC |
| 16 | Scramble preview and scramble history [4] | Same in TIM and stage drills | **New** | R1 | Small, and useful when a scramble is misapplied |
| 17 | Keep the screen awake while scrambling (fixed after a review) [5] | Timer and drill screens | **New** (not stated) | R1 | Trivial, and a known annoyance |
| 18 | Data exports (a Premium feature); a user exported data to analyse it with ChatGPT [7][3] | Full export, and import for restore, of the attempt log and Rouxtines | Partly. Twisty/Cubic Timer export (`scope.md` R1 "Platform and foundations", Timer). Full raw export is **new** | R1 | Data lives only on the phone (`scope.md` "Out"), so users need a backup path. Keep it free (the rules at the top of `scope.md`) |
| 19 | Incognito sessions; insights and training unlock after 12 solves [4][21][30] | "Counts toward stats" switch; minimum sample size | Yes (`rouxtine-design.md §2, §5`) | R1 (already) | Already planned |
| 20 | "Plan one step further" for prediction; cross training coming [8] | INS scope ladder for FB inspection | Yes (`scope.md` "Modules in R1", INS row; `training-regimens.md` §6.5, scope ladder) | R1 (already) | Rouxtine is ahead here |
| 21 | Metronome training: 60 BPM, +20 BPM after a clean solve [8]; a community metronome plugin [20] | Optional metronome mode in slow solves | Research only (`training-regimens.md` "Lookahead"); not in scope.md | R2+ (with SLOW), optional | Coaches disagree about metronome drills, so make it optional |
| 22 | Optional WCA notation ("Rw" instead of "r") [4] | Display setting for wide moves | **New** (scope covers notation: `scope.md` R1 "Platform and foundations") | R2+ | Low value. Roux material writes r |
| 23 | Target time → "which split is holding you back", from 25M+ solves [1] | Goal in the wizard; findings against the user's own baseline | Partly (`rouxtine-design.md §5, §6`) | skip (crowd benchmark) | Needs pooled data and a server (`scope.md` "Out"). No Roux split corpus exists (`rouxtine-design.md §5`) |
| 24 | Daily scramble with a global leaderboard [1] | A date-seeded "scramble of the day", the same for every user, with no leaderboard | **New** | Later | Feasible offline. Comparison would happen on Discord. Weekly competition is already "Later" (`scope.md` "Later") |
| 25 | PvP matches, prize tournaments, accounts, cloud sync [1][4][7] | — | Out (`scope.md` "Out") | skip | Needs accounts and a server |
| 26 | Games & Plugins with an MIT SDK [18][19][23] | — | Out (the rules at the top of `scope.md`) | skip | Not Roux training. It also runs unreviewed remote code [19] |
| 27 | Solve-critique videos and pro reconstructions [25][26] | Pro-solve study | Yes. REC (`scope.md` R2+ item 2); videos linked, low priority (`scope.md` R2+ item 8) | R2+ (already) | Already planned |

## 3. Things to avoid (from reviews and listings)

- **Losing the user's data.** A 1-star App Store review (2026-10-18, v2.6.1) reports that after repeated "Solve could not be saved" errors, "all of my data, history, and solves were erased" after 5,000 solves, and support had not replied within the week [5]. For Rouxtine:
  - write each attempt to the append-only log before the UI confirms it (`rouxtine-design.md §9`);
  - never discard an attempt when a write fails;
  - offer a free export and import (idea #18).
- **UI that does not fit the phone.** A Play review says "some of the interface is clipped … no way of scrolling or resizing it to see my cross time" [3]. Add small-width and large-font Roborazzi checks to the existing screenshot set.
- **Learning that is hard to reach.** An App Store review says it takes "a little too many clicks to get into the learning mode" and "seems geared for people who have learned the whole methods" [5]. Rouxtine has a beginner path (`scope.md` R1 "Learn"). Keep each case one tap from its learn view.
- **Training behind a subscription.** Reviews ask for "a onetime payment option" [3] or call it "expensive and overrated" [28]. One reviewer notes "only practice drills are behind a paywall" [5]. Rouxtine already rules this out (the rules at the top of `scope.md`).
- **Making the user stare at the screen during drills.** A reviewer found it "a bit unusual, messing with my 3d depth perception", and asked for drills to "reflect the cube's current state" [5]. acubemy later added state sync [21]. In SC drills, show the case as the user holds it (idea #10), and keep glances short.
- **Screen sleeping mid-scramble.** Reported, and promised as a fix [5]. See idea #17.
- **Mixing stats that do not compare.** A reviewer did not want 2x2 times mixed with 3x3 times [3]. acubemy fixed this with per-event sessions in v2.5.0 [4]. Rouxtine already separates drill stats from solve stats (`rouxtine-design.md §3`).
- **Advertising-linked data.** Play Data safety lists email, device IDs and app interactions as shared or collected for "Advertising or marketing" [2]. The website runs Meta and Reddit pixels after consent [16]. Rouxtine stays opt-in, with no ad ID (`scope.md` R1 "Platform and foundations", crash reporting).
- **Numbers that don't agree.** The site says benchmarks use "3 million+ solves" [6], while the stores say "25M+" [1][4]. The FAQ says there are no iOS in-app purchases, but the App Store lists them [7][4]. Keep store text and the FAQ in sync with each release, and make no unsourced claims.

## 4. Sources

Read 2026-10-31 unless noted. Reddit returned HTTP 403 through the proxy, so [28] is a third-party snapshot. One YouTube watch page ([27]) returned a captcha, so only its title was read, via oEmbed.

1. Google Play listing (description, ratings, "What's new", developer): https://play.google.com/store/apps/details?id=com.acubemy.acubemy&hl=en&gl=US
2. Google Play Data safety: https://play.google.com/store/apps/datasafety?id=com.acubemy.acubemy&hl=en&gl=US
3. Google Play public reviews (the "See all reviews" feed for en-US, de-DE and es-ES). Cited reviews: 2026-06-23 (5★, clipped UI, wrong-OLL alert), 2026-07-06 (4★, de, sync and "Show Algorithm" request), 2026-07-11 (5★, export to ChatGPT), 2026-07-26 (5★, under-18 tournaments), 2026-09-04 (4★, es, mixed 2x2/3x3 times), 2026-09-23 (4★, one-time payment), 2026-10-13 (5★, gyro tracking). Same URL as [1]
4. App Store listing (description, version history v1.11.0–v2.7.1, in-app purchases, privacy labels): https://apps.apple.com/us/app/acubemy/id6760343507
5. App Store customer reviews (RSS for us, au and de): https://itunes.apple.com/us/rss/customerreviews/page=1/id=6760343507/sortby=mostrecent/json. Cited reviews: 2026-07-10 (4★, "too many clicks"; 5★, right OLL/PLL shown), 2026-08-16 (5★, step labels, longer averages), 2026-09-01 (5★, au, paywall and screen staring), 2026-10-18 (1★, data erased), 2026-10-25 (5★, screen sleep; 5★, de, hands-free use)
6. acubemy home page: https://acubemy.com/
7. acubemy FAQ (scrambles, turn metric, cube table, plans and prices, connection, tournament rules): https://acubemy.com/faq
8. "How to Improve at 3x3: The Three Pillars of Speed", 2026-10-13: https://acubemy.com/blog/how-to-improve-at-3x3
9. Public solve page linked from [8]: https://acubemy.com/blog/solves/146e76ae-3c7f-4fad-987c-42613a48a2ed
10. "GAN i4 Review", 2026-06-30, updated 2026-07-27: https://acubemy.com/blog/gan-i4-review
11. "GAN iCarry 4 Review", 2026-03-05: https://acubemy.com/blog/gan-icarry-4-review
12. "GAN16 ui Review", 2026-08-25: https://acubemy.com/blog/gan-16-ui-review
13. "MoYu WeiLong V11 AI Review", 2026-10-23: https://acubemy.com/blog/moyu-weilong-v11-ai-review
14. Blog index and "Welcome to the acubemy Blog", 2026-03-04: https://acubemy.com/blog, https://acubemy.com/blog/welcome-to-acubemy
15. Imprint: https://acubemy.com/imprint
16. Privacy policy: https://acubemy.com/privacy-policy
17. Terms of Service, last updated 2026-07-06, §1: https://acubemy.com/terms-of-service
18. Games & Plugins page: https://acubemy.com/plugins
19. acubemy plugin SDK README (MIT): https://github.com/acubemy/plugin-sdk
20. GitHub repository search for "acubemy": acubemy/plugin-index, acubemy/plugin-template, GustavoBitten/acubemy-metronome, wallpaperman42/Cube-Racing, TanGrumser/watercolor-turns: https://github.com/search?q=acubemy&type=repositories
21. "The way you train has changed – acubemy Devlog [v2.4.0]", 2026-10-22, description and chapters: https://www.youtube.com/watch?v=YVxB8UpGNdI
22. "acubemy - Level up your cubing" (trailer), 2026-07-22: https://www.youtube.com/watch?v=OeORHlpvHC8
23. "We built games with my cube as the controller, and so can you!", 2026-09-28: https://www.youtube.com/watch?v=W3pXcmbBRhA
24. Cubing with Alex, "This App Makes Smart Cubes Worth It", 2026-07-10, sponsored: https://www.youtube.com/watch?v=AMtBD3im6ac
25. "Solve Critique: 34.87 … Fingertricks and Pair Selection", 2026-07-09: https://www.youtube.com/watch?v=rSmUb063k7s
26. "The Dark Magic of Xuanyi's 2.51 World Record (Reconstruction)", 2026-10-06: https://www.youtube.com/watch?v=k1oy_tPlXuw
27. "We Analyzed over 25 MILLION SOLVES and Found the BEST Algorithms" (title only): https://www.youtube.com/watch?v=2V6KmFhi-Jc
28. r/Cubers thread "About Smart cubes", third-party snapshot: https://reddit.sentinel-team.org/posts/1tdt4kx/snapshots/2026-05-16T14%3A59%3A27.9736Z
29. Cuboss, "Which smart cube to buy" comparison (calls acubemy a web app; undated): https://cuboss.com/which-smart-cube-to-buy-comparison/
30. Rouxtine `docs/research/training-regimens.md`: the acubemy row in §3 and its sources [58]–[62] (home, FAQ, insights page, training page, sponsored video)
31. acubemy timer page ("Playground"; ao5–ao1000, Tournaments, Daily Scramble): https://acubemy.com/playground

## 5. Open questions / unverified

- **How deep Roux support goes.** No public Roux reconstruction, Roux insight screen or Roux training card was found. It is unknown whether acubemy splits LSE into EO, LR and 4c, or how it detects M and r without a gyro. A free account would show this. That would be ordinary use, not reverse engineering, but it means signing up and accepting their terms; it is the owner's call.
- **What "L4" and "R4" mean** in the devlog [21]. My reading is a four-quarter-turn gesture on the physical cube, backed by a review that says the app works without taking a hand off the cube [5]. Unverified.
- **"Pick up and put down times for smart cubes"** [4]: whether this times the moment the cube is lifted (from the gyro?) or the first and last move is not stated.
- **Smart timers:** no public statement about GAN Halo, QiYi timer or Stackmat support.
- **The "3 million+" vs "25M+" solves** figures disagree [6][1], and neither is backed by a source. Treat both as marketing.
- **The plan generator** ("your entire training plan is built around reaching that goal" [1]) is behind a login. Public material describes only the card queue [21].
- **Whether the app is "vibecoded"** cannot be judged from public sources. Public facts: the developer says they "let Claude build" a plugin [23], and the SDK README suggests loading "a file an AI assistant just wrote for you" [19]. That says nothing about the main app.
- **Android minSdk and offline use:** the Play page does not show the minimum Android version. A Spanish review says it "works without Internet" [3]; unverified.
- **The data-loss review** [5] is one report on v2.6.1. Whether it was fixed is not public.
