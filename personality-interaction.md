# Personality ↔ Post-Race Interaction table (RE pass, Jun 2026)

Scope: the table that scales the **post-race interaction** effect by the horse's **personality**.
Drives how much "bonding" each response gives (which card value the bond lands on, hearts, trust or
condition, is not established). ROM: `epr-22336c.ic22` (Rev C), file offsets.

**Read this first (corrected 2026-09-28):**
- DOC has **five** personalities: **Imposing, Honest, Rough, Coward, Sloppy**. The game's
  personality classifier has only five outputs. "Too soft" and "Strict" are NOT personalities: they
  are menu answer labels stored right after the five names.
- **Which reply button is which column of the table is UNVERIFIED** (confidence 0.4, never traced).
  The reply menu has 14 verbs (Psyche up, Praise, Flatter, Hug, Ignore, Leave, Blandish, Scold,
  Sooth, Apologize, Astonish, Comfort, Wake up, Sooth) against 5 columns.
- The game's own post-race lines say the race matters too ("Because of the way %s finished, your
  praise is falling on deaf ears."; "Your horse may be scolded for a bad race.").
- So **do not give per-button advice** (for example "use Comfort on a Rough horse") and do not quote
  a multiplier for a named button or a named personality from this file.

## Earlier 7x6 reading (retracted)
An earlier table here (a 7x6 read starting at 0x0E7CF0, with rows named after seven "Check" labels
including "Too soft" and "Strict", plus a "row coherence" defence) was retracted on 2026-06-06 and
removed on 2026-09-28: it absorbed 12 unrelated floats and treated two menu labels as personalities.
The current decode is below.

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
## What the 6x5 table means, and what is not known
**Mechanic (byte-exact):** `bond_gain = M*(100 - bond)`: M is how strongly a (personality-state,
response) pair pulls the bond toward 100; **negative M lowers it**. Rows = 3 personality tiers × a
runtime flag (*(R5+0x44), unidentified), so the same reply can help in one flag state and hurt in the
other; cols = 5 responses. (Corrected 2026-09-28: the tier grouping here used to list seven "Check"
labels. The inferred grouping, 0.7, puts Imposing in tier 0 and Honest, Rough, Coward, Sloppy in tier
1; it also put "Too soft" and "Strict" in tier 2, but those are not personalities, so what tier 2
holds is unknown.)
**Still open:** what writes the personality value *(R5+0x1C) and the flag *(R5+0x44), and which reply
button feeds which column (col index *(R5+0x14)). **Column-to-button mapping: UNVERIFIED (0.4). Give
no per-button advice.** conf: table+formula 0.9; tier grouping 0.7; response names 0.4.

## What is NOT byte-proven (the open items)
- **Column names.** The reader's literal pool does hold the table base (see the correction above), so
  the table itself is located. What is NOT traced is which reply button feeds which column: the
  **column-to-button mapping is INFERRED at 0.4**, and the menu's 14 verbs cannot be matched one to
  one with 5 columns. (Corrected 2026-09-28: this bullet used to say no pointer to the table exists
  and that a 7-row personality axis was solid.)
- **Personality → row.** The card personality byte (0-255) is sorted by the game into **five
  classes** (Imposing, Honest, Rough, Coward, Sloppy) by its high nibble (see
  card-seed-trait-readers.md). The 8-band names Calm/Firm/Sensitive/Moody/Gentle/Proud come from an
  outside database and do not occur in the ROM. How the five classes map to the 3 row tiers is not
  pinned (0.7). (Corrected 2026-09-28.)
- **Target.** The formula writes *(R5+0x68) of a work struct that is not identified. Earlier notes
  guessed it lands on the card's trust (`a2[36]`) or condition (`a2[44]`) field and that condition
  feeds the race formula: both UNVERIFIED (card-seed-trait-readers.md found no gameplay reader for the
  condition byte). Whether the bond is the hearts meter is not established, and the exact delta per
  reply is not isolated. (Corrected 2026-09-28.)

## To close it (needs a trace or in-game observation)
1. Disasm the reader: done 2026-06-06 (reader at `0x0C027F80`, see the correction). Still to trace:
   the writers of the column index *(R5+0x14), the personality value *(R5+0x1C) and the flag
   *(R5+0x44).
2. In-game: on one horse of known personality, repeat the same post-race situation, pick each
   response and read the hearts before and after (note the race result too) → confirms the
   column→action mapping + the base step (multiplier → actual points).

## Provenance
Table values + boundaries: direct ROM read. Personality classes: the ROM classifier in
card-seed-trait-readers.md (five classes); the 8-band list in derived-attrs.md is not ROM text.
Interaction menu: game-text.md block `Interaction Menu & Result Text` @0x0E83D0 (393 strings).
Confidence (6x5 read): values and formula 0.9, tier grouping 0.7, exact column names 0.4,
byte→row map 0.5. (Corrected 2026-09-28: this line used to rate the retracted 7x6 read.)
