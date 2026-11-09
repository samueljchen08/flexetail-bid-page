# How the estimate is built, and where the numbers come from

This document answers two questions: *where does each number come from*, and *how is a range
turned into a planning number*. The short version: it is an assembly-based parametric estimate at
AACE Class 5 (concept screening) accuracy, priced from a cost book in which every line carries a
low / mid / high unit price, a written rationale, a confidence grade, and links to the public
pricing evidence behind it.

## 1. Estimate class and what the range means

Concept-stage estimates are classified by the Association for the Advancement of Cost Engineering
(AACE RP 18R-97). A **Class 5** estimate is made from 0 to 2 percent design definition (a rendering
and a model, which is what we have) and carries an expected accuracy of roughly **-20% / -50% on
the low side to +30% / +100% on the high side**. The tool is explicit about this. Nothing in it is
a bid; it answers "is this a $3M or a $10M job" and tells you which lines to quote first.

## 2. Takeoff: from design to quantities

The estimator never prices "a project". It prices quantities:

| Driver | Example (The Briar) | Where it came from |
|---|---|---|
| Fit-out containers by use | 18 (4 restaurant, 2 restroom, 12 retail) | Renderings + plan diagram embedded in the .skp |
| Shell-only tower containers | 2 towers x 4 = 8 | Plan legend "4 stack/tower" |
| Envelope sq ft, floor sq ft, wall sq ft | 1,230 / 320 / 912 per 40 HC | Container geometry |
| LED video wall sq ft | 4,800 | Counted plaza-facing faces at ~300-380 sq ft each |
| Deck sq ft, guardrail lf, stair flights | 3,000 / 600 / 4 | Second-floor outlines in the plan |
| Site, hardscape, rink, courts | 22,000 / 14,000 / 7,500 / 8,400 | Plan scale: a 40-ft module = 185 px |
| Container-miles | 26 x 400 | NC factory to Atlanta |

Every image-derived count carries a low and high in `examples/the_briar/TAKEOFF_NOTES.md`. An OBJ
export of the SketchUp model replaces the image reading with measured geometry.

## 3. Unit prices: the cost book

`data/costbook.json` has one entry per assembly. Each entry has:

- **low / mid / high** installed unit price (material + labor), national or Southeast US, 2025.
  - *Low*: a favorable quote, a volume purchase, or a simple condition. For containers this is
    Flexetail's own $2,800 quote. For spray foam it is a large open-area job at 2 inches.
  - *Mid*: typical installed cost a competent sub would quote today without negotiation.
  - *High*: premium specification or difficult condition. Heavy storefront glazing, rock in the
    grading, fine-pitch LED, a 4-high tower that needs a braced steel frame.
- **rationale**: how the range was built and what moves a job from low to high.
- **confidence**: `high` when two or more independent public sources agree within ~25%; `medium`
  when sources exist but scatter or need conversion (e.g. $/m2 to $/sqft, or a residential figure
  used as a proxy); `low` when the line rests on one source or on estimator judgment. Low-confidence
  lines are the ones to quote first.
- **sources**: title, URL, the figure the source actually quotes, and the access date. Government
  data (EIA, DOT bid tabs, municipal fee schedules) is preferred, then published national cost
  guides (HomeAdvisor/Angi, Fixr, HomeGuide, Craftsman), then manufacturer and integrator price
  pages, then case studies and trade press. RSMeans is cited where a figure is publicly visible.
  **Flexetail's own quotes override all of the above** where they exist (containers, HVAC).

Two lines come straight from Flexetail's call notes and are treated as ground truth for this
client: one-trip 40-ft high-cube containers quoted at $2,800 to $7,000, and HVAC units at $2,500
to $5,000.

## 4. Factors applied on top of the cost book

- **Region.** Factory lines are multiplied by the factory state's construction cost index and
  field lines by the site state's index, relative to a US average of 1.00. The indexes are taken
  from a public city cost index (see the `region_factors` source in the cost book); Atlanta and
  the Carolinas run below the national average, New York and Boston well above.
- **Finish level** (basic 0.75 / standard 1.0 / premium 1.4) scales interior walls, flooring,
  ceilings, exterior accents, and decks.
- **Glazing level** (standard 0.6 / storefront 1.0 / heavy 1.45) scales storefront and the
  structural cutting that goes with it.

## 5. Markups

Percentages of hard cost: permits and impact fees, general conditions, contingency. Overhead and
profit is a percentage of hard cost plus general conditions. Each percentage is itself a
low / mid / high range with sources.

## 6. From ranges to a planning number

Three numbers are reported and they mean different things:

- **Every line at most likely**: the deterministic mid. A clean point estimate, but it ignores
  that construction risk is asymmetric.
- **P10 to P90 planning range and P50**: a 5,000-run Monte Carlo. Each line is drawn from a
  triangular distribution over its low / mid / high. One additional factor per run, triangular
  over 0.80 / 1.00 / 1.50, is applied to all hard cost at once. This represents the correlated
  risk that line-by-line sampling misses: scope growth as design matures, escalation, schedule
  pressure, unknown site conditions. The 0.80 / 1.50 bounds are the AACE 18R-97 Class 5 band for a project with limited reference data (Table 1: -20% to -50% / +30% to +100% at 80% confidence). Note that the contingency line (10 / 15 / 20%) and this factor overlap in purpose: contingency is the markup a design-builder carries in the bid; the factor is estimate accuracy. Both are reported so the overlap is visible rather than hidden.
  Because every line's high tail is longer than its low tail, the P50 sits above the
  deterministic mid. That is the correct direction.
- **All-low to all-high bounds**: every line at its low, or every line at its high. These are
  not realistic outcomes; they bracket the arithmetic.

## 7. Sensitivity

The tornado chart moves one line at a time from its low to its high with every other line at
mid and ranks the swings. It is the procurement to-do list: the top lines are where a second or
third quote changes the bid the most. On The Briar the LED wall swing is larger than the next
three lines combined.

## 8. Known limits

- Unit prices are national or Southeast averages adjusted by index, not Atlanta quotes.
- The container count and LED area for The Briar are read from images, not measured.
- Soft costs use industry-norm percentages, not this developer's actual permit and fee schedule.
- No schedule, financing, land, tenant improvement, or operating cost is included.
- The cost book is designed to be overwritten by purchase orders. Each line replaced by a real
  quote narrows the range and raises its confidence.
