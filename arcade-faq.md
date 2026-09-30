> **CORRECTION 2026-09-30 (knowledge archive pass after the FAQ decode review):** The Rev D ceiling is now verified: racing raises each internal only to 55, and growth switches off once the three total more than 160 (about 161 in practice); quote it (FAQ revd-stat-ceiling). Source: the public FAQ (play.johnreevesiii.com/faq), FAQ decode review 2026-09-30.

# Arcade FAQ: the questions players actually ask (updated 2026-09-28)

Player-facing answers for the online arcade at https://play.johnreevesiii.com. Written from real
community questions. Where a detail is a community heuristic rather than decoded fact, it says so. On
game mechanics, canon.md (the verified answer key) wins over this file.

## Community vocabulary (seals, jackpots, anomalies)
- **Bloodline grade seals** (Studbook, stable, Genealogy, Breeding Lab): struck at BIRTH from the
  foal's hidden TRUE birth internals, never the card's printed numbers. Full ladder and the OG tag:
  see **bloodline-seals.md**, which is authoritative and supersedes anything older.
  One at the 45 ceiling = **Blue Chip**, two = **Elite**, all three = **Trifecta** (renamed from
  "Jackpot" on 2026-07-18). Above those sit the 50-rungs: Anomaly, Double Anomaly, Triple Anomaly.
- **"Anomaly"** is the rare +5 birth band (~1 in 32): ALL THREE internals get +5 at once, the only way
  past the 45 ceiling. Because it lifts all three, the number of 50s equals the number of stats the
  pairing brought to the wall, so **one 50 = Anomaly, two = Double Anomaly, three = Triple Anomaly**.
  Since 2026-09-17 "Anomaly" means exactly ONE 50, not "any 50". "Fishing" or "shiny hunting" =
  re-rolling pairings for the band.
- **A caution many players share**: anomalies have big INTERNALS, but a mass-produced anomaly can still
  lose to a well-raised horse. (Unverified: the common belief that races are won mainly by externals
  rather than internals has not been confirmed. Do not state it as fact.)
- **"Monster"** just means an exceptionally strong racer.

## How do I get double circles (◎)? (corrected 2026-09-28)
- Breeding symbols are FIXED AT BIRTH. Band thresholds per external (shown 1-16): ✕ = 1-4, △ = 5-8,
  ○ = 9-12, ◎ = 13-16. A foal's band is the rounded-down AVERAGE of its sire's and dam's bands, with
  no roll. Racing, training, feeding and retiring never change a horse's bands.
- (Corrected 2026-09-28: this answer used to say the symbols are set at retirement and that you should
  raise a horse's externals before retiring it. That was wrong. Verified against the ROM decode and
  measured on 1,300+ saved horses, 400+ of them retired: the bands never moved.)
- So ◎ comes from PARENTS, not from a longer career: a foal can't beat its better parent's band, and
  for ◎ the two parents' shown values must add up to 26 or more (◎ 16 with ○ 10, for example).
- It is normal for double circles to take generations: pick the strongest parents on the externals
  you want, breed, keep the best foals, repeat. Breed like-to-like: parents that are both ◎ (or both
  ✕) on enough externals also earn the pedigree bonus on internals. Starting CPU stock (Thunder Boy,
  Ferranti's Folly, SaraBeara, Scarecrow etc.) mostly carries circles, not double circles.

## "All O" horses: which CPU sires and dams have every external at circle or better?
- "All O" / "all circles" means all SIX externals at ○ or better (◎ counts). "All double circles" means
  all six at ◎. Bands are graded on the display value (raw+1), the rule the site uses.
- **No CPU sire or dam in ANY version has all six at ◎.** The most is four: Scarecrow ◎◎××◎◎ (Rev C,
  Rev D); Sukeakurou (スケアクロウ) and Osutaakaapetto (オスターカーペット) in the JP versions.
- All six at ○ or better, by version (answer per version, the catalogs differ):
  - **Rev C** (6): dam **Lovely Run** ○○○○○○ (the ONLY such dam); sires Broadway Dreu, Glass Glider,
    Hit Maker, Sunday Silence, Flash Point.
  - **Rev D** (9): dam **Lovely Run** (still the only dam); sires Broadway Dreu, Glass Glider, Hit Maker,
    Flash Point, Maverick, The Rain Maker, It's About Time, Bet the Rent.
  - **DOC 2000** (6): dam Shinkouraburii (シンコウラブリイ); sires Eajihaado, Gurasuwandaa (Grass Wonder),
    Hittomeekaa, Sandeesairensu (Sunday Silence), Buraianzutaimu.
  - **DOC '99** (2): dam Shinkouraburii, sire Sandeesairensu.
- Lovely Run's internals are modest (22/32/44), so she is a bands pick, not an internals pick.

## When should I retire a horse? (corrected 2026-09-28)
- Retiring does NOT change breeding symbols; they were fixed at birth. Retire when you want to stop
  racing the horse or start breeding from it. Retiring is done in-game at the cabinet.
- Measured fact (fleet data, 4,722 save pairs, 2026-07-29): internal-stat growth (speed, stamina,
  sharp) is race-driven and front-loaded, and the wall is at ~20 races (near-zero gain 20-24, about
  zero on average from 25). Half of it happens in the first ten races. A foal's birth internals come
  from its parents' CURRENT internals, so racing a parent up helps the internals it passes on (up to
  the 45 birth wall), but not its bands.
- Sega's World Edition brochure: a horse is asked about retirement after its 20th race and "will no
  longer be competitive (this usually occurs between 30-45 races)" (OFFICIAL).
- Breed limits: a cap of 20 in-game breedings per retired horse at a cabinet is widely stated but not
  yet verified in the ROM; the online Breeding Lab is uncapped. "Breeding a horse too many times
  wears its symbols down" is an old handbook claim, never verified.

## Names (corrected 2026-09-28)
- The card's name field holds up to 18 characters (letters, digits, and spaces all count). Japanese
  cards use katakana names.
- There is no general rename. A Lab or Starter Lab foal can be named (or renamed) after the roll,
  until its first race. Cabinet foals are named at the cabinet. Your STABLE name is changed with the
  pencil icon beside your name at the top of the site.

## Records and leaderboards behavior
- The game stores race times in 0.05-second increments, so a time you saw on screen can display
  slightly differently on the record board.
- Online track records come from the HOUSE cabinets, and only after the cabinet writes its save
  file (which happens when the cabinet restarts); the site picks them up about 15 minutes after that.
  So a record set today may appear after the next cabinet restart, not right away. Only the best time
  per course and version is kept. (Corrected 2026-09-28: this used to say "within a few minutes".)
- Cab C (Classic, Rev D) records stay on the cabinet and do not post. Records set on community-hosted
  (home) cabinets do not post to the official board either. Home cabinets do feed the live Whip
  Charts race data when the race caller is enabled.
- There are THREE different record boards (online arcade, classic community uploads at
  doc.johnreevesiii.com, and the printed 2004 national records). Don't compare across them.

## Seats and sessions
- One account, one seat. To change seats: leave your seat (🚪, also possible from your phone via
  the seat QR), then claim the other seat. After leaving, there is a 3-minute cooldown before you can
  play on a DIFFERENT cabinet (sitting back down where you were is always allowed). Winner's Circle
  members skip it on Cabs A, B and JP, but not on Cab C (Classic), and a hop to another house cabinet
  within 3 minutes of racing can still be refused at card load. When every cabinet is full there is
  one line for the next free seat, and Winner's Circle members are served first in it.
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

## Starter Lab (free breeding, since 2026-09-25)
- The Starter Lab is the free, basic Breeding Lab for players who are NOT in Winner's Circle. It lives at
  https://play.johnreevesiii.com/breeding (free players see it there automatically).
- How it works: pick a sire and a dam from the house CPU stallions and mares (the Rev C roster), press
  "Breed my foal" once, and meet the foal (its seal, coat, and colt or filly). You can name it right there
  before its first race. It goes straight into your stable with the 🧪 LAB mark and races on Cab A and Cab B.
- Why it exists: every roll at a cabinet holds a racing seat. The Starter Lab moves casual breeding off
  the cabinets so the seats go back to racing. Cabinet breeding still works exactly as before.
- Limits of the basic version: 3 foals a day per account and per connection, resetting at midnight
  Pacific; letting a foal go does not give the roll back. House CPU horses only, Rev C only (no breeding
  your own horses, Studbook stock, or the DOC 2000 Legends). Colt or filly and coat are rolled for you;
  silks are plain white and white. No predictions, no stats behind the pick, no re-rolls.
- Winner's Circle members use the full Breeding Lab instead: their own horses, the Studbook and the
  Legends, choosing colt or filly and silks (8 patterns, 15 colors), the predicted foal before breeding,
  Japanese output, and as many foals as they like.

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
  one back, held for review (it can race, but can't be a Lab parent until an operator clears it).
- The card popup on /stable shows breeding counts: 🧬 = in-game breeds on the card, 🧪 = your Lab
  breedings with it.

## Money and grinding
- Earnings come from purses; G1 races pay the most, and the Japan Cup / Derby Owners Cup tier is
  the community's favorite money grind (a top horse can clear about $2M a race there). The
  multi-million stables you see on the leaderboard are many wins at that tier, compounded across a
  stable, not a trick.

## Rider skill vs horse strength (corrected 2026-09-28)
- Both matter, but how much riding matters, and which riding, is not pinned down. Measured across
  5,590 fleet races, how closely a ride follows the community handbook whip chart does NOT predict
  the finish (podium rates are nearly identical for A and E ride grades). The Whip Charts are the
  community's tradition, not a proven edge. (This answer used to say timing "measurably changes
  outcomes"; the measurement says otherwise for chart-following.)
- The per-rider Race Program on /whips scores how classical a ride was, not how good it was.

## Special coats
- Special coats exist on player cards: Okapi, Cow, Panda, Orange Panda, Platinum, White, Zebra,
  Cow_2 and Tiger (these names come from the community card tool; any other special code shows as
  plain "Special"). They're rare. The Breeding Lab's prediction panel shows the site's estimate of
  special-coat odds, but the real per-variant odds are not decoded from the ROM. Players believe
  lineage matters (breeding within lines that have produced specials); that is unverified.
  Treat coat hunting as a long game.
- White markings and hood/blaze patterns are part of the appearance genetics on the card; foals from
  the Lab inherit appearance from the same system the game uses.

## Stat ranges cheat sheet (use these exact numbers)
- Internals (Speed / Stamina / Sharp): birth base capped at 45 each (135 sum). The ~1-in-32 anomaly
  band adds +5 to all three (50/50/50, 150 sum). Racing then grows current internals above birth
  values; the card caps each at 60 (Rev C). Rev D (Cab C) behaves differently and its ceiling is
  still being verified, so don't quote one for Rev D.
- Two DIFFERENT external numbers (corrected 2026-09-28, this line used to mix them up):
  - BREEDING BANDS (the symbols): stored 0-15, shown 1-16, fixed at birth. 13-16 = ◎, 9-12 = ○,
    5-8 = △, 1-4 = ✕. Six bands, shown total max 96.
  - CURRENT externals (Start, Corner, Out-of-box, Competing, Tenacious, Spurt): stored 0-63, shown
    1-64. They change during a career, affect racing and set running style. Never divide one to get
    a symbol.
- Dirt aptitude: 0-255. It is a penalty mitigator for dirt races (turf is the default surface);
  higher dirt does NOT reduce turf ability. Rough leans: 100 or less = turf-ish, 170+ = dirt-ish.
- Breeding count on a card: the card keeps a breed counter; a cap of 20 in-game breedings per retired
  horse is widely stated but not yet verified in the ROM. The online Lab is uncapped (since 2026-07-26).

## Externals: 0-63 is the real Rev C scale ("64 cap" is a spreadsheet display convention)
- On the Rev C card (the version our cabinets run), a current external is stored as a byte holding
  0-63, and the game displays value+1. The famous "64 cap" is NOT in the ROM; it comes from the
  community's Super Juicer spreadsheet, whose own VBA proves the point: its read path adds +1 to the
  card byte for display, and its write path validates 1-64 then executes new_data = new_data - 1
  before writing. Type 64 into the sheet and the card receives 63; read it back and the sheet shows
  64 again. That round-trip illusion is where the myth came from.
- ROM evidence (Rev C): the byte-verified card map lists current externals as "u8 0-63, value-1,
  display 1-64" and the breeding bands (the card field labelled "retirement externals", though they
  are set at birth) as "u8 0-15, value-1, display 1-16" (same convention, two scales). Across the
  entire decoded CPU roster, current externals top out at exactly 63 and never 64, even though the
  byte could hold larger. The decoded formulas (the leg-type Start-rank rule, breeding averages and
  band thresholds at raw 12/4) consume the RAW value; the +1 exists only on screen. In the DOC 2000 sibling codec, three externals are 6-bit
  bit-packed fields where 64 literally cannot be represented.
- Practical rule: saying "64" as a display number is fine (it means stored 63), but any MATH done on
  the 1-64 scale (averages, breeding predictions, band boundaries) comes out shifted by one. Compute
  on 0-63.
