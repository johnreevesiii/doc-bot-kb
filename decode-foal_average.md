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
a sire-vs-dam pedigree bonus, two bit-mixed 16-bit words (§D), and a random-byte noise gate that moves
the PRINTED birth stats and, on Band 1, lifts every true internal by 5 (see BYTE-EXACT FOLLOW-UP and
CONSUMER WARNING). (Corrected 2026-09-28: said "aptitude-banded RNG noise"; the gate is a uniform
random byte, and the pedigree row is the sire's own externals, not a grandparent's.)

This is in the breeding *selection/commit* subsystem (0x04C000–0x053000), the same region
that loads the sire/dam catalog and the selected-parent record cells — NOT the offering
minigame the earlier passes covered.

## Records (fixed RAM cells, all in the 0x0C21Axxx work area)
| cell (runtime) | role |
|---|---|
| `0x0C21A530` | **parent A (sire)** record — selected on breeding screen (written at 0x0412da/0x04f6a8/0x05059c/0x050e02) |
| `0x0C21A56C` | **parent B (dam)** record — selected on breeding screen (0x04ccbc/0x04db44/0x0505a8/0x050a98/0x050c82) |
| `0x0C21A5A8` | **foal / result** record being built |
| `0x0C21A564..0x0C21A569` | the SIRE's own six externals (sire cell + 0x34), read by the pedigree count. Corrected 2026-09-28: was labelled "grandparent / 2nd-line"; it is not a grandparent row (0x0C21A530 + 0x34 = 0x0C21A564) |
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
truth table not exhausted, hence 0.7.) (Label DISPUTED 2026-09-28: a later decode,
card-seed-trait-readers.md §2, traces these two words to card bytes 60-63 and reads them as the COAT
word and the run-style-seed/personality word, not track/surface/distance aptitudes. The bit-mixing
itself is not in dispute.)

## E. Pedigree / bloodline threshold bonus  0x0C052BF0-0x0C052D20  (confidence 0.85)
After the external averages, a counter is accumulated over the 6 externals by comparing the
DAM's externals (`dam[0x34..0x39]`) with the SIRE's externals (`0x0C21A564..0x569` = sire + 0x34):
```
cnt = 0
for each of the 6 externals:
    if dam >= 12 and sire >= 12: cnt++      ; both high (display 13-16 = ◎)
    if dam <  4  and sire <  4 : cnt++      ; both low  (display 1-4  = ✕)
```
Then tiered bonus added to the THREE INTERNAL stats (foal +0x40/+0x44/+0x48):
```
if cnt == 4 or cnt == 5:  foal.sp(+0x44) += 2 ; foal.st(+0x40) += 1 ; foal.sh(+0x48) += 2
if cnt == 6:              foal.sp(+0x44) += 2 ; foal.st(+0x40) += 3 ; foal.sh(+0x48) += 3
clamp each of +0x40/+0x44/+0x48 to max 45
copy foal.st/sp/sh (low byte) -> foal[+0x74]/[+0x75]/[+0x76]
```
=> small **bloodline bonus** (+1..+3 per stat) when sire and dam MATCH, both high or both low, on 4 or
more externals; then internals hard-clamped to 45. Sire and dam count equally; it raises internals,
never externals. (Corrected 2026-09-28: an earlier, looser count here compared the dam with a
"grandparent / 2nd-line" row and counted each parent separately. The byte-exact count below is
per-external matches, and the second row is the sire's own externals.)

## F. Birth noise gate  0x0C052D90-0x0C052E8E+  (byte-exact, see BYTE-EXACT FOLLOW-UP below)
One uniform random byte (0-255) picks the outcome. Band 1, [92,100), about 1 birth in 32: the
PRINTED copy (display bytes 0x74/0x75/0x76) loses 12 on one stat or 5 on all three (only where it
is over 15), AND every true
internal (+0x40/+0x44/+0x48) gains 5. Band 2, [180,188), about 1 in 32: the PRINTED copy gains 12 on
one stat or 5 on all three, only where it is under 40; true internals untouched. No other bands.
(Corrected 2026-09-28: the first-pass description here, "keyed by a rating value", "further bands
repeat the pattern", was superseded by the byte-exact decompile below and removed.)

## Comparison to the community model (corrected 2026-09-28)
- Community simulator: internals = parent average; `ac` = average ± up to 18; external bands =
  average ± ~2 clamped 1-16; "Stamina/Speed Bloodline" and "Dirt Dynasty" bonuses; sex pick.
- ROM truth: internals = **exact floor-average** with ±5 soft clamps at 45/10 (§A), a small
  sire-vs-dam pedigree bonus (+1..+3, §E), hard cap 45, then the noise gate (§F): Band 1 lifts all
  three TRUE internals by 5 while the card prints lower; Band 2 raises only the printed copy. External
  bands = **exact floor-average** (§B), with NO spread; the only external roll is the 0-3 roll on the
  fresh foal's CURRENT externals (2 x band + roll + offset, see "Derived external displays" below).
  Sex = `rand & 1` (§C). None of the community's bloodline bonuses exist.
- (Corrected 2026-09-28: this said the noise pulls internals down by 5-12, that the external ± 2
  spread comes from the §E/§F layer, and that the community ± 18 was on internals. All three were
  wrong: the -12/-5 hits only the printed copy, §E/§F never touch externals, and the ± 18 was the
  community's `ac` roll.)

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
- parent A 0x0C21A530 / parent B 0x0C21A56C / foal 0x0C21A5A8 / sire externals 0x0C21A564
  (= parent A + 0x34; formerly mislabelled "2nd-line", corrected 2026-09-28)
- RNG state 0x0C21A124 (core 0x0C0B1E60) ; rand wrapper 0x0C09A1F8

## Confidence per claim
- Two-parent floor-average EXISTS and is at 0x0C052B0C: **0.92**
- Internal st/sp/sh = floor((sire+dam)/2) via 0x053414, ±5 soft clamp, cap 45: **0.95**
- 6 externals = floor((sire+dam)/2) (0x052B58–BEC): **0.95**
- Sex = rand&1: **0.85**
- Bloodline threshold bonus (>=12 / >3, +1..+3, §E): **0.85** (byte-exact below; sire vs dam)
- Aptitude = per-bit RNG inherit (not average): **0.7** (the "aptitude" label is DISPUTED, see §D)
- Birth noise gate (§F): byte-exact (see BYTE-EXACT FOLLOW-UP)
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
DAT_0c052e90=180 (0xB4), DAT_0c052e8e=188 (0xBC). Note the direction: the ±12/±5 lands on the
PRINTED copy (+0x74..0x76) only; Band 1 also adds 5 to every TRUE internal. (Clarified 2026-09-28.)

**Pedigree count (§E), exact:** for each of the 6 externals, compare DAM[0x34+i] with the
SIRE's own external (0x0C21A564..569 = sire + 0x34): `cnt++ if (dam>=12 && sire>=12)`, and
`cnt++ if (dam<4 && sire<4)`. cnt==4|5 -> internals st+1/sp+2/sh+2 ; cnt==6 -> st+3/sp+2/sh+3 ;
cap 45. (Rewards sire/dam CONSISTENCY, high-high or low-low; computable from the two parents alone.
Corrected 2026-09-28: this row was called "2nd-line/grandparent" and said to need a grandparent row.)

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
matching 3.1%×selector-prob. The pedigree bonus is exactly computable from the two parents (the
"grandparent row" turned out to be the sire's externals; corrected 2026-09-28). ac stays a per-bit
blend (no ac<->mask bridge). Everything else (averages, soft-clamp, cap45, sex, noise bands) is
byte-exact.

## CONSUMER WARNING: which card bytes the noise acts on (added 2026-09-10, copied 2026-09-28)
§F is byte-exact, but tools built on it read the wrong field for months. The two numbers it
separates live at DIFFERENT card offsets:
- PRINTED birth stats: World card bytes 113/114/115 (sta/spd/shp); JP 0x15 sta, 0x11 spd, 0x14 shp.
  Frozen at birth.
- TRUE internals: World card bytes 69/73/77 (sta/spd/shp); JP 0x03 sta, 0x02 spd, 0x04 shp. What the
  horse races on; they climb with racing.
Band 1 DIPS the printed bytes and RAISES the internals, so a card printing 45/45/33 or 40/40/40 from a
45/45/45 pairing is a 50/50/50 horse. Band 2 lifts the printed bytes alone, so a printed 50 or 51 is
genuine (base 39 + 12) and is NOT evidence of editing; above 51 nothing writes. Max true birth internal
is 50, max printed is 51. For about 94% of foals the two are equal. Only a 0-race card still states its
birth values; after that the site's birth record is the witness.

---

## APTITUDE GRADE DECODER — FUN_0c0534a4 (byte-exact)
(Label DISPUTED 2026-09-28: card-seed-trait-readers.md §2 identifies this same function (by its
runtime address) as the COAT classifier, whose output indexes the coat-name table GRAY / CHESTNUT / BLACK /
BAY / BROWN / SPECIAL / WHITE. The table logic below is byte-exact either way; "aptitude grade" and
"higher = stronger" are not established. Do not quote this as an aptitude grade.)
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
