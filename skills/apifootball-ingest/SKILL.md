---
name: apifootball-ingest
description: Use ao escrever ou depurar código que chama a API-Football v3 (api-sports.io) — cliente cache-first que respeita a cota (~100 req/dia no free), lê API_FOOTBALL_KEY do .env e resolve team_id de forma robusta (normalização + lookup por liga/temporada + fuzzy), já que /teams?search falha com hífen/acento.
---

# Ingestão API-Football (cache-first + team_id robusto)

Implementações reais para copiar o padrão (não reescreva do zero):

| Onde | O que tem |
|---|---|
| `bulkfoot/services/predict/analytics/app/services/insights_service.py` (idêntico em `foot_predict/analytics/`) | `_normalize_name`, `_fetch_league_teams`, `_resolve_team_id`, caches em memória |
| `bulkfoot/services/predict/analytics/src/football_data_api.py` | cache SQLite `data/match_cache.db` com TTL |
| `bulkfoot/services/predict/analytics/src/league_id.py` | `LEAGUE_ID_MAP` (código → id, ex. `BSA: 71`, `PL: 39`), `resolve_league_id`, `/leagues?current=true` com `lru_cache` |
| `Football Analysts/football_analyst/apifootball.py` | cliente cache-first em JSON (`sb_cache/apifootball/`), erros no body |
| `Football Analysts/webapi/apifootball.py` | cache SQLite com TTL por tipo + tratamento de 429 |

## Regras do cliente

1. **Chave**: `os.getenv("API_FOOTBALL_KEY")`, header `x-apisports-key`, base `https://v3.football.api-sports.io`. A chave fica em `services/predict/analytics/.env` (gitignored; modelo em `.env.example`). Nunca logue nem commite.
2. **Cache antes da rede**: chave de cache = endpoint + params ordenados. TTLs usados: fixtures finalizadas 24h, stats de fixture 30 dias (resultado não muda após FT). Se não há chave mas há cache, sirva o cache.
3. **Erros**: a API costuma responder HTTP 200 com `{"errors": {...}}` para cota/auth — sempre cheque `payload["errors"]` (vazio pode vir como `{}` ou `[]`). Trate `429` como "cota diária esgotada, tente amanhã / use cache".
4. **Economize cota**: lista curada de ligas em vez de chamar `/leagues` toda vez; `--stats` (1 chamada por fixture) só quando houver cota sobrando.
5. **Season** = ano de início (2023-24 → `2023`). Copa do Mundo = league `1`.

## Resolver team_id (o ponto que quebra)

`/teams?search=` falha com nomes como "Atlético-MG", "São Paulo", "Red Bull Bragantino". Ordem que funciona (`_resolve_team_id`):

1. Normalizar: NFKD sem acentos → lower → `.`, `-`, `/` viram espaço → só `[a-z0-9 ]` → remove tokens `fc cf sc ec afc ac as ss rc cd ud club de` e números.
2. **Lookup por liga+temporada**: `GET /teams?league=<id>&season=<ano>` → dict `{nome_normalizado: id}` (1 chamada, cacheada por `(league, season)`). Match exato no nome normalizado.
3. **Fuzzy** no mesmo dict: `difflib.get_close_matches(norm, keys, n=1, cutoff=0.75)` — logue o match para auditar.
4. Só então `GET /teams?search=` com o nome original e depois o normalizado.
5. Retorne `None` (não exceção) se nada bater, e cacheie os acertos.

Ao chamar, sempre passe `league_id` (via `resolve_league_id("BSA")`) e `season` — sem eles cai direto no search frágil.

## Verificação rápida

```bash
cd ~/Claude/Projects/bulkfoot/services/predict/analytics
source venv/bin/activate   # ou .venv
python -c "from app.services.insights_service import _resolve_team_id as r; print(r('Atlético-MG', 71, 2024), r('São Paulo', 71, 2024))"
```
Gasta até 1–2 requisições na primeira vez; a segunda é servida do cache em memória do processo.
