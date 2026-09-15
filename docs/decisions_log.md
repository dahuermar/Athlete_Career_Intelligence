## [2026-09-15] Minimum games played threshold for season-level clustering

**Decision:** Filter out player-seasons with fewer than 25 games played
in the regular season.

**Context:** Player-season rows built from game-level box scores include
short call-ups, players traded mid-season, and long-term injury cases —
these add noise to season averages and don't represent a real,
observable playing style for clustering purposes.

**Data considered:** Histogram of games_played across all 14,569
player-seasons shows a natural dip in the distribution around 20-30
games, separating a "marginal/injured" population from the "rotation
player" population. 25 games (~30% of an 82-game season) falls inside
that dip.

**Alternatives considered:** Original proposal was a stricter 60%
threshold (~49 games); relaxed to 30% after reviewing the histogram
shape, since the natural break in the data sits lower than the initial
estimate and a stricter cutoff would have discarded legitimate
mid-season trades/debuts.

**Outcome:** games_played_min = 25.

---

## [2026-09-15] Minimum average minutes per game threshold for season-level clustering

**Decision:** Filter out player-seasons averaging under 8 minutes per
game, applied on top of the games_played_min = 25 filter above.

**Context:** Needed to exclude noise from garbage-time appearances
without losing legitimate low-minute specialists (e.g. bench shooters,
defensive specialists) who can be real, consistent role players despite
limited minutes — e.g. Jordan Clarkson-type veterans. External reference
(The Athletic) defines "Deep Bench / Specialist" as under 15 minutes per
game, which suggested an initial 10-15 minute cutoff might exclude
legitimate role players.

**Data considered:** Among player-seasons already passing the
games_played_min = 25 filter (n=11,530), tested thresholds from 5 to 15
minutes:
- 5 min: keeps 11,448 (loses 82)
- 6 min: keeps 11,344 (loses 186)
- 8 min: keeps 11,017 (loses 513, 4.4% of filtered population)
- 10 min: keeps 10,474 (loses 1,056)
- 12 min: keeps 9,799 (loses 1,731)
- 15 min: keeps 8,656 (loses 2,874, 25% of filtered population)

Loss accelerates sharply above 10 minutes, with no comparable natural
break as seen in the games_played distribution.

**Alternatives considered:** 10-15 minutes (rejected — matches the
external "specialist" definition, but the games_played_min = 25 filter
already removes true garbage-time noise; a consistent 25+ game role
player with low minutes is a legitimate specialist archetype, not noise,
so a stricter minutes cutoff would remove exactly the players this
project is meant to surface).

**Outcome:** avg_minutes_min = 8, combined with games_played_min = 25.

## [2026-09-15] Missing value imputation for shooting percentage features

**Decision:** Fill missing values in `tp_pct` (850 cases) and `ft_pct`
(2 cases) with 0, rather than dropping rows or imputing with the mean.

**Context:** Both columns are computed as makes/attempts recalculated
from season totals (see the earlier decision on avoiding averaged
per-game percentages). A null value in either column means the player
had zero attempts of that shot type all season, not a data quality
issue — e.g. traditional interior players who never attempt a
3-pointer, or low-usage role players who never draw a shooting foul.

**Data considered:** All 850 `tp_pct` nulls and both `ft_pct` nulls
were checked against their corresponding `*a_total` (attempts) column,
confirming 0 attempts in every case — no evidence of a data quality
problem or a bug in the aggregation pipeline.

**Alternatives considered:** Dropping these rows (rejected — would
remove legitimate, common playing styles like traditional post players
from the dataset, biasing the clustering); imputing with the column
mean (rejected — would fabricate a shooting tendency the player never
actually exhibited, distorting exactly the kind of signal this
clustering is meant to capture).

**Outcome:** `tp_pct` and `ft_pct` filled with 0 for zero-attempt
player-seasons. All 17 feature columns now
