# Local (offline) pack + cabinet hosting FAQ (updated 2026-07-26)

TWO different packs exist; don't mix them up:
1. The **LOCAL / offline pack** (Tournament Edition): the downloadable full game you run entirely on
   your own PC(s) with Flycast. Standalone, includes card reading; it can NEVER write to the online
   cloud stable. Get it from the community downloads at doc.johnreevesiii.com.
2. The **online cabinet HOST pack**: for hosting your own cabinet INTO the online lobby at
   play.johnreevesiii.com so remote players can join. Type /hostpack in Discord for the download +
   login. This one pairs with the arcade, streams, and saves to the cloud (server-validated).

## Local pack: setup basics (community-tested answers)
- The game is one MAIN (multiboard) plus satellite instances. Start the satellites and the main with
  the provided Play/launch files; the satellites say "starting network" until the main comes up.
- Flycast network settings if configuring by hand: network type "naomi", the MAIN set to
  "multiboard", each satellite "single", every instance on a DIFFERENT port.
- Windows Firewall: allow Flycast through the firewall the first time it runs; a blocked firewall is
  the #1 "satellites never connect" cause.
- Flycast won't open at all when you click Play: try launching flycast.exe by itself first (driver /
  VC-runtime errors show there), update GPU drivers, and check the antivirus didn't quarantine it.
- You don't need all satellites running to play: run the main plus however many seats you want (the
  4-sat build is commonly played with 1-2 satellites open). Rev C with a full 8-seat mesh is known
  to hang more often than 6; if your rig hangs, run fewer seats.
- Two PCs: a second PC can run a satellite; point that satellite's network config at the main PC's
  IP. Both PCs need the firewall open and the same version.
- Controllers: map buttons per-satellite inside Flycast input settings. An Xbox pad driving ALL
  windows at once means the pad is mapped in every instance; unmap it (or set Player 1 only) in the
  instances it shouldn't control. Two people CAN play on one PC with different controls per
  satellite window.
- Performance: multiple Flycast windows fight for the GPU. Set Windows power mode to High
  performance, try disabling hardware-accelerated GPU scheduling, and disable per-instance VSync if
  windows stutter (the desktop compositor synchronizes them). Audio clipping usually just means the
  PC is at its limit; fewer satellites helps.
- SAVING horses locally: press the card/save action on the SATELLITE you played (before shutting
  down) so the horse writes to that seat's card file; the pack's card manager / suite page can then
  read, back up, or load the card files. A retired horse loaded from the card manager can be used
  for breeding in-game. Save problems on a fresh PC are usually write permissions on the install
  folder (don't run from inside a zip; extract fresh to a real folder).
- The full game download is large; if a download link fails or the page won't load, say so in
  #bug-reports (links occasionally rotate with releases).

## Online hosting: setup and troubleshooting
- Get the pack with /hostpack. Extract FRESH (never over an old folder), run "1 - Start Cabinet"
  then "2 - Start Streaming". The Setup Wizard walks first-time config, and Cabinet Doctor writes a
  hardware-report.json you can drag into Discord: DOC Bot reads it and diagnoses encoder/GPU issues.
- You do NOT need to enable UPnP or port-forward: the pack opens an outbound tunnel itself.
- Screens at half speed, or a main crashing during streaming: almost always the video ENCODER (GPU)
  rather than the game. Run Cabinet Doctor and drop the report in Discord; the pack picks the best
  encoder it finds (NVENC on NVIDIA, etc.), and very old or missing GPUs fall back to CPU encoding
  which can't keep up.
- Moving your cabinet to a different PC = it's a new cabinet: pair it again (register, then operator
  approval). Approval is quick; ask in #hosting-beta if it sits pending.
- "Cabinet offline" for SOME viewers while others can play: the viewer's network is blocking the
  tunnel domain (trycloudflare.com), commonly DNS filters, Pi-hole/NextDNS blocklists, VPNs, or
  ad-blockers. Fixes on the viewer side: switch DNS to 1.1.1.1 or 8.8.8.8, try a phone hotspot, or
  whitelist trycloudflare.com. A relay migration (everything on johnreevesiii.com) is rolling out to
  hosts to remove this failure mode entirely.
- Version updates: the lobby shows "Update required" on old packs. Get the newest pack via
  /hostpack, extract fresh, re-run the wizard. Announcements post when a new pack ships.
- Host machines: any modern gaming PC works; the important part is a GPU with a hardware encoder
  (NVIDIA NVENC is the smoothest path). Community hosts run everything from laptops to Ryzen
  desktops.
- Local seats on a hosted cabinet: a host can flex seats between ONLINE (lobby-claimable) and LOCAL
  (in-person) from the Host section on /stable. Local-seat play saves through the host-stable path;
  if a local seat isn't saving, make sure you're on a current pack and ask in #hosting-beta.
- Records: home cabinets don't set official online track records; those come from the community
  cabinets. Home cabinets DO feed the live Whip Charts race telemetry (race caller; opt-out file
  available), and horses grown on an APPROVED community cabinet are verified and fully portable.
- Private play: a host can keep seats local-only (in-person friends) or share the lobby link for
  open online play. Seat access can effectively be limited by flexing seats local; there is no
  password-protected private lobby feature today.

## Getting help fast
- Drop cabinet problems in #hosting-beta (hosting) or #widows-emulation (local pack), with your PC
  specs (CPU, GPU, RAM) and what step fails; screenshots help a lot.
- Hardware-report.json from Cabinet Doctor (host pack) gets an instant automated readout from
  DOC Bot in any channel.
- Anything broken on the arcade site itself goes to #bug-reports; every post there is flagged
  straight to John.

## Can I clear my cabinet's track records so I can set my own? (Fresh Machine Reset)
- Yes. The "DOC Fresh Machine Reset" tool wipes a self-hosted cabinet's machine memory (nvram) so it
  boots like a brand-new machine: empty record board, fresh race calendar. On next boot the game
  recreates its own factory memory automatically, and from then on the host's own records save
  session to session as normal.
- It does NOT touch horses, cards, or the cloud stable (those are server-side, never in the machine),
  it backs up the old memory first (reversible), and it requires typing YES before acting. Run it
  with the game closed; the first boot afterward takes slightly longer (self-setup) - normal.
- How to get it: ask in #ask-doc-bot and John (or the bot) will send the small zip
  (DOC-Fresh-Machine-Reset.zip). First shipped to Rayna 2026-07-31.
