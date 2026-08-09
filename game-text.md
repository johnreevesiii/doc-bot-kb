# Derby Owners Club — Game Text & String Catalog (all 4 versions)

**KEY:** `game-text`
**Scope:** where all in-game text lives in the 4 NAOMI program ROMs, how it is encoded,
the block structure, string categories/counts, and control-code/placeholder conventions.
**Verification:** every offset/count below was extracted from the real `.ic22` bytes with
python3 (parse routine identical to `DOC-ROM-Studio.html`'s `parseBlock`). Confidence noted per claim.

## ROMs covered
| Tag | Build | File | Lang | Size |
|---|---|---|---|---|
| drbyocwc | WE Rev C | epr-22336c.ic22 | EN ASCII | 0x400000 (4 MB) |
| derbyocw | WE EX Rev D | epr-22336d.ic22 | EN ASCII | 0x400000 |
| derbyo2k | DOC 2000 JP | epr-22284a.ic22 | EUC-JP | 0x400000 |
| derbyoc | DOC '99 JP | epr-22099b.ic22 | EUC-JP | 0x400000 |

---

## 1. Encoding & string framing (VERIFIED on bytes, conf 0.98)

- **EN (Rev C / Rev D):** 7-bit ASCII. Strings are **NUL (0x00) terminated**. Multiple strings
  packed back-to-back, padded with trailing 0x00 to the next record. `DOC-ROM-Studio.parseBlock`
  walks printable runs (0x20–0x7E) and treats anything else as a separator.
- **JP (DOC2000 / DOC'99):** **EUC-JP**. ASCII passes through as single bytes; Japanese chars are
  2-byte JIS X 0208 (lead 0xA1–0xFE) or half-width kana (0x8E + 0xA1–0xDF). NUL terminated.
  Verified e.g. `は`=`a4 cf`, ideographic comma `、`=`a1 a2`, `。`=`a1 a3`.
- **Embedded newline:** byte **0x0A** appears literally inside multi-line strings in BOTH languages.
  - EN example @0x129FF4 (Rev C): `"Wow! It's great!\n...But your horse is running away."`
    raw = `... 67 72 65 61 74 21 **0a** 2e 2e 2e 42 75 74 ...`
  - JP example @0xEC018 (DOC2000): `"%sは、初めての\n  レースで精一杯頑張りました。"`
    raw = `25 73 a4 cf a1 a2 bd e9 a4 e1 a4 c6 a4 ce **0a** 20 20 a5 ec ...` (newline + 2 leading spaces).
- **Leading 0x0F prefix bytes:** some strings carry one-or-more `0x0F` bytes immediately before the
  text, e.g. @0x12B4A9 `0f 0f 0f "You should check the other horse."`. Likely a display/format
  attribute marker the engine strips; `parseBlock` skips them as non-printable. Conf 0.6 on meaning,
  0.95 on existence.
- **No color/markup codes found** in the text proper. The only intentional in-string control code is
  0x0A (newline). A strict census (strings ≥85% printable) of the main EN dialogue block shows 0x0A
  as the dominant sub-0x20 byte (12 in that block); other low bytes come from adjacent binary tables.

## 2. Placeholder (printf-style) conventions (VERIFIED, conf 0.95)
Used identically in EN and JP. From the 5 largest EN Rev C text blocks:
`%s` ×207, `%1d` ×3, `%d` ×1, `%0d` ×1 (plus `%2d` seen in Ranking labels `"  %2d  "`).
- `%s` = horse name / sire / dam substitution (`"%s has stamina."`, JP `"%sは、…"`).
- `%d`/`%0d`/`%1d`/`%2d` = numeric counts (`"Won %d out of %d races"`, `"NEXT RACE IS %0dR %s"`).
- The ROM-Studio editor protects these via `specsOf()` regex `/%[-0-9.]*[sdxcu%]/g` so an edited
  string keeps the same placeholder set.

---

## 3. EN curated block map — Rev C (drbyocwc) — ALL 26 VERIFIED against bytes
Counts are live string counts from `parseBlock(a,b)`. **Total = 1705 strings.** Conf 0.97.

| # | Block | start | end | #strings | example |
|--|--|--|--|--|--|
| 1 | Horse Race Comments | 0x104548 | 0x107DFA | 221 | "He's quite a popular racehorse. He has super stamina." |
| 2 | Trainer & Race Dialogue | 0x128F38 | 0x12B767 | 277 | "It's time to run the Debut race." |
| 3 | Trainer Comments (2) | 0x122FA0 | 0x123740 | 56 | "-n is having fun." |
| 4 | Interaction Menu & Result Text | 0x0E83D0 | 0x0EA492 | 393 | "Psyche up" / "Praise" / "Flatter" |
| 5 | Foal / New Horse Comments | 0x107E26 | 0x1081B0 | 37 | "He is a pretty colt foal." |
| 6 | Feeding Comments | 0x127B78 | 0x12802B | 14 | "It's unusual for a horse to dislike carrots." |
| 7 | Post-Food Comments | 0x12874C | 0x128D66 | 55 | "Speed"/"Stamina"/"Sharp" |
| 8 | Leg-Type Change Messages | 0x12755C | 0x1277BC | 17 | "Your horse's racing style has changed." |
| 9 | Stable / Event Messages | 0x0CA798 | 0x0CAE20 | 58 | "Your horse took off a hood." |
| 10 | Farm & Card Tutorial | 0x103CF0 | 0x104230 | 61 | "This is the farm." |
| 11 | Auto-Suggested Horse Names | 0x10FF70 | 0x11048C | 167 | "Sunny"/"Comical"/"Dark" |
| 12 | Pre-Race Well-Wishes | 0x128EA8 | 0x128F10 | 8 | "Do your best." / "Good Luck!" |
| 13 | Banned Names List | 0x12B7A2 | 0x12BC17 | 149 | "anal"/"anus"/"arse" (profanity filter) |
| 14 | Coin / Insert-Card Prompts | 0x10FCC8 | 0x10FE70 | 8 | "TO CREATE A NEW HORSE. PRESS START BUTTON." |
| 15 | Attract Mode Text | 0x10E804 | 0x10EA00 | 18 | "YOUR HORSE HAS BEEN REGISTERED FOR NEXT RACE." |
| 16 | G1 Selection Screen | 0x0EBA8C | 0x0EBB77 | 8 | "Select a horse that will come in first place." |
| 17 | Name-Entry Prompt | 0x10FEF4 | 0x10FF6F | 5 | "The horse name has been chosen." |
| 18 | Track Condition (Pre-Race) | 0x10EA05 | 0x10EAB2 | 10 | "TRACK CONDITION GOOD" |
| 19 | Track Condition | 0x10EB3D | 0x10EB62 | 4 | "GOOD"/" GOOD TO SOFT"/" SOFT" |
| 20 | Pre-Race Lead / Style Text | 0x10EAB3 | 0x10EB3C | 9 | "FAVORITE"/"FRONT-RUNNER"/"START DASH" |
| 21 | Leg-Type Labels (Retirement) | 0x0EE270 | 0x0EE297 | 3 | "Speed type"/"Stamina type"/"Sharp type" |
| 22 | Retirement Screen Text | 0x0ED5B4 | 0x0EE0A6 | 93 | "START"/"CORNER"/"OUT OF THE BOX" |
| 23 | Retirement Info (SIRE/DAM) | 0x0EBEF8 | 0x0EBFF0 | 10 | "SIRE:%s"/"DAM :%s"/"Life time race results : Won %d out of %d races" |
| 24 | Race Board Text | 0x0C898C | 0x0C8A64 | 15 | "NEXT RACE IS %0dR %s"/"WINNER" |
| 25 | Ranking Screen Labels | 0x0C80B8 | 0x0C8120 | 4 | "TOTAL EARNINGS RANKING" |
| 26 | Presented By / Created By | 0x10E7BC | 0x10E800 | 5 | "Presented By"/"Created By"/"Original Game" |

These are exactly the 26 blocks in `DOC-ROM-Studio.html` `GAMETEXT[]` (confirmed line-for-line).

### EN text "macro regions" (Rev C)
- **0x0C8000–0x0CB000:** UI labels, ranking/board, stable/event messages (near the track tables).
- **0x0E8000–0x0EE300:** interaction menu, G1 select, retirement screens (a big UI cluster).
- **0x103C00–0x108200:** tutorial + the 221 race comments + foal comments (the "flavor text" pool).
- **0x10E700–0x110500:** attract/coin/name-entry/track-condition + the 167 auto-name list.
- **0x122F00–0x12BC20:** all trainer/feeding/leg-type dialogue + the banned-words list.

---

## 4. EN — Rev D (derbyocw) — RELOCATED; mapped by anchor strings (conf 0.9)
Rev D (the 2005 "EX" build) re-laid-out and edited the dialogue, so the Rev C curated ranges
**do not align** (verified: applying Rev C offsets to Rev D yields garbage / wrong strings). That is
why `DOC-ROM-Studio.html` falls back to a flat `textScan` for Rev D
(`[0xC7000,0xCB000],[0xE7000,0xEF000],[0x103000,0x10A000],[0x10F300,0x113000],[0x127000,0x12F000]` →
~3094 candidate strings, noisy because it includes table fragments).

**Anchor offsets located in Rev D (use these to build curated Rev D blocks):**
| Category | Rev C | Rev D | shift |
|--|--|--|--|
| Trainer dialogue ("Debut race") | 0x128F4D | 0x12B1A5 | +0x2258 |
| Trainer dialogue block start | 0x128F38 | ~0x12B190 | +0x2258 |
| Banned list ("anal") | 0x12B7A8 | 0x12DCF0 | +0x2548 |
| Coin prompt ("TO CREATE…") | 0x10FCC8 | 0x10FF24 | +0x025C |
| Presented By | 0x10E7BC | 0x10FDFC | +0x1640 |
| Horse Race Comments ("He's quite a popular…") | 0x104548 | 0x104C75 | +0x072D |
| Retirement SIRE/DAM ("SIRE:%s") | 0x0EBEF8 | 0x0EC328 | +0x0430 |

Verified Rev D blocks (parsed at the derived starts):
- **Trainer & Race Dialogue** ~0x12B190 → 411 strings (same opening line "It's time to run the Debut race.").
- **Banned Names List** 0x12DCF0 → 148 strings, **identical content to Rev C** (anal/anus/arse/…).
- **Coin/Insert-Card Prompts** 0x10FF24 → note the **punctuation edit**: Rev D uses commas
  ("TO CREATE A NEW HORSE**,** PRESS START BUTTON.") where Rev C used periods.
  Rev D also **adds** restricted-race text ("Restricted race. Only horses that have / raced 10 times or less may be entered.").
- **Horse Race Comments** 0x104C75 → 196 strings, and Rev D **prepends the horse name** into the
  comment ("Jim's Gent, He's a tough horse…"; "Vinny, He was a tremendous sprinter…") — Rev C did not.
- **Track condition** text moved to ~0xC8560 ("TRACK CONDITION", "FAVORITE", "FRONT-RUNNER"); the
  string "GOOD TO SOFT" was removed/reworded in Rev D.
- **Copyright:** Rev C = `"SEGA/Hitmaker,2001"` (@0x10E7FC); Rev D = `"SEGA,2001,2005"` (@0x10FE27).

Whole-ROM ASCII-run census (≥4 chars): Rev C 18,276 / Rev D 18,758 (Rev D has more text overall,
consistent with added content). JP: DOC2000 16,631 / DOC'99 16,063 (fewer ASCII runs — most text is
multibyte JP).

---

## 5. JP text map — DOC 2000 (derbyo2k) & DOC '99 (derbyoc) — EUC-JP (conf 0.9)
ROM-Studio's `JPSPEC.textRegion` = DOC2000 `[0x0C0000,0x140000]`, DOC'99 `[0x0B0000,0x130000]`.
A density scan (≥60% of chars are kana/CJK/fullwidth) found the real text clusters:

### DOC 2000 (derbyo2k) dense-JP regions
| region | content (decoded samples) |
|--|--|
| 0xCA000 | coat/sex labels: 栗毛 (chestnut), 特殊 (special), 牝（メス）(mare) |
| 0xEB000 | interaction menu: 気合いを入れる (psyche up), ほめる (praise), おだてる (flatter) |
| 0xEC000–0xED000 | race/result dialogue: "%sは、初めての\n  レースで精一杯頑張りました。" , "%sは%d着でした" |
| 0xF0000 | card-error / OK-button prompts: "カードエラーです … かかりの人を呼んで下さい。" |
| 0x105000–0x108000 | horse comments / sire descriptions; more coat colors 栃栗毛, 青鹿毛 |
| 0x109000 | system: ネットワーク エラー (network error), エントリーのじゅんび中。 |
| 0x10C000–0x113000 | **name tables** (horse / sire / dam katakana): サンデーブランチ, ジェニュイン, メジロロンザン, コンドルパサー, オグリキャップ … |

### DOC '99 (derbyoc) dense-JP regions
| region | content |
|--|--|
| 0xDC000 | feeding/difficulty labels: あまい (sweet), きびしい (strict), ヒヨコ |
| 0xDD000 | interaction results: "%sは、\n うれしそうに遊んでいます。" |
| 0xE1000 | retirement: 賞賛と歓喜, "%s号の生涯は", 血統、育成、鍛錬によって |
| 0xF2000–0xF4000 | horse comments: "現役時代はかなり人気が高く、…", "臆病な性格ですが\n 逃げ足は…" |
| 0xF8000–0xFB000 | **name tables**: メジロドーベル, メジロパーマー, ボルテックス, マンハッタンレディ … |
| 0xBDCA7 | track names: "東京 ダート １２００Ｍ" (Tokyo Dirt 1200M) |
| 0xFD000 | card prompts: "カードがなくなりました。", "セレクトボタンを押しながらチェックボタンで" |
| 0x111000 | interaction (uses `-n` honorific token like EN block 3): "-nは\n嬉しくて興奮しています。" |
| 0x115000–0x116000 | training method help: 単走調教方法説明, プール調教方法説明, "…逆効果です！！" |

**JP placeholders/control codes match EN:** `%s`, `%d` and literal `0x0A` newline all present in JP
strings (verified at 0xEC018). JP also uses fullwidth digits/letters (Ｍ, ＯＫ, Ｇ１) inside text.

**JP profanity filter:** present conceptually but stored differently; the EN `anal/anus/…` ASCII list
has no direct JP analog at the same place (JP name entry is kana — see card spec). Open item.

### Cross-reference to other subsystems
- The JP **name tables** (0x10C000+ in DOC2000, 0xF8000+ in DOC'99) overlap the racing NAME table
  region documented in the stat-table RE; these are the 244 racing names + 84 sire + 84 dam in EUC-JP.
  `jp_mater_names.json` already holds 167 EUC-JP breeding "mater" names from this zone.
- Coat/sex/leg-type label strings (栗毛 / 牝 / 逃げ etc.) are the human-readable side of the
  numeric coat/grade/leg-type fields in the racing stat records.

---

## 6. Category taxonomy (unified across versions)
1. **Dialogue / flavor** — trainer lines, race comments, foal comments, feeding/post-food, leg-type
   change, stable events. Heaviest user of `%s`/`%d` and 0x0A newlines.
2. **Menus / UI labels** — interaction verbs (Psyche up/Praise/Flatter), retirement stat headers
   (START/CORNER/OUT OF THE BOX), ranking & race-board labels, track-condition labels.
3. **Names** — auto-suggested horse names (167, EN), plus the EUC-JP horse/sire/dam name tables (JP).
4. **Banned words** — 149-entry ASCII profanity blocklist (EN, identical Rev C/Rev D) for name entry.
5. **Attract / coin / card** — insert-card prompts, attract loop text, card-error messages.
6. **Branding** — Presented By / Original Game / SEGA copyright (version-stamped).

## 7. Version differences summary
- **Rev C vs Rev D (EN):** Rev D relocated all dialogue (+0x04xx…+0x25xx depending on block),
  edited punctuation (period→comma in prompts), prepended horse names into race comments, added
  restricted-race prompt text, removed/reworded some track-condition strings, and bumped copyright to
  `SEGA,2001,2005`. Banned list unchanged. Rev D has ~480 more ASCII runs overall.
- **DOC2000 vs DOC'99 (JP):** Same categories, different base addresses (DOC'99 text sits lower,
  ~0xB0000–0x118000; DOC2000 ~0xC0000–0x117000). Different name/sire/dam rosters (per the horse DBs).
  DOC'99 uses the `-n` honorific token in interaction results (mirrors EN block 3 "-n is having fun.").

---

## 8. How verified (repro)
Helper scripts in `C:/Users/johnr/AppData/Local/Temp/` (gt_probe.py, gt_revd*.py, gt_jp*.py,
gt_ctrl.py, gt_codes.py): they load each `.ic22`, run the same `parseBlock` as DOC-ROM-Studio for EN,
and a NUL-delimited EUC-JP decoder for JP, print offsets + decoded samples + raw hex. All 26 Rev C
counts, the Rev D anchor offsets, the JP cluster map, and the control-byte/placeholder census above
were produced from the actual ROM bytes (not from the doc alone).

## 9. Open questions
- Exact semantics of leading **0x0F** prefix bytes on some EN strings (display attribute? color/face?).
- A clean **curated Rev D block table** (start/end per category) — anchors are found; ranges need
  finalizing and adding to `DOC-ROM-Studio.html` alongside the existing scan fallback.
- Whether JP has an on-ROM **banned-words** list (EN list is ASCII; JP kana name-entry filter not yet located).
- Whether the JP dialogue blocks have a curated index/pointer table (EN seems to be flat NUL-packed;
  JP may use a pointer array given the multibyte width).
- The `-n` / `E5-n` tokens in EN block 3 ("Trainer Comments (2)") — these look like name-substitution
  control sequences distinct from `%s`; decode the token format.

## 10. Tool ideas this unlocks
- **Rev D curated block table** for DOC-ROM-Studio (replace the noisy flat scan with labeled sections
  using the anchor+shift table in §4).
- **Cross-version string diff tool**: align Rev C↔Rev D by content to auto-generate a translation/edit
  map (find every reworded line, e.g. comma vs period, added restricted-race text).
- **JP↔EN string aligner**: pair EN block N with the JP dense region of the same category (menu verbs,
  race comments, prompts) to build a bilingual glossary and verify a fan re-translation byte-budget.
- **Profanity-list editor / exporter**: dump/edit the 149-word EN banned list (and search for a JP one).
- **Full string extractor/importer** (.po-style): export all 1705 EN strings (and JP) with offset+cap,
  re-import with byte-budget checks (reuse `writeStr` cap logic).
