# Breeding / Mater Inheritance System — Adversarial Verification

Verifier pass over `breeding-system.md`. Every claim below was re-extracted directly from the four
`.ic22` ROMs with python3 (bytes only, never loaded into context) and cross-checked against
`DOC_Breeder_Studio_data.json` and `DOC Breeding and Racing Strategy.txt`.

Date: 2026-06-03. Tooling: `C:/Users/johnr/AppData/Local/Temp/brd*.py`.

---

## VERDICT: mostly-solid (field map is correct; the counts, the "two separate arrays" model, and the JSON-index alignment are WRONG)

The 60-byte record field map, the JP layouts, the stride/base offsets, the `ac` range, the pad/reserved
bytes, the breedFoal()/deriveRunningStyle() transcription, and the ac↔dirt correlation are all CONFIRMED
against bytes. Three structural claims are wrong or overstated and need correction.

---

## CONFIRMED claims (re-extracted from bytes)

### Table bases & strides (all 4 versions) — CONFIRMED
| Version | base | stride | result |
|---|---|---|---|
| Rev C drbyocwc | `0x10BF1C` | 60 | 167 contiguous index-valid records (idx 1..167), breaks at k=167 |
| Rev D derbyocw | `0x10D264` | 60 | 177 contiguous index-valid records, breaks at k=177 |
| derbyo2k | `0x11106C` | 60 | 167 contiguous records (idx@+0), breaks at k=167 |
| derbyoc | `0x0F9680` | 56 | 167 contiguous records (idx@+0), breaks at k=167 |

The "dam base" the doc lists (`0x10D2CC` Rev C, `0x10E614` Rev D) is real but it is NOT a separate
array — it is exactly `sire_base + 60*84`, i.e. the offset of record #85 inside ONE contiguous block.
`(0x10D2CC - 0x10BF1C)/60 == 84.0` and `(0x10E614 - 0x10D264)/60 == 84.0`.

### 60-byte EN record field map — CONFIRMED (Rev C, 167 records)
- `+0x00` name 24B ASCII, NUL-padded — confirmed (Maple Syrup, Heart Lake, …).
- `+0x18` st u32, `+0x1C` sp u32, `+0x20` sh u32 — confirmed. Ranges: st 10..54, sp 12..63, sh 10..60.
- `+0x24` ac (low byte of u32) — confirmed, range 0..255.
- `+0x28` reserved u32 = **0 in 167/167 records** — confirmed.
- `+0x2C` composite 4B — confirmed present, high entropy.
- `+0x30` externals[6] — confirmed, **every byte in 0..15** across all 167 records.
- `+0x36..0x37` pad = 00 00 in 167/167 — confirmed.
- `+0x38` index u16 (1-based) — confirmed, runs 1..167.
- `+0x3A..0x3B` pad = 00 00 in 167/167 — confirmed.

Worked example Maple Syrup @ 0x10BF1C matches the doc byte-for-byte:
st=39,sp=19,sh=34,ac=240,reserved=0,composite=[0,224,17,240],ext=[15,3,6,10,4,12],index=1. ✓

### JP layouts — CONFIRMED
- derbyo2k stride 60: idx u32@+0, name EUC-JP@+4, st@+28, sp@+32, sh@+36, ac@+40, composite@+48, ext@+52.
  #1 トロットサンダー (35,27,33,217) ext[4,8,4,12,14,8]; #167 ヒシアマゾン. Parses cleanly for all 167.
- derbyoc stride 56: idx u32@+0, name EUC-JP@+4 (16B), st@+24, sp@+28, sh@+32, ac@+36, composite@+44, ext@+48.
  #1 ティンバーウルフ (34,41,19,211); #2 ホワイトノーザー. Parses cleanly for all 167.

### Cross-version stat alignment — CONFIRMED (with a caveat, see below)
- derbyo2k #1 トロットサンダー (st35,sp27,sh33,ac217,ext[4,8,4,12,14,8]) == EN Rev C **idx 2 Heart Lake**
  (same st/sp/sh/ac/ext). So JP idx N ≈ EN idx N+1 at the head of the list — same horse set.
- ホワイトノーザー appears in o2k#2 and oc#2 with identical (19,34,39,182) and identical composite
  `[1,234,0,48]`. CONFIRMED: composite is stable per-horse **within JP**.

### `ac` = dirt/course aptitude correlation — CONFIRMED (source was the Strategy file, not the JSON)
Reproduced from the 344 commented horses embedded in `DOC Breeding and Racing Strategy.txt`:
- dirt-only comments: **n=15, mean 184.5, range 85..255** (doc said 185)
- turf-only comments: **n=3, mean 88.7, range 35..163** (doc said 89)
- both-surface: **n=9, mean 156.2** (doc said 156)
Named examples confirmed against Breeder Studio JSON: El Condor Pasa 252, Jade Robbery(revC) 248,
Maple Syrup 240, Bubble Boy 252, City Commandant 0, Black Lily 35, Miami Beach 41. ✓
The correlation is real and reproducible. Meaning of `ac`=dirt is well-supported (the doc's 0.65 on
meaning is fair given it is still an inference, not a ROM-code proof).

### Inheritance + running-style algorithm — CONFIRMED as faithful transcription
breedFoal() in the Strategy file matches §6.1 exactly: st/sp/sh = floor(parent avg); ac = floor(avg +
(rand-0.5)*36); externals = clamp(avg + floor((rand-0.5)*4), 1, 16); bonuses Stamina (sire.st>=45 &&
dam.st>=40 → +2), Speed (sire.sp>=45 && dam.sp>=40 → +2), Dirt Dynasty (both ac>220 → +20); clamps
st10-60/sp10-65/sh10-60/ac0-255; dominant chosen but unused. deriveRunningStyle() matches §6.2 exactly
(range<=3→AL; greater==0→FR; ==1→SD; >=3→LS; else SR). Correctly flagged community heuristic, not ROM.

### Pierogi Prince placeholder — CONFIRMED
Rev D record #85 (first "dam", idx 85) = "Pierogi Prince", ac=0, ext=[1,1,15,15,15,1]. Exactly as the
doc describes. (It is a joke/placeholder dam at the sire/dam split point.)

---

## WRONG / OVERSTATED claims (corrections)

### 1. Record count: NOT 168 (Rev C) / 178 (Rev D). It is **167 / 177** in the ROM.
The doc says "84 sires + 84 dams = 168" (Rev C) and "89 + 89 = 178" (Rev D), and that this was
"confirmed by reading the +56 index of every record." The ROM index field contradicts this:
- Rev C: index runs 1..167 then record 168 (k=167) has idx field = 23216 and a garbage name
  ("Pete O Pete" with absurd stats). **167 records, not 168.**
- Rev D: index runs 1..177 then breaks (idx 24364, "Two Months Salary"). **177 records, not 178.**
- The JP versions are 167 each (which the doc got right).

The "168/178" and "89+89" came from `DOC_Breeder_Studio_data.json` (which lists revC=168, revD=178,
split exactly M=84/F=84 at id 84|85, and M=89/F=89 at id 89|90), NOT from the ROM index field. The JSON
likely carries one extra synthesized/duplicate entry per EN version. The doc conflated the JSON count
with a byte-verified count and mis-attributed it to the +56 field.

### 2. "Two physically separate 60-byte arrays" — WRONG. It is ONE contiguous array.
Rev C and Rev D store the whole pool as a single contiguous block from the sire base; the "dam base" is
just `base + 84*60`. There is no second array, no `idx 85..168` restart — the index increments
continuously 1..167 (Rev C) / 1..177 (Rev D). The sire/dam (M/F) distinction is metadata applied at
the 84|85 (Rev C) / 89|90 (Rev D) split, not a physical array boundary, and there is no per-record sex
byte in the 60-byte record (the JSON supplies sex; the ROM record does not appear to). The split point
is record #85 in BOTH Rev C and Rev D (the "Pierogi Prince" entry), which contradicts the doc's claim
that Rev D dams start at #90 — needs follow-up, but the contiguous-array correction stands.

### 3. "JSON id k == ROM record k; 84/84 sires and 84/84 dams round-trip exactly" — WRONG ordering.
The ROM table and the Breeder Studio JSON are the SAME SET of horses in DIFFERENT order:
- ROM Rev C rec84 = "Pentire", but JSON revC id84 = "Zephyr Hills". rec85 = "Ferranti's Folly", JSON
  id85 = "5th Avenue". Rev D ROM rec1 = "Maple Syrup", JSON revD id1 = "Banana Boy" (JSON is roughly
  alphabetical; ROM is game-table order).
- Matching by NAME instead of position: 161/167 ROM names also appear in the JSON; the 6 "misses" are
  spelling/space variants (Love J vs Love\xa0J, Hit Maker vs Hitmaker, K.L.. Hibiscus vs K.L. Hibiscus,
  Black¡¡Sapphire vs Black Sapphire, L.A. Machingun vs L. A. Machinegun, Cherry vs Cherry␠).
- Of the 161 name-matched horses, **156 have byte-identical st/sp/sh/ac**; 5 differ by one stat
  (Big Man sh31 vs 32, Deep Trouble ac183 vs 187, Antique Doll st20 vs 30, Prime Suspect ac220 vs 202,
  Prime Jewel field-shift) — minor JSON transcription noise.

So the field map and the table identity are right, but the doc's specific "round-trips at index k"
verification is not how it actually lines up. The data is reconcilable by NAME, not by INDEX.

### 4. EN↔JP composite is NOT byte-identical (the doc only claimed JP↔JP, which is true).
White Norther (JP comp `[1,234,0,48]`) maps by stats to EN Rev C "Judge Angelucci" whose composite is
`[1,204,171,52]`. The composite is stable JP-to-JP but differs EN-to-JP. The doc's §3 only asserts
JP↔JP stability, so this is a clarification, not a contradiction — but the §3 line equating Trot
Thunder's composite to Heart Lake's would be wrong if read that way (Heart Lake comp `[2,204,255,60]`
vs Trot Thunder `[2,206,58,50]`).

---

## UNVERIFIABLE / UNCHANGED (left as the doc has them)
- composite (name+44) byte-meanings b0/b1/b2/b3 — only byte positions certain; meaning still TBD. Doc is
  appropriately hedged. Not independently advanced here.
- ac vs personality (Interpretation B) — doc-derived from DOCWE Source-of-Truth; the dirt reading is the
  empirically supported one. Agree with doc's resolution.
- The actual ROM SH-4 breeding routine — not located; averaging model is a heuristic. Agree.

---

## NET
- Field map / offsets / strides / ranges / pad bytes: **SOLID, byte-verified, all 4 versions.**
- breedFoal/deriveRunningStyle transcription and ac↔dirt correlation: **SOLID, reproduced.**
- Record counts (167/177 not 168/178), "two separate arrays" (it's one), and JSON-index alignment
  (same set, different order, match by name): **need correction in breeding-system.md.**
