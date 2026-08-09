# Japanese DOC Card RE — Working Findings (derbyo2k / DOC 2000)

## Reader / container (CONFIRMED)
- Flycast `dev` `card_reader.cpp`: `gameId == " DERBY OWNERS CLUB WE ---------"` → `DerbyBRCardReader` (SanwaCRP1231BR, US WE).
  Else → `DerbyLRCardReader` (SanwaCRP1231LR). derbyo2k / derbyoc / derbyoc2 = LR.
- BR vs LR differ ONLY in the serial status-byte protocol (LR returns simplified '0'/'1'). 
- Card storage identical for both: `TRACK_SIZE = 0x45`, `cardData[0x45*3]` = **207 bytes = 3 tracks x 69**.
- => The `.card` FILE container is identical US vs JP. "Different card-writing system" = encoding + layout of the bytes, not the container or reader file format.

## US card (reference, plaintext)
- Names plaintext ASCII, forward, `00` padding. Track3: `SEGABEF0` @0x8A, `30 10` marker @0x9C.

## JP derbyo2k card (DIFFERENT)
- Names are NOT ASCII. Encoded via a **game-internal 1-byte-per-kana table** (NOT Shift-JIS).
- Padding/terminator byte = **0x7d** (not 0x00).
- NO `SEGABEF0`, NO `30 10` marker observed (but tracks 2-3 dirty — see caveat).

### Track 1 layout (emerging)
- 0x01 = 0x70 constant; 0x0F = 0xfd constant; 0x18 = 0x0d constant; 0x20=0x03, 0x21=0x02 constant.
- 0x25-0x27 = **3 lead bytes** (per-card, unknown field; values look pointer-ish e.g. c1 a7 c2).
- **0x28+ = horse name**, variable length, `0x7d`-padded. (Earlier 0x27 guess was off by one.)
- After name's 7d run, another block then more 7d -- likely SIRE then DAM names (mirrors US a1 = name/sire/dam).
- NOTE: in-game GATE numbers (the boxed 1/4/11 on screen) are NOT the sat folder numbers.

### CHARACTER TABLE — SOLVED (base kana)
Column-major gojuon, 15 consonant-rows per vowel column.
Rows (0-14): (none) K S T N H M Y R W G Z D B P
Columns: a=0x00-0x0e, i=0x0f-0x1d, u=0x1e-0x2c, e=0x2d-0x3b, o=0x3c-0x4a
- Vowels: ア=00 イ=0f ウ=1e エ=2d オ=3c (step 15) — SCREENSHOT-CONFIRMED (sat1 = アイウエオ).
- ト=3f ホ=41 メ=33 レ=35 — CONFIRMED (sat4 = トホメレ, matches gate-1 screenshot).
- sat3 decodes to シヒニヌ (11 14 13 22), sat2 = コ + ext (3d 4c 4d 4f 50).
- Implemented in `jp_decode.py` (reusable). 3/4 cards decode to clean katakana.

### Extended region — SOLVED (small kana + marks)
- 0x4b ァ, 0x4c ィ, 0x4d ゥ, 0x4e ェ, 0x4f ォ, 0x50 ャ, 0x51 ュ, 0x52 ョ, 0x53 ッ, 0x54 ー
- 0x45 = ン (the o-column W slot holds ン, not ヲ) — confirmed by real dam name.
- TODO (low priority, names are katakana): digits, ヴ, ゛゜, punctuation.

### Field layout in track 1 (CONFIRMED)
- 0x00-0x1f: VOLATILE (changes frame-to-frame while running) -- live state + leaked heap. NOT persistent stats.
- 0x18=0x0d, 0x20=0x03, 0x21=0x02, 0x01=0x70 = constant header skeleton; zeros at 0x0a/10/13/19/1c-1e/22-24.
- 0x25-0x27 = 3 lead bytes (vary per horse; purpose TBD, maybe horse ID / pedigree index).
- **0x28 = horse name** (7d-term) -> **sire** (7d-term) -> **dam** (7d-term, runs into volatile tail).
- Tracks 2-3 (0x45-0xCE): NEVER written by DOC 2000 -> pure leaked heap pointers (distinct family per process).

### STRUCTURAL CONCLUSION
DOC 2000 = IDENTITY + PEDIGREE card, NOT a full-stat card (unlike US WE which used all 3 tracks for
stats/earnings/G1/silks). Persistent content = header skeleton + name/sire/dam only. Career/stats likely
live in the cabinet, keyed by the card. => "Editing a JP card" = editing name (+ maybe pedigree); there are
no on-card stat fields like the US format. NEEDS one confirm experiment (see Next).

### Known horses (frozen capture, Flycast closed -> _jp_re/captures/frozen1/)
| card | player name | sire (real DOC) | dam (real DOC) |
|---|---|---|---|
| sat1 | アイウエオ | トリックパワー (Trick Power) | アイリッシュダンス (Irish Dance) |
| sat2 | コィゥォャ (gate-4) | トランスウインド (Trans Wind) | ファイナルレコード (Final Record) |
| sat3 | シヒニヌ | ジェイドロバリー (Jade Robbery) | マンハッタンレディ (Manhattan Lady) |
| sat4 | トホメレ (gate-1) | トランスウインド | シンコウラブリイ (Shinko Lovely) |
Sire/dam decoding to real horse names = full validation of the table + field layout.

## EXTERNAL VALIDATION (DOC_COMPLETE_HORSE_DATABASE_DERBYO2K.md + adjacent CLAUDE.md)
- The OneDrive DOC RE project has a 244-horse DOC 2000 database + a near-complete card spec.
- ALL 7 sire/dam names I decoded match the DOC 2000 mater table EXACTLY (Trick Power #24, Trans Wind #5,
  Jade Robbery #81, Manhattan Lady #120, Final Record #97, Shinko Lovely #138, Irish Dance #140). Char table = proven.
- IMPORTANT: that project's `CLAUDE.md` "CARD FORMAT" is for **EPR-22336C = Rev C World Edition (BR reader)** =
  the US 3-track stat card I already cracked. It does NOT describe the JP `derbyo2k` (LR) card. So WE vs JP really
  are different card systems (matches the handoff premise). The JP/LR layout = THIS document's empirical RE.
- ROM stores sire names in EUC-JP; the CARD uses the separate 1-byte kana table I cracked. Different encodings.
- WE T2 has fields that FLUCTUATE in play (condition[44], rest[22], hearts[37], trust[36]). => the "volatile"
  JP T1 bytes 0x02-0x08 (which changed in bounded small ranges between live reads) may be REAL compact dynamic
  stat fields, not leak. 0x0b-0x17 (high/random) still look like leak. Needs a stat-screen capture to map.

## CAVEAT — dirty captures
- Tracks 2-3 (and bits of track 1: 0x0B-0x0E, 0x16-0x17) contain Windows heap pointers
  (e.g. `b070 884d a602` = ptr 0x000002a64d8870b0). Buffer was dirty (Flycast still running and/or
  LR reader leaves unwritten tracks uninitialized). Real stat/parent layout in tracks 2-3 unverified.
- NEED a clean capture (Flycast closed) with a KNOWN canonical name to (a) confirm byte order and
  full char table, (b) see true tracks 2-3.

## Captures on disk
- `_jp_re/captures/derbyo2k_sat{1..4}_unknownname.card` (sat1=トホメレ, sat4=コィウォヤ known; sat2/3 unknown)

## Next
1. CONFIRM identity-card hypothesis (decisive, low-effort): relaunch derbyo2k, let the 4 horses re-save,
   close again -> frozen2. Diff frozen1 vs frozen2: expect 0x28-0x42 (name/sire/dam) IDENTICAL and
   0x00-0x1f + tracks2-3 to CHANGE (new heap leak) => proves no persistent on-card stats.
2. (Optional) Race/develop one horse, re-save, diff -> see if ANY on-card byte tracks career/stats.
3. Then tooling: JP name/pedigree decode+encode into suite/creator/converter (char table is done).
