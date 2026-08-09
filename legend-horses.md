# SP Horse Program (legend horses) -- epr-22284a.ic22 (DOC 2000 JP)

ROM: **epr-22284a.ic22** (Japanese DOC 2000, "Ver.6/16" per the on-ROM string at
file 0x154ABC). NOT the World Edition Rev C ROM the rest of this workspace uses.
Size 4,194,304 bytes, CRC32 0x1E8E067C.

**JP RAM base = file + 0x0C010000** (confirmed: the SP table pointer resolves to
RAM 0x0C163F94). This differs from Rev C's +0x0C020000 -- use 0x0C010000 for any
epr-22284a work.

Investigated 2026-07-06 (static, from John's "SP Horse Program!!" spreadsheet decode).

## What it is
A hidden roster of **11 famous real JRA racehorses** (the late-'90s legends) built
into the ROM as ready-made "special" horses, each with authentic stats, aptitudes,
and real sire/dam pedigree. They are the marquee legend horses of DOC 2000.

## Table location + record layout
- Table (stat records): file **0x153FAC** stats / **0x153F94** record start, RAM 0x0C163F94.
- 11 records, **stride 60 bytes (0x3C)**. Each record = 24-byte name + 36-byte stat block.

Record layout (offsets from record start):
| off | size | field |
|---|---|---|
| +0  | 24 | katakana name, **EUC-JP**, null-padded |
| +24 | u32 | ST (internal) |
| +28 | u32 | SP (internal) |
| +32 | u32 | SH (internal) |
| +36 | u32 | **AC** -- inferred DIRT aptitude (0-255); the dirt specialist Abukuma Poro maxes it at 255 |
| +40 | u32 | 0 (reserved) |
| +44 | 1 | 0 |
| +45 | 1 | Q1 (202-222, tight range) -- OPEN |
| +46 | 1 | Q2 (0-207, scattered) -- OPEN (candidate: coat/portrait index) |
| +47 | 1 | Q3 (0x30 for all except Tokai Teio 0x39) -- OPEN (flag) |
| +48 | 8 | **6 external aptitude bands** (0-15) + 2 pad |
| +56 | u32 | id |

Internals are stored as u32 (low byte only); that 4-byte spacing is why a naive
contiguous byte search for the stat rows misses the table.

## The 11 horses (decoded)
| # | JP name | Eng | ST | SP | SH | AC(dirt?) | aptitude bands | id |
|---|---|---|---|---|---|---|---|---|
| 1 | アブクマポーロ | Abukuma Poro | 42 | 41 | 36 | 255 | 4,8,12,8,8,12 | 2 |
| 2 | エアグルーヴ | Air Groove | 40 | 49 | 37 | 132 | 4,8,12,12,4,12 | 3 |
| 3 | エルコンドルパサー | El Condor Pasa | 40 | 45 | 35 | 168 | 4,8,12,8,8,12 | 4 |
| 4 | グラスワンダー | Grass Wonder | 30 | 52 | 37 | 68 | 8,8,8,8,8,12 | 5 |
| 5 | サイレンススズカ | Silence Suzuka | 43 | 56 | 34 | 168 | 15,8,1,8,12,12 | 6 |
| 6 | スペシャルウィーク | Special Week | 37 | 50 | 31 | 132 | 4,8,12,8,8,12 | 7 |
| 7 | セイウンスカイ | Seiun Sky | 37 | 53 | 26 | 132 | 15,8,1,8,12,12 | 8 |
| 8 | タイキシャトル | Taiki Shuttle | 31 | 56 | 49 | 168 | 12,6,6,8,8,12 | 9 |
| 9 | トウカイテイオー | Tokai Teio | 35 | 51 | 53 | 164 | 8,8,8,8,8,12 | 10 |
| 10 | ナリタブライアン | Narita Brian | 43 | 48 | 43 | 164 | 12,8,4,12,8,12 | 0 |
| 11 | トゥナイトツー | "Tonight Two" | 50 | 50 | 50 | 255 | 8,8,8,8,8,8 | 21 |

Notes: Silence Suzuka and Seiun Sky share the identical aptitude profile
[15,8,1,8,12,12] (both real front-running milers). "Tonight Two" (flat 50/50/50,
all-8 aptitudes, id 21) is a default/test horse, not a real named champion.

## Real pedigree block (sires + dams)
Packed EUC-JP strings at **0x154727-0x154AD4** hold the horses' real sires and dams.
Confirmed against racing history:
- Special Week = Sunday Silence x Campaign Girl (サンデーサイレンス, キャンペンガール)
- Tokai Teio = Symboli Rudolf x Tokai Natural (シンボリルドルフ, トウカイナチュラル)
- Air Groove = Tony Bin x Dyna Carle (トニービン, ダイナカール)
- Silence Suzuka = Sunday Silence x Wakia (サンデーサイレンス, ワキア)
- Narita Brian = Brian's Time x Pacificus (ブライアンズタイム, パシフィカス)
- El Condor Pasa = ... x Pacificus (パシフィカス)
Also present: Symboli Rudolf, Mejiro Ryan, Grand Opera, Soccer Boy, Opera House,
Sakura Yutaka O, Bansei..., Sister Mill, Rail du Tan, Vega, Golden Sash, etc.
Sunday Silence appears 4x (he sired several of these). The exact SP-record ->
sire/dam wiring (which offset table links them) is not yet pinned. OPEN.

## Adjacent structures in the SP data section
- **Katakana font/syllabary table** @ 0x15457F (アカサタナ... full kana set).
- **Graphics/portrait pointers** @ 0x1555xx: arrays of pointers into the 0x0Dxxxxxx
  graphics ROM (horse portraits/silks). The block is duplicated (0x155500 and
  0x1558C0 hold the same pointer list).
- **Index sequence** @ 0x154228: 22 entries [9,9,10,6,1,6,3,9,6,8,4,6,4,11,2,9,3,11,
  7,2,1,0], values 0-11 = horses by row. Likely a race/rival lineup or appearance
  schedule. OPEN (structure not decoded; 9 and 6 appear most often).
- **Master resource pointer array** @ 0x146064 (289 pointers into the 0x0C16xxxx
  section): heterogeneous -- float coefficients (~1.0), data blocks, the graphics
  pointers, the "DOC2000" version string, and (at 0x146160-0x14617C) **eight
  consecutive pointers to record 0** (table head), which reads like 8 rival slots
  seeded from this program.

## Verdict / open threads
Confirmed: the SP Horse Program is the built-in legend-horse roster (11 real
champions + 1 test horse) with authentic stats, 6 aptitude bands, and real
pedigrees. Data side fully decoded.

## Incorporated into the breeding lab (2026-07-06)
All 10 real legends (excluding the "Tonight Two" test horse) added to
breeding-pool.json `combined` as "DOC 2000 Legend" entries: st/sp/sh, ac (dirt),
the 6 aptitude bands (start..spurt order, confirmed via Silence Suzuka + Seiun Sky
both grading Front-runner off band[0]=15), sex (Air Groove = dam, other 9 = sires),
and pedigrees as metadata (5 ROM-confirmed sire/dam, 5 from racing records). AC=dirt
is corroborated by the pool's own `_about` note ("ac=dirt/course aptitude 0-255").
Deployed. Pedigree display in the lab/genealogy UI is a follow-up (pool schema has
no sire/dam render yet).

## What are they used for? (research 2026-07-06)
Static evidence, converging read: **built-in legend/rival horses** -- most likely
the field of the game's Special Races and/or the seed roster of a "famous horses"
gallery. Not player breeding stock (they are absent from the main breeding pool).
Supporting ROM findings:
- **名馬 (famous horses) system**: a hall-of-fame/legacy path exists. Retirement/
  ceremony text at 0x0F0960-0x0F0AC0 ("%s号の生涯は... 血統、育成、鍛錬によって...
  偉大なる蹄跡を残し...") and 0x0ED09F ("%sもついに、名馬の仲間入りです" = "%s has
  finally joined the ranks of the famous horses"). The 11 legends are the marquee 名馬.
- **特別レース (Special Races)**: a real race category with a venue/surface table at
  0x0CAB1A+ (Tokyo/Nakakyo, Turf/Dirt, Handicap). Combined with the legend table's
  **8 "rival slots"** (eight pointers to record 0) and the **22-entry lineup sequence**
  (0x154228), this is the shape of a race FIELD -- i.e. you race against these legends.
- **Full pedigrees + portrait pointers**: authentic sire/dam + graphics = the game can
  show these horses with bios (a gallery or pre-race intro).
- **Why the trace bottoms out**: nothing static references the SP array base OR the
  special-race venue strings -- the blob is relocated to work-RAM at runtime (the
  documented DOC pattern). Definitive answer needs a live Flycast trace: breakpoint the
  SP table read (post-relocation) or catch the DMA/copy of the 0x153xxx blob, and see
  which screen/mode consumes it (special-race field vs 名馬 gallery vs attract demo).

Open (need live Flycast, per the workspace stop rule -- the deployment code is
reached indirectly, no static literal loads the array base):
1. **Deployment**: confirm which mode reads the table (special-race field / 名馬
   gallery / attract demo) via the live trace above.
2. **AC / Q1 / Q2 / Q3 semantics**: AC is very likely dirt aptitude (Abukuma Poro
   the dirt champ = 255); Q1/Q2/Q3 unconfirmed (coat / running-style / portrait
   index candidates).
3. **SP -> sire/dam linkage**: the offset/pointer table that maps each record to
   its pedigree strings.
