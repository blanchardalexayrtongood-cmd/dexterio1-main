# VOCAB MISMATCH ANALYSIS: "bullish" vs "uptrend"

## CONTEXTE

**Problème identifié:** MarketStateEngine produit `daily_structure = "bullish"` mais les playbooks dans `playbooks.yml` attendent `structure_htf: ["uptrend", "downtrend"]`.

## SOURCE DE VÉRITÉ

**Playbooks.yml (structure_htf attendue):**
```yaml
- playbook_name: "NY_Open_Reversal"
  context_requirements:
    structure_htf: ["uptrend", "downtrend"]

- playbook_name: "London_Sweep_NY_Continuation"
  context_requirements:
    structure_htf: ["uptrend", "downtrend"]
```

**MarketStateEngine (production actuelle):**
```python
# engines/market_state.py produit:
market_state.bias = 'bullish' | 'bearish' | 'neutral'
market_state.daily_structure = 'bullish' | 'bearish' | 'neutral' | 'unknown'
```

## ROOT CAUSE

**Vocabulaire incohérent entre 2 composants:**
- MarketStateEngine utilise le vocabulaire financier standard: bullish/bearish
- Playbooks.yml utilise le vocabulaire ICT/structure: uptrend/downtrend/range

**Ce n'est PAS un bug, c'est une incohérence de design.**

## SOLUTION P0 (ACTUELLE - BYPASS)

Pour débloquer P0, le patch AGGRESSIVE bypass la vérification `structure_htf` entièrement.

**Code actuel (ligne 767-774):**
```python
if not is_backtest_aggressive:
    if playbook.structure_htf and structure not in playbook.structure_htf and structure != 'unknown':
        return None, None
else:
    # BYPASS actif : on trace la raison
    if playbook.structure_htf and structure not in playbook.structure_htf and structure != 'unknown':
        bypasses_applied.append(f'structure_htf_mismatch:{structure}_not_in_{playbook.structure_htf}')
```

**Impact:** Permet aux playbooks de matcher, mais ignore complètement la vérification de structure.

## SOLUTION P1 (RECOMMANDÉE - MAPPING)

**Option A: Mapping dans _evaluate_playbook_conditions (MINIMAL)**

Ajouter un mapping simple avant la vérification:

```python
# 2. Vérifier structure HTF avec normalisation vocabulaire
structure_raw = market_state.get('daily_structure', 'unknown')

# MAPPING: Normaliser MarketStateEngine → Playbook vocabulary
STRUCTURE_MAPPING = {
    'bullish': 'uptrend',
    'bearish': 'downtrend',
    'neutral': 'range',
    'unknown': 'unknown'
}
structure = STRUCTURE_MAPPING.get(structure_raw, structure_raw)

# Vérification normale (pas de bypass nécessaire)
if playbook.structure_htf and structure not in playbook.structure_htf and structure != 'unknown':
    return None, None
```

**Avantages:**
- Patch minimal (~5 lignes)
- Préserve la logique de vérification
- Pas de bypass nécessaire
- Compatible avec SAFE et AGGRESSIVE

**Inconvénient:**
- Le mapping est "hardcodé" dans playbook_loader.py

---

**Option B: Normaliser MarketStateEngine (PROPRE mais plus invasif)**

Modifier MarketStateEngine pour qu'il produise directement le bon vocabulaire:

```python
# engines/market_state.py
def create_market_state(...):
    # Au lieu de:
    bias = 'bullish'
    daily_structure = 'bullish'
    
    # Produire:
    bias = 'bullish'  # Garder pour compatibilité
    daily_structure = 'uptrend'  # Vocabulaire ICT
```

**Avantages:**
- Solution architecturale propre
- Un seul vocabulaire dans tout le système
- Pas de mapping nécessaire

**Inconvénients:**
- Plus invasif (modifie MarketStateEngine)
- Risque de casser d'autres composants qui utilisent bias/structure
- Nécessite tests complets

---

**Option C: Dual vocabulary dans MarketState (COMPATIBLE)**

Ajouter les 2 champs dans MarketState:

```python
# models/market_data.py
class MarketState(BaseModel):
    bias: str  # 'bullish', 'bearish', 'neutral' (legacy)
    daily_structure: str  # 'bullish', 'bearish', 'neutral' (legacy)
    
    # Nouveau champs ICT vocabulary
    structure_type: str  # 'uptrend', 'downtrend', 'range' (ICT)
```

Et dans MarketStateEngine:

```python
def create_market_state(...):
    bias = 'bullish'
    daily_structure = 'bullish'  # legacy
    structure_type = 'uptrend'  # nouveau
```

Et dans playbook_loader.py:

```python
# Utiliser le nouveau champ
structure = market_state.get('structure_type', 'unknown')
```

**Avantages:**
- Backward compatible
- Vocabulaire clair et séparé
- Pas de mapping

**Inconvénients:**
- Duplication de données
- Plus de maintenance

## RECOMMANDATION

**Pour P1 (déblocage immédiat):** **Option A - Mapping dans playbook_loader.py**

**Justification:**
1. Patch minimal (5 lignes)
2. Pas de risque de régression
3. Facile à tester
4. Permet de retirer le bypass AGGRESSIVE pour structure_htf

**Code P1:**

```python
# /app/backend/engines/playbook_loader.py
# Ligne ~767, remplacer:

# AVANT (P0 - bypass)
if not is_backtest_aggressive:
    if playbook.structure_htf and structure not in playbook.structure_htf and structure != 'unknown':
        return None, None

# APRÈS (P1 - mapping)
structure_raw = market_state.get('daily_structure', 'unknown')
STRUCTURE_MAPPING = {'bullish': 'uptrend', 'bearish': 'downtrend', 'neutral': 'range', 'unknown': 'unknown'}
structure = STRUCTURE_MAPPING.get(structure_raw, structure_raw)

if playbook.structure_htf and structure not in playbook.structure_htf and structure != 'unknown':
    return None, None
```

**Impact P1:**
- Retirer le bypass AGGRESSIVE pour structure_htf
- Garder seulement le bypass pour candlestick_patterns (moteur pas câblé)
- Réduire les bypasses de 32 → 16

## STATUT

- **P0:** LOCKED avec bypass
- **P1:** Recommandation = Option A (mapping)
- **P2:** Refactorisation complète (Option B ou C) si nécessaire
