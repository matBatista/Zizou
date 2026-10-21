---
name: multi-session-git
description: Use no início de qualquer sessão que vai editar um repo que outras sessões/agentes de IA também editam (foot_manager, bulkfoot, Zizou...) e sempre antes de commitar — checar estado, isolar por área/branch/worktree e fazer commits pequenos e frequentes.
---

# Git com múltiplas sessões/agentes

Origem: seção "Version control & multi-session workflow" de `~/Claude/Projects/foot_manager/CLAUDE.md`. Git é a **única** camada de coordenação entre sessões.

## Ao começar

```bash
git status                 # tem trabalho não commitado de outra sessão? NÃO sobrescreva — pergunte.
git log --oneline -5
git worktree list          # quais worktrees/agentes existem
git fetch && git status -sb   # atrás do remoto? git pull --ff-only
```

## Enquanto trabalha

- **Uma sessão por vez** é o padrão seguro. Duas ao mesmo tempo só se isoladas:
  - por área (uma em `api/` Go, outra em `mobile/` TS), ou
  - por branch / `git worktree add ../<repo>-<feature> -b <branch>` (Claude Code: `.claude/worktrees/`).
- **Não toque em worktree de outro agente** (não edite, mova, mergeie nem remova) a menos que o usuário peça.
- Commit a cada pedaço funcionando. Mensagem diz o quê e por quê (`feat(api): ...`, `fix(mobile): ...`).
- `git add <caminhos>` explícitos — evite `git add -A`/`.` (pega `.env`, `.claude/worktrees`, arquivos de outra sessão).

## Antes de commitar/push

```bash
git status && git diff --stat          # só o que você mudou?
git diff --cached --name-only | grep -E '\.env$|secret|\.key$' && echo "PARE: segredo no stage"
git pull --rebase --autostash          # se o remoto andou
git push
```

## Repos com regras próprias

- **bulkfoot**: fluxo Issue → branch `<tipo>/<N>-<slug>` → PR com "Closes #N"; skill local `bulkfoot/.claude/skills/github-flow`. Use `gh ... -R matBatista/bulkfoot` (o checkout tem remotes extras `manager-svc`, `analysts-svc`).
- **foot_manager / Football Analysts / Zizou**: commit direto em `main` é aceito; ainda assim commits pequenos.
