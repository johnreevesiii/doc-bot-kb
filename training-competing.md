# Myth #3: "Competing/strong horses play differently in Rev D" (Rev C->D nerf)

ROMs: Rev C epr-22336c.ic22 (CRC 0x50053F82) vs Rev D epr-22336d.ic22 (World
Edition EX, CRC 0xE6C0CB0C). Both 4 MB, RAM=file+0x0C020000. Investigated
2026-06-23 with docre + diff scripts. **Status: OPEN, PAUSED 2026-06-23.**

## >>> RESUME HERE (stopping point + next moves) <<<
UPDATE 2026-07-04 -- CURVE FN DEFINITIVELY RULED OUT (was the open structural lead).
  The curve fn (C 0x0C064118 / D 0x0C064280) is 100% identical C vs D: the +84>threshold
  gate, the strength-band->coefficient assignment, AND every coefficient. The threshold table
  it reads is filled by an INSTRUCTION-IDENTICAL builder (C 0x0C06403A / D 0x0C0641A6) from
  BYTE-IDENTICAL source data (float-src p5 @0C12F084/0C1308E4, struct-arr p6 @0C12EFF4/0C130854;
  only a runtime index-ptr's static image differs = relocated work-RAM noise). The banding is
  byte-identical in BOTH revs -- horseStruct[+40] band -> horseStruct[+88] coeff:
    >85 -> -1.8 ; 75-85 -> -1.6 ; 60-75 -> -1.4 ; 40-60 -> -1.1 ; <=40 -> -0.6.
  CAUGHT-FALSE-POSITIVE (same trap the 06-23 pass warned of): the -1.6/-1.4/-1.8 triple IS real
  float coeffs (NOT a pointer as 06-23 guessed) -- but Rev C ALSO has all three (0xc064264=-1.8,
  0xc06426c=-1.6, 0xc064270=-1.4); the recompile merely REORDERED the pool. No difference.
  => The nerf is NOT in the curve. It is UPSTREAM (how horseStruct[+40] strength + [+84] are
     computed -- where "total internal strength" would enter) or DOWNSTREAM (the consumer of the
     [+88] coefficient). horseStruct arrives in r4 at the curve fn.
  ALSO 2026-07-04 -- ALL located race-formula coefficient pools byte-identical C vs D. Diffed all 9
  from race-formula.md §4 (file offsets): distance@0x10F210, phase/speed@0x46168, falloff@0x53928,
  style-twinA@0x7C258, style-twinB@0x7C3C8, weight-set@0x828BC, style-mtx@0x102760, wt-penalty@0xE7CA8,
  pos/cond@0x102C00. Two APPARENT diffs, both cleared: (i) 0x7C258 = the real pool is only ~9 floats
  (0.6,0.09,0.3,0.25,1.3,0.195,0.65,0.845,0.676) identical; the "diff" was a 1-ULP wobble in an adjacent
  GARBAGE float past the pool. (ii) 0x828BC = the known "2.0 rebuilt inline" refactor (D drops the 2.0
  literal + shifts 0.4/0.3/3.0 up one; same values). => THE ENTIRE LOCATED STATIC DATA LAYER (curve fn +
  its table + all race pools + distance table) IS IDENTICAL C vs D.
  REFRAME (John 2026-07-04): internals are MAX CAPS (hit late-career), so NOT the first-8-races effect.
  The real signature is EXTERNALS -- high-external horses dragged in the first ~8 races in Rev D; "competing
  doesn't matter" = the COMPETING external = the MID-RACE phase (race-formula.md §1). So the nerf is a
  CODE-LOGIC difference in the race-math (same coefficients, different arithmetic/branches), or in how an
  external stat is computed/fed -- NOT a data/coefficient retune.
  STATIC SWEEP DONE 2026-07-04 -- DEFINITIVELY ALL-IDENTICAL. Disassembled + diffed C vs D:
   - penalty curve fn (C 0x0C064118 / D 0x0C064284 [entry; 0x0C064280 = an embedded literal, recompile put
     mov#116 before the prologue -- near-false-positive #3, watch this]): identical (gate+band+coeffs+tail).
   - its CALLER (C 0x0C064874 / D 0x0C064B08, both `bsr`): identical gating block -- race_counter<7 AND
     horseStruct[+136]!=0 -> call penalty_fn(r4=horseStruct). byte-for-byte.
   - phase/speed math (before pool 0x0C066168/0x0C066394): identical cap-ladder (85/80/70/60/50) + style==1/2
     multipliers + coeffs (mova targets reloc +0x22c, same values).
   - all 9 §4 coefficient pools + distance table + table-builder+sources: identical.
   => NO static code-logic difference exists in the race engine or the first-8-races penalty. Same code,
      recompiled. The Rev D effect is in the INPUTS (how horseStruct[+40]/[+84]/[+136] are grown/computed
      upstream) or is runtime-emergent. Blind static needle-hunt of the growth code is LOW-YIELD (6/6 candidate
      fns identical this session).
  RECOMMENDATION (supersedes earlier NEXT items): switch to the LIVE Flycast trace (race-formula.md §7 opt-2).
  In an early-career race with a high-EXTERNAL horse, matched conditions, both ROMs: watch horseStruct[+40],
  [+84], [+136] + the per-tick speed word. The ONE field that diverges C vs D points straight at the differing
  upstream computation -- far cheaper than disassembling the whole growth system hoping to spot 1 instruction.
  (Harness: _sh4 GDB-Flycast, DOC_RACE_FORMULA_SH4.md; runtime base +0x20000, per-horse stride 0x2A0.)
  TRACE SCRIPTS WRITTEN 2026-07-04 (knowledge/sh4-race-formula/trace/): watch_penalty.py (Z0 BP at the
  penalty fn, logs r4 struct operands +0x28/+0x54/+0x74/+0x88/+0x58 per hit -> penalty_Rev{C,D}.csv);
  watch_field_writer.py (Z2 write-watch on a diverging field -> logs the writer PC to diff C vs D);
  WATCH_PENALTY_README.md (the 5-step workflow). PREREQ: interpreter mode (Dynarec.Enabled=no) + a real
  high-external early-career card loaded IN PERSON. Run watch_penalty C then D, diff, then writer-watch.

WHERE WE STOPPED: confirmed Rev D differs in PLAY (John: total-internal-strength
penalty -- one internal at ~42 in ANY slot beats 60/60/60; worst in first ~8
races). Static byte-diff did NOT find it and is the WRONG tool here: Rev D is a
full recompile, and ROM literal pools hold relocated RAM/code pointers that look
like float coeffs -- I burned 2 false leads (the 0x828BC "2.0" no-op refactor,
and a "+116/+84 penalty curve" whose table is actually RUNTIME-BUILT in work-RAM,
not in the ROM). Net: stat data + named race pools are equivalent C==D; no clean
static coefficient diff explains the effect.

NEXT MOVES (pick up here):
1. PRIMARY -- LIVE Flycast (only thing that can read runtime-built values):
   a. 2x2 effect test: build 2 cards, 60/60/60 vs one-internal-42 (any slot),
      identical otherwise; race matched conditions in BOTH ROMs; ~5-10 trials.
      Predict: Rev D -> 162-card beats 180-card; Rev C -> equal/180 wins.
   b. Mechanism trace: watch curve fn (Rev D 0x0C064280), runtime table at
      0x0C21D000, horseStruct[+84] & [+116] during early-career races; and/or
      find where the 3 internals are summed + gated.
2. SECONDARY -- targeted static (logic, not bytes): find code that reads all 3
   internals, sums them, gates on the total (new in Rev D vs Rev C). Needs the
   runtime internal struct offsets first.
3. When mechanism is confirmed: "make Rev D play like Rev C" = neutralize the
   total-strength penalty (patch the gate/curve). Then write the myth page.
KEY ADDRESSES: curve fn Rev C 0x0C064118 / Rev D 0x0C064280; runtime table ptr
slot Rev C 0x0C064238->0x0C21A234, Rev D 0x0C0643A4->0x0C21D000; struct fields
+116 (index, capped 7 = career-race-counter?), +84 (compared float), +40,+140.
LESSON: do NOT trust float-runs from literal pools on this recompile; verify any
"pool" is real ROM data (not a pointer) before claiming a C/D difference.
## >>> end resume block <<<

## The claim (from play, John)
Rev D feels night-and-day vs Rev C for strong horses in the first ~8 races.
Specific tell: internals max at 60/60/60 in both, but in Rev D a horse RACES
better when ONE internal is ~42 instead of 60 -- and the 42 can sit on ANY of
the three (stamina/speed/sharp), in any arrangement. So it is NOT a per-stat
(Sharp) penalty: it is a TOTAL-internal-strength penalty (sum ~162 races better
than ~180). Rev D penalizes being too strong overall; Rev C has no such penalty.

## What is NOT the difference (ruled out)
- 244-horse stat table incl. all 6 externals (start/corner/oob/competing/
  tenacious/spurt): byte-identical C==D.
- Distance->multiplier table: byte-identical.
- 8 of 9 previously-located race pools: byte-identical. The 9th (0x828BC "2.0")
  is a no-op refactor: Rev D rebuilds 2.0 inline (fldi1+fadd). The per-external
  growth dispatch + 6-loop fn is instruction-identical C vs D.
  => the FIRST-PASS conclusion ("engine identical") was WRONG: it only compared
  those 9 named pools, not the whole race code.

## A structural lead (but NOT statically comparable)
Consumer fn (logic IDENTICAL in both revisions): Rev C @0x0C064118, Rev D
@0x0C064280. It reads horseStruct[+116] as an index, increments it and CAPS it
at 7 (`mov #7; cmp/ge; add #1`), looks up table[index], compares to
horseStruct[+84] (float) via fcmp/gt, and branches. Also touches +40, +140.
- +116 behaves like a CAREER-RACE COUNTER capped at ~7-8 (zeroed in care init
  0x0355EE) -- which lines up with John's "first ~8 races" tell.
- BUT the table is NOT a ROM coefficient pool. `mov.l 0xc064238,r0` is a POINTER
  load: r0 = *(0x0C064238) = 0x0C21A234 (C) / 0x0C21D000 (D), i.e. a pointer into
  the 0x0C21xxxx WORK-RAM region. The curve is BUILT AT RUNTIME; the static ROM
  image shows nan/0 there. So this curve's values CANNOT be compared statically;
  only the (relocated) RAM address differs.

## CORRECTION / lesson
Two earlier "candidate differences" were ARTIFACTS of misreading literal pools as
coefficient tables: (a) the 0x828BC "2.0 removal" is a no-op refactor; (b) the
"Rev D prepended -1.6/-1.4/-1.8" curve was the literal-pool slot bytes / a RAM
pointer, not a coefficient array. On a full recompile, ROM literal pools hold
relocated RAM/code pointers that masquerade as float coefficients. Static
byte/float diffing is NOT reliably converging for this myth.

## What is actually established
- The thing players call the skill data (stat table, 6 externals) is identical.
- The named race pools are equivalent.
- No clean static coefficient difference has been confirmed that explains the
  effect. The one structural lead (race-count-gated curve fn) reads runtime RAM,
  so its C-vs-D difference (if any) is not visible in the static image.
- John's in-play evidence (total-internals penalty in Rev D; 60/60/42 in any
  arrangement beats 60/60/60; night/day in first ~8 races) is strong and stands.

## Open items / the reliable path
1. LIVE Flycast trace is the right tool (the race engine always needed it). See
   "How to TEST" below. Watch the curve fn (Rev D 0x0C064280) + the runtime
   table at 0x0C21D000 + horseStruct[+84]/[+116] during early-career races.
2. Static (higher-effort, lower-reliability): trace the BUILDER that fills the
   runtime table at 0x0C21A234 (C) / 0x0C21D000 (D); if the builder logic differs
   C vs D, that is the mechanism. Compare LOGIC, not literal-pool bytes.
3. Identify +116 (career-race counter?) and +84 by their writers (candidates:
   0x0355EE care-init zero; 0x057966/0x058A9A/0x05905A).

## How to TEST the theory (what John asked)
EMPIRICAL (definitive, Flycast) — the 2x2:
- Build the SAME horse/card twice: Card A = 60/60/60, Card B = 60/60/42
  (only Sharp differs; identical externals/style/everything else).
- Race each card under identical conditions (same track/distance/field; repeat
  N times for RNG, or reuse a seed) in BOTH Rev C and Rev D. Record finishing
  position/time.
- Prediction if the theory holds: Rev D -> B (42) beats A (60); Rev C -> A>=B
  (no penalty). A clean 2x2 isolates "Rev D penalizes high Sharp."
- Sharpen with Flycast memory watch: watch the curve fn (Rev D 0x0C064294) and
  horseStruct[+84]/[+116] during a race to see the penalty branch fire for the
  60/60/60 horse but not the 60/60/42 one.
STATIC (mechanism): finish open items 1-2 above.

## Repro
    python find_penalty.py       # comprehensive C/D coefficient sweep
    python find_penalty2.py      # 0x044xxx region + consumer
    python diff_curve.py         # the +116/+84 curve: identical logic, diff table
    # Rev C curve fn: docre.load();      docre.dis(0x0C064118,0x40)
    # Rev D curve fn: docre.load_rev('D'); docre.dis(0x0C064280,0x40)
