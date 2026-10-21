---
name: docker-compose-stack
description: Use ao orquestrar (ou adicionar um serviço a) um docker-compose com Postgres + vários backends + web, passando segredos via .env/env_file sem commitar chaves e encadeando healthchecks. Base: bulkfoot/docker-compose.yml.
---

# Stack docker compose (db + serviços + web)

Referência: `~/Claude/Projects/bulkfoot/docker-compose.yml` (+ `.env.example`, `.gitignore`).
Topologia: `db` (postgres:16-alpine) → `manager-api` :8080 (Go) · `predict-api` :8000 (FastAPI) · `analyst-api` :8001 (FastAPI) → `web` :3000 (Next.js gateway, ver [nextjs-gateway](../nextjs-gateway/SKILL.md)).

## Segredos — três jeitos, todos fora do git

1. **Interpolação obrigatória** do `.env` da raiz (ao lado do compose):
   ```yaml
   POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:?set POSTGRES_PASSWORD in .env}
   JWT_SECRET: ${JWT_SECRET:?set JWT_SECRET in .env}
   ```
   O `:?` faz o compose recusar subir se faltar — melhor que default fraco.
2. **`env_file`** por serviço para chaves de API: `env_file: [./services/predict/analytics/.env]` (contém `API_FOOTBALL_KEY`, `OPENAI_API_KEY`).
3. Sempre comite só o `.env.example` com chaves vazias; confirme com `git check-ignore -v .env services/predict/analytics/.env` antes de commitar.

## Adicionar um serviço

```yaml
  stats-api:
    build:
      context: ./services/stats
      dockerfile: Dockerfile
    env_file: [./services/stats/.env]
    ports: ["8002:8002"]
    depends_on:
      db:
        condition: service_healthy    # só se usa o banco
```
- Entre containers use o **nome do serviço** (`db:5432`, `http://stats-api:8002`), nunca `localhost`.
- `DATABASE_URL` monta com a senha interpolada: `postgres://brassfoot:${POSTGRES_PASSWORD}@db:5432/brassfoot?sslmode=disable`. Use só caracteres URL-safe na senha.
- Adicione o serviço ao `depends_on` do `web` e as URLs em `build.args` + `environment` do `web`.

## Rodar / depurar

```bash
cd ~/Claude/Projects/bulkfoot
cp .env.example .env                                    # preencha JWT_SECRET (openssl rand -hex 32) e POSTGRES_PASSWORD
cp services/predict/analytics/.env.example services/predict/analytics/.env   # API_FOOTBALL_KEY
docker compose config >/dev/null && echo ok             # valida interpolação sem subir
docker compose up --build                               # http://localhost:3000
docker compose ps
docker compose logs -f manager-api
docker compose down          # mantém volume pgdata;  down -v APAGA o banco (confirme antes)
```

Cuidado: `services/predict/analytics/models/` (~400 MB de .joblib) é gitignored — copie de `foot_predict/analytics/models/` ou rode `python train_and_save_model.py`, senão o predict sobe sem modelo.
