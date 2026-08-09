# Foal Two-Parent Average — FOUND (epr-22336c.ic22)

ROM base 0x0C000000; runtime = static + 0x20000; ROM-baked pointers are RUNTIME.
This file documents the routine that builds a foal/breeding-card from SIRE + DAM as the
**floor-average of the two parents' stats**, confirming the live observation
(foal speed 46 = floor((52+40)/2)).

## TL;DR / Verdict (confidence 0.92)

The two-parent average is **REAL and located**, but it is NOT in the breeding-screen
candidate generator (0x05E000–0x062500), exactly as the prior decodes (`candidate_gen.md`,
`foal_cross.md`) concluded. It lives in the **foal-build / card-creation routine at static
`0x0C052B0C`** (runtime `0x0C072B0C`), which reads the two SELECTED parent records and writes
a foal record whose internal and external stats are `floor((parentA + parentB) / 2)`, plus
pedigree-threshold bonuses, aptitude bit-blend, and aptitude-banded RNG noise.

This is in the breeding *selection/commit* subsystem (0x04C000–0x053000), the same region
that loads the sire/dam catalog and the selected-parent record cells — NOT the offering
minigame the earlier passes covered.

## Records (fixed RAM cells, all in the 0x0C21Axxx work area)
| cell (runtime) | role |
|---|---|
| `0x0C21A530` | **parent A (sire)** record — selected on breeding screen (written at 0x0412da/0x04f6a8/0x05059c/0x050e02) |
| `0x0C21A56C` | **parent B (dam)** record — selected on breeding screen (0x04ccbc/0x04db44/0x0505a8/0x050a98/0x050c82) |
| `0x0C21A5A8` | **foal / result** record being built |
| `0x0C21A564..0x0C21A569` | a 6-byte parallel parent stat row (grandparent / 2nd-line) used ONLY for the pedigree-threshold count |
| `0x0C21A124` | RNG state word (core rand at runtime 0x0C0B1E60) |
| `0x0C3C0FC0/FC4` and `0x0C3C0FCC/FD0/FD4` | scratch temps used by the helpers |

The foal-build at 0x0C052B0C is reached via the breeding state machine (no direct `bsr`
xref — dispatched through a pointer/jump table, normal for this engine). The parent cells are
populated by the sire/dam pickers in 0x04C–0x050.

## Record stat layout (confirmed this session)
- **+0x1C, +0x20, +0x24 (u32)** : the three INTERNAL stats (st / sp / sh) on the *parent*
  records. (read by helper 0x053414)
- **+0x40, +0x44, +0x48 (u32)** : the three INTERNAL stats (st / sp / sh) on the *foal*.
- **+0x34..+0x39 (6 bytes)** : the six EXTERNAL stats on the *parent* records.
- **+0x6C..+0x71 (6 bytes)** : the six EXTERNAL stats on the *foal*.
- **+0x74, +0x75, +0x76 (3 bytes)** : the three internal stats copied down to display bytes.
- **+0x5A (byte)** : SEX (set to `rand & 1`).
- **+0x30 (word @+48), +0x32 (word @+50)** : aptitude / running-style bitfields.

## The averaging — exact formulas + constants

### A. Three internal stats st/sp/sh  → helper `FUN @0x0C053414`  (confidence 0.95)
Caller (0x0C052B30–0x0C052B3E) loads the parent words and passes A+? in regs / B on stack:
```
R4 = sire[+0x1C]  ; R7 = dam[+0x1C]   (st)
R5 = sire[+0x20]  ; stack = dam[+0x20] (sp)
R6 = sire[+0x24]  ; stack = dam[+0x24] (sh)
```
Inside 0x053414 (idiom `add; mov #0,Rx; cmp/gt sum,Rx; addc Rx,sum; shar sum`):
```
st_tmp = (sire.st + dam.st) >> 1     ; signed round-to-zero, but stats>=0 so = floor avg
sp_tmp = (sire.sp + dam.sp) >> 1
sh_tmp = (sire.sh + dam.sh) >> 1
foreach v in {st_tmp, sp_tmp, sh_tmp}:
    if v > 45: v -= 5          ; soft cap pull-down (high-ceiling tax)
    if v < 10: v += 5          ; soft floor pull-up
foal[+0x40] = st_tmp ; foal[+0x44] = sp_tmp ; foal[+0x48] = sh_tmp
```
=> **internal stat = floor((sire + dam)/2)**, then ±5 soft clamps at the 45 ceiling / 10 floor.
This is the byte-verified source of the live "foal sp 46 = avg(52,40)" result
(floor((52+40)/2) = 46). The /2 here is the literal floor average of the two parents.

### B. Six external stats  → inline block 0x0C052B58–0x0C052BEC  (confidence 0.95)
For each external byte off ∈ {0x34,0x35,0x36,0x37,0x38,0x39} (parent) → dst ∈
{0x6C,0x6D,0x6E,0x6F,0x70,0x71} (foal):
```
a = (u8) sire[off]
b = (u8) dam[off]
foal[dst] = (a + b) >> 1        ; same signed-round idiom, unsigned bytes => plain floor avg
```
=> **external stat = floor((sire_ext + dam_ext)/2)**.  The pairing order in the code
(processed 0x34,0x36,0x35,0x37,0x38,0x39 — interleaved, but each is its own off→dst average).

### C. Sex pick  (confidence 0.85)
0x0C052B48: `R0 = rand_wrapper(0x0C09A1F8); foal[+0x5A] = R0 & 1`.  Foal sex = uniform 0/1,
independent of parents.

### D. Aptitude / running-style bitfields  → helpers 0x053154 & 0x05333E  (confidence 0.7)
Called first (0x052B24, 0x052B2C) with `sire[+0x30]`/`dam[+0x30]` and `sire[+0x32]`/`dam[+0x32]`
(16-bit aptitude masks). These are **NOT numeric averages**: they store the two parent masks
to scratch (0x3C0FC0/FC4) and do per-bit RNG-gated inheritance (0x05333E loops 8 bits,
`rand_int()&1` selects/shifts each bit; 0x053154 does threshold-gated mask clears on the
0xC000 / 0x8000 high bits). i.e. track/surface/distance aptitudes are inherited bit-by-bit
with coin-flips and gate thresholds, not averaged. (Decoded enough to classify; full per-bit
truth table not exhausted — hence 0.7.)

### E. Pedigree / bloodline threshold bonus  0x0C052BF0–0x0C052D20  (confidence 0.85)
After the external averages, a counter `R4` is accumulated over the 6 external bytes by
comparing the DAM row (`dam[0x34..0x39]`) AND the 2nd-line row (`0x0C21A564..0x569`) against
two thresholds: `>= 12` (R5) and `> 3` (R9):
```
cnt = 0
for off in the 6 externals:
    if dam[off]    >= 12: cnt++ ;  if line2[off] >= 12: cnt++
    if dam[off]    >  3 : cnt++ ;  if line2[off] >  3 : cnt++   (per-byte, see code)
```
Then tiered bonus added to the THREE INTERNAL stats (foal +0x40/+0x44/+0x48):
```
if cnt == 4 or cnt == 5:  foal.sp(+0x44) += 2 ; foal.st(+0x40) += 1 ; foal.sh(+0x48) += 2
if cnt == 6:              foal.sp(+0x44) += 2 ; foal.st(+0x40) += 3 ; foal.sh(+0x48) += 3
clamp each of +0x40/+0x44/+0x48 to max 45
copy foal.st/sp/sh (low byte) -> foal[+0x74]/[+0x75]/[+0x76]
```
=> small **bloodline bonus** (+1..+3 per stat) when enough parent/grandparent externals clear
the 12 / 3 thresholds; then internals hard-clamped to 45.

### F. Aptitude-banded RNG noise on internals  0x0C052D90–0x0C052E8E+  (confidence 0.7)
Keyed by a rating value R4 (an aggregate 0..~200). Within band [92,100): roll `R5=(rand>>1)&3`;
depending on R5∈{0,1,2,3} subtract 12 (or, for R5==3, subtract 5 from each) from the display
bytes 0x74/0x75/0x76 if they exceed 15; then if foal[+0x40/+0x44/+0x48] < 55 add 5. Further
bands at [180,188) etc. repeat the pattern. This is the random ± noise layer on top of the
clean average. (Banding structure decoded; exhaustive per-band magnitudes not fully tabulated.)

## Comparison to the community model
- Community: **internals ≈ avg ± up to 18; externals = avg ± ~2 clamped 1–16; bloodline
  threshold bonuses; sex pick**.
- ROM truth: internals = **exact floor-average** with ±5 soft clamps at 45/10 (§A), a small
  pedigree-threshold bonus (+1..+3, §E), hard cap 45, then **banded RNG noise** (§F) that can
  pull internals down by 5–12 in certain rating bands. Externals = **exact floor-average**
  (§B), with the ±2-style spread coming from the §E/§F adjustment layer rather than at the
  average step itself. Sex = `rand & 1` (§C). Bloodline thresholds use `>=12` and `>3` cuts on
  parent + 2nd-line external rows (§E). The community "±18" on internals is the *cumulative*
  effect of §E bonus + §F band noise, not a single uniform draw.

## Why prior passes missed it (reconciliation)
`foal_cross.md` correctly proved the offering minigame (0x05E–0x062) is single-source and that
the only `*0.5` float loaders (0x059c34, 0x08c224) and the breeding-region integer-halves are
NOT two-parent averages. It also predicted the real cross lives in the parent→template / card
path "outside 0x05E000–0x062500." That prediction is confirmed: the cross is at **0x052B0C**,
in the breeding *commit* code, and uses **integer `(a+b)>>1`** (not the float `*0.5`). The
integer-halving cluster at 0x052B64–0x053436 (8 sites) is exactly this routine + its helper —
it was outside both prior scans' decompiled windows. The two parent records (0x21A530/0x21A56C)
are distinct cells written by the sire/dam pickers, satisfying the "reads TWO distinct parent
records, writes a third" signature that 0x05E–0x062 lacked.

## Address index (also appended to ghidra/targets.txt)
- `0x0C052B0C`  foal-build / two-parent cross (MAIN; runtime 0x0C072B0C)
- `0x0C053414`  internal st/sp/sh averager `(a+b)>>1` + ±5 soft clamp → foal +0x40/44/48
- `0x0C053154`  aptitude bitfield blend helper (high-bit gate)
- `0x0C05333E`  aptitude bitfield blend helper (8-bit RNG per-bit inherit)
- `0x0C0615C6`  (NOT the cross) display-prep loop that copies records via pools
  0x0C171328/0x0C171528 — investigated per task 1; it is a per-candidate render/copy
  (calls graphics routine 0x0C073544), single-source, no two-parent blend.
- parent A 0x0C21A530 / parent B 0x0C21A56C / foal 0x0C21A5A8 / 2nd-line 0x0C21A564
- RNG state 0x0C21A124 (core 0x0C0B1E60) ; rand wrapper 0x0C09A1F8

## Confidence per claim
- Two-parent floor-average EXISTS and is at 0x0C052B0C: **0.92**
- Internal st/sp/sh = floor((sire+dam)/2) via 0x053414, ±5 soft clamp, cap 45: **0.95**
- 6 externals = floor((sire+dam)/2) (0x052B58–BEC): **0.95**
- Sex = rand&1: **0.85**
- Bloodline threshold bonus (>=12 / >3, +1..+3, §E): **0.85**
- Aptitude = per-bit RNG inherit (not average): **0.7**
- Banded RNG noise on internals (§F): **0.7**
- 0x0615C6 pool-writer is display copy, not the cross: **0.85**

---

## BYTE-EXACT FOLLOW-UP (decompiled fn_0c052b0c.c + helper disasm)

The §F "banded RNG noise" is now fully decompiled and is BYTE-EXACT (not modeled):

```
// after internals (+0x40/44/48) are floor-avg + soft-clamp + pedigree-bonus + cap45,
// they are copied to display bytes +0x74/+0x75/+0x76, then:
r = rand_byte()      // FUN 0x0C09A1F8, value & 0xFF  (0..255)  -- the noise gate
// BAND 1  [92,100)  (~3.1%):  sel = (RNGstate@0x21A124 >> 1) & 3
if (92 <= r < 100):
    sel==0 && disp[0x74]>15 -> disp[0x74] -= 12     // st
    sel==1 && disp[0x75]>15 -> disp[0x75] -= 12     // sp
    sel==2 && disp[0x76]>15 -> disp[0x76] -= 12     // sh
    sel==3 -> each of disp[0x74/75/76] -= 5 if >15  // all three
    then each internal +0x40/44/48 += 5 if < 55     // (hidden u32 bumped up)
// BAND 2  [180,188) (~3.1%):  sel = (RNGstate >> 3) & 3
elif (180 <= r < 188):
    sel==0 && disp[0x74]<40 -> disp[0x74] += 12
    sel==1 && disp[0x75]<40 -> disp[0x75] += 12
    sel==2 && disp[0x76]<40 -> disp[0x76] += 12
    sel==3 -> each disp[0x74/75/76] += 5 if <40
// NO other bands.
```
So noise is a uniform-random-byte gate hitting two narrow windows; the clean
floor-average is the result ~94% of the time, with rare ±12/±5 nudges. Band thresholds:
DAT_0c052e90=180 (0xB4), DAT_0c052e8e=188 (0xBC).

**Pedigree count (§E) — exact:** for each of the 6 externals, compare DAM[0x34+i] and the
2nd-line/grandparent row (0x0C21A567/568/569 ...): `cnt++ if (dam>=12 && line2>=12)`, and
`cnt++ if (dam<4 && line2<4)`. cnt==4|5 -> internals st+1/sp+2/sh+2 ; cnt==6 -> st+3/sp+2/sh+3 ;
cap 45. (Rewards dam/grandparent CONSISTENCY, high-high or low-low. Needs the grandparent row.)

**Derived external displays (end of routine):** foal[0x62/0x64/0x63/0x65/0x66/0x67] =
`((rand>>3)&3) + foal_ext[i]*2 ± 1` (first uses -1, rest +1); externals first masked &0x0F.

**Aptitude bit-inheritance (helpers 0x05333E + 0x053154) — exact LOGIC:** 0x05333E loops 8 bits,
for each calls the RNG core, `rand&1` decides shift, builds the inherited 16-bit mask from sire
(loop 1, high bits via &0x8000 then >>2) then dam (loop 2, &0x4000 then >>1); 0x053154 gates the
top 0xC000 bits (clears via &0xBFFF / &0x7FFF) when style selectors >=5 / >2. The +0x30/+0x32
masks are course/surface/distance aptitudes inherited bit-by-bit with coin-flips, NOT averaged.
(Bridge from catalog ac byte 0-255 to these 16-bit masks is unresolved -> predictor keeps a
per-bit ac blend as a faithful proxy of "random mix of the two parents.")

## PREDICTOR (breeding-lab.html) — noise now byte-exact
`breedFoal()` implements the two bands exactly (rand&0xFF gate, [92,100) -12/-5, [180,188) +12/+5,
selector picks the stat). Validated: a 46-avg→41 foal stays 41 ~98.5% with -12/-5 at ~0.8% each,
matching 3.1%×selector-prob. Pedigree bonus stays approximated (no grandparent row in the catalog);
ac stays a per-bit blend (no ac<->mask bridge). Everything else (averages, soft-clamp, cap45, sex,
noise bands) is byte-exact.

---

## APTITUDE GRADE DECODER — FUN_0c0534a4 (byte-exact)
Decodes a 16-bit aptitude/style mask -> a grade INDEX, via two ROM tables (one contiguous
32-byte block; tableA=0x10BE78 idx0-15, tableB=0x10BE88 idx16-31):
```
T = [1,3,3,3,2,6,6,5,2,6,6,5,2,4,4,4, 13,15,15,7,12,10,10,11,13,10,10,11,9,14,14,8]
grade(mask):
  if (mask & 0xC000) == 0:  return T[16 + ((mask>>4)&0xF)]   // no gate -> tableB via the 0x00F0 nibble
  if (mask & 0x3000) == 0:  return T[(mask>>8)&0xF]          // gated  -> tableA via the 0x0F00 nibble
  return 0                                                   // both top tiers set -> 0
```
Called from 23 status/display sites (0x04B-0x04F catalog/status subsystem). The returned value
(1/2/3/4/5/6/7/8/9/10/11/12/13/14/15) is the game's internal grade index; **higher = stronger**.
The index->on-screen LETTER GLYPH is one further table (the aptitude symbol sprites), not yet
dumped -- it is purely cosmetic (the grade index already orders aptitude correctly).

Wired into breeding-lab.html as `gradeFromMask()`; the predictor shows the foal's inherited
aptitude grade index next to the sire/dam grades (byte-exact comparison). Tables verified:
0x10BE78 sits right after the breeder-name data (Nathan Runner / Michigan Blue / Golden Planet).
