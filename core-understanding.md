# Derby Owners Club — Core Reverse-Engineering Knowledge Base

**Status:** Authoritative master synthesis of 13 verified subsystem decodes (Jun 2026).
**Scope:** Sega NAOMI arcade title *Derby Owners Club*, all four program ROMs:
World Edition Rev C (WE-C), World Edition EX Rev D (WE-D), DOC 2000 JP (o2k), DOC '99 JP (oc).
**Method:** Every offset/value below was extracted from real `.ic22` ROM bytes, real `.card`/`.raw`
captures, and real cabinet `.eeprom`/`.sram` images, then cross-checked against the per-version
horse databases, the 681-horse Breeder Studio JSON, and the live tool sources
(`DOC-Card-Creator.html`, `DOC-ROM-Studio.html`). Corrected values from verifier passes are
applied throughout; where a prior doc was wrong it is called out inline as **[CORRECTED]**.

Confidence convention used per claim: **1.0** byte-proven across all rows; **0.9–0.95** strongly
evidenced; **0.5–0.85** narrowed/partial; **<0.5** hypothesis. "verified N/N" = matched every horse.

---

## 0. The four ROMs (canonical fingerprint table)

| key | edition | folder | file | size | date@0x130 | serial@0x134 | reader | 0x8000 sig (first 8 bytes) |
|---|---|---|---|---|---|---|---|---|
| WE-C | World Edition Rev C | drbyocwc | epr-22336c.ic22 | 4,194,304 | 2001-10-30 | `BEF0` | BR | `dc 99 02 0c 9c c8 21 0c` |
| WE-D | World Edition EX Rev D | derbyocw | epr-22336d.ic22 | 4,194,304 | 2001-10-30 | `BEF0` | BR | `09 00 4a d2 0e e3 47 d0` |
| o2k | DOC 2000 JP | derbyo2k | epr-22284a.ic22 | 4,194,304 | 1999-10-01 | `BBX0` | LR | `16 2f 04 7f fc f5 fc f6` |
| oc | DOC '99 JP | derbyoc | epr-22099b.ic22 | 4,194,304 | 1999-10-01 | `BAX0` | LR | `18 8b ee 02 f1 53 28 38` |

- All four are **4 MB** (the AI Master Architecture doc's "2 MB" is wrong).
- DOC II (8-satellite, serial `BDY0`) exists but its ic22 was not in this corpus; references are noted where relevant.
- **Version fingerprinting:** 0x134 serial + 0x130 date + the 0x8000 16-byte run uniquely identify a build even on edited ROMs (the 0x8000 region is version-specific SH-4 code, never touched by table editors). [CORRECTED] oc's sig is `188bee02f1532838` (no stray space — a prior transcription showed `188bee02f15328 38`).
- [CORRECTED] Header word at **0x144 is `00 00 00 00`** for all 4 ROMs (a prior doc said `03 00 00 00`; the `03` region count lives at 0x15C).
- WE-C and WE-D share the same header date/serial (Rev D was **not** re-stamped); disambiguate them only by the 0x8000 sig or table offsets.

---

## 1. Program architecture (NAOMI / SH-4)

### 1.1 Cart header (0x000000–0x00041F)
Standard Sega NAOMI ROM-board header, identical structure across all 4; only titles/date/serial differ.

| off | width | field | notes |
|---|---|---|---|
| 0x000 | 16 | `NAOMI ` platform tag | identical all 4 |
| 0x010 | 32 | `SEGA ENTERPRISES,LTD.` | identical all 4 |
| 0x030 | — | **8 region title slots, 0x20 wide each** at 0x030/0x050/0x070/0x090/0x0B0/0x0D0/0x0F0/0x110 (0x100 total) | [CORRECTED] slots are 0x20-wide, not 0x30. Slot 0 = `" DERBY OWNERS CLUB WE ---------"` (WE) vs `" DERBY OWNERS CLUB ------------"` (JP). **Slot 0 is the card-reader trigger:** "WE" → Sanwa BR path; blank → LR path. |
| 0x130 | 2+1+1 | build date: year(LE u16), month, day | WE 2001-10-30, JP 1999-10-01 |
| 0x134 | 4 | SEGA game serial (ASCII) | BEF0 / BBX0 / BAX0 / BDY0 |
| 0x144 | 4 | format word `00 00 00 00` | [CORRECTED] all 4 |
| 0x15C | 4 | region/mode count `03 00 00 00` | identical all 4 |
| 0x160 | var | NAOMI ROM-board / M2 load+encryption descriptor | opaque board metadata; not needed for game RE (conf 0.6) |
| 0x260/0x280/0x2A0 | 0x20 ea | menu strings | `CREDIT TO NEW GAME START`, `CREDIT TO CARD GAME START` (DOC-specific card start), `CREDIT TO CONTINUE` |
| 0x420 | 13 | SH-4 boot/init micro-descriptor | byte-identical all 4 |

The game's effective region is selected by cabinet EEPROM/BIOS, not a single ROM region byte.

### 1.2 Code/data memory map (file offsets; treat 0x1000 as program base)

| region | range | content | conf |
|---|---|---|---|
| Header | 0x000000–0x000FFF | NAOMI ID + titles + menu strings | 1.0 |
| Main code (block 1) | 0x001000–~0x0C0000 | SH-4 executable, ~94% dense (prologue at 0x1000: `24 d0 25 d1 12 20 09 00`) | 1.0 |
| Text/data | ~0x0C0000–0x100000 | track/race/G1 names, game text (ASCII EN / EUC-JP JP) | 1.0 |
| Table data | 0x100000–~0x118000 | racing-stat + name + sire/dam tables, index/count tables, breeding comments | 1.0 |
| Code/data block 2 | ~0x130000–0x160000 | extended logic + pointer/index tables | 1.0 |
| Food/items | ~0x166A7C (WE-C) / 0x171F34 (o2k) / 0x15C9EC (oc) | food DB | 1.0 |
| Build credits | ~0x17FF80 | ASCII NEC Home Multimedia / Client Server credits — marks end of data; DOC built on NEC middleware | 1.0 |
| Gap | ~0x180020–0x2FFFFF | mostly zero (WE-C is 0xFF-filled past 0x200000) | 1.0 |
| Second program image | 0x300000+ | near-duplicate of the 0x1000 SH-4 program (same prologue, different PC-relative displacements; first 16 bytes identical WE-C↔o2k). Likely a 2nd bank / boot mirror. | 0.8 |
| Tail | 0x340000–0x3FFFFF | sparse extra tables (o2k has more here) | 0.6 |

### 1.3 Pointer / index architecture (addresses are o2k-specific)
1. **Function-pointer table @ 0x15729C** — array of LE32 ROM addresses, the best disassembly seed list. [CORRECTED] confidence ~0.6, not 1.0: there are **295 contiguous in-range values** (not 200), targets verified include ptr[0]→0x3E7C0, ptr[1]→0x5AC60, ptr[5]→0xCAE0, but **not all targets begin with SH-4 prologues** — ptr[1]'s target is `0b 00 09 00` (rts;nop), and 162 targets begin with word 0x0000 (data/padding). It is a useful seed list, not a clean pure function table.
2. **Index/lookup table @ 0x15B234** — 16-bit LE entries (87,14,15…) = sire breeding-comment indices; a horse's position indexes this to get a comment-string index (positional linkage). conf 1.0.
3. **RAM-pointer table @ 0x15B1E0** — LE32 pointers into live RAM. [CORRECTED] spans **0x0166xxxx–0x0174xxxx** (entries 12–20 are in 0x017xxxxx), not strictly 0x016xxxxx; entry[0]=0x016659E0 correct. Runtime data binding; watch these in an emulator to find live race state.

### 1.4 Recommended disassembly path
Ghidra with the SuperH/SH-4 LE module; import raw, seed functions from 0x15729C, label the data tables (§ below), follow XREFs from `record+9..+14`/`+29..+31`/`+5`/`+8`/`+21`/`+23` to localize the race solver, and from sire `name+36`/the 0x15B234 index to localize breeding. Confirm live in Flycast/DEMUL RAM-watch keyed off 0x15B1E0. The Beer-Experiment ROM diff (§7) is a zero-disasm differential anchor for the item/condition code region.

---

## 2. The two horse rosters and two stat scales (read this before any stat work)

DOC has **two completely separate horse tables with two different external-stat scales**. Conflating them is the #1 historical error.

| | table | records | externals scale | internals scale | who |
|---|---|---|---|---|---|
| A | **Racing stat table** | 244 | **0–63** (6-bit) | 0–63 | **CPU opponents** (proven static, see §3) |
| B | **Sire/Dam breeding table** | 168 (WE-C)/178 (WE-D) split; 167/177 indexed in ROM | **1–16** band | ~10–60 | breeding stock |

- Scale relationship: racing externals ≈ **4× breeding band** (63/16≈3.94). A horse retiring to stud bins its 0–63 stats into 1–16 bands. The aptitude symbols ✕△○◎ are exactly the four quartiles of the 1–16 band (`symFor`: ≥13→◎, ≥9→○, ≥5→△, ≥1→✕, 0→·).
- Internals share the 0–63 scale on both tables (no rescale).
- Player-horse live stats are on the **card** (full 0–63 externals) — neither table is the player's horse.

---

## 3. Racing stat table (244 CPU-opponent records) — FULL BYTE DECODE

**Framing (proven):** This table is the roster of **CPU opponents**, not the player's horse. Proof: diffing the beer-edited ROM vs base changes *only* the food table (0x167xxx) and *zero* bytes here. CPU horses don't breed/age, so there is no sex/career/condition/age — but personality, coat, style, grade ARE stored per CPU horse (see below).

### 3.1 Locations & geometry

| ver | record start | stride | count | name table | name stride/enc |
|---|---|---|---|---|---|
| WE-C | 0x108E03 | 32 | 244 | 0x10AD50 | 18 / ASCII |
| WE-D | 0x10A14B | 32 | 244 | 0x10C098 | 18 / ASCII |
| o2k | 0x10AD1B | 32 | 244 | 0x10CC68 | 18 / EUC-JP |
| oc | 0x0F6902 | **28** | 244 | 0x0F8480 | 18 / EUC-JP |

**Cross-version identity [CORRECTED — this overturns a former headline]:**
- WE-C == WE-D: byte-identical for all 244 records. **TRUE.**
- WE-C == o2k: **FALSE.** They share the 32-byte *layout* but **22 of 244 records differ** (first divergence at record index 12). The 2000 rebalance touched real stat bytes: dirt(+5), externals(+9..14), flag-B(+16), coat(+22), the hidden 16-bit field(+23/+24), internals(+29..31). Stats are **~91% shared** with o2k, not 100%. A "unified editor writes to all three" / "patch transposer applies a WE-C edit to o2k unchanged" is therefore **UNSAFE for ~22 horses**.
- oc ('99): substantially rebalanced/partially-overlapping roster on the 28-byte layout. [CORRECTED] **92 of 244 core stat blocks are byte-identical to WE-C** (e.g. #1 Gold Queen) — so "genuinely different roster" overstates it; it is partially overlapping.

### 3.2 32-byte record map (WE-C / WE-D / o2k), offsets record-start-relative

| off | hex | field | range | meaning / verification | conf |
|---|---|---|---|---|---|
| +0 | 00 | pad | 0 | const | 1.0 |
| +1 | 01 | **HIDDEN-A / grade-aux** | 0–2 (175/62/7) | real per-horse field; not 1:1 with grade/style/coat/personality; fairly static (never changed in the 22 o2k diffs). Candidate: growth-type / season / availability flag. | 0.5 |
| +2 | 02 | **id (copy 1)** | 1–244 | = 1-based roster index | 1.0 |
| +3 | 03 | **id (copy 2)** | 1–244 | == +2 (two single-byte copies, not a 16-bit hi/lo) | 1.0 |
| +4 | 04 | pad | 0 | const | 1.0 |
| +5 | 05 | **DIRT aptitude** | 0–255 | surface affinity; matches DB Dirt 244/244 | 1.0 |
| +6,+7 | 06,07 | pad | 0 | const | 1.0 |
| +8 | 08 | **GRADE** | 0–3 | 0=Ungraded 1=G3 2=G2 3=G1; 244/244 vs DB | 1.0 |
| +9 | 09 | ext **Start** | ~11–63 | per-phase ability | 1.0 |
| +10 | 0A | ext **Corner** | ~14–59 | | 1.0 |
| +11 | 0B | ext **OOB** (out-of-box) | ~4–63 | | 1.0 |
| +12 | 0C | ext **Competing** | ~8–63 | | 1.0 |
| +13 | 0D | ext **Tenacious** | ~3–62 | | 1.0 |
| +14 | 0E | ext **Spurt** | ~4–63 | | 1.0 |
| +15 | 0F | pad | 0 | const | 1.0 |
| +16 | 10 | **HIDDEN-B / flag-B** | 0–2 (200/37/7) | [CORRECTED — NOT constant] real per-horse field, independent of HIDDEN-A; o2k rebalanced it in 9 of the 22 diffs (e.g. horse #13 differs ONLY here, 0→1). Candidate: hidden growth/availability. | 0.55 |
| +17..+20 | 11–14 | pad | 0 | const | 1.0 |
| +21 | 15 | **RUNNING STYLE** | 0,1,2,3,7 | 0=Front-runner 1=Start dash 2=Last spurt 3=Stretch-runner 7=Almighty; **244/244 deterministic** (STORED, contradicting "leg type is derived" — true for CPU horses). | 1.0 |
| +22 | 16 | **COAT color** | enum | 0=Default 192=Chestnut 193=Black 199=Brown 202=Bay 204=Dark Gray 207=Light Gray 222=Special; 244/244 vs DB | 1.0 |
| +23 | 17 | **PERSONALITY (banded) / HIDDEN-X low** | 0–255 | banded 0-47 Rough/48-63 Imposing/64-111 Calm/112-127 Firm/128-175 Sensitive/176-191 Moody/192-239 Gentle/240-255 Proud (matches DB by band 244/244); also the low byte of a 16-bit value with +24 | 0.9 |
| +24 | 18 | **HIDDEN-X high / personality-aux** | clustered | high-byte clusters 0xA0(186),0x30(30),0xC0(9),0xF0(7),0x00(12); o2k rebalanced it → real attribute. Candidate: 16-bit temperament word `[+23][+24]` and/or distance/surface aptitude composite. | 0.5 |
| +25 | 19 | **id echo** | 0–243 | [CORRECTED] a 1-byte id echo = (record index) low byte; for the 244th record it holds 0 (off-by-one wrap), NOT a literal "mod 256" rule. Third id copy. | 1.0 |
| +26..+28 | 1A–1C | pad | 0 | const | 1.0 |
| +29 | 1D | int **Stamina** | 0–60 | energy pool; matches DB | 1.0 |
| +30 | 1E | int **Speed** | 0–63 | top-speed ceiling | 1.0 |
| +31 | 1F | int **Sharp** | 0–60 | acceleration/responsiveness | 1.0 |

**Worked example — WE-C horse #1 "Gold Queen" (32-byte raw):** [CORRECTED — the canonical record is 32 bytes; a prior dump showed only 31]
```
00 01 01 01 00 a8 00 00 03 2c 23 13 20 28 2e 00 00 00 00 00 00 02 cf 0e a0 01 00 00 00 17 25 30
```
→ id=1, dirt=168(0xa8), grade=3(G1), ext=[44,35,19,32,40,46], style=2(Last spurt), coat=0xCF(Light Gray), personality/HIDDEN-X=0x0E low/0xA0 high, idEcho=1, int=[23,37,48]. Matches DB exactly.

### 3.3 28-byte record map (oc / DOC '99) — same semantics, tighter packing
Drops one id copy and several pad bytes. Offsets record-start-relative:
`+0 HIDDEN-A · +1/+2 id (dup) · +3 pad · +4 DIRT · +5,+6 pad · +7 GRADE · +8 pad · +9..+14 externals · +15,+16 pad · +17 HIDDEN-B · +18 RUNNING STYLE (0–3, no Almighty in '99) · +19 COAT · +20 PERSONALITY/HIDDEN-X low · +21 HIDDEN-X high · +22 id echo · +23 pad · +24 Stamina · +25 Speed · +26 Sharp · +27 pad`.
Note: appearance bytes are adjacent in both layouts — 32B: style+21/coat+22/pers+23; 28B: style+18/coat+19/pers+20.

### 3.4 Remaining unknowns
HIDDEN-A (+1), HIDDEN-B (+16), and 16-bit HIDDEN-X (+23/+24 — partly the personality band). Best route to labels: an in-game opponent stat / sire-encyclopedia screen. The o2k rebalance evidence proves HIDDEN-B and HIDDEN-X are live gameplay attributes, not padding.

---

## 4. Derived attributes (running style, personality, aptitude symbols, growth type)

### 4.1 Running style (5 styles): `Front-runner, Start dash, Last spurt, Stretch-runner, Almighty`
- **Player card:** STORED at Track1 byte 7 (a1[7]), 0–255, mapped `floor(byte7/51)` → styles 0–4. This is the game-authoritative seed. [Subtlety] WE actually derives the *displayed* leg type at runtime from the externals (`legTypeFromExt`: rank of Start among {Start,OOB,Comp,Tenac,Spurt}, Corner excluded; all-equal→Almighty). Edited/maxed cards push byte 7 to 255 (÷51=5, out of range) → treat byte 7 as a **stored seed to preserve on edit, not trusted for display**.
- The Card-Creator's "Start-rank among externals" rule is a **display heuristic only** — the editor never writes a1[7], so on editor-made cards byte 7 is leftover and the two models diverge (verified: Caitin byte7=1→ROM Front-runner vs tool Start dash; Gulf byte7=255→ROM Almighty vs tool Last spurt). Likely original intent: the externals-rank rule is how the game assigns the *initial* style at birth, then bakes it into byte 7 (which can drift via training — proven by the "Leg-Type Change Messages" block @0x12755C).
- **CPU table:** STORED at +21 (244/244, §3). The 5 "Almighty" CPU horses are exactly those with all six externals = 31 (sentinel all-rounder).

### 4.2 Personality (8 ROM bands / 5 in-game "Check" labels)
- **Stored only on the player card** at Track1 byte 6 (a1[6]), 0–255; and **banded** on the CPU table at +23 (244/244). NOT on the sire/dam table directly (but see §5.4 composite).
- 8-band ROM truth (authoritative): 0-47 Rough · 48-63 Imposing · 64-111 Calm · 112-127 Firm · 128-175 Sensitive · 176-191 Moody · 192-239 Gentle · 240-255 Proud. [CORRECTION-NOTE] the exact cut points are ROM-derived and cannot be re-confirmed from card bytes alone.
- In-game "Check" English labels: Imposing/Honest/Rough/Coward/Sloppy (+ Too soft/Strict edges) @ WE-C 0x0E84A4. JP romaji labels (Doudou/Sunao/Arai/Okubyou/Zubora) @0x107DFC and a duplicate cluster [CORRECTED] **@0x0EB614** (the anchor 0x0EB61C was ~1 record late).
- The Card-Creator collapses 0–255 to **5 lossy anchors {R:0,I:48,C:64,H:80,S:208}** — a faithful rebuild must store the raw byte.
- Personality drives the interaction-effect multiplier table (38 IEEE-754 floats @0x0E7D00; Hug/Praise/Scold/Flatter scaled ×2.0..−2.0).

### 4.3 Aptitude symbols ✕△○◎ — derived, not stored
Pure quartiles of the 1–16 breeding external via `symFor` (≥13◎/≥9○/≥5△/≥1✕/0·); applies to racing externals after ÷4 binning. No other trigger.

### 4.4 Growth / internal "type"
`Speed type / Stamina type / Sharp type` strings @0x0EE270 (WE-C) are the **internal-stat-bias labels** (this block is the retirement/registration UI, continuing into `Stud reg./Dam reg./Sire/Dam`). No distinct stored "growth-curve" byte found in either table; a horse's dominant internal = its "type" (conf med-high, absence inferred from full-column scans).

---

## 5. Breeding / mater inheritance system

### 5.1 The pool
One logical breeding-stock pool. EN World Edition splits it as 84 sires + 84 dams; the JP versions merge it into one "mater" array. Stats line up record-for-record across versions (Trot Thunder = Heart Lake, etc.).

| ver | sire base | dam base | mater base (JP merged) | stride | name |
|---|---|---|---|---|---|
| WE-C | 0x10BF1C | 0x10D2CC | — | 60 | 24B ASCII, name-first |
| WE-D | 0x10D264 | 0x10E614 | — | 60 | 24B ASCII, name-first |
| o2k | — | — | 0x11106C | 60 | index-first (+0), 20B EUC-JP at +4 |
| oc | — | — | 0x0F9680 | **56** | index-first (+0), 16B EUC-JP at +4 |

**Record counts [CORRECTED]:** the ROM `+56` index field runs **1..167 (WE-C / o2k)** and **1..177 (Rev D)**, then turns to garbage (Rev C breaks at idx 167 → "Pete O Pete" junk). The 168/178 + "84 sires + 84 dams"/"89+89" split (sex M id≤84, F id≥85) comes from `DOC_Breeder_Studio_data.json`, **not** from the ROM index field.

**Layout [CORRECTED]:** the EN pool is **ONE contiguous 60-byte block**, not two physically separate arrays. The claimed "dam base" 0x10D2CC = exactly `sire_base + 84*60` = record #85; the index increments continuously 1..167 with **no restart at 85**. (The 0x10D2CC label is still a useful pointer to record #85, but it is not a second array.)

### 5.2 60-byte EN sire/dam record (anchored at name `O = base + 60*k`)

| rel off | width | field | scale / notes | conf |
|---|---|---|---|---|
| +0x00 | 24 | **Name** | ASCII, 0-padded | 1.0 |
| +0x18 (+24) | u32 | **Stamina (st)** | low byte ~10–60 | 1.0 |
| +0x1C (+28) | u32 | **Speed (sp)** | ~10–65 | 1.0 |
| +0x20 (+32) | u32 | **Sharp (sh)** | ~10–60 | 1.0 |
| +0x24 (+36) | u32 (low byte) | **ac** = course/dirt aptitude | 0–255 | byte 1.0 / meaning 0.65 |
| +0x28 (+40) | u32 | reserved 0 | | 0.95 |
| +0x2C (+44) | 4 | **composite (Packed_u32_5)** | packed bitfield, partial decode | byte 1.0 |
| +0x30 (+48) | 6 | **Externals** start,corner,oob,comp,tenac,spurt | each 0–15 (1–16 band) | 1.0 |
| +0x36 (+54) | 2 | pad | | 0.95 |
| +0x38 (+56) | u16 | **record index** (1-based) | 1..167/177 | 1.0 |
| +0x3A (+58) | 2 | pad | | 0.9 |

Worked example — WE-C sire #1 "Maple Syrup": st=39, sp=19, sh=34, ac=240, composite=[0x00,0xE0,0x11,0xF0], externals=[15,3,6,10,4,12], index=1 (matches Breeder JSON exactly).

JP records put **index first (+0)** then name (+4) then the same stat block; o2k stride 60, oc stride 56 (16B name). Composite is **stable per horse across JP versions** (e.g. White Norther [1,234,0,48] in both o2k and oc).

### 5.3 The "ac" byte (name+36) — meaning
Two analyses: (A) **course/dirt aptitude** 0–255 — favored, with clean correlation (dirt-only-comment breeders avg ac≈185, turf-only ≈89, both ≈156); (B) the Source-of-Truth doc labels it the personality byte. Resolution: treat **ac = dirt/course aptitude** (used directly by every shipping tool); the same byte may also seed personality banding. Note: `name+36` (single ac byte stored as u32) and `name+44` (the real 4-byte composite) are **two distinct fields 8 bytes apart** — earlier "4-byte composite ac at name+36" conflated them.

### 5.4 The composite (name+44, Packed_u32_5) — partial
4 bytes, byte-stable per horse: `b0`∈{0,1,2,3} (2-bit category) · `b1` clustered 0xC0–0xEF · `b2` high-entropy (Source-of-Truth doc's PersonalityByte) · `b3` multiples of 0x10/0x30/0xA0/0xC0/0xF0 (nibble-packed flags). Best model: packed inheritance/affinity + appearance + personality composite consumed by the breeding routine. b0/b1/b3 meanings are TBD — **this is the gate to the exact inheritance rule**. [CORRECTED] the composite is **NOT byte-identical EN↔JP** (only JP↔JP is stable): e.g. White Norther JP [1,234,0,48] vs its EN stat-twin Judge Angelucci [1,204,171,52]. There is also a separate still-unknown 3-byte block at name+45..47 (first nibble clusters 0xC–0xE).

### 5.5 Foal inheritance (community model — NOT yet ROM-confirmed)
The only explicit algorithm available (`breedFoal()` JS), consistent with observed foals but a reconstruction:
```
st/sp/sh = floor(parent average)
ac       = floor(parent avg + (rand-0.5)*36)        // ±18 noise
each external = clamp(floor(parent avg) + floor((rand-0.5)*4), 1, 16)   // ±2
bloodline bonuses: sire.st≥45&dam.st≥40 → st+2; sire.sp≥45&dam.sp≥40 → sp+2; sire.ac>220&dam.ac>220 → ac+20
clamps: st 10-60, sp 10-65, sh 10-60, ac 0-255
sex = 50/50; style = deriveRunningStyle(externals)
```
The chosen "dominant parent" does **not** weight the math (display only), and the model ignores the name+44 composite — so the real ROM rule is almost certainly richer (affinity/line bonuses). The actual SH-4 breeding routine has not been disassembled.

### 5.6 Cross-version name mapping [CORRECTED]
Reconcile EN↔JP **by NAME, not index** — the JSON is ~alphabetical while the ROM is game order (e.g. ROM rec84="Pentire" vs JSON revC id84="Zephyr Hills"; Rev D ROM rec1="Maple Syrup" vs JSON revD id1="Banana Boy"). By name: 161/167 match (6 are spelling/space variants); 156/161 of those match st/sp/sh/ac exactly.

---

## 6. Tracks, courses, and the G1 calendar

**These are null-terminated DISPLAY string tables, not binary race records.** Surface (TURF/DIRT) and distance are literal text inside each string; there is no per-entry attribute byte. The real per-race binary (grade/surface/distance/prize/month, G1→course binding) lives in a separate **undecoded** schedule table (candidate: o2k 0x0CAD7B+, `xx04` markers + `80 3f ff ff ff ff 05 24` rows).

### 6.1 Offsets per version

| section | oc '99 | o2k 2000 | WE-C | WE-D |
|---|---|---|---|---|
| Course list | 0x0BD875 | 0x0CA335 | 0x0C6940 | 0x0C6260 |
| G1 names | 0x0BDAD5 | 0x0CA62D | 0x0C6CA0 | 0x0C65C0 |
| Aligned display names | 0x0BDC2D | 0x0CA7AB | 0x0C6DF0 | 0x0C6710 |
| Special races | 0x0BDEE7 | 0x0CAB0B | 0x0C70C8 | 0x0C69E8 |
| Handicap races | — (none) | 0x0CAC5B | 0x0C7248 | 0x0C6B68 |

WE-D = WE-C content shifted uniformly **−0x6E0**.

### 6.2 Counts per version [CORRECTED where noted]

| metric | oc '99 | o2k 2000 | WE-C | WE-D |
|---|---|---|---|---|
| Courses | **29** [CORRECTED, was 30; venue sum 東京6+阪神6+中山6+京都8+セガ3+中京0=29] | 36 | 36 | 36 |
| Venues | 5 (no Chukyo) | 6 | 6 | 6 |
| SEGA courses | 3 | 5 | 5 | 5 |
| Chukyo / SOUTHERN PARK | no | yes | yes | yes |
| G1 entries | **19** [CORRECTED, was 20; ends ダービーオーナーズカップ] | 21 | **19 string runs (18 real + NO NAME)** [CORRECTED, was "20 (19 races + NO NAME)"] | same as WE-C |
| Special races | 10 | 12 | 12 | 12 |
| Handicap races | none | 12 | 12 | 12 |
| Encoding | EUC-JP | EUC-JP | ASCII | ASCII |
| Aligned-display venues | all 5 | all 6 | **4 (no NORTHERN/SOUTHERN PARK)** | 4 |

### 6.3 Lineage
DOC '99 (29 courses / 5 venues / 19 G1 / no handicap) → DOC 2000 (adds Chukyo venue, +2 SEGA courses, a 21st G1 [高松宮記念 + ジャパンカップダート], + handicap table) → World Edition (English venue names, same 36 courses, handicap table, but aligned-display table drops NORTHERN/SOUTHERN PARK — a WE-specific omission).

### 6.4 Venue localization (EN ↔ JP ↔ real JRA)
EASTERN CITY=東京(Tokyo) · WESTERN HILL=阪神(Hanshin) · NORTHERN PARK=中山(Nakayama) · CENTRAL CITY=京都(Kyoto) · SEGA=セガ(fictional) · SOUTHERN PARK=中京(Chukyo). Surface tokens 芝=TURF, ダート=DIRT; distances are fullwidth digits + Ｍ.

### 6.5 Notes
- WE G1 list (in order): WINTER STAKES, DOC 1000/2000 GUINEAS, SPRING CLASSIC, AMERICAN DERBY, HONG KONG OAKS/DERBY, AMERICAN OAKS, SUMMER GRAND PRIX, SUPER DIRT GRAND PRIX, **NO NAME** (placeholder), STAYERS STAKES, QUEEN ELIZABETH CUP, MILE CHAMPIONSHIP, JAPAN CUP, SPRINTERS STAKES, DERBY OWNERS CUP, JAPAN CUP DIRT, SPRINTERS TROPHY.
- `bb f7` inside "QUEEN ELIZABETH [+] CUP" is an inline wide-glyph/center control code (one race name).
- [CORRECTED] several WE-C course-list offsets in the source doc are 1–3 bytes high (leading inline control bytes) — content/order/count are correct.
- EN↔JP exact G1 mapping is positional-guess only (WE 18 real + NO NAME vs JP 21).

---

## 7. Items / feeding / beer

Single packed array of **44-byte food records**, terminated by an all-zero (idx=0) record. 44-byte stride in **all four** versions (independent of the racing-table stride).

### 7.1 Locations

| ver | table start | foods | terminator |
|---|---|---|---|
| WE-C | 0x166A7C | 45 | rec 45 @ 0x167238 |
| WE-D | 0x16980C (= WE-C +0x2D90) | 45 | byte-identical food data, relocated |
| o2k | 0x171F34 | 45 | |
| oc | 0x15C9EC | **41** | no banana, no beer |

### 7.2 44-byte record layout

| off | width | field | notes |
|---|---|---|---|
| +0 | 24 | Name | ASCII (EN) / EUC-JP (JP), 0-padded |
| +24 | u32 LE | Graphic pointer | RAM addr, usually 0x0D8xxxxx |
| +28..+34 | 7 | **Stat-effect deltas** (cols 0–6) | the applied boosts |
| +35 | 1 | Effect-class flag | 0x01 normal feed; 0x00 growth-only (mushrooms + bananas) |
| +36 | 1 | Rarity/size flag | 0x00 small/common, 0x01 large/special |
| +37 | 1 | reserved 0 | |
| +38 | 2 | reserved 0 | |
| +40 | u32 LE | **Food index/ID** | 1..39 real; trailing dupes reuse 39/1; terminator 0. **NOT unique** — writing an out-of-range index (44/45) crashes at boot (init builds a ~39-entry lookup array). |

### 7.3 Effect columns
Cols 0/1/2 = **Speed / Stamina / Sharp** (anchored by the on-screen "Speed/Stamina/Sharp/Friendship" label block @0x12874C), conf 0.85. Col 3 most plausibly = **Friendship**; cols 4–6 are hidden/internal growth stats with **no confirmed UI name** (conf ~0.55). [CORRECTED] the "Spirit" @0x110274 and "Power" @0x10BDFA strings are NOT feed-stat labels (Power is the horse name "Power Drift"; Spirit is in an award/menu list) — do not bind columns to them. Per-column value range 0..7; "large" variants roughly double the base. KOREAN GINSENG = +2 to all six of cols 0–5; LARGE = +4 all.

### 7.4 Per-version differences
- **oc '99 has only 41 foods, NO banana, NO beer** — beer & banana were ADDED in DOC 2000. (Corrects any note implying '99 had beer.)
- o2k ships both beers (生中 DRAFT, 黒生中 BLACK DRAFT) with **all-zero effect** (disabled placeholder).
- WE-C/WE-D carry the identical 45-food table.
- Genuine per-version tuning: CUBE SUGAR, GREEN SALAD, LARGE ? MUSHROOM effect values differ between '99 and 2000/WE.

### 7.5 The beer experiment (differential anchor)
`beer_effects_test.ic22` vs base = **exactly 12 bytes** in two runs: DRAFT cols0–5 → +2, BLACK cols0–5 → +4 (a verbatim KOREAN-GINSENG clone). Col6 (+34) and the effect-class flag (+35) were left untouched; index stayed at 1 (collides with CARROT). No authentic beer effect exists in any ROM — every shipped beer has a null effect.

---

## 8. Cabinet save (EEPROM + SRAM / BBSRAM)

Two separate stores. **Career/leaderboard data is entirely in the 32 KB BBSRAM; the EEPROM is identity + dips only.** This is a near-factory (NPC demo) save; only ~10.6 KB of 32 KB is used.

### 8.1 EEPROM (128 bytes) — JVS machine identity + dip settings
- ASCII `BEF0` tag @0x02 (same family as the card's SEGABEF0); `DERBY OWNERS CLUB AM3` game-ID string @0x30; game CRC/magic `60 04 10 a3` @0x2C; dip block @0x45.
- [CORRECTED] mirroring is **two separately-mirrored JVS sub-blocks** (system 0x00-0x11→0x12-0x23; game 0x2c-0x4f→0x54-0x77), not one 0x00-0x4f block doubled at 0x50. No career data here.

### 8.2 SRAM top-level layout
```
0x0000  bookkeeping header A (coin/play counters + dip flags)
0x0100  second bookkeeping header (related/staggered — NOT a verbatim copy of A) [CORRECTED]
0x01f8  SAVE REGION 1 (primary): 16B checksum hdr (doubled at 0x208) +
        rank-0 marker + money leaderboard copy1 + copy2 + 57-entry track-record table
0x15c8  SAVE REGION 2 (backup), region delta +0x13c4 [CORRECTED, was +0x13d0]
0x2998  end of used data
```

### 8.3 Bookkeeping header (0x0000)
LE32 counters at +0x00 (play/coin A), +0x04 (≈A), **+0x08 (total runtime / program-pace section counter — the highest-value target for the parked battery-resume hack)**; setting-flag bytes at 0x10/0x20/0x30/0x40/0x4c. [CORRECTED] Header A is **not** mirrored verbatim at 0x100 — 14 byte positions differ in the first 0x50; 0x100 is a distinct second header beginning with the +0x08 counter.

### 8.4 Money leaderboard ("richest horses" Top-50)
50 records × 32 bytes, sorted descending by LE32 prize money (417,935,500 → 4,628,541). **[CORRECTED] Region-2 copies start at 0x15f4 (copy1) / 0x1c34 (copy2), NOT 0x1634** — an editor writing region 2 at 0x1634 would corrupt the backup. Region 1 copies at 0x230 / 0x870.

| off | width | field |
|---|---|---|
| +0x00 | u8 | flag0 (0–7) — display/sex/medal bitfield, UNCONFIRMED |
| +0x01 | u8 | flag1 (0xc0–0xde) — packed grade/coat/gen marker, UNCONFIRMED |
| +0x02 | LE32 | **money** (sort key) |
| +0x06 | LE16 | meta_lo (0 in copy1; set in copy2) |
| +0x08 | u8 | per-horse sub-counter (0/1/2) |
| +0x09..+0x0b | — | copy2 writes `80 16 00` → **0x001680 = 5760** [CORRECTED, doc reversed bytes as 0x168000] likely date/season stamp |
| +0x0c | 20 | name (ASCII/EUC-JP) |

Names join to the per-version horse DB (e.g. "City Commandant" = #222 G1). This is the factory attract leaderboard.

### 8.5 Track-record table
57 records × 28 bytes @0x0f7c (region 1) / 0x2340 (region 2). +0x00 holder name (20B, all "Hitmaker is Sega" #123 in factory save), +0x14 **LE16** time in **1/40-s units** (cs = raw×2.5; 3384 = 8460 cs = 1.24.60 — corrected 2026-06-08, was mis-read as LE32 cs; see tools/online/TRACK_RECORD_TIME_PRECISION.md), +0x16 separate 2B field, +0x18 date/reserved.

### 8.6 Master vs satellite
Master (cwm) and satellite (cw) SRAMs are **byte-identical except per-board bookkeeping/checksums/trailer** — leaderboards/track records are **shared cabinet-wide data** replicated over the link. Region-1 checksum is the 16-bit word @0x1f8/0x208 (cw 0x3536 / cwm 0x259a). [CORRECTED] region delta is **+0x13c4** (= the length word at 0x1fc = 5060), not +0x13d0.

### 8.7 Tie to cards
The leaderboard stores only name + money + 2 flag bytes (no card ID / stat block). A career dashboard reads names+money+times from SRAM and joins to the ROM 244-record DB by name. The EEPROM `BEF0` tag ties cabinet and card to the same title.

---

## 9. US / World Edition card — full 207-byte decode

**Container:** 207 bytes = 3 tracks × 69 (0x45). Track1 0x00–0x44, track2 0x45–0x89, track3 0x8A–0xCE. **Logical bytes are stored reversed per track:** logical `aT[k]` (1-based, **0-based track index t∈{0,1,2}**) lives at file offset **`t*69 + (69-k)`**. The 207-byte `.card` is the *decoded* payload; the `.raw`/physical layer carries the XOR `multiCode` + 2-byte checksum. **There is no whole-card checksum in the decoded payload** — any byte is freely editable; re-encoding regenerates valid checksums. The 4-byte horse UID is the only on-card redundancy.

**Detection:** ASCII `SEGABEF0` at file **0x8A–0x91** ⇒ US/WE. The SEGAxxxx marker's last 4 chars = the ROM serial (WE BEF0 / DOC2000 BBX0 / DOC'99 BAX0 / DOC II BDY0). WE-C vs WE-D cards are NOT distinguishable from bytes (same layout/marker).

### 9.1 Track 1 — identity & genetics (a1, file 0x00–0x44)

| logical | file off | field | notes |
|---|---|---|---|
| a1[2..5] | 0x40–0x43 | **UID** (4-byte horse id) | triplicated identically across all 3 tracks; per-horse key |
| a1[6] | 0x3F | **Personality** 0–255 → 8 bands (§4.2) | tool collapses to 5 lossy anchors |
| a1[7] | 0x3E | **Running-style seed** 0–255, ÷51 | stored seed; display derived from externals (§4.1) |
| a1[8] | 0x3D | **Coat base** (63 = special trigger) | |
| a1[9] | 0x3C | **Coat modifier** (special sub-id when a1[8]=63) | |
| a1[11..29] | 0x2C–0x3A | **Dam name** (18 ASCII, reversed) | |
| a1[31..49] | 0x14–0x2C | **Sire name** (18 ASCII) | |
| a1[51..69] | 0x00–0x12 | **Horse name** (18 ASCII) | |

### 9.2 Track 2 — career & status (a2, file 0x45–0x89). General formula: `off = 69 + (69-k)`.

| logical | file off | field |
|---|---|---|
| a2[2..5] | 0x85–0x88 | UID (dup) |
| a2[13] | 0x7D | Silk color 2 (palette 0–14) |
| a2[14] | 0x7C | Silk color 1 (primary, palette 0–14) |
| a2[15] | 0x7B | Silk pattern (0–7) |
| a2[16] | 0x7A | **Sex** (0=Male 1=Female 2=Gelding) |
| a2[18],a2[19] | 0x78,0x77 | Last race result + track index (partial) |
| a2[20],a2[21] | 0x76,0x75 | Current race result + track index (partial) |
| a2[22] | 0x74 | Rest/fatigue timer (partial) |
| a2[23] | **0x73** | Retire internal SHARP (=45 on Scarecrow) [CORRECTED — 0x73, not 0x4B] |
| a2[24],a2[25] | 0x72,0x71 | Retire internal SPEED, STAMINA |
| a2[26] | **0x70** | **Hood** (0–63) [CORRECTED — 0x70, not 0x73] |
| a2[27] | 0x6F | owner/stable assoc (TBD; =44 on one stable's 3 cards, 0 elsewhere) |
| a2[28..33] | 0x69–0x6E | Retirement externals (Spurt,Tenac,Comp,OOB,Corner,Start; value-1, 1–16 bands) |
| a2[34] | 0x68 | Wins duplicate (= a2[49]) |
| a2[35] | 0x67 | Total races (0–64) |
| a2[36] | 0x66 | Trust (partial) |
| a2[37] | 0x65 | Hearts (display = (val+1)/4) |
| a2[38..43] | 0x5F–0x64 | Current externals (Spurt,Tenac,Comp,OOB,Corner,Start; value-1, 1–64) |
| a2[44] | 0x5E | Condition/fitness (partial) |
| a2[45] | 0x5D | Experience (partial) |
| a2[46..49] | 0x5A–0x5C,0x59 | Out (4th+), Show (3rd), Place (2nd), Won |
| a2[51..53] | 0x55–0x57 | Earnings: dollars = (a2[51]·65536 + a2[52]·256 + a2[53])·1000 |
| a2[55..57] | 0x51–0x53 | **G1 titles bitfield** (18 races across 3 bytes) |
| a2[61]/a2[65]/a2[69] | 0x4D/0x49/0x45 | Internal SHARP / SPEED / STAMINA (current, 0–60) |

External order (memorize): both current and retirement externals are stored **Spurt, Tenacious, Competing, OOB, Corner, Start** as logical index increases (Start is the highest index of its block). Display = card+1.

### 9.3 Track 3 — breeding/visual/markers (a3, file 0x8A–0xCE)
a3[2..5]=0xCA–0xCD UID(dup); a3[50]/a3[51]=0x9D/0x9C format markers `10`/`30`; a3[53]=0x9A breed count (offspring=val/2); a3[57]=0x96 retired flag; a3[61]=0x92 **Dirt ability** (0–255); a3[62..69]=0x8A–0x91 `SEGABEF0` marker. Clean WE cards have genuinely all-zero unused regions in tracks 2/3.

### 9.4 Low-confidence / next targets
a2[27] (owner/stable id?), a2[18–22] (race-history encoding — needs a developed-horse card), a2[36]/a2[44]/a2[45] (trust/condition/experience labels track each other on fresh cards), whether a1[7] is ever read post-creation.

---

## 10. JP card (DOC 2000 / DOC '99) — identity/pedigree only

The JP card uses the Sanwa CRP1231LR (LR) reader path. **Decisive finding: JP horse stats/sex/leg-type are NOT on the card and NOT in the cabinet nvram** — the JP card is an identity + pedigree card; the cabinet computes/holds live stats in volatile RAM keyed by the card's lead ID. Three independent proofs: (1) same-horse re-save diffs persist only 0x25–0x42; (2) no SEGA stat marker on any JP capture; (3) zero kana-name hits across all derbyo2k nvmem files.

### 10.1 Track 1 byte map (the only meaningful track)

| offset | status | meaning |
|---|---|---|
| 0x01 | const 0x70 | header constant |
| 0x02–0x1f | volatile | NOT persistent stats (change between re-saves of the same horse; kills the "compact dynamic stats" hypothesis) |
| 0x18 | const 0x0d | header constant |
| **0x20=0x03, 0x21=0x02** | const | format marker. [CORRECTED] verified for **DOC 2000 only** (16/16 cards). **DOC '99 cards show 0x20=17, 0x21=6** (2/2). The 03/02 header is DOC-2000-specific, not universal JP. (US/JP detection is unaffected — absence of SEGABEF0 is the real discriminator.) |
| 0x22–0x24 | const 0 | reserved |
| **0x25–0x27** | persistent | 3 lead-ID bytes = per-card unique ID/serial (stable across re-saves, high-bit/pointer-ish, NOT a 0–243 index into the racing table) |
| 0x28 … | persistent | kana **name**, then **sire**, then **dam** (mater table), each 0x7d-terminated |
| **0x43–0x44** | per-write | 2-byte trailer/nonce — NOT a plain sum/xor over the name region; most consistent with a per-write counter/RNG/CRC over a buffer including volatile bytes |

Tracks 2–3 are **Windows heap-pointer leak** (LE pointers repeating on a 0x20 stride; identical across captures only because same Flycast process). DOC 2000 never writes tracks 2–3. [CORRECTED minor] the trailer-bleed/name-sum hex constants in the source doc had transcription slips (e.g. sat2 name-region sum is 0x0701 not 0x092b; sat3 bleeds [88][7c] not [8d][7c]; the sat4 tracks-2/3 pointers differ by one byte rather than being byte-identical) — the mechanisms hold, only the literal bytes were off.

### 10.2 Kana table (SOLVED, byte-validated, conf 0.97)
Game-internal **1-byte-per-kana** (not Shift-JIS/EUC-JP). Column-major gojuon, 15 consonant-rows per vowel column. Rows 0–14: (none) K S T N H M Y R W G Z D B P. Columns (step 15): a=0x00–0x0e, i=0x0f–0x1d, u=0x1e–0x2c, e=0x2d–0x3b, o=0x3c–0x4a. Vowel anchors ア=0x00 イ=0x0f ウ=0x1e エ=0x2d オ=0x3c. Extended: 0x45=ン, 0x4b–0x4f small ァィゥェォ, 0x50–0x52 ャュョ, 0x53 ッ, 0x54 ー. Terminator/pad = 0x7d. ROM mater names are EUC-JP @ o2k 0x11106C / oc 0x0F9680; the ROM-EUC-JP → Unicode → card-kana bridge is byte-proven (164/167 fully kana-encodable).

### 10.3 Status
Edit path solved (a renamed card loaded into full gameplay). **CREATE not solved** — the 0x25–0x27 lead-ID scheme and the 0x43–0x44 trailer the cabinet accepts for a never-seen horse are the remaining unknowns. derbyoc ('99) card not yet captured (assumed same LR/identity format, unconfirmed).

---

## 11. Appearance (coat, silks, hood, sex)

**Two independent coat-encoding systems — keep separate:**

| | where | bytes | semantics |
|---|---|---|---|
| A. Player-card coat | card a1[8],a1[9] | 2 | base coat + special sub-id (starred specials live ONLY here) |
| B. CPU-record coat | racing table +22 (WE/o2k) / +19 (oc) | 1 | enum 0xC0–0xDE; generic Special, no variant selector |

[RECONCILED — no contradiction] **The CPU coat byte is ONE physical byte named under two conventions.** It is at **record-start +22** (relative to the record start, e.g. 0x108E03 WE-C — this is the convention doc-core uses) **= recBase+13** (relative to DOC-ROM-Studio's `recBase = record-start+9 = 0x108E0C`). Both point to the same absolute byte (WE-C horse #1: 0x108E19 = 0xCF = Light Gray), **244/244 vs the DB**. The '99 (oc) 28-byte record puts it at **record-start +19**. o2k (JP) is the same as WE: record-start +22.
> The earlier "+13 NOT +22" wording conflated the two bases: under **recBase**, coat = +13 and +22 = sharp; under **record-start**, coat = +22 and +31 = sharp. There is no real disagreement. Canonical for doc-core: **record-start +22 (32-byte) / +19 (28-byte)**, equivalently recBase+13.

### 11.1 Coat enums
- **Special (card a1[8]=63, a1[9]=sub-id):** 0 Okapi, 16 Cow, 48 Panda, 64 Platinum, 80 White, 112 Org Panda, 192 Zebra, 208 Cow_2, 240 Tiger (else generic Special). These 9 named specials are **player-card-only**; CPU records have a single generic Special=222 with no variant byte.
- **CPU enum (verified 244/244):** 0=Default, 192=Chestnut, 193=Black, 199=Brown, 202=Bay, 204=Dark Gray, 207=Light Gray, 222=Special.
- ROM coat-name string tables: simple set @0x0C68F0 (GRAY/CHESTNUT/BLACK/BAY/BROWN/SPECIAL/WHITE), granular set @0x0EE0AC (adds DARK CHESTNUT BAY / CHESTNUT BAY / DARK BAY). MALE/FEMALE/GELDING strings sit immediately after both → stat screen renders coat+sex from adjacent tables.
- **⚠ Latent tool bug:** the Card-Creator's `getColorName` and the ROM enum **disagree at 202/204/207** (tool says Brown/Chestnut/Bay; ROM says Bay/Dark Gray/Light Gray). The ROM enum is authoritative; the bug is latent because real cards use values the heuristic gets right. Fix = use the verified enum.

### 11.2 Silks & hood
- Silk pattern a2[15] (0x7B), 0–7: H-stripe/Half/Diagonal/Dots/Stars/V-stripe/Diamonds/Diamond rows.
- Silk color1 a2[14] (0x7C, primary) + color2 a2[13] (0x7D), palette 0–14 (Black…Yellow). Color1 is at the LOWER offset.
- Hood a2[26] (**0x70** [CORRECTED]), 0–63 (full range used; only partial label set known).
- Silk/hood names are **not** ASCII in the ROM (numeric palette indices only); only coat names and food names are ASCII. Hues/hood names are observational (~0.8).
- JP cards carry **no** coat/silk/hood (identity-only); JP appearance is cabinet-side, undecoded.

---

## 12. Race performance model (the hard one)

**Status: partial / empirical + located. NOT byte-decodable** — the stat→speed function is SH-4 FPU code, not a clean table. What is proven: stat *roles*, the *6-phase machine*, style-as-behavior, and the *located coefficient pools* (including one confirmed data table).

### 12.1 The 6-phase machine (verified)
`START → CORNER → OUT OF THE BOX → COMPETING → TENACIOUS → SPURT`. The six externals are 1:1 per-phase abilities (the phase-relevant external dominates speed in its phase). [CORRECTED] the ROM string is "OUT OF THE BOX" (often abbreviated OOB). Externals are roughly balanced (~220 total regardless of style → per-phase weights, not a single power score).

### 12.2 Stat roles (verified by statistics over 244 records)
- **Externals** = per-phase weights (local).
- **Internals** = global capacities: Stamina = energy pool / how long high speed holds; Speed = top-speed ceiling; Sharp = acceleration + whip responsiveness.
- corr(ext_total, int_total) ≈ 0.20 (WE-C) / 0.56 ('99) → independent axes.
- **Running style = behavior tag** (which phase to commit energy to), NOT a power tier.

### 12.3 Located coefficient data
- **DISTANCE → MULTIPLIER table (the single clearest race-math data recovered):** WE-C @0x10F210, WE-D @0x110A70, o2k @0x11439C (byte-identical across the three; **absent in '99**, which has a different track set). 9 distances (m): 1700,1800,2000,2100,2200,2400,2500,3000,3200. 12 multipliers: 1.391, 1.231, 1.032, 1.062, 0.889, 0.865, 0.842, 0.821, 0.800, 0.727, 0.667, 0.640. [CORRECTED] the curve is **per-distance non-monotonic** (index 2→3 ascends 1.0323→1.0625), not strictly descending — strengthening the "9-key lookup set indexing a 12-value factor table" reading. conf 0.9 distance-related / 0.6 exact use.
- **~7 embedded FPU float32 coefficient pools** inside the code region (0x46168, 0x53928 falloff/drain curve, twin 0x7C258/0x7C3C8 = two style branches, 0x828BC, 0x102760 per-style/personality matrix, 0xE7CA8, 0x102C00 per-position offsets). [CORRECTED] the code-adjacency proof: the bytes immediately before each pool are *more float32*, not 0xF0xx opcodes; the real evidence is high regional FP-instruction-word density near the pools (33/27/20% in ±256B) vs 2% at the isolated distance table. [CORRECTED] the 0x102760 pool head is `1.0×6, 0.9, 0.6, 1.0, 0.9, 0.9, 0.92` (not the "0.4 0.52 0.6 -0.6" row). conf 0.85 these are race pools / 0.4 each semantic label.
- **Track geometry table @0x0C8500** (72-byte float records, coords ±3000, lengths 1000–3000) = track spline geometry defining where phase boundaries fall — an *input*, disambiguated as NOT the stat→speed formula.

### 12.4 Working empirical model (best current understanding)
Per tick: `phase = phase_for(geometry, distance)`; `phase_ability = external[phase]`; `base_pace = distance_mult(distance)`; `surface_factor = f(dirt 0–255, track surface)`; `target = SpeedCap(internal_speed)`; `accel = g(internal_sharp, phase_ability)`; energy drains (curve @0x53928?) gated by Stamina; condition/trust/hearts multiply (card T2[44]/[36]/[37]); style biases which phase energy is committed (pools @0x7C258/0x102760); whip timing-window boosts accel via Sharp at trust/energy cost; small per-tick RNG. The `f/g/h/clamp` arithmetic is unproven — needs SH-4 disasm or a MAME race-trace (the highest-ROI path; the located pools confirm the constants).

---

## 13. Game text & string catalog

All in-game strings are NUL-terminated entries directly in the program ROM: **EN = 7-bit ASCII, JP = EUC-JP**. The only intentional in-string control code is **0x0A (literal newline)**; placeholders are printf-style (%s = name substitution, %d/%0d/%1d/%2d = numerics; [CORRECTED] whole-ROM set is wider: also %02d,%04x,%3d,%5d,%x,%u). [CORRECTED] leading attribute-byte family is **{0x0F, 0x03, 0xFF}**, not just 0x0F (meaning TBD — display attribute/color/face?).

- **EN Rev C (WE-C):** 26 curated labeled blocks, 1705 strings, all byte-verified. Categories: dialogue/flavor, menus/UI, names (167 auto-name list), banned-words (149-entry ASCII profanity list), attract/coin/card, branding.
- **EN Rev D (WE-D):** relocated + content-edited; Rev C offsets do not align. Anchors + per-block shifts derived (e.g. trainer dialogue +0x2258, banned list +0x2548). Edits: prepends horse name into race comments ("Jim's Gent, He's a tough horse…"), period→comma in prompts ("NEW HORSE, PRESS START"), added restricted-race text, copyright `SEGA,2001,2005` (vs WE-C `SEGA/Hitmaker,2001`). [CORRECTED] copyright true starts 0x10E7EE (Rev C) / 0x10FE22 (Rev D); flat textScan candidate count is 3702; %s count in the 5 largest EN blocks is 187.
- **JP (o2k/oc):** same categories, EUC-JP, different address ranges. [CORRECTED] ROM-Studio JPSPEC textRegion for DOC'99 is **[0x0C0000,0x140000]** (same as DOC2000), not the lower [0x0B0000,0x130000] (that was a density observation). JP track names stored fullwidth (e.g. "東京 ダート １２００Ｍ" @0xBDCA7 in DOC'99). JP reuses the `-n` honorific-substitution token (distinct from %s).

Cross-reference: the JP name tables (o2k ~0x10C000+, oc ~0xF8000+) overlap the racing/breeder name regions; coat/sex/leg-type label strings (栗毛/牝/逃げ) are the human-readable side of the numeric racing-record fields.

---

## 14. Cross-version summary (master offset grid)

| structure | oc '99 | o2k 2000 | WE-C | WE-D |
|---|---|---|---|---|
| serial / date | BAX0 / 1999-10-01 | BBX0 / 1999-10-01 | BEF0 / 2001-10-30 | BEF0 / 2001-10-30 |
| reader path | LR | LR | BR | BR |
| racing stats | 0x0F6902 /28 | 0x10AD1B /32 | 0x108E03 /32 | 0x10A14B /32 |
| racing names | 0x0F8480 /18 | 0x10CC68 /18 | 0x10AD50 /18 | 0x10C098 /18 |
| sire/dam (EN) or mater (JP) | 0x0F9680 /56 | 0x11106C /60 | 0x10BF1C (sire) / #85 (dam) /60 | 0x10D264 / +84*60 /60 |
| food table | 0x15C9EC (41 foods) | 0x171F34 (45) | 0x166A7C (45) | 0x16980C (45) |
| courses / G1 | 0x0BD875 / 0x0BDAD5 | 0x0CA335 / 0x0CA62D | 0x0C6940 / 0x0C6CA0 | 0x0C6260 / 0x0C65C0 |
| coat byte in CPU rec (record-start conv.) | +19 | +22 | +22 | +22 |
| distance→mult table | (absent) | 0x11439C | 0x10F210 | 0x110A70 |
| stat table identity | partial overlap (92/244 = WE-C) | 222/244 = WE-C (22 differ) | == WE-D | == WE-C |

Roster changes: WE-C→WE-D = 16 racing renames + 26 sires + all 84 dams renamed (Export licensing). DOC'99→DOC2000 = 64 racing renames. The breeding rosters differ in **order** (game order vs alphabetized JSON) but largely the same set (reconcile by name).

---

# OVERVIEW: state of understanding

## What is now FULLY understood (byte-proven, 1.0)
- **Racing stat record** — all confirmed fields (3× id, dirt, grade, 6 externals, 3 internals, running style, coat, banded personality) across all 4 versions, both 32- and 28-byte layouts. Proven to be the CPU-opponent roster.
- **Sire/dam breeding record** — full field layout (name, st/sp/sh, ac, composite, 6 externals, index) verified against the 681-horse JSON; pool structure (one contiguous block, continuous index) corrected.
- **US/WE card** — every load-bearing byte across 3 tracks: UID, identity/genetics, career/status, G1 bitfield, earnings, silks/hood/sex, breeding/markers; no on-card checksum.
- **JP card** — kana table, field layout, lead-ID + per-write trailer characterized; the on-card-vs-cabinet question resolved (identity-only, stats not on card or in nvram).
- **Food table** — 44-byte record, 7 effect columns (cols 0–2 = Speed/Stamina/Sharp), per-version food counts and tuning, the beer experiment exactly characterized.
- **Tracks/G1** — these are display string tables; all offsets/counts per version; full EN↔JP venue localization.
- **Coat/silks/hood/sex appearance** — both card and CPU systems, with the latent getColorName bug; CPU coat at record-start +22 (32-byte) / +19 (28-byte), equivalently DOC-ROM-Studio recBase+13.
- **Cabinet nvram** — EEPROM identity, SRAM money leaderboard + 57-entry track-record table, master/satellite replication, region layout (with corrected offsets/deltas).
- **Version diff & architecture** — header markers, memory map, the corrected stat-table identity (WE-C==WE-D only; o2k 22/244 differ), pointer/index tables, 4 MB layout.

## What is PARTIAL
- **Race performance formula** — roles + 6-phase machine + confirmed distance table + located FPU pools, but the closed-form stat→speed arithmetic needs SH-4 disasm or a MAME memory-trace.
- **Breeding inheritance rule** — community averaging model only; the real ROM routine and the name+44 composite's role are undecoded.
- **Hidden racing-record fields** — HIDDEN-A (+1), HIDDEN-B (+16), 16-bit HIDDEN-X (+23/+24); proven real (o2k rebalanced them) but semantics narrowed, not labeled.
- **Binary race-schedule table** — grade/surface/distance/prize/month and G1→course binding live in an undecoded binary block (candidate o2k 0x0CAD7B+).
- **JP card CREATE recipe** — lead-ID scheme + trailer algorithm for a never-seen horse unsolved (edit path works).
- **Effect columns 3–6, leaderboard flag0/flag1, the second 0x300000 program image** — narrowed, not pinned.

## Single highest-confidence area
**The 244-record racing stat table.** Every byte is classified and every confirmed field matches the per-version horse database 244/244 across all four ROMs, with the CPU-opponent framing independently proven by the beer-edit ROM diff. It is the bedrock the card decode, appearance, derived-attrs, and race-model work all build on.

## Single shakiest area
**The race performance model.** It is the only subsystem that is fundamentally not byte-decodable: the stat→speed function is SH-4 FPU code. We have stat roles, the phase machine, one confirmed data table, and located (but not semantically pinned) coefficient pools — but the actual arithmetic is unproven and requires disassembly or a live emulator trace to close.
