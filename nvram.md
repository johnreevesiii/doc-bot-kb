# Derby Owners Club — Cabinet Save State (EEPROM + SRAM) Decode

KEY: `nvram`  |  Status: substantially decoded (leaderboards, track records, bookkeeping header, EEPROM all mapped; per-record attribute bytes partially mapped)

Scope: the NAOMI cabinet battery save, NOT the player card (card = `card` area, 207 bytes). This is
the path to a **career / hall-of-fame dashboard** tool that reads what the cabinet remembers.

---

## 0. Files & sizes (verified)

From `C:/Users/johnr/Downloads/demul07_280418/nvram/` (this is a **Demul** nvram set):

| file | bytes | role |
|------|------:|------|
| `derbyocw.eeprom`  | 128   | JVS serial EEPROM (machine ID + dip/settings), **satellite** board |
| `derbyocw.sram`    | 32768 | BBSRAM battery save (bookkeeping + leaderboards), **satellite** board |
| `derbyocwm.eeprom` | 128   | JVS EEPROM, **master** board (`m` suffix) |
| `derbyocwm.sram`   | 32768 | BBSRAM battery save, **master** board |

`m` = the multiboard **master**; the non-`m` is a **satellite**. Confirmed by diffing the two SRAMs:
the bookkeeping header differs per board but the leaderboard payload is byte-identical (shared
cabinet-wide data). This matches the multiboard architecture in `BATTERY_RESUME_FINDINGS.md`
(Flycast uses `<rom>.nvmem` for the 32KB BBSRAM and `<rom>.eeprom` for the 128B JVS EEPROM).

Only the first **0x2998 (~10.6 KB)** of the 32 KB SRAM is used (last nonzero byte 0x2994);
the remaining ~22 KB is zero/reserved. Verified: total 32768, nonzero=4082.

---

## 1. EEPROM (128 bytes) — machine identity + dip settings

This is the **standard NAOMI JVS EEPROM**, not game career data. Layout (verified on both
`derbyocw.eeprom` and `derbyocwm.eeprom`, which are byte-identical):

```
000000  ef 12 10 42 45 46 30 18 00 1b 02 01 05 05 31 11   ...BEF0.......1.
000010  11 11 ef 12 10 42 45 46 30 18 00 1b 02 01 05 05   .....BEF0.......
000020  31 11 11 11 2a f1 28 28 2a f1 28 28 60 04 10 a3   1...*.((*.((`...
000030  44 45 52 42 59 20 4f 57 4e 45 52 53 20 43 4c 55   DERBY OWNERS CLU
000040  42 20 41 4d 33 02 00 00 fc c4 00 00 03 01 07 00   B AM3...........
000050  01 00 00 00 60 04 10 a3 44 45 52 42 59 20 4f 57   ....`...DERBY OW
000060  4e 45 52 53 20 43 4c 55 42 20 41 4d 33 02 00 00   NERS CLUB AM3...
000070  fc c4 00 00 03 01 07 00 01 00 00 00 ff ff ff ff   ................
```

Structure (NAOMI/Naomi JVS "system" + "game" EEPROM, mirrored two halves):
- `0x00` `ef 12` — JVS EEPROM CRC of the system block (mirrored at `0x12`).
- `0x02` `10 42 45 46 30` — contains the ASCII **`BEF0`** tag. This is the same family as the card
  signature `SEGABEF0` (the card sig in the `card` area is `SEGABEF0` at card 0x8A). `BEF` = the
  Sega game/region code; `BEF0` here is the JVS system serial tag.
- `0x07` `18 00 1b 02 01 05 05` + `0x0e` `31 11 11 11` — JVS system settings (coin/region/monitor
  flags). `0x18..0x29` = `2a f1 28 28 2a f1 28 28` — mirrored 4-byte block (per-coin-slot config).
- `0x2c` `60 04 10 a3` — **game EEPROM CRC / magic** (`a3 10 04 60` BE). Repeated at `0x54`.
- `0x30` ASCII **`DERBY OWNERS CLUB AM3`** — the game ID string ("AM3" = Sega AM3 dev team).
  Repeated at `0x58`.
- `0x45` `02 00 00 fc c4 00 00 03 01 07 00 01 00 00 00` — game dip block (settings). Mirrored at `0x6d`.
- `0x7c` `ff ff ff ff` — terminator/unused.

The whole 0x00-0x4f block is duplicated at 0x50-0x9f as the redundant copy (JVS standard:
two identical halves guarded by the leading CRC). **No money, no horses, no career data lives in
the EEPROM** — it is identity + dips only. Career/leaderboard data is entirely in the SRAM.

Note from `BATTERY_RESUME_FINDINGS.md`: on Flycast a `.eeprom` file only appears if the game writes
the EEPROM. In this Demul set the EEPROM is present (Demul always writes it). DOC writes it once at
init with the dips above.

---

## 2. SRAM (32 KB BBSRAM) — top-level layout (verified)

```
0x0000  bookkeeping header A (12 used bytes) + machine settings flags
0x0100  bookkeeping header B = mirror/backup of header A
0x01f8  ===== SAVE REGION 1 (primary) =====
0x01f8    region header / checksum (16 B, doubled at 0x208)
0x0218    rank-0 marker entry ("Big Shuttle")
0x0230    MONEY LEADERBOARD copy 1  — 50 records x 32 B  (0x0230..0x0850)
0x0870    MONEY LEADERBOARD copy 2  — 50 records x 32 B, with extra metadata (0x0870..0x0e90)
0x0f7c    TRACK-RECORD TABLE        — 57 records x 28 B  (0x0f7c..0x15b8)
0x15c4    region-1 trailer / per-board word
0x15c8  ===== SAVE REGION 2 (backup) =====  (same structure, delta +0x13d0)
0x15c8    region header / checksum
0x1634    money leaderboard copy 1
...       money leaderboard copy 2
0x2340    track-record table
0x2988    region-2 trailer
0x2998  end of used data; rest of SRAM is zero
```

Region 2 is a **redundant backup** of Region 1 (DOC writes two copies so a corrupt/interrupted
write can be recovered). Confirmed: "City Commandant" appears at 0x23c, 0x87c (copy 2), 0x1600
(region 2), 0x1c40 (region 2 copy 2). The two regions are NOT bit-identical (different checksum
words + the leaderboard-copy-2 metadata), but carry the same horse list/money.

### 2a. Bookkeeping header (0x0000, mirror at 0x0100)

12 used bytes, three LE32 counters, then machine-setting flag bytes at 0x10/0x20/0x30/0x40/0x4c.

| offset | type | satellite (cw) | master (cwm) | meaning (inferred) |
|--------|------|---------------:|-------------:|--------------------|
| +0x00  | LE32 | 3017 (0x0bc9)  | 3496 (0x0da8)| play/coin counter A |
| +0x04  | LE32 | 3017 (0x0bc9)  | 3487 (0x0d9f)| play/coin counter B (≈A) |
| +0x08  | LE32 | 28138 (0x6dea) | 13237(0x33b5)| total runtime / cumulative counter |
| +0x10  | u8   | 1              | 0            | setting flag 1 |
| +0x20  | u8   | 1              | 0            | setting flag 2 |
| +0x30  | u8   | 3              | 0            | setting flag 3 |
| +0x40  | LE32 | 3              | 0            | setting flag 4 |
| +0x44  | u8   | 1              | 0            | setting flag 5 |
| +0x4c  | u8   | 1              | 0            | setting flag 6 |

The master's header (cwm) is "freshly initialized" (settings all 0, smaller counters); the satellite
(cw) has nonzero settings (3017 plays). The +0x08 value (28138 vs 13237) is plausibly the
**program/pace section counter** discussed in BATTERY_RESUME_FINDINGS.md (the counter that makes a
battery-backed cabinet resume mid-program). Header A (0x00) is mirrored verbatim at 0x100.
**This is the highest-value target for the "resume" hack** — it differs per board and is one of the
few non-leaderboard mutable counters.

### 2b. Region checksum / header (0x01f8)

```
0x1f8  36 35 00 00 c4 13 00 00 c4 13 00 00 00 00 00 00
0x208  36 35 00 00 c4 13 00 00 c4 13 00 00 00 00 00 00   (doubled)
0x218  29 04 00 00  "Big Shuttle"....                    (rank-0 marker)
```
- +0x1f8 `0x3536` (cw) / `0x259a` (cwm) — **16-bit checksum** of the region (the one byte-pair that
  differs master vs satellite; matches the "CRC @0x1f8" note in BATTERY_RESUME_FINDINGS.md).
- +0x1fc `0x13c4` (=5060) and +0x200 `0x13c4` — a length or record-count word, doubled.
- The 16-byte header is written twice (0x1f8 and 0x208).
- 0x218 holds `0x429` (=1065) then "Big Shuttle" — appears to be the current "active/last" horse or
  the rank-0 sentinel for the leaderboard that follows.

---

## 3. MONEY LEADERBOARD ("richest horses" hall of fame) — DECODED

50 records, 32 bytes each, starting at **0x0230** (region 1) / **0x1634** (region 2).
Sorted **descending by prize money**. Verified by decoding all 50 records: money runs
417,935,500 → 4,628,541 monotonically.

### Record format (32 bytes)
| offset | width | field | notes |
|--------|------:|-------|-------|
| +0x00 | u8 | `flag0` | small value 0,1,2,3,7 — per-record attribute bitfield (see below) |
| +0x01 | u8 | `flag1` | 0xc0–0xde range — grade/type/owner code (see below) |
| +0x02 | LE32 | **money** | prize earnings, the sort key |
| +0x06 | LE16 | meta_lo | 0 in copy 1; copy 2 sets this region |
| +0x08 | u8 | sub | mirrors a count (0/1/2) seen in copy 2 |
| +0x09..+0x0b | — | meta | copy 2 writes `80 16 00` here (see copy-2 below) |
| +0x0c | 20 B | **name** | ASCII (EN) / EUC-JP (JP), NUL-padded |

### Verified examples (region 1, copy 1, top of board)
```
rank  flag0 flag1 money        name
 1    1     0xcc  417,935,500  City Commandant
 2    0     0xcc  301,477,906  Trash Talker
 3    1     0xca  288,362,486  Dancing with Yoshi
 4    3     0xca  275,303,080  Big Wave
 5    0     0xcc  262,144,272  Southern Cross
 6    3     0xcf  249,098,240  Wild Card
 7    3     0xca  235,970,560  Shogun
 ...
50    1     0xc1    4,628,541  Big Shuttle
```
All 50 names cross-check against `DOC_COMPLETE_HORSE_DATABASE_DRBYOCW.md` (e.g. "City Commandant"
= horse #222, G1; "Trash Talker" = #220 G3; "Cosmic Princess" = #234). The list is the game's
factory **demo/attract leaderboard** populated with stock NPC horses (no player card was saved here).

### flag0 / flag1 interpretation (PARTIAL — needs more confirmation)
- `flag0` (0–7) is a small bitfield. It does NOT cleanly equal leg-type (e.g. Wild Card start-dash=3
  but Shogun stretch-runner=3). It is more likely a display/sex/medal bitfield. UNCONFIRMED.
- `flag1` (0xc0–0xde) does NOT cleanly equal coat color either (City Commandant coat=Special flag1=0xcc;
  Trash Talker coat=Black flag1=0xcc). The high nibble is constant 0xc/0xd; the low nibble (0–e)
  varies. Best guess: a packed (grade<<? | coat?) or a generation marker. UNCONFIRMED — decode by
  editing one record in Demul and observing the in-game leaderboard render.

### Copy 2 (0x0870) — same 50 horses + extra metadata
Identical money/name list, but bytes +0x06..+0x0b are populated:
```
0x870  01 cc 8c 30 e9 18 00 00 00 80 16 00  City Commandant
                                ^^ ^^ ^^  -> +0x09=0x80 +0x0a=0x16 +0x0b=0x00  (0x168000 = 1,474,560)
0x8d0  03 ca ...                00 80 16 02  Big Wave   (+0x08=0x02)
0x930  03 ca ...                00 80 16 01  Shogun     (+0x08=0x01)
```
`0x00168000` is constant across most rows — likely a **date/season stamp** (when the record was set),
with +0x08 carrying a small per-horse counter (wins-at-record? generation?). Copy 2 is the
"with timestamp" view; copy 1 is the "current standings" view. (Two leaderboards, same payload,
different metadata columns.) Best confirmed by editing and observing.

---

## 4. TRACK-RECORD TABLE (fastest time per race/course) — DECODED

**57 records**, 28 bytes each, starting at **0x0f7c** (region 1) / **0x2340** (region 2).
57 = the number of races/courses in the DOC season program (ties to the race-program "sections"
in BATTERY_RESUME_FINDINGS.md). Verified: contiguous 28-byte records from 0xf7c to 0x15b8 = 57 entries.

### Record format (28 bytes)
| offset | width | field | notes |
|--------|------:|-------|-------|
| +0x00 | 20 B | **holder name** | ASCII/EUC-JP, NUL-padded. All "Hitmaker" in this factory save |
| +0x14 | **LE16** | **record time** | **1/40-second units**; cs = raw*5//2 (×2.5). raw 3384 = 8460 cs = 1.24.60. [corrected 2026-06-08 — was wrongly read as LE32 centiseconds; see `tools/online/TRACK_RECORD_TIME_PRECISION.md`] |
| +0x16 | LE16 | field16 | separate 2B field (marker 0x0762 / flags) — NOT part of the time |
| +0x18 | LE32 | tail | date-stamp on real records; 0 in this factory save |

### Verified sample
```
race  raw    cs      M.SS.hh   holder     (raw = stored 1/40-s value; cs = raw×2.5)
  0   3876   9690    1.36.90   Hitmaker
  1   3384   8460    1.24.60   Hitmaker
  2   3960   9900    1.39.00   Hitmaker
 24   7508   18770   3.07.70   Hitmaker
 25   7956   19890   3.18.90   Hitmaker (longest course)
```
"Hitmaker" = horse #123 "Custom Design / **Hitmaker is Sega**" in the DB — the factory default
record-holder placeholder. These are factory default placeholder times (all slots "Hitmaker"),
set to be easily beaten by real play. When a player beats a course record the game overwrites
the name (EUC-JP on JP versions) and the time here. **This is the per-course "track record" board.**

---

## 5. Master vs Satellite diff (full, verified)

Diffing `derbyocw.sram` vs `derbyocwm.sram`, every differing byte:
```
0x0000-0x0009  header counters (per board)        cw=c90b…6dea  cwm=a80d…33b5
0x0010,0x0020,0x0030,0x0040-0x0044,0x004c  setting flags  cw=1/1/3/3,1/1  cwm=0
0x0100-0x0101,0x0108,0x0118,0x0128,0x0138-0x013c,0x0144  mirror of header (0x100)
0x01f8,0x0208  region-1 checksum                  cw=0x3536  cwm=0x259a
0x15c4-0x15d0  region-1 trailer word              cw=…250a…09  cwm=…0c…00
0x2988-0x2994  region-2 trailer word              (same pattern)
```
Everything else (the entire money leaderboard + track-record payload) is **byte-identical** between
the two boards. Conclusion: **leaderboards/track records are shared cabinet-wide data** replicated to
every board over the link; only per-board bookkeeping (coin counts, dip flags, region checksum, the
trailer counter) is board-local. This is consistent with BATTERY_RESUME_FINDINGS.md's finding that
the master re-initializes program state on boot but the battery payload is otherwise persisted.

The `0x15c4 / 0x2988` trailer holds `25 0a 00 00 … 09` (cw) — a LE16 `0x0a25` (=2597) plus a `09`,
likely a **secondary checksum + record-count** for the track-record table. Master (init) has `0x0c`.

---

## 6. What ties to cards?

- The money leaderboard stores **only name + money + 2 attribute bytes** — it does NOT embed a card
  ID, owner string, or the full horse stat block. A player who registered a card and topped the
  board would appear here by horse name only. (So a "career dashboard" reads names+money+times from
  nvram, then joins to the ROM 244-record stat table / horse DB by name.)
- The card's `SEGABEF0` signature shares the `BEF0` tag found in the EEPROM at 0x02 — same Sega
  game/region family code, confirming card and cabinet belong to the same title.
- The open question from the card spec ("JP on-card stats/sex/leg-type — are they on the card or in
  nvram?") is **resolved in the negative for this nvram set**: the SRAM here contains only
  leaderboard/track-record/bookkeeping data, NOT per-card horse stat blocks. The per-horse stats are
  read from the ROM stat table (244 records) keyed by the horse id stored on the card. Cards in active
  play likely keep their live stats on-card; the cabinet nvram only records hall-of-fame outcomes.

---

## 7. How verified (provenance)

- All offsets extracted with `python3` slicing the real files (helpers in `C:/Users/johnr/AppData/Local/Temp/`).
- Leaderboard: decoded all 50 records; money strictly descending 417,935,500→4,628,541; names match
  `DOC_COMPLETE_HORSE_DATABASE_DRBYOCW.md`.
- Track records: 57 contiguous 28-byte "Hitmaker" records; times 33–79 s match course lengths.
- Master/satellite: full byte diff of the two 32 KB SRAMs (only header/checksum/trailer differ).
- EEPROM: full 128-byte dump; `BEF0` + `DERBY OWNERS CLUB AM3` ID confirmed, two mirrored halves.

---

## 8. Open questions

1. **flag0 / flag1 semantics** in the money record (+0x00/+0x01). Decode by editing one record in
   Demul nvram and watching the attract-mode leaderboard (does the coat/grade/sex icon change?).
2. **Header +0x08 counter** (28138 / 13237): confirm it is the program/pace "section" counter from
   BATTERY_RESUME_FINDINGS.md. If so, editing it should change which race-program section the cabinet
   resumes at — the long-parked "battery resume" goal.
3. **Copy-2 metadata** `0x168000` + the +0x08 sub-byte: date stamp vs win-count. Edit-and-observe.
4. Are there OTHER tables (owner/jockey rankings, breeding hall-of-fame) in the unused 0x2998–0x8000
   span on a *played* cabinet? This save is near-factory (only NPC demo data). Need a SRAM from a
   cabinet with real player cards registered to see if a per-card career block appears.
5. Confirm the same layout on the JP versions (`derbyo2k` / `derbyoc`) — no JP nvram in this set;
   the stride may shrink (JP ROM stat records are 28 B vs 32 B on EN, EUC-JP names).
6. The trailer LE16 at 0x15c4 (0x0a25) — confirm it is a checksum and find the algorithm (needed to
   write valid edited saves that survive the cabinet's integrity check).

---

## 9. Tool ideas this unlocks

- **DOC Career / Hall-of-Fame Dashboard**: parse `*.sram`, render the Top-50 money leaderboard and
  the 57 per-course track records, joining horse names to the per-version stat DB (grade, coat,
  leg-type, stats) for a rich "richest horses" + "course records" view. All fields needed are
  decoded above.
- **NVRAM editor**: set a horse's prize money / install a custom track-record holder name. Needs the
  0x1f8 + 0x15c4 checksum algorithm (open Q6) to produce a save the cabinet accepts.
- **Battery-resume patcher** (revives the parked goal): expose header +0x08 as the program-section
  counter; let the user pick a starting race-program section, write both header + mirror (0x00/0x100),
  recompute checksums. Validates BATTERY_RESUME_FINDINGS.md hypothesis end-to-end.
- **Bookkeeping reader**: surface coin/play counts (header +0x00/+0x04) and dip settings (EEPROM
  0x45 block + SRAM flags 0x10..0x4c) for an operator "earnings report".
