# JP Card — Full Decode + the On-Card-vs-Cabinet Question

KEY: `jp-card`
Scope: the 207-byte Japanese DOC card written by **derbyo2k** (DOC 2000, product code `BBX0`) and
**derbyoc** (DOC '99, product code `BAX0`). These use the Sanwa **CRP1231LR** reader path
(`DerbyLRCardReader`), distinct from the US/WE **BR** path. Container is byte-identical to the US card;
only the on-card *encoding and field layout* differ.

Status: **name / sire / dam + kana encoding: SOLVED and byte-validated.** Lead bytes and the 2-byte
trailer: **characterized** (identity ID + per-write nonce). On-card stats / sex / leg-type: **proven
absent** — the JP card is an identity/pedigree card, not a full-stat card; stats are not in the cabinet
nvram either (they are computed/held in volatile cabinet RAM keyed by the card identity, with no
per-card persistence found in the emulated battery).

**2026-07-24: FIRST REAL PHYSICAL CARD DECODED** (MSR605 scans, see §11): format confirmed on real
hardware; kana correction ユ=0xa5; real cabinets write a SECOND track absent from emulation.

---

## 1. How verified (assets used)

- 8 real captured cards (all 207 bytes), two captures per satellite:
  - `_jp_re/captures/frozen1/derbyo2k_sat{1..4}.card`
  - `_jp_re/captures/derbyo2k_sat{1..4}_unknownname.card`
  - sat2/sat3/sat4 are the **same horse** in both capture sets (clean re-save pairs); sat1 differs
    (the "unknownname" sat1 is a different horse: シツテ / サイレントブルームバースデイローズ), so sat1 is *not* a
    clean pair and was excluded from the persistence diff.
- ROM `derbyo2k/epr-22284a.ic22` (4,194,304 bytes) for the mater (sire/dam) EUC-JP source.
- ROMs `drbyocwc/epr-22336c.ic22`, `derbyocw/epr-22336d.ic22` for product-code contrast.
- Cabinet nvram: `Play-4sat/sat{1..4}/data/derbyo2k.zip.nvmem`, `Play-4sat/data/derbyo2k.zip-main.nvmem`,
  and the 8-sat equivalents (each 32,768 bytes).
- `_jp_re/jp_decode.py`, `jp_encode.py`, `jp_mater_names.json`; `Tools/doc_suite_server.ps1`.

Every offset/value below was extracted from the real bytes in this session, not asserted from a doc.

---

## 2. Container (CONFIRMED, conf 1.0)

- 207 bytes = **3 tracks × 0x45 (69)**. `TRACK_SIZE=0x45`, `cardData[0x45*3]`. Identical file container
  to the US card. Verified: all 8 captures are exactly 207 bytes.
- Logical track bytes are stored **reversed per track** in the Sanwa stream, but the `.card` file as
  captured is already in logical order (offsets below are file offsets as the tools read them).
- Reader split (from Flycast `card_reader.cpp`): `gameId == " DERBY OWNERS CLUB WE ---------"` → BR
  (US). Anything else (DOC 2000/'99/II) → **LR**. The NAOMI header title confirms the routing:
  - WE ROMs: title `" DERBY OWNERS CLUB WE ---------"` (verified at ROM 0x30).
  - derbyo2k / derbyoc: title `" DERBY OWNERS CLUB ------------"` (no "WE") → LR reader.
  BR vs LR differ only in the serial status-byte protocol; the byte container is the same.
- **2026-07-25 hardware research** (`Tools\JP-Card-Capture\SANWA_READER_RESEARCH.md`, sourced):
  the JP DOC 2000 cabinet's physical unit is the **CRP-1231AR-10** (Sega P/N 601-10822); DOC WE =
  CRP-1231BR-10 (P/N 601-11082). Flycast's "DerbyLRCardReader" is a protocol-class label, not the
  model. Reader personality = one socketed 128KB firmware ROM on an H8/3003 (MAME chihiro has an
  LR firmware dump); BR adds a front shutter + shutter command. **Track selection is a GAME
  decision, not firmware**: the read/write commands take a track-select byte (0x30-0x32 single,
  0x33-0x35 pairs, 0x36 all three, per YACardEmu CardIo.cpp) — so JP single-track cards are a
  software choice and tracks 2/3 are physically writable. Sibling 1999 Sanwa/JSW datasheet (FCC
  N8U199903ACR, UERW-301, PDF saved alongside): F2F, 210 bpi tracks 1/3 + 75 bpi track 2,
  coercivity 300-4000 Oe switchable, 0.68-0.80mm cards. The wide-track arcade geometry we
  measured remains publicly undocumented (our measurement is the only source).

---

## 3. JP vs US discrimination (CONFIRMED, conf 1.0)

- **US/WE**: ASCII `SEGABEF0` at **0x8A** (BEF0 = WE product code @ROM 0x134), plaintext ASCII name from
  byte 0, `30 10` marker at 0x9C, all 3 tracks used for stats/earnings/G1/silks.
- **JP**: **no** `SEGA****` marker anywhere; header skeleton `0x20=0x03, 0x21=0x02`; kana name at 0x28.
- Product codes @ROM 0x134 (verified): WE `BEF0`, DOC 2000 `BBX0`, DOC '99 `BAX0`, DOC II `BDY0`.
  A hypothetical JP *full-stat* card would therefore carry `SEGABBX0` / `SEGABAX0`, never `SEGABEF0`.
  No such marker was found on any emulated JP card (consistent with identity-only writes).
- **DETECTION CORRECTION (2026-07-24, proven live):** SEGABEF0@0x8A does NOT discriminate US vs JP.
  derbyo2k rewrites only track 1, so a JP save over a reused World slot keeps a STALE SEGABEF0
  (seat1 コンスタンティヌス on the office rig carried both 03 02 @0x20 AND SEGABEF0 @0x8A). The ONLY
  valid JP test is `b[0x20]==0x03 && b[0x21]==0x02` (World cards read ASCII text there, never 03 02);
  check it FIRST. `is_jp_card`/`Test-JpCard` fixed accordingly in doc_suite_server.py/.ps1 and the
  cabinet-pack slot_agent.ps1 (name decode bound also fixed to 0x31 = fixed 9-byte slot).

---

## 4. Track 1 byte map (the only meaningful track) — VERIFIED

Constancy table computed across all 8 cards (`vary` = differs between cards/captures):

| offset | value | status | meaning |
|---|---|---|---|
| 0x00 | vary | volatile | leaked/live, not card data |
| 0x01 | **0x70** | CONST | header constant |
| 0x02–0x08 | vary | **volatile** | changes between re-saves of the *same* horse → NOT persistent stats (see §6) |
| 0x09 | 00/02 | vary | volatile |
| 0x0a | **0x00** | CONST | |
| 0x0b–0x0e | vary | volatile/leak | |
| 0x0f | fd (mostly), fc/16 | ~const | header-ish but not stable; treat as volatile |
| 0x10 | **0x00** | CONST | |
| 0x11–0x12 | vary | volatile | |
| 0x13 | **0x00** | CONST | |
| 0x14–0x17 | vary | volatile/leak | |
| 0x18 | **0x0d** | CONST | header constant |
| 0x19 | **0x00** | CONST | |
| 0x1a–0x1b | vary (small) | volatile | |
| 0x1c,0x1d,0x1e | **0x00** | CONST | |
| 0x1f | 00/20/e0 | vary | volatile |
| 0x20 | **0x03** | CONST | **format marker hi** |
| 0x21 | **0x02** | CONST | **format marker lo** |
| 0x22,0x23,0x24 | **0x00** | CONST | reserved |
| **0x25,0x26,0x27** | vary, **PERSISTENT** | 3 lead bytes | per-card identity / serial (see §5) |
| **0x28 … 7d** | kana | PERSISTENT | **horse name** |
| (after 7d) | kana | PERSISTENT | **sire name** (mater table) |
| (after 7d) | kana | PERSISTENT | **dam name** (mater table) |
| (after dam 7d) | 0x00 pad | — | zero pad to the trailer |
| **0x43,0x44** | vary, **per-write** | trailer | 2-byte write nonce / checksum (see §7) |

Header constants confirmed: `0x01=70, 0x0a=00, 0x10=00, 0x13=00, 0x18=0d, 0x19=00, 0x1c..1e=00,
0x20=03, 0x21=02, 0x22..24=00`.

### Tracks 2–3 (0x45–0xCE): NOT card data — heap leak (CONFIRMED, conf 0.97)
For the same-horse pairs, the nonzero "persistent" bytes in tracks 2–3 are little-endian Windows heap
pointers repeating on an 0x20 stride, e.g.:
- sat2: `d0 5f cc 96 a4 01` at 0x5c **and** 0x7c → pointer `0x000001A696CC5FD0`; `50 57 d1 96 a4 01`,
  `80 9c 44 8f a4 01` elsewhere.
- sat4: `10 6c 0b 05 bc 02` repeated at 0x5c/0x7c/0x9c/0xbc → pointer `0x000002BC050B6C10`.
They are identical across the two captures only because both were taken from the **same Flycast
process** (same heap base). They carry no horse data. DOC 2000 never writes tracks 2–3.

---

## 5. Lead bytes 0x25–0x27 = per-card identity (conf 0.85)

Stable across re-saves of the same horse (sat2 `c1 a7 c2`, sat3 `4c 82 10`, sat4 `cb 25 c0` —
unchanged between frozen1 and unknownname captures). Different per horse, with no low-range/0–243
index pattern (values like `ca 0b 20`, `c1 a7 c2`, `4c 82 10`, `cb 25 c0`, `89 03 d4` — high bits set,
pointer-ish). Best interpretation: a **per-card unique ID / serial** assigned at card creation, used by
the cabinet to key the horse's (cabinet-side, volatile) career/stat record. They are NOT a direct index
into the ROM 244-record racing-stat table.

For EDIT: keep them verbatim. For brand-new CREATE: scheme unknown — copy a template card's lead bytes
or test in-cabinet acceptance.

---

## 6. The decisive stats question — STATS ARE NOT ON THE CARD (conf 0.9)

Three independent confirmations:

1. **Same-horse re-save diff.** For sat2 and sat4 (identical horse in both captures), the *only*
   persistent region is `0x25–0x42` (lead bytes + name/sire/dam). Bytes 0x02–0x1f changed between the
   two saves of the same horse, so they cannot be persistent stat fields — this **kills the earlier
   "0x02–0x08 might be compact dynamic stats" hypothesis** from FINDINGS.md. Tracks 2–3 are heap (§4).
2. **No SEGA stat marker.** No `SEGABBX0`/`SEGABAX0`/any `SEGA****` stamp on any JP capture, frozen or
   re-raced. US full-stat cards always carry the marker; its absence means no stat block was written.
3. **Cabinet nvram has no per-card horse data.** Searched all derbyo2k `*.nvmem` (4-sat + 8-sat, ~4,800
   nonzero bytes each = machine settings/bookkeeping) for the on-card kana name byte sequences
   (アイウエオ `00 0f 1e 2d 3c`, コ+ext `3d 4c 4d 4f 50`, シヒニヌ `11 14 13 22`, トホメレ `3f 41 33 35`):
   **zero hits in every file.** Names — and by extension the horse records — are not in the emulated
   battery.

Conclusion: **JP DOC 2000/'99 cards are identity + pedigree cards.** The card stores who the horse is
(name + sire + dam + a unique ID); the cabinet computes/holds the live stats, sex, leg-type, hearts,
trust, condition, record and earnings in volatile RAM keyed by the lead-byte ID for the duration of the
session. There is no on-card stat layout to map (no hardware reader is needed to "find" it — it does
not exist on the card). This is the opposite of the US/WE card, which packs the full stat record across
all 3 tracks.

Caveat (conf): the emulated rig never demonstrated cross-session persistence (matches the parked
battery-resume finding: the multiboard master re-initializes program/records on boot). On *real* linked
cabinets a horse's career may persist in the master's BBSRAM keyed by the card ID — but that is
cabinet-side, still not on the card. A physical DOC 2000 card scanned for `SEGABBX0` with populated
tracks 2–3 would be the only thing that could overturn "identity-only," and emulation evidence is
strongly against it.

---

## 7. Trailer 0x43–0x44 = per-write 2-byte nonce/checksum (conf 0.8)

> **RESOLVED 2026-07-25 (conf 1.0):** it IS a checksum, pinned from the ROM — a CRC-16/CCITT
> (poly 0x1021) run in the high 16 bits of a 32-bit register, init 0xDEBDEB00, over card
> 0x00..0x42 (67 bytes) with an 8-bit final flush, stored little-endian (card[0x43]=low,
> card[0x44]=high). Reproduces 6/6 real cards. The load path recomputes and **rejects** on
> mismatch, so a created card must carry a correct trailer. Full write-up + reference code:
> `reverse-engineering/findings/jp-card-trailer.md`. (The "not a plain sum/xor" note below
> stands; the earlier CRC-CCITT-init-0/0xFFFF guess failed for the 4 reasons in that file.)

For the same horse, the name region 0x28–0x42 is byte-identical across the two captures, but 0x43–0x44
changes (sat2 `8d 4b` → `ad f9`; sat4 `e2 46` → `92 d2`). Tested and rejected: it is **not** a plain
sum/xor over the name region (sat2's two captures share identical name-region sums `0x092b` yet have
different trailers), nor a clean additive checksum over track 1 (0x00–0x42). Because the volatile bytes
0x00–0x1f (uninitialized in emulation) differ between writes, the trailer is most consistent with a
**per-write value** — a write-sequence counter, RNG seed, or a CRC computed over a buffer that includes
those volatile/leaked bytes. Exact algorithm undetermined; for EDIT, recomputing it is unnecessary if
the cabinet doesn't validate it (the in-game rename test in §9 accepted edited cards). For a clean CREATE
recipe, this is the one remaining unknown to pin down (test whether the cabinet rejects a zeroed trailer).

---

## 8. Kana character table — SOLVED + byte-validated (conf 0.97)

Game-internal **1-byte-per-kana** table (NOT Shift-JIS, NOT EUC-JP). Column-major gojuon, 15
consonant-rows per vowel column.

- Rows 0–14: (none) K S T N H M Y R W G Z D B P
- Columns: a = 0x00–0x0e, i = 0x0f–0x1d, u = 0x1e–0x2c, e = 0x2d–0x3b, o = 0x3c–0x4a (step 15)
- Vowel anchors: ア=0x00 イ=0x0f ウ=0x1e エ=0x2d オ=0x3c (screenshot-confirmed sat1 = アイウエオ)
- Non-existent kana (yi, ye, wi, we, wu) keep their grid slot but are unused (None in the table).
- **Extended:** 0x45=ン (the o-column W slot holds ン, not ヲ), 0x4b ァ, 0x4c ィ, 0x4d ゥ, 0x4e ェ, 0x4f ォ,
  0x50 ャ, 0x51 ュ, 0x52 ョ, 0x53 ッ, 0x54 ー.
- Terminator / pad = **0x7d**.
- Not yet mapped (low priority — names are katakana): digits, ヴ, dakuten/handakuten marks as standalone,
  punctuation.

Reference: `_jp_re/jp_decode.py` (forward), `jp_encode.py` (inverse, round-trip exact vs 4 cards).

### Decoded horses (frozen1)
| card | name | sire (mater idx, 1-based) | dam (mater idx) |
|---|---|---|---|
| sat1 | アイウエオ | トリックパワー (23) | アイリッシュダンス (139) |
| sat2 | コィゥォャ | トランスウインド (4) | ファイナルレコード (96) |
| sat3 | シヒニヌ | ジェイドロバリー (80) | マンハッタンレディ (119) |
| sat4 | トホメレ | トランスウインド (4) | シンコウラブリイ (137) |

All 7 distinct sire/dam names resolve to exact entries in `jp_mater_names.json`. Byte-level proof: the
sat4 sire bytes `3f 08 45 20 1e 0f 45 48` equal mater idx 4's `kana_hex` exactly (トランスウインド).
Note the decoded dam strings show a trailing `[8d][7c]`/`[ad][f9]` etc. — those are the 0x43–0x44
**trailer bytes** bleeding past a missing/late terminator in the naive 3-field walk, NOT real kana. The
true dam name ends at its last mapped kana before the zero-pad/trailer.

### ROM ↔ card bridge (conf 0.95)
Mater names live in the ROM as **EUC-JP** at `derbyo2k` 0x11106C, stride 60, name at +4. Verified:
0x11106C `トロットサンダー`, 0x1110a8 `ホワイトノーザー`, 0x1110e4 `ビワハヤヒデ`, 0x111120 `トランスウインド`,
0x11115c `ワイルドキャット`. 167 names total, 164 fully kana-encodable. Pipeline ROM EUC-JP → Unicode →
card kana table is byte-proven, and is the source for the editor's sire/dam dropdown.

---

## 9. In-game validation (from FINDINGS, consistent with the byte evidence)
- The card kana name field IS the in-game horse name (sat2 コィゥォャ, sat3 シヒニヌ displayed verbatim).
- A from-edit card (renamed to ジョン, sire kept トリックパワー) loaded into full gameplay (training 成功, time
  trial) — structure accepted, implying the cabinet does not hard-reject an edited name + unchanged
  lead bytes (and likely does not strictly validate the 0x43–0x44 trailer).

---

## 10. Open questions
1. CREATE recipe for a brand-new JP card: what lead-byte (0x25–0x27) scheme and what 0x43–0x44 trailer
   does the cabinet accept for a never-before-seen horse? (Edit path is solved; create is not.)
2. Exact 0x43–0x44 trailer algorithm — is it validated at all? Test: load a card with the trailer
   zeroed; if accepted, the editor can ignore it.
3. Cross-session persistence on real hardware: does a real linked DOC 2000 master persist career stats
   in BBSRAM keyed by the lead-byte ID? (Cabinet-side, not on-card; needs real hardware — emulation
   never persists due to the multiboard master reset, see BATTERY_RESUME_FINDINGS.)
4. Physical-card tiebreaker: scan a real DOC 2000 card for `SEGABBX0` + populated tracks 2–3. Absence
   confirms identity-only for hardware too; presence would mean Flycast's LR write path is the gap.
5. derbyoc (DOC '99) card: **RESOLVED 2026-07-26** - marker = **11 06** at 0x20/0x21 (NOT 03 02 as assumed); layout otherwise IDENTICAL to derbyo2k (2 live cards decode fully + trailer-CRC-match). Cloud/suite/box all accept both markers now.
   captured yet to confirm the identical 0x20/0x21 skeleton + kana name.

---

## 11. Physical-card findings (2026-07-24, MSR605 scans of a real DOC 2000 card)

Tooling: `C:\DerbyOwnersClub\Tools\JP-Card-Capture\msr_capture.py` (MSR605 on COM4, raw mode,
BPC 8/8/8, 210 bpi). Captures in `Tools\JP-Card-Capture\captures\` (card01 + scan2 = same card,
8 swipes). Card face (photo-verified): ジユウノメガミ, 牝馬(mare) 栗毛(chestnut), 父 バクシンプルー,
母 ナマシイタケ, "DOC2000" logo.

1. **Format CONFIRMED on real hardware.** The name track reads at the MSR605's ISO track-2 head
   position, byte-reversed on the stripe (as the Sanwa stream predicts), F2F at 210 bpi. Recovered
   logical track byte-identical across 7/8 swipes: header `03 02 00 00 00` @0x20, lead ID `48 28 31`,
   horse/sire/dam kana fields 0x7d-terminated, trailer `56 3c`. Decode matches the printed card
   face field-for-field. Sire and dam are NOT in the 167-name mater table → bred (2nd+ gen) horse
   with player-named parents; open Q4's "SEGABBX0 stat card" scenario: NOT seen (no SEGA marker
   anywhere on the stripe).
2. **Kana grid table CONFIRMED on real cards — and an MSR605 capture gotcha.** ユ=0x25 (printed
   ジユウノメガミ) and ヤ=0x07 (printed シュウジバカヤロウ) validate the previously-unobserved Y-row grid
   values; all 14 emu header constants match clean physical reads exactly. GOTCHA (initially
   misread as "+0x80 real-card differences"): raw reads whose logical track recovers at a
   NON-byte-aligned bit shift carry pseudo-random MSB flips (e.g. ユ 0x25 read as 0xa5; lead ID
   c6 ca 32 read as 46 4a b2). Proven by two swipes of the same card (ハカジュール) differing ONLY
   in MSBs — one matching all constants + ROM sire/dam, one not. Rule: **trust a recovery only if
   the 14-constant skeleton matches exactly; otherwise re-swipe** (varying speed/direction changes
   the framing). 2026-07-24 bulk scan: 15-card batch, 9 distinct cards recovered, several clean.
   NEW format fix: name/sire/dam are **fixed 9-kana slots** at 0x28/0x31/0x3a padded 0x7d (not
   terminator-walked; max-length names have NO pad — the old 3-field walk merges them).
   First-gen cards confirmed: ショウリ (sire ROM#4 トランスウインド, dam ROM#147 パルフィカス) and
   ハカジュール (sire ROM#78 ウイニングチケット, dam ROM#155 スマートサクセス) — both parents stock =
   unbred; player-named parents = bred, exactly as the identity-card model predicts.
3. **"Second track" WITHDRAWN — single wide track confirmed.** The data at the MSR605's ISO
   track-1 head position is an edge read of the SAME physical track (proven: card アメリカンレディー's
   T1-head stream decodes as its own dam/name/sire fields, byte-reversed). Matches the US card
   probe, where heads 1+2 returned byte-identical streams (one Sanwa track spans two ISO head
   pitches; Sanwa track pitch ≈ 2x ISO). US card geometry: heads 1+2 = a1 (name track, decoded
   "Fiyah Mixtape"), head 3 = the dense stat track, and the third (SEGA) track sits beyond head
   3's reach (flips/shims cannot reach it — MSR605 mechanical limit). On ALL JP cards scanned the
   head-3 position is EMPTY → JP DOC 2000 cards carry ONE written track, exactly as emulation
   showed. **Open Q4 substantially closed: identity-only confirmed on real hardware.**
   **2026-07-25 FULL COVERAGE:** John's modified MSR206U (COM5, PL2303GC; tool `msr206_live.py`)
   shifts the head band one track pitch: US card now reads ALL THREE Sanwa tracks on distinct
   heads (T1=a1 names, T2=a2 stats, T3=a3 — SEGABEF0 recovered at 5 bit errors + the 30 10
   marker). JP cards under the same geometry: T2/T3 still empty → single-track identity card
   now proven across every physical band. **Open Q4 CLOSED.** The a3-band pickup is edge-weak
   (bit errors) — multi-swipe voting required for clean a3 reads.
4. **The 0x00-0x1f header region IS the horse's career record — largely decoded 2026-07-24**
   (full field map + evidence: `Tools\JP-Card-Capture\CAREER_BLOCK_FINDINGS.md`; both generations
   share the structure right-aligned to the 03 02 header, old '99 index = offset+1 + a 7f/ff
   sentinel). CONFIRMED: 0x02-0x04 = current internal-stat triplet; 0x11/0x15/0x14 = birth stats
   (order stat1,stat3,stat2; byte-identical to current on the fresh card22 zero-control); old-gen
   0x1b = total races, 0x1c = wins x4, 0x1d = places x4 (wins+places<=races holds on all 24 old
   cards and uniquely repaired card31's ambiguous bits). LIKELY: 0x17 bit3 = sex (0=male,
   1=female; correct on all 7 known-sex horses), 0x1f bits5+6 = retired, 0x16 bits4+5 =
   has-raced. Falsifiers logged: DOC-2000 counters are NOT a straight carryover (ショウリ reads
   4 wins/0 races there), coat is NOT a plain byte, ハカジュール's G1 signal (bit7 cluster at
   0x09/0x0c/0x12) could still be a capture artifact. Next data: photograph the 24 old-book
   faces; before/after rescan of one card played on the DOC 2000 cabinet mesh.
   (Original note: the emu "volatile heap leak" verdict was Flycast masking real fields.)
   Evidence from the 10-card batch (5 clean photo-verified: ハカジュール, ショウリ, アメリカンレディー,
   アメリカンチャンプ, ブラウンエンジェル): 0x02-0x04 and 0x11-0x12 hold values in the internal-stat
   range (0x18-0x2e), and ハカジュール - the only card printed G1出走可能 - has the batch's highest
   values; the printer also knew sex/coat at write time. **DOC 2000 RACE COUNTER CRACKED
   2026-07-25 (office-rig before/after diff, live-verified 2→3): races = (byte[0x12] >> 4) + 1.**
   **CORPUS VALIDATION 2026-07-25:** 6 real played derbyo2k cards from the live cloud stable
   (races 1/2/2/2/3/3) — races formula holds 6/6; current triplet 0x02-04 + birth 0x11/0x15/0x14
   confirmed with visible growth (current ≥ birth, e.g. ウィロー cur 35/15/42 vs birth 34/12/41);
   sex bit 0x17&8 gives a sensible F/F/M/F/M/M split. NEW LEADS (need before/after or ground-truth
   to pin, don't display yet): (a) **0x05-0x07 = a second clean stat-range triplet** (0d/1c/14,
   21/12/0c, ... — 3 consecutive in-range bytes right after the current internals; candidate
   second stat set / retirement-peak / condition); (b) **win/earnings region = 0x09,0x0a,0x1b,0x1c**
   — zero on 5 of 6 cards INCLUDING a 3-race horse, nonzero ONLY on キキーウイロー (0x09=40, 0x0a=10,
   0x1b=02, 0x1c=10): tracks WINNING, not race count, exactly as a wins+earnings block should.
   High-entropy bytes 0x00/0x08/0x0b-0x0e/0x16/0x17 = ID/nonce/checksum material. Constants on
   these cabinet cards: 0x01=70, 0x10=00, 0x13=00, 0x18=0d, 0x19=00, 0x1d=00, 0x1e=00.
   Corpus saved at `Tools\JP-Card-Capture\jp_played_corpus.json`. To finish wins/places/earnings:
   race one of these horses to a WIN + save, diff the 0x09/0x0a/0x1b/0x1c region.
   Coat candidates 0x09/0x1a (the 芦毛 pair
   matches on both: 00/0f) but ショウリ vs ハカジュール (both 黒鹿毛) differ at 0x1a → not a plain
   coat byte; sex byte undetermined (1 stallion vs 3 mares = too many candidate offsets). Needs
   the remaining coats (青毛/特殊/栗毛 are exactly the damaged cards) or a rescan after real play.
5. **Batch status (10 physical cards):** 5 clean + photo-verified (above; ショウリ 12/13 constants,
   rest 13/13). ジユウノメガミ = old green "OWNER'S CARD" stock (not DREAM HORSES 2000): all 20
   reads byte-identical but only 9/13 constants and TWO face conflicts (stripe 0xa5 where print
   says ユ=0x25; stripe プ 0x2c where print says ブ 0x2b) - either an earlier-generation write
   format or a degraded stripe; unresolved (no reverse-direction read ever recovered). 4 cards
   unreadable/garbled despite stable captures: バクシンブラッデー (7/13, stable garble),
   メッッッッッッス (captures truncate right after the name field), ボボボーボボボボー and
   バッチリフェロモン (never recovered). Tooling: `Tools\JP-Card-Capture\` msr_capture.py
   (--bulk auto card-detect) + analyze2.py (constant-validated recovery, consensus, field hunt).

6. **Second card batch ("old book", 24 cards) CRACKED 2026-07-24** - initially looked like a
   different format, but it is the SAME kana table and layout, captured in mirrored bit
   orientation (the apparent separator `12 78 60` = bit-mirrored `03 0f 24` = タイム, the owner's
   naming motif; apparent pad `5f` = mirrored 0x7d). Layout matches DOC 2000: ~33-byte binary
   block (career record, zeroed on a fresh card - card22 トウカイテイオー unraced = negative
   control), header `03 02 NN 00 00` (NN nonzero here, meaning TBD), 3-byte lead, three 9-byte
   kana fields, short trailer. 23/24 decoded (19 high-conf), sires/dams validated against the
   **DOC '99 ROM name tables newly extracted from `epr-22099b.ic22`: 244-entry mater @0xF8480
   stride 18, 168-entry stock @0xF964C stride 56** (`derbyoc99_rom_names.json` /
   `derbyoc99_stock_names.json`). Breeding lineage links across physical cards confirmed (e.g.
   card13 sire = card12 horse). MSR605 read artifacts = deterministic single-bit flips at bit
   position == recovery shift; two reads at different shifts intersect to a unique decode -
   re-swipe cards 18/21/30/31/33 at varied speed/direction to resolve their ambiguous bytes.
   Full spec `Tools\JP-Card-Capture\OLD_FORMAT_FINDINGS.md`, decoder `old_decode.py`, results
   `old_decode_results.json`.

7. **FUSION COMPLETE 2026-07-25 — entire collection decoded** (`FUSION_RESULTS.md` /
   `final_cards.json`; two-reader fusion MSR605 + modified MSR206U): **25/25 '99 cards + 11/11
   DOC 2000 cards, zero ambiguous characters.** Supersedes the earlier "MSB scramble/stubborn
   corruption" caveats: the real cause was a missing recovery transform — some cards need
   **mirror framing** (`shift THEN reverse`, not reverse-then-shift); the framing shift is a
   **per-card property baked at write time** (same shift on both readers). With it, ジユウノメガミ
   and all "stubborn" cards decode 13/13 CLEAN (the 0xa5 byte = ユ 0x25 under correct framing;
   `candidates()`/`read_quality()` in msr_capture.py now include the mirror family). New kana:
   **0x55 = ヴ** (via エアグルーヴ; added to msr_capture, doc_suite_server.py/.ps1, slot_agent.ps1).
   Two printed faces were photo-misreads, stream is authoritative: sire パンダマン (not バンダマン),
   sire バクシンプルー (not ブルー). Fusion also found a 25th old card session A missed (アホッタレ /
   エアジハード / エアグルーヴ). '99 and DOC-2000 record layouts are IDENTICAL (content conventions
   differ: header NN, binary[2] 0x71 vs 0x70). Remaining opens: trailer checksum unidentified
   (exhaustive sum/XOR/CRC scan failed over 15 byte-exact cards), タイムスリップ dam wants a face
   photo, 11 A-only cards would upgrade to high with an MSR206 re-swipe.

## 11b. Career block decoded FROM THE ROM (2026-07-25) — supersedes the §11.4 corpus guesses

Full RE of the JP program ROM's own card codec: `reverse-engineering/findings/jp-card-io.md`
(ROM `epr-22284a.ic22`, CRC32 1E8E067C). **Base correction: the JP ROM loads at RAM
0x0C020000, same as WE — NOT 0x0C010000** (string-pointer self-reference test 504 vs 94;
mater data anchor at FILE 0x11106C is base-independent and still holds).

Card codec: **serializer 0x0C076B00** (record->card), **deserializer 0x0C076E44**
(card->record), bitfield insert helper 0x0C0C5970, width pool 0x0C076CB0. It is the SAME
code as the WE ROM's "old-format DOC2000 codec" (identical record offsets + byte-identical
helper) => **the JP horse struct is homologous to the WE record.** Bytes 0x02-0x17 and 0x1F
mapped field-by-field, confirmed bidirectionally in ROM AND against all 17 corpus cards:

| byte | field (CONFIRMED) |
|---|---|
| 0x02 / 0x03 / 0x04 | current internal **Speed / Stamina / Sharp** (ROM pins the ORDER; was guessed stamina/speed/sharp) |
| 0x05 / 0x06 / 0x07 | current **externals** Start / Corner / #3 (resolves the "second triplet" mystery — NOT internals) |
| 0x09 / 0x0A | **win record**, packed: SHOW=`(w08>>8)&0x3F`, PLACE=`(w08>>14)&0x3F`, WON=`(w08>>20)&0x3F` where w08=LE(card[8..0xB]). This IS the corpus "0x09/0x0A grow on winners". |
| 0x0B | wins hi-bits + **hearts** (0-63) |
| 0x11 / 0x15 / 0x14 | **birth** internal Speed / Stamina / Sharp |
| 0x12 | high nibble = races-1 (= OUT counter rec123). **races=(0x12>>4)+1 CONFIRMED 17/17** |
| 0x17 | bit3 = **sex** (0=M,1=F); hi nibble = silk color1 |
| 0x1F | top 3 bits = **silk PATTERN** (NOT "retired bits 5+6" — that guess is wrong) |
| 0x43 / 0x44 | trailer = **CRC-16-CCITT (poly 0x1021) over card[0x00..0x42]**, LE (resolves §7/§10 Q2) |

Corrections to the §11.4 / CAREER_BLOCK_FINDINGS corpus guesses: stat order is Spd/Sta/Shp;
0x05-07 are externals; 0x09/0x0A are win/place/show (not hearts/coat); 0x1F top-3-bits is
silk pattern (not retired).

**Trailer 0x43/0x44 PINNED 2026-07-25 (resolves §7 + the §10 open trailer Q, conf 1.0):**
CRC-16/CCITT (poly 0x1021) in the high 16 bits of a 32-bit reg, init 0xDEBDEB00, over card
0x00..0x42 (67 bytes) + 8-bit flush, result >>16 stored LE (card[0x43]=low, card[0x44]=high);
6/6 real cards; loader recomputes + rejects on mismatch (validator 0xC075142, gate 0xC075EB4),
writer 0xC0765A2 — so a CREATE must compute it. Algo + Python ref: `findings/jp-card-trailer.md`.

### Round 2 (2026-07-25): earnings, retirement, externals, coat/personality/G1 — all resolved
The serializer 0x0C076B00 is the COMPLETE card writer (its tail after `bra 0xc076cd0`,
0x0C076CD0-0x0C076DE2, writes 0x18-0x27 + kana names). No separate routine; the round-1
"wrapper reframing / sibling" guess was wrong. Corpus-validated:

- **EARNINGS** = `card[0x1A] | card[0x1B]<<8 | (card[0x1C]&3)<<16` (rec+80, 18-bit).
  Monotonic with career + win-weighted: 1 race 15-110, winner キキーウイロー 750,
  ハカジュール 1840, タイムスリップ 3720. This is the corpus 0x1B/0x1C "win/earnings"
  candidate — it is EARNINGS, separate from the win record at 0x09/0x0A.
- **RETIRED = card[0x01] bit0** (deserializer 0x0C076EA8 reads it into rec+95). Set only on
  タイムスリップ in the corpus. card[0x01] = `0x70(const hi nibble) | rec114<<1 | retired`;
  the "0x70 const, bit0 varies" is retired, not a "collection marker."
- **6 externals** (Start/Corner/OOB full bytes 0x05/06/07; Comp/Tenac/Spurt 6-bit in
  word0x0C): Comp=`(LE(0x0C..0x0F)>>20)&0x3F`, Tenac=`>>14&0x3F`, Spurt=`>>8&0x3F`.
  Leg-type caveat: Start(byte) vs Comp/Tenac/Spurt(6-bit) are scale-mixed — verify the
  intended comparison on the live rig before rendering leg type.
- **Retirement externals** (rec108-113, 4-bit): 0x12&0xF; `(LE(0x14..17)>>22)&0xF`;
  0x0C hi/lo nibbles; 0x08 hi/lo nibbles.
- **Silks**: color1=`card[0x17]>>4`, color2=`(card[0x16]>>2)&0xF`, pattern=`card[0x1F]>>5`.
- **Coat/personality DO exist on the JP card** — the "lead ID 0x25-0x27" (§5) is REINTERPRETED
  as genetics: card[0x24]=coat_mod (0 for normal coats -> looked like const 0), 0x25=coat_base,
  0x26=seed_lo, 0x27=personality_hi (rec+86 coat word, rec+88 seed|personality word).
  Header 03 02 @0x20/0x21 = rec+84 marker word 0x0203; 0x22/0x23 = rec127/rec126.
- **G1 titles**: field present (rec+76, ~21-bit) at card 0x1C bits2-3 + 0x1D + 0x1E + 0x1F
  bits0-4; **0 on all 17 corpus cards** (none have G1 wins; eligibility != titles).
- **Dirt**: card[0x00] = rec+94, but corpus 0x00 is high-entropy (ambiguous: wide value or
  doubles as an id).

Full details + exact (word,bit,width) per field: `reverse-engineering/findings/jp-card-io.md` §8.
Remaining open: exact leg-type external comparison (scale), earnings display scale, CRC init.

## 12. Tool ideas this unlocks
- **JP card creator/editor** (extend the Stable Management System): kana name input + sire/dam dropdowns
  sourced from `jp_mater_names.json`, writing via `jp_encode.write_fields`. Edit path is fully ready;
  gate CREATE behind resolving open Q1/Q2.
- **`parseCardFacts` JP branch**: show name/sire/dam + lead-ID; explicitly label "stats are cabinet-side
  (identity card)" instead of "pending hardware reader," since we now know there are no on-card stats.
- **JP↔EN pedigree cross-reference**: map each mater name to its 244-record racing-stat index so the JP
  card can *display* the sire/dam's ROM stats (pulled from the racing table) even though the foal's own
  stats aren't on the card.
- **Trailer probe harness**: auto-generate cards with varied 0x43–0x44 values and log cabinet
  acceptance, to nail the CREATE recipe.
