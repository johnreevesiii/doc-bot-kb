> **CORRECTION 2026-09-30 (knowledge archive pass after the FAQ decode review):** Still out of date here: The JP catalog anchor 0x11106C (DOC 2000) / 0x0F9680 (DOC '99), index first with the name at +4, is the pre-fix reading: records are name-first from file 0x111034 (DOC 2000, #1 = モミジサンデー) and 0x0F964C (DOC '99), fixed 2026-08-08. jp_mater_names.json was extracted with that pre-fix reading (it lacks モミジサンデー). The running-style line 'All five exactly equal = Almighty' is wrong: Almighty needs ALL SIX current externals equal, Corner INCLUDED; if Start, Out of the Box, Competing, Tenacious and Spurt are equal but Corner differs, the horse is a Front-runner (FAQ almighty). The 8-range personality table is not ROM text; the game has five personalities (FAQ personality-types). Source: the public FAQ (play.johnreevesiii.com/faq), FAQ decode review 2026-09-30.

# Breeding / Mater Inheritance System — Core Knowledge

KEY: `breeding-system`
Scope: the sire/dam breeding-stock tables, the JP "mater" (種馬/繁殖) inheritance name list, and how a
foal's stats derive from sire + dam. Covers all four versions. Every offset/field below was extracted
and confirmed from the real `.ic22` ROMs (not from a doc alone). Where a claim rests on a doc or a
community tool, it is flagged.

---

## 0. TL;DR

- **Foal inheritance (DECODED from the ROM, see §6 and decode-foal_average.md):** each external band =
  floor((sire + dam) / 2), no roll, fixed for life. Each internal = floor-average of the parents, minus 5
  if over 45, plus 5 if under 10, plus a small pedigree bonus (sire and dam both ◎ or both ✕ on 4+
  externals), capped at 45; about 1 birth in 32 then adds 5 to all three true internals. (Corrected
  2026-09-28: this line used to give the community simulator's model, externals ± 2 plus invented
  "bloodline" bonuses, as the best reconstruction. That model is superseded and removed from §6.)
- **Sire/Dam tables are ONE breeding-stock pool**, 84 sires then the dams on the EN World Edition
  (Rev C), in a single 1-based index space. (Corrected 2026-09-28: the adversarial re-read found ONE
  contiguous array of 167 valid records on Rev C and 177 on Rev D, not two arrays of 168/178; see §1.)
- The **JP "mater" list (167 entries, `jp_mater_names.json`) is the JP equivalent of that same pool**,
  merged into a single 167-record array. Stats line up record-for-record with the EN sires/dams.
- **60-byte sire/dam record (EN), anchored at the name:**
  `name[24]` · `st u32(+24)` · `sp u32(+28)` · `sh u32(+32)` · `ac byte(+36)` · `0 u32(+40)` ·
  `composite/Packed_u32_5 (+44, 4 bytes)` · `externals[6] (+48..+53)` · `pad (+54,+55)` ·
  `index u16 (+56)` · `pad (+58,+59)`.
- **What "ac" (name+36) encodes:** a 0-255 value = **course / dirt aptitude** (high = dirt, low = turf).
  Evidence: dirt-only-comment breeders average ac≈185, turf-only ac≈89. The community Breeder Studio
  JSON and DOC-ROM-Studio both name this field `ac` and use it as the dirt/surface preference.
  *Conflict:* the "DOCWE Master Source of Truth" doc instead labels the byte at name+36 as the
  **personality byte** (8-range table). Both interpretations are recorded below; the dirt correlation
  is the empirically stronger one.

---

## 1. Table locations (verified against bytes, all 4 versions)

| Version | folder | sire base | dam base | mater base (JP combined) | record stride | name field |
|---|---|---|---|---|---|---|
| WE Rev C  | drbyocwc | `0x10BF1C` | `0x10D2CC` | — (split EN) | 60 | 24B ASCII |
| WE EX Rev D | derbyocw | `0x10D264` | `0x10E614` | — (split EN) | 60 | 24B ASCII |
| DOC 2000 JP | derbyo2k | — (merged) | — | `0x11106C` | 60 | 20B EUC-JP (name at +4) |
| DOC '99 JP | derbyoc  | — (merged) | — | `0x0F9680` | **56** | 16B EUC-JP (name at +4) |

Counts (Corrected 2026-09-28; the old "84+84 = 168" and "89+89 = 178" came from the Breeder Studio
JSON, not the ROM, per the adversarial re-read in verify-breeding-system-verification.md):
- Rev C: **167** records in one contiguous block (index 1..167, then garbage). 84 sires, then dams.
- Rev D: **177** records (EX edition extends the pool). Whether records 85-89 are sires or dams is
  DISPUTED (the site's catalog lists them as sires).
- derbyo2k: **167** mater records, sequential idx 1..167.
- derbyoc: **167** mater records, sequential idx 1..167, stride 56.

The index runs on without restarting (read from the `+56` field); the "dam base" is just the offset of
record 85. JP holds the same 167-horse pool in one array. Rev D record 85 is "Pierogi Prince", ac=0,
externals `[1,1,15,15,15,1]`, a placeholder/joke entry.

---

## 2. The 60-byte sire/dam record (EN Rev C / Rev D) — FULL field map

Anchored at the **name** offset (`O = base + 60*k`). All multi-byte values little-endian.

| rel off | width | field | scale / notes | confidence |
|---|---|---|---|---|
| `+0x00` | 24 | **Name** | ASCII, `00`-padded (latin-ish; apostrophes present) | 1.0 verified |
| `+0x18` (+24) | u32 | **Stamina (st)** | ~10-60 (raw int in low byte) | 1.0 verified |
| `+0x1C` (+28) | u32 | **Speed (sp)** | ~10-65 | 1.0 verified |
| `+0x20` (+32) | u32 | **Sharp (sh)** | ~10-60 | 1.0 verified |
| `+0x24` (+36) | u32 (low byte used) | **ac = course/dirt aptitude** | 0-255 (mean ~148). See §4. | 0.8 verified-byte / 0.65 meaning |
| `+0x28` (+40) | u32 | **reserved / 0** | always 0 across all records | 0.95 verified |
| `+0x2C` (+44) | 4 bytes | **composite (Packed_u32_5)** | packed bitfield, see §5 | 1.0 byte-verified / meaning partial |
| `+0x30` (+48) | 6 | **Externals** start, corner, oob, competing, tenacious, spurt | each 0-15 (1-16 band) | 1.0 verified |
| `+0x36` (+54) | 2 | pad | `00 00` | 0.95 verified |
| `+0x38` (+56) | u16 | **record index** | 1-based; Rev C sires 1-84, dams 85-167 (corrected 2026-09-28 from 85-168) | 1.0 verified |
| `+0x3A` (+58) | 2 | pad | `00 00` | 0.9 verified |

Worked example — Rev C SIRE #1 `Maple Syrup` @ `0x10BF1C`:
```
name      = "Maple Syrup"
+24 st=39  +28 sp=19  +32 sh=34  +36 ac=240  +40=0
+44 composite = [0x00, 0xE0, 0x11, 0xF0]
+48 externals = [15,3,6,10,4,12]   (start,corner,oob,competing,tenacious,spurt)
+56 index = 1
```
Matched exactly to Breeder Studio JSON (`ver:revC, english:"Maple Syrup", st:39,sp:19,sh:34,ac:240,
start:15,corner:3,oob:6,comp:10,ten:4,spurt:12`). 84/84 sires and 84/84 dams round-trip. (Corrected
2026-09-28: the JSON is in a different order from the ROM, so it matches by NAME, not by index; 161 of
167 ROM names match. See verify-breeding-system-verification.md.)

### Reconciling the two offset conventions (the handoff's open item)
- The handoff says "externals at name-12, ac at name+36." The "name-12" reading is the SAME 6 external
  bytes — they physically live at `name+48` of the record they belong to, which is `name-12` of the
  *next* record. DOC-ROM-Studio reads `ext = nameOff-12` and labels them with record k; that is correct
  because record k's externals = (record k-1)'s `name+48`. Anchored to the owning name, externals =
  `name+48`. Verified: `rom[O-12:O-6] == rom[(O-60)+48:(O-60)+54]` for every record.
- The handoff's "4-byte composite ac at name+36" conflates two distinct fields:
  - `name+36` = the single `ac` byte (stored as a u32; only the low byte is meaningful).
  - `name+44` = the real 4-byte **composite / Packed_u32_5** (high entropy, see §5).
  These are 8 bytes apart. DOC-ROM-Studio's `ac = u32(nameOff+36)` reads the ac byte (as a u32) — which
  is why its "ac" column shows 240, 217, 182… (the dirt-aptitude values), not the composite.

---

## 3. JP record layout (derbyo2k stride 60 / derbyoc stride 56)

JP records put the **index FIRST**, then the name, then the same stat block. Anchored at record start
`O = base + stride*k`:

derbyo2k (stride 60):
| rel off | field |
|---|---|
| `+0` u32 | **index** (1..167) |
| `+4` … | **Name** EUC-JP (≤20 bytes), `00`-padded |
| `+28` u32 | st |
| `+32` u32 | sp |
| `+36` u32 | sh |
| `+40` byte | ac |
| `+48` 4B | composite |
| `+52` 6B | externals |

derbyoc (stride 56, 4 bytes shorter — name field is 16B):
| rel off | field |
|---|---|
| `+0` u32 | index |
| `+4` … | Name EUC-JP (≤16 bytes) |
| `+24` u32 | st |
| `+28` u32 | sp |
| `+32` u32 | sh |
| `+36` byte | ac |
| `+44` 4B | composite |
| `+48` 6B | externals |

Cross-version validation (same horse, same stats):
- derbyo2k #1 `トロットサンダー` (Trot Thunder): st=35,sp=27,sh=33,ac=217,ext=[4,8,4,12,14,8] —
  byte-identical to EN Rev C sire #2 `Heart Lake` (st=35,sp=27,sh=33,ac=217,ext=[4,8,4,12,14,8]).
  => JP mater idx N ≈ EN sire idx (N) for the first block; the JP list is the merged EN sire+dam pool
  with JP names.
- `ホワイトノーザー` appears in both derbyo2k (#2) and derbyoc (#2) with identical stats
  (st=19,sp=34,sh=39,ac=182) and identical composite `[1,234,0,48]` — so the composite is stable
  per-horse across JP versions (not random/leak).

`jp_mater_names.json` (167 entries) was extracted at `0x11106C + 60*k + 4` (EUC-JP) — confirmed: that
is exactly the name field of this mater table. The kana_hex column is the on-card 1-byte kana encoding
(see card-format spec), bridged ROM-EUC-JP ↔ Unicode ↔ card-kana.

---

## 4. What `ac` (name+36) encodes — the central question

Two analyses disagree; here is the evidence on each side.

**Interpretation A — `ac` = course / dirt aptitude (favored).**
- Community Breeder Studio JSON and DOC-ROM-Studio both name the field `ac` and treat it as the
  dirt/surface preference. The racing-stat table has an independent `dirt` field (also 0-255), and the
  in-game horse screen shows a 芝(turf)/ダート(dirt) aptitude.
- Independent correlation against the 344 breeder comment strings ("good on dirt", "races well on turf",
  etc.):
  - dirt-only comments: n=15, mean ac = **185** (min 85, max 255)
  - turf-only comments: n=3, mean ac = **89** (min 35, max 163)
  - both-surface comments: n=9, mean ac = **156**
  - High-ac (>200) examples: El Condor Pasa 252, Jade Robbery 248, Maple Syrup 240, Bubble Boy 252.
  - Low-ac (<60) examples: City Commandant 0, Black Lily 35, Shampoo&Conditioner 38, Miami Beach 41.
  This is a clean monotonic dirt↑ / turf↓ relationship. Interpretation A.

**Interpretation B — `ac` = personality byte (DOCWE Master Source of Truth).**
- That doc states "Personality byte is stored in `u32_3` (low byte)" where u32_3 = name+36, and gives an
  8-range table (Rough 0-47, Imposing 48-63, Calm 64-111, Firm 112-127, Sensitive 128-175, Moody
  176-191, Gentle 192-239, Proud 240-255).
- The doc ALSO says (for its "Table B" packed layout) that the **personality byte is byte 2 of
  Packed_u32_5** (= name+44 b2), not name+36. So the doc is internally inconsistent about where
  personality lives. The name+44 b2 values (17, 255, 171, 112…) decode to scattered personalities with
  no surface correlation.

**Resolution.** `ac` (name+36) is byte-verified and used directly as `ac` by every shipping community
tool; the dirt correlation is strong and independent. Treat **ac = dirt/course aptitude (0-255)** as the
working meaning. The "personality" label is a competing analysis; it is plausible the same byte also
seeds personality banding (the game may reuse one byte), but the dominant, testable effect is dirt
aptitude. Flagged as 0.65 confidence on meaning, 1.0 on the byte itself.

---

## 5. The composite field (name+44, "Packed_u32_5") — partially decoded

4 bytes, byte-verified, stable per horse across JP versions. Distributions (84 Rev C sires):
- `b0` ∈ {0,1,2,3} only — a 2-bit category. Dist: 0→15, 1→28, 2→33, 3→8.
- `b1` clustered in `0xC0..0xEF` (high nibble always 0xC or 0xE). Looks like a small enumerated field
  plus flags.
- `b2` near-unique per horse (high entropy) — per the Source-of-Truth doc this is the **PersonalityByte**
  (8-range decode); under that reading the composite carries personality here, freeing name+36 for ac.
- `b3` heavy on multiples of 0x10/0x30/0xA0/0xC0/0xF0 — nibble-packed flags (distance/grade/affinity?).

It does NOT map cleanly to distance aptitude (tested vs short/mile/middle/long comment buckets — b0/b1
do not separate the buckets). Best current model: **a packed inheritance/affinity + appearance +
personality composite** consumed by the breeding routine. Concretely the most defensible per-byte read,
combining the doc with the byte evidence, is:
- `b0` = growth/grade or leg-bias category (2-bit), `b1` = coat/appearance + flags,
  `b2` = personality (8-range), `b3` = packed hidden-affinity / distance flags.
All of `b0/b1/b3` are MEANING-TBD; only the byte positions are certain. This is the biggest remaining
open item for this subsystem. (Update 2026-09-28: card-seed-trait-readers.md §2 later decoded these
four bytes as coat modifier, coat base, run-style seed and personality, the starter horses' genes; the
guesses above predate it.)

(Note: the Source-of-Truth doc's "Table B @0x10D500, 74 records, name-last" is a DIFFERENT view of the
same data region — a packed-attribute mirror — and its Packed_u32_5 byte map (b0 CoatColorIdx, b1
CoatModifier, b2 Personality, b3 Attr23) is the cleanest external hint we have for name+44's bytes.)

---

## 6. Foal inheritance rule (DECODED; corrected 2026-09-28)

The ROM foal routine is decoded byte-exact (decode-foal_average.md). What it does:
- **Internals** (stamina, speed, sharp): floor((sire + dam) / 2); minus 5 if the result is over 45,
  plus 5 if under 10; plus the pedigree bonus; capped at 45. For a player's horse the parent values
  are its CURRENT (raced-up) internals (site port; 3 of 3 cabinet births reproduced).
- **Pedigree bonus:** count the externals where sire AND dam are both 12+ raw (◎) or both under 4 (✕).
  4-5 matches: stamina +1, speed +2, sharp +2. 6 matches: stamina +3, speed +2, sharp +3. Sire and dam
  count equally; no grandparent is read.
- **Birth band:** about 1 birth in 32 (Band 1) adds 5 to all three TRUE internals while the card prints
  lower; another 1 in 32 (Band 2) raises only the printed numbers.
- **External bands:** floor((sire + dam) / 2) on each of the six. No roll. Fixed for life.
- **Current externals of the fresh foal:** 2 x band + a 0-3 roll + an offset (Start -1, others +1).
- **Sex:** coin flip.

(Corrected 2026-09-28: this section used to print the community simulator's `breedFoal()` model from
`DOC Breeding and Racing Strategy.txt` as the working rule: externals rolling ± 2, "Stamina/Speed
Bloodline" +2 bonuses, a "Dirt Dynasty" +20 and ac ± 18. None of that is in the ROM routine. The model
was removed; do not quote its bonus rules.)

## 6.2 Running style (corrected 2026-09-28)
Style = where Start ranks among Start, OOB, Competing, Tenacious and Spurt (Corner ignored). Count how
many of the other four are strictly greater than Start: 0 = Front-runner, 1 = Start dash, 2 = Last
spurt, 3 or 4 = Stretch-runner. All five exactly equal = Almighty. It follows the horse's CURRENT
externals, so it can change during a career. Breeding-stock records do not store a style.
(Corrected 2026-09-28: the community `deriveRunningStyle()` printed here swapped Last spurt and
Stretch-runner and called any spread of 3 or less Almighty, and a line said leg type =
`floor(externals/51)`. All three were wrong.)

## 6.3 What is still open (corrected 2026-09-28)
- The Rev D foal routine has not been diffed against Rev C (all decodes are on Rev C).
- How dirt aptitude (`ac`) passes to the foal is not fully traced; the site uses the plain
  floor-average of the parents.
- (Corrected 2026-09-28: this used to say the ROM routine had never been located and that the name+44
  composite probably adds affinity or line bonuses. The routine is decoded, and name+44 holds coat,
  run-style seed and personality, see §5.)

---

## 7. Scales (confirmed)
- Externals on the breeding-stock table: **0-15** (the 1-16 aptitude band). Symbol map (DOC-ROM-Studio /
  Breeder Studio): `×` 1-4, `△` 5-8, `○` 9-12, `◎` 13-16.
- Internals st/sp/sh: ~10-60 raw (stored as u32, low byte).
- `ac`: 0-255 (dirt aptitude).
- Racing-stat table externals are a DIFFERENT 0-64 scale and a DIFFERENT roster — do not cross-map
  breeding stock to racing CPU horses by stat (names rarely coincide; when they do, stats differ).

---

## 8. How each claim was verified
- Extracted real bytes from all four `.ic22` ROMs with python3 (Bash tool), never reading binaries into
  context. Sample dumps: Rev C sires #1-8, Rev C dams #1-6, Rev D sires/dams #1-3, derbyo2k mater
  #1-6 & #166-167, derbyoc mater #1-5.
- Confirmed field semantics by matching decoded `st/sp/sh/ac/externals` to `DOC_Breeder_Studio_data.json`
  (681 horses) — exact for every spot-checked horse.
- Confirmed ac=dirt via correlation of name+36 against 344 breeder comment strings.
- Confirmed record counts/index space by reading the `+56` index of every record. (Corrected
  2026-09-28: the re-read found 167/177, not 168/178; see §1.)
- Inheritance rule read from the live `breedFoal()` JS in `DOC Breeding and Racing Strategy.txt`.
  (Superseded 2026-09-28 by the ROM decode; §6 now gives the decoded rule.)
- Field-meaning hints (personality 8-range, Packed_u32_5 byte map) read from
  `DOCWE_Master_Source_of_Truth_v1.0.md`; flagged as doc-derived and cross-checked against bytes.

---

## 9. Open questions
1. Decode name+44 composite bytes b0/b1/b3 (growth/grade? distance aptitude? hidden affinity flags?).
   ANSWERED 2026-09-28: coat modifier, coat base, run-style seed, personality (card-seed-trait-readers.md).
2. Settle ac definitively: is name+36 dirt-aptitude, personality, or both? Method: capture a breeding-
   stock horse's in-game course-aptitude + personality screen and match to its name+36 and name+44.b2.
3. Locate the actual ROM breeding routine (SH-4 code) to replace the averaging heuristic with the real
   algorithm. ANSWERED 2026-09-28: decoded byte-exact (decode-foal_average.md, summarised in §6). No
   "dominant parent" weighting exists; sire and dam count equally.
4. derbyoc2 (DOC II) mater table not located here (handoff did not include its ic22 in the 4); add when
   available.
5. Reconcile EN 168-slot pool vs JP 167 list: identify the dropped/placeholder slot precisely.
   (Update 2026-09-28: the Rev C ROM itself holds 167 valid records; the 168th came from the JSON.)

## 10. Tool ideas this unlocks
- **JP mater dropdown + foal predictor in the Stable Management System**: sire/dam picker driven by
  `jp_mater_names.json` (167) and the EN 168 pool, running the §6 model to preview foal st/sp/sh/ac +
  running style. Data ready now.
- **Breeding-stock editor in DOC-ROM-Studio (already partially present)**: extend to edit st/sp/sh/ac +
  externals + the name+44 composite per record, byte-exact, all 4 versions. Needs §5 decode to label
  the composite meaningfully; usable today as raw bytes.
- **Cross-version mater diff**: align EN sires/dams ↔ JP mater by stats to produce an authoritative
  "same horse, all 4 names" table (we proved the alignment via Trot Thunder=Heart Lake etc.).
