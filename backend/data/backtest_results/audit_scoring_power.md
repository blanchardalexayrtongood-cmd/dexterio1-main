# Audit 3 — scoring predictive power

- Total trades: 5780
- Min n per playbook: 30

## Global (all playbooks merged)

- n = **5780**
- Pearson r = **-0.0778** (p=0.0, r²=0.006)
- Spearman r = **-0.091** (p=0.0)

### E[R] by grade bucket

| Grade | n | avg_R | WR% | avg_score |
|---|---:|---:|---:|---:|
| A | 3458 | -0.0919 | 19.4 | 0.438 |
| B | 261 | -0.0791 | 17.6 | 0.385 |
| C | 453 | -0.0334 | 29.8 | 0.202 |

- Monotone decreasing with grade? **False**
  - Sequence (A+→C): [-0.0919, -0.0791, -0.0334]

### Decile analysis (score quantiles)

| Decile | n | avg_R |
|---|---:|---:|
| 0 | 587 | -0.0419 |
| 1 | 1124 | -0.0531 |
| 2 | 51 | -0.1447 |
| 3 | 620 | -0.0991 |
| 4 | 508 | -0.1503 |
| 5 | 584 | -0.0889 |
| 6 | 574 | -0.1059 |
| 7 | 584 | -0.2679 |
| 8 | 579 | -0.1281 |
| 9 | 569 | -0.0674 |

## Per-playbook correlations

| Playbook | n | Pearson r | p | r² | Spearman r | monotone? | er_sequence |
|---|---:|---:|---:|---:|---:|---|---|
| Aplus_03_IFVG_Flip_5m | 35 | -0.1508 | 0.3872 | 0.0227 | 0.0577 | False | [-0.1386, -0.0296, -0.0348] |
| Aplus_04_HTF_15m_BOS_v1 | 55 | -0.2984 | 0.0269 | 0.089 | -0.0451 | False | [-0.4918, -0.0407] |
| BOS_Scalp_1m | 135 | 0.1466 | 0.0897 | 0.0215 | 0.2214 | None | None |
| DAY_Aplus_1_Liquidity_Sweep_OB_Retest | 2157 | -0.2082 | 0.0 | 0.0434 | -0.1416 | False | [-0.1479, -0.0467] |
| Engulfing_Bar_V056 | 86 | -0.0694 | 0.5255 | 0.0048 | -0.1119 | None | None |
| FVG_Fill_Scalp | 40 | 0.0101 | 0.9507 | 0.0001 | -0.0802 | None | None |
| IFVG_5m_Sweep | 33 | -0.4901 | 0.0038 | 0.2402 | -0.4618 | False | [-0.4998, 0.0034] |
| Liquidity_Sweep_Scalp | 132 | 0.0255 | 0.7714 | 0.0007 | -0.0217 | False | [-0.0279, -0.1264, -0.0336] |
| Morning_Trap_Reversal | 83 | 0.0927 | 0.4044 | 0.0086 | -0.0025 | False | [0.1055, -0.2043, -0.1573] |
| SCALP_Aplus_1_Mini_FVG_Retest_NY_Open | 2868 | -0.0999 | 0.0 | 0.01 | -0.0504 | False | [-0.0344, -0.0322, -0.0187] |

## Lecture

- **r² < 0.05** → le score n'explique <5% de la variance du r_multiple. Scoring est essentiellement du bruit.
- **|r| < 0.1** → corrélation inexistante. Les grades A+/A/B ne discriminent pas.
- **Monotonicity FALSE** → grade plus haut ≠ meilleur trade. Les thresholds sont mal calibrés OU les poids sont mauvais.
- **Decile analysis** : si E[R] ne monte pas avec le décile de score, le score n'ordonne rien.

## Implication

Si |r| < 0.1 ET monotonicity FALSE → **le système de grading est décoratif, pas prédictif**.
Actions possibles (plan séparé) :
- Refondre les poids du scoring avec régression sur les trades historiques (fitted on training half, tested on holdout).
- Ou abandonner le grading, utiliser un seuil binaire simple (détecté ou non).
- Ou remplacer le score par un classifier meta-labeling (López de Prado) — interdit par règle anti-patterns tant que E[R]>0 pas atteint rule-based.
