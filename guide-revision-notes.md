# Strategy-Guide Revision Notes (stash for the NEXT project)

Holding pen for findings that don't belong in HIDDEN_RULES.md (they don't change how you *play*)
but DO correct or sharpen the old community strategy guides. When we start the guide-revision
project (old guides saved in `~/Downloads`), work through this list: for each item, find where the
guides repeat the old claim and replace it with what the ROM/RE now shows. Cite the byte evidence.

Format per item: **Claim in the old guides → What we now know (with evidence) → Where it likely
appears in a guide.**

---

## 1. "The game reuses breeding horses as CPU racers, just renamed" — MYTH (corrected)
- **Old claim:** to save memory, the breeding-stock horses and the CPU racing field are the same
  horses with different names.
- **What we now know:** they are **two separate ROM tables** with different record formats, in
  different regions, overlapping only by *name* (both pull from the same pool of famous real
  Thoroughbreds). Even a shared name has different stats in each table.
  - CPU roster: records `0x108E03`, 32-byte (externals 0–63, internals as single bytes, dirt at +5).
  - Breeding sire/dam: `0x10BF1C` / `0x10D2CC`, 60-byte (name inline, u32 stats, ac/dirt at +36,
    externals 0–15 band scale).
  - **Worked example (Rev C, the one that prompted this):** *Sunday Silence* is both a CPU racer
    (rec #110 `0x109BC3`, **dirt 0xFF = 255**, internals 32/27/49, ext 50/44/41/48/44/62) and a
    breeding sire (idx 68 `0x10CED0`, **dirt 0xD8 = 216**, st/sp/sh 31/49/40, ext 9/9/8/8/8/11).
    Same name, independently authored records, different dirt.
  - Across all 14 names that appear in both tables, racer-minus-breeder dirt runs +103…−182, no
    pattern. So there is no "translate the racer's dirt to get the breeder's."
- **Likely guide locations:** breeding sections that tell you a breeder's stats by pointing at the
  CPU racer of the same name; any "these are the same horses" trivia.
- **Evidence:** decoded `doc_core_roster.json` + `doc_core_breeding.json`; raw ROM dump
  epr-22336c.ic22; also footnoted in `ROM_ARCHITECTURE.md` §4.

---

## STATUS
- **Decision (Jun 6 2026): errata now, clean HTML rewrite later.**
- **Errata companion DRAFTED:** `Community-Pack/doc-site/handbook-errata.md` (covers breeding, dirt/
  aptitude + the reuse myth, personality/bond, internals cap & jackpot odds; race/whip/feeding
  reviewed and left standing). Not yet published as a site page. This file is the rewrite's spec.
- NEXT: (1) optional deeper scan of Advanced CPU-pairs/jackpot pages + Mini, fold any new claims in;
  (2) RE the one open item (over-breeding degrades externals?) before the rewrite repeats/kills it;
  (3) publish errata as `handbook-errata.html` (Suite style) + link from the site; (4) full rewrite.

## TARGET GUIDES (downloaded morning of Jun 6 2026, in `~/Downloads`)
The community **DOCWE Handbook** trilogy, reader editions (text extracted to
`_core/_guides_extract/{beginners,advanced,mini}.txt`):
- `DOCHandbookBeginners3_reader_edition.pdf` — 68 pp ("the Horse" Stable / DOCWE Handbook)
- `DOCHandbookAdvanced3_reader_edition.pdf` — 44 pp (Jackpot testing, CPU pairs, whip charts; Gabe Gomez et al.)
- `DOCHandbookMini3_reader_edition.pdf` — 40 pp

Verdict tags below: **CONFIRMED** (guide is right, we now prove it) · **CORRECT** (guide is
wrong, replace) · **SHARPEN** (right idea, mechanism wrong/incomplete) · **ADD** (hidden rule the
guide never had) · **VERIFY** (guide claims a mechanic we haven't pinned — RE target).

## 2. Breeding mechanics (Beginners pp.17–18, L553–604) — the big cluster
- **CONFIRMED — externals = floor((sire+dam)/2).** Guide L580–585 states it exactly ("El Condor
  Pasa 9 + Ferranti's Folly 15 → 12") and our foal-build RE confirms it byte-exact. Keep it; add a
  "(confirmed from the ROM)" note.
- **CORRECT — "at birth the computer multiplies each stat by 2 and throws a random +/- factor in"**
  (L599–602). That is the old averaging myth. Real foal build (FUN_0C052B0C, byte-verified):
  floor-average of each stat, a ±5 soft-clamp at the 45/10 edges, a pedigree bloodline bonus, then
  a decoded banded-LCG noise (two 8-wide gates ~3.1% each: one pulls internals down ~12, one pushes
  up ~12). Not a uniform "+/-". That banded noise is the real reason same-parent foals differ.
- **CORRECT/CAP — internals cap at 45, not 60/65.** The jackpot ceiling. Explains why maxed lines
  plateau and why "Well Balanced" foals appear once a line tops out (Beginners "well balanced"
  passage; Advanced jackpot section pp.9–11).
- **SHARPEN — "dirt/off-track set by lineage; Thunder Boy is the #1 dirt sire, his line is higher"**
  (L568–571). Right that aptitude is inherited; wrong mechanism. Dirt/aptitude is **per-bit,
  RNG-gated inheritance of the name+44 composite MASK from BOTH parents** (sire odd-bits | dam
  even-bits, neighbor jitter), not a sire "dynasty" multiplier. The raw `ac` dirt byte itself is
  NOT inherited. So a strong-dirt sire raises the odds via its mask bits, but it is not a guaranteed
  pass-down, and the dam's bits matter equally.
- **ADD (HIDDEN RULE) — favorable / unfavorable pairs.** The game rewards parent CONSISTENCY: per
  external, if BOTH parents are ≥12 (or BOTH <4), it counts; cnt 4–5 → internals +1/+2/+2,
  cnt 6 → +3/+2/+3 (then cap 45). Guides never knew this — it is the real lever behind "good pairs."
- **ADD — jackpot odds are now computable.** "Jackpot = blessed by a random factor" (Beginners FAQ
  L352; Advanced pp.9–13) is now byte-exact: the up-band is ~3.1% per stat, stacked with the
  pedigree bonus on a maxed line. We can give real numbers instead of folklore.
- **VERIFY — over-breeding degrades externals after ~10–15 breeds** (L592–596). We track a breed
  count (card a3[53]/2) but have NOT confirmed a degradation mechanic. Flag as an RE target before
  repeating the claim.

## 3. Still to scan (next pass)
- Race/whip mechanics: START stat → early vs late race (Beginners L457/468/510), whip windows &
  per-track whip charts (Advanced) — cross-check against the recovered race formula (dirt 4-band
  curve, distance table, speed clamp [10,160]); much of the whip-timing is empirical and may stand.
- Feeding: Advanced food lists & special-food routes vs `doc_core_food.json` (44-byte table).
- "Specials" / handicap-race weight-by-earnings (Beginners FAQ) — verify against data.
- CPU Pairs / jackpot-trigger races (Advanced pp.8+) vs roster + race schedule.
