# What the Hong Kong Community Knew — the Bridge (2002-2004)

KEY: `community-hongkong`
Scope: the Hong Kong DOC scene as the *bridge* that carried the game from Japanese into English
— what it faithfully relayed from Sega's own English FAQ (so it is OFFICIAL, not a Hong Kong
discovery) versus what Hong Kong genuinely originated (the whip-chart craft, the first English
board, the double-circle origin story). Read `community-overview` first for the tag system.

Recovered from the vetted Hong Kong wing at doc.johnreevesiii.com/preservation/hong-kong,
compiled from the two surviving Hong Kong sites (expertdoc.topcities.com and Small Wing's Farm),
the first English-language board (VoyForums 109792), and Sega's own official export FAQ, cross-
checked against the project's ROM work. The findings belong to the Hong Kong players credited by
their public handles below. This is the newest and thinnest wing; its gaps are stated honestly.

## The one thing to get right about Hong Kong

The other two wings document what a community *knew*. This wing documents a *transmission*. Most
of what the English scene calls "the Hong Kong internals model" **did not originate in Hong
Kong**. It originated inside Sega, in an official English FAQ Sega published for the export game
(hitmaker.co.jp DOCWE faq.html, 2002). Hong Kong's two flagship sites cite that FAQ on every
page — they literally printed "Some information are quoted from DOCWE official web site." They
were the conduit, and an honest one.

**The load-bearing consequence: English-and-Japanese agreement on the internals wording is one
source, not two.** Both derive from Sega's English FAQ, which Hong Kong relayed. When you see the
same internals language in a Japanese page and an English page, do not count it as two scenes
independently confirming each other. Only the parts the Western scene *added* (the Best Right
jackpot method, the numeric rosters, the modern ROM decode) are genuinely downstream-independent.

The transmission chain, end to end: **Sega/Hitmaker Japan** (the ~250-page official DOC 2000
strategy corpus) → **Sega's official English export FAQ (2002)** (carries the Speed/Stamina/Sharp
type descriptions and the "type = highest internal" rule, in English) → **Hong Kong relay
(2002-2004)** (expertdoc + Small Wing's Farm paraphrase the FAQ, add first-hand DOC 2000 testing,
run the first English board) → **US/Western scene (2003 on)** (credit Hong Kong by name, then add
their own).

This does not diminish Hong Kong — it sharpens what they actually did. Their role was relay and
translation, done honestly, plus real original work of their own.

## Case one: the internals model (OFFICIAL — Sega's, relayed by Hong Kong)

The intellectual centre of the page. Hong Kong's flagship site (expertdoc, horseability.htm,
2002) and Sega's FAQ are the same text. The internal set is **Speed / Stamina / Sharp**; a
horse's **"Type" is the highest of the three**; the whip-effect ordering (Stamina lowest whip
effect, Sharp highest, especially in the final dash) and the stamina-usage ordering are Sega's
own wording. Even the abbreviation the whole scene uses, **"SR" for Sharp, comes straight from
Sega's label "SHARP RUNNER TYPE"**.

- Tag: **OFFICIAL** for the whole internals-type model — it is Sega's printed claim, relayed.
- Of it, only the three internal names **Speed / Stamina / Sharp are separately CONFIRMED** in
  the ROM string table by the project's own decode; the rest (the type-is-highest rule, the whip
  orderings) is Sega's statement, not independently ROM-verified here.
- So: the "Hong Kong internals model" the Western scene credited to "the Hong Kong Boys" is a
  near-verbatim paraphrase of an official Sega document. Hong Kong's contribution was to find it,
  read it, and carry it into English — exactly what a bridge is for.

## Case two: the breeding formula (CONFIRMED — Hong Kong overruled Sega and won)

Hong Kong stated the foal formula as flatly as possible (expertdoc breeding.htm):
"RoundDown((S+D)/2)," with a worked example (sire 57, dam 56, foal 55). Small Wing's Farm reached
the same clean floor-average from first-hand testing. But Sega's *own* official Japanese breeding
page had hedged: "This does not simply average the two horses' abilities; the foal is more likely
to inherit the blood of the parent with the stronger heredity," layering on bloodlines and nicks.

**The project's byte-level decode of the export ROM is a deterministic floor average, per stat —
so Hong Kong's simplification is the one that matches World Edition, and Sega's vaguer wording is
the outlier.** The striking result: on breeding, the relay was more accurate than the source it
relayed. (Two things the project's decode adds that appear in no Hong Kong source: the roughly
one-in-thirty-two jackpot and dud bands on foal internals, and the birth caps. Hong Kong had the
clean rule, not the rare exceptions to it.)

## The Hong Kong Boys: who carried it (public handles only)

"The Hong Kong Boys" is the American FAQ author's phrase for the upstream players: "The order is
taken from the info the Hong Kong Boys gave us after two years of figuring it out." Credited by
public handle and public role only:

| Handle | Public role | What they carried or made |
|---|---|---|
| **expertdoc** | The Hong Kong collective behind the flagship site expertdoc.topcities.com and VoyForums board 109792; "experienced players of both DOC 2000 v.2 and DOC World Edition" | The single most-credited contributor in the scene. Ran the internals, breeding, racing, whip, record and location pages; posted daily "Updated!" changes 2002-2004. |
| **Wing / "Small Wing"** | The DOC 2000 breeding master; his site is "Small Wing's Farm" (doc2000.topcities.com/docwe/) | Author of the eBay-sold "Wing's Menu" (the Great Escape front-runner chart plus last-spurt and whip-point tables). Ran his own first-hand breeding tests; his ruling was treated as canon: "After 20th generations there is NO difference. Just basis on you LUCKY to born the GOOD horse." |
| **Mr.P / "Mr. Puckman"** | The second revered Hong Kong expert, near-always paired with Wing on the boards | Contributed the time-conversion tables and the "Pescape," a dirt-course variant of the Great Escape front-runner technique. |
| **April Stable** | Hong Kong player active late 2002 | Posted the first Hong Kong track records (2002-11-18), the seed set for expertdoc's Hong Kong records column. |
| **Tim Howard** | A verified board handle (posts dated 2003-08-21) and a menu/whip-chart author alongside Wing | COMMUNITY CLAIM: that he was a real contributor is settled from primary capture; who he was beyond the handle is not, and is not published. |

Note: "Small Wing's Farm" was long assumed a lost separate domain; it is actually Wing's own
doc2000.topcities.com/docwe/, which is why Wing's first-hand breeding tests now read as Hong Kong
evidence rather than orphaned pages.

## What Hong Kong genuinely originated

Subtract what Hong Kong relayed from Sega and a real body of original Hong Kong work remains:

- **The whip-chart craft — COMMUNITY CLAIM (Hong Kong origination).** Hong Kong's native art
  form; the Western scene credited it inline by name. The canonical front-runner meta chart,
  "The Great Escape," carries the line "Thanks to expertdoc in Hong Kong" on the American sites.
  "Wing's Menu" was the single most-referenced purchasable chart in the scene. expertdoc's
  lastspurt.htm gives a full 12-distance whipping-point and final-dash-point table. Important
  caveat: the project's confirmed whip findings mean the charts cannot work for the mechanical
  reason the scene thought — but the charts prescribe *spaced whips at counted intervals*, which
  is what a refractory whip lockout actually rewards, so the technique may have been right while
  the theory was wrong. Either way, the craft of writing them down per distance and per leg type
  is a Hong Kong invention.
- **The first English-language board — COMMUNITY CLAIM.** expertdoc's VoyForums board 109792,
  opened 2002, is the first English-language DOC board on record. Before doc.rbcb.net (August
  2003) became the American hub, this Hong Kong board was where the English scene actually talked.
- **The double-circle origin story — COMMUNITY CLAIM (mechanism testable).** The origin of the
  export game's most famous artefact is a Hong Kong story: the all-double-circle "glitch
  aristocracy" horses "came into existence in Hong Kong." Hong Kong players took DOC 2000 v.2
  horses with all-✕ stats and ran them on World Edition machines, where the stats wrapped past the
  bottom into all double circles, then spread worldwide by eBay. A first-hand 2002 board post:
  "that [double-circle horse] is from Hong Kong and bred from Japanese version cards." The
  wraparound mechanism is directly testable against the card byte format and is on the RE list —
  so treat the mechanism as claimed, not confirmed.
- **The special-horse (coat) odds table — COMMUNITY CLAIM (Hong Kong origination).** expertdoc's
  special.htm is the origin of the colour-horse odds table the Western scene later repeated: eight
  colours (White, Silver, Tiger, Zebra, Cow, two Pandas, African Deer) with breeding odds from
  CPU×CPU 1/256 up through Self×CPU 1/64, Self×Self 1/16, CPU×Special 1/8, Self×Special 1/4, to
  Special×Special 1/2, and the rule that if either parent is special the foal is special or grey.
  Checkable against the ROM; a Hong Kong origination, not a relay.

## What Hong Kong did NOT originate

- **The Best Right jackpot method is a US addition, not Hong Kong — COMMUNITY CLAIM (US origin).**
  The American scene's trick for reading a foal's hidden internals off a benchmark CPU horse is
  absent from every Hong Kong source (expertdoc's cards.htm is an empty stub). Getting the
  direction of that arrow right is the same discipline that makes the internals finding honest:
  relay where it was relay, invention where it was invention. (Detail in `community-english-we`.)

## The Hong Kong record, in primary data (COMMUNITY CLAIM)

- **The two Hong Kong arcades** (expertdoc's location.htm): Silvercord (新港中心) in Tsim Sha Tsui
  (尖沙咀), HKD 10 / 3 credits (~$0.43/race); GoldStar in Tsuen Wan (荃灣), HKD 10 / 5 credits
  (~$0.26/race). US venues in the same records ran roughly $2.31-$3.50 a race — one plausible
  material reason the deepest early testing (Wing's generation runs, the whip-point tables) came
  out of Hong Kong: the game was simply cheaper to study there.
- **The Hong Kong records column** (expertdoc's record.htm, seeded by April Stable's 2002-11-18
  posts): a full column across all six venues, names a mix of English and in-jokes (Big Rock,
  Space Sega Pig, Blissful April, April Athena, Big General, and more). A sharp first-hand caveat
  from April Stable posting the region's first records: "AND DOUBLE CIRCLE HORSE is not that
  useful, all horses above are not double circle horse" — on-the-record scepticism about the
  export's famous glitch aristocracy, from the place the glitch horse came from.
- **The last machine (2011) — COMMUNITY CLAIM.** A Cantonese thread on discuss.com.hk records the
  endgame: by 2011 the only DOC 2000 left in Hong Kong stood in a basement arcade (地牢機鋪) at
  Jordan Road and Woosung Street. Because the manufacturer no longer made DOC 2000 magnetic cards,
  players were told to bring their own MTR subway ticket (地鐵車飛) or a scrap Wangan Midnight card
  (灣岸廢車卡) to use as a horse card — the same cause that killed the machines everywhere: not the
  hardware, the cards. A World Edition horse was only ever a few hundred bytes on a magnetic
  stripe, and any stripe would do.

## The key fact for the bot

**Hong Kong's home game was DOC 2000 (the Japanese edition), not World Edition — COMMUNITY CLAIM,
from the recovered forum tail.** Wing on the board: "We do not have any DOCWE in HK... but we do
have a lot DOC2000." The HK experts mastered the Japanese game, read Sega's English export FAQ,
and relayed World Edition knowledge to the West by card and board exchange without playing much
World Edition themselves. That is the bridge in one sentence: a scene fluent in one edition,
documenting another, for a third audience. (The only DOC 2000 machine known in North America, per
the same threads, was in Toronto.)

## Honest gaps (flag, do not fill)

This wing is thinner than its neighbours and still growing. Mr.P (confirmed as "Puckman") and Tim
Howard have real identities resolvable, if at all, only from living memory, not the public web —
neither is published beyond the public handle. A single web summary claims the in-game "SEGA"
venue is loosely based on Sha Tin, Hong Kong's real racecourse — flagged as a lead, not a fact,
and not to be stated as confirmed.
