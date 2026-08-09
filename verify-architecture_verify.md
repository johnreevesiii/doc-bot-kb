# Adversarial Verification — `architecture.md` (NAOMI / SH-4 program architecture)

KEY: `architecture_VERIFY`
Verifier pass over `C:/DerbyOwnersClub/_core/areas/architecture.md`. Every claim
re-extracted from the real `.ic22` bytes with python3 (no binary read into context).
All four program ROMs used. Helper scripts in `C:/Users/johnr/AppData/Local/Temp/verify*.py`.

**Overall verdict: MOSTLY-SOLID.** The crown-jewel claim (the 244-record racing-stat
decode) is *fully confirmed at 244/244* against the per-version horse DBs, on BOTH
the 32-byte (WE/JP-2000) and 28-byte ('99) layouts. Header, menu strings, data-table
offsets, name/sire/dam tables, food, build credits, and the 0x300000 second image all
confirm. **Three claims are off and need correction** (title-slot widths, "record body
byte-identical across 3 versions", and the §5 function-pointer-table characterization).

---

## CONFIRMED (re-extracted from bytes)

### §1 Header — CONFIRMED (with one structural correction)
All 4 ROMs are exactly 4,194,304 B. Extracted:
| key | date(0x130) | serial(0x134) | title0 | sig@0x8000 |
|---|---|---|---|---|
| drbyocwc | 2001-10-30 | `BEF0` | ` DERBY OWNERS CLUB WE ---------` | `dc 99 02 0c 9c c8 21 0c` |
| derbyocw | 2001-10-30 | `BEF0` | ` DERBY OWNERS CLUB WE ---------` | `09 00 4a d2 0e e3 47 d0` |
| derbyo2k | 1999-10-01 | `BBX0` | ` DERBY OWNERS CLUB ------------` | `16 2f 04 7f fc f5 fc f6` |
| derbyoc  | 1999-10-01 | `BAX0` | ` DERBY OWNERS CLUB ------------` | `18 8b ee 02 f1 53 28 38` |

- `NAOMI`+spaces @0x000, `SEGA ENTERPRISES,LTD.` @0x010, region count `03 00 00 00` @0x15C — all 4 identical. ✓
- Menu strings @0x260/0x280/0x2A0 = `CREDIT TO NEW GAME START` / `CREDIT TO CARD GAME START` / `CREDIT TO CONTINUE`. ✓
- Init descriptor @0x420 = `00 10 02 0c 00 00 02 0c 02 00 01 01 00` then FF — byte-identical drbyocwc & derbyo2k. ✓
- WE vs JP reader trigger lives in title slot 0 (`WE` vs blank). ✓

**CORRECTION (structural):** the doc's §1 table lists title slots at 0x030, 0x060,
0x080, 0x0A0... with width 0x30. The real NAOMI title slots are **0x20 (32 B) wide at
0x030, 0x050, 0x070, 0x090, 0x0B0, 0x0D0, 0x0F0, 0x110** (8 slots, 0x100 total). The
slot *contents* the doc lists are right (EXPORT/KOREA/AUSTRALIA/?/!/@), but the offsets
and the 0x30 width are wrong. Slot4 is unambiguously `IN AUSTRALIA`.

**Minor label nit:** the §1 prose at 0x134 says "`BBX0`(2000)". derbyo2k carries `BBX0`
but its header date is **1999-10-01**, not 2000 (the marketing name is "DOC 2000"). The
table row is correct; only the inline "(2000)" is loose.

### §2 / §3 code layout + 0x8000 — CONFIRMED
- 0x1000 = `24 d0 25 d1 12 20 09 00 ...` (drbyocwc & derbyo2k identical first 8B). SH-4 LE prologue pattern. ✓
- 0x8000 8-byte values are version-unique fingerprints (re-extracted, match seed exactly). The doc's reframing ("it's version-specific code, not a dedicated signature field") is reasonable and the fingerprint use is valid. ✓
- 0x200000+ in drbyocwc is FF-filled. ✓
- Build credits @0x17FF80 (drbyocwc): real ASCII incl. `NEC Corpolation` (sic), `Home Multimedia Development Division`, `Client Server Software Development Division`, `NEC Software,Ltd`. ✓ NEW, confirmed.
- **0x300000 second image — CONFIRMED:** drbyocwc 0x300000 = `1f d0 20 d1 12 20 09 00 19 c7 1f d1 06 62 1b 22`, and derbyo2k 0x300000 is **byte-identical** to it. Same `12 20 09 00` prologue family as 0x1000 with different PC-relative displacements. The "identical first 16 B across versions" claim holds. (Still "likely a second bank/boot mirror"; nature unproven — fair conf 0.8.)

### §4 Racing-stat table (244 records) — **CONFIRMED 244/244** (the strongest result)
Record starts & strides re-extracted and validated:
- drbyocwc `0x108E03`, derbyocw `0x10A14B`, derbyo2k `0x10AD1B` — stride **32**. ✓
- derbyoc `0x0F6902` — stride **28**. ✓

**Full 1:1 decode validation vs `DOC_COMPLETE_HORSE_DATABASE_DRBYOCWC.md` (all 244):**
| field | offset (32-byte) | result |
|---|---|---|
| grade | +8 {0 Ungraded,1 G3,2 G2,3 G1} | **244/244** |
| dirt | +5 | **244/244** |
| externals (start/corner/oob/comp/ten/spurt) | +9..+14 | **244/244** |
| internals (stamina/speed/sharp) | +29/+30/+31 | **244/244** |
| coat | +22 {0,192,193,199,202,204,207,222 enum} | **244/244** |
| personality | +23 (banded 0-47 Rough … 240-255 Proud) | **244/244** |
| style/leg-type | +21 {0 FR,1 SD,2 LS,3 SR,7 Almighty} | **244/244** |

The banded personality boundaries and the "style is STORED not derived" claim are both
**confirmed** by the perfect match. This is a genuinely excellent decode.

**28-byte derbyoc layout — CONFIRMED 244/244** vs `DOC_COMPLETE_HORSE_DATABASE_DERBYOC.md`:
grade `+7`, dirt `+4`, externals `+9..+14`, internals `+24/+25/+26`, coat `+19`,
personality `+20`, style `+18` — all **244/244**. ✓

Column distribution scan (drbyocwc, 244 recs) matches doc: +0 const0, +1 {0:175,1:62,2:7},
+2/+3 ID 1-244, +4 const0, +5 dirt(8/104/136/137 common), +6/+7 const0, +8 grade(80/65/50/49),
+9..+14 externals, +21 style(0-7), +22 coat, +23 pers, +24 aux, +25 0-based ID, +29..+31 internals.

### §5 index/RAM tables — CONFIRMED
- **Index table @0x15B234 (derbyo2k):** u16 LE = `87,14,15,16,17,18,...` exactly the doc's "Entry0:87, Entry1-73:14-86". ✓ conf 1.0.
- **RAM-ptr table @0x15B1E0 (derbyo2k):** entry[0]=`0x016659E0` matches doc. ~21 LE32 RAM pointers. ✓ (refinement below.)

### §6 other tables — CONFIRMED
- Name table drbyocwc @0x10AD50 stride18: id1 `Gold Queen`, id94 `Royal Rumble`, id167 `White Knight`. ✓ Rev D @0x10C098 same. ✓
- Sire table @0x10BF1C stride60: sire1 `Maple Syrup`, ac@name+36 = `f0 00 00 00`. ✓ Rev D @0x10D264 same (Maple Syrup, ac=0xF0). ✓
- Dam table @0x10D2CC: dam1 `Ferranti's Folly`, dam2 `Speed Queen`. ✓ Rev D @0x10E614 `Pierogi Prince`. ✓
- Food @0x171F34 derbyo2k: `a5 cb a5 f3 a5 b8 a5 f3` = EUC-JP ニンジン (carrot). ✓

---

## CORRECTIONS / OFF CLAIMS

1. **§4 "record body BYTE-IDENTICAL across drbyocwc, derbyocw, derbyo2k" — OFF.**
   Verified by concatenating all 244 records (32B each) and comparing:
   - drbyocwc == derbyocw → **True** (the two WE builds are identical bodies).
   - drbyocwc == derbyo2k → **FALSE** (differ across the full table).
   The doc was fooled by spot-checking only rec0/rec1/rec243 (which DO match). Evidence:
   +16 field distribution is `{0:200,1:37,2:7}` for both WE builds but `{0:201,1:34,2:9}`
   for derbyo2k. **Restate as:** "32-byte body is identical between Rev C and Rev D;
   derbyo2k (JP-2000) shares the layout but not all values."

2. **§4 "+15..+20 const 0" — OFF.** Byte **+16 is a real field** (values 0/1/2, dist
   200/37/7 in WE), not a constant. +15,+17,+18,+19,+20 are const 0 across 244; +16 is not.
   (Meaning of +16 unknown — candidate "grade-aux pair" with +1; both are small-int flags.)

3. **§5 function-pointer table @0x15729C — PARTIALLY OFF (overstated at conf 1.0).**
   The table exists and ptr[0]=`0x3E7C0`, ptr[1]=`0x5AC60`, ptr[5]=`0xCAE0` match the doc
   exactly. BUT:
   - The doc's "each target shows prologues (`2f e6` push, `4f 22` sts.l PR)" is **not
     true at the target's first bytes.** Extracted: ptr[0]→`83 66 31 60`, ptr[1]→`0b 00
     09 00` (= `rts; nop`, a function *end*), ptr[5]→`f6 6d 0b 00`. The `2f e6`/`4f 22`
     opcodes appear *a few bytes in* on some targets, not at offset 0.
   - "200 contiguous valid pointers" undercounts/over-cleans: **295** contiguous values
     fall in 0x1000–0x300000, but **162 of the targets begin with the 16-bit word 0x0000**
     (point at zero-padded/data regions), and some targets are clearly data (ptr[10]→
     `00 00 00 43 00 00 00 44`). So it is NOT cleanly "200 real function entries, conf 1.0."
   - **Restate as:** "@0x15729C is a large LE32 address table (~200-295 entries) into the
     code image; it is a useful disassembly *seed* list, but entries are a mix of true
     function entries, mid-routine addresses, and data pointers — confidence ~0.6, not 1.0.
     Verify each target in Ghidra before treating as a function."

4. **§5 RAM-ptr table "21 pointers into 0x016xxxxx" — slightly imprecise.** Entries
   [0]-[11] are 0x0166xxxx-0x016Fxxxx; entries [12]-[20] are **0x017xxxxx**
   (0x017061E0 … 0x0174ABC0). So ~21 RAM pointers spanning **0x0166xxxx-0x0174xxxx**, not
   strictly 0x016xxxxx. entry[0]=0x016659E0 is correct.

---

## EXTENDED (new, from this pass)

- **Beer/feeding effect bytes localized.** Diffing `beer_effects_test.ic22` vs base
  `epr-22336c.ic22` (both 4 MB): **only 12 bytes differ, in 2 runs:**
  `0x1671FC-0x167201` (6 B: `00*6` → `02*6`) and `0x167228-0x16722D` (6 B: `00*6` → `04*6`).
  This is in a parameter block ~0xA0 before the food *name* table (food names @0x171F34).
  Looks like per-item stat-delta / effect values (zeroed in base, set to 2 and 4 in the
  edited ROM). This is the concrete item-effect region for follow-on RE — base game has
  these as 0, the experiment wrote effect magnitudes. (NEW, conf 0.7 on "these are the
  feeding/item effect magnitudes"; conf 1.0 that these 12 bytes are the only delta.)

- **Cross-version table-body relationship clarified:** Rev C and Rev D are the *same*
  244×32 racing table; the JP-2000 build differs. Use Rev C↔Rev D as a free differential
  pair (only names/code shift, stats identical) when validating the 32-byte map.

---

## How verified (scripts)
`C:/Users/johnr/AppData/Local/Temp/verify{1..18}.py` — header/serial/sig, title slots &
menus, record starts, 244-col distribution, 244/244 DB cross-check (32B and 28B), fn-ptr
table extent + target prologues, index/RAM tables, name/sire/dam, food, credits,
0x300000 image, cross-version body identity, beer diff. All read specific byte ranges
only; no binary loaded into context.
