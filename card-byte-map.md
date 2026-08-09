# The Derby Owners Club Player Card — Corrected Byte Map

**What this is:** the authoritative, byte-verified map of the 207-byte US / World-Edition player
`.card`, with (a) what each byte *does in the game*, and (b) **where older community maps were
wrong** — including the mistakes that have caused real, unintended edits. Verified against real
cards by the `doc_card.py` codec (round-trips byte-exact, selftest 11/11). Cross-references the
gameplay effects in `HIDDEN_RULES.md` and the layout rules in `ROM_ARCHITECTURE.md`.

---

## Why this document exists (a cautionary tale)

A longtime community card editor — call him **JJ** — modified cards and sold them. He'd used an
early card-reader/editor whose field map was incomplete. When he moved to a more thorough tool and
changed *the value he'd always changed*, he **unknowingly altered the horse's personality** — because
the byte he was used to editing sat right next to, or was mislabeled as, the personality byte in the
old map. He wasn't careless; **the map was wrong.**

That's the whole problem: the community has been editing off a *moderate* understanding of the card,
and a wrong offset doesn't error — it just silently changes the wrong thing. We made the same
mistakes early on; this session we corrected them against the real bytes. This document publishes the
corrected map so nobody else ships a horse with a stat or personality they didn't mean to touch.

> **We audited our OWN editor and found the exact culprit — and fixed it.** The Stable Management
> System (DOC-Card-Creator) had the offsets *right* (personality at a1[6]=0x3F, hood at a2[26]=0x70,
> correct track-reversal), but its personality control was a **lossy 5-choice dropdown** that wrote
> only `{0,48,64,80,208}`. So **re-saving any card whose personality byte wasn't one of those five
> silently rewrote it** — 100→80, 128/180/200→64, 244→208 — *even if you never touched personality.*
> That is exactly what changed JJ's horse. **Fixed (Jun 2026):** the editor now preserves the exact
> 0–255 personality byte on round-trip and only changes it when you deliberately pick a different
> personality. The byte-exact `doc_card.py` codec was always correct (it preserves the raw byte);
> the HTML editor was the one carrying the lossy step.

---

## The one rule that breaks every naive map: tracks are stored REVERSED

The card is **207 bytes = 3 "tracks" of 69 (0x45)**. Within a track, the logical bytes are stored
**back-to-front**: logical `aN[k]` (k = 1..69) lives at **file offset `t*69 + (69 − k)`** (t = 0,1,2).

- Names therefore read "forward" out of a region that looks backwards in a raw hex view.
- **If a tool addresses the card by raw file offset without undoing the reversal, every field after
  the first is shifted/mirrored** — the root cause of most historical mismaps.
- There is **NO whole-card checksum** in the 207-byte payload. Any byte can be edited and re-saved
  and the cabinet accepts it (integrity lives in the physical `.raw` layer, not here). Good news for
  editors; also why a wrong edit silently "sticks."

---

## The danger cluster — file 0x3C–0x3F (this is the JJ trap)

Four **adjacent** bytes on track 0, all cosmetic-looking, one of which is **not** cosmetic:

| file | logical | field | byte-exact? | gameplay effect |
|---|---|---|---|---|
| 0x3C | a1[9] | Coat **modifier** | yes | cosmetic (coat variant, only when base=63) |
| 0x3D | a1[8] | Coat **base** | yes | cosmetic (coat color) |
| 0x3E | a1[7] | **Run-style seed** | yes | **creation seed only** — does NOT reliably set the displayed running style; leg-type is derived from the externals at race time. Editing it for "running style" is a known false lever. |
| **0x3F** | **a1[6]** | **PERSONALITY (0–255)** | yes | **drives the post-race bond multiplier** (see Hidden Rules): the *right* response per personality builds the bond ×2.0, the *wrong* one **lowers** it. Change this and the horse reacts differently to you forever. |

> **Correction:** personality is a single **0–255** byte at **0x3F**. Many tools expose it as a
> 5-choice picker (R/I/C/H/S) that writes only `{0,48,64,80,208}` — a **lossy** simplification of the
> real 8-band scale. Editing "running style" at 0x3E or a coat byte and landing on 0x3F (or using a
> tool that mislabeled this cluster) is how a personality gets changed by accident.

---

## The other documented wrong→right corrections

| field | OLD / community understanding | CORRECTED (byte-verified) | why it matters |
|---|---|---|---|
| **Hood** | "0x73" | **0x70** (a2[26]) | **0x73 is retirement SHARP**, a real stat. Editing "hood" at 0x73 off an old map silently nerfs/boosts a stat. |
| **Personality** | 5 buckets {R,I,C,H,S} | raw **0–255**, 8 bands @0x3F | the 5-bucket write is lossy and collapses real variety; band breakpoints affect bonding |
| **Run-style (a1[7] @0x3E)** | "the running style" | **creation seed only**; display leg-type = derived from externals | editing it to change style usually does nothing visible |
| **Track storage** | raw forward offsets | **reversed per track** (`t*69+(69−k)`) | the master cause of shifted maps |
| **Checksum** | "don't edit, it'll corrupt" | **no whole-card checksum** | cards are freely editable; bad edits don't get caught |
| **Personality on CPU horses** | "horses have a personality byte" | **CPU racing horses do NOT store personality**; it's player-card-only | don't look for it in the roster table |

---

## Full corrected field map (file offsets)

> Reversed-track caveat applies — these are the **final file offsets** (already un-reversed), so a
> hex editor can use them directly. Stride between tracks = 0x45.

| file offset | field | bytes | gameplay effect | conf |
|---|---|---|---|---|
| 0x00–0x12 | **Horse name** | 18 ASCII (reversed) | cosmetic / identity | exact |
| 0x14–0x26 | **Sire name** | 18 ASCII | pedigree (breeding lineage) | exact |
| 0x28–0x3A | **Dam name** | 18 ASCII | pedigree | exact |
| 0x3C–0x3D | **Coat** (mod/base) | 2 | cosmetic | exact |
| 0x3E | **Run-style seed** | 1 | creation seed only (see above) | exact |
| **0x3F** | **Personality** | 1 (0–255) | **post-race bonding multiplier** | exact (effect formula exact; response-names inferred) |
| 0x40–0x43 | **UID / horse id** | 4 (mirrored on all 3 tracks) | identity / lineage links | exact |
| 0x45–0x4D | **Internals** stamina/speed/sharp | 3 (cap 60) | core racing stats; **externals matter more for base ability** | exact |
| 0x51–0x53 | **G1 titles** | 3 (bitfield) | career titles | exact |
| 0x55–0x57 | **Earnings** | 3 | prize money total | exact |
| 0x59–0x67 | **Race record** total/won/place/show/out (+hearts @0x65) | 5 (+1) | career stats; hearts = stamina hearts | exact |
| 0x5F–0x64 | **Externals (current)** start/corner/oob/competing/tenacious/spurt | 6 (stored display−1) | **the dominant driver of race ability**; also derive the displayed leg type | exact |
| 0x69–0x6E | **Retirement externals** | 6 | breeding-stock value | exact |
| **0x70** | **Hood** | 1 (0–63) | cosmetic (NOT 0x73) | exact |
| 0x71–0x73 | **Retirement internals** stamina/speed/sharp | 3 (cap 45) | breeding-stock value (**0x73 = ret. sharp, not hood**) | exact |
| 0x7A | **Sex** | 1 (0 M / 1 F / 2 Gelding) | breeding role | exact |
| 0x7B–0x7D | **Silk** pattern/color1/color2 | 3 | cosmetic (color1=0x7C, color2=0x7D) | exact |
| 0x8A–0x91 | **`SEGABEF0` marker** | 8 ASCII | card-type signature (US/WE) | exact |
| 0x92 | **Dirt aptitude** | 1 (0–255) | dirt vs turf suitability (banded; see Hidden Rules) | exact |
| 0x96 / 0x9A | **Retired flag / breeds** | 1 / 1 | retirement state, breed count | exact |

---

## How to use this safely (for editors / the community)

1. **Edit by these file offsets**, or use `doc_card.py` / the Stable Management System, which apply
   the reversal for you. Don't trust a raw-offset edit from an older tool.
2. **Treat 0x3C–0x3F as a single careful zone.** If you only meant to change a coat, you're one byte
   from personality.
3. **Hood is 0x70.** If your old workflow said 0x73, you've been editing retirement sharp.
4. There's no checksum to "protect" you — verify your edit by decoding the card back (round-trip).

*Provenance:* `doc_card.py` (codec, byte-exact round-trip), `_core/areas/us-card.md` +
`us-card.VERIFY.md` (the corrections: hood 0x70 not 0x73; 0x3C–0x3F cluster a1[9..6]; lossy 5-bucket
personality; reversed-track rule; no whole-card checksum). Gameplay effects → `HIDDEN_RULES.md`.
