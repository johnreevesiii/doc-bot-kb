# Arcade FAQ: the questions players actually ask (updated 2026-07-26)

Player-facing answers for the online arcade at play.johnreevesiii.com. Written from real community
questions. Where a detail is a community heuristic rather than decoded fact, it says so.

## Community vocabulary (seals, jackpots, anomalies)
- **Bloodline grade seals** (shown in the Studbook, stable, and Genealogy): struck at BIRTH from the
  foal's hidden birth internals. One stat at the 45 ceiling = **Blue Chip**, two at the ceiling =
  **Elite**, all three = **Jackpot** 🏇🏇🏇. The seal is about the birth roll, not race results.
- **"Anomaly"** is the community's name for a foal that hit the rare +5 birth band (roughly 1 in 32
  rolls): ALL THREE internals get +5, which is the only way past the 45 ceiling (up to ~50/50/50,
  the ~250 stat-sum ceiling seen in-game). "Fishing" or "shiny hunting" = re-rolling pairings for it.
- **A caution the community itself discovered**: anomalies have big INTERNALS, but races are won by
  EXTERNALS and riding. A mass-produced anomaly with weak externals loses to a well-raised horse.
- **"Monster"** just means an exceptionally strong racer, usually strong externals + good riding.

## How do I get double circles (◎)?
- Breeding symbols are set at RETIREMENT from the horse's externals. Band thresholds per external:
  ✕ = 1-4, △ = 5-8, ○ = 9-12, ◎ = 13-16.
- A foal's external is the AVERAGE of its parents' bands (deterministic, no roll). So two ○ parents
  around 9-12 average to ○ again; to reach ◎ you must RAISE the horse's externals in-game (racing
  and training grow them) and retire it while they're high, then pair parents whose bands average
  13+. Breed like-to-like: matched strong externals also earn the pedigree bonus.
- It is normal for double circles to take generations: raise, retire well, pair matched parents,
  repeat. Starting CPU stock (Thunder Boy, Ferranti's Folly, SaraBeara, Scarecrow etc.) mostly
  carries circles, not double circles.

## When should I retire a horse?
- Measured fact (fleet data, 4,722 save pairs, 2026-07-29): stat growth is race-driven and
  front-loaded, and the wall is at ~20 races (near-zero gain 20-24, zero from 25). Half of career
  growth happens in the first ten races.
- Community heuristic that matches the data: when training gains shrink to almost nothing (the tiny
  single-arrow gains), the horse is done growing; retire when its externals are at their peak if you
  want the best breeding symbols. The game's own breeding limit (20 in-game breeds per retired
  horse) still applies at cabinets; the online Breeding Lab is uncapped.

## Names
- The card's name field holds up to 18 characters (letters, digits, and spaces all count). Japanese
  cards use katakana names. Renames are allowed by the arcade (the horse's identity is tracked by
  its card serial and birth stats, not the name).

## Records and leaderboards behavior
- The game stores race times in 0.05-second increments, so a time you saw on screen can display
  slightly differently on the record board.
- Online track records update automatically from the cabinets shortly after a race; the Hall of Fame
  and leaderboard pages refresh within a few minutes at most. If a record still looks stale after
  that, report it in #bug-reports.
- Records set on community-hosted (home) cabinets do not currently post to the official online
  track-record board; the official board is fed by the community cabinets. Home cabinets do feed
  the live Whip Charts race data when the race caller is enabled.
- There are THREE different record boards (online arcade, classic community uploads at
  doc.johnreevesiii.com, and the printed 2004 national records). Don't compare across them.

## Seats and sessions
- One account, one seat. To change seats: leave your seat (🚪, also possible from your phone via
  the seat QR), then claim the other seat. Free-tier members may hit a short seat cooldown after
  leaving; Winner's Circle members have none and also skip the line.
- "No seat available" while seats look open usually means the seats are reserved as LOCAL (in-person)
  seats by the host, or a cooldown is active.
- Spectating is free: you can watch any cabinet's stream without a seat.

## Verified vs unverified, and racing others
- Unverified horses CAN race: on your own self-hosted cabinet, against whoever plays there.
- They CANNOT load onto community seats, rank on the leaderboard, or publish to the Studbook.
- Everything bred in the Breeding Lab or grown on a community cabinet is verified automatically.
  If your foal shows verified, that's normal and good.

## Breeding lock ("race the parent first")
- The lock means: this foal was bred from a parent that was still UNRETIRED and under 20 races. Race
  that parent to 20 (or retire it) and the foal unlocks automatically.
- CPU/ROM horses never lock anything. If the message names what looks like a CPU horse, you have a
  horse with that same name in YOUR OWN stable, and that card is the parent it wants raced.

## Studbook and stable features people miss
- Studbook has a filter/sort bar (public pool / my stable active / retired, plus sorting).
- Share codes: open one of your retired horses in the Studbook and use the share option to mint a
  ONE-breed code for another player ("poke trade").
- Genealogy shows a horse's full family tree, including offspring tracking, culled (archived)
  ancestors, and linebreeding crosses like "3S×4D" (ancestor at gen 3 sire-side and gen 4 dam-side;
  informational only).
- The Breeding Advisor ranks pairings toward a goal (max internals, breeding ability, dirt, a
  running style, and more); the Potential Mates tool scores community studs for one of your horses.
- The Glue Factory (bottom of the Stable page) holds every horse you deleted, and ♻ Restore brings
  one back.
- The card popup on /stable shows breeding counts: 🧬 = in-game breeds on the card, 🧪 = your Lab
  breedings with it.

## Money and grinding
- Earnings come from purses; G1 races pay the most, and the Japan Cup / Derby Owners Cup tier is
  the community's favorite money grind (a top horse can clear about $2M a race there). The
  multi-million stables you see on the leaderboard are many wins at that tier, compounded across a
  stable, not a trick.

## Rider skill vs horse strength
- Both matter and neither fully dominates. The horse sets the ceiling (externals especially), but
  whip/hold timing is a large real effect: the Whip Charts exist because timing measurably changes
  outcomes, and the per-rider Race Program on /whips scores how close a ride was to "perfect going".
  A great rider on a clearly weaker horse can steal races, but not consistently against a much
  stronger horse ridden competently.

## Special coats
- Special coats exist on player cards: Okapi, Cow, Panda, Orange Panda, Platinum, White, Zebra,
  Tiger, and other starred variants. They're rare, and the Breeding Lab's prediction panel shows the
  special-coat odds for any pairing before you breed. Lineage influences coat outcomes, so players
  chasing a specific special (a zebra, say) breed within lines that have already produced specials.
  Exact per-variant odds are not fully decoded; treat coat hunting as a long game.
- White markings and hood/blaze patterns are part of the appearance genetics on the card; foals from
  the Lab inherit appearance from the same system the game uses.

## Stat ranges cheat sheet (use these exact numbers)
- Internals (Speed / Stamina / Sharp): birth base capped at 45 each (135 sum). The ~1-in-32 anomaly
  band adds +5 to all three (50/50/50, 150 sum, the "~250 total" players quote includes externals).
  Racing then grows current stats above birth values; the on-screen ~55 per internal is the ceiling.
- Externals (Start, Corner, Out-of-box, Competing, Tenacious, Spurt): each stored 0-15 raw and
  DISPLAYED as value+1, so 1-16 on screen. Six externals, display total maxes at 96; typical horses
  sum 40-70. Bands per external on the display value: 13-16 = ◎, 9-12 = ○, 5-8 = △, 1-4 = ✕.
- Dirt aptitude: 0-255. It is a penalty mitigator for dirt races (turf is the default surface);
  higher dirt does NOT reduce turf ability. Rough leans: 100 or less = turf-ish, 170+ = dirt-ish.
- Breeding count on a card: the game caps at 20 in-game breedings per retired horse (the online Lab
  is uncapped as of 2026-07-26).

## Externals: 0-63 is the real Rev C scale ("64 cap" is a spreadsheet display convention)
- On the Rev C card (the version our cabinets run), a current external is stored as a byte holding
  0-63, and the game displays value+1. The famous "64 cap" is NOT in the ROM; it comes from the
  community's Super Juicer spreadsheet, whose own VBA proves the point: its read path adds +1 to the
  card byte for display, and its write path validates 1-64 then executes new_data = new_data - 1
  before writing. Type 64 into the sheet and the card receives 63; read it back and the sheet shows
  64 again. That round-trip illusion is where the myth came from.
- ROM evidence (Rev C): the byte-verified card map lists current externals as "u8 0-63, value-1,
  display 1-64" and retirement externals as "u8 0-15, value-1, display 1-16" (same convention, two
  scales). Across the entire decoded CPU roster, externals top out at exactly 63 and never 64, even
  though the byte could hold larger. Every decoded engine formula (the per-phase race formula, the
  leg-type Start-rank rule, breeding averages and band thresholds at raw 12/4) consumes the RAW 0-63
  value; the +1 exists only on screen. In the DOC 2000 sibling codec, three externals are 6-bit
  bit-packed fields where 64 literally cannot be represented.
- Practical rule: saying "64" as a display number is fine (it means stored 63), but any MATH done on
  the 1-64 scale (averages, breeding predictions, band boundaries) comes out shifted by one. Compute
  on 0-63.
