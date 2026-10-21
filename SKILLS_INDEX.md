# Índice de skills

Cada skill é `skills/<nome>/SKILL.md` (frontmatter `name` + `description`). Instalação: ver [README](README.md#instalar).

## Existentes

| Skill | Quando usar | Base |
|---|---|---|
| [fly-deploy-api](skills/fly-deploy-api/SKILL.md) | Deploy/diagnóstico de API conteinerizada no Fly.io (Postgres attach, secrets, /health) | foot_manager |
| [go-migration](skills/go-migration/SKILL.md) | Nova migration golang-migrate embutida (embed.FS) na API Go do ManagerFC | foot_manager/api |
| [apifootball-ingest](skills/apifootball-ingest/SKILL.md) | Cliente API-Football cache-first, ciente de cota, com resolução robusta de team_id | foot_predict, Football Analysts |
| [footballdata-odds](skills/footballdata-odds/SKILL.md) | Odds de fechamento 1X2/OU da football-data.co.uk + match rate (sem Brasileirão) | bulkfoot/services/predict |
| [dixon-coles-backtest](skills/dixon-coles-backtest/SKILL.md) | Backtest de ROI com split temporal contra closing odds | bulkfoot/services/predict |
| [expo-screen](skills/expo-screen/SKILL.md) | Nova tela/aba/modal Expo Router + store Zustand persistido | LIFEOS/mobile, foot_manager/mobile |
| [nextjs-gateway](skills/nextjs-gateway/SKILL.md) | Next.js como gateway (rewrites env-driven / route handler) para vários backends | bulkfoot/web |
| [docker-compose-stack](skills/docker-compose-stack/SKILL.md) | Compose com db + serviços + web e segredos via .env sem commitar | bulkfoot |
| [statsbomb-viz](skills/statsbomb-viz/SKILL.md) | Shot map, pass map, radar e métricas (xG, xT…) com football_analyst | Football Analysts |
| [multi-session-git](skills/multi-session-git/SKILL.md) | Disciplina de git com várias sessões/agentes no mesmo repo | foot_manager/CLAUDE.md |

Fora do Zizou, já existe `bulkfoot/.claude/skills/github-flow` (Issue → branch → PR → merge-check) — específica do bulkfoot.

## Necessárias (gaps a criar)

| Skill | Por quê | Onde ajudaria |
|---|---|---|
| `supabase-sync` | LifeOS é só local (AsyncStorage); falta sync/backup na nuvem com resolução de conflitos | LIFEOS |
| `eas-build-publish` | Nenhum projeto tem `eas.json`; publicar na App Store/Play (EAS build/submit, credenciais, API URL de produção) | foot_manager/mobile, LIFEOS |
| `mobile-api-config` | `foot_manager/mobile/services/api.ts` usa `localhost:8080` em dev e `https://api.managerfc.com` em prod — que não bate com o app Fly (`managerfc-api.fly.dev`); padronizar URL por ambiente (`EXPO_PUBLIC_*`) | foot_manager/mobile |
| `ci-setup` | Padronizar GitHub Actions (go test, pytest+ruff, tsc/expo, next build) — hoje só bulkfoot e Football Analysts têm CI | foot_manager, foot_predict, LIFEOS |
| `test-setup` | Testes para o mobile (Jest/RNTL) e para a FastAPI do predict — hoje inexistentes | foot_manager/mobile, LIFEOS, foot_predict |
| `new-project-scaffold` | Novo projeto com CLAUDE.md, .env.example, .gitignore de segredos, CI e launch.json | todos |
| `subtree-sync` | Atualizar `bulkfoot/services/*` a partir dos repos upstream e resolver divergência do predict (que não é subtree) | bulkfoot |
| `model-train-deploy` | Retreinar e versionar os `.joblib` (~400 MB, gitignored) e levá-los ao container | foot_predict, bulkfoot |
| `fly-deploy-python` | Variante do deploy Fly para as FastAPIs (predict/analyst), hoje só o Go está no Fly | foot_predict, Football Analysts |
