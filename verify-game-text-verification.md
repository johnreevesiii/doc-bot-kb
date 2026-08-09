# Adversarial Verification — `game-text` subsystem

**Target doc:** `C:/DerbyOwnersClub/_core/areas/game-text.md`
**Method:** Independent re-extraction from the real `.ic22` bytes via python3 (exact port of
DOC-ROM-Studio `parseBlock`, EUC-JP NUL-delimited decoder). All four 4 MB ROMs.
**Verdict:** mostly-solid. The core claims (encoding, framing, the entire 26-block Rev C table
totaling 1705, the punctuation/copyright edits, JP cluster map, placeholder regex) reproduce
from the bytes. Two attribution/number nits found and corrected below.

---

## Confirmed (re-extracted from bytes)

### §1 Encoding & framing — CONFIRMED (conf 0.98)
- EN NUL-terminated ASCII, JP EUC-JP NUL-terminated. Verified.
- EN newline example @0x129FF4 Rev C: bytes `57 6f 77 21 ... 67 72 65 61 74 21 0a 2e 2e 2e 42 75 74 ...`
  decode = `"Wow! It's great!\n...But your horse is running away.\x00"`. EXACT.
- JP newline example @0xEC018 DOC2000: bytes `25 73 a4 cf a1 a2 ... 0a 20 20 a5 ec ...`,
  EUC-JP decode = `%sは、初めての\n  レースで精一杯頑張りました。`. EXACT (newline + 2 spaces).
- Kana byte claims EXACT: `は`=`a4cf`, `、`=`a1a2`, `。`=`a1a3`.
- 0x0F prefix @0x12B4A9 Rev C: `0f 0f 0f "You should check the other horse."`. EXACT.
  **EXTEND:** the byte right after that string's NUL is `03 03 "You ne..."` — there is a SECOND
  prefix family (0x03 repeats) alongside 0x0F. And the copyright neighbor uses `0f ff 0f` (0xFF
  between 0x0F). So the leading-attribute bytes are at least {0x0F, 0x03, 0xFF} — richer than the
  doc's "0x0F only". Meaning still unknown (display/face/color attribute), but the byte set is wider.

### §2 Placeholders — CONFIRMED (conf 0.95)
- `specsOf()` regex in DOC-ROM-Studio is literally `/%[-0-9.]*[sdxcu%]/g`. EXACT match to doc.
- `writeStr` cap logic exists (null-terminated in-place writer). Confirmed.
- All placeholder TYPES exist in Rev C (whole-ROM census): `%s`×312, `%d`×111, `%2d`×33, `%%`×17,
  `%02d`×13, `%1d`×8, `%u`×6, `%3d`×6, `%04x`×6, `%0d`×4, `%x`×5. So %s/%d/%0d/%1d/%2d all present.
- **NIT (number):** doc's per-block figure "%s ×207 in the 5 largest EN blocks". My exact-port
  census of the 5 largest blocks (4,2,1,11,13) gives **%s ×187, %d ×1** — not 207/3/1/1. The
  *kinds* are right and %s dominance is right; the exact tallies depend on which block set / whether
  newline-split fragments are counted. Low-severity.

### §3 EN Rev C block table — FULLY CONFIRMED (conf 0.99)
Re-counted all 26 blocks with an EXACT port of DOC-ROM-Studio `parseBlock` (printable run
0x20–0x7E; 0x0A is a SEPARATOR not in-string; min length 2). **All 26 counts match and total = 1705.**

| # | Block | start | end | claim | got |
|--|--|--|--|--|--|
| 1 | Horse Race Comments | 0x104548 | 0x107DFA | 221 | 221 ✓ |
| 2 | Trainer & Race Dialogue | 0x128F38 | 0x12B767 | 277 | 277 ✓ |
| 3 | Trainer Comments (2) | 0x122FA0 | 0x123740 | 56 | 56 ✓ |
| 4 | Interaction Menu & Result | 0x0E83D0 | 0x0EA492 | 393 | 393 ✓ |
| 5 | Foal / New Horse | 0x107E26 | 0x1081B0 | 37 | 37 ✓ |
| 6 | Feeding | 0x127B78 | 0x12802B | 14 | 14 ✓ |
| 7 | Post-Food | 0x12874C | 0x128D66 | 55 | 55 ✓ |
| 8 | Leg-Type Change | 0x12755C | 0x1277BC | 17 | 17 ✓ |
| 9 | Stable / Event | 0x0CA798 | 0x0CAE20 | 58 | 58 ✓ |
| 10 | Farm & Card Tutorial | 0x103CF0 | 0x104230 | 61 | 61 ✓ |
| 11 | Auto-Suggested Names | 0x10FF70 | 0x11048C | 167 | 167 ✓ |
| 12 | Pre-Race Well-Wishes | 0x128EA8 | 0x128F10 | 8 | 8 ✓ |
| 13 | Banned Names | 0x12B7A2 | 0x12BC17 | 149 | 149 ✓ |
| 14 | Coin/Insert-Card | 0x10FCC8 | 0x10FE70 | 8 | 8 ✓ |
| 15 | Attract Mode | 0x10E804 | 0x10EA00 | 18 | 18 ✓ |
| 16 | G1 Selection | 0x0EBA8C | 0x0EBB77 | 8 | 8 ✓ |
| 17 | Name-Entry Prompt | 0x10FEF4 | 0x10FF6F | 5 | 5 ✓ |
| 18 | Track Condition (Pre) | 0x10EA05 | 0x10EAB2 | 10 | 10 ✓ |
| 19 | Track Condition | 0x10EB3D | 0x10EB62 | 4 | 4 ✓ |
| 20 | Pre-Race Lead/Style | 0x10EAB3 | 0x10EB3C | 9 | 9 ✓ |
| 21 | Leg-Type Labels (Ret) | 0x0EE270 | 0x0EE297 | 3 | 3 ✓ |
| 22 | Retirement Screen | 0x0ED5B4 | 0x0EE0A6 | 93 | 93 ✓ |
| 23 | Retirement SIRE/DAM | 0x0EBEF8 | 0x0EBFF0 | 10 | 10 ✓ |
| 24 | Race Board | 0x0C898C | 0x0C8A64 | 15 | 15 ✓ |
| 25 | Ranking Labels | 0x0C80B8 | 0x0C8120 | 4 | 4 ✓ |
| 26 | Presented/Created By | 0x10E7BC | 0x10E800 | 5 | 5 ✓ |
| | **TOTAL** | | | **1705** | **1705 ✓** |

- DOC-ROM-Studio `GAMETEXT[]` array matches this table label-for-label and offset-for-offset.
- Cited examples confirmed present (not always the *first* string of a block, but in-block):
  e.g. block 16 "Select a horse that will come in first place." and block 17 "The horse name has
  been chosen." both verified.
- **Caveat for re-doers:** if you DON'T mirror parseBlock exactly (e.g. treat 0x0A as in-string),
  counts drift (I got 272/255/148 before fixing). The doc is right because it uses the same routine.

### §4 EN Rev D — CONFIRMED with minor offset imprecision (conf 0.9)
- Debut-race anchor: Rev C 0x128F38 → Rev D 0x12B190 = **shift +0x2258** (doc says +0x2258). ✓
- Banned "anal": Rev D @0x12DCF0 (doc 0x12DCF0). ✓  Rev D banned count = 148 (≈ Rev C 149).
- Coin prompt: Rev D @0x10FF24 (doc 0x10FF24). ✓
- **Punctuation edit CONFIRMED (nice catch):** Rev C `"TO CREATE A NEW HORSE. PRESS..."` (period,
  byte `2e`) vs Rev D `"TO CREATE A NEW HORSE, PRESS..."` (comma, byte `2c`). Verified at the bytes.
- Horse-comment anchor Rev D @0x104C75 (doc 0x104C75) ✓; name-prepend confirmed — `"Jim's Gent"`
  is present in Rev D @0x104CAC (Rev C race comments do not embed the horse name there).
- Presented By: Rev C @0x10E7BC, Rev D @0x10FDFC — both EXACT to doc.
- **Whole-ROM ASCII run census (≥4 chars) — EXACT to doc:** Rev C 18,276 / Rev D 18,758 /
  DOC2000 16,631 / DOC'99 16,063.
- **NIT (offset):** copyright string offsets are slightly off. The strings are right
  (`SEGA/Hitmaker,2001` Rev C, `SEGA,2001,2005` Rev D) but they START at 0x10E7EE (Rev C) and
  0x10FE22 (Rev D); the doc cited 0x10E7FC / 0x10FE27, which land mid-string ("2001" / "2001,2005").
- **NIT (number):** Rev D flat textScan candidate count. ROM-Studio scan regions are correct
  (`[0xC7000,0xCB000],[0xE7000,0xEF000],[0x103000,0x10A000],[0x10F300,0x113000],[0x127000,0x12F000]`).
  My exact-port parse of those regions yields **3702** candidate strings, not the doc's "~3094".
  Soft "~" claim, but off by ~600.

### §5 JP map — CONFIRMED by searching the exact encoded strings (conf 0.92)
DOC 2000 (derbyo2k):
- 栗毛 (chestnut) @0xCA2DA → 0xCA000 region ✓
- 気合いを入れる @0xEB5C0, ほめる @0xEB5D0, おだてる @0xEB5D8 → 0xEB000 interaction menu ✓
- カードエラー @0xCDFB4, 0xF0860, 0x1057E8, 0x115130… → 0xF0000 card-error region present ✓
- コンドルパサー @0x10CE40, オグリキャップ @0x10CE60 → 0x10C000+ name table ✓

DOC '99 (derbyoc):
- あまい (sweet) @0xDC028 → 0xDC000 ✓
- 賞賛と歓喜 (retirement) @0xE1248 → 0xE1000 ✓
- メジロドーベル @0xF9008, メジロパーマー @0xF901A → 0xF8000+ name table ✓
- 東京 (track names) @0xBD878 → near 0xBDCA7 ✓
- カードがなくなりました @0xFD808 → 0xFD000 ✓
- `-n` honorific token @0x1111A0 → 0x111000 region ✓ (also EN block 3 has `-n`/`E5-n`: verified
  `"-n is having fun.\x00E5-n looks very happy."` @0x122FA0).

JPSPEC (from ROM-Studio source, cross-checked):
- DOC2000 textRegion `[0x0C0000,0x140000]`; racing name table `recBase:0x10AD1B, stride 32, count 244`
  — the name strings I found at 0x10CE40 fall inside this, confirming the §5 cross-reference that the
  text name tables overlap the racing NAME table region.
- DOC'99 racing table `recBase:0x0F6902, stride 28, count 244` — consistent with the dense JP text /
  name peak at 0xF0000.

---

## Corrections

1. **§5 DOC'99 textRegion attribution is WRONG.** Doc says "ROM-Studio's JPSPEC.textRegion = ...
   DOC'99 `[0x0B0000,0x130000]`." The actual ROM-Studio source sets DOC'99 `textRegion:[0x0C0000,0x140000]`
   — the SAME as DOC2000. There is no `[0x0B0000,0x130000]` in JPSPEC. The "DOC'99 text sits lower
   (~0xB0000)" idea is a valid density observation (my per-64KB scan shows JP bytes from 0xB0000 with
   peaks at 0xF0000 / 0x110000), but it must NOT be attributed to ROM-Studio's JPSPEC value.

2. **§4 Rev D flat-scan count "~3094"** → re-extraction gives **3702** with the exact parseBlock.

3. **§4 copyright offsets** (0x10E7FC / 0x10FE27) point mid-string; true starts are
   0x10E7EE (Rev C) and 0x10FE22 (Rev D).

4. **§2 per-block %s tally** "207" → my 5-largest-block census gives 187. Kinds correct.

## Extensions (new, beyond the doc)
- Leading attribute-byte set is **{0x0F, 0x03, 0xFF}**, not just 0x0F (e.g. `03 03` prefix at
  0x12B4CA-ish right after the 0x0F string; `0f ff 0f` around the copyright/attract block). The
  engine strips all of them. Good follow-up RE target for the §9 "0x0F semantics" open question.
- Whole-ROM Rev C placeholder spectrum is wider than documented: also `%02d`, `%04x`, `%3d`,
  `%5d`, `%x`, `%u` (numeric width/format and hex specifiers — likely debug/score/time fields).
- DOC'99 JP byte-density by 64KB block (lead 0xA1–0xFE): peaks 0xF0000≈37% (name tables) and
  0x110000≈26% (interaction/help); 0xC0000 and 0x100000 are ~0% (code/binary tables).
