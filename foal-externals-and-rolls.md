# Foal externals and the birth "rolls" (byte-exact, decoded from the ROM 2026-08-09)

How a bred foal's external stats are actually set, straight from the ROM foal-build routine. This settles the recurring "do a foal's externals roll?" question.

## There are TWO different external numbers on a foal

Each external (Start, Corner, OOB, Competing, Tenacious, Spurt) exists in two forms:

- **Breeding band (the breeding value, the circles you see; older notes call it the "retirement" value, but it is fixed at BIRTH and retiring does not change it):** this is the FLOOR-AVERAGE of the two parents' bands, and it is STATIC. (Corrected 2026-09-28: label.) It does not roll. A foal's breeding bands are a fixed function of its parents, which is exactly why inbreeding a line toward a target configuration is predictable.
- **Current external (the developed 0-63 value on a fresh foal):** this is `2 x band + a roll of 0 to 3 + an offset` (Start uses -1, the other five use +1), and each external draws its OWN 0-3 roll. THIS is the roll players notice: two foals from the same pairing share the same breeding bands but start with slightly different current externals.

So both things are true at once: the breeding value does not roll, but the fresh current externals do (a small 0-3 wiggle per external, on top of twice the band).

## Internals and dirt (for contrast)

- Internals (Speed / Stamina / Sharp): the base internal is the FLOOR-AVERAGE of the parents, then a soft clamp (subtract 5 above 45, add 5 below 10), a small pedigree bonus when the SIRE and DAM match (both ◎ or both ✕) on 4 or more externals, and a rare (~3%) hidden +5 band that lets elite pairings reach the ~50s. (Corrected 2026-09-28: said "when enough parent/grandparent externals are high"; the second row the game reads is the sire's own externals, and both-low matches count too.)
- Dirt aptitude: the plain floor-average of the two parents. (Label 2026-09-28: this is the site's rule, set by the operator; how the ROM passes dirt on is not fully traced.)

## Practical takeaways

- To lock in a bloodline's circles, breed toward the target bands: the bands are the deterministic average, so they hold generation to generation.
- Do not expect two siblings' current externals to be identical. The small 0-3 roll per external is normal and does NOT change the breeding bands they will pass on.

Common follow-up: no, the current-external roll does not make one sibling a better breeder than another; siblings pass on the same breeding bands.
