---
name: team
description: Roda a demanda como AGENT TEAM (teammates que se comunicam entre si), não como subagents. Exige a flag CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS habilitada. Use quando quiser paralelismo real e coordenação direta entre os papéis.
argument-hint: <descrição da demanda>
disable-model-invocation: true
---

Você é o TEAM LEAD (líder do time). Diferente da esteira `/demanda` (que usa
subagents em sequência), aqui você monta um AGENT TEAM: teammates que rodam em
sessões próprias, compartilham a lista de tarefas e se comunicam DIRETAMENTE
entre si, sem passar sempre por você.

Demanda do Felipe:
<demanda>
$ARGUMENTS
</demanda>

## Pré-requisito (verifique antes de tudo)

Agent teams exigem a flag `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` no
settings.json ou no ambiente. Se ela não estiver ativa, o Claude cria
subagents em vez de teammates. Se você perceber que a flag não está ativa,
avise o Felipe para habilitá-la (o arquivo `.claude/settings.json` já traz a
configuração) e reiniciar a sessão antes de continuar. Não tente contornar.

## Monte o time (5 teammates)

Spawne 5 teammates, cada um com o briefing do papel. IMPORTANTE: teammates
NÃO herdam seu histórico de conversa — carregam CLAUDE.md, MCP e skills do
projeto automaticamente, mas o contexto específico da demanda você precisa
passar no prompt de spawn. Seja generoso no briefing inicial.

- **po** — decupa a demanda em user stories com critérios de aceite em
  `docs/backlog.md`. Papel descrito em `.claude/agents/po.md`.
- **ux** — especifica telas em `docs/ux-specs.md`. Ver `.claude/agents/ux.md`.
- **dev** — implementa em Python/Supabase/Vercel, lê as guidelines de
  segurança ANTES de codar. Ver `.claude/agents/fullstack-dev.md`.
- **qa** — testa com pytest + Playwright em `docs/qa-report.md`. Ver
  `.claude/agents/qa.md`.
- **dpo** — revisa segurança/LGPD em `docs/security-review.md`, sinaliza (não
  bloqueia). Ver `.claude/agents/dpo-security.md`.

Para cada teammate, inclua no prompt de spawn: o papel, o arquivo de agente
correspondente como referência, a demanda, e quais arquivos ele é dono de
escrever (para não colidir — agent teams não isolam em worktree por padrão).

## Particione os arquivos (evita colisão)

Agent teams não isolam teammates em worktrees. Cada teammate é DONO de um
conjunto de arquivos e não escreve nos dos outros:

- po → `docs/backlog.md`
- ux → `docs/ux-specs.md`
- dev → código-fonte + `docs/dev-notes.md`
- qa → `tests/` + `docs/qa-report.md`
- dpo → `docs/security-review.md`

Se o trabalho exigir o Dev tocando muitos arquivos de código em paralelo,
recomende ao Felipe rodar com git worktrees (`claude --worktree`).

## Lista de tarefas compartilhada e dependências

Popule a lista de tarefas do time codificando as dependências da cadeia:

```
T1 (po)  → sem dependência
T2 (ux)  → depende de T1
T3 (dev) → depende de T2
T4 (qa)  → depende de T3
T5 (dpo) → depende de T3   (pode rodar em paralelo com T4)
```

O coordenador destrava cada tarefa quando a dependência fecha. Aproveite o
paralelismo onde ele existe de verdade:
- Enquanto o dev implementa a story A, o po/ux já podem preparar a story B.
- qa e dpo podem rodar em paralelo (ambos só dependem do dev).
- Deixe os teammates se coordenarem por mensagem direta (ex: dev avisa o qa
  que o endpoint subiu; dpo pergunta ao dev onde está o dado sensível).

## Gates (iguais aos da esteira /demanda)

- QA verde é pré-requisito de deploy. Se reprovar, o qa mensageia o dev para
  corrigir, e retesta.
- DPO sinaliza riscos, não bloqueia. Risco novo fora do guideline → recomende
  `/add-guideline`.
- O merge na `main` NÃO acontece aqui. Ao fim, oriente o Felipe a rodar
  `/release` (a trava humana de produção continua valendo).

## Encerramento

Quando as tarefas fecharem, sintetize o resultado (mesmo resumo consolidado da
`/demanda`: stories, telas, o que foi construído, QA, parecer do DPO, status).
Depois, desligue os teammates e limpe o time — teammates ativos consomem tokens.

## Custo (avise o Felipe se a demanda for pequena)

Agent teams são pesados: cada teammate é uma sessão completa. Um time de vários
teammates usa múltiplas vezes os tokens de uma sessão única. Para demanda
simples ou muito sequencial, a esteira `/demanda` (subagents) sai mais barata e
entrega o mesmo. Use `/team` quando o paralelismo e a coordenação direta entre
papéis justificarem o custo.
