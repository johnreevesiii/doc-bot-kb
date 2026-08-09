# nvram.md — Adversarial Verification Report

Verifier pass over `C:/DerbyOwnersClub/_core/areas/nvram.md`. Every claim below was
re-extracted independently from the real binaries with `python3` byte-slicing
(`C:/Users/johnr/AppData/Local/Temp/nv*.py`). Files used:
`C:/Users/johnr/Downloads/demul07_280418/nvram/derbyocw.{sram,eeprom}` and `derbyocwm.{sram,eeprom}`.

## Overall verdict: MOSTLY SOLID

The core decode (money leaderboard, track-record table, EEPROM identity, master/satellite
diff) is correct and reproduced exactly. Three numeric/offset claims are WRONG and must be
corrected before any editor/dashboard tool is built on the doc.

---

## CONFIRMED claims (re-extracted from bytes)

| # | Claim | Result |
|---|-------|--------|
| 1 | File sizes: eeprom 128, sram 32768 (both boards) | CONFIRMED |
| 2 | Both EEPROMs byte-identical | CONFIRMED (`ee==eem` True) |
| 3 | SRAM last nonzero ~0x2998; nonzero count 4082 (cw) | CONFIRMED: cw last nonzero idx+1 = 10645 (0x2995), nonzero=4082; cwm 10633/4066 |
| 4 | Header LE32 counters cw +0x00=3017 +0x04=3017 +0x08=28138; cwm 3496/3487/13237 | CONFIRMED exactly |
| 5 | Setting flags cw [0x10,0x20,0x30,0x40,0x44,0x4c]=1,1,3,3,1,1; cwm all 0 | CONFIRMED |
| 6 | Region checksum @0x1f8 cw=0x3536, cwm=0x259a; doubled at 0x208 | CONFIRMED (bytes 36 35 -> LE16 0x3536; 0x1f8 block == 0x208 block) |
| 7 | 0x218 = LE32 0x429 (1065) then ASCII "Big Shuttle" | CONFIRMED |
| 8 | Money LB @0x230, 50 x 32B, strictly descending 417,935,500 -> 4,628,541 | CONFIRMED (monotonic True; first 417935500, rec50 4628541) |
| 9 | Record fields: +0x00 flag0, +0x01 flag1, +0x02 LE32 money, +0x0c 20B name | CONFIRMED |
| 10 | flag0 small set {0,1,2,3,7} | CONFIRMED (dist 0:10,1:21,2:1,3:16,7:2) |
| 11 | flag1 range 0xc0-0xde | CONFIRMED (values {0xc0,0xc1,0xc7,0xca,0xcc,0xcf,0xde}) |
| 12 | Top-of-board example rows (ranks 1-7, rank 50) f0/f1/money/name | CONFIRMED byte-for-byte |
| 13 | Names cross-check to DB: City Commandant #222 G1, Trash Talker #220 G3 | CONFIRMED in DRBYOCW.md |
| 14 | Copy-2 LB @0x870, same names, +0x06..+0x0b populated | CONFIRMED |
| 15 | Track-record table @0xf7c, 57 x 28B, ends exactly 0x15b8 | CONFIRMED (0xf7c+57*28 = 0x15b8) |
| 16 | Track fields: +0x00 20B name, +0x14 **LE16** time (1/40-s; cs=raw×2.5), +0x16 2B field, +0x18 tail | byte layout CONFIRMED; time UNIT corrected 2026-06-08 (was mislabeled "LE32 cs") |
| 17 | Track sample RAW times (1/40-s; ×2.5 → cs): race0 3876→1.36.90, race1 3384→1.24.60, race25 7956→3.18.90; holder "Hitmaker" | raw bytes CONFIRMED; seconds corrected 2026-06-08 (old 38.76/33.84/79.56 mis-scaled ×1; see TRACK_RECORD_TIME_PRECISION.md) |
| 18 | Hitmaker = horse #123 "Custom Design / Hitmaker is Sega" | CONFIRMED in DB |
| 19 | Master/satellite diff: only 32 bytes differ; leaderboard + track payload byte-identical | CONFIRMED (32 diffs; LB 0x230-0xe90 and track 0xf7c-0x15b8 both identical) |
| 20 | Diff offsets (header, 0x100 block, 0x1f8/0x208, 0x15c4, 0x2988-0x2994) | CONFIRMED (matches my full diff grouping) |
| 21 | EEPROM @0x30 "DERBY OWNERS CLUB AM3", BEF0 tag at 0x03 and 0x15, game CRC 600410a3 @0x2c and @0x54 | CONFIRMED |
| 22 | EEPROM two mirrored halves | CONFIRMED (sys block 0x00-0x11 == 0x12-0x23; game 0x2c-0x4f == 0x54-0x77; CRC 0x12ef both) |
| 23 | Region-2 track table @0x2340, trailer @0x2988 | CONFIRMED (predicted from +0x13c4 delta; "Hitmaker"/3876 present; trailer bytes identical to region1) |
| 24 | Trailer @0x15c4 holds LE16 0x0a25 (2597) + byte 0x09 | CONFIRMED |

---

## WRONG / OFF claims (corrections required)

### A. Region-2 money-LB start offset is WRONG (doc 0x1634 -> actual 0x15f4)
Doc section 2 / 3 says "MONEY LEADERBOARD ... 0x1634 (region 2)". The actual region-2
copy-1 record-0 ("City Commandant", money 417,935,500) starts at **0x15f4** (name ASCII
at 0x1600 = 0x15f4 + 0xc). The doc itself correctly cites the *name* at 0x1600 in section 2,
so it is internally contradictory. Verified by searching the LE32 money key 417935500:
occurs at rec starts 0x230, 0x870, **0x15f4**, 0x1c34.
- Region-2 copy-1 LB: **0x15f4**
- Region-2 copy-2 LB: **0x1c34**

### B. Region delta is +0x13c4, NOT +0x13d0
Doc section 2 says region 2 = "same structure, delta +0x13d0". Measured delta (region2 copy1
minus region1 copy1) = 0x15f4 - 0x230 = **0x13c4** (5060). This is the same value stored as the
LE32 length word at **0x1fc** (= 5060 = 0x13c4) — so the doc's own "length/record-count word"
at 0x1fc is actually the region stride, and the doc's stated delta contradicts it by 0x0c.
(Region-2 track @0x2340 and trailer @0x2988 happen to be correct because they equal
region1+0x13c4; only the money-LB start and the stated delta are wrong.)

### C. Copy-2 metadata constant is 0x1680 (5760), NOT 0x168000 (1,474,560)
Doc section 3 "Copy 2" reads bytes +0x09=0x80 +0x0a=0x16 +0x0b=0x00 and calls it
"0x168000 = 1,474,560". Little-endian, those three bytes = 0x00<<16 | 0x16<<8 | 0x80 =
**0x001680 = 5760**. The doc reversed the byte order. The raw bytes ("80 16 00") and the
+0x08 sub-byte set {0,1,2} are correct; only the decoded integer is wrong. (Whatever 0x1680
means — season/date stamp candidate — the magnitude is 5760, not 1.47M.)

### D. "Header A (0x00) is mirrored verbatim at 0x100" is WRONG
Doc section 2a says the 0x00 header is "mirrored verbatim at 0x100". It is NOT: `cw[0:12] != cw[0x100:0x10c]`.
0x100 is a related but shifted/different structure (it begins with the +0x08 counter value 0x6dea,
i.e. the two leading LE32 counters from 0x00 are absent and the layout is staggered). 14 byte
positions differ between 0x00 and 0x100 in the first 0x50. The 0x100 block IS part of the
per-board bookkeeping (it appears in the master/satellite diff list), but it is a second distinct
header, not a verbatim copy of header A.

---

## MINOR / NUANCE

- EEPROM mirror boundary: doc says "0x00-0x4f duplicated at 0x50-0x9f". More precisely it is two
  JVS sub-blocks each mirrored separately: system block 0x00-0x11 -> 0x12-0x23, and game block
  0x2c-0x4f -> 0x54-0x77. The "one 0x50 block doubled at 0x50" framing is approximately right but
  not exact (the 0x24-0x2b region `2af128282af12828` sits between and is itself a 4-byte pattern
  doubled). Practical impact: low.
- 0x1fc "length or record-count word" (doc): confirmed = 5060 = 0x13c4 = the region stride
  (see correction B). It is the region payload length / inter-region delta, not a 50-record count.
- flag0/flag1 still UNDECODED (doc honestly flags this). My distributions give a head start:
  flag1 only ever takes {0xc0,0xc1,0xc7,0xca,0xcc,0xcf,0xde}. Spot check vs DB: City Commandant
  (G1, coat Special) f1=0xcc; Trash Talker (G3, Black) f1=0xcc — so f1 is NOT grade and NOT coat
  alone (doc's "UNCONFIRMED" is correct). Big Shuttle (rank50) f1=0xc1, f0=1.

## JP-version note
The doc's open-Q5 ("confirm layout on derbyo2k / derbyoc — no JP nvram in this set") is accurate:
this Demul set is the WE (derbyocw) cabinet only. No derbyo2k/derbyoc nvram exists here, so the
JP SRAM stride (28B vs 32B records / EUC-JP names) remains unverified. Correct to leave as open.

## Net
Decode methodology is sound and the high-value tables reproduce exactly. Fix the 4 errors
(region-2 LB offset 0x15f4, delta 0x13c4, copy-2 value 0x1680, drop "verbatim mirror at 0x100")
before building the NVRAM editor — an editor that writes region 2 at 0x1634 would corrupt
the backup copy.
