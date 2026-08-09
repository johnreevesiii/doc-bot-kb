# derived-attrs.md — Adversarial Verification

Verifier pass against real ROM bytes (epr-22336c/d, epr-22284a, epr-22099b) + tool source.
Helper: `C:/Users/johnr/AppData/Local/Temp/verify_re.py` (+ v_racing/v_breed/v_strings/v_alm/v_eb).
Date: 2026-06-03. Verdict: **MOSTLY-SOLID** (every load-bearing offset confirmed in bytes; only the
÷51 running-style *opcode* and a few enum *meanings* remain inference, which the doc already flags).

## Build signatures (CONFIRMED, byte-exact @0x8000)
All four match the doc: drbyocwc `dc99020c9cc8210c`, derbyocw `09004ad20ee347d0`,
derbyo2k `162f047ffcf5fcf6`, derbyoc `188bee02f1532838`. All 4 ROMs = 4,194,304 bytes.

## §1 Racing + breeding tables (CONFIRMED)
- Racing table starts/strides verified by per-column min/max/uniq scan over 244 records, all 4 ROMs:
  drbyocwc 0x108E03/32, derbyocw 0x10A14B/32, derbyo2k 0x10AD1B/32, derbyoc 0x0F6902/28. The +2/+3
  id columns hit exactly 1..244 uniq=244 at all four → table bounds correct.
- WE 32-byte map confirmed: +1 class(0-2), +2/+3 id, +5 dirt(0-255,59u), +8 grade(0-3),
  +9..+14 externals, +16 sex(0-2), +21 enum(0-7,5u), +22 coat(0/192-222,8u), +23 sub(81u),
  +24 composite(28u), +25 id3, +29/30/31 internals. **Matches the doc's table exactly.**
- derbyoc 28-byte = same minus 4 pad: dirt+4, grade+7, ext+9..+14, sex+17, coat+19, internals
  +24/25/26, extra enum +18(0-3). **Confirmed by scan.**
- **Nuance (minor correction):** doc says externals "span min 3 .. max 63". True for the *aggregate*
  (min 3 occurs at +13 tenacious). But the +9 *start* column min is **11** in the WE ROMs (not 3).
  In derbyoc all six ext columns reach 3. Not wrong, just worth noting the per-column floors differ.
- Breeding tables: drbyocwc sire 0x10BF1C / dam 0x10D2CC, stride 60. **Byte-exact confirmed:**
  Maple Syrup st=39/sp=19/sh=34/ac=240/ext=[15,3,6,10,4,12] idx=1 b45-47=`e011f0`; Heart Lake,
  Judge Angelucci, Song Sung Blue, Trust Me, Wild Sun(ac=255) all match JSON. Dam table first idx=85
  → confirms sire 1-84 / dam 85-168 split. Ext at +48..+53 observed 3..15 (1-16 band scale). ✔

## §2 Running style (CONFIRMED labels; ÷51 = INFERENCE)
- 5-style EN cluster at exactly drbyocwc 0x128ED0 / derbyocw 0x12B128:
  `Front-runner / Start dash / Last spurt / Stretch-runner / Almighty`. "Almighty" at 0x128F08 /
  0x12B160. ✔ JP ROMs: zero hits for any EN style word (EUC-JP localization confirmed).
- `legTypeFromExt()` (Card-Creator L1286) matches doc word-for-word (corner excluded, rank-based,
  all-equal→Almighty). **No `a1[7] =` assignment exists** (grep empty) → "editor never writes byte 7"
  is TRUE. ✔
- **`floor(byte7/51)` is a derivation, not byte-proven.** 255/5≈51 buckets cleanly and the doc
  itself marks this HIGH-confidence-by-inference (5-style table + project CLAUDE.md), not disassembly.
  Fair as stated; the actual SH-4 opcode reading byte 7 was not located. Treat as strong hypothesis.
- §2c all-equal-31 sentinel: **CONFIRMED byte-exact** — the only 5 records with all six externals
  equal are ids 8,41,83,174,182, all =31. ✔

## §3 Personality (CONFIRMED)
- EN labels @0x0E84A4: `Imposing, Honest, Rough, Coward, Sloppy, Too soft, Strict` (7) ✔
- Romaji @0x107DFC: `Doudou, Sunao, Arai, Okubyou, Zubora` ✔. Duplicate cluster: "DouDou" sits at
  0x0EB614 (just before the doc's 0x0EB61C), then Sunao/Arai/Okubyou/Zuboro/_HAPPY_... ✔ (doc's
  0x0EB61C anchor is one record late but the cluster is real).
- `PERSONALITY_MAP={R:0,I:48,C:64,H:80,S:208}` (L451) and `getPersonalityCode` bands match doc's
  5-bucket model exactly. ✔ a1[6] read L667 / write L807. ✔
- Interaction floats @0x0E7D00: real IEEE-754 LE (0.8, 1.0, 0.8, 0.1, 0.1, 0.0, 0.2, 1.0, 1.2, 1.5,
  0.8, 2.0...) — consistent with "×2.0..−2.0 multiplier table." ✔

## §4 Aptitude symbols (CONFIRMED)
`symFor(v)` (ROM-Studio L248): v>=13→◎, v>=9→○, v>=5→△, v>=1→✕, else · — matches doc's quartile
logic exactly. ✔ `ac` at name+36 confirmed = JSON. name+45..47 3-byte block confirmed varying
(`e011f0`,`ccff3c`,`ccab34`...) with first-nibble C-E clustering — still PARTIALLY UNKNOWN as doc says.

## §5 Growth type (CONFIRMED)
@0x0EE270: `Speed type / Stamina type / Sharp type / Stud reg. / Dam reg. / Sire / Dam`. ✔ The
correction that this is the retirement string block (NOT a leg-type table) is valid.

## Net
No false offsets found. Two pedantic notes: (a) per-column external floors differ (start col min 11
on WE, not 3); (b) the 0x0EB61C romaji-dup anchor is ~1 record late (cluster begins 0x0EB614 "DouDou").
The ÷51 running-style mapping and several enum *meanings* (+1 class, +21, +24) remain reasonable
inference, correctly flagged as open in §9. Everything load-bearing reproduces from the bytes.
