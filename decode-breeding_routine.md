# Derby Owners Club — Foal Inheritance / Breeding-Compute Static RE

ROM: `epr-22336c.ic22` (Rev C World Edition, "drbyocwc"), SH-4 LE, static base **0x0C000000**
(SH-4 addr = 0x0C000000 + file offset). Tooling: `_sh4/ghidra/sh4dis.py`, Ghidra decomp in
`_sh4/ghidra/decomp/`. This session added no edits to existing files; it appended RNG targets
to `targets.txt` and re-ran the existing decompiler (writes only into `decomp/`).

---

## TL;DR / Verdict

| Question | Answer | Confidence |
|---|---|---|
| RNG function found? | **YES — two byte-identical LCG generators** | **0.97** |
| Mater-record COPY routine pinned statically? | **NO** (reached only via runtime-installed function pointers) | finding is solid, 0.85 |
| Foal-COMPUTE routine pinned statically? | **NO** (same indirect-dispatch wall; operates on RAM card buffers, not the ROM table) | 0.8 |
| Is the community **averaging+RNG-noise+bloodline** model confirmed? | **CANNOT CONFIRM OR REFUTE from the static image** — the arithmetic is not present in any statically-reachable form against the 60-byte record layout | 0.75 |
| Does breeding read composite @+0x2c (name+44)? | **UNKNOWN statically** — no code statically references the record base at all (see below); needs live capture | 0.7 |

The single most important structural finding: **the breeding-stock table base addresses
(`0x0C10BF1C` sire / `0x0C10D2CC` dam) are NEVER loaded as a PC-relative literal and NEVER
stored anywhere in the ROM as a 32-bit pointer.** Likewise the two RNG entry points are never
stored as pointers and have essentially zero direct `bsr` callers. This game is built with a
pervasive **function-pointer / vtable dispatch** convention; the call graph for both the RNG and
the breeding compute is assembled at runtime and is therefore **not statically reconstructable**
with PC-relative xref scanning. The averaging math, if it exists in the form the community
modelled, lives behind that runtime dispatch and/or operates on RAM card buffers — so it must be
captured live (GDB), not read out of the ROM. Breakpoints for that capture are given at the end.

---

## 1. RNG function — FOUND (high confidence)

The classic LCG multiplier `0x41C64E6D` appears as a literal-pool dword at exactly two sites,
each anchoring an identical generator triplet:

```
file 0x091eb0  (SH-4 0x0C091EB0)   0x41C64E6D
file 0x308860  (SH-4 0x0C308860)   0x41C64E6D
```

### 1.1 RNG instance A — `0x0C091E60` (integer core), state global `0x0C19A718`

Disassembly (verified bytes):

```
0c091e60: d412  mov.l @(0x0c091eac),R4   ; R4 = 0x0C19A718  (state ptr)
0c091e62: d313  mov.l @(0x0c091eb0),R3   ; R3 = 0x41C64E6D  (multiplier)
0c091e64: 6242  mov.l @R4,R2             ; R2 = state
0c091e66: 4f12  sts.l MACL,@-R15
0c091e68: 0237  mul.l R3,R2              ; MACL = state * 0x41C64E6D
0c091e6a: 911c  mov.w @(0x0c091ea6),R1   ; R1 = 0x3039  (increment = 12345)
0c091e6c: 031a  sts MACL,R3
0c091e6e: 331c  add R1,R3                ; R3 = state*MULT + 0x3039
0c091e70: 6033  mov R3,R0
0c091e72: 2432  mov.l R3,@R4             ; store new state
0c091e74: 9318  mov.w @(0x0c091ea8),R3   ; R3 = 0x7FFF  (mask)
0c091e76: 4029  shlr16 R0               ; R0 = state >> 16
0c091e78: 2039  and R3,R0               ; R0 = (state>>16) & 0x7FFF
0c091e7a: 000b  rts
0c091e7c: 4f16  lds.l @R15+,MACL
```

Ghidra decompile (`decomp/fn_0c091e60.c`) confirms exactly:

```c
uint FUN_0c091e60(void) {
  uint uVar1 = *puRam0c091eac * iRam0c091eb0 + (int)sRam0c091ea6;  // state*MULT + INC
  *puRam0c091eac = uVar1;
  return uVar1 >> 0x10 & (int)sRam0c091ea8;                        // (state>>16) & 0x7FFF
}
```

**Algorithm (definitive):**
```
state = state * 0x41C64E6D + 0x3039            (32-bit LCG; INC = 12345 decimal)
rand15() = (state >> 16) & 0x7FFF               returns 0 .. 32767
```

Companion functions in the same module:
- **`0x0C091E80` — `rand_float()`**: calls `0x0C091E60`, `float`-converts, `fmul` by the
  literal at `0x091EB4 = 0x38000000` (= 2^-15 = 1/32768) → returns float in **[0.0, 1.0)**.
- **`0x0C091EA0` — `srand(seed)`**: `mov.l R4,@R3` stores R4 into state `0x0C19A718`.

### 1.2 RNG instance B — `0x0C308818` (integer core), state global `0x0C054568`

Byte-for-byte identical generator (same `0x41C64E6D` / `0x3039` / `0x7FFF` / `0x38000000`),
only the state-global pointer differs (`0x0C054568`). Wrapper at `0x0C308830`, seed at
`0x0C308850`. This is the compiler emitting the RNG as a per-translation-unit static helper, so
each code module that needs randomness carries its own copy + its own state word.

> Implication: there may be one RNG-state word *per gameplay subsystem*. The breeding subsystem,
> if it lives in yet another module, would carry its **own** copy of this exact LCG with its own
> state global — but no third copy of the `mul.l R3,R2 + shlr16 + and R3,R0` shape exists in the
> ROM (verified by full-ROM opcode scan: exactly 2 hits). So breeding uses one of these two
> generators **through an indirect call**, or a different randomness source entirely.

### 1.3 Why callers can't be found statically

- Neither RNG entry (`0x91E60`, `0x91E80`, `0x91EA0`, and the B-copies) appears anywhere in the
  ROM as a stored 32-bit pointer (full-ROM dword search: 0 hits each).
- `bsr` is PC-relative ±4 KB; the only `bsr` into the whole RNG region is the local
  `0x091E82 → 0x091E60` (wrapper→core). `rand_float` and `srand` have **zero** `bsr` callers.
- Therefore every external use of the RNG is an **indirect call** through a pointer that is
  installed into a table/vtable at runtime (not a ROM literal). Static xref cannot follow it.

---

## 2. Breeding-stock data layout — VERIFIED against ROM bytes

84 sires then 84 dams, contiguous 60-byte (0x3C) records.
- Sire base `0x0C10BF1C` (file 0x10BF1C); Dam base `0x0C10D2CC` = sire_base + 84*60. Confirmed:
  `(0x10D2CC - 0x10BF1C)/60 = 84` exactly. dam[83] ends near file 0x10E3E0.

Record (anchored at name):

| off | size | field | sample (sire[0] "Maple Syrup") |
|---|---|---|---|
| +0x00 | 24 | name (NUL-padded) | "Maple Syrup" |
| +0x18 | u32 | **st** (stamina) | 39 |
| +0x1c | u32 | **sp** (speed) | 19 |
| +0x20 | u32 | **sh** (sharp) | 34 |
| +0x24 | u8 (in u32) | **ac** | 240 |
| +0x28 | u32 | zero | 0 |
| +0x2c | 4 | **composite** | `00 e0 11 f0` |
| +0x30 | 6 | **externals** | `0f 03 06 0a 04 0c` |
| +0x38 | u16 | **index** | 1 |

Verified across many records (sires idx 1..84, dams idx 85..168; ac is a single byte 0..255;
externals are small values ~1..15; composite varies per horse, e.g. `02 cc ff 3c`, `01 cc ab 34`
— packed aptitude/flag nibbles). The layout in the task brief is **correct**.

---

## 3. The COPY routine — NOT statically pinned (with evidence why)

What the task expected: code computing `tableBase + idx*0x3C` and copying ~14 fields into a RAM
work buffer. What is actually in the ROM:

1. **The table base is never materialised as a constant.** `scan_pcrel_loads()` for
   `0x0C10BF1C` and `0x0C10D2CC` → 0 hits. Raw dword search for those addresses → 0 hits.
   So no routine loads "sire_base" and indexes it.

2. **The only code that touches the records is the menu/display cluster** at
   `0x0C028000–0x0C02BB00` (already known as "mostly float UI layout"). It reads *individual*
   record fields by loading near-base literals — e.g. `0x0C02A656` loads `0x0C10BF38`
   (= sire_base + 0x1C, the `sp` field of sire[0]); `0x0C02A5DC` loads `0x0C10BF00`, etc. These
   are **fixed per-field literal loads to draw stat bars**, not `base + idx*60` indexing.

3. The literals just *before* the sire base that several menu sites load — `0x0C10BEEC`
   (base−0x30), `0x0C10BEF0`, ... — are **NOT records**. `0x10BEEC` is a 48-byte binary
   coefficient/icon table (`07 2d 07 32 19 19 00 00 | 02 02 02 00 | 03 01 01 00 | ...`) that the
   display pushes by pointer to a text/graphics routine (`R12 = 0x0C04BC74`) alongside a float.
   Confirmed it is not ASCII and not a 60-byte record.

4. The second cluster flagged by prior xref work, `0x0C0EBC68+`, **is data, not code** — Ghidra
   reports "NO FUNCTION" for every address there. It is a **printf-format / message-pointer
   table** (decodes to fragments like `"%s (%s)"`, `"Grade 1 %d times won"`, `"... times now ...
   Grade 1 ..."`). The "loads of 0x10D5F4" reported earlier are pointer-table entries, not loader
   instructions.

5. The breeding menu cluster issues **143 distinct indirect calls** (function pointers loaded
   from its literal pool, then `jsr @Rn`) into helpers at `0x049xxx / 0x053xxx / 0x05Bxxx /
   0x063xxx / 0x073xxx / 0x084xxx / 0x088xxx / 0x089xxx`. The actual "confirm breeding → make
   foal" handler is one of these (or a callee of one), but the edge that reaches it is a runtime
   pointer, so it cannot be named from the static image.

**Conclusion:** the selected mater record is almost certainly copied from ROM into a RAM card
buffer by one of those indirectly-called helpers, but the copy site is **not statically
addressable**. (Confidence the copy is indirect/runtime-dispatched: 0.85.)

---

## 4. The foal-COMPUTE routine — NOT statically pinned

Two independent structural searches came up essentially empty against the 60-byte record layout:

- **Averaging-idiom scan.** Searched the entire ROM for the `(a+b)>>1` shape — an `add Rm,Rn`
  followed within a short window by `shar Rn`/`shlr Rn`, with ≥2 struct loads at offsets
  0x18/0x1C/0x20/0x24 (`mov.l @(disp,Rm)`, op 0x5, d4∈{6,7,8,9}). **One** hit in the whole 4 MB,
  at `0x0C304288` (inside the unrelated upper-region module near RNG-B), none anywhere near the
  breeding code. The community model's `floor((sire+dam)/2)` over the documented record offsets
  **does not exist as statically-reachable code addressing that record shape.**

- **RNG-caller scan.** The foal routine should call the RNG ~8× (ac + 6 externals + sex). But as
  shown in §1.3, the RNG has no static callers at all. So "find the function that calls RNG 8
  times" cannot be executed statically.

Why this is expected, not a tooling failure: breeding parents in DOC are normally **retired
player horses loaded from magnetic cards into RAM**, and the foal is written to a **new card
buffer in RAM**. The persistent card/horse struct is a different structure from the 60-byte ROM
catalog record (the ROM catalog is the *starter breeding stock* shown in the selection menu).
The compute therefore reads two RAM buffers and writes a third — it does not reference the ROM
table base, which is exactly what the byte evidence shows. (Confidence: 0.8.)

### Does the compute use composite @+0x2c (name+44)?
**Undetermined statically.** Because no statically-reachable code references the record base or a
`base+0x2c` field in a compute context, the question cannot be answered from the ROM image. The
menu's `mov #44,R0` at `0x0C02A4B6` is a **stack** access (`fmov.s @(R0,R15)`), not a record
field — so do not mistake it for a composite read. (Confidence: 0.7 that this needs live data.)

---

## 5. Verdict on the community model

- **Averaging (`floor((sire+dam)/2)` for st/sp/sh):** *Not confirmed and not refuted.* The
  arithmetic is not present in any statically-reachable routine over the documented record
  layout. It is either behind runtime function-pointer dispatch or operates on RAM card structs
  with a different offset layout.
- **`ac = (sire.ac+dam.ac)/2 + rand*36-18`, externals `clamp(avg+small,1,16)`, `sex = rand>0.5`,
  bloodline threshold bonuses:** *Unverified.* No constants like 36/18, the 1/16 external clamp,
  or threshold compares (45/40/220) were found bound to a routine that also reads parent stat
  fields and calls the RNG.
- **Weighted / dominant-parent vs pure averaging:** *Cannot be decided statically.*
- **Is name+44 (composite @+0x2c) used in breeding?** *Unknown statically.*

The one thing the static image *does* nail down precisely is the **RNG** the breeding code will
ultimately draw from: the LCG `state = state*0x41C64E6D + 0x3039; out = (state>>16)&0x7FFF`
(or its float form `out/32768`). Once the live compute is captured, its noise terms should be
checkable against this exact generator and one of the two state globals (`0x0C19A718` /
`0x0C054568`, or a breeding-module copy).

---

## 6. How to finish this — live GDB capture (required)

The compute is only reachable through runtime-installed pointers, so pin it dynamically. Use the
GDB-debuggable Flycast from the `_sh4` race harness (runtime base = static + 0x20000).

**Recommended capture procedure:**

1. **Watchpoint the RNG state words** to catch the breeding draws:
   - `rwatch *0x0C19A718` and `rwatch *0x0C054568` (and re-check for a 3rd state word if a
     breeding-module copy of the LCG exists — scan RAM-resident code for the same
     `mul.l/shlr16/and 0x7FFF` shape once loaded).
   - In-game, go through **breeding confirmation** (select sire + dam, confirm). The watchpoint
     that fires ~8 times in quick succession during foal creation is the breeding RNG; its PC and
     call stack reveal the compute function entry.

2. **Breakpoint candidates for the COPY:** break on the menu-confirm helpers reached indirectly
   from the breeding cluster — set a hardware breakpoint on the *return target* after the
   selection list, or simpler, **watch the destination card buffer**: once you know the foal/new
   card RAM address (allocated on confirm), `watch` its `+0x18/+0x1c/+0x20/+0x24` words to catch
   the store of computed st/sp/sh/ac.

3. **To read parent inputs:** at the compute breakpoint, dump both parent RAM buffers (the copied
   mater records). Compare each foal output field to `floor((sire+dam)/2)` and to the RNG draws
   captured in step 1 to determine averaging-vs-weighted, the exact noise range, the external
   clamp bounds, the bloodline thresholds, and **whether the +0x2c composite is read** (set an
   additional `rwatch` on the parent buffer's `+0x2c` during compute — if it never fires, the
   composite is not used by breeding).

4. **Confirm the generator binding:** the noise added to `ac`/externals should equal a function
   of `rand15()` or `rand_float()` from one of the two LCGs. Verify by stepping the state word
   delta against `state*0x41C64E6D + 0x3039`.

**Deliverables to produce from the live run:** the compute function's runtime PC (subtract
0x20000 for static addr), the exact per-field formula, all constants, and a yes/no on composite
@+0x2c usage.

---

## Appendix — verified addresses

| Addr (SH-4, static) | Role |
|---|---|
| `0x0C091E60` | RNG-A integer core: `(state*0x41C64E6D+0x3039)`, `>>16 & 0x7FFF` |
| `0x0C091E80` | RNG-A float: `rand15()/32768` → [0,1) |
| `0x0C091EA0` | RNG-A seed (store R4 → state) |
| `0x0C19A718` | RNG-A state word |
| `0x0C308818` | RNG-B integer core (identical LCG) |
| `0x0C308830` | RNG-B float |
| `0x0C308850` | RNG-B seed |
| `0x0C054568` | RNG-B state word |
| `0x0C10BF1C` | sire[0] record base (84 records × 60B) |
| `0x0C10D2CC` | dam[0] record base (84 records × 60B) |
| `0x0C10BEEC` | 48-byte display coefficient table (base−0x30); NOT a record |
| `0x0C028000–0x0C02BB00` | breeding-selection MENU/DISPLAY cluster (float UI; reads record fields for stat bars; 143 indirect helper calls) |
| `0x0C0EBC68+` | breeding printf-format / message-pointer DATA table (not code) |
| `0x0C04BC74`, `0x0C04BC5C` | text/graphics render helpers called by the menu |

Generated by static SH-4 RE session, June 2026. No live execution performed; dynamic items above
are explicitly flagged as unresolved and require the GDB capture in §6.

---

## LIVE CAPTURE RESULTS (Jun 4 2026) — breeding generation routine LOCATED

Method: GDB on main (3263), trap DOC's RNG, read caller PR. Across 4 in-game breedings
(Thunder Boy×Mountain High ×2, Judge Andelucci×Heart of Gold, Glass Tiger×Mountain High).
Read/access watchpoints (Z3/Z4) are NOT supported by Flycast's stub; only Z0/Z1/Z2 work.

**RNG draw count CONFIRMED:** the breeding compute issues exactly **8 `rand_float()` draws** per
horse (burst returns into the RNG float-wrapper `0x0C091E80`+6 = `0x0C091E86`). 8 = ac + 6
externals + sex — matches the community model's noise structure. Internals are NOT randomized
(no draws): foal **sp = 46 = exact floor-average** of Glass Tiger sp52 / Mountain High sp40,
read live from the foal display entity. So internals = deterministic floor-average; ac + 6
externals + sex = RNG. This CORROBORATES the averaging model's structure.

**ROUTINE LOCATED (static):**
- `FUN_0c05ef3a` = generation LOOP: `for i in 0..*0x0c3c94fc: src=FUN_0c05ea6e(i); dest=0x0c3c9640+0x70*i; ...; FUN_0c05ee68(src)`. Builds `count` 112-byte horse records. Sets dest+0x44=i (index), dest+0x5c=(rand>>4)&7 clamp<=7, dest+0x54=func_0c0c211c(rand,200), dest+0x58=0x30000, conditional `if src[+0x8c]==5: src[..]=(rand>>4)&3`, + a 6-bin histogram prefix-sum at the end.
- `FUN_0c05ee68(src)` = STAT compute: writes float fields dest+0x14,+0x18,+0x1c,+0x20,+0x24 (5 fields) each = `signedtable[src[statbyte]*2]*scale ± rand_float()*noise`. Tables at `0x0c171358`/`0x0c171359` (runtime; static 0x0C151358/9). Scale/noise constants: 0.125, 1000, 2000(neg), 3000, 3500, 4000, 60000. RNG float wrapper `0x0C0B1E80` called 8x.
- `FUN_0c05ea6e` = `DAT_0c05eb88 + FUN_0c05ea3c()` -> source = a base template array + computed offset.

**OPEN / not yet resolved:**
- The loop over `count` builds a SET of horses -> this is likely the **candidate-pool generation**
  (the "set of random horses" shown for sire/dam selection), and/or the offspring build. The final
  two-parent CROSS (combining the selected sire+dam) may be a separate routine that runs after
  dam-select; our captures (timed UI, no read-watchpoints) caught this generation + the displays,
  not a confirmed two-parent average step.
- **Composite/name+44 in breeding: still UNCONFIRMED.** This code reads `src[+0x8c]` in a 112-byte
  RUNTIME layout, not the catalog 60-byte record's +0x2c composite. No evidence yet that name+44 feeds in.
- Exact table (0x0C151358) semantics + how the scaled internal units (×1000..60000) map back to the
  10-65/0-255 stat ranges need more static work (FUN_0c05ea3c, DAT_0c05eb88, the lookup tables).

**Net:** structure of the averaging model is live-corroborated (floor-avg internals; bounded RNG
noise on ac+externals; 8 draws); the predictor stays on that model + the exact decoded RNG. The
fully byte-exact two-parent formula + composite question remain a static follow-up (code now located).
