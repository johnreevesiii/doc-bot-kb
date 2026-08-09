# Adversarial Verification: tracks-races (KEY: tracks-races-verify)

Independent re-extraction of the offset/format claims in
`C:/DerbyOwnersClub/_core/areas/tracks-races.md`, pulled directly from the four
real `.ic22` ROM byte streams (never read into context as text; only specific byte
ranges extracted with python3). Verdict: **mostly-solid**. The structural picture
is correct and almost every offset/count confirms. Two real numeric errors found
(a `derbyoc` course count and the WE G1 "races" count), plus a few offset rows in
the doc that are 1-3 bytes off because of inline control bytes.

---

## What confirmed (verified against bytes)

### 0x8000 build signatures — ALL FOUR confirmed exactly
```
drbyocwc epr-22336c  dc99020c9cc8210c   (matches doc)
derbyocw epr-22336d  09004ad20ee347d0   (matches doc)
derbyo2k epr-22284a  162f047ffcf5fcf6   (matches doc)
derbyoc  epr-22099b  188bee02f1532838   (doc wrote "188bee02f15328 38" w/ stray space; bytes = 188bee02f1532838)
```
All four ROMs are exactly 4,194,304 bytes (0x400000), uncompressed. Confirmed.

### WE Rev C (drbyocwc) course list @ 0x0C6940 — CONFIRMED
- 36 entries, null-terminated ASCII, no per-entry attribute byte. Confirmed.
- Venue breakdown EASTERN CITY 7 / WESTERN HILL 6 / NORTHERN PARK 6 / CENTRAL CITY 8
  / SEGA 5 / SOUTHERN PARK 4 = 36. **Exact match.**
- Surface split 26 TURF + 10 DIRT. **Exact match.**
- Raw hex of first entry `45 41 53 54 45 52 4E ... 31 36 30 30 4D 00` =
  `"EASTERN CITY TURF 1600M\0"`. Confirmed (attributes are literal text, no binary).
- NOTE (minor): doc lists NORTHERN PARK rows at 0xC6A94/0xC6AB0/0xC6ACC/0xC6AE8/0xC6B04
  and a couple SOUTHERN/CENTRAL rows at slightly different offsets than reality
  (real: 0xC6A91/0xC6AAD/0xC6AC9/0xC6AE5/0xC6B01, 0xC6C49/0xC6C65/0xC6C81). The
  drift is 1-3 bytes and is caused by leading inline control bytes on those strings.
  Content, order, and count are all correct; only some printed offsets are approximate.

### WE Rev C G1 names @ 0x0C6CA0 — CONFIRMED (with a count correction, see below)
All 19 non-empty strings decode exactly as the doc lists them, in order:
WINTER STAKES, DOC 1000 GUINEAS, DOC 2000 GUINEAS, SPRING CLASSIC, AMERICAN DERBY,
HONG KONG OAKS, HONG KONG DERBY, AMERICAN OAKS, SUMMER GRAND PRIX,
SUPER DIRT GRAND PRIX, NO NAME, STAYERS STAKES, QUEEN ELIZABETH CUP,
MILE CHAMPIONSHIP, JAPAN CUP, SPRINTERS STAKES, DERBY OWNERS CUP, JAPAN CUP DIRT,
SPRINTERS TROPHY.
- `QUEEN ELIZABETH CUP`: raw bytes `...454c495a414245544820 BB F7 435550` — the
  `bb f7` control pair sits between "ELIZABETH " and "CUP". **Confirmed exactly.**
- `NO NAME` placeholder present at 0xC6D52. Confirmed.
- Inline separators `0f`, `ff 0f`, `0f ff 0f` confirmed in the raw hex of each run.

### WE Rev C Special @ 0x0C70C8 (12) and Handicap @ 0x0C7248 (12) — CONFIRMED
Exactly 6 venues x 2 surfaces each, in the full venue order, with the left-padded
venue column format, e.g. `"SEGA          TURF (SPECIAL)"` and
`"SOUTHERN PARK DIRT (HANDICAP)"`. Both counts = 12. **Exact match.**

### Coat/sex labels before course list — CONFIRMED
`BLACK, BAY, BROWN, SPECIAL, WHITE, MALE, FEMALE, GELDING` all present at
0xC6901..0xC693E, preceded by `GRAY`/`CHESTNUT`. Matches doc.

### WE EX Rev D (derbyocw) shift -0x6E0 — CONFIRMED
- Course list @ 0x0C6260: 36 entries, **byte-identical** to Rev C
  (`d[0xC6940:+24] == d[0xC6260:+24]` is True), string lists equal.
- G1 @ 0x0C65C0: identical 19 strings.
- Special @ 0x0C69E8: 12, first = "EASTERN CITY  TURF (SPECIAL)".
- Handicap @ 0x0C6B68: 12, first = "EASTERN CITY  TURF (HANDICAP)".
All confirmed.

### DOC 2000 JP (derbyo2k) — CONFIRMED
- Course list @ 0x0CA335: 36 (counted by full-width Ｍ token a3cd = 36). Decodes to
  東京芝１６００Ｍ ... 中京ダート１７００Ｍ. Confirmed.
- G1 @ 0x0CA62D: **21** null-delimited entries, ending 高松宮記念 and including
  ジャパンカップダート. Confirmed.
- Special @ 0x0CAB0B: 12 (レース label count = 12). Confirmed, incl. 中京/Chukyo.
- Handicap @ 0x0CAC5B: 12 (東京 芝 ... 中京 ダート （ハンデ）). Confirmed, incl. Chukyo.
- `錏` artifacts on 安田記念 / 宝塚記念 / 菊花賞 confirmed (stray 0xff control byte
  before the real name).
- Binary metadata region: `80 3f ff ff ff ff 05 24` IS present at **0x0CADB2**
  (doc said "0x0CAD7B+", correct neighborhood). The `xx04` marker = `8c 04` at
  0x0CAD8B. The leading `0f ff 0f ff 0f` then floats (`a6 42`=83.0, `20 43`=160.0,
  `80 41`=16.0, repeated `80 3f`=1.0) — clearly IEEE-754 LE float records. Doc is
  right that this exists and is undecoded; it is real, not imagined.

### DOC '99 JP (derbyoc) — MOSTLY CONFIRMED (one count wrong)
- NO ハンデ (handicap) anywhere in the entire ROM. `find(ハンデ)` = NOT FOUND.
  **Confirmed** — handicap table genuinely does not exist in '99.
- NO 高松宮記念 and NO ジャパンカップダート anywhere in ROM. **Confirmed.**
- No 中京/Chukyo venue in the course region (token count = 0). **Confirmed.**
- SEGA = 3 courses. **Confirmed** (東京6/阪神6/中山6/京都8/セガ3, no Chukyo).
- Special @ 0x0BDEE7: 10 (レース label count = 10). **Confirmed.**
- G1 @ 0x0BDAD5: 19 null-delimited entries, last = ダービーオーナーズカップ. (doc count
  correction below.)

---

## CORRECTIONS (claims that are off/wrong)

### C1. derbyoc '99 course count is 29, NOT 30
The doc (Sections 6 and 7) states 30 courses for '99. The actual count is **29**.
Two independent methods agree:
- Full-width Ｍ (EUC `a3cd`) terminators in 0x0BD875..0x0BDAD5 = **29** (last at
  0xBDAD2, immediately before the G1 block at 0xBDAD5).
- Venue token sum: 東京 6 + 阪神 6 + 中山 6 + 京都 8 + セガ 3 + 中京 0 = **29**.
So the '99 venue distribution is 5 venues / 29 courses (not 30). Everything else in
the '99 lineage claim (5 venues, no Chukyo, SEGA=3, no handicap, 20-ish G1) stands;
just the course total should read 29. This also flows into the Section 7 summary
table ("Courses: 30" -> 29).

### C2. WE G1 "19 races" overcounts by one — it is 18 real races + NO NAME
The WE G1 block is **19 non-empty string runs** (20 null-delimited slots incl. one
trailing empty). One of the 19 is the `NO NAME` placeholder, so there are **18 real
race names + NO NAME**. The doc's phrasing "20 strings (19 races + 'NO NAME')" is
internally inconsistent: it counts NO NAME inside the 19 and then lists it again.
Correct statement: *"19 string runs (18 real G1 names + 1 NO NAME placeholder)."*
The enumerated list in the doc (Section 2) is itself correct and contains exactly
19 names including NO NAME — only the summary count label is wrong. Section 7's
"20 strings (19)" should read "19 strings (18)".

### C3. derbyoc '99 G1 count: doc says 20, bytes show 19
The '99 G1 region 0x0BDAD5..0x0BDC2D holds **19** null-delimited non-empty entries
(ending ダービーオーナーズカップ). The doc says 20. Likely the author counted a
trailing empty/control slot the same way the WE "20" arose. The substantive claim
(no 高松宮, no JCD vs DOC 2000's 21) is correct; the raw count should be 19, not 20.

### C4. 0x8000 sig for derbyoc has a stray space in the doc
Doc line 12 prints `188bee02f15328 38`. The actual 8 bytes are `188bee02f1532838`
(no space). Cosmetic.

---

## Net assessment per section

| Doc claim | Status |
|---|---|
| All four 0x8000 sigs | CONFIRMED (1 cosmetic space typo) |
| Tables are null-terminated display strings, no attribute byte | CONFIRMED |
| WE course list 36 / 7-6-6-8-5-4 / 26T+10D @0x0C6940 | CONFIRMED |
| WE G1 names + QE CUP `bb f7` + NO NAME @0x0C6CA0 | CONFIRMED (count label off, C2) |
| WE Special 12 / Handicap 12 @0x0C70C8/0x0C7248 | CONFIRMED |
| Coat/sex labels | CONFIRMED |
| Rev D -0x6E0 shift, byte-identical, all sub-offsets | CONFIRMED |
| o2k 36 courses / 21 G1 / 12 special / 12 handicap | CONFIRMED |
| o2k binary metadata `80 3f ff ff ff ff 05 24` region | CONFIRMED present @0x0CADB2 |
| derbyoc '99: no handicap, no Chukyo, SEGA=3, no 高松宮/JCD | CONFIRMED |
| derbyoc '99 course count = 30 | WRONG -> 29 (C1) |
| derbyoc '99 G1 count = 20 | OFF -> 19 (C3) |

## Method
- python3 over exact byte ranges only; ASCII blocks parsed null-delimited keeping
  printable runs; JP blocks parsed null-delimited and EUC-JP decoded after stripping
  leading inline control bytes. Counts cross-checked with token searches
  (full-width Ｍ `a3cd`, ダート `a5c0a1bca5c8`, レース `a5eca1bca5b9`, ハンデ
  `a5cfa5f3a5c7`, venue 2-char EUC tokens) to avoid relying on the noisy
  control-byte stripping. Float interpretation of the o2k binary region by reading
  LE IEEE-754 words.
