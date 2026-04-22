# EC audit replay

- Source: `ec_audit_merged_oct_aug.jsonl`
- Records replayed: **14** / 14 (invalid = bars missing or horizon empty)
- Horizon: 300 min (1m bars)

## Summary by event (global)

| Event | n | peak_R p50 | peak_R p80 | mae_R p20 | hit_TP % | hit_SL % | hit_0.5R % | hit_1R % |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| rejected_no_commit | 9 | +0.423 | +0.864 | -0.775 | 11.1 | 22.2 | 33.3 | 11.1 |
| passed | 5 | +0.896 | +1.415 | -0.488 | 20.0 | 20.0 | 80.0 | 40.0 |

## Per playbook

### Aplus_03_v2

| Event | n | peak_R p50 | peak_R p80 | mae_R p20 | hit_TP % | hit_SL % | hit_0.5R % | hit_1R % |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| passed | 5 | +0.896 | +1.415 | -0.488 | 20.0 | 20.0 | 80.0 | 40.0 |
| rejected_no_commit | 9 | +0.423 | +0.864 | -0.775 | 11.1 | 22.2 | 33.3 | 11.1 |

## Lecture

- Comparer `rejected_no_commit` vs `passed` sur peak_R / hit_TP.
- Si rejected peak_R ≥ passed → gate **destructeur** (éjecte des setups meilleurs ou égaux).
- Si rejected peak_R < passed → gate **protecteur** (éjecte des setups pires).
- Note intrabar : ce replay applique SL-avant-TP sur même bar (conservateur). Même convention que l'engine.
- Horizon limité à 300min par défaut — raisonnable pour 5m playbooks mais peut tronquer winners très lents.
