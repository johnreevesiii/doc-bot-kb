# Appearance — Coat colors, silks, hood (`appearance`)

KEY: `appearance` · Subsystem: how a horse's *visual* identity (coat color incl. starred specials,
silk/jacket pattern + two silk colors, hood) is encoded — both on the **player card** and in the
**244-record CPU stat table** in the program ROM. Sex is included because its byte sits in the same
card block and shares a ROM string table with the coat names.

Status (Jun 2026): **card-side appearance fully VERIFIED byte-for-byte** on 9 real US/WE cards.
**ROM CPU-record coat enum fully VERIFIED** 244/244 against the per-version DBs and located the ROM
coat *name* string tables. Two NEW findings: (1) the card decoder's `getColorName` and the ROM coat
enum **disagree** on byte values 202/204/207; (2) the named **starred specials are a player-card-only
feature** — the CPU record has only a generic `Special` flag with **no** per-variant selector byte.

There are **TWO independent coat-encoding systems** in DOC. Keep them separate:

| | where | bytes | semantics |
|---|---|---|---|
| **A. Player-card coat** | card track1 `a1[8]`,`a1[9]` | 2 bytes | base coat + special-coat sub-id (the starred coats live ONLY here) |
| **B. CPU-record coat** | ROM racing table `+22` (`+19` on '99) | 1 byte | enum 0xC0–0xDE (Chestnut..Special); generic `Special`, no variant |

---

## 1. Player-card appearance block (US / World Edition) — VERIFIED

Container is the standard 207-byte / 3×69 card (see `us-card`). Logical index `aT[k]` lives at file
offset `T*69 + (69-k)`. All offsets below confirmed by decoding 9 real cards
(`C:/DerbyOwnersClub/Card-Library/*.card` + `Tools/Cards/*`).

| field | logical | file off | type / range | example (verified) |
|---|---|---|---|---|
| **Coat base** | a1[8] | **0x3D** | u8; `63` = special-coat trigger, else a base-coat byte | Caitin=77 Bay, Scarecrow=129 Black, DD=196 Chestnut, BabyBoy=203 Brown |
| **Coat modifier** | a1[9] | **0x3C** | u8; special sub-id when a1[8]=63, else 0 | Gulf/Phil=112→Org Panda, Xi=48→Panda |
| **Silk color 2** | a2[13] | **0x7D** | u8 palette 0–14 | Scarecrow=White(13), WillyJR=Teal(3) |
| **Silk color 1** | a2[14] | **0x7C** | u8 palette 0–14 | Scarecrow=Yellow(14), Gulf=Red(12) |
| **Silk pattern** | a2[15] | **0x7B** | u8 0–7 | Caitin=7, BabyBoy=2, WillyJR=6 |
| **Sex** | a2[16] | **0x7A** | 0=Male 1=Female 2=Gelding | Caitin=1 F, Phil=2 Gelding, DD=0 M |
| **Hood** | a2[26] | **0x73** | u8 0–63 | Scarecrow=21, BabyBoy=24, WillyJR=7, most=0 |

`aT[k]` file offset = `T*69 + (69-k)`. a1[8]→0x3D, a1[9]→0x3C; a2[13]→0x7D, a2[14]→0x7C, a2[15]→0x7B,
a2[16]→0x7A, a2[26]→0x73. (Confirmed: matches `populateForm()`/`buildArraysFromForm()` in
`Tools/DOC-Card-Creator.html`.)

### Verified decode dump (real cards)
```
Caitin_Clark   c1c2= 77   0 -> Bay        pat7 sc1=LightBlue sc2=White  hood 0 sexF
Gulf_of_America c1c2=63 112 -> Org Panda  pat1 sc1=Red       sc2=Yellow hood 0 sexF
Xi_Jinping     c1c2=63  48 -> Panda       pat0 sc1=Black     sc2=White  hood 0 sexM
Scarecrow_II   c1c2=129  0 -> Black       pat0 sc1=Yellow    sc2=White  hood21 sexM
BabyBoy        c1c2=203  0 -> Brown       pat2 sc1=Yellow    sc2=LtGreen hood24 sexM
WillyJR        c1c2=129  0 -> Black       pat6 sc1=Maroon    sc2=Teal   hood 7 sexM
```

---

## 2. Coat color — the full code→name tables

### 2a. Special (starred) coats — card only (a1[8]=63, a1[9]=sub-id) — VERIFIED
Source: `COLOR_OPTIONS` in `DOC-Card-Creator.html`; confirmed live on Gulf/Phil (112→Org Panda) and
Xi (48→Panda). The full sub-id table:

| a1[9] | name | | a1[9] | name |
|---|---|---|---|---|
| 0 | Okapi ⭐ | | 80 | White ⭐ |
| 16 | Cow ⭐ | | 112 | Org Panda ⭐ |
| 48 | Panda ⭐ | | 192 | Zebra ⭐ |
| 64 | Platinum ⭐ | | 208 | Cow_2 ⭐ |
| | | | 240 | Tiger ⭐ |

Any other a1[9] with a1[8]=63 → generic "Special". The 9 named specials are spaced 16 apart
(0,16,48,64,80,112,192,208,240) — a coarse palette index, not contiguous. **These named specials do
NOT exist in the CPU record** (see §3): the CPU table only has the generic `Special` enum value 222.

### 2b. Normal coats — card decoder `getColorName(c1,c2)` (a1[8] families)
The Card-Creator classifies a1[8] into Bay/Black/Brown/Chestnut/Gray by membership in low/mid/high
"mirror" bands (×1/×2/×3 of a base). Editor write-anchors (`COLOR_OPTIONS`): Bay=77, Black=129,
Brown=69, Chestnut=64, Gray=127.
- Bay: {77,78,79, 141,142,143, 205,206,207}
- Black: {65,66,67, 129,130,131, 193,194,195}
- Brown: {69,70,71,73,74,75, 133..139, 197,198,199,201,202,203}
- Chestnut: {64,68,72,76, 128,132,136,140, 192,196,200,204}
- else → Gray

### 2c. CPU-record coat enum — ROM table `+22` — VERIFIED 244/244
Single byte in the racing stat record. Mapped 1:1 against `DOC_COMPLETE_HORSE_DATABASE_DRBYOCWC.md`:

| byte | name | byte | name |
|---|---|---|---|
| 0 (0x00) | Default | 204 (0xCC) | Dark Gray |
| 192 (0xC0) | Chestnut | 207 (0xCF) | Light Gray |
| 193 (0xC1) | Black | 222 (0xDE) | Special |
| 199 (0xC7) | Brown | | |
| 202 (0xCA) | Bay | | |

Distribution (drbyocwc/derbyocw identical): 0×1, 192×4, 193×3, 199×15, 202×39, 204×54, 207×107,
222×21. derbyo2k nearly identical (199×14, 202×40). derbyoc ('99, coat at **+19**, 28-byte record):
different roster — 192×6,193×6,199×15,202×38,204×49,207×103,222×26.

### 2d. ROM coat NAME string tables (NEW — located)
Two ASCII coat-name tables in the WE-C ROM (null-delimited, with interleaved `0F xx 0F` UI color
control codes):
- **Simple set @ 0x0C68F0:** `GRAY, CHESTNUT, BLACK, BAY, BROWN, SPECIAL, WHITE`
  (immediately followed by sex: `MALE, FEMALE, GELDING` @ 0x0C6926+).
- **Granular set @ 0x0EE0AC:** `GRAY, DARK CHESTNUT BAY, CHESTNUT BAY, BLACK, BAY, DARK BAY, DARK,
  SPECIAL, WHITE` (then `MALE, FEMALE, GELDING` again).
The granular set explains the low/mid/high "mirror" byte families: e.g. 0xC0..0xCF span
Chestnut→Light Gray shades. (The `WHITE` at 0x166F4C is the food item "WHITE MUSHROOM", not a coat.)

### 2e. ⚠ DISCREPANCY (NEW): card decoder vs ROM enum disagree at 202/204/207
The same byte value is named differently by the two systems:

| byte | card `getColorName` | ROM CPU enum | agree? |
|---|---|---|---|
| 192 | Chestnut | Chestnut | ✅ |
| 193 | Black | Black | ✅ |
| 199 | Brown | Brown | ✅ |
| **202** | **Brown** | **Bay** | ❌ |
| **204** | **Chestnut** | **Dark Gray** | ❌ |
| **207** | **Bay** | **Light Gray** | ❌ |
| 222 | Gray (no special match) | Special | ❌ |

The ROM enum (§2c, backed by 244 DB horses + the ROM name tables) is the **authoritative** game
behavior. The Card-Creator's `getColorName` mirror-band heuristic is an approximation that
mislabels the high range; real player cards happen to use values (77,129,196,203) that the heuristic
gets right, so the bug is latent. **Tool fix:** replace `getColorName`'s family lists with the
verified enum {192:Chestnut,193:Black,199:Brown,202:Bay,204:Dark Gray,207:Light Gray,222:Special,
63→special sub-id table}. (Confidence high — both sides extracted from bytes.)

---

## 3. Silks (jacket) — pattern + two colors — VERIFIED

Three card bytes; **no in-ROM name table** (the labels below are the curated tool set, palette
indices observed in-game). The CPU record has NO silk bytes (CPU horses race without an owner jockey).

- **Silk pattern** `a2[15]` (0x7B), 0–7 (`silkType` in tool):
  `0 Horizontal Stripe · 1 Half and Half · 2 Diagonal Stripe · 3 Dots · 4 Stars · 5 Vertical Stripe
  · 6 Diamond Patterns · 7 Diamond Rows`.
- **Silk color 1** `a2[14]` (0x7C) and **Silk color 2** `a2[13]` (0x7D), each palette index 0–14
  (`SILK_COLORS`): `0 Black · 1 Grey · 2 Blue · 3 Teal · 4 Brown · 5 Maroon · 6 Green · 7 Light Green
  · 8 Magenta · 9 Light Blue · 10 Purple · 11 Pink · 12 Red · 13 White · 14 Yellow`.
- Note byte order: in the card, color 2 (0x7D) precedes color 1 (0x7C) — i.e. `a2[14]`=primary
  ("Color 1") at the lower file offset 0x7C, `a2[13]`="Color 2" at 0x7D. Verified by the tool's
  `populateForm` (sc1←a2[14], sc2←a2[13]).
- These 15 silk-color names are **not** ASCII in the ROM (searched: only coat names + food strings
  are ASCII; silk palette is numeric). So the names are a tool convention, not ROM ground truth — a
  hardware/in-game capture would confirm exact hues. (Confidence: byte positions 1.0; name mapping
  observational ~0.8.)

---

## 4. Hood — VERIFIED

`a2[26]` (0x73), u8 0–63. Tool's curated `hood` dropdown values:
`0 None · 1 Black · 2 Blue · 7 Custom · 15 Yellow/Black · 25 Green · 30 Star Hood · 38 Standard ·
63 Funny`. Real cards observed: Scarecrow=21, BabyBoy=24, WillyJR=7 — so the full 0–63 range is used
and the dropdown is a partial label set (many intermediate values are valid, names unmapped). Like
silks, hood names are not ASCII in the ROM. CPU record has no hood byte. (Confidence: position 1.0;
0–63 range 1.0; intermediate names unknown.)

---

## 5. Sex — VERIFIED (related; shares ROM string table)

`a2[16]` (0x7A): `0=Male, 1=Female, 2=Gelding`. Confirmed: Caitin=1(F), Phil=2(Gelding), DD/Xi=0(M).
ROM string table places `MALE / FEMALE / GELDING` directly after the coat names at both 0x0C6926 and
0x0EE106 — i.e. the in-game stat screen renders coat + sex from adjacent string tables. CPU record
has **no** sex byte (confirmed in `horse-stats`: CPU opponents don't breed). (Confidence 1.0.)

---

## 6. JP cards (DOC 2000 / DOC '99) — no on-card appearance

Verified on 8 JP `.card` files (frozen + raced derbyo2k): header `0x20=0x03,0x21=0x02`, no `SEGA`
marker, tracks 2-3 all zero. **No coat/silk/hood bytes on JP cards** — appearance, like JP stats, is
cabinet-side (keyed to the lead bytes 0x25-0x27 / the cabinet nvram). Consistent with `jp-card`. The
JP coat/silk values would have to be read from a cabinet nvram capture or hardware reader. (Confidence
1.0 that they're absent from the card.)

---

## 7. Cross-version summary (all 4)

| | card coat a1[8/9] | card silk/hood | CPU-record coat byte off | CPU coat enum |
|---|---|---|---|---|
| WE Rev C (drbyocwc) | yes (specials work) | yes | **+22** (rec @0x108E03) | 0xC0–0xDE, identical |
| WE EX Rev D (derbyocw) | yes | yes | **+22** (rec @0x10A14B) | byte-identical to Rev C |
| DOC 2000 (derbyo2k) | identity card only | none on card | **+22** (rec @0x10AD1B) | same enum, roster rebalanced (199×14,202×40) |
| DOC '99 (derbyoc) | identity card only | none on card | **+19** (rec @0x0F6902, **28-byte**) | same enum, different roster |

Card container is identical across versions; only US/WE write the full appearance block. The CPU
coat enum is shared across all four (the granular ROM name table @0xEE0AC is the same concept; '99
uses the tighter 28-byte record so coat sits at +19 not +22).

---

## 8. Still open / low-confidence

- **Starred-special selector for CPU horses:** NONE found. The 21–26 coat=222 CPU horses carry no
  byte that ranges over the 9 special sub-ids (tested HIDDEN-X lo/hi @+23/+24 — does not correlate;
  11/21 specials have lo=0). Conclusion: CPU "Special" renders one generic special skin; the named
  Panda/Zebra/Tiger/etc. are a player-card creation feature only. (Confidence ~0.8; an in-game
  encyclopedia capture of a Special CPU horse would settle whether it visually varies.)
- **Silk color hues (15) and hood names (full 0–63):** exact visual mapping is observational (no ROM
  string table). Need an in-game silk/hood selection screen capture to lock hues/names.
- **a1[8] mid-band families (141-143 etc.):** the card decoder claims ×2/×3 mirrors are the same
  named color, but the ROM enum only documents the 0xC0–0xDE high band. Whether the renderer actually
  treats 77 and 205 as identical "Bay" shades, or as light/dark variants, is unconfirmed (no real
  card uses the mid band). Recommend capturing one.
- **JP coat/silk storage location:** in cabinet nvram, undecoded (blocked on a JP nvram/hardware capture).

---

## 9. How verified (reproducible)
- Card decode: `C:/Users/johnr/AppData/Local/Temp/app_cards.py` — decoded 9 real US cards, printed
  coat/silk/hood/sex; all self-consistent with the tool.
- ROM coat enum: `app_coatmap2.py` cross-checked ROM `+22` byte vs the DB "Coat" column for all 244
  horses → 1:1 {192:Chestnut … 222:Special}. `app_rom.py` dumped distributions for all 4 ROMs.
- ROM name tables: `app_strtab.py` extracted null-delimited strings at 0x0C68F0 (simple) and 0x0EE0AC
  (granular), plus the adjacent MALE/FEMALE/GELDING sex strings.
- Special-selector hunt: `app_special.py` / `app_hx.py` — per-column analysis of the 21 coat=222
  records proved no special-variant byte exists in the CPU record.
- JP: `app_jp.py` — 8 JP cards confirmed identity-only (no SEGA marker, tracks 2-3 zero).

## 10. Tool ideas this unlocks
- **Coat/silk/hood preview swatch** in the Stable card editor: render a small horse silhouette +
  jockey silk using the decoded enums so users see the actual coat/silk before saving.
- **`getColorName` fix:** patch the Card-Creator to use the verified ROM coat enum (§2e) and the full
  special sub-id table, eliminating the latent 202/204/207 mislabel.
- **CPU-roster coat exporter:** regenerate the DB "Coat" column byte-exact (already proven 244/244)
  and tag which CPU horses are "Special".
- **Silk/coat capture harness:** drive the in-game appearance-select screen in Flycast to label the
  15 silk hues, the full 0–63 hood set, and confirm whether CPU "Special" coats visually differ.
