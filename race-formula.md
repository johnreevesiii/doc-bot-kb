# Race Performance Model — How Stats Become Race Results

KEY: `race-formula` · Subsystem: the SH-4 race simulation that turns a horse's stats + track + condition
into per-tick speed/position and a finishing order.

Status: **partial / empirical + located.** The full stat-to-speed function is **FPU code in the SH-4
program ROM, not a clean data table**, so it is NOT byte-decodable the way the horse-stat table was.
What IS now nailed down (byte-verified): the meaning of the six "external" stats as **per-phase**
abilities, the three "internal" stats as **global capacities**, the role of running style as a
**behavior tag (not a power tier)**, and — newly discovered this session — the **race-physics literal
pools embedded in the code** (including a confirmed, version-stable **distance→multiplier table**).
A complete closed-form formula requires SH-4 disassembly or MAME memory-trace; tractability assessed in §7.

> Framing: the player horse's live stats are read from card/cabinet; the 244-record table
> (`horse-stats.md`) is the CPU-opponent roster. Both feed the SAME race engine. The engine is what this
> doc is about. The stat *fields* are fully decoded in `horse-stats.md`; here we decode their *function*.

---

## 1. The 6 phases ↔ the 6 external stats (the core insight, verified)

DOC races are explicitly modeled as a sequence of **6 phases**, and the six external stats are named
1:1 after them. This is confirmed by the in-ROM stat-name strings (CLAUDE.md @0x0ED5B4) and by the
phase list in the AI "Master Architecture"/MAME briefing:

```
START  →  CORNER  →  OUT_OF_BOX  →  COMPETING  →  TENACIOUS  →  SPURT
gate      first      breaking      mid-race       late push    final
break     turn       from pack     positioning                 sprint
```

| phase | external stat (rec offset, 32B fmt) | what it governs |
|---|---|---|
| gate break | **Start** (+9) | jump speed out of the gate / early accel |
| first corner | **Corner** (+10) | speed held through turns |
| break from pack | **OOB** out-of-box (+11) | repositioning out of the early bunch |
| mid-race | **Competing** (+12) | sustained cruising / jockeying mid-race |
| late push | **Tenacious** (+13) | holding/fighting in the late-middle |
| final sprint | **Spurt** (+14) | closing kick to the wire |

So the engine is a **phase machine**: as the horse crosses each phase boundary on the track, the
phase-relevant external stat becomes the dominant input to its speed. **Verified**: the externals are
roughly balanced across the roster (each ~3–63, total ~220 regardless of running style — §3), which is
exactly what you'd expect if they are *per-phase weights* rather than a single power score.

### The 3 internal stats = global capacities (verified scaling)
| internal (rec offset 32B) | range (244 horses) | role (model) |
|---|---|---|
| **Stamina** (+29) | 0–60 | size of the energy pool that drains over the race; gates how long high speed is sustainable |
| **Speed** (+30) | 0–63 | top-speed ceiling the phase math scales toward |
| **Sharp** (sharpness, +31) | 0–60 | acceleration/responsiveness — how fast the horse reaches its phase speed, and whip responsiveness |

Internals are global (apply all race), externals are local (apply in their phase). corr(ext_total,
int_total) ≈ 0.20 on WE-C and ≈ 0.56 on '99 — weak/moderate, i.e. they are **independent axes**, not
redundant. (Computed over all 244 records; script in §8.)

---

## 2. Running style = behavior tag, NOT power (verified)

Style (`horse-stats` rec +21: 0 Front / 1 Start-dash / 2 Last-spurt / 3 Stretch / 7 Almighty) does
**not** change the horse's total ability — avg external-total by style is flat (~218–227 across all five
styles; see §8 output). Its job is to bias **which phase the horse spends its energy in / commits to**:

- **Front-runner (0):** commits speed early, tries to lead from gate — leans on Start/Corner, must not
  empty Stamina before Spurt.
- **Start-dash (1):** explosive gate burst (Start-weighted), then settle.
- **Last-spurt (2):** conserves, dumps energy in the Spurt phase (Spurt/Sharp-weighted closing kick).
- **Stretch-runner (3):** accelerates through the final stretch (Tenacious→Spurt).
- **Almighty (7, WE only):** versatile; the engine picks the best phase to commit by situation.

This matches the duplicated style-coefficient pools found in code (§4: the `0x7C258`/`0x7C3C8` twin
clusters and the repeated 4-wide rows at `0x102760`), which read like per-style coefficient rows.

---

## 3. Stat scaling that feeds the engine (verified)

- **Externals stored 0–63, displayed 1–16.** Card external display = `card_value + 1` (CLAUDE.md card
  spec, T2[38..43]); the CPU table stores the raw engine value 0–63. (Display banding X/A/O/@ is a
  separate breeder-UI mapping; see `horse-stats.md`.)
- **Internals 0–63**, used raw.
- **Dirt aptitude** (rec +5 / card T3[61]) **0–255**: surface-match multiplier. On a dirt track a horse
  with high Dirt keeps its speed; low Dirt is penalized. (Turf is the inverse / default.) This is the
  surface-weighting input. Confidence on *direction* high; exact curve = code.
- **HIDDEN-X 16-bit** (rec +23/+24) clustered high-byte (0xA0/0x30/0xC0/0xF0): top candidate for a
  **distance/surface aptitude composite** that indexes the distance machinery in §4 (rebalanced in the
  2000 build → it is a live race input, not cosmetic). Unconfirmed semantics — flagged in `horse-stats.md`.

---

## 4. Race-physics constant pools located in the SH-4 code (NEW this session)

The stat→speed math is FPU code. Its **coefficient literal-pools are embedded in the code region** and
were located by scanning for runs of plausible-magnitude little-endian float32 and confirming SH-4 FPU
instructions immediately precede them (fmov/fmul-class opcodes in the 0xF0xx range + `mova`/`bf`). All
offsets below are in **drbyocwc / epr-22336c** and were dumped from real bytes (§8).

### 4.1 DISTANCE → MULTIPLIER table — **CONFIRMED, version-stable** (highest-value find)
- **drbyocwc @0x10F210**, derbyocw @0x110A70, derbyo2k @0x11439C (byte-identical values in all three;
  absent in '99 derbyoc, which has a different track/roster set).
- Distances (9× f32, meters): `1700 1800 2000 2100 2200 2400 2500 3000 3200`
- Multipliers (12× f32, descending): `1.391 1.231 1.032 1.062 0.889 0.865 0.842 0.821 0.800 0.727 0.667 0.640`
- These are real race distances. The descending multiplier curve is a **distance normalization / base-pace
  factor**: longer race → lower per-unit factor (a horse can't hold sprint pace over 3200m). The 9-vs-12
  count mismatch means the distance array is a *lookup key set* indexing into a longer (12-entry) curve,
  or the 12-curve covers a superset incl. sub-1700m sprints. This is the clearest single piece of the
  race formula recovered as data. **Confidence 0.9** it is distance-related; 0.6 on exact use.

### 4.2 Embedded FPU coefficient clusters (inside code @0x46xxx–0x82xxx)
Confirmed to be code-adjacent literal pools (SH-4 FPU ops precede each — verified bytes in §8):
| addr (WE-C) | values (leading) | reading |
|---|---|---|
| `0x46168` | `0.5 1.2 0.6 2.5 0.45 2.4 0.15 0.24 1.5` (preceded by 80,70,60,50 caps) | phase/speed coefficients w/ speed caps 50–80 |
| `0x53928` | `0.9 0.8 0.7 0.5 0.3 0.2 0.1 0.027 0.6` | monotone falloff curve — pace/stamina-drain or per-segment speed profile |
| `0x7C258` **and** `0x7C3C8` (twin) | `0.6 0.09 0.3 0.25 1.3 0.195 0.65 0.845 0.676` | duplicated = used in two branches (two running-style code paths) |
| `0x828BC` | `0.5 0.7 1.8 1.5 0.6 2.0 0.4 0.3 3.0` | weight set with a 3.0 — possibly whip/burst multiplier row |
| `0x102760` (48×) | rows of `... 0.4 0.52 0.6 -0.6 ...` repeated 4-wide | per-style/per-personality coefficient matrix |
| `0xE7CA8` (60×) | `1.0…0.8 0.6 … 2.0 1.5 -1.5 -1.0` mixed | weight/penalty matrix (12-wide rows) |
| `0x102C00` (204×) | tiered negatives `-0.35 … -2.9` then small `+0.25 0.2 0.15 0.1` | per-position/per-condition adjustment offsets |

These are coefficients **into** the formula, not the formula. Exact wiring (which register holds which
stat, how products accumulate into velocity) needs disassembly. **Confidence 0.85** these are the race
pools (code-adjacency + magnitude + version stability); **0.4** on each individual semantic label.

### 4.3 Track geometry table (NOT the stat formula — disambiguated)
- **@0x0C8500**, 72-byte records of f32 triplets `(x, ?, length)` with coords in ±3000 and lengths
  1000–3000. This is **track spline/shape geometry** (where corners are, segment lengths), i.e. it tells
  the phase machine *where* phase boundaries fall on each course. It is an *input to* the race (defines
  the phases per track) but is **not** the stat→speed math. Decoded enough to rule it out as the formula.
  CLAUDE.md's "96×72 track parameter table" = this geometry, confirmed by bytes (§8).

---

## 5. Working model (empirical synthesis — the honest "best current understanding")

Per simulated tick, for each horse, the engine is consistent with:

```
phase            = phase_for(track_geometry@0x0C8500, current_distance)   // 6-phase machine
phase_ability    = external[phase]                       // 0..63, from rec/card
base_pace        = distance_mult(race_distance)          // table @0x10F210 (1.39..0.64)
surface_factor   = f(dirt_aptitude, track.surface)       // dirt 0..255; turf default
target_speed     = SpeedCap(internal_speed)              // ceiling ~ caps seen near 0x46168 (50..80)
accel            = g(internal_sharp, phase_ability)      // sharpness → how fast target is reached
energy          -= drain(phase, current_speed, internal_stamina)   // falloff curve @0x53928
condition_mod    = h(condition, trust, hearts)           // card T2[44]/[36]/[37], mood inputs
style_bias       = style_coeff_row(running_style)         // pools @0x7C258 / @0x102760
v(tick)          = clamp( target_speed * base_pace * surface_factor * style_bias
                          * phase_ability_norm * condition_mod, 0, cap )
                   - (energy<=0 ? stamina_penalty : 0)    // running on empty tanks speed
position        += v(tick); finishing order = arrival at finish distance
+ whip: timing window briefly boosts accel/target via internal_sharp, costs trust/energy if mistimed
+ small RNG variance per tick (race-to-race noise)
```

- **Which stat matters per phase:** the external named after that phase (Start@gate … Spurt@finish);
  Sharp governs accel into every phase; Speed sets the ceiling; Stamina sets how many phases you can hold.
- **Distance weighting:** table @0x10F210 (confirmed).
- **Dirt/surface weighting:** Dirt aptitude 0–255 vs track surface (direction confirmed; curve = code).
- **Condition/trust/hearts:** multiplicative mood/fitness modifiers from the card (T2[44]/[36]/[37]).
- **Leg-type interaction:** style biases WHICH phase the horse commits energy to (§2), via the per-style
  coefficient pools — it reshapes the energy/speed allocation across the 6 phases, not the totals.

Everything in `f()/g()/h()/clamp` is parameterized by the §4 pools but the exact arithmetic is unproven.

---

## 6. Cross-version notes
- The distance→mult table (@0x10F210 family) is **byte-identical across WE-C, WE-D, DOC2000**, absent in
  '99 → the three later builds share one race engine + one track/distance set; '99 is a separate engine
  generation (also a different 28-byte stat record, different roster — see `horse-stats.md`).
- The 2000 rebalance touched HIDDEN-X (rec +23/+24) on several horses → that field is a real race input,
  reinforcing the §3 "distance/surface aptitude composite" candidacy.
- Style value 7 (Almighty) exists only in WE; '99 has only styles 0–3 → '99 has no "versatile" branch in
  its style-coefficient logic.

---

## 7. Tractability — exactly what full decode requires (honest assessment)

**Can it be cracked from data alone? No.** The roster stat table is data (done). The *formula* is SH-4
FPU code; only its coefficient pools are data (now located). To get the closed-form function you need
**one of**:

1. **SH-4 disassembly** of the race loop. Cost/risk:
   - No off-the-shelf SH-4 disassembler in this environment (capstone has no SH-4 backend). Options:
     Ghidra (has an SH-4/SH-2 module), `sh-elf-objdump` (binutils SH target), or m68k-style hand-decode.
   - Entry: find the code that **references** the §4 pools. SH-4 loads float literals via `mov.l
     @(disp,PC)` then `fmov`/`fmul`; cross-reference the pool addresses (0x10F210, 0x53928, 0x7C258…) to
     their loaders to find the race-math functions. The track-geometry reader (@0x0C8500) and the
     per-tick position updater are the two functions to recover.
   - Effort: this is a *bounded* RE task now that the pools are located (you're looking for PC-relative
     loads of ~8 specific addresses), but still days of focused SH-4 work. **Medium-hard, tractable.**

2. **MAME/Flycast memory trace** (the AI briefing's recommended path, easier than static disasm):
   - Run drbyocwc in MAME `-debug`; set watchpoints on the horse stat addresses during a race; `trace`
     the race loop; watch the per-horse "current speed" RAM word update each frame. Read off the
     coefficients live. This directly yields the formula behavior without reading SH-4 by hand.
   - **This is the highest-ROI next step.** The pools in §4 give you the constants to *confirm* the trace.

3. **Pure empirical (no code):** race known horses in the emulator and curve-fit finishing
   times/positions vs stats per distance/surface. Yields a *behavioral* model good enough for a
   recreation/odds tool, never the exact ROM arithmetic. Valid, lower-fidelity.

**A partial/empirical model (this doc) is a legitimate stopping point**: phases↔stats, internals'
roles, style-as-behavior, the confirmed distance table, and the located coefficient pools are enough to
build a *plausible* race sim and an odds/strength estimator today.

---

## 8. How verified (reproducible)
- All offsets dumped from the real `.ic22` via python3 struct on Windows paths. Helper scripts in
  `C:/Users/johnr/AppData/Local/Temp/rf_*.py`.
- **Distance table**: `rf_xver.py`/`rf_xver2.py` — signature `1700,1800,2000,2100,2200,2400,2500,3000,3200`
  found at 0x10F210 (WE-C), 0x110A70 (WE-D), 0x11439C (2000); same 12 multipliers each; not present in '99.
- **Float pools**: `rf_floats.py` scanned 0x200–0x180000 for runs of ≥8 small float32 with ≥40% nonzero;
  32 clusters found; `rf_floats2/3.py` dumped the race-relevant ones with context.
- **Code-adjacency proof**: `rf_code.py` dumped 48 bytes before 0x53928 / 0x7C258 / 0x46168 — SH-4 FPU
  opcodes (0xF0xx fmov/fmul-class), `mova`, `nop` 0x0009, `bf` 0x8b precede each pool → these are literal
  pools loaded by FP race math, not standalone data tables.
- **Track geometry**: `rf_track2.py` decoded 0x0C8500 as 72-byte float records → coordinates/lengths
  (±3000 / 1000–3000), i.e. spline geometry, not stat math.
- **Stat roles/scaling**: `rf_stats.py` — per-column external ranges (each ~3–63), internal ranges
  (0–63), ext-total ~220 flat across all 5 styles (style≠power), corr(ext,int)=0.20 (WE-C)/0.56 ('99).
- Phase↔stat naming cross-checked against ROM stat-name strings (CLAUDE.md @0x0ED5B4) and the AI
  Master-Architecture phase list.

---

## 9. Open questions
1. Exact arithmetic of v(tick): how external[phase], Speed cap, Sharp accel, and the §4 coefficients
   combine (product? weighted sum? piecewise?). Needs disasm or trace.
2. Exact distance-multiplier semantics: is 0x10F210's curve a base-speed scaler, a finishing-time
   normalizer, or a stamina-budget scaler? (9 keys vs 12 values — map the indexer.)
3. Dirt aptitude curve: linear vs banded 0–255 → surface penalty.
4. HIDDEN-X (rec +23/+24): does it index the distance machinery / a distance-aptitude per horse?
5. Stamina drain model and the "running on empty" speed penalty (is 0x53928 the drain curve?).
6. Whip math: timing window width, sharp-scaled boost, trust/energy cost (33 whip messages @ ROM, but no
   numbers yet).
7. Condition/Trust/Hearts → race multiplier magnitudes (card T2[44]/[36]/[37]).
8. Which §4 pool is per-style vs per-personality vs per-phase (0x7C258 twin and 0x102760 rows are the
   prime style-coefficient candidates — confirm by tracing their loaders).

---

## 10. Tool ideas this unlocks
- **Empirical race simulator (v0):** 6-phase machine using decoded stats + confirmed distance table +
  surface/dirt + style-as-behavior. Good enough for an odds/"who wins" predictor and a recreation
  prototype, even before the exact formula. Ground-truth/tune via MAME race captures.
- **Distance-aptitude / "best trip" calculator:** combine a horse's externals (which phases it's strong
  in) + the 0x10F210 distance curve to recommend the optimal race distance per horse.
- **Pool-loader cross-reference tool:** scan the SH-4 code for `mov.l @(disp,PC)` loads that resolve to
  the §4 pool addresses → auto-locate the race-math functions (the bounded entry point for full disasm).
- **MAME race-trace harness:** scripted watchpoints on stat + speed RAM during a race to read the formula
  live (the high-ROI path; pools in §4 confirm the constants).
- **Opponent strength/odds ranking:** with phase weights + internals + distance curve, compute a true
  per-distance power ranking of the 244 CPU horses (extends the `horse-stats` "difficulty calculator").
