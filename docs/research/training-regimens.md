# Training regimens: prior art and a design space for modules and Rouxtines

> Section 5 was decided with the owner on 2026-10-29. The outcome is [rouxtine-design.md](../rouxtine-design.md), which wins where the two differ: daily streak plus weekly goal, generated Rouxtines in R1, share formats including an https link. Offline-first with no backend or accounts is now in HANDOFF. The "Open item 5/6" references below point to an older HANDOFF. Those items were settled in docs/scope.md and docs/rouxtine-design.md and are not today's numbered questions. HANDOFF "Don't" limits csTimer, wodzik/cube and FsrsKt to ideas only: port py-fsrs (MIT).

Researched 2026-10-28 from coach blogs and videos (titles, descriptions and chapters only), trainer source code and shipped bundles, app-store listings and reviews, help centres, and papers. Every claim was checked by a second pass; where that pass corrected a claim, the corrected version is used. `[n]` refers to the numbered sources at the bottom. Section 5 is design, not fact: the options are proposals for the owner, who has not decided them yet (HANDOFF "Open" item 6 [165]).

Vocabulary used below (proposed, not yet in `CONTEXT.md`):
- **Module**: one training tool, e.g. "CMLL recognition". It can have several **modes**.
- **Block**: one step of a Rouxtine: a module, its settings, a case **selector** and a **stop rule**.
- **Rouxtine**: a saved, ordered list of blocks, plus an optional schedule.
- **Run**: one session of a Rouxtine. It is recorded and never edited afterwards.
- **Benchmark**: a fixed Rouxtine that measures rather than trains.

## 1. Summary

- **Coaches do not publish a fixed regimen.** Feliks Zemdegs says to spend at least 50 % of practice on timed solves once you are comfortable with your method, more on deliberate practice when fixing fundamentals, and a growing share of timed solves as you get faster. Feliks prefers loose schedules to strict ones [1]. Jayden McNeill decides the mix by daily time budget: with 30–60 min a day, alternate a solves day and a drills day [4]. The only Roux-specific warm-up advice found is partial SpeedBLD [9]. No source found describes a cool-down.
- **No per-level Roux split table was found.** The two references are Scheopner's idealised model (FB/SB/CMLL/LSE = 17/27/23/33 %) [27] and two measured world-class Cubeast ao100s (about 12–14/33–36/21–23/30–31 %) [38][39]. Recognition is about 21–23 % of total solve time and about half of CMLL time in those ao100s, counting AUF as recognition [38][39][42]. CubeSkills' CFOP tool flags a step more than 20 % above typical [36][37]. A Roux equivalent has to be built.
- **Among the tools we could inspect, none builds a cross-module Roux plan.** Roux trainers are single-module [55][56][57]. SCT diagnoses Roux stages automatically from smart-cube solves (seconds lost per stage, with a cause and an action) but builds no plan [164]. Plan features are CFOP-only (CuberPal [63]) or could not be inspected (acubemy's plan generator needs a login [61]). The upstream roux-trainers picks cases uniformly at random [55].
- **Case selection prior art is rich and copyable as ideas:** natural, equal and recap orders (csTimer, jperm, Roman's trainers) [43][48][49]; slowest-first bags with a floor (Cubeast 8:1, SpeedcubeQuest's weighted sum with a 0.12 floor) [42][45]; optimal-length ladders (wodzik/cube: +1 move at 80 % optimal over the last 10) [56]; and auto-graded SRS (Cubedex, since 2026-04-21, already with CMLL and EOLR sets) [44]. That last point contradicts `roux-training-features.md` line 75 [166].
- **Other domains agree on structure.** A routine is an ordered list of units, each with its own repetition count or stop rule (KovaaK's, Aimlabs, Strong, Hevy) [67][72][98][103]. Templates are separate from session records, and Hevy asks before folding only *structural* edits back into the template [106]. Routines are shared as codes, links or files [67][73][125]. Weakness dashboards need minimum sample sizes (Lichess) [120].
- **The loudest complaints in other domains are things Rouxtine has already ruled out or can avoid:** paywalled analysis and upsells (Aimlabs, KovaaK's) [78][79], caps on user routines (Strong: 3, Hevy: 4) [98][103], silent plan regeneration (Fitbod) [109][110], forced beginner paths (Yousician) [97], and review backlogs (Anki, Chessable) [113][126]. Rouxtine has no paywall by decision [165].
- **Algorithms:** use FSRS-6 for recall-type cases, with one scheduled review per case: the first cold attempt after it falls due (6.5). Port py-fsrs (MIT) to Kotlin, or use FsrsKt (pure Kotlin, GPL-3.0) [131][140]. FSRS-7 is released only in ts-fsrs 6.0.0 (2026-10-28) and is not settled [128][129]. For in-session weighting, use log-time EMAs per case and channel, shrunk toward the set mean [143][153].
- **Licences:** Rouxtine is GPL-3.0-only [165]. GPL-3.0 code (csTimer, wodzik/cube, FsrsKt) is licence-compatible. AGPL code (keybr, lila, Anki) could be combined under GPLv3 §13, with those parts staying AGPL, but HANDOFF already rules out TNoodle (AGPL) without permission [165], so treat AGPL sources as ideas-only unless the owner decides otherwise. SCT has no licence and stays ideas-only [164]. This is not legal advice. The formulas are small, so reimplementing them is simplest.
- **Recommended shape (section 5):** typed, versioned modules with app-wide visibility toggles. A Rouxtine is an ordered list of blocks with stop rules and an optional loose weekly rotation. Ship starter templates chosen by a placement benchmark, plus a free builder. Adapt automatically *inside* a block; propose changes *across* blocks as a diff the user accepts. Rank weaknesses by seconds lost per stage and by cause (pause, turning speed, movecount). Share self-contained codes with no server. Use a weekly practice-day goal with automatic freezes, not a daily streak.

## 2. How cubers and Roux coaches structure practice

### Deliberate practice versus timed solves

- Feliks Zemdegs splits practice into deliberate practice and subconscious practice (timed solves and averages) [1]. Deliberate practice includes learning and drilling algs, learning new methods, experimenting, studying example solves for inefficiencies, focusing on one aspect, new fingertricks, and "any sort of drill or practice that isn't complete solves" [1].
- Feliks' ratio: if you are satisfied with your techniques, spend at least 50 % of practice on timed solves. If you want to improve fundamentally, spend much more on deliberate practice. As you get faster, more of your practice should be timed solves. At least 50 % of Feliks' own practice is timed solves [1].
- Jayden McNeill sets the mix by daily time budget [4]:
  - **2 h or more a day:** at least 30 min of intentional practice on most days. Jayden's examples are unlimited inspection to plan more of the first pair, and slow solves where you force yourself not to look at what you are solving. Spend the rest on ao50/100/200 sessions or more focused practice, and do an average of 50–100 solves regularly.
  - **30–60 min a day:** don't do solves and intentional practice on the same day. Do solves one day and only intentional practice the next.
  - **Under 1 h a day,** you must make a careful schedule and stick to it. **Under 2 h a day,** you must make sacrifices or accept a serious disadvantage. **With more than 1 h a day,** you can skip intentional practice for days if you apply ideas strictly in your solving sessions.
  - The post is CFOP-oriented and dated only "May 11".
- A retailer blog (SpeedCubeShop, January 2026) suggests 20–30 min of focused practice a day, deciding the focus in advance, one step per session, 10–15 min on a single case, and tracking averages rather than singles. It says lookahead should become a focus only once recognition and execution are easy [40]. Treat it as low-weight advice.

### Schedules

- Feliks has "never followed a strict practice regimen" [1]. Feliks finds loose schedules work better: plan which events to cover over a timeframe so nothing is neglected, and before a small competition practise only its events for a few weeks [1].
- Feliks also says to practise only when you feel like it, because unfocused practice is less effective. Feliks quotes Stefan Huber: "Breaks make you better", and often hears of people setting a PB average after a week without cubing. Separately: after long stagnant periods, times often drop 1–2 s within days; stagnating is "completely natural" [1]. People who enjoy tracking numbers may still want a schedule [1].
- CubeSkills has a 7-lesson "Practicing" module (Intermediate) with lessons including "Components vs Full Solves", "Practicing Inspection" and "Practice Schedule Thoughts". Its stated goal is "to transform what you've consciously learned into a subconscious action" [3]. The videos need an account and were not watched.

### Session shape

No coach source gives a step-by-step session template. What exists:
- **Warm-up.** The Roux Reader calls Kian's partial SpeedBLD "arguably underrated as a practice approach" and says to do it "regularly as warmup before solves" [9][18]. One warm-up video gives no content in its description [22]. The other is a hand-warming routine for cold hands, not a cubing drill [23].
- **Cool-down.** None found. Aimlabs, in aim training, tells users to order tasks "to warm up, peak, and cool down during a session" [72].
- **Practice unit.** Kian proposes Mo10Ao5 (mean of 10 averages of 5) as both a practice unit and a measure of speed [15]. Kian also has videos on practising effectively and on TPS, with a stated goal of 8 TPS in solves [16][17].
- **Analysis, then a plan.** Kian's "Why You're Not Sub-X With Roux" series has five videos: Sub-20, Sub-15, Sub-12 and Sub-10 (2016), and Sub-8 (2017-02-08) [12][14]. The Sub-20 to Sub-10 descriptions combine level tips, a slow-solve demonstration and a critique of a named viewer's reconstructed solve (Isaac Vega, Thibaud Ou, Iuri Grangeiro, Waffo and Ryan Peh) [12]. Sub-20 has no chapters, so its slow-solve demonstration could not be checked [12]. Sub-8, "the fifth and final video", critiques a solve by Kavin Tangtartharakul (GuRoux) [14]. Sub-15 has 50+ machine-style chapters, including "Solving splits and targets", "Target times for each step", "Slow solving practice", "Deliberate practice", "Identifying pauses" and "Transition analysis" [13]. Sub-12 chapters include Move Count, Inspection, Ugly Cases, CMLL, LSE, Slow Solve and Reconstruction [12]. The target numbers could not be read.
- **A Roux solver's weekly plan.** John Mark Grimes (13.6 s average, full CMLL with all-angle recognition, full EOLR) explains a weekly training plan built by analysing their own solves. The analysis found time losses in CMLL, LSE and second block [33].
- Charlie Eggins' practice guide, linked from Roux Resources, has chapters "Fixing your fundamentals", "Analyzing your solves", "Targeted practice techniques" and "Learn and grind strategy" [19][20].
- J Perm's "HOW I PRACTICE EFFECTIVELY & IMPROVE FASTER" (2020-04-01) has a description ending "*April Fools!", attached to the final solve's reconstruction. Whether the whole video is a joke is unclear, so it is not used [24]. J Perm's other practice videos say "It's important to vary your practice!" [25], and J Perm's Roux video has a "Practice + Slow Solves" chapter [26].

### What to practise, by level

- **CubeSkills brackets (CFOP-framed)** [2]:
  - Beginner (over 60 s): fingertricks, drilling algorithms, colour neutrality.
  - Intermediate (about 15–60 s): slow untimed solves to "boost your lookahead and efficiency"; reduce F2L movecount.
  - Advanced (under about 15 s): "Constantly assess your weaknesses"; learn extra alg sets; watch fast solvers' example solves. Focused practice is what turns tutorials into improvement.
- **Roux level gates.** Scheopner: novice to intermediate "once you are sub-30 or so", advanced "once you are sub-20 or so" [27]. Roux Reader: learn full CMLL as soon as possible, at the latest at about 20–30 s. Learn EOLR only after full CMLL, from about 15 s [10].
- **rouxcubing "Rouxdmap" technique table** [11]:

| Level | Techniques |
|---|---|
| Basic | Square/line blockbuilding, inspect full FB, basic SB algs, CMLL algs and execution, EO flowchart |
| Intermediate | Inspect FB+DR, SBLP, multi-inserts, multi-angle CMLL recognition, EOLR, DFDB, misoriented centers, dots skipping |
| Advanced | Inspect FB+SS, nonlinear blockbuilding, SS blockbuilding, CMLLEO, EOLRb, 4c influencing, ULUR/UFUB neutrality |
| Niche | Inspect F2B, CPFB, nonmatching centers, non-matching blocks, SBLP+CO (after CPFB), NMCMLL/42, Pinkie Pie, ACMLL/TCMLL, Nautilus L5E |

  Its video roadmap is Kian's Sub-X series, and its text roadmap says "Coming soon!" [11].
- **FB drills (Roux Reader)** [9]:
  - Random last-pair cases in trainers, checked against reference solutions.
  - Untimed solves trying different ways to solve the same LP case.
  - Example solves and reconstructions ("Trainers can only help you so far").
  - Plan FB fingertricks in inspection; plan FS first if a full FB is out of reach.
  - Zhouheng's tracking trainer, and partial SpeedBLD as a warm-up.
  - Choose FB solutions by your own ergonomics rather than movecount, as this is "more precise and correlated to actual times".
- **Inspection scope grows step by step.**
  - Scheopner [28]: a novice locates the DL edge, the L center and one pair, using more than 15 s while learning. An intermediate locates all FB pieces, plans the first square or line, and tracks the rest. An advanced solver fully plans FB and locates and tracks at least one edge, usually DR. Very advanced solvers inspect and track an SB square.
  - acubemy: practise without the 15 s limit, always planning one step further, and add the next step once it fits in 15 s. "This is how Xuanyi Geng and Yiheng Wang built their prediction skills" [8].
  - Grimes planned a week around Xuanyi Geng's methods. By Tuesday Grimes could not plan F2B blind and dropped to FB+SS, "which was still probably too hard" [34]. Antoine Cantin's video on Xuanyi's 4+ pair planning has a "Strategic daily training" chapter [35].
- **Lookahead: coaches disagree** [6][7][8]:
  - CubeSkills' framework has three levels: Spotting, Tracking, Knowing. Its drills: plan pair 1, find pair 2 before turning, track it while executing; solve pairs blindfolded; visualise short sequences; "two look F2L"; predict the next step during the last pair. Do these drills in isolation; in timed solves, track only as much as doesn't slow you down [7].
  - Metronome drills appear in CubeSkills [7] and acubemy (start at 60 BPM, add 20 BPM once a full solve has no missed beats, one beat of grace on recognition steps) [8]. csTimer has a metronome tool [43].
  - Jayden McNeill treats poor lookahead mostly as a symptom of inefficient, unstandardised solutions, and says "screw metronome practice". Jayden grants that slow turning for a day or two can help in the right context [6].
  - All three are CFOP/F2L-framed. Roux Resources links Dylan Miller's lookahead guide as "for CFOP, but the same principles can apply to Roux" [19][21].
- **Alg choice.** Time each alg as a small average (avg5, mo3, or three reps in a row), not as a best single, which drifts toward "high risk, low reward algorithms" [5].

### Weakness metrics coaches use

1. **Stage splits against typical splits at your level** (CubeSkills CFOP tool [36]); Grimes' plan from their own time losses [33].
2. **Movecount against level targets or the optimum** (Scheopner [28][29]); Kian's "Move Count" chapter [12].
3. **Pauses, especially at transitions** (Kian's "Identifying pauses" and "Transition analysis" [13]); acubemy's "fluency", the share of solve time spent turning [8].
4. **Recognition versus execution per step.** csTimer measures recognition as first move minus step start [43], and Cubedex as first move minus "case shown" [44]. Cubeast counts AUF turns as part of recognition [42], so its numbers are not directly comparable.
5. **TPS while turning,** with execution timed per alg as small averages [5][8].
6. **Fingertricks, regrips and lockups,** found by reviewing solves [9]. The Roux Reader warns that FB's moveset is "intrinsically more difficult to fingertrick" and that rushing causes "bad lockups" [9]. No inspected tool measures lockups; smart-cube tools see only time gaps and turn counts. CuberPal's video feedback claims to spot pauses and rotations, for CFOP only [63].

acubemy's "Three Pillars" article frames time as movecount ÷ TPS, with fluency on top [8]. In its 16.53 s example (63 turns, 46 % fluency), removing pauses would save about 8.9 s, world-record TPS (14) about 3.1 s, and an average CFOP movecount about 0.7 s. The article does this by hand for one solve and tells readers to do it for theirs; it does not say the app computes it [8]. It advises lowering movecount "before you grind thousands of solves", and splitting practice into "small, targeted modules" that are each measurable [8].

### Reference splits and movecounts

- **CubeSkills CFOP Solve Splits Tool** [36][37]:
  - Typical Cross/F2L/OLL/PLL splits for averages from 60 s to 8 s, interpolated linearly. Examples: 60 s = 7/35/8/10, 30 s = 4/15.5/4.5/6, 20 s = 2.8/10.2/3.0/4.0, 10 s = 1.2/5/1.65/2.15.
  - Under 20 % deviation is "not very significant". Only the single largest positive deviation over 20 % is flagged as "something to focus on". The data are "partially based on sub 10 solver data" from a speedsolving thread.
  - Feliks warns that splits depend on what you know: two 20 s solvers might split 4-11-2-3 and 2-9-4-5. Feliks says the better question is which step can be improved most easily, and to reflect on your solves before gathering data. Feliks points to csTimer's multi-phase timer for collecting splits [36].
- **Roux splits:**

| Source | FB | SB | CMLL | LSE | Notes |
|---|---|---|---|---|---|
| Scheopner "Idealized Roux Step Breakdown" [27] | 17 % | 27 % | 23 % | 33 % | A model: STM-optimal step solvers, top-solver reconstructions, and "heavily" Kian's calculation from a 2018 seminar [32]. Achievable turns 5.5/12.75/10.75/13 at 9/13/13/11 TPS = 3.60 s. Assumes no pauses. Written July 2025. The author offers the solve fractions as per-step targets "scalable to your current skill level" [27]; the measured ao100s below differ (FB 12–14 % vs 17 %) [38][39]. |
| eff, Cubeast ao100, 5.58 s [38] | 13.7 % | 32.8 % | 23.2 % | 30.3 % | Measured (about 2025). Turns 7.3/16.4/10.7/14.2; 48.5 turns at 8.6 TPS. Recognition: SB 0.21 s, CMLL 0.61 s, LSE 0.37 s. Inspection used 6.1 s. |
| Fahmi, Cubeast ao100, 6.46 s [39] | 12.3 % | 35.5 % | 20.9 % | 31.3 % | Measured (about 2021). Turns 6.6/15.8/10.1/13.0. Recognition: SB 0.23 s, CMLL 0.69 s, LSE 0.52 s; none shown for FB. Inspection used 11.6 s. |
| SCT "intermediate (~20-30s)" profile [164] | 20 % | 30 % | 22 % | EO 12 + LR 9 + 4c 7 | No source given in the code; the repo has no licence. |

  Percentages for the ao100s are computed from Cubeast's step times. Both ao100s are linked from Roux Resources [19].
- **What the comparison shows.** Real FB share (12–14 %) is below the model's 17 %, and SB (33–36 %) above 27 % [27][38][39]. With recognition removed, eff's FB share is 17.5 % (0.77 of 4.41 s of execution), matching the model, so the FB gap is the recognition and pauses of the other stages, which the model leaves out by design [27][38]. SB stays at about 37 % of execution because real SB uses more turns (16.4 vs 12.75) at lower TPS (10.0 vs 13) [27][38]. This rests on two solvers only, so reference shares should come from measured data per level once it exists.
- **Recognition.** Total recognition is 21.3 % (eff, 1.19 s) and 22.3 % (Fahmi, the 1.44 s listed above; Cubeast's own total is 1.46 s, or 22.6 %) of solve time, and 47–51 % of CMLL time. Cubeast counts AUF turns as recognition [38][39][42].
- **No per-level Roux split table was found.** Kian's Sub-15 video has a "Target times for each step" chapter, but transcripts were not readable [13]. The rouxcubing text roadmap is "Coming soon!" [11]. This is a negative search result.
- **Movecount targets (STM, Scheopner)** [28][29][30][31]:

| Stage | Novice | Intermediate | Advanced | Optimal average |
|---|---|---|---|---|
| FB | < 13 | < 10 | < 9 | 6.21 fixed orientation, 5.19 x2y, 4.74 full CN |
| SB | < 22 | < 19 | < 16 | 9.96; about 13 "at speed" is an ultimate goal |
| CMLL | – | – | – | 2-look 20.47, 1-look 10.75 |
| LSE | – | – | – | 11.13 (median 11, 95th percentile 14) |

  Work on colour neutrality once FB averages under 9 moves [28].

## 3. Prior art

### Cubing tools

| Product | Unit of practice | Routine composition | Stopping rules | Templates / custom / auto | Weakness feedback | Sharing | What to borrow |
|---|---|---|---|---|---|---|---|
| Cubeast + Academy (web, closed) [41][42] | Exercise: "Algorithm drill" or "Intuitive solving" | User-built exercises. Drill options: case set, case order, mirror, grey out stickers, hint after, pause, hide after first turn. Intuitive: min/max depth, sample states, show optimal length, hide after first turn (on by default) | Rounds are counted, not set | Custom | Case order random (least-practised first) or prioritise time / recognition / execution: 3 successes per case×AUF first, then the i-th slowest of n (i = 0 for the slowest) gets ceil(8·(n−i)/n) copies in a shuffled bag; "The slowest case is 8 times more likely to be shown than fastest case". Steps tab: recognition, execution, turns, TPS per stage. Filter syntax. Case recognition is Premium | Public share links for solve sets [38] | Stats per case×AUF; prioritising recognition or execution; hide after first turn. Rank by mean, not by accumulated sums, which the client appears to use |
| csTimer (web, GPL-3.0) [43] | Solve or training scramble | Sessions; Roux training scrambles (2nd Block, CMLL, LSE, `LSE <M, U>`) | None | Custom | Manual multi-phase timer (1–10 phases). Smart cube: Roux reconstruction fb/sb/cmll/l6e with insp/exec/turns/TPS per step. "Training Stat." (count, best, mean) cannot identify CMLL | Export file; each record holds the penalty and every phase's end time [43] | Probability modes Actual / Equal / Random order (each case once per cycle) |
| Cubedex (web, MIT) [44] | Case rep (smart cube or timer) | Selected cases; CMLL (42), L6E-EO (11), L6E-EOLR (46) | Time Attack: every selected case once | Custom | Smart Order: auto-graded FSRS-like SRS. Prioritise failed cases. "Slow Cases First" sorts by best time. Cases auto-become learned | Not checked | Automatic grading thresholds; recognition from "case shown" to first move, discarded above 15 s |
| SpeedcubeQuest (web, closed) [45][46][47] | Case rep: Flashcard, Scramble or Virtual Cube | Filter by status (Not known, Learning, Getting there, Muscle Memory, Failing) | Open-ended | Custom | Weight = 0.12 + Σ slider × factor over status, freshness, accuracy, speed, "seen too few times". Roux step breakdown in the Solve Analyzer | Not checked | Tunable weights with a floor; timer sessions marked "partial practice" skip records |
| jperm.net (web) [48] | Case rep (OLL, PLL, COLL, ZBLL, VLS, WV…; no CMLL) | All, Groups, Status, or N slowest by average | Open-ended | Custom | "Balanced" (equal, drawn without replacement) or "Realistic". Optionally include "Trash" cases | – | Include look-alike unlearned cases "to avoid mixing them up" |
| wodzik/cube (web, GPL-3.0) [56] | Stage scramble with a known optimal length: FS 2–6, FB 3–8, FB+DR 2–7, SS 3–10, SB last slot 2–9, EOLR 3–10, LSE 3–12 (CMLL has no length range) | Per-step trainers | Time Attack; Ladder | Custom | "Your moves / optimal", hint = first optimal move. Ladder: +1 move when ≥ 80 % of the last 10 at this length were optimal. Recognize mode times the first turn. Scramble mode times recognition from the last scramble move, after the user has seen the case, and its stats are kept apart from Recognize mode: "Scramble and Recognize apart: their times don't compare". Per-case stats split solves, drills and recognition. Speedsolves are split into FB/SB/CMLL/LSE with per-stage recognition and execution, detected through the MPL-2.0 cubecore library [168] | – | The ladder; separate in-solve and drill stats; separate stats per recognition protocol. Code is licence-compatible |
| Cubegrass block trainer (web) [57] | Block scramble by "Level" (solution length): FB 1–9, SB 1–14, SB square 1–10, SB LP 3–9 | Multi-select levels | Open-ended | Custom | Solution on demand | – | Length as difficulty |
| onionhoney roux-trainers (upstream) [55] | Stage case | Per trainer | Open-ended | Custom | None: uniform random over selected cases | – | Baseline to replace |
| Roman's trainers (OLL, ZBLL; no licence) [49][50] | Case rep; rouxcubing lists the ZBLL trainer and Tao Yu's Alg-Trainer as usable for CMLL [51] | Selected cases | "Recap": each case once | Custom | ZBLL: 20 % of picks take the least-practised case, the rest are uniform | – | Recap mode |
| acubemy (iOS, Android, web) [58][59][60][61][62] | Smart-cube solve, drill | Marketed as "builds a training plan"; the training page (login only) "suggests new algorithms to learn based on your skill" [61]. Not inspected. A sponsored July 2026 video thanks it for "Roux support" [62]. Its blog says cross and metronome training tools are coming [8] | ? | Auto (claimed) | Insights for CFOP, Roux and ZZ after 12 solves per method. Post-solve alerts on time you could have saved, with a drill for the better alg. "Personalized Training" (Premium, €4.99/month or €49.99/year): SRS for OLL, PLL and F2L only | – | Roux insights exist; plan quality unknown |
| SCT, speedcube-trainer (web, Roux-first, also CFOP; no licence) [164] | Smart-cube solve (speed or slow mode); alg drill | None: separate timer, drill and training pages, no plan | – | – | "What to work on": ranks steps by seconds lost per solve, mean total × (step share − reference share), only when the share is more than 0.04 above the reference. One cause per step, checked in order: pause ratio > reference + 0.08 = recognition/lookahead; moves > 1.25 × reference = efficiency; TPS < 3.2 = execution. Each cause comes with an action (drill the algs, slow solves without pausing, replay solves for shorter solutions, TPS drills). Separately flags only one "least consistent step": the largest sd among steps with n ≥ 8, sd > 0.55 × mean and mean > 1.5 s | Not checked | Ideas only (no licence). Borrow the cause order and the one-worst-only consistency rule. Avoid raw share differences, the unsourced reference profile and the fixed 3.2 TPS cut-off |
| CuberPal (iOS, CFOP) [63] | Solve, Focus Sprint, SRS drill | "Tell CuberPal your current average and your goal, and follow a personalized training plan" | Focus Sprint = a short set of solves | Auto | Weekly recap; forecast of your average; manual SRS grading (OLL, PLL, COLL, ZBLL, F2L); video feedback per CFOP phase. Claims smart-cube and smart-timer support; streaks and reminders | Imports csTimer and Twisty Timer | Goal-and-average wizard; weekly recap; short focus sprints |
| Cube Gym (iOS) [64] | Case drill; named, resumable sessions | Learning/Learned marks "to build a focused curriculum"; full CMLL | – | Custom | Separate recognition-ID and execution drills; data stays on device | – | Separate recognition and execution modes |

Nathan's Cube Trainer (iOS) has "weak-case tracking and review" and a CFOP segmented timer, with no Roux mention [65]. Jade's EOLRb and LSE (LaSEr) trainers are linked from Roux Resources [19][66].

### Other domains

| Product | Unit of practice | Routine composition | Stopping rules | Templates / custom / auto | Weakness feedback | Sharing | What to borrow |
|---|---|---|---|---|---|---|---|
| KovaaK's (aim) [67][68][69][70] | Scenario | Ordered playlist; per-scenario play count. 388,182 playlists | A play count per scenario (1 for 27 of the 36 entries in the 3 most-subscribed playlists) | Custom; users subscribe to others' playlists (the top three have 79k–98k subscribers) | Browser shows aim-type mix (e.g. 34/33/33 %) and author before subscribing; "Benchmark Tracker" | Word share code (e.g. `KovaaKsBouncingSilverBinding` for Voltaic Novice S5) resolved by the server; the code differs from the name, which confused at least one user | Show a routine's composition before import; never let the code and the name diverge |
| Voltaic benchmarks [71] | Benchmark scenario | Novice / Intermediate / Advanced; Clicking, Tracking, Switching, each with subcategories | – | Template | Rank per level | Shipped as ordinary KovaaK's playlists | Benchmarks are just routines |
| Aimlabs [72][73][74][75][76][77] | Task | Ordered playlist "to warm up, peak, and cool down"; reps per task; shorter focused playlists recommended | Reps | Custom; Steam Workshop; Adaptive Tasks and Progression are paid (console listing) | Benchmarks: rank is "a floor set by your least developed" subcategory; train the lowest, re-test periodically. Season 2 is live | Deeplink with tasks, settings and order; needs a link handler and a download | Rank as the floor of the weakest skill |
| keybr (typing) [80][81][82] | Generated lesson | Guided adaptive mode is one lesson type among word lists, books, custom text, numbers, code | Daily goal 30 min (0–120) | Auto inside the lesson | Per-key confidence = target time ÷ smoothed time. By default the included key with the lowest best-ever confidence still below target is in every word; with "recover keys", the lowest current confidence. A new letter unlocks once every included letter has reached target speed *at least once*; "recover keys" makes it strict. Uses dictionary words, padding with pseudo-words | – | Confidence ratio; lenient unlock with a strict option. Some users still got stuck |
| Monkeytype (typing) [83][84] | Test: time, words, quote, custom, zen | – | Time or word count | Custom | "Practice words…": missed, slow or both, as a one-tap follow-up. Opt-in "weakspot" mode | Settings URL; warns above 2000 characters | One-tap "drill missed / drill slow" after a block |
| Modacity (music) [85][86][87] | Playlist item | Practice playlists "or make it up as you go" | Per-item time | Custom | Guided loop: pick a problem, choose a strategy, record, "did you improve?". Notes shown next time. Statistics and unlimited Deliberate Practice Mode are Premium ($12.99/month) | – | Notes carried into the next run |
| Melodics (music) [88][89][90][91][92][93] | Lesson | Weekly Goals matched to the current Stage, refreshed Monday | 15 Stages gated by three readiness dials; uncapped lesson scores | Curated + auto weekly | XP is "a dedication rating", not skill | – | Keep dedication and skill apart; Auto BPM +10 until natural tempo |
| Yousician (music) [94][95][96][97] | Song or exercise | Daily Session; learning path; Workouts for missing stars | Session goal in stars or minutes | Path + practice mode | Auto-adjust speed (25–125 % in 5 % steps, piano) | – | "Move on, then cycle back"; avoid forcing advanced users through beginner levels |
| Strong (gym) [98][99][100][101][102] | Exercise set | Workout Template; folders (iPhone, per 2022 docs) | Sets | Custom; 3 templates free | – | OS share sheet; recipient needs the app | Template vs session record; avoid its four-way update prompt and the cap |
| Hevy (gym) [103][104][105][106][107] | Set | Routines in folders; library of 26 programs | Sets | Custom (4 free); library; Trainer (Pro) generates a program "using an algorithm" | Progressive-overload advice; progress report | Link to hevy.com; folder link or image | Ask "Update Routine?" only for structural changes; image card |
| Fitbod (gym) [108][109][110] | Exercise | Auto-generated every time from goal, equipment, split, duration and history | Duration | Auto | Muscle recovery 0–100 % heat map; periodic Max Effort Days | – | "Recommend More, Less, or Exclude" steering; *not* silent regeneration |
| Chessable [111][112][113][114][115] | Move in a variation | Course | Review queue | – | SRS per move over 8 levels (about 4 h to 6 months); a mistake resets the timer. Chessable 3.0 (May 2023) added puzzles from your Chess.com mistakes and the PRO perk "Track difficult moves" | – | A daily review timebox; reviews before new material |
| Aimchess [116][117][118][119] | Puzzles from your own mistakes | Weekly personalised study plan | One daily session (free) | Auto | Six aspects compared "to others with the same rating", phrased as concrete percentages | – | Concrete phrasing; avoid undirected lists and noisy stats |
| Lichess puzzles [120] | Puzzle | Theme, opening, "From my games", colour | Streak: one wrong move ends the run | Custom | Dashboard over 1–90 days: improvement areas (≥ 3 failures, below your global level) and strengths, ignoring themes under 1/40 of the window. "N to replay". One difficulty setting (rating −600 to +600) carries across themes | Each theme has a URL | Minimum samples, a chosen window, "train these" + replay |
| Anki + FSRS [121][122][123][124][125][127] | Card | Decks with shared presets; filtered decks from a search | Daily new (20) and review (200) caps | Custom + filtered | Leech at 8 lapses | `.apkg`; sharing strips scheduling, marked and leech tags | Quick filtered sessions; leech handling; strip personal data when sharing |

## 4. Transferable patterns and anti-patterns

### Patterns

| Pattern | Seen in | Use in Rouxtine |
|---|---|---|
| Ordered blocks, each with its own stop rule | KovaaK's, Aimlabs, Strong, Hevy, Modacity [67][72][98][103][86] | Rouxtine = ordered blocks; stop rules: reps, time, coverage, target, until-miss, first-of |
| Template vs immutable session record; consent only for structural edits | Hevy (Update / Keep, structural only) [106]; Strong's four options draw complaints [100][102] | Ask "Update Rouxtine?" only when blocks were added, removed or reordered |
| Pluggable case selection | csTimer [43], jperm [48], Cubeast [42], Cubedex [44], Roman [49][50], SpeedcubeQuest [45] | A `CaseSelector` per block: Recap, Equal, Realistic, Weakest, SRS, Ladder |
| Bootstrap, then weight, with a floor | Cubeast (3 successes, then up to 8:1) [42]; SpeedcubeQuest (0.12 floor, "seen too few times") [45] | Weakest selector: 3 successes per case×angle first, then bounded weights |
| Recognition and execution timed separately | Cubeast [42], csTimer [43], Cubedex [44], wodzik [56], Cube Gym [64] | Every case module logs `rec_ms` and `exec_ms`, with a `rec_valid` flag (6.1); recognition-only and execution-only modes share case stats |
| Stats per case and angle | Cubeast keys by case and AUF [42] | Per (case, AUF/angle) for CMLL, EOLR, 4c |
| Automatic SRS grading from timing and mistakes | Cubedex [44]; CuberPal grades by hand [63] | Auto-grade from logged outcomes; keep a manual override |
| Case status ladder | jperm [48], SpeedcubeQuest [46], Cubedex [44], Cube Gym [64] | One status model for all case modules |
| Gated introduction of new items | keybr [81], Anki new-card cap [121], Melodics Stages [90] | Add the next CMLL/EOLR case when the active set is ready; daily cap; manual "add next now" |
| Optimal-length ladder | wodzik (80 % of last 10) [56], Cubegrass levels [57], Cubeast depth [42] | Adaptive depth for FB, FS, FB+DR, SS, SBLP, EOLR |
| Coverage runs | Roman recap [49], csTimer Random order [43], Cubedex and wodzik Time Attack [44][56] | "Coverage" block: e.g. all 42 CMLL once, timed |
| Assessment separate from practice | Aimlabs, Voltaic [75][71], Melodics Stages [90], Fitbod Max Effort Days [108], Aimchess reports [116]. Not universal: keybr and Anki adapt continuously, and Lichess is a rolling window [81][124][120] | A "Roux benchmark" Rouxtine, suggested monthly |
| Improvement areas with minimum samples and a window | Lichess [120]; Aimchess phrases findings as percentages [116] | Hide a finding until enough attempts exist; 7/30/90-day windows; "Train this" button |
| Suggest, never silently rewrite | Fitbod regenerates silently and draws "So random. No logic." [109][110]. Several apps adapt automatically: Aimchess adaptive tactics, Hevy Trainer, Aimlabs+ Adaptive Tasks, Melodics Weekly Goals [117][107][77][92] | Automatic inside a block; cross-block changes shown as a diff with a reason |
| Difficulty matching and ramps | Lichess rating ± delta, widening when the pool is thin [120]; Melodics Auto BPM [88]; Yousician auto speed [96] | Ramps for inspection time, scramble depth, metronome TPS |
| Drills kept out of records | SpeedcubeQuest "partial practice" [47] | Only "Timed solves" blocks feed PBs and averages |
| Dedication separate from skill | Melodics [89] | Practice time and streaks in one place, skill in another |
| Forgiving consistency | Melodics weekly streak + auto freezes [93]; Yousician weeks in a row [94]; Monkeytype day-boundary offset [84]; keybr minutes goal [81] | Weekly practice-day goal with freezes and a configurable rollover hour |
| Notes carried forward | Modacity [85][86] | Personal per-run block note and rating, shown at the next run, never shared (5.3, 5.10) |
| Starter library, fork and edit | Hevy library [105], Strong samples [99], KovaaK's/Voltaic [68][71] | Bundled starter Rouxtines, forkable |
| Quick filtered sessions | Anki filtered decks and Custom Study [124], Lichess themes [120], Monkeytype custom [84] | "Quick session": module + filter + limit + order, with a "counts toward SRS and stats" toggle (Anki's optional rescheduling [124]) |
| Self-contained share payload without personal data | Aimlabs deeplink (tasks, settings, order) [73], Monkeytype URL [84], Anki `.apkg` without scheduling [125] | Versioned text code + link + QR + file; structure only |
| Goal and time-budget wizard | CuberPal [63]; Jayden McNeill's budget rules [4] | Ask average, goal, minutes per day, smart cube; pick and tune a template |
| Averages as the progress unit | Kian's Mo10Ao5 [15]; small averages for alg timing [5] | Headline Mo10Ao5, ao50 secondary (5.7); rank cases by recent average, never best single |
| Loose rotation, not a calendar | Feliks [1] | Weekly coverage; skippable, swappable blocks |
| Technique roadmap with gates | Rouxdmap [11], Roux Reader FAQ [10], Scheopner [27][28] | Tag modules by level; suggest the next module when a gate is met |

### Anti-patterns

- **Uniform random as the main case order.** Tao Yu's Alg-Trainer, Twisty Timer's trainer, CubeDesk and the upstream roux-trainers all pick uniformly [52][53][54][55]. Alg-Trainer also has "Go in order", but no shuffled recap or weighting ([related-projects.md §4](related-projects.md)). Reps go to known cases, and nothing says what to drill.
- **Weighting weak cases with no floor or bootstrap,** so known cases starve. Cubeast keeps at least one copy per case after a bootstrap; SpeedcubeQuest has a 0.12 floor [42][45].
- **Ranking cases by best single,** which favours high-risk algs [5]. Cubedex's "Slow Cases First" sorts by best time [44].
- **Ranking by accumulated sums.** Cubeast's client tracker appears to rank by running sums per case, not means, so frequently seen cases would look slower and be shown even more [42]. Use means or EMAs.
- **Absolute split targets that ignore what the solver knows** [36]. Using Scheopner's fractions as per-level targets, as Scheopner's page suggests, conflicts with measured shares; this is our inference from [27][38][39].
- **Strict, forced calendars** [1].
- **Paywalled analysis and upsells inside a trainer.** Aimlabs: "Every screen has 9 ads for premium service" (270 helpful) [78]. KovaaK's: "make people pay for your trainer and then have a subscription inside it too" (464 helpful) [79].
- **Bloated navigation around a simple loop:** "getting lost navigating through this hell of a UI" [78].
- **Silent plan regeneration** that discards edits [109][110].
- **Rigid linear gating.** Yousician: "even if you put advanced it still makes you do like 30 minutes of beginner levels" [97]. Another reviewer praises being able to skip lessons, so it is partly possible [97]. keybr: a user stuck on R and T wanted to quit [82].
- **Snowballing review backlogs.** Anki: 806 reviews after 4 missed days, about 1,700 after two years [126]. Chessable: "you will wake up to 300 reviews" [113].
- **Confusing template prompts.** Strong's four options; a 2026 review calls them "3 illogical 'finish workout' options" [100][102].
- **An undirected library:** "a list of lists with no particular order… Needs direction" (Aimchess) [118].
- **Statistics shown without sample-size rules:** "All of my statistics fluctuate and don't seem to mean much" (Aimchess) [118]. Lichess needs minimum counts [120].
- **Caps on user routines:** Strong 3 free templates, Hevy 4 [98][103]. Some reviewers accept the cap [102].
- **Online dependency for owned content.** Chessable users report courses that would not open offline; Offline Mode is itself a PRO feature, so these are paying users [115].
- **Share identifiers that differ from what users see** (KovaaK's code vs name) [69].
- **Practice that only counts in one sanctioned way:** Yousician "won't record your time practiced unless you're actually following along" [97].
- **Counting drills in full-solve PBs** [47].
- **Per-case recognition stats that hide a weak angle.** Cubeast is the counter-example: it keys stats by case×AUF to avoid this [42].
- **CFOP-only planning presented as general** (acubemy's personalised training covers OLL/PLL/F2L; CuberPal coaches Cross/F2L/OLL/PLL) [59][63].
- **Citing April Fools or marketing as evidence:** J Perm's 2020-04-01 video [24]; acubemy's "3 million solves" benchmark claim [58]; Aimchess's "31% faster" claim, whose study was not found [116].

## 5. Design space for Rouxtine

Each question lists options and one recommendation. Recommendations follow from sections 2–4, from decisions already in HANDOFF (no paywall, GPL-3.0-only, smart cubes supported, no domain [165]), and from offline-first (owner's brief, not yet recorded in HANDOFF). "No server or accounts" is an assumption that follows from offline-first; it is listed for confirmation in section 7.

### 5.1 What a module is, and its settings

**Options.**
- A. A module is a screen with free-form settings.
- B. A module is a typed, versioned unit with a declared contract.
- C. Like B, and one module can expose several modes that share one statistics store.

**Recommendation: C.** Each module declares:
- `id`, `version`, stage(s), level tag (Rouxdmap Basic / Intermediate / Advanced / Niche [11]) and an optional gate (Roux Reader: full CMLL by 20–30 s, EOLR from about 15 s [10]; CN once FB is under 9 moves [28]).
- **Skills trained,** as stage × channel: recognition, execution, efficiency (movecount), planning, lookahead. Weakness detection routes to modules through this field (5.6).
- **Modes,** e.g. CMLL: recognition-only (tap the case name), execution-only (case named, start when ready), combined, smart-cube. Modes share per-case stats. SpeedcubeQuest offers Flashcard, Scramble and Virtual Cube modes in one CMLL trainer [46]; Cube Gym separates recognition-ID and execution drills [64].
- **Inputs it accepts:** touch, stackmat/GAN/QiYi timer, smart cube, keyboard.
- **Settings schema** with defaults: case set and filters, orientation/CN, AUF randomisation, inspection limit, hints (reveal after N s), metronome, mirror. Cubeast's split of options by exercise kind is a good model [42].
- **Default selector and stop rule** (5.3, 5.5) and the per-attempt payload it logs (section 6).

Proposed starting catalogue (from the feature map [166] and the coaches above):

| Module | Skills | Modes / notes |
|---|---|---|
| FB inspection planner | FB planning, efficiency | Scope ladder DL+centre+pair → FS/line → FB → FB+DR → FB+SS [28]; time limit unlimited → 30 s → 15 s, moved up and down by a scope ladder (6.5); optional hide-after-first-turn on a smart cube [42]; optional deadline score when a limit is shown (6.5) [144] |
| Block trainers: FS, FB, FB+DR, SS, SB | Efficiency, execution | Random or optimal-length ladder [56]; compare with solver solutions [9] |
| SB last-pair trainer | SB recognition, efficiency | Per LP case; weakest selector |
| Tracking trainer | Lookahead | Zhouheng-style piece tracking [9] |
| Partial SpeedBLD FB | Planning, lookahead | Warm-up role by default [9][18] |
| CMLL recognition | CMLL recognition | Tap-to-name, first-turn timing, any angle; stats per angle. Setting "include unlearned look-alikes", as jperm suggests, so learned and unlearned cases are not confused [48]. Optional deadline score in timed mode (6.5) [144] |
| CMLL execution | CMLL execution | Small averages per alg (mo3) [5]; alg variants tracked separately |
| NMCLL / CMLL prediction | SB→CMLL recognition | Advanced/Niche [11] |
| EO / EOLR / EOLRb | LSE recognition, efficiency | Gate: after full CMLL, about 15 s [10] |
| 4c | LSE recognition, execution | Advanced [11] |
| Timed full solves | All | Touch timer, BLE timers, manual splits |
| Smart-cube solves | All, with automatic splits | FB/SB/CMLL/LSE at minimum; finer EO/LR/4c where detectable |
| Slow solves | Efficiency, lookahead (all stages) | Untimed or metronome-paced full solves [2][9]; optional no-pause rule (SCT's "slow solves without pausing" [164]) or Jayden McNeill's rule of not looking at the pieces being solved [4]; per-stage movecount against the solver optimum, automatic on a smart cube, otherwise from the entered solution; never feeds PBs or averages (5.7). Intermediate (CubeSkills [2]); also Kian's "Slow solving practice" chapter [13] and the feature map's v1 slow-solve mode [166] |
| Reconstruction review | Analysis | Worst solve of the run; compare each stage with the solver optimum |
| Coverage run | Any case set | Every case once, timed [49][44] |

Studying top solvers' example solves and reconstructions (Roux Reader [9][10], CubeSkills advanced [2]) has no module here. It is content rather than a drill, so it is deferred to HANDOFF Open item 5 (curriculum and content scope) [165]. Reconstruction review covers only the user's own solves.

### 5.2 App-wide module toggles vs per-Rouxtine composition

**Options.**
- A. App-wide toggles only: the home screen lists enabled modules.
- B. Composition only: modules exist only inside Rouxtines.
- C. Both, with different jobs.

**Recommendation: C.**
- An **app-wide toggle** controls visibility: the library, Quick session and suggestions. Users hide what does not apply ("no smart cube", "not learning EOLRb yet"). Hidden modules are never suggested. This keeps navigation lean, which is the Aimlabs complaint [78].
- **Composition** lives in Rouxtines. A Rouxtine may reference a hidden module (for example an imported one). On import or run, offer to show it; never block.
- Any visible module can be opened on its own as a **Quick session**: module + filter + limit + order, like Anki filtered decks [124]. A Quick session can be saved as a block.
- A Quick session has a **"Counts toward SRS and stats"** toggle, on by default. Anki filtered decks reschedule by default, and turning that off gives a preview mode [124]. With it off, attempts are still logged (flagged `is_preview`, 6.2) but update neither FSRS state, case estimates, findings nor the Twisty/Cubic export.
- The default visible set comes from the placement benchmark or the level the user picks (5.4).

### 5.3 Rouxtine structure: blocks, stopping rules, schedule

**Options for structure.**
- A. A flat list of modules.
- B. Ordered blocks, each with a stop rule and a role.
- C. B plus a loose weekly rotation.
- D. A periodised programme (weeks and phases).

**Recommendation: C.**
- **Block** = {module, mode, settings, selector, stop rule, role, `description`}. `description` is the author's instruction ("plan FB+DR, no rotations") and is shared with the Rouxtine. The personal per-run note and rating live on the run's block record (`run_block.note`), are shown at the next run and are never shared or exported (5.8, 5.10). Roles: warm-up, focus, coverage, timed solves, review. Show the estimated duration of each block and of the run before starting.
- **Keep drill blocks and timed-solve blocks apart.** CubeSkills has its lookahead drills done in isolation, and in timed solves only as much tracking as doesn't slow you down [7].
- **Stop rules:** `reps(N)`, `time(T)`, `coverage` (each selected case once), `target(metric, threshold, window)` (e.g. CMLL recognition mean under 0.8 s over the last 20), `untilMiss(skips = 1)` (the block ends at the first mistake, with one skip allowed; Lichess Puzzle Streak ends the run on one wrong move and allows one skip per run [120]), and `firstOf(...)`. Default to `firstOf(time, reps)` so a block never runs on forever; `untilMiss` is always wrapped in `firstOf(untilMiss, time)`. Fixed repetitions per unit are what KovaaK's and Aimlabs use [67][72]; mastery gates come from keybr and Melodics [81][90].
- **Schedule:** optional "practice days per week" and optional day variants. Jayden McNeill's short-budget rule gives two variants (A = drills, B = solves) [4]. "Today's Rouxtine" picks the variant that covers what was neglected this week. Every block can be skipped or swapped. This follows Feliks' loose schedules [1].
- **Several Rouxtines.** The builder allows any number (5.4), and each can have its own schedule and day variants. Options:
  - A. One active Rouxtine drives "Today"; the others can be started at any time.
  - B. Several Rouxtines placed in a weekly plan, e.g. a CMLL Rouxtine on two days and a solves Rouxtine on the others.
  - C. No active Rouxtine: "Today" suggests one from coverage across all runs.
  - **Recommended: A** (*proposed*). Day variants already give one Rouxtine a weekly rotation, so B would add a second calendar layer to what is meant to be a loose schedule [1]. Switching the active Rouxtine takes one tap. Any run of any Rouxtine, and any Quick session, counts toward the weekly goal (5.9) and toward the coverage "Today" uses to pick the neglected variant.
- **Runs** are immutable records. At the end, ask "Update Rouxtine?" only if blocks were added, removed or reordered. Settings tuned during a run (case weights, ladder depth, metronome tempo) carry forward silently. Offer two choices, Update or Keep, as Hevy does [106], not Strong's four [100].
- Default template (proposal):
  1. Warm-up, 5 min: partial SpeedBLD FB [9], or a coverage run of known CMLLs.
  2. One or two focus blocks chosen from diagnosis (5.6).
  3. Optional slow solves, about 5 min: untimed or metronome-paced, with per-stage movecount against the solver optimum (5.1) [2][13]. They are focus work and never feed the headline (5.7).
  4. Timed solves: 12 solves (ao12), or 50 solves for a full Mo10Ao5 [15]. Either feeds the headline (5.7).
  5. Optional review: reconstruct the worst solve and compare each stage with the solver optimum.
  The focus/solves split starts drill-heavy while the method is being learned, and moves to at least 50 % solves once it is complete [1]. The wizard's minutes per day (5.4) picks the shape:

| Minutes per day | Template shape | Basis |
|---|---|---|
| Under 30 | One focus block per day | Proposal; cf. SpeedCubeShop's 20–30 min on one step [40] |
| 30–60 | A/B day variants: intentional practice only one day (A), timed solves the next (B) | Jayden McNeill [4] |
| 60–120 | One daily Rouxtine: a focus block plus an ao50 timed block. Focus blocks may be skipped for some days if the solves apply their ideas | Proposal; the skip rule is Jayden's for more than 1 h a day [4] |
| 120 or more | At least 30 min of focus blocks on most days (Jayden's examples: unlimited-inspection planning and slow solves); the rest timed solves (ao50–ao200 sessions) or more focus work, with an ao50–ao100 regularly | Jayden McNeill [4] |

### 5.4 Templates vs builder vs auto-generated

**Options.**
- A. Templates only.
- B. Builder only.
- C. Fully auto-generated (Fitbod, Hevy Trainer, CuberPal) [108][107][63].
- D. Templates + builder (fork and edit) + rule-based suggestions.
- E. D, plus an opt-in "Auto Rouxtine" later.

**Recommendation: D now, E later.**
- **Starter Rouxtines** keyed to the learning path: beginner FB/SB, 2-look → full CMLL, "full CMLL by 20–30 s", "EOLR from about 15 s", CN warm-up, smart-cube review [10][11]. All forkable, offline.
- **Placement** picks the starter, so experienced users skip beginner content [97]. Options:
  - A. The full benchmark (5.7). It is long, and beginners cannot do its CMLL recognition and EOLR parts, which brings back the forced path Yousician users complain about [97].
  - B. Wizard answers only (current average, known sets).
  - C. Wizard answers plus an adaptive subset: an ao5 and the FB inspection test, with the CMLL recognition and EOLR parts only if the user says they know those sets.
  - **Recommended: C.** Keep the full benchmark for the monthly re-test. Placement can always be skipped by picking a starter by hand.
- **Wizard:** current average, goal, minutes per day, smart cube yes/no, which then tunes the starter. CuberPal asks for average and goal [63]; minutes per day picks a row of the budget table in 5.3, built on Jayden's rules [4]. The rules are deterministic and readable. Hevy states its programs are "generated using an algorithm and do not rely on AI" [107].
- **Builder** with no limit on the number of Rouxtines [98][103].
- **Auto Rouxtine** (E) only as an explicit opt-in that never overwrites a saved Rouxtine [109].

### 5.5 Adaptation inside a block vs suggestions at the Rouxtine level

**Options.**
- A. None: uniform random (the upstream today) [55].
- B. Adaptation inside blocks only.
- C. Automatic inside blocks, plus cross-block suggestions the user accepts.
- D. Fully automatic rewriting (Fitbod) [108][109].

**Recommendation: C.**
- **Inside a block** the selector adapts without asking. Selectors:
  - `Recap`: each case once per cycle [49][43].
  - `Equal` / `Realistic`: uniform, or natural case probability [43][48].
  - `Weakest`: bootstrap, then bounded weights (6.5) [42][45].
  - `SRS`: due cases first, then `Weakest` (6.5).
  - `Ladder`: for intuitive steps (6.5) [56].
- Interleave cases by default. In Shea and Morgan's motor-task study, random practice gave worse performance during acquisition than blocked practice, but better retention after 10 min and 10 days when tested under changed context, and better transfer [150]. Allow blocked reps only for a brand-new case's first few attempts.
- After a block, offer one-tap "Drill missed" and "Drill slow", like Monkeytype's "Practice words…" [84].
- Show why a case was picked: due, slow, missed, new or not seen.
- **Across blocks,** propose changes as a visible diff with a reason, e.g. "EOLR recognition 2× slower than CMLL over 14 days: add an EOLR block?". The user accepts or dismisses. A saved Rouxtine never changes without consent [109][110].

### 5.6 Weakness detection from drill data, stage splits and smart-cube data

**Data sources.**
- **Drill data:** per case × channel (recognition, execution, errors).
- **Stage splits:** manual multi-phase timing (csTimer-style [43]) or smart-cube detection.
- **Smart-cube move stream:** pauses, TPS while turning, movecount against the solver optimum per stage [43][164].

**Options for the yardstick.**
- A. A fixed reference split table (Scheopner's model [27], SCT's profile [164]).
- B. Measured references (world-class ao100s [38][39]; later, opt-in community data).
- C. The user's own baseline and absolute targets (solver optimum, level movecounts).
- D. Peer comparison by level, which needs a server (Aimchess [116]).

**Recommendation: C first, A/B as labelled context.**
- **Rank stages by seconds lost, split by cause.** This is acubemy's three-pillar counterfactual [8] applied per Roux stage. SCT already automates a share-based version for Roux, with one cause per step [164]:
  - **Movecount:** stage moves vs the solver optimum and Scheopner's level targets [28][29][30][31].
  - **Turning speed:** TPS while turning (pauses removed).
  - **Pauses:** the stage-entry pause and mid-stage pauses.
  This needs no reference split table. It uses only the user's own data, the solver and published movecount targets. When one cause has to be named, check pauses first, then movecount, then TPS, as SCT does [164].
- **Consistency:** the coefficient of variation of each stage's time over the window. Monkeytype reports consistency as a CV mapped to 0–100 [83]. Report only the single least-consistent stage, as SCT does [164].
- **Share vs reference** as a secondary view. Compare log ratios, not raw percentages, because shares sum to 1 [158]. Use CubeSkills' 20 % band and flag only the largest deviation [36] (6.5). Label the reference "provisional" and account for what the user knows: someone on 2-look CMLL should not be told CMLL is slow [36].
- **Drill-level findings** (Lichess-style): "Improvement areas" and "Strengths" per stage and case group, against the user's own baseline, over 7/30/90 days, hidden below a minimum sample [120]. Phrase them concretely, as Aimchess does [116], e.g. "CMLL recognition is 48 % of your CMLL time". Recognition findings use only attempts with valid recognition times (6.1).
- **Routing table (stage × cause → module).** "No module yet" marks a gap in the 5.1 catalogue.

| Stage | Pause / recognition | Movecount | Low TPS while turning | High variance (least-consistent stage only) |
|---|---|---|---|---|
| FB | FB inspection planner (next scope or a shorter limit, by its scope ladder, 6.5); partial SpeedBLD FB | Block trainers (FS, FB) with optimal-length ladder; Slow solves (5.1), per-stage movecount against the solver optimum | Block trainers (FS, FB) at a fixed, comfortable length, timed. No dedicated FB fingertrick module yet | Case-level review: reconstruct the slowest solves' FB |
| SB | Tracking trainer; SB last-pair trainer | Block trainers (SS, SB) with ladder; SB last-pair trainer against solver solutions | Block trainers (SS, SB) at a fixed length, timed. No dedicated SB execution module yet | Case-level review of the slowest solves' SB, grouped by last-pair case |
| CMLL | CMLL recognition (any angle) | Alg choice: CMLL execution, comparing alg variants by small averages, not by length alone [5] | CMLL execution (small averages per alg) | Per-case review: the slowest CMLL cases in those solves, then "Drill slow" |
| EO / LR | EO / EOLR trainer | EO / EOLR trainer, movecount vs optimum | No module yet (an EO/LR execution mode) | Per-case review of EO and EOLR cases, then "Drill slow" |
| 4c | 4c trainer | 4c trainer, movecount vs optimum | 4c trainer (execution mode) | Per-case review of 4c cases |
| SB→CMLL transition | NMCLL / CMLL prediction | – | – | – |

- Smart-cube recognition must use one definition. Cubeast counts AUF as recognition [42]; csTimer and Cubedex do not [43][44]. Imported numbers are not directly comparable.

### 5.7 Goals and progress

**Options.**
- A. One time goal (sub-X).
- B. Per-module mastery and coverage.
- C. Periodic benchmarks.
- D. Dedication metrics (time, streaks).

**Recommendation: all four, kept apart.**
- **Headline:** Mo10Ao5 (Kian [15]) over the last 50 timed solves, from timed-solves blocks only; ao50 is the secondary number, and ao5/ao12 are shown per run. Averaging rules are in 6.5: WCA for ao5 [159], and for ao12 and larger the 5 % trimmed mean that Twisty Timer and Cubic Timer use, so exported numbers match what those apps show (Rouxtine follows Cubic Timer's arithmetic; Twisty differs when an average holds two or more DNFs) [53][167]. Drill blocks never feed PBs [47].
- **Per module:** the case status ladder and a per-case confidence bar (6.5) [81][46].
- **Benchmark Rouxtine:** a fixed-seed FB inspection test, SB efficiency vs solver optimum, a CMLL recognition coverage run, an EOLR test and an ao12 with splits. Suggest re-running it monthly. Aimlabs advises training the weakest subcategories and coming "back to the Benchmarks periodically" [75]; Fitbod reassesses with Max Effort Days [108]. Placement uses only an adaptive subset of it (5.4).
- **Dedication** (practice time, days, streak) on its own screen. Melodics calls XP "a dedication rating", not skill [89].
- **Forecasts:** show time-to-goal only with enough data and with uncertainty (6.5). keybr hides its estimate when the fit is poor (R² < 0.5) [81].

### 5.8 Sharing without a server

**Options.**
- A. A file (`.rouxtine`) through the share sheet or the file picker.
- B. A self-contained text code, e.g. `RXT1.` + base64url(deflate(payload)) + checksum.
- C1. A custom-scheme link carrying the code (`rouxtine://…`). It needs no domain and opens the app directly, but does nothing on a device without the app.
- C2. An https link with the payload in the URL fragment. It could fall back to a page that explains how to get the app, but it needs a host page and, to open the app directly, verified App Links; neither was researched for `nauroth.github.io` (section 7).
- D. A QR code of the code.
- E. An image card summarising the Rouxtine (Hevy shares folders as an image [104]).

**Recommendation: B as the canonical form; v1 wraps it in A, C1, D and E.** Add C2 only if the App Links question in section 7 resolves. A C2 link would carry the same code, so links of both kinds stay valid.
- **No lookup.** KovaaK's codes need a server [67]; Aimlabs deeplinks need a link handler plus a download [73]. Rouxtine has no domain [165]. It is also assumed to have no server or accounts, which follows from offline-first (owner's brief, not yet recorded in HANDOFF; see section 7). If a backend is ever added, this recommendation still holds, because codes work without one.
- **Payload:** format version, minimum app version, name, description, blocks (module id + version, mode, settings, selector, stop rule, role, block `description`), schedule. **Never** stats, SRS state or personal run notes (`run_block.note`), as Anki strips scheduling and leech tags when sharing [125].
- **Name inside the payload,** shown on import, so the code and what users see never diverge [69].
- **Compact.** Monkeytype warns its settings URLs may fail above 2000 characters [84].
- **Import:** preview the composition first (as KovaaK's shows the aim-type mix [68]), save as a copy, and keep unknown modules from a newer app version as placeholders with a warning. When a shared Rouxtine is imported again, ask whether to update the earlier copy. Anki keeps the newer version by default and, since 23.10, lets the importer choose to always or never update [125].

### 5.9 Reminders and streaks

**Options.**
- A. None.
- B. A daily streak.
- C. A weekly goal (N practice days) with freezes.
- D. A daily-minutes goal.
- E. Local reminders.

**Recommendation: C + E, with D optional; no daily streak by default.**
- **Weekly goal**, default 3 days a week, with freezes earned and applied automatically. Melodics locks a week at 3 days and applies freezes automatically, while keeping a daily streak in a less prominent Insights tab [93]. Yousician counts weeks in a row [94].
- **Configurable rollover hour,** as Monkeytype allows −11 to +12 h in 0.5 h steps [84].
- **Optional minutes goal.** keybr defaults to 30 min a day [81]; Yousician lets users pick stars or minutes per session [94].
- **Local, offline reminders,** off by default; ask for permission when the user turns them on.
- **Keep streaks secondary** to skill progress [89]. Feliks says forced practice is less effective [1]. No retention data comparing daily and weekly streaks was found.

### 5.10 Run lifecycle and export

Every attempt is saved as it happens (6.2), so these questions decide how a run is closed and what leaves the app, not whether data is lost.

**An interrupted or abandoned run.**
- A. Resume it later. Cube Gym has named, resumable sessions [64].
- B. Close it as a partial record: finished blocks count, and the open block is marked unfinished.
- C. Discard it.
- **Recommendation: A, falling back to B.** A run left mid-way can be resumed the same local day at the block where it stopped. After that, or when the user ends it, it closes as partial. Its attempts count toward stats and SRS, because they were real attempts. The attempt in progress when the app lost focus is stored as `interrupted` (6.2). Discard only on explicit request; it marks the run's attempts excluded from stats and SRS, or deletes them if the user asks.

**A block whose hardware is not connected** (smart cube or BLE timer).
- A. Skip the block.
- B. Fall back to touch, or another input the module accepts.
- C. Block the whole run until the device connects.
- **Recommendation: B where the module has a fallback, otherwise A; never C.** Check each block's required inputs before the run starts, and say what changes (e.g. smart-cube solves become touch-timed solves without automatic splits). A device that disconnects mid-block pauses the block and offers reconnect, fallback or skip. `input_device` is logged per attempt (6.2), so fallback attempts never mix into smart-cube-only stats.

**What "Export to Twisty/Cubic Timer" writes** (HANDOFF Timer decision, categories such as "Roux FB" and "CMLL" [165]; format in `timer-integration.md` [166]).
- A. Timed full solves only, as "Roux full".
- B. Also timed drill attempts, one category per stage or module ("Roux FB", "Roux SB", "CMLL", "LSE"), as `timer-integration.md` proposes [166].
- C. B, with the user choosing which modules to export.
- **Recommendation: C, defaulting to B.** Drill categories stay separate from "Roux full", so drill times never feed full-solve PBs in either app, matching Rouxtine's own rule (5.7). Export only attempts that have a solve time: no tap-to-name quizzes, no recognition-only modes and no preview Quick sessions (5.2); warm-up blocks are off by default. Penalty = 1 for +2, with the 2 s already added to Time(millis); Penalty = 2 for DNF; round times to 10 ms for Twisty (`timer-integration.md` §A) [166]. The comment field may carry module, mode and case, never `run_block.note`.

**Notes.** `block.description` belongs to the Rouxtine and is shared with it (5.8). `run_block.note` is personal: shown at the next run, never shared and never exported.

## 6. Per-attempt data and algorithms

### 6.1 Clocks and timing protocol

- `System.currentTimeMillis()` "may jump backwards or forwards unpredictably". `elapsedRealtime()` includes deep sleep and is "the recommend basis for general purpose interval timing". `uptimeMillis()` stops in deep sleep and is the basis of `System.nanoTime()` [160].
- `MotionEvent.getEventTime()` and `getEventTimeNanos()` (API 34) are in the `uptimeMillis()` base [161]. In the AOSP source, `Choreographer.FrameCallback.doFrame(frameTimeNanos)` is in the `System.nanoTime()` base and is the scheduled frame start, not the callback run time [162]. Touch stamps and frame stamps are therefore directly comparable.
- **Rule:** time within an attempt in the uptime/nanoTime base (touch events, frame stamps, BLE arrival times converted to it). Use `elapsedRealtimeNanos` only for spans that may cross deep sleep. Use UTC wall time plus the time-zone offset only for calendar bucketing. Never subtract stamps from different bases. `timer-integration.md` line 196 names `getEventTime()` and `elapsedRealtime*()` side by side without this caveat [166].
- **"Case shown"** is the frame start of the first frame that draws the case [162], not the moment state was set.
- **Recognition is only measurable if the user has not seen the case yet.** In physical and smart-cube case drills the user applies the setup scramble and sees the case on the cube before the app "shows" it, so release − shown or first move − shown then measures re-recognition. That value would feed the `Weakest` weights, FSRS Hard grading (over 1.35 × median, 6.5) and findings such as "CMLL recognition is 48 % of your CMLL time" (5.6). No source gives a blind-setup rule, but wodzik/cube (GPL-3.0) meets the same problem: its Scramble mode times recognition from the last scramble move, after the user has seen the case, and it keeps those stats apart from on-screen Recognize mode, because "their times don't compare" (`src/pages/TrainersPage.tsx`) [56]. This supports keeping recognition stats separate per protocol. Options:
  - A. Trust `rec_ms` only for cases the app draws: tap-to-name quiz, virtual cube, case image. Shown = the first frame that draws the case; recognition = answer tap, release or first virtual move − shown.
  - B. Blind setup for physical and smart cubes: the user applies the setup without looking at the U face (for example holding it U face down), and only then looks. Touch: the user presses and holds the pad, turns the U face into view, and releases to start execution, so recognition = release − press. Smart cube: the app detects that the setup is complete, and the first move after it starts the clock, so recognition = first move − setup completed. Both spans include bringing the U face into view, so compare them only within the same protocol. Recorded with `is_setup_blind`.
  - C. Record only total time for physical-cube drills; `rec_ms` stays empty.
  - **Recommendation: A + B, with C as the fallback.** Store `rec_valid` per attempt: true for A, true for B only when the user confirmed a blind setup, false otherwise. Keep valid recognition stats per protocol (A or B), never pooled, as wodzik keeps Scramble and Recognize apart [56]. Invalid `rec_ms` is still logged, but it is excluded from recognition stats, the recognition channel of `Weakest`, findings and FSRS Hard grading, which then uses execution time and mistakes only.
- **Execution:** touch, stop − release; smart cube, case solved − first move. In full solves, call the gap before a stage's first move the "stage-entry pause", not recognition, because lookahead overlaps it.

### 6.2 Common envelope (every module)

Store every attempt as an append-only row. Everything else (EMAs, shrunk means, FSRS state, plateau flags) is derived and can be rebuilt by replay with a `model_version`. FSRS-7 migration in particular requires replaying the full history with real timestamps [128].

| Field | Notes |
|---|---|
| `attempt_id`, `run_id`, `rouxtine_id`, `block_id` | UUIDs; a Quick session has no Rouxtine |
| `module_id`, `module_version`, `mode` | e.g. `cmll`, `3`, `recognition` |
| `input_device` | touch, stackmat, GAN/QiYi timer, smart cube model + firmware, keyboard |
| `started_utc_ms`, `tz_offset_min` | Day bucketing for streaks and the weekly goal; FSRS elapsed time (6.5) |
| `t_shown`, `t_first_action`, `t_exec_start`, `t_end` | Monotonic ns, one base (6.1) |
| `rec_ms`, `exec_ms`, `total_ms`, `inspection_ms` | Derived but stored for export |
| `rec_valid`, `is_setup_blind` | Whether `rec_ms` may feed recognition stats, `Weakest`, findings and FSRS grading (6.1) |
| `outcome` | ok, mistake_corrected, wrong_case, wrong_alg, alg_looked_up, DNF, +2, abort, interrupted. Store a DNF as an outcome, never a time (design rule); WCA averaging counts a DNF as the worst result [159] |
| `mistakes`, `self_grade` (optional), `index_in_run`, `is_warmup`, `is_preview` | `is_preview`: a Quick session that does not count toward SRS and stats (5.2) |
| `is_scheduled_review`, `derived_grade`, FSRS state before and after (S, D, R, due) | For SRS replay and audits. `is_scheduled_review` is true only for the first cold attempt after the case's due time (6.5) |
| `app_version`, `model_version` | |

### 6.3 Module payloads

| Module | Extra fields |
|---|---|
| Case drills (CMLL, EOLR, 4c, SBLP) | `case_set_id`, canonical `case_id` (AUF-invariant), `presented_auf`, orientation (colour on D/L, y rotation), `alg_id`, `alg_len_stm`/`etm`, scramble or setup + generator + seed, pre-moves, active set size |
| Recognition quiz | answer, correct answer, number of options, mask type |
| FB inspection planner | scope level, time limit, inspection used, FB colour/orientation chosen, planned solution, executed moves, plan-vs-executed match, movecount vs optimum for that orientation and for the best allowed orientation, hide-after-first-turn on/off |
| Block trainers (FS, FB, FB+DR, SS, SB, SBLP) | target optimal length (ladder level), user moves, optimal length, hint used |
| EO / EOLR / EOLRb / 4c | EO case, misoriented-centre flag, method (EOLR, EOLRb; FUBU/BU/DFDB), movecount vs optimum |
| Slow solves | scramble, pace (untimed or metronome BPM), no-pause rule on/off, entered or detected solution; per stage: moves, solver optimum, pause time (smart cube). Never feeds PBs or averages |
| Timed full solves | scramble, penalty, splits (FB, FS/SS, LP, CMLL, EO, LR, 4c) each with start, first move, end, moves, pause time, TPS; solver optimum per stage, computed later in the background |
| Smart-cube solves | raw event stream (face, direction, cube timestamp, host timestamp, missed/recovered flag), fitted timestamps, gyro if available |

### 6.4 Smart-cube ingestion

1. Store raw events. gan-web-bluetooth gives each move a host and a cube timestamp; either can be null for missed-then-recovered moves [163].
2. Fill missing cube stamps at ±50 ms from neighbours, then fit cube time to host time by linear regression per solve. GAN clocks drift; the method is credited to Chen Shuang and csTimer [163].
3. Merge two same-face quarter turns into a half turn. SCT uses a 160 ms window [164].
4. Merge opposite-face pairs into slices. An M move arrives as R and L' 6–121 ms apart; SCT merges within 140 ms and rebuilds M = R L' x', re-framing every later move, or stage detection fails [164].
5. Detect stages. Licence-compatible references: csTimer replays the moves and checks FB, SB and CMLL facelet masks in all 24 orientations (`getRouxProgress` in `src/js/lib/cubeutil.js`, used by `src/js/stats/recons.js`; GPL-3.0) [43]. wodzik/cube splits speedsolves into FB/SB/CMLL/LSE with per-stage recognition and execution (`src/logic/stageDetection/rouxStages.ts`, GPL-3.0) [56]. It delegates detection to the cubecore library, whose Roux method also splits LSE into EO, UL/UR and L4E (`roux/index.js`, MPL-2.0) [168]. So the finer EO/LR/4c splits of 5.1 have a reference too; SCT stays ideas-only [164]. csTimer merges a one-move step into the *preceding* step [43].
6. Detect pauses at gaps of at least max(180 ms, 2.2 × the median gap) [164]. A stage's pre-first-move pause belongs to that stage [164].
7. Count moves in STM and ETM and compute TPS with and without pauses. csTimer's HTM counter counts each face once per same-axis run (R L R counts 2) [43].

### 6.5 Algorithms

Values marked *proposed* are design defaults, not sourced facts.

**Per case × channel estimates** (channels: recognition, execution; plus an error channel):
- Work in log time: x = ln(seconds). Pelánek models problem-solving times as log-normal, citing Jarušek and Pelánek (2012) [144]. Ratios become differences: +0.10 is about 10.5 % slower.
- EMA: m ← m + a·(x − m), with the first sample initialising m. *Proposed* half-life 8 attempts, so a = 1 − 2^(−1/8) ≈ 0.083, which is pandas' half-life formula a = 1 − exp(−ln 2 / h) [143]. keybr uses a = 0.1, a half-life of about 6.6 samples [81].
- *Proposed* caps before updating: recognition at 15 s (Cubedex discards above 15 s [44]); execution at 3 × the case median. Log the raw value anyway.
- Decayed counts: n ← λn + 1 and e ← λe + error, with λ = 2^(−1/8) (*proposed*). Error estimate p̂ = (e + 1)/(n + 10) (*proposed* prior).
- Shrinkage toward the set mean, normal–normal model: m̃ = (κ·μ_set + n·m)/(κ + n), with pseudo-count κ = σ²/τ² [153]. *Proposed:* κ = 4 until the module has 50 attempts, then estimate τ² by method of moments. Posterior sd u = σ/√(κ + n).
- Expected time, so a long alg is not "weak" (Pelánek's time-Elo idea, E = d − θ in log time [144]): execution E = ln(L/TPS_user), where L is the alg's STM length and TPS_user is the median of L_j/t_j over the set; recognition E = the set median.
- Slowness s = m̃ − E. Confidence c = T_target / exp(m̃), keybr-style [81]. T_target defaults to 0.85 × the current set median (*proposed*) or comes from the user's goal.

**`Weakest` selector** (*proposed*, assembled from sourced pieces):
- Bootstrap: serve each case × angle until it has 3 successful reps (Cubeast [42]).
- Weight W = max(0, s + 0.5·u) + 2·p̂ + (1 − R), where R is FSRS retrievability, or 1 if the case is not scheduled. The u term is an uncertainty bonus in the spirit of UCB1 [157]; UCB1's guarantees assume rewards in [0, 1], so this is a heuristic.
- Probability P ∝ 0.1·mean(W) + W, then clip so the most likely case is at most 8 × the least likely. The floor follows SpeedcubeQuest's 0.12 [45]; the 8:1 bound follows Cubeast [42].
- Cooldown: skip the last min(2, N − 1) cases. After a failure, reinsert the case 4 items later, at most 2 extra repeats per pass (Cubedex's values) [44].
- To draw k distinct cases at once, give each case the key u^(1/P) with u ~ U(0, 1) and take the k largest (Efraimidis and Spirakis, A-ES) [156].

**Other selectors.** `Recap`: a shuffled permutation, each case once per cycle [43][49]. `Realistic`: natural case probability [43][48]. `Ladder` for intuitive modules: +1 move when ≥ 80 % of the last 10 at the current length were optimal [56]; *proposed* −1 when under 40 %.

**Scope ladder for the FB inspection planner** (*proposed*). acubemy's principle: practise without the 15 s limit, always planning one step further, and add the next step once it fits in 15 s [8]. A plan succeeds when the executed solution matches it (plan-vs-executed match, 6.3). When at least 80 % of the last 10 plans at the current scope and limit succeeded, tighten the limit one step (unlimited → 30 s → 15 s); at 15 s, move up one scope with no limit. Tighten the limit before widening the scope. Under 40 %, loosen the limit one step, or at no limit drop back one scope. The thresholds mirror `Ladder`: 80 % of the last 10 from wodzik [56], 40 % proposed.

**New-case gating** (for learning CMLL, EOLR, NMCLL):
- keybr's default adds a letter once every included letter has reached target speed at least once; its strict option uses current confidence [81]. *Proposed default:* every active case has reached c ≥ 1 at least once and has p̂ ≤ 0.2. Strict option: every active case at c ≥ 1 now and p̂ ≤ 0.2, as keybr's "recover keys" requires every included key at current confidence ≥ 1 [81].
- *Proposed* cap: 3 new cases per day, like Anki's new-card cap (default 20) [121]. Always offer "add next case now" [82].

**FSRS for recall-type cases** (CMLL alg recall, EOLR/EOLRb solutions, 4c recognition, NMCLL→CMLL mapping; not FB planning or full solves):
- **Version.** FSRS-7 (34 weights, dual stability, fractional days) is the default of ts-fsrs 6.0.0, published 2026-10-28, and of its betas since 2026-09-22 [128]. It exists on fsrs-rs `main` but in no released crate: fsrs 6.6.2 is FSRS-6 only, and Anki pins it [129][134]. FSRS-6 (21 weights) is the newest version on the algorithm wiki and in py-fsrs 6.3.2 [130][131].
- **Benchmark.** srs-benchmark (about 10k Anki users, about 727M reviews), log loss without same-day reviews: FSRS-7 0.3401, FSRS-6 0.3460, FSRS-7 with default weights 0.3620, HLR 0.4694 [132]. With same-day reviews included: FSRS-7 0.3206, FSRS-7 default 0.3399, FSRS-6 0.3842 [132]. FSRS-7 is called "the only version that can give realistic predictions of probability of recall for same-day reviews" [132]. The parameter-free MOVING-AVG baseline beats FSRS-7 on log loss (0.3369) but not on AUC (0.7001 vs 0.7167) [132].
- **Migration.** FSRS-6 weights and memory states cannot be converted to FSRS-7. Rebuild by replaying the full history with real review times; never invent the fast stability as 0.8 × stability [128]. The README's FSRS-7 default weights differ from those in the libraries and in the benchmark's own code [128][132].
- **FSRS-6 formulas** (py-fsrs 6.3.2) [131]:
  - Ratings: Again = 1, Hard = 2, Good = 3, Easy = 4.
  - Retrievability R(t, S) = (1 + F·t/S)^(−w20), with F = 0.9^(−1/w20) − 1, so R(S, S) = 0.9.
  - Interval I = (S/F)·(r^(−1/w20) − 1), rounded to whole days and clamped to [1, max interval].
  - Initial: S₀ = w[G−1]; D₀(G) = w4 − e^(w5·(G−1)) + 1.
  - Difficulty: D' = clamp[1,10]( w7·D₀(4) + (1 − w7)·(D + (10 − D)/9 · (−w6·(G − 3))) ).
  - After recall: S' = S·(1 + e^w8·(11 − D)·S^(−w9)·(e^(w10·(1 − R)) − 1)·[w15 if Hard]·[w16 if Easy]).
  - After a lapse: S' = min( w11·D^(−w12)·((S + 1)^w13 − 1)·e^(w14·(1 − R)), S/e^(w17·w18) ).
  - Same day (fewer than one whole day elapsed): S' = S·e^(w17·(G − 3 + w18))·S^(−w19), with the increase at least 1 for Hard, Good and Easy. Elapsed time is counted in whole 24-hour periods (`timedelta.days`, `scheduler.py` L232 and L269–L271), not calendar days, both here and in R.
  - Default weights: `[0.212, 1.2931, 2.3065, 8.2956, 6.4133, 0.8334, 3.0194, 0.001, 1.8722, 0.1666, 0.796, 1.4835, 0.0614, 0.2629, 1.6483, 0.6014, 1.8729, 0.5425, 0.0912, 0.0658, 0.1542]`.
  - Defaults: desired retention 0.9, learning steps 1 min and 10 min, relearning step 10 min, maximum interval 36,500 days, fuzz on (not applied below 2.5 days).
- **Grading.** Anki: Hard is a passing grade; pressing Hard on a forgotten card makes "all intervals… unreasonably high" [122]. Cubedex grades automatically: abort = Again; a mistake, or recognition or execution over 1.35 × the median of the last 8 successful reps (at least 3), = Hard; Easy after 11 straight successes at or below the baseline; otherwise Good [44]. Cubedex counts every non-aborted rep as a success, so mistaken Hard reps still count toward "learned" (11 of the last 12, at least 5 reviews) [44]. *Proposed for Rouxtine:* Again = DNF, abort, wrong case, wrong alg or alg looked up; Hard = corrected mistake, or over 1.35 × own median (recognition counts only when `rec_valid`, 6.1); Good otherwise; Easy manual only.
- **Scheduling rule** (*proposed*): a case's scheduled review is the first cold attempt after its due time has passed (a new case is due at once). Only that attempt gets `is_scheduled_review` (6.2) and updates FSRS state. Attempts on cases that are not due yet, such as those `Weakest` draws, update only the per-case estimates. Turn off FSRS learning and relearning steps, so every interval is at least one day; repetition within a run is the selector's job. py-fsrs uses the same-day formula when less than one whole 24-hour period has elapsed [131], so feeding every in-session rep would distort stability. A review after the due time is always at least a day after the previous one, so py-fsrs outputs stay valid test vectors for the port. A rule of one review per local calendar day would not: attempts at 21:00 and at 08:00 the next day are 11 h apart, which py-fsrs treats as a same-day review with 0 elapsed days. The alternative is to count elapsed days as local calendar days with the 5.9 rollover hour and pass that in place of `timedelta.days`, so a case due at 08:30 is still served to a user who practises at 08:00. That is a deliberate deviation from py-fsrs and needs its own test vectors. Due cases go first in a block, capped per run. When over the cap, serve the lowest retrievability first, which is how Anki's "relative overdueness" order works under FSRS [124], and spread the rest over several days. Cases that fall due on a day the user does not plan to practise wait for the next practice day and are served there lowest retrievability first; FSRS uses the real elapsed time, so the late review needs no correction. Anki's Easy Days is prior art for fitting the load to the week: each weekday is set to Normal, Reduced or Minimum, and new intervals are shifted slightly to move due dates off the lighter days, without changing existing intervals [122]. *Proposed:* an optional per-weekday load setting, prefilled from the weekdays the user marks as practice days (5.3, 5.9). Nate Solon advises reviews before new material and a ten-minute daily timebox [114].
- **FSRS needs data.** Anki warns it "may not perform well" with fewer than "a few hundred" reviews [122], and suggests re-optimising about monthly [122]. The FSRS wiki reports 20–30 % fewer reviews than SM-2 at the same retention, from simulation [127].
- **Library choice:**
  - Port py-fsrs 6.3.2 (MIT, about 300 lines of scheduling logic) to Kotlin and test it against py-fsrs outputs [131]. *Recommended.*
  - FsrsKt 1.0.1: pure Kotlin Multiplatform FSRS-6, GPL-3.0 (compatible), 276 lines, no tests, last commit 2026-07-24 [140]. Use as a cross-check or vendor it.
  - fsrs4s 0.3.0: Scala 3, Apache-2.0, based on py-fsrs; pulls in the Scala runtime and cats-core [141].
  - fsrs-rs (BSD-3-Clause) through JNI or UniFFI, only if on-device optimisation is wanted later [129]. rs-fsrs-java is a JNI binding to FSRS-5 [139].
  - Not usable: FSRS-Kotlin, listed as FSRS-6 by awesome-fsrs [136], has no LICENSE file and imports app classes (`app.arteh.flashcard.db.*`, `R.color`) [137]; android-fsrs implements FSRS-4 [138].
- **SM-2** for reference: intervals 1, 6, then × EF; EF starts at 2.5 with a floor of 1.3; a grade under 3 restarts repetitions. It is day-granular and only captures hesitation through the self-grade ("correct response after a hesitation") [142].

**Recognition mastery** (Pelánek and Řihák) [145]:
- Time-aware value: a wrong answer scores 0; a correct one scores linearly from 1 at 0 s down to 0 at twice the median response time, so v = max(0, 1 − t/(2·median)), and a median-speed answer scores 0.5.
- EMA θ ← α·θ + (1 − α)·v with θ₀ = 0. For the two-option grammar exercises they analysed, umimecesky.cz used α = 0.9 (α varies by exercise type) with a main mastery level of 0.95 [145]; their MatMat analysis used α = 0.8.
- Their finding: the input data (whether to use response times) and the threshold matter more than the learner model.
- *Proposed:* mastery when the binary EMA ≥ 0.95 and the time-aware EMA ≥ 0.5, with α = 0.8. From θ₀ = 0, a binary EMA reaches 0.95 after 14 straight correct answers at α = 0.8 (0.8¹⁴ ≈ 0.044), or 29 at α = 0.9 (0.9²⁹ ≈ 0.047).
- **Optional deadline score** for modes that show a visible time limit (FB inspection planner, timed recognition). Math Garden's "high speed, high stakes" rule (Klinkenberg et al. 2011, as summarised by Pelánek) scores S = (2·correct − 1)·(a·d − a·t), where d is the time limit and a a discrimination parameter: a fast answer gains or loses a lot, and the score tends to 0 as time runs out [144]. Pelánek says it suits systems that show timing and deadlines; where timing is hidden, use the time-aware value above [144].

**Stage weakness from splits** (*proposed*):
- Counterfactual seconds per stage s, from smart-cube or manual splits: pause_s, moves_s, turning time τ_s = t_s − pause_s, TPS_s = moves_s/τ_s. Savings: from pauses = pause_s − target pause; from movecount = (moves_s − target moves)/TPS_s; from TPS = τ_s − moves_s/target TPS. Targets: solver optimum + a margin, Scheopner's level movecounts [28][29], and the user's own best 10 % TPS for that stage. Target pause (*proposed*): the user's own best-decile (10th-percentile) pause for that stage over the window, so pause savings are measured like TPS savings, against what the user has already done. acubemy's "remove pauses" counterfactual uses 0 [8]; show that only as an upper bound, because even the world-class ao100s spend 21–23 % of solve time on recognition [38][39]. Do not use SCT's pause ratios as a norm; their reference profile is unsourced [164]. This automates acubemy's by-hand per-solve method [8]. SCT automates a share-based version for Roux [164]; Rouxtine differs by using log ratios, shrinkage and the solver optimum.
- Share view: y = ln t_s − ln(π_s·T), with π_s the provisional reference share. Shrink: δ̂ = n·ȳ/(n + κ_s), κ_s = σ²/τ² ≈ 5 (σ ≈ 0.35, τ ≈ 0.15). Flag when δ̂ − 0.84·sd > ln 1.2, i.e. about 80 % posterior probability of being more than 20 % slow, matching CubeSkills' band [36]. Report only the top-ranked flagged stage, as CubeSkills flags only the largest deviation [36]. Seconds lost = T̄·π_s·(e^δ̂ − 1).
- Consistency: per stage, the coefficient of variation of stage time over the window, with at least 8 solves (SCT's minimum [164]). Flag only the single least-consistent stage [164] and route it to a case-level review of the slowest solves (5.6). Use log ratios because shares are compositional [158]; Greenacre et al. find plain pairwise log ratios sufficient [158].

**Plateaus and regressions** (*proposed*):
- Daily value = median ln(time) over at least 5 full solves (or a module's attempts). Window = the last 14 practice days, at least 8.
- Plateau when: the Mann-Kendall trend test has p ≥ 0.10 (Kendall's tau-b, e.g. Commons Math 3.6.1 `KendallsCorrelation`, Apache-2.0 [155]); the Theil-Sen slope (median of pairwise slopes [154]) projects under 1 % gain over 30 days; and an exponential fit T(n) = A + B·e^(−n/τ) puts current time within 3 % of the asymptote A. Heathcote et al. found the exponential fit individual learners better than the power law in all unaveraged data sets [151]; applying it to one user is our inference.
- Regression alarm: one-sided CUSUM, S = max(0, S + x − μ₀ − k), on residuals with σ = 1.4826·MAD, k = 0.5σ, h = 4σ. In standard units this gives an in-control average run length of 336 and 8.38 for a 1σ shift [146].
- Response: suggest rotating the focus to the top seconds-lost stage, adding cases, or changing drill type. Word a plateau as normal, not as an alarm: Feliks says long stagnant periods often end with a 1–2 s drop within days, and that stagnating is "completely natural" [1]. Do not extrapolate with polynomials. keybr fits a polynomial of degree 1–3 to at most the last 30 samples since the latest slowdown, and reports "remaining lessons" only when R² ≥ 0.5 [81].

**Leeches.** Anki flags a leech at 8 lapses and warns every 4 lapses after that. Its remedies: rewrite, delete, or suspend one of two interfering items until the other is learned [123]. Anki's code default only tags; the manual says it also suspends [121][123]. *Proposed for Rouxtine:* after 8 lapses, suggest a different alg (fewer regrips), a different recognition method, or a slow-execution drill, and stop raising the case's weight.

**Not for v1.**
- Elo for student–item pairs: P = 1/(1 + e^−(θ − d)), with an uncertainty function U(n) = a/(1 + b·n); a = 1, b = 0.05 were fitted on real data [144]. With one learner, user skill and case difficulty are hard to separate, and shrunk log-time residuals already cover the need.
- Glicko-2: start 1500 / RD 350 / volatility 0.06; it "works best" with at least 10–15 games per player per rating period [147]. Lichess rates each puzzle attempt as a Glicko game between user and puzzle, damped by a weight for the chosen theme, with the puzzle's rating change capped at ±700 [120]. Revisit only if anonymised community data is ever pooled.
- Hick-Hyman effects on recognition across set sizes: see open questions.

**Averages.** WCA ao5: drop the best and worst, mean of the other 3; one DNF counts as the worst, two make the average DNF. Results under 10 min are truncated to hundredths and averages rounded. Inspection must be strictly under 15 s: +2 at 15.00 s, DNF at 17.00 s [159]. The WCA defines no larger average, so for ao12, ao50, ao100 and ao1000 match Twisty Timer and Cubic Timer, the export targets: trim ceil(5 % of n) results from each end (ao12: 1, ao50: 3, ao100: 5, ao1000: 50). DNFs sort as the worst results, and the average is DNF when DNFs outnumber the trim. mo3 is a plain mean and is DNF with any DNF. Both apps fix ao5 and ao12 at 5 % (which gives the WCA ao5) and let the user change the trim for ao50 and larger, default 5 % [53][167]. They differ with two or more DNFs: Cubic Timer divides the middle sum by n − 2·trim [167], while Twisty Timer also subtracts each DNF after the first from the divisor, so its own test expects 102.41 s for an ao50 with 3 DNFs whose trimmed mean is 97.76 s [53]. *Proposed:* follow Cubic Timer, with the trim fixed at 5 %. Mo10Ao5 = mean of 10 ao5s [15]; *proposed*: computed over the last 50 timed solves in consecutive groups of 5, and DNF if any of its ao5s is DNF.

### 6.6 Licences of code and ideas

Rouxtine is GPL-3.0-only [165]. Not legal advice. HANDOFF already rules out TNoodle (AGPL) without permission [165], so treat AGPL sources (keybr, lila, Anki) as ideas-only unless the owner decides otherwise (section 7).

| Source | Licence | Use |
|---|---|---|
| py-fsrs [131] | MIT | Port; keep the notice |
| ts-fsrs 6.0.0 [128] | MIT, with runtime dependency srs-kit (BSD-3-Clause); Node ≥ 24 | Reference for FSRS-7 |
| fsrs-rs, fsrs-optimizer [129][133] | BSD-3-Clause | Optional optimiser via JNI/UniFFI |
| FsrsKt [140] | GPL-3.0 | Compatible; cross-check or vendor |
| fsrs4s [141] | Apache-2.0 | Compatible; heavy dependencies |
| FSRS-Kotlin [137] | No LICENSE file (README says MIT) | Don't use |
| Anki [134], Anki-Android-Backend [135] | AGPL-3.0-or-later, GPL-3.0 | Anki: ideas only (HANDOFF rule). Neither is needed |
| Cubedex [44], gan-web-bluetooth [163], Tao Yu Alg-Trainer [52], scalachess [149] (its Glicko code is "loosely ported" from goochjs/glicko2) | MIT | Ideas or code with notice |
| goochjs/glicko2 [148] | BSD-2-Clause | Not needed for v1 |
| csTimer [43], wodzik/cube [56], Twisty Timer [53], Cubic Timer [167] | GPL-3.0 | Compatible |
| cubecore [168] | MPL-2.0, not marked "Incompatible With Secondary Licenses" | Compatible (MPL-2.0 §3.3); files ported from it keep the MPL notice |
| CubeDesk [54] | README/LICENSE GPL-3.0-or-later; `package.json` says "All Rights Reserved" | Ideas only until clarified |
| keybr [81] | LICENSE AGPL-3.0 (`package.json` says GPL-3) | Ideas; reimplement |
| lila [120] | AGPL-3.0-or-later | Ideas; reimplement |
| SCT [164], Roman's trainers [49][50] | No licence | Ideas only |
| Cubeast [42], SpeedcubeQuest [45] | Closed (shipped bundles read) | Ideas only |
| Apache Commons Math 3.6.1 [155] | Apache-2.0 | Optional for Kendall tau |

## 7. Sources and open questions

### Open questions / unverified

- **No tool builds a Roux-specific cross-module plan:** true for every tool inspected, but acubemy has Roux insights and a "builds a training plan" claim, and its training page needs a login [58][60][61]. Unconfirmed.
- **Hick-Hyman law:** mean choice reaction time T = b·log₂(n + 1), and a probability-weighted form for unequal probabilities, with little or no increase for highly familiar responses. Checked only against Wikipedia, not the primary papers [152]. If it holds, recognition times from a 42-case set, a 7-case subset or an over-sampled distribution are not directly comparable, so keep them separate.
- **Kian's target times per step** and the content of CubeSkills' "Practicing" lessons were not readable [13][3]. YouTube transcripts failed; only titles, descriptions and chapters were used. The playlist RSS feed and cached watch pages do give metadata [12].
- **speedsolving.com and the r/Roux wiki** returned 403 and are not cited. A search snippet points to a free MIT CFOP+Roux trainer with an Android APK and CMLL 42 / LSE 15 sets. It may be `cellonty/cube-trainer`, already listed in `roux-training-features.md` [166]; not confirmed.
- **Cubeast's ranking:** the client sums times per case; the server's initial case-stats format was not checked [42].
- **Modacity's current Deliberate Practice UI:** the cited source is dead; the loop comes from 2018–2022 third-party descriptions [87].
- **Aimchess's "31% faster" claim:** no matching UBC study was found [116]. Its pricing differs between the website and the App Store [116][117]. Aimchess and Chessable belong to Chess.com [119].
- **Fitbod "AI insights":** one 2026 reviewer links erratic plans to them; no Fitbod source was checked [110].
- **Not read:** CubeSkills lesson videos, CubeDesk's trainer page and acubemy's training page (logins); cubesolv.es, so there is no Roux reconstruction corpus for split distributions; super-memory.com (SM-2 was read from SuperMemo's reprint [142]); the Lichess blog and keybr's GitHub issues.
- **Strong's help articles** date from 2021–2022, so "iPhone-only folders" may be stale [99][100].
- **acubemy's "3 million solves" benchmark** is marketing [58].
- **Confirm no backend or accounts.** Offline-first comes from the owner's brief and is not in HANDOFF's Decided list; HANDOFF records only "No domain" [165]. Sections 5 and 5.8 assume no server or accounts. If confirmed, add "offline-first; no backend or accounts" to HANDOFF.
- **Recognition timing on physical cubes:** the blind-setup protocol (6.1) is untested; whether users follow it, and how much turning the U face into view adds, is unknown.
- **App Links without a domain:** whether a verified https link on `nauroth.github.io` can open the app, and how much a QR code can hold, were not researched. The first decides whether 5.8 adds C2.
- **AGPL code:** treated as ideas-only, following HANDOFF's TNoodle rule [165]. Using keybr, lila or Anki code would need an explicit owner decision.
- **Daily vs weekly streaks:** no retention data was found; the weekly trend rests on Melodics (which keeps a daily streak too) and Yousician [93][94].
- **Metronome and lookahead drills for Roux:** all sources are CFOP-framed, and coaches disagree [6][7][8].
- **A reference split table for intermediate Roux levels** was not found. SCT's profile is unsourced [164]. Opt-in local data could seed one later.
- **Cubegrass "Level"** equals the length of the returned solution; that it is optimal is consistent with samples but not stated [57].
- **FSRS-7 adoption:** wait until it is released in more than one reference implementation with stable defaults [128][129][132].
- **Corrections to other notes (not edited here):** `roux-training-features.md` line 75 says no surveyed tool has SRS, but Cubedex has had it since 2026-04-21 [44][166]. `timer-integration.md` line 196 needs the clock-base caveat from 6.1 [160][161][166]. `roux-training-features.md` also lists Cubedex CMLL support as unconfirmed; it has CMLL, L6E-EO and L6E-EOLR sets [44][166].

### Sources

Coaching and Roux content
[1]: https://www.cubeskills.com/blog/cubing-practice-tips
[2]: https://www.cubeskills.com/blog/how-to-get-faster
[3]: https://www.cubeskills.com/tutorials/practicing
[4]: https://www.jaydenmcneillcubing.com/blog/blog-post-ten-awpyf
[5]: https://www.jaydenmcneillcubing.com/blog/blog-post-two-92mhs
[6]: https://www.jaydenmcneillcubing.com/blog/blog-post-one-zm35x
[7]: https://www.cubeskills.com/blog/lookahead-progression-framework
[8]: https://acubemy.com/blog/how-to-improve-at-3x3
[9]: https://book.rouxers.com/en/fb.html
[10]: https://book.rouxers.com/en/faqs.html
[11]: https://rouxcubing.com/#/rouxdmap
[12]: https://www.youtube.com/playlist?list=PLBHocHmPzgIh_ioQur_FmQ8vdRykjQJbn (metadata via https://www.youtube.com/feeds/videos.xml?playlist_id=PLBHocHmPzgIh_ioQur_FmQ8vdRykjQJbn). Videos: Sub-20 https://www.youtube.com/watch?v=qvakHvnoyO4, Sub-12 https://www.youtube.com/watch?v=G1a9yHuLzZA, Sub-10 https://www.youtube.com/watch?v=hubeq4DptyI
[13]: https://www.youtube.com/watch?v=cM7cYmDzj24 (Sub-15)
[14]: https://www.youtube.com/watch?v=AOi1-Qpi5u0 (Sub-8)
[15]: https://www.youtube.com/watch?v=oL1cWfA9J6k
[16]: https://www.youtube.com/watch?v=DjPnRCumeGo
[17]: https://www.youtube.com/watch?v=KASTPHkU8eU
[18]: https://www.youtube.com/watch?v=4KLFyN6ZDwk
[19]: https://resources.rouxers.com/
[20]: https://www.youtube.com/watch?v=4PIMiKQHCXE
[21]: https://www.youtube.com/watch?v=pitj8kFJqn8
[22]: https://www.youtube.com/watch?v=HN1m7NPR6gE
[23]: https://www.youtube.com/watch?v=kjvPJUstJrU
[24]: https://www.youtube.com/watch?v=9CNJtFVkXW4
[25]: https://www.youtube.com/watch?v=7xW2Lnea7MA
[26]: https://www.youtube.com/watch?v=MBGnqDjSBPE
[27]: https://www.scheopner.com/cube/
[28]: https://www.scheopner.com/cube/firstblock.php
[29]: https://www.scheopner.com/cube/secondblock.php
[30]: https://www.scheopner.com/cube/cmll.php
[31]: https://www.scheopner.com/cube/lse.php
[32]: https://www.youtube.com/watch?v=81XdAt5m1uc
[33]: https://www.youtube.com/watch?v=D77Cc_aYHKM
[34]: https://www.youtube.com/watch?v=eVGKGFt1KRo
[35]: https://www.youtube.com/watch?v=BUD5OJB9F6M
[36]: https://www.cubeskills.com/blog/cfop-solve-splits-tool
[37]: https://docs.google.com/spreadsheets/d/1jI56qznJAdqdTXqN9MyuQilijZNi7AIyLQpLjzbLqeA/
[38]: https://app.cubeast.com/public/share/oQfw8VwiRSwAU7MXcyYaj4
[39]: https://cbst.link/s/uvbYM9h8xn79c5rjRSfsZr
[40]: https://speedcubeshop.com/a/blog/new-year-faster-solves-how-to-improve-cubing-speed-in-2026

Cubing tools
[41]: https://www.cubeast.com/
[42]: https://app.cubeast.com/main.843d00e3c1cda244.js (live bundle on 2026-10-28)
[43]: https://github.com/cs0x7f/cstimer/tree/5f8e46cb78da6a487fc01ba913a6ce6fd101e28d (`src/js/timer.js`, `src/js/timer/giiker.js`, `src/js/lib/cubeutil.js`, `src/js/stats/recons.js`, `src/js/stats/trainstat.js`, `src/js/scramble/scramble.js`, `src/lang/en-us.js`)
[44]: https://github.com/poliva/cubedex/tree/8a8b5b99ef69386d874bbb9662702efff165c1cd (`src/lib/srs.ts`, `src/hooks/useTrainingState.ts`, `src/views/HelpView.tsx`, `src/data/defaultAlgs.json`; SRS added in commit 113b8b3)
[45]: https://speedcube.quest/assets/drillWeighting-B1jZBA10.js
[46]: https://speedcube.quest/trainers and https://speedcube.quest/trainers/algorithms/cmll
[47]: https://speedcube.quest/change-log
[48]: https://jperm.net/algs/pll and https://jperm.net/algs
[49]: https://github.com/Roman-/oll_trainer/blob/a2c6e48bc27116f2898f3a9a4523d54011a4363f/scripts/timer.js#L56-L72
[50]: https://github.com/Roman-/zbll/blob/e57eaef6b64c5eaf4bbafe3256eacd65a5e7a4bb/src/stores/SessionStore.js#L76-L97
[51]: https://rouxcubing.com/#/rouxsources?mainTab=1&subTab=6
[52]: https://github.com/tao-yu/Alg-Trainer/blob/cb95945c3e4839ee1e4b058d4a79bc022615b220/js/RubiksCube.js#L1392-L1402
[53]: https://github.com/aricneto/TwistyTimer/blob/2fcd6f7d2697a0810f37245ac033f1da12448382/app/src/main/java/com/aricneto/twistytimer/puzzle/TrainerScrambler.java#L115-L139; averages: `app/src/main/java/com/aricneto/twistytimer/stats/{AverageCalculator,Statistics}.java`, `app/src/main/res/values/integers.xml`, `app/src/test/java/com/aricneto/twistytimer/stats/AverageCalculatorTestCase.java` (`testLargeAverage`) at the same commit
[54]: https://github.com/dominikbeveridge/cubedesk/blob/a13e2fe95db88433a588585bb3a5f77a7cad8c2f/client/components/trainer/Trainer.tsx#L83-L90
[55]: https://github.com/onionhoney/roux-trainers/blob/0129460a5c30082ea0e29b4489623889118078ec/src/lib/Algs.tsx#L395-L405 and `src/reducers/CmllStateM.tsx#L40-L41`; also nauroth/roux-trainers at b85221f, `src/lib/Selector.tsx`, `src/lib/Algs.tsx:385-417`
[56]: https://github.com/wodzik/cube/tree/4bdeb4557cc3dec31cd582f7395537f6b194f65d (`README.md`, `src/pages/TrainersPage.tsx`, `plan-trainer.md`, `src/components/stageGroups.ts`)
[57]: https://cubegrass.appspot.com/block_trainer/ (`block_trainer.js`)
[58]: https://acubemy.com/
[59]: https://acubemy.com/faq
[60]: https://acubemy.com/statistics/insights
[61]: https://acubemy.com/training
[62]: https://www.youtube.com/watch?v=AMtBD3im6ac (sponsored)
[63]: https://apps.apple.com/us/app/id6758110729 and https://www.cuberpal.com/
[64]: https://apps.apple.com/us/app/id6758565472
[65]: https://apps.apple.com/us/app/id6767834369
[66]: https://enbyne.tech/tools/EOLRb and https://enbyne.tech/tools/laser

Other domains
[67]: https://kovaaks.com/webapp-backend/playlist/playlists?page=0&max=3
[68]: https://kovaaks.com/kovaaks/playlists
[69]: https://steamcommunity.com/app/824270/discussions/0/3774617156468221911
[70]: https://steamcommunity.com/app/824270/discussions/0/2248931052754715779
[71]: https://app.voltaic.gg/blog/announcing-the-voltaic-season-5-aiming-benchmarks-beta-for-kovaaks/
[72]: https://aimlabs.com/articles/aimlabs/how-to-create-and-edit-playlists-on-aimlabs/
[73]: https://support.aimlabs.com/hc/en-us/articles/12140984985623-How-do-I-share-my-custom-playlist (read via https://support.aimlabs.com/api/v2/help_center/en-us/articles/12140984985623.json)
[74]: https://aimlabs.com/articles/aimlabs/the-aimlabs-official-benchmarks-first-season-is-now-live/
[75]: https://aimlabs.com/articles/aimlabs/what-does-your-aimlabs-benchmarks-rank-actually-mean/
[76]: https://aimlabs.com/articles/category/benchmarks/
[77]: https://store.playstation.com/en-is/concept/10003284
[78]: https://steamcommunity.com/app/714010/negativereviews/?browsefilter=toprated&filterLanguage=english
[79]: https://steamcommunity.com/app/824270/negativereviews/?browsefilter=toprated&filterLanguage=english
[80]: https://www.keybr.com/help
[81]: https://github.com/aradzie/keybr.com/tree/05a37bc5265f65ff538c59c613db29442f345d51 (`packages/keybr-lesson/lib/{guided,target,settings,dailygoal,learningrate,learningsession}.ts`, `packages/keybr-result/lib/keystats.ts`, `packages/keybr-math/lib/filter.ts`, `packages/page-help/lib/HelpPage.tsx`, `LICENSE`)
[82]: https://groups.google.com/g/keybr/c/b4D1xFvS-48
[83]: https://monkeytype.com/about
[84]: https://github.com/monkeytypegame/monkeytype (`frontend/src/ts/test/practise-words.ts`, `frontend/src/ts/test/weak-spot.ts`, `frontend/src/ts/components/modals/ShareTestSettings.tsx`, `packages/schemas/src/users.ts`)
[85]: https://itunes.apple.com/lookup?id=1351617981
[86]: https://www.modacity.co/
[87]: https://ensembleschools.com/blog/apps-for-musicians/
[88]: https://support.melodics.com/en/articles/6777027-practice-mode
[89]: https://support.melodics.com/en/articles/15091765-understanding-your-progress-in-melodics
[90]: https://support.melodics.com/en/articles/15091931-stages
[91]: https://support.melodics.com/en/articles/15092017-lesson-scoring
[92]: https://support.melodics.com/en/articles/15091960-weekly-goals
[93]: https://support.melodics.com/en/articles/16979766-weekly-streaks
[94]: https://support.yousician.com/hc/en-us/articles/201558312-How-to-navigate-in-Yousician
[95]: https://support.yousician.com/hc/en-us/articles/206912609-The-best-way-to-practice-guitar
[96]: https://support.yousician.com/hc/en-us/articles/204796991-Practice-and-Play-modes-in-piano
[97]: https://itunes.apple.com/us/rss/customerreviews/page=1/id=959883039/sortby=mostrecent/json
[98]: https://help.strongapp.io/article/105-about-templates
[99]: https://help.strongapp.io/article/107-organize-templates
[100]: https://help.strongapp.io/article/177-update-template
[101]: https://help.strongapp.io/article/109-share-workout-or-template
[102]: https://itunes.apple.com/us/rss/customerreviews/page=1/id=464254577/sortby=mostrecent/json
[103]: https://help.hevyapp.com/hc/en-us/articles/35119778922263
[104]: https://www.hevyapp.com/features/share-folders-routines/
[105]: https://help.hevyapp.com/hc/en-us/articles/36011518408983-How-to-Access-and-Use-Hevy-s-Routine-and-Program-Library
[106]: https://help.hevyapp.com/hc/en-us/articles/38387296276375-Update-Routine-vs-Keep-Original-Routine
[107]: https://help.hevyapp.com/hc/en-us/articles/38385724273047-Hevy-Trainer-Explained-How-It-Builds-Your-Workout-Program
[108]: https://help.fitbod.me/hc/en-us/articles/360004429814-How-Fitbod-Creates-Your-Workout
[109]: https://help.fitbod.me/hc/en-us/articles/42333470514455-Why-did-my-workout-change-Understanding-workout-refreshes
[110]: https://itunes.apple.com/us/rss/customerreviews/page=1/id=1041517543/sortby=mostrecent/json
[111]: https://support.chess.com/en/articles/10319322-how-does-the-spaced-repetition-scheduling-work and https://support.chess.com/en/articles/10319267-when-will-the-variations-be-ready-for-my-review
[112]: https://www.chess.com/news/view/announcing-chessable-3
[113]: https://www.chess.com/forum/view/general/chessable-7
[114]: https://www.zwischenzug.gg/p/how-to-use-chessable
[115]: https://itunes.apple.com/us/rss/customerreviews/page=1/id=1523279049/sortby=mostrecent/json and https://itunes.apple.com/lookup?id=1523279049
[116]: https://aimchess.com/
[117]: https://apps.apple.com/us/app/aimchess-learn-chess-online/id1524941307 and https://itunes.apple.com/lookup?id=1524941307
[118]: https://itunes.apple.com/us/rss/customerreviews/page=1/id=1524941307/sortby=mostrecent/json
[119]: https://www.chess.com/news/view/chesscom-acquires-pmg
[120]: https://github.com/lichess-org/lila/tree/ffa4be8221645f143b05948dd591545f2b004225 (`modules/puzzle/src/main/{PuzzleTheme,PuzzleDifficulty,PuzzlePath,PuzzleDashboard,PuzzleReplay,PuzzleFinisher}.scala`, `modules/rating/src/main/Glicko.scala`, `translation/source/puzzle.xml`, `conf/routes`, `COPYING.md`)
[121]: https://github.com/ankitects/anki/blob/main/rslib/src/deckconfig/mod.rs
[122]: https://docs.ankiweb.net/deck-options.html
[123]: https://docs.ankiweb.net/leeches.html
[124]: https://docs.ankiweb.net/filtered-decks.html
[125]: https://docs.ankiweb.net/exporting.html and https://docs.ankiweb.net/importing/packaged-decks.html
[126]: https://forums.ankiweb.net/t/restoring-number-of-reviews-to-previous-back-up/45566, https://forums.ankiweb.net/t/havent-practiced-in-ages-1700-reviews-waiting/25107, https://forums.ankiweb.net/t/how-to-shift-anki-due-cards-back-to-resume-reviews-as-if-no-days-were-missed/63114
[127]: https://github.com/open-spaced-repetition/awesome-fsrs/wiki/ABC-of-FSRS

Algorithms and platform
[128]: https://github.com/open-spaced-repetition/ts-fsrs/tree/b69d6ca2dca55579198c913fd55a43d6bf703b34 (`packages/fsrs/CHANGELOG.md`, `docs/src/en-US/guide/model/fsrs-7.mdx`, `docs/src/en-US/guide/migration.mdx`, `packages/fsrs/src/models/fsrs-7/constants.ts`) and https://registry.npmjs.org/ts-fsrs
[129]: https://github.com/open-spaced-repetition/fsrs-rs/tree/3be7d2af54fd186fc6a5cbb4e8c393bc9c19ca4e and https://crates.io/api/v1/crates/fsrs (crate 6.6.2: https://static.crates.io/crates/fsrs/fsrs-6.6.2.crate)
[130]: https://github.com/open-spaced-repetition/awesome-fsrs/wiki/The-Algorithm
[131]: https://github.com/open-spaced-repetition/py-fsrs/blob/9446cb06605c597a063aeee49f7d188d42e34dc2/fsrs/scheduler.py and https://pypi.org/pypi/fsrs/json
[132]: https://github.com/open-spaced-repetition/srs-benchmark/blob/main/README.md (fetched 2026-10-28)
[133]: https://github.com/open-spaced-repetition/fsrs-optimizer/blob/ac2a82d222c4ea809985236e2a52e058da524c40/LICENSE
[134]: https://github.com/ankitects/anki/blob/main/Cargo.toml and https://github.com/ankitects/anki/blob/main/LICENSE
[135]: https://github.com/ankidroid/Anki-Android-Backend
[136]: https://github.com/open-spaced-repetition/awesome-fsrs/blob/0ffe54907988a32500b09007090637fd85b90e04/README.md
[137]: https://github.com/open-spaced-repetition/FSRS-Kotlin/tree/670f2f895f316a9f78dfd19b991d9bc523c9b037
[138]: https://github.com/open-spaced-repetition/android-fsrs/tree/04be511631681b20b6934e72cd94ac25060adcb3
[139]: https://github.com/open-spaced-repetition/rs-fsrs-java/tree/6afdf42a42b131f3230b61aceba1cbf0d7b66f26
[140]: https://github.com/therockyt/fsrskt (Maven `one.felsen.fsrskt:fsrskt:1.0.1`)
[141]: https://github.com/jwbargsten/fsrs4s (Maven `org.bargsten:fsrs4s_3:0.3.0`)
[142]: https://www.supermemo.com/en/blog/application-of-a-computer-to-improve-the-results-obtained-in-working-with-the-supermemo-method
[143]: https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.ewm.html
[144]: Pelánek 2016, Computers & Education 98:169-179, doi:10.1016/j.compedu.2016.03.017: https://www.fi.muni.cz/~xpelanek/publications/CAE-elo.pdf
[145]: Pelánek and Řihák 2018, doi:10.1080/13614568.2018.1476596: https://www.fi.muni.cz/~xpelanek/publications/nrhm-mastery.pdf
[146]: https://www.itl.nist.gov/div898/handbook/pmc/section3/pmc323.htm, https://www.itl.nist.gov/div898/handbook/pmc/section3/pmc3231.htm, https://www.itl.nist.gov/div898/handbook/pmc/section3/pmc324.htm
[147]: http://www.glicko.net/glicko/glicko2.pdf
[148]: https://github.com/goochjs/glicko2/tree/5f39e38539bc292267905acbe1dc964b9c173927
[149]: https://github.com/lichess-org/scalachess (`LICENSE`, `rating/src/main/scala/glicko/impl/README.md`)
[150]: Shea and Morgan 1979, doi:10.1037/0278-7393.5.2.179: https://gwern.net/doc/psychology/spaced-repetition/1979-shea.pdf
[151]: Heathcote, Brown and Mewhort 2000: https://link.springer.com/article/10.3758/BF03212979
[152]: https://en.wikipedia.org/wiki/Hick%27s_law
[153]: Gelman et al., BDA3, §2.5: http://www.stat.columbia.edu/~gelman/book/BDA3.pdf
[154]: https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.theilslopes.html
[155]: https://commons.apache.org/proper/commons-math/javadocs/api-3.6.1/org/apache/commons/math3/stat/correlation/KendallsCorrelation.html
[156]: Efraimidis and Spirakis 2006, Information Processing Letters 97:181-185, doi:10.1016/j.ipl.2005.11.003; survey: https://arxiv.org/pdf/1012.0256
[157]: Auer, Cesa-Bianchi and Fischer 2002: https://homes.di.unimi.it/~cesabian/Pubblicazioni/ml-02.pdf
[158]: Greenacre et al. 2023, Statistical Science 38(3), doi:10.1214/22-STS880: https://arxiv.org/pdf/2201.05197
[159]: https://www.worldcubeassociation.org/regulations/ (version 2026-04-01: 9f1, 9f8, 9f9, 10e3, 10e4, A3a1, A4d1, A4d2)
[160]: https://developer.android.com/reference/android/os/SystemClock
[161]: https://developer.android.com/reference/android/view/MotionEvent
[162]: https://developer.android.com/reference/android/view/Choreographer.FrameCallback and https://android.googlesource.com/platform/frameworks/base/+/refs/heads/main/core/java/android/view/Choreographer.java
[163]: https://github.com/afedotov/gan-web-bluetooth/tree/65173e2cdd0fa38ef384f237f6da8d76c177ed1e (`src/gan-cube-protocol.ts`, `src/utils.ts`, `README.md`)
[164]: https://github.com/dadadadas111/speedcube-trainer/tree/6e7606d5dc6d0e9a8c3fb1f4d9394bce0c7926d4 (`README.md`, `src/cube/moveStream.ts`, `src/analysis/solve.ts`, `src/analysis/recommend.ts`; no licence)
[165]: This repo: `README.md` (licence) and `docs/HANDOFF.md` (decisions)
[166]: This repo: `docs/research/roux-training-features.md`, `docs/research/timer-integration.md`
[167]: https://github.com/hato-ya/CubicTimer/tree/cfdf5f2b1d7a3a10c319fcd216abffc9291a82b5 (`app/src/main/java/com/hatopigeon/cubictimer/stats/{AverageCalculatorSuper,AverageCalculator,Statistics}.java`, `app/src/main/res/values/integers.xml`; GPL-3.0)
[168]: https://registry.npmjs.org/@wodzik/cubecore/-/cubecore-0.1.13.tgz (`@wodzik/cubecore` 0.1.13, published 2026-09-30: `roux/index.js`, `roux/index.d.ts`, `LICENSE`; MPL-2.0; repo https://github.com/wodzik/cubecore, credited in the wodzik/cube README [56])
