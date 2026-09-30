# Optimising Match-Importance Weights for International Football Team Rankings

International football ranking systems do not treat all matches equally. In the FIFA weighting scheme used in the framework studied here, matches receive different importance factors depending on the competition type: **1.0 for Friendly matches, 2.5 for Qualifiers, 3.0 for Confederation Championships, and 4.0 for FIFA World Cup matches**. These weights are intended to reflect the idea that results from more competitive tournaments should contribute more strongly to the estimation of a team's current strength.

This project investigates whether the importance assigned to different types of international football matches can be **learned from data rather than kept fixed**.

It extends the maximum-likelihood ranking framework of **Ley, Van de Wiele, and Van Eetvelde**, *Ranking soccer teams on the basis of their current strength: A comparison of maximum likelihood approaches*. Their framework combines statistical team-strength models with **time decay** and **match-importance weighting**, so that recent matches and more important competitions have greater influence on the estimated strength of a team.

For international matches, however, Ley et al. used the predefined FIFA importance factors directly and kept them **constant throughout the analysis**. The framework therefore evaluates different statistical ranking models, but does not investigate whether the numerical importance assigned to Friendly, Qualifier, Confederation Championship, and World Cup matches is itself the most suitable choice for predictive performance.

This project focuses on that open question. Instead of treating the four importance factors as fixed inputs, I evaluate alternative weight combinations and measure how they affect the ability of the ranking models to predict future international match outcomes.

The analysis uses three models from the original framework:

- **Independent Poisson**
- **Bivariate Poisson**
- **Thurstone–Mosteller**

The models are evaluated on historical international football results using a **rolling temporal validation setup**, and predictive performance is measured with the **Ranked Probability Score (RPS)**.

The aim is not to redefine the sporting importance of different competitions, but to investigate a modelling question:

> **Can data-driven match-importance weights improve the predictive performance of international football ranking models compared with the fixed weights used in the original framework?**

---

## 1. Background

International football rankings are intended to represent the current strength of national teams. A useful ranking system should therefore give more influence to matches that contain stronger information about a team's present ability.

Ley et al. proposed a family of maximum-likelihood ranking models in which the contribution of a historical match depends on two components:

1. **Time decay** — recent matches receive more weight than older matches.
2. **Match importance** — different competition types receive different importance factors.

In their international-football analysis, the importance factors followed the FIFA-style hierarchy used in the original framework:

| Match category | Baseline importance weight |
|---|---:|
| Friendly | 1.00 |
| Qualifier | 2.50 |
| Confederation championship | 3.00 |
| FIFA World Cup | 4.00 |

These values were used as predefined weights rather than estimated from the data.

This leaves an interesting extension: although it is reasonable that competitive matches should generally receive more weight than friendlies, the exact numerical differences between the categories do not necessarily have to be fixed in advance.

---

## 2. Aim of This Project

This project keeps the main modelling idea of Ley et al. but treats the four match-importance factors as values that can be selected empirically.

The analysis asks whether another weighting scheme can produce better **out-of-sample match predictions** while keeping the underlying ranking framework comparable.

The four categories considered are:

- **Friendly**
- **Qualifier**
- **Confederation championship**
- **FIFA World Cup**

For each statistical ranking model, different combinations of these weights are evaluated and compared with the baseline weighting scheme.

---

## 3. Dataset

The analysis uses the public **International Football Results** dataset by Mart Jürisoo, available on Kaggle:

[International Football Results from 1872](https://www.kaggle.com/datasets/martj42/international-football-results-from-1872-to-2017)

The complete dataset contains historical international football results over a very long period. For this project, only the most recent **eight-year period, January 2018 to January 2026**, is used.

This gives:

- **7,800 matches** before tournament filtering
- **5,789 matches** in the final modelling dataset

The analysis uses the following match information:

- date
- home team
- away team
- home score
- away score
- tournament
- neutral-ground indicator

---

## 4. Match Classification

To remain comparable with the four-category structure used in the original ranking framework, tournament names in the dataset are mapped into four broad groups.

### Friendly

- International Friendly

### Qualifier

Examples include:

- FIFA World Cup qualification
- UEFA Euro qualification
- African Cup of Nations qualification
- AFC Asian Cup qualification
- Copa América qualification
- Gold Cup qualification / related qualifying competitions
- Oceania Nations Cup qualification

### Confederation Championship

Examples include:

- UEFA European Championship
- African Cup of Nations
- AFC Asian Cup
- Copa América
- CONCACAF Gold Cup
- OFC Nations Cup

### FIFA World Cup

- FIFA World Cup

Competitions that do not fit the four-category structure are excluded from this analysis. This includes competitions such as the **UEFA Nations League**, **CONCACAF Nations League**, and a number of smaller regional competitions.

The final dataset contains:

| Category | Matches | Share |
|---|---:|---:|
| Friendly | 2,059 | 35.6% |
| Qualifier | 2,967 | 51.3% |
| Confederation | 635 | 11.0% |
| World Cup | 128 | 2.2% |
| **Total** | **5,789** | **100%** |

The strong dominance of qualifier matches is particularly relevant because a large importance factor further increases their influence on the likelihood used to estimate team strengths.

---

## 5. Ranking Models

Three different statistical ranking models are studied. They represent both score-based and outcome-based approaches.

### Independent Poisson Model

The Independent Poisson model uses the **number of goals scored by each team**.

For every match, expected home and away goals depend on the estimated strengths of the two teams together with a home-advantage effect. Home and away goals are then modelled using Poisson distributions.

Because the model uses the actual score, a 4–0 result contains more information than a 1–0 result.

---

### Bivariate Poisson Model

The Bivariate Poisson model extends the Independent Poisson approach by allowing the two teams' goal counts to share a common component.

This introduces dependence between home and away scores and provides a more flexible representation of football scores while retaining the underlying team-strength structure.

---

### Thurstone–Mosteller Model

The Thurstone–Mosteller model takes a different approach.

Instead of modelling the exact number of goals, it models only the final outcome:

- home win
- draw
- away win

Each team has an underlying strength, and the probability of each result depends on the difference between the strengths of the two teams, together with home advantage and a draw threshold.

This provides an outcome-based comparison with the two score-based Poisson models.

---

## 6. Weighted Maximum-Likelihood Ranking

All three models estimate latent team strengths using **weighted maximum likelihood**.

Each historical match receives a combined weight based on:

\[
w_i = w_i^{time} \times w_i^{importance}
\]

### Time component

Older matches gradually become less influential through an exponential time-decay function.

The idea is that a match played recently should provide more information about a team's current strength than a match played several years ago.

### Importance component

The second component represents the type of match.

This is the component investigated in this project. Rather than automatically accepting the baseline values for Friendly, Qualifier, Confederation, and World Cup matches, alternative combinations are evaluated from the data.

---

## 7. Rolling-Window Evaluation

Model performance is evaluated using a **rolling temporal validation scheme** rather than a random train/test split.

Each evaluation window contains:

- **11 months of training data**
- **1 following month of test data**
- a **1-month forward shift** before the next window

The first window trains on January–November 2018 and predicts December 2018. The process then moves forward month by month until the end of 2025.

This produces:

- **73 rolling evaluation windows**
- **4,768 out-of-sample match predictions**

A test match is evaluated only when both teams have appeared in the corresponding training period.

This design is important because football ranking is naturally a temporal problem: future matches should be predicted using only information available before those matches occurred.

---

## 8. Searching for Better Match-Importance Weights

The four importance factors are treated as tunable values.

A structured search is performed over different combinations around the baseline weighting scheme. For every candidate combination:

1. the ranking model is fitted on each training window;
2. team strengths are estimated using the weighted historical matches;
3. probabilities for the following month's matches are generated;
4. predictive performance is calculated;
5. performance is averaged across all rolling windows.

A total of **1,050 weight combinations are evaluated for each model**.

The purpose is not to change the statistical model itself, but to determine whether the information contributed by different competition types is better represented by another relative weighting scheme.

---

## 9. Evaluation Metric: Ranked Probability Score

Prediction quality is measured using the **Ranked Probability Score (RPS)**.

RPS evaluates the predicted probabilities for:

- home win,
- draw,
- away win.

It is particularly suitable for football because these outcomes are ordered. Predicting a draw when the home team wins is less severe than strongly predicting an away win.

**Lower RPS indicates better probabilistic predictions.**

---

## 10. Results

The best-performing weight combinations **within the tested search grid** were:

| Model | Friendly | Qualifier | Confederation | World Cup | Average RPS |
|---|---:|---:|---:|---:|---:|
| Bivariate Poisson | 0.75 | 1.50 | 1.50 | 4.00 | **0.200408** |
| Independent Poisson | 1.50 | 1.75 | 1.50 | 3.50 | **0.195324** |
| Thurstone–Mosteller | 0.75 | 1.50 | 1.50 | 3.50 | **0.250993** |

For comparison, the baseline weighting scheme produced average RPS values of:

| Model | Baseline RPS |
|---|---:|
| Bivariate Poisson | 0.201733 |
| Independent Poisson | 0.196168 |
| Thurstone–Mosteller | 0.252194 |

Across the three models, the search produced a consistent overall pattern:

- **Qualifier weights were lower** than the baseline value of 2.50.
- **Confederation weights were consistently lower**, with 1.50 selected by all three models.
- **World Cup weights remained relatively close to the original high weighting.**
- **Friendly weights were less stable and depended more strongly on the model.**

The World Cup category contains only 128 matches in the working dataset, so the available data provide less information for distinguishing between nearby World Cup weight values.

Overall, the results suggest that the relative gap between qualification/confederation matches and other match types can be smaller when the objective is **predictive team-strength estimation** rather than defining the sporting importance of a competition.

---

## 11. Main Takeaway

The main contribution of this project is not a new football ranking model. Instead, it examines an assumption inside an existing ranking framework.

Ley et al. demonstrated that weighted maximum-likelihood models can provide effective estimates of international team strength. Their framework already accounted for both recency and competition importance, but the numerical match-importance factors were kept fixed.

This analysis treats those factors as an empirical modelling choice.

Across three different model families, the experiments show that the best-performing combinations within the tested grid consistently place **less relative emphasis on qualifier and confederation matches** than the baseline scheme.

This illustrates a broader modelling principle:

> Domain-based weights can provide a sensible starting point, but their predictive value can also be evaluated and calibrated using data.

---

## 12. Repository Structure

```text
Aundrila_Acharjee_Team_Ranking/
│
├── dataset/
│   └── results.csv
│
├── notebooks/
│   ├── bivariate_poisson.ipynb
│   ├── Indipendant_Poisson.ipynb
│   └── thurstone.ipynb
│
├── results/
│   ├── grid_search_bivariate.csv
│   ├── grid_search_independent_poisson_results.csv
│   └── grid_search_thurstone_mosteller_results.csv
│
└── README.md
```

### `dataset/`

Contains the international football match data used for the analysis.

### `notebooks/`

Contains the implementations of the three ranking approaches:

- Independent Poisson
- Bivariate Poisson
- Thurstone–Mosteller

Each notebook performs the complete model-specific analysis and evaluates alternative match-importance weights.

### `results/`

Contains the grid-search results for each model. Each CSV records the tested importance-weight combinations and their corresponding average RPS.

---

## 13. Future Extensions

Several extensions would be useful for further analysis:

- treat the UEFA Nations League and similar competitions as an additional match category;
- investigate finer or continuous optimisation of the importance weights;
- evaluate uncertainty in RPS differences using resampling or bootstrap methods;
- study whether the preferred weighting structure changes across different historical periods;
- examine whether tournament importance interacts with team strength, region, or match location.

---

## Reference

Ley, C., Van de Wiele, T., & Van Eetvelde, H.  
**Ranking soccer teams on the basis of their current strength: A comparison of maximum likelihood approaches.**  
*Statistical Modelling*, 19(1), 55–73.

Original paper:  
https://www.researchgate.net/publication/330589080_Ranking_soccer_teams_on_the_basis_of_their_current_strength_A_comparison_of_maximum_likelihood_approaches

Dataset:  
https://www.kaggle.com/datasets/martj42/international-football-results-from-1872-to-2017
