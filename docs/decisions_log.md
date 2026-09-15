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

## [2026-09-15] Number of clusters (k) for player archetype model

**Decision:** k=12, using KMeans on the 6-component PCA representation.

**Context:** Silhouette score analysis showed a clear statistical
optimum at k=3 (0.278), declining steadily as k increases. However,
k=3 is too coarse for the project's goal — it would likely separate
little more than broad positional groups (bigs/wings/guards), not the
finer playing-style archetypes needed for market inefficiency analysis.

**Data considered:**
1. Re-ran silhouette analysis using only 2 PCA components instead of
   6: the same k=3 > k=8 > k=12 ordering held, confirming the decline
   is structural (a real continuum of playing styles), not noise from
   extra PCA dimensions.
2. At k=12, only 4.6% of player-seasons had negative silhouette values
   (poorly assigned), the lowest of any candidate k tested — improving
   as k increased, even as the overall average silhouette score fell.
3. Cluster size balance checked across k=9 (894–1,579), k=10
   (739–1,556), and k=12 (477–1,433) — all reasonably balanced, no
   candidate produced a dominant mega-cluster or negligibly small ones.

**Alternatives considered:** k=3 (rejected — statistically optimal
but too coarse for the project's scouting purpose); k=9 (close
runner-up — best global silhouette among practical candidates and
largest minimum cluster size, but coarser granularity and a slightly
higher poorly-assigned rate than k=12).

**Outcome:** k=12 selected for the final KMeans model, applied to the
6-component PCA representation (random_state=42, n_init=10). The
choice prioritizes assignment quality and archetype granularity over
maximizing the global silhouette score, anchored loosely to the
interpretive frame of a 12-player NBA roster (not a statistical
justification).

## [2026-09-15] Adding player height as a clustering feature

**Decision:** Add `heightInches` (from Players.csv) as an 18th feature,
re-running PCA (now 7 components, 87.8% variance) and KMeans (k=12
unchanged) on the expanded feature set.

**Context:** Sanity-checking the original 17-feature model against
known players (Curry, Jokić, Embiid, Durant, Gobert, Claxton, Young,
Edwards, Giddey, Johnson) revealed that stylistically distinct
superstars — particularly Jokić and Curry — were landing in the same
cluster. Both share high usage and low assisted-shot percentage
(both create their own offense), but are fundamentally different
archetypes physically and positionally. `rebounds_pg` was already in
the feature set but wasn't a strong enough signal on its own to
separate them, since rebounding varies season to season while height
is a fixed structural trait.

**Data considered:** Re-ran the same player sanity check with height
included. Jokić moved out of the guard-creator cluster into a
dedicated "Franchise Big / Point-Center" cluster shared with Embiid —
a much more basketball-coherent pairing. Gobert and Claxton converged
into the same "Rim Protector" cluster in their established seasons.
Silhouette score decreased slightly (0.1690 → 0.1574 at k=12), a
worthwhile tradeoff for the conceptual separation gained.

**Alternatives considered:** Keeping the 17-feature model as-is
(rejected — the guard/big conflation would misrepresent player type
for the project's core use case); using position flags (guard/
forward/center, also available in Players.csv) instead of continuous
height (not tested — height is more granular and avoids relying on
sometimes-arbitrary positional labels).

**Outcome:** Final feature set = 18 columns (17 original + heightInches),
7 PCA components, k=12. Two of Jokić's most recent seasons (2024-25,
2025-26) still cluster with elite guards due to historically extreme
assist rates for his position — noted as a defensible edge case, not
a modeling failure, since those seasons are genuine playmaking outliers.
