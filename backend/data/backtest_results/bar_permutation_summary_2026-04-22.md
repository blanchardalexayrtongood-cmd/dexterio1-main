# Bar permutation test v1 — ESCALADE USER synthèse stat sig (§0.7 item #5)

**Date** : 2026-04-22 (tech debt §0.7 livré pendant attente décision user Leg 5)
**Harness** : [backend/scripts/bar_permutation_test.py](../../scripts/bar_permutation_test.py)
**Test** : sign-permutation on R-multiples, 2000 iterations, seed=42

## Résultats

### survivor_v1 cohort (4 survivors × 4w, n=90)

| Métrique | Valeur |
|---|---:|
| E[R] observé | **−0.0215** |
| Permuted E[R] mean | −0.0002 |
| Permuted E[R] std | 0.0221 |
| Permuted p5 / p95 | −0.0364 / +0.0362 |
| **p two-sided** | **0.334** |
| Gate O5.3 PASS | FALSE |

Le E[R] observé reste **dans la bande centrale** des permutations — n=90 insuffisant pour distinguer −0.0215 de zéro. Cohérent avec "no edge ni positive ni négative significative", juste trades proches de BE.

Per-playbook splits : aucun p<0.05.

| Playbook | n | E[R] | p |
|---|---:|---:|---:|
| News_Fade | 10 | +0.0012 | 0.996 |
| Engulfing_Bar_V056 | 26 | −0.0101 | 0.878 |
| Session_Open_Scalp | 12 | −0.0139 | 0.728 |
| Liquidity_Sweep_Scalp | 42 | −0.0361 | 0.155 |

### HTF_Bias_15m_BOS 12w solo (n=58)

| Métrique | Valeur |
|---|---:|
| E[R] observé | −0.0357 |
| p two-sided | 0.159 |
| Gate O5.3 PASS | FALSE |

**Per-direction split (révélateur) :**

| Direction | n | E[R] | p two-sided |
|---|---:|---:|---:|
| **LONG** | 42 | **−0.0704** | **0.022** |
| SHORT | 16 | +0.0554 | 0.174 |

- **LONG HTF 12w p=0.022 → statistiquement significatif** (α=0.05). La performance LONG négative n'est pas artefact d'échantillonnage — c'est un effet systématique sur le corpus jun-nov 2025.
- SHORT n=16 non-significatif (n trop petit), E[R]=+0.055 anecdotique.

## Lecture

### Bear case cross-playbook 2025 uptrend LONG-toxicité : **confirmé statistiquement**

5 data points cross-playbook (Engulfing 1.1, Morning_Trap 1.2, IFVG 2.1, VWAP 2.2, HTF 2.3) notent asymétrie LONG toxique / SHORT meilleur. **HTF 12w est le premier où p<0.05 formellement démontré** (LONG p=0.022).

### E[R] proche BE mais p non-significatif (survivor_v1)

Les quasi-BE de survivor_v1 (E[R] −0.01 à −0.04) ne sont **pas statistiquement distinguables de zéro** à n=90. Ce qui signifie :
- L'absence d'edge n'est pas prouvée à ce n — on ne peut pas exclure un edge faible ±0.03R.
- Pour distinguer de zéro à p<0.05 sur E[R]=0.03R typique, il faudrait ~n ≥ 200-400 (estim. rapide depuis std permuted 0.022).
- Cohérent avec user bar strict (E[R]>0.10R + n>30) : même si edge réel existe à +0.02R, il est sous le bar utile.

### Implications Leg 5

- Option **E (paper baseline)** : cohort survivor_v1 à E[R]≈−0.02 p=0.33 = "indécis mais sous bar" — paper test valide l'infra sans réelle promesse d'alpha, ce qui est une décision défendable.
- Option **C (refonte portfolio-first)** : possible avec subset SHORT-only SPY/QQQ uptrend, mais n<30 sur tout playbook individuel rend fragile.
- Option **D (playbook académique)** : pas informé par ces p-values — un nouveau signal n'est pas couplé au bear case existant.

Ce test ajoute une brique au bear case, ne change aucune décision Leg 1-4.2.
