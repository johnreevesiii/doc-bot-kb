# The Hidden Rules of Derby Owners Club

The mechanics the game never tells you — recovered by reverse-engineering the ROM, translated into
"here's the rule → here's what to do." Every claim carries a **confidence tag** and, where it lives
on the player card, the **byte** (see `CARD_BYTE_MAP.md`).

**Confidence:** **[exact]** = decoded byte-for-byte · **[strong]** = decoded but approximate /
calibrated · **[inferred]** = pattern-derived, not yet byte-proven.

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

### 3. ~6% of foals are random jackpots or duds. **[exact]**
A hidden dice roll: about **3% of foals take −12 (or −5) to a stat**, and about **3% get +12 (or
+5)**. Your "ruined" foal or "god roll" was usually this, not your pairing.
→ **Do:** re-breed; variance is built in. Don't over-read a single foal.

### 4. Sex is a coin flip; aptitude is inherited bit-by-bit (not averaged). **[exact / strong]**
Foal **sex = a 50/50 random bit** — you can't steer it. **Dirt/turf aptitude & running-style are
inherited per-bit** from the parents' hidden masks (the on-card "composite"), **not averaged** — so
aptitude can jump, not just blend. The flat dirt-number (card 0x92) is *not* what's inherited.
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

### 6. "Best distance" is about stamina & aptitude, not speed. **[exact]**
The distance table is a **pace normalizer** (distance × its multiplier ≈ constant), so raw speed
doesn't make a horse a "sprinter." What suits a distance is whether it can **sustain** it.
→ **Do:** match distance to **stamina/aptitude**, not top speed. (Use the Race Simulator's *Best
trip*.)

### 7. Stamina drain runs on a stat-total ladder. **[exact]**
Per-segment stamina drain is selected by the **sum of the six stat bytes** crossing thresholds
(220 / 240 / 250 / 260 / 270 / 280 / 300) — higher totals drain *less* per step.
→ **Do:** a high overall stat *total* improves stamina efficiency, not just the stamina stat alone.

### 8. Condition is stepped, not smooth. **[exact]**
Condition acts through breakpoints at **92 / 95.5 / 96 / 99 / 99.5** — crossing one is a real jump.
→ **Do:** push condition **just past 96 (and 99)** before a big race; being at 95 vs 96 is a step,
not a nudge.

### 9. Per-tick speed is clamped to [10, 160]; ability has a floor of ~10. **[exact / strong]**
No horse moves below 10 or above 160 per tick; even a weak horse keeps a floor.
→ **Do:** don't expect a stat gap to produce runaway blowouts — the clamp keeps races close.

### 10. Whip grades: hold ×1, light ×2, hard ×3 — and it costs energy. **[exact]**
The whip/hold adds a graded velocity boost but spends stamina.
→ **Do:** save the hard whip (×3) for the closing stretch; early whipping burns the tank.

---

## Bonding — the right words build it, the wrong words destroy it

### 11. Post-race bond gain = `multiplier × (100 − current bond)`, and the multiplier can be negative. **[exact formula / inferred response-names]**
Each post-race response has a **personality-specific multiplier** (×2.0 great … **−2.0 actively
lowers the bond**). Gain shrinks as bond approaches 100 (diminishing returns). A "Too soft" horse
loves every response; a "Strict" horse **rejects affection** and only responds to the firm option.
*(Card: personality 0x3F.)*
→ **Do:** match the response to the personality. **Coddling a strict horse lowers its bond.** Build
bond early (when it's low) for the biggest gains.

---

## Feeding, roster & version meta

### 12. Each food has a known stat payload; growth items behave differently. **[exact]**
Foods add specific Speed/Stamina/Sharp (and hidden extras); a separate **class flag** marks
"growth" items vs ordinary feed. (Use the Feeding Advisor.)
→ **Do:** feed to the stat you want; don't burn rare growth items as filler.

### 13. "Almighty" CPU horses have all six externals = 31. **[exact]**
A sentinel profile — versatile, strong at every phase.
→ **Do:** spot them when scouting (Roster Browser flags them) and avoid soft assumptions about the field.

### 14. The meta differs by version. **[exact]**
Rev C = Rev D for CPU stats; DOC 2000 rebalanced a dozen records and renamed the roster; DOC '99 is a
different roster/track/food set (no beer/banana).
→ **Do:** know which version you're playing before importing "best horse" lists from another.

---

*Provenance per section:* breeding → `_sh4/decode/foal_average.md` + `DOC_CORE.md` (byte-exact:
floor-avg, ±5 soft clamp, cap 45, pedigree count, banded RNG, sex, per-bit aptitude). Racing →
`areas/race-formula.md` + `_sh4/decode/*` (dirt bands, distance normalizer, drain ladder, condition
gates, clamp, whip; stat→ability dominance is [strong] from live calibration). Bonding →
`areas/personality-interaction.md` (formula exact; response identities inferred). Feeding →
`areas/items-feeding.md`. Versions → `areas/version-diff.md`.
