---
name: footballdata-odds
description: Use quando precisar de odds históricas de fechamento (1X2 e Over/Under 2.5) para ligas europeias — baixar os CSVs da football-data.co.uk, parsear, casar com fixtures da API-Football por nome fuzzy + data e reportar o match rate. Não cobre Brasileirão/MLS/Argentina/copas.
---

# Odds de fechamento da football-data.co.uk

Implementação real: `~/Claude/Projects/bulkfoot/services/predict/analytics/src/footballdata_odds.py`
(**só existe no bulkfoot**; o `foot_predict/analytics` standalone tem apenas o `.pyc` — copie o `.py` de lá se precisar).

Por que existe: a API-Football só guarda odds dos **últimos 7 dias**, então não serve para backtest.

## Cobertura (decorar isso)

`LEAGUE_TO_DIV`: `PL→E0, ELC→E1, SA→I1, LL→SP1, BL→D1, FL1→F1, DED→N1, PPL→P1`.
**Não cobre BSA (Brasileirão), ARG, MLS nem copas.** Jogos dessas ligas ficam no treino, mas saem do backtest de ROI — avise o usuário em vez de tentar "achar" odds.

## Uso

```python
from src.footballdata_odds import load_fd_odds, merge_odds, csv_url

csv_url("PL", 2023)          # https://www.football-data.co.uk/mmz4281/2324/E0.csv  (season 2023 = 2023-24)
odds = load_fd_odds("PL", 2023)   # date, home_team, away_team, odds_h/d/a, odds_over25/under25
merged, report = merge_odds(matches, odds, date_tolerance_days=1)
print(report)  # {"overall_rate": 0.998, "matched": ..., "total": ..., "by_league": {...}}
```

`matches` precisa de `date, league, season, home_team, away_team` (schema do `fetch_multi_season.py`).

## Detalhes que importam

- CSV é `latin-1`, ler com `on_bad_lines="skip"`. Datas `dd/mm/yy` ou `dd/mm/yyyy`.
- Prioridade de colunas 1X2: `AvgC*` (média de fechamento) → `PSC*` (Pinnacle fechamento) → `B365C*` → abertura (`Avg*`, `PS*`, `B365*`). OU 2.5: `AvgC>2.5` … `B365<2.5`. Linha sem 1X2 é descartada.
- Matching por liga: normaliza nome → aplica `TEAM_ALIASES` (ex. `manchester united→man united`, `atletico madrid→ath madrid`) → fuzzy `cutoff=0.86` → mesmo par (mandante, visitante) com data mais próxima dentro de ±1 dia.
- **Sempre reporte o match rate por liga.** Baseline real (2021–2024, PL/SA/LL/BL/FL1): 99.8% (`data/backtest_roi_2021-2024.txt`). Se uma liga cair muito, adicione alias em `TEAM_ALIASES` (chave e valor já no formato normalizado) em vez de baixar o cutoff.
- Download acontece em runtime (`pd.read_csv(url)`), sem cache próprio: em loops, cacheie você mesmo.
