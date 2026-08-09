# Derby Owners Club — Derived / Seldom-Documented Attributes

Subsystem KEY: `derived-attrs`
Scope: leg-type / running-style derivation, personality, ability/aptitude symbols (X/A/O/@),
growth/internal "type", and the relationship between the 0-64 racing externals and the 1-16
breeding (sire/dam) externals. Covers all four ROMs.

All offsets below were extracted live from the real `.ic22` images; confidence + how-verified is
noted per claim. Helper scripts written during this pass:
`C:/Users/johnr/AppData/Local/Temp/doc_re.py` (ROM byte dumper) and `.../card_re.py` (card decoder).

ROM build signatures (re-confirmed @0x8000, 8 bytes):
- drbyocwc (WE Rev C, epr-22336c): `dc99020c9cc8210c`
- derbyocw (WE EX Rev D, epr-22336d): `09004ad20ee347d0`
- derbyo2k (DOC 2000 JP, epr-22284a): `162f047ffcf5fcf6`
- derbyoc  (DOC '99 JP, epr-22099b): `188bee02f1532838`

---

## 1. The two stat rosters and their two scales (CONFIRMED)

There are **two independent horse tables** with **two different external-stat scales**. Conflating them
is the single biggest source of confusion in prior docs.

### 1a. Racing table (244 records) — the CPU/foundation roster — externals 0-63
- Record starts: drbyocwc `0x108E03` /32, derbyocw `0x10A14B` /32, derbyo2k `0x10AD1B` /32,
  derbyoc `0x0F6902` /28.
- Externals live at record+9..+14 = **start, corner, oob, competing, tenacious, spurt**.
- VERIFIED: across all 4 ROMs the six externals span **min 3 .. max 63** (244 records each).
  So the racing scale is effectively **0-63** (a 6-bit field), NOT 0-64. (Display in-game adds nothing
  for CPU horses; the +1 display rule is a *card* convention — see §6.)
- Internals (stamina/speed/sharp) at +29/+30/+31 (WE) or +24/+25/+26 (derbyoc 28-byte), range **0-63**.

### 1b. Sire/Dam breeding tables (84 + 84) — the breeding stock — externals 1-16
- Name tables: drbyocwc sire `0x10BF1C` / dam `0x10D2CC`; derbyocw sire `0x10D264` / dam `0x10E614`.
  Stride **60**, 84 records each.
- **Corrected field map (vs seed "externals at name-12"): the breeding externals are at name+48..+53**,
  one byte each, on the **1-16 band scale** (observed 0-15):
  `+48 start, +49 corner, +50 oob, +51 comp, +52 tenac, +53 spurt`.
- Internals: `name+24 stamina (st)`, `name+28 speed (sp)`, `name+32 sharp (sh)` — each a 4-byte LE
  field but only the low byte is used (0-63).
- `ac` aptitude composite: **single byte at name+36** (0-255). This is the breeder JSON's `ac`.
- `name+56` = a small index/id (1,2,3,... per record).
- VERIFIED byte-exact against `DOC_Breeder_Studio_data.json` (681 horses, all versions): e.g. drbyocwc
  "Maple Syrup" record = st 39(0x27), sp 19(0x13), sh 34(0x22), ac 240(0xF0), ext [15,3,6,10,4,12] —
  matches the JSON exactly. "Heart Lake", "Judge Angelucci", "Song Sung Blue", "Trust Me" all matched.
- The breeder JSON `id` 1-84 = sires (sex M), 85-168 = dams (sex F); 168 entries per version
  (revA=derbyoc, revB, revC=drbyocwc, revD=derbyocw). The JSON is alphabetized; the ROM tables are not.

### 1c. Scale relationship (CONFIRMED by ranges; ÷4 mapping HIGH confidence)
- Racing externals 0-63, breeding bands 1-16. 63/16 ≈ 3.94, so **racing ≈ 4× breeding band**
  (band b ⇒ racing ~4b-1..4b). This is the same relation the aptitude symbols use (§4).
- Internals share the same 0-63 scale on both tables (no rescale needed).
- The 1-16 retirement/breeding externals are therefore a **coarse 4:1 quantization** of the fine 0-63
  racing externals. A horse retiring to stud gets its 0-63 stats binned into 1-16 bands for the
  next-generation breeding table.

---

## 2. Leg type / running style (RESOLVED — two competing models reconciled)

There are **5 running styles**, label table (English) in order:
`Front-runner, Start dash, Last spurt, Stretch-runner, Almighty`.
- VERIFIED label offsets: drbyocwc `0x128EE0`-ish (cluster 0x128ED0..0x128F08),
  derbyocw `0x12B128`. JP ROMs (derbyo2k/derbyoc) carry the same 5-style system but with EUC-JP
  labels (English "Front-runner"/"Almighty" absent — searched, zero hits).
- Matches the Card-Creator `LEG_TYPES[0..4]` array exactly.

### 2a. The authoritative game rule: STORED on the card at T1 byte 7, `floor(byte7 / 51)`
- The game reads a dedicated **byte 7 of card track 1** (0-255) and maps it via **÷51** to the 5
  styles: 0-50→Front-runner, 51-101→Start dash, 102-152→Last spurt, 153-203→Stretch-runner,
  204-255→Almighty. VERIFIED: ÷51 produces clean 5-way bucketing on the boundary values.
- This is the source of truth for what the running engine uses. CONFIDENCE HIGH (ROM 5-style table +
  card byte + the project CLAUDE.md "Running Style: Style = floor(byte7/51)").

### 2b. The Card-Creator tool's "Start-rank among externals" rule = a DISPLAY HEURISTIC, not the ROM rule
- `legTypeFromExt()` in DOC-Card-Creator.html derives a style from where **Start** ranks among the
  externals *excluding Corner* (all-equal ⇒ Almighty; Start highest ⇒ Front-runner; 2nd ⇒ Start dash;
  3rd ⇒ Last spurt; 4th/5th ⇒ Stretch-runner).
- IMPORTANT FINDING: the editor **never writes a1[7]** (grep: no `a1[7] =` assignment). It only writes
  externals (a2[38..43]) and then *re-derives* a leg-type label from those externals for the quick-view.
  So on **editor-made cards, byte 7 is leftover/garbage** and the two models disagree.
- DECISIVE EVIDENCE (real cards, decoded live):
  | card | T1[7] | ÷51 (ROM) | ext-derived (tool) |
  |---|---|---|---|
  | Caitin Clark | 1 | Front-runner | Start dash |
  | DD | 213 | Almighty | Stretch-runner |
  | Gulf of America | 255 | Almighty | Last spurt |
  | Scarecrow II | 0 | Front-runner | Stretch-runner |
  | Test Tube Flycast (game-raced) | 86 | Start dash | Front-runner |
  They diverge on most cards ⇒ the two are genuinely different. The ROM uses byte7/51; the tool's
  externals rule is an approximation it shows because it doesn't track/author byte 7.
- LIKELY ORIGINAL INTENT: the *seed's* "start-stat rank among externals" rule is plausibly how the game
  **assigns the initial running style at birth** (from the foal's starting externals), which then gets
  baked into byte 7 and can drift via training. Not yet disassembled — see Open Questions.

### 2c. Leg type is NOT stored in the racing (CPU) table
- VERIFIED: scanning all 32/28 record columns of the 244-record table, **no column matches the
  externals-derived leg type** for >90% of rows. The 5 "Almighty" CPU horses are exactly the 5 records
  whose six externals are all equal to **31** (ids 8,41,83,174,182 in drbyocwc) — a sentinel/all-rounder
  value. CPU running style is therefore computed on the fly from externals, while player horses carry the
  stored byte-7 value.

---

## 3. Personality (Rough/Imposing/Coward/Honest/Sloppy) — STORED on card byte 6 only

### 3a. Where stored
- **Card Track 1 byte 6** (a1[6]), a 0-255 value. VERIFIED present and varied on every real card
  (e.g. Caitin 64, DD 180, Gulf 0, Phil 80, AA 244, Test Tube 48).
- **NOT stored on the 244 CPU racing horses**: scanning the racing table, no column carries the
  personality byte ranges. CONFIRMED — matches the ROM-Studio note "personality is not stored on CPU
  horses." CPU horses get personality assigned at runtime / don't expose it.

### 3b. Two label sets in the ROM (both VERIFIED by string extraction)
- **English "Check"/assessment labels** at drbyocwc `0x0E84A4`:
  `Imposing, Honest, Rough, Coward, Sloppy, Too soft, Strict` (7 labels).
- **Japanese romaji personality labels** at `0x107DFC`:
  `Doudou, Sunao, Arai, Okubyou, Zubora` (5) — and a duplicate cluster at `0x0EB61C`
  (`DouDou, Sunao, Arai, Okubyou, Zuboro`).

### 3c. Two byte-range interpretations (reconciled)
- **Card-Creator 5-bucket model** (`getPersonalityCode`, what the tool authors/shows):
  R(Rough) ≤47; I(Imposing) the gaps; C(Coward) 64-79 / 128-143 / 192-207; H(Honest) 80-111 / 144-175;
  S(Sloppy) ≥208. Editor write map `PERSONALITY_MAP = {R:0, I:48, C:64, H:80, S:208}`.
- **8-band ROM model** (mechanics deep-dive, `÷32`-ish bands over 0-255):
  Rough 0-47 (Arai), Imposing 48-63, Calm/Sunao 64-111, Firm/Strict 112-127, Sensitive/Okubyou 128-175,
  Moody 176-191, Gentle/Doudou 192-239, Proud 240-255.
- RECONCILIATION: both read the **same card byte 6**. The 8-band model is the finer ROM truth; the 5
  English labels are what the in-game "Check" command surfaces (Rough/Imposing/Coward/Honest/Sloppy +
  Too soft/Strict edge labels). The tool's 5-bucket function is a lossy presentation of the same byte.
- Personality drives the interaction-effect multiplier table at ROM `0x0E7D00` (38 IEEE-754 floats;
  Hug/Praise/Scold/Flatter scaled ×2.0..−2.0 by personality) — documented, not re-verified this pass.

---

## 4. Ability / aptitude symbols (X / A / O / @  ⇔  ✕ △ ○ ◎) — DERIVED, not a separate stored field

- The ◎○△✕ shown in-game are computed from the **breeding (1-16) external value** by `symFor(v)`
  in DOC-ROM-Studio.html, VERIFIED logic:
  - `v >= 13 → ◎` ("@", band 13-16, top)
  - `v >= 9  → ○` ("O", band 9-12)
  - `v >= 5  → △` ("A", band 5-8)
  - `v >= 1  → ✕` ("X", band 1-4, weakest)
  - `v == 0  → ·`
- So the X/A/O/@ "ability/aptitude bands" are just the four quartiles of the 1-16 breeding scale, one
  symbol per external (start/corner/oob/comp/tenac/spurt). The trigger is purely the external's band;
  nothing else gates the symbol. CONFIDENCE HIGH.
- Because racing externals ≈ 4× breeding bands (§1c), the same quartile logic applied to a 0-63 racing
  external (÷4 then band) yields the same symbol — consistent system across both rosters.

### 4a. `ac` composite (name+36) and the name+45..47 bytes
- `ac` (name+36, 0-255) is the breeder JSON's headline "aptitude composite". VERIFIED equal to JSON.
  Its internal meaning (weighted blend of internals?) is not fully decoded but it tracks overall quality
  (Wild Sun ac=255 with strong internals; lower-tier sires ~120-150).
- A separate 3-byte block at **name+45..+47** varies per sire (e.g. Maple `E0 11 F0`, Heart Lake
  `CC FF 3C`). First nibble clusters at 0xC-0xE across sires ⇒ likely a coat/trait + breeding-comment /
  inheritance-weight composite (the deep-dive mentions "inheritance weights V0-V3 + coat/trait + comment
  index"). PARTIALLY UNKNOWN — see Open Questions. (This is distinct from `ac`; the seed's "4-byte
  composite at name+36" conflated `ac` with this region.)

---

## 5. Growth type / internal "type" (CLARIFIED)

- The block at drbyocwc `0x0EE270` previously tagged "Leg-Type Labels (Retirement)" actually reads
  **`Speed type`, `Stamina type`, `Sharp type`** (VERIFIED extraction). These are the three **internal
  stat-type / growth labels** — i.e. which internal (speed/stamina/sharp) a horse is biased toward.
  It is NOT a leg-type table (that mislabel is corrected).
- There is **no evidence of a distinct stored "growth curve / growth type" byte** in either the racing
  table or the sire/dam table. The three internals (stamina/speed/sharp at +29/30/31 racing,
  name+24/28/32 breeding) plus the "Speed/Stamina/Sharp type" label are the entirety of the "type"
  system found. A horse's dominant internal = its "type." CONFIDENCE MEDIUM-HIGH (absence-of-field is
  inferred from full-column variance scans showing no other plausible enum).
- The 0x0EE270 block continues with `Stud reg. / Dam reg. / Sire / Dam` — the retirement/registration
  UI labels, confirming this is the **retirement screen** string block (where 0-63 stats are shown as
  1-16 bands and the horse is registered to stud/dam).

---

## 6. Racing-table full byte map (244 records) — UNKNOWN bytes decoded

drbyocwc, record stride 32, start 0x108E03 (offsets from record start). Verified via per-column
min/max/uniq over all 244 records + cross-ref to name table @0x10AD50/18 and breeder data.

| off | field | range | confidence | notes |
|---|---|---|---|---|
| +0  | (zero) | 0 | high | padding |
| +1  | class/gen flag | 0-2 | med | 175×0 / 62×1 / 7×2; not correlated to dirt. Likely roster class or generation. |
| +2  | horse id | 1-244 | high | sequential |
| +3  | id (dup) | 1-244 | high | mirrors +2 |
| +4  | (zero) | 0 | high | padding |
| +5  | **dirt aptitude** | 0-255 | high | 59 uniq; matches card a3[61] dirt scale |
| +6,+7 | (zero) | 0 | high | padding |
| +8  | **grade** | 0-3 | high | 0 Ungraded,1 G3,2 G2,3 G1 (matches GRADE enum) |
| +9..+14 | **externals** start/corner/oob/comp/tenac/spurt | 3-63 | high | the six racing stats |
| +15 | (zero) | 0 | high | |
| +16 | **sex** | 0-2 | high | 200 M(0) / 37 F(1) / 7 gelding(2) |
| +17..+20 | (zero) | 0 | high | padding |
| +21 | minor enum | 0-7 | med | 1×98 2×69 3×44 0×30 7×3 — candidate jockey/silk or distance pref; UNCONFIRMED |
| +22 | **coat color** | 0/192-222 | high | 207,204,202,222,199,192,193 (matches COAT enum) |
| +23 | sub-coat / pattern | 0-255 | med | 81 uniq; pairs with +22 (special-coat modifier) |
| +24 | composite | 0-250 | med | dominated by 0xA0/0x30 family — silk/jockey or breeding hint |
| +25 | id (3rd copy) | 1-243 | high | |
| +26..+28 | (zero) | 0 | high | padding |
| +29 | **internal stamina** | 0-60 | high | |
| +30 | **internal speed** | 0-63 | high | |
| +31 | **internal sharp** | 0-60 | high | |

derbyoc (28-byte stride, start 0x0F6902) is the **same record minus 4 padding bytes**: dirt +4,
grade +7, externals +9..+14, **sex +17**, coat +19, internals **+24/+25/+26**. An extra small enum at
+18 (0-3). VERIFIED by per-column scan.

Reconciliation of the two offset conventions (both correct, different origins):
- **Seed / Card-Creator convention** measures from the literal record start (0x108E03): dirt+5,
  start+9 ... internals+29/30/31.
- **DOC-ROM-Studio convention** uses `recBase = recordStart + 9` (= 0x108E0C) so it lists
  coat at recBase+13 (=+22), grade at recBase−1 (=+8), dirt at recBase−4 (=+5), internals at
  recBase+20/21/22 (=+29/30/31). Algebraically identical; the −0x9 shift is the only difference.

---

## 7. Card (player-horse) layout for these attributes — where derived attrs actually live

US/WE card = 207 bytes = 3 tracks × 69, logical bytes stored **reversed per track**; tool reads with
`arr[k] = bytes[t*69 + (69-k)]`, 1-indexed. Confirmed on real cards.

| field | location | notes |
|---|---|---|
| Personality | **T1 byte 6** (a1[6]) | 0-255 → §3 bands. Editor writes via PERSONALITY_MAP. |
| Running style | **T1 byte 7** (a1[7]) | 0-255, `floor/51` → 5 styles (§2). Editor does NOT write it. |
| Coat base / modifier | T1 byte 8 / byte 9 | byte8=63 ⇒ special coat keyed by byte9 |
| Sex | T2 byte 16 (a2[16]) | 0 M / 1 F / 2 gelding |
| Current externals | T2 bytes 38-43 (a2[38..43]) | start=43,corner=42,oob=41,comp=40,ten=39,spurt=38; **display = value+1** |
| Internals | T2 bytes 61/65/69 | sharp/speed/stamina (0-60) |
| Dirt ability | T3 byte 61 (a3[61]) | 0-255 |
| Retired flag | T3 byte 57 | 0 active / 1 retired |
| Breed count | T3 byte 53 | offspring = value/2 |

So on a **player card**, personality and running style are first-class STORED bytes (T1[6], T1[7]); the
externals are stored at full 0-63 resolution (T2[38-43], +1 for display). On the **CPU racing table**
they are stored at 0-63 too but personality/running-style are derived. On the **sire/dam table** the
externals are pre-binned to 1-16 and shown with ◎○△✕ symbols.

---

## 8. Cross-version summary

| attribute | drbyocwc (RevC) | derbyocw (RevD) | derbyo2k (DOC2000) | derbyoc (DOC'99) |
|---|---|---|---|---|
| racing rec start / stride | 0x108E03 /32 | 0x10A14B /32 | 0x10AD1B /32 | 0x0F6902 /28 |
| externals scale | 0-63 | 0-63 | 0-63 | 0-63 |
| sex offset in rec | +16 | +16 | +16 | +17 |
| internals offsets | +29/30/31 | +29/30/31 | +29/30/31 | +24/25/26 |
| running-style labels | 0x128EE0 (EN) | 0x12B128 (EN) | EUC-JP | EUC-JP |
| personality labels | EN @0x0E84A4 + JP @0x107DFC | EN (shifted) | EUC-JP | EUC-JP |
| sire/dam tables | 0x10BF1C / 0x10D2CC | 0x10D264 / 0x10E614 | (see jp_mater) | (see jp_mater) |
| breeding ext scale | 1-16 | 1-16 | 1-16 | 1-16 |

All four use the identical SYSTEM (5 styles, 0-63 racing ext, 1-16 breeding bands, ◎○△✕ quartiles,
byte-6 personality, byte-7 running style on cards). Only string localization and the derbyoc 28-byte
packing differ.

---

## 9. Open questions

1. **Birth-time running-style assignment.** Confirm by SH-4 disassembly whether the game computes a
   foal's initial byte-7 from its starting externals (the seed's "Start rank among externals" rule) and
   whether/how training mutates byte 7. The "Leg-Type Change Messages" block (drbyocwc 0x12755C) proves
   running style *changes during a career* — find the code that rewrites byte 7.
2. **name+45..47 sire composite.** Decode the 3-byte block (inheritance weights V0-V3? coat/trait +
   breeding-comment index?). First nibble clusters 0xC-0xE.
3. **`ac` composite formula.** What blend of internals/externals yields ac (name+36, 0-255)?
4. **Racing-table +21 (0-7) and +24.** Identify (jockey/silk? distance preference? hidden affinity?).
5. **racing +1 (0-2) class flag.** Roster class vs generation vs special-event horse?
6. **JP on-card personality/running-style.** US cards store them at T1[6]/T1[7]; the derbyo2k/derbyoc
   LR cards appear identity-only (name/sire/dam). Confirm whether JP cabinets keep personality/style
   in nvram keyed by the 0x25-0x27 lead bytes (needs a hardware reader / stat-screen capture).
7. **Confirm 8-band vs 5-bucket personality in-game.** Capture the "Check" screen for horses with byte6
   in each band to lock the exact label boundaries (Too soft / Strict edge cases).

---

## 10. Tool ideas this unlocks

- **Correct running-style authoring**: extend DOC-Card-Creator to actually WRITE a1[7] from a chosen
  style (style×51 + offset), instead of only deriving a label from externals. Fixes the divergence in §2b.
- **Aptitude-symbol card view**: render ◎○△✕ per external on the card quick-view by binning the 0-63
  card externals ÷4 into the 1-16 → symFor() pipeline, so player cards show the same symbols the game's
  retirement/stud screen shows.
- **Retirement preview**: given a player card's 0-63 externals, show the 1-16 breeding bands it will
  register at stud/dam with (the ÷4 quantization), i.e. predict the offspring inheritance table entry.
- **Personality calculator**: 0-255 byte ⇒ both the 5 English Check labels and the 8 ROM bands, plus the
  Hug/Praise/Scold/Flatter multiplier (from 0x0E7D00) for the recommended interaction.
- **CPU roster browser with derived style**: list all 244 CPU horses with their on-the-fly leg type
  (externals rule) + grade + coat + sex + dirt, since CPU style is not stored.
