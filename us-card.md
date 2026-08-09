# US / World Edition Card — Full 207-byte Decode (`us-card`)

**Status:** Container + every load-bearing field VERIFIED against real `.card` bytes (Jun 2026).
All claims below were confirmed by extracting bytes from real WE/Rev-C/Rev-D cards on disk and
decoding them with the exact logic in `Tools/DOC-Card-Creator.html`. Confidence is noted per field.

This document is the canonical US-card spec. It supersedes the byte-table fragments scattered in
the OneDrive `CLAUDE.md` / `DOC_Card_Field_Mapping_v1.md` (which it confirms and corrects).

---

## 1. Container & two index conventions (CRITICAL — read first)

- A card is **207 bytes = 3 tracks × 69 bytes (0x45)**. File offsets: track1 `0x00–0x44`,
  track2 `0x45–0x89`, track3 `0x8A–0xCE`.
- **Logical bytes are stored REVERSED per track.** The decoder builds a 1-indexed logical array
  `arHex[1..69]` per track where `card[t*69 + i] = arHex[69 - i]`. So **logical index `k` lives at
  file offset `t*69 + (69-k)`**. Names read "forward" out of the reversed region.
- Two index conventions exist in the wild and BOTH appear in older docs — reconcile them:
  - **Card-Creator convention (authoritative here):** 1-based logical `aN[k]`, `k = 1..69`.
    `populateForm()` / `buildArraysFromForm()` use this. All offsets in this doc are this convention.
  - **DOC-ROM-Studio `recBase` convention:** that file uses `recBase = recordStart+9` for the ROM
    *racing stat table* (a different structure in the ROM, not the card). Do NOT confuse the ROM
    stat-record layout (32-byte records at e.g. 0x108E03) with the on-card track layout. They share
    field *names* (dirt/start/internals) but live in completely different places and orders.
- Encoding on the physical/`.raw` side is XOR with a variable `multiCode` (128…1) plus a 2-byte
  checksum pair that must agree (`encodeTrack`/`decodeTrack`). The 207-byte `.card` is the *decoded*
  payload (what Flycast stores); offsets here are into that decoded 207.

### How verified
Decoded 34 real US cards (Card-Library ASCII horses, Tools/Cards, all Play-4sat/8sat/RevR
`drbyocwc`/`derbyocw` station cards). Names/sire/dam decode to clean ASCII; G1/coat/earnings
self-consistent. Example dump of `Scarecrow_II.card` bytes `0x88..0x9F`:
`21 00 53 45 47 41 42 45 46 30 86 00 00 00 00 00 00 00 00 00 30 10 00 00`
→ `!.SEGABEF0..........0...` (SEGABEF0 at 0x8A, `30 10` marker at 0x9C/0x9D).

---

## 2. Card-type detection (per file, reliable)

| Test | Result |
|---|---|
| ASCII `SEGABEF0` at file offset **0x8A–0x91** | **US / World Edition** card |
| no SEGABEF0; bytes `0x20=0x03, 0x21=0x02` | JP (DOC 2000 / DOC ’99) identity card (see `jp-card`) |

The `SEGAxxxx` marker is **version-specific**: the trailing 4 chars = the ROM product code at
header 0x134. WE = `SEGABEF0`; DOC RevB `SEGABAX0`; DOC2000v2 `SEGABBX0`; DOC II `SEGABDY0`.
US/WE cards in this corpus are all `SEGABEF0`. (Confidence 1.0.)

---

## 3. Track 1 — Identity & genetics  (file 0x00–0x44, logical `a1[1..69]`)

| logical | file off | field | type / range | verified | example |
|---|---|---|---|---|---|
| a1[2..5] | 0x43,0x42,0x41,0x40 | **UID** (4-byte horse id) | u32, copied identically to a2[2..5] & a3[2..5] | YES — all 3 tracks equal on every card | Scarecrow = `21 9E 1C 69` |
| a1[6] | 0x3F | **Personality** | u8 0-255 → 8 bands | YES | Scarecrow=0 (Rough), BabyBoy=164 (Sensitive) |
| a1[7] | 0x3E | **Running-style seed** | u8 0-255, ÷51 → 0-4 style | YES (value present, wide spread) | 0,1,32,86,213,255 seen |
| a1[8] | 0x3D | **Coat base** | u8 (63 = special-coat trigger) | YES | 77=Bay, 129=Black, 63=special |
| a1[9] | 0x3C | **Coat modifier** (special id when a1[8]=63) | u8 | YES | Gulf: a1[8]=63,a1[9]=112 → Org Panda |
| a1[10] | 0x3B | 0 / unused | — | YES (always 0) | |
| a1[11..29] | 0x3A..0x2C | **Dam name** | 18 ASCII chars, reversed, 0-padded | YES | "Candy Kane" |
| a1[30] | 0x27 | name-field separator | always 0 | YES | |
| a1[31..49] | 0x2C..0x14 | **Sire name** | 18 ASCII, reversed | YES | "Scarecrow" |
| a1[50] | 0x13 | separator | always 0 | YES | |
| a1[51..69] | 0x12..0x00 | **Horse name** | 18 ASCII, reversed | YES | "Scarecrow II" |

String read = `getString(a,start,end)` walking DOWN from start to end+1, keeping printable bytes.
Name=`a1[69..51]`, Sire=`a1[49..31]`, Dam=`a1[29..11]`. (Card-Creator uses 18-char windows; the
boundary bytes a1[30]/a1[50] act as separators.)

### Personality bands (a1[6]) — 8-type ROM truth vs 5-type tool
ROM-derived (use this): `0-47 Rough · 48-63 Imposing · 64-111 Calm · 112-127 Firm · 128-175
Sensitive · 176-191 Moody · 192-239 Gentle · 240-255 Proud`. The Card-Creator collapses these to
5 jockey letters (R/I/C/H/S) via `getPersonalityCode` and writes only the 5 anchor values
`{R:0,I:48,C:64,H:80,S:208}` — that is a LOSSY editor simplification, not the real scale. Flag for
any rebuild: store the raw 0-255 byte, don't round to 5 anchors. (Confidence: bands ROM-derived,
high; tool-lossiness confirmed in source.)

### Coat (a1[8]/a1[9]) — VERIFIED tables
- Special (a1[8]=63): a1[9] 0=Okapi,16=Cow,48=Panda,64=Platinum,80=White,112=Org Panda,
  192=Zebra,208=Cow_2,240=Tiger. (Gulf_of_America: 63/112 → Org Panda, confirmed.)
- Normal: Bay `77-79/141-143/205-207`, Black `65-67/129-131/193-195`,
  Brown `69-71/73-75/133-135/137-139/197-199/201-203`,
  Chestnut `64/68/72/76/128/132/136/140/192/196/200/204`, else Gray. The low+high mirrors (×2/×3
  of the base) are real game variants the renderer treats as the same named color.

### Running style (a1[7]) — corrected understanding
Old docs: `style = floor(a1[7]/51)` → 5 styles (Front-runner / Start-dash / Last-spurt /
Stretch-runner / Almighty). VERIFIED the byte exists and spans 0-255, but maxed/edited cards hit
255 (`÷51 = 5`, out of range), so the byte is a **stored seed**, not the authoritative display value.
At runtime WE derives the shown leg type from the externals (`legTypeFromExt`: rank of START among
{Start,OOB,Comp,Tenac,Spurt}, Corner excluded; all-equal → Almighty). So: a1[7] = creation/seed
running-style; on-screen leg type = computed from externals. Keep a1[7] on edit, don't trust ÷51 for
display. (Confidence high.)

---

## 4. Track 2 — Career & status  (file 0x45–0x89, logical `a2[1..69]`)

| logical | file off | field | type / formula | verified | example |
|---|---|---|---|---|---|
| a2[2..5] | 0x88..0x85 | UID (dup) | u32 = a1[2..5] | YES | |
| a2[13] | 0x7D | **Silk color 2** | u8 0-14 | YES | |
| a2[14] | 0x7C | **Silk color 1** | u8 0-14 | YES | |
| a2[15] | 0x7B | **Silk pattern** | u8 0-7 | YES | 0=H-stripe…7=Diamond rows |
| a2[16] | 0x7A | **Sex** | 0=Male 1=Female 2=Gelding | YES | |
| a2[18],a2[19] | 0x78,0x77 | **Last race** result + track index | u8×2 | partial (Doc-derived; nonzero on raced horses) | Caitin 11/255 |
| a2[20],a2[21] | 0x76,0x75 | **Current race** result + track index | u8×2 | partial | |
| a2[22] | 0x74 | **Rest / fatigue timer** | u8 (0=rested) | partial | |
| a2[23] | 0x4B→0x?? (0x... see note) | **Retire internal SHARP** | u8 0-45 | YES | Scarecrow ret=45 |
| a2[24] | — | **Retire internal SPEED** | u8 0-45 | YES | |
| a2[25] | — | **Retire internal STAMINA** | u8 0-45 | YES | |
| a2[26] | 0x73 | **Hood** | u8 0-63 | YES | Scarecrow=21 |
| a2[27] | 0x6F | **owner/stable assoc (TBD)** | u8 | partial — =44 on 3 "Trump-stable" cards, 0 elsewhere | Gulf/Phil/Xi=44 |
| a2[28..33] | 0x6E..0x69 | **Retirement externals** Spurt,Tenac,Comp,OOB,Corner,Start (value-1, display 1-16) | u8 0-15 | YES | |
| a2[34] | 0x68 | **Wins duplicate** | = a2[49] | YES | Scarecrow 5=5 |
| a2[35] | 0x67 | **Total races** | u8 0-64 | YES | Scarecrow=6 |
| a2[36] | 0x66 | **Trust** | u8 (0-100+) | partial — small values 0-3 on fresh cards | |
| a2[37] | 0x65 | **Hearts** | display = (val+1)/4 | YES | Scarecrow 55→14 |
| a2[38..43] | 0x64..0x5F | **Current externals** Spurt,Tenac,Comp,OOB,Corner,Start (value-1, display 1-64) | u8 0-63 | YES | |
| a2[44] | 0x5E | **Condition / fitness** | u8 (fluctuates 15-124) | partial | mostly 1 on fresh |
| a2[45] | 0x5D | **Experience** (only increases) | u8 | partial — tracks trust on fresh sample | |
| a2[46] | 0x5C | **Out** (4th+) | u8 | YES | |
| a2[47] | 0x5B | **Show** (3rd) | u8 | YES | |
| a2[48] | 0x5A | **Place** (2nd) | u8 | YES | |
| a2[49] | 0x59 | **Won** | u8 | YES | |
| a2[50] | 0x58 | earnings high pad | always 0 in corpus | YES | |
| a2[51] | 0x57 | **Earnings byte hi** | see formula | YES | |
| a2[52] | 0x56 | **Earnings byte mid** | | YES | |
| a2[53] | 0x55 | **Earnings byte lo** | dollars=(51·65536+52·256+53)·1000 | YES | Scarecrow=4322 → $4,322,000 |
| a2[55] | 0x53 | **G1 bitfield byte 55** | bitfield | YES | |
| a2[56] | 0x52 | **G1 bitfield byte 56** | bitfield | YES | Scarecrow=128 → Japan Cup |
| a2[57] | 0x51 | **G1 bitfield byte 57** | bitfield | YES | |
| a2[61] | 0x4D | **Internal SHARP (current)** | u8 0-60 (cap) | YES | |
| a2[65] | 0x49 | **Internal SPEED (current)** | u8 0-60 | YES | |
| a2[69] | 0x45 | **Internal STAMINA (current)** | u8 0-60 | YES | |

(The card-offset column for a2[23-25] retire-internals: a2[k] → 0x45+(69-k). a2[25]=0x59? No —
a2[k] file offset = 69+(69-k). a2[23]→0x67? careful: 69+(69-23)=0x73? Recompute: 69+(69-23)=69+46=115=0x73.
Use the formula `off = 69 + (69-k)` for any a2[k]; the column above lists the principal fields,
compute the rest with that formula. a2[69]=0x45, a2[1]=0x89.)

> Earnings cap (editor): $262,500,000, multiple of $1,000 → internal max 262500. Max internal that
> fits 3 bytes is far higher; the editor caps for UI sanity.

### External stat order (memorize)
Both current and retirement externals are stored in the order **Spurt, Tenacious, Competing,
OutOfBox, Corner, Start** as logical index *increases*, i.e. Start is the HIGHEST index of its block:
- Current: Start=a2[43], Corner=a2[42], OOB=a2[41], Competing=a2[40], Tenacious=a2[39], Spurt=a2[38].
- Retirement: Start=a2[33], Corner=a2[32], OOB=a2[31], Competing=a2[30], Tenacious=a2[29], Spurt=a2[28].
Display = card+1 (current range 1-64, retirement bands 1-16: ✕1-4 △5-8 ○9-12 ◎13-16). VERIFIED.

### G1 titles bitfield (a2[55],a2[56],a2[57]) — VERIFIED map
18 races spread non-contiguously across 3 bytes (`G1_RACES` in Card-Creator):
- **a2[57]:** bit1=Winter Stakes(1), bit2=Doc1000(3), bit4=Doc2000(4), bit8=Spring Classic(5),
  bit16=American Derby(7), bit32=Hong Kong Oaks(18), bit64=Hong Kong Derby(17), bit128=American Oak(6).
- **a2[56]:** bit1=Summer GPX(8), bit2=Super Dirt GPX(9), bit16=Stayers(11), bit32=QE II(12),
  bit64=Mile Champ(13), bit128=Japan Cup(15).
- **a2[55]:** bit1=Sprinter Stakes(10), bit4=Derby Owners Cup(16), bit8=Japan Cup Dirt(14),
  bit16=Sprinter Trophy(2).
Confirmed live: Scarecrow_II a2[56]=128 = Japan Cup, 1 title. (Confidence high.)

---

## 5. Track 3 — Breeding, visual, markers  (file 0x8A–0xCE, logical `a3[1..69]`)

| logical | file off | field | type | verified |
|---|---|---|---|---|
| a3[2..5] | 0xCD..0xCA | UID (dup) | u32 | YES |
| a3[50] | 0x9D | **format marker `0x10`** | const | YES |
| a3[51] | 0x9C | **format marker `0x30`** | const | YES (`30 10` pair at 0x9C/0x9D) |
| a3[53] | 0x9A | **Breed count** | offspring = val/2 (editor stores breeds·2) | YES |
| a3[57] | 0x96 | **Retired flag** | 0=active 1=retired | YES |
| a3[61] | 0x92 | **Dirt ability** | u8 0-255 | YES (Gulf=255, Caitin=107) |
| a3[62..69] | 0x91..0x8A | **`SEGABEF0` marker** | 8 ASCII, reversed-into-track so reads forward at 0x8A | YES |

Note the apparent overlap: card byte 0x92 = a3[61] = **dirt**; in the marker-region ASCII dump it
shows as the printable char right after "SEGABEF0" (e.g. 'k'=107 for Caitin). That is dirt, not part
of the signature. Signature is exactly 0x8A–0x91; dirt at 0x92; pad; `30 10` at 0x9C/0x9D.

Tracks-2/3 unused bytes are 0 in clean cards (large 0x00 runs at a3[6..49], a3[54..56], a3[58..60]).

---

## 6. The 4-byte horse UID (newly emphasized)

`a1[2..5] == a2[2..5] == a3[2..5]` on every card (verified on all 34). `createNewHorse()` sets it to
4 random bytes and copies to all three tracks. Card offsets: **0x40-0x43 / 0x85-0x88 / 0xCA-0xCD**.
This is the per-horse identity key (cabinet likely keys session/cabinet records to it). For EDIT keep
it; for fresh CREATE the editor randomizes it. (Confidence high.)

---

## 7. Parity / checksum

There is **no whole-card checksum in the 207-byte decoded payload.** The integrity protection lives
in the `.raw`/physical-track encoding layer: each 72-byte raw track carries a 2-byte XOR checksum
pair that `encodeTrack` brute-forces `multiCode` to make agree (`chksum1==chksum2`); `decodeTrack`
inverts it. The `.card` (Flycast) form is post-decode and unprotected — any byte can be edited and
re-saved; re-encoding to `.raw` regenerates valid checksums. The triple-stored UID is the only
on-card redundancy. (Confidence high — confirmed in `encodeTrack`/`decodeTrack` source.)

---

## 8. Still-unknown / low-confidence bytes (next targets)

- **a2[27] (0x6F):** =44 on the 3 cards of one stable (Gulf/Phil/Xi), 0 elsewhere → owner/stable id
  or pedigree-line index. Needs a multi-horse same-stable capture to confirm.
- **a2[18-22] (last/current race + rest):** Doc-derived; need a card from a horse mid-career (race
  history) to read result/track-index encoding (`field=(val/6)+1, pos=(val%6)+1`, with a >95 branch).
- **a2[36] trust vs a2[44] condition vs a2[45] experience:** labels are Doc-derived; on fresh cards
  these are tiny (0-3) and a2[45] tracks a2[36]. Confirm against a developed horse + in-game screen.
- **a1[7] runstyle seed:** confirm whether the game ever reads it post-creation or fully derives leg
  type from externals (current evidence: derived).
- Big 0x00 regions in tracks 2/3 are genuinely unused by WE (not leak — clean cards are all-zero).

---

## 9. Cross-version note (4 versions)

The 207-byte **container is identical across all 4 versions** (Flycast `TRACK_SIZE=0x45`,
`cardData[0x45*3]`). What differs:
- **US / WE (drbyocwc Rev C, derbyocw EX Rev D):** FULL 3-track stat card exactly as above; marker
  `SEGABEF0`. Rev C vs Rev D are NOT distinguishable from card bytes (same layout, same marker) — the
  Card-Creator only tags the *filename* with the version. Verified: BabyBoy (Rev D) and WillyJR (Rev C)
  decode identically structured.
- **DOC 2000 / DOC ’99 (derbyo2k, derbyoc):** **identity-only** cards — name/sire/dam in kana at 0x28,
  `0x03 0x02` header, NO SEGA marker, tracks 2-3 unwritten. Stats are cabinet-side. See `jp-card`.
- Full-stat JP cards would carry `SEGABBX0`/`SEGABAX0` markers IF real hardware writes them; emulation
  has only produced identity-only JP cards so far (open question, needs a physical-card scan).

---

## 10. Tool ideas this unlocks

1. **`doc_card.py` — single canonical codec.** Pure-Python encode/decode mirroring the JS, exposing
   the full field map (this doc) as a dict. Lets the suite, converter, and any batch tool share one
   source of truth instead of re-implementing offsets. Needs: this map (done).
2. **Card differ / lineage viewer.** Diff two `.card` files field-by-field (great for the beer
   experiment, breeding, and "what changed after a race" captures) and group cards by shared UID /
   a2[27] stable id. Needs: this map + a small CLI.
3. **Raw-byte field probe harness.** Given many cards, auto-correlate the still-unknown bytes
   (a2[27], a2[18-22], a2[36/44/45]) against known fields to finish the decode. Needs: a corpus of
   *developed* (raced) cards — currently most on disk are fresh.
4. **Personality-fidelity fix for the editor.** Replace the lossy 5-anchor personality with the raw
   0-255 byte + the 8-band ROM labels, and replace the ÷51 runstyle display with `legTypeFromExt`.
   Needs: nothing new (both are in this doc).

---

## 11. Fleet census (2026-07-19) — 6,102 real cloud cards, per-byte statistics

Probe harness from §10.3 finally ran: every byte of every card in the arcade DB (774 living +
5,328 Glue Factory archive), plus longitudinal growth via card_history. Findings:

**Resolved / confirmed:**
- All unmapped gaps are CONSTANT across 6,102 cards: a2[6..12], a2[17], a2[54], the bytes between
  the three current internals (0x46-48/0x4A-4C/0x4E-50), a3's zero regions. Genuinely unused.
- **a2[27] "stable id" MYTH BUSTED**: always 0 fleet-wide. The =44 sighting was an editor artifact
  on those 3 disk cards, not a game field.
- Earnings hi byte (a2[51]) still 0 on every card (fleet max earnings < $65.5M).
- a2[18..21] (file 117-120): the "last/current race" reading holds for RACED cards (0-11 enum =
  field×pos per the (val/6)+1 formula, byte 0-255 track idx), BUT the game writes creation defaults
  b119=43, b120=3 on UNRACED cards (b120=11 on a 124-card minority — origin uncorrelated so far).

**New mysteries (open):**
- **a2[45] (file 93) is a 2-bit BIRTH TRAIT, not "experience"**: {2: 92%, 0: 3.0%, 1: 2.4%, 3: 2.4%}
  across the fleet, stamped nonzero at creation on ~97% of cards. NOT a growth-rate class (growth
  test: gain/race identical 1.26-1.32 across all four values, 242 horses with 4+ race history).
  Candidate semantics: feeding/temperament class (Career-Log "food a horse hates" lore), precocity
  shape (front-loaded vs late — untested), or an AI/behavior class. **The Lab always writes 2; the
  game rolls non-2 on ~8% of births.**
- **a1[7] (file 62) runstyle seed: the Lab always writes 0; ~30% of game-born cards are nonzero**
  (128 distinct values seen). Not a uniform roll (70% zero among game-born) — path- or
  version-dependent. If this seed feeds race behavior, Lab foals all share the zero class —
  POSSIBLE mechanism for CF-28 ("lab anomalies ignore the whip"). Needs a targeted RE pass on
  what reads a1[7] at race time.
- **UID bytes are STRUCTURED in-game, not random**: byte 2-of-4 (file 65/134/203) is zero on ~93%
  of cards with only 8 distinct values ever — in-game serials look like a counter+field layout,
  only the editor randomizes all 4. (Compatible with v7 serial-lock; nothing to change.)

**Lab fidelity gaps found**: creation defaults match the game (b93 mode=2, b116=1, b119=43,
b120=3 ✓) EXCEPT the Lab never rolls the rare b93 classes (game: ~8% non-2) and never writes a
runstyle seed (game: ~30% nonzero). Decision pending: mimic the game's observed birth
distributions vs decode the semantics first.
