---
name: go-migration
description: Use ao adicionar ou alterar schema/seed do Postgres na API Go do ManagerFC (foot_manager/api ou bulkfoot/services/manager/api) — cria o par NNNN_nome.up.sql/.down.sql em internal/db/migrations, embutido via embed.FS e aplicado pelo golang-migrate no startup.
---

# Nova migration golang-migrate (embed.FS) — ManagerFC

## Como funciona hoje

- Arquivos: `api/internal/db/migrations/NNNN_descricao.{up,down}.sql` (4 dígitos, sequencial; a última é `0014_formations`).
- `api/internal/db/db.go`: `//go:embed migrations/*.sql` + `iofs.New(...)` + `m.Up()`; troca `postgres://` por `pgx5://` porque o driver é `pgx/v5`. `migrate.ErrNoChange` é ignorado.
- `api/cmd/server/main.go` chama `db.Migrate(databaseURL)` **antes** de abrir o pool; falha = `log.Fatalf("migrations failed")` = app não sobe (inclusive no Fly).

## Passos

1. Descubra o próximo número (não reutilize nem pule):
   ```bash
   cd ~/Claude/Projects/foot_manager/api
   ls internal/db/migrations | sort | tail -2
   ```
2. Crie o par com o mesmo prefixo, ex. `0015_minha_mudanca`:
   ```bash
   N=0015; NAME=minha_mudanca
   touch internal/db/migrations/${N}_${NAME}.up.sql internal/db/migrations/${N}_${NAME}.down.sql
   ```
3. `up.sql`: seja idempotente quando der (`ADD COLUMN IF NOT EXISTS`, `CREATE TABLE IF NOT EXISTS`) — padrão já usado em `0014_formations.up.sql`. Backfill de dados vai no mesmo arquivo, depois do DDL.
4. `down.sql`: desfaz na ordem inversa (`DROP COLUMN IF EXISTS ...`). Nunca deixe vazio.
5. Atualize `internal/model/*` e `internal/repository/*` que leem/escrevem as colunas novas.
6. Rode localmente (migrations aplicam sozinhas no startup):
   ```bash
   cd ~/Claude/Projects/foot_manager
   docker compose up -d                       # Postgres 16 em :5432 (user/db brassfoot)
   cd api && go run ./cmd/server              # log deve mostrar a migração aplicada
   go test ./...
   ```
7. Teste o down→up se a migration é destrutiva (CLI do golang-migrate, se instalada):
   ```bash
   migrate -path internal/db/migrations -database "pgx5://brassfoot:brassfoot@localhost:5432/brassfoot?sslmode=disable" down 1
   migrate -path internal/db/migrations -database "pgx5://brassfoot:brassfoot@localhost:5432/brassfoot?sslmode=disable" up
   ```

## Cuidados

- **Nunca edite uma migration já aplicada** (em produção ou no banco de outra pessoa) — crie uma nova.
- Se o banco ficar `dirty` (migration falhou no meio), conserte o SQL e rode `migrate ... force <versão_anterior>` antes de subir de novo.
- Existe uma cópia do serviço em `bulkfoot/services/manager` (git subtree de `foot_manager`). Faça a migration no `foot_manager` e puxe com `git subtree pull --prefix=services/manager manager-svc main --squash` no bulkfoot, para os números não divergirem.
- Seeds de clubes reais (`0006`, `0008`, `0011`) são grandes: para novos dados de seed, prefira migration nova a editar as antigas.
