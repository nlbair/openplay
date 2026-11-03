# openplay

**Open men's soccer event data, with xG on every shot.**

openplay is a free snapshot of match event data: every pass, shot, tackle, interception and more, each with pitch coordinates. It covers **26,536 matches** and **41.6 million events** from the big five European leagues, UEFA club competitions and the major international tournaments, from 2014/15 to 2025/26. Every shot also has an expected goals (xG) value.

This repository holds the data release and its documentation. Two packages read the data with the same functions:

- Python: [openplay-py](https://github.com/nlbair/openplay-py)
- R: [openplay-r](https://github.com/nlbair/openplay-r)

> **Status: v0.1.0 preview.**

## Coverage

| Competition | Seasons | Matches played | With events | Coverage |
|---|---|---:|---:|---:|
| Premier League | 2014/15 to 2025/26 | 4,560 | 4,556 | 99.91% |
| La Liga | 2014/15 to 2025/26 | 4,560 | 4,559 | 99.98% |
| Serie A | 2014/15 to 2025/26 | 4,561 | 4,549 | 99.74% |
| Bundesliga | 2014/15 to 2025/26 | 3,672 | 3,670 | 99.95% |
| Ligue 1 | 2014/15 to 2025/26 | 4,236 | 4,229 | 99.83% |
| UEFA Champions League | 2014/15 to 2025/26 | 1,622 | 1,622 | 100% |
| UEFA Europa League | 2014/15 to 2025/26 | 2,226 | 2,225 | 99.96% |
| UEFA Super Cup | 2014 to 2025 | 12 | 12 | 100% |
| FIFA Club World Cup | 2025 | 63 | 63 | 100% |
| FIFA World Cup | 2018, 2022, 2026 | 232 | 232 | 100% |
| UEFA European Championship | 2016, 2020, 2024 | 153 | 153 | 100% |
| Copa America | 2015, 2019, 2024 | 84 | 84 | 100% |
| Africa Cup of Nations | 2015, 2019, 2021, 2023, 2025 | 240 | 240 | 100% |
| UEFA Nations League A | 2018/19 to 2024/25 | 192 | 181 | 94.27% |
| UEFA Nations League B | 2018/19 to 2024/25 | 162 | 161 | 99.38% |
| **Total** | | **26,575** | **26,536** | **99.85%** |

Champions League and Europa League cover the main phases only (no qualifiers). The 39 missing matches have no event data on the source; they are not scraping failures. They are listed in `known_gaps.csv`. A further 58 matches have event data with a known problem (partial events, an incomplete lineup, or events that disagree with the official score). Those are listed in `data_issues.csv`.

## What's in the release

Each file is a separate download in the [GitHub release](https://github.com/nlbair/openplay/releases). `checksums.json` and `SHA256SUMS.txt` give a SHA-256 for every file.

| File | One row per | Rows |
|---|---|---:|
| `events_<comp>_<season>.parquet` (125 files) | event | 41.6 million |
| `matches.parquet` | match | 26,547 |
| `lineups.parquet` | player in a matchday squad | 1.07 million |
| `player_match_stats.parquet` | player per match played | about 780,000 |
| `team_match_stats.parquet` | team per match | 53,094 |
| `shots.parquet` | shot | about 700,000 |
| `players.parquet` | player | 20,782 |
| `teams.parquet` | team | 422 |
| `competitions.parquet` | competition | 22 |
| `formations.parquet` | formation spell | 245,468 |
| `cards.parquet` | card | 115,499 |
| `coverage.csv`, `known_gaps.csv`, `data_issues.csv`, `qualifier_dictionary.csv` | see the data dictionary | |

Every column is described in [DATA_DICTIONARY.md](DATA_DICTIONARY.md).

## Quick start

**Python**

```python
import openplay as op

shots = op.shots("eng", "2024/25")
events = op.events("eng", "2024/25", columns=["match_id", "type", "x", "y", "player"])
stats = op.player_match_stats("eng", "2024/25")
op.coverage_report("nla")
```

**R**

```r
library(openplay)

shots <- shots("eng", "2024/25")
events <- events("eng", "2024/25", columns = c("match_id", "type", "x", "y", "player"))
stats <- player_match_stats("eng", "2024/25")
coverage_report("nla")
```

**DuckDB, no package needed** (after downloading the files)

```sql
SELECT player, sum(goals) AS goals, round(sum(xg), 1) AS xg
FROM 'player_match_stats.parquet'
WHERE comp = 'eng' AND season = '2425'
GROUP BY player ORDER BY goals DESC LIMIT 10;

SELECT * FROM read_parquet('events_*.parquet', union_by_name = true) LIMIT 5;
```

## Expected goals (xG)

xG comes from a model created by Noah Bair. It is applied to every shot in the event data, and the match and season xG figures are sums of those shot values. Own goals and shots from a team's own half have no xG.

## Things to know

- **Coordinates.** Both axes run from 0 to 100, and every team attacks toward x = 100. To get metres, multiply x by 1.05 and y by 0.68.
- **Minutes.** Minutes are "expanded minutes": they count straight through stoppage time, so a full match is usually 93 to 100 minutes.
- **Scores.** `home_score` and `away_score` include extra time but not penalty shootouts. Shootout kicks are in the events and shots tables (flagged `is_shootout`), but are left out of player and team stats.
- **Repeated event IDs.** A few events share an ID. See `event_id_sub` and the data dictionary.
- **Qualifier columns.** Different competition-seasons can have different `q_` columns. Read them together with `union_by_name` in DuckDB, or with arrow datasets.
- **Source stats.** Stats are counted from the events with documented definitions, so they can differ slightly from figures published elsewhere.
- **Snapshot.** This is a snapshot, not a live feed. 2025/26 is complete; 2026/27 is not included.

## License

The data is licensed under [Creative Commons Attribution 4.0 (CC-BY 4.0)](LICENSE). You can reuse it if you credit "openplay (Noah Bair)". The Python and R packages are MIT-licensed in their own repositories.

Maintained by Noah Bair.
