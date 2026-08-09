# Myth #2: "Beer is a food" (and does it do anything?)

ROM: epr-22336c.ic22, World Edition Rev C, CRC32 0x50053F82. RAM = file + 0x0C020000.
Investigated 2026-06-23 with docre.py. Verdict: MIXED.

## Objective
Go beyond "beer exists in the food table." Determine (a) how beer is reached
by the feed system, (b) what the food record's data fields actually do, and
(c) what effect, if any, feeding beer has on the horse.

## Finding 1: beer is a complete, structurally normal food record
Food table: 0x0C186A7C, 45 entries, 0x2C (44) byte stride.
- id 43 DRAFT BEER       @0x0C1871E0  data ptr 0x0D805AE8
- id 44 BLACK DRAFT BEER @0x0C18720C  data ptr 0x0D8086D8
Both have a full 24-byte name, a data-block pointer (into graphics ROM
0x0D8xxxxx, opaque here), and the same field layout as carrots or apples.
The accessor food_ptr (0x0C09AD3E) is pure arithmetic (id*44 + table), no
range check, so beer resolves exactly like any food. See memory-map.md.

## Finding 2: the record's effect payload is six stat-growth bytes (0x1C..0x21)
Comparing all 45 records (analyze_foods.py), the bytes at record offsets
0x1C..0x21 are the food's stat-growth payload. Proof: the "LARGE" variant of
a food is the same six bytes scaled up. The cleanest case:
- KOREAN GINSENG (36):       1C..21 = 02 02 02 02 02 02
- LARGE KOREAN GINSENG (37): 1C..21 = 04 04 04 04 04 04   (exact doubling)
Other size pairs differ only in these bytes. So 0x1C..0x21 are six growth
values (the horse's trainable parameters). The remaining fields:
- 0x22       = 00 (padding)
- 0x23       = flag byte: 1 for most foods, 0 for the mushroom and banana group
- 0x24 dword = variant flag: 0 = base, 1 = large/special   (beer = 1)
- 0x28 dword = category/menu index: foods 0..38 get 1..0x27 sequentially;
               pudding/banana share 0x27; beer = 1
NOTE: the six bytes are proven to be the growth payload by the doubling
pattern; mapping each byte to a named on-screen stat (Speed, Stamina, etc.)
would require tracing the training/growth calc and was not done here.

## Finding 3: beer's growth payload is all zeros
DRAFT BEER and BLACK DRAFT BEER both have 0x1C..0x21 = 00 00 00 00 00 00.
Feeding beer adds zero to every growth parameter. It is the only food whose
entire stat payload is zero (compare: even CARROT is 02 .. and gives a point).

## Finding 4: beer is explicitly special-cased in the stable/care code
ROM-wide cmp/eq scan: three sites compare the selected food id to BOTH 43 and
44 with no count bound (#45) nearby, i.e. genuine beer-only branches, all in
the stable region:

1) 0x0C0A4110 (draw/UI). Beer 43 and 44 each use a HARDCODED display offset
   (words at 0x0C0A41A4 = 0x764, 0x0C0A41A6 = 0x790; delta 0x2C = one stride)
   into a separate buffer instead of FOOD_TABLE + id*44, then read +0x18 (the
   data pointer) to draw. Normal foods fall through to the usual id*44 path.

2) 0x0C0A5B44 (feed handler). Banana (41/42) and beer (43/44) are grouped into
   a special branch. Beer writes a reaction-state byte at 0x0C21A4F4:
   DRAFT BEER -> 1, BLACK DRAFT BEER -> 2, plus a flag at 0x0C3DDEDC. This is a
   "which novelty item was chosen" reaction selector, not a stat update.

3) 0x0C0A5FA0 (feed handler). Beer id triggers a dedicated table lookup
   (mov.l @(r0,r3),r3) into beer-only pointer arrays at 0x0C187E60 and
   0x0C187E8C (just past the food table). Those hold ~6 pointers each into the
   0x0C149xxx region: beer's own reaction/animation sequence.

The feed/UI function itself never reads the growth bytes (0x1C..0x21); for
normal foods the growth is applied by the separate training calc, while for
beer the handler runs the reaction path above.

## Conclusion (MIXED)
"Beer is a food" is true in the strongest sense: it is a fully implemented,
special-cased, feedable item, not a hollow gag. But "beer is a food that boosts
your horse like other feed" is false: its six growth bytes are all zero, so it
confers no stat gain. Instead the game special-cases beer to play a unique
reaction (state 1 for draft, 2 for black draft, with dedicated asset tables).
Net: beer is real and feedable, but nutritionally inert, a novelty reaction.

## Finding 5: is beer "turned off" in Rev C? (access vs effect)
Tested the hypothesis that beer is in the game but disabled for us.
- ACCESS and EFFECT are separate systems. The stat bytes (0x1C..0x21) are
  effect only; the offer gate never reads them. Setting beer to 02/04 would
  make it grant stats, not make it appear.
- The offer gate (0x0C04806C) reads runtime array 0x0C21D738[food], indexes
  AVAIL[value*16 + mode_col]; value < 3 => offered. Keyed on the runtime value,
  not the food id.
- Writer of 0x0C21D738 LOCATED: care-state init, fill loop 0x0C03563C, base
  care+0x80, each entry = runtime divide (0x0C0C211C), 40-wide. Values computed
  at runtime, not a static table; beer indices 43/44 are above the 40-wide fill
  and get no beer-specific value.
- NO beer-specific disable exists in any static layer (record, accessor, AVAIL
  rows, initializer). Beer also carries live reaction code, which argues it is
  meant to be encountered.
Conclusion: beer is not deliberately disabled. Its offered/not state in a given
session is a runtime-computed value, confirmable only by a live RAM read of
0x0C21D738[43/44] in Flycast (beer = 0x0C21D7E4 / 0x0C21D7E8) or by observing
the feed menu in play.
Repro: trace_avail.py, trace_arraywriter.py, disasm 0x0C0355E0 region.

## Open thread (carried)
The writer of runtime eligibility array 0x0C21D738 is still not statically
located (computed as care-state base + 0x80). Separately, naming each of the
six growth bytes to an on-screen stat needs the training/growth calc traced.

## Repro
    python analyze_foods.py     # food param table + LARGE-pair deltas
    python trace5.py            # the three beer special-case branches
    python trace6.py            # display offsets, reaction states, asset tables
