# What the Japanese Community Knew (1999-2001)

KEY: `community-japanese`
Scope: the knowledge the Japanese DOC community worked out on their own — the stat model, the
breeding law, training/feeding, running style, and the "atari" luck theories — each claim
tagged by confidence and, where our reverse-engineering later checked it, marked CONFIRMED or
DISPUTED. Read `community-overview` first for the tag system and the load-bearing hedging rule.

The Japanese players worked this game out with no ROM access and no help from Sega, by breeding
thousands of horses and writing the results down on Yahoo! GeoCities Japan and 2ch (now 5ch).
These sources describe DOC (1999), DOC 2000 and DOC 2000 Ver.2 — **not** the Western World
Edition Rev C / Rev D. The Japanese "Ver.2" is not the Western "Rev D"; do not merge them.
Findings recovered and translated on the vetted Japanese wing at
doc.johnreevesiii.com/preservation/japanese. The original work belongs to the period site
authors and posters named there (Y's Lab / 雪之丞, ばあにん☆, 競馬エクウス, すみだDOC, かぐらステーション,
京都DOC, GUTCHI, and the 2ch arcade board, among others).

## The race calendar (CONFIRMED)

The Japanese versions run **19 rounds of 8 races, and every round's 8R is that round's G1**
(the Western releases run to 6R instead). Recovered from a period fan site and since verified
against the game: the player card stores G1 wins as a 19-bit field, one bit per round, and on
a live cabinet a round-15 G1 win flipped the matching card bit exactly as the recovered table
predicts. The Japanese venue names map to the Western forms as 東京 = Eastern City, 阪神 = Western
Hill, 中山 = Northern Park, 京都 = Central City, セガ = SEGA. Four of the nineteen (桜花賞, オークス,
秋華賞, エリザベス女王杯) are fillies'/mares' races a colt cannot enter, matching real Japanese racing.

## The stat model

- **Nine numbers in two families — CONFIRMED.** Every horse has six **externals** (表パラ,
  "front parameters": Start, Corner, Out-of-box, Competing, Tenacious, Spurt) and three
  **internals** (裏パラ, "back parameters": Speed, Stamina, Sharp). The Japanese community
  posted all six externals and all three internals, correctly named, to 2ch on 2000-06-22; our
  ROM carries the same three internal names in a string table and the six externals as a
  contiguous block. Six for six and three for three, from two independent investigations 25
  years apart. (Caution the community itself flagged: the fourth string near the internals,
  "Friendship", is the hearts/bond stat, not a fourth internal.)

- **The visible gauges are 0-63, drawn as 16 blocks of 4 — CONFIRMED.** Period sources
  (ばあにん☆, Y's Lab) wrote "gauge MAX is 64 — really 0 to 63," making the off-by-one explicit
  themselves. Our decode agrees: the write cap in the ROM is 63, and on the physical card the
  six externals are stored as value-minus-one. The "60" everyone remembers is a practical
  ceiling the period build targets top out at, not a code cap.

- **The 16-Step Theory — COMMUNITY CLAIM (good working model, not a separately confirmed
  field).** The most influential idea in the whole Japanese corpus (Y's Lab). Behind each
  on-screen mark sits a hidden value with four levels: ◎ (二重丸) = 13-16 = 3 points, ○ (丸) =
  9-12 = 2, △ (三角) = 5-8 = 1, ✕ (バツ) = 1-4 = 0. So an all-○ horse scores 12 points and a
  perfect horse 18. Our RE has **not** found a separate hidden 4-bit field; the community's
  "16-step value" is most likely the 0-63 external byte divided by four (the block count),
  which makes their inheritance arithmetic and the ROM's the same operation. Treat "16 steps"
  as a good model of a real thing, not a confirmed separate field. Two traps when reading the
  period sources: (1) Y's Lab's 1-16 is deliberately off by one (he says "strictly 0 to 15");
  subtract 1 before comparing to a raw byte. (2) "Points" means two different things — the
  ◎=3/○=2/△=1/✕=0 score (max 18) and the "internal score" 内部点数 (sum of the six 1-16 values,
  runs into the low 60s). A useful worked check: when period sites published exact 1-16 values
  for breeding parents, decoding those same parents from the ROM showed the published *marks*
  were 98-100% accurate but the published *numbers* only 60-72% — the shape of their model was
  right; the digits drifted as they were copied site to site.

- **The birth comment ladder — COMMUNITY CLAIM.** The game prints a comment at birth reading
  the internals: nine tiers, best to worst (すみだDOC recovered it whole), from "A horse that
  can compete with the world" down to no comment at all. The community's own insight that has
  held up: the comment measures only the *sum* of the internals, never their distribution, so
  a 60/10/10 horse and a 27/27/26 horse get the same comment. Tier 1 was believed
  not to exist in practice; tier 3 is what people actually aimed for. A separate retirement
  ladder (seven tiers, Y's Lab, claimed to read win rate) exists but its texts were not fully
  recovered; players named its top tier 費美 ("praise"). Do not confuse the two ladders.

## Breeding: the inheritance law (CONFIRMED — the strongest agreement in the archive)

The single most important thing the Japanese community worked out, and the flagship agreement
of the whole project:

**child = floor((sire + dam) / 2) per stat, fraction discarded.** The Japanese players derived
this by breeding horses and writing down results (June 2000); we derived the identical
operation from the foal-build routine in the ROM 26 years later, neither knowing the other's
answer. The period statement (京都DOC): "(Parameter) = ((sire's) + (dam's)) ÷ 2. Note: the
calculated value is rounded down." One かぐらステーション poster had the internals right in March
2000: "the foal's internals are set at the average of the parents' internals, plus alpha" —
exactly our decode (floor average, then a pedigree bonus of +1 to +3, then a rare band).

Consequences the community built a whole scene on:
- **Every fractional pairing leaks half a point**, so careless breeding makes a line decay.
- **Parents' *current* stats are averaged, not their birth stats (CONFIRMED)** — so racing and
  training a horse up before retiring it genuinely improves what it passes on. This is the
  mechanical basis of 代重ね (daigasane, generational stacking), the strategy the entire
  Japanese scene was built around.
- **The ceiling trap (COMMUNITY CLAIM, follows from the arithmetic):** a 13-grade parent
  (◎, values 13-16) can't be improved because 13 averaged with the best available ○ (12) still
  floors to 12. Breeding to CPU horse スケアクロウ (Scarecrow) "again and again never produces a ◎".

What carries to the foal: six externals (yes, floor average — CONFIRMED); three internals (yes,
floor average then clamped — CONFIRMED); sex (no, a coin flip — CONFIRMED); race record,
earnings, G1 titles, coat, silks, markings (no, separate card fields — CONFIRMED). Dirt/heavy
aptitude carries but not by simple averaging (DISPUTED in detail, see below).

## Dirt and heavy-going aptitude (DISPUTED — the messiest area in the corpus)

The Japanese community gave at least three incompatible accounts of what aptitude even is
(one of the internals; on the same 16 steps as externals; or something else). Our ROM decode
says aptitude lives in bitfields inherited bit by bit with coin flips and thresholds, so it can
*jump*, not just blend. The one thing everyone agreed on, and the useful part: aptitude is a
graded quantity every horse has ("not 'has it' or 'doesn't', but 'high or low'"), it is
heritable, and it decides dirt races. A specific COMMUNITY CLAIM (京都DOC) is the "branch rule":
own×own and CPU×CPU take the floor average, but own×CPU *adds* the CPU horse's bonus to your
horse's aptitude — which would explain why the period strategy of stacking CPU dirt horses
ratchets aptitude upward without limit. Unverified and worth testing. (Note there is also an
internal inconsistency on our own side: the arcade's flat-dirt-number model has matched live
foals, yet the ROM decode says the flat number is not what is inherited — unresolved.)

## Training, feeding, and the damper

- **The training menus — COMMUNITY CLAIM.** Regular structure: each solo menu raises one
  external a lot; each paired menu raises two (one a lot, one a little). Solo turf → Start;
  solo dirt → Tenacious; solo Wood → Corner; solo hill → Spurt; the paired and pool menus
  raise combinations. Whether **Rest** raises stats at all is DISPUTED — one author contradicts
  himself on it; unverified.
- **The training grades and payouts — COMMUNITY CLAIM.** Perfect (exactly on target time / a
  dead heat), Great Success (within ±0.20s / a head), Success (within 0.8s / a length),
  Failure (missed by 0.8s+). A career-start solo Perfect pays about 11 gauge, Great Success 9,
  Success 6. The two best sites independently agreed that aptitude changes the payout (a solo
  Perfect runs ~8 gauge at the low end up to ~14 at the high end) — the strongest internal
  corroboration in the archive, agreeing to within one point. Practical lesson that follows:
  train the stats your horse is already good at, and use food for the ones it is not.
- **Why a horse stops improving — CONFIRMED (and the community had the cause right in effect).**
  Every period source noticed growth stops and none could measure it ("correcting ability
  through training cannot be done forever," 競馬エクウス, 1999; a 2ch thread: "with nothing but
  Perfects it stops growing from race 10-12"). Our decode: the gain coefficient falls from 1.0
  to 0.10 as the sum of the six externals climbs from 220 to 300. It is not a budget you spend,
  it is a tax on how strong you already are — Perfects early get you to a high total early,
  which is exactly why growth appears to stop. One player (GUTCHI, Feb 2000) called the cause
  correctly: a high Out-of-box is a disadvantage "because when Out-of-box is high, the speed of
  parameter gain dulls sooner" — four months before anyone else.
- **Failure used on purpose — COMMUNITY CLAIM (with a CONFIRMED sting).** Failing costs a flat
  amount and drops specific stats. Because running style is re-derived from current stats every
  race, players deliberately *failed* a chosen menu to bring one stat back down and reset a
  horse's style. The sting they never found: negative deltas bypass the growth damper entirely
  and are amplified ×2 or ×3, so late in a career one bad training can cost several sessions of
  gain.
- **Hearts — the mechanic nobody in period got right (CONFIRMED, our biggest edge over them).**
  The community treated hearts as a bond/whip-response meter with no numeric thresholds. In
  fact hearts *gate internal-stat growth* in the once-per-race settlement: the scale is 0-63
  with thresholds around 24/36/42 (roughly 6/9/11 displayed hearts). Low hearts quietly rot a
  horse — crossing a threshold can flip a +1 into a -1, or -1 into -3. (Honest caveat that
  should travel with this: from race 20 on, a code quirk means only about 6% of races move
  anything at all, heavily damping the ladder's practical impact.)

## Food (CONFIRMED direction, COMMUNITY CLAIM specifics)

**Food moves the externals; it does not touch Speed, Stamina or Sharp — CONFIRMED live on a
cabinet.** This settles a period argument: it matches the Japanese food tables (which have no
internals column) and the 2ch consensus by 2001 ("the meters rise, but no 裏パラ are added"), and
it *contradicts* a June 2000 post crediting certain foods with raising breeding ability — that
June 2000 claim is wrong. The specific 44-entry DOC 2000 food table (競馬エクウス) is COMMUNITY
CLAIM: the strongest single feeds (Big Korean Ginseng, Super Mamushi Dango, Sugar Cube,
Vegetable Salad, Big Banana, etc.) are gated behind multi-step campaigns (win several G1s of
different kinds, then a back-to-back Great Success), not single results. A useful design note:
several foods move exactly one external (carrot → Start, fodder → Corner, cabbage → Tenacious),
which makes a horse's food *dislikes* a control mechanism, not a drawback — you cannot stop a
horse that eats everything from drifting off level. **Beer (生中 / 黒生, IDs 43/44) is a CONFIRMED
oddity:** real feedable foods whose entire stat payload is all zeros — the only foods in the
game that do nothing at all except play a reaction animation. The Japanese sources that credited
beer with raising stats/aptitude were DISPUTED at source and are wrong.

## Running style (CONFIRMED — the flagship agreement)

Running style is **not a stored property**; it is re-derived from the stats before every race,
from a single comparison: **among the five externals excluding Corner, where does Start rank?**

| Start's rank among the 5 non-Corner externals | Japanese | Style |
|---|---|---|
| 1st | 逃げ | Front-runner |
| 2nd | 先行 | Stalker |
| 3rd | 差し | Closer |
| 4th or 5th | 追い込み | Deep closer |
| all six equal (Corner included) | 自在 | Almighty |

Corner is excluded from the ranking but included in the all-equal test — an odd rule nobody
would invent twice by accident. ばあにん☆ published exactly this in June 2000; we derived the
same rule from the ROM 26 years later without knowing his page existed. Two details his page
also carries that our decode has: Start wins ties, and a horse with Start at MAX is always a
front-runner unless it is Almighty. Players used this deliberately — because a failed training
drops known stats by a known amount, you can push a horse back to the style you wanted (advice
from 2ch: "stop about 3 dots short of your target so one carrot can still flip the leg type").

The period **Almighty builds** (all six externals equal, ~48 each) were the pinnacle of
Japanese play — reached either by the "precision build" (built on food dislikes and deliberate
failure, using the style readout as an instrument to compare six numbers the player can't see —
a remarkable practical confirmation of the rank rule with no byte access) or by
"banana-pickling" (バナナ漬け): feed Big Banana (uniform +1 to all six) 20-26 times until the whole
set converges on the ceiling together, because clamped stats stop rising while lagging ones
catch up. The board used Big Banana as the instrument without ever knowing the underlying rule.

## "Atari" luck (DISPUTED — the ROM settles it, closer to one theory but neither exactly)

当たり馬 (atari, "a hit" like a winning lottery ticket) was the period name for a foal that came
out stronger than its inheritance implied. The community had two incompatible theories: (A) a
per-bloodline variance band with a high roll = atari; (B) a hidden multiplier (normally
1.00-1.25, atari 1.50-1.75, ceiling 2.55 — that 2.55 = 255/100 is suspicious). **CONFIRMED
resolution:** the foal build rolls a random byte after averaging and hits two narrow ~3% windows
— a DOWN band (one internal -12, or all three -5) and an UP band (one internal +12, or all three
+5), with ~94% getting the clean floor average. So roughly 3% of foals are jackpots and 3% are
duds; the rest is pure arithmetic. There is no per-bloodline band and no persistent multiplier —
your "ruined" foal was usually the dud roll, not your pairing. But there **is** a real,
CONFIRMED bloodline bonus and it rewards *matching*: the game scores how consistent the parent
and second-line external rows are, and a well-matched pair earns a small internals bonus, then
internals are hard-capped at 45. The community sensed a "favourable pairing" effect — it is
real, and it is specifically about matching external profiles (breed like to like).

- **"Total-54 theory" (COMMUNITY CLAIM, only partly true):** Y's Lab held every breeding-stock
  horse sums to 54 across its six values (revised to 56 after Ver.2). Tested against his own
  recovered tables: 54 and 56 really are the two most common totals, but the real distribution
  runs 42-59. Treat 54 and 56 as modes, not a rule.
- **The monoculture the community lived (field evidence for the arithmetic):** repeated floor-
  averaging toward a fixed strong parent converges. A March 2001 2ch post: "Train your
  Seiun-para properly and stack it 30 generations. That horse and another person's, stacked the
  same way, will have exactly the same ability. You cease to feel any originality at all." Six
  of eight horses at one tournament were the same Seiun Sky spread — exactly what the confirmed
  floor-average math predicts.

## Version-specific: the Ver.2 nerf (COMMUNITY CLAIM, JP Ver.1 → Ver.2 only)

Under DOC 2000 Ver.2, a horse whose marks sum to 14 points or more is knocked down a whole mark
band across all six externals at retirement — but only if it has been *raced* on Ver.2. A parent
never used on Ver.2 still breeds at full strength; race it once on Ver.2 and it permanently
drops. And the drop is a filter in the new ROM, not written to the card — take a dropped horse
back to a Ver.1 cabinet and it breeds at full strength again. (This is a Japanese Ver.1 → Ver.2
event, NOT the Western Rev C → Rev D event — do not conflate them. What it legitimately shows:
Sega did ship band-shifting nerfs mid-life, as a transform at retirement plus an import filter.)

## The open questions the Japanese community left

The claims worth the most testing are where the community contradicts itself or the ROM: whether
the dirt-aptitude branch rule is real, whether Rest raises stats at all, and why a Big-Banana
Almighty draws longer betting odds than a Ginseng-fed one with identical visible gauges (the
community noticed the odds seem to leak hidden state). Each is answerable on a cabinet.
