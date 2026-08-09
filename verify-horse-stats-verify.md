# Adversarial Verification — `horse-stats.md` (Racing Horse Record full byte decode)

Verifier pass date: 2026-06-03. Method: independent re-extraction of bytes from all four
real `.ic22` ROMs via python3, cross-checked against the four `DOC_COMPLETE_HORSE_DATABASE_*.md`
ground-truth tables (244 rows each). Helper scripts in `C:/Users/johnr/AppData/Local/Temp/verify_hs*.py`,
`xcheck.py`, `xver.py`, `hidden.py`, `beer.py`, `names.py`, `examples.py`, `roster.py`, `roster.py`.

## VERDICT: SOLID. The decode holds up byte-for-byte. No material errors found.

Every load-bearing structural and semantic claim reproduced exactly from the binaries. Two tiny
wording imprecisions noted below; neither changes any offset, width, or value.

---

## Confirmed claims (re-extracted from bytes)

### Geometry (all 4 ROMs)
- All four ROMs are exactly 4,194,304 bytes. CONFIRMED.
- Record start / stride / count:
  - drbyocwc `0x108E03`, stride 32, 244 — CONFIRMED
  - derbyocw `0x10A14B`, stride 32, 244 — CONFIRMED
  - derbyo2k `0x10AD1B`, stride 32, 244 — CONFIRMED
  - derbyoc `0x0F6902`, stride **28**, 244 — CONFIRMED
- Name tables: WE-C `0x10AD50` stride 18, WE-D `0x10C098` stride 18 — CONFIRMED.
  Decoded #1="Gold Queen", #2="First Star", #16="Kowloon Volcano", #18="Victory Lap",
  #22="Lucky Girl" (matches §3 examples exactly).
- derbyo2k name region is non-ASCII (zero printable bytes after stat table) — consistent with the
  doc treating it as JP/EUC-JP.

### Column structure (per-column min/max/distinct over 244)
The 32-byte layout's CONST-pad columns and varying columns match the doc precisely on all three
32B versions: pads at +0,+4,+6,+7,+15,+17,+18,+19,+20,+26,+27,+28; id/id2 at +2/+3 (1..244,
244 distinct); dirt +5; grade +8 (0..3); externals +9..+14; HIDDEN-A +1 (0..2); HIDDEN-B +16 (0..2);
style +21 (vals {0,1,2,3,7}); coat +22 (8 values); HIDDEN-X +23/+24; idecho +25; internals +29/+30/+31.
The 28-byte derbyoc layout matches the doc's §4 map (pads at +3,+5,+6,+8,+15,+16,+23,+27; HIDDEN-A +0;
id +1/+2; dirt +4; grade +7; ext +9..+14; HIDDEN-B +17; style +18; coat +19; HIDDEN-X +20/+21;
idecho +22; internals +24/+25/+26). CONFIRMED.

### Semantic fields — cross-checked vs the 244-row DB for ALL FOUR versions
Parsed each `DOC_COMPLETE_HORSE_DATABASE_*.md`, mapped grade {Ungraded:0,G3:1,G2:2,G1:3},
style {Front-runner:0,Start dash:1,Last spurt:2,Stretch-runner:3,Almighty:7},
coat {Chestnut:192,Black:193,Brown:199,Bay:202,Dark Gray:204,Light Gray:207,Special:222,Default:0},
and compared every byte field to every horse:
- grade, dirt, style, coat, start, corner, oob, comp, tenac, spurt, stam, speed, sharp,
  id, id2 — **0 mismatches across all 244 horses in all 4 versions.** CONFIRMED.
- Grade meaning (0=Ungraded..3=G1) at +8 (32B) / +7 (28B): 244/244 clean. CONFIRMED.
- Running style at +21 (32B) / +18 (28B), deterministic, value 7=Almighty present only in 32B
  builds; '99 max style is 3 (no Almighty). CONFIRMED.
- Coat enum at +22 / +19: every byte in {0,192,193,199,202,204,207,222}. CONFIRMED.
  Spot examples: #1=207 Light Gray, #6=204 Dark Gray, #16=222 Special, #18=192 Chestnut,
  #22=199 Brown. CONFIRMED.

### §3 example record (WE-C horse #1 "Gold Queen")
Re-extracted: id=1, dirt=168, grade=3(G1), ext=[44,35,19,32,40,46], style=2(Last spurt),
coat=207(0xCF Light Gray), hiddenX=0xA00E (lo=0x0E,hi=0xA0), idecho=1, int=[23,37,48]. EXACT MATCH.

### Hidden-field distributions
- 32B HIDDEN-A (+1): {0:175,1:62,2:7}. HIDDEN-B (+16): {0:200,1:37,2:7}. CONFIRMED real, small, 0–2.
- 32B HIDDEN-X distinct (lo,hi) pairs = **133** — matches doc exactly.
- 32B HIDDEN-X high-nibble clusters: 0xA0=186, 0x30=30, 0xC0=9, 0xF0=7, 0x00=12 — matches doc exactly.
- 28B HIDDEN-A (+0) {0:173,1:64,2:7} and HIDDEN-B (+17) {0:206,1:35,2:3} — match doc's §4 figures exactly.

### Cross-version diffs
- WE-C vs WE-D: **byte-identical for all 244** (empty diff). CONFIRMED.
- WE-C vs DOC-2000: **exactly 22 differing records** at #[13,37,38,78,100,101,109,110,111,115,116,117,
  122,123,156,157,160,161,194,195,200,223]. CONFIRMED.
- Per-offset diff counts on the 22: +16 (HIDDEN-B) changed in **9** records; +23 in 8, +24 in 5 (the
  doc's "cols 23/24 differ in 8/5"); also stat cols +9..+14, +29..+31 changed. CONFIRMED.
- Horse #13 differs ONLY at +16 (0→1). CONFIRMED exactly.

### Beer-Experiment framing proof (the most important structural claim)
Diff of `beer_effects_test.ic22` vs base `epr-22336c.ic22`: exactly **12 differing bytes**, all in
`0x1671FC..0x16722D` (the food/item table). **ZERO** differing bytes inside the racing table
`[0x108E03..0x10AC83)`. CONFIRMED — this validates the doc's central framing that the 244-record
table is the static CPU-opponent roster, not the player horse (player-mutable stats live on
card/nvram, and feeding/beer only touches the food table).

---

## Minor imprecisions (do NOT affect any offset/value)

1. **idecho "mod 256" wording.** Doc §3 row +25 says idecho `== horse_number mod 256 (so #244 → 0)`.
   The *byte value* is verified correct: for #1..#243 the byte == horse#, and #244's byte is **0**
   (not 244). But 244 mod 256 = 244, so the arithmetic explanation is imprecise. The observed byte (0
   at #244) is real; a cleaner description is "a 1-byte echo of the id that holds 0 for the 244th
   record" (looks like a 0-based wrap / off-by-one in the table generator, not a true mod-256). The
   conclusion (it's an id echo) is sound; only the formula is loose. Confidence the byte=0 at #244: 1.0.

2. **§3 raw-record example dropped one pad byte.** The printed hex string in §3 (`00 01 0101 00 a8 ...`)
   is 31 bytes; the true 32-byte record is
   `00 01 01 01 00 a8 00 00 03 2c 23 13 20 28 2e 00 00 00 00 00 00 02 cf 0e a0 01 00 00 00 17 25 30`
   (one extra `00` in the +15..+20 pad run). The decoded VALUES the doc lists are all correct; only the
   transcribed byte string is short by one pad byte. Cosmetic.

3. **"'99 is a genuinely different roster."** Verified the core stat block (dirt/grade/ext/style/coat/int)
   is identical for **92 of 244** horses between WE-C and '99 (e.g. #1 Gold Queen identical), and differs
   for the other 152 (e.g. #2, #3 differ in grade/style/coat). So it is better described as a
   *substantially rebalanced / partially overlapping* roster than a wholly different one. The doc's intent
   ("different coats/styles per id") is correct; "genuinely different roster" slightly overstates it.

## Not independently re-derived (left as the doc's open questions; no contradiction found)
- Semantic *labels* for HIDDEN-A / HIDDEN-B / HIDDEN-X remain unconfirmed (need an in-game opponent
  stat/encyclopedia readout). The doc correctly marks these 0.5–0.6 confidence. I confirmed they are
  real per-horse fields (vary across horses, HIDDEN-B independently rebalanced in 2000), but their
  meaning is still unproven — agree with the doc.
