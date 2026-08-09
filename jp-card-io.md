# JP DOC 2000 card I/O routine + career-block byte map (from the JP program ROM)

Static SH-4 RE of `derbyo2k/epr-22284a.ic22` (DOC 2000, product code BBX0, 4,194,304
bytes, CRC32 **1E8E067C**). Goal: recover the byte map of the 207-byte card's career
block (track-1 bytes 0x00-0x1F) directly from the game's own card serialize/deserialize
routine, finishing the decode that corpus inference could not fully pin.

Session 2026-07-25. Toolkit: docre.py (WE default; loaded against the JP ROM with
`docre.load(<jp path>, check=False)`). Validation corpus: the 17 real DOC 2000 cards in
`Tools/JP-Card-Capture/jp_career_corpus_merged.json`.

**Bottom line: FOUND and DECODED.** The JP card codec is at **0x0C076B00 (serializer,
record -> card) / 0x0C076E44 (deserializer, card -> record)**. Bytes 0x02-0x17 and 0x1F
of the career block are mapped field-by-field, confirmed bidirectionally in the ROM, and
validated against all 17 corpus cards. The JP horse struct is **homologous to the WE
record** (identical field offsets, identical codec, byte-identical bitfield helper).

---

## 0. Base-address correction (important)

Prior notes (jp-card.md, sp-horse-program.md) assumed the JP ROM loads at RAM base
**0x0C010000**. That is WRONG for code addressing. Two independent tests put it at
**0x0C020000** (same as the WE ROM, standard NAOMI cart base):

- String-pointer self-reference: number of ROM ASCII strings whose (file_offset + base)
  appears elsewhere as a literal LE dword = **504** at base 0x0C020000 vs **94** at
  0x0C010000.
- Mater-table data anchor still holds at FILE offset 0x11106C (EUC-JP `トロットサンダー`,
  `ホワイトノーザー`, `ビワハヤヒデ`), matching jp-card.md §8. Data offsets are base-
  independent; the base only matters for RAM/code addresses.

So docre's default convention (RAM = file + 0x0C020000) works unchanged for the JP ROM.
All addresses below are RAM.

---

## 1. The card I/O routines (all in the JP ROM)

| Routine | RAM addr | Role |
|---|---|---|
| **Serializer** (record -> card career block) | **0x0C076B00** | writes card bytes 0x00-0x17 + 0x1F from the horse record |
| **Deserializer** (card -> record) | **0x0C076E44** | inverse; body verified (e.g. `mov.b @(2,r13)->rec[68]`) |
| Orchestrator / card-write wrapper | 0x0C0765xx (calls serializer via `bsr 0xc076b00` @0x0C0765A2) | calls serializer, then the checksum, writes the trailer |
| Bitfield **INSERT** helper | **0x0C0C5970** | `(r0=value, r2=word ptr, r1=(HB<<8\|LB))`; byte-identical to WE 0x0C0C1F70 |
| Bitfield EXTRACT helper | 0x0C0C59A8 | inverse; byte-identical to WE 0x0C0C1FA8 |
| Field width/position pool | 0x0C076CB0 | the (HB,LB) constants per packed field |
| **Trailer checksum** fn | **0x0C0750EC** | CRC-16-CCITT over card[0x00..0x42] (see §5) |
| Leg-type/rank helper | 0x0C076A22 | ranks card[2]/[3]/[4]; confirms those three are the internals |

The serializer is a straight-line function: source register r13 = horse record, dest
r14 = card image (base = card byte 0). Byte-aligned fields are stored with `mov.b
r0,@(disp,r14)`; sub-byte fields are inserted into 32-bit card words at `card + {8,12,16,
20,28}` via the insert helper. This is the **same code** as the WE ROM's documented
"old-format (DOC2000, bit-packed) codec" (WE serializer 0x0C021EC8): identical record
field offsets, identical helper, identical width pool. The two builds share this codec.

### Bitfield helper semantics (both ROMs identical)
`extract(word, HB, LB) = (word << HB) >>_logical (32 - LB)`. A field of width **LB**
occupies bits **[31-HB : 32-HB-LB]** of the 32-bit word. The word is loaded
little-endian from the card bytes, so bit31 = MSB of the highest card byte in the word
(`card+disp+3`), bit0 = LSB of `card+disp+0`.

---

## 2. Career-block byte map 0x00-0x1F (CONFIRMED from the codec)

Record field names use the WE record layout (homologous, from
`findings/card-seed-trait-readers.md` §1.2). "rec[N]" = record byte offset N.

| card byte | source | field | confidence |
|---|---|---|---|
| 0x00 | rec[94] (serializer) | dirt/surface flag **but reframed** (see §4) | codec-CONFIRMED; physical = volatile |
| 0x01 | `((rec[114]&7)<<1)\|(rec[95]&1)` **but reframed** | physical = 0x70 format tag \| bit0 | codec-CONFIRMED; physical = const |
| **0x02** | rec[68] | **current internal SPEED** | CONFIRMED (corpus) |
| **0x03** | rec[64] | **current internal STAMINA** | CONFIRMED (corpus) |
| **0x04** | rec[72] | **current internal SHARP** | CONFIRMED (corpus) |
| **0x05** | rec[98] | **current external #1 (Start)** | CONFIRMED (resolves corpus 0x05-07 mystery) |
| **0x06** | rec[99] | **current external #2 (Corner)** | CONFIRMED |
| **0x07** | rec[100] | **current external #3** | CONFIRMED |
| 0x08 | rec[112]bits0-3, rec[113]bits4-7 (packed) | two retirement-external nibbles | codec-CONFIRMED |
| **0x09** | SHOW(rec122) bits0-5 \| PLACE(rec121) bits6-7 | **race record: 3rd-place & 2nd-place counts (packed)** | CONFIRMED (corpus) |
| **0x0A** | PLACE(rec121) hi \| WON(rec120) low nibble | **race record: 2nd & 1st (win) counts (packed)** | CONFIRMED (corpus) |
| 0x0B | WON(rec120) hi 2 bits \| hearts(rec104) bits2-7 | wins hi + **hearts (0-63)** | codec-CONFIRMED |
| 0x0C | rec[111]bits0-3, rec[110]bits4-7 | two retirement-external nibbles | codec-CONFIRMED |
| 0x0D | ext#6(rec103) \| ext#5(rec102) low | packed current externals 5,6 | codec-CONFIRMED |
| 0x0E | ext#5(rec102) hi \| ext#4(rec101) low | packed current external 4,5 | codec-CONFIRMED |
| 0x0F | ext#4(rec101) hi \| rec[130] (last-race datum, 6b) | packed | codec-CONFIRMED |
| 0x10 | (low byte of word, unwritten) | ~0 | codec |
| **0x11** | rec[117] | **BIRTH internal SPEED** | CONFIRMED (corpus) |
| **0x12** | rec[108] low nibble \| **(races-1)** high nibble (from OUT=rec123) | **race counter: races = (0x12>>4)+1** | CONFIRMED (corpus, live 2->3) |
| 0x13 | (races-1) overflow bits | high race-count bits | codec-CONFIRMED |
| **0x14** | rec[118] | **BIRTH internal SHARP** | CONFIRMED (corpus) |
| **0x15** | rec[116] | **BIRTH internal STAMINA** | CONFIRMED (corpus) |
| 0x16 | silkColor2(rec93) \| retExt2(rec109) bits | packed silk + retire-ext | codec-CONFIRMED |
| **0x17** | silkColor1(rec92) hi nibble \| **SEX(rec90) = bit3** \| retExt2 low bits | **bit3 = sex (0=M,1=F)** | CONFIRMED (corpus + deserializer `tst #8`) |
| 0x18-0x1E | NOT written by this codec | see §4 (0x18=0x0d const, 0x1A/0x1B/0x1C vary) | out of codec scope |
| **0x1F** | silk PATTERN(rec91) in bits 5-7 | **silk pattern (top 3 bits)** | CONFIRMED (corpus: all values are multiples of 0x20) |

### Exact packed layout of the win region (the task's headline target)
`word0x08 = card[0x08] | card[0x09]<<8 | card[0x0A]<<16 | card[0x0B]<<24` (LE), then:
- SHOW  (rec122) = `(word0x08 >> 8)  & 0x3F`  -> card[0x09] low 6 bits
- PLACE (rec121) = `(word0x08 >> 14) & 0x3F`  -> card[0x09] top 2 + card[0x0A] low 4
- WON   (rec120) = `(word0x08 >> 20) & 0x3F`  -> card[0x0A] top 4 + card[0x0B] low 2
- hearts(rec104) = `(word0x08 >> 26) & 0x3F`  -> card[0x0B] top 6

So the corpus's "0x09/0x0A grow on winners" observation is exactly the **win/place/show
counters**. OUT (rec123) lives in card[0x12] and equals **races-1** on every card, which
IS the confirmed race counter, not an "unplaced" count.

---

## 3. Corpus validation (all 17 real cards)

Running the decode above on `jp_career_corpus_merged.json`:

- Internal triplet 0x02/03/04 and birth triplet 0x11/15/14 in-range on all 17, with
  current >= birth (growth) as expected.
- **OUT (card[0x12] high nibble + carry) = races - 1 on 17/17** (races 1,2,2,2,2,2,2,2,
  2,3,3,3,3,3,4,10,15 -> OUT 0,1,1,1,1,1,1,1,1,2,2,2,2,2,3,9,14). This is the definitive
  ROM anchoring of races = (card[0x12]>>4)+1.
- WON <= races on 16/17; the only miss is タイムスリップ (15 races, WON+PLACE+SHOW=16),
  which is the known low-confidence read (9/13 constants, flagged in FUSION_RESULTS).
- キキーウイロー is the ONLY low-race card with WON>0 (WON=1, PLACE=1 over 3 races),
  exactly matching the corpus note that it was the sole "winner" (its 0x09=0x40,
  0x0A=0x10). Clean confirmation the win field is right.

Corrections this forces on the earlier corpus-only findings
(`CAREER_BLOCK_FINDINGS.md`):
- 0x02/03/04 stat ORDER is **Speed, Stamina, Sharp** (was guessed "stamina/speed/sharp").
- 0x05-0x07 = **current externals** (Start/Corner/#3), NOT a "second internal triplet".
- 0x09/0x0A = **win/place/show record**, NOT "hearts/coat".
- 0x1F top 3 bits = **silk pattern**, NOT the "retired bits 5+6" guess.
- 0x1B/0x1C (the other corpus win/earnings candidate) are **not** the win record and are
  **not written by this codec** (see §4); they are a separate field (earnings/lifetime).

---

## 4. What the codec does NOT write, and the 0x00/0x01 reframing

The career serializer writes card bytes 0x00-0x17 and 0x1F only. Bytes **0x18-0x1E** are
written by a different routine (not located): corpus has 0x18 = 0x0d constant, 0x19/0x1D/
0x1E ~ 0, and 0x1A/0x1B/0x1C varying with career (candidate lifetime earnings/fans). The
kana name/sire/dam region (0x28-0x42) and the 03 02 header (0x20-0x27) are likewise
written by separate identity/header code, not this career codec.

**0x00/0x01 anomaly:** the serializer writes card[0x00]=rec[94] and
card[0x01]=`((rec114&7)<<1)|(rec95&1)` (max 0x0F), yet the physical corpus shows 0x00
volatile and 0x01 = 0x70 (const, bit0 = collection marker). Since bytes 0x02-0x17
match the serializer exactly across 17 real cards, the card base is unquestionably byte 0;
the only consistent reading is that the **wrapper reframes bytes 0x00 and 0x01 after
serialization** (0x01 <- 0x70 format/version tag; 0x00 <- a per-write id/nonce), leaving
0x02+ intact. jp-card.md §4 already tags 0x00 volatile and 0x01 a header constant, which
this matches.

---

## 5. Trailer 0x43/0x44 = CRC-16-CCITT (resolves jp-card.md §7/§10 open Q2)

The orchestrator, right after the serializer, calls the checksum fn **0x0C0750EC** with
r4 = card buffer, r5 = **0x43 (67)** = length, then writes the 16-bit result to
**card[0x43] (low byte) and card[0x44] (high byte, via shlr8)**. The checksum is a
bit-serial CRC (inner processor 0x0C0750C0): test MSB (mask 0x8000 @0x0C0750E4), shift
left, and XOR polynomial **0x1021** (@0x0C0750E8) when the MSB was set = **CRC-16-CCITT**.
So the trailer is `CRC16-CCITT(card[0x00..0x42])`, stored little-endian at 0x43/0x44. This
is the missing piece for a clean CREATE recipe (jp-card.md §10 Q2): recompute this CRC
over bytes 0x00-0x42 after any edit. (Init/final-xor details: seed loaded from
0x0C0751EC then masked by 0xFF00; a standard CCITT variant. Verify the exact init by
recomputing against a known card if a byte-exact CREATE is needed.)

---

## 6. Homology verdict

**The JP DOC 2000 horse struct is homologous to the WE record.** The JP serializer
(0x0C076B00) reads the exact same record offsets as the WE serializer (0x0C021EC8):
rec+64/68/72 internals, +98..103 externals, +90 sex, +91..93 silks, +94 dirt, +104
hearts, +108..118 retirement/birth internals+externals, +120..123 win/place/show/races,
+130 last-race. The bitfield insert helper and the width/position pool are byte-identical
between the two ROMs. The two builds ship the same card codec.

---

## 7. Unresolved + best next probe

1. **Bytes 0x18-0x1E writer** (esp. 0x1A/0x1B/0x1C, the corpus earnings/lifetime
   candidate, and 0x18=0x0d). Not written by the career codec. Earnings (rec[80]) is not
   touched by this serializer either, so it is written by the same separate routine.
   **Best next probe:** trace the orchestrator 0x0C0765xx forward/backward from the
   `bsr 0xc076b00` at 0x0C0765A2 to find the sibling that writes the header (03 02),
   lead-ID, and the 0x18-0x1E block; grep the ROM for `mov #80,r0` (rec[80] earnings) and
   `add #24,r2` (card word 0x18) inside 0x0C075000-0x0C078000.
2. **0x00/0x01 reframing** confirmed by contradiction, not yet by finding the exact
   overwrite instructions (same orchestrator hunt as above).
3. **CRC init/final-xor**: poly 0x1021 is certain; the init constant path (0x0C0751EC ->
   mask 0xFF00) should be verified by recomputing against one byte-exact card before
   trusting a from-scratch trailer.

---

## 8. Round 2 (2026-07-25): earnings, retirement, full externals, coat/personality/dirt/G1

**Correction to round 1:** there is NO separate "sibling" routine and NO wrapper
"reframing." The serializer 0x0C076B00 is the COMPLETE card writer. Its tail (after the
literal-pool jump `bra 0xc076cd0`, code at **0x0C076CD0-0x0C076DE2**) writes earnings, the
coat/seed/personality/marker bytes at 0x20-0x27, the G1 field, and the kana name/sire/dam
(via helper 0x0C076E3C at card+0x28/+0x31/+0x3A). The tail uses the same insert helper
(ptr @0x076E34 = 0x0C0C5970) with a second width pool at 0x0C076E20.

### 8.1 EARNINGS (rec+80, a record long) — CONFIRMED, corpus-validated
Stored as an **18-bit split** (serializer 0x0C076CEA-0x0C076D06):
- low 16 bits: `mov.w @(80,r13)` -> word0x18 at (HB=0,LB=16) = bits[31:16]
- high 2 bits: `(rec80 >> 16) & 3` -> word0x1C at (HB=30,LB=2) = bits[1:0]

**Decode:** `earnings = card[0x1A] | card[0x1B]<<8 | (card[0x1C]&3)<<16`.
Validated on all 17 corpus cards: 0 on none but monotonic with career and win-weighted:
1 race -> 15-110, 3 races -> 30-210 (loser) but **750 for the sole winner キキーウイロー**,
ハカジュール (10 races) -> 1840, タイムスリップ (15 races) -> 3720. This IS the corpus's
"0x1A/0x1B grow with career" field. Scale not pinned (raw stored units; likely a x1000 or
points display). This resolves the corpus's OTHER win/earnings candidate (0x1B/0x1C): it is
**earnings**, distinct from the win record at 0x09/0x0A.

### 8.2 RETIREMENT flag — CONFIRMED = card[0x01] bit0
Deserializer 0x0C076EA8: `rec[95] = card[1] & 1`. rec+95 is the WE retired flag. So
**retired = card[0x01] & 1**. Corpus: set only on タイムスリップ (15 races), clear on the
other 16 (all mid-career). The rest of card[0x01]: bits1-3 = rec[114] (unknown 3-bit,
0 on all corpus cards), high nibble = **0x7 format constant**. So card[0x01] =
`0x70 | (rec114 << 1) | retired`, which is exactly the corpus "0x70 const, bit0 varies"
(bit0 was mislabeled a "collection marker"; it is the retired flag).

### 8.3 Exact external bit-layout (item 2) — from the insert-helper calls
Current externals (WE order Start/Corner/OOB/Comp/Tenac/Spurt = rec+98..103). A field of
width LB sits at bits [31-HB : 32-HB-LB] of the little-endian word `card+disp`:

| external | rec | card location | extract |
|---|---|---|---|
| #1 Start | 98 | byte 0x05 (whole) | `card[0x05]` |
| #2 Corner | 99 | byte 0x06 (whole) | `card[0x06]` |
| #3 OOB | 100 | byte 0x07 (whole) | `card[0x07]` |
| #4 Comp | 101 | word0x0C (HB=6,LB=6) bits[25:20] | `(LE(card[0x0C..0x0F]) >> 20) & 0x3F` |
| #5 Tenac | 102 | word0x0C (HB=12,LB=6) bits[19:14] | `(LE(card[0x0C..0x0F]) >> 14) & 0x3F` |
| #6 Spurt | 103 | word0x0C (HB=18,LB=6) bits[13:8] | `(LE(card[0x0C..0x0F]) >> 8) & 0x3F` |

Note the WIDTH difference: #1-#3 are full bytes (0-255), #4-#6 are 6-bit (0-63). Comp/
Tenac/Spurt decode 0-63 and in-range on all 17 corpus cards. **Leg-type caveat:** the WE
rule is "Start's rank among the 5 non-corner externals" (Start,OOB,Comp,Tenac,Spurt). Those
5 are all recoverable, BUT Start/OOB are full-byte scale while Comp/Tenac/Spurt are 6-bit,
so a naive numeric rank is scale-mixed (in the corpus Start byte frequently reads 140-210,
which would always outrank the 6-bit trio). Confirm the intended comparison (are the byte
externals meant to be masked to 6 bits too, or is Start genuinely a wider stat?) with a
before/after external read on the live rig before rendering leg type from JP cards.

Retirement externals (rec+108..113, 4-bit each):
| rec | card location | extract |
|---|---|---|
| 108 | word0x10 bits[19:16] | `card[0x12] & 0x0F` |
| 109 | word0x14 bits[25:22] | `(LE(card[0x14..0x17]) >> 22) & 0x0F` |
| 110 | word0x0C bits[7:4] | `card[0x0C] >> 4` |
| 111 | word0x0C bits[3:0] | `card[0x0C] & 0x0F` |
| 112 | word0x08 bits[7:4] | `card[0x08] >> 4` |
| 113 | word0x08 bits[3:0] | `card[0x08] & 0x0F` |

Silks:
| field | rec | card location | extract |
|---|---|---|---|
| silk COLOR1 | 92 | word0x14 bits[31:28] | `card[0x17] >> 4` |
| silk COLOR2 | 93 | word0x14 bits[21:18] | `(card[0x16] >> 2) & 0x0F` |
| silk PATTERN | 91 | word0x1C bits[31:29] | `card[0x1F] >> 5` |

### 8.4 Coat / personality / dirt / G1 (item 3) — do they exist on the JP card?

| field | on card? | offset (from ROM) | notes |
|---|---|---|---|
| **Coat** | **YES** | card[0x24] = coat_mod, card[0x25] = coat_base (rec+86 word, `mov.w` @0x0C076CD6) | mod=0 for normal coats -> card[0x24] reads 0 (why the corpus called 0x24 "const 0"); base at 0x25 |
| **Personality / seed** | **YES** | card[0x26] = seed_lo, card[0x27] = personality_hi (rec+88 word, @0x0C076CDE) | the seed\|personality genome word |
| **Dirt** | writes card[0x00] = rec+94 (@0x0C076B4E), read back at 0x0C076E9E | corpus 0x00 is high-entropy, not a clean 0/1 flag -> either dirt is a wide value or 0x00 doubles as an id; ambiguous |
| **G1 titles** | **YES (field present, 0 in corpus)** | card word0x1C: rec+76 low-19-bits at bits[28:10] (@0x0C076D08) + `(rec76>>19)&3` at bits[3:2] (@0x0C076D1A) | ~21-bit G1 bitfield spanning card 0x1C(bits2,3)+0x1D+0x1E + 0x1F bits0-4; **0 on all 17 corpus cards** (none have won a G1 yet, incl. G1-*eligible* ハカジュール), so eligibility != titles |

**Major reinterpretation of the "lead ID" (jp-card.md §5):** card bytes 0x24-0x27 are NOT a
random serial. They are `(coat_mod, coat_base, seed_lo, personality_hi)` = the horse's coat
+ genome word, written by the serializer tail. This is why they are stable per-horse and
look "pointer-ish/high-entropy." It also means the JP card DOES carry coat + personality +
seed (contra the round-1 "identity-only" framing). card[0x20]/[0x21] = 03/02 comes from
rec+84 = 0x0203 (the JP marker word, `mov.w` insert @0x0C076D30), and card[0x22]/[0x23] =
rec+127/rec+126.

### 8.5 Net career-block corrections after round 2
- 0x1A/0x1B (+ 0x1C bits0-1) = **earnings** (18-bit). 0x1C bits4-9 = rest timer (rec119);
  0x1C bits2-3 + 0x1D + 0x1E + 0x1F bits0-4 = **G1 titles** (0 in corpus). 0x1F bits5-7 =
  silk pattern. So bytes 0x18-0x1E ARE written by the serializer tail after all.
- retired = card[0x01] bit0 (not "0x1F bits5+6").
- byte 0x18 = a rec124/stack-derived counter (reads 0x0d on played cards, 0xfd on タイム).
