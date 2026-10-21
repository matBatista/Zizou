---
name: dixon-coles-backtest
description: Use para avaliar se o modelo Dixon-Coles do foot_predict bate o mercado — backtest de ROI com split temporal estrito (treina temporadas anteriores, testa a mais recente) contra odds de fechamento da football-data.co.uk, mercados 1X2 e OU 2.5.
---

# Backtest de ROI (Dixon-Coles vs closing odds)

Script real: `~/Claude/Projects/bulkfoot/services/predict/analytics/backtest_roi.py` (doc: `PIPELINE.md` na mesma pasta).
Depende de `src/dixon_coles.py`, `src/footballdata_odds.py` (ver skill [footballdata-odds](../footballdata-odds/SKILL.md)) e `src/odds_utils.py::find_value_bets`.

## Passos

```bash
cd ~/Claude/Projects/bulkfoot/services/predict/analytics
python -m venv venv && source venv/bin/activate && pip install -r requirements.txt   # 1a vez

# 1. Dataset multi-temporada (API-Football, cache em data/match_cache.db; respeita ~100 req/dia)
python fetch_multi_season.py --leagues PL LL SA BL FL1 --seasons 2022 2023 2024 --out matches_multi.csv

# 2. Backtest (baixa odds sozinho)
python backtest_roi.py --data matches_multi.csv --min-edge 0.03 --xi 0.0019
python backtest_roi.py --data matches_multi.csv --markets 1x2 ou25
```

Flags: `--min-edge` (edge mínimo modelo vs mercado justo), `--xi` (decaimento temporal; `0` = nenhum), `--stake` (flat), `--markets 1x2 ou25`.

## Regras de método (não quebre)

- **Split temporal estrito por liga**: treina `season < última`, testa `season == última`. Liga com < 2 temporadas é pulada. Nunca embaralhe nem use k-fold aleatório — vaza futuro.
- Só jogos finalizados (`FT/AET/PEN/FINISHED`) com gols preenchidos.
- Só ligas com odds (`LEAGUE_TO_DIV`). Incluir `BSA` no `--leagues` não dá erro, mas o Brasileirão é ignorado no backtest.
- Confira o **match rate de odds** impresso antes de interpretar o ROI.

## Interpretação

- Baseline salvo (`data/backtest_roi_2021-2024.txt`, 7.167 jogos): **ROI −8.42%**, 2953 bets, hit 34.9%. Isso é o resultado honesto esperado — closing odds são eficientes.
- ROI positivo pequeno numa liga/temporada = sorte in-sample até repetir em várias. É feedback de calibração, **não recomendação de aposta**.
- Ao mudar o modelo, rode o mesmo comando e compare com o baseline; salve o novo relatório em `data/backtest_roi_<anos>.txt`.
- Avaliação sem odds (RPS + log-loss vs baseline de frequência): `python eval_dixon_coles.py --data matches_multi.csv --xi 0.0019`.
