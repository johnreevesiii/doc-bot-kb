# Derby Owners Club — Systematic 4-Version Diff (version-diff)

Definitive cross-version offset + roster map for the four 4 MB NAOMI program ROMs.
Every offset/format claim below was extracted from the real `.ic22` bytes (see "how verified").

## 0. The four ROMs

| key   | folder    | file              | size    | edition                       |
|-------|-----------|-------------------|---------|-------------------------------|
| revc  | drbyocwc  | epr-22336c.ic22   | 4194304 | World Edition Rev C (US, BR reader) |
| revd  | derbyocw  | epr-22336d.ic22   | 4194304 | World Edition EX Rev D (Export, BR reader) |
| o2k   | derbyo2k  | epr-22284a.ic22   | 4194304 | DOC 2000 (Japan, LR reader)   |
| oc    | derbyoc   | epr-22099b.ic22   | 4194304 | DOC '99 Original (Japan, LR reader) |

All four are exactly 4,194,304 bytes, uncompressed.

## 1. NAOMI header / build markers (offset 0x000–0x160) — VERIFIED

Standard Sega NAOMI cartridge header. Fields confirmed by byte extraction:

- **0x000**: `"NAOMI"` (16-byte padded) — all 4.
- **0x010**: `"SEGA ENTERPRISES,LTD."` (publisher, 32-byte) — all 4.
- **0x030 + i*0x20**: 8 region title strings (32 bytes each). These distinguish WE vs JP:
  - revc/revd `[0]`,`[1]` = `"DERBY OWNERS CLUB WE ---------"`
  - o2k/oc   `[0]`,`[1]` = `"DERBY OWNERS CLUB ------------"` (no "WE")
  - `[2]`=`IN EXPORT`, `[3]`=`IN KOREA`, `[4]`=`IN AUSTRALIA`, `[5..7]`=`IN ? / ! / @` placeholders — identical across all 4.
- **0x130**: 4-byte **build date**, little-endian `year(u16) month(u8) day(u8)`:
  - revc & revd: `d1 07 0a 1e` → 0x07d1=2001, 10, 0x1e=30 → **2001-10-30**
  - o2k & oc:    `cf 07 0a 01` → 0x07cf=1999, 10, 0x01 → **1999-10-01**
  - NOTE: Rev C and Rev D share the *same* header date/serial — the header was not re-stamped for the Rev D rebuild. Disambiguate revc/revd only by the 0x8000 signature or the table offsets.
- **0x134**: 4-char **game serial**:
  - revc/revd = `"BEF0"`
  - o2k = `"BBX0"`
  - oc  = `"BAX0"`
  (Sega NAOMI game numbers; the 22336/22284/22099 EPR numbers are the mask-ROM part numbers.)
- **0x144**: `03 00 00 00` (region/format word) — identical all 4.

### Build signature @ 0x8000 (16 bytes) — VERIFIED, the canonical version fingerprint
The boot region is never touched by table editors, so this is the edit-proof discriminator.
First 8 bytes match the handoff exactly; full 16 bytes:
```
revc: dc 99 02 0c 9c c8 21 0c  e0 b8 21 0c 80 b6 21 0c
revd: 09 00 4a d2 0e e3 47 d0  2b 42 32 20 33 88 01 8b
o2k:  16 2f 04 7f fc f5 fc f6  0b 4a fc f4 86 90 b6 f3
oc:   18 8b ee 02 f1 53 28 38  5a 48 2c 73 2d f3 3c fd
```
*How verified:* `vdiff.py` / `vdiff3.py` dumped 0x000–0x160 and 0x8000 for all 4.

## 2. Racing stat table (244 horse records) — VERIFIED + EXTENDED

Record anchors (record-start convention, matches handoff):

| ver  | record-start | stride | end of 244 block |
|------|--------------|--------|------------------|
| revc | 0x108E03     | 32     | 0x10AC83         |
| revd | 0x10A14B     | 32     | 0x10BFCB         |
| o2k  | 0x10AD1B     | 32     | 0x10CB9B         |
| oc   | 0x0F6902     | 28     | 0x0F83A2         |

(ROM-Studio's `recBase` = record-start + 9; its RACING_F field offsets are relative to that anchor. Both conventions reconciled below.)

### MAJOR FINDING — the 32-byte stat table is byte-identical across revc, revd, AND o2k
- `revc == revd` for all 244 records: **TRUE** (byte-exact).
- `revc == o2k` for all 244 records: **TRUE** (byte-exact).
So WE Rev C, WE EX Rev D, and JP DOC 2000 ship the **same horse stat data**; they differ only in the NAME tables, dialogue text, and code. Only **oc (DOC '99)** uses a different 28-byte layout and different stat values.
*How verified:* `vfull.py` compared all 244 records pairwise.

### Full 32-byte field map (offsets from record-start) — EXTENDED (was ~10/32 mapped)
Derived from per-column variance over all 244 records (`vfull.py`, `vmap.py`):

| off  | hex | field            | range / decode                                   | confidence |
|------|-----|------------------|--------------------------------------------------|------------|
| +0   | 00  | const 0          | always 0                                         | high |
| +1   | 01  | flag A           | 0/1/2 (175/62/7 horses) — class/availability flag| med |
| +2   | 02  | **horse ID lo**  | = (index+1), 1..244, ==+3                         | high |
| +3   | 03  | **horse ID hi/dup** | == +2 for all 244                              | high |
| +4   | 04  | const 0          |                                                  | high |
| +5   | 05  | **dirt aptitude**| 0..255 (the "AC"/dirt byte)                       | high |
| +6   | 06  | const 0          |                                                  | high |
| +7   | 07  | const 0          |                                                  | high |
| +8   | 08  | **grade**        | 0=Ungraded,1=G3,2=G2,3=G1 (80/65/50/49)           | high |
| +9   | 09  | **start** (ext)  | ~11..63                                           | high |
| +10  | 0a  | **corner** (ext) | ~14..59                                           | high |
| +11  | 0b  | **oob/pace** (ext)| ~4..63                                           | high |
| +12  | 0c  | **competing** (ext)| ~8..63                                          | high |
| +13  | 0d  | **tenacious** (ext)| ~3..62                                          | high |
| +14  | 0e  | **spurt** (ext)  | ~4..63                                            | high |
| +15  | 0f  | const 0          |                                                  | high |
| +16  | 10  | flag B           | 0/1/2 (200/37/7) — pairs with flag A; growth/avail| med |
| +17–20| 11–14 | const 0     |                                                  | high |
| +21  | 15  | category         | 0/1/2/3/7 (30/98/69/44/3) — leg-type or size class| med |
| +22  | 16  | **coat color**   | 207 LtGray,204 DkGray,202 Bay,222 Special,199 Brown,192/193 Chestnut/Black | high |
| +23  | 17  | unknown U1       | 0..255, steps of 16 (0,14,80,128,140,242) — hi nibble = some grade/affinity index | low |
| +24  | 18  | unknown U2       | clusters 0x30/0xa0/0xa1/0xa5/0xaf (48,160–175) — looks like a growth/maturity code | low |
| +25  | 19  | **seq ID**       | (index+1)&0xFF, 1..243,0                          | high |
| +26–28| 1a–1c | const 0     |                                                  | high |
| +29  | 1d  | **stamina** (internal)| 0..60                                        | high |
| +30  | 1e  | **speed** (internal)  | 0..63                                        | high |
| +31  | 1f  | **sharp** (internal)  | 0..60                                        | high |

Mapped fields raised from ~10 to ~22 of 32 bytes. Remaining genuine unknowns: **+1, +16, +21, +23, +24** (5 variable bytes; the rest are confirmed-constant padding). Hypotheses: +1/+16 = a 2-flag growth/availability pair; +21 = leg-type or body-size class (5 buckets); +23/+24 = a paired hidden-ability / growth-curve code (U2 clusters near 0xA0 like a "B/A growth" enum).

### oc (DOC '99) 28-byte layout — VERIFIED
oc = the 32-byte record with the leading const-0 and one interior pad removed; externals/internals land at the same *relative* spot:

| off  | field           |
|------|-----------------|
| +1,+2 | horse ID (1..244, dup) |
| +4   | dirt aptitude   |
| +7   | grade           |
| +9..+14 | start/corner/oob/competing/tenacious/spurt |
| +18  | a `02`/`01` flag (= revc +0x15 region) |
| +19  | coat (`cf`/`cc`...) |
| +24,+25,+26 | stamina/speed/sharp (internals) |

*How verified:* `voc.py` aligned oc h1 `01 01 01 00 a8 00 00 03 00 2c 23 13 20 28 2e ... 17 25 30` against revc h1; externals (2c 23 13 20 28 2e) and internals (17 25 30) match byte-for-byte.

## 3. Racing NAME tables — VERIFIED

| ver  | name base   | stride | encoding   |
|------|-------------|--------|------------|
| revc | 0x10AD50    | 18     | ASCII, 0x00-terminated |
| revd | 0x10C098    | 18     | ASCII      |
| o2k  | 0x10CC68    | 18     | EUC-JP, 0x00-terminated |
| oc   | 0x0F8480    | 18     | EUC-JP     |

JP name-table bases were *found*, not previously documented: walk back in 18-byte EUC-JP-valid steps from the known marker `トロットサンダー` (o2k @0x10d3ca, oc @0x0f8be2) to the first record. o2k base 0x10CC68 sits right after its padded stat block; oc base 0x0F8480.
*How verified:* `vjp2.py` / `vjp3.py` decoded clean katakana (`アイオーユー`, `アインブライド`, … `マチカネキンノホシ` for o2k #244).

## 4. ROSTER DIFFERENCES (the headline) — VERIFIED

### WE Rev C → Rev D: exactly 16 racing names changed (`vname.py`)
Real-world racehorse names in Rev C were swapped for fictional names in the Export Rev D build (licensing), plus 2 typo/case fixes:

| # | Rev C | Rev D |
|---|-------|-------|
| 6 | Gold fighter | Gold Fighter (case) |
| 27 | El Condor Pasa | Steppin' Out |
| 101 | Helissio | Kingston's Fury |
| 102 | End Sweep | Heart Breaker |
| 110 | High-Rise | Mud Slinger |
| 111 | Sunday Silence | King Teige |
| 116 | Dream well | Strideright |
| 117 | Indigenous | Blink of an Eye |
| 123 | Oriental Express | Custom Design |
| 157 | French Deputy | Sunnyboy |
| 161 | Judge Angelucci | Break Away |
| 181 | Tomrrow's Dream | Tomorrow's Dream (typo) |
| 195 | Hector Protector | Blue Lou |
| 197 | Agnes World | Winnin' Easy |
| 201 | Brocco | Time Flies |
| 207 | Flower Dance | Mr. Vice President |

### WE Rev C → Rev D breeders: 26 sires + ALL 84 dams renamed (`vsd.py`)
The Export build replaced essentially the entire female (dam) breeder roster (84/84 differ) and 26 of 84 sires. Real names (Sunday Silence, Helissio, Tony Bin, Carnegie as sires; many dams) → generic fictional names (Big E, Maverick, Vinny, Pierogi Prince, etc.). Stride 60, externals at name-12 (8 meaningful bytes, 1–16 bands), 4-byte composite "ac" at name+36 (first byte = dirt aptitude 0–255). Sire/dam bases: revc 0x10BF1C / 0x10D2CC; revd 0x10D264 / 0x10E614.

### JP DOC '99 → DOC 2000: 64 racing names changed (`vjp3.py`)
Far larger refresh than the WE Rev C→D change. The first ~2 horses shared (アイオーユー, アインブライド) then heavy divergence — the 2000 edition rotated in newer real Japanese horses (テイエムオペラオー, アドマイヤベガ, ナリタトップロード, etc.) replacing the '99 roster (ナリタブライアン-era names). 64/244 differ. (See file for full list; representative: #3 アドラーブル→コクトジュリアン, #195 アラビアンナイト→テイエムオペラオー, #197 エミーローズ→アグネスワールド.)

## 5. G1 races & tracks — VERIFIED

- **G1 race names**: revc 0x0C6CA0..0x0C6DF0, revd 0x0C65C0..0x0C6710 (shifted −0x6E0). **Identical content** between Rev C and Rev D: WINTER STAKES, DOC 1000/2000 GUINEAS, SPRING CLASSIC, AMERICAN DERBY/OAKS, HONG KONG OAKS/DERBY, SUMMER GRAND PRIX, SUPER DIRT GRAND PRIX, STAYERS STAKES, QUEEN ELIZABETH CUP, MILE CHAMPIONSHIP, JAPAN CUP, SPRINTERS STAKES, DERBY OWNERS CUP, JAPAN CUP DIRT, SPRINTERS TROPHY. (Strings separated by `0x0f`/`0xff` color/format control bytes.)
- **Track tables**: revc 4 sections from 0x0C6940 (Course List, Display Names, Special Races, Handicap Races); revd shifted −0x6E0 (from 0x0C6260). Course list identical in content (EASTERN CITY / WESTERN HILL / NORTHERN PARK / CENTRAL CITY / SEGA / SOUTHERN PARK, turf/dirt, 1200–3200M). The revd −0x6E0 shift is the same delta seen on G1 and the racing tables (Rev D's data block is relocated lower because dialogue text grew).

## 6. Game text / dialogue — partial (see other areas)
- revc: 26 curated null-/0x0A-delimited ASCII blocks (offsets in DOC-ROM-Studio.html GAMETEXT).
- revd: dialogue edited and relocated; ROM-Studio falls back to a scanned list over 5 regions (0x0C7000-0xCB000, 0xE7000-0xEF000, 0x103000-0x10A000, 0x10F300-0x113000, 0x127000-0x12F000).
- o2k/oc: EUC-JP text across 0x0C0000–0x140000 / 0x0B0000–0x130000.
- Breeding "mater" names: 167 EUC-JP in jp_mater_names.json (cross-validated against decoded JP cards).

## 7. Item/feeding effects table — NEW
Diffing the edited `beer_effects_test.ic22` vs base drbyocwc (`vbeer.py`) shows only **12 changed bytes at 0x1671FC–0x167201 and 0x167228–0x16722D** (two groups of 6, base 00 → 02 / 04). This locates the feeding/item-effect table near **0x167200**, entirely separate from the stat/name/breeder tables. Worth a dedicated RE pass (see "items" area).

## 8. Cross-version offset master table

| structure        | revc        | revd        | o2k         | oc          |
|------------------|-------------|-------------|-------------|-------------|
| build sig        | 0x8000      | 0x8000      | 0x8000      | 0x8000      |
| date (hdr)       | 2001-10-30  | 2001-10-30  | 1999-10-01  | 1999-10-01  |
| serial           | BEF0        | BEF0        | BBX0        | BAX0        |
| racing stats     | 0x108E03/32 | 0x10A14B/32 | 0x10AD1B/32 | 0x0F6902/28 |
| racing names     | 0x10AD50/18 | 0x10C098/18 | 0x10CC68/18 | 0x0F8480/18 |
| sire (84)        | 0x10BF1C/60 | 0x10D264/60 | TBD         | TBD         |
| dam (84)         | 0x10D2CC/60 | 0x10E614/60 | TBD         | TBD         |
| G1 names         | 0x0C6CA0    | 0x0C65C0    | TBD         | TBD         |
| tracks (sec1)    | 0x0C6940    | 0x0C6260    | TBD         | TBD         |
| item/feed effects| ~0x167200   | (same?)     | TBD         | TBD         |

## 9. Open questions
- Decode racing-record +1, +16, +21, +23, +24 (the 5 unmapped variable bytes). Likely growth-type / leg-type / hidden-ability. Cross-ref a known horse's in-game growth label vs these bytes.
- Locate o2k/oc sire & dam tables and confirm whether the JP breeder roster matches the WE one (the 32-byte stat table already proves stats are shared o2k==revc).
- Confirm the item/feed table layout at 0x167200 (what each slot/byte means; does it exist at the same offset in revd/o2k/oc?).
- JP on-card stats/sex/leg-type: stats are NOT in the racing table on the card; the racing-table identity (o2k==revc) suggests the cabinet keys career data by horse ID — confirm against nvram.

## 10. Tool ideas this unlocks
- **Version fingerprinter**: read 0x8000 (16B) + 0x130 date + 0x134 serial → exact build ID, even on edited ROMs.
- **Roster diff viewer**: side-by-side 244 racing names + 84+84 breeders across all 4, highlighting the 16 (WE C→D) / 64 (JP '99→2000) / 110 breeder changes.
- **Unified horse editor**: since revc/revd/o2k share the 32-byte stat table byte-for-byte, one editor writes stats to all three (only the name table offset changes per version); oc needs the 28-byte packer.
- **Cross-version patch transposer**: apply a stat edit made on revc to revd/o2k automatically (same bytes, different base) and to oc (re-pack 32→28).
