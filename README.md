# Zizou

A Manager — central de **skills reutilizáveis do Claude Code** entre os projetos em `~/Claude/Projects/` (ManagerFC, foot_predict, bulkfoot, Football Analysts, LifeOS).

## Conteúdo

| Caminho | O quê |
|---|---|
| [`skills/`](skills/) | Skills instaláveis: uma pasta por skill com `SKILL.md` (frontmatter `name`/`description` + passos com comandos e caminhos reais) |
| [`SKILLS_INDEX.md`](SKILLS_INDEX.md) | Tabela das skills existentes + gaps a criar |
| [`commands/README.md`](commands/README.md) | Cheatsheet de comandos por projeto (db/API/Expo, FastAPI, compose, viz, fly, gh) |
| [`projects/README.md`](projects/README.md) | Índice dos projetos: stack, repo, como rodar, status |

## Instalar

Skills do Claude Code são lidas de `~/.claude/skills/<nome>/SKILL.md` (todas as sessões) ou `<repo>/.claude/skills/<nome>/SKILL.md` (só aquele repo). Para usar a biblioteca inteira via symlink (edições no Zizou valem na hora):

```bash
mkdir -p ~/.claude/skills
for s in ~/Claude/Projects/Zizou/skills/*/; do ln -sfn "$s" ~/.claude/skills/"$(basename "$s")"; done
```

Ou só uma skill num projeto: `ln -sfn ~/Claude/Projects/Zizou/skills/go-migration ~/Claude/Projects/foot_manager/.claude/skills/go-migration`.

## Adicionar uma skill

1. `skills/<nome-kebab>/SKILL.md` com frontmatter:
   ```yaml
   ---
   name: <nome-kebab>
   description: <quando usar — controla o acionamento>
   ---
   ```
2. Leia o projeto real antes e cite só comandos/caminhos que existem; o que faltar vira `TODO` explícito.
3. Adicione uma linha em `SKILLS_INDEX.md` (e tire do gap, se estava lá).

## Orquestrador de agentes

O "manager de agentes" (orquestração de várias sessões/agentes) é uma frente **separada, em desenvolvimento** num worktree próprio (`.claude/worktrees/agent-manager-locker-room-*`, branch `claude/agent-manager-locker-room-*`). Esta biblioteca de skills não depende dele.
