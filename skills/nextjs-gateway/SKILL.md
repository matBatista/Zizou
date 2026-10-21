---
name: nextjs-gateway
description: Use ao expor um ou mais backends através de um app Next.js como gateway de API — rewrites() em next.config.js dirigidos por env (localhost em dev, nomes de serviço no Docker) e route handler quando precisa de timeout longo, allowlist ou auth. Base: bulkfoot/web.
---

# Next.js como gateway de API (bulkfoot/web)

Arquivos reais em `~/Claude/Projects/bulkfoot/web/`:
- `next.config.js` — rewrites `/api/analyst/:path*` e `/api/manager/:path*`.
- `app/api/predict/[...path]/route.ts` — proxy manual do predict (timeout 5 min, allowlist, login obrigatório em `train_model`).
- `.env.local.example`, `Dockerfile` (build args), `lib/*.ts` (clientes usam `BASE = "/api/<svc>"`).

## Adicionar um backend novo (ex. `stats` na porta 8002)

1. `next.config.js`:
   ```js
   const STATS = process.env.STATS_API_URL || "http://localhost:8002";
   // em rewrites():
   { source: "/api/stats/:path*", destination: `${STATS}/:path*` },
   ```
2. `.env.local.example`: `STATS_API_URL=http://localhost:8002`.
3. `web/Dockerfile` (estágio build): `ARG STATS_API_URL=http://localhost:8002` + `ENV STATS_API_URL=$STATS_API_URL`.
4. `docker-compose.yml` → serviço `web`: em `build.args` **e** em `environment`: `STATS_API_URL: http://stats-api:8002` (nome do serviço, não localhost).
5. Cliente: `web/lib/stats.ts` com `const BASE = "/api/stats"` — o browser nunca fala direto com o backend.

## Rewrite vs route handler

- **Rewrite** (padrão): passa tudo, sem código. Os valores são **lidos no `next build`** — mudar a env em runtime não muda o destino; por isso o Dockerfile recebe build args.
- **Route handler** (`app/api/<svc>/[...path]/route.ts`): use quando precisar de (a) timeout longo, (b) allowlist de paths (`ALLOWED`), (c) checar login (o predict valida o cookie `mfc_token` chamando `${MANAGER}/api/v1/manager/me`), (d) filtrar headers (`FORWARD_HEADERS` — nunca repasse cookies do browser). Lê `process.env` em **runtime**. Não declare rewrite e handler para o mesmo prefixo.

## Verificar

```bash
cd ~/Claude/Projects/bulkfoot/web
cp .env.local.example .env.local && npm install && npm run dev
curl -s localhost:3000/api/manager/health      # deve responder o /health do Go
curl -s localhost:3000/api/predict/health
npm run build && npm run lint                  # CI roda npm ci + npm run build
```
