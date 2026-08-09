# Adversarial Verification — items-feeding.md

Verifier: independent re-extraction from the 4 real program ROMs + beer_effects_test.ic22 (Bash + python3, EUC-JP/latin1 decode). Date 2026-06-03.
Method: dumped the 44-byte food records at each claimed base, walked to the idx=0 terminator, diffed the beer ROM, checked text-label offsets, and tallied value ranges/flag domains. Binaries were never read into context — only decoded values were printed.

Verdict: **mostly-solid**. The structural core (table location, geometry, record layout, effect bytes, per-version differences, beer diff) reproduces EXACTLY from bytes. Two soft claims about UI label sourcing for effect columns 3-6 are weak/mis-sourced, but the doc already flags them at conf 0.55 and lists them as open questions, so they are not asserted as fact.

---

## CONFIRMED (reproduced byte-for-byte)

### §1 Table location & geometry — CONFIRMED all 4 versions
| Version | base (claimed=found) | recsz | foods | terminator (idx=0) |
|---|---|---|---|---|
| drbyocwc (RevC) | 0x166A7C | 44 | 45 | rec45 @ 0x167238 |
| derbyocw (RevD) | 0x16980C | 44 | 45 | rec45 @ 0x169FC8 |
| derbyo2k (2000) | 0x171F34 | 44 | 45 | rec45 @ 0x1726F0 |
| derbyoc (1999)  | 0x15C9EC | 44 | 41 | rec41 @ 0x15D0F8 |
- RevD relocation delta = 0x16980C - 0x166A7C = **0x2D90** (doc's figure exact), and RevD food data is byte-for-byte identical to RevC (same gp, same eff).
- derbyoc table is genuinely 44-byte records (not the 28-byte racing stride) — confirmed; ends after ビッグプリン (LARGE PUDDING, idx 39), NO bananas / NO beer. Doc's correction of BEER-FINDINGS stands.

### §2 Record layout — CONFIRMED
- +0..23 name (ASCII on EN, EUC-JP on JP — JP names decode cleanly: ニンジン … 生中/黒生中).
- +24 u32 graphic ptr (0x0D8xxxxx typical; PUDDING/BANANA point to other banks: RevC 0x0D5C5610 / 0x0D801960; 2K 0x0D5D6C70 / 0x0D801960; 99 0x0D5BCB20 / no banana).
- +28..34 = 7 effect cols; +35 class flag; +36 size flag; +37, +38..39 reserved; +40 u32 index.
- Reserved fields verified across all 45 RevC records: **+37 always 0x00; +38..39 always 0x0000.**
- Index field domain: real foods 1..39; trailing CHEESE/PUDDING/LARGE PUDDING/BANANA/LARGE BANANA all idx=39; both BEERs idx=1; terminator idx=0. Matches the "index is a trap" claim exactly.

### §3 Effect columns / value ranges — CONFIRMED (numeric), labels partial
- Per-column max across the 45 RevC foods = **[5, 6, 7, 4, 4, 5, 1]** for cols 0..6. Confirms "0..7, col2 peaks at 7 (? MUSHROOM), col6 only 0/1." 
- Single-stat anchor foods reproduce: CARROT 0200000000000001, FODDER 0002000000000001, WATERMELON 0000020000000001, APPLE 0000000000020001, KOREAN GINSENG 0202020202020001, LARGE KOREAN GINSENG 0404040404040001 — all exact.
- On-screen labels block @ **0x12874C** confirmed: bytes are `Speed\x00\x01\x02Stamina\x00Sharp\x00\xfd\xfdFriendship\x00\xfeMost favorite food...`. So the visible feed-screen stat labels are **Speed / Stamina / Sharp / Friendship** (4 labels), immediately followed by the food flavor-text strings. Anchors cols 0/1/2 = Speed/Stamina/Sharp solidly. **NEW**: the 4th visible label is "Friendship," which is a better candidate for col 3 than the doc's "Spirit/Guts" guess.

### §4 Class flag +35 — CONFIRMED
- Domain {0,1}. The exactly-7 records with +35==0: MUSHROOM, LARGE MUSHROOM, WHITE MUSHROOM, ? MUSHROOM, LARGE ? MUSHROOM, BANANA, LARGE BANANA. Matches §4 list verbatim.

### §5 Full catalog (RevC) — CONFIRMED
Every one of the 45 rows (name, eff[+28..35], idx) reproduces. +36 size flag spot-checked: CARROT/APPLE=0 (small), BUNCH/LARGE CARROT/LARGE APPLE/CHEESE=1 (large), MUSHROOM=0 — matches §5 small/large labels.

### §6 Per-version tuning deltas — CONFIRMED exact
- CUBE SUGAR: derbyoc `0000040400040001` -> 2K/WE `0000060400050001`. ✔
- GREEN SALAD: derbyoc `0100000002030001` -> 2K/WE `0100000002050001`. ✔
- LARGE ? MUSHROOM: derbyoc `0400040400000000` -> WE/2K `0300030400000000`. ✔
- 2K vs WE effect data otherwise identical; only graphic pointers differ (different asset banks) — confirmed.

### §7 Beer diff — CONFIRMED exact
beer_effects_test.ic22 vs epr-22336c.ic22: **exactly 12 bytes, 2 runs**:
- 0x1671FC-0x167201: 000000000000 -> 020202020202 (DRAFT, rec@0x1671E0 +28..33)
- 0x167228-0x16722D: 000000000000 -> 040404040404 (BLACK, rec@0x16720C +28..33)
+34 (col6) and +35 (flag) untouched; index left at 1. All as documented. No authentic beer effects exist in any ROM (all ship 0000000000000001).

---

## CORRECTIONS / WEAK SPOTS

1. **"Power" label @ 0x10BDFA is NOT a stat label — it is a horse name.** Bytes there are `Power Drift\x00...Bone Dragon\x00...Triple Drago...` (a horse-name table). The doc cites it as evidence for a col 3-6 stat name; that sourcing is wrong. (Doc rated cols 3-6 at conf 0.55 and as open Q1, so it is not asserted as fact — downgrade, not a hard error.)

2. **"Spirit" label @ 0x110274 is in an award/menu list, not a feed-stat label.** Context: `Spirit\x00\x0fColor\x00...Runner\x00Ace\x00Champ\x00...Memories` — looks like a title/award list. Not evidence that col 3 = "Spirit." The stronger, byte-adjacent candidate for the 4th feed stat is **Friendship** (from the 0x12874C block).

3. Minor: the other "Sharp" hit at 0x11005C is inside a temperament word list (Charmy/Lucky/Hot/Cool/Sharp/Friendly/Funky...), unrelated to feeding — worth noting so a future editor does not bind feed columns to that block.

Net: cols 0/1/2 = Speed/Stamina/Sharp are solid (anchored at 0x12874C). Col 3 is most likely **Friendship**. Cols 4-6 remain unbound and the doc's Spirit/Power guesses should be treated as unsupported.

---

## EXTENSIONS (new, verified)

- **4 on-screen feed labels, not 7.** The UI shows Speed/Stamina/Sharp/Friendship; the record carries 7 effect columns. So 3 of the 7 columns are not surfaced as named bars on the feed screen (hidden growth stats). This reframes open Q1: only col 3 likely maps to a visible label (Friendship); cols 4-6 are internal.
- **derbyo2k PUDDING/banana banks differ from RevC**: 2K uses 0x0D5D6C70 for puddings (RevC 0x0D5C5610), banana 0x0D801960 (same). derbyoc puddings 0x0D5BCB20. Confirms per-version asset-bank divergence while structure/effects hold.
- **Terminator is a full 44-byte all-zero record in every version** (not just a zero index). Verified RevC/RevD/2K @ their term offsets; the food-list walker can safely use idx==0 as the stop condition.
