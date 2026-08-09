# Known issues, recent fixes, and corrections (updated 2026-07-31)

The honest ledger: what was broken, what got fixed, and what's still open, so answers about "why did
X happen" can cite the real cause. Player-visible items only. When something here contradicts an
older document, this file and arcade-live-rules win.

## Currently open / known limitations
- Self-hosted (community) cabinets do not yet feed exact horse identity into the Whip Charts; their
  rides show as "Unknown horse" unless the rider saves soon after racing. A cabinet-pack update
  carries the fix; hosts get it with their next pack upgrade.
- Records set on community-hosted cabinets do not post to the official online track-record board.
- The Stretch-runner handbook chart appears to fight how the game actually plays; almost no rider
  follows it (see the whip telemetry findings).
- Very new horses bred moments ago won't appear in the Breeder's Cup until campaigned (10+ races or
  retired). That's the anti-farm rule, not a bug.
- Track records leave a cabinet only when the machine writes its save, so a new record can take a
  little while to appear on the board. The board footer shows the honest cadence.

## Fixed (July 2026) — if a player mentions these, they're already resolved
- **"No open seats" on an empty cabinet (JP cab, late July):** a seat-station service outage made
  the safety system refuse to auto-assign seats it couldn't verify empty. Fixed with self-healing
  watchdogs; if it ever recurs, /reset or a report in #bug-reports gets it bounced.
- **"My save said 'already saved' but the races didn't stick" (JP cab, one evening):** an
  auto-cleanup watcher misread the JP game's screens and ejected a live player's card mid-session.
  The watcher was recalibrated for the JP game; the affected horse was recovered with nothing lost.
- **Genealogy showing wrong/archived parents (late July):** two family-tree resolution bugs — one
  hid culled ancestors ("UNKNOWN LINE" on lines that existed), one let a culled re-roll shadow the
  real kept horse (Anomaly parents showing as archived 0-race Jackpots). Both fixed; trees now
  prefer the living, correct card, and culled re-rolls of a kept horse never appear as lineage.
- **Whip Charts showing the wrong leg type (LemonGo case):** the charts were deriving running style
  from birth aptitudes instead of current (developed) externals. Fixed; chart styles now match the
  in-game screen exactly, and Race Program scores grade against the correct handbook zone.
- **Whip/leaderboard pages showing stale or partial data:** two data-window bugs (a race-identity
  feed outage and a query cap) meant charts sometimes showed only a slice of recent races. Both
  fixed with self-healing.
- **OG Jackpot invisible in the Breeder's Cup:** it always SCORED (same as Elite) but displayed as
  plain Elite. Now the best-foal chip shows the true OG Jackpot seal and the 🕹️ Old School
  achievement exists for breeding the original 45/45/33.
- **The 20-breed Lab stud limit (lived 07-24 to 07-26):** removed. Lines culled under it are
  restorable via the Glue Factory's ♻ Restore.
- **"Saved twice and it never saved" (mid-July, house cabinets):** stale seat-card reads on save;
  fixed with a save guard plus on-screen save receipts showing exactly what was banked.
- **Records not registering for ~3 weeks (mid-July):** a cabinet's records uploader was silently
  failing; fixed, and the backlog of records was recovered and posted.
- **CPU-sired Lab foals false-locking each other:** a breeding chain-lock bug; fixed. CPU parents
  never lock anything (if a lock message names what looks like a CPU horse, a same-named horse in
  the player's OWN stable is the real parent).

## Corrections to old community beliefs (data-backed)
- **Externals are stored 0-63, not 1-64.** The "64 cap" came from the Super Juicer spreadsheet's
  display convention (+1 on read, -1 on write). Any math done on 1-64 is off by one.
- **The growth plateau is ~20 races, not 25-30.** Measured from 4,722 save pairs; near-zero gain at
  20-24 races and zero after 25.
- **Internals don't win races; externals and riding do.** Anomaly-fishing produces big internals and
  poor race results if the externals are weak.
- **The 2003 handbook's zone scripts don't measurably improve finishing.** In head-to-head races the
  only component that correlates with finishing better is a steady whip rhythm through the final
  stretch. Zone discipline and rocket openers show no effect.
- **Beer has no coded stat effect.** Every shipped beer entry is a zeroed clone; it's flavor only.
- **Breeding externals never roll.** A foal's externals are the exact average of the parents' bands;
  only the internal +5 anomaly band (~1 in 32) is luck.

## "cloudflared not bundled - skipping tunnel" when starting streaming (self-hosting)
- Diagnosis: the pack zip DOES bundle the tunnel program (runtime\cloudflared.exe in every pack since
  0.7.x; verified present in 0.7.12 and 0.8.0). If Step 2 prints "cloudflared not bundled", the file
  was removed from the host's EXTRACTED copy, almost always by antivirus quarantine. AVs (including
  Windows Defender) routinely false-flag cloudflared as riskware. This is NOT a pack bug and NOT the
  host's fault, and the game/proxy [ok] lines mean the rest of the setup is healthy.
- A missing/empty public_whep_base complaint at the same time is downstream of the same cause: that
  value is stamped from the tunnel URL, so no tunnel = no whep base. Fix the tunnel first.
- Fix (walk the host through it, no jargon): Windows Security -> Virus & threat protection ->
  Protection history -> find the quarantined "cloudflared" entry -> Actions -> Restore/Allow. Then
  add the pack folder as an AV exclusion (Manage settings -> Exclusions -> Add folder) so it stays.
  If Protection history is empty: add the folder exclusion FIRST, delete the pack folder, re-extract
  the zip fresh, and the file survives. Then re-run "2 - Start Streaming" and look for the tunnel
  line to show [ok] with a URL.
- Related, separate quirk: a host whose ISP/DNS blocks trycloudflare addresses may not be able to
  open cabinet pages THEMSELVES even though their own tunnel works and everyone else sees their
  cabinet fine. Workaround is switching the host's DNS to 1.1.1.1 or 8.8.8.8. Only bring this up if
  the tunnel says [ok] but pages won't load for the host.

## Self-hosted cabinet dies / python.exe crashes after about an hour (laptop hosts)
- Diagnosis: the host machine is going to SLEEP on default Windows power settings. Sleep tears down
  the screen-capture sessions, and the python capture processes die on wake. The near-exact
  "about an hour, every time" regularity is the tell: load/memory crashes look random, power timers
  look like clockwork. Laptops are the usual case; desktops with sleep enabled can hit it too.
- Fix for the host: Settings -> System -> Power & battery -> Screen and sleep -> set BOTH the sleep
  timer and the screen-off timer (plugged in) to Never while hosting, and keep the laptop plugged
  in. The screen must stay awake too, the capture needs the display compositing.
- Pack-side fix: a keep-awake guard (the cabinet holds the machine awake by itself while streaming,
  released when hosting stops) is built and ships with the next pack update, so this fix is only
  needed on packs up to 0.8.0.
- If it still dies with sleep disabled, escalate to John: the next suspect is a GPU driver reset
  (AMD/Intel iGPU under sustained encode), which needs Event Viewer digging.

## SPECIAL HOST: Rayna / RBS / irocubabe runs a PRIVATE VPS tunnel (read before diagnosing her)
- Rayna's ISP (Buzz BroadBand, rural Alabama) is blocked by Cloudflare, so the standard trycloudflare
  tunnel can NEVER work for her, no pack version fixes it. Her cabinet uses a private reverse tunnel
  to cab-rayna.johnreevesiii.com instead: Windows' built-in ssh dialing out to our server with a
  scoped key (kit "DOC-rayna-vps-tunnel-v3.zip", DM'd 2026-07-31; it deliberately avoids any extra
  downloaded program because antivirus flags tunnel tools).
- For HER, "cloudflared not bundled - skipping tunnel" in Step 2 is CORRECT AND EXPECTED: her
  cloudflared.exe is deliberately renamed cloudflared.exe.off so the pack doesn't fight her ISP.
  NEVER tell Rayna to restore, re-download, or un-rename cloudflared, and never diagnose her missing
  cloudflared as antivirus or a pack bug. Her hosting flow: Step 1, Step 2 (ignore the tunnel
  warning), then "START MY TUNNEL.cmd" (leave the window open; green CONNECTED banner = live).
- If her tunnel window shows [X] or won't say CONNECTED: have her screenshot the window, then
  escalate to John. Known July gotchas on her setup: a just-dropped tunnel needs ~30s before the
  line frees up for reconnection (her kit waits automatically), and her cabinet clock once ran >60s
  slow (her ISP's addresses geolocate wrong, Windows auto-timezone picked Caracas), which breaks
  sitting down with "token not yet valid" - fix is correcting Windows time/timezone manually.
- General lesson encoded here: before diagnosing any host's tunnel problem, check whether they have
  a special arrangement in this document. Standard advice can be WRONG for special hosts.

## Correction (2026-07-31): the "antivirus ate cloudflared" diagnosis has an exception
- The AV-quarantine entry above is the right diagnosis for most hosts, but NOT for hosts with a
  private-tunnel arrangement (see the Rayna entry): for them the file is renamed .off on purpose.
  Check for a special arrangement before prescribing the AV fix.

## JP cabinet was accepting / creating World Edition cards (fixed 2026-08-01)
- Symptom on the JP cabinet: World Edition horses could sometimes be loaded onto its seats, and
  horses saved there could come back tagged as World Edition cards. Root cause: when one of the
  cabinet's per-seat services was unreachable, the version lock silently went permissive and the
  save path fell back to a World Edition default. The JP cabinet's seat-service outages in late
  July made both real.
- Fixed fail-closed: the cabinet now knows its own game version outright, so even with a seat
  service down, JP seats refuse World Edition cards and JP saves are tagged correctly.
- Player-visible bonus: the phone's seat-load list now MARKS horses the cabinet's game can't read
  (a greyed "✕ EN horse" / "✕ other JP game" chip with an explanation) instead of showing a
  working-looking button that errors on tap. If a horse shows that chip, it's a game-version
  mismatch, not a broken cabinet: World Edition horses play on the World Edition cabinets, JP
  horses on the matching JP cabinet.
- Reminder of the version rule: World Edition Rev C/D cards interchange freely; JP DOC 2000
  (derbyo2k) and JP DOC '99 (derbyoc) are DIFFERENT GAMES and don't cross-load with anything,
  including each other.
