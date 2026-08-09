# DOC Race-Engine Coefficient Pools — Byte-Exact Catalog

ROM: `_sh4/rom/epr-22336c.ic22` (4,194,304 bytes, drbyocwc). All values are
little-endian IEEE-754 float32 read directly via `python3 struct`. SH-4 address =
`0x0C000000 + file_offset`. Cross-references via `ghidra/sh4dis.py`
(`scan_pcrel_loads`) + the target-EA scan from `find_pool_xrefs.py`.

> **Loader note (applies to every pool below).** Re-running the XREF scan confirms
> `RACE_FORMULA_FINDINGS.md §7`: **none** of these tables is reached by a direct
> `mov.l @(disp,PC)` whose literal value equals the pool base, and (except where
> noted) none is reached by a code-adjacent `mov.l @(disp,PC)` float load. They are
> addressed by a **base register computed from a larger data-block pointer plus a
> runtime displacement/index**, or copied to RAM at init and read from there. So the
> "loader" for most pools is the data-block/dispatch path, not a single PC-relative
> instruction. The nearest plausible block-base literal (and the function that loads
> it) is listed per pool as the loader handle. The two style pools are the exception:
> their setup is `FUN_0c07c164` (§5), reached at 0x0C05796A/0x0C057982.

---

## 1. `0x0C10F210` — distance → multiplier table  (21 f32)

9 distance keys (meters), then 12 multipliers.

| | values |
|---|---|
| keys (f32 ×9) | 1700.0, 1800.0, 2000.0, 2100.0, 2200.0, 2400.0, 2500.0, 3000.0, 3200.0 |
| mult (f32 ×12) @0x0C10F234 | 1.3913, 1.2308, 1.0323, 1.0625, 0.8889, 0.8649, 0.8421, 0.8205, 0.8000, 0.7273, 0.6667, 0.6400 |

- **count:** 21 plausible f32 (0x0C10F210..0x0C10F264). The word at +0x54
  (0x0C10F264 = 0x45552333) begins unrelated data, so the run ends at 21.
- **loaders:** none. No 4-byte literal == 0x0C10F210 and no `mov.l @(disp,PC)`
  resolves into it (confirmed by `scan_pcrel_loads`). Sits in the data block pointed
  to as a region (0x100000–0x118000); reached via a computed base, or via the RAM
  copy. The live value `1.0323` (2000 m) was observed in the per-horse RAM struct at
  `+0x10` of each phase record (`FINDINGS §9`), confirming this curve is read at
  runtime from a relocated copy. Nearest block-base literals are at 0x0C10E370+
  (loaded by code around 0x0C02BAB0), ~0xEA0 below the table.
- **curve:** the 12 multipliers are a **monotone falloff** 1.39 → 0.64 (one anomaly:
  index 3 = 1.0625 > index 2 = 1.0323). Short-distance horses get a >1 boost,
  long-distance get <1. 9 keys vs 12 multipliers: the multiplier list is longer than
  the key list (the extra 3 are sub-band / interpolation entries, the 9-vs-12 indexer
  is open Q#2).

---

## 2. `0x0C046168` — speed-term coefficient row  (9 f32)

`0.5000, 1.2000, 0.6000, 2.5000, 0.4500, 2.4000, 0.1500, 0.2400, 1.5000`

- **count:** 9 plausible f32 (0x0C046168..0x0C04618C). +0x24 (0x0C04618C =
  0x8B01F465) is the start of code, so the coefficient run is 9.
- **loaders:** no direct float load nor base literal in [-0x80,+0x40]. This pool is
  in the **code region** adjacent to the speed-math cluster near `FUN_0c044ab4`
  (0x0C044AB4). Nearest block-base literal 0x0C046000 (off −0x168) loaded at
  0x0C31E680 (second-image dispatch); the 0x0C0451xx/0x0C0452xx literals (loaded by
  the 0x0C31Exxx/0x0C322xxx driver) bracket this block. Reached as base+disp from
  that driver.
- **reading:** mixed small/large positive coefficients — interleaved pairs of a small
  weight (0.5, 0.6, 0.45, 0.15) and a large gain (1.2, 2.5, 2.4, 1.5). Looks like
  per-term `(weight, gain)` pairs feeding the weighted-sum accumulator, not a
  monotone curve.

---

## 3. `0x0C053928` — stamina / drain curve  (9 f32)

`0.9000, 0.8000, 0.7000, 0.5000, 0.3000, 0.2000, 0.1000, 0.0270, 0.6000`

- **count:** 9 plausible f32 (0x0C053928..0x0C05394C). +0x24 (0x0C05394C =
  0x8B0D8802) is code. Run = 9.
- **loaders:** no direct float/base literal at the pool. Code-region pool; nearest
  bracketing block-base literals 0x0C053700 (off −0x228, loaded by 0x0C018842,
  0x0C052936, 0x0C079ADC, 0x0C08109A) and 0x0C053788 (off −0x1A0, many loaders).
  This is the `@0x53928` drain pool cited in `FINDINGS §9.4` as the stamina-drain
  curve; its consumers are reached via that block base.
- **curve:** first 8 values are a clean **monotone falloff** 0.9 → 0.027 (a decaying
  drain/efficiency ramp). The 9th (0.6) is an out-of-sequence trailing constant
  (likely a base/reset multiplier appended to the curve, not part of the ramp).

---

## 4. `0x0C07C258` — per-style coefficient row A  (9 f32)

`0.6000, 0.0900, 0.3000, 0.2500, 1.3000, 0.1950, 0.6500, 0.8450, 0.6760`

- **count:** 9 plausible f32 (0x0C07C258..0x0C07C27C). +0x24 is code/data.

## 5. `0x0C07C3C8` — per-style coefficient row B  (9 f32)

`0.6000, 0.0900, 0.3000, 0.2500, 1.3000, 0.1950, 0.6500, 0.8450, 0.6760`

- **count:** 9 plausible f32 (0x0C07C3C8..0x0C07C3EC). +0x24 is code/data.
- **relationship:** the two rows are **0x170 (368 B = 92 f32) apart** but the gap
  between them is code, not floats — they are **two discrete 9-value rows**, not a
  contiguous 2×N matrix. In this build both rows hold the **identical** 9 values
  (the per-style differentiation is applied by the setup code, not by differing
  table data here).
- **loaders (both 4 & 5):** these are the twin style pools from
  `RACE_FORMULA_FINDINGS §5`. Setup function **`FUN_0c07c164`** writes per-style
  coeffs into the per-horse struct (offsets 16,20,52,56,88,92,96,104,108,140); style
  is later read from struct `+0x74`. The reachable PC-relative handles are the
  literals **0x0C07C10E** (loaded `mov.l` @ **0x0C05796A**, then `jsr`) and
  **0x0C07C11C** (loaded @ **0x0C057982**, then `jsr`) — i.e. the setup/accessor code
  immediately preceding the pools at 0x07C164. No literal targets the float rows
  directly; they are consumed as base+disp inside the setup routine.
- **reading:** a **per-style coefficient row** (one weight per race-math term:
  0.6/0.09/0.3/0.25/1.3/0.195/0.65/0.845/0.676). Value #5 (1.3) is the dominant gain;
  the rest are sub-1 weights. The 0.6 lead value is the byte-exact `99 99 19 3F`
  confirmation from §5.

---

## 6. `0x0C0828BC` — coefficient row  (9 f32)

`0.5000, 0.7000, 1.8000, 1.5000, 0.6000, 2.0000, 0.4000, 0.3000, 3.0000`

- **count:** 9 plausible f32 (0x0C0828BC..0x0C0828E0). +0x24 (0x0C0828E0 =
  0x0C1466DC) is data/code. Run = 9.
- **loaders:** no direct float/base literal at the pool. Code-region pool; nearest
  bracketing block-base literals 0x0C0826D4 (off −0x1E8, loaded @ 0x0C00AA48) and
  0x0C0823D2 (off −0x4EA, loaded @ 0x0C0616F2). Reached as base+disp.
- **reading:** mixed weights/gains, **not monotone** — small weights (0.5, 0.6, 0.4,
  0.3) interleaved with large multipliers (1.8, 1.5, 2.0, 3.0). Same `(weight, gain)`
  shape as pool 0x046168; likely another phase/condition coefficient row for the
  accumulator.

---

## 7. `0x0C102760` — coefficient matrix  (48 f32)

Header rows + a 4-column per-row style/offset block.

```
[ 0] 1.0  1.0  1.0  1.0  1.0  1.0  0.9  0.6      <- row of base/condition multipliers
[ 8] 1.0  0.9  0.9  0.92 1.0  1.0  1.0  1.0
[16] 0.4  0.52 0.6  -0.6   |  0.4  0.79 0.44 -0.43   <- 4-col rows: (a, b, c, -d)
[24] 0.4  0.7  0.3  -0.65  |  0.4  0.16 0.52 -0.84
[32] 0.55 0.7  0.3   0.65  |  0.4  0.52 0.6  -0.6
[40] 0.4  0.52 0.6  -0.6   |  0.4  0.79 0.44  0.43
```

- **count:** 48 plausible f32 (0x0C102760..0x0C102820). At +0xC0 (idx 48,
  0x0C102820) the data turns into ASCII debug strings
  ("RED  YOU…/WORK ORDER…/NETWORK ERROR…"), so the coefficient run ends at 48.
- **loaders:** no base/pointer literal at the pool. The one "code-adjacent" hit
  reported by the scanner (@0x0C1027D4 reading +0xA0=0.4) is a **false positive** —
  0x1027D4 is *inside this data table*, decoded as if it were an instruction. So there
  is no real instruction loader; the table is in the 0x100000–0x118000 data block and
  reached via a computed base (nearest block-base literal 0x0C101C70, off −0xAF0,
  loaded @ 0x0C01893C).
- **reading:** first 16 are **per-row base/condition multipliers** clustered at
  1.0 with a few reductions (0.9, 0.92, 0.6). The remaining 32 form **eight 4-tuples**
  `(0.4, b, c, ±d)` where the 4th column carries **negative offsets** (−0.6, −0.43,
  −0.65, −0.84, …) — a per-style row table where column 1 is a fixed weight (0.4) and
  column 4 is a signed correction.

---

## 8. `0x0C0E7CA8` — coefficient matrix  (60 f32)

```
[ 0] 1.0 1.0 1.0 1.0 1.0 | 0.8 0.6 0.8 0.5 0.8 | 0.3 1.0 0.3 1.0 0.3 | 0.0 0.0 0.0
[18] 1.0 1.0 1.0 1.0     | 0.8 1.0 0.8 0.1 0.1 0.0 0.2
[29] 1.0 1.2 1.5 0.8 2.0 1.0 1.5 1.5 1.2 2.0 1.2 1.0 2.0 0.5 2.0
[44] 0.5 -1.5 -1.0 -0.5 2.0
[49] 2.0 2.0 2.0 1.0 2.0 1.0  | -2.0 -1.2 -1.0 -0.5 | 2.0
```

- **count:** 60 plausible f32 (0x0C0E7CA8..0x0C0E7D98). At idx 60 (0x0C0E7D98) the
  run of clean coefficients ends (following words leave the small-magnitude band).
- **loaders:** no base/pointer literal targeting the pool. Data-block pool; nearest
  block-base literal 0x0C0E7C10 (off −0x98) loaded @ 0x0C009C8E, with a series of
  block-base literals 0x0C0E7644..0x0C0E76E4 loaded by code around 0x0C003A64–
  0x0C003AEA. Reached as base+disp from that init/dispatch path.
- **reading:** a **multi-row coefficient matrix**. Opening rows are mostly 1.0 base
  multipliers with per-column reductions (0.8/0.6/0.5/0.3) and a zero column. The
  later rows mix gains up to 2.0 and include **negative entries** (−1.5, −1.0, −0.5,
  −2.0, −1.2) — signed per-style/per-phase adjustments, not a monotone curve.

---

## 9. `0x0C102C00` — signed offset matrix  (204 f32)

The largest pool: a structured table of signed offsets. First three blocks of 12
(rows of 12 = per-distance-class), then sign-banded sub-tables.

```
[  0] -0.35 -0.45 -0.75 -0.80 -0.80 -1.40 -1.40 -1.50 -1.55 -1.55 -2.10 -2.00
[ 12] -0.64 -0.54 -0.54 -0.74 -0.79 -0.89 -0.89 -0.70 -0.70 -0.72 -1.75 -1.75
[ 24] -1.38 -1.30 -1.25 -1.34 -1.59 -1.54 -1.69 -1.75 -1.75 -1.75 -2.90 -2.90
[ 36] -0.05 -0.05 -0.05 -0.05 -0.10 -0.10 -0.20 -0.20 -0.20 -0.20 -0.25 -0.25
[ 48]  0.25  0.20  0.15  0.10  0.05  0.00  0.00 -0.05 -0.10 -0.15 -0.20 -0.25
[ 60]  0.20  0.10  0.00  0.00 -0.05 -0.10 -0.15 -0.20 -0.25 -0.25 -0.35 -0.40
[ 72] -1.00 -1.00 -1.10 -1.15 -1.20 -1.60 -1.60 -2.30 -2.40 -2.50 -3.12 -3.22
[ 84]  0.00  0.08  0.16  0.24  0.32  0.40  0.48  0.56  0.70  0.78  0.88  1.00
[ 96] -1.00 -0.90 -0.80 -0.70 -0.70 -0.60 -0.50 -0.40 -0.30 -0.20 -0.10  0.00
[108]  1.11  1.11  0.85  0.50  0.35  0.90  2.00  1.60  0.50  0.50  0.30  1.011
[120]  1.60  1.60  1.25  0.40  0.30  0.90  0.50  0.80  0.60  0.60  0.40  0.90
[132]  0.10  0.20  1.411 0.50  0.70  1.01  0.40  0.30  0.45  1.10  1.05  0.90
[144]  0.60  0.60  0.70  0.85  1.05  1.011 1.01  1.01  1.25  1.40  2.20  1.011
[156]  1.74  1.74  1.161 0.60  0.65  0.90  1.001 1.90  1.161 0.60  0.20  1.09
[168]  1.001 1.54  1.161 0.30  0.30  1.09  0.30  0.55  1.415 0.65  0.55  1.09
[180]  0.30  0.50  0.45  1.162 0.80  1.09  0.40  0.60  0.60  1.162 1.30  1.09
[192]  0.14  0.61  0.65  1.101 1.70  1.011 0.50  1.20  1.20  1.30  1.95  1.011
```

- **count:** 204 plausible f32 (0x0C102C00..0x0C102F2C). At idx 204 (0x0C102F30 =
  0x03020100) an `00 01 02 03 …` byte index-array begins, so the float table ends at
  204.
- **loaders:** no base/pointer literal targeting the pool. The several "code-adjacent"
  hits the scanner reports (@0x1029FA, 0x102A0E … @0x102D54 reading +0x180=−1.0) are
  **false positives** — those addresses lie *inside this very data table* and only
  look like `mov.l @(disp,PC)` when force-decoded. There is no genuine instruction
  loader; the table is in the 0x100000+ data block, reached via a computed base
  (nearest block-base literal 0x0C101C70, off −0xF90, loaded @ 0x0C01893C, the same
  block base as pool 0x102760).
- **reading:** a **signed offset matrix**. Rows 0–2 (and 6) are **negative penalty
  rows** monotonically deepening (−0.35 → −3.22). Rows at idx 36–71 are **graded
  ±ramps** centered on 0 (e.g. idx 48: +0.25 → −0.25 symmetric, idx 84: +0.00 →
  +1.00 rising, idx 96: −1.00 → 0.00 rising) — classic per-position/per-phase
  correction ramps. From idx 108 on, the values shift to **positive multipliers**
  (~0.3–2.2, repeating sub-1/super-1 cadence with markers like 1.011/1.161/1.162
  every ~12) — a per-style gain block. Overall: a large per-phase/per-style table of
  signed corrections (negatives) and gains (positives), the richest of the nine pools.

---

## Summary table

| pool (SH-4) | file off | f32 count | shape | loader handle |
|---|---|---|---|---|
| 0x0C10F210 | 0x10F210 | 21 (9 keys + 12 mult) | monotone falloff 1.39→0.64 | block-base ~0x10E370 (@0x02BAB0); RAM copy read at runtime |
| 0x0C046168 | 0x046168 | 9 | (weight,gain) pairs | block-base 0x046000 (@0x31E680) |
| 0x0C053928 | 0x053928 | 9 | drain falloff 0.9→0.027 +trailing 0.6 | block-base 0x053700/0x053788 (@0x018842 …) |
| 0x0C07C258 | 0x07C258 | 9 | per-style coeff row A | setup FUN_0c07c164 (@0x05796A) |
| 0x0C07C3C8 | 0x07C3C8 | 9 | per-style coeff row B (== A here) | setup FUN_0c07c164 (@0x057982) |
| 0x0C0828BC | 0x0828BC | 9 | (weight,gain) pairs | block-base 0x0826D4 (@0x00AA48) |
| 0x0C102760 | 0x102760 | 48 | 16 base mult + 8×(0.4,b,c,±d) | data-block 0x101C70 (@0x01893C); no instr loader |
| 0x0C0E7CA8 | 0x0E7CA8 | 60 | multi-row matrix w/ negatives | block-base 0x0E7C10 (@0x009C8E) |
| 0x0C102C00 | 0x102C00 | 204 | signed offset/gain matrix | data-block 0x101C70 (@0x01893C); no instr loader |

**Caveat on "loaders":** per `find_pool_xrefs.py`, *zero* of these nine pools has a
PC-relative literal whose value equals its base, and the only `mov.l @(disp,PC)`
"hits" into the 0x102760 / 0x102C00 tables are mis-decoded data bytes. The genuine
access path is base-register + runtime displacement from the listed data-block /
dispatch pointers (or, for 0x10F210, a RAM copy). The two style pools are the only
ones with a named consuming function, `FUN_0c07c164` (§5). All float values above are
byte-exact from the ROM.
