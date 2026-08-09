# Items / Feeding / Beer Effects — Core RE

KEY: items-feeding
Scope: the food/feeding table that the stable feed menu reads to apply stat boosts to a horse, plus the disabled "beer" placeholders. Covers all four versions.

All offsets verified by extracting bytes from the real ROMs (Bash + python3, EUC-JP / latin1 decode). Confidence noted per claim.

---

## 1. The Food Table — location & geometry (VERIFIED, conf 0.98)

The feed menu is driven by a single packed array of fixed-size food records, terminated by an all-zero record whose trailing index = 0.

| Version (set) | ROM file | Table start | Record size | Food count | Terminator |
|---|---|---|---|---|---|
| WE Rev C (drbyocwc) | epr-22336c.ic22 | **0x166A7C** | 44 | **45** | rec 45 @ 0x167238 (idx=0) |
| WE EX Rev D (derbyocw) | epr-22336d.ic22 | **0x16980C** | 44 | **45** | rec 45 @ 0x169FC8 (idx=0) |
| DOC 2000 JP (derbyo2k) | epr-22284a.ic22 | **0x171F34** | 44 | **45** | rec 45 (idx=0) |
| DOC '99 JP (derbyoc) | epr-22099b.ic22 | **0x15C9EC** | 44 | **41** | rec 41 (idx=0) |

How located: ASCII `CARROT\0` for the EN sets; for JP sets, scanned 0x100000–0x1C0000 for two consecutive 44-byte records whose offset+24 u32 has high byte 0x0d (a graphic RAM pointer) and offset+40 u32 = 1 then 2 (the running index). Single unambiguous hit in each JP ROM.

NOTE the table is 44-byte records in ALL FOUR versions (including derbyoc, even though derbyoc's *racing* stat table uses a 28-byte stride — the food table is unaffected by that).

---

## 2. Record layout (44 bytes) (VERIFIED, conf 0.95)

Offsets relative to record start:

| Off | Width | Field | Notes |
|---|---|---|---|
| `+0` | 24 | **Name** | ASCII null-padded on EN; **EUC-JP** null-padded on JP |
| `+24` | u32 LE | **Graphic pointer** | RAM addr, almost always `0x0D8xxxxx` (texture/sprite). PUDDING/BANANA point to other banks (0x0D5C5610, 0x0D801960). |
| `+28` | 7 bytes | **Stat-effect deltas** (cols 0–6) | the actual boosts applied to the horse (see §3) |
| `+35` | 1 byte | **Effect-class flag** (a.k.a. byte+7) | 0x01 = normal "feed/condition" food; 0x00 = the growth/no-recovery class (all mushrooms + bananas). conf 0.8 on meaning |
| `+36` | 1 byte | **Rarity/size flag** (fl0) | 0x00 for the common small base foods, 0x01 for large/special/rare variants. conf 0.85 |
| `+37` | 1 byte | reserved (fl1) | always 0x00 across all 45 records |
| `+38` | 2 bytes | reserved (mid) | always 0x0000 |
| `+40` | u32 LE | **Food index / ID** | 1..39 for the real catalog; the last few "duplicate" foods reuse idx 39; terminator = 0 |

Reconciliation with the racing-table convention (DOC-ROM-Studio uses recBase=recordStart+9): that convention is for the *racing stat* records, NOT the food table. The food table uses simple record-start-relative offsets as above; do not apply the +9 recBase shift here.

### The index field is a trap (VERIFIED behaviorally via BEER-FINDINGS, conf 0.9)
`+40` is NOT a unique row id. Real foods run 1..39; the trailing "extra" foods (CHEESE, PUDDING, LARGE PUDDING, BANANA, LARGE BANANA, and both BEERs) all carry **idx=39** or **idx=1**. The game builds a ~39-entry lookup array at init keyed by this index; writing an out-of-range index (44/45) to beer crashes at boot. Index must stay within the existing 1..39 range.

---

## 3. Effect bytes [+28..+34] = 7 stat-delta columns (VERIFIED via single-stat foods, conf 0.9)

Each of bytes +28..+34 is an unsigned per-stat boost magnitude. Mapping derived from foods that move exactly one column, confirmed by the small→large pairs doubling the value (e.g. APPLE +2 → LARGE APPLE +4):

| Col | Rec off | Pure single-stat foods (value) | Inferred stat |
|---|---|---|---|
| 0 | +28 | CARROT(2), BUNCH OF CARROTS(5) | **Speed** (carrots = #1 favorite, headline stat) |
| 1 | +29 | FODDER(2), FODDER w/ GREEN TEA(5), WHITE MUSHROOM(5) | **Stamina** (fodder = "good shape / nutritious") |
| 2 | +30 | WATERMELON(2/4), ? MUSHROOM(7) | **Sharp(ness)** — matches the 3 internals Speed/Stamina/Sharp |
| 3 | +31 | JAPANESE RADISH(2/4) | 4th growth stat (Spirit / Guts) |
| 4 | +32 | CABBAGE(2/4) | 5th growth stat (Power / Temperament) |
| 5 | +33 | APPLE(2/4) | 6th growth stat |
| 6 | +34 | (none pure; only in BANANA/LARGE BANANA + LARGE BANANA-class) | 7th stat (condition/mood) |

The header text block "Post-Food Comments" (0x12874C in drbyocwc) literally lists **Speed / Stamina / Sharp / Friendship** as the on-screen stat labels, anchoring cols 0/1/2 = Speed/Stamina/Sharp. Cols 3–6 are the secondary growth stats whose exact UI names (Spirit, Power, etc., labels found at 0x110274 "Spirit", 0x10BDFA "Power") still need an in-game confirmation. conf on cols 0-2 = 0.85; cols 3-6 = 0.55.

Value range per column: 0..7 (col2 max 7 on ? MUSHROOM; most caps at ~5). The "large" variant of a food roughly doubles its base deltas.

KOREAN GINSENG = `0202020202020001` (+2 to all six of cols 0–5), LARGE KOREAN GINSENG = `0404040404040001` (+4 all). These are the only "boost everything" foods — and are exactly the template the beer experiment copied.

---

## 4. Effect-class flag +35 (byte+7) (conf 0.8)

Value 0x00 only on: MUSHROOM, LARGE MUSHROOM, WHITE MUSHROOM, ? MUSHROOM, LARGE ? MUSHROOM, BANANA, LARGE BANANA. All other foods = 0x01.

Interpretation: 0x01 = ordinary feed that also restores fatigue/condition ("good to feed when it feels tired" flavor text on bananas — note banana is 0x00, so the relationship is more nuanced); 0x00 marks the special **growth-only** class (the mushrooms "help your horse grow" / "may be effective"). Treated as a behavior class selector by the feed routine. Confirm by toggling in-game.

---

## 5. Full catalog (drbyocwc / WE Rev C — identical data in Rev D) (VERIFIED)

idx, name, effect hex [+28..+35], size-flag:

```
 1 CARROT                 0200000000000001  small   (Speed+2)
 2 BUNCH OF CARROTS       0500000000000001  large   (Speed+5)
 3 LARGE CARROT           0300000001000001  large   (Speed+3, col5+1)
 4 FODDER                 0002000000000001  small   (Stamina+2)
 5 FODDER WITH GREEN TEA  0005000000000001  large   (Stamina+5)
 6 FODDER WITH GARLIC     0003000001000001  large   (Stamina+3, col5+1)
 7 APPLE                  0000000000020001  small   (col5+2)
 8 LARGE APPLE            0000000000040001  large   (col5+4)
 9 WATERMELON             0000020000000001  small   (Sharp+2)
10 LARGE WATERMELON       0000040000000001  large   (Sharp+4)
11 JAPANESE RADISH        0000000200000001  small   (col3+2)
12 LARGE JAPANESE RADISH  0000000400000001  large   (col3+4)
13 CABBAGE                0000000002000001  small   (col4+2)
14 LARGE CABBAGE          0000000004000001  large   (col4+4)
15 CUBE SUGAR             0000060400050001  large   (Sharp+6,col3+4,col5+5)
16 GREEN SALAD            0100000002050001  large   (Speed+1,col4+2,col5+5)
17 HERBAL DUMPLING        0101030102010001  large   (multi-stat)
18 SUPER HERBAL DUMPLING  0101050103010001  large   (multi-stat, stronger)
19 PINEAPPLE              0000010001000001  small
20 LARGE PINEAPPLE        0000020002000001  large
21 ORANGE                 0001000100000001  small
22 LARGE ORANGE           0002000200000001  large
23 STRAWBERRY             0100000000010001  small
24 LARGE STRAWBERRY       0200000000020001  large
25 CAMEMBERT CHEESE       0000020100000001  small
26 BLUE CHEESE            0000040400000001  large
27 MUSHROOM               0100010001000000  growth (flag00)
28 LARGE MUSHROOM         0200020002000000  growth
29 WHITE MUSHROOM         0005000000000000  growth (Stamina+5, no recovery)
30 ? MUSHROOM             0000070000000000  growth (Sharp+7!)
31 LARGE ? MUSHROOM       0300030400000000  growth
32 CORN                   0000010000020001  small
33 GREEN APPLE            0002000001000001  small
34 LARGE GREEN APPLE      0004000002000001  large
35 TURNIP                 0001020000000001  small
36 LARGE TURNIP           0002030000000001  large
37 KOREAN GINSENG         0202020202020001  large (+2 all 6 stats)
38 LARGE KOREAN GINSENG   0404040404040001  large (+4 all 6 stats)
39 CHEESE                 0200000002000001  large
(39) PUDDING              0203000002000001  large
(39) LARGE PUDDING        0406000004000101  large (note col6=01)
(39) BANANA               0000010100010000  growth
(39) LARGE BANANA         0101010101010100  growth (col6=01)
( 1) DRAFT BEER           0000000000000001  disabled placeholder, all-zero effect
( 1) BLACK DRAFT BEER     0000000000000001  disabled placeholder, all-zero effect
```

JP names (derbyo2k, EUC-JP, same order/effects): ニンジン, ニンジンの束, 大きなニンジン, 飼い葉, 杜仲茶入り飼い葉, ガーリック入り飼い葉, リンゴ, ビッグリンゴ, スイカ, ビッグスイカ, 大根, ビッグ大根, キャベツ, 大きなキャベツ, 角砂糖, 野菜サラダ, まむし団子, スーパーまむし団子, パイナップル, ビッグパイナップル, ミカン, ビッグミカン, イチゴ, ビッグイチゴ, カマンベールチーズ, ブルーチーズ, キノコ, ビッグキノコ, 白いキノコ, ？なキノコ, ビッグ？キノコ, トウモロコシ, 青リンゴ, ビッグ青リンゴ, カブ, 大きなカブ, 朝鮮ニンジン, ビッグ朝鮮ニンジン, チーズ, プリン, ビッグプリン, バナナ, ビッグバナナ, **生中 (DRAFT BEER), 黒生中 (BLACK DRAFT BEER)**.

---

## 6. Per-version differences (VERIFIED)

- **derbyoc (DOC '99) has only 41 foods** — table ends after プリン/ビッグプリン (idx 39). **No bananas, no beer at all.** This CORRECTS the BEER-FINDINGS phrasing: beer (and banana) were ADDED in DOC 2000; the '99 original never had them.
- **derbyo2k (DOC 2000)** is the first version with all 45 (adds CHEESE-onward block + bananas + the two beers). Beers ship with all-zero effects (disabled placeholder) — confirmed in bytes: 生中/黒生中 eff = `0000000000000001`.
- **WE Rev C & Rev D** carry the identical 45-food table forward; Rev D is byte-for-byte identical food data, just relocated +0x2D90 (0x166A7C → 0x16980C). Same graphic pointers, same effects.
- **Effect tuning between '99 and 2K/WE:** a few records were re-balanced. CUBE SUGAR: derbyoc `0000040400040001` → 2K/WE `0000060400050001` (Sharp 4→6, col5 4→5). GREEN SALAD: derbyoc `0100000002030001` → 2K/WE `0100000002050001` (col5 3→5). LARGE ? MUSHROOM differs too (derbyoc `0400040400000000` vs WE `0300030400000000`). So the food table is a genuine per-version tuning surface, not frozen.
- Graphic pointers differ between '99 and 2K (different asset banks), but effects/structure are stable.

---

## 7. The beer experiment — exactly what changed (VERIFIED byte-diff)

`beer_effects_test.ic22` vs base `epr-22336c.ic22`: **exactly 12 bytes differ, in 2 runs**, nothing else:

```
0x1671FC–0x167201  000000000000 -> 020202020202   (DRAFT BEER, rec 0x1671E0 + 28..33)
0x167228–0x16722D  000000000000 -> 040404040404   (BLACK DRAFT BEER, rec 0x16720C + 28..33)
```

So the editor wrote +2 to all of DRAFT's first six stat columns and +4 to BLACK's — copying the KOREAN GINSENG / LARGE KOREAN GINSENG pattern verbatim. It deliberately left record offset +34 (col6) and +35 (the 0x01 effect-class flag) untouched, and left the index at 1 (colliding with CARROT). Result per BEER-FINDINGS: boots fine, but whether beer surfaces in the menu was never confirmed because index 1 collides with CARROT rather than getting its own slot. The crash seen with index 44/45 is the §2 index-array trap, not the effect bytes.

There are NO authentic beer effect values anywhere in any ROM — every version that has beer ships it with a null effect. Any beer effect is invented.

---

## 8. Open questions

1. Exact UI names for effect columns 3–6 (radish/cabbage/apple/banana stats). Labels "Spirit" (0x110274) and "Power" (0x10BDFA) exist; need an in-game feed-screen capture to bind columns→labels. Cols 0/1/2 = Speed/Stamina/Sharp are solid.
2. Meaning of effect-class flag +35 (0x00 growth vs 0x01 feed) and how it interacts with col6 — confirm by feeding a flag-toggled food in DEMUL/Flycast and watching condition vs growth bars.
3. The food-list builder routine (SH4) — does the menu iterate records to the idx=0 terminator, use a hard count constant, or a separate food-ID list? Needed to surface beer with a clean unique index. Disassembly target: code that reads 0x166A7C and the ~39-entry index array.
4. Does feeding modify the **on-card** stored stats or only the cabinet/nvram working horse? (Ties into the unresolved JP on-card stats question.)
5. Whether `+36` rarity flag affects shop availability/price or is purely cosmetic (large-icon vs small).

---

## 9. Tool ideas this unlocks

- **Food/Feed Table editor in DOC-ROM-Studio**: add a FOOD block (offsets in §1) that decodes all 44-byte records — name (ASCII/EUC-JP per version), 7 stat deltas as labeled sliders, class flag, rarity flag, index — with the index-range guard (1..39) baked in so users can't reproduce the boot crash. Needs: §1/§2/§3 (done).
- **"Enable Beer" one-click patch**: writes a chosen effect to 生中/黒生中 and assigns a safe in-range index; warns it shares a slot. Needs: resolution of open Q3 (menu builder) to do it cleanly.
- **Cross-version food diff report**: auto-emit the §6 tuning deltas for any two versions (already scripted; promote into the suite). Needs: §1 offsets per version (done).
- **Feed simulator**: given a horse's current 7 stats and a chosen food, predict the post-feed stats (delta-add, with the class flag governing condition vs growth). Needs: open Q2 confirmation of how flag/col6 apply.
