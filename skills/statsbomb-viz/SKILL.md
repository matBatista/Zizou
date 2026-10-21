---
name: statsbomb-viz
description: Use para gerar shot map, pass map, radar de jogadores ou métricas (xG, xA, PPDA, field tilt, xGChain, xT) a partir do StatsBomb Open Data com a lib football_analyst (projeto "Football Analysts"), que é cache-first em sb_cache/.
---

# Visualizações e métricas StatsBomb (football_analyst)

Projeto: `~/Claude/Projects/Football Analysts` (repo `matBatista/Football-Analysts`; cópia em `bulkfoot/services/analysts`). Python **3.10+** (o `python3` do macOS é 3.9 — ver `SETUP.md`).

## Setup

```bash
cd ~/Claude/Projects/"Football Analysts"
python3.11 -m venv .venv && source .venv/bin/activate     # qualquer 3.10+
pip install -e ".[dev]"
```

## Gerar as viz (rodar da raiz do projeto)

```bash
python examples/run_shot_map.py   # outputs/shot_map.png
python examples/run_pass_map.py
python examples/run_radar.py
python examples/run_xt.py         # ranking de xT por jogador (texto)
```

Ou em código (API pública de `football_analyst/__init__.py`):

```python
import football_analyst as fa
import football_metrics as fm

db = fa.StatsBomb()                       # cache-first: sb_cache/
MATCH = 8658                              # final da Copa 2018, França 4-2 Croácia
fa.shot_map(MATCH, db=db, save_path="outputs/shot_map.png")         # bolha = xG
fa.pass_map(MATCH, player="Antoine Griezmann", db=db, save_path="outputs/pass_griezmann.png")
fa.pass_map(MATCH, team="France", db=db, save_path="outputs/pass_team.png")
fa.player_radar("Antoine Griezmann", "Luka Modrić", [MATCH], db=db, save_path="outputs/radar.png")

ev = db.events(MATCH)
fm.expected_goals(ev, team="France"); fm.ppda(ev, "France", "Croatia")
fm.field_tilt(ev, "France"); fm.xg_chain(ev, "Antoine Griezmann")
fm.xt_by_player(ev).head(10); fm.xt_added(ev, team="France")
```

Achar ids: `db.competitions()` → `db.matches(competition_id, season_id)` (ex. `db.matches(43, 3)` = Copa 2018).

## Cuidados

- Todas as funções de viz aceitam `db=` — passe a mesma instância para reaproveitar o cache; `save_path` grava PNG, `show=True` abre janela. Crie `outputs/` se não existir.
- Nomes de jogador são exatamente os do StatsBomb, com acento (`"Luka Modrić"`); `db.lineups(match_id)` lista os nomes válidos.
- StatsBomb Open Data é histórico. Para jogos recentes (Copa 2026) use `fa.WorldCup` (openfootball, sem chave) ou `fa.APIFootball` (precisa `API_FOOTBALL_KEY`, sem coordenadas → sem shot/pass map). Ver [apifootball-ingest](../apifootball-ingest/SKILL.md).
- Referência de métricas: `metrics_cheat_sheet.md`. Antes de commitar: `ruff check . && pytest -q` (é o que o CI roda).
