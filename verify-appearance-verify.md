# Adversarial verification — `appearance.md` (Coat colors, silks, hood, sex)

Verifier pass: 2026-06-03. Method: independently re-extracted every offset/format claim from the
real ROM bytes and real `.card` files with python3 (Windows paths), cross-checked against the
per-version horse DBs and the two encoder/decoder tools (`DOC-Card-Creator.html`,
`DOC-ROM-Studio.html`). Did **not** read any binary into context — only printed decoded values.

**Overall verdict: mostly-solid.** The substance (which bytes mean what, the coat enum, the string
tables, the card field set, JP cards carry no appearance) is correct and reproducible. But the doc
has **two concrete offset errors** that would mislead anyone using it as a spec:

1. **US/WE CPU coat byte is at record offset `+13`, NOT `+22`.** (`+22` is the `sharp` stat.)
2. **Card hood byte is at file offset `0x70`, NOT `0x73`.** (logical index a2[26] and value are right.)

Everything else verified.

---

## Scripts (reproducible, in `C:/Users/johnr/AppData/Local/Temp/`)
- `v_strtab.py` — ROM coat/sex string tables @0x0C68F0 & 0x0EE0AC
- `v_cpu2.py` / `v_cpu3.py` — US coat byte: distribution + 244/244 per-horse vs DB at +13 (and +22 fails)
- `v_cpu4.py` — coat offset across all 4 versions
- `v_jp.py` / `v_jp2.py` — JP coat offset scan + 244/244 per-horse vs JP DBs
- `v_card2.py` — card appearance fields from 6 real US cards (correct 0-based-track offset formula)
- `v_hood.py` — proves hood is 0x70 not 0x73
- `v_jpcard.py` — JP card header + absence of US appearance block

---

## CLAIM-BY-CLAIM

### ✅ ROM coat/sex string tables (§2d, §5) — CONFIRMED byte-exact
Both tables exist exactly as claimed in drbyocwc (epr-22336c):
- **0x0C68F0 (simple):** `GRAY · CHESTNUT · BLACK · BAY · BROWN · SPECIAL · WHITE` then `MALE · FEMALE · GELDING` (MALE/FEMALE/GELDING start at 0x0C6928, doc said "0x0C6926+"; the `0F`/`FF 0F` UI color-control bytes precede each string, so ±2 is just where you anchor — substantively correct).
- **0x0EE0AC (granular):** `GRAY · DARK CHESTNUT BAY · CHESTNUT BAY · BLACK · BAY · DARK BAY · DARK · SPECIAL · WHITE` then `MALE · FEMALE · GELDING`. Matches.
- Interleaved control codes are `00 0F FF 0F` between entries — confirmed in hex dump.

### ❌ CPU-record coat OFFSET (§2c, §7) — WRONG OFFSET, right enum & distribution
Doc says US coat is at record **+22** (and record base 0x108E03). **Both wrong for US/WE.**
The authoritative tool `DOC-ROM-Studio.html` (line 169/173) says `RACING_F.coat = 13`,
`recBase = 0x108E0C`, `recStride = 32`. Verified against bytes:

| offset tried | base/stride | in-enum | result |
|---|---|---|---|
| **+13** | 0x108E0C / 32 | **244/244** | dist 207×107,204×54,202×39,222×21,199×15,192×4,193×3,0×1 — exact DB match; **244/244 per-horse** vs `DOC_COMPLETE_HORSE_DATABASE_DRBYOCWC.md` |
| +22 (doc) | 0x108E0C / 32 | **1/244** | values 31–56 = the `sharp` stat, not a coat |

So the **enum map is correct** {0:Default,192:Chestnut,193:Black,199:Brown,202:Bay,204:Dark Gray,
207:Light Gray,222:Special} and the **distribution numbers in the doc are correct**, but the doc's
stated US record offset (+22) and base (0x108E03) are not. The likely cause: the doc copied the
**JP** record's coat offset (+22, which IS correct for derbyo2k — see below) and mis-applied it to
the US record, whose field order differs (US uses RACING_F: dirt:-4, grade:-1, start:0…sharp:22,
coat:13). derbyocw (Rev D) confirmed identically: coat at +13, base 0x10A154, 244/244.

### ✅ JP coat offsets (§7) — CONFIRMED (the doc is RIGHT here)
- **derbyo2k:** coat at **+22**, base 0x10AD1B, stride 32 → 207×107,204×54,202×40,222×21,199×14,192×4,193×3,0×1; **244/244 per-horse** vs DERBYO2K DB. Matches doc's "199×14,202×40".
- **derbyoc ('99):** coat at **+19**, base 0x0F6902, stride **28** → 207×103,204×49,202×38,222×26,199×15,192×6,193×6,0×1; **244/244 per-horse** vs DERBYOC DB. Matches doc's exact numbers.
- Note: the JP field-map in DOC-ROM-Studio (`JPSPEC`) does **not** list a coat field — so +22/+19 were the doc's own contribution, and they hold up against the bytes. Good catch by the original author; the only mistake was reusing +22 for US.

### ⚠️ Card appearance file offsets (§1) — VALUES & most offsets right; ONE offset wrong (hood)
The tool reads `arr[k] = bytes[t*69 + (69-k)]` with **t = 0-based track** (a1↔t=0). So the doc's
own prose formula "`aT[k]` = `T*69+(69-k)`" only yields the right offsets if T is 0-based (a1→0),
which contradicts the array names but matches the listed hex. Extracting the 6 real US cards:

| field | logical | doc off | verified off | notes |
|---|---|---|---|---|
| Coat base | a1[8] | 0x3D | **0x3D ✅** | |
| Coat modifier | a1[9] | 0x3C | **0x3C ✅** | |
| Silk color 2 | a2[13] | 0x7D | **0x7D ✅** | |
| Silk color 1 | a2[14] | 0x7C | **0x7C ✅** | |
| Silk pattern | a2[15] | 0x7B | **0x7B ✅** | |
| Sex | a2[16] | 0x7A | **0x7A ✅** | |
| **Hood** | a2[26] | **0x73** | **0x70 ❌** | a2[26] = 1·69+(69−26)=112=0x70. Byte 0x73 holds an unrelated 45. Logical index & value are correct; only the hex is wrong. |

Verified-card dump (independent re-extraction, matches the doc's §1 sample exactly except it
confirms hood lives at 0x70):
```
Caitin_Clark    cb=77  cm=0   pat7 sc1=LightBlue sc2=White  sexF       hood0   -> Bay
Scarecrow_II    cb=129 cm=0   pat0 sc1=Yellow    sc2=White  sexM       hood21  -> Black
Gulf_of_America cb=63  cm=112 pat1 sc1=Red       sc2=Yellow sexF       hood0   -> Org Panda
Xi_Jinping      cb=63  cm=48  pat0 sc1=Black     sc2=White  sexM       hood0   -> Panda
DD              cb=196 cm=0   pat0 sc1=Black     sc2=White  sexM       hood0   -> Chestnut
Phil_Jackson_2  cb=63  cm=112 pat1 sc1=Red       sc2=Yellow sexGelding hood0   -> Org Panda
```
All silk/sex/coat values match the doc's verified dump. (Doc listed BabyBoy/WillyJR which live in
`Tools/Cards/`; the six in `Card-Library/` corroborate the same offsets.)

### ✅ Special (starred) sub-id table (§2a) — CONFIRMED vs tool + live cards
`COLOR_OPTIONS` in DOC-Card-Creator.html: Okapi=0, Cow=16, Panda=48, Platinum=64, White=80,
Org Panda=112, Zebra=192, Cow_2=208, Tiger=240 (all with a1[8]=63). Matches doc. Live-confirmed:
Gulf/Phil cm=112→Org Panda, Xi cm=48→Panda.

### ✅ getColorName mirror bands & §2e discrepancy — CONFIRMED
`getColorName` lists verified: Bay {77,78,79,141,142,143,205,206,207}, Black {65,66,67,129…195},
Brown {69,70,71,73,74,75,133-139,197-203 incl 202}, Chestnut {64,68,72,76,128,132,136,140,192,196,200,204}.
So card heuristic says 202→Brown, 204→Chestnut, 207→Bay, while the ROM CPU enum says 202→Bay,
204→Dark Gray, 207→Light Gray. **The §2e discrepancy is real and correctly documented.** ROM enum
is authoritative (244/244 DB-backed). The proposed `getColorName` fix is sound.

### ✅ Silks (§3), Hood range (§4), Sex (§5) — CONFIRMED (positions); names observational
- Silk pattern a2[15] 0–7, sc1 a2[14], sc2 a2[13]; byte order (sc2 at lower-ID logical, sc1 written
  to 0x7C) matches `populateForm` (sc1←a2[14] @0x7C, sc2←a2[13] @0x7D). Verified.
- SILK_COLORS 15-entry list matches tool; not ASCII in ROM — name mapping observational (~0.8), as doc states.
- Hood 0–63, no ROM string table — observed 21/24/7 in real cards; range claim sound. (offset fix: 0x70.)
- Sex a2[16] 0=M,1=F,2=Gelding; confirmed Caitin=1(F), Phil=2(Gelding), DD/Xi=0(M); ROM MALE/FEMALE/
  GELDING strings adjacent to coat strings at both tables. Verified.

### ✅ JP cards carry no appearance (§6) — CONFIRMED
4 derbyo2k/derbyoc `.card` files: len 207, header **0x20=0x03, 0x21=0x02**, **no SEGA marker**
(US marker is precisely `SEGABEF0` @0x8A per `detectCardKind`; doc's looser "SEGA marker" is fine).
US appearance offsets on a JP card hold name bytes / zeros (a1[8]@0x3D = 35/4/83/38 = kana name
chars; sex/hood = 0). So no US-format appearance block on JP cards. Verified.

### §8 open items — reasonable
The "no CPU special-variant selector byte" conclusion wasn't independently re-derived here, but is
consistent: the CPU record is 32 bytes (28 on '99) with coat as a single enum byte and no spare
selector field that ranges over 9 sub-ids. Left as the doc states (~0.8).

---

## CORRECTIONS THE DOC NEEDS
1. **§2c/§7:** US/WE CPU coat byte is at record **+13** (base **0x108E0C** WE-C / **0x10A154** WE-D,
   stride 32), NOT +22/0x108E03. (+22 = the `sharp` stat.) JP offsets (+22 o2k, +19 oc) are correct.
2. **§1/§7:** Card **hood** is at file offset **0x70** (a2[26] = 1·69+(69−26)=112), NOT 0x73.
3. **§1 formula clarity:** `aT[k]` file offset uses a **0-based** track index (a1→track 0), i.e.
   `(T−1)·69 + (69−k)` if T is 1-based, or `t·69+(69−k)` with t=0,1,2. The listed hex offsets are all
   correct under that reading (except hood). Worth stating explicitly to avoid the +69 trap.
