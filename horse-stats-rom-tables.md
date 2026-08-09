# Horse stats: Speed, Stamina, Sharp (the three core "internal" stats)

ROM: World Edition Rev C, epr-22336c.ic22, CRC32 0x50053F82.
Convention: RAM = file + 0x0C020000. All addresses are RAM addresses.
Investigated 2026-06-25 (docre static RE, multi-agent sweep + reconciliation).

## Verdict in one line
The three core player-raised stats are confirmed by ROM string labels as
**Speed, Stamina, Sharp** (a 4th label, Friendship, is a relationship stat, not a
race stat). Speed is proven end to end (fed -> accumulator -> stat byte -> read by
race physics and by the ranking function, with a hard cap of 63). Stamina is
proven on both sides with the same cap. Sharp is confirmed as a real stat that
the race engine reads, but its training-side commit writer could not be located
statically, so it is flagged reader-strong / writer-partial.

## The third stat name is "Sharp" (CONFIRMED, objective 4)
A static stat-label table exists in ROM. Dumped directly:

    0x0C14874C  "Speed\0"
    0x0C148754  "Stamina\0"
    0x0C14875C  "Sharp\0"
    0x0C148764  "Friendship\0"
    0x0C148770  "Most favorite food."

Field order in the label table is Speed [0], Stamina [1], Sharp [2], Friendship
[3]. Corroborated by three more ROM string regions:
- 0x0C10E270  "Speed type" / "Stamina type" / "Sharp type" (UI classification)
- 0x0C126F74  ", SPEED TYPE" / ", STAMINA TYPE" / ", SHARP TYPE" (in-sentence)
- 0x0C148FE0  "%s is a sharp horse." / "%s is a well balanced horse."

Candidates Sharpness, Spirit, Guts, Subtle, Power, Spurt, Tempo do NOT appear as
standalone stat labels anywhere. The third stat is Sharp, not any of those.
Note: names like Power/Guts/Intelligence/Tempo that appeared in an earlier draft
of memory-map.md were worker assumptions, not ROM strings. They have been struck.

## Architecture: there are three representations of a horse's stats
The single most important correction from this session: the care/training stat
bytes and the race stat bytes are NOT the same field copied 1:1. There are three
distinct structures.

1. PLAYER HORSE RECORD (care/training), base 0x0C21A5A8 (runtime work-RAM).
   - +0x62..+0x67 (6 bytes, contiguous): race-format stat snapshot. +0x62 Speed,
     +0x63 Stamina, +0x64 Sharp, +0x65..+0x67 three more. Written by post-race
     writeback (see bridge below). (Earlier notes calling this block "6 externals"
     or "talent counters" were misreads; it is the race-format stat mirror.)
   - +0x68 Speed, +0x6A Stamina, +0x6C third-stat slot (care format, stride 2 with
     a flag byte at the odd offset between). These are the FED/TRAINED stats,
     capped at 63, and read live by the physics integrator.

2. RACE FIELD DATA STRUCT (one per horse in the field, incl. CPU horses), reached
   at runtime via race_info[+0] (race_info base 0x0C21A294).
   - +40 Speed, +41 Stamina, +42 Sharp (bytes), plus +43/+44/+45 and +28..+30
     (additional mode-weighted bytes, unlabeled). Read by the ranking function.

3. RACE PHYSICS STRUCT (floats, per horse), base 0x0C21A234 (Rev C, runtime).
   - Velocity, drag, pace accumulator etc. (see memory-map.md). The trained Speed
     byte is fed into this integrator as a coefficient.

### The bridge (how care and race views connect) -- 1:1 copy REFUTED
- The byte memcpy at 0x0C0229E4 (entry 0x0C0228DC) is a SAME-OFFSET clone:
  `mov #N,r0 ; mov.b @(r0,r5),r2 ; mov.b r2,@(r0,r4)` for N = 98..119, with
  R5 = 0x0C21A5A8 (player record), R4 = 0x0C21BA04 (a same-layout duplicate
  record). Stats stay at +0x68/+0x6A; it never writes +40/+41/+42.
- A full-ROM scan for any `read off 104 -> store off 40` (and 106->41, 108->42,
  and the reverse) returned ZERO hits. There is no training+0x68 -> race+40
  translation copy.
- What actually links them (both proven):
  - LIVE physics read: function 0x0C064C6E reads the trained Speed byte directly
    via literal 0x0C21A610 (= player record +0x68) and uses it as a float
    coefficient in the per-tick velocity integrator (R14 = physics struct):
        0C064C6E  mov.l 0xc064cd8,r1   ; r1 = 0x0C21A610 (record+0x68, Speed)
                  mov.b @r1,r2         ; r2 = Speed byte
                  ...  float ; fmac fr0,fr2,fr1
    (Speed also loaded at 0x0C069A2A.)
  - POST-RACE writeback: function 0x0C0772C4 at site 0x0C0775FC copies the race
    DATA bytes back into the player record's contiguous +0x62 block:
        0C0775FC  mov #40,r0 ; mov.b @(r0,r14),r3 ; mov #98,r0 ; mov.b r3,@(r0,r4)
    with R14 = race DATA struct, R4 = 0x0C21A5A8. So race +40/+41/+42
    (Speed/Stamina/Sharp) are snapshotted to record +0x62/+0x63/+0x64.
  - Layout-converter functions 0x0C0772C4 / 0x0C0778F0 / 0x0C0815D8 shuffle these
    blocks (e.g. record +99..+104 -> dest +73..+78, a fixed -26 shift). The exact
    pre-race routine that seeds race DATA +40 from the player record was not
    pinned statically; it runs through these converters. OPEN (runtime-traceable).

## Per-stat findings

### SPEED -- confidence HIGH (full chain proven)
- ROM label: "Speed" (table 0x0C14874C[0]).
- Care/training field: player record +0x68 = abs 0x0C21A610, 1 byte, range 0..63.
- Growth accumulator: care_state +0x40 = abs 0x0C21D6F8 (LONG), fed by food byte 0.
- Cap constant: 63 (0x3F). Enforced in the Speed writer:
      0C036AE6  ... cmp/pz r4 ; (bt) mov #0,r4      ; low clamp to 0
      0C036AEC  mov #63,r13 ; cmp/gt r13,r4 ; ... mov r13,r3   ; high clamp to 63
  (No clamp to 60 anywhere; see Cap note.)
- WRITER (training): function 0x0C036A94. R14 = care_state (0x0C21D6B8, set at
  0x0C036AC2). Reads the Speed accumulator care_state+0x40:
      0C036ADA  mov #0x40,r4 ; add r14,r4 ; mov.l @r4,r4   ; r4 = Speed acc
  adds the training delta, clamps to [0,63], then commits the byte to the record:
      0C036B42  mov.b r2,@r3        ; r3 = 0x0C21A610 (record+0x68), from lit 0x0C036C3C
      0C036C1C  mov.b r4,@r3        ; clamped int written again
  Dispatch: reached only via the training-activity table entry [0x0C0EA8B8]
  (no direct JSR/BSR caller).
- READER (race, velocity): function 0x0C064C6E reads record+0x68 live as the
  velocity float coefficient (disasm above). This is Speed entering the sim as a
  velocity term.
- READER (race, ranking): stat-sum function 0x0C05C75C reads race DATA +40:
      0C05C7D8  mov #40,r0 ; mov.b @(r0,r14),r3 ; extu.b r3,r3   ; r14 = DATA struct
  summed (mode-weighted) into a float performance score that orders the field.
- Race effect: Speed sets the horse's velocity ceiling / pace coefficient; higher
  Speed means a higher top-end, and it is the dominant term in sprint modes
  (x2 in mode 2, x4 in mode 3 of the ranking weight).

### STAMINA -- confidence HIGH (both sides + cap proven; writer fn less pinned)
- ROM label: "Stamina" (table 0x0C14874C[1]).
- Care/training field: player record +0x6A = abs 0x0C21A612, 1 byte, range 0..63.
- Growth accumulator: care_state +0x44 = abs 0x0C21D6FC (LONG), fed by food byte 1.
- Cap constant: 63 (0x3F). Enforced by cap checks that read +0x6A and skip the
  increment when already at 63:
      0C037D5E  ... mov.b @(...),r0 ; extu.b ; cmp/eq #63,r0 ; bt [skip]   (reads 0x0C21A612)
      0C046E78 / 0C046E96  same cmp/eq #63 pattern on record+0x6A
- WRITER (training): the Stamina commit lives in the training-activity dispatch
  path (candidate handler 0x0C036D4C at table entry [0x0C0EA8BC]); the final byte
  store uses a computed base+offset rather than a 0x0C21A612 literal, so the exact
  store instruction is less cleanly isolated than Speed's. The field, accumulator,
  and cap are all proven; the precise commit instruction is PARTIAL.
- READER (race, ranking): stat-sum 0x0C05C75C reads race DATA +41:
      0C05C7DC  mov #41,r0 ; mov.b @(r0,r14),r2 ; extu.b r2,r2
  Doubled (x2) in distance mode 1 (shll at 0x0C05C82E), x1 in sprint modes.
- Race effect: Stamina is the endurance weight; it matters most in long-distance
  races (doubled in route mode) and least in sprints.

### SHARP -- confidence MIXED (name + race reader proven; training commit NOT located)
- ROM label: "Sharp" (table 0x0C14874C[2]). Name CONFIRMED (objective 4).
- Race-format snapshot field: player record +0x64 (written back from race +42).
- Care-format slot: player record +0x6C = abs 0x0C21A614. This slot has ZERO
  literal references, no `mov #108,r0` byte store, and no `add #108` in the
  training code region. No direct writer was found for +0x6C.
- Growth accumulator: care_state +0x48 = abs 0x0C21D700 (LONG), fed by food byte 2
  (WATERMELON = [0,0,2,0,0,0], a sole-grower). The accumulator is real (grouped
  with the other stat accumulators in three init-clear blocks at 0x034158 /
  0x029BEA / 0x04D22A). So feeding DOES accumulate Sharp growth.
- MISSING LINK: the accumulator -> final-stat-byte commit for Sharp was not
  located. Speed's accumulator(+0x40) -> byte(+0x68) writer (0x0C036A94) has no
  Sharp counterpart that I could prove. This is the bottom-out: per the workspace
  stop rule, I am not inventing one.
- Cap constant: none provable (no +0x6C writer/check to read it from). By analogy
  with Speed/Stamina it is presumably 63, but that is INFERRED, not proven.
- READER (race, ranking): stat-sum 0x0C05C75C reads race DATA +42:
      0C05C7E6  mov #42,r0 ; mov.b @(r0,r14),r2 ; extu.b r2,r2
  Doubled (x2) in distance mode 1 (shll at 0x0C05C83A) alongside Stamina; x1 in
  sprints. So Sharp behaves as a long-race closing-kick weight.
- Race effect: Sharp is the closing-kick / late-race weight; it is amplified in
  distance races (doubled with Stamina) and neutral in sprints.
- FLAG: Sharp is reader-strong, writer-partial. The fed accumulator exists; the
  commit writer and the cap instruction were not found statically. Resolve by live
  Flycast: feed WATERMELON, watch care_state+0x48 (0x0C21D700) and record +0x64 /
  +0x6C, confirm which byte the Sharp accumulator commits to and its ceiling.

## Cap reconcile: the enforced cap is 63 (0x3F), not 60
- Speed and Stamina both clamp to 63 (0x3F) with the instructions cited above.
- No internal-stat clamp to 60 exists. Every `cmp/eq #60` site in the ROM
  (0x0C07A04E, 0x0C07D4D8, 0x0C07E3A4, 0x0C02B6C2, 0x0C053F66) operates on LONG
  values: lap/position counters and a menu state-machine index (47/59/60/61/62/63
  chain), not stat bytes. The race reader at 0x0C05C75C reads the raw stat bytes
  with no 60 ceiling.
- John's perceived "max 60" is therefore a display ceiling or a practical reachable
  maximum, not a hard code cap. The code cap is 63.

## Food growth byte -> stat mapping (sole-grower evidence, byte0/1/2 = the 3 core)
Food record table base 0x0C186A7C, stride 44; growth payload at record +0x1C..+0x21
(6 bytes). Each byte has a dedicated sole-grower food (dumped directly):

    byte 0 (+0x1C): CARROT id0      [02 00 00 00 00 00]  -> Speed   (accumulator +0x40, CONFIRMED)
    byte 1 (+0x1D): FODDER id3      [00 02 00 00 00 00]  -> Stamina (accumulator +0x44)
    byte 2 (+0x1E): WATERMELON id8  [00 00 02 00 00 00]  -> Sharp   (accumulator +0x48) [INFERRED]
    byte 3 (+0x1F): RADISH id10     [00 00 00 02 00 00]  -> unlabeled trainable param
    byte 4 (+0x20): CABBAGE id12    [00 00 00 00 02 00]  -> unlabeled trainable param
    byte 5 (+0x21): APPLE id6       [00 00 00 00 00 02]  -> unlabeled trainable param
    all six:        KOREAN GINSENG id36                  [02 02 02 02 02 02]
LARGE variants double each byte (proven earlier). The byte k -> accumulator
care_state+(0x40 + 4k) correspondence is confirmed for byte0->+0x40 (Speed); the
generic apply loop (byte k -> field k) was not re-disassembled this round, so
byte1->Stamina and byte2->Sharp are ordinal + sole-grower inferences (MEDIUM).
Bytes 3..5 grow real params that ROM does not label as core race stats; they are
left unnamed deliberately.

## Confidence summary
| Stat | Name | Care field | Race read | Writer | Cap | Overall |
|---|---|---|---|---|---|---|
| Speed   | proven | +0x68 (0x0C21A610) | +40 (0x0C05C7D8) + physics 0x0C064C6E | 0x0C036A94 PROVEN | 63 PROVEN | HIGH |
| Stamina | proven | +0x6A (0x0C21A612) | +41 (0x0C05C7DC) | dispatch path, PARTIAL | 63 PROVEN | HIGH |
| Sharp   | proven | +0x6C (0x0C21A614) slot; acc +0x48 | +42 (0x0C05C7E6) | NOT LOCATED | inferred 63 | MIXED (reader-strong, writer-partial) |

## Open threads (runtime / Flycast)
1. Sharp training commit: which instruction writes the Sharp stat byte from
   accumulator care_state+0x48, and its cap. Static search bottomed out; resolve
   live (feed WATERMELON, watch 0x0C21D700 and record +0x64/+0x6C).
2. Pre-race seeding: the routine that populates race DATA +40/+41/+42 from the
   player record before a race (runs through converters 0x0C0772C4 / 0x0C0778F0 /
   0x0C0815D8). Trace live to confirm record+0x68 -> DATA+40 actually happens.
3. Bytes 3..5 of the food payload and the matching accumulators +0x4C/+0x50 grow
   unlabeled params; identify what they drive (not core race stats per the label
   table).
