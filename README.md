# Premier League Bargains & Rip-offs of 2024–25

A data-driven look at which Premier League players over- and under-deliver on their salary, using one season of public performance and wage data.

> **Headline result:** A regression on per-90 performance and share-of-team output explains roughly **32% of the variation in log salary** across 288 outfield regulars. The biggest **bargains** of 2024–25 are Rodrigo Muniz (Fulham) and Myles Lewis-Skelly (Arsenal); the biggest **rip-off** is Marcus Rashford. Manchester United and Chelsea each contribute three players to the top-ten rip-off list. **Read the [full report](PL_bargains_and_ripoffs_report.pdf) for the complete story, methodology, and limitations.**

---

## The question

Premier League salaries are shaped by a long list of factors no public dataset can fully see: commercial appeal, agent leverage, contract timing, and how badly a club wanted a player on the day they signed. We asked a narrower, answerable version of the question: **using only on-pitch performance and team context, which players are paid the most relative to their measurable contribution, and which are paid the least?**

The model is not a verdict on any individual player's quality. It is a verdict on the gap between what a player produces on the pitch and what their club currently pays them, given how the rest of the league is priced.

---

## Data

The three source CSVs in this repository were downloaded from a public [Kaggle bundle](https://www.kaggle.com/datasets/flynn28/2025-premier-league-stats-matches-salaries) covering the 2024–25 Premier League season. The bundle aggregates two genuinely independent upstream collections:

- **Performance data** (`player_stats.csv`, `team_stats.csv`): on-pitch statistics scraped from match-event data and aggregated into season totals.
- **Salary data** (`player_salaries.csv`): weekly and annual gross wages compiled from publicly reported and estimated figures.

The two collections are produced by different organizations and cover overlapping but not identical sets of players, which is what makes the merge the substantive analytical step rather than a formality.

---

## What's in this repository

| File                                 | Purpose                                                                                                                                  |
| ------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `player_stats.csv`                   | Source: per-player season totals (571 player-seasons)                                                                                    |
| `player_salaries.csv`                | Source: per-player weekly and annual salaries (684 rows)                                                                                 |
| `team_stats.csv`                     | Source: per-team season totals (20 rows)                                                                                                 |
| `project_2_data_preparation.ipynb`   | Notebook 1 of 2: dedupes, validates, and merges the three sources                                                                        |
| `PL_players_and_teams_stats.csv`     | Output of Notebook 1: the merged player-level dataset (514 rows × 34 columns) used as input to the analysis                              |
| `project_2_analysis.ipynb`           | Notebook 2 of 2: filters, engineers per-90 and share-of-team features, fits the regression, runs the partial F-test, and ranks residuals |
| `PL_bargains_and_ripoffs_report.pdf` | Self-contained report written for a non-technical audience                                                                               |

---

## How to reproduce

**Prerequisites.** Python 3.10+ with `pandas`, `numpy`, `matplotlib`, `seaborn`, `statsmodels`, and a Jupyter runtime.

**Order of execution.**

1. Open `project_2_data_preparation.ipynb`. Restart the kernel and run all cells. This reads the three source CSVs, dedupes mid-season transfers (matching on the pair of player name _and_ year of birth to guard against namesakes), validates the merge with a strict 1:1 cardinality check, and writes `PL_players_and_teams_stats.csv`.
2. Open `project_2_analysis.ipynb`. Restart the kernel and run all cells. This reads the merged CSV, filters to outfield regulars with at least 900 minutes played (≈ 10 full matches, the standard footballing threshold for stable per-90 rates), engineers the predictors, fits two nested regressions, runs the partial F-test, and produces the bargain and rip-off leaderboards.

Both notebooks produce all of their figures and tables inline. The final report (`PL_bargains_and_ripoffs_report.pdf`) summarizes the analysis for a non-technical audience and is the recommended starting point for readers who do not want to run the code.

---

## Method in brief

We model the **logarithm of annual salary** as a function of three groups of predictors:

1. **Playing-time and demographics.** Minutes played, age, primary position.
2. **Per-90 performance rates.** Goals, assists, progressive passes, and progressive carries, each expressed per 90 minutes played so high-minute and low-minute players are compared on the same scale.
3. **Share of team output.** What fraction of the player's team's goals, assists, expected goals, progressive passes, and progressive carries did this player contribute? These features require the team-level data to compute, which is what makes the team merge structurally necessary.

We compare two nested versions of the model. The **baseline** uses only individual stats and explains 23.2% of the variation in log salary. The **full model** adds the share-of-team measures and lifts that to 31.6%. A partial F-test confirms the team-share measures collectively add real explanatory power (**F(5, 274) = 7.82, p ≈ 6.7 × 10⁻⁷**, roughly a 1-in-1.5-million chance of arising by random luck).

Each player's **residual** in the full model is the gap between their actual log salary and the model's prediction. Negative residuals are bargains; positive residuals are rip-offs. The top ten in each direction are the headline outputs of the analysis and appear in the report.

---

## Caveats worth knowing before reading the report

- **One season only.** A player who looks under-paid here may be on a contract signed years ago when they were less productive; the data cannot see that history.
- **Goalkeepers are excluded.** The dataset has no goalkeeper-specific statistics, so they cannot be fairly evaluated on outfield-style contribution metrics.
- **Salary figures are third-party estimated.** Premier League clubs do not publish exact wages, so the salary values are compiled from publicly reported and estimated figures. Treat the rankings as directionally informative, not penny-precise.
- **The model only sees on-pitch contribution.** Real-world salaries also reflect commercial value, contract leverage at signing, and squad needs at the time of the deal. Players flagged as rip-offs may be perfectly justifiable on dimensions the data does not capture.

The full set of limitations and a longer discussion live in the report.

---

## Authors and context

By **Annie Tran** and **Mohamed Doumbia**, completed for **MA705, Bentley University, Spring 2026**. The full report (`PL_bargains_and_ripoffs_report.pdf`) is written as a standalone deliverable for a non-technical reader; this README is a technical entry point for someone browsing the repository directly.
