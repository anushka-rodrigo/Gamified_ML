# Gamified ML: predicting which video games are highly rated

A first end-to-end machine learning project. Question: **can we predict whether a video game will be highly rated (user rating ≥ 4.0 out of 5) from its metadata alone, and what drives high ratings?**

Short answer: metadata gets you about **2× better than guessing**, no more. Era (when the game came out), platform, genre and publisher/developer size explain most of what the models can see. Adding a critic score (Metacritic) helps a lot, but that information only exists after release.

The full report on the project is accessible in here: [Full project report (PDF)](reports/Gamified_ML_Project_Report.pdf)

## Data

- Source: a scrape of the RAWG video-game database (snapshot around 2019-2020), 474,417 games, 27 columns.
- Only 11,994 games (2.5%) have a real user rating. After requiring at least 10 ratings per game (so scores are not noise) and removing 91 rows with impossible/missing release dates, **8,878 games** remain.
- Target: `high_rated = 1` if rating ≥ 4.0 (26% of games).

## Results

Scores are on games the model never trained on. ROC-AUC: 0.5 = coin flip, 1.0 = perfect. PR-AUC has a "no skill" baseline equal to the share of positive games.

| Setup | Model | ROC-AUC | PR-AUC | No-skill PR-AUC |
|---|---|---|---|---|
| Random split, metadata only | Logistic Regression (tuned) | 0.816 | 0.602 | 0.26 |
| Random split, metadata only | Gradient Boosting (tuned) | 0.829 | 0.607 | 0.26 |
| **Time split** (train ≤ 2015, test 2016-2020) | Logistic Regression (tuned) | 0.707 | 0.407 | 0.20 |
| **Time split** | Gradient Boosting (tuned) | 0.696 | 0.386 | 0.20 |
| Leakage demo (uses post-release columns; NOT a valid model) | Gradient Boosting | 0.94 | 0.85 | 0.26 |
| Metadata + Metacritic (post-release), random split | Gradient Boosting | 0.871 | 0.691 | 0.26 |

Default-threshold behaviour of the gradient boosting model on the random test set (1,776 games, 461 truly high): flags 38% of games, catches 77% of the good ones (recall 0.77), and about half of its flags are really good (precision 0.52).

## Key findings

1. **Era dominates.** Older games rate higher (about 62% "high" for 2000 releases vs about 13% for 2016). This is largely survivorship bias: the old games still being rated are the remembered classics.
2. **Console-era, big-publisher, established-studio games rate higher** than modern PC / indie / casual / many-genre games.
3. **Random vs time split gap.** AUC drops from about 0.83 to about 0.70 when predicting *future* games from *past* ones. This is distribution shift, and tuning does not fix it.
4. **Leakage is dangerous.** Columns like `rating_top` and `added_status_*` push AUC to 0.94, but they are consequences of the rating, not causes.
5. **Models reach the same answer by different routes.** Gradient boosting reads the era from `release_year`; logistic regression infers it from publishers and platforms. Feature importance depends on the model when features overlap.
6. **Ceiling of metadata.** The model misses indie darlings (Overcooked, Nidhogg, Flower) and over-rates beloved retro classics that land just below 4.0. Nothing in the columns says "this game is brilliant".

## Project layout

```
Gamified_ML/
├── data/
│   ├── raw_data/game_info.csv          # original scrape (do not edit)
│   └── processed/model_data.csv        # output of Stage 2 (features + target)
├── models/
│   ├── logreg_final.joblib
│   └── gradboost_final.joblib
├── notebook/gamifiedML.ipynb           # all the work, in stage order
└── README.md
```

## Reproduce

1. Put `game_info.csv` in `data/raw_data/`.
2. Install: `pip install pandas numpy matplotlib seaborn scikit-learn scipy joblib`
3. Run `notebook/gamifiedML.ipynb` top to bottom. Sections in order:
   1. Exploratory data analysis (EDA)
   2. Cleaning and feature building (saves `model_data.csv`)
   3. Splitting, baseline models, leakage demo, ablation
   4. Tuning, threshold, saving models
   5. Interpretation (coefficients, permutation importance, error analysis)
   6. Metacritic experiment
4. Everything uses `random_state=42`, so results should match within about ±0.01.

Saved `.joblib` models only reload with the same scikit-learn version they were saved with.

## Main decisions (and why)

| # | Decision | Choice |
|---|---|---|
| 1 | Minimum ratings per game | 10 (removes noisy scores, keeps about 9k games) |
| 2 | "Highly rated" threshold | ≥ 4.0 (meaningful, 26% positives) |
| 3 | Which columns the model may use | Pre-release metadata only; 11 post-release columns dropped |
| 4 | Bad dates | Drop 91 rows (missing year or after 2020) |
| 5 | Rare categories | Own column only if ≥ 10 games (developers, publishers) or ≥ 30 (genres, platforms) |
| 6 | Studio size | `dev_experience`, `pub_experience` = log of games made in the full dataset |
| 7 | Splitting | Random stratified 80/20 for development, time split as a realism check |
| 8 | Metrics | ROC-AUC, PR-AUC, F1 (not accuracy) |
| 9 | Imbalance | `class_weight='balanced'` |
| 10 | Grey-area feature | Dropped `in_series` (possible leakage, tiny benefit) |
| 11 | Features per model | Logistic regression: 453; gradient boosting: lean 72 |
| 12 | Tuning | 5-fold CV, optimise PR-AUC |
| 13 | Threshold | F1 rule from out-of-fold training predictions (ended up about 0.5) |
| 14 | Final models | Keep both (interpretable + accurate) |
| 15-17 | Interpretation | Importance on held-out data, grouped features, trust agreement between models |
| 18 | Metacritic | Kept out of the main model; run as a separate post-release experiment |

## Limitations

- Snapshot data from 2019-2020; ratings for newer games are immature.
- Only games with at least 10 user ratings are modelled (about 2% of the raw data), so results describe "games people bother to rate".
- Associations, not causes.
- Metadata cannot see game quality; residual errors are mostly indie standouts and retro near-misses.

