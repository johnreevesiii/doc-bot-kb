# Derby Owners Club RE — Open Questions, Shaky Claims & Ranked Next Steps

A completeness critic's pass over all 13 subsystem findings and their verifier verdicts.
This is the "what we still don't know" ledger and the prioritized RE plan to close it.
Source docs live in `C:/DerbyOwnersClub/_core/areas/*.md` (+ their `*VERIFY*.md` / `*VERIFICATION*.md` companions).

Verifier verdicts at a glance: horse-stats 96% (solid), us-card 93%, game-text 93%, appearance 93%,
jp-card 90%, breeding 88%, tracks-races 90%, items-feeding 90%, nvram 90%, version-diff 90%,
derived-attrs 90%, race-formula 90%, architecture 90%. Nothing is below 88%, but several of the
90% verdicts carry HEADLINE-level corrections (see §2) that matter more than the score implies.

---

## 0. The one thing that actually blocks the prize: the race-performance formula

Everything else is "nice to have." The race-performance formula is the keystone, and it is the ONLY
subsystem that is explicitly **not byte-decodable from data** — it is SH-4 FPU code. Current status is
"located, not decoded": the coefficient pools and the distance→multiplier table are found, the stat
*roles* are inferred from statistics, but the actual per-tick arithmetic `v(tick) = ...` is unproven
(product vs weighted-sum vs piecewise is a guess). No amount of further table-dumping closes this. It
requires either a **MAME/Flycast memory trace** or **SH-4 disassembly**. See §3 for the ranked plan;
this is steps 1–3.

Hard dependencies that block the formula specifically:
- **HIDDEN-X (rec +23/+24)** semantics are unknown (horse-stats, derived-attrs, race-formula all flag
  it). race-formula §3 hypothesizes it indexes the distance machinery / encodes per-horse distance
  aptitude, but this is unconfirmed. If HIDDEN-X is a distance-aptitude key, it is a formula input and
  must be decoded before the formula is complete.
- **Dirt aptitude (rec +5, 0–255) → surface penalty curve** is direction-only (high dirt = keeps speed);
  the curve (linear vs banded) is unknown.
- **The §4 coefficient pools have ~0.4 confidence on each semantic label.** We know they're race pools
  (code-adjacency + version-stability), not what each row does. That mapping only comes from tracing
  their loaders.

---

## 1. STILL UNKNOWN — never investigated or genuinely open

### Race formula (the blocker)
- Exact arithmetic combining `external[phase]`, Speed cap, Sharp accel, and the §4 coefficients into
  per-tick velocity. (product / weighted-sum / piecewise — all three are live hypotheses.)
- The 9-key-vs-12-value index mapping in the distance→multiplier table @0x10F210. Why 9 distances but
  12 multipliers? Which multiplier binds to which distance, and what are the extra 3 for (sub-1700m
  sprints? a superset curve?).
- Whether the distance curve is a base-speed scaler, a finishing-time normalizer, or a stamina-budget
  scaler. Three different semantics, none ruled out.
- Stamina drain model and the "running on empty" speed penalty. Is 0x53928 the drain curve?
- Whip math: timing-window width, sharp-scaled boost magnitude, trust/energy cost. 33 whip messages in
  ROM but zero numbers recovered.
- Condition / Trust / Hearts → race-multiplier magnitudes (card T2[44]/[36]/[37]). Labels are also
  shaky (see §2 us-card).

### Hidden/unlabeled stat-record fields (feed the formula or appearance)
- **HIDDEN-A (+1)**: 0/1/2 class/grade-aux flag — meaning unknown. (architecture confirms +1 is a real
  3-value field, not padding; version-diff lists it among the 5 undecoded variable bytes.)
- **HIDDEN-B (+16)**: real field (values 0/1/2, distribution differs WE vs derbyo2k), meaning unknown.
  architecture explicitly corrected the seed claim that +16 is constant-0.
- **HIDDEN-X (+23/+24)**: 16-bit field, high-byte clusters (0xA0/0x30/0xC0/0xF0). The single
  highest-value unknown because it is the leading candidate for a per-horse distance/surface aptitude
  composite (race-formula §3). Could also be a pointer/index into a portrait/voice/AI/pedigree table.

### Breeding inheritance (the second-hardest unknown)
- The real SH-4 breeding routine has **never been disassembled.** The entire foal-inheritance model in
  breeding-system.md is the *community simulator's* parent-averaging heuristic, which provably ignores
  the name+44 composite — so the real ROM rule is almost certainly richer and is currently unknown.
- name+44 composite bytes b0/b1/b3 (growth? grade? distance aptitude? hidden line/affinity flags?) —
  the gate to the exact inheritance rule, undecoded.
- name+45..47 3-byte composite (derived-attrs) — separate, also undecoded.
- name+36 "ac" byte: dirt-aptitude vs personality vs both is an **unresolved conflict** between two docs
  (breeding-system says dirt aptitude by comment correlation; an earlier source-of-truth doc said
  personality). Empirical correlation only, never confirmed against an in-game screen.

### JP on-card / cabinet stats (a genuine "we cannot answer this in emulation" wall)
- Whether real linked DOC 2000 hardware persists career stats in master BBSRAM keyed by the lead-byte
  ID. Emulation never persists (multiboard master reset), so this is **unanswerable without hardware.**
- The CREATE recipe for a brand-new JP card (0x25–0x27 lead-ID scheme + 0x43–0x44 trailer the cabinet
  accepts). Edit path solved; create path unsolved.
- The exact 0x43–0x44 trailer algorithm, and whether the cabinet validates it at all (untested:
  does a zeroed trailer get accepted?).
- derbyoc (DOC '99) card never captured — format assumed same as DOC 2000, unconfirmed against bytes.

### NVRAM / cabinet career data (blocked on a non-factory save)
- Whether a cabinet with real registered player cards writes a per-card career block into the unused
  0x2998–0x8000 span. Current SRAM is near-factory NPC demo data; **need a played, non-factory SRAM.**
- Money-record flag0 (+0x00, 0–7) and flag1 (+0x01, 0xc0–0xde) meaning.
- Header +0x08 LE32 confirmed as the resume/program-section counter (the long-parked battery-resume
  problem) — strongest candidate but unproven.
- JP cabinet nvram layout — no JP nvram present in the set at all.

### Things essentially not investigated
- **DOC II (8-satellite, derbyoc2) tables**: its mater/breeding table, track/race tables, and stat
  record layout were not in the provided 4 ROMs. Breeding, tracks-races, and version-diff all flag this
  as a gap. Entirely uninvestigated.
- **The binary race-schedule table** that holds real grade/surface/distance/prize/month per race and
  binds each G1 to a physical course. tracks-races proved the track/G1 tables are *display strings only*;
  the actual schedule binary is undecoded. Candidate offset noted (derbyo2k 0x0CAD7B+) but never decoded.
- **0x300000 second SH-4 program image** (architecture): true second bank vs boot mirror vs
  relocated/overflow code — needs a code-diff vs 0x1000, not done.
- **JP dialogue layout**: pointer/index table vs flat NUL-packed (EN is flat) — not determined.
- **JP name-entry profanity/banned-words filter** — EN ASCII list found, no JP analog located.
- The SH-4 food-list builder routine (walks to idx=0 terminator vs count constant vs separate food-ID
  array) — needed to surface beer cleanly without the boot crash. Not disassembled.

---

## 2. SHAKY CLAIMS — need a second pass (corrections that change conclusions)

These are ordered by blast radius. The first few are HEADLINE corrections that invalidate tool plans.

1. **[version-diff / architecture — HEADLINE, HIGH IMPACT] The 32-byte racing stat table is NOT
   byte-identical across Rev C, Rev D, AND DOC 2000.** Rev C == Rev D is byte-exact; Rev C vs derbyo2k
   has **22 differing records of 244** (first divergence at record 12), spanning real stat bytes
   (dirt +5, externals +9..14, flagB +16, coat +22, HIDDEN-X +23/24, internals +29..31). Both the
   version-diff doc and the architecture doc made the "all three identical" claim; architecture only
   spot-checked rec 0/1/243 (which happen to match). **Action: any "unified editor writes to all three"
   or "patch transposer applies a Rev C edit to o2k unchanged" tool is UNSAFE for ~22 horses.** Re-verify
   with a full 244-record diff before building those tools. Stats are ~91% shared with o2k, not 100%.

2. **[appearance — RESOLVED, was a convention mismatch, not a defect]** The CPU coat byte is one physical
   byte under two address conventions: **record-start +22** (doc-core) = **recBase+13** (DOC-ROM-Studio,
   recBase = record-start+9). Same absolute byte (WE-C #1 = 0x108E19 = Light Gray), 244/244 vs the DB; '99
   28-byte = record-start +19. The "+13 NOT +22" framing conflated the bases (under recBase, +22 = sharp;
   under record-start, +31 = sharp). doc-core uses record-start +22/+19 and is correct; no tool is corrupted.

3. **[breeding-system — HIGH IMPACT] Record counts and ordering are wrong.** ROM mater counts are 167
   (Rev C) / 177 (Rev D), NOT 168/178 (the 168/178 + 84-sires/84-dams split comes from the JSON, not the
   ROM index field). The pool is ONE contiguous block, not two physically separate sire/dam arrays
   (claimed "dam base" = base + 84*60 = record #85, index increments continuously with no restart). And
   the "JSON id k == ROM record k" round-trip is wrong on ORDERING — **reconcile by NAME, not index**
   (JSON is ~alphabetical, ROM is game order; 161/167 names match, 156/161 of those match stats). Also:
   EN↔JP name+44 composite is NOT byte-identical (only JP↔JP is stable). **Action: re-anchor the breeder
   record editor and foal predictor to name-keyed joins; fix the index/count claims before shipping.**

4. **[nvram — MEDIUM/HIGH IMPACT, data-corruption risk] Region-2 money leaderboard start is 0x15f4 /
   0x1c34, NOT 0x1634** (the doc even cites the name at 0x1600 = 0x15f4+0xc, an internal contradiction).
   Region delta is +0x13c4, not +0x13d0. Copy-2 metadata constant is 0x1680 (5760), not 0x168000 (byte
   order reversed). "Header A at 0x00 mirrored verbatim at 0x100" is FALSE (14 byte positions differ).
   **An editor writing region 2 at 0x1634 would corrupt the backup save.** Fix before any NVRAM editor.

5. **[race-formula — MEDIUM IMPACT, weakens the headline evidence] Two §4 proofs are wrong.** (a) The 12
   distance multipliers are NOT monotonically descending — index 2→3 goes 1.0323→1.0625 (ascending); it
   is a non-monotonic per-distance factor table. (b) The "SH-4 FPU opcodes immediately precede each pool"
   code-adjacency proof is wrong: the bytes immediately before each pool are more float32. The corrected
   evidence is *regional* FP-instruction density (33%/27%/20% within ±256B vs 2% at the isolated distance
   table). The pools are still very likely race pools, but the adjacency claim as stated does not hold —
   **this is the load-bearing evidence for "these are the formula's coefficients," so it must be restated
   precisely** and ideally re-grounded by actually finding the PC-relative loaders (which also advances §3).

6. **[architecture — MEDIUM IMPACT] Two over-stated structural claims.** (a) The function-pointer table
   @0x15729C is NOT a clean 200-entry function table at conf 1.0 — it's 295 contiguous in-range values,
   162 of which begin with word 0x0000 (data/padding), and at least one target is `rts;nop` not a
   prologue. It's a useful *seed list* at conf ~0.6. (b) +16 is a real field, not constant-0. Both affect
   the Ghidra seed script's reliability — treat the pointer table as candidates to triage, not ground
   truth.

7. **[us-card — MEDIUM IMPACT, label confidence] trust=a2[36] / condition=a2[44] / experience=a2[45]
   labels are unconfirmed** — on fresh cards a2[45] tracks a2[36], muddying them. These feed the
   condition_mod term of the race formula (§5 h()), so the labels matter to the formula too. Also a2[27]
   (0x6F) stable-id, a2[18–22] race/rest fields are low-confidence pending a *raced-horse* card. Plus a
   hex-cell typo (a2[26] hood at 0x70 not 0x73) — cosmetic but propagated into appearance too.

8. **[tracks-races — LOW/MEDIUM IMPACT, count errors] derbyoc course count is 29 not 30; WE G1 is 18
   real + NO NAME (19 string runs) not "19 races + NO NAME"; derbyoc G1 is 19 entries not 20.** Several
   WE course offsets are 1–3 bytes high (leading inline control bytes). Counts/offsets feed any track
   editor; fix before shipping the JP track editor.

9. **[derived-attrs / horse-stats — LOW IMPACT, confidence calibration] The floor(byte7/51) running-style
   mapping and the +1/+21/+24 enum meanings are inference, not byte-proven disassembly.** The docs flag
   this, but the confidence on these should be read as "strong inference" not "proven." The idEcho "mod
   256" framing is imprecise (it's a 1-byte id echo that wraps to 0 at record 244, not literal mod 256).

10. **[game-text — LOW IMPACT, mostly cosmetic] Several offset/count fixes** (JPSPEC textRegion is
    [0x0C0000,0x140000]; Rev D flat scan = 3702; copyright true starts 0x10E7EE/0x10FE22; %s count 187;
    leading attribute-byte family is {0x0F,0x03,0xFF}). The leading-0x0F/0x03/0xFF prefix meaning
    (display attribute? color? face index?) is genuinely unknown and worth a pass.

---

## 3. RANKED NEXT RE STEPS — concrete experiment / extraction / disassembly per step

Ranked by (impact on the race-formula prize) × (tractability). Steps 1–4 are the critical path to the
formula; 5–8 unblock formula *inputs*; 9+ are breadth.

### TIER A — crack the race formula (the keystone)

**1. MAME/Flycast race memory-trace harness. [highest ROI, unblocks the prize]**
- Experiment: run drbyocwc in MAME `-debug`. Identify the per-horse "current speed" and "current
  distance/position" RAM words for one CPU horse during a race (start by watching RAM during a known
  race and correlating monotonic increase = position, oscillating = speed). Set watchpoints on the horse
  stat block in RAM (cross-check via architecture's 0x15B1E0 RAM-pointer targets) and on the speed word.
- `trace` the race loop for one tick; capture which stat addresses are read and in what order before the
  speed word is written. This directly reveals product-vs-sum structure.
- Confirm against the located §4 constants (0x10F210 distance table, 0x53928 falloff, 0x7C258 style
  twins). If the traced multiplies use those literals, you have the formula's skeleton.
- Deliverable: the `v(tick)` arithmetic, replacing race-formula §5's `f()/g()/h()` guesses.

**2. Pool-loader cross-reference (static, complements step 1). [bounded, makes step 3 cheap]**
- Extraction: scan the SH-4 code region for `mov.l @(disp,PC)` instructions whose resolved target is one
  of the ~8 known pool addresses (0x10F210, 0x53928, 0x7C258, 0x7C3C8, 0x46168, 0x828BC, 0x102760,
  0xE7CA8, 0x102C00). Each hit is an entry into a race-math function.
- This also *fixes the §2 #5 shaky evidence*: it replaces the incorrect "opcodes immediately precede the
  pool" claim with the correct "here are the instructions that actually load the pool."
- Deliverable: a list of race-math function entry points to feed Ghidra (step 3).

**3. SH-4 disassembly of the race loop in Ghidra. [medium-hard, now bounded by steps 1–2]**
- Ghidra has an SH-4/SH-2 module. Load the 4MB image, seed it with the function entry points from step 2
  and the (de-rated, conf ~0.6) 0x15729C pointer table — *triage* the pointer table, don't trust it
  whole.
- Recover the per-tick position updater and the track-geometry reader (@0x0C8500). Decompile the speed
  computation to confirm/replace the empirical model.
- Deliverable: closed-form `v(tick)`, dirt-aptitude curve, stamina-drain model, whip math.

**4. Re-verify the cross-version stat-table identity with a full 244-record diff. [cheap, unblocks tools
   AND formula inputs]**
- Extraction: byte-compare all 244 records WE-C vs derbyo2k (not spot-check). Confirm the 22 divergent
  records and which fields differ. The divergent fields (dirt, externals, flagB, coat, HIDDEN-X,
  internals) are exactly the formula inputs — the divergence pattern is a free hint about what HIDDEN-X
  and flagB *do* (anything rebalanced between versions is a live gameplay input, not cosmetic).
- Deliverable: corrected version-diff headline + safe scope for the unified editor + a shortlist of
  formula-relevant fields.

### TIER B — decode the remaining formula INPUTS

**5. Decode HIDDEN-X (+23/+24), HIDDEN-A (+1), HIDDEN-B (+16). [directly feeds the formula]**
- Method that already cracked names/style: capture an in-game opponent stat / sire-encyclopedia /
  distance-aptitude screen for several horses with known, varied HIDDEN-X values; correlate the
  displayed attribute to the byte. Cross-reference with step 4's divergence list.
- Test the pointer hypothesis: check whether HIDDEN-X values are dense small integers (index into an
  84-entry pedigree or portrait/voice table) vs spread (a packed aptitude composite).
- Deliverable: HIDDEN-X label; confirm/deny "distance-aptitude key" for the formula.

**6. Dirt-aptitude curve (rec +5, 0–255 → surface penalty). [formula input]**
- Experiment: in MAME, race two horses identical except for the dirt byte across the 0–255 range on a
  dirt track; measure finishing-time delta. Fit linear vs banded. (Pairs naturally with step 1's harness.)

**7. Capture a played, non-factory cabinet SRAM + a raced-horse card. [unblocks nvram + us-card labels]**
- Experiment: play several races in the emulator with registered cards, dump SRAM, and diff against the
  factory save to see if a per-card career block lands in 0x2998–0x8000. Save a card *after* races to
  populate a2[18–22] (race/rest) and to confirm trust/condition/experience labels (a2[36]/[44]/[45]) —
  the latter feed the formula's condition_mod term.
- Also: edit a money-record flag0/flag1 in Demul and watch the attract-mode leaderboard render to decode
  them; edit header +0x08 and observe which race section the cabinet boots to (resume-counter test).

**8. Disassemble the breeding routine + decode name+44 / name+45..47 composites. [the second formula —
   inheritance]**
- Static: find the SH-4 routine that reads the mater table and writes foal stats; recover whether the
  name+44 composite and a "dominant parent" actually weight foal stats (vs the community averaging
  heuristic).
- Experiment: breed two known parents in the emulator repeatedly, log foal stats, and curve-fit against
  parent stats + composites to expose noise/dynasty-bonus terms.
- Resolve the name+36 ac conflict: capture ONE breeding-stock horse's in-game course-aptitude AND
  personality screens and match the byte (settles dirt-aptitude vs personality vs both).

### TIER C — breadth / lower priority

**9. Locate + decode the binary race-schedule table** (grade/surface/distance/prize/month per race;
binds each G1 to a course). Start at derbyo2k 0x0CAD7B+ (the `xx04 + 80 3f ff ff ff ff 05 24` rows).
This also gives the real EN↔JP G1 mapping (currently positional guess).

**10. Code-diff the 0x300000 second program image vs 0x1000** to classify it (second bank / boot mirror /
overflow). Cheap once Ghidra is loaded for step 3.

**11. DOC II (derbyoc2 / 8-satellite) extraction**: locate its mater table, track/race tables, and stat
record layout — entirely uninvestigated. Needs the DOC II ROM, which was not in the provided set
(blocked on acquiring it).

**12. JP CREATE-card recipe + trailer algorithm**: test whether the cabinet accepts a zeroed 0x43–0x44
trailer; brute the 0x25–0x27 lead-ID scheme for a never-seen horse. (Blocked on real hardware for the
authoritative answer; emulator can at least test acceptance.)

**13. Small label cleanups** (cheap, low value): leading 0x0F/0x03/0xFF string-attribute meaning
(game-text); feed effect columns 3–6 UI names (items-feeding); silk/hood palette hues (appearance);
the -n / E5-n name-substitution tokens (game-text). All observational, none block the formula.

---

## 4. Reminders for the next agent
- "Solid 90–96%" verdicts still carry HEADLINE corrections (§2 #1, #2). Read the `*VERIFY*.md`
  companions before trusting a doc's headline.
- Anything rebalanced *between versions* (stat-table 22 records, HIDDEN-X, food values) is by definition
  a live gameplay input, not cosmetic — use cross-version diffs as a free oracle for "what matters."
- The two true walls are hardware-only: real linked DOC 2000 BBSRAM career persistence, and physical JP
  card scans. Note them as out-of-scope-for-emulation rather than re-attempting in Flycast.
- Fix the data-corruption-risk offsets (§2 #4 nvram region-2, #2 appearance coat) BEFORE building any
  editor that writes those bytes.
