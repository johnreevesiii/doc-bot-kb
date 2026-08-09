# Card bytes 62 (a1[7] "running-style seed") and 93 (a2[45] "birth trait"): what the game actually does with them

Static SH-4 RE, epr-22336c.ic22 (WE Rev C), docre convention (RAM = file + 0x0C020000).
Session 2026-07-18. Question: what game code READS these two card fields after creation,
and can a1[7]=0 (Breeding Lab) cause the CF-28 "lab horses ignore the whip" behavior.

One-line verdicts:
- **a1[7] (file 62)**: it is real GENETIC material, not a behavior input. Read post-creation
  only by (a) the breeding cross, which bit-mixes both parents' seed|personality word into the
  foal, and (b) appearance plumbing that carries the word around verbatim. The one accessor
  that would decode the seed's low nibble (0x0C073506) has ZERO callers (dead code). No race,
  whip, AI, or physics code reads it. **CF-28 cannot be caused by a1[7]. VERDICT: inheritance
  payload only.** (High confidence.)
- **a2[45] (file 93)**: stamped at birth as a deterministic 2-bit HASH, `(rec[1] + rec[3] +
  rec[91]) & 3` (two name bytes + silk pattern), by 0x0C0739F2. It is carried card <-> record
  <-> display entity but **no consumer was found anywhere: not feeding, not race, not care.
  VERDICT: inert birth tag (probably an intended "individuality" stamp that nothing ever
  reads back).** (Moderate-high confidence; coverage notes below.)

---

## 1. The WE card I/O architecture (all new, byte-verified)

### 1.1 Decoded card image, stride-72 tracks
The canonical decoded card lives at **0x0C21B92C** as three 72-byte track slots
(69 payload + 3 spare). Mapping to the 207-byte `.card` file:

    image_offset(f) = 0x0C21B92C + 72*(f div 69) + (f mod 69)     [f = .card file offset]

Proof (validator, entry region 0x0C023460):
- UID redundancy check: `mov #64,r0; mov.w @(r0,r4)` vs +136 vs +208, and +66/+138/+210.
  Track offset 64..67 = file 0x40..0x43 = the triple-stored UID (us-card section 6). The game
  ONLY checks equality across the three tracks, it does not validate the value. Editor-random
  UIDs pass.
- Marker check at image +144..151 (= file 0x8A..0x91, track 3 start): accepts ASCII
  "SEGABEX0" (immediate cmp chain at 0x0C0234D0..0x0C023520) OR "SEGABEF0" (absolute byte
  pointers 0x0C21B9BD..0x0C21B9C3, chain at 0x0C023526..0x0C02356C). The ROM contains no
  "SEGABEF0" string; the product code "BEF0" sits in the game header at file 0x134.
- Also gates: earnings dword @+88 (file 85..88) >= 1000 when mode var [0x0C21A13C] >= 2;
  retired flag dword @+156 (file 0x96, a3[57]) == 1; sex byte @+125 (file 122, a2[16]).
  Returns classification codes 5..9.

Staging chain (reader frame -> image): raw frame `*0x0C21C7DC` = 0x0C21BADC with tracks at
+6/+75/+144, per-track work buffers `*0x0C21C7EC/F0/F4` = 0x0C21BDDC/BEDC/BFDC, overlap-safe
memmove at **0x0C0C2590**, dispatcher stubs 0x0C022B20..0x0C022C4B (all `mov #69,r6`).
Pointer vars initialized at 0x0C021C0C (pool 0x0C021CA0..0x0C021D20).

### 1.2 The WE card parser (card image -> horse record)
**Function 0x0C02266A** (two entries by arg):
- arg != 1: dst = **player record 0x0C21A5A8**, src = image 0x0C21B92C, aux = 0x0C21A62C
- arg == 1: dst = record slot B 0x0C21A63C (stride 148 from slot A), aux = 0x0C21A6C0

Record layout extracted from the parser body (0x0C0226AE..0x0C022876 and continuation):

| record off | source (file offset) | field |
|---|---|---|
| +0..19 | 0..17 | horse name (18 ASCII + 2 NUL) |
| +20..39 / +42..61 | 20..39 / 40..59 | sire / dam name |
| +64/+68/+72 (longs) | 69/73/77 | current internals stamina/speed/sharp (a2[69]/[65]/[61]) |
| +76 (long) | 81..84 | G1 bitfield bytes |
| +80 (long) | 85..88 | earnings |
| +84 (word) | 156,157 | a3[51]/a3[50] format marker 0x30,0x10 |
| **+86 (word)** | **60,61** | **coat word = a1[9] mod (lo) / a1[8] base (hi)** |
| **+88 (word)** | **62,63** | **a1[7] SEED (lo) / a1[6] personality (hi)** |
| +90..93 | 122..125 | sex, silk pattern, silk color1, silk color2 |
| +94/+95 | 146/150 | dirt (a3[61]), retired (a3[57]) |
| **+96** | **93** | **a2[45] birth trait** |
| +97 | 94 | condition (a2[44]) |
| +98..103 | 95..100 | current externals Start,Corner,OOB,Comp,Tenac,Spurt |
| +104..107 | 101..104 | hearts, trust, total races, wins-dup |
| +108..113 | 105..110 | retirement externals Start..Spurt |
| +116..118 | 113..115 | retirement internals stamina/speed/sharp |
| +119..123 | 116, 89..92 | rest timer; won/place/show/out |
| +128..131 | 117..120 | current+last race pairs (census bytes b117..b120) |

Key instruction evidence for the two targets:

    0C022718  mov #62,r0 ; mov.w @(r0,r5),r2      ; src word at file 62,63
    0C02271C  mov #88,r0 ; mov.w r2,@(r0,r4)      ; -> record +88   (a1[7] lo, a1[6] hi)
    0C02276C  mov #96,r0 ; mov.b @(r0,r5),r2
    0C022770  mov.b r2,@(r0,r4)                   ; image +96 (file 93) -> record +96

So: **a1[7] lives at 0x0C21A600 (record+88 low byte); a2[45] lives at 0x0C21A608 (record+96).**
The WE writer (record -> image, mirror function at 0x0C022914 region) reads +88/+96 back
(reads at 0x0C022986 / 0x0C0229D8). A separate OLD-FORMAT (DOC2000/kana, bit-packed) codec
pair also exists: serializer 0x0C021EC8 and deserializer 0x0C02227C over record slot B,
using bitfield insert/extract helpers 0x0C0C1F70/0x0C0C1FA8; it has no a2[45] equivalent
(it zeroes +96, see 0x0C0225BC).

### 1.3 Downstream copies (where the fields travel)
- Record -> display/race ENTITY builders (family: 0x0C078C80 body 0x0C078D52; also
  0x0C077B40 region, 0x0C077E80 region, 0x0C078150 region, 0x0C079020 region):
  rec w+86 -> ent+52 (coat), rec w+88 -> **ent+54** (seed at +54, personality +55),
  rec+96 -> **ent+70**, rec+97 -> ent+71, internals bytes -> ent+60..62, externals ->
  ent+72..77, hearts -> ent+78. Builder 0x0C078C80 registers the entity pointer into the
  race slot table (stride 0x2A0 area at 0x0C21F7E8+), tying it to the known race arena.
- Entity -> race_data frame (race struct = entity+32): copiers at 0x0C0625xx/0x0C0628xx
  move +52..+56 etc. so seed/personality land at **race_data+22/+23**. The trait (ent+70)
  is NOT copied into race_data (no `mov #70` source and no `mov #38` destination in those
  copiers; global scans confirm).

---

## 2. a1[7]: every located post-creation reader

1. **Breeding cross = the real consumer.** Cross function entry **0x0C072B0C**. Parents are
   two 60-byte catalog-format buffers: sire 0x0C21A530, dam 0x0C21A56C (player record
   0x0C21A5A8 sits right after; 0x0C061250 clears all three at registration). The cross:
   - internals: floor-average + bloodline bonus. Bloodline points counted per external
     (both parents >= 12 or both <= 3, comparisons at 0x0C072BEE..0x0C072CD6); 4-5 points:
     sta+1, spd/shp+2; 6 points: +2/+3/+3; clamp 45 (`mov #45,r4; cmp/gt` 0x0C072D22..).
   - retirement externals: per-slot floor-average into record+108..113.
   - sex: RNG & 1 -> record+90.
   - **seed|personality word: genetic BIT-MIX, function 0x0C07333E.** Inputs r4/r5 = sire/dam
     word at buffer w+50 (= catalog record +0x2E..0x2F). Algorithm, byte-exact:

         pass 1 (sire), masks 0x8000,0x2000,...,0x0002 (odd bit positions):
             r0 = rand()            ; LCG core at 0x0C0B1E60 (static 0x091E60 + 0x20000)
             bit = (rand&1) ? (sire<<1) : sire   ; i.e. foal bit n = sire bit n or n-1
         pass 2 (dam), masks 0x4000,0x1000,...,0x0001 (even positions):
             bit = (rand&1) ? dam : (dam>>1)     ; foal bit n = dam bit n or n+1
         result -> record+88 (literal 0x0C21A600 at [0x0C073410]) and global 0x0C3C0FD8

     So each foal bit is a coin flip between two adjacent bits of one parent; odd bits come
     from the sire, even bits from the dam. **a1[7] is inherited chromosome-style, jointly
     with personality.** This explains the fleet stats: zero-seed parents give zero-seed
     foals (Lab writes 0, so all Lab lineages stay 0), and mixed parents give the wide
     "128 distinct values" spread.
   - The coat word gets its own mixer (0x0C073154, special-coat aware, same module).
2. **Catalog origin decoded.** The 84+84 breeding catalog (docre base 0x0C12BF1C, name+0)
   composite field **+0x2C..0x2F = (coat_mod, coat_base, runstyle_seed, personality)**.
   Seed column across all 168: 80 distinct values, 32 zeros. Personality column: anchored
   values (0, 48, ...). This answers breeding_routine.md's open question: name+44 IS used,
   it is the coat/seed/personality genome of the starter horses.
3. **Appearance plumbing (carries, does not decode).** 0x0C04CDFE(dst, coat_w, seed_pers_w)
   stores the coat class (from 0x0C0734A4) at dst+8 and the RAW words at dst+24/+26.
   Called from care-init 0x0C034130 (r5=w+86, r6=w+88), entity path 0x0C077FF6, and the
   catalog candidate builder 0x0C071F38 (composite words). No decoder of dst+26 low byte
   was located.
4. **Nibble accessors (the old "style = seed/51" ghost):**
   - 0x0C0734EC(word) = table 0x0C12BEF8[bits 12..15] -> class 0..4. Input is the +88 word,
     so this uses ONLY the personality (a1[6]) high nibble. Caller example 0x0C058DF6 maps
     class 0..4 to flag bits 0x20/0x800/0x100/0x1000/0x400 in an object's +16 flags
     (animation/reaction selection); about 29 pointer-call sites (care, feed reactions
     0x0A5xxx, race presentation 0x053xxx..0x059xxx).
   - **0x0C073506(word) = table 0x0C12BF08[bits 0..3], the ONLY code that would decode
     a1[7]'s low nibble: ZERO callers in the whole image. Dead code.**
   - 0x0C0734A4 = COAT classifier (input is the +86 coat word, not +88): base>=0x40 uses
     base low nibble via 0x0C12BE78, base<0x40 (special) uses mod high nibble via
     0x0C12BE88; output indexes the 16-pointer name table 0x0C151218 ->
     GRAY/CHESTNUT/BLACK/BAY/BROWN/SPECIAL/WHITE strings at 0x0C0E68F0.
     Verified against the us-card coat table (Bay 77 -> BAY, Black 65 -> BLACK, special
     mod 80 -> WHITE, etc.). Earlier docs conflated this with a runstyle accessor.
5. **Race engine: nothing.** Global scans for the seed's addresses in every frame:
   record+88 (r0-indexed imm 88 near record base loads), entity+54/+55 (global imm scan),
   race_data+22/+23 (global imm scan). All hits are the codec, the builders, the
   entity<->race converters (0x0C0781xx/0x0C0790xx), one untyped store-to-global at
   0x0C0B8D12 (saves a +22 byte of an untyped struct to [0x0C0B8F08], result-screen
   territory), and the input-test region 0x0C32B022. The whip/input handler (docre
   0x0C089280 family) reads only the velocity vector +0x0C/+0x10/+0x14 and input bits;
   the stat-sum, curve, and physics functions read +40..45/+76/+84/+116. None touch
   +22/+23/+38/+39.

**CF-28 verdict: a1[7]=0 on Lab foals cannot change whip response; no code path exists from
the byte to race behavior.** If Lab horses really do feel different, the mechanism with an
actual code path is **personality (a1[6])**: it drives the 0..4 reaction-class flags
(0x0C0734EC) used across care/feeding/presentation, and the Lab writes only the 5 lossy
anchor values {0,48,64,80,208} instead of the full 0..255 range (us-card section 3 already
flags the anchor lossiness). Anchors 0/48/64/80/208 map to classes 2/0/3/1/3, so Lab horses
cluster into fewer reaction classes than game-born ones.

---

## 3. a2[45]: birth stamp decoded, and the reader hunt

### 3.1 The birth writer (found)
**0x0C0739F2** (r4 = player record), called from the cross at 0x0C072B44 and via function
pointer [0x0C07E15C] from the candidate/foal generation module (jsr at 0x0C07E054):

    0C0739F4  r5 = rec[1] + rec[3]          ; name bytes 2 and 4
    0C073A00  r2 = rec[91]                  ; silk pattern
    0C073A04  rec[96]  = (r5 + r2) & 3      ; a2[45]  <- THE BIRTH TRAIT
    0C073A10  rec[97]  = (rec[3] + rec[5] + rec[92]) & 3   ; condition seed (silk color 1)
    0C073A2E  rec[105] = (rec[1] + rec[5] + rec[93]) & 3   ; trust seed     (silk color 2)

So a2[45] is a **deterministic 2-bit hash of the record's name bytes and silk pattern at
the moment of foal creation**. It is stamped alongside the initial condition and trust
values (which explains the us-card observation that a2[45] "tracks trust" on fresh cards:
they are sibling hashes of the same inputs).

Fleet skew interpretation (INFERRED, not verified live): when the cross runs on a fresh
record (name/silk not yet meaningful) the sum degenerates to a constant, giving the
dominant class 2; when breeding runs with a populated record (rebreeding a retired card),
the hash is effectively random over {0,1,2,3}, giving the rare classes. The Lab's constant
2 therefore matches the majority case exactly. A live Flycast breeding capture of rec[1],
rec[3], rec[91] at the 0x0C0739F2 call would settle the order-of-operations.

### 3.2 Readers (exhaustive hunt, none found)
Search coverage, all on the full 4MB image:
- record+96/+97: global scan for `mov #96/#97` immediates followed by @(r0,Rn) access.
  Hits: WE parser (0x0C02276C read), WE writer (0x0C0229D8 read), old-format codec zeroing
  (0x0C0225BC), birth writer (0x0C073A04), the record->entity builders (0x0C077BDC,
  0x0C078DC8 reads), and two false positives that decode as data/other structs
  (0x0C0A835C sits in a literal pool; 0x0C0813D6 reads a long at +96 of work struct
  *[0x0C081430]=0x0C3C9B80, a 0..3 state machine unrelated to the record).
- entity+70/+71: global imm scan. Writers = the five builders + a generic clear
  (0x0C0636FE). Readers = word reads at 0x0C07B4D2/0x0C0630CE/0x0C02DDxx group, ALL of
  which feed sin/cos (0x0C0B1A00 / 0x0C0B1FA0, verified trig by the 2^-15 scaling and
  quadrant reduction) and belong to camera/heading objects, not the horse entity.
- race_data+38/+39: global imm scan. Only the entity<->race format converters
  (0x0C077E94..) which shuttle the byte back and forth. The race-slot copiers do not
  propagate the trait at all.
- Feeding hypothesis (items-feeding): the feed/care subsystem reads care_state 0x0C21D6B8,
  the availability tables, and record stat bytes; no path reads record+96. Rejected.

**Conclusion: a2[45] is written at birth, faithfully round-tripped through card and
entity structures, and never consumed.** Remaining blind spots for both fields: access
through a parameter-passed record pointer using a displacement pattern my scans miss
(e.g. computed offsets), or GBR-relative addressing. Given every other field access in
this codebase uses the r0-indexed idiom the scans cover, the risk is low.

---

## 4. Corrections and side findings for other docs

- **memory-map.md race DATA struct labels are suspect**: the post-race writeback
  (0x0C0772C4 region, writes at 0x0C077604..0x0C07762E) copies race+40..45 into record
  +98..+103, which this decode proves are the six CURRENT EXTERNALS (Start..Spurt), not
  "Speed/Stamina/Sharp internals + 3 externals". The stat-sum weighting analysis should be
  re-read in that light (consistent with breeding-race-meta: externals win races).
- **Catalog composite (+0x2C) decoded** (see 2.2): (coat_mod, coat_base, seed, personality).
  Closes an open question in sh4-race-formula/decode/breeding_routine.md.
- **Bloodline bonus formula located statically** (0x0C072BEE..0x0C072D3E): external-based
  points, thresholds >=12 / <=3, bonuses +1/+2/+2 and +2/+3/+3, internal clamp 45.
- **Old-format codec** (0x0C021EC8/0x0C02227C) shows WE can read/write DOC2000-style
  bit-packed kana cards (marker SEGABEX0 accepted): version-up path.
- The 0x0C061250 record-clear writes sentinel 0xFFFF into +86/+88 and 255/11 pairs into
  +128..131, which likely explains the census's b120=11 minority variant.
