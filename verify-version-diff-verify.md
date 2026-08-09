# Adversarial Verification — version-diff.md (DOC 4-version diff)

Independent re-extraction from the real `.ic22` bytes (all four 4 MB NAOMI ROMs).
Date: 2026-06-03. Method: python3 byte slicing, never reading binaries into context.
Helper scripts: `C:/Users/johnr/AppData/Local/Temp/v*.py`.

## Verdict: MOSTLY-SOLID

The doc is largely accurate and well-evidenced. Header/signature, name tables, breeder
counts, oc 28-byte layout, G1/track offsets, and the beer/item table all confirm exactly.
**One headline claim is WRONG and one minor header field is wrong.** Corrections below.

---

## CONFIRMED claims (byte-verified)

### Section 1 — Header / build markers
- 0x000 = `NAOMI` (16B padded), 0x010 = `SEGA ENTERPRISES,LTD.` — all 4. CONFIRMED.
- Title strings: revc/revd = `DERBY OWNERS CLUB WE`, o2k/oc = `DERBY OWNERS CLUB` (no WE).
  Actual strings begin at 0x031 with a leading space (` DERBY OWNERS CLUB WE --------- `),
  doc's "0x030" with leading-space-included quote is consistent. CONFIRMED.
- 0x130 build date (LE u16 year, u8 mon, u8 day):
  - revc & revd: `d1 07 0a 1e` -> 2001-10-30. CONFIRMED.
  - o2k & oc: `cf 07 0a 01` -> 1999-10-01. CONFIRMED.
  - revc and revd share identical header date/serial. CONFIRMED (important: they are NOT
    distinguishable by header — only by 0x8000 sig and table offsets).
- 0x134 serial: revc/revd `BEF0`, o2k `BBX0`, oc `BAX0`. CONFIRMED all 4.
- 0x8000 16-byte build signature: all 4 match the doc's listed bytes EXACTLY. CONFIRMED.
  This is the edit-proof version fingerprint.

### Section 2 — Racing stat table anchors / field map
- Anchors (record-start, stride): revc 0x108E03/32, revd 0x10A14B/32, o2k 0x10AD1B/32,
  oc 0x0F6902/28. CONFIRMED (records decode sanely at all four).
- **revc == revd for all 244 records: TRUE, byte-exact.** CONFIRMED.
- Field map on revc CONFIRMED:
  - +0 const 0; +2/+3 = horse ID = (index+1), dup (both verified across 244).
  - +4,+6,+7,+15,+17..20,+26..28 all const 0 (verified distinct-set == {0}).
  - +8 grade dist = {0:80, 1:65, 2:50, 3:49}. CONFIRMED exactly.
  - +9..+14 externals ranges: +9 11-63, +10 14-59, +11 4-63, +12 8-63, +13 3-62, +14 4-63.
    CONFIRMED (matches doc's ~11..63 etc.).
  - +29 stamina 0-60, +30 speed 0-63, +31 sharp 0-60. CONFIRMED.
  - +1 flagA dist {0:175, 1:62, 2:7}. CONFIRMED exactly.
  - +16 flagB dist {0:200, 1:37, 2:7}. CONFIRMED exactly.
  - +21 category dist {1:98, 2:69, 3:44, 0:30, 7:3}. CONFIRMED exactly.
  - +22 coat top values {207:107, 204:54, 202:39, 222:21, 199:15, 192:4, 193:3}. CONFIRMED.
- **+25 seq ID: doc claims "(index+1)&0xFF, 1..243,0". MY CHECK: it is NOT a strict
  (index+1)&0xFF for all 244** — the equality test returned False. BUT the sample
  (first6 = 1,2,3,4,5,6; last3 = 242,243,0) matches the doc's stated pattern, so it is
  approximately a second id/seq byte that mostly tracks index+1 but deviates somewhere.
  PARTIAL — flagged below.

### Section 2 — oc 28-byte layout
- oc horse #1 externals `2c 23 13 20 28 2e` at oc+9..+14 == revc+9..+14 byte-for-byte.
- oc internals `17 25 30` at oc+24..+26 == revc+29..+31 byte-for-byte.
- oc +1/+2 == index+1 (id dup) for all 244. CONFIRMED.
- oc +7 grade dist {0:35,1:85,2:68,3:56} (sane 4 buckets). CONFIRMED structurally.
- oc +19 coat dist matches revc coat palette {207,204,202,222,199,192}. CONFIRMED.

### Section 3 — Name tables
- Bases/stride: revc 0x10AD50/18, revd 0x10C098/18, o2k 0x10CC68/18, oc 0x0F8480/18.
- revc ASCII decodes clean (Gold Queen, First Star, ... Gold fighter, Sunday Star...). CONFIRMED.
- o2k/oc EUC-JP decode clean katakana. o2k #1 アイオーユー, #2 アインブライド, #244 マチカネキンノホシ.
  CONFIRMED exactly (matches doc).
- トロットサンダー marker present in o2k at idx 106 -> offset 0x10d3ca. CONFIRMED exactly.

### Section 4 — Roster diffs
- **WE revc->revd: exactly 16 racing names changed.** Indices (1-based): 6,27,101,102,110,
  111,116,117,123,157,161,181,195,197,201,207. ALL 16 names match the doc's table
  byte-for-byte (Gold fighter->Gold Fighter, El Condor Pasa->Steppin' Out, ...,
  Tomrrow's Dream->Tomorrow's Dream, Flower Dance->Mr. Vice President). CONFIRMED exactly.
- **WE breeders revc->revd: 26 sires + 84/84 dams differ.** CONFIRMED exactly.
  Examples confirmed: sire Vinny, Maverick, Malski; dam Pierogi Prince, It's About Time,
  Bet the Rent, Mr. Original. Bases revc sire 0x10BF1C / dam 0x10D2CC; revd sire 0x10D264 /
  dam 0x10E614, stride 60 — all decode sanely. CONFIRMED.
  (Minor: doc's prose says revc sires include "Sunday Silence, Helissio, Tony Bin, Carnegie";
   the actual revc sire records #1-8 are Maple Syrup/Heart Lake/Judge Angelucci/etc. The
   *count* and replacement examples are right; that specific real-name list is loose/wrong.)
- **JP oc->o2k: exactly 64 racing names differ.** CONFIRMED exactly. Spot pairs confirmed:
  #3 アドラーブル->コクトジュリアン, #195 アラビアンナイト->テイエムオペラオー, #197 エミーローズ->アグネスワールド.

### Section 5 — G1 races & tracks
- revc G1 0x0C6CA0 and revd G1 0x0C65C0 both contain identical G1 names (WINTER STAKES,
  DOC 1000/2000 GUINEAS, SPRING CLASSIC, AMERICAN DERBY/OAKS, HONG KONG OAKS/DERBY,
  SUMMER GRAND PRIX, SUPER DIRT GRAND PRIX, STAYERS STAKES, QUEEN ELIZABETH CUP, MILE
  CHAMPIONSHIP, JAPAN CUP, SPRINTERS STAKES, DERBY OWNERS CUP, JAPAN CUP DIRT,
  SPRINTERS TROPHY). CONFIRMED. (Note: there is also a "NO NAME" slot between SUPER DIRT
  GRAND PRIX and STAYERS STAKES, not mentioned in doc — minor.)
- Track table revc 0x0C6940 / revd 0x0C6260: identical EASTERN CITY TURF/DIRT 1200-2400M.
- **-0x6E0 shift verified arithmetically:** 0x0C6CA0-0x6E0 = 0x0C65C0 (G1), and
  0x0C6940-0x6E0 = 0x0C6260 (tracks). CONFIRMED exactly.

### Section 7 — Item/feeding (beer) table
- Diff of `beer_effects_test.ic22` vs base `epr-22336c.ic22`: exactly 12 changed bytes,
  two contiguous groups of 6: 0x1671FC-0x167201 (00->02) and 0x167228-0x16722D (00->04).
  CONFIRMED EXACTLY. Locates feed/item-effect table near 0x167200.

---

## CORRECTIONS (claims that are WRONG / off)

### CORRECTION 1 (HEADLINE) — "revc == o2k for all 244 records" is FALSE
The doc's MAJOR FINDING that the 32-byte stat table is byte-identical across revc, revd AND
o2k is only TRUE for revc==revd. **revc vs o2k has 22 differing records** (out of 244).
First divergence at record index 12 (horse #13). Differences span real stat bytes, not just
flags:
- offsets that differ across the 22 recs: +5(dirt,1), +9..+14 externals (6,2,3,5,4,3),
  +16 flagB(9), +22 coat(1), +23 U1(8), +24 U2(5), +29..+31 internals (5,3,4).
- e.g. rec 12 differs only at +16 (revc 00 vs o2k 01); rec 99 differs at +23 (92 vs 178) and
  +24 (160 vs 169); several differ in externals/internals.
IMPACT: the doc's "unified horse editor writes to all three identically" and "patch
transposer applies revc edit to o2k unchanged" (Sections 0 intro, 10) are UNSAFE for those
22 horses. The stat data is ~91% shared (222/244 identical) but NOT fully shared with o2k.
revc==revd remains fully safe.

### CORRECTION 2 — header word at 0x144
Doc says 0x144 = `03 00 00 00`. Actual bytes at 0x144 are `00 00 00 00` for all 4 ROMs.
(Possibly the doc meant a different offset; the value `03` does not appear at 0x144.)

### PARTIAL — +25 "seq ID == (index+1)&0xFF"
The strict equality test fails across all 244, though the sampled pattern (1..6 then
242,243,0 at the tail) matches the doc's loose description. So +25 is a near-sequential id
byte with at least one deviation; the "(index+1)&0xFF" formula is not exact. Low impact.

---

## Confidence summary
- Header/sig/serial/date: 0.99 (byte-exact).
- revc==revd stats byte-identical: 0.99.
- revc==o2k stats byte-identical: REFUTED (0.99 confidence it is false; 22 diffs).
- Field map (+0..+31): 0.9 (distributions/ranges all reproduced; +25 formula partial).
- Name tables + 16/64 roster diffs: 0.99 (exact match).
- Breeder 26 sire / 84 dam: 0.97 (counts + examples exact; one loose prose detail).
- oc 28-byte layout: 0.95.
- G1/tracks + -0x6E0 shift: 0.98.
- Beer/item table @0x167200: 0.99.
