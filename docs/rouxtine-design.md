# What a Rouxtine looks like

Decided 2026-10-29 with the owner (grilling Q11–Q15 and follow-ups). Evidence and the option
analysis are in [training-regimens.md §5–6](research/training-regimens.md). What ships when is
in [scope.md](scope.md); everything here is R1 unless marked.

## 1. Words

- **Module:** one training tool, e.g. "CMLL recognition". It has one or more **modes** that share its statistics.
- **Block:** one step of a Rouxtine: a module, a mode, settings, a case **selector**, a **stop rule** and a **role**.
- **Rouxtine:** a saved, ordered list of blocks, with optional **day variants**.
- **Run:** one session of a Rouxtine (or of a Quick session). It is recorded and never edited afterwards.
- **Quick session:** a single module started on its own, outside any Rouxtine.
- **Template:** a ready-made Rouxtine shipped with the app (ours or community-contributed).
- **Finding:** a weakness the analysis is confident about.
- **Suggestion:** a proposed change to a Rouxtine with a stated reason, which the user accepts or dismisses.

## 2. Modules

Every module declares:
- `id` and `version`;
- stage(s), level tag (first solve / beginner / intermediate / advanced / niche) and an optional gate (e.g. EOLR after full CMLL);
- **skills trained** as stage × channel (recognition, execution, efficiency, planning, lookahead), which is how findings route to modules;
- modes, e.g. CMLL: recognition-only, execution-only, combined;
- inputs it accepts: touch, typed algs, timer, smart cube (R2+);
- a settings schema with defaults, its default selector and stop rule, and its per-attempt payload (§6).

**Toggles (Q11):** an app-wide toggle hides a module from the library, Quick sessions and suggestions. Rouxtines do the composing. A Rouxtine may contain a hidden module, for example an imported one; the app offers to show it and never blocks the run. Any visible module runs as a Quick session. A "counts toward stats and SRS" switch, on by default, lets a user practise without moving their numbers.

## 3. Rouxtines

A Rouxtine has:
- a name and description;
- level and goal tags;
- its **source**: template, community template, custom, generated or imported;
- a schema `version`;
- ordered **blocks**;
- optional **day variants** (A/B…);
- an estimated duration per block and in total.

A **block** has:
- a module, mode and settings;
- a **selector:** Recap, Equal/Realistic, Weakest, SRS or Ladder (§5);
- a **stop rule:** `reps(N)`, `time(T)`, `coverage` (each case once), `target(metric, threshold, window)`, `untilMiss(skips=1)` or `firstOf(...)`. The default is `firstOf(time, reps)`, so a block never runs forever;
- a **role:** warm-up, focus, coverage, timed solves or review;
- an author's `description` ("plan FB+DR, no rotations"), which is shared with the Rouxtine.

**Default shape of a template:** warm-up (about 5 min) → one or two focus blocks → timed solves (ao12, or Mo10Ao5) → optional review. Drill blocks and timed-solve blocks stay separate, and drill attempts never feed PBs or averages.

**Runs:** a run follows the blocks in order, and any block can be skipped or swapped. At the end the app shows a summary. It asks "Update Rouxtine?" only if blocks were added, removed or reordered, with two choices: Update or Keep. Tuning inside a block, such as case weights or ladder depth, carries forward silently. Each block's personal note and "how did it go" rating stay with the run, show again at the next run, and are never shared.

## 4. Today

- One **active Rouxtine** drives the Today screen. Switching takes one tap, and any Rouxtine or Quick session can still be started at any time.
- With day variants, Today picks the variant that covers what was neglected this week. Example: for a 30–60 min budget, A = drills, B = solves, following Jayden McNeill's advice.
- Today also shows the streak, weekly progress, and any pending suggestions.

## 5. Adaptation and analysis

**Inside a block (automatic, Q13):** the selector chooses cases without asking.
- **Recap:** each case once per cycle.
- **Equal / Realistic:** uniform, or by natural case probability.
- **Weakest:** bootstrap each case first, then weight towards slow or missed cases, with a floor and a cap.
- **SRS:** FSRS-6 for recall-type cases (CMLL algs, EOLR/EOLRb, 4c, NMCLL), with one cold review per case per day, due cases first.
- **Ladder:** optimal-length steps for intuitive modules (FB, FS, SS, EOLR), moving up one move at 80 % optimal over the last 10.

Cases are interleaved by default, and the app shows why each case was picked (due, slow, missed, new). After a block it offers "Drill missed" and "Drill slow".

**Across blocks (suggestions only):**
1. The analysis turns the attempt log into **findings**. It ranks seconds lost per stage and per case group, split into recognition, execution and efficiency (movecount against the solver optimum). It measures against the user's own baseline over 7, 30 or 90 days and needs a minimum sample size.
2. Reference split tables are labelled provisional, because no per-level Roux table exists.
3. Findings become **suggestions** shown as a diff with a reason. Example: "EOLR recognition is 2× slower than CMLL over 14 days: add an EOLR recognition block?"
4. The user accepts or dismisses each one. A saved Rouxtine never changes without consent.

**Dashboard:** improvement areas and strengths per stage and case group. Dedication (time, streak) sits apart from skill. The headline metric comes from timed solves only.

## 6. Generation, onboarding and templates

- **Onboarding:**
  - **Wizard:** current average, goal, minutes per day, which sets the user knows, smart cube yes/no.
  - **Short placement test**, which can be skipped: an ao5 and an FB inspection test, plus CMLL and EOLR recognition only for users who say they know them.
  - The result picks or assembles a starter Rouxtine, so experienced users skip beginner content.
- **Generated Rouxtines (Q12, Q14):** "Make me a Rouxtine" builds a **new** Rouxtine from the wizard answers and the current findings. The rules are deterministic and readable, and the app shows why each block is there. It never overwrites an existing Rouxtine.
- **Recommendations:** modules to unlock next (gates such as "full CMLL by 20–30 s", "EOLR from about 15 s"), and what to focus on now.
- **Templates in R1, by level:**
  - first solve;
  - beginner Roux (sub-60 → 30);
  - intermediate (sub-30 → 15, full CMLL);
  - advanced (sub-15, EOLR).
- **Templates in R1, by goal:**
  - CMLL push;
  - FB/inspection planning;
  - LSE efficiency;
  - 10-minute warm-up;
  - 30–60 min A/B week.
- **Community templates (Q12, Q15):**
  - Templates are JSON files in this repo under `templates/`, in the same schema as share codes.
  - Contributors add or change them by pull request. CI validates the schema, module ids and settings. A maintainer reviews and merges, and the next release bundles them.
  - No online fetch in R1.

## 7. Sharing (Q15)

- **Share code:** versioned, with a checksum, carrying structure and settings only. No stats, notes, SRS state or personal data.
- **Formats:** a text code, a QR code, or a `.rouxtine` file through the Android share sheet.
- **App links** (a tappable URL) need a hosted verification file. They could live on the free `nauroth.github.io` Pages site, since no domain will be bought. Later.
- **Import:** the app shows a preview and saves the result as a copy.

## 8. Motivation

- **Daily streak with freezes and rest days** (the owner prefers a streak). Freezes are earned and applied automatically, and planned rest days do not break the streak. There is a configurable day-rollover hour.
- **Weekly practice-days goal**, default 3 days.
- Both can be hidden in settings.
- Optional local reminders. No pressure copy, no punishment for breaks: coaches say breaks help.

## 9. Data (per attempt)

Every module writes to one append-only attempt log:
- the module, mode, case id and angle, and the scramble or state;
- recognition, execution and total time, with a monotonic clock for durations;
- mistakes, DNF and hint use, and movecount against solver optimum where relevant;
- the run and block ids, an `is_preview` flag, and a model version.

Everything above (estimates, FSRS state, findings) is derived and can be recomputed by replaying the log. Formulas and defaults: [training-regimens.md §6](research/training-regimens.md).
