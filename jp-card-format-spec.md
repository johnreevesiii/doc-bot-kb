# Japanese DOC Card Format Spec (derbyo2k / DOC 2000, derbyoc / DOC Original)

Status: name/pedigree + encoding SOLVED & validated. On-card stat layout: BLOCKED (needs a hardware
JP card reader or an in-game stat-screen capture to map; see "Open").

## 1. Container (same as US)
- 207 bytes = 3 tracks x 69 (0x45). Same `.card` file, same built-in Flycast reader.
- Reader class differs only at the serial layer: JP uses `DerbyLRCardReader` (SanwaCRP1231**LR**),
  US/WE uses `DerbyBRCardReader` (**BR**). Selected in Flycast by gameId == " DERBY OWNERS CLUB WE ---------".
  The BR/LR split changes the status-byte protocol only; the on-card byte container is identical.

## 2. How to tell a JP card from a US card (per-file, reliable)
- US/WE: ASCII signature **`SEGABEF0`** at offset **0x8A** (+ `30 10` marker at 0x9C); plaintext name from byte 0.
- JP: **no** `SEGABEF0`; header skeleton **`0x20=0x03, 0x21=0x02`**; name is kana-encoded at 0x28.

## 3. Track 1 layout (JP)
| offset | field | notes |
|---|---|---|
| 0x00-0x1f | VOLATILE | live state + uninitialized heap; changes frame-to-frame while running. Not stable card data. |
| 0x01 | 0x70 | constant |
| 0x18 | 0x0d | constant |
| 0x20,0x21 | 0x03,0x02 | record/format marker (constant) |
| 0x22-0x24 | 0x00 | constant |
| 0x25-0x27 | 3 lead bytes | vary per horse; purpose TBD (likely horse ID / pedigree index) |
| **0x28** | **horse name** | kana table (sec 5), **0x7d-terminated**, variable length |
| (after 7d) | **sire name** | kana table, 0x7d-terminated -- a real DOC mater-table name |
| (after 7d) | **dam name** | kana table; runs into the volatile track-1 tail |

Tracks 2-3 (0x45-0xCE): NOT written by DOC 2000 -> uninitialized heap (a distinct pointer family per
process). DOC 2000 is an identity/pedigree card; it does NOT pack full stats onto the card the way US/WE does.

## 4. Validation
Auto-assigned sire/dam fields decode to exact DOC 2000 mater-table names (cross-checked vs
DOC_COMPLETE_HORSE_DATABASE_DERBYO2K.md): Trick Power, Trans Wind, Jade Robbery, Manhattan Lady,
Final Record, Shinko Lovely, Irish Dance. 7/7 exact.

## 5. Character table (game-internal, 1 byte per kana)
Column-major gojuon: 15 consonant-rows per vowel column.
Rows 0..14 = (none) K S T N H M Y R W G Z D B P.
Columns: a=0x00-0x0e, i=0x0f-0x1d, u=0x1e-0x2c, e=0x2d-0x3b, o=0x3c-0x4a.
Vowel anchors: ア=0x00 イ=0x0f ウ=0x1e エ=0x2d オ=0x3c (step 15).
Kept slots (non-existent kana yi/ye/wi/we/wu) occupy their grid index but are unused.
Extended:
- 0x45 = ン  (the o-column W slot holds ン, not ヲ)
- 0x4b ァ, 0x4c ィ, 0x4d ゥ, 0x4e ェ, 0x4f ォ, 0x50 ャ, 0x51 ュ, 0x52 ョ, 0x53 ッ, 0x54 ー
- Not yet mapped (low priority; names are katakana): digits, ヴ, ゛/゜, punctuation.
Padding/terminator byte = **0x7d**.
Reference impl: `_jp_re/jp_decode.py`.

## 6. Tooling status
- Card Suite handler (`Tools/doc_suite_server.ps1` + `.py`): auto-detects JP vs US per card and
  decodes JP names for the station + library lists. US cards unchanged. Verified on all 4 live cards.
- **Stable management (shipped):** the suite's Library section is now a true Stable.
  - Servers: `_stable.json` manifest (per-card version tag + manual order); `/api/library` and
    `/api/stations` return `kind`/`version`/`order`/`bytes`; new `POST /api/stable/order`; tag-on-save.
  - Client (`DOC-Card-Creator.html`): `parseCardFacts(bytes)` (US full via populateForm offsets +
    legType=floor(a1[7]/51); JP name/sire/dam). Hover quick-view (sire/dam, leg-type icon, sex icon,
    unicode stat bars ▰▱, internals, coat/record/earnings). Active/Retired groups (US) or Roster (JP);
    "Other versions" greyed/collapsed. Drag-reorder + Name/Stat/Status sorts (persisted). Version scope:
    load-to-station blocks US↔JP family mismatch, warns on same-family cross-rev. JP create/edit blocked
    (editor builds US-format only).
  - Progressive: JP rows show name/sire/dam + "stats pending hardware reader"; they light up once the JP
    stat layout (sec "Open") is mapped — `parseCardFacts` JP branch is the single place to extend.
  - Verified in-browser: JP session (katakana names, JP quick-view) and US session (full stat quick-view,
    Active group, Σ totals). Order round-trip persists to `_stable.json`.
- Editor create/edit path remains US-only (builds US 3-track cards). Full JP create/edit deferred until the
  stat layout + a valid-JP-card construction recipe are known.

## VERSION FINGERPRINTS — card stat marker = "SEGA" + ROM product code @0x134 (Jun 2 2026)
The WE "SEGABEF0" card marker is NOT fixed: BEF0 is just WE's product code at header offset 0x134.
Each DOC version stamps its own code, so a full-stat card's marker is version-specific:
| Version | folder | code @0x134 | title (Japan slot) | stat-card marker |
|---|---|---|---|---|
| DOC Rev B | derbyoc | BAX0 | DERBY OWNERS CLUB | SEGABAX0 |
| DOC 2000 v2 | derbyo2k | BBX0 | DERBY OWNERS CLUB | **SEGABBX0** |
| DOC II | derbyoc2 | BDY0 | DERBY OWNERS CLUB II | SEGABDY0 |
| World Edition (A/C/D/T/R) | drbyocw* / derbyocw / derbyocr | BEF0 | DERBY OWNERS CLUB WE | SEGABEF0 |
- Re-scanned ALL emulated derbyo2k cards (frozen + freshly RACED) for SEGABBX0: NONE present (no SEGA stamp at all).
  So in EMULATION, DOC 2000 v2 writes identity-only cards even after a real race. Confirmed not a "horse wasn't developed" artifact.
- PHYSICAL CARD TEST (the tiebreaker): scan a real DOC 2000 v2 card and grep for **SEGABBX0**. If present + tracks 2-3 populated,
  real DOC2000v2 hardware writes full stat cards (structurally identical to WE) and Flycast's JP (LR) card-write path is the gap that
  explains why our emulated captures are bare. If absent, DOC2000v2 is genuinely identity-only.
- ROOT CAUSE of earlier "no card data": the derbyo2k 4-sat rig was HUNG on multiboard sync (race never started). After a clean
  reset + all 5 windows clicked, races run and horses develop. Logging now enabled in Play-4sat emu.cfg (main + sat1).

## VALIDATED IN-GAME (Jun 2 2026) — identity is card-side and writable
- **The card's kana name field IS the in-game horse name.** Hand-built/edited cards loaded in the derbyo2k
  rig and the horse displayed the exact card name: sat2 card コィゥォャ and sat3 card シヒニヌ both matched on
  the in-game status/race screens. A from-edit card (sat1 renamed to ジョン, sire kept トリックパワー) loaded into
  full gameplay (training "成功", time trial) — structure accepted.
- **Names are NOT in the cabinet nvmem** (searched all 5 derbyo2k `*.nvmem` for the kana + EUC-JP names: zero
  hits). So name/sire/dam live ON THE CARD; editing the card renames the horse. (Open: where STATS live — the
  loaded horses show full stat bars + earnings yet tracks 2-3 are blank; stats may be cabinet-side keyed by the
  lead bytes, or in an undecoded card region.)
- **Write path proven:** `jp_encode.py` `encode()`/`write_fields()` round-trip exactly vs 4 real cards.
  Reverse kana table = inverse of the cracked forward table (81 kana). Overflow + unmapped-kana guarded.
- **Authentic name list:** `jp_mater_names.json` = 167 ROM mater (sire/dam) names extracted from
  derbyo2k epr-22284a @ idx-run 0x11106C+60*k+4 (EUC-JP), 164/167 fully kana-encodable. Bridge proven:
  ROM EUC-JP ↔ Unicode ↔ card kana. These are the editor's sire/dam dropdown source.
- **Lead bytes 0x25-0x27:** still per-horse id (no pattern across 4 cards). For EDIT, keep them; for
  brand-new CREATE, scheme TBD (copy a template / test acceptance).
- NEXT: add JP create/edit to the Stable Mgmt System (kana name input + sire/dam dropdowns from
  jp_mater_names.json + valid .card writer via jp_encode.write_fields). Editor `parseCardFacts` JP branch
  + a new build path are the integration points.

## Open (blocked on hardware / capture)
- Map any on-card stat fields. Method: read a horse's in-game stat screen (externals, internals, hearts,
  trust, condition, record, earnings) and match the known values to frozen-card bytes -- same approach
  that cracked the names. Requires a modified JP card reader OR a clear in-game stat-screen screenshot.
- Confirm whether DOC 2000 stores any career data on-card at all, or keys it to cabinet storage.
