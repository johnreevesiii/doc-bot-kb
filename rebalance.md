# Rev C Rebalance (RB01) test edition

Status: LIVE TEST since 2026-10-04 on one community cabinet. Everything here is CONFIRMED against the game program and the arcade code unless marked otherwise. Treat the balance itself as UNMEASURED: the test exists to measure it.

## What it is
- A test edition of World Edition Rev C with three targeted race-engine changes, running on the 8-seat cabinet named "Rev C Rebalance" in the lobby (a community-hosted Host Pack cabinet, not a house cabinet). Open to everyone with a lobby account.
- Build name RB01, mechanics B2 plus S1. Game id on the site: docrebal. Card marker SEGARE01 (stock World Edition cards carry SEGABEF0).
- It is NOT a replacement for Rev C. The house cabinets, Rev C and Rev D horses, leaderboards and the Breeder's Cup are untouched.

## Why (the two findings)
- The dirt rating is two values packed in one byte: upper half H (0-15) leans dirt, lower half L (0-15) leans wet going. 255 = H15 L15. In stock Rev C, H only ever helps (on dirt) and L only ever helps (all dirt races, and soft or heavy turf); on firm turf the dirt rating does nothing. So 255 is never worse than any other number in any race, and breeding collapsed onto the eight founder crosses that reach 255 (Thunder Boy or Wild Sun x Ferranti's Folly, Love J, Lemonade or Princess).
- The Sharp acceleration helper compares the wrong value (one race setup clears to zero), so every horse in World Edition and DOC 2000 gets the same 1.12 multiplier regardless of Sharp. DOC '99 does not have the slip. Sharp still counts in the three-stat total.

## What changed (RB01 B2 + S1)
- Surface and going (B2): H still helps on dirt but now costs the same horse on turf in every going; low L helps firm going (Good, Good to Soft), high L helps wet going (Soft, Heavy). Centred so that averaged over the race program and weather mix all 256 values get the same bonus. Four roles: firm turf (low H, low L), wet turf (low H, high L), firm dirt (high H, low L), wet dirt (high H, high L). 255 keeps wet dirt.
- Sharpness (S1): the helper reads the horse's real Sharp: 1.12 below 20, then 1 + 0.009 per point (1.36 at 40, 1.54 at 60). One acceleration term, not top speed.
- Unchanged: all 168 founders byte for byte, coats, names, breeding arithmetic, race program (27 turf, 9 dirt), course list, weather table, training, care, externals, card layout. 265 bytes changed in the game program.

## Cards and isolation (site rules, CONFIRMED in code)
- Rebalance horses save as their own edition and show a blue balance-scale badge marked RB01 in the stable.
- They load only on the Rebalance cabinet. Rev C, Rev D, combined, JP and DOC II horses are not offered there and are refused if asked. A Rebalance horse is refused on every other cabinet.
- They are off every leaderboard and record board, do not race or score in the Breeder's Cup, cannot be bred in the online Breeding Lab, cannot be used as a Lab parent, and are not listed in the Studbook. Breeding happens in the game from the original founders.
- Stable size counts them like any other horse.

## Where to point people
- FAQ: "Why does everyone breed Thunder Boy?" (slug why-thunder-boy) and "Is Sharp broken?" (slug sharp-bug).
- Dirt Number Map: https://doc.johnreevesiii.com/tools/dirt-number-map.html (all 256 values, stock vs Rebalance).
- The explainer PDF is attached to the 2026-10-04 post in #announcements. Feedback goes in #general with "Rev C Rebalance" in the first line; questions about how something works go to the #faq forum.

## Known issues (as of 2026-10-04)
- A frame counter shows in the corner of each board: a diagnostic overlay, removed at the next game restart.
- One stall at Race 2 seen on 2026-10-03, passed on retry, cause unknown.
- No claim of a finished balance. Report placings and times, not purses.
