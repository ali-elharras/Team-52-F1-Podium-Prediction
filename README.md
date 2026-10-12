# Team 52: Formula 1 Podium Prediction (Milestone 1)

For every driver in every Formula 1 race, we predict whether they finish on the podium (top 3), using only information that is available **after qualifying and before the race starts**.

| ID | Name |
|---|---|
| 58-7882 | Ali Amr Hassan Elharras |
| 58-3014 | Ahmed Hany Ibrahim Elhussieny Mashaal |
| 58-1804 | Marwan Yasser Fahmy Gaber Mohamed |
| 58-1753 | Ahmed Mohamed Sayed Ahmed Abdelghaffar |

**Kaggle notebook:** https://www.kaggle.com/code/alielharras/team-52-f1-podium-prediction

## Repository contents

| File / folder | What it is |
|---|---|
| `Team 52 F1 Podium Prediction.ipynb` | The full pipeline (run-all notebook) |
| `Team 52 F1 Podium Prediction Analytical Report.pdf` | The analytical report |
| `requirements.txt` | Python libraries used |
| `data/raw/` | The 14 original Kaggle CSV files (read only) |

- **Notebook:** audit, cleaning, data-engineering questions, feature engineering, models, experiments, explainability and the inference function, in that order.
- **Report:** written answers to the three data-engineering questions with figures, the selected features and why each one is valid before the race, all plots, the model comparison and the known limitations.

## Data

Formula 1 World Championship (1950-2024), Kaggle / Ergast:
https://www.kaggle.com/datasets/rohanrao/formula-1-world-championship-1950-2020

The 14 relational tables (circuits, constructors, constructor_results, constructor_standings, drivers, driver_standings, races, results, qualifying, lap_times, pit_stops, sprint_results, seasons, status) are included in `data/raw/`. The raw files are never modified; all cleaning happens on copies in memory.

- **Unit of analysis:** one driver entering one race.
- **Target:** `podium = 1` if `positionOrder <= 3`, otherwise `0`.

## How to run

**Locally**

```bash
pip install -r requirements.txt
jupyter notebook "Team 52 F1 Podium Prediction.ipynb"
```

Then use **Run All**. The notebook reads the data from `data/raw/` and runs top to bottom in about a minute.

**On Kaggle**

The notebook is published with all outputs at https://www.kaggle.com/code/alielharras/team-52-f1-podium-prediction. To run your own copy, import the notebook, attach the dataset above (or upload the 14 CSV files from `data/raw/` as a dataset) and click **Run All**. The notebook finds the CSV files under `/kaggle/input/` automatically, so no code changes are needed.

## What the notebook does

| Section | Summary |
|---|---|
| 1. EDA before cleaning | Audit of placeholders (`\N` and empty cells), qualifying coverage per season, field size and podium rate over time. |
| 2. Cleaning | Placeholders converted to missing values, dates and lap times typed, 2 impossible qualifying times removed, primary and foreign keys verified. |
| 3. Grain | Exactly one row per (raceId, driverId): Indianapolis 500 and non-starters excluded, shared drives resolved by keeping the best `positionOrder`. 24,571 rows. |
| 4. Data-engineering questions | Front-row conversion by circuit, home advantage controlling for grid position, and mechanical-retirement rates by constructor. |
| 5. Feature engineering | 15 leakage-safe features in 6 groups (grid, qualifying, driver form, constructor form, standings, context), checked with an automatic leakage test. |
| 6. Split | By season: train up to 2019, validation 2020-2021, test 2022-2024. |
| 7. Models | Logistic regression (baseline), random forest, gradient boosting and a shallow neural network, all on the same rows, features and pipeline. |
| 8. Experiments | Feature-group ablation and two preprocessing-order experiments. |
| 9. Explainability | Permutation importance and SHAP (global), SHAP waterfall and LIME (local). |
| 10. Inference | `predict_podium(record)` demonstrated on a complete and a partially missing record. |

## Main results

**Data-engineering questions**

- **Q1:** Monaco (90%), Suzuka (89%) and Yas Marina (86%) convert front-row starts best in 2014-2024. The circuit ranking changed after 2014, and the grid and qualifying definitions of front row give almost the same ranking.
- **Q2:** No home advantage once grid position is controlled for (home odds ratio 0.96, 95% CI 0.81-1.14).
- **Q3:** Lotus F1, Toro Rosso and McLaren had the highest mechanical-retirement rates in 2014-2021, and Alfa Romeo, Alpine and AlphaTauri in 2022-2024. The overall rate fell from 9.5% to 5.8% of starts.

**Models (test set, 2022-2024)**

| Model | ROC-AUC | PR-AUC | F1 |
|---|---|---|---|
| Logistic regression (final model) | 0.936 | 0.708 | 0.685 |
| Random forest | 0.941 | 0.710 | 0.684 |
| Gradient boosting | 0.939 | 0.721 | 0.688 |
| Shallow neural network | 0.921 | 0.660 | 0.556 |

PR-AUC is the primary metric because podiums are rare (about 15% of rows), and F1 is the secondary metric. Logistic regression is the final model because it had the best validation PR-AUC (0.739). Grid position is by far the most important feature group.

## Known limitations

- Qualifying data is missing before 1994 and patchy until 2002, so the models are trained on 2003-2019.
- Regulation changes (2014, 2022) shift which teams are fast, so the first races of a new era are harder to predict.
- The validation and test sets are small (39 and 68 races).
- The explanations show what the model relies on, not what causes podiums.
