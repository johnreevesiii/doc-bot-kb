# DOC Race-Program Battery Resume — Investigation (PARKED, May 31 2026)

## Goal
On real DOC cabinets a 2032 battery makes the machine resume at **1R of whatever race-program
section it was in** at power-off (dead battery -> always boots to the first section, "Winter Stakes").
In our Flycast setup every boot resets to Winter Stakes. Wanted: make Flycast resume the section.

## What it IS (confirmed by the owner, who ran real cabinets)
- A **saved program/pace counter** in NAOMI battery memory. Boot -> 1R of the saved section.
- Advances on its own over runtime ("time as in pace"), whether or not you play.
- Dead battery on real hw == our symptom (always Winter Stakes).

## Ruled out: the RTC / clock
- `aica_if.cpp GetRTC_now()`: with `GGPO=no` (our config) the RTC seeds from the **host clock**,
  ticks +1/s, and is NOT persisted to a file (only inside savestates).
- Test: planted a `.rtc` set **+30 days** (2026-06-30) -> booted **still Winter Stakes**. Owner: changing
  the clock does NOT fast-forward the game (pace is its own counter, not wall-clock). => RTC is not the lever.

## RTC patch (built, but NOT the fix — kept for reference)
- Fork **johnreevesiii/flycast**, branch **doc-rtc-battery**, from exact dev commit **1370b74fb**
  (matches the owner's working build). Patch: `aica_if.cpp` persist/restore `RealTimeClock` to
  `<rom>.rtc`; `nvmem.cpp saveFiles()` calls `aica::saveRtc()`; guarded to arcade master.
- CI: use artifact **flycast-x86_64-w64-mingw32** (MinGW RelWithDebInfo, ~20.7MB, the *published* Windows
  build). The **flycast-x86_64-pc-windows-msvc** artifact (14.4MB) is a CI-only variant and **HANGS DOC
  after race 1** — do not use it. Local src `C:\dev\flycast-rtc`, build dl `C:\dev\flycast-rtc-build`.

## Flycast DOES persist the full NAOMI battery (proven)
- **BBSRAM** 32KB -> `<rom>.nvmem` (saved + loaded; files present and changing).
- **JVS EEPROM** 128B -> `<rom>.eeprom` in `maple_jvs.cpp` (loads on boot if present, saves whenever the
  game writes EEPROM). **No `.eeprom` file exists** => DOC never writes the EEPROM => counter not there.

## Clean controlled test (drbyocwc multiboard) — DECISIVE
- Advanced Winter Stakes -> **Sprinters Trophy** section, clean-closed. Battery (master `-main.nvmem`)
  changed **91 bytes** vs baseline (CRC @0x1f8, header @0x208, record data @0xdcc; 0xdcc holds ASCII
  race records e.g. "Kowloon"/"Blockout"/"Dock of"). So the section/records DID save.
- Relaunch -> **back to Winter Stakes; records also reset.**
- Post-boot battery (C) differs from both baseline (A) and Sprinters (B): A/B/C all distinct shas.
  => battery is actively saved AND loaded every session, yet the master comes up fresh.
- Snapshots: `_jp_re/battery_test/clean/{A_main_baseline,B_main_sprinters,C_main_postboot}.nvmem`.

## CONCLUSION
NOT a persistence gap. Flycast saves+loads the complete battery (BBSRAM + EEPROM) correctly, proven
three ways. The **multiboard MASTER re-initializes its game state (program section + records) on every
boot** — almost certainly during the network/link handshake that starts a fresh race program for the
satellites. Real linked cabinets read their position from the battery and resume; Flycast's master starts
fresh. DOC requires multiboard (>=4 sats guard), so a single-instance control isn't possible.

## To resume later (deep, uncertain)
Reverse-engineer WHY the master resets the program at boot and intercept it:
- Flycast naomi multiboard master init / m3comm reset path (`core/hw/naomi/`), and `naomi_Reset`.
- How DOC reads its program/pace counter from BBSRAM at boot vs. the master wiping/recomputing it.
- Likely needs DOC SH-4 RE + several build/test cycles. Fork/branch + local src above are ready.

## Status: PARKED. Install reverted to the official binary (20,669,952 B); orphan .rtc files removed.
The card system + Stable Management System are done/shipped; this is a Flycast multiboard limitation,
not a regression (the stock binary behaves identically).
