# EC audit replay

- Source: `ec_audit_4w_merged.jsonl`
- Records replayed: **59** / 59 (invalid = bars missing or horizon empty)
- Horizon: 300 min (1m bars)

## Summary by event (global)

| Event | n | peak_R p50 | peak_R p80 | mae_R p20 | hit_TP % | hit_SL % | hit_0.5R % | hit_1R % |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| rejected_no_commit | 45 | +0.390 | +0.761 | -1.076 | 28.9 | 53.3 | 42.2 | 6.7 |
| passed | 14 | +0.611 | +1.429 | -0.923 | 42.9 | 21.4 | 64.3 | 35.7 |

## Per playbook

### Aplus_03_v2

| Event | n | peak_R p50 | peak_R p80 | mae_R p20 | hit_TP % | hit_SL % | hit_0.5R % | hit_1R % |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| passed | 14 | +0.611 | +1.429 | -0.923 | 42.9 | 21.4 | 64.3 | 35.7 |
| rejected_no_commit | 45 | +0.390 | +0.761 | -1.076 | 28.9 | 53.3 | 42.2 | 6.7 |

## Lecture

- Comparer `rejected_no_commit` vs `passed` sur peak_R / hit_TP.
- Si rejected peak_R ≥ passed → gate **destructeur** (éjecte des setups meilleurs ou égaux).
- Si rejected peak_R < passed → gate **protecteur** (éjecte des setups pires).
- Note intrabar : ce replay applique SL-avant-TP sur même bar (conservateur). Même convention que l'engine.
- Horizon limité à 300min par défaut — raisonnable pour 5m playbooks mais peut tronquer winners très lents.
