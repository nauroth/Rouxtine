# What a Rouxtine looks like

Decided 2026-10-29 with the owner (grilling Q11–Q15 and follow-ups). Evidence and the option
analysis are in [training-regimens.md §5–6](research/training-regimens.md). What ships when is
in [scope.md](scope.md); everything here is R1 unless marked. The pre-code audit the same day
filled gaps the decisions left open. Those rules are marked *(default)*: they follow the
research, and the owner may override them. Numbers marked *(default)* live in one versioned
analysis config, so they can change without a data migration.

## 1. Words

- **Module:** one training tool, e.g. "CMLL recognition". It has one or more **modes** that share its statistics.
- **Block:** one step of a Rouxtine: a module, a mode, settings, a case **selector**, a **stop rule** and a **role**.
- **Rouxtine:** a saved, ordered list of blocks, with optional **day variants**.
- **Run:** one session of a Rouxtine (or of a Quick session). It is recorded and never edited afterwards.
- **Quick session:** a single module started on its own, outside any Rouxtine.
- **Template:** a ready-made Rouxtine shipped with the app (ours or community-contributed).
- **Finding:** a weakness the analysis is confident about.
- **Suggestion:** a proposed change to a Rouxtine with a stated reason, which the user accepts or dismisses.
- **Profile:** the user's app-wide answers and settings that many modules read: level, known sets, M-hand, orientation set, cube colour scheme, naming scheme.

## 2. Modules

Every module declares:
- `id` and `version` (§10);
- stage(s), level tag (first solve / beginner / intermediate / advanced / niche) and an optional gate (e.g. EOLR after full CMLL). A gate is advisory: it orders suggestions, the roadmap and the default visible set, and never hides or locks a module the user turns on. Each gate is a condition the app can check over the profile and the log, e.g. `known(cmll) == 42 && headline <= 30 s` *(default)*;
- **skills trained** as stage × channel (recognition, execution, efficiency, planning, lookahead), which is how findings route to modules;
- modes, e.g. CMLL: recognition-only, execution-only, combined;
- inputs, required and optional: touch, typed algs, timer, smart cube (R2+);
- a settings schema with defaults, its default selector and stop rule, and its per-attempt payload (§9);
- *(default)* the selectors it allows (`none` for modules without cases, such as TIM and LSX); its progression (none, a length ladder with bounds, the INS scope ladder, or lessons for BEG); the stop rules and roles it allows; typed metrics with units that `target` can use (e.g. `rec_mean_ms`, `optimal_rate`, `ao5_ms`); a `defaultSecondsPerRep` for duration estimates; and which settings come from the profile (§7).

**Shared statistics** *(default)*: the skill model is keyed by case, not by module: (case set, case, angle where relevant, channel, protocol). Every module and mode that trains a case set reads and writes the same per-case estimates, so CMX and CMR share one FSRS card per (case set, case). Recognition estimates stay separate per protocol (§9). Module ids stay the unit for the library, toggles and suggestion routing.

**Toggles (Q11):** an app-wide toggle hides a module from the library, Quick sessions and suggestions. Rouxtines do the composing. A Rouxtine may contain a hidden module, for example an imported one; the app offers to show it and never blocks the run. Any visible module runs as a Quick session. A Quick session has a "counts toward stats and SRS" switch, on by default, which lets a user practise without moving their numbers. *(default)* Blocks in a Rouxtine always count, so the switch is for Quick sessions only.

## 3. Rouxtines

A Rouxtine has:
- a name and description;
- level and goal tags;
- its **source**: template, community template, custom, generated or imported;
- a `formatVersion` and a `revision` (§10);
- ordered **blocks**;
- optional **day variants** (A/B…);
- an estimated duration per block and in total.

A **block** has:
- a module, mode and settings;
- a **selector:** Recap, Equal/Realistic, Weakest, SRS, Ladder or none (§5);
- a **stop rule:** `reps(N)`, `time(T)`, `coverage` (each case once), `target(metric, threshold, window)`, `untilMiss(skips=1)`, `done` (the module says the block is complete: lesson finished, quiz passed) or `firstOf(...)`. The default is `firstOf(time, reps)`, so a block never runs forever. Every stop rule must be bounded: `target` and `untilMiss` only inside `firstOf` with `time` or `reps`, and the builder wraps them in `time(15 min)` *(default)*. The builder, import and template CI all check this;
- a **role:** warm-up, focus, coverage, timed solves or review. *(default)* Warm-up marks its attempts `is_warmup`, which keeps them out of findings and, by default, out of the Twisty/Cubic export. Timed solves is allowed only on TIM (and SC later), and only it feeds PBs and the headline. Focus, coverage and review are labels;
- an author's `description` ("plan FB+DR, no rotations"), which is shared with the Rouxtine.

A BEG block can set `lesson = next`, which resolves to the next unfinished lesson, so the "first solve" template is written with blocks like any other.

**Default shape of a template:** warm-up (about 5 min) → one or two focus blocks → timed solves (ao12, or Mo10Ao5) → optional review. In R1 a warm-up is a Recap `coverage` block over the known CMLL cases (CMX or CMR), optionally followed by a short `time(T)` LSX block with selector `none`; the researched Roux warm-up, partial SpeedBLD FB, comes with INS in R2+. Drill blocks and timed-solve blocks stay separate, and drill attempts never feed PBs or averages.

**Runs:** a run follows the blocks in order, and any block can be skipped or swapped. At the end the app shows a summary. It asks "Update Rouxtine?" only after a structural change, with two choices: Update or Keep. *(default)* A structural change is a block added, removed, reordered or swapped, or the user editing a block's settings, selector or stop rule during the run. Follow-up blocks ("Drill missed", "Drill slow") and skips are not structural. Tuning inside a block, such as case weights or ladder depth, carries forward silently. It is derived from the attempt log and never stored in the Rouxtine, so the saved Rouxtine does not change. Each block's personal note and "how did it go" rating stay with the run, show again at the next run, and are never shared.

### Data model *(default)*

- **Rouxtine** {id (UUID), originId, revision, formatVersion, name, description, level, goals, provenance, archivedAt}. Each saved edit creates a new revision, which keeps a JSON snapshot.
- **Variant** {id, name, order}. A Rouxtine without variants has one implicit variant.
- **Block** {id (UUID, stable across revisions), variantId, order, module, moduleVersion, mode, settings, selector, stop, role, description}.
- **Run** {id, rouxtineId?, rouxtineRevision, variantId?, status, dayKey}. A Quick session is a run with no Rouxtine.
- **RunBlock** {id, runId, sourceBlockId?, blockSnapshot (JSON), kind = planned | followUp | quick, status = done | skipped | partial, note, rating}. The note comes back at the next run through `sourceBlockId`.
- **Adaptive state** (ladder level, bootstrap progress, case weights, metronome tempo) is derived from the attempt log, keyed by (module, mode, case-pool signature). The same drill in two Rouxtines shares it.
- Deleting a Rouxtine archives it; its runs keep their snapshots.

### Run lifecycle *(default)*

- The run lives in Room and is written in the same transaction as each attempt: status, current block, active time and stop-rule counters per block, and a selector snapshot (seed, draw index, Recap permutation and position, cooldown and reinsertion queues).
- After process death the screen is rebuilt from Room. The attempt in progress becomes `interrupted`: it is not a rep, it stays out of timing stats, and the app offers to redo it.
- `time(T)` counts active time only (elapsedRealtime deltas). It pauses while the run is paused or the app is in the background.
- One open Rouxtine run at a time. Quick sessions are separate runs that close on exit.
- An open run can be resumed within the same day (§8). Otherwise it closes as partial at the next app open, and its attempts count.
- Discard is explicit. It writes `exclude` amendments (§9), or deletes the attempts if the user asks.
- Before a run starts, each block's required inputs are checked. A block that needs something missing (a smart cube in R2+, or a paid add-on if one ever exists) falls back to touch or timer input where the module allows, and is skipped otherwise. It never blocks the run or an import ([training-regimens.md §5.10](research/training-regimens.md)).

## 4. Today

- One **active Rouxtine** drives the Today screen. Switching takes one tap, and any Rouxtine or Quick session can still be started at any time.
- With day variants, Today picks the variant that covers what was neglected this week. Example: for a 30–60 min budget, A = drills, B = solves, following Jayden McNeill's advice.
- Today also shows the streak, weekly progress, and any pending suggestions.

**Today rules** *(default)*:
- No active Rouxtine (onboarding skipped, or the active one deleted): Today shows "Start here" with three actions (answer the wizard, pick a template, make me a Rouxtine) and the Quick session list.
- Picking the variant: for each variant, this week's minutes on its modules, from any run or Quick session, divided by the variant's estimated minutes. The lowest ratio wins. Ties go to the variant run least recently, then to variant order.
- Once today's variant is done: "Done for today" with the run summary, "Run again" and the other variants.
- On a rest day (§8): a rest card, with the Rouxtine one tap away.

## 5. Adaptation and analysis

**Inside a block (automatic, Q13):** the selector chooses cases without asking.
- **Recap:** each case once per cycle.
- **Equal / Realistic:** uniform, or by natural case probability. Natural probabilities are precomputed per case set from the random-state generator and versioned with the set.
- **Weakest:** bootstrap each case first, then weight towards slow or missed cases, with a floor and a cap. The bootstrap counts per case × angle in recognition modes and per case in execution modes. It is derived from the log, so it spans runs, and it covers only the active pool (below).
- **SRS:** FSRS-6 for recall-type cases (CMLL algs, EOLR/EOLRb, 4c, NMCLL). A card is (case set, case), AUF-invariant. A case's one scheduled review is the first cold attempt after its due time; new cases are due at once. Only that attempt updates FSRS state; other attempts update only the per-case estimates. FSRS learning and relearning steps are off, so every interval is at least a day. Due cases go first, lowest retrievability first, capped per run (*default:* 20, or a third of the block's reps); the rest wait for the next practice days. With nothing due, SRS serves new cases through the gate (below), then falls back to Weakest. Grading follows [training-regimens.md §6.5](research/training-regimens.md): Again, Hard, Good, and Easy only by hand; recognition time counts toward Hard only when `rec_valid`. R1 uses the default FSRS-6 weights, with no optimisation on the phone, and tests against py-fsrs 6.3.2 outputs. (A "one review per calendar day" rule would break those tests: 21:00 and 08:00 the next day are a same-day review to FSRS.)
- **Ladder:** optimal-length steps for intuitive modules (FB, FS, SS, EOLR), moving up one move at 80 % optimal over the last 10 and down one under 40 % *(default)*. Each module declares its bounds (*default*, wodzik's ranges: FS 2–6, FB 3–8, SS 3–10, EOLR 3–10). The start is the last level in the log for the same module, mode and pool, else the module default. A sampler indexed by depth on the solver's pruning tables draws states of the exact length, with a retry budget. It falls back to the nearest length and logs the length served.

Cases are interleaved by default, and the app shows why each case was picked (due, slow, missed, new). After a block it offers "Drill missed" and "Drill slow".

**Selector rules** *(default)*:
- An empty pool (after filters and the active pool) means the block can't start. The builder flags it, and a run offers "Edit filter" or "Skip".
- A pool of one case repeats it, with no cooldown.
- `coverage` counts cases; the module says whether angles count too.
- `reps(N)` counts completed attempts, DNFs included. Aborted and `interrupted` attempts don't count.
- Stop rules are checked between attempts, so `time(T)` lets the current attempt finish.

**Case status and new cases** *(default)*:
- Each case has a status: not started, learning, known or leech, with the mastery rules of [training-regimens.md §6.5](research/training-regimens.md).
- A case block's active pool is known plus learning cases, plus optional look-alikes in CMR.
- New cases enter through the gate: at most 3 a day, with "Add next case now".
- Users can mark cases known in bulk. The onboarding answer "I know full CMLL" marks all 42.

**What R1 can measure** *(default)*. Until smart cubes (R2+), the app sees moves only on its own cube:

| Module | Recognition | Execution | Efficiency |
|---|---|---|---|
| CMR | Measured (app-drawn case) | – | – |
| CMX | Only with a confirmed blind setup (§9); otherwise total time only | Measured (touch timer) | – |
| EO, EOLR, L4C, LSX | Measured on the M/U pad | Measured on the pad | Measured on the pad |
| FBT, FLP, SS, SLP | – | Total time | Self-reported or typed |
| INS | – | – | Self-reported or typed; plan held or broke |
| TIM | – | Manual stage splits | – |

For block drills on a physical cube, the app asks after the solution reveal "How many moves did you use?". The answer is prefilled with the optimum and offers +1, +2 and +3 or more; typing the solution, checked by the cube model, is optional. Each attempt logs `efficiency_source` (measured, typed, self_report, or smart_cube in R2+). Self-reports are never pooled with measured values. The Ladder and the INS scope ladder run on this input in R1.

**Across blocks (suggestions only):**
1. The analysis turns the attempt log into **findings**, against the user's own baseline over 7, 30 or 90 days. In R1 there are two kinds:
   - **Drill findings** per case group: recognition (only `rec_valid` attempts, per protocol), execution, and efficiency where moves are known (measured and self-reported kept apart).
   - **Stage findings** from manual TIM splits only: each stage's share of the solve against a provisional reference, as log ratios with CubeSkills' 20 % band, adjusted for the sets the user knows (someone on 2-look CMLL is not told CMLL is slow).

   Splitting a stage's lost seconds into pauses, movecount and TPS needs the moves, so it comes with SC, SLOW and REC in R2+.
2. Reference split tables are labelled provisional, because no per-level Roux table exists.
3. Findings become **suggestions** shown as a diff with a reason. Example: "CMLL recognition on Pi cases is 2× slower than on your other CMLL cases over 30 days: add a CMR block on Pi?"
4. The user accepts or dismisses each one. A saved Rouxtine never changes without consent.

Findings route to modules through the routing table of [training-regimens.md §5.6](research/training-regimens.md), shipped as a data file *(default)*. Each cell names its module and the first release that has it. R1 fallbacks: an SB pause goes to SLP; inconsistency goes to "Drill slow" in that stage's case module; the SB→CMLL transition is shown as a finding with no block suggested.

**Cold start and thresholds** *(default)*:

| Output | Shown when |
|---|---|
| Case-group finding | ≥ 20 counted attempts in the window, ≥ 3 on every case named, and the posterior rule of §6.5 (flag when δ̂ − 0.84·sd > ln 1.2) |
| Recognition finding | ≥ 20 `rec_valid` attempts from one protocol |
| Stage finding | ≥ 12 timed solves with complete splits |
| Consistency | ≥ 8 solves |
| Plateau | The rule in §6.5 (≥ 8 practice days, ≥ 5 solves a day) |
| Headline | Mo10Ao5 from 50 timed solves; else ao12 from 12; else ao5; else "–" |

Below a threshold the dashboard shows progress toward it (e.g. "CMLL: 12 of 20 attempts"), and Today says that suggestions start after a few sessions. Generation with no findings uses the wizard answers only.

**Suggestion rules** *(default)*:
- Kinds: add a block, remove a block (skipped in the last 5 runs), narrow a pool to a weak case group, rotate the focus (plateau), try a module (gate met), template updated (§10). Tuning inside a block is never a suggestion.
- Computed after a run closes and at the first app open of a day, never during a run.
- At most 3 pending, at most 1 new a day, and 1 per finding.
- A dismissed suggestion (same kind, target and finding) stays quiet for 14 days, then until the finding's seconds lost grow by 25 % or more. "Don't suggest this again" mutes that kind for that module.
- A suggestion expires after 30 days, or once its finding has stayed below the thresholds for 7 days in a row.
- Each suggestion targets a Rouxtine id and revision. If the Rouxtine changed since, the diff is recomputed, or the suggestion is dropped if it no longer applies. Accepting creates a new revision and offers Undo.
- Hidden modules are never suggested. When the only route is a hidden module, the finding says "The module for this is hidden. Show it?".

**Generated text** *(default)*: every generated sentence (findings, suggestion reasons, "why this case", generated names, block reasons, duration estimates) is a string resource with typed placeholders (`<plurals>` for counts, formatters for durations and ratios). A reason code stored on the finding or suggestion picks it; code never concatenates sentences. One Kotlin enum lists the reason codes, and a test checks that each has a resource. Each new string goes through `humanizer` when it is added.

**Dashboard:** improvement areas and strengths per stage and case group. Dedication (time, streak) sits apart from skill. The headline metric comes from timed solves only.

## 6. Generation, onboarding and templates

- **Onboarding** (the answers go into the profile, which the user can edit later):
  - **Wizard** *(the branches are a default)*:
    - "Can you solve a cube?" No: the first-solve template, no placement test.
    - "Do you solve with Roux?" Not yet: the switcher lesson plus the beginner Roux template. No ao5, because a time from another method would misplace the user; the FB inspection test is optional.
    - Everyone: goal and minutes per day. Roux solvers also: current average and which sets they know.
    - The smart-cube question waits for R2+, when the answer drives something.
  - **Short placement test** for Roux solvers, which can be skipped: an ao5 and an FB inspection test, plus CMLL recognition only for users who say they know full CMLL. A measured ao5 beats a self-reported average.
  - The result picks or assembles a starter Rouxtine, so experienced users skip beginner content *(mapping is a default)*: slower than 30 s, or no full SB yet → beginner Roux; 15–30 s → intermediate with full CMLL, otherwise CMLL push; under 15 s → advanced with EOLR, otherwise intermediate.
- **Generated Rouxtines (Q12, Q14):** "Make me a Rouxtine" builds a **new** Rouxtine from the wizard answers and the current findings. The rules are deterministic and readable, and the app shows why each block is there. It never overwrites an existing Rouxtine. The steps *(default)*:
  1. Inputs: the profile, the budget, case statuses, visible modules, findings that pass the thresholds, and the app's release. A measured level (the latest ao12, or Mo10Ao5, once there are 12 timed solves) beats the self-reported one.
  2. Shape: the budget table in [training-regimens.md §5.3](research/training-regimens.md). 30–60 min a day gives variants A (drills) and B (solves).
  3. Warm-up: picked from a level → warm-up table.
  4. Focus: the top two findings by seconds lost, ties broken FB → SB → CMLL → LSE. Each goes through the routing table to the first visible module in this release, preferring one whose gate is met, with the pool narrowed to the finding's case group and the module's default selector. With no findings, the level template's focus blocks.
  5. Timed block: ao12 under 60 min a day, Mo10Ao5 (50 solves) from 60 min.
  6. Duration per block: the user's median seconds per rep for that module and mode over 30 days (with 20 attempts or more), else the module's `defaultSecondsPerRep`, times the reps, capped by `time(T)`. To fit the budget, focus blocks shrink down to 3 minutes; then the lowest-priority focus block goes.
  7. Each block stores its reason (finding id and reason code), which the UI shows. The result is named "Made for you, <date>", with provenance {generatorVersion, inputsHash}. The app asks "Make it active?" and never activates it on its own.
  8. No randomness: the same inputs give the same Rouxtine, covered by golden tests.
- **Recommendations:** modules to try next (gates such as "full CMLL by 20–30 s", "EOLR from about 15 s"), and what to focus on now.
- **Templates in R1, by level:**
  - first solve;
  - beginner Roux (sub-60 → 30);
  - intermediate (sub-30 → 15, full CMLL);
  - advanced (sub-15, EOLR).
- **Templates in R1, by goal:**
  - CMLL push;
  - FB/inspection planning;
  - LSE efficiency;
  - 10-minute warm-up (a Recap `coverage` block of known CMLLs, then a short `time(T)` LSX block);
  - 30–60 min A/B week.
- **Community templates (Q12, Q15)** *(the pipeline is a default)*:
  - Templates are JSON files in this repo under `templates/`, in the same schema as share codes, plus template-only metadata: `id`, `revision`, `authors` (GitHub handles, shown on the template card) and `budgetMinutes`.
  - Contributors **make** a template by pull request, or **suggest** one through a GitHub issue form, "Suggest a Rouxtine template" (share code, level, goal, notes). A maintainer or agent turns an accepted suggestion into a `templates/*.json` pull request with a `:tools:decodeShareCode` task. The issue form is the `/triage` surface; template pull requests get a review by hand. Any in-app link to the form or the repo is in the `foss` build only, because the repo page shows Ko-fi.
  - CONTRIBUTING.md states the terms and asks for a DCO sign-off, which CI checks (both pending HANDOFF "Open" question 6). The pull-request checklist covers Roux only, realistic durations, and text passed through `humanizer`.
  - One pure-Kotlin `:format` module holds the format and the module registry (ids, versions, settings serializers, case sets). The app and CI both use it. It generates `templates/schema.json` for contributors' editors, and CI checks that file is current.
  - A CI JVM test decodes every template strictly: the file name equals `id`; ids are unique; `revision` goes up whenever the content changes against `main`; module ids exist and `moduleVersion` is not newer than the module; settings decode with no unknown keys; case ids exist; selectors, stop rules and roles are allowed by their module; every stop rule is bounded; level and goal tags come from the enums; text fits the caps and the share code fits the QR budget (§7); the estimated duration is within ±20 % of `budgetMinutes`. It warns when a template overrides a profile setting (§7).
  - A Gradle task copies `templates/` into the app's assets at build time, and an app test loads every bundled template. A maintainer reviews and merges, and the next release bundles the result.
  - Template text becomes generated string resources (`template_<id>_name`, `_description`, `_block_<n>`) from the English JSON, so Weblate can translate it later. Share codes stay untranslated user text.
  - No online fetch in R1.

## 7. Sharing (Q15)

- **Share code:** versioned, with a checksum, carrying structure and settings only. No stats, notes, SRS state or personal data.
- **Formats:** a tappable link, a text code, a QR code, or a `.rouxtine` file through the Android share sheet. The text code is the canonical form; the others carry it.
- **Link:** `https://nauroth.github.io/r/#<code>`. The code sits in the URL fragment, so it never reaches a server. Android App Links are verified through `/.well-known/assetlinks.json` in the user-site repo `nauroth/nauroth.github.io`: the file must sit at the host root, and a `.nojekyll` file lets Pages serve the dot-directory. It lists both packages with their SHA-256 fingerprints: the release key (Play App Signing) for `io.github.nauroth.rouxtine` and the dev key for `.debug`. Without the app, the page shows the code, a copy button and plain install text, with no Ko-fi or donate wording, because the `play` build links to it. The same site hosts the privacy policy.
- **Wire format** *(default)*: `RXT1.` + base64url(raw DEFLATE(UTF-8 JSON)) + `.` + base64url(CRC-32 of the JSON). Settings are stored as a diff from the module's defaults. The importer finds the `RXT<n>.` token inside surrounding text and strips spaces and line breaks.
- **Caps** *(default)*: name up to 60 characters, description 280, block description 140; up to 4 variants and 12 blocks per variant. A measured 7-block A/B Rouxtine with descriptions gave a 1,124-character code, which is QR version 27 at error correction M. The QR and link budget is 2,000 characters (version 38 at M). Above it, the QR and link options are hidden with a reason, and the text code and file still work. Before building sharing, encode every bundled template and measure it.
- **Untrusted input** *(default)*: inflate with a hard output cap (64 KiB); refuse files over 256 KiB before reading; a lenient decoder range-checks every field and never crashes past the preview screen.
- **QR:** made with ZXing core (Apache-2.0) in both flavours, in byte mode, holding the text code. Scanning uses CameraX and ZXing and asks for the camera only when used. "Import from image" decodes a QR in a screenshot chosen with the photo picker, because QR codes travel as images in chats. Google's code scanner and ML Kit are closed and stay out (HANDOFF "Money").
- **File:** holds the text code. The app accepts ACTION_SEND with text/plain and checks for the `RXT` prefix. It has no `*/*` VIEW or SEND filter, which would put Rouxtine in "Open with" and the share sheet for every file type. A `.rouxtine` path filter does not match the content:// URIs that file managers hand out, so the ways in for a file are "Import from file" (ACTION_OPEN_DOCUMENT) and "Paste code".
- **Profile settings** *(default)*: settings that come from the profile (M-hand, orientation set, colour scheme, naming scheme, recognition system) are `inherit` in a block by default. Share codes encode `inherit`, never the sender's values. A block overrides one only when the drill depends on it (e.g. an x2y planning block), and the import preview shows the override.
- **Import:** the app shows a preview and saves the result as a copy. Re-imports and unknown or newer modules follow §10.

## 8. Motivation

- **Daily streak with freezes and rest days** (the owner prefers a streak). Freezes are earned and applied automatically, and planned rest days do not break the streak. There is a configurable day-rollover hour.
- **Weekly practice-days goal**, default 3 days.
- Both can be hidden in settings. They are still computed while hidden.
- Optional local reminders. No pressure copy, no punishment for breaks: coaches say breaks help.

**How the two fit** *(default; the owner to confirm, HANDOFF "Open")*: flexible rest days. A missed day counts as a rest day as long as the week can still reach its goal, so meeting the weekly goal never breaks the streak; with the default goal of 3, four days a week are free. Only a missed day that makes the goal unreachable uses a freeze, and the streak breaks only when no freeze is left. Fixed rest weekdays are an option for people who plan their week. Freezes: one earned for each week the goal is met, at most 2 banked. The streak then counts days on plan, and the two numbers agree.

**Day rules** *(default)*:
- A day is the local date of (attempt start − rollover hour), using the time-zone offset of that attempt.
- A practice day has at least one completed block, 5 minutes of attempts, or a finished lesson. Quick sessions count, preview ones included, because dedication is not skill.
- A day skipped or repeated only by a time-zone change doesn't break the streak.
- A new rollover hour or weekly goal applies from the next day or week. The week starts on the locale's first day, which is also a setting.
- When a day closes (worked out at the next app open), the app writes a day-ledger row {day, practiced | rest | frozen | missed, goal, freezesAfter}. The streak reads the ledger, and closed days are never recomputed, so later edits, deletions or a new rest-day plan don't rewrite past streaks.
- Reminders are off by default. Turning them on asks for the notification permission (Android 13+). They use inexact WorkManager or AlarmManager scheduling, with no exact-alarm permission, and are rescheduled after a reboot, a time-zone change or a clock change. No reminder on a rest day or once the user has practised.

## 9. Data (per attempt)

Every module writes to one append-only attempt log. The envelope and payloads in [training-regimens.md §6.2–6.3](research/training-regimens.md) are the spec, with these additions *(default)*:
- ids: `attempt_id` (UUIDv7, so ids sort by time), `run_id`, `rouxtine_id`, `block_id`;
- the module, `module_version`, mode, case set, case id and angle, orientation set and M-hand, and the scramble or state with its generator and seed;
- `input_device` (M/U pad, touch timer, typed, smart cube later);
- recognition, execution and total time;
- `rec_valid`, `protocol` (onscreen, blindSetup or none) and `is_setup_blind`;
- `outcome`: ok, mistake_corrected, wrong_case, wrong_alg, alg_looked_up, DNF, +2, abort or interrupted. A DNF is an outcome, never a time;
- mistakes and hint use; movecount against the solver optimum, with `efficiency_source` (§5);
- `started_utc_ms`, `tz_offset_min`, `zone_id`, and the day computed at write time (§8);
- `index_in_run`, `is_warmup`, `is_preview`;
- `is_scheduled_review`, with the FSRS state before and after;
- `app_version` and `model_version`.

**Timing rules:**
- Time within an attempt in one monotonic base: `MotionEvent.getEventTime()` and Choreographer frame times (the uptime/`nanoTime` base). Use `elapsedRealtimeNanos` only for spans that may cross deep sleep, and UTC plus the offset only for day bucketing. Never subtract stamps from different bases.
- "Case shown" is the start of the first frame that draws the case.
- Recognition stats come only from `rec_valid` attempts: cases the app draws (CMR, the LSE pad) or a confirmed blind setup. They are kept per protocol, never pooled. In R1, CMX on a physical cube records total time only, unless the user follows the blind-setup protocol ([training-regimens.md §6.1](research/training-regimens.md)).

**Source of truth and caches** *(default)*:
- The source of truth is: attempts; `attempt_amendments` {attemptId, kind = penalty | delete | exclude | restore, value, at}; runs and run blocks; Rouxtines and their revisions; suggestion decisions; the day ledger (§8); the profile.
- Everything else (estimates, FSRS state, findings, PBs) is a cache tagged with `model_version`. It can be dropped and rebuilt by replaying the log. Every reader applies the amendments, so +2, DNF, deleting an accidental stop and discarding a run all work on an append-only log.
- A hard delete, when the user asks, removes the rows and rebuilds the affected keys.
- A `model_version` bump, or a change to the analysis config, rebuilds the caches in a background worker, key by key, with progress shown.
- Room: `exportSchema = true`, schemas committed, and one MigrationTestHelper test per version. Destructive migration only for cache tables.

Formulas and defaults: [training-regimens.md §6](research/training-regimens.md).

## 10. Ids and versions *(default)*

- **Module ids** are frozen lowercase slugs (`cmx`, `cmr`, `eolr`…), never reused. Display names come from string resources.
- **`moduleVersion`** is the module's settings-schema version. Each bump ships a pure `migrate(from, settingsJson)` with a unit test. Attempts keep the version they were logged with.
- **Case-set ids and case ids** are frozen at the first release. Renames go through the alias maps only ([scope.md](scope.md), canonical case ids).
- **Rouxtines** carry `formatVersion` (the payload schema) and `revision` (a content counter). The payload also carries the minimum app version.
- **`originId`** (a UUID) is set when a Rouxtine is created, and copies, exports and imports keep it. Importing a payload whose `originId` matches a saved Rouxtine asks "Update the earlier copy or save a new one?".
- **Templates** carry `templateId` and `revision`. Picking a template saves a copy with provenance {templateId, revision}. When a later release bundles a higher revision, the app raises a "Template updated" suggestion that shows the diff; it never applies it on its own.
- **Import rules:**
  - a `formatVersion` above the supported one is refused with "Update the app to open this";
  - an unknown module, or a `moduleVersion` newer than the installed one, becomes a disabled placeholder that keeps its raw JSON: shown in the preview, skipped in runs, checked again after an app update;
  - an older `moduleVersion` is migrated;
  - unknown settings are dropped and listed in the preview;
  - a retired module becomes a placeholder that names its replacement.

## 11. Accessibility *(default)*

Part of every module's definition of done:
- A cube colour-scheme setting with presets, including high-contrast and colour-blind-friendly ones, and optional glyphs per sticker colour on the Canvas cube. Recognition depends on telling red from orange and green from blue.
- Case images carry no case identity in their content description until the user has answered, so TalkBack doesn't give the answer away.
- Next to the whole-screen hold-to-start, the timer has a focusable start/stop control with a custom accessibility action, and the space bar works on a keyboard.
- Cube turn animations and shape morphs are skipped when animations are removed (animator duration scale 0); check what Compose already does.
- M/U pad targets are at least 48 dp.
- Optional haptic or voice cues at 8 s and 12 s of inspection.
- Every chart has a text summary.
- Roborazzi baselines at font scale 2.0 next to light and dark; Roborazzi's accessibility-check module in CI; `accessibility-review` on each new screen.
