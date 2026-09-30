> **CORRECTION 2026-09-30 (knowledge archive pass after the FAQ decode review):** Four answers below are out of date; the public FAQ now says: (1) Almighty needs ALL SIX current externals equal, Corner INCLUDED; if Start, Out of the Box, Competing, Tenacious and Spurt are equal but Corner differs, the horse is a Front-runner (FAQ almighty). The 'All five equal = Almighty' line is wrong. (2) Rev D ceiling is verified: racing raises each internal only to 55, and ordinary per-race growth switches off once the three add up to more than 160, so a developed Rev D horse settles at about 161; it is a ceiling, not a freeze (FAQ revd-stat-ceiling). (3) No inbreeding penalty is CONFIRMED for World Edition and DOC 2000 only; DOC '99's breeding code is not fully decoded (FAQ inbreeding-penalty). (4) DOC II inbreeding counts only close relatives (sire x his daughter, mare x her son, a shared sire, a shared dam); each match cuts the foal's grade and growth ceilings by 0 to 8 percent and never its birth stats (FAQ doc2-inbreeding). Source: the public FAQ (play.johnreevesiii.com/faq), FAQ decode review 2026-09-30.

# DOC canon: verified answers to the questions players ask (2026-09-28)

## About this file (read first)
This is DOC Bot's verified answer key, built on 2026-09-28 by checking 143 real player questions and
the bot's answers against the game's ROM decode, the arcade's saved-card data and the website's code.
On game mechanics, if any other file in this knowledge base disagrees with canon.md, canon.md wins.
Tags used here:
- CONFIRMED: decoded from the game ROM, or measured in the arcade's own data.
- SITE: how the website works today.
- OFFICIAL: Sega's own printed material.
- COMMUNITY CLAIM: what players believed. Say who believed it. Never state it as fact.
- UNKNOWN: no verified answer exists yet. Say "we don't know yet" and do not guess.
Always write links in full with https:// so Discord makes them clickable.

## When are breeding symbols set? Can training or retiring improve them?
- Breeding symbols (the ✕ △ ○ ◎ on each external) are FIXED AT BIRTH. Each is the rounded-down average
  of the sire's and dam's band for that external, with no roll. CONFIRMED.
- Racing, training, feeding, winning and retiring NEVER change them. CONFIRMED (ROM decode, and measured
  on 1,300+ horses with a save history, 400+ of them retired: the bands never moved).
- Earlier answers from DOC Bot, and older notes, said symbols "lock in at retirement" and that you
  should raise externals before retiring. That was wrong (corrected 2026-09-28).
- Bands are shown 1-16: ✕ 1-4, △ 5-8, ○ 9-12, ◎ 13-16. CONFIRMED.

## How do I get double circles (◎) or all circles?
- From PARENTS, over generations. A foal's band can never beat its better parent's band on that
  external. For ◎ the two parents' shown values must add to 26 or more (◎ 16 with ○ 10); for ○ or
  better they must add to 18 or more (◎ 13 with △ 5 gives ○ 9), so both parents do NOT need ○.
  CONFIRMED.
- Pick a sire and dam that are strong on the externals you want, breed, keep the best foals, repeat.
- For the game's built-in CPU horses, use the catalog tools; no CPU horse in any version has all six ◎
  (the most is four). CONFIRMED.

## Breeding symbols vs current externals: what's the difference?
- Two different numbers per external. CONFIRMED.
- The BREEDING BAND (the symbol): stored 0-15, shown 1-16, fixed at birth, passed to foals.
- The CURRENT external: stored 0-63, shown 1-64. A foal starts near twice its band plus a small roll,
  then it rises and falls during its career. It affects racing and sets running style. It is NOT
  passed on.
- Never divide a current external to get a symbol (a current 48 does not mean ○).
- Owners can see their own horse's current externals in the card popup on their /stable page. SITE.

## Does breeding a horse many times wear its symbols down? Is there a breed limit?
- "Over-breeding wears symbols down" comes from the 2004 export handbook and has never been verified.
  COMMUNITY CLAIM (export handbook).
- At a cabinet the card keeps a breed counter, and a cap of 20 breedings per retired horse is widely
  stated but not yet verified in the ROM. Say "believed to be 20". UNKNOWN.
- The online Breeding Lab has no breed cap (since 2026-07-26). SITE.

## What is the pedigree bonus?
- The game counts the externals where BOTH parents (sire and dam) are ◎ (raw 12+) or BOTH are ✕ (raw
  0-3). 4-5 matches: stamina +1, speed +2, sharp +2. 6 matches: stamina +3, speed +2, sharp +3. Then the
  45 birth cap applies. CONFIRMED.
- Only the sire and dam count, equally. It raises INTERNALS, never externals. (Older notes said "dam and
  grandparent line"; that label was wrong.)

## Is there an inbreeding penalty? Do wins or G1s make better foals?
- World Edition, DOC 2000 and DOC '99: no inbreeding check or penalty. The foal routine only looks at
  the two parents. CONFIRMED. DOC II is different: shared ancestors apply small multipliers. CONFIRMED.
- Wins, earnings and G1 titles are not passed on. Winners often make good parents only because their
  stats are good. CONFIRMED.
- Online Lab rules: both parents must be retired; a foal of an unretired parent with under 20 races is
  locked from breeding until that parent reaches 20 races or retires. SITE.

## How are a foal's internals (speed, stamina, sharp) decided?
- Start from the rounded-down average of the parents' CURRENT internals. An average over 45 loses 5
  (under 10 gains 5). Add the pedigree bonus. Cap each at 45. CONFIRMED.
- So racing a parent up helps the internals it passes on, up to the 45 wall. Parents averaging 46-49
  give 41-44, worse than exactly 45 or 50+. CONFIRMED.
- About 1 birth in 32 catches the rare band: all three true internals gain 5 (up to 50) while the card
  PRINTS lower. Nothing after conception changes the odds. CONFIRMED (measured 3.5% cabinet, 2.7% Lab).
- Another rare band raises only the printed numbers, on stats under 40. CONFIRMED.
- For about 94% of horses the printed birth stats ARE the true birth internals. Don't tell an ordinary
  owner their card is lying. CONFIRMED (measured). The card popup on /stable shows them as "born x/y/z",
  so a Blue Chip's 45 is usually visible there. SITE.
- Birth internals are only the starting point: racing grows them afterwards (see growth below).

## What do the bloodline seals mean?
- Seals grade TRUE birth internals: three / two / one 50 = Triple Anomaly / Double Anomaly / Anomaly;
  any 46-49 = Prodigy; three / two / one 45 = Trifecta / Elite / Blue Chip; otherwise no seal. SITE.
- OG is a tag on a Triple Anomaly whose card prints 45/45/33 (deeper purple) or 40/40/40 (lighter). SITE.
- Triple Anomaly is the TOP rung, but not the rarest: on 2026-09-17 there were 117 Triple Anomalies and
  15 Double Anomalies. CONFIRMED (measured).
- The Triple Anomaly Watch and Bloodline Pyramid are on the Census tab of
  https://play.johnreevesiii.com/leaderboard (the Watch lists cabinet-born 150s only). SITE.
- Full ladder: bloodline-seals.md.

## How do stats grow over a career? When should I retire?
- Internals grow from RACING, not time: about 2.1 points per race early, about 0.15 at races 20-24,
  about zero on average after 25; over half in the first 10 races; +30 to +35 in total. CONFIRMED
  (measured on 4,722 save pairs). This curve is for internals; the externals curve is UNKNOWN.
- The exact per-race growth rules, and whether stats can decline late in a career, are still being
  verified. Don't quote specific numbers, and don't tell players that racing late is harmless. UNKNOWN.
- Retiring does NOT change breeding symbols. Retire when you want to stop racing or start breeding.
  Retiring is done in-game at the cabinet; the website has no retire button. SITE.
- Sega's brochure: a horse is asked about retirement after race 20 and "will no longer be competitive
  (this usually occurs between 30-45 races)". OFFICIAL.

## What is the stat ceiling?
- Birth internals: 45 each (50 with the rare band). CONFIRMED.
- Current internals: the card caps each at 60 on World Edition Rev C (Cabs A and B). CONFIRMED.
- Rev D (Cab C) behaves differently and is still being verified: don't quote a ceiling for Rev D.
- Current externals: 0-63 stored, 1-64 shown. CONFIRMED.

## What decides running style (leg type)? Can it change?
- Where Start ranks among Start, Out of the Box, Competing, Tenacious and Spurt (the CURRENT
  externals): 1st Front-runner, 2nd Start dash, 3rd Last spurt, 4th or 5th Stretch-runner. Corner is
  ignored; a tie goes to Start. All five equal = Almighty (very rare). SITE + OFFICIAL.
- It CAN change during a career, because current externals change; the game says "Your horse's
  racing style has changed." CONFIRMED.
- Training drills steer it (measured): Turf builds Start, Slope/Hill and the Spurt drill build Spurt,
  Dirt builds Tenacious, Wood mainly Corner and OOB. CONFIRMED (measured).
- A breeding predictor's style for an unborn foal is an estimate: the parents' bands only shape the
  foal's starting externals, and the style can change as it races. Style is not a power tier: CPU
  external totals are flat by style. CONFIRMED.

## Does a race run in phases? What does each external or internal do in a race?
- The popular model of SIX RACE PHASES, one per external (Start, Corner, Out of the Box, Competing,
  Tenacious, Spurt), came from early notes and guides and is UNVERIFIED. Do not teach it, and never
  assign an external to a part of the race.
- What each external does during a race is not pinned down yet. UNKNOWN.
- What Speed, Stamina and Sharp each do individually is not decoded. UNKNOWN. Sega's own description:
  Stamina has the least whip effect and Sharp the most, "especially in the final dash". OFFICIAL (say it
  as Sega's claim).
- Distance advice for a style or stat line ("front-runners fade past 2000m") has no verified source.
  UNKNOWN.

## Does whip timing matter? What are the whip charts?
- Measured across 5,590 fleet races, how closely a ride follows the handbook whip chart does NOT predict
  the finish. CONFIRMED (measured).
- Kaerey's Whip Charts (https://play.johnreevesiii.com/whip-charts.html) are the community's riding
  tradition from the old handbooks. Rocket Start, Super Start and rhythmic whip are handbook terms, and
  the handbooks disagree on some of them. COMMUNITY CLAIM (export handbooks).
- The exact whip rules the game uses are still being verified. Don't quote whip counts or penalties.
- Live ride telemetry: https://play.johnreevesiii.com/whips . SITE.

## What do foods do?
- Every per-food number the Feeding Advisor shows matches the ROM food table (all 43 foods). Rev C, Rev D
  and DOC 2000 are identical; DOC '99 differs on three foods. CONFIRMED.
- WHETHER THE FIRST THREE FOOD COLUMNS RAISE THE INTERNALS OR THE EXTERNALS IS NOT SETTLED. The decodes
  and one live test disagree. Do not call them Speed / Stamina / Sharp as fact, and do not call feeding
  "the internal lever". UNKNOWN.
- White Mushroom: +5 in the second column only (a "+4 Speed" figure came from the 2004 guide chart).
  CONFIRMED.
- Beer: a real food record with no stat effect; the game plays a reaction. CONFIRMED.

## How do I get a food? What about liked and disliked foods?
- The feed menu only offers foods that pass a per-food check; what drives it is not decoded. UNKNOWN.
  No shop, drop or rotation system is documented in the game either: never describe one.
  The 2004 handbook lists how players believed each food was earned. COMMUNITY CLAIM (export handbook).
- The website has no food shop or inventory. The Feeding Advisor (https://play.johnreevesiii.com/feeding,
  Winner's Circle) plans against the ROM table but doesn't know what a cabinet is offering. SITE.
- A horse reacts to liked and disliked foods and it affects its hearts; the exact amounts are still
  being verified. How a horse's likes are assigned, and whether they are inherited, is UNKNOWN.

## Personality and post-race replies
- Five personalities: Imposing, Honest, Rough, Coward, Sloppy. "Strict" and "Too soft" are not
  personalities. Inherited from both parents. CONFIRMED.
- DO NOT give button-by-button reply advice. Which reply button is which in the game's reply table is
  not verified, the menu has 14 replies, and the game's own messages say the right reply depends on how
  the race went. UNKNOWN. (Older notes printed per-button numbers from a table that was later
  retracted.)
- How a post-race reply changes the horse's hearts (or any other bond value) is not established. UNKNOWN.
- "You are using the whip at the wrong timing" is a post-race message about whipping during the race,
  not about when you pressed a reply. CONFIRMED (game text).
- Pasture tells (Rough kicks, Imposing rears, Honest shakes its head, Coward shimmies, Sloppy lies down)
  come from the export handbook and are untested. COMMUNITY CLAIM (export handbook).
- "My horse doesn't feel like training": the game has those messages; what triggers them is UNKNOWN.

## Legend horses (DOC 2000)
- DOC 2000's ROM holds a legend table of 11 records: 10 real JRA champions (Abukuma Poro, Air Groove,
  El Condor Pasa, Grass Wonder, Silence Suzuka, Special Week, Seiun Sky, Taiki Shuttle, Tokai Teio,
  Narita Brian) plus a flat 50/50/50 "Tonight Two", believed to be a test entry. The table is not in
  World Edition. CONFIRMED.
- In the ORIGINAL game the legends are not breeding stock. On this SITE the 10 real legends can be bred
  in the Lab's "All versions" catalog (starred); Air Groove is the only legend dam. SITE.
- Air Groove's legend record: Stamina 40, Speed 49, Sharp 37, externals △○◎◎△◎ (three ◎),
  Stretch-runner. CONFIRMED.

## Same horse, different names (DOC 2000, World Edition, Rev D)
- Seven legends also have an ordinary DOC 2000 catalog entry identical to a World Edition horse:
  Special Week = Special Holiday, Tokai Teio = Helissio, Air Groove = Hollywood Hills, Grass Wonder =
  Glass Glider, Abukuma Poro = Bubble Boy, El Condor Pasa = El Condor Pasa, Taiki Shuttle = Big Man.
  CONFIRMED.
- The race-opponent roster and the breeding catalog carry DIFFERENT English names for the same Japanese
  horse (Oguri Cap = breeding sire Wild Jaguar but racer Gray Bullet; Air Groove = Hollywood Hills /
  racer Golf of Singapore; Sakura Laurel = Cherry Song / Royal Flush). Breeding questions use the catalog names. CONFIRMED.
- Japanese names: search the katakana exactly as written, or its romaji. CONFIRMED.
- The CPU catalog lists internals in Stamina / Speed / Sharp order; relabel before quoting.

## Rev C vs Rev D, and DOC '99 vs DOC 2000
- World Edition (Rev C) is where the Japanese horses got English names. CONFIRMED.
- Rev C to Rev D: 16 race-roster names changed (14 real names to fictional, plus 2 spelling fixes). In
  the breeding catalog, matched by identity: 2 sires and about 34 dams renamed, 24 sires dropped, about
  30 new horses added, 58 sires and 49 dams unchanged. Never say "all 84 dams were renamed". CONFIRMED.
- Rev D also changed more than names: its track-condition (going) system is split by surface, and it
  adds restricted-race text; Sega's EX material lists new features. CONFIRMED + OFFICIAL.
- DOC '99 to DOC 2000: 64 of 244 race-roster slots hold different horses (replacements, not renames).
  CONFIRMED.
- Coats: basic Gray, Chestnut, Black, Bay, Brown (plus Special, White). Nine named special coats come
  from the community card tool; real odds are not decoded. CONFIRMED / COMMUNITY CLAIM.

## Membership: what is free, what is Winner's Circle?
- Free for everyone: racing at the cabinets, a 25-horse cloud stable, breeding at a cabinet, and the
  Starter Lab at https://play.johnreevesiii.com/breeding (pick a Rev C CPU sire and dam, 3 rolls a
  Pacific day, name the foal before its first race). SITE.
- Winner's Circle is a subscription: $8 a month or $80 a year at https://play.johnreevesiii.com/membership .
  It is NOT a donor or invite-only tier. It includes the full Breeding Lab, Studbook with Potential
  Mates, Breeding Advisor, Breeding Planner, Genealogy, Feeding Advisor and a 200-horse stable.
  Breeding your OWN horses online needs Winner's Circle; the free Starter Lab uses CPU parents only. SITE.

## Breeding Lab: which games? What are "greedy" and "outcross"?
- The Lab breeds World Edition Rev C, DOC 2000 and DOC '99 (DOC II has its own Labs). Rev D (Cab C)
  horses can't be bred in the Lab; a Rev D Lab exists but is hidden, with no date. SITE.
- Breeding Planner: "greedy" picks, each generation, the mate whose predicted foal scores best on your
  goal; "outcross" does the same but every Nth generation picks for a second stat you choose. Neither
  looks at relatedness. SITE.
- Naming: a Lab or Starter Lab foal is named on the Lab's result panel, any time before its first race;
  there is no general rename.
  Stable name: the pencil icon beside your name at the top of the site. SITE.

## Stable, Glue Factory, seats, records
- Deleting a horse moves it to the Glue Factory (bottom of /stable). Restore brings it back HELD FOR
  REVIEW (it can race, but can't be a Lab parent until an operator clears it). No member-side permanent
  delete; a cabinet-autosaved foal can only get a removal request. SITE.
- Seats: after leaving, a 3-minute cooldown before playing a DIFFERENT cabinet (returning to your seat
  is always fine). Winner's Circle skips it on Cabs A, B and JP, not on Cab C. When every cabinet is
  full there is one line, and Winner's Circle members are served first in it. SITE.
- Track records reach the board after the house cabinet writes its save (on restart), then about 15
  minutes. Cab C records stay on the cabinet. SITE.

## Breeder's Cup and DOC Bot itself
- Breeder's Cup v2, live since 2026-09-27 at https://play.johnreevesiii.com/breeders : win 3, place 2,
  show 1, G1 title 10, a quarter of a bred horse's foals' points, and a one-time seal bonus when a
  committed foal first wins. Full rules: breeders-cup.md. SITE.
- A DM to DOC Bot is passed to John, not answered. /ask works in any channel; chat answers happen in
  #ask-doc-bot. SITE.

## What we don't know yet (say so, don't guess)
- What each external, and each internal, does during a race.
- Whether foods raise internals or externals.
- Which post-race reply works best, and when.
- The cabinet breed cap, and whether over-breeding changes anything.
- How foods are earned, and how liked and disliked foods are assigned.
- The externals growth curve over a career; the Rev D stat ceiling.
- Special-coat odds; the best distance for a running style.
When a player asks one of these, say it isn't verified yet and that the knowledge base is being
reviewed. If they have evidence, pass it to John.
