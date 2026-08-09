# Tracks, Courses, and the G1 Calendar (KEY: tracks-races)

Derby Owners Club (Sega NAOMI). Decode of the track / course string tables, the
special- and handicap-race tables, and the G1 race-name list across all four
program ROMs. All offsets below were extracted from the real `.ic22` bytes and
are shown with representative decoded values and the exact method of confirmation.

ROMs:
- WE Rev C  `drbyocwc` `epr-22336c.ic22`  sig@0x8000 `dc99020c9cc8210c`
- WE EX Rev D `derbyocw` `epr-22336d.ic22` sig@0x8000 `09004ad20ee347d0`
- DOC 2000 JP `derbyo2k` `epr-22284a.ic22` sig@0x8000 `162f047ffcf5fcf6`
- DOC '99 JP  `derbyoc`  `epr-22099b.ic22` sig@0x8000 `188bee02f15328 38`

(8000 sig for drbyocwc verified this session: `d[0x8000:0x8008].hex()` == `dc99020c9cc8210c`.)

---

## 1. Big picture: these are DISPLAY string tables, not binary race records

Every "track / course / special / handicap / G1" table located so far is a table
of **null-terminated display strings**, not a packed binary record array. There is
NO leading attribute byte (grade / surface / distance / prize) per entry. The
surface (TURF/DIRT) and distance (e.g. `1600M`) are encoded *inside the string
text itself*. Confirmed by raw-hex dump of the WE course list: entries are pure
ASCII separated only by `00`, e.g.

```
0x0C6940 45 41 53 54 45 52 4E 20 43 49 54 59 20 54 55 52 46 20 31 36 30 30 4D 00   "EASTERN CITY TURF 1600M\0"
```

Implication: the *gameplay* race attributes (actual grade, prize purse, entry
conditions, schedule slot) live in a separate binary schedule table that is NOT
these string blocks. A candidate binary region was found immediately after the JP
handicap strings (o2k `0x0CAD7B+`, see Section 6) with `xx04` byte markers and
repeating `80 3f ff ff ff ff 05 24` patterns — likely the race/ranking metadata —
but it is not yet decoded. The string tables drive only the on-screen labels.

The JP text stream uses inline format/control codes between strings:
`0x0f` (line/format), `0xff 0x0f`, and multi-byte run-ins like `f0 8b f0` /
`fb cd f8`. These are display control codes, NOT per-race data. The same is true
of the English `00 ff 0f` / `00 0f` separators inside the G1 block.

---

## 2. WE (Rev C `drbyocwc`) — region map (verified)

| Section | Start | End (excl.) | Entries | Stride |
|---|---|---|---|---|
| Course List (compact) | `0x0C6940` | `0x0C6CA0` | 36 | variable, ~24B null-term |
| G1 Race Names | `0x0C6CA0` | `0x0C6DF0` | 20 strings (19 races + "NO NAME") | variable |
| Course Display Names (aligned) | `0x0C6DF0` | `0x0C70C8` | 26 | variable, ~28B |
| Special Races | `0x0C70C8` | `0x0C7248` | 12 | 32B aligned |
| Handicap Races | `0x0C7248` | `0x0C73C8` | 12 | 32B aligned |

Immediately *before* the course list (`0x0C6900..0x0C6940`) are coat-color and sex
labels: `BLACK`, `BAY`, `BROWN`, `SPECIAL`, `WHITE`, `MALE`, `FEMALE`, `GELDING`
(useful for the racing-table coat/sex decode in the horses subsystem).

### Course List (36 courses, 6 venues, verified counts)
Venue breakdown (confirmed by parsing the block):
EASTERN CITY 7, WESTERN HILL 6, NORTHERN PARK 6, CENTRAL CITY 8, SEGA 5, SOUTHERN PARK 4.
Surfaces: **26 TURF + 10 DIRT** = 36 total.

Full list (drbyocwc), offset / text:
```
0x0C6940 EASTERN CITY TURF 1600M     0x0C6A78 NORTHERN PARK TURF 1200M
0x0C6958 EASTERN CITY TURF 1400M     0x0C6A94 NORTHERN PARK TURF 1600M
0x0C6970 EASTERN CITY DIRT 1600M     0x0C6AB0 NORTHERN PARK TURF 1800M
0x0C6988 EASTERN CITY TURF 2000M     0x0C6ACC NORTHERN PARK TURF 2000M
0x0C69A0 EASTERN CITY TURF 2400M     0x0C6AE8 NORTHERN PARK TURF 2500M
0x0C69B8 EASTERN CITY DIRT 1200M     0x0C6B04 NORTHERN PARK DIRT 1800M
0x0C69D0 EASTERN CITY DIRT 2100M     0x0C6B20 CENTRAL CITY TURF 1200M
0x0C69E8 WESTERN HILL TURF 1200M     0x0C6B38 CENTRAL CITY TURF 1400M
0x0C6A00 WESTERN HILL TURF 1600M     0x0C6B50 CENTRAL CITY TURF 1600M
0x0C6A18 WESTERN HILL TURF 2000M     0x0C6B68 CENTRAL CITY TURF 2000M
0x0C6A30 WESTERN HILL TURF 2200M     0x0C6B80 CENTRAL CITY TURF 2200M
0x0C6A48 WESTERN HILL DIRT 1400M     0x0C6B98 CENTRAL CITY TURF 3000M
0x0C6A60 WESTERN HILL DIRT 1200M     0x0C6BB0 CENTRAL CITY TURF 3200M
                                     0x0C6BC8 CENTRAL CITY DIRT 1200M
0x0C6BE0 SEGA TURF 2400M             0x0C6C30 SOUTHERN PARK TURF 1200M
0x0C6BF0 SEGA DIRT 2000M             0x0C6C4C SOUTHERN PARK TURF 1800M
0x0C6C00 SEGA TURF 1600M             0x0C6C68 SOUTHERN PARK TURF 2000M
0x0C6C10 SEGA TURF 1800M             0x0C6C84 SOUTHERN PARK DIRT 1700M
0x0C6C20 SEGA DIRT 1400M
```

### G1 Race Names (drbyocwc 0x0C6CA0..0x0C6DF0) — 20 strings, 19 real races
```
WINTER STAKES        DOC 1000 GUINEAS    DOC 2000 GUINEAS    SPRING CLASSIC
AMERICAN DERBY       HONG KONG OAKS      HONG KONG DERBY     AMERICAN OAKS
SUMMER GRAND PRIX    SUPER DIRT GRAND PRIX   NO NAME (placeholder)
STAYERS STAKES       QUEEN ELIZABETH [+] CUP  MILE CHAMPIONSHIP
JAPAN CUP            SPRINTERS STAKES    DERBY OWNERS CUP    JAPAN CUP DIRT
SPRINTERS TROPHY
```
Notes verified from raw hex:
- `QUEEN ELIZABETH ` and `CUP` are two ASCII runs separated by bytes `bb f7`
  (a wide-glyph/center code in the text stream); they are ONE race name.
- `NO NAME` (`0x0C6D54`) is a live placeholder slot, not removed.
- Separators observed: leading `00` (string terminator) followed by format codes
  `0f`, `ff 0f`, or `0f ff 0f`. These are display controls, not attributes.

### Course Display Names (the aligned, double-space variant) — WE QUIRK
Only **26** of the 36 courses appear in this aligned table:
EASTERN CITY 7, WESTERN HILL 6, CENTRAL CITY 8, SEGA 5. **NORTHERN PARK and
SOUTHERN PARK are intentionally absent** from the WE aligned display table
(verified by scanning `0x0C6DF0..0x0C70C8`). Those two venues fall back to the
compact Course List string on the relevant screen. The JP ROMs (Section 5/6) DO
include all 6 venues in their aligned table, so this is a Western-Edition-specific
omission. Example aligned format: `"SEGA          TURF 2400M"` (venue left-padded
to a fixed column).

### Special Races (12) and Handicap Races (12)
Both blocks are exactly **6 venues x 2 surfaces** (TURF/DIRT). Ordering is the
full venue set: EASTERN CITY, WESTERN HILL, NORTHERN PARK, CENTRAL CITY, SEGA,
SOUTHERN PARK.
```
Special  (0x0C70C8): "<VENUE pad> TURF (SPECIAL)"  / "... DIRT (SPECIAL)"
Handicap (0x0C7248): "<VENUE pad> TURF (HANDICAP)" / "... DIRT (HANDICAP)"
```

---

## 3. WE EX (Rev D `derbyocw`) — identical content, shifted -0x6E0

Verified by dumping the shifted regions:

| Section | Rev D Start |
|---|---|
| Course List | `0x0C6260` (36 entries, byte-identical strings) |
| G1 Names | `0x0C65C0` (20 strings, identical text) |
| Display Names | `0x0C6710` |
| Special Races | `0x0C69E8` |
| Handicap Races | `0x0C6B68` |

The Rev D course list and G1 list decode to exactly the same 36 courses / 20 G1
strings as Rev C; only the file offsets differ (uniform `-0x6E0` shift vs Rev C).
This matches DOC-ROM-Studio.html `SPEC_REVD.trackSecs`.

---

## 4. Venue name localization map (EN <-> JP)

The Western Edition renamed the real Japanese racecourses to fictional American
ones. Confirmed by 1:1 positional alignment of the course lists (same venue order,
same distances, same surfaces):

| WE venue | JP venue (kanji / reading) | Real-world track |
|---|---|---|
| EASTERN CITY | 東京 (Tokyo) | Tokyo Racecourse |
| WESTERN HILL | 阪神 (Hanshin) | Hanshin |
| NORTHERN PARK | 中山 (Nakayama) | Nakayama |
| CENTRAL CITY | 京都 (Kyoto) | Kyoto |
| SEGA | セガ (Sega) | fictional Sega track |
| SOUTHERN PARK | 中京 (Chukyo) | Chukyo |

Surface tokens: 芝 (shiba) = TURF, ダート (daato) = DIRT. Distances are full-width
digits + `Ｍ` (e.g. `１６００Ｍ`).

---

## 5. DOC 2000 JP (`derbyo2k`) — full structure (EUC-JP)

Located by searching for EUC-JP `ダート` (dirt = `a5c0a1bca5c8`). Tables, in order:

| Section | Start | Entries |
|---|---|---|
| (coat/sex labels) | `0x0CA300` | 黒鹿毛, 特殊, 牝（メス）, せん, ... |
| Course List (compact) | `0x0CA335` | 36 |
| G1 Race Names | `0x0CA62D` | **21** |
| Course Display Names (aligned, all 6 venues) | `0x0CA7AB` | 36 |
| Special Races (特別レース) | `0x0CAB0B` | 12 (incl. 中京/Chukyo) |
| Handicap Races (ハンデ) | `0x0CAC5B` | 12 (incl. 中京/Chukyo) |
| binary race/ranking metadata | `0x0CAD7B+` | not decoded; `xx04` + `80 3f ff ff ff ff 05 24` rows, then 獲得賞金ランキング (prize-money ranking) text |

JP course list (36) decodes to the same venue/distance set as WE, e.g.
`東京芝１６００Ｍ`, `東京ダート１６００Ｍ`, ... `中京ダート１７００Ｍ`.

### JP G1 list (o2k, 21 races) — the canonical JRA G1 calendar
```
1  フェブラリーステークス  February Stakes      (WINTER STAKES)
2  桜花賞                 Oka Sho              (DOC 1000 GUINEAS)
3  皐月賞                 Satsuki Sho          (DOC 2000 GUINEAS)
4  天皇賞（春）            Tenno Sho (Spring)   (SPRING CLASSIC)
5  ＮＨＫマイルカップ       NHK Mile Cup         (AMERICAN DERBY?)
6  オークス（優駿牝馬）     Yushun Himba (Oaks)  (HONG KONG OAKS)
7  日本ダービー（東京優駿）  Japanese Derby       (HONG KONG DERBY)
8  安田記念               Yasuda Kinen         (AMERICAN OAKS)
9  宝塚記念               Takarazuka Kinen     (SUMMER GRAND PRIX)
10 スーパーダートグランプリ  Super Dirt GP        (SUPER DIRT GRAND PRIX)
11 秋華賞                 Shuka Sho            (STAYERS STAKES?)
12 天皇賞（秋）            Tenno Sho (Autumn)
13 菊花賞                 Kikuka Sho
14 エリザベス女王杯         Queen Elizabeth II Cup (QUEEN ELIZABETH CUP)
15 マイルチャンピオンシップ  Mile Championship    (MILE CHAMPIONSHIP)
16 ジャパンカップ          Japan Cup            (JAPAN CUP)
17 スプリンターズステークス  Sprinters Stakes     (SPRINTERS STAKES)
18 有馬記念               Arima Kinen          (DERBY OWNERS CUP)
19 ダービーオーナーズカップ  Derby Owners Cup     (DERBY OWNERS CUP)
20 ジャパンカップダート     Japan Cup Dirt       (JAPAN CUP DIRT)
21 高松宮記念             Takamatsunomiya Kinen (SPRINTERS TROPHY)
```
(The EN<->JP G1 mapping above is positional/best-effort; the WE list reorders and
renames, and collapses 安田/宝塚/菊花 differently — see open questions. The JP G1
count is 21 vs the WE 20 raw strings / 19 real races.)

Some JP G1 entries carry a leading `錏` (0x錏 EUC artifact = the inline format
byte `0xff` mis-decoded) e.g. `錏安田記念`, `錏宝塚記念`, `錏菊花賞` — the `錏`
is the stray control byte, the real name follows.

---

## 6. DOC '99 JP (`derbyoc`) — the SMALLER original (key version difference)

Located by searching EUC-JP `ダート`. The '99 original has a reduced track set:

| Section | Start | Entries |
|---|---|---|
| Course List (compact) | `0x0BD875` | **30** (NO Chukyo venue; SEGA has only 3) |
| G1 Race Names | `0x0BDAD5` | **20** (no 高松宮記念, no ジャパンカップダート) |
| Course Display Names (aligned) | `0x0BDC2D` | 30 |
| Special Races (特別レース) | `0x0BDEE7` | **10** (5 venues x 2; no Chukyo) |
| Handicap Races | — | **NONE** (handicap table does not exist in '99) |

Confirmed differences vs DOC 2000 / WE:
- **No 中京 (Chukyo / SOUTHERN PARK) venue at all.** Course list jumps 京都→セガ→(G1).
- **SEGA has only 3 courses** in '99: `セガ芝１６００Ｍ`, `セガ芝１８００Ｍ`,
  `セガダート１４００Ｍ` (vs 5 in DOC 2000: adds 芝2400 + ダート2000).
- **No Handicap races.** After the 10 special-race strings (`0x0BDEE7..0x0BE00E`)
  the bytes go straight into unrelated binary + item text (メンコ=mask,
  シャドーロール=shadow-roll). Handicap races were added in DOC 2000.
- **G1 list = 20** and ends at `ダービーオーナーズカップ` (Derby Owners Cup);
  it lacks `高松宮記念` and `ジャパンカップダート` which DOC 2000 added.

So the lineage is:
DOC '99 (30 courses / 5 venues / 20 G1 / no handicap)
  → DOC 2000 (36 courses / 6 venues incl. Chukyo / 21 G1 / +handicap table)
  → World Edition Rev C/D (same 36 courses, English venue names, 20 G1 strings,
    handicap table present, but aligned display table drops NORTHERN/SOUTHERN).

---

## 7. Cross-version summary table

| Metric | derbyoc '99 | derbyo2k 2000 | drbyocwc Rev C | derbyocw Rev D |
|---|---|---|---|---|
| Courses | 30 | 36 | 36 | 36 |
| Venues | 5 | 6 | 6 | 6 |
| SEGA courses | 3 | 5 | 5 | 5 |
| Chukyo/SOUTHERN | no | yes | yes | yes |
| G1 races | 20 | 21 | 20 strings (19) | 20 strings (19) |
| Special races | 10 | 12 | 12 | 12 |
| Handicap races | none | 12 | 12 | 12 |
| Course-list start | 0x0BD875 | 0x0CA335 | 0x0C6940 | 0x0C6260 |
| G1 start | 0x0BDAD5 | 0x0CA62D | 0x0C6CA0 | 0x0C65C0 |
| Aligned display venues | all 5 | all 6 | 4 (no N/S Park) | 4 (no N/S Park) |
| Text encoding | EUC-JP | EUC-JP | ASCII | ASCII |

---

## 8. How verified
- All string contents pulled with a `parseBlock`-equivalent Python scanner over
  the exact ROM byte ranges (skip non-printable, read printable run, on EN; for JP
  read null-delimited chunks and `euc-jp`-decode after stripping the 1–3 leading
  inline-format control bytes).
- Venue/surface counts produced by `Counter` over the parsed course list.
- Raw-hex dumps confirmed: no per-entry binary attribute byte in the EN tables;
  attributes (TURF/DIRT, distance) are literal text.
- Rev D offsets confirmed by dumping the `-0x6E0`-shifted ranges and matching text.
- JP tables located by EUC-JP needle search for `ダート` (`a5c0a1bca5c8`) and
  `ターフ`/`芝`, then walking forward.
- 0x8000 build signature re-confirmed for drbyocwc.

---

## 9. Open questions / not-yet-decoded
1. **Binary race-schedule / G1-calendar table.** The string tables are display
   only. The actual per-race binary (grade enum, surface flag, distance value,
   prize purse, month/slot, entry conditions, which courses host which G1) is
   elsewhere. Strong candidate: o2k `0x0CAD7B+` region of `xx04` markers and
   `80 3f ff ff ff ff 05 24` rows just after the handicap strings. Needs decode.
2. **G1 -> course binding.** Which physical course (venue+distance+surface) each
   G1 runs on is not in the name table. Likely in the binary schedule table.
3. **EN<->JP G1 exact mapping.** WE has 19 real races + "NO NAME"; JP has 21. The
   collapse/rename (e.g. which JP race became SUMMER GRAND PRIX vs STAYERS STAKES)
   is positional-guess only; confirm against in-game calendar order.
4. **The `bb f7` inside QUEEN ELIZABETH CUP** — identify the exact glyph/control.
5. **DOC II (8-satellite)** variant not covered here (different ROM/rig).

---

## 10. Tool ideas this unlocks
- **Course/Race table editor tab** (extend DOC-ROM-Studio): already parses these
  4 sections; add the JP offsets (o2k `0x0CA335 / 0x0CA62D / 0x0CA7AB / 0x0CAB0B`
  and oc `0x0BD875 / 0x0BDAD5 / 0x0BDC2D / 0x0BDEE7`) and the version-aware counts
  (oc: 30 courses, no handicap) so the JP ROMs get the same Tracks/G1 tabs as EN.
- **Cross-version diff report** of the track set (auto-generate Section 7 table
  from any 4 ROMs) — proves provenance and catches edited ROMs.
- **G1 calendar visualizer** once the binary schedule table (open Q1) is decoded:
  map each G1 to month + course + distance + purse.
- **Localization map exporter** (Section 4) for the card-creator UI so JP and EN
  course pickers stay in sync.
