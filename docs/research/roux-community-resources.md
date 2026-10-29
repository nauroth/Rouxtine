# Roux community resources: what Rouxtine should contain

Researched 2026-10-28 from rouxcubing.com, resources.rouxers.com, the Roux Reader (book.rouxers.com), the Roux Tutorial (tutorial.rouxers.com) and every resource they link. Purpose: decide what Rouxtine should contain (drills, lessons, alg sets, tools) so that Roux learners at every level can improve, and what each piece may legally reuse. onionhoney's trainers are already known, so the focus is on what goes beyond them or differs from them. `[n]` refers to the numbered sources at the bottom.

Conventions:
- A bare `src/...:N` is a line in [nauroth/roux-trainers](https://github.com/nauroth/roux-trainers) (the fork of onionhoney/roux-trainers that Rouxtine ports from) at `0129460`, v2.3.
- **FM** = section 2 of [roux-training-features.md](roux-training-features.md) (the accepted feature map) [136]. **TR** = `docs/research/training-regimens.md` in nauroth/rouxtine [138]. **HANDOFF** = `docs/HANDOFF.md` there [137].
- Levels used throughout: **L0** first-time solver, **L1** switching from a beginner method or CFOP, **L2** beginner Roux, **L3** intermediate, **L4** advanced, **L5** expert (defined in 3.2).
- Method: pages were rendered with Playwright, Google Docs and Sheets were exported (including hidden tabs), git repos were cloned read-only, and licences were checked in repos and footers. YouTube videos were **not watched**: only their titles and channels (via oEmbed) and where they are linked were used, so what a video covers is inferred from those.

## 1. Summary

- **The hubs are link directories, not courses.** rouxcubing.com (Knut Peterson) has tutorials, a 26-tab link library, a 25-term glossary, a weekly Roux-only competition and the **Roux Techniques Table**, a 4-level by 4-step ladder [1][2][4][5]. resources.rouxers.com is a single curated link page [7]. rouxcubing.com's own text tutorial and "Text Roadmap" both say "Coming soon!" [2][3].
- **Nobody has finished a beginner or a "what to learn when" text.** The Roux Tutorial's beginner FB stops at "What is a pair?", its beginner SB page is empty, and its planned third part (when to learn what) was never written [21][28][30]. About half of the Roux Reader's headings are empty and it has not changed since 2022-01-15 [9][12][17]. Rouxtine can own this gap: a complete beginner course plus a level roadmap.
- **Only a little content may be embedded.** The Roux Reader is MIT [17], Tronto's FMC tutorial is CC BY 4.0 [126], and Tao Yu's Alg-Trainer code is MIT [73]. Every Google Doc and Sheet (see the licence column in 6.1), rouxcubing.com, resources.rouxers.com, the tutorial, rouxl.es, athefre's site, Kian's site, SpeedCubeDB, Scheopner and cubinghistory state no licence, so they stay all rights reserved [6][8][29][37][38][99][100][102][125]. Write our own text, regenerate algs with the solver or use onionhoney's GPL data, and link out.
- **Recognition is the time sink even at world level.** In two public Cubeast ao100s (5.58 s and 6.46 s), CMLL recognition took 0.61–0.69 s, against 0.64–0.69 s of execution, and total recognition was 1.19–1.46 s per solve [105][106]. Recognition and prediction drills deserve priority.
- **Biggest drill gaps against onionhoney** (sections 4 and 9):
  - Beginner drills: Sune angle, headlights, EO counting and the EO flowchart, 4b [22][23][76].
  - A full-LSE free-solve trainer with an efficiency score [84].
  - DFDB 4c prediction questions during 4b [88][89].
  - CMLL recognition systems with timing: all-angle, NMCMLL/ATCRM, Straughan four-sticker, three-corner [14][60][64][101].
  - CMLL→LSE effect and EO prediction [52][48].
  - An SB last-pair case trainer on the community's 65-case taxonomy, front and back slot [38][39].
  - FB+DR tier feedback and "find 2+ solutions" [31], and framework depth gates [33].
  - Linear Roux FMC, plan-then-reveal pro-solve study, and a weekly-competition mode [1][117][118].
- **Data model.** Sources disagree on case names:
  - CMLL is keyed as group plus slot (O3, H1, Pi1 … L6) in most 2H sheets [45][46][53]. Gilles Roux's original A–H × 1–6 grid [121], SpeedCubeDB names [112], Atlas names [50] and roux-trainers ids differ.
  - EO groups are called 3/1, Arrow and 3LRF-1F in different sources [78][83][84].
  - Rouxtine needs canonical case ids plus alias maps (6.3).
- **Handedness matters.** EOLR sheets exist for right-hand and left-hand M [79][80]. The EO flowchart has lefty and righty halves [76]. CMLL lists differ by LSE handedness [47] and by OH hand [53][54].
- **OH is Roux's strongest event.** Roux solvers hold 34 % of the WCA OH average top 100 against 6 % for 3x3 [1], and Roux mains' median OH average is 20.4 s against 31.2 s for CFOP mains (small sample) [115]. Treat OH as a profile across all modules, not a separate app area.
- **Defects found:**
  - onionhoney v2.3 hides the tracking trainer: its tab is commented out, so the Roux Reader's `#tracking` link falls back to the default mode (`src/components/AppView.tsx:83`, `:86-99`) [11].
  - LaSEr's misoriented-centre setting is broken, and the cause was found [84].
  - rouxcubing's "CMLLEO" item opens `localhost:3000`, and two table items open nothing [2].
  - The SB Last Slot sheet is deleted (HTTP 410) [36].
  - Several links in the Reader and the tutorial return 404 [9][15][20].

## 2. Source inventory

### 2.1 Hubs

| Hub | Sections | Licence / terms | What it covers | State |
|---|---|---|---|---|
| **rouxcubing.com** [1]–[6] (Knut Peterson; React app on GitHub Pages) | Home (What is Roux, competitiveness charts, weekly comp); Rouxtorials (Roux for Beginners; Roux for Experienced Cubers); Rouxsources: F2B (4 sub-tabs), CMLL (7), LSE (6), Big Cubes (3), Misc (6); Rouxdmap to Sub-X (video roadmap, Text Roadmap, Roux Techniques Table) | No licence. Footer "Copyright © RouxCubing.com 2026". The source repo has no LICENSE and `package.json` is private [6]. robots.txt grants nothing. Contact: Discord `@_knut_` [1] | 216 distinct URLs site-wide, most of them in the 26 sub-tabs [4]; a 25-entry glossary [5]; 41 Rouxer YouTube channels [4]; a weekly Roux-only comp (3x3, OH, FMC, 4x4) [1] | Deployed 2026-10-26. Text tutorial and Text Roadmap are "Coming soon!"; a Sub-60…Sub-8 roadmap stepper exists only in code [2][3]. Defects: SBLP and SBLP+CO open nothing; CMLLEO opens localhost; the 2-look sheet is labelled "OH CMLL" [2][4] |
| **resources.rouxers.com** [7][8] (Anto Kam, eff; Hugo) | One page since 2026-09-07: General Resources, Beginner Tutorials, First Block, Second Block, CMLL, Last Six Edges (with an EO flowchart image, a DFDB written guide image and EOLR docs), Miscellaneous ("unexplored algsets/techniques"), Big Cubes; Important General Stuff: Speedsolving Advice, Competition Advice, Fingertricks, Reconstructions, Roux Lore, Contributing | No licence file at HEAD or in history; theme hugo-book is MIT [8] | 47 unique article links, short notes, two images copied from rouxl.es (DFDB) and the tutorial repo (EO flowchart), byte-identical to them [7][89][29] | Last change 2026-09-11 [8] |
| **The Roux Reader** [9]–[18] (Zhouheng Sun, Antonio Kam; mdBook) | Introduction, Overview, Blockbuilding (FB, SB, FB+DR), L10P (CMLL, LSE), Hardware, Resources for Beginners, Tips on Switching from CFOP, FAQ, Contributors; Chinese edition | **MIT**, "Copyright (c) 2020 Rouxers" [17]. The bundled TwistySim widget has no licence header [17] | Substantive chapters: FB (pair relations, influencing with 17 examples, taste, planning) [11]; orientation/CN [10]; DR-first SB [12]; CMLL recognition progression [14]; EOLRb insertions and 8 non-cycle influencing cases [15]; milestones FAQ [16] | Frozen since the 2022-01-15 deploy [17]. Stubs: FB+DR, Hardware, Beginner Resources, most SB/CMLL/LSE subsections; the Switching page returns 404 [9][12][14][15]. The Chinese edition adds a 4-strategy FB taxonomy and colour-neutrality terms [18] |
| **The Roux Tutorial** [19]–[31] (Antonio Kam; mdBook; companion to a UC Berkeley DeCal course) | Beginners: Notation, FB, SB, CMLL, LSE. Intermediate: FB, SB, CMLL, LSE | **No licence** in the repo or its history [29]; the predecessor repo also has none [30] | 12 pages with 69 interactive cubes. Finished parts: notation [20], beginner CMLL [22], beginner LSE [23], intermediate FB [24] and SB [25] | Last deploy 2021-10-29 [29]. Beginner FB stops at pairs [21]; beginner SB is empty [28]; intermediate CMLL is three links [26]; intermediate LSE is one sentence [27]; no advanced part [19]; the beginners landing page links Notation with a broken relative URL (404), though the page itself works [20] |

### 2.2 Other sites deep-read (alg sheets are in section 6, tools in section 7, videos in section 8)

| Source | Kind | Covers | Licence |
|---|---|---|---|
| Kian Mansour's Roux Method Guide [37] | Site + 5 docs | Beginner playlist, SB algs, CMLL (EO-preserving and EO-flipping alg per case), OH CMLL (LH/RH), EO, EOLR (~60 intuitive cases) | None stated |
| rouxl.es (Anto Kam) [99] | Hub + 3 images | OH CMLL progression; LSE progression (EO flowchart → LR edge to D during EO → DFDB → EOLR); MC DFDB | None (repo has no LICENSE) |
| athefre (Michael James Straughan) [100][101][59][65][66] | Site | CMLL prediction, OCLL prediction, intuitive CP, NMCMLL, ACMLL, ATCRM, Straughan four-sticker recognition, EOLR and DFDB history | None |
| cubinghistory.com Roux page [102] | History | Roux timeline with inventors and dates, 48 footnotes | "Copyright (c) 2026 Michael James Straughan"; repo has no LICENSE |
| Trevor Scheopner [125] | Tutorial + 3 solvers | Novice/Intermediate/Advanced per step, optimal movecount statistics, CMLL standard-sticker recognition, LSE case codes | None |
| Gilles Roux's original pages [121] | Primary source (c. 2003–2010) | Method design, A–H × 1–6 CMLL grid, SB tables, LSE 4a/4b/4c, 28 EO shortcuts, time-budget movecounts | None |
| Mark Fiend [122], Ruwix [123] | Tutorials | CMLL nicknames and recognition cues; Ruwix's beginner LSE goes DF/DB first then EPLL | None; Ruwix is commercial |
| r/Cubers Megasurvey 6 [115] | Survey | Roux share, learning curve, neutrality, inspection depth, OH, alg learning, smart cubes, timers | None stated |
| r/rouxcubing [118] | Community | Wiki beginner and advanced pages, 2022 resource thread, user requests | Reddit terms |
| Tronto, FMC tutorial §2.1 [126] | Tutorial | Blockbuilding principles; Roux tips for FMC | **CC BY 4.0** |
| basilio.dev, Bas's guides and alg sheets [51][77] | Cheat sheets | CMLL and EOLR one-page sheets | None |
| SpeedCubeDB [38][63][112], reco.nz [103], cubesolv.es [110], Cubeast shares [105][106], cubedb.net [113], VisualCube [114] | Databases/tools | See sections 6–7 | See section 7 |

### 2.3 Blocked or inaccessible

- speedsolving.com wiki (Roux method, CEOR, Conjugated CxLL): Cloudflare challenge; not routed around [42][72][98].
- Reddit: the BU 4c guide and r/rouxles were blocked by Reddit's network security [90][104]. r/rouxcubing rendered [118].
- Roux Facebook group: login wall [119].
- cubingmethods.com: Vercel bot checkpoint, HTTP 429 [120]. rubiks.fandom.com and dune.fandom.com: Cloudflare challenge [124][128].
- SB Last Slot sheet: deleted, HTTP 410 [36]. SpeedCubeDB reconstruction index: HTTP 404 [111].
- Ello move GIF: dead host [133]. VisualCube's own host returns an error instead of images [114].
- GitHub web pages and API for third-party repos: blocked by the session proxy; anonymous `git clone` worked and was used [8][17][29].
- rouxers/tutorial's 3 open issues: not readable [29].
- Louis' FB playlist: oEmbed 403, title unknown [11]. archive.org: unreachable from this session.
- YouTube: titles only, no video watched (section 8).

### 2.4 Not deep-read on purpose

- Discord invites: Roux Method Speedsolvers (`discord.gg/VJunGCZBGZ`, linked from both hubs) [135], One-Handed Solvers, and an old invite `discord.gg/B6GmMgq` in the hub's history [1][7][8].
- onionhoney's own pages (`#fb`, `#cmll`, `#eopair`, `#ss`, `#fbdr`, `#tracking`), linked from all four hubs [4][7][11][16][25].
- The rouxers GitHub repos (book: MIT; tutorial: no licence; resources: no licence), read only for licence and history [8][17][29].

### 2.5 Non-Roux links on the hubs (filter them)

- Trainers for other methods' sets: Roman Strakhov's ZBLL trainer and most of Tao Yu's sets [4][73][75].
- Other methods: CEOR, YruRU and Nautilus L5E (listed under Misc Blockbuilding and Misc LSE) [4][41][92]. The hub's "CEOR (to do CPFB instead of FB, and COLS instead of SBLP)" sheet is a CEOR alg compilation; only its CP-line and F-move CP-tracing tabs touch Roux (CP-first OH) [7][143].
- Site tooling links (Hugo Book theme and its shortcode docs, mdBook releases, GitHub's pull-request docs, MathJax, Travis CI): no Roux content [8][17].
- General or CFOP-based advice (filtered: not in section 8, not shown in the app): Dylan Miller's lookahead guide (CFOP, "principles transfer"), Antoine Cantin's F2L planning video (the hub's link text says "planning F2B and beyond is completely doable"), Patrick Ponce's OH turning guide [4][7].
- Mindset links: the Dune litany and a Wikipedia parable [7].
- Commercial: the weekly comp is sponsored by SpeedCubeShop (discount code, a gift-card draw) [1]; the sponsor link now lands on the store's homepage [134].
- Atlas's sheet hides ZBLL, ZZLL, COLL and Phasing tabs [50].
- The Roux Reader's L10P page compares CMLL with OLL+PLL, and one chapter is "Switching from CFOP" [13][9].
- CFOP analogies (research only, not for app text): rouxl.es says FB "is quite similar to planning a cross" and that with FB+DR planned, SB "is usually quite similar to doing 2 CFOP pairs" [99]; Bas's CMLL sheet marks about 12 algs that are also standard OLL algs [51]. Section 3 keeps only neutral wording.

## 3. Curriculum by level

### 3.1 The Roux Techniques Table (rouxcubing.com)

Reproduced exactly from rouxcubing.com, Rouxdmap to Sub-X, by Knut Peterson, © RouxCubing.com 2026 [2]. The site calls it "A loose guide for when you should learn different Roux techniques." Items in one cell are separate rows on the site.

| | First Block | Second Block | CMLL | LSE |
| --- | --- | --- | --- | --- |
| Basic | Square/line blockbuilding <br> Inspect full FB | Basic SB algs | Algs <br> Execution | EO flowchart |
| Intermediate | Inspect FB+DR | SBLP <br> Multi-inserts | Multi-angle recognition | EOLR <br> DFDB <br> Misoriented centers <br> Dots skipping |
| Advanced | Inspect FB+SS <br> Nonlinear blockbuilding | SS Blockbuilding | CMLLEO | EOLRb <br> 4c Influencing <br> ULUR/UFUB Neutrality |
| Niche | Inspect F2B <br> CPFB <br> Nonmatching centers | Non-matching blocks <br> SBLP+CO (after CPFB) | Pinkie Pie <br> NMCMLL/42 <br> ACMLL/TCMLL | Nautilus L5E |

Notes on the table [2]:
- Each item opens a Rouxsources sub-tab. "SBLP" and "SBLP+CO (after CPFB)" open nothing.
- "CMLLEO" opens a leftover `http://localhost:3000` URL whose intended target is CMLL > CMLL Recognition. That tab's EO material is Zhouheng Sun's video on predicting LSE EO from CMLL [4]. In the wider community, CMLLEO (first called KCLL, Thom Barlow, about 2010) means choosing CMLL algs that also orient LSE edges [102].
- "Nautilus L5E" belongs to a different method, Nautilus [92].
- The table appears in the app only if Knut agrees (section 10). Rouxtine can build its own ladder anchored on the same technique names.

### 3.2 Levels

| Level | Who | Typical speed (3x3 average) | Anchors |
|---|---|---|---|
| L0 First-time solver | Has never solved a cube | first solve | Hub: Roux is "a fairly good beginner method" [7]; tutorial beginner track [19]; Kian's and DeeDubb's beginner videos [3] |
| L1 Switcher | Solves with a beginner method or CFOP | any | rouxcubing "Roux for Experienced Cubers" [3]; tutorial intermediate track [24][25]; rouxl.es FB and SB pages [99] |
| L2 Beginner Roux | Solves with Roux | about 60 → 30 s | Techniques Table "Basic" [2]; Scheopner Novice [125]; Megasurvey: Roux mains' median 38.7 s under 3 months, 32.5 s at about 6 months [115] |
| L3 Intermediate | Full CMLL under way | about 30 → 15 s | Scheopner Intermediate "once you are sub-30 or so" [125]; full CMLL at the latest by 20–30 s [16]; Techniques Table "Intermediate" [2] |
| L4 Advanced | EOLR, FB+DR | about 15 → 10 s | EOLR from about 15 s [16]; Scheopner Advanced "once you are sub-20 or so" [125]; FB+DR guide "realistically sub-12" [31]; Techniques Table "Advanced" [2] |
| L5 Expert | Prediction, variants | sub-10 | Techniques Table "Niche" [2]; hub "Miscellaneous" (unexplored sets) [7]; Kian's Sub-X series, linked as the video roadmap, runs from Sub-20 to Sub-8 [2][138] |

Speeds are guides only. rouxl.es: "Learn the next one when you feel you're ready to - no need to wait for sub-x" [99]. The survey found that Roux mains improve more slowly than CFOP mains in the first year (32.5 s against 22.5 s at about 6 months) and catch up by about 1.5 years ("the learning curve is harsher for Roux") [115].

### 3.3 Stages by level

| Level | Inspection / FB | SB | CMLL | LSE |
|---|---|---|---|---|
| L0 | Cube anatomy, fixed centres, notation incl. M ("follows L") and r [20]. FB as a 1x2x3 on L with one fixed orientation; DL edge first (5 worked placements), then pairs [21]. Noblion's sheet: 6 pictured FB insertions [32] | DR edge, then pairs with R/r/U [32]. A beginner pair system: corner location × destination [34]. Scheopner novice: DR then pairs, open-slot / insert / close-slot, under 22 moves [125]. The tutorial page is empty [28] | Two algs. Sune repeated for orientation (reduce to one oriented corner, or simulate Sune from all 4 angles), then T-perm with headlights on the left [22]. Noblion uses Sune + J-perm [32]. r/rouxcubing wiki: 2 algs, then 9, then 42 [118] | 4a: good/bad edges against U/D centres; arrow + `M' U2 M` swap; count bad edges to choose the angle. 4b: both L/R edges to D, align, `M2`. 4c: fix the centre, then swaps [23]. Noblion: a 6-case 4c table plus a 3-cycle rule [32] |
| L1 | x2y at least; solvers who are already colour-neutral go fully CN [24]. Four FB approaches: DL first, line-line square + pair, S-move insert, line-line [24]. Plan the whole FB in inspection; it takes getting used to [99] | `<R,r,U,M>`, avoid F at first; wide inserts `r U r'`, `r U' r'`; DR first "unless a pair presents itself" [25]. With FB+DR planned, SB is usually two pair insertions, unless a blockbuilding solution appears [99] | 2-look as a stopgap, then full CMLL [26]. Some CMLL algs are last-layer algs the user may already know; Bas's sheet marks about 12 [51] | Same as L0; the intermediate LSE page is a stub [27]; Donovan's tutorial [3] |
| L2 (Basic) | "Square/line blockbuilding; Inspect full FB" [2]. Square+pair vs line+line; pair relations (edge position/orientation, corner position, D-sticker) [11]. Scheopner novice: L centre then two pairs, under 13 moves, use extra inspection time [125]. Survey: 65 % of Roux mains always plan FS or a line, 16 % always plan the full FB [115] | "Basic SB algs" [2]: inserts and M-pairings `R U' M' U r'`, `r U' M U R'` [25]; DR-first as default with freestyle shortcuts [12] | "Algs; Execution" [2]. Full CMLL "as soon as possible", at the latest at 20–30 s [16]. 1.5-look as a bridge [58]. Recognition stage II: fixed angles with U moves, peeking [14]. Learning order H, Pi, Bowtie, Headlights, Chameleon, Sune, Antisune [125] | "EO flowchart" [2][76]. Then track L/R edges during EO and force one to D, for pauseless 4a→4b [99] |
| L3 (Intermediate) | "Inspect FB+DR" [2]. Influencing, FB taste, FS-first then LP prediction [11]. Frameworks: master depth 3 to 90 % and depth 4 to 50 %, then real solves [33]. Scheopner intermediate: "3 pairs of 2" or "2 pairs of 3", under 10 moves, plan the first square or line [125] | "SBLP; Multi-inserts" [2]. SBLS 65 cases [38]; pair choice and multislotting [4]; recognise pairs by the square's L/R stickers [34] | "Multi-angle recognition" [2]. All-angle stage III with hidden-sticker inference [14]; L-case recognition [61]; per-set angle systems [60] | "EOLR; DFDB; Misoriented centers; Dots skipping" [2]. EOLR after full CMLL, from about 15 s [16]; learn it from descriptions [15], "intuitively, utilizing alternative solutions" [7]; "Don't avoid misoriented centers" [99] |
| L4 (Advanced) | "Inspect FB+SS; Nonlinear blockbuilding" [2]. FB+DR: DR tier list and influencing techniques [31]; FB+SS and FB+SB are planned "only if easy" by 91–93 % of Roux mains [115] | "SS Blockbuilding" [2]. F-move multislotting [35]; tracking-friendly last-pair choices [100] | "CMLLEO" [2]: EO prediction from CMLL [4]; EO-preserving vs EO-flipping alg variants [37] | "EOLRb; 4c Influencing; ULUR/UFUB Neutrality" [2]. EOLRb 4c non-cycle influencing [15]; 6-flip repertoire [4]; non-`<M,U>` EOLRb [85] |
| L5 (Niche) | "Inspect F2B; CPFB; Nonmatching centers" [2] | "Non-matching blocks; SBLP+CO (after CPFB)" [2]. Passive blocks for ACMLL [66]; twisted inserts for TCMLL [68] | "Pinkie Pie; NMCMLL/42; ACMLL/TCMLL" [2]. Straughan, ATCRM and three-corner recognition [101]; OCLL and CMLL prediction during the last SB pair [59] | "Nautilus L5E" [2] (another method; see 10). LRb backtracking, 4c^3 [4]; EOBF [15][142] |

### 3.4 Cross-cutting skills by level

| Level | Lookahead | TPS and fingertricks | Colour neutrality | Efficiency | OH | Competition |
|---|---|---|---|---|---|---|
| L0 | n/a | Notation; M direction takes time to learn [20] | One fixed orientation (blue L, green R) [21] | n/a | n/a | n/a |
| L1 | Carries over | M moves are new; `F`/`B`/`r2` inserts replace slot habits [24] | x2y minimum; full CN if already CN [24] | — | — | Weekly comp is open to all Roux solvers [1] |
| L2 | Treat solve steps "as a transition to the next one rather than a binary switch" [99]; "calm and fluid turning" in FB, to leave time to track SB pieces [11] | `<R,r,U,M>` inserts [25] | Gilles: one colour set first, then neutral [121]; Scheopner: neutrality only once FB is under about 9 moves [125]; Reader: x2y "highly recommended" [10] | Scheopner novice: FB under 13, SB under 22 [125] | Fingertrick videos (Anto, Kian) [4]; rouxl.es OH CMLL path: F/Fw/D tricks → 2-look → full → full with eido [99] | Weekly comp with 3x3, OH, FMC and 4x4 events [1] |
| L3 | FB is planned statically, SB is tracked dynamically [10]; pauseless 4a→4b→4c [99]; tracking trainer and partial SpeedBLD as warm-up [11] | Jayden McNeill on turning [7]; flamepelt CMLL fingertricks [4]; John Mark's fingertrick text notation [49] | x2y → CN: "worthy of exploring, but don't force yourself" [10]; dual (x2y) is the most common choice (57 %) [115] | Scheopner intermediate: FB under 10, SB under 19 [125]; slow solves and example solves [16]; linear FMC weekly [1] | Roux OH: SBLP front and back slots [39]; OH CMLL sheets [53][54][55] | Charlie Eggins' practice guide [7] |
| L4 | FB+DR planning frees time to inspect SB during FB [31]; plan FB+SS when it is easy, as most Roux mains do [115] | Per-step TPS in two 5.6–6.5 s ao100s: SB about 8–10, CMLL about 15 [105][106] | Optimal FB 6.21 moves fixed → 4.74 full CN [125]; 6.70 → 5.14 HTM [142] | Scheopner advanced: FB under 9, SB under 16 [125]. eff's 50 example solves (FB+SS idealised) average 38.3 STM: FB 6.0, SS+SB 12.0, CMLL 9.8, LSE 10.5 [107] | Table abuse for slice moves [102]; OH LSE and OH dots fingertricks [4] | Anxiety and equanimity links [7] |
| L5 | CMLL prediction during the last SB pair [59][101] | — | Dual x2y ("four-bottom") neutrality, used by Fahmi [18] | Gilles' movecount by thinking time: 15 s ≈ 48, 1 min ≈ 40, 5 min ≈ 37 [121] | OH TCMLL [68]; CP-first RUroux [40] | Sharing over gatekeeping [127] |

### 3.5 Where the sources disagree

- **When to learn full CMLL.** The Reader says as soon as possible, at the latest at 20–30 s [16]; the tutorial treats 2-look as a stopgap [26]; 1.5-look is offered as a bridge [58]. rouxl.es says not to wait for sub-x times [99].
- **2-look corners.** Sune + T-perm [22], Sune + J-perm [32], the 9-alg 2-look (7 OCLL + 2 swaps) [57], or Sune/Antisune + Jb/Y [123].
- **Beginner LSE.** The community path is 4a → 4b → 4c [23][118]. Ruwix teaches DF/DB first, then EO, then M-slice EPLL, and calls its own path "quite inefficient" [123]. Rouxtine should teach 4a/4b/4c.
- **Colour neutrality timing.** The Chinese edition says widen it as early as possible, before habits set [18]. Scheopner says only once FB is under about 9 moves [125]. The Reader says x2y is recommended and full CN is optional [10].
- **CMLLEO.** rouxcubing's (broken) CMLLEO item points at its EO-prediction material and does not define the term; it is not in the 25-term glossary [2][4][5]. Cubinghistory and the r/rouxcubing wiki use it for choosing CMLL algs that orient or influence the LSE edges [102][118].
- **Meyer.** Fengyuan Lou thinks about 95 % of Roux users would do better with another big-cube method [97]. The weekly comp allows only Roux-based 4x4 methods [1].

## 4. Drill catalogue

### 4.1 Proposed modules

Each drill below is either its own **module** (a toggleable training tool, HANDOFF and TR 5.1 [137][138]) or a **setting/mode** of one. Proposed modules (codes used in the table):

| Code | Module | Notes |
|---|---|---|
| BEG | Beginner course | Guided steps (notation, FB, SB, 2-look corners, LSE) with mini-drills |
| FBT | FB trainer | Random-state FB/FS/line, optimal-length levels, solution reveal |
| INS | Inspection planner | Timed inspection, scope ladder, plan entry, hidden-cube modes |
| FLP | FB last pair (+DR) | |
| PAIR | Pair spotting | New, small |
| TRK | Tracking | |
| SS | SB square | |
| SLP | SB last pair | |
| CHN | Chain flow | FS → FBLP → SS → SBLP (FM v1) |
| CMX | CMLL cases | Execution and alg sets |
| CMR | CMLL recognition | Name the case; recognition systems |
| PRD | Prediction | Freeze before the last SB pair or before CMLL |
| EO | EO (4a) | |
| EOLR | EOLR / EOLRb | |
| L4C | 4b / 4c | |
| LSX | LSE sandbox | Full LSE free-solve |
| TIM | Timed solves | Timer, sessions, manual splits |
| SLOW | Slow solves and linear FMC | |
| REC | Reconstruction study | Own and imported solves |
| WK | Weekly comp | Needs the organiser's agreement |
| SC | Smart-cube solves | Automatic splits |
| BIG | Big-cube Roux pack | later |
| CPF | CP-first OH pack | later, scope question |

App-wide profiles that many modules read: orientation set (U/F colour pairs), OH hand, M hand (lefty/righty), move-set profile (2H, OH, no-F), naming scheme, recognition system.

### 4.2 Catalogue

"onionhoney" = already in roux-trainers v2.3. "FM" = already in the feature map [136].

**Beginner and notation**

| ID | Drill | Trains | Lvl | Sources | How an app implements it | Module / setting | onionhoney | FM |
|---|---|---|---|---|---|---|---|---|
| B1 | Move quiz: watch a move, copy it; "which way does M turn?" | Notation, M direction | L0 | [20][113] | Animated move on the app's cube; tap the move that was shown, or do it on a real cube | Mode of BEG | No | No (FM has the notation engine only) |
| B2 | DL-edge and pair insertions, step by step | First FB | L0 | [21][32] | Lesson cards, each with a setup and "try it" on the virtual cube; the engine checks the step | Mode of BEG | No | No |
| B3 | SB pair system: corner location × destination | First SB | L0–L2 | [34][32][125] | Interactive 3×2 grid; then random pair cases, user picks the move family | Mode of BEG | No | No |
| B4 | Sune angle: which U angle leaves one oriented corner | 2-look corners | L0 | [22][130] | Top view; tap the AUF; animate Sune; later "predict all 4 angles" | Mode of BEG | No | Partly (2-look CMLL lesson, MVP) |
| B5 | Headlights: which side, or none | Corner permutation | L0 | [22][131][32] | Top + side strips; tap the side; feedback shows the U move before the alg | Mode of BEG | No | Partly |
| B6 | EO by counting: arrow + swap, choose the angle that gives 4 bad edges | 4a | L0–L2 | [23] | Virtual LSE with big M/M'/U/U' buttons; engine checks EO | Mode of EO ("intuitive") | No (no 4a-only target) | Yes (EO trainer, MVP) |
| B7 | Hidden DB edge inferred from parity | EO recognition | L0–L2 | [122][132][125] | Quiz hides DB and asks its state | Setting of EO | No | No |
| B8 | 4b: both L/R edges to D, align, `M2`; highlight UL/UR | 4b | L0–L2 | [23][84] | Virtual LSE with target 4b and a "highlight L/R edges" toggle | Mode of L4C | Partly: "4b for MC" stage only (`src/Config.tsx:338`) | No |

**First Block and inspection**

| ID | Drill | Trains | Lvl | Sources | How an app implements it | Module / setting | onionhoney | FM |
|---|---|---|---|---|---|---|---|---|
| F1 | Random-state FB, then compare with ranked solver solutions | FB efficiency | L2–L5 | [11][16][44] | Plan and solve; reveal top N solutions with ergonomic ranking | Module FBT | Yes (`#fb`, analyzer) | Yes (MVP) |
| F2 | Optimal-length ladder with mastery gates ("frameworks": depth 3 at 90 %, depth 4 at 50 %, near-instant) | Pattern vocabulary | L2–L3 | [33][44][43] | Cases of exact optimal length N; success within a time budget; promote at the thresholds, then prompt real solves | Setting of FBT (difficulty + stop rule) | Partly: level sliders FB 3–9, FS 1–6 (`src/Config.tsx:83-97`), no gates | No (TR has a ladder) |
| F3 | Inspection scope ladder: DL+centre → FS/line → FB → FB+DR → FB+SS → F2B; time unlimited → 15 s | Planning depth | L2–L5 | [2][11][31][7][115] | Countdown; user enters or performs the plan; compare with the best plan over the user's orientation set | Module INS (scope and time are settings) | Partly: analyzer over x2y/CN, no timer | Partly (15 s planner, MVP; no scope or time ladder) |
| F4 | Framework spotter / "give up fast" | Choosing a start quickly | L3 | [33] | Show a scramble for N s; user names a framework type or "bad"; solver scores it | Mode of INS | No | No |
| F5 | FB taste: pick the easiest of 8 (or 24) FBs fast | Quick evaluation | L3 | [11][10] | Tap the best location/colour within T s; reveal per-orientation optimal lengths | Mode of INS | Partly: analyzer quiz with spoiler | Partly |
| F6 | FS first, then predict the LP pieces (edge position/orientation, corner position/D-sticker) | Planning past FS | L3 | [11] | After the FS plan, tap where the LP edge and corner will be | Mode of INS | No | No |
| F7 | Pair relations: which edge states pair with this corner | Pair logic | L2–L3 | [11] | Show a corner and its D-sticker; tap valid edge positions | Mode of PAIR | No | No |
| F8 | Find the pair: the single move that joins a pair (align then join) | Spotting pairs | L1–L3 | [126] | Show the cube; tap the move; timed | Module PAIR | No | No |
| F9 | Ready-made block: expand it, build around it, or ignore it | Using premade pieces | L3 | [126] | Scrambles with a premade pair/square; reveal the optimal FB for each choice | Mode of FBT | Partly: difficulty filter (`src/Config.tsx:307`) | No |
| F10 | Influencing quiz (17 worked examples) | Two-stage influencing | L3–L4 | [11] | Show the stage-1 solution; user picks the influencing move; examples may be bundled (MIT) | Lesson exercise + mode of FLP | No | No |
| F11 | Last pair (+DR): find 2+ different solutions before reveal | Solution variety | L3–L4 | [11][31] | User enters two solutions; app verifies both | Mode of FLP | Partly: FBLP(+DR) trainer | Partly (FBLP(+DR), MVP) |
| F12 | DR tier call-out and DR-aware solution ranking | FB+DR | L4 | [31] | After FS, tap Good/Okay/Bad; each solution shows the DR tier it leaves (18-placement table, re-derived by BFS) | Setting of FLP | Partly: DR position filters (`src/Config.tsx:263`) | No |
| F13 | Keyhole `u' R u`: predict where DR ends up | DR influence | L4 | [31] | Pick a DR start; predict; animate | Lesson exercise | No | No |
| F14 | Partial SpeedBLD FB | Planning; warm-up | L2–L5 | [11][4] | Cube hides at the first move; real-cube self-report, or a smart cube checks | Mode of INS (warm-up role) | No | Yes (later) |
| F15 | Calculate-N: find the N moves that scrambled the cube, N = 3 → 10 | Inspection calculation | L3–L4 | [126] | App applies N moves (or picks an FB of optimal length N), hides the cube, user types the solution | Mode of INS | No | No |
| F16 | Piece tracking | Tracking | L3 | [11] | N hidden moves; predict a piece | Module TRK | Partly: code exists, tab hidden in v2.3 (`src/components/AppView.tsx:83`) | Yes (v1) |
| F17 | Neutrality ladder: y2 → x2y2 → x2y → CN | Colour neutrality | L2–L4 | [10][115][125][18] | Orientation set chosen once, used by every block trainer; advisor estimates moves saved from the user's FBs | App-wide setting + advisor card | Partly: 24-orientation selector (`src/Config.tsx:199`) | Partly |
| F18 | FB approach tags: DL + 2 pairs, line-line 2x2x1 + pair, S insert, line-line, BL edge fix, premade square | Approach vocabulary | L1–L3 | [24][109][18] | Tag solver solutions; filter; quiz "which approach?" | Setting of FBT | No | No |
| F19 | "Panic find" a BL edge instead of DL | Fallback start | L2 | [7] | Basis setting | Setting of FBT | Yes: basis Default/DL/BL (`src/Config.tsx:218`) | No |
| F20 | Plan fingertricks with the solution; prefer ergonomic pairs over raw movecount | Ergonomics | L3 | [11] | Ergonomic ranking; lesson | Setting of FBT/FLP | Partly: ergonomic evaluator (`src/lib/Evaluator.tsx:7-37`) | Partly |

**Second Block**

| ID | Drill | Trains | Lvl | Sources | How an app implements it | Module / setting | onionhoney | FM |
|---|---|---|---|---|---|---|---|---|
| S1 | SB square with DR filters, DR first; DR-influence mode: DR in a good but unsolved position, influence the SS pair while inserting DR | SS | L2–L4 | [12][25][44][31] | Random state; solver solutions | Module SS | Yes (`#ss`) | Yes (MVP) |
| S2 | Restricted move set `<R,r,U,M>`, no F, slow and efficient | SB fundamentals | L1–L2 | [25][121] | Move-set profile filters solver output and checks user input | App-wide move-set profile | Partly: M/r solver, no profile | No |
| S3 | Basic insert cards (`r U r'`, `r U' r'`, `R U' M' U r'`, `r U' M U R'`) | SB inserts | L1–L2 | [25] | Case cards with setup, random AUF | Case subset of SLP | No | Partly |
| S4 | SB last pair, 65 cases in 6 groups, front or back slot | SBLP | L3 | [38][39][35][37] | Random F2B-minus-pair; recognise; execute; solver solutions; per-case learn state | Module SLP | No tab (pruners `sb`/`lpsb` exist) | Yes (MVP) |
| S5 | Pair recognition by the square's L/R stickers, masked | Recognition | L3 | [34] | Flash a masked case; time recognition separately | Mode of SLP | No (the sheet notes onionhoney cannot mask this way) | No |
| S6 | Multislotting and F-move tricks (15 cases) | Advanced inserts | L4 | [35][37][4] | Cases with both slots open; F/S/B allowed | Case subset of SLP | No | No |
| S7 | Approach comparison: pair-first, DR-first, DR-neutral | SB strategy | L3 | [33][12] | Show the shortest solution per approach | Setting of SS | No | No |
| S8 | Pair choice after DR | SB strategy | L3–L4 | [12][4] | User picks a pair; reveal both lengths | Mode of SS | No | No |
| S9 | FB → SB tracing: where does an SB piece end up | Transition | L3–L4 | [33][31] | After the plan, tap a piece's destination | Mode of CHN or TRK | No | Partly (chain flow, v1) |
| S10 | SB pace: pair, edge, last pair under 2 s each | Fluency | L3 | [121] | Split targets with a smart cube or manual splits | Setting of SC/TIM | No | No |
| S11 | Tracking-friendly last pair: keep U corners still until the final insert | Prediction readiness | L4–L5 | [100] | Rank equal-length solutions by this rule | Setting of SLP | No | No |
| S12 | Twisted or passive insert decision (TCMLL, ACMLL) | Expert SB | L5 | [68][66] | Show the LP state; choose normal vs twisted/flipped; show moves saved | Mode of SLP | No | No |

**CMLL**

| ID | Drill | Trains | Lvl | Sources | How an app implements it | Module / setting | onionhoney | FM |
|---|---|---|---|---|---|---|---|---|
| C1 | 2-look: how many oriented corners? which shape? adjacent or diagonal swap? | 2-look recognition | L2 | [57] | Two quick taps, then execute | Set of CMX/CMR | No | Yes (MVP) |
| C2 | 1.5-look: case → AUF + base alg, then a known case | Bridge to full | L2 | [58] | Decompositions generated from the base algs the user knows | Set of CMX | No | No |
| C3 | Full 42, random state, random AUF, optional SB-LP trigger before the case | CMLL | L2–L3 | [45]–[49][73][74] | Group/case selection | Module CMX | Yes (`#cmll`, triggers at `src/Config.tsx:184`) | Yes (MVP) |
| C4 | Name the case from any angle without a pre-AUF; timed; stats per case and angle | Recognition | L3 | [14][60][49][61] | Tap the case name (or alg); mask optional | Module CMR | Partly: random AUF; "recog stickers only" for T only (`src/components/CmllTrainerView.tsx:126-134`); no name input or timing | Yes (MVP) |
| C5 | Peeking when 90° off | Recognition | L2–L3 | [14] | Fixed off-angle view; a "peek" control reveals the side at a time cost | Setting of CMR | No | No |
| C6 | Hidden-sticker inference and confusable pairs (e.g. U X vs U Bottom Row) | Recognition | L3 | [14][112] | U/F/R only; two-way choice; queue weighted by the user's errors | Setting of CMR | No | No |
| C7 | Recognition system: U-sticker + pattern (default), Iuri's L, NMCMLL/ATCRM (L/R first), Straughan four-sticker, three-sticker tracking, three-corner, conditional | Recognition | L3–L5 | [61][64][65][101][59] | Mask to the stickers the system reads; group cases accordingly | Setting of CMR | Partly: NMCLL groups (`src/Config.tsx:137`), hyperorientation L/R or F/B mode (`:189`), obscure non-L/R (`:368`) | Partly (NMCLL, v1) |
| C8 | Execution timing per alg; mo3; "order loop" (repeat the alg until solved again) | Execution | L2–L3 | [125][138] | Case named; time from first move to solved | Mode of CMX | No | Partly (v1) |
| C9 | Recap / time attack: every selected case once, total and per-case times | Coverage | L3 | [68][75] | Selector and stop rule | Setting of any case module | Partly: batch of 6 (`src/reducers/CmllStateM.tsx:95`) | No (TR: coverage) |
| C10 | Mirror across M or S | Mirrored execution | L3, OH | [73] | Mirror toggle | Setting | No | No |
| C11 | CMLL → LSE effect; pick the variant that leaves better EO | EO control | L4 | [52][48][55][37] | Alg card shows flipped and moved LSE edges; drill: given case and edges, pick a variant | Mode of CMX + alg card | No | No |
| C12 | Predict EO after CMLL | Transition | L4 | [4] | Show case and alg; predict the bad-edge pattern | Mode of PRD | No | Partly (later) |
| C13 | OCLL prediction during the last SB pair (27 twist states × 5 inserts) | Prediction | L4–L5 | [59][100] | Freeze 2–3 moves before the insert; tap the OCLL; animate | Mode of PRD | No | Partly (later) |
| C14 | CMLL prediction by tracking (Straughan or ATCRM stickers) | Prediction | L5 | [59][101] | Freeze before the last pair, show only tracked stickers; name case + AUF | Mode of PRD | No | Partly (later) |
| C15 | Intuitive CP: which two corners swap | Prediction | L4 | [59] | Quiz | Mode of PRD | No | No |
| C16 | NMCMLL with non-matching blocks; 2-look NMCMLL subset as a stepping stone | Pseudo blocks | L5 | [64][65][121] | SS scrambles on the other D colour; NMCMLL recognition | Setting of SS + CMR/CMX | Partly: NMCLL groups (`src/Config.tsx:137`), matching blocks only | Yes (v1) |
| C17 | Pinkie Pie (OLLCP after pairing ULUR on D) | L10P variant | L5 | [62][63][4] | CMX set keyed by CMLL group + EO pattern | Set of CMX | Partly: `ollcp_algs_raw` (`src/lib/Algs.tsx:120`), tab marked TODO (`src/components/AppView.tsx:80`) | No |
| C18 | TCMLL (vanilla 86; OH TCMLL-N) | Twisted corner | L5 | [68][69][70] | Scrambles with a twisted DFR corner | Set of CMX | No | No |
| C19 | ACMLL after passive F2B; 2-look ACMLL subset (CO + F2B fix, then CP) as a stepping stone | Passive blocks | L5 | [66][67] | SLP ends with a chosen defect, then the ACMLL case | Set of CMX + mode of SLP | No | No |
| C20 | Conjugated CMLL (R setup, CMLL, undo) | Variant | L5 | [72][73] | SB corner left unsolved | Set of CMX (scope, section 10) | No | No |
| C21 | Learn an alg as a chain of named triggers (show the chunks, drill each chunk, then the whole alg) | Alg memorisation | L0–L3 | [100][51][22] | Alg card split into triggers; chunk-by-chunk recall | Mode of CMX and BEG | No | No |
| C22 | Insert choice for OCLL: given the last-pair state, pick the insert variant (`R U R'`, `r U r'`, `R U2 R'` …) that gives the best OCLL | OCLL influence | L4–L5 | [59][126] | Show each variant's resulting OCLL and moves | Mode of PRD | No | No |

**LSE**

| ID | Drill | Trains | Lvl | Sources | How an app implements it | Module / setting | onionhoney | FM |
|---|---|---|---|---|---|---|---|---|
| E1 | EO flowchart: tap the branch, apply the trigger, reach the arrow (lefty or righty) | 4a | L2 | [76][132][2] | Small decision tree with an animated cube | Mode of EO | No | Partly |
| E2 | 4c cases: centre fix, swaps, dots | 4c | L0–L3 | [23][125][121] | Random 4c states; M/U buttons or a real cube; solution reveal | Module L4C | Yes (`#4c`) | Yes (MVP) |
| E3 | DFDB prediction during 4b: cycle or 2e2e? matching centre? stacked or opposite? optional: does the 3-cycle start with M2 (Sasaki)? | Pauseless 4b→4c | L3 | [88][89][7][125][102] | Freeze before the L/R insertion; 2–3 timed taps; then 4c | Mode of L4C | Partly: "M2 to 4c" stage, no questions | Partly (4c with recognition method, MVP) |
| E4 | Other 4c systems: BU, EZ4c, FUBU | 4c recognition | L3 | [90][91][102] | Recognition-system setting with own hints | Setting of L4C | No | Partly (FUBU/BU/DFDB, MVP; no EZ4c) |
| E5 | Misoriented-centre 4c (separated / together) | MC | L3–L4 | [99] | Centre-offset states | Setting of L4C | Partly: misaligned centre (`src/Config.tsx:326`) | Partly |
| E6 | EOLR by EO group; solve freely, then compare with optimal (efficiency %); "fallback route" reveal for bad cases (EO, then LR) | EOLR | L3 | [78][86][87][83][84][77] | Virtual LSE; MC allowed toggle; weight per EO pattern; the fallback shows its move-count cost | Module EOLR | Partly: EO and MC filters, solutions shown (`src/Config.tsx:68-74,344`); no free-solve score | Yes (MVP) |
| E7 | Two-step EOLR recognition: EO group, then L/R placement | Recognition | L3 | [78][77] | Two taps, timed separately | Mode of EOLR | No | No |
| E8 | Reduction stepper: reduce to an arrow sub-case (Best, Good, Adjacent, Bottom, Bad) | Intuitive EOLR | L3 | [86][78][77] | User applies M/U; app names the arrow sub-case reached | Mode of EOLR | No | No |
| E9 | Weighted EO patterns (raise weak ones) | Weak cases | L3 | [84] | Weight sliders, or automatic from history | Setting (selector) | Partly: on/off flags | Partly (SRS, v1) |
| E10 | EOLRb corner relation (matching / opposite / adjacent); learn only cases that save moves | EOLRb | L4 | [82][81] | U layer + L/R edges only; tap the relation, then the case | Mode of EOLR | Partly: EOLRb option (`src/Config.tsx:356`) | Partly (EOLRb trainer, MVP; no corner-relation drill) |
| E11 | EOLRb 4c non-cycle influencing / dots skipping (8 cases) and regular insertions (3) | Dots avoidance | L4 | [15][4] | Case subset; explanation text may be bundled (MIT) | Set of EOLR | No | Yes (v1) |
| E12 | ULUR/UFUB neutrality | Flexibility | L4 | [2][4] | Target setting | Setting of EOLR | Yes: ULUR/UFUB (`src/Config.tsx:332`) | No |
| E13 | 6-flip repertoire incl. non-`<M,U>` EOLRb | Bad cases | L4 | [85][4] | Case subset; curated non-MU alg next to the MU optimum | Set of EOLR | Partly: 6-flip EO filter | No |
| E14 | 4c influencing: LRb backtracking, 4c^3 | Expert LSE | L5 | [4] | Lesson + case subset | Set of EOLR/L4C | No | No |
| E15 | EOBF / EOdM target | Variant | L5 | [15][142] | Target setting | Setting of EOLR | Yes: EOdM (`src/Config.tsx:356`) | Yes (later) |
| E16 | Full LSE free-solve with efficiency score; QTM-only option | LSE fluency | L2–L4 | [84] | Random-state LSE; M/U buttons; efficiency = optimal ÷ yours | Module LSX | No | No |
| E17 | LSE sprint: timed LSE-only ao5/ao12, reaction split off | LSE speed | L3 | [110] | Timed LSE states on a real cube | Mode of LSX | No | No |
| E18 | Gilles' centre-orientation EO shortcuts (28 cases); orient centres with r at the end of SB | Efficiency | L4 | [121] | Case subset | Set of EO | No | No |
| E19 | EO with LR-edge control: finish 4a with at least one L/R edge on D, then go into 4b without a pause | Pauseless 4a→4b | L2–L3 | [99] | Virtual LSE; target = EO plus ≥1 L/R edge on D; flag whether it was forced; time the 4a→4b gap | Mode of EO | No | No |

**Whole solve, OH, competition**

| ID | Drill | Trains | Lvl | Sources | How an app implements it | Module / setting | onionhoney | FM |
|---|---|---|---|---|---|---|---|---|
| W1 | Slow solves: per-stage movecount against the solver; optional no-pause rule | Efficiency, lookahead | L2+ | [16][118][7] | Enter or smart-cube the solution; per-stage comparison | Module SLOW | No | Yes (v1) |
| W2 | Linear Roux FMC: STM, no NISS or insertions; step editor | Efficiency | L3+ | [1][117][126] | Step lines with `//` comments; validate; STM/HTM; compare with optima; export a link | Mode of SLOW | No | No |
| W3 | Time-budget movecount (15 s / 30 s / 1 min / 5 min) | Planning vs speed | L4 | [121] | Thinking timer; compare with Gilles' table | Setting of SLOW | No | No |
| W4 | Study a pro solve: inspect and plan, then reveal step by step | Ideas, planning | L2+ | [118][107][103][109] | Import scramble + recon; plan; reveal | Mode of REC | Partly: recon analyzer (`src/components/AnalyzerView.tsx`) | Partly (recon import, v1) |
| W5 | Per-step recognition vs execution, and TPS | Weakness finding | L3+ | [105][106] | Smart cube or manual splits; stacked bar | SC/TIM stats | No | Yes (v1) |
| W6 | Split balance against 17/27/23/33 % (FB/SB/CMLL/LSE) | Weakness finding | L3 | [125] | Stats card | Stats | No | Partly (later) |
| W7 | Movecount benchmarks per stage | Efficiency | L2+ | [142][125][107] | Stats card from the app's own solvers | Stats | No | Partly |
| W8 | Weekly Roux comp: 3x3, OH, 4x4: Ao5 on 5 fixed scrambles; FMC: 1 scramble, STM, linear only; WCA-style inspection | Competition practice | all | [1][116] | This week's scrambles; time; open a prefilled Google Form for the user to submit | Module WK | No | No |
| W9 | Lookahead progressions (no pausing; don't look at solved pieces; treat each step as a transition into the next) | Lookahead | L3 | [10][99][11][138] | Slow-solve settings | Setting of SLOW | No | Partly |
| W10 | Competition routine: user mantra or breathing cue; neutral feedback after a bad single | Nerves | L3+ | [7][128][129] | Pre-solve screen in a competition mode; no bundled quotes | Setting of TIM | No | No |
| W11 | OH profile: hand, OH alg sets, F/B/D-avoiding solution ranking, mirroring | OH | L3+ | [53][54][55][39][73] | App-wide profile read by every module | App-wide setting | No | Partly (OH set, v1) |
| W12 | Trigger and fingertrick timing (table abuse, OH dots, `M U M'`) | Execution | L2+ | [4][102] | Time repeated execution of a named trigger | Set of CMX ("triggers") | No | No |
| W13 | Big-cube parity CMLL (43) and parity 4c (24); Meyer/Stadler/CR4 split timer | Big-cube Roux | L3+ | [95][96][93][94][97] | Separate pack | Module BIG | No | No |
| W14 | CP recognition (6 CP cases) and CPFB with restricted moves | CP-first OH | L5 | [41][43][40] | Separate pack | Module CPF | No | No |

## 5. Learning content catalogue

Licence status: **E** = may embed or adapt with the licence notice; **O** = write our own text (the facts, positions and ideas are free to re-derive; credit the source); **L** = link only.

| Topic | What to teach or show | Lvl | Sources | Licence status |
|---|---|---|---|---|
| What Roux is, why Roux | Four steps, no rotations, 2-gen M/U finish, lower movecount, more planning; share of top-100 averages (6 % 3x3, 34 % OH, per rouxcubing.com's charts; date of the data not stated) | L0–L1 | [1][115][125][121] | O (stats as facts with credit); L to the survey |
| Cube basics and notation | Pieces, fixed centres, face/slice/wide moves, rotations; "M follows L"; `r` = `Rw`; metrics STM/ETM/HTM/QTM (STM counts M as one) | L0 | [20][113][126] | O (the tutorial has no licence); Tronto notation section E (CC BY) |
| Glossary | rouxcubing's 25 terms (4a, 4b, 4c, BU, CMLL, CPFB, DFDB, EO, EOLR, EOLRb, EOLRc, FB, L10P, LR, LSE, MC, NMB, NMC, NMCMLL, Pinkie Pie, SB, TCMLL, UFUB, ULUR, and a joke entry) plus FS, SS, SP, LP, DR, F2B, AUF, x2y/CN, arrow, 6-flip, dots, influencing, framework, D edges, pure/diag corner, CMLLEO/KCLL, ACMLL, EOBF | all | [5][11][33][61][89][102] | O (write our own definitions: rouxcubing's wording has no licence; the term list itself is fact) |
| Complete beginner method | FB → SB → 2-look corners → LSE 4a/4b/4c with interactive steps; nobody has a finished text version | L0 | [21][22][23][32][3][30] | O |
| Coming from another method | Neutral-language on-ramp: x2y, `F`/`B`/`r2` inserts, `<R,r,U,M>`, "algs you may already know" | L1 | [24][25][99][3] | O (no CFOP steps or sets) |
| FB strategy and planning | Approaches; pair relations; influencing (two forms, two intents); FS-first planning; LP prediction; taste; fingertrick planning; BL panic edge; frameworks; zwischenzugs | L2–L4 | [11][24][33][7][18] | Reader parts E (MIT); Frameworks O; tutorial O |
| FB+DR | DR tier list (Good 0–2 / Okay 3 / Bad 4–5 moves); influencing techniques; "don't force it" | L4 | [31] | O (tier list re-derived; ask Zhouheng before reusing the 10 examples) |
| Blockbuilding principles | Align then join, move it out of the way, destroy and restore, keyhole, one move two goals, influence later steps, ready-made blocks | L2–L4 | [126] | E (CC BY 4.0, credit Tronto; re-make examples that credit third parties) |
| Second Block | DR-first vs freestyle (DR-first is the only SS strategy that keeps a choice between the two remaining pairs); inserts; pair choice; multislotting; SBLS taxonomy | L2–L4 | [12][25][35][38] | Reader E; rest O |
| CMLL basics | 2-look, 1.5-look, the 42-case taxonomy (8 OCLL groups × CP), naming maps, case probabilities and expected movecount (about 10.3–10.4 STM incl. AUF for 2H sheets) | L2–L3 | [57][58][45][48][49][121] | O |
| CMLL recognition | Reader's 3 stages (orientation only → fixed angles + peeking → all angles with "each colour twice, adjacent colours only" inference) and its U X example; per-group cue cards; systems compared by sticker count | L2–L5 | [14][122][60][61][59][101] | Reader E; others O (credit Iuri, Athefre, Liam Highducheck, effperm) |
| CMLL alg choice | EO-preserving vs EO-flipping variants; AUF-saving; LSE handedness; OH | L3–L4 | [37][48][55][47] | O |
| LSE fundamentals | EO definition; arrow; swaps; flowchart (lefty/righty); 4b; 4c cases; "LSE is intuitive" | L0–L2 | [23][76][13][125] | Reader quote E; flowchart redrawn O |
| 4c recognition | DFDB decision tree (cycle/2e2e, matching centre, stacked/opposite); BU; EZ4c; MC DFDB | L3 | [88][89][91][99][102] | O (credit Anto, XtremeCubez, John Mark Grimes, Straughan) |
| EOLR / EOLRb | Why (5.63 vs 7.38 STM); EO group names and placement classes; reduce to an arrow; MC finishes; corner relations; dots skipping; 6-flip | L3–L4 | [87][78][86][82][15] | Reader's EOLRb sections E; rest O |
| Lookahead and tracking | Static FB vs dynamic SB; pauseless transitions; partial SpeedBLD; tracking | L2–L4 | [10][99][11] | Reader E; rest O |
| Colour neutrality | Orientation sets; statistics; when to widen | L2–L4 | [10][18][115][125][142] | Reader E; stats O |
| Fingertricks and TPS | 2H and OH tricks; table abuse; text fingertrick notation | L2–L5 | [4][7][49][102] | L (videos); O for short tips |
| Efficiency benchmarks | Per-step movecount and TPS targets by level | L2–L5 | [125][142][107][105][106] | O (recomputed with the app's solver where possible) |
| Example solves and reconstructions | How to study them; where to find them | L2+ | [11][16][103][107][108][109] | Reader advice E; data L |
| Practice methodology | Deliberate practice vs solves; warm-up; slow solves | L2+ | [7][11][138] | O; L to videos |
| Competition | WCA basics; weekly comp; nerves | L2+ | [1][7][129] | O; L to the litany; L only to the parable: the translation on Wikipedia quotes Larre et al. 1993, so it is not CC BY-SA [129] |
| Lore and history | Gilles Roux 2003; timeline of MC (2004), NMB (2004), FUBU/BU/DFDB (2010–2013), CMLLEO (~2010), EOLR (2012/2016), Pinkie Pie (2011), ACMLL (2021), prediction (2020–2024); "Midori" | all | [102][121][5] | O (facts with credit); L to cubinghistory |
| Advanced technique library | NMB, NMC, nonlinear F2B, CPFB, ACMLL, TCMLL, Tyrannical Caterpillar, Pinkie Pie, conjugated CMLL, EOBF; marked as niche or experimental | L4–L5 | [2][7][66][68][72][118] | O |
| Hardware | Roux-friendly cubes (M-slice) | all | [9][118] | Gap: the Reader's chapter is empty [9]; nothing to reuse |
| Big-cube Roux | Meyer, Stadler, CR4 step orders; parity at CMLL and 4c | L3+ | [93][94][95][96][97] | O; L |

## 6. Alg and case sets catalogue

### 6.1 Sets

Ways to obtain: **GPL** = onionhoney's GPL data (roux-trainers `src/lib/Algs.tsx`); **SOL** = generate with the app's solver; **USR** = user-entered; **MIT** = MIT-licensed source with notice; **ASK** = permission needed to copy.

| Set | Cases | Sources | Licence of sources | Legal way to obtain |
|---|---|---|---|---|
| Beginner FB/SB piece insertions | 6 + 10 | [32] | None | SOL/O (short intuitive moves) |
| Beginner corners: Sune repeated + T-perm (or J-perm) | 2 algs | [22][32][130][131] | None | Standard algs; O |
| 2-look CMLL | 7 orientation + 2 permutation (11 + 4 algs) | [57] (two identical copies) | None | The 2 swap cases are O cases in the GPL CMLL data; the orientation algs are standard short algs (SOL or USR) |
| 1.5-look CMLL | 9 base algs; 42 recipes (9 one-look) | [58] | None | SOL (decompose in-app) |
| CMLL (2H) | 42 | [45]–[50][51][37][112] | None | **GPL** (`src/lib/Algs.tsx:28-72`); alternatives USR/SOL |
| CMLL statistics | 42 + skip | [48][49] | None | Compute |
| OH CMLL (LH and RH) | 42 each | [53][54][55][56][37] | None | USR; SOL with OH move-set ranking; Tao Yu's OH CMLL list ships in MIT code but is a compilation, so credit or ASK [73] |
| NMCMLL | 42 + solved; 84 patterns | [64][65][121] | None | **GPL** map (`src/lib/Algs.tsx:163-296`); compute patterns |
| Straughan four-sticker patterns | 162 for 42 cases | [101] | None | Compute from the cube model |
| OLLCP (Pinkie Pie) | 331 [62]; 342 entries [63] | [62][63] | None | GPL partial (`src/lib/Algs.tsx:120`, 7 groups); SOL for the rest |
| TCMLL | 86 vanilla; TCMLL-N 258 (+172 WIP) | [68][69][70] | None | SOL; ASK |
| ACMLL | 43 per variant (flipped pair ×4, outside ×2, swaps, front cycle, spiral) | [66][67] | None | SOL (Athefre asks for generated sets) |
| L5C / SL5C | 614 | [71] | None | Out of scope (section 10) |
| Conjugated CMLL | 42 algs under R setups (672 combinations in Alg-Trainer) | [72][73] | n/a | Derive from CMLL |
| Parity CMLL (4x4) | 43 | [95] | None | SOL; ASK |
| SB last pair (SBLS) | 65 in 6 groups; 130 with back slot | [38][39] | None | SOL (`sb`/`lpsb` pruners); USR |
| SB last pair with DR + back square | 47 (+15 multislot / F-move) | [35][37] | None | SOL (allow F/S/B for the 15) |
| SB square / last slot per slot | 180 / 108 | [50] | None | SOL |
| SB pairs by L/R stickers | 64 | [34] | None | SOL |
| FB influencing examples | 17 | [11] | **MIT** | MIT (bundle with notice) |
| FB+DR worked examples; DR tier list | 10 (15 solutions); 18 placements | [31] | None | Tier list re-derived (BFS reproduced it exactly); examples ASK or make our own |
| EO (4a) states | 9 non-solved (mirrors merged); 10–12 named in some sources | [76][132][38][125][84] | None | SOL |
| EOLR | 51 named / 94 placements [78]; 53 optimal [87]; 123 keyed by position pair [79]; 128 [50] | [78]–[80][86][87][50][37] | None | SOL (`eolr` pruner) |
| EOLRb | about 104 sub-cases [82]; 960 states / 2,720 optimal solutions [83]; 720 [50] | [82][83][50] | None | SOL |
| Non-`<M,U>` EOLRb | 5 cases, 10 2H + 6 OH algs | [85] | None | SOL with r/R/F move set; credit Neo |
| EOLRb insertions; 4c non-cycle influencing | 3; 8 | [15] | **MIT** | MIT |
| 4c | 4 non-solved cases; 48 DFDB pure M/U [50] | [23][125][50] | None | SOL |
| DFDB / EZ4c decision trees | 3 branches / 6 leaves | [88][89][91] | None | O (re-express) |
| Gilles' EO shortcuts | 28 (9 groups) | [121] | None | SOL |
| Parity 4c (4x4) | 24 oriented-centre cases | [96] | None | SOL (odd permutations × centre states) |
| Alternative LSE splits (L5E, ELL, L4E) | 360 / 30 / … | [50] | None | Out of core |
| CP-first sets (CPLine 114; WV, SV, OCLS) | various | [40][43][143] | None | Out of scope (section 10) |

### 6.2 What onionhoney's GPL data already covers

CMLL 42 with names (`src/lib/Algs.tsx:28-72`), the NMCLL→CMLL map and display parity (`:163-326`), seven OLLCP groups (`:120`), and solvers for FB, FS, FBDR, SS, SB, LSE, EOLR(b) and EOdM [136]. With the solver, every set the community uses for core Roux can be produced without copying a sheet. Missing from the GPL data: 2-look and 1.5-look tables (derive), OH algs (user or solver), SBLS naming (adopt the taxonomy, generate the algs).

### 6.3 Case ids and naming maps (data model)

- **CMLL:** canonical id = OCLL group + slot (O3, O6, H1, H2, H3, H6, Pi1–6, U1–6, T1–6, S1–6, As1–6, L1–6), as in the rouxl.es image ids that eff's, Anto's, Sevilz's and John Mark's sheets share [45][46][48][49][53]. Aliases:
  - pattern names: Kian/SpeedCubeDB/Anto [37][112][46]
  - eff's descriptive names [45]
  - Gilles' A–H × 1–6 grid and Mark Fiend's nicknames (Bowtie, Headlights, Chameleon, Superman) [121][122]
  - Atlas's sticker-relation names [50]
  - roux-trainers ids, e.g. `u_x_checkerboard` = U X [112]
  - Watch out: U slot 1 and slot 6 names are swapped between Sevilz's and Anto's sheets [55].
- **EO / EOLR groups:** 3/1 = Arrow = 3LRF-1F; 1/1 = 1F-1B; 4/0 = 4-0; 0/2 = 0-2; 2o/0 = 2FB-0; 2o/2 = 2LR-2; 2a/0 = 2BL-0; 2a/2 = 2BL-2; 4/2 = 6-flip [78][86][83][84][87].
- **SB last pair:** SpeedCubeDB's U / Up / Um / D / Dp / Dm groups + edge position + `m` [38], also used by Azusa's OH guide [39].
- **TCMLL:** letters A–H [68] vs 2x2 TCLL shape names (Strolley, Gun, Twoface…) [69] vs nickname groups (Pac-Man, Shuriken…) [70].

## 7. Tools and trainers landscape

| Tool | What it does | Worth matching or beating | Licence |
|---|---|---|---|
| onionhoney roux-trainers [139] | Baseline: FB/FS/FBDR/SS/FBSS trainers with optimal-length levels, FB analyzer (x2y/CN), CMLL (case/group, AUF, triggers, NMCLL, recog stickers, batch), 4c, EOLR/EOLRb/EOdM with MC modes, recon analyzer | Beat on: touch, timing of recognition, free-solve scoring, SRS, beginner path. Tracking tab hidden in v2.3; OLLCP tab TODO (`src/components/AppView.tsx:80-83`) | GPL-3.0 |
| Cubegrass block trainer [44] | FB, SB, SS, SB-LP at a chosen optimal length; masked 2D net; all optimal solutions | Multi-select of lengths; show all optima, then rank by ergonomics | None; server closed |
| Jade's EOLRb trainer [83] | Keyboard LSE; 8 EO pattern checkboxes; 960-state precomputed optimal table; "your solution vs optimal"; mirrors, random AUF only where it changes the case, 24 colour orientations | Precompute tables at build time; add 6-flip and misaligned centres (missing there); touch input | None (code readable, unlicensed) |
| LaSEr v0.1 (Jade) [84] | Full LSE, 4a, EOLR, EOLR-b, 4c; on-screen M/U buttons; weights per EO state; efficiency score | One LSE trainer with five targets; generate the target state first, then solve it (its MC bug comes from building cases as inverses of truncated solutions) | None |
| Tao Yu Alg-Trainer [73] | CMLL and OH CMLL drilling; obfuscated scrambles with a random LSE before the case; x2/y preorientation; mirror across M/S; user-defined sets; set statistics; Giiker | Scramble with a random LSE state; self-test mode; set statistics card | **MIT** |
| kSim 2 [74] | Simulator with random-state CMLL (LSE edges greyed) and LSE (random M2) scramblers; ACube optimal solver; timer; move-history rewind | "Stage complete when unmasked stickers are solved" predicate; LSE scrambles with offset centres | None (ISC boilerplate only) |
| Roman's ZBLL trainer [75] | Not Roux. Case-group grid with tri-state selection; recap mode; least-practised weighting (20 %); equal-length scrambles; presets; notes per case; 19 languages | Selection grid, recap, presets for CMLL/EOLR/TCMLL | None |
| Edde's CEOR trainer [43] | Block trainer by exact optimal length with OH move sets (`RrUu`, `RrUuFfS`, RU), incl. Roux SS and CPFB; session CSV export | Exact-length case generation; move-set profiles; left/right mirror | None |
| badochov CMLL visualiser [52] | Draws each CMLL alg's effect on the 6 LSE edges; PDF export | Compute the same "LSE fingerprint" on every alg card | None |
| Scheopner's solvers [125] | Server-side optimal FB (with a 24-checkbox neutrality set), SB `<U,R,r,M>` and LSE `<M,U,E2>` solvers | Neutrality picker by DL-edge colour pair | None; no public source |
| SpeedCubeDB [38][63][112] | CMLL, SBLS 65, EO4A 9; votes; 3-state learnt flag; group counters; animated player; Practice covers PLL only | Learnt counters per group; setup alg per case; offline | None; ads |
| Cubedex [138] | Smart-cube alg drilling with auto-graded SRS, including CMLL and EOLR sets (since 2026-04-21) | SRS for case sets | MIT (per FM) |
| cubedb.net [113] | Recon editor/viewer: STM/HTM/QTM/ETM, Roux F2B/CMLL highlighting with block colour pair, mirror/invert, state in the URL; CFOP-only critique | Import/export cubedb links; a Roux solve critique (the gap) | None; obfuscated |
| alg.cubing.net / Twizzle [117][141] | Recon links with `setup=` and `alg=`; Twizzle is the successor | URL import/export | MIT / MPL-2.0 or GPL-3.0+ |
| reco.nz [103] | 1,186 Roux reconstructions (527 3x3, 659 OH); filters; split table FB/SS/SP/CMLL/EOLR/EP with STM and TPS | Link per filter (the `Roux=on` parameter is case-sensitive); same split vocabulary | None; ask before bundling |
| cubesolv.es [110] | 5,239 reconstructions; Roux steps labelled; a `3x3RouxLSE` event; unmaintained since 2020–2022 | Step vocabulary; "LSE sprint" idea | None shown |
| Cubeast [105][106] | Smart-cube ao100 shares; Roux as 4 steps (left block, right block, CMLL, LSE) with recognition vs execution; per-case stats are Premium | Finer Roux steps; free per-case stats | Proprietary |
| smart-roux-timer (eff) [140] | Xiaomi smart cube Roux phase detection (FB/SB/CMLL/EO/EP) | Reference for phase detection | GPL-3.0 |
| VisualCube [114] | Image API with stage masks incl. `f2b`, `cmll`, `line` | Mask list as a checklist; render on-device (the host is broken) | LGPL-3.0+; images free to copy |
| basilio.dev tools [77] | Audio Timer (microphone pause detection, look vs turn time), Recon Viewer | Pause detection without a smart cube (later idea) | None |
| Batch Solver, HARCS | Named as the generators behind many sheets [40][68][71] and in FMC teaching [126] | Not examined | Unknown |

## 8. Creator and video index for "learn more" links

Titles and channels come from YouTube oEmbed; nothing was watched. "Linked from" gives the hub that lists it. All are link-only.

### 8.1 Beginner, switching, roadmap, practice

| Level | Title (channel) | URL | Linked from |
|---|---|---|---|
| L0 | How to Solve the Rubik's Cube Tutorial (Roux Method for Beginners) (Kian Mansour), playlist | https://www.youtube.com/playlist?list=PLBHocHmPzgIjnAbNLHDycgaCP5IqiwnU9 | [3][7] |
| L0 | How to Solve a Rubik's Cube Easily - Part 1: Fundamentals and Notation (DeeDubb) | https://www.youtube.com/watch?v=mB-y0XQiN0M&list=PLajHGvYF36nSsL1r_DqrpDY07TnJwqEpn | [3] |
| L1 | Donovan's Roux Tutorial (Part 1/5) Left Block (Lubix Cube) | https://www.youtube.com/watch?v=ESYJ-DFx0Qo&list=PL754ADDA2E38A21AA | [3][7] |
| L2–L5 | Why You're Not Sub-X with Roux (Kian Mansour), playlist | https://www.youtube.com/playlist?list=PLBHocHmPzgIh_ioQur_FmQ8vdRykjQJbn | [2] |
| L2+ | Partial Speed-BLD: My New Practice Technique (Roux) (Kian Mansour) | https://www.youtube.com/watch?v=4KLFyN6ZDwk | [11][4] |
| General | How to get ahead of 99% of cubers (Swift Cubing - Charlie Eggins) | https://youtu.be/4PIMiKQHCXE | [7] |

### 8.2 First Block and Second Block

| Level | Title (channel) | URL | Linked from |
|---|---|---|---|
| L2 | Roux Blockbuilding Basics: Transition from Beginner to Intermediate (Kian Mansour) | https://www.youtube.com/watch?v=LgIoz-Thh4k | [4] |
| L2 | Quick Roux tips #1: An introduction to line first blocks. (Iuri Grangeiro) | https://www.youtube.com/watch?v=i9zxR5mkgQs | [11][4] |
| L2 | Roux First Block PAIR+ PAIR Example Solves (Kavin Tangtartharakul) | https://www.youtube.com/watch?v=sRTVptb2QrY | [11] |
| L2 | Roux First Block SQUARE + PAIR Example Solves (Kavin Tangtartharakul) | https://www.youtube.com/watch?v=YyY_okZ5Fj0 | [11] |
| L2–L3 | Finding an Efficient First Block in Inspection (teoidus) | https://youtu.be/0Cq3YDud1dA | [11][4][7] |
| L2–L3 | Roux Method Tutorial: First Block Efficiency … (CriticalCubing) | https://www.youtube.com/watch?v=b9rNvrVUsv0 | [4] |
| L3 | [Alex Lau] Roux First Block Tips (Cubing World) | https://www.youtube.com/watch?v=4sEnfMMMJyE | [4] |
| L2–L3 | Louis' First Block Tutorial (title unresolved) | https://www.youtube.com/playlist?list=PLAmLIbUNCUR_e9lmyvcjAb18qlnJ72hGv | [11] |
| L4 | Roux FB + DR Example Solves (White/Yellow) (Zhouheng Sun) | https://www.youtube.com/watch?v=PTGK37AL50E | [4] |
| L4–L5 | Advanced Roux Techniques (Kian Mansour): NMC, nonlinear blocks, NMB; MC at t=351, UFUB at t=486 | https://www.youtube.com/watch?v=L7pIPmSo24c | [4] |
| L5 | Non-Matching Centers Tutorial (Roux) (Mr.Roux86) | https://www.youtube.com/watch?v=_jE-V7P7kIY | [4] |
| L2–L3 | Roux Method Tutorial: Second Block … Solving SB Efficiently (CriticalCubing) | https://www.youtube.com/watch?v=_A7wDUisLDs | [4] |
| L3 | F2B Tech Ep.2: Build your SS Pair "Corner First" (Zhouheng Sun) | https://www.youtube.com/watch?v=eGHJVvvbdXY | [25] |
| L3–L4 | Advanced Roux Second Block: Pair Choice (Kian Mansour) | https://www.youtube.com/watch?v=naip-4TSgdw | [4] |
| L4 | Advanced Roux Second Block: Multislotting (Kian Mansour) | https://www.youtube.com/watch?v=izeYpYLl3-A | [4] |

### 8.3 CMLL

| Level | Title (channel) | URL | Linked from |
|---|---|---|---|
| L2–L3 | Roux Method: The Best CMLL Algorithms and Fingertricks (flamepelt) | https://www.youtube.com/watch?v=xHvPxLlfbzQ | [4] |
| L3 | Better CP Recognition (James Macdiarmid) | https://www.youtube.com/watch?v=vwBR-N29t3k | [4][7] |
| L3 | How to Recognize CMLLs from All Angles (Kian Mansour) | https://www.youtube.com/watch?v=MKZ7JX-hW4g | [14][4] |
| L3 | Full CMLL: All Angle Recognition (John Mark Grimes), playlist; timestamps per case in the author's sheet [49] | https://www.youtube.com/playlist?list=PLn0ZYJiGQcT8qAf8-7g5WHqaA-Zke9ij0 | [4] |
| L3, OH | Per-case OH CMLL demos (Rouxles), 55 videos linked from the OH sheet [53]; Iuri's OH executions (Drive) [56] | see [53][56] | [4] |
| L4 | A New Framework For Predicting LSE EO From CMLL (Zhouheng Sun) | https://www.youtube.com/watch?v=rlqXLsMgAvo | [4] |
| L5 | Roux Method: Pinkie Pie Variant (Kian Mansour) | https://www.youtube.com/watch?v=TPpZ-wzsZe4 | [4] |
| L5 | Roux Method: Tyrannical Caterpillar (TCMLL Variant) (Kian Mansour) | https://www.youtube.com/watch?v=nJhj1uhtRvw | [4] |
| L5 | TCMLL 1a execution compilation (Cordelia, Dropbox) | see [70] | [4] |

### 8.4 LSE

| Level | Title (channel) | URL | Linked from |
|---|---|---|---|
| L2–L3 | Crash Course in Intermediate LSE (teoidus) | https://youtu.be/jqa4wcZMq9Y | [7] |
| L3 | Roux Method: LSE 4C 3-Cycle Recognition (Kian Mansour), BU and DFDB | https://www.youtube.com/watch?v=AwTJ5-ZDaew | [4] |
| L3 | Roux: Fastest Way to Recognize 4c! (DFDB system) (crystalcuber) | https://www.youtube.com/watch?v=wvSBSZ_cyzc | [7] |
| L3 | LSE DFDB Recognition Guide [Roux] (Jihu Mun) | https://www.youtube.com/watch?v=pI0-9tUfWzg | [4] |
| L3 | DFDB - The Ultimate 4c Recognition Method (John Mark Grimes) | https://www.youtube.com/watch?v=bHJ3KqDIQs8 | [4] |
| L3 | Roux DFDB Recognition Tutorial - Pauseless 4C (Richard Peng) | https://www.youtube.com/watch?v=0j5E2frUGVw | [4] |
| L3 | [OUTDATED] EZ 4c: The Best 4c Recognition System (John Mark Grimes) | https://www.youtube.com/watch?v=pWCTCA9ZM-A | [4] |
| L3 | Full EOLR Tutorial (Kian Mansour), playlist | https://www.youtube.com/playlist?list=PLBHocHmPzgIiUBTa7Hs3-xcrBKavfD_zX | [4][37] |
| L3 | EOLR Tutorial: Easy, No Algs, Improve LSE Efficiency (John Mark Grimes), playlist; timestamps per case in [87] | https://www.youtube.com/playlist?list=PLn0ZYJiGQcT-MMtDB67YGoV8T6JQNbhgh | [4] |
| L3 | How to do EOLR without learning algs for Roux Method (SpeedCubeReview) | https://www.youtube.com/watch?v=KG0Jg71hzmE | [4] |
| L3 | [OUTDATED] EOLR Tutorial Episode 1: The Arrow Case (John Mark Grimes); rouxcubing labels it "MOG Cubing's Full EOLR Tutorial" | https://www.youtube.com/watch?v=R5j_1Ym82_M | [4] |
| L4 | All important EOLRb cases!!!!! (and some less important) (Viljo Elo) | https://www.youtube.com/watch?v=sHcGR938EPE | [4] |
| L4 | Roux Method Dots Skipping (Kian Mansour) | https://www.youtube.com/watch?v=2juB7PEeTQw | [4] |
| L4 | Roux Method 6-Flip Repertoire (Kavin Tangtartharakul) | https://www.youtube.com/watch?v=awvvDPldYUA | [4] |
| L4 | Best Ways to Solve Roux EOLR 6-flip Cases (Knut Peterson) | https://www.youtube.com/watch?v=TaopkUfmFF4 | [4] |
| L5 | 6 flip cycle avoidance (Dylan Govic) | https://www.youtube.com/watch?v=p0oO6w2QlR4 | [4] |
| L5 | lrb backtracking (Dylan Govic) | https://www.youtube.com/watch?v=4zIAXfUr5vM | [4] |
| L5 | 4c^3 (Dylan Govic) | https://www.youtube.com/watch?v=BpkkDfBrD9A | [4] |

### 8.5 OH, fingertricks, example solves, big cubes, mindset

| Area | Title (channel) | URL | Linked from |
|---|---|---|---|
| OH | [Antonio Kam] OH fingertricks (Cubing Scene) | https://www.youtube.com/watch?v=4WK0cORg6CQ | [4][7] |
| OH | Rubik's Cube One-Handed Fingertricks (Roux) (Kian Mansour) | https://www.youtube.com/watch?v=ggNmk4mv6Y0 | [4] |
| OH | OH LSE finger tricks roux method (Kavin Tangtartharakul) | https://www.youtube.com/watch?v=5xJCt4VQYFo | [4] |
| OH | OH Dots Fingertrick for Roux (Kian Mansour) | https://www.youtube.com/watch?v=hwXX6GYIHq4 | [4] |
| Turning (general) | Why you suck at turning (Jayden McNeill) | https://youtu.be/KJdV8EhhnKU | [7] |
| Example solves | Example Solves (Kian Mansour), playlist | https://www.youtube.com/playlist?list=PLBHocHmPzgIjAxbZuzs1cWGLqCxykhOyz | [4] |
| Example solves | Example Solves (Kavin Tangtartharakul), playlist | https://www.youtube.com/playlist?list=PLXiPs1z2Pwm5QPrtqB5J5rJ2DH6tbBU8T | [4][25] |
| Example solves | Roux Walkthrough/Examples Solves (CriticalCubing), playlist | https://www.youtube.com/playlist?list=PLqatB_kou3Nb9afWaL7ZJyP_5PhiU0Bf7 | [4] |
| Example solves | Example Solves (Kangaroux), playlist | https://www.youtube.com/playlist?list=PL_rFXDH_hdudEspzR7g_DITZOzj47WFyO | [4] |
| Example solves, OH | Roux OH Example Solves (Rouxles [Anto]) | https://www.youtube.com/watch?v=V6KIVvStcQY | [4] |
| Example solves | Example Solves Archive: 13 Roux 3x3 and 8 Roux OH videos tagged with the solver's average (6.6–18 s) | see [108] | [4] |
| Big cubes | 4x4 Meyer Method Tutorial (Kian Mansour) | https://www.youtube.com/watch?v=-L86AodUUzE | [4][7] |
| Big cubes | How to use Roux Method on 4x4 \| Meyer Method & Lewis Method Tutorial (Rowan Fortier, "Blobinati") | https://www.youtube.com/watch?v=g8ct_J3RRPc | [4] |
| Big cubes | 5x5 meyer m slice edge pairing tutorial / example solves (3DegreeK) | https://www.youtube.com/watch?v=glgDBb8sT4k, https://www.youtube.com/watch?v=PvScTZIxuBo | [4] |
| Big cubes | M-slice 5x5 walkthrough solve. (MichalHalczuk) | https://www.youtube.com/watch?v=UtxCyOa4-6A | [4] |
| Big cubes | M-Slice Edge Pairing \| 4x4 Meyer Example Solves (Knut Peterson) | https://www.youtube.com/watch?v=HjRBCxYdckE | [4] |
| Mindset (not cubing) | Here's Proof You Don't Have Free Will (zuckerabudi), offered for competition nerves | https://youtu.be/jDuyKbJusKY | [7] |
| Lore (joke) | Coaching session with Midori (Zhouheng Sun) | https://youtu.be/pwRMbsdIWHE | [5][7] |

### 8.6 Text sites and channels

- Text sites by creator: Kian Mansour [37]; Anto Kam (rouxl.es) [99]; Michael James Straughan (athefre, cubinghistory) [100][102]; Trevor Scheopner [125]; Gilles Roux [121]; Mark Fiend [122]; Basilio Noris (cheat sheets) [51][77]; SpeedCubeDB [38]; reco.nz [103].
- rouxcubing.com lists 41 Rouxer YouTube channels [4]:
  - A–J: Abhijeet Ghodgaonkar, Akash Sreedharan, Alexey Tsvetkov, CubingWithAlex, Anto Kam (Rouxles), Cordelia Hu, CriticalCubing, Doyul Kim, Dwyane Ramos, Dylan Govic, Eden Robinson-Rechavi, Eff, Elijah La Porte, Fahmi Aulia Rachman, Fouzan Zayn, Hamish Bassig, Iuri Grangeiro, Jasper Murray, Joe Briggs (Shadowslice), Joshua Marais, Jihu Mun.
  - K–Z: Kangaroux, Kavin Tangtartharakul (Guroux), Kian Mansour, Knut Peterson, Kymo Hendriks, Louis de Mendonça (TDM), Magnus Lensch, MOG Cubing, Nicholas Archer, Rayyan Ali, Richard Peng, Reuel Franz, Sean Patrick Villanueva, Sevilz, Shawn Boucké (SpeedCubeReview), Takumi Kato, Teri McAcy, Tyler Gee, Viljo Elo, Zhouheng Sun.
- The Roux Reader credits Zhouheng Sun, Antonio Kam, Vincent Wong and Stanley Su [17].

## 9. Feature-map delta

Against FM §2 [136]. Priorities: **MVP**, **v1**, **later**.

### 9.1 Changed rows

| FM row | Was | Now | Why |
|---|---|---|---|
| 0 · Curriculum: milestone gates | v1 | Static **Roadmap screen MVP** (level ladder, links to modules and lessons); automatic gates stay v1 | The community has no finished text roadmap [2][30]; a static ladder (3.1–3.3) costs little and links each item to a module or lesson |
| 4 · Beginner LSE lessons | MVP | Widen to a **complete beginner course** with mini-drills: notation lesson and move quiz, FB, SB, 2-look corners, LSE (B1–B8). **MVP if 10.1 = yes**; otherwise the beginner LSE lessons stay MVP as FM has them | No finished beginner text exists [3][21][28]; first-time solvers are in the audience [137]. Own text and diagrams on the cube engine |
| 1 · Partial SpeedBLD warm-up | later | **v1**, as a hidden-cube mode of the inspection planner | Reader: "arguably underrated … warmup before solves" [11]; TR uses it as the warm-up role [138]; cheap once INS exists |
| 1 · Piece-tracking trainer | v1 | v1, unchanged; check the upstream code works first | Upstream hides the tab in v2.3 (`src/components/AppView.tsx:83`) |
| 3 · Per-case alg choice; execution timing; OH set | v1 | **User's main alg per case MVP** (typed, validated by the cube model); alternatives, execution timing per case and OH profile (OH hand, OH alg sets, mirroring; app-wide, read by every module) v1 | On-device alg sheets are the most-used learning tool among Roux mains (40 of 49) [115]; every 2H sheet offers several algs per case [45]–[49]; OH sheets exist for both hands [53][54][73] |
| 3 · Recognition-only drill | MVP | MVP, with per-(case, angle) stats and a confusables queue | All-angle recognition is the Intermediate item [2]; confusable pairs [14][112] |
| 2 · SB last pair trainer + SBLP reference | MVP | MVP, on SpeedCubeDB's 65-case taxonomy, front/back slot | Community ids [38][39] |
| 4 · EO (4a) trainer | MVP | MVP, adding the flowchart mode, the LR-edge-control mode for pauseless 4a→4b (E19) and lefty/righty M | Basic LSE item "EO flowchart" [2][76]; rouxl.es's next LSE step is forcing an L/R edge to D during EO [99] |
| 4 · 4c trainer with recognition method | MVP | MVP, adding DFDB prediction questions during 4b | Intermediate item "DFDB" [2][88][89] |
| 2 · CMLL / EO prediction drill at SB last pair | later | Split: **CMLL→LSE effect on alg cards and EO-variant choice v1**; OCLL/CMLL/CP prediction and insert choice for OCLL (C13–C15, C22) stay later | Advanced item "CMLLEO" [2][52][37] |
| 5 · Stats by stage + "what to work on" | later | Benchmark cards (movecount per stage, split balance) **v1**; diagnosis stays as TR plans | Reference numbers exist [125][142][105][106] |

### 9.2 New rows

**0 Foundations and profiles**

| Feature: what / why | Pri | Reuse |
|---|---|---|
| Canonical case ids with alias maps (CMLL, EO groups, SBLS, TCMLL); naming scheme setting. Sources disagree on names (6.3) | MVP | roux-trainers CMLL ids (`src/lib/Algs.tsx:28-72`) |
| M-hand setting (lefty/righty), mirrors U↔U' in EO/EOLR/LSE solutions | MVP | [76][79][80] |
| Touch M/M'/U/U' input (double tap for 2) for every LSE module | MVP | Idea from LaSEr's buttons [84] |
| Move-set profile (2H, OH, no-F) for solver ranking and user input | v1 | Edde's move-set modes [43]; evaluator (`src/lib/Evaluator.tsx`) |
| Recap / coverage selector for every case module | MVP if TR's coverage run is not already MVP | [68][75][138] |
| Recon URL import/export (cubedb, alg.cubing.net, Twizzle) | v1 | [113][117][141] |

**Learn (new section)**

| Feature: what / why | Pri | Reuse |
|---|---|---|
| Glossary (own definitions, linked from every jargon term); beginners get lost in jargon [118][123] | MVP | [5] as the term list |
| Resources: curated outbound links with link-health checks (videos, sheets, sites, community) | MVP | Section 8; links rot (Stadler page moved [94], SB sheet deleted [36]) |
| Technique library (FB, SB, CMLL recognition, LSE, advanced/niche) with level tags; adapted Reader sections with MIT notice | v1 | [9]–[16][17][126] |
| Lore/timeline cards with inventors and years | later | [102] |

**1 First Block**

| Feature: what / why | Pri | Reuse |
|---|---|---|
| Optimal-length ladder with mastery gates (F2) | MVP (setting) | Upstream level sliders (`src/Config.tsx:83-97`) |
| Pair spotting module (F7, F8) | v1 | Solver |
| FB taste, framework spotter, LP prediction, calculate-N (F4–F6, F15) as INS modes | v1 | `analyze_fb` (`src/lib/Analyzer.ts:145-206`) |
| "Find 2+ solutions" and DR-tier ranking in FBLP(+DR) (F11, F12) | v1 | `fbdr` pruner; tier table re-derived [31] |
| Influencing quiz with the Reader's 17 examples (F10) | v1 | MIT text [11][17] |
| Ready-made block decision; approach tags (F9, F18) | later | Solver |

**2 Second Block**

| Feature: what / why | Pri | Reuse |
|---|---|---|
| L/R-sticker masked pair recognition (S5) | v1 | Masks (`src/lib/CubeLib.tsx:777-873`) |
| Basic insert cards and multislot subset (S3, S6) | v1 | `sb`/`lpsb` pruners |
| Approach comparison, pair choice (S7, S8) | later | Solver |
| Twisted/passive insert decision (S12) | later | With TCMLL/ACMLL |

**3 CMLL**

| Feature: what / why | Pri | Reuse |
|---|---|---|
| 1.5-look path (C2) | v1 | Generated from base algs [58] |
| Recognition-system setting: U-sticker default; NMCMLL/ATCRM v1; Straughan, three-corner later (C7) | v1 / later | NMCLL data (`src/lib/Algs.tsx:163-326`) |
| Peeking and hidden-sticker inference modes (C5, C6) | v1 | 3D view |
| Trigger drills: learn an alg as a chain of named triggers, time a named trigger (C21, W12) | v1 | Timer |
| Pinkie Pie (OLLCP), TCMLL, ACMLL (C17–C19) | later | GPL OLLCP partial; solver |

**4 LSE**

| Feature: what / why | Pri | Reuse |
|---|---|---|
| LSE sandbox: full LSE free-solve, efficiency %, QTM-only (E16) | MVP | `lse` pruner; ship precomputed tables [83] |
| Two-step EOLR recognition and reduction stepper (E7, E8) | v1 | `eolr` pruner |
| EO-pattern weights; 6-flip subset with curated non-MU algs (E9, E13) | v1 | [84][85] |
| LSE sprint (E17) | v1 | Timer |
| EZ4c recognition system (E4) | v1 | Own diagrams |
| 4c influencing advanced, Gilles' EO shortcuts (E14, E18) | later | Solver |

**5 Full solve, competition, packs**

| Feature: what / why | Pri | Reuse |
|---|---|---|
| Plan-then-reveal pro-solve study (W4); users ask for it [118] | v1 | `analyze_roux_solve` (`src/lib/Analyzer.ts:59-130`) |
| Linear Roux FMC with step editor (W2, W3) | v1 | Cube model; analyzer |
| Example-solve watch list sorted by solver average | v1 | Links [108] |
| Weekly comp mode (W8) | later (needs Knut's agreement and a scramble feed) | [1][116] |
| Competition-mode pre-solve screen (W10) | later | Own text |
| Big-cube Roux pack (W13) | later | Separate scope (10.4) |
| CP-first OH pack (W14) | later, only if 10.5 says yes | — |

## 10. Scope questions for the owner

Each has a recommended answer.

1. **Teach from zero?** The hubs say Roux works as a first method [7], but no complete beginner text exists [3][21][28]. *Recommend: yes.* Ship a short interactive beginner course in the MVP: notation, FB, SB, 2-look corners, 4a/4b/4c. Write our own text; do not copy the tutorial, which has no licence [29].
2. **How to serve people switching from CFOP without CFOP content?** *Recommend:* one "Coming from another method" lesson in neutral wording ("if you can already solve the cube"). Allowed: x2y, F/B inserts, and reusing algs the user may already know (Sune, T-perm, J-perm) as Roux beginner algs. Not allowed: CFOP steps, sets or comparisons. Drop the Reader's OLL/PLL comparison and its "Switching from CFOP" page [13][9].
3. **OH?** Roux is over-represented in OH: 34 % of the WCA OH average top 100 [1], and OH is one of the four weekly-comp events [1]. *Recommend: in scope as a profile* (hand, alg sets, move-set ranking) from v1. OH-only niche sets such as TCMLL-N go in later.
4. **Big-cube Roux** (Meyer, Stadler, CR4, parity CMLL/4c)? The weekly comp allows only Roux-based 4x4 methods [1]. 18 of the survey's 21 Meyer users are Roux mains [115], but Fengyuan Lou thinks most Roux users are better off with another big-cube method [97]. *Recommend: later*, as an optional pack: a 4x4 split timer plus parity CMLL and parity 4c drills. Keep the 3x3 core first.
5. **Where is the Roux boundary for variants?**
   - **In, as niche techniques:** NMB/NMC, CPFB, Pinkie Pie, TCMLL, ACMLL, conjugated CMLL. The Techniques Table lists all of them [2]. The weekly comp names ZBroux and "Pinky Pie" as allowed Roux variants [1]. ZBroux was not researched.
   - **Out, apart from a glossary entry and a link:** methods with their own steps, namely CEOR, YruRU, Nautilus (including "Nautilus L5E"), 42, L5C/L7E and 2GLL [41][43][92][71][143]. Conjugated CMLL stays in as a technique even though the 42 method does not. Also out: SpeedHeise2, WaterRoux and Pikashit, which the r/rouxcubing wiki lists with ZBroux and 42 as similar or derived methods; they were not researched [118].
   - *Recommend this split.*
6. **Videos: link or embed?** *Recommend: link out* to the YouTube app or browser, with title, channel, level and a link check. Embedding needs a WebView player, fights the offline-first stance, and gains little [138]. Never download or re-host videos ([56][70] are personal shares).
7. **Content that needs permission.**
   - Knut Peterson (Discord `@_knut_`): the Techniques Table wording and the weekly-comp scramble feed [1][2].
   - Zhouheng Sun: the 10 FB+DR examples [31].
   - Anto Kam: EO flowchart and DFDB images. Redraw them anyway [76][89].
   - Kian Mansour: the SB and CMLL docs [35][37].
   - The authors of the EOLR, TCMLL and OH CMLL sheets [78][68][53].
   - reco.nz: any bundled reconstructions [103]. eff: the example-solve set [107].
   - *Recommend:* ship with solver-generated and user-entered data; send polite requests; add sets only after written consent; credit inventors everywhere.
8. **Embed the Roux Reader?** It is MIT [17]. *Recommend:* adapt selected sections (FB chapter, orientation, DR-first, CMLL recognition, EOLRb insertions and influencing, FAQ milestones) with the MIT notice on the Licences screen. Do not mirror the whole book: it is frozen since 2022 and half empty [9][17].
9. **Default recognition system and naming.** *Recommend:* U-sticker + pattern recognition by default; group+slot ids with Kian/SpeedCubeDB pattern names shown; other schemes as settings (6.3).
10. **M-hand and OH-hand defaults.** Sheets exist for both hands [76][79][80][53][54]. *Recommend:* ask both during onboarding, changeable later.
11. **Community links (Discord, subreddit, Facebook).** *Recommend:* one Community card with outbound links only. Do not scrape community content: Reddit and Facebook posts belong to their authors [118][119].
12. **Mindset and lore content.** *Recommend:* link only, with a user-editable mantra field. Do not bundle the Dune litany (copyrighted novel text) [128]. Keep "Midori" as a glossary joke with a link [5].

## 11. Sources and open questions

### Sources

Hubs and their repositories
- [1] rouxcubing.com, Home (What is Roux, competitiveness charts, Weekly Competition #607): https://rouxcubing.com/#/
- [2] rouxcubing.com, Rouxdmap to Sub-X (Roux Techniques Table): https://rouxcubing.com/#/rouxdmap
- [3] rouxcubing.com, Rouxtorials: https://rouxcubing.com/#/rouxtorials
- [4] rouxcubing.com, Rouxsources (sub-tabs `?mainTab=N&subTab=M`; F2B, CMLL, LSE, Big Cubes, Misc): https://rouxcubing.com/#/rouxsources
- [5] rouxcubing.com, Roux Acronyms and Definitions: https://rouxcubing.com/#/rouxsources?mainTab=4&subTab=5
- [6] KnutP/RouxCubingWebsite (source; no licence found): https://github.com/KnutP/RouxCubingWebsite
- [7] Roux Resources: https://resources.rouxers.com/
- [8] rouxers/resources (repo and history): https://github.com/rouxers/resources
- [9] The Roux Reader, Overview: https://book.rouxers.com/en/overview.html
- [10] The Roux Reader, Blockbuilding: https://book.rouxers.com/en/blockbuilding.html
- [11] The Roux Reader, First Block: https://book.rouxers.com/en/fb.html
- [12] The Roux Reader, Second Block: https://book.rouxers.com/en/sb.html
- [13] The Roux Reader, L10P: https://book.rouxers.com/en/l10p.html
- [14] The Roux Reader, CMLL: https://book.rouxers.com/en/cmll.html
- [15] The Roux Reader, LSE: https://book.rouxers.com/en/lse.html
- [16] The Roux Reader, FAQ: https://book.rouxers.com/en/faqs.html
- [17] rouxers/book (MIT, "Copyright (c) 2020 Rouxers"; contributors; last deploy 2022-01-15): https://github.com/rouxers/book
- [18] The Roux Reader, Chinese edition: https://book.rouxers.com/cn/index.html
- [19] Roux Tutorial, Introduction: https://tutorial.rouxers.com/introduction.html
- [20] Roux Tutorial, Notation: https://tutorial.rouxers.com/beginners/notation.html
- [21] Roux Tutorial, Beginner First Block: https://tutorial.rouxers.com/beginners/first-block.html
- [22] Roux Tutorial, Beginner CMLL: https://tutorial.rouxers.com/beginners/cmll.html
- [23] Roux Tutorial, Beginner LSE: https://tutorial.rouxers.com/beginners/lse.html
- [24] Roux Tutorial, Intermediate First Block: https://tutorial.rouxers.com/intermediate/first-block.html
- [25] Roux Tutorial, Intermediate Second Block: https://tutorial.rouxers.com/intermediate/second-block.html
- [26] Roux Tutorial, Intermediate CMLL: https://tutorial.rouxers.com/intermediate/cmll.html
- [27] Roux Tutorial, Intermediate LSE: https://tutorial.rouxers.com/intermediate/lse.html
- [28] Roux Tutorial, Beginner Second Block (empty): https://tutorial.rouxers.com/beginners/second-block.html
- [29] rouxers/tutorial (no licence): https://github.com/rouxers/tutorial
- [30] Rouxles/roux-tutorial (predecessor, drafts; no licence): https://github.com/Rouxles/roux-tutorial
- [31] Zhouheng Sun, The FB+DR Guide (PDF): https://raw.githubusercontent.com/Rouxles/roux-tutorial/master/fbdr/Zhouheng%27s%20FBDR%20Guide.pdf

First Block and Second Block resources
- [32] Noblion, Roux Beginner Supplemental Guide: https://docs.google.com/document/d/12AC3GM32M9U1IGUvFjkTOH6pq2GDPtT-v_NJgz9a4zs/
- [33] Shadowslice, Frameworks: a structured approach to blocks: https://docs.google.com/document/d/1JPLt7uJmPr6sn__4jJxvWsiVTiaexGm1vt95mRtN-6I
- [34] Fahmi and Neo, recognising blocks via L/R stickers (incl. hidden SB Theory and List tabs): https://docs.google.com/spreadsheets/d/16Ow0vWEFsGoPAGWD8LSUPrQ_L4BGskEa4gXBAnuGlvk
- [35] Kian and Louis, Second Block Algs: https://docs.google.com/document/d/1bX50jAOM_veHsVJeLYGvSrH4uW9xrubep1zOSLdE4Vk/edit
- [36] SB Last Slot Algs (deleted, HTTP 410): https://docs.google.com/spreadsheets/d/1xd1LMv-0EOlEEOOPODLPzFSDZzFmd4D_KVxpgnkHrJ8/edit
- [37] Kian Mansour, Roux Method Guide (Home, Roux Tutorial, Second Block, CMLL, OH CMLL, EO, EOLR): https://sites.google.com/view/kianroux/home
- [38] SpeedCubeDB, Second Block Last Slot: https://speedcubedb.com/a/3x3/SBLS
- [39] Azusa, OH SBLP - Front and Back (PDF): https://drive.google.com/file/d/1MeXeYMFU0OQKWMfHJk1Joa3R4qOFK4c4/view
- [40] Edde, RUroux/CPRoux spreadsheet: https://docs.google.com/spreadsheets/d/1JTHl4hlsvxWD9bOW8EJQQwMKfhwuipw1IXYVtKcvhe8/edit
- [41] YruRU method (Yash Mehta): https://devagio.github.io/YruRU/index.html
- [42] SpeedSolving wiki, CEOR (blocked): https://www.speedsolving.com/wiki/index.php?title=CEOR
- [43] Edde, CEOR trainer: https://edde.github.io/CEORtrainer/
- [44] Cubegrass block trainer: https://cubegrass.appspot.com/block_trainer/

CMLL resources
- [45] eff, 2H CMLL list: https://docs.google.com/spreadsheets/d/1uMmjhGZzPCxPPODiU-N7juNagW6QYh2HKSFpF7XB42A
- [46] Anto, 2H CMLL List: https://docs.google.com/spreadsheets/d/1uc5V3G-kRH3qV5b6Lq1yzTjnqSMS3D3dZcsCIOiJqcI
- [47] Teri, 2H CMLL (lefty and righty LSE tabs): https://docs.google.com/spreadsheets/d/1sGeA4uf-f_hB4Ox7gCybm40SFy_e86_C7vP5LuD7WlE
- [48] Sevilz, 2H CMLL (LSE edge-effect images, statistics): https://docs.google.com/spreadsheets/d/1MCWLpopbQbQ5IW_g3m90keChY-q1jdx9w9yzSpQaGzY
- [49] John Mark Grimes, CMLL sheet (fingertrick notation, video timestamps): https://docs.google.com/spreadsheets/d/1T94xPQU6-bIXUpT0GbdVNFLbpryEgXEmeJcI77SejEk
- [50] Atlas, Roux Method Algorithms: https://docs.google.com/spreadsheets/d/1Lbuks-O0WCwJmqdrkWzmBAmfUpNyvPu75rGlGnKyw6s
- [51] Basilio Noris, CMLL cheat sheet: https://basilio.dev/cubing/images/ALGS-CMLL.jpg
- [52] Hubert Badocha, CMLL Visualiser: https://badochov.github.io/cmll-visualiser/
- [53] Anto Kam, OH CMLL List: https://docs.google.com/spreadsheets/d/1WHnksK4yyv63sv3Es-PuKfnFRtZGafqp6rEuEfOx-cg/edit
- [54] 刘埃尔, Right-Handed OH CMLL List: https://docs.google.com/spreadsheets/d/1n0RW9g889rg7Lk4xh1WpQlzwM-QtTgnz5O1Oj-WEXBs/edit
- [55] Sevilz, OH CMLL: https://docs.google.com/spreadsheets/d/13B4247VUgoPhPsoEKktQVHsxiPFGuzAOmAXTxc-IsiY/edit
- [56] Iuri, OH CMLL executions (Drive folder): https://drive.google.com/drive/folders/1owfcX3SJSXQr5_Ly9Sex_4rSMLZ-gIEb
- [57] 2 Look CMLL (two identical copies): https://docs.google.com/spreadsheets/d/1pful9u4oOiB2mzqICBZYJyHWF2VzTaRoha9fq8R8oTw and https://docs.google.com/spreadsheets/d/1EdCeGlotJ76MyVw02_N1El_SXXSIMJRhYvZn4gZkQk8
- [58] Derin Eryilmaz, 1.5 Look CMLL Algorithms: https://docs.google.com/document/d/1YLIB3U-1UKNpTn-XiN-8VXmC8lQgeYgLyEpELgVCc1w
- [59] Athefre, CMLL Prediction (incl. OCLL prediction, Intuitive CP): https://sites.google.com/site/athefre/roux/cmll-prediction
- [60] Multi angle CxLL recog (Drive folder): https://drive.google.com/drive/folders/1eysbDbEtKzKpwRBN_oih-xaKRBOjRIKA
- [61] Iuri Grangeiro and Alex Friedman, L CMLL recognition: https://docs.google.com/document/d/1_GJF5lYnAWwxxkxLlBSnaHFayjEVd8-PG7tnDUwk71o/edit
- [62] Sevilz, OLLCP sheet: https://docs.google.com/spreadsheets/d/1jRng_UP6y8zqwbC1ptVqOlqk7FKq0AU_E6CJGLUUJmY
- [63] SpeedCubeDB, OLLCP: https://speedcubedb.com/a/3x3/OLLCP
- [64] Athefre, NMCMLL List: https://docs.google.com/spreadsheets/d/1BUt299RvoSDdtUEEMnGALo7jVqHnuhZS64-KZZ6TEKI/edit
- [65] Athefre, NMCMLL page: https://sites.google.com/site/athefre/roux/roux
- [66] Athefre, ACMLL page: https://sites.google.com/site/athefre/roux/acmll
- [67] ACMLL List (legacy): https://docs.google.com/spreadsheets/d/1z3GGqY9_J4124mWYgYjH_QuwaqMiGVB1WHxL8IemVeU
- [68] franz, cordelia, tyler, alvaro, TCMLL-N OH: https://docs.google.com/spreadsheets/d/1ucLxYATDLN82UuJuEaDLFuKhaVHlKUaqicId2_LNISA
- [69] Montu, TCMLL algs: https://docs.google.com/document/d/10wenyuYdJhJd70lka0m01SdCK1OK2OgHGuaPCFMCQc4/edit
- [70] Cordelia, TCMLL 1a execution compilation (Dropbox): https://www.dropbox.com/scl/fi/q9rx51ex5p8so2mrn7xxx/tcmll-1a-compilation-full.mp4?rlkey=ce2rok823r3hwmbu8e8l0afz1
- [71] Athefre, L5C and SL5C: https://docs.google.com/spreadsheets/d/1ZL1U8cAuzgVXe9xDMtpfXk5PLvG0PpcKULKiPHIDLG0
- [72] SpeedSolving wiki, Conjugated CxLL (blocked): https://www.speedsolving.com/wiki/index.php?title=Conjugated_CxLL
- [73] Tao Yu, Alg Trainer (MIT): https://tao-yu.github.io/Alg-Trainer/
- [74] Kirjava, kSim 2: http://ksim.kirjava.xyz/
- [75] Roman Strakhov, ZBLL trainer: https://bestsiteever.ru/zbll/ (now https://www.bestsiteever.net/zbll)

LSE resources
- [76] Anto Kam, LSE Edge Orientation (4a) flowchart, lefty and righty: https://rouxl.es/images/LSEEO.png
- [77] Basilio Noris, EOLR cheat sheet (site also hosts Audio Timer and Recon Viewer): https://basilio.dev/cubing/images/ALGS-EOLR.jpg
- [78] EOLR (doc #1): https://docs.google.com/document/d/1dvGERLfN-0rVwN914HH1zRPHLfdM6d5rPfe0HxOMK08
- [79] TDM (Louis de Mendonça), full 2H EOLR, right-hand M: https://docs.google.com/document/d/1CV3DEwIZYn0g4bnXDN6-2bxNpc7I0woFL1tnslhYQr4
- [80] TDM, full 2H EOLR, left-hand M: https://docs.google.com/document/d/15At5K7N1Yg1Qfh0_KZyUJZhBqLJVmXdPvLN1hmrRYoc
- [81] EOLR & EOLRb (working doc): https://docs.google.com/document/d/1OQ7hcB06bYbr8eH9SLTW2W5tozk9_nJ5MzqNgxHOnUk
- [82] PiKeeper, EOLRb: https://docs.google.com/document/d/1svAgKNnvBMtAfZ_K_RL-7XcK1WlrgP3cP8ph907Zaq4
- [83] Jade, EOLRb trainer: https://enbyne.tech/tools/EOLRb
- [84] Jade, LaSEr v0.1: https://enbyne.tech/tools/laser
- [85] Neo (cuar03), non-MU EOLRbs: https://cuar03.github.io/web/3x3/EOLRb/index.html
- [86] EOLR described (doc #2): https://docs.google.com/document/d/1rb5M9_5CTlozLu9acFIgqq9LYYKaUdnMDX0vcLOURT4/edit
- [87] Optimal EOLR: https://docs.google.com/spreadsheets/d/1sLrjfuVFF41es4W371zeKBR1n98-TLXCigrqrRhnjtQ/edit
- [88] Anto Kam, DFDB recognition for pauseless 4c: https://docs.google.com/document/d/1L9v-iWOKkdnymCYud0IWmAT50cEWwWtxU443I4b21AA/edit
- [89] Anto Kam, DFDB guide image: https://rouxl.es/images/DFDB.png
- [90] "Made a Roux 4c cycle BU recog guide" (r/Cubers; blocked): https://www.reddit.com/r/Cubers/comments/69rubn/made_a_roux_4c_cycle_bu_recog_guide/
- [91] John Mark Grimes, EZ4c diagram: https://drive.google.com/file/d/1Z_oF4Sv1A9YvTzHFD78wp_kGHGmO4Hrm/view
- [92] Nautilus method, L5E: https://sites.google.com/view/nautilus-method/l5e

Big cubes
- [93] CriticalCubing, CR4 4x4 method: https://docs.google.com/document/d/150UxosBFn5Trhi16wa4qv2u06mTweXHSsDH6iv5JeRc/edit
- [94] Thomas Stadler, 4x4 Stadler Methode (moved; old URL 404): https://www.speedcubing.ch/l%C3%B6sungsanleitungen-1/4x4-stadler-methode/
- [95] TDM/Louis, Bigcube Parity OLL/PLL and C(M)LL: https://docs.google.com/spreadsheets/d/1KY3DT86JLpPUDb98km4ck_hrpdRooC8vg2U8Y1zJipk/edit
- [96] TDM, Bigcube parity 4c: https://docs.google.com/document/d/1iT22CcA3wGJHfoAQPriFbNNaTYJoUKYUOskYINH5Ypw/edit
- [97] Fengyuan Lou, guide to Meyer (Pastebin): https://pastebin.com/av0MbjDe

Reference sites, history
- [98] SpeedSolving wiki, Roux method (blocked): https://www.speedsolving.com/wiki/index.php/Roux_method
- [99] Anto Kam, Rouxles (home, FB, SB, CMLL, LSE pages): https://rouxl.es/
- [100] Michael James Straughan, athefre site: https://sites.google.com/site/athefre/main
- [101] Athefre, Straughan Recognition (incl. Three Sticker Tracking): https://sites.google.com/site/athefre/recognition/straughan-recognition
- [102] CubingHistory.com, Roux: https://www.cubinghistory.com/3x3/Methods/Roux

Reconstructions and databases
- [103] reco.nz, Roux filter: https://reco.nz/?sort=id_desc&Roux=on
- [104] r/rouxles (blocked): https://www.reddit.com/r/rouxles/
- [105] eff, Cubeast ao100 share: https://app.cubeast.com/public/share/oQfw8VwiRSwAU7MXcyYaj4
- [106] Fahmi (UNO), Cubeast ao100 share: https://cbst.link/s/uvbYM9h8xn79c5rjRSfsZr
- [107] eff, 50 2H Roux example solves: https://docs.google.com/spreadsheets/d/1WvcXC7tXIxxnzPmsMBbV43SuVs33oaUW3LSDFzVAriQ/edit
- [108] Example Solves Archive: https://docs.google.com/spreadsheets/d/1XfbJ7zC_GRNjeLxmcUX_lyGmNhLqb-coG8Jl94UGOGA/edit
- [109] Roux Reconstructions (technique-tag taxonomy): https://docs.google.com/spreadsheets/d/16OiJavT44OOGgxPjxk8sJ4NvTnfqM8oz2miVH0rB3bs/edit
- [110] CubeSolves: http://cubesolv.es
- [111] SpeedCubeDB reconstruction index (404): http://speedcubedb.com/r/index/
- [112] SpeedCubeDB, CMLL U X and U Bottom Row: https://speedcubedb.com/a/3x3/CMLL/U_X and https://speedcubedb.com/a/3x3/CMLL/U_Bottom_Row
- [113] cubedb.net: https://cubedb.net/
- [114] VisualCube (Conrad Rider; LGPL): http://cube.rider.biz/visualcube.php

Community and general
- [115] Basilio Noris et al., Cubers Megasurvey 6 (2022 data): https://basilio.dev/cubing/megasurvey6/
- [116] RMS weekly comp participation summary and submission forms: https://bit.ly/RMSWeeklyCompSummary ; https://forms.gle/H5xoCWvGKnZ92fQt8 (3x3), https://forms.gle/UgRLW3K1d5KkMGfM9 (OH), https://forms.gle/1P9VUgZmA1pibwvL9 (FMC), https://forms.gle/bHNjncvEcPvFWBP2A (4x4)
- [117] Anomic, winning weekly FMC solution (alg.cubing.net): https://bit.ly/4zTaaa7
- [118] r/rouxcubing (wiki, 2022 resource thread, feed): https://www.reddit.com/r/rouxcubing/
- [119] Roux Facebook group (login wall): https://www.facebook.com/groups/1482303418674425/
- [120] CubingMethods, Roux (blocked): https://www.cubingmethods.com/en/roux and https://www.cubingmethods.com/en/roux/tutorial
- [121] Gilles Roux, original method pages: http://grrroux.free.fr/method/Intro.html
- [122] Mark Fiend, Learning the Roux method: http://www.markfiend.com/roux.html
- [123] Ruwix, Roux method tutorial: https://ruwix.com/the-rubiks-cube/different-rubiks-cube-solving-methods/roux-method/
- [124] Rubik's Cube Wiki (Fandom), Roux Method (blocked): https://rubiks.fandom.com/wiki/Roux_Method
- [125] Trevor Scheopner, How to Solve a Rubik's Cube using the Roux Method (incl. FB/SB/LSE solvers): https://www.scheopner.com/cube/index.php
- [126] Sebastiano Tronto, Fewest Moves Tutorial (CC BY 4.0), §2.1: https://fmcsolves.cubing.net/fmc_tutorial_ENG.pdf#section.2.1
- [127] Jayden McNeill, on gatekeeping information (Pastebin): https://pastebin.com/rcczqYWT
- [128] Litany Against Fear (Dune wiki; blocked): https://dune.fandom.com/wiki/Litany_Against_Fear
- [129] The old man lost his horse (Wikipedia, CC BY-SA 4.0; its English translation is quoted from Larre et al. 1993): https://en.wikipedia.org/wiki/The_old_man_lost_his_horse#Translation
- [130] Last Layer Corner Orientation chart (imgur): https://i.imgur.com/IIYopnA.png
- [131] Last Layer Corner Permutation chart (imgur): https://i.imgur.com/BgTHffg.png
- [132] LSE EO flowchart, righty-M half (imgur): https://i.imgur.com/97lzRoZ.png
- [133] Animated move GIF (Ello; dead host): https://assets1.ello.co/uploads/asset/attachment/4123434/ello-optimized-81fbd399.gif
- [134] SpeedCubeShop weekly-comp sponsor link (lands on the store homepage): https://speedcubeshop.com/rouxcomp
- [135] Roux Method Speedsolvers Discord (not read): https://discord.com/invite/VJunGCZBGZ

Project files and code
- [136] roux-training-features.md, §2 feature map: [roux-training-features.md](roux-training-features.md)
- [137] nauroth/rouxtine, HANDOFF: https://github.com/nauroth/rouxtine/blob/main/docs/HANDOFF.md
- [138] nauroth/rouxtine, training-regimens.md: https://github.com/nauroth/rouxtine/blob/main/docs/research/training-regimens.md
- [139] onionhoney/roux-trainers: https://github.com/onionhoney/roux-trainers
- [140] eff, smart-roux-timer (GPL-3.0): https://github.com/eff917/smart-roux-timer
- [141] Twizzle editor (cubing.js): https://alpha.twizzle.net/edit/
- [142] onionhoney, Roux movecount stats: https://docs.google.com/spreadsheets/d/1EectP3O_qwQp_2WohbDrlohxhs2ee17sjFfxZr_-D1g/edit
- [143] CubeAlgs101 (with MikeZB, USBB), Advanced CEOR: https://docs.google.com/spreadsheets/d/1F0zPKlARw3GUTODys_wrtfP7cqgzS7Kse_Cwv67XBQA/edit

### Open questions / unverified

- **Videos were not watched.** Every claim about a video's content comes from its title, its link text and where it is linked. Level tags in section 8 are inferred.
- **Blocked sources:**
  - SpeedSolving wiki pages (Roux method, CEOR, Conjugated CxLL, NMCMLL), the BU guide on Reddit, r/rouxles, the Facebook group, cubingmethods.com and the Fandom wiki could not be read [42][72][90][98][104][119][120][124]. Their content and licences are unknown.
  - The BU system description rests on a search snippet only.
- **rouxcubing.com's licence** was checked through GitHub search results and the published source map, not the repo itself (API blocked) [6].
- **rouxers/tutorial's 3 open issues** were not read [29].
- **Authorship is unconfirmed** for:
  - the EOLR doc #1 [78] and "EOLR described" (its "LSETool" author is unidentified) [86]
  - the Optimal EOLR sheet (probably John Mark Grimes) [87]
  - the 2-look CMLL sheet (probably from Anto's template) [57]
  - the SB Algs doc (attributed to Kian and Louis by link text only) [35].
- **Unclear meanings:** "1a" in Cordelia's TCMLL compilation [70]; "raise the dot" in the EZ4c diagram [91]; the exact trigger condition for Pinkie Pie (Kian's video not watched) [4].
- **ZBroux** is named as an allowed variant in the weekly comp [1] but was not researched.
- **Competitiveness figures differ by date.** rouxcubing shows 6 % (3x3) and 34 % (OH) of the top-100 averages [1]; Scheopner, in July 2025, gives 6 and 32 of 100 [125]. Re-check before showing either in the app.
- **The Megasurvey's Roux subgroup is small** (49 Roux mains) [115].
- **onionhoney's hidden tracking trainer:** whether hiding it was intentional (it was labelled Beta) is unknown, and whether `TrackerView.tsx` still works was not tested (`src/components/AppView.tsx:83`).
- **Permissions:** whether Knut, Zhouheng, Anto, Kian or the sheet authors will grant reuse is unknown (10.7).
- **The DR tier list** was re-derived by a BFS over `<R,U,M,r>` during this research and matched the guide [31]. The app's own solver must reproduce it before shipping.
- **Archived threads not read:** the 2012 "Roux Misoriented Centers Table" thread (now only on archive.org) and other archived speedsolving threads cited by cubinghistory [100][102].
- **Cubedex's CMLL and EOLR SRS** is taken from training-regimens.md [138] and was not re-checked here.
