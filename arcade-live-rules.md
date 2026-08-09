# The Online Arcade: current rules and features (updated 2026-07-26)

This file describes the LIVE online Derby Owners Club arcade at play.johnreevesiii.com as it works
today. When an older document in this knowledge base contradicts this file, THIS file wins.

## The arcade at a glance
- play.johnreevesiii.com is a browser-playable version of the Sega DOC arcade game. The lobby shows
  live cabinets: two house cabinets, "8-player (A)" and "6-player (B)", plus community-hosted
  cabinets run by players from their own homes.
- You sign in, claim a seat on a live cabinet, and play in the browser. Seats are input-locked to
  their holder. One account, one seat at a time.
- Your cloud stable lives at play.johnreevesiii.com/stable (works on a phone). From a seat's QR code
  you can load a stable horse into the seat, save it back, or leave the seat from your phone.
- Chat has per-cabinet channels, a 🌐 translate button (for our Japanese players), and self-mute
  tools. There is a per-cabinet "Join Voice" link into the Discord voice rooms.

## Membership tiers
- Free ("Stable" tier): sign in, play, save up to 25 horses in the cloud stable.
- Winner's Circle (WC): the supporter tier, granted to donors (the donation model funds the server
  costs) and by invite. WC perks: the Breeding Lab and foal generator, the community Studbook
  (publish, browse, share codes, "Potential Mates" matchmaker), the Feeding Advisor (/feeding), the
  Career Log (/career), Breeding Advisor, one-way card export, skip-the-line into seats, and no seat
  cooldown. Donations happen via the Membership page.

## Breeding Lab rules (as of 2026-07-26)
- The 20-breed stud limit was REMOVED on 2026-07-26. No horse's stud or broodmare career ends in the
  Lab anymore, and re-rolling pairings ("shiny hunting" for the rare all-stats +5 jump) is allowed.
  Breed counters (🧬 in-game breeds on the card, 🧪 your Lab uses) still display but are
  informational only.
- At a REAL cabinet the game itself still enforces its own on-card limit of 20 breedings per retired
  horse. That is the ROM's rule and unchanged; only the online Lab is uncapped.
- Parents must be RETIRED to breed (both Lab and game).
- The "puppy mill" chain lock: a foal bred from a parent that was still unretired and under 20 races
  is locked from breeding until that parent reaches 20 races or retires. It unlocks automatically.
- The Lab supports all four game versions: World Edition Rev C/D combined, DOC 2000 (derbyo2k), and
  DOC '99 (derbyoc). Japanese foals get katakana names and real JP pedigree handling.
- You can pick the foal's name, sex, and silk pattern/colors. Sex does not change stats.

## How foal stats actually work (byte-exact, decoded from the ROM)
- Internals (speed/stamina/sharp): the foal's base internal is the floor-average of the two parents'
  CURRENT internals, then a soft clamp (any average over 45 loses 5, under 10 gains 5), then the
  pedigree bonus, hard-capped at 45, EXCEPT the rare noise band: roughly 1 in 32 births lands a +5
  bump to ALL THREE internals, which is how elite pairings reach ~50/50/50 (the ~250 total ceiling
  players see in-game). This is the "anomaly" or "jackpot" players fish for.
- Pedigree bonus: the game counts externals where BOTH parents are strong (12+) or BOTH weak (<4);
  4-5 matches = +1/+2/+2, 6 matches = +3/+2/+3. Breed like-to-like.
- Externals (Start, Corner, Out-of-box, Competing, Tenacious, Spurt): the foal's external is the
  plain average of the parents' breeding bands. It does NOT roll: breeding externals are
  deterministic. The breeding symbols set at retirement (✕ < △ < ○ < ◎) show each external's
  breeding band.
- Dirt aptitude: inherited as the average of the parents' dirt values.
- IMPORTANT strategy fact: races are won by EXTERNALS (and riding), not by internals. Mass-produced
  lab "anomalies" with big internals but weak externals race poorly.

## Horse growth (measured from live fleet data, 2026-07-29)
- A horse's stats grow from RACING, not from time. Growth is front-loaded and the wall is at ~20
  races. Measured average total stat gain per race, from 4,722 before/after save pairs: ~2.1 (races
  0-4), ~1.5 (races 5-9), ~1.4 (10-14), ~0.8 (15-19), ~0.15 (20-24), zero from 25 on. Half of a
  career's growth lands in the first ten races. Training style and feeding shape WHICH stats grow.
- The Feeding Advisor at /feeding ranks foods by stat gain (from the decoded ROM food table) and can
  build a feeding plan for a specific stable horse.

## Running style (leg type)
- Running style is derived, not stored: it is the rank of the horse's Start value among its 5
  non-corner externals. 1st = Front-runner, 2nd = Start-dash, 3rd = Last-spurt, 4th/5th =
  Stretch-runner, all-equal = Almighty.

## Genealogy, Studbook, and bloodlines
- Genealogy (family tree) shows a 5-generation pedigree. "Linebreeding crosses" notation like
  "Thunder Boy 3S×4D" means that ancestor appears at generation 3 on the sire's side and generation
  4 on the dam's side. Crosses are informational: the breeding engine gives no bonus or penalty for
  duplicated ancestors.
- Bloodline grade seals rate a horse's birth-roll variance; shown in the Studbook, stable, and
  Genealogy.
- The Studbook (WC) lets you publish retired horses for the community to breed with, browse others',
  get "Potential Mates" suggestions scored by the real breeding model, and share ONE-breed codes
  ("poke trades") for private stud deals.
- 10 DOC 2000 legend horses (Special Week, Tokai Teio, etc., decoded from the JP ROM) are in the
  breeding pool as sires/dams with real pedigrees; they wear a ⭐ Legend badge.

## The Glue Factory (deleted horses) and Restore
- Deleting a horse sends it to the Glue Factory: an archive that keeps the full card, pedigree, and
  signature, so a horse used for breeding is never truly lost. Its lineage still shows in Genealogy.
- NEW 2026-07-26: every archived horse has a ♻ Restore button (bottom of the Stable page) that
  brings it straight back into your live stable, stable space permitting.

## Records and leaderboards (three DIFFERENT boards, don't mix them up)
1. ONLINE arcade records: set live on the online cabinets. The default "the record" board.
2. CLASSIC community leaderboard at doc.johnreevesiii.com: players upload saves from their own home
   setups (Rev C, Rev D, DOC 2000). Separate from the online arcade.
3. NATIONAL records: the printed 2004 benchmark times from the DOC handbooks. Historical.
- The arcade Hall of Fame (play.johnreevesiii.com/leaderboard) has wins-by-stable, lifetime breeding
  (Stud King / Broodmare Queen), Dynasty, Consistent, Iron horse, and Fanciest boards.
- The Breeder's Cup is the competitive bloodline leaderboard (WC).

## Whips and riding
- Kaerey's community Whip Charts (play.johnreevesiii.com/whip-charts.html) are the riding reference.
- The live Whip Charts page at /whips shows real telemetry from the cabinets: race-shape by leg
  type, handbook-zone bands, and a per-rider Race Program that scores your ride against "perfect
  going".

## Verified vs unverified horses
- A "verified" (community-verified ✓) horse was bred in the Breeding Lab or grown/saved on a
  community cabinet and carries an un-editable origin signature. It can rank on leaderboards,
  publish to the Studbook, and load onto community seats.
- "Unverified" means it entered another way (e.g. a self-hosted setup). It does NOT automatically
  mean cheated, but it cannot rank/publish/load on community seats. "Under review" clears
  automatically or after an operator glance.

## Cabinet resets (house cabinets A and B)
- Any server member can run /reset in Discord. Two levels:
  - "Bounce": relaunches the video only. Fixes a black or frozen lobby feed, no reboot, drops
    nobody. Try this first for video problems.
  - "Full reset": reboots the whole cabinet. Fixes a hung game, but bumps anyone seated (the bot
    warns which seats are occupied first).
- The bot verifies the screens actually came back (brightness + motion) and reports in-channel.
  Community-hosted cabinets can only be restarted by their own host.

## Hosting your own cabinet
- Community members can host their own cabinet into the online lobby. Type /hostpack in Discord for
  the current pack download + login. The pack has a Setup Wizard and a Cabinet Doctor; if the bot is
  given a hardware-report.json (drag it into Discord) it reads out a diagnosis.
- The current pack supports pairing a cabinet as World Rev C/D, DOC 2000, or DOC '99. Cabinets on
  old packs show "Update required" in the lobby.
- Hosts never receive database keys; saves are validated server-side.

## Controls: defaults and custom mapping (added 2026-07-31)
- Default seat controls (shown on the lobby's Keyboard Controls guide): arrow keys = select, Z = OK,
  Enter = Start, X = Hold, C = Whip, I = insert card. Controllers work out of the box: A or LT =
  Whip, B or RT = Hold, stick or D-pad = select, Start = Start, Y = card.
- CUSTOM MAPPING: the ⌨ button (in the seat's ⌄ More drawer, bottom of the right-edge controls) opens "Customize controls".
  Click a binding, then press any key OR any controller button; the capture auto-detects the device.
  Works with any pad (8BitDo, Xbox, PlayStation). Binding an in-use key moves it. Esc and F are
  reserved (close / fullscreen). The panel also has a toggle for the on-screen Whip/Hold buttons.
- Mappings save PER DEVICE (browser storage), not per account: set it once per machine. "Reset to
  defaults" restores the published layout instantly.

## Horse transfers: give a horse to another owner FOR KEEPS (added 2026-07-31)
- A Winner's Circle member can transfer any of their horses to another player permanently, the
  digital version of handing over the physical card. Everything travels with the horse: stats,
  card serial, origin signature, pedigree, and the full race record (wins/earnings now count toward
  the NEW owner's stable, exactly like a real card carries its history).
- How: on the Stable page, the 🎁 button on a horse's row mints a one-time transfer code (7-day
  expiry, cancellable until used, one open code per horse). Send the code to the new owner any way
  you like; they redeem it on THEIR Stable page under "Have a transfer code?". Anyone can RECEIVE
  a horse (receiving is not WC-gated); minting codes is the WC side.
- Rules: the horse can't be loaded in a cabinet seat during transfer; the receiver needs room in
  their stable (free tier 25); the studbook listing unpublishes on transfer (new owner can
  republish); any open one-breed poke-trade codes for that horse are cancelled by the transfer.
- Every completed transfer is permanently recorded (who gave it, who received it, when), so a
  horse's chain of ownership is always on record. Transfers don't weaken clone protection: the
  horse MOVES, it is never copied, and duplicate card serials are still blocked everywhere.
- TWO-WAY TRADES have a real ATOMIC SWAP (added same day): both owners mint a transfer code for
  the horse they're giving and exchange the codes; then EITHER owner opens "Have a transfer code?"
  on their Stable page, taps "Two-way trade? Swap two codes atomically", enters both codes, and
  both horses change hands in one all-or-nothing step. There is no moment where one side has given
  and not received; if anything is wrong (a code expired/revoked, a horse in a seat, an owner
  changed), NOTHING moves. Both codes become the swap's ledger entries.

## Per-cabinet HOUSE RULES (added 2026-07-31, requested by Rayna)
- A cabinet host can write their own house rules (up to 500 characters) for their cabinet. Players
  see a "📜 House rules" button on that cabinet's lobby block; tapping it shows the host's text.
- Where hosts set it: Stable page → the "🎛 Host — seat control" section → "📜 House rules" link →
  type the rules → Save. Clearing the box and saving removes them. Changes appear on the lobby
  within about a minute.
- House rules are the HOST's social rules for their machine (examples: family-friendly names only,
  no seat-camping, scheduled racing nights). They don't change game mechanics, and arcade-wide
  rules (anti-cheat, bans, seat cooldowns) always still apply on top. Rules text is run through the
  same word filter as public names.

## Volume mixer (added 2026-08-01, requested by Kaerey)
- Cabinets stream at different loudness (each host's capture gain differs). Two controls fix it:
- LOBBY: next to a cabinet's sound toggle there's now a volume slider. The level you set is
  REMEMBERED PER CABINET, so a loud cabinet stays turned down for you permanently.
- IN A SEAT: the 🎚 button (in the ⌄ More drawer of the right-edge controls) opens a mixer with two sliders: "🏇 Race" (the
  main-screen broadcast; same per-cabinet memory as the lobby slider) and "📺 Seat" (your seat's
  own betting/menu sounds; a personal preference remembered across all cabinets). Blend to taste.
- iPhone/iPad: the sliders intentionally don't appear; iOS routes media volume through the
  hardware volume buttons, so use those.

## Quick Career-Log notes from the seat (added 2026-08-01, community request)
- While seated, the 📓 button (in the ⌄ More drawer of the right-edge controls) opens a quick-note panel for the
  horse you have loaded. Type a note (up to 500 chars, e.g. "fed carrots before R3 - big stretch
  surge") and Save: it lands directly on that horse's Career Log timeline, no page switch.
- The panel links to the full Career Log (/career) for the structured food/training entries and
  the race-by-race history. Notes are private to the owner.
- Typing in the panel never drives the game: keystrokes are ignored by the seat controls while a
  text box has focus.


## Seat controls layout (updated 2026-08-02): 5 primary buttons + a More drawer
- The right-edge controls in a seat now show five primary buttons: 🔊 sound, 📊 low-data,
  ⛶ fullscreen, 🐎 stable, 🚪 leave, plus a ⌄ arrow. Tapping ⌄ opens the More drawer with the
  rest: 📺 seat sounds, 🎚 volume mixer, 📓 quick note, 👥 who's playing, ⌨ controls mapper,
  🎮 on-screen pad toggle, ⤢ screen adjust, 📱 phone QR, and the cabinet reset.
- If a player says a button is "missing", it's almost certainly inside the ⌄ More drawer now.

## Ride GRADES (A+ / A / B / C / D / E) in the Career Log and Whip Charts
- The letter shown on a race in a horse's Career Log is the RIDE grade for that race: how closely
  the rider's whip timing matched the community handbook chart for that horse's running style. It
  is NOT a grade on the horse, its stats, or its bloodline.
- Score 0-100 -> letter: 90+ = A+, 80+ = A, 70+ = B, 60+ = C, 45+ = D, below 45 = E.
- Three weighted components: OPENER 20% (whips in the first ~6% of the race, off the gate),
  ZONE DISCIPLINE 50% (whip inside your style's active window, stay quiet through its rest
  stretch - the biggest factor), FINISH RHYTHM 30% (a steady whip cadence through the last 15%).
- CRUCIAL CONTEXT to include whenever explaining a low grade: measured across 5,590 fleet races,
  the grade does NOT predict finishing position. Podium rates are nearly identical for A and E
  rides, and ~45% of ALL rides grade E. It means "how classical was that ride", not "how good".
  The only component that leans the right way in head-to-head races is the finish rhythm.
- The per-race breakdown of the three parts lives on the Whip Charts page (/whips), per rider.
