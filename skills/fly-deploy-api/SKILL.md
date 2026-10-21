---
name: fly-deploy-api
description: Use ao fazer deploy (ou redeploy/diagnóstico de 502) de uma API conteinerizada no Fly.io — criar app, provisionar/anexar Postgres, setar secrets como JWT_SECRET, rodar fly deploy e verificar /health. Base real: ManagerFC (foot_manager) e bulkfoot/services/manager.
---

# Deploy de API no Fly.io

Referência viva: `~/Claude/Projects/foot_manager/DEPLOY.md` + `fly.toml` (mesmos arquivos em `bulkfoot/services/manager/`).

## Pré-checagem (antes de qualquer comando)

1. O nome do app está em `fly.toml` (`app = "..."`). No ManagerFC: `managerfc-api`, região `gru`, porta interna `8080`. **Sempre leia o fly.toml** em vez de supor o nome.
2. Confirme que existe um `Dockerfile` no mesmo diretório do `fly.toml` (`[build] dockerfile = "Dockerfile"`, contexto = esse diretório).
3. Confirme que a API lê `PORT`, `DATABASE_URL` e os secrets do ambiente e expõe `GET /health` (ManagerFC: `api/cmd/server/main.go` faz `log.Fatal` se `DATABASE_URL`/`JWT_SECRET` faltam).
4. `fly auth whoami` — se não estiver logado, **peça ao usuário** para rodar `fly auth login` (ação da conta dele).

## Passos (troque `APP` e `DB` pelos nomes reais)

```bash
APP=managerfc-api      # de fly.toml
DB=managerfc-db

# 1. Criar app (pule se já existe: fly apps list)
fly apps create $APP --org personal

# 2. Postgres gerenciado + attach (attach seta o secret DATABASE_URL sozinho)
fly postgres create --name $DB --region gru --vm-size shared-cpu-1x --volume-size 1
fly postgres attach $DB --app $APP

# 3. Secrets restantes — gere, nunca escreva o valor em arquivo/commit
fly secrets set JWT_SECRET="$(openssl rand -hex 32)" --app $APP

# 4. Deploy (rodar no diretório do fly.toml)
fly deploy

# 5. Verificar
fly status --app $APP
fly logs --app $APP          # espere "migrations ..." e "Listening on :8080"
curl https://$APP.fly.dev/health   # esperado: {"status":"ok"}
```

`fly postgres create` e `fly apps create` cobram/criam recursos na conta do usuário: confirme com ele antes.

## Diagnóstico de 502 "App is not listening on expected port"

O processo morreu antes do `Listen`. Ordem:

```bash
fly logs --app $APP            # procure "DATABASE_URL is not set" / "JWT_SECRET is not set" / "migrations failed"
fly secrets list --app $APP    # precisa listar DATABASE_URL e JWT_SECRET
fly status --app $DB           # cluster Postgres saudável?
fly postgres attach $DB --app $APP   # re-anexar se DATABASE_URL sumiu
fly machines list --app $APP
```

## Cuidados

- `auto_stop_machines = true` + `min_machines_running = 0`: a primeira request depois de ociosidade é lenta (cold start). Não confunda com falha.
- Nunca commite `.env`; em produção os valores vêm só de `fly secrets`.
- Migrations rodam no startup (ver skill [go-migration](../go-migration/SKILL.md)); uma migration quebrada derruba o deploy — teste localmente antes.
- Build local equivalente: `docker build -t $APP .` e `docker run --rm -e DATABASE_URL=... -e JWT_SECRET=dev -p 8080:8080 $APP`.
- Úteis: `fly ssh console --app $APP`, `fly postgres connect --app $DB`, `fly open --app $APP`.
