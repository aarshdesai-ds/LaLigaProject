# La Liga Statistical Analysis (2015–2024)

A multi-season statistical investigation of La Liga match data comparing three predictive models for team performance: **Betting Odds**, **Pythagorean Expectation**, and **Expected Goals (xG)**.

---

## Table of Contents

- [Overview](#overview)
- [Research Questions](#research-questions)
- [Dataset Description](#dataset-description)
- [Methodology](#methodology)
- [Key Findings](#key-findings)
- [Project Structure](#project-structure)
- [Requirements](#requirements)
- [How to Run](#how-to-run)
- [Results Summary](#results-summary)
- [Ideas for Extension](#ideas-for-extension)

---

## Overview

This project analyzes **3,040 La Liga matches** across 8 seasons (2015–16 through 2022–23) to evaluate three different approaches to predicting team win percentages:

1. **Bet365 Betting Odds** — Market-derived implied probabilities
2. **Pythagorean Expectation (PE)** — A goal-ratio formula originally developed for baseball
3. **Expected Goals (xG) Model** — An advanced football metric based on shot quality

The core question: *which model best explains actual team performance, and do teams systematically outperform or underperform any of these benchmarks?*

---

## Research Questions

- How well do Bet365 implied probabilities predict actual match outcomes in La Liga?
- Can Pythagorean Expectation (borrowed from baseball) predict football win percentages?
- Does xG-based modelling outperform simpler goal-ratio approaches?
- Are there systematic biases in betting markets for La Liga?
- Which teams consistently over- or under-perform their modelled expectations?

---

## Dataset Description

### La Liga Match Data (9 files)
`La Liga 2015-16.csv` through `La Liga 2023-24.csv`

Each file contains ~380 rows (one per match) and 63 columns:

| Column Group | Columns | Description |
|---|---|---|
| Match Info | `Date`, `HomeTeam`, `AwayTeam`, `Div` | Basic match identifiers |
| Full-Time Results | `FTHG`, `FTAG`, `FTR` | Goals and result (H/D/A) |
| Half-Time Results | `HTHG`, `HTAG`, `HTR` | Half-time score and result |
| Shot Stats | `HS`, `AS`, `HST`, `AST` | Total shots and shots on target |
| Disciplinary | `HF`, `AF`, `HY`, `AY`, `HR`, `AR` | Fouls, yellow/red cards |
| Corners | `HC`, `AC` | Corners for each side |
| Bet365 Odds | `B365H`, `B365D`, `B365A` | Decimal odds for H/D/A outcomes |
| Other Bookmakers | `BWH/D/A`, `IWH/D/A`, `PSH/D/A`, etc. | Odds from multiple sportsbooks |
| Asian Handicap | `AHh`, `B365AHH`, `B365AHA` | Asian handicap lines and odds |
| Over/Under | `B365>2.5`, `B365<2.5` | Goals over/under 2.5 market |

### Expected Goals Data (8 files)
`XG 2015-16.csv` through `XG 2022-23.csv`

Each file contains season-level team summaries (20 rows, one per team):

| Column | Description |
|---|---|
| `team` | Team name |
| `matches` | Games played (38 per season) |
| `wins`, `draws`, `loses` | Match outcome counts |
| `goals` | Actual goals scored |
| `ga` | Goals conceded |
| `points` | Actual league points |
| `xG` | Expected Goals (cumulative) |
| `xGA` | Expected Goals Against |
| `xPTS` | Expected Points derived from xG model |

### Squad Valuations
`teams.csv` — Squad market values (in millions EUR) for teams across 5 major European leagues (La Liga, Premier League, Serie A, Bundesliga, Ligue 1).

---

## Methodology

### Model 1: Betting Odds (Bet365)

Implied win probabilities are extracted from decimal odds using margin-adjusted normalization:

```
P(Home Win) = (1/B365H) / [(1/B365H) + (1/B365D) + (1/B365A)]
```

Calibration is measured using the **Brier Score**. Predictions are classified by the highest-probability outcome, and a confusion matrix evaluates accuracy across Home/Draw/Away outcomes.

### Model 2: Pythagorean Expectation

Adapted from Bill James' baseball formula, applied to football goal data:

```
PE = Goals For² / (Goals For² + Goals Against²)
```

This is computed both **game-by-game** (cumulative within a season) and at the **full-season level**. OLS regression of `win_percentage ~ PE` is used to assess predictive power.

### Model 3: Expected Goals (xG) Pythagorean Expectation

The same Pythagorean formula is applied using xG and xGA instead of actual goals:

```
xG_PE = xG² / (xG² + xGA²)
```

This measures whether underlying chance creation quality predicts performance better than actual results.

### Residual Analysis

Residuals are computed for each model:

```
Residual = Model_Predicted_Win% - Actual_Win%
```

Cross-model residual correlations reveal whether modelling errors are shared (systematic mis-assessment of team quality) or independent (model-specific bias).

---

## Key Findings

### 1. Pythagorean Expectation is Remarkably Predictive
- **Season-level R² = 0.886** — explains 88.6% of variance in actual win percentages
- Correlation of **0.941** between PE and actual win %
- Consistent across all 8 seasons (range: 0.82–0.95 per season)

### 2. Betting Odds Are Systematically Biased
- **Mean residual of +0.126** — odds overestimate win probability by ~12.6% on average
- Home team outcomes are consistently over-predicted
- Draws are systematically predicted as home wins
- Brier scores range from 0.53–0.60 across seasons

### 3. xG is Useful but Less Predictive Than Actual Goals
- **Season-level R² = 0.766** (vs 0.886 for actual PE)
- xG PE correlates more strongly with betting odds (0.928) than actual results (0.875)
- Suggests market odds incorporate xG-like process metrics, while actual outcomes reflect execution

### 4. Model Correlation Structure

|  | Actual Win% | Odds Win% | PE Win% | xG Win% |
|---|---|---|---|---|
| **Actual Win%** | 1.000 | 0.844 | **0.941** | 0.875 |
| **Odds Win%** | 0.844 | 1.000 | 0.865 | 0.928 |
| **PE Win%** | **0.941** | 0.865 | 1.000 | 0.917 |
| **xG Win%** | 0.875 | 0.928 | 0.917 | 1.000 |

### 5. Residual Independence by Model Type
- PE and xG residuals are correlated (0.640) — both capture similar "quality gap"
- Odds residuals are largely independent (0.215 with PE) — market errors have different drivers

---

## Project Structure

```
LaLigaProject/
│
├── La Liga Project.ipynb       # Main analysis notebook
├── README.md
│
├── Team_Data/                  # Match-level data by season + squad valuations
│   ├── La Liga 2015-16.csv
│   ├── La Liga 2016-17.csv
│   ├── La Liga 2017-18.csv
│   ├── La Liga 2018-19.csv
│   ├── La Liga 2019-20.csv
│   ├── La Liga 2020-21.csv
│   ├── La Liga 2021-22.csv
│   ├── La Liga 2022-23.csv
│   ├── La Liga 2023-24.csv
│   └── teams.csv
│
└── XG_Data/                    # Expected Goals season summaries
    ├── XG 2015-16.csv
    ├── XG 2016-17.csv
    ├── XG 2017-18.csv
    ├── XG 2018-19.csv
    ├── XG 2019-20.csv
    ├── XG 2020-21.csv
    ├── XG 2021-22.csv
    └── XG 2022-23.csv
```

---

## Requirements

```
Python 3.8+
pandas
numpy
statsmodels
matplotlib
seaborn
beautifulsoup4
jupyter
```

Install dependencies:

```bash
pip install pandas numpy statsmodels matplotlib seaborn beautifulsoup4 jupyter
```

---

## How to Run

1. Clone the repository:
   ```bash
   git clone https://github.com/aarshdesai-ds/laligaproject.git
   cd laligaproject
   ```

2. Install dependencies:
   ```bash
   pip install pandas numpy statsmodels matplotlib seaborn beautifulsoup4 jupyter
   ```

3. Launch Jupyter:
   ```bash
   jupyter notebook "La Liga Project.ipynb"
   ```

4. Run all cells sequentially (`Cell > Run All`).

---

## Results Summary

| Model | Season-Level R² | Correlation w/ Actual Win% | Mean Bias |
|---|---|---|---|
| Pythagorean Expectation | **0.886** | **0.941** | ~0.000 (unbiased) |
| Expected Goals (xG) PE | 0.766 | 0.875 | ~0.000 (unbiased) |
| Bet365 Odds | — | 0.844 | +0.126 (overestimates) |

---

## Ideas for Extension

The current analysis is a strong foundation. Here are concrete directions to take it further:

### 1. Squad Value vs. Performance
`teams.csv` contains squad market valuations for La Liga teams. A natural extension is to regress win percentage (or points) on squad value — and then identify which teams consistently **over- or under-perform relative to their budget**. This would surface "value" clubs (e.g., Athletic Bilbao) and underperforming big spenders.

### 2. Post-COVID Home Advantage Study
The 2019–20 and 2020–21 seasons were played partially or fully without fans due to COVID-19. The dataset provides a rare natural experiment to quantify the **true value of home advantage** by comparing home win rates and odds calibration with and without crowds.

### 3. Betting Market Efficiency & Strategy
The systematic +12.6% overestimation of win probabilities by odds opens the door to a formal **market efficiency test**. Using the PE residuals to find teams whose odds consistently mismatch their goal-based expectations could identify exploitable inefficiencies — a classic sports economics study.

### 4. Team Trajectory Analysis
Rather than aggregating to full-season level, track **how PE evolves game-by-game** for each team and identify hot streaks vs. cold spells. Do teams with early-season high PE maintain it? Does PE converge toward xG PE over the course of a season?

### 5. Promoted/Relegated Team Analysis
Newly promoted clubs often show the most extreme over/underperformance vs. expectations. Isolating teams in their first La Liga season and testing whether PE, xG, or odds better accounts for their adjustment period is a natural analytical angle.

### 6. Optimal Pythagorean Exponent
The standard formula uses exponent 2. For baseball the optimal is ~1.83; for basketball it's ~14. **Fitting the optimal exponent empirically** using this La Liga dataset (via grid search or MLE to minimize residuals) could produce a football-specific version that outperforms the generic formula.

### 7. Combining All Three Models (Ensemble)
Build a **weighted ensemble** of PE, xG PE, and odds to produce a single win probability estimate. Given their different biases and correlation structures, a combined model might outperform any individual predictor. A simple linear regression with all three as inputs is the natural starting point.

### 8. Cross-League Comparison
`teams.csv` covers Premier League, Serie A, Bundesliga, and Ligue 1 teams. Adding match data from those leagues and running the same PE analysis would allow a **cross-league comparison**: is PE equally predictive across different playing styles and league structures?

### 9. Identifying "Lucky" vs. "Good" Teams
The residual scatter (`xG_PE residual` vs `PE residual`) already hints at this taxonomy. Teams that beat both PE and xG expectations are likely genuinely strong; teams that beat PE but not xG may be "lucky" (overperforming their underlying chances). Formalizing this season-by-season reveals consistent team identities.

### 10. Machine Learning Extensions
Replace OLS with tree-based models (Random Forest, XGBoost) using all available match statistics — shots, corners, fouls, cards, odds lines — to predict full-season performance. Comparing feature importances against the simple PE formula would test whether any in-game metric meaningfully adds information beyond goals scored and conceded.

---

*Data sourced from [football-data.co.uk](https://www.football-data.co.uk) and [understat.com](https://understat.com). Analysis covers La Liga seasons 2015–16 through 2022–23.*
