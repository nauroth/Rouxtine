# Research data (work in progress)

Intermediate outputs of the 2026-10-28/29 research workflows behind the notes in
`docs/research/` on `main`. Kept on this side branch so a later session can re-check claims or
re-synthesise without crawling again. Not part of the app.

- `round2/`: community-site crawl (`hub-*.json`), deep reads of linked pages and Google
  Docs/Sheets (`deep-*.json`), YouTube titles via oEmbed (`videos-oembed.tsv`), and the
  claims plus verifier verdicts for licensing, minSdk, delivery and smart cubes.
- `regimens/`: claims and verdicts behind `training-regimens.md`.
- `workflows/`: the workflow scripts that produced them.

Paths inside these files point at a session scratchpad that no longer exists. The raw page
texts were not kept: most sources have no licence, so they must not be republished.
- `audit/`: the 2026-10-29 pre-code audit (`audit-results.json`: four audit lenses, Play
  Billing and analytics research with verdicts) and its 2026-10-31 review
  (`check-2026-10-31.json`). The reviewed fixes were merged into `main` on 2026-10-31.
