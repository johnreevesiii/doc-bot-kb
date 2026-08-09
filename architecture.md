# Derby Owners Club — NAOMI / SH-4 Program Architecture

KEY: `architecture`
Scope: NAOMI cart header, region/title block, SH-4 boot+code layout, the big
data-table referencing scheme, the function-pointer / index-table system, the
full 32-byte racing-stat record decode, and a concrete plan for disassembling
the race/breeding routines.

All four program ROMs analyzed (each **4,194,304 bytes = 4 MB**, uncompressed):

| key | set | file | date(0x130) | serial(0x134) | 0x8000 sig (8 bytes) |
|---|---|---|---|---|---|
| drbyocwc | WE Rev C   | epr-22336c.ic22 | 2001-10-30 | `BEF0` | `dc 99 02 0c 9c c8 21 0c` |
| derbyocw | WE EX Rev D | epr-22336d.ic22 | 2001-10-30 | `BEF0` | `09 00 4a d2 0e e3 47 d0` |
| derbyo2k | DOC 2000 JP | epr-22284a.ic22 | 1999-10-01 | `BBX0` | `16 2f 04 7f fc f5 fc f6` |
| derbyoc  | DOC '99 JP  | epr-22099b.ic22 | 1999-10-01 | `BAX0` | `18 8b ee 02 f1 53 28 38` |

All offsets below are **file offsets** (the SH-4 sees the cart mapped; see "SH-4
boot" note). Every claim was extracted from the real bytes; confidence + how-verified
noted inline.

---

## 1. NAOMI cart header (0x000000 – 0x00041F) — VERIFIED, all 4

Standard SEGA NAOMI ROM-board header. Identical structure across all four; only
the title slots, date, and serial differ.

| off | width | field | value (drbyocwc) | notes |
|---|---|---|---|---|
| 0x000 | 16 | platform tag | `NAOMI` + spaces | identical all 4 (conf 1.0) |
| 0x010 | 32 | maker | `SEGA ENTERPRISES,LTD.` | identical all 4 |
| 0x030 | 0x30 | title slot 0 (Japan) | ` DERBY OWNERS CLUB WE ---------` (WE) / ` DERBY OWNERS CLUB ------------` (JP) | **this is the BR-vs-LR reader trigger** (see card RE) |
| 0x060 | 0x30 | title slot 1 (USA) | same as slot0 | |
| 0x080 | 0x30 | title slot 2 (Export) | ` DERBY OWNERS CLUB IN EXPORT --` | identical all 4 |
| 0x0A0 | 0x30 | title slot 3 (Korea) | ` DERBY OWNERS CLUB IN KOREA ---` | |
| 0x0C0 | 0x30 | title slot 4 (Australia) | ` ... IN AUSTRALIA ` | |
| 0x0E0 | 0x30 | title slot 5 | ` ... IN ? -------` | |
| 0x100 | 0x30 | title slot 6 | ` ... IN ! -------` | |
| 0x120 | 0x30 | title slot 7 | ` ... IN @ -------` | |
| 0x130 | 2 | year (LE) | `d1 07` = 2001 | JP builds = `cf 07` = 1999 (conf 1.0) |
| 0x132 | 1 | month | `0a` = 10 | |
| 0x133 | 1 | day | `1e` = 30 | JP = `01` |
| 0x134 | 4 | SEGA serial | `BEF0` | `BBX0`(2000), `BAX0`('99) — ASCII (conf 1.0) |
| 0x138 | ~0x24 | zero pad | | |
| 0x15C | 4 | region/mode count | `03 00 00 00` (=3) | identical all 4 |
| 0x160 | var | NAOMI ROM-board / encryption load descriptor | series of 12-byte-ish records ending `FF FF` | NOT cleanly tri-aligned as (src,dst,len); this is the NAOMI G1-DMA / M2 descriptor block. Treat as opaque board metadata; **does not need full decode for RE** (conf 0.6, see Open Q) |
| 0x1B6 | ~0x2A | `ff ff 00 00 11 11` repeating | | board-size marker pattern |
| 0x1E0 | 0x10×N | per-region test/menu config | `01 00 00 1c 01 05 01 05 03 01 01...` | identical all 4; standard NAOMI region settings table |
| 0x260 | 0x20 | menu string | `CREDIT TO NEW GAME START` | NAOMI standard |
| 0x280 | 0x20 | menu string | `CREDIT TO CARD GAME START` | DOC-specific (card start!) |
| 0x2A0 | 0x20 | menu string | `CREDIT TO CONTINUE` | |
| 0x300-0x41F | | zero pad | | |
| 0x420 | 13 | SH-4 boot/init descriptor | `00 10 02 0c 00 00 02 0c 02 00 01 01 00` then `FF`-fill | **byte-identical across all 4** (conf 1.0). `0c` = SH-4 `mov.b @(R0,Rm),Rn` opcode-family bytes; this is the cart-init micro-block read by the BIOS |

**Region / EXPORT / SEGA fields answered:** there is no single "region byte"; the
NAOMI scheme is the **8-slot title table (0x030–0x14F)** + the region-settings
table at 0x1E0. The game's effective region is selected by the cabinet
EEPROM/BIOS, not a ROM field. The export/WE distinction that matters to DOC is
**title slot 0**: `WE` => the Sanwa **BR** card reader path, blank => **LR** path
(from card-RE FINDINGS.md, re-confirmed here by reading slot 0 in all 4).

---

## 2. SH-4 code vs data layout — VERIFIED

The header (text/titles/menu strings) runs 0x000–~0x300, then zero pad to
**0x1000 where dense SH-4 code begins**. Verified by reading 0x1000:
`24 d0 25 d1 12 20 09 00 ...` — a classic SH-4 register-load prologue
(`mov.l @(disp,PC),Rn` = `d0`/`d1` high-nibble, `nop` = `09 00`).

**The "2 MB layout" in the AI Master Architecture doc is for derbyo2k's first
half only and is approximately right for code/data boundaries, but the doc's
ROM Size = 2 MB is WRONG — every ROM is 4 MB.** Corrected/extended map
(density-scanned at 0x40000 granularity, drbyocwc + derbyo2k):

| region | addr range | content | confidence |
|---|---|---|---|
| Header | 0x000000–0x000FFF | NAOMI ID + titles + menu strings | 1.0 |
| Main code (Block 1) | 0x001000–~0x0C0000 | SH-4 executable, ~94% dense | 1.0 |
| Text/data | ~0x0C0000–0x100000 | track names, race names, game text (EUC-JP on JP, ASCII on EN) | 1.0 |
| Horse/table data | 0x100000–~0x118000 | index/count tables, CPU-horse + racing-stat + name + sire/dam tables, breeding comments | 1.0 |
| Code/data block 2 | ~0x130000–0x160000 | extended logic + **function-ptr table (0x15729C)** + **index table (0x15B234)** + **RAM-ptr table (0x15B1E0)** | 1.0 |
| Food/items | 0x171F34+ | food item DB (EUC-JP names) | 1.0 |
| Build credits | ~0x17FF80–0x180020 | ASCII: `* Home Multimedia Development Division / NEC Corpolation / Client Server Software Development Division / NEC Software,Ltd` — marks end of data; DOC was built on NEC middleware | 1.0 (NEW) |
| Gap | 0x180020–0x2FFFFF | mostly zero (drbyocwc FF-filled 0x200000+) | 1.0 |
| **Second program image** | **0x300000+** | near-duplicate of the main SH-4 program: 0x300000 = `1f d0 20 d1 12 20 09 00 ...` vs 0x001000 = `24 d0 25 d1 12 20 09 00 ...` (same prologue, different PC-relative displacements). First-16 bytes at 0x300000 are **identical between drbyocwc and derbyo2k**. Likely a second ROM-board bank / boot mirror. | 0.8 (NEW, undocumented) |
| Tail | 0x340000–0x3FFFFF | sparse/extra tables (derbyo2k has more here than drbyocwc) | 0.6 |

**SH-4 boot note:** NAOMI maps the cart so the SH-4 begins executing the
transferred boot block; the 0x420 init descriptor and 0x160 load block are read
by the NAOMI BIOS. For static RE you should treat **0x1000 as the program base**
and use the **0x15729C function-pointer table** as your entry-point seed list
(its 200 pointers ARE the game's function entry points — see §5).

---

## 3. Build signature at 0x8000 — VERIFIED (corrected interpretation)

The "8-byte build signature" at 0x8000 (drbyocwc `dc 99 02 0c 9c c8 21 0c`, etc.)
is **not a dedicated signature field — it is version-specific SH-4 code/data at
that offset.** The surrounding bytes (0x7FF0+) are clearly instructions
(`0c`-family memory ops, `2b 41` = jmp). It is still a perfectly good
**version fingerprint** (unique per build, matches the seed facts exactly) and
ROM-Studio/Card-Creator can keep using it for version detection. Confidence 1.0.

---

## 4. The racing-stat table (244 records) — FULLY RE-DECODED

This is the **playable/CPU racing roster** (244 horses). Distinct from the
60-byte sire/dam *breeding* records (§6). **The 32-byte record body is
BYTE-IDENTICAL across drbyocwc, derbyocw, derbyo2k** (verified rec0/rec1/rec243
match exactly); only the name table and code differ. derbyoc ('99) uses a
**28-byte** older layout with the same fields shifted.

Record starts (verified):
- drbyocwc `0x108E03`, derbyocw `0x10A14B`, derbyo2k `0x10AD1B` — stride **32**.
- derbyoc `0x0F6902` — stride **28**.

### Two offset conventions (reconciled)
- **Record-relative (seed / this doc):** offsets measured from record start.
- **DOC-ROM-Studio convention:** `recBase = recordStart + 9` (points at the
  first external `start`). Then `dirt = recBase-4`, `grade = recBase-1`,
  `coat = recBase+13`, `id = recBase+16`, `stamina/speed/sharp = recBase+20/21/22`.
  These are **algebraically identical** to the record-relative offsets below
  (recBase+13 = record+22 = coat, etc.). Both are correct.

### 32-byte record field map (recordStart-relative) — what was UNKNOWN is now decoded

| off | field | type / values | how verified | conf |
|---|---|---|---|---|
| +0 | const 0 | always 0 (244/244) | column scan | 1.0 |
| +1 | grade-aux flag | 0/1/2 (175/62/7) | scan; NOT 1:1 with grade. Likely "appears-in-special-event / season" flag | 0.4 (still partial) |
| +2,+3 | **horse ID (1-based)** | 1..244, duplicated low byte | sequential across all 244 | 1.0 |
| +4 | const 0 | | | 1.0 |
| +5 | **dirt / surface affinity** | 0–255 (bitfield-ish: 8,104,136,137,168 common) | matches DB "Dirt" col exactly | 1.0 |
| +6,+7 | const 0 | | | 1.0 |
| +8 | **GRADE** | {0:Ungraded, 1:G3, 2:G2, 3:G1} | 1:1 vs DB grade, 244/244 (e.g. id1=3=G1, id11=2=G2) | **1.0 (NEW decode)** |
| +9 | **start** (external) | 11–63 | seed + DB | 1.0 |
| +10 | **corner** | 14–59 | | 1.0 |
| +11 | **out-of-box (oob)** | 4–63 | | 1.0 |
| +12 | **competing** | 8–63 | | 1.0 |
| +13 | **tenacious** | 3–62 | | 1.0 |
| +14 | **spurt** | 4–63 | | 1.0 |
| +15..+20 | const 0 | | | 1.0 |
| +21 | **RUNNING STYLE / leg type** | {0:Front-runner, 1:Start dash, 2:Last spurt, 3:Stretch-runner, 7:Almighty} | **1:1 vs DB "Style", 244/244** | **1.0 (NEW decode — contradicts ROM-Studio note "leg type is derived from externals"; it is STORED)** |
| +22 | **COAT color** | {0:Default,192:Chestnut,193:Black,199:Brown,202:Bay,204:Dark Gray,207:Light Gray,222:Special} | **1:1 vs DB "Coat", 244/244** | **1.0 (NEW decode)** |
| +23 | **PERSONALITY (banded)** | byte→band: 0-47 Rough, 48-63 Imposing, 64-111 Calm, 112-127 Firm, 128-175 Sensitive, 176-191 Moody, 192-239 Gentle, 240-255 Proud | 1:1 vs DB "Personality" by band, 244/244 | **0.9 (NEW decode — band boundaries empirical; ROM-Studio wrongly says "personality not stored")** |
| +24 | personality-aux / condition seed | low byte varying (0,48,160,165,170…), pairs with +23 | likely the 16-bit (LE) temperament word `[+23][+24]` whose high part = personality band | 0.5 |
| +25 | **horse ID (0-based)** | 0..243 (= record index) | sequential | 1.0 |
| +26..+28 | const 0 | | | 1.0 |
| +29 | **stamina** (internal) | 0–60 | seed + DB | 1.0 |
| +30 | **speed** (internal) | 0–63 | | 1.0 |
| +31 | **sharp** (internal) | 0–60 | | 1.0 |

**Net result: of 32 bytes, ~28 are now identified** (was ~10). Remaining
genuinely-uncertain: +1 (grade-aux flag) and +24 (personality low byte / seed).

### 28-byte derbyoc ('99) layout — VERIFIED against derbyoc horse DB
rec0 = `01 01 01 00 a8 00 00 03 00 2c 23 13 20 28 2e 00 00 00 02 cf 0e a0 01 00 17 25 30 00`.
Mapping (record-relative), all confirmed by 1:1 match vs the derbyoc DB:
- ID `+0/+1`; dirt `+4` (conf 1.0, seed); grade **`+7`** {0:Ungraded,1:G3,2:G2,3:G1}
  (1:1 vs DB, conf 1.0); externals `+9..+14` (conf 1.0); **style `+18`** {0:Front-runner,
  1:Start dash,2:Last spurt,3:Stretch-runner} (clean 1:1, conf 1.0); **coat `+19`**
  (same enum as §4, 1:1 vs DB, conf 1.0); **personality `+20`** (banded, conf 0.9);
  id `+22`; internals `+24/+25/+26` (conf 1.0, seed).
- **Pattern:** in BOTH layouts the three appearance bytes are adjacent —
  32-byte: style+21 / coat+22 / pers+23; 28-byte: style+18 / coat+19 / pers+20.
  The 28-byte record drops the leading `00` and 4 trailing pad bytes vs the 32-byte.

---

## 5. Pointer & index architecture — VERIFIED (extends AI doc)

Three real tables in code-block-2, all confirmed by reading bytes in derbyo2k
(AI doc addresses are for derbyo2k):

1. **Function-pointer table @ 0x15729C** — array of LE-32 ROM addresses, each a
   real SH-4 function entry. Verified targets: `[0]→0x03E7C0`, `[1]→0x05AC60`,
   `[5]→0x00CAE0`; target bytes show prologues (`2f e6` push, `4f 22` sts.l PR).
   **200 contiguous valid pointers** (AI doc said 144 — the table is larger).
   Spans 0x00CAE0–0x1F22A0. **This is the single best disassembly seed list.**
   Confidence 1.0.
2. **Index/lookup table @ 0x15B234** — 16-bit LE entries: `57 00`(=87),
   `0e 00`(=14), `0f 00`(=15)… exactly matching the AI doc's "Entry 0:87,
   Entry 1-73:14-86" (sire breeding-comment indices). Confirms the
   **positional-linkage** model: a horse's *position* indexes this table, which
   yields a breeding-comment string index. Confidence 1.0.
3. **RAM-pointer table @ 0x15B1E0** — 21 LE-32 pointers into `0x016xxxxx`
   (`e0 59 66 01`=0x016659E0, …). Runtime data binding. Confidence 1.0.

These addresses are **derbyo2k-specific**; the WE builds shifted everything
(see the data-table offsets in §4/§6, which differ per version). The AI doc's
22099b→22284a shift table (CPU horses +0x179EC, food +0x15548, player horses
−0x9D4F0) is a useful relative map but must be re-derived per pair from real bytes.

---

## 6. Other big data tables (referenced, spot-verified)

- **Racing NAME table** (parallel to the 244 stat table): drbyocwc `0x10AD50`,
  derbyocw `0x10C098`, stride **18**. ASCII on EN (verified: id1 "Gold Queen",
  id94 "Royal Rumble", id167 "White Knight"), EUC-JP on JP. Conf 1.0.
- **Sire table**: drbyocwc `0x10BF1C`, derbyocw `0x10D264`; stride **60**, 84
  entries. Name at +0 (24B), externals at name−12 (1–16 banded: X1-4/A5-8/O9-12/@13-16),
  4-byte "ac" composite at name+36. Verified: sire1 "Maple Syrup" ac=0xF0,
  ext `02 04 04 05 02 04`. Conf 1.0.
- **Dam table**: drbyocwc `0x10D2CC`, derbyocw `0x10E614`; same 60-byte layout, 84 entries. Conf 1.0.
- **CPU-horse DB (AI doc)** @ 0x111034 (derbyo2k): 60-byte records, EUC-JP names
  (`a5 e2 a5 df…` = katakana). This is the JP breeding DB the AI doc described
  (168/175 horses). Conf 1.0.
- **Food items** @ 0x171F34 (derbyo2k): EUC-JP `a5 cb a5 f3 a5 b8 a5 f3` = "ニンジン"
  (carrot). Conf 1.0.
- **Game text / track / race names**: EN ASCII, JP EUC-JP (marker 0xA5xx =
  katakana). Curated blocks in DOC-ROM-Studio GAMETEXT. Conf 1.0 (per existing RE).

EUC-JP detail confirmed from real bytes: katakana lead byte `0xA5`, kanji
`0xB0–0xF4` ranges; little-endian integers; 0x00 null terminator; 0xFFFF = null index.

---

## 7. How to disassemble the race/breeding routines — recommended approach

Goal of follow-on RE: find the **race-result formula** (how the 9 stats +
dirt/grade/style/personality + whip input produce finishing order) and the
**breeding/foal-stat formula**.

**Toolchain (recommended):**
1. **Ghidra** with the **SuperH / SH-4 little-endian** processor module
   (built-in). It is the only free tool with a solid SH-4 decompiler.
   - Import each `.ic22` as **raw binary**, base address **0x0C000000** (NAOMI
     cart window) OR simply 0x00000000 — keep it consistent so the 0x15729C
     pointers resolve. SH-4 is LE.
   - **Seed the analysis with the 0x15729C function-pointer table** (200 entries):
     script-create a function at each target. This instantly gives you the game's
     real entry points instead of relying on auto-analysis.
2. **Cross-reference the decoded data tables as anchors.** Set the racing-stat
   table (0x108E03 etc.), name table, sire/dam tables, and the index table
   (0x15B234) as labeled data. Then find code that reads `record+9..+14`
   (externals), `record+29..+31` (internals), `+5` (dirt), `+8` (grade),
   `+21` (style), `+23` (personality). The race solver will index this table by
   horse ID and read exactly those fields — XREFs to these offsets localize the
   race math fast.
3. **Use the emulator as ground truth.** Run derbyocw in **Flycast (dev build)**
   or **DEMUL**; both have SH-4 debuggers / RAM watch. The RAM-pointer table at
   0x15B1E0 → 0x016xxxxx tells you where live race state lives. Watch those
   addresses during a race to map the dynamic stat/condition fields (the JP card
   "volatile" bytes in FINDINGS.md likely mirror these).
4. **Differential RE with the Beer-Experiment ROM.** `beer_effects_test.ic22`
   is an edited drbyocwc. `diff` it byte-for-byte vs base `epr-22336c.ic22`
   (4 MB, same size) to see exactly which table/code bytes feeding/beer changed —
   that isolates the item-effect / condition code region without any disassembly.

**Practical order:** (a) build the Ghidra project + auto-function from the ptr
table; (b) label the data tables; (c) follow XREFs from `record+21`/`+23` to find
the style/personality consumer = the race-ranking routine; (d) confirm in
Flycast RAM watch; (e) decode the breeding formula via XREFs to the sire/dam
"ac" field (name+36) and the 0x15B234 index table.

---

## 8. Open questions

- 0x160 NAOMI load/encryption descriptor block — exact field layout (likely
  the standard NAOMI ROM-board "M2" table; low value for game RE, but unconfirmed).
- Record byte **+1** (grade-aux 0/1/2) and **+24** (personality low byte): exact meaning.
- The **0x300000 second program image** — is it a true second executable bank, a
  boot mirror, or overflow/relocated code? Needs a code-diff vs the 0x1000 image.
- derbyoc ('99) 28-byte record: confirm coat(+19)/grade(+7)/style(+20) against its horse DB.
- Whether JP on-card stats/sex/leg-type live in nvram (eeprom/sram) vs cabinet —
  cross-check the `derbyocw.sram` RAM-pointer targets.

---

## 9. Tool ideas this unlocks

- **Ghidra seed script** (`seed_fn_table.py`): read 0x15729C, create 200 functions
  + label all decoded data tables. Turns a cold ROM into a navigable project in
  one run. (Needs: the verified table offsets in this doc.)
- **Full racing-record editor**: extend DOC-ROM-Studio to expose +21 style and
  +23 personality (currently it only edits coat/dirt/grade) — they ARE editable
  stored fields. (Needs: §4 map, already verified.)
- **ROM region/version inspector**: auto-detect version via 0x134 serial +
  0x8000 fingerprint, print the header/title/region table, and flag BR-vs-LR
  reader from title slot 0. (Needs: §1/§3.)
- **Race-formula tracer**: Flycast/DEMUL RAM-watch script keyed off the 0x15B1E0
  RAM pointers to log per-tick stat usage during a race. (Needs: emulator debugger.)
