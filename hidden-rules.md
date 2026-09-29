# The Hidden Rules of Derby Owners Club

The mechanics the game never tells you — recovered by reverse-engineering the ROM, translated into
"here's the rule → here's what to do." Every claim carries a **confidence tag** and, where it lives
on the player card, the **byte** (see `CARD_BYTE_MAP.md`).

**Confidence:** **[exact]** = decoded byte-for-byte · **[strong]** = decoded but approximate /
calibrated · **[inferred]** = pattern-derived, not yet byte-proven. (Added 2026-09-28:
**[UNVERIFIED]** = an earlier claim that later checking did not support; do not teach it as fact.
**[DISPUTED]** = the decodes disagree.)

---

## Breeding — the game rewards matched bloodlines and taxes high averages

### 1. Foal stats are the floor-average of the parents — then docked. **[exact]**
Internals (stamina/speed/sharp) = `floor((sire + sire) / 2)`… `floor((sire+dam)/2)`, then a **soft
clamp**: any result **over 45 loses 5**, any result under 10 gains 5. So a parent average of 46 comes
out as **41**. Internals are then **hard-capped at 45** — *not* 60/65 as widely believed.
→ **Do:** don't expect a foal to beat ~45 internals from breeding; the ceiling is real, and pairing
two maxed parents wastes the overflow.

### 2. The bloodline bonus — favorable & unfavorable pairs. **[exact]**
The game counts, across the six externals, how **consistent** the two parents are: for each external,
**+1 if BOTH parents are strong (≥12)** and **+1 if BOTH are weak (<4)**. That count drives a bonus:
- count 4–5 → **+1 stamina, +2 speed, +2 sharp**
- count 6 → **+3 stamina, +2 speed, +3 sharp** (then cap 45)

→ **Do:** **breed like-to-like.** Two parents strong in the *same* externals get rewarded; a
mismatched pair (one strong, one weak per external) earns nothing. This is the "favorable pair"
reward you sensed — it's real and it's specifically about *matching* external profiles.

## Breeding, continued: the hidden birth roll and bit-by-bit inheritance

### 3. About 1 foal in 16 hits a hidden birth roll, and the card can mislead you. **[exact]**
(Corrected 2026-09-28: this rule used to say about 3% of foals lose 12 or 5 and about 3% gain, which
had the direction backwards.) About **1 birth in 32 (Band 1)** raises all three TRUE internals by +5
while the card's printed birth stats go DOWN (one stat -12, or all three -5). Another **1 in 32
(Band 2)** raises only the printed numbers (+12 on one stat, or +5 on all three, on stats under 40)
and leaves the true internals alone. For about 94% of foals the printed birth stats are the true ones.
→ **Do:** a "ruined" printed foal (for example 45/45/33 or 40/40/40) may be your best foal; a printed
jackpot may be cosmetic.

### 4. Sex is a coin flip; aptitude is inherited bit-by-bit (not averaged). **[exact / strong]**
Foal **sex = a 50/50 random bit**: you can't steer it. **Dirt/turf aptitude is inherited per-bit**
from the parents' hidden masks (the on-card "composite"), **not averaged**, so aptitude can jump,
not just blend. The flat dirt-number (card 0x92) is *not* what's inherited. (Corrected 2026-09-28:
running style is NOT inherited this way, as this rule used to say. The displayed style is computed
from the horse's CURRENT externals; the old "run-style seed" byte passes to foals but no race code
reads it.)
→ **Do:** to fix aptitude, pick parents whose aptitude *masks* you want; averaging dirt numbers is
the wrong mental model. *(Card: composite/aptitude inheritance is [exact] in logic; the mask→letter-
grade display decode is [strong].)*

---

## Racing — externals run the race; internals are the support cast

### 5. Externals dominate base ability; internals matter less than you think. **[strong]**
In controlled tests, swinging internals across their whole range barely moved a horse's base race
ability, while externals (start/corner/oob/competing/tenacious/spurt) drove it. *(Card: externals
0x5F–0x64; internals 0x45–0x4D.)*
→ **Do:** train and breed for **externals** first; internals are a smaller multiplier on top.

### 6. "Best distance" is about stamina & aptitude, not speed. **[UNVERIFIED]**
(Corrected 2026-09-28: this was tagged [exact]. The distance table's exact use is only 0.6
confidence in race-formula.md, and no decode links a stat to a best distance. Do not give
distance advice from this rule.) The distance table is a **pace normalizer** (distance × its
multiplier ≈ constant), so raw speed doesn't make a horse a "sprinter." What suits a distance is
whether it can **sustain** it.
→ **Do:** treat any "best trip" advice as a guess, not a rule.

## Racing, continued: stamina, condition, the speed clamp and the whip

### 7. Stamina drain runs on a stat-total ladder. **[UNVERIFIED]**
(Corrected 2026-09-28: this was tagged [exact]. It comes from early race-formula notes, and reading
this routine as a race stamina drain is UNVERIFIED; do not teach it as fact.) The early notes said
per-segment stamina drain is selected by the **sum of the six stat bytes** crossing thresholds
(220 / 240 / 250 / 260 / 270 / 280 / 300), with higher totals draining less.
→ **Do:** no advice from this rule until it is verified.

### 8. Condition is stepped, not smooth. **[exact]**
Condition acts through breakpoints at **92 / 95.5 / 96 / 99 / 99.5** — crossing one is a real jump.
→ **Do:** push condition **just past 96 (and 99)** before a big race; being at 95 vs 96 is a step,
not a nudge.

### 9. Per-tick speed is clamped to [10, 160]; ability has a floor of ~10. **[exact / strong]**
No horse moves below 10 or above 160 per tick; even a weak horse keeps a floor.
→ **Do:** don't expect a stat gap to produce runaway blowouts — the clamp keeps races close.

### 10. Whip grades: hold ×1, light ×2, hard ×3, and it costs energy. **[UNVERIFIED]**
(Corrected 2026-09-28: this was tagged [exact]. The graded-whip reading comes from early notes and is
UNVERIFIED; do not teach whip grades or a whip energy cost as fact.) The early notes said the
whip/hold adds a graded velocity boost but spends stamina.
→ **Do:** no whip-grade advice from this rule. The whip charts are community handbook tradition.

---

## Bonding — the right words build it, the wrong words destroy it

### 11. Post-race bond gain = `multiplier × (100 − current bond)`, and the multiplier can be negative. **[exact formula / reply mapping UNVERIFIED]**
A **personality-dependent multiplier table** exists (×2.0 … **−2.0 actively lowers the bond**), and
some replies can lower the bond. Gain shrinks as bond approaches 100 (diminishing returns). Which
reply button is which column, what else selects the row (a runtime flag, possibly the race result),
and how the bond reaches hearts are NOT decoded. *(Card: personality 0x3F.)* (Corrected 2026-09-28:
this rule used to describe "Too soft" and "Strict" horses. Those are not personalities; DOC has five:
Imposing, Honest, Rough, Coward, Sloppy. That description came from a retracted table.)
→ **Do:** give no per-button advice (see personality-interaction.md).

---

## Feeding, roster & version meta

### 12. Each food has a known stat payload; growth items behave differently. **[exact payload / DISPUTED stats]**
Every food's per-column numbers are decoded byte-for-byte (use the Feeding Advisor). (Corrected
2026-09-28: this rule used to say foods add Speed/Stamina/Sharp as fact. Whether the first three
columns raise the internals Speed/Stamina/Sharp or the externals is DISPUTED, not settled (see
items-feeding.md).) A separate **class flag** may mark "growth" items vs ordinary feed (a 0.8-confidence
reading, not confirmed in game).
→ **Do:** quote a food's column numbers, not a promised stat gain; don't burn rare growth items as
filler.

### 13. CPU horses' running styles are stored per horse, not computed. **[exact]**
(Corrected 2026-09-28: this rule used to say the "Almighty" CPU horses are the ones with all six
externals = 31. That is wrong: the three World Edition Almighty CPU horses have unequal externals, and
the all-31 CPU horses are not Almighty.) A player horse is different: its style is computed from its
current externals (Start's rank among Start, OOB, Competing, Tenacious, Spurt; Almighty only when
those five are equal).
→ **Do:** read a CPU horse's style from the roster, not from its stats.

## Version meta and provenance

### 14. The meta differs by version. **[exact]**
Rev C = Rev D for CPU stats (16 race-roster names differ); DOC 2000 differs from World Edition in 22 of
244 CPU records and carries the original Japanese names (World Edition is where the horses got
English names); DOC '99 is a different roster/track/food set (no beer/banana). (Corrected 2026-09-28:
this used to say DOC 2000 "rebalanced a dozen records and renamed the roster".)
→ **Do:** know which version you're playing before importing "best horse" lists from another.

---

*Provenance per section:* breeding → `_sh4/decode/foal_average.md` + `DOC_CORE.md` (byte-exact:
floor-avg, ±5 soft clamp, cap 45, pedigree count, banded RNG, sex, per-bit aptitude). Racing →
`areas/race-formula.md` + `_sh4/decode/*` (dirt bands, distance normalizer, drain ladder, condition
gates, clamp, whip; stat→ability dominance is [strong] from live calibration; the drain ladder,
whip grades and distance advice are UNVERIFIED as of 2026-09-28). Bonding →
`areas/personality-interaction.md` (formula exact; response identities inferred). Feeding →
`areas/items-feeding.md`. Versions → `areas/version-diff.md`.
