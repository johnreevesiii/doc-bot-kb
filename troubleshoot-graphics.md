# Troubleshooting: "my video / graphics aren't showing in the lobby" (host cabinet)

_Written 2026-07-17 after the 0.7.2 pack rollout. Companion to `TROUBLESHOOTING-cant-seat-players.md`
(that one is the seat-claim path; this one is the video/capture path). Keep both current._

## Rule #0 — name the exact symptom first (each has a DIFFERENT fix)
"Graphics aren't showing" is four different failures. Ask the host which one it is before doing anything:

| Symptom | What it means | Go to |
|---|---|---|
| Cabinet **not in the lobby at all** | Not approved, or "2 - Start Streaming" never ran | §1 |
| Tile shows, **video black**, but the game IS running on the box | Boot race: Step 2 started before the Flycast windows finished loading | §2 |
| **Sound but NO video** — black on the site AND black locally, every seat | The video ENCODER (not the network) | §3 ← the 0.7.2 hot one |
| **No video to SOME viewers only** (others see it fine) | A network relay (NAT/TURN) path issue for those viewers | §4 |

> The version gate is NOT the cause here. The lobby floor stayed at **0.7.1**, so 0.7.1 and 0.7.2 cabinets
> both pass. A 426/"Update required" only ever hits a cabinet below 0.7.1. This doc is purely the capture path.

## §1 — Cabinet absent from the lobby
- Must be **approved** (operator approves via the NTFY push) AND **streaming**. Approval alone shows nothing —
  the host still has to run "1 - Start Cabinet" then "2 - Start Streaming" on the box before it heartbeats.
- Check furlong: `status='live'` + a fresh `last_heartbeat`. `version=null` + no heartbeat = it has never come
  online (host hasn't actually started it, or it died before the first beat).

## §2 — Tile black, game running locally → boot race
- The boards were still booting when "2 - Start Streaming" ran, so the capture grabbed a blank frame.
- **Fix: just re-run "2 - Start Streaming"** once every Flycast window shows the game. Fixes most of these.

## §3 — Sound but no video everywhere → the ENCODER escape hatch  ★ the 0.7.2 fix
This is the "like last time" case (first hit 2026-07 with **arglifed's RTX 5080**). Root cause: a **brand-new
GPU whose NVENC output the bundled ffmpeg/browsers cannot decode.** NVENC runs without error and produces a
stream, so audio is fine but the picture is black, on the site AND locally. Because nothing errors, **0.7.2's
automatic software fallback does NOT trigger** (its `_enc_ok()` probe only checks that the encoder *runs*, not
that the browser can *decode* it). So the host must force software encoding by hand:

**Fix (either one, then re-run "2 - Start Streaming"):**
1. Drop an **empty file named `use-software-video.txt`** into the pack folder (same folder as "2 - Start
   Streaming.ps1"). This is the easy one to tell a host.
2. Or set the env var `DOC_VIDEO_ENCODER=libx264` (can also pin `nvenc`/`qsv`/`amf` explicitly).

`stream_window_wgc.py` reads both at `_pick_encoder()` and returns software libx264 (browser-safe). Confirmed
present in the shipped 0.7.2 zip.

**Confirm the cause before assuming encoder:** have the host open **`stream_mainl.log`** in the pack folder —
its first lines name the exact failure (encoder error / capture error / window not found). "Encoder-ran-but-
undecodable" is the too-new-GPU case → the marker fix. A *headless / no-display* signature → §3b.

### §3b — headless box (no real display) also gives sound-but-no-video
Windows with no monitor falls back to a software display adapter → the capture gets audio but no frames.
- **Fix:** attach a monitor, or a **$7 HDMI dummy plug** (fools Windows into a real 1080p60 GPU display).
- **Do NOT host over Remote Desktop** — disconnecting the RDP session tears down the GPU desktop the capture
  needs. Use a game-streaming remote (Parsec) that keeps a real console session alive.
- Background in the pack: HOST-GUIDE "Hosting without a monitor" + `PERF-cpu-affinity-and-frame-pacing.md`.

## §4 — no video to SOME viewers only → relay/TURN
- If most viewers see the cabinet fine but specific ones get black/offline, it's their NAT/TURN path, not the
  host. Not a capture problem. (Distinct from §3, which is black for EVERYONE including the host locally —
  because audio and video ride one WebRTC transport, so "audio yes / video black for all" is always a codec
  problem, never relay.)

## Fast decision tree
1. In the lobby at all? No → §1.
2. Black but game running locally? → re-run Step 2 (§2). Fixed? Done.
3. Still sound-but-no-video for everyone? → open `stream_mainl.log`:
   - encoder ran but browser can't decode / brand-new GPU → **`use-software-video.txt` marker** (§3).
   - no real display / headless → **HDMI dummy plug**, stop RDP (§3b).
4. Only some viewers black? → their relay/TURN path (§4), not the host.

## Known hosts / notes
- **arglifed** (Lon Lon Circuit + Las Vegas Downs): **RTX 5080** → needs the `use-software-video.txt` marker
  on every fresh pack extract (the marker does not survive a fresh extract into a new folder — re-add it).
- The DOC Bot KB (`doc-bot-kb.txt` / `/opt/doc-bot/doc_kb.txt`) now carries the §3 encoder fix, so a host who
  @mentions the bot with "audio but no video" gets pointed straight at the marker.

See also: `cabinet-pack/HOST-GUIDE.md` (Troubleshooting + headless), `cabinet-pack/PERF-cpu-affinity-and-frame-pacing.md`,
`COMMUNITY-GUIDE-self-healing-cabinet-stream.md`, and `TROUBLESHOOTING-cant-seat-players.md`.
