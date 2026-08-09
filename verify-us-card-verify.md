# Adversarial Verification — `us-card.md` (US/World 207-byte card)

**Verifier run:** Jun 2026. Method: independent re-decode of all 207-byte `.card` files in
`Card-Library/`, `Tools/Cards/`, and `Play-*` station data using the reversal convention
`card[t*69 + (69-k)] = arHex[k]` (the exact rule in `Tools/DOC-Card-Creator.html` lines 904-916),
cross-checked against the ROM product-code byte at NAOMI header `0x134` in all four `.ic22` ROMs.
Binaries never read into context; only specific byte ranges extracted via python3.

**Overall verdict: SOLID.** Every load-bearing field offset, the container layout, the SEGABEF0
detection, the G1 bitfield, the coat tables, the earnings formula, the UID triplication, and the
cross-version product-code mapping all reproduce exactly from real bytes. Two cosmetic/scope errors
found (one wrong hex in a table cell; one over-generalized JP header claim). Nothing load-bearing is
broken.

---

## What I confirmed (with byte evidence)

### Container & reversal (sec 1) — CONFIRMED 1.0
- All 7 Card-Library cards + Tools/Cards + station cards are exactly 207 bytes.
- Reversal rule verified directly in source (`card[t*69+i] = arr[69-i]`, line 913) and by the fact
  that names/sire/dam decode to clean ASCII only under that rule.
- `getString` walks DOWN start→end+1 keeping printable bytes (source line 547). Confirmed.

### Card-type detection (sec 2) — CONFIRMED 1.0
- `SEGABEF0` at file 0x8A-0x91 present on every US card (38 US cards found across the tree).
- 19 non-US cards have zeroes there. Detection is reliable.
- **ROM product-code claim CONFIRMED at the binary level:** header `0x134` holds the 4-char code:
  - drbyocwc Rev C → `BEF0`  → SEGABEF0
  - derbyocw Rev D → `BEF0`  → SEGABEF0  (identical to Rev C)
  - derbyo2k DOC2000 → `BBX0` → SEGABBX0  (matches doc "DOC2000v2 SEGABBX0")
  - derbyoc DOC'99 → `BAX0`  → SEGABAX0  (matches doc "DOC RevB SEGABAX0")
  - DOC II `SEGABDY0` — not in our 4-ROM set, unverified but consistent with the pattern.

### Track 1 identity/genetics (sec 3) — CONFIRMED 1.0
- a1[6] pers @0x3F, a1[7] style @0x3E, a1[8] coat @0x3D, a1[9] mod @0x3C, a1[10] @0x3B — all offsets correct.
- Names: Name=a1[69..51], Sire=a1[49..31], Dam=a1[29..11] (source populateForm lines 663-665).
  Examples decoded clean: "Scarecrow II"/"Scarecrow"/"Candy Kane", "Caitin Clark"/"Vinny"/"Can't Stop Now".
- Personality examples: Scarecrow=0, BabyBoy=164. Confirmed.
- **Coat special table CONFIRMED exactly** against `COLOR_OPTIONS` (lines 434-448):
  63/0 Okapi, 63/16 Cow, 63/48 Panda, 63/64 Platinum, 63/80 White, 63/112 Org Panda, 63/192 Zebra,
  63/208 Cow_2, 63/240 Tiger. Live: Gulf 63/112 → Org Panda; Xi 63/48 → Panda.
- Normal coat tables (Bay/Black/Brown/Chestnut/Gray) match `getColorName` (lines 566-570) verbatim.
- Personality bands: the doc correctly flags the tool's `getPersonalityCode` as a LOSSY 5-letter
  collapse. I reproduced the tool function; it does NOT preserve the 8 bands (e.g. p=128 → 'C',
  p=180 → 'I'). The doc's "store raw 0-255" recommendation is correct. The specific 8-band cut
  points the doc lists are ROM-derived (not in the card tool); I could not independently confirm the
  exact band boundaries from card bytes alone — flagged as could-not-check, not wrong.

### Track 2 career/status (sec 4) — CONFIRMED 1.0 (one table typo)
Offset formula `off = 69 + (69-k)` verified for every listed a2 field:
- a2[13] sc2 0x7D, a2[14] sc1 0x7C, a2[15] silkpat 0x7B, a2[16] sex 0x7A — OK
- a2[23/24/25] retire-internals → 0x73/0x72/0x71 (Scarecrow 45/45/45) — OK
- **a2[26] hood → 0x70, NOT 0x73.** The doc's sec-4 table cell lists hood at "0x73", which is
  actually a2[23] (retire SHARP). Real byte: 0x70 = 21 (Scarecrow hood). 0x73 = 45 (ret SHARP).
  This is a copy error in one table cell; the doc's own parenthetical at lines 139-142 gives the
  right formula, so the body self-corrects.
- a2[27] @0x6F: =44 on Gulf/Phil/Xi (the "Trump-stable" trio), 0 elsewhere — CONFIRMED the pattern.
- a2[35] races 0x67, a2[37] hearts 0x65 (Scarecrow 55→14, formula (v+1)/4 → floor 14) — OK.
- a2[34] == a2[49] ("wins duplicate"): holds on ALL cards (Scarecrow 5=5, Xi 20=20, Caitin 0=0).
- Externals order Spurt..Start confirmed (Start highest index): cur a2[43..38], ret a2[33..28],
  display = card+1. OK.
- Earnings: dollars = (a2[51]·65536 + a2[52]·256 + a2[53])·1000. Scarecrow 0/16/226 = 4322 →
  $4,322,000. EXACT.
- Internals cur sharp/spd/sta a2[61/65/69] @0x4D/0x49/0x45 — OK (Xi all 60, capped).

### G1 bitfield (sec 4) — CONFIRMED 1.0
Reproduced `G1_RACES` (lines 455-474) and decoded live:
- Scarecrow a2[56]=128 → Japan Cup (exactly 1 title). Matches doc.
- Xi b55=29 b56=243 b57=77 → 14 titles, all decoding to valid race names. Bit map exact.
- Gulf b55/56/57=0 → no titles. OK.

### Track 3 (sec 5) — CONFIRMED 1.0
- a3[50]=0x10 @0x9D, a3[51]=0x30 @0x9C (the `30 10` pair) present on every US card.
- a3[53] breeds @0x9A, a3[57] retired @0x96 (Xi=1, only retired card found), a3[61] dirt @0x92
  (Gulf=255, Caitin=107). All OK. SEGABEF0 occupies a3[62..69] = 0x91..0x8A; dirt at 0x92 is the
  "char after the signature" — confirmed exactly as the doc warns.

### UID (sec 6) — CONFIRMED 1.0
a1[2..5]==a2[2..5]==a3[2..5] at file offsets 0x40-0x43 / 0x85-0x88 / 0xCA-0xCD on EVERY card.
Note: the doc's "Scarecrow = 21 9E 1C 69" is the logical a1[2..5] order; raw file bytes 0x40-0x43
read `69 1C 9E 21` (reversed). Consistent, just two different orderings of the same 4 bytes.

### Checksum (sec 7) — CONFIRMED (source-level) 0.95
No whole-card checksum in the 207-byte payload (any byte editable + re-saved). The XOR/multiCode +
2-byte checksum pair lives in `encodeTrack`/`decodeTrack` (lines 525-545). Confirmed in source;
the .card form is post-decode and unprotected.

### Cross-version (sec 9) — CONFIRMED 1.0
- Container identical across versions (all 207 bytes).
- Rev C (WillyJR) and Rev D (BabyBoy) cards both `SEGABEF0`, same layout, same `30/10` markers —
  NOT distinguishable from card bytes. Confirmed.
- JP cards (derbyo2k, derbyoc) are identity-only: no SEGABEF0, tracks 2-3 effectively unwritten.

---

## Corrections / nits

1. **(table typo, low impact)** Sec 4 table lists **a2[26] hood at file offset 0x73**. Correct offset
   is **0x70**. 0x73 is a2[23] (retire SHARP). The doc's logical index (a2[26]) is right and its own
   formula note fixes it; only the hex cell is wrong.

2. **(over-generalization)** Sec 2/9 claim JP identity cards have **byte 0x20=0x03, 0x21=0x02**. This
   is verified ONLY for **DOC 2000 (derbyo2k)** — 16/16 cards show 3/2. **DOC '99 (derbyoc)** cards
   show **0x20=17, 0x21=6** (2/2 cards). So `0x03 0x02` is a DOC-2000-specific header, not a
   universal JP marker. US-vs-JP detection is unaffected (absence of SEGABEF0 is the real test), but
   the `jp-card` spec should record the per-version difference.

3. **(scope, could-not-check)** The exact 8-band personality cut points (0-47 Rough, 48-63 Imposing,
   etc.) are ROM-derived and not present in the card-tool source; not falsifiable from card bytes
   alone. The doc honestly marks these as ROM-derived. No correction, just noting the limit of
   card-byte verification.

4. **(garbled cell)** Sec 4 row for a2[23] prints "0x4B→0x?? (0x... see note)" — cosmetically broken
   in the table though the follow-up paragraph computes it correctly (0x73). Worth cleaning up.

## Bottom line
The us-card decode is trustworthy and reproducible from raw bytes. Every field a tool or rebuild
would depend on (offsets, formulas, bitfields, signature, UID, earnings, coat) verified exact. Fix
the hood-offset cell, scope the JP `03 02` header to DOC 2000, and tidy the a2[23] table cell.
