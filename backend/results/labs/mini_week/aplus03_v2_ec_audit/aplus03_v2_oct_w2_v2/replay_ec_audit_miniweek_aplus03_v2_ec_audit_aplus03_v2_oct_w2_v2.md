# EC audit replay

- Source: `ec_audit_miniweek_aplus03_v2_ec_audit_aplus03_v2_oct_w2_v2.jsonl`
- Records replayed: **8** / 8 (invalid = bars missing or horizon empty)
- Horizon: 300 min (1m bars)

## Summary by event (global)

| Event | n | peak_R p50 | peak_R p80 | mae_R p20 | hit_TP % | hit_SL % | hit_0.5R % | hit_1R % |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| rejected_no_commit | 6 | +0.414 | +0.842 | -0.617 | 16.7 | 16.7 | 33.3 | 0.0 |
| passed | 2 | +0.374 | +0.514 | -0.831 | 50.0 | 50.0 | 50.0 | 0.0 |

## Per playbook

### Aplus_03_v2

| Event | n | peak_R p50 | peak_R p80 | mae_R p20 | hit_TP % | hit_SL % | hit_0.5R % | hit_1R % |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| passed | 2 | +0.374 | +0.514 | -0.831 | 50.0 | 50.0 | 50.0 | 0.0 |
| rejected_no_commit | 6 | +0.414 | +0.842 | -0.617 | 16.7 | 16.7 | 33.3 | 0.0 |

## Lecture

- Comparer `rejected_no_commit` vs `passed` sur peak_R / hit_TP.
- Si rejected peak_R ≥ passed → gate **destructeur** (éjecte des setups meilleurs ou égaux).
- Si rejected peak_R < passed → gate **protecteur** (éjecte des setups pires).
- Note intrabar : ce replay applique SL-avant-TP sur même bar (conservateur). Même convention que l'engine.
- Horizon limité à 300min par défaut — raisonnable pour 5m playbooks mais peut tronquer winners très lents.
