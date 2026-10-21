# Índice de projetos

Todos em `~/Claude/Projects/`. Status = estado do repo local em 2026-10-21 (último commit).

| Projeto | Stack | Repo GitHub | Como rodar | Status |
|---|---|---|---|---|
| **ManagerFC** (`foot_manager/`) | Go 1.25 + Fiber + Postgres 16 (pgx/v5, golang-migrate) · Expo 52 + Expo Router + Zustand | [matBatista/foot_manager](https://github.com/matBatista/foot_manager) | `docker compose up -d` → `cd api && go run ./cmd/server` (:8080) → `cd mobile && npx expo start` | Ativo. Multi-carreira, formações, narração, mercado/negociação. Deploy Fly configurado (`managerfc-api`, região gru). |
| **foot_predict** (`foot_predict/analytics/`) | Python · FastAPI · scikit-learn (Dixon-Coles + RandomForest) · API-Football · football-data.co.uk | [matBatista/foot-predict-analytics](https://github.com/matBatista/foot-predict-analytics) | `pip install -r requirements.txt` → `uvicorn main:app --reload` (:8000) | Ativo (insights, H2H, escanteios via API-Football). Pasta raiz `foot_predict/` não é repo; o git está em `analytics/`. `backtest_roi.py` e `src/footballdata_odds.py` só existem na cópia do bulkfoot. |
| **bulkfoot** (`bulkfoot/`) | Next.js 14 (UI + gateway) · docker compose · subtrees dos 3 serviços | [matBatista/bulkfoot](https://github.com/matBatista/bulkfoot) | `cp .env.example .env` → `docker compose up --build` (:3000) | Ativo. Fluxo Issue→PR (skill `github-flow` local). CI: go test, pytest, next build. |
| **Football Analysts** (`Football Analysts/`) | Python 3.10+ · statsbombpy · matplotlib/mplsoccer · FastAPI (`webapi/`) | [matBatista/Football-Analysts](https://github.com/matBatista/Football-Analysts) | `pip install -e ".[dev]"` → `python examples/run_shot_map.py` | Ativo. Shot/pass map, radar, 19 métricas (xG, xT…), camadas WorldCup e API-Football. CI: ruff + pytest. |
| **LifeOS** (`LIFEOS/mobile/`) | Expo 56 + Expo Router · Zustand + AsyncStorage (local-first) · TypeScript | — (sem remote configurado) | `cd mobile && npm install && npx expo start` | MVP: 6 módulos (hoje, hábitos, metas, diário, finanças, relatórios). `app/_layout.tsx` registra `add-goal`, mas `app/add-goal.tsx` não existe. Sem sync na nuvem. |
| **Zizou** (`Zizou/`) | Markdown (skills Claude Code) | [matBatista/Zizou](https://github.com/matBatista/Zizou) | ver [README](../README.md) | Esta biblioteca. Orquestrador de agentes em worktree separado. |

Outras pastas em `Projects/` não indexadas aqui: `A Manager`, `Brassfoot`, `bulkfoot-motion`, `daily-challenge`, `welcome-to-docker`.

## Relações

- `bulkfoot/services/manager` ← subtree de `foot_manager` (remote `manager-svc`).
- `bulkfoot/services/analysts` ← subtree de `Football Analysts` (remote `analysts-svc`).
- `bulkfoot/services/predict` ← cópia de `foot_predict` (sem subtree; já divergiu — tem backtest/odds a mais).
- `bulkfoot/web` faz login via API do ManagerFC (cookie `mfc_token`) e protege `/api/predict/train_model` com ele.
