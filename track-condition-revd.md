# Myth #1 on Rev D: the going system is DIFFERENT (and surface-specific)

ROMs: Rev C epr-22336c.ic22 vs Rev D epr-22336d.ic22 (World Edition EX). Ran the
Myth #1 ("turf can be Good to Soft") analysis on Rev D. Investigated 2026-06-23.
**Result: Rev D replaced the going system. The going IS surface-specific in Rev D,
and "GOOD TO SOFT" does not exist.** So Myth #1's answer is version-dependent.

DEPLOYED 2026-06-23 as a "Footnote: Rev D is different" block on the live Myth #1
page (doc.johnreevesiii.com/doc-myths.html#track-condition-turf), verified 200.
Page presents the dirt/turf vocab table + the 0x0C173208 split as proof; the
exact +4 surface arithmetic is described as the table layout (not over-claimed
as instruction-traced).

## Rev C (recap, deployed verdict: FALSE)
4 track-condition values on ONE scale, surface-independent:
0=GOOD, 1=GOOD TO SOFT, 2=SOFT, 3=HEAVY (strings at file 0x10EB3C block).
Renderer reads the value raw and indexes a 4-string table; no surface read.
=> turf CAN be Good to Soft. (live at doc.johnreevesiii.com/doc-myths.html)

## Rev D: the going vocabulary is expanded and surface-split
Going terms present (going_terms_cd.py):
- Rev C: GOOD, GOOD TO SOFT, SOFT, HEAVY  (4; "GOOD TO SOFT" present)
- Rev D: FAST, GOOD, HEAVY, MUDDY, FIRM, SOFT, YIELDING  (7; NO "GOOD TO SOFT")
These are real-world going words split by surface: dirt = Fast/Good/Heavy/Muddy;
turf = Firm/Good/Soft/Yielding.

## Proof: the Rev D display table is surface-partitioned
Display string table @0x0C173208 (string-pointer-verified; each entry resolves
to a real "TRACK CONDITION  X" string at 0x0C1301AC..):
  index 0 FAST   |  index 4 FIRM
  index 1 GOOD   |  index 5 GOOD
  index 2 HEAVY  |  index 6 SOFT
  index 3 MUDDY  |  index 7 YIELDING
  (indices 8-10 = WEATHER SUNNY/CLOUDY etc.)
Indices 0-3 = the DIRT going set (Fast/Good/Heavy/Muddy); 4-7 = the TURF going
set (Firm/Good/Soft/Yielding). The surface-exclusive terms land in their own
blocks: FAST/MUDDY (dirt-only) at 0/3; FIRM/YIELDING (turf-only) at 4/7.
Renderer @0x0C075676 indexes table[going_value] with going_value read AS-IS from
a RAM global (0x0C21D080); no +4 added at render time, so the surface offset is
already baked into the stored 0..7 value (turf = condition+4). The dirt/turf
partition of the table is itself conclusive that the index encodes surface.

## Conclusion / how it bears on the claim
- Rev C: "Good to Soft" is a real, surface-INDEPENDENT condition -> turf can be
  Good to Soft. (Myth #1 = False on Rev C.)
- Rev D: the going was rebuilt as surface-SPECIFIC. Turf races show Firm/Good/
  Soft/Yielding; dirt races show Fast/Good/Heavy/Muddy. "Good to Soft" was
  removed entirely.
=> The claimant's intuition ("Good to Soft can't be turf") matches REV D's
surface-split going, mis-applied to Rev C. The ROM shows the two builds use
different going systems. Strongest, most surgical framing for the page.

## Open (nice-to-have, not required for the finding)
- Trace the exact writer of 0x0C21D080 to show value = base_condition +
  (turf?4:0) at the instruction level (currently inferred from the table layout;
  many writers + literal-pool indirection). Cleanest confirmed live in Flycast:
  watch 0x0C21D080 on a turf race (expect 4-7) vs a dirt race (expect 0-3).

## Repro
    python goodtosoft_cd.py   # strings + tables, C vs D
    python going_terms_cd.py  # going vocabulary diff
    python revd_going.py      # Rev D display table 0x0C173208 (surface-split)
    python revd_going2.py     # renderer literal pool + going-value global
    # Rev D strings: docre.load_rev('D'); docre.strings(0x0C1301A0,0x110,4)
