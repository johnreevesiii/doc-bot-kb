# HIDDEN-A (+0x01) / HIDDEN-B (+0x10) — Consumer & Role Decode (SH-4 static RE)

ROM: `epr-22336c.ic22` (drbyocwc / WE Rev C). SH-4 base 0x0C000000. RUNTIME = static + 0x20000.
Method: full static disasm via `sh4dis.py` + the existing Ghidra C decomp (`fn_0c044ab4.c`, `fn_0c07c164.c`),
cross-checked against the live gate capture `trace/horse_structs.bin` (12 structs, stride 0x2A0).

## TL;DR verdict (race-sim relevance)

| field | roster off | copied into race struct? | survives to race time? | read by race formula? | ROLE | race-sim needed? |
|---|---|---|---|---|---|---|
| **HIDDEN-A** | +0x01 | yes, as high byte of a 16-bit word → **struct +0x34** | **NO — clobbered** | no | not a runtime input on this path | **NO** (conf 0.8) |
| **HIDDEN-B** | +0x10 | yes, as a raw byte → **struct +0x44** | **NO — clobbered** by phase-flag init (=1) | the formula reads struct+0x44, but it = phase flag 1, not HIDDEN-B | not a runtime race input on this path | **NO** (conf 0.7) |

Neither HIDDEN-A nor HIDDEN-B reaches `FUN_0c044ab4` (the speed formula) as its roster value.
Both are written into the per-horse race struct at race-init, then overwritten by later init passes
before the first speed tick. **Neither is required for a byte-exact single-race simulation.**

No indexed table (`base + field*rowsize` / switch) keyed on HIDDEN-A or HIDDEN-B was found anywhere.

---

## 1. The working roster lives in RAM at 0x0C128C78 (stride 0x20), not at the ROM table

- The ROM roster (analyst static `0x108E03`) is **never loaded as a literal** anywhere (verified: no pcrel
  literal and no stored 4-byte pointer to 0x0C128E03 / 0x0C108E03 / any ±0x40 neighbour exists in the ROM).
- At boot, `FUN_0c0413e0` builds a **working roster in RAM at `0x0C128C78`** (stride 0x20, 244 records,
  same field layout) from the cart-mapped ROM roster (source ptr literal `0x0DF9F85E`, dest `0x0C128C78`,
  also a parallel table at `0x0C12AC78`). NOTE: file offset 0x128c78 in the static image is unrelated ASCII
  text ("king like Jell-O…"); `0x0C128C78` is a **RAM work address**, confirming the roster is relocated.
- All runtime roster reads index `0x0C128C78 + id*0x20` (the `shll2;shll2;shll` = `<<5` idiom is explicit
  at e.g. 0x0C0580EA and 0x0C058FC0). The +8=grade / +9=start / +10=corner reads there match the
  `horse-stats.md` layout 1:1, proving 0x0C128C78 is the roster.

### Complete set of roster byte-offsets read by ANY runtime consumer (union over all 7 consumers of
0x0C128C78 and 3 of 0x0C12AC78):

```
{0x02, 0x04, 0x08, 0x09, 0x0A, 0x0C, 0x0D, 0x0E, 0x0F, 0x10, 0x12, 0x13, 0x14,
 0x15, 0x16, 0x17, 0x18, 0x19, 0x1B}
```
- **+0x10 (HIDDEN-B) IS in this set.**  **+0x01 (HIDDEN-A) is NOT.** (conf 1.0 for the set itself)
- (+0x05 dirt is consumed implicitly: a `mov.l @(4,R13)` dword grab pulls bytes +4..+7. The +0 dword is
  never grabbed; a `mov.w @R13` word grab DOES pull +0..+1 — see §3 — but its destination is clobbered.)

Consumer sites (all load 0x0C128C78):
`0xC0413FC` (boot build/sort), `0xC0580E0` + `0xC0581DC` + `0xC0582E0` (race-init, int path),
`0xC058FB4` + `0xC0591AA` (race-init, float path), `0xC08C948` (HIDDEN-X +0x17/+0x18/+0x19 reader).

---

## 2. HIDDEN-B (+0x10): copied to race struct +0x44, then overwritten — NOT a live race input

### 2a. Where it is read & stored (race-init struct populator)
Two sibling race-init functions expand each roster record into the 0x2A0 per-horse race struct (R14 = struct
base; the `=0x02A0` stride literals confirm the struct). Both read **roster +0x10 exactly once** and store it
as a **single byte to struct +0x44**:

- int path:  `0x0C05815C: mov #16,R0 ; 0x0C05815E: mov.b @(R0,R13),R3 ; 0x0C058162: mov.b R3,@(R0=0x44,R14)`
- float path: `0x0C059034: mov.b @(R0=0x10,R13),R3 ; 0x0C059038: mov.b R3,@(R0=0x44,R14)`

Full roster→struct map (float path 0xC058FB4), verified:
```
roster +0x08 -> struct +0xBC (as float)     roster +0x10 (HIDDEN-B) -> struct +0x44 (byte)
roster +0x09 -> struct +0xC0 (as float)     roster +0x12..+0x16    -> struct +0x46..+0x4A
roster +0x0A -> struct +0xC4 (as float)     roster +0x17..+0x19    -> struct +0x4B..+0x4E
roster +0x02 -> struct +0x3C (word, id)     roster +0x14..+0x19    -> struct +0x11A..+0x11F (2nd copy)
```
HIDDEN-B's ONLY destination is **struct +0x44**.

### 2b. The race formula DOES read struct +0x44 …
`FUN_0c044ab4` (the speed formula, unaff_r13 = 0x2A0 struct base; it reads +0x28/+0x6C/+0x74 header/phase
fields too) reads `*(unaff_r13 + 0x44)` 4× (decomp lines 413, 418, 755, 760) and feeds it into a
fused multiply-accumulate in the speed term. So +0x44 IS load-bearing for speed.

### 2c. … but +0x44 no longer holds HIDDEN-B by race time
Live gate capture `trace/horse_structs.bin`, struct+0x44 raw bytes for all 12 horses = `01 00 00 00` (=int 1).
HIDDEN-B's real distribution is {0:200, 1:37, 2:7}; an all-12 = 1 is statistically impossible if it were
HIDDEN-B. This matches `horse-stats.md`/struct-map's "+0x44 = phase active-flag/count = 1". So a later phase-
record init overwrites +0x44 with the constant flag 1 **after** the byte copy and **before** the formula runs.
=> The formula reads the phase flag, NOT HIDDEN-B.  (conf 0.7; residual risk: the overwrite could in principle
be conditional, but the capture shows 1 for every horse at the gate, i.e. the state the formula actually sees.)

---

## 3. HIDDEN-A (+0x01): rides the +0x00 word into struct +0x34, then fully clobbered — DEAD on race path

- HIDDEN-A is never read as a standalone value by any roster consumer (it is absent from the §1 union).
- It is captured only incidentally: `0x0C05811C / 0x0C058FF2: mov.w @R13,R3` reads the little-endian word at
  roster +0x00..+0x01 ( = pad(0) | HIDDEN-A<<8 ) and stores it to **struct +0x34** (`mov #52,R0; mov.w R3,@(R0,R14)`).
- That landing slot is then fully overwritten: the style-coefficient init `FUN_0c07c164` does
  `*(undefined4 *)(param_2 + 0x34) = uRam0c07c25c` (decomp L37) writing the 4 bytes `00 00 80 3F` = **float 1.0**.
- Live gate capture confirms: struct+0x34 = `00 00 80 3F` (1.0) for all 12 horses. HIDDEN-A's high byte at
  +0x35 is destroyed.
- `FUN_0c044ab4` does not read +0x34 at all (0 occurrences). The twin/0x0A race functions that touch +0x34
  read the clobbered 1.0, not HIDDEN-A.
=> HIDDEN-A has **no runtime consumer** on the race path; it is overwritten before use.  (conf 0.8)

---

## 4. No growth/aptitude table indexing for either field

- Searched the whole ROM for the table-index idiom (`mov #1,R0`/`mov #16,R0` → `mov.b @(R0,Rm)`) and the
  `<<5` stride pattern; none of the roster-based readers use HIDDEN-A or HIDDEN-B as a `base + field*rowsize`
  index or a switch selector. The only `+0x01`-with-mask reader found (`FUN_0c0022xx`, `and #1` then `>>1 & 7`)
  operates on the **player/card-horse record** (source ptr `*(0x0C21C7EC)`, different field layout: +5/+6 are
  `&63` stats), NOT the CPU roster — irrelevant to CPU HIDDEN-A.
- Because the CPU roster is the table of CPU OPPONENTS (no breeding/aging/growth for CPU horses, per
  `horse-stats.md` §intro), a "career stat-growth-type" consumer would not be expected here, and none exists.

---

## 5. Consumer addresses (for the record)

- Race-init struct populators (read roster +0x10 → struct +0x44): **0x0C0580E0 family** and **0x0C058FB4 family**
  (specifically the byte copy at 0x0C05815E and 0x0C059034).
- Word-grab that incidentally carries HIDDEN-A → struct +0x34: **0x0C05811C** and **0x0C058FF2**.
- Speed formula reading struct +0x44: **FUN_0c044ab4** (0x0C044AB4) at internal refs +0x44 (decomp L413/418/755/760).
- Slot clobbers: +0x34 by **FUN_0c07c164** (style-coef, 0x0C07C164 L37); +0x44 by the phase-flag init (sets =1;
  evidenced by the all-horses=1 gate capture; exact writer not isolated but its effect is byte-verified in trace).

## 6. Race-sim-exactness verdict

- **HIDDEN-A (+0x01): NOT needed** for byte-exact race sim (no consumer; clobbered). conf 0.8.
- **HIDDEN-B (+0x10): NOT needed** for byte-exact race sim on the evidence here — it is copied to struct+0x44
  but overwritten by the phase flag (=1) before the formula reads it; the formula reads the flag, not HIDDEN-B.
  conf 0.7. (To push to ~0.95, isolate the exact +0x44 phase-flag writer and confirm it is unconditional, and
  spot-check a gate capture of horses known to have HIDDEN-B=2 to confirm +0x44 still reads 1.)

Both fields remain genuine per-horse roster attributes (HIDDEN-B was independently rebalanced in DOC-2000),
but their effect, if any, is on something other than a single race's speed math on the paths reachable from
the roster — most plausibly an out-of-race / encyclopedia / odds-or-AI-selection use that does not feed
`FUN_0c044ab4`. They are out of scope for race-sim byte-exactness.
