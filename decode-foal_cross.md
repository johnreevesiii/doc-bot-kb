# Foal Cross Routine — SH-4 Static RE (epr-22336c.ic22)

ROM base 0x0C000000; runtime = static + 0x20000; pointers baked in ROM are runtime.
The 112-byte (0x70) candidate records live at runtime 0x0C3C9640 (count @0x0C3C94FC,
work pointer @0x0C3C9B80). NOTE: the ROM literal pools bake the RUNTIME addresses
(0xC3C9640 / 0xC3C9B80 / 0xC3C94FC), not the static ones — verified via
scan_pcrel_loads (0xC3A9640 has zero loaders; 0xC3C9640 has 5).

## TL;DR / Verdict

I could NOT find a routine in this ROM that reads TWO selected candidate (or
sire+dam) records and writes a foal whose internal stats are the float/int average
`(fa+fb)/2` of the two parents. The signature ("foal internals = exact parent
average") does NOT appear as a two-parent read-add-halve in the breeding subsystem.

What the breeding subsystem (0x05E000–0x062500) actually contains is a **single-source
candidate GENERATOR plus a presentation/compatibility minigame**. Each offered foal's
internal stats come from `template_stat * scale ± rand*noise` (one template, not two
parents). See evidence below.

Two reasonable interpretations of the live observation (foal sp 46 = avg(52,40)):

1. The two "parents" you saw are not both read by the offering code. The averaging
   happens **upstream**, where the breed/parent TEMPLATE that FUN_0c05ee68 copies from
   is itself produced — most plausibly at race-retirement / card-write (the path that
   turns a raced horse into a breeding template), which is OUTSIDE the 0x05E–0x062
   offering minigame and was not located in this ROM region.
2. OR the average is a coincidental artifact of `template_stat * scale` landing on the
   midpoint for that one observation. The generator's exact arithmetic (below) does NOT
   structurally guarantee an average, so I rate this lower.

Confidence that "there is no two-parent floor-average cross inside 0x05E000–0x062500":
**0.85**. Confidence that the offered-foal stats are single-template `*scale±noise`:
**0.9**.

## Evidence

### 1. Candidate GENERATOR is single-source (not a cross)
- `FUN_0c05ef3a` (static 0x0C05EF3A): loop `for i in [0,count)`: builds record at
  `base + i*0x70`, seeds fields +0x44 (index), +0x5c (rand&7), +0x54, +0x58, +0x28,
  then calls `FUN_0c05ee68(src)` where `src = FUN_0c05ea6e()`.
- `FUN_0c05ea6e` → `FUN_0c05ea3c() + DAT_0c05eb88`. `FUN_0c05ea3c` (0x0C05EA3C) scans a
  **12-entry table** at runtime 0x0C171328 (static 0x0C151328) for an entry whose
  byte@0x113 matches a key, and returns that ONE template pointer. Single parent/breed
  template — there is no second parent pointer.
- `FUN_0c05ee68` (0x0C05EE68) writes the 5 float stats:
  - `+0x14 = lookup(src_stat0) * scale  - rand_float()*noise0`
  - `+0x18 = (prev +0x14) + bias`        (chained from the just-written field)
  - `+0x1c = lookup(src_stat1) * scale  - rand_float()*noise1`
  - `+0x20 = (prev +0x1c) + bias`
  - `+0x24 = const0 + rand_float()*const1`
  where `scale = FUN_0c05f11c()` (a float from table 0x05f1a8 indexed by a global),
  `lookup(x)` is a signed-byte table read `*(char*)(table + x*2)`. All inputs trace to
  ONE `src` template + RNG. No second record is dereferenced.
- RNG: `rand_float` wrapper runtime 0x0C0B1E80 / int core 0x0C0B1E60 (confirmed). The
  generator draws ~8 floats per horse (matches the known noise model).

### 2. The 0x060xxx cluster is the presentation / compatibility minigame, not a cross
All work-area functions that touch 0x0C3C9B80 / 0x0C3C9640 were decompiled and read:
- `FUN_0c060df6` (entry 0x0C060DF6, covers xref 0x060e16/0x060e56): reads candidate
  stat +0xc, compares to an RNG-driven threshold, increments/decrements a reaction
  counter at record+0x4c. UI "reaction gauge."
- `FUN_0c060e8c` (0x0C060E8C, xref 0x060e90): compares the work record's stat fields
  +0x14/+0x18/+0x1c/+0x20/+0x24 against the active candidate's +0x28, writes a result
  code at +0x58, then calls FUN_0c060df6. Pure UI evaluation.
- `FUN_0c060fca/06102e/0610ea/0610fc`: tiny state setters (write record+0x58/+0x60).
- `FUN_0c0613ba` (0x0C0613BA): per-tick state machine, bumps timers +0x44/+0x48,
  dispatches on +0x60, writes a per-record byte. Animation.
- `FUN_0c0614ae` (0x0C0614AE): the per-record loop driver — `base + idx*0x70` confirms
  0x70 stride — calls per-record update helpers (0x061492, 0x060986, 0x0613b0) and
  shuffles two 10-byte history buffers (record +0x78/+0x7c). Animation/UI, single
  record per iteration.

### 3. The 0x05F4xx functions iterate ONE record at a time
- `FUN_0c05f44a`, `FUN_0c05f4c2`: `for i in [0,count)`: `rec = base + i*0x70`; write/clamp
  the +0x28 threshold from an external table indexed by `FUN_0c05f70a(i)`. Single record.
- `FUN_0c05f70a` = linear search of a class table (returns index). `FUN_0c05f73e`,
  `FUN_0c05f770` = float-constant table reads indexed by class id (thresholds/scales),
  NOT parent stats.

### 4. The one weighted-blend I found is MATE COMPATIBILITY, not foal stats
`FUN_0c060668` (entry 0x0C060668; body at 0x0607C8) — found via the integer-halving
scan (add→shar at 0x0606ac and 0x060708). It computes a **3:1 weighted blend**, not a
1:1 average, and uses it as a matchmaking threshold:
- `blendA = (byteA0*3 + byteA1) >> 2` over the work record's bytes @0x95/0x96
- per candidate j (idx `< count`, accessed via FUN_0c05f732): require
  `byte[0x94] >= 0x30` and `byte[0x8a] == 2`, then `blendB = (byte94*3 + byte95) >> 2`
  (+10 if `byte95 > 0x3c`); compare `blendB >= blendA`; on pass call
  `FUN_0c05fd5c(work, 1|2)` (sets a "compatible" flag). This is a partner-selection
  filter. `>>2` (3:1), not `>>1` (1:1) — does not match the foal-average signature, and
  it writes a flag, not a stat.

### 5. Whole-ROM scans for the average idiom came up empty in breeding
- Integer `add Rm,Rn; shar/shlr Rn` (same reg, within 8 bytes): **22 sites ROM-wide**;
  in breeding range only 0x029c28 (UI name-layout centering, references string
  "d_Pepper", `(a+b+6)/2 * -0x16 + base` = screen X), and 0x0606ac/0x060708/0x06070a
  (the 3:1 matchmaking blend above). None is a 1:1 stat average of two records.
- Float `*0.5` constant (0x3F000000): only **2 loaders ROM-wide** (0x059c34, 0x08c224),
  NEITHER in the breeding range. 0x059c34 = `field += statbyte * 0.5` accumulator
  (single-source pedigree/rating sum, not a two-parent average). 0x08c224 = `x + 0.5`
  round-before-truncate. So there is no `(fa+fb)*0.5` in breeding.
- Candidate array base 0xC3C9640 is loaded at only 5 sites (0x05efb0, 0x05f44a,
  0x05f4c8, 0x0609ec, 0x0614be) — every one is a single-index `base + i*0x70` loop.
  No function forms TWO distinct `idx*0x70` offsets into this array, which a
  sire-record + dam-record average would require.

## 112-byte record field map (refined this session)
- +0x0C : float "current/display stat" the UI reads (FUN_0c060df6 / FUN_0c060668)
- +0x14,+0x18,+0x1c,+0x20,+0x24 : 5 float internal stats (set by generator)
- +0x28 : float comparison threshold (set from class table by FUN_0c05f44a/4c2)
- +0x44 : int index; +0x48 : timer; +0x4c : reaction counter; +0x50 : flag
- +0x54 : seeded value; +0x58 : result/reaction code; +0x5c : small int (rand&7)
- +0x60 : UI state selector (0/1/2/3); +0x64 : 0
- +0x6c : substate; +0x78/+0x7c : 10-byte history ring buffers (FUN_0c0614ae)
- bytes +0x8a/+0x8b/+0x94/+0x95/+0x96/0x113 : breed/class/grade key bytes used by
  FUN_0c060668 (matchmaking) and FUN_0c05ea3c (template lookup)

## Per-field formula of what DOES run (the offered-foal generator)
For each offered candidate i, with single template `src`:
```
scale  = f32_table_05f1a8[ global_idx ]            // FUN_0c05f11c
s0     = (int8) byte_table[ src.stat0 * 2 ]         // signed lookup
rec.+0x14 = s0 * scale  -  rand_float() * noise0    // noise0 = f32 @05ef80
rec.+0x18 = rec.+0x14   +  bias_a                   // bias from 05ef84/88
s1     = (int8) byte_table2[ src.stat1 * 2 ]
rec.+0x1c = s1 * scale  -  rand_float() * noise1
rec.+0x20 = rec.+0x1c   +  bias_b
rec.+0x24 = const0      +  rand_float() * const1    // 05ef90/94/98
rec.+0x5c = (rng >> 4) & 7   (clamped to 7)         // aptitude-ish
rec.+0x28 = class_table[ class(i) + 0x50 ]          // threshold, later
```
This is NOT a parent average; it is template * scale ± noise.

## "Hidden/composite" field check
The generator copies the foal's stat SOURCE from a single template's bytes (the breed
template at 0x151328, keyed by byte@0x113). I found no read of a hidden per-parent
"composite" byte being blended from two parents in this region. The only multi-byte
"composite" reads (0x8a/0x94/0x95/0x96) feed the compatibility filter, not the foal
stats.

## What to do next to fully close it (recommended)
The two-parent floor-average is most likely realized when a RACED horse is converted to
a breeding template (retirement / card-write), feeding the table at static 0x151328
(runtime 0x171328). That writer is OUTSIDE 0x05E000–0x062500 and was not in scope here.
Recommended next step: find writers of runtime 0x0C171328 (scan_pcrel_loads for
0xC171328) and the card-save serializer; the `(parentA+parentB)>>1` is expected there.
Alternatively, confirm dynamically: log the FUN_0c05ea3c template bytes vs. the two
on-screen parents to see whether the template is already pre-averaged before the offer.

## Confidence per claim
- Generator FUN_0c05ee68/ef3a is single-template `*scale±noise`: **0.9**
- 0x060xxx cluster is UI/animation, not a cross: **0.9**
- FUN_0c060668 is 3:1 matchmaking blend (not foal stats): **0.85**
- No `(fa+fb)/2` (int or float) over two candidate records exists in 0x05E–0x062: **0.85**
- The real two-parent average lives in the retirement/card-write path (outside scope): **0.5**
