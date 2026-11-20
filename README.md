# Fantasy basketball team analysis

We built a Jupyter notebook that compares each team in an ESPN fantasy basketball league against the league average, which helped us win our league. For every team, it shows in which of the **9 scoring categories** the roster is above or below average, making it clear which categories to target in trades and pickups.

## How it works

The notebook covers the 9 scoring categories of the league.

- **Percentages:** field goal percentage (FG%) and free throw percentage (FT%).
- **Counting stats:** three-pointers made (3PM), rebounds (REB), assists (AST), steals (STL), blocks (BLK), turnovers (TO) and points (PTS).

The notebook reads the league through the [espn-api](https://github.com/cwendt94/espn-api) package and computes the stats in four steps.

1. **Player stats.** For each player, we take the per-game averages of the current season. When a player has no games yet, we use ESPN's season projections instead.
2. **Team stats.** We sum the averages of all players who are not on injured reserve (IR). FG% and FT% are computed from the summed makes and attempts, rather than averaged per player, which would give low-volume shooters too much weight.
3. **Normalization.** We apply [min-max normalization](https://en.wikipedia.org/wiki/Feature_scaling) to each category, so the team with the lowest value in the league scores 0 and the team with the highest value scores 1. This puts counting stats such as PTS and percentages such as FG% on the same scale.
4. **Delta.** We subtract the normalized league average from the normalized team stats. A positive delta means the team is above the league average in that category.

## Output

The notebook saves one bar chart per team to `out/`. Turnovers are shown as -TO, so a higher bar is better in every category. Green bars mark categories above the league average and red bars categories below it, with a stronger color for a larger delta. The figure below shows the chart for one team in our league.

![Delta from league average for Beverly Hills Larpers](out/Beverly%20Hills%20Larpers.png)

## Setup

1. Install the dependencies.

   ```bash
   pip install espn_api python-dotenv matplotlib jupyter
   ```

2. Copy `.env.example` to `.env` and fill in the values.

   ```
   LEAGUE_NAME=Your league name
   LEAGUE_ID=123456789
   YEAR=2027
   ESPN_S2=your_s2_authentication_token
   ESPN_SWID=your_swid_identifier
   ```

   - **LEAGUE_ID** is the number in the URL of your league page.
   - **ESPN_S2 and ESPN_SWID** are needed for private leagues. Open the fantasy home page in your browser and copy the `espn_s2` and `SWID` cookies from the developer tools (Application > Cookies).

3. Open `team_analysis.ipynb` and run all cells. The charts and the number of games played (`games_played.json`) are written to `out/`.

## Website

We publish the charts on [GitHub Pages](https://ta5946.github.io/fantasy-basketball/). A GitHub Actions workflow updates them every day at 11:11 UTC, which is 12:11 in Slovenia. It reads the `.env` values from the repository secrets (`ESPN_S2`, `ESPN_SWID`) and variables (`LEAGUE_NAME`, `LEAGUE_ID`, `YEAR`), set under Settings > Secrets and variables > Actions.

## Limitations

- **Projections early in the season.** Before a player's first game, the notebook uses ESPN projections, so results in the first weeks reflect ESPN's expectations rather than real performance.
- **Relative scale.** Normalization depends on the best and worst team in the league, so a delta of 0.2 in one league is not comparable to a delta of 0.2 in another.
- **Roster size.** Team stats are sums over all non-IR players, so a team with an empty roster spot is shown as weaker than it is.

## Credits

The ESPN data is fetched with [espn-api](https://github.com/cwendt94/espn-api) by cwendt94.
