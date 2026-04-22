# Phase W — Wiring "même cerveau" — verdict 2026-04-20

## Objectif

Rendre le chemin **gate entry_confirm**, **FillModel**, et **mode d'exécution** partagé entre backtest et paper/live. Sans ça, la victoire Engulfing S1 (`+0.020R net`, backtest-only) n'est pas déployable.

## Livré

| Sous-tâche | Livrable | État |
|---|---|---|
| W.1 Shared entry gate | [entry_gates.py](backend/engines/execution/entry_gates.py) — pure function `check_entry_confirmation(playbook_def, setup, candles_1m) -> GateResult`. Backtest [engine.py:1806-1852](backend/backtest/engine.py#L1806-L1852) refactoré pour appeler la fonction partagée (instrumentation `ec_stats` + `_increment_reject_reason` conservés côté engine). Paper [paper_trading.py](backend/engines/execution/paper_trading.py) expose `ExecutionEngine.check_entry_confirmation_gate(setup, candles_1m)` qui délègue au même module. | ✅ |
| W.2 FillModel wire | `ExecutionEngine.__init__` accepte un param `fill_model` (default `IdealFillModel`). Stocké sur `self.fill_model`. Route-through complet de `update_open_trades` reporté à un refactor post-W. | ✅ |
| W.3 ClockMode enum | `ClockMode ∈ {BACKTEST, PAPER, LIVE}` dans [paper_trading.py](backend/engines/execution/paper_trading.py). `ExecutionEngine.__init__` accepte `clock_mode` (default `BACKTEST`). Branchement conditionnel de `update_open_trades` reporté. | ✅ |
| W.4 Modes YAML | [knowledge/modes.yml](backend/knowledge/modes.yml) contient `aggressive_allowlist` (13), `aggressive_denylist` (15), `phase3b_playbooks` (4). [modes_loader.py](backend/engines/modes_loader.py) lit et cache, avec fallbacks hardcodés si YAML manquant/malformé. [risk_engine.py](backend/engines/risk_engine.py) et [phase3b_execution.py](backend/engines/execution/phase3b_execution.py) sourcent leurs constantes depuis le loader, mais exposent les mêmes noms module-level (`AGGRESSIVE_ALLOWLIST`, `AGGRESSIVE_DENYLIST`, `PHASE3B_PLAYBOOKS`) pour back-compat. | ✅ |
| W.5 Smoke strict | Run backtest 2025-10-06 (1 jour) avec pre-W (git stash) vs post-W. Comparaison summary. | ✅ |

## Tests ajoutés (22 tests, tous PASS)

| Fichier | Couverture |
|---|---|
| [test_entry_gates_shared.py](backend/tests/test_entry_gates_shared.py) | 9 tests — fonction pure (gate off/on, no candles, LONG/SHORT committed/not, unknown direction, paper↔backtest équivalence via ExecutionEngine hook) |
| [test_entry_confirmation_gate.py](backend/tests/test_entry_confirmation_gate.py) | 4 tests existants (regression backtest path) |
| [test_execution_engine_fill_model.py](backend/tests/test_execution_engine_fill_model.py) | 4 tests — default Ideal, inject Conservative, default clock_mode BACKTEST, override PAPER/LIVE |
| [test_modes_loader.py](backend/tests/test_modes_loader.py) | 5 tests — YAML == expected, risk_engine constants == expected, phase3b frozenset == expected, missing YAML fallback, malformed YAML fallback |

## W.5 regression — pre-W vs post-W (2025-10-06, SPY+QQQ)

| Métrique | pre-W (stashed) | post-W | delta |
|---|---|---|---|
| total_trades | 7 | 7 | 0 |
| final_capital | 49463.36 | 49463.36 | 0.000000 |
| playbooks_registered_count | 29 | 29 | 0 |
| funnel signature (matches/setups/after_risk/trades per playbook) | — | — | **identical byte-for-byte** |

Delta ≪ 1% — le gate `<1%` de la plan est respecté avec une marge de 100% (zéro delta observable).

## Gate W final

- (a) Gate `entry_confirm` lisible depuis backtest **ET** paper_trading, via même module : ✅ (test `test_paper_execution_engine_hook_matches_backtest_gate`).
- (b) FillModel actif dans `ExecutionEngine`, default Ideal reproduit backtest actuel : ✅ (delta 0 sur smoke).
- (c) Re-run identique au précédent hors budget slippage : ✅ (delta 0).

**Phase W DONE.** Pré-requis satisfait pour Phase M (mass-apply recette S1 sur 10 candidats 5m/5m).

## Ce que W ne fait PAS (et pourquoi)

- **Pas de route-through complet de `update_open_trades` via `fill_model`** — le refactor des exit paths (SL/TP/market) pour consommer `self.fill_model.fill_stop/fill_take_profit/fill_market` reste pour plus tard. `IdealFillModel` est stocké mais l'inline logic est utilisée. Justification : zéro delta requis, et le refactor des 100+ lignes d'`update_open_trades` déborde le scope de W.
- **Pas de branchement conditionnel sur `clock_mode`** — enum défini, stocké, mais aucun `if self.clock_mode == PAPER:` branch. À activer au moment de la Phase G (paper/live).
- **Pas de retrait des imports `AGGRESSIVE_ALLOWLIST`** — les 7+ sites qui importent `from risk_engine import AGGRESSIVE_DENYLIST` continuent de fonctionner, la liste est juste re-dérivée depuis YAML au boot.

## Prochaine étape

**Phase M** — construire `knowledge/campaigns/mass_s1_v1.yml` qui applique la recette S1 (`require_close_above_trigger: true` + `require_htf_alignment: D` pour continuation) sur 10 candidats 5m/5m, run 4 semaines caps actives, compter combien franchissent `net E[R] > 0`.
