# Rouxtine feature scope

Decided 2026-10-29 with the owner, from the research in `docs/research/`. This file is the
source of truth for what ships when. It supersedes the priorities in
[roux-training-features.md §2](research/roux-training-features.md) and the delta in
[roux-community-resources.md §9](research/roux-community-resources.md); those notes keep the
evidence and the code to reuse. How Rouxtines work is in [rouxtine-design.md](rouxtine-design.md).

- **R1** = first public release (Google Play + GitHub APK).
- **R2+** = following releases, roughly in the order listed.
- **Later** = wanted, not planned yet.
- **Out** = never in the app.

Rules that apply to every item:
- Roux and everything that applies to Roux, nothing else. Niche Roux techniques (NMB, CPFB, Pinkie Pie, TCMLL, ACMLL…) count as Roux. Other methods, including Roux-like ones with their own steps (CEOR, Nautilus, 42…), are out: no lessons, no sets, no glossary entries.
- No paywalls, ads or in-app purchases. Our own text everywhere. Third-party content only with a licence or written consent (see [roux-community-resources.md §10](research/roux-community-resources.md) item 7).
- Every trainer is a **module** (see the design doc) from day one.

## R1: first release

### Platform and foundations

- Cube model and notation: M/r/S/E/f, wide moves, rotations, centres tracked. Port `CubieCube` / `Move` / `MoveSeq`, fixing the parser that drops unknown tokens.
- Random-state stage scrambles (masked random state → scramble).
- Canvas cube, 2D and 3D, with per-stage sticker masks and turn animation.
- Touch move input: an M/M'/U/U' pad for every LSE module; typed alg entry validated by the cube model.
- Canonical case ids with alias maps (CMLL group+slot, EO groups, SBLS 65-case taxonomy). Naming-scheme setting, U-sticker + pattern recognition by default.
- M-hand setting (right by default; mirrors LSE solutions). It is not asked during onboarding.
- Timer: touch timer, sessions, ao5/ao12/Mo10Ao5, manual per-stage splits. Twisty/Cubic Timer export.
- Attempt log (append-only) and the skill model that every module feeds ([rouxtine-design.md](rouxtine-design.md) §5, §9).
- Settings, light/dark theme, Material 3 Expressive, English strings in resources.
- About / Licences screen with the GPL notices, source location as plain text, and MIT notices.
- Two build flavours: `play` (no donation link anywhere) and `full` (Ko-fi link; GitHub APK).

### The Rouxtine engine (owner: all of it in R1)

- Module library with app-wide visibility toggles, plus Quick sessions for any visible module.
- Rouxtine builder: ordered blocks (module, mode, settings, case selector, stop rule, role).
- Guided runs; immutable run records; "Update Rouxtine?" only after structural edits.
- One active Rouxtine drives the **Today** screen, with day variants (e.g. drills day / solves day).
- Bundled templates by level and goal, including community templates merged via GitHub pull requests.
- Onboarding wizard plus a short placement test that picks or assembles a starter Rouxtine.
- Case selectors inside blocks: Recap, Equal/Realistic, Weakest, SRS (FSRS-6) and Ladder.
- Practice-data analysis: weaknesses per stage and case (recognition, execution, efficiency), shown on a dashboard.
- Suggestions: modules to add and things to focus on, shown as a diff with a reason. The user accepts or dismisses them, and saved Rouxtines never change silently.
- Generated Rouxtines: "make me a Rouxtine" from wizard answers and practice data. The result is always a new Rouxtine.
- Sharing: a text code, QR code or file, carrying structure only.
- Motivation: daily streak with freezes and rest days, weekly practice-days goal (default 3), optional reminders. Dedication stats are kept apart from skill stats.

### Learn

- Beginner course from zero, in our own words, with mini-drills: notation and move quiz, FB, SB, 2-look CMLL, beginner LSE.
- "If you can already solve the cube" lesson for switchers. No CFOP content.
- Glossary with our own definitions, linked from jargon everywhere.
- Roadmap screen: a level ladder (first solve → beginner → intermediate → advanced → niche) linking each technique to its module or lesson. Our own wording; rouxcubing's table is used only as evidence.

### Modules in R1

Codes are from [roux-community-resources.md §4.1](research/roux-community-resources.md).

| Code | Module | R1 contents |
|---|---|---|
| BEG | Beginner course | Lessons and mini-drills above |
| FBT | FB trainer | Random-state FB/FS/line, solution reveal ranked for ergonomics, optimal-length ladder |
| INS | Inspection planner | Timed inspection, scope ladder (DL+centre+pair → FS → FB → FB+DR → FB+SS), compare with the best across x2y or full CN |
| FLP | FB last pair (+DR) | Position filters |
| SS | SB square | Front/back, DR filters, DR-first |
| SLP | SB last pair | 65-case taxonomy, front and back slot, reference |
| CMX | CMLL cases | 2-look and full 42, the user's main alg per case, random AUF, optional SB-LP pre-moves |
| CMR | CMLL recognition | Tap-to-name, per-(case, angle) stats, confusables queue |
| EO | EO (4a) | Misoriented centres, flowchart mode, LR-edge control |
| EOLR | EOLR / EOLRb | Non-MC / MC / both / where-optimal filters |
| L4C | 4b / 4c | Recognition method (FUBU/BU/DFDB), DFDB prediction during 4b |
| LSX | LSE sandbox | Full LSE free-solve, efficiency %, QTM |
| TIM | Timed solves | Timer module, so solves can be a Rouxtine block |

## R2+: next releases

1. **Smart cubes** (the owner is buying a QiYi AI and a GAN i4):
   - protocol core with QiYi, then GAN Gen4, then Gen2 / MoYu AI 2023, then MoYu32, then GoCube;
   - a smart-cube solves module (SC) with automatic Roux splits (FB/SB/CMLL/LSE, plus finer EO/LR/4c where detectable) and cube-clock timing;
   - smart-cube input in every case module.

   Plan: [smart-cubes.md](research/smart-cubes.md). R2 if hardware testing goes well.
2. **Slow solves (SLOW)** with movecount per stage against solver par. **Reconstruction study (REC):** import (cubedb, alg.cubing.net, Twizzle), automatic stage split, plan-then-reveal pro-solve study.
3. **Benchmark Rouxtine** (monthly re-test) and benchmark cards (movecount per stage, split balance).
4. **More modules:**
   - Tracking (TRK); chain flow FS → FBLP → SS → SBLP (CHN); prediction before the last pair or before CMLL (PRD);
   - FS / pseudo-FS / line opening quiz; partial SpeedBLD warm-up mode in INS; pair spotting;
   - FBLP+SS and FS+DR combined trainers; "find 2+ solutions" and DR tiers.
5. **CMLL:**
   - NMCLL / non-matching blocks; alternative algs and per-case execution timing; 1.5-look path;
   - peeking and hidden-sticker modes; trigger drills; NMCMLL/ATCRM recognition systems;
   - CMLL→LSE effect on alg cards.
6. **LSE:** two-step EOLR recognition stepper; EO-pattern weights; LSE sprint; EZ4c; EOLRb 4c influencing.
7. **Technique library:** adapted sections of the Roux Reader (MIT, with notice).
8. **Learn-more links** to videos and sheets, with link-health checks. The owner rates this low priority.
9. **Translations** through Weblate, once R1 strings settle.
10. **Move-set profile** (2H / no-F) for solver ranking.

## Later

- One-handed: OH hand, OH alg sets, mirroring, OH move-set ranking (owner: future, not a priority).
- Big-cube Roux pack: 4x4 split timer, parity CMLL and parity 4c (owner: maybe in future).
- Niche CMLL: Pinkie Pie, TCMLL, ACMLL. Straughan and three-corner recognition. Advanced 4c influencing.
- Weekly-competition mode: needs Knut Peterson's agreement and a scramble feed.
- Community card with outbound links (Discord, subreddit): owner says maybe.
- Lore and timeline cards.
- Online refresh of the community template library from GitHub. R1 bundles templates in each release.
- F-Droid or IzzyOnDroid listing. GitHub releases already cover Obtainium users.

## Out

- Any non-Roux method, step, alg set or scramble type, including glossary entries for other methods.
- Mindset and quote content.
- Paywalls, ads, in-app purchases, accounts or a server of our own.
- Copying text, images or alg sheets from sources without a licence or consent.
