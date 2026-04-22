# R.3 — Aplus_03_IFVG_Flip_5m TP calibration — verdict 2026-04-21

## Protocole

- **Overlay** : [r3_aplus03_tpcalib_v1.yml](backend/knowledge/campaigns/r3_aplus03_tpcalib_v1.yml) = `mass_s1_v1.yml` + modifs ciblées sur `Aplus_03_IFVG_Flip_5m` :
  - `tp1_rr: 2.0 → 0.70` (cible p80 peak_R M = 1.19R, marge)
  - `tp2_rr: 4.0 → 1.50`
  - `breakeven_at_rr: 1.5 → 0.40`
  - `trailing_trigger_rr: 1.0 → 0.50`
  - `trailing_offset_rr: 0.5 → 0.25`
  - `min_rr: 2.0 → 0.70`
- S1 entry_confirm + entry_buffer conservés. HTF alignment non appliqué (Family A IFVG flip = intrinsèquement contre-tendance locale).
- **Conditions** : 4 semaines (jun_w3 + aug_w3 + oct_w2 + nov_w4), SPY+QQQ, caps actives, allowlist restreinte à Aplus_03.

## Résultats

| Week | n | E[R] | WR | PF | final_cap |
|---|---:|---:|---:|---:|---:|
| jun_w3 | 6 | -0.143 | 16.7% | 0.10 | $48,930 |
| aug_w3 | 12 | -0.008 | 41.7% | 0.80 | $49,707 |
| oct_w2 | 5 | -0.198 | 20.0% | 0.02 | $49,008 |
| nov_w4 | 12 | +0.002 | 66.7% | 1.06 | $50,151 |
| **Total** | **35** | **-0.055** | **42.9%** | — | — |

Net après slippage (-0.065R) : **E[R] net = -0.120**.

## Comparaison R.3 vs M baseline

| Métrique | M (TP fixed 2R) | R.3 (TP 0.70R) | Delta |
|---|---:|---:|---:|
| n | 13 | 35 | +22 |
| E[R] | -0.047 | -0.055 | **-0.008** |
| WR | 38.5% | 42.9% | +4.4 pts |
| TP1 hits | 0 | 10 | +10 |
| Σ R | -0.62 | -1.93 | -1.31 |
| peak_R p60 | 0.55 | 0.64 | ≈ |
| peak_R p80 | 1.19 | 0.73 | **-0.46** |

## Lecture mécaniste

1. **La calibration fait ce qu'elle doit faire** : BE 1.5→0.40 laisse passer plus de trades (n 13→35), TP1=0.70R devient atteignable (0→10 hits), WR remonte (38.5→42.9%).

2. **Mais E[R] ne s'améliore pas** : 15 wins × ~0.65R - 20 losses × ~1.0R ≈ -10R attendu vs -1.93R observé. Les winners capturent moins que les losers perdent, **parce que le SL à `fvg_extreme` produit des pertes ~1R alors que peak_R p80 du sample R.3 n'est que 0.73R**.

3. **Asymétrie structurelle du signal** : l'IFVG flip seul ne produit **pas** de mouvement directionnel exploitable à un R/R > 1:1. Le mean reversion vers la zone d'invalidation est modeste (p80 peak_R 0.73R en R.3 sample, 1.19R en M sample). Un TP à 0.65R = winners trop petits pour payer 1R de perte.

## Verdict structurel (reclass 5-classes)

**Aplus_03 → de IMPROVE vers REWRITE partial.**

Raisons :
- 2 leviers testés (S1 gate M + TP recalibration R.3) → aucun ne produit E[R] > 0.
- Le signal IFVG flip **seul** ne porte pas d'edge. MASTER Family A **complète** exige un stack 5 couches : sweep + IFVG + breaker/OB/FVG entry 1m + D/4H bias + liquidity draw TP. On teste 1/5.
- Prochaine tentative productive = **Aplus_01 Sweep+IFVG+Breaker** (Family A full) ou **supprimer le signal isolé** (KILL Aplus_03 tel qu'implémenté).

## Schema YAML — blocker confirmé

Aplus_01 (Family A full) nécessite :
- `require_sweep: true` **avec** direction alignée (sweep opposé au IFVG direction)
- `tp_logic: liquidity_draw` (pas fixed RR) — **absent du schema YAML actuel**
- `entry_tf: 1m` dans la zone post-IFVG — **pipeline 1m confirm-in-zone absent**
- Breaker detector sur la bougie qui casse l'IFVG — détecteur à vérifier (fichier `breaker_block.py` existe, usage réel ?)

**Avant de créer Aplus_01, il faut étendre le schema YAML** (C-new.1 du plan). Sans ça, Aplus_01 aura les mêmes pathologies que Aplus_03 : TP inatteignable + entry imprécise.

## Prochaine étape proposée

**C-new.1 — Schema YAML extension** avant toute recreation Family A/B/F :
- Ajouter `tp_logic: liquidity_draw` + pipeline lookup des pools session high/low (pipeline : reader → daily_liquidity_pools → setup pairing).
- Vérifier `setup_tf: 15m` support end-to-end (Family B).
- Ajouter `entry_tf: 1m` state machine (setup 5m pending queue → wait → 1m bar touches zone → execute).
- Premarket session context (Family F).

Coût estimé : ≥ 1-2 jours engine work. Gate : un test unitaire `tp_logic: liquidity_draw` produit un TP = session high le plus proche above entry, pas un fixed RR.

**Alternative plus rapide** : accepter terminal Aplus_03 KILL, créer **Aplus_04 HTF+15m BOS** (Family B) — détecteurs existent déjà (BOS + HTF bias), schema plus simple. Tester si 15m setup pipeline donne edge avant de lourdir le schema.

## Rappel 5-classes post-R.3

- **SAVE** : 0
- **IMPROVE** : 2 (Engulfing_Bar_V056, Morning_Trap_Reversal — 1 levier chacun encore)
- **REWRITE partial** : 3 (FVG_Fill_V065, Range_FVG_V054, **Aplus_03_IFVG_Flip_5m** ← ajouté)
- **RECREATE from scratch** : 2 (Asia_Sweep_V051, London_Fakeout_V066) + 3 jamais instanciés (Aplus_01/02/04)
- **KILL** : 2 confirmés (Liquidity_Sweep_Scalp, Liquidity_Raid_V056) + 1 DEFER (OB_Retest_V004 n=2)
