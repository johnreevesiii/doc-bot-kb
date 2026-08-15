# The Three DOC Communities — and How to Read Their Claims

KEY: `community-overview`
Scope: who the three historical Derby Owners Club player communities were (Japan, Hong Kong,
the export/World Edition West), the confidence-tag system used across all of the community
docs, and the load-bearing rule for answering "did community X believe Y". This is the index
doc for the community wing; the per-community detail lives in `community-japanese`,
`community-hongkong`, and `community-english-we`.

Sourced from the project's vetted preservation wings at doc.johnreevesiii.com/preservation
(japanese / hong-kong / english). None of the period findings are ours; they belong to the
players and site authors named on those pages.

## Why this doc exists

The rest of this knowledge base is game mechanics and reverse-engineering: what the game
actually does, verified in the ROM. This wing is different. It records what each historical
DOC community *believed and worked out on their own*, decades ago, with no ROM access. Some
of it later proved exactly right. Some proved wrong. A lot was one person's guess quoted
onward until it sounded like fact. So when a player asks "did the Japanese community think
X?" or "is the Hong Kong internals model real?", the honest answer separates *what a
community claimed* from *what has been confirmed*.

## The three communities

- **The Japanese community (1999-2001)** — the origin. They got the game first (DOC '99,
  DOC 2000, DOC 2000 Ver.2) and worked it out with no source code and no help from Sega, by
  breeding thousands of horses and writing the results down on Yahoo! GeoCities Japan and
  2ch (now 5ch). They built the *instruments*: the 16-Step Theory, the breeding math, the
  trick of reading hidden stats off the betting odds. Most of it lived on sites that are now
  dead; what survives, survives in web archives. See `community-japanese`.

- **The Hong Kong community (2002-2004)** — the bridge. Hong Kong sat on a real language
  border with cheap machines. Their two flagship sites (expertdoc.topcities.com and Small
  Wing's Farm) *relayed* Sega's own official English export FAQ into English, added first-hand
  DOC 2000 testing and the whip-chart craft, and ran the first English-language DOC board.
  Their home game was DOC 2000 (the Japanese edition), not World Edition. See
  `community-hongkong`.

- **The export / World Edition / Western community (2002-2011)** — the West. They got the
  game two to three years late, imported half their knowledge from Hong Kong and Japan, and
  organized the rest around the American export game: a different race schedule, a printed G1
  earnings gate, tournaments, and an eBay card economy. They built the *institutions*:
  handbooks with credit pages, national record tables, tournament rulebooks. See
  `community-english-we`.

One line each: **Japan built the instruments. Hong Kong carried them across. The West built
the institutions.** The reason all three read as one picture and not three separate scenes is
Hong Kong — much of the "agreement" between the Japanese and English records actually runs
through the Hong Kong relay back to a single Sega document, so it is one source, not two
(detail in `community-hongkong`).

## The claim tags (use these verbatim)

Every substantive community claim carries one of four tags so its confidence is visible:

| Tag | Meaning |
|---|---|
| **CONFIRMED** | Verified in the ROM bytes, or in a controlled live test on a cabinet. |
| **OFFICIAL** | Sega's own published statement (brochure, manual, FAQ). Authoritative for what Sega *said and intended* — but marketing copy and FAQs are not a ROM dump, so OFFICIAL is not automatically CONFIRMED. |
| **COMMUNITY CLAIM** | A period community source said it. Not independently verified. It may still be true. Always attributed to *which* community. |
| **DISPUTED** | Sources disagree. Both sides named. Treat as unresolved. |

Every COMMUNITY CLAIM is attributed to one of: **the Japanese community**, **the Hong Kong
community**, or **the export/World Edition (Western) community**.

## THE LOAD-BEARING RULE

**Anything not CONFIRMED or OFFICIAL must be stated as "the <community> community
believed/found...", never as settled fact.** If a claim is COMMUNITY CLAIM or DISPUTED, hedge
it and name the community. Do not launder a period belief into a stated mechanic. Two
supporting rules from the source pages:

- **Repetition is not confirmation.** Much of each corpus traces back to a single anonymous
  "card analysis" quoted onward. Five sources agreeing can be one source, not five
  (the English corpus in particular shares sentences and even typos across sites).
- **Cross-community agreement is not automatically independent.** When a Japanese page and an
  English page say the same thing, check the credit chain — it is often the same knowledge
  relayed through Hong Kong, not two scenes confirming each other. The genuinely strong
  evidence is where a community's claim and the project's independent ROM decode agree,
  because those were derived 25 years apart by completely different methods.

## Key RESOLVED facts (safe to state as settled)

These are CONFIRMED or OFFICIAL and can be answered without hedging:

- **The six export venues** (World Edition, the game most Western cabinets ran):
  **Eastern City, Western Hill, Northern Park, Central City, Southern Park, and Sega** (the
  sixth venue, literally named "Sega", hosts the Super Dirt Grand Prix and the season-closing
  Derby Owners Cup). Community abbreviations: E.C., W.H., N.P., C.C., S.P., SEGA. (Note: the
  official US race schedule lists five; the community consistently used six. Whether "Sega" is
  a distinct venue-table entry or a special label on two races is DISPUTED on the count.)

- **The 16-round export schedule, G1 as race 6.** The World Edition season is a 16-round
  rotation. Each round is six races: 1R (a Handicap), 2R, 3R ("Special", higher purse), 4R,
  5R, and the round's **G1 as race 6**. (The Japanese versions instead run 8 races per round
  with the G1 as race 8 — do not merge the two structures.)

- **The $1,000,000 G1 earnings gate — OFFICIAL, Sega's own print.** The Sega World Edition
  sales brochure (NTRA co-branded, c. 2002), page 7, states verbatim: races 1-5 are open to
  all comers; "Race 6 is reserved for G-1 eligible horses. To qualify for a G-1 race, your
  horse must have lifetime earnings of $1 million dollars or more." The community record
  (the 2004 Handbooks, FunTrivia) matches this exactly. The on-screen face of the gate is the
  string "Grade 1 Eligible". (OFFICIAL and community-corroborated; the ROM check itself has
  not yet been located, so it is not additionally tagged CONFIRMED.)

- **Hong Kong 6R region mode swaps two races — CONFIRMED.** A World Edition machine put into
  Hong Kong 6R mode substitutes two regional variants: the **Hong Kong Derby (race option
  17)** in place of the American Derby (round 7), and the **Hong Kong Oaks (race option 18)**
  in place of the American Oaks (round 6). Same slots, same distances and rules, different
  name — not new races. This is why period sources recorded both "American" and "Hong Kong"
  names for the same two events; it is a region setting, not a dispute.

## What to send a player

- If they ask what a community *believed*, answer from the relevant `community-*` doc and keep
  the tag: "the Japanese community found... (COMMUNITY CLAIM)", etc.
- If they ask whether a belief is *true*, say whether it is CONFIRMED, OFFICIAL, still just a
  COMMUNITY CLAIM, or DISPUTED — and hedge anything that is not CONFIRMED/OFFICIAL.
- If they ask about export structure, venues, the G1 gate, or Hong Kong 6R, use the RESOLVED
  facts above directly.
