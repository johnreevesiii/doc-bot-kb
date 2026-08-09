# Personality ↔ Post-Race Interaction table (RE pass, Jun 2026)

Scope: the table that scales the **post-race interaction** effect by the horse's **personality**.
Drives how much "bonding" (trust/condition) each response gives. ROM: `epr-22336c.ic22` (Rev C),
file offsets.

## The table — DECODED (boundaries + values byte-exact; structure high-confidence)
- **Location:** `0x0E7CF0 .. 0x0E7D94` = **42 IEEE-754 floats** (the older note "38 @0x0E7D00" missed
  the first row). Bounded: `0x0E7CE0..EC` = 0.3/0/0/0 (other data) before; all-zero after `0x0E7D94`.
- **Shape: 7 rows × 6 columns** (42 = 7×6). **Rows = the 7 in-game "Check" personalities**, in the
  ROM string order at `0x0E84A4`: **Imposing, Honest, Rough, Coward, Sloppy, Too soft, Strict**
  (7 labels ⇒ 7 rows; high confidence). **Columns = 6 post-race responses** (the emotional actions
  from the interaction menu at `0x0E83D0`: Praise / Flatter / Hug / Comfort / Sooth / Scold-class).
- **Values = effect multipliers** (×2.0 great … 0 nothing … −2.0 backfire), matching the prior note
  "Hug/Praise/Scold/Flatter scaled ×2.0..−2.0 by personality".

### The 7×6 matrix (row = personality, col = response 1..6)
```
 Imposing   1.0  1.0  1.0  1.0  0.8  1.0
 Honest     0.8  0.1  0.1  0.0  0.2  1.0
 Rough      1.2  1.5  0.8  2.0  1.0  1.5
 Coward     1.5  1.2  2.0  1.2  1.0  2.0
 Sloppy     0.5  2.0  0.5 -1.5 -1.0 -0.5
 Too soft   2.0  2.0  2.0  2.0  1.0  2.0
 Strict     1.0 -2.0 -1.2 -1.0 -0.5  2.0
```
**Row coherence = strong domain validation** (why we trust the 7×6 read):
- **Too soft** → everything is positive/high (a soft horse loves any attention).
- **Strict** → coddling responses are negative (−2.0/−1.2/−1.0/−0.5); only col0 and **col6 (+2.0)**
  help ⇒ col6 is the firm/"Scold"-class response (discipline works on a strict horse).
- **Honest** → near-zero across the board (indifferent to interaction).
- **Sloppy** → cols 4–6 backfire (−1.5/−1.0/−0.5).

## CORRECTION (Jun 6 2026) — reader disassembled; the table is 6×5, not 7×6
Found the reader at **`0x0C027F80`** (sh4dis); literal pool at file 0x28048+ holds the table base
ptr **`0xC107D20` = file 0x0E7D20** and divisor **100.0**. Verified index math:
```
effect_multiplier M = table[ base(0x0E7D20) + row*20 + col*4 ]   ; row stride = 5 floats
bond_gain = M * (100 - currentBond)                              ; FR4=100 divisor; stored *(R5+0x68)
row = f(personality *(R5+0x1C)): tier = (p>=1) + (p>=5)  -> 0/1/2, then *2 + flag(*(R5+0x44)) -> 0..5
col = response index *(R5+0x14)  (0..4 = 5 post-race responses)
```
So the **real table is 6 rows × 5 cols = 30 floats @0x0E7D20** (the earlier "7×6 @0x0E7CF0" wrongly
absorbed 12 unrelated preamble floats at 0x0E7CF0). Byte-exact values:
```
 row0   1.2  1.5  0.8  2.0  1.0
 row1   1.5  1.5  1.2  2.0  1.2
 row2   1.0  2.0  0.5  2.0  0.5
 row3  -1.5 -1.0 -0.5  2.0  2.0
 row4   2.0  2.0  1.0  2.0  1.0
 row5  -2.0 -1.2 -1.0 -0.5  2.0
```
**Mechanic (byte-exact):** `bond_gain = M*(100 - bond)` — M is how strongly a (personality-state,
response) pair pulls the bond toward 100; **negative M lowers it**. Rows = 3 personality tiers
(the 7 Check labels collapse: tier0={Imposing}, tier1={Honest,Rough,Coward,Sloppy}, tier2={Too soft,
Strict}) × a runtime flag (*(R5+0x44)); cols = 5 responses. **Still open:** what writes the
personality value *(R5+0x1C) (card-byte → tier classifier) and the flag *(R5+0x44); the exact 5
response identities/order (col index source *(R5+0x14)). conf: table+formula 0.9; tier grouping 0.7;
response names 0.4.

## What is NOT byte-proven (the open items)
- **No direct pointer** to the table exists in the ROM (searched runtime `0x0C107CF0`/`0x0C107D00`
  and file-base variants — zero hits). It's reached by computed/PC-relative (`mova`) addressing, the
  same wall as the FPU/race tables. So the **exact column→response-name mapping and orientation are
  INFERRED** (0.4), not traced. The *values* and the *personality-row axis* are solid (0.85/0.9).
- **byte 6 → which of the 7 rows**: card personality is a 0–255 byte (8-band model: Rough/Imposing/
  Calm/Firm/Sensitive/Moody/Gentle/Proud) but the table rows are the 7 "Check" labels. The exact
  byte→row index is the game's runtime classification (not pinned here); the advisor maps via the
  Card-Creator 5-bucket → nearest Check label (approximate).
- **trust vs condition target**: the multiplier scales an interaction effect that lands on the card's
  trust (`a2[36]`) / condition (`a2[44]`) fields (low-confidence labels, OPEN_QUESTIONS #7); condition
  feeds the race-formula condition term. The exact delta = multiplier × (base interaction step) is not
  isolated.

## To close it (needs a trace or in-game observation)
1. Disasm the reader: find the `mova` that loads `0x0C107CF0`-ish (in the interaction/result subsystem
   near the `0x0E83D0` menu strings) → get the index math (personality row, response col order).
2. In-game: on one horse of known personality, pick each response and watch the trust/condition bar →
   confirms the column→action mapping + the base step (multiplier → actual points).

## Provenance
Table values + boundaries: direct ROM read (this pass). Personality labels/bands: derived-attrs.md
§3 (byte 6, 8-band + 5-bucket + 7 Check labels, string-verified). Interaction menu: game-text.md
block `Interaction Menu & Result Text` @0x0E83D0 (393 strings). Confidence: values 1.0, 7-row
personality axis 0.9, 6-col responses 0.7, exact column names 0.4, byte→row map 0.5.
