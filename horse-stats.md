# Racing Horse Record — Full Byte Decode (all 4 versions)

KEY: `horse-stats` · Subsystem: the 244-record **CPU-opponent** stat table in the NAOMI program ROM.

Status: **complete structural decode.** Every byte of the 32-byte (WE/2000) and 28-byte ('99) record
is now classified as one of: confirmed-meaning, id/echo, constant-pad, or "real per-horse field, semantic
narrowed but not 100% pinned" (the two hidden fields). Verified byte-for-byte against the four ROMs and
the per-version `DOC_COMPLETE_HORSE_DATABASE_*.md` ground-truth tables (244 horses each).

> IMPORTANT framing correction: this table is the roster of **CPU opponents**, NOT the player's horse.
> Proof: diffing the Beer-Experiment edited ROM (`beer_effects_test.ic22`) vs base `epr-22336c.ic22`
> shows the 12 changed bytes are ALL in the food/item table at `0x1671FC..0x16722D` — **zero** bytes
> in the racing table changed. Player-horse mutable stats live on the card / in cabinet nvram, not here.
> Hence there is no "sex / personality / condition / career" on these records: CPU horses don't breed,
> don't age, and (per DOC-ROM-Studio hint, now confirmed) **personality is not stored** — the DB's
> "Personality" column is inferred/derived, it does not map cleanly to any byte (see §4).

---

## 1. Table location & geometry (verified)

| Version | folder | ROM file | record start | stride | count | name table | name stride |
|---|---|---|---|---|---|---|---|
| WE Rev C | drbyocwc | epr-22336c.ic22 | `0x108E03` | 32 | 244 | `0x10AD50` | 18 |
| WE EX Rev D | derbyocw | epr-22336d.ic22 | `0x10A14B` | 32 | 244 | `0x10C098` | 18 |
| DOC 2000 JP | derbyo2k | epr-22284a.ic22 | `0x10AD1B` | 32 | 244 | (JP, EUC-JP) | — |
| DOC '99 JP | derbyoc | epr-22099b.ic22 | `0x0F6902` | **28** | 244 | (JP) | — |

All four ROMs are 4,194,304 bytes. The three later builds (WE-C, WE-D, 2000) use an **identical 32-byte
record format**; '99 uses a tighter **28-byte** format (no `idHi` duplicate, fewer pad bytes).

Cross-version identity (verified): **drbyocwc and derbyocw records are byte-identical for all 244**
(same roster, same balance). **derbyo2k differs in 22/244 records** — same layout, rebalanced values.
**derbyoc ('99) is a genuinely different roster** (different coats/styles per id).

---

## 2. Reconciling the two offset conventions

The seed facts use **record-start-relative** offsets; `DOC-ROM-Studio.html` uses
`recBase = recordStart + 9` and offsets relative to that. They describe the same absolute bytes:

| field | seed (rec-start +N) | ROM-Studio (recBase ±N) | absolute (WE-C, horse#1 @0x108E03) |
|---|---|---|---|
| dirt | +5 | recBase−4 | 0x108E08 |
| grade | +8 | recBase−1 | 0x108E0B |
| start | +9 | recBase+0 | 0x108E0C |
| internals stam/speed/sharp | +29/+30/+31 | recBase+20/21/22 | 0x108E20/21/22 |
| id | +2/+3 | recBase+? (ROM-Studio `id:16` is **wrong** for this) | 0x108E05/06 |

Note: ROM-Studio's `RACING_F` (`f:{... id:16 ... coat:13 ... grade:-1}`) is the **JP-2000 (recBase) view**
and its `id:16`/`coat:13` are mis-set for the 244 CPU table — use the absolute map in §3 instead.
ROM-Studio's later block (`f:{dirt:5,start:9,...stamina:29,...}` for derbyo2k and the `derbyoc` 28-byte
`f:{dirt:4,...stamina:24}`) IS correct and matches my byte verification.

---

## 3. Annotated record layout — 32-byte format (drbyocwc / derbyocw / derbyo2k)

Offsets are record-start-relative. "verified" = matched against the 244-horse DB for every horse unless noted.

| off | hex | field | range | meaning / how verified | conf |
|---|---|---|---|---|---|
| +0 | 0x00 | pad | 0 | constant 0x00 all 244 | 1.0 |
| +1 | 0x01 | **HIDDEN-A** | 0–2 | real per-horse field; does NOT predict grade/style/coat/personality. Never changed in the 22 2000-vs-WE diffs (more static than HIDDEN-B). Candidate: growth-type / aptitude flag. | 0.5 |
| +2 | 0x02 | **id** (copy 1) | 1–244 | == horse number for all 244 (verified) | 1.0 |
| +3 | 0x03 | **id** (copy 2) | 1–244 | == horse number for all 244 (verified). Two single-byte copies, not a 16-bit hi/lo. | 1.0 |
| +4 | 0x04 | pad | 0 | constant | 1.0 |
| +5 | 0x05 | **DIRT aptitude** | 0–255 | matches DB Dirt for all (e.g. #1 Gold Queen=168, #2 First Star=8, #16 Kowloon=250, #18 Victory Lap=104) | 1.0 |
| +6 | 0x06 | pad | 0 | constant | 1.0 |
| +7 | 0x07 | pad | 0 | constant | 1.0 |
| +8 | 0x08 | **GRADE** | 0–3 | 0=Ungraded 1=G3 2=G2 3=G1. Verified vs DB for all (#1=3=G1, #2=1=G3, #16=0=Ungraded). **Corrects** any claim that grade lives elsewhere. | 1.0 |
| +9 | 0x09 | ext **Start** | ~11–63 | matches DB Start (#1=44) | 1.0 |
| +10 | 0x0A | ext **Corner** | ~14–59 | matches DB Corner (#1=35) | 1.0 |
| +11 | 0x0B | ext **OOB** (out-of-box) | 4–63 | matches DB OOB (#1=19) | 1.0 |
| +12 | 0x0C | ext **Competing** | 8–63 | matches DB Comp (#1=32) | 1.0 |
| +13 | 0x0D | ext **Tenacious** | 3–62 | matches DB Tenac (#1=40) | 1.0 |
| +14 | 0x0E | ext **Spurt** | 4–63 | matches DB Spurt (#1=46) | 1.0 |
| +15 | 0x0F | pad | 0 | constant | 1.0 |
| +16 | 0x10 | **HIDDEN-B** | 0–2 | real per-horse field, independent of HIDDEN-A (both can be nonzero). **Changed by JP-2000 rebalance** in 9 of the 22 differing records (e.g. horse #13 differs ONLY at +16: 0→1). Does not predict grade/style/coat/total. Candidate: hidden growth/aptitude. | 0.55 |
| +17 | 0x11 | pad | 0 | constant | 1.0 |
| +18 | 0x12 | pad | 0 | constant | 1.0 |
| +19 | 0x13 | pad | 0 | constant | 1.0 |
| +20 | 0x14 | pad | 0 | constant | 1.0 |
| +21 | 0x15 | **RUNNING STYLE** | 0,1,2,3,7 | 0=Front-runner 1=Start dash 2=Last spurt 3=Stretch-runner 7=Almighty. **Perfectly deterministic** 244/244 vs DB Style. **NEW** (seed left +21 unknown). | 1.0 |
| +22 | 0x16 | **COAT COLOR** | enum | 192=Chestnut 193=Black 199=Brown 202=Bay 204=Dark Gray 207=Light Gray 222=Special (0=Default). Verified vs DB for all (#1=207 Light Gray, #6=204 Dark Gray, #16=222 Special, #18=192 Chestnut, #22=199 Brown). | 1.0 |
| +23 | 0x17 | **HIDDEN-X low** | 0–255 | low byte of a 16-bit per-horse value (see +24). 133 distinct (lo,hi) pairs across 244. | 0.6 |
| +24 | 0x18 | **HIDDEN-X high** | clustered | high byte; nibble clusters: 0xA0 (186 horses), 0x30 (30), 0xC0 (9), 0xF0 (7), 0x00 (12). Does NOT correlate with grade, dirt, coat, or total stat. **Changed in the 2000 rebalance** (cols 23/24 differ in 8/5 of the 22 diffs) → it IS a real attribute. Candidate: distance/surface aptitude bitfield OR hidden pedigree-affinity composite (analogous to the sire table's 4-byte "ac"). | 0.55 |
| +25 | 0x19 | **id echo** | id&0xFF | == horse_number mod 256 (so #244 → 0). Third id copy. | 1.0 |
| +26 | 0x1A | pad | 0 | constant | 1.0 |
| +27 | 0x1B | pad | 0 | constant | 1.0 |
| +28 | 0x1C | pad | 0 | constant | 1.0 |
| +29 | 0x1D | int **Stamina** | 0–60 | matches DB Stam (#1=23) | 1.0 |
| +30 | 0x1E | int **Speed** | 0–63 | matches DB Speed (#1=37) | 1.0 |
| +31 | 0x1F | int **Sharp** | 0–60 | matches DB Sharp (#1=48) | 1.0 |

**Example raw record (WE-C, horse #1 "Gold Queen"):**
`00 01 0101 00 a8 00 00 03 2c 23 13 20 28 2e 00 00 00 00 00 02 cf 0e a0 01 00 00 00 17 25 30`
decodes to: id=1, dirt=168, grade=G1, ext=[44,35,19,32,40,46], style=2(Last spurt), coat=0xCF(Light Gray),
hiddenX=0xA00E, idEcho=1, int=[23,37,48]. Matches DB exactly.

---

## 4. Annotated record layout — 28-byte format (derbyoc / DOC '99)

Same field semantics, packed tighter (drops one id copy and four pad bytes). Offsets rec-start-relative.

| off | field | range | verified |
|---|---|---|---|
| +0 | **HIDDEN-A** | 0–2 | analog of 32B +1. dist: 0×173, 1×64, 2×7 |
| +1 | **id** | 1–244 | == horse# |
| +2 | **id** (copy) | 1–244 | == horse# |
| +3 | pad | 0 | const |
| +4 | **DIRT** | 0–255 | verified vs '99 DB |
| +5 | pad | 0 | const |
| +6 | pad | 0 | const |
| +7 | **GRADE** | 0–3 | 0=Ung 1=G3 2=G2 3=G1 — 244/244 clean |
| +8 | pad | 0 | const |
| +9..+14 | ext Start/Corner/OOB/Comp/Tenac/Spurt | — | verified |
| +15 | pad | 0 | const |
| +16 | pad | 0 | const |
| +17 | **HIDDEN-B** | 0–2 | analog of 32B +16. dist: 0×206, 1×35, 2×3 |
| +18 | **RUNNING STYLE** | 0–3 | 0=Front 1=Start dash 2=Last spurt 3=Stretch — 244/244 clean (no Almighty in '99) |
| +19 | **COAT** | enum | same enum, fully verified (192/193/199/202/204/207/222) |
| +20 | **HIDDEN-X low** | 0–255 | analog of 32B +23 |
| +21 | **HIDDEN-X high** | clustered | analog of 32B +24; hi-nibble clusters 0xA0/0x30/0xC0/0xF0 |
| +22 | **id echo** | id&0xFF | == horse# |
| +23 | pad | 0 | const |
| +24 | int **Stamina** | 0–60 | verified |
| +25 | int **Speed** | 0–63 | verified |
| +26 | int **Sharp** | 0–60 | verified |
| +27 | pad | 0 | const |

So the 28→32 expansion added: a second id copy (+3), and three extra pad bytes scattered through the record
(the WE format is 0-padded more generously, likely for 4-byte field alignment).

---

## 5. What is now KNOWN vs the seed's "~10 of 32 mapped"

Newly decoded / corrected beyond the seed's known set (id, dirt, externals, internals):
- **+8 GRADE** (0-3) — confirmed at +8, not "recBase-1 ambiguous".
- **+21 RUNNING STYLE** (5 values incl. Almighty=7) — fully deterministic. Seed listed +21 as unknown.
- **+22 COAT** — full 8-value enum verified against all 244 (seed had coat as candidate only).
- **+2/+3 and +25 = three id copies** — the "horse id" candidate resolved; it's literally the 1-based roster index, duplicated.
- Confirmed **personality is NOT stored** (no byte predicts it).
- Confirmed **sex / career / condition / age are NOT on this table** (CPU opponents; player data is elsewhere).

Still hidden (real fields, semantics narrowed but not pinned — need an in-game opponent stat/encyclopedia
screen to label):
- **HIDDEN-A (+1 / 28B +0)** small 0–2, fairly static.
- **HIDDEN-B (+16 / 28B +17)** small 0–2, JP-2000 rebalanced it independently of stats.
- **HIDDEN-X (+23/+24 16-bit / 28B +20/+21)** clustered high byte; rebalanced in 2000; best guesses are
  distance/surface aptitude bitfield or a hidden affinity/pedigree-compat composite (cf. sire "ac").

---

## 6. How everything was verified (reproducible)

- Loaded all four `.ic22` (4 MB each) and sliced records by the stride/start above.
- Per-byte column stats over 244 records identified constant-pad vs varying columns.
- Parsed each `DOC_COMPLETE_HORSE_DATABASE_*.md` markdown table (244 rows: grade, dirt, 6 externals,
  3 internals, coat, style) and cross-checked **every** byte field against **every** horse — externals,
  internals, dirt, grade, coat, style all match 244/244.
- Confirmed style/grade/coat are 1:1 deterministic (no value collisions).
- Confirmed CPU table is static by diffing the beer-edited ROM (changes only in food table @0x167xxx).
- Cross-version diff (WE-C vs WE-D byte-identical; vs 2000 = 22 differing records; '99 = different roster).
- Helper scripts in `C:/Users/johnr/AppData/Local/Temp/hs*.py`; pickled ROM cache at `hs.pkl`.

---

## 7. Open questions
- Label HIDDEN-A / HIDDEN-B / HIDDEN-X by reading an in-game opponent stat screen or the breeding/"sire
  encyclopedia" screen and matching values to bytes (same method that cracked names). The JP-2000 rebalance
  evidence says they matter; we just lack a labeled readout.
- Is HIDDEN-X (16-bit) a pointer/index into another table (e.g. portrait, voice, AI-behavior, or pedigree
  affinity)? Check whether its value ever indexes a 244- or 84-entry table.
- Does the player-horse record (card / cabinet nvram) reuse this same field order? (Likely yes for the
  externals/internals/dirt/coat block — worth aligning the card decoder to this map.)

---

## 8. Tool ideas this unlocks
- **CPU roster exporter / web viewer**: one parser, all 4 versions, emits full annotated stats incl. the
  now-decoded style + coat + grade (the existing DB .md files can be regenerated byte-exact from ROM).
- **Cross-version diff visualizer**: highlight the 22 horses DOC-2000 rebalanced and which field changed
  (already have the data; just render it).
- **Opponent-difficulty calculator**: with style+grade+all stats decoded, compute a true power ranking and
  expose "strongest CPU horse" objectively (DB teases this; we can compute it).
- **ROM-Studio field-map fix**: patch `RACING_F` so `id`/`coat`/`grade` use the absolute map in §3 (current
  `id:16`,`coat:13` are wrong for the 244 table) and add `style:+21` so the editor can edit running style.
