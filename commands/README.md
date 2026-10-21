# Cheatsheet de comandos

Comandos lidos dos projetos reais (READMEs, CLAUDE.md, TESTING.md, Dockerfiles, CI). Caminhos relativos a `~/Claude/Projects/`.

## ManagerFC — `foot_manager/`

```bash
cd foot_manager
docker compose up -d                                   # Postgres 16 em :5432 (brassfoot/brassfoot)
cd api && cp .env.example .env && go mod tidy && go run ./cmd/server   # API :8080, migrations no startup
go build ./... && go vet ./... && go test ./...        # dentro de api/
curl localhost:8080/health
cd ../mobile && npm install && npx expo start          # Expo (QR / Expo Go)
npx expo start --web --port 19006                      # web (config do .claude/launch.json)
npx tsc --noEmit                                       # typecheck mobile
docker build -t managerfc-api .                        # imagem de produção (raiz)
```

## foot_predict — `foot_predict/analytics/` (cópia em `bulkfoot/services/predict/analytics/`)

```bash
cd foot_predict/analytics
python -m venv venv && source venv/bin/activate && pip install -r requirements.txt
cp .env.example .env                                   # API_FOOTBALL_KEY, OPENAI_API_KEY
uvicorn main:app --reload                              # http://127.0.0.1:8000  (GET /health)
python -m http.server 5500 -d view                     # front estático (view/)
python train_and_save_model.py                         # retreina modelos (models/ é gitignored)
python eval_dixon_coles.py --data matches_multi.csv --xi 0.0019
python fetch_multi_season.py --leagues PL LL SA BL FL1 BSA --seasons 2022 2023 2024 --out matches_multi.csv
# só na cópia do bulkfoot:
python backtest_roi.py --data matches_multi.csv --markets 1x2 ou25
```

## bulkfoot — `bulkfoot/`

```bash
cd bulkfoot
cp .env.example .env                                   # JWT_SECRET, POSTGRES_PASSWORD
cp services/predict/analytics/.env.example services/predict/analytics/.env
docker compose up --build                              # web :3000, manager :8080, predict :8000, analyst :8001
docker compose logs -f manager-api
docker compose down                                    # (down -v apaga o banco)
cd web && cp .env.local.example .env.local && npm install && npm run dev   # só o web, backends no host
npm run build && npm run lint                          # biome
git subtree pull --prefix=services/manager manager-svc main --squash
git subtree pull --prefix=services/analysts analysts-svc main --squash
```

## Football Analysts — `Football Analysts/`

```bash
cd "Football Analysts"
source .venv/bin/activate                              # Python 3.10+
pip install -e ".[dev]"
python examples/run_shot_map.py
python examples/run_pass_map.py
python examples/run_radar.py
python examples/run_xt.py
python examples/run_worldcup.py
python examples/run_apifootball.py                     # precisa API_FOOTBALL_KEY (ou cache)
uvicorn webapi.app:app --port 8001                     # wrapper FastAPI usado pelo bulkfoot
ruff check . && pytest -q                              # = CI
```

## LifeOS — `LIFEOS/mobile/`

```bash
cd LIFEOS/mobile
npm install
npx expo start                                         # dev server
npx expo start --web                                   # web em :8081
npx tsc --noEmit
```

## Fly.io (ManagerFC API)

```bash
fly auth whoami
fly deploy                                             # na pasta do fly.toml
fly status --app managerfc-api
fly logs --app managerfc-api
fly secrets list --app managerfc-api
fly secrets set JWT_SECRET="$(openssl rand -hex 32)" --app managerfc-api
fly postgres attach managerfc-db --app managerfc-api
fly ssh console --app managerfc-api
fly postgres connect --app managerfc-db
curl https://managerfc-api.fly.dev/health
```

## GitHub (gh)

```bash
gh repo view matBatista/<repo> --web
gh issue create -R matBatista/bulkfoot --title "..." --body "..."
gh pr create -R matBatista/bulkfoot --fill             # descrição com "Closes #N"
gh pr list -R matBatista/bulkfoot
gh pr checks <N> -R matBatista/bulkfoot
gh run list -R matBatista/<repo> --limit 5
gh run view <run-id> --log-failed
```

## Expo / EAS (ainda não configurado em nenhum projeto — sem eas.json)

```bash
npm install -g eas-cli && eas login
eas build:configure
eas build --platform ios
eas submit --platform ios
```
