# race-formula.md — Adversarial Byte Verification

Verifier pass against the real `.ic22` ROMs (all 4 versions). Every claim below was re-extracted
from raw bytes via python3 struct on the Windows ROM paths; no claim taken from the doc alone.
Helper scripts: `C:/Users/johnr/AppData/Local/Temp/v_*.py`.

**Overall verdict: MOSTLY-SOLID.** The headline structural findings (6-phase model + phase↔stat
naming, the distance→multiplier table and its cross-version locations, the FPU coefficient pool
values, the stat-role ranges/correlations, all cross-version notes) are byte-accurate. Two
descriptive claims are wrong/overstated and are corrected below; neither changes the model.

---

## CONFIRMED (re-extracted from bytes)

### Distance→multiplier table (§4.1) — CONFIRMED EXACTLY, highest-value find
- `v_dist.py`: signature `1700,1800,2000` found at **0x10F210 (WE-C), 0x110A70 (WE-D), 0x11439C (2000)**,
  byte-identical, and **absent in '99 (derbyoc)**. Exactly the claimed offsets.
- 9 distances: `1700 1800 2000 2100 2200 2400 2500 3000 3200` — exact.
- 12 multipliers: `1.3913 1.2308 1.0323 1.0625 0.8889 0.8649 0.8421 0.8205 0.8000 0.7273 0.6667 0.6400` — exact.
- All three ROMs are 4,194,304 bytes (matches horse-stats.md).

### Phase-name strings (§1) — CONFIRMED VERBATIM
- `v_misc.py`: ASCII at **0x0ED5B4** = `START . CORNER . OUT OF THE BOX . COMPETING . TENACIOUS . SPURT`,
  immediately followed by race result strings ("Glory and admiration", "%s has so many victories").
  The 6 externals are named 1:1 after the 6 race phases. This is the cornerstone of the model — solid.
  (Pinpoints: CORNER@0xED5BC, COMPETING@0xED5D4, TENACIOUS@0xED5E0.)

### Stat roles / scaling (§1, §3, §8) — CONFIRMED EXACTLY
Re-derived from the verified horse-stat table (WE-C @0x108E03/32B; '99 @0x0F6902/28B):
- WE-C externals (+9..+14) per-column: 11-63 / 14-59 / 4-63 / 8-63 / 3-62 / 4-63 → matches "~3-63".
- WE-C internals: Stamina(+29) **0-60**, Speed(+30) **0-63**, Sharp(+31) **0-60** → exact.
- WE-C ext_total mean **220.9** ("~220") and flat by style: {Front 225.1, StartDash 218.8, LastSpurt 222.3,
  Stretch 220.0, Almighty 227.3} → exact match to claimed ~218-227, confirming **style ≠ power**.
- WE-C **corr(ext_total,int_total) = 0.195 ≈ 0.20** → exact.
- '99 **corr = 0.556 ≈ 0.56** → exact. '99 style values present = {0,1,2,3} only (no Almighty=7).
- '99 internals at +24/+25/+26: 0-60 / 0-63 / 0-60. id at +1 = horse number (validates offsets).
- Dirt aptitude (+5) WE-C range **0-255** (59 distinct values, genuine spread) → confirms surface-input claim.

### FPU coefficient pools (§4.2) — VALUES CONFIRMED EXACTLY
`v_pools.py` / `v_more.py` dumped the leading f32 of each pool in WE-C; all match the doc:
- 0x46168: `0.5 1.2 0.6 2.5 0.45 2.4 0.15 0.24 1.5`; speed caps **80,70,60,50** present just before it.
- 0x53928: `0.9 0.8 0.7 0.5 0.3 0.2 0.1 0.027 0.6` (monotone falloff).
- 0x7C258 **and** 0x7C3C8: both `0.6 0.09 0.3 0.25 1.3 0.195 0.65 0.845 0.676` — genuinely a twin (byte-identical pair).
- 0x828BC: `0.5 0.7 1.8 1.5 0.6 2.0 0.4 0.3 3.0` (has the 3.0).
- 0xE7CA8: mixed weight/penalty (`1.0…0.8 0.6 0.5 0.3`).
- 0x102C00: tiered negatives `-0.35 -0.45 -0.75 -0.8 … -2.1 -2.0` (matches "tiered negatives").

### Code-adjacency, weaker form (§4.2) — DIRECTIONALLY CONFIRMED
- `v_code2.py`: 0xFxxx (SH-4 FP-class) instruction-word density in ±256 bytes is **33% @0x46168,
  27% @0x53928, 20% @0x7C258** vs only **2% @0x10F210**. So the §4.2 pools really are sitting in/among
  dense FP-instruction code (literal pools), while the distance table is an isolated pure-data table.
  The *region-level* "code-adjacent literal pool" claim holds. (See CORRECTION re: the exact bytes.)

### Track geometry (§4.3) — CONFIRMED (format loose)
- `v_track.py`: 0x0C8500 holds float records; leading triplets sit in ±3000 with lengths ~1000-1600
  (rec0 `-1600, 2, 1310, 250, 50, 1150 …`), consistent with spline/segment geometry, not stat math.
  Correctly disambiguated. (Exact "72-byte / (x,?,length) triplet" record stride is a plausible model,
  not independently proven — a few records contain large outliers.)

### Cross-version notes (§6) — CONFIRMED
- WE-C vs WE-D stat records: **0 differences** (byte-identical roster) → confirmed.
- WE-C vs 2000: **exactly 22 differing records** (matches horse-stats.md); **HIDDEN-X(+23/+24) differs
  in 8** of them → confirms "2000 rebalance touched HIDDEN-X on several horses" (§3/§6).
- Almighty(style=7) present in WE, absent in '99 (only 0-3) → confirmed.
- Distance table absent in '99 → '99 is a separate engine generation → confirmed.

---

## CORRECTIONS (claims that are wrong or overstated)

### C1 — Multipliers are NOT "descending" (§4.1, §8) — FACTUAL ERROR
The doc labels the 12 multipliers "(descending)". They are **not monotonic**: index 2→3 goes
**1.0323 → 1.0625 (ascending)**. The values printed in the doc are correct; only the "descending"
characterization is false. Likely the array is a per-distance factor keyed to the 9-distance array (not
a sorted curve), which actually *strengthens* the "lookup key set indexing a factor table" reading in §4.1.
Correction: call it a **per-distance factor table (non-monotonic)**, not a descending curve.

### C2 — "SH-4 FPU opcodes immediately precede each pool" (§8 code-adjacency proof) — OVERSTATED/WRONG
`v_code.py`: the 16/48 bytes **immediately before** 0x53928, 0x7C258, 0x46168 are **more float32 values**,
not 0xFxxx opcodes:
- before 0x53928: `0x3f000000`=0.5, `0x3ecccccc`=0.4 (continuing float data)
- before 0x7C258: `0x3f000000`=0.5, `0x3ecccccc`=0.4
- before 0x46168: `0x428c0000`=70.0, `0x42700000`=60.0, `0x42480000`=50.0 (the caps)
So the literal-pool conclusion is right, but the specific "opcodes precede the pool" evidence in §8 is
not what's in those bytes. The correct evidence is the **regional FP-instruction density** (C-confirmed
above), not the immediately-preceding words. Fix §8 wording.

### C3 — 0x102760 "rows of 0.4 0.52 0.6 -0.6 repeated 4-wide" (§4.2) — DESCRIPTION OFF
`v_more.py`: head of 0x102760 = `1.0 ×6, 0.9, 0.6, 1.0, 0.9, 0.9, 0.92`. The pool exists with plausible
coefficients, but the specific "0.4 0.52 0.6 -0.6 four-wide row" is not at the pool head as described.
Either the cited row is deeper in the pool or the pattern was misread. Low-impact (it's a 0.4-confidence
semantic label anyway), but the literal example should be corrected or dropped.

### Trivia (not errors)
- ROM string is **"OUT OF THE BOX"**; doc abbreviates "OUT_OF_BOX"/"OOB". Harmless.

---

## NET ASSESSMENT
- Every load-bearing, byte-decodable claim (offsets, table contents, stat ranges, correlations,
  cross-version identity/rebalance counts, phase strings) is **exactly correct**.
- The errors are in **prose characterization** of data the doc otherwise extracted correctly
  (C1 "descending", C2 "opcodes precede", C3 example row). None invalidate the empirical model.
- The doc is appropriately honest that the closed-form stat→speed function is SH-4 FPU code and not
  byte-decodable; that framing is sound. The §5 working model is a defensible synthesis, explicitly
  flagged as unproven arithmetic.
- Confidence ratings in the doc are reasonable; if anything the distance-table 0.9 could be raised given
  byte-identical cross-version stability, and the §8 code-adjacency claim should be downgraded pending C2.
