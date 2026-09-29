# Foal growth / base-vs-current stats - decode (epr-22336c.ic22)

Decoded 2026-06-30 from the foal-build `FUN_0c052b0c` (Ghidra `decomp/fn_0c052b0c.c`) cross-referenced
with the card layout (`_core/areas/us-card.md`). Answers: "does breeding set a growth ceiling?" Companion
to `foal_average.md` (the floor-average current/base internals) and `breeding_routine.md`.

## TL;DR / Verdict (confidence 0.8)
DOC has **no birth-fixed numeric stat ceiling.** Breeding sets the foal's **BASE** stats (floor-average of
the parents, internals capped at 45) and its **CURRENT** stats (at birth ~= base, with a rare RNG band
nudge). The horse then **GROWS at play-time**: CURRENT climbs from base toward the universal caps
(internals 60, externals 63) as it trains/races, and how far/fast it grows is governed by the inherited
**aptitude / growth-type fields** (the `+0x30`/`+0x32` masks, the `+0x5e` blend, and the HIDDEN-A/HIDDEN-B
flags), NOT by a number stamped at birth. (Corrected 2026-09-28: the growth-type part is an UNVERIFIED
hypothesis from these early notes. Stage 2 below rules out HIDDEN-A/B, and a later scan of all 207 card
bytes across 206 horses found no byte that predicts which stat grows or how fast (measured 2026-07-20).
Do not teach a per-horse growth type as fact. The "no birth-fixed ceiling" part stands.)

So the two stat tiers on the card are **base ("retire" naming is misleading) vs current**, not
current-vs-ceiling.

## The two tiers (FUN_0c052b0c writes both)
| record (foal @ runtime 0x0C21A5A8) | card field (us-card.md) | range | meaning |
|---|---|---|---|
| `+0x40/+0x44/+0x48` (u32, capped 0x2d=45, then +5 on Band 1) | card bytes 69/73/77 = a2[69/65/61] "current internals" | birth 0-50, then grows | **TRUE** internals: bred floor-avg + pedigree bonus (+5 on Band 1); what the horse races on; climbs with racing |
| `+0x74/+0x75/+0x76` (= base copy, then band noise) | card bytes 113/114/115 = a2[25/24/23] "retire internals" | 0-51 | **PRINTED** birth stats: frozen at birth; equal the true birth internals unless a band fired |
| `+0x6c..+0x71` (floor-avg & 0xf) | a2[28-33] "retire externals" (better name: birth bands) | 0-15 | **BASE** externals (band 0-15), fixed for life |
| `+0x62/+0x64/+0x63/+0x65/+0x66/+0x67` (= base*2 + (rand>>3 & 3) + offset, +2 if coat/cond) | a2[38-43] "current externals" | 0-63 | **CURRENT** externals (~= 2x base at birth; grows) |

Internals cap at 45 in the build (`if 0x2d < v: v = 0x2d`) = the BASE cap. Then one random byte moves the
two copies apart: Band 1 (~1 in 32) takes 12 off one PRINTED stat (or 5 off all three) AND adds 5 to every
TRUE internal; Band 2 (~1 in 32) adds 12 (or 5) to PRINTED stats under 40 only. So a fresh foal's true
internals = base, or base + 5 on all three. Externals: base is the 0-15 band; current is ~2x the band.
(Corrected 2026-09-28: the table above had the two internal rows' card bytes swapped, and this paragraph
said the noise moves the current internals up or down by up to 12. The record-to-card map is now
confirmed by the card parser in card-seed-trait-readers.md §1.2; see also the CONSUMER WARNING in
decode-foal_average.md.)

## Inherited growth-type / aptitude (early hypothesis, UNVERIFIED; corrected 2026-09-28)
`FUN_0c052b0c` also sets, from the parents:
- `+0x30` style mask via `FUN_0c053154`, `+0x32` aptitude mask via `FUN_0c05333e` (per-bit sire-odd /
  dam-even blend; already in the Breeding Lab model).
- `+0x5e` = a nibble blend of parent `+0x28` values masked by `DAT_0c05312c` (a "line/affinity" byte).
- `+0x28` (parent) = a "pedigree line" value lazily initialised from RNG if zero (`0x44 + (rand&7)*0x10 +
  (rand>>?&7)`), then blended into the foal `+0x5e`.
- HIDDEN-A (`+0x01`) / HIDDEN-B (`+0x10`) - candidate growth-type/aptitude flags (horse-stats.md, conf 0.5).

These fields - not a numeric ceiling - are what make one foal grow into a star and another plateau.
(Corrected 2026-09-28: UNVERIFIED hypothesis. A later scan of every card byte across 206 horses found no
byte that predicts growth, and the +0x30/+0x32 words are read elsewhere as coat and personality genes
(card-seed-trait-readers.md). Do not teach this as fact.) Their
exact effect on the growth CURVE (rate per race / training, and the per-stat max under a given growth-type)
is the remaining decode (it lives in the training/feed/race-progression code, items-feeding.md +
horse-stats.md, not the foal-build).

## Implications for the WC foal generator (docCard.ts) - corrected
- **Internals: CORRECT as-is.** A fresh foal has current ~= base = floor-avg (<=45). Our generator sets
  card current = retire = floor-avg, which is exactly a just-bred foal. (The earlier "no growth headroom"
  note was wrong - there is no birth ceiling; growth is play-time.)
- **Current externals: UNDERSET (minor).** We set card current externals (a2[38-43]) = base band (0-15);
  a real fresh foal has current externals ~= 2x base (the +0x62-67 formula). Fix: set a2[38-43] =
  clamp(base*2 + 1 + offset, 0, 63) (offset -1 for start, +1 others) and a2[28-33] = base. Apply + verify
  during the generator's in-game load test.
- **Growth-type: NEUTRAL (the real fidelity gap).** We inherit the aptitude byte (ac) but not the full
  `+0x5e`/line + HIDDEN-A/B growth-type. A generated foal will grow on a default/neutral curve rather than
  the bred one until those are decoded + written. This is the next decode (training-progression).

## STAGE 2 (started 2026-06-30) - the growth CURVE + growth-type

### Key redirect (avoids a dead end)
The obvious "growth-type" candidates **HIDDEN-A (+0x01) / HIDDEN-B (+0x10) are CPU-OPPONENT roster fields,
not player-horse growth** (`hidden_ab.md`): they live in the CPU roster at RAM `0x0C128C78` (244 opponents,
stride 0x20), and **CPU horses don't breed/age/grow**. Both get clobbered before the race formula anyway.
The only player-side `+0x01` reader works on a DIFFERENT record - the **player card-horse at runtime
`0x0C21C7EC`** (its own layout; `+5/+6` are `&63` stats). So: do NOT chase HIDDEN-A/B for foal growth.

### The real target = the PLAYER card-horse growth subsystem
Player growth (CURRENT internals/externals rising base->60/63) happens at play-time via feeding + racing on
the **player card-horse record (`0x0C21C7EC` family)** - the same record the foal-build (`0x0C21A5A8`) feeds
into and that serialises to the 207-byte card. This subsystem was scoped OUT of the race-formula work (which
only needed the CPU roster + speed math), so it's largely undecoded.

Known inputs already decoded:
- **Feeding deltas** (`items-feeding.md`): each food's bytes `+28..+34` = 7 unsigned per-stat boosts
  (speed/stamina/sharp + 4 growth columns + condition). The "feed routine" ADDS these to the horse; the
  apply routine's address isn't pinned yet. (Corrected 2026-09-28: which stats the food columns feed is
  DISPUTED: decodes read the first three as speed/stamina/sharp, one live test saw food move externals
  and not internals. Do not name the columns as fact; see items-feeding.md.)
- Player record stat fields: see `stat_attribution.md` + `us-card.md` (current vs base internals/externals).

### Next decode (stage 2 proper - needs a focused pass + a live capture)
1. **Locate the feed/train apply routine**: find writers to the player card-horse CURRENT-stat fields that
   add the `items-feeding` `+28..+34` deltas (search the feed-menu code for the food-record read + a
   per-stat add into the `0x0C21C7EC` record; clamp at 60/63).
2. **Locate the post-race growth**: find where finishing a race increments current stats (the growth per
   race), and what gates the amount (the per-horse growth-type field on the PLAYER record - candidate the
   `+0x28` line / `+0x5e` blend the foal-build set, or a card hidden byte - NOT the CPU HIDDEN-A/B).
3. **Live capture (the decisive step)** - turnkey runbook: `G:\My Drive\DerbyOwnersClub\PROMPT-live-growth-capture.md`.
   In GDB-Flycast, watch the player card-horse current-stat bytes;
   feed one food -> confirm the +delta + which column; race once -> measure the per-race gain; repeat across
   horses with different `+0x28`/`+0x5e` to isolate the growth-type effect + the curve toward 60.
4. Then inherit those growth-type fields in the foal generator so generated foals grow like bred ones.

### STAGE 2 RESULT - cross-sectional regression of real player cards (2026-06-30, conf 0.85)
Decoded all `stable_cards` in SQL (`get_byte(decode(card_b64,'base64'),off)`; current internals bytes
69/73/77, base/"retire" internals 113/114/115, races byte 103) over 235 deduped US horses. **The base-vs-
current model is CONFIRMED and the growth curve is quantified:**
- **234/235 horses have current >= base internals** (1 exception). Base IS the floor; current grows above it.
- **corr(growth, races) = 0.873**; growth = current - base internals (summed over st+sp+sh).
- **The curve is diminishing-returns / front-loaded:**

  | races | n | growth (Sigma internals over base) | per-race | avg current Sigma |
  |---|---|---|---|---|
  | 0 | 34 | ~0.4 | - | 100 (~base) |
  | 1-4 | 85 | +7.3 | 5.4 | 107 |
  | 5-9 | 35 | +19.2 | 2.7 | 120 |
  | 10-19 | 24 | +28.5 | 2.1 | 134 |
  | 20-29 | 51 | +35.2 | 1.5 | 138 |
  | 30+ | 6 | +35.5 | 1.0 | 139 |

- Net: a horse gains ~**+35 internal points total (~+12 per stat)** over a career, front-loaded
  (~5/race early -> ~1/race late), **plateauing by ~25-30 races** [REVISED 2026-07-29: fleet telemetry
  (4,722 consecutive-save pairs, card_history) pins the wall EARLIER — avg total gain/race: 2.14 (races
  0-4), 1.51 (5-9), 1.40 (10-14), 0.76 (15-19), 0.15 (20-24), ~0.00 (25+). **The practical wall is ~20
  races**; half of career growth lands in the first 10]; top single stats reach the 60 cap by
  ~10-19 races. Linear fit ~1.25 pts/race (corr 0.87), but the per-band rate shows it's clearly concave.
- **Implications:** (a) the foal generator's fresh foal (current=base) is correct - it will grow this curve
  in play; (b) "growth potential" is mostly universal (race-driven, capped at 60/stat), NOT a big per-foal
  ceiling - so the generator doesn't need to set a ceiling, only the (minor) current-externals + the
  growth-TYPE. (c) Per-horse growth-TYPE variance (the +0x28/+0x5e gate; later scan found no such
  card byte, UNVERIFIED, corrected 2026-09-28) is the residual to isolate -
  segment this same query by that field once the daily snapshot has captured a few weeks of per-horse
  deltas (longitudinal will separate growth-type far better than this cross-section).
- Caveats: cross-section confounds base/breeding + training/feeding + growth-type; dedup by
  (member,horse_name); feeding also adds points (not just racing) so per-race is an upper estimate.

### STAGE 2 alternative - PASSIVE data analysis (no GDB) - John's idea 2026-06-30
We have real player data in Supabase (furlong). Two passive routes to the growth curve:
- **Longitudinal (best, build going forward):** `card_stats_daily` already snapshots CAREER RECORD daily per
  horse (snapshot_date, member_id, birth_key, horse_name, earnings, w/p/s, races, g1) - but NOT internal
  stats. `stable_cards.card_b64` holds only the LATEST card (dedup-by-birth, no history). EXTEND the daily
  snapshot job to also decode `card_b64` -> current internals (st/sp/sh) + externals and store them. Over
  weeks this yields real stat-progression vs races-run per horse = an EMPIRICAL growth model, no emulator.
  Dual-use as player analytics. (Decode card_b64 server-side with the same offsets as card-facts.js /
  docCard.ts: current internals card[69/73/77]=st/sp/sh, ac card[146], current externals card[95..100].)
- **Cross-sectional (first pass NOW):** regress current stats on races-run / earnings across the existing
  ~200+ latest cards in `stable_cards` -> a population estimate of "stat gain per race" + the ceiling, even
  before longitudinal data accumulates. Pair with `card_stats_daily` to weight by activity.
This complements the GDB capture (which gives the exact per-event formula); the data path gives the
real-world curve + validates the formula against actual horses.

## Next decode steps (stage 1 residuals)
1. Resolve the band thresholds + masks: `DAT_0c052e8e/0c052e90` (up-band range), `DAT_0c05312c` (`+0x5e`
   mask), `DAT_0c05312e` - dump from the ROM literal pool (sh4dis / fn_0c052b0c PTR_DAT_* targets).
2. Decode the record->card serialiser (foal record 0x0C21A5A8 -> 207-byte card) to lock the
   record-offset -> card-byte map (confirms which tier each a2[...] is). DONE: the card parser in
   card-seed-trait-readers.md §1.2 (record +64/+68/+72 = card 69/73/77; +116..118 = card 113..115).
3. Decode the training/race STAT-PROGRESSION (how current rises toward 60 + the role of +0x5e/HIDDEN-A/B):
   the growth CURVE. Sources: items-feeding.md, horse-stats.md, the post-race stat-update code.
4. Live-capture: breed a foal in-game, dump its fresh card (base vs current), race it a few times, dump
   again -> confirm the base-static / current-grows model + measure the growth rate per growth-type.
