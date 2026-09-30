# Raw dataset: `game_info.csv`

A table of video games with metadata and player-generated statistics. This is the **untouched original data** for the Gamified ML project. Do not edit it: all cleaning happens in the notebook, and the cleaned output goes to `data/processed/`.

## At a glance

| | |
|---|---|
| File | `game_info.csv` |
| Size | about 76 MB |
| Rows | 474,417 (one per game) |
| Columns | 27 |
| Row key | `id` (unique) |
| Encoding / format | UTF-8, comma-separated, header row |
| Snapshot period | Records last updated between 2019-01-09 and 2020-12-22 (from the `updated` column) |
| Release dates | 1962-01-01 to 2033-01-03 (see known issues) |

## Provenance

The column names match the fields of the **RAWG video game database API** (rawg.io), so the data appears to be a scrape of RAWG from around 2019-2020.

- Original source / download link: **[fill in]**
- Date downloaded: **[fill in]**
- Licence / terms of use: **[fill in and check the source's terms before redistributing]**
- If you re-publish this data anywhere, credit the original source.

> The column descriptions below were written from inspecting the data and from the meaning of the matching RAWG fields. Anything marked *(inferred)* is my best interpretation, not something the file states.

## Conventions used in the file

- **Multi-value columns** (`platforms`, `developers`, `genres`, `publishers`) hold several values in one cell, separated by `||`. Example: `Adventure||Puzzle`. They must be split before use.
- **A rating of `0` means "not rated by anyone", not "terrible".** Only about 2.5% of games have a real rating.
- **Zero-filled counts.** Most count columns are `0` for the vast majority of games (see the "% zero" column), because most games in the database are obscure and have no player activity.
- **Dates** in `released` are `YYYY-MM-DD` text. `updated` is an ISO timestamp (`YYYY-MM-DDTHH:MM:SS`).

## Column dictionary

| Column | Type | Meaning | Missing | Notes |
|---|---|---|---|---|
| `id` | integer | Unique game ID in the source database | 0% | Unique. Range 1 to 525,551 |
| `slug` | text | URL-friendly version of the name | 0% | **Not fully unique** |
| `name` | text | Game title | 0% | 2 duplicated names |
| `metacritic` | float | Metacritic critic score (0-100) | **99.0%** | Only about 4,700 games have one. Median 75 |
| `released` | text (date) | Release date | 5.1% | Includes future dates up to 2033 |
| `tba` | boolean | "To be announced" flag | 0% | `True` for 2,341 games |
| `updated` | text (timestamp) | When the record was last updated in the source | 0% | Range 2019-01-09 to 2020-12-22 |
| `website` | text (URL) | Official website | **86.3%** | |
| `rating` | float | Average user rating, 0-5 | 0% | **`0` = no rating (97.5% of rows)** |
| `rating_top` | integer | The highest / most common star value users gave, 0-5 *(inferred)* | 0% | 0 when unrated. **Derived from the ratings themselves** |
| `playtime` | integer | Average reported playtime, in hours *(inferred)* | 0% | 95.1% are 0. Max 1,600 |
| `achievements_count` | integer | Number of achievements the game has | 0% | 96.3% are 0. Max 12,322 |
| `ratings_count` | integer | Number of users who rated the game | 0% | 92.3% are 0. Max 4,289 |
| `suggestions_count` | integer | Number of "similar games" suggestions linked to the game *(inferred)* | 0% | Only 7.9% are 0. Max 1,839 |
| `game_series_count` | integer | Number of other games in the same series | 0% | 99.4% are 0. Max 28 |
| `reviews_count` | integer | Number of written reviews | 0% | 92.1% are 0. Max 4,334 |
| `platforms` | text (multi) | Platforms the game is on | 0.8% | 51 distinct values (PC, Web, iOS are the most common) |
| `developers` | text (multi) | Studios that made the game | 1.8% | About 215,000 distinct names |
| `genres` | text (multi) | Genre tags | **21.7%** | 19 distinct values (Action, Adventure, Puzzle are the most common) |
| `publishers` | text (multi) | Publishers | **70.3%** | About 42,000 distinct names |
| `esrb_rating` | text | ESRB age rating | **88.2%** | Everyone 10+ (36,682), Teen (10,031), Mature (4,859), Everyone (3,837), Adults Only (405), Rating Pending (50) |
| `added_status_yet` | integer | Users who added the game as "not yet played" *(inferred)* | 0% | 94.9% are 0 |
| `added_status_owned` | integer | Users who mark it as owned | 0% | 87.5% are 0 |
| `added_status_beaten` | integer | Users who mark it as beaten | 0% | 94.2% are 0 |
| `added_status_toplay` | integer | Users who plan to play it | 0% | 94.1% are 0 |
| `added_status_dropped` | integer | Users who dropped it | 0% | 95.0% are 0 |
| `added_status_playing` | integer | Users currently playing it | 0% | 98.0% are 0 |

## Known issues and quirks

1. **Most games have no rating.** Only 11,994 of the 474,417 games (2.5%) have `rating > 0`. Any analysis of ratings is really an analysis of this small, self-selected minority.
2. **Small-sample ratings are noisy.** Among rated games the median `ratings_count` is 19. A rating based on 5 votes is far less reliable than one based on 500.
3. **Impossible or placeholder dates.** `released` goes up to 2033, and some dates are in the future relative to the snapshot. Treat dates after about 2020 as unreleased or placeholder.
4. **Missing dates.** About 5% of rows have no release date.
5. **Post-release columns are consequences of ratings.** `rating_top`, `ratings_count`, `reviews_count`, `playtime` and the `added_status_*` columns are created by players *after* a game is out, and are partly caused by how much they liked it. Using them to predict ratings is data leakage.
6. **Heavy missingness.** `metacritic` (99%), `esrb_rating` (88%), `website` (86%) and `publishers` (70%) are mostly empty.
7. **Duplicates.** `id` is unique, but `slug` is not, and 2 game names appear twice. Check before joining on names or slugs.
8. **Very high-cardinality text columns.** Developers (about 215k values) and publishers (about 42k) are too numerous to encode one-by-one; most appear once.
9. **Survivorship effects.** Old games that appear with ratings are mostly the ones people still remember, which biases comparisons across eras.

## How this project uses the data

| Columns | Use |
|---|---|
| `rating` | Turned into the target: `high_rated = 1` if `rating >= 4.0` (only for rated games with at least 10 ratings) |
| `released`, `genres`, `platforms`, `developers`, `publishers`, `esrb_rating`, `website`, `achievements_count` | Converted into model features |
| `rating_top`, `ratings_count`, `reviews_count`, `suggestions_count`, `playtime`, all `added_status_*` | **Deliberately excluded** from the main model (leakage) |
| `metacritic` | Kept out of the main model, used in a separate experiment |
| `id`, `name`, `slug`, `tba`, `updated` | Reference only (not model features) |

After filtering (at least 10 ratings, valid release date), 8,878 games remain for modelling.

## Quick load

```python
import pandas as pd

df = pd.read_csv("data/raw_data/game_info.csv")
print(df.shape)                              # (474417, 27)

rated = df[df["rating"] > 0]                 # only games with a real rating
print(len(rated))                            # 11994

# split a multi-value column into Python lists
df["genre_list"] = df["genres"].str.split(r"\|\|")
```

## Ethics and limitations

- The data describes games, not individual people, but ratings come from user activity on the source platform.
- Because coverage of rated games is small and skewed toward well-known titles, conclusions should not be generalised to "all video games".
