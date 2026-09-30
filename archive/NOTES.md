# Archived KB files (2026-09-30)

Moved here on branch `claude/61-archive` after the FAQ decode review (doc-ops #61). Each file below is
wholly superseded: its main claim is wrong, and everything true in it already lives in a current KB file
or on the public FAQ (play.johnreevesiii.com/faq). They are kept for history only.

**Why a subfolder is enough:** DOC Bot's `search_kb` lists only the top-level `*.md` files of the KB
folder and does not recurse, so nothing in `archive/` is indexed. Merging this branch to `master`
changes the live bot (the VPS pulls `master` about every 15 minutes); the top-level correction notes
in the same commit bump the KB modified time, so the bot reloads without a restart or `/kb`.

The G: copies of the same sources were archived the same day under
`G:\My Drive\DerbyOwnersClub\_archive\knowledge-superseded-2026-09-30\` (see ARCHIVE-NOTES.md there).

| File | What it claimed | What is true now | Superseded by |
|---|---|---|---|
| `jp-card-format-spec.md` | DOC 2000 / DOC '99 cards are identity and pedigree only; on-card stats "blocked" | The JP card carries the full career block (internals, externals, breeding values, W/P/S/OUT, hearts, earnings, G1 titles, sex, coat, personality, silks), sealed by a checksum | `jp-card-io.md`, `jp-card.md` sections 11 and 11b; FAQ jp-card-format |
| `jp-findings.md` | May 2026 working notes: "DOC 2000 = IDENTITY + PEDIGREE card, NOT a full-stat card" | Same as above; its kana table is kept in `jp-card.md` section 8 | `jp-card-io.md`, `jp-card.md` |
| `horse-stats-rom-tables.md` | June "training struct": record +0x68 = Speed, +0x6A = Stamina, capped at 63, fed by food column 0 | +0x68 is +104 decimal, the Relationship (hearts) byte; the internals are +64/+68/+72 (decimal) and are clamped at 60 on Rev C. The Speed/Stamina/Sharp label finding is already in `canon.md` | `canon.md`; FAQ stat-ceiling, foods-raise-which-stats |
| `training-competing.md` | Myth #3 hunt, status OPEN / PAUSED, with a "curve table built at runtime" lead | The curve table is static and identical in Rev C and Rev D; the Rev D difference is the post-race growth ceiling (55 per internal, growth off above a 160 total) | FAQ revd-stat-ceiling |

Partly stale files were left in place with a dated correction note at the top (23 files, same commit).
