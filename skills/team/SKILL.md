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
avise o Felipe para adicionar ao `.claude/settings.json` do projeto:

```json
{ "env": { "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1" } }
```

e reiniciar a sessão antes de continuar. Não tente contornar.

Verifique também que os 7 agentes (`po`, `ux`, `tech-lead`, `backend-dev`,
`frontend-dev`, `qa`, `dpo-security`) têm `SendMessage`, `TaskList`,
`TaskUpdate`, `TaskGet` na `tools:` do frontmatter. Sem isso, teammates não
conseguem se falar nem ler a lista de tarefas — o `/team` degrada em
silêncio para uma esteira `/demanda` com você relayando cada troca (já
aconteceu: um teammate tenta avisar outro, não consegue, e o erro só aparece
como retrabalho depois). Se faltar em algum, avise o Felipe e pergunte se
quer que você adicione antes de spawnar o time.

## Monte o time (7 teammates)

Spawne os teammates, cada um com o briefing do papel. IMPORTANTE: teammates
NÃO herdam seu histórico de conversa — carregam CLAUDE.md, MCP e skills do
projeto automaticamente, mas o contexto específico da demanda você precisa
passar no prompt de spawn. Seja generoso no briefing inicial.

Os 7 papéis já estão instalados como subagents no seu ambiente — spawne cada
teammate a partir do subagent correspondente:

- **po** — decupa a demanda em user stories com critérios de aceite em
  `docs/backlog.md`.
- **ux** — especifica telas em `docs/ux-specs.md`.
- **tech-lead** — decide por story se é frontend-only ou cruza pro backend,
  e publica o contrato em `docs/tech-spec.md`. Spawne DEPOIS do ux, ANTES
  dos devs.
- **backend-dev** — implementa o serviço Python contra o contrato, quando a
  story exigir. Se o tech-spec só tiver stories frontend-only, não precisa
  spawnar este teammate agora.
- **frontend-dev** — implementa Next.js/React/TS; CRUD simples direto no
  Supabase, ou contra o contrato do backend quando existir.
- **qa** — testa com pytest + Playwright em `docs/qa-report.md`.
- **dpo** (`dpo-security`) — revisa segurança/LGPD em `docs/security-review.md`,
  sinaliza (não bloqueia).

Para cada teammate, inclua no prompt de spawn: o papel, o arquivo de agente
correspondente como referência, a demanda, e quais arquivos ele é dono de
escrever (para não colidir — agent teams não isolam em worktree por padrão).

## Particione os arquivos (evita colisão)

Agent teams não isolam teammates em worktrees. Cada teammate é DONO de um
conjunto de arquivos e não escreve nos dos outros:

- po → `docs/backlog.md`
- ux → `docs/ux-specs.md`
- tech-lead → `docs/tech-spec.md`
- backend-dev → código do serviço Python (pasta backend do projeto) +
  `docs/dev-notes-backend.md`
- frontend-dev → código do app Next.js (pasta frontend do projeto) +
  `docs/dev-notes-frontend.md`
- qa → `tests/` + `docs/qa-report.md`
- dpo → `docs/security-review.md`

Vantagem estrutural desta divisão: backend-dev e frontend-dev normalmente
escrevem em pastas FÍSICAS diferentes do repo (ex: `forecast/` vs `web/`),
então rodar os dois em paralelo já tem baixo risco de colisão por
construção — não só por convenção. Se mesmo assim o trabalho exigir muitos
arquivos em paralelo, recomende ao Felipe rodar com git worktrees
(`claude --worktree`).

## Lista de tarefas compartilhada e dependências

Popule a lista de tarefas do time codificando as dependências da cadeia:

```
T1 (po)          → sem dependência
T2 (ux)          → depende de T1
T3 (tech-lead)   → depende de T2
T4 (backend-dev)  → depende de T3 (só se a story cruzar pro backend)
T5 (frontend-dev) → depende de T3 (pode rodar em paralelo com T4)
T6 (qa)          → depende de T4 (se existir) e T5
T7 (dpo)         → depende de T4/T5   (pode rodar em paralelo com T6)
```

O coordenador destrava cada tarefa quando a dependência fecha. Aproveite o
paralelismo onde ele existe de verdade:
- Enquanto backend-dev e frontend-dev implementam a story A, o po/ux já
  podem preparar a story B, e o tech-lead já pode publicar o contrato dela.
- qa e dpo podem rodar em paralelo (ambos só dependem dos devs).
- Deixe os teammates se coordenarem por mensagem direta (ex: backend-dev
  avisa o frontend-dev que o endpoint subiu; dpo pergunta ao backend-dev
  onde está o dado sensível; frontend-dev pergunta ao tech-lead se o
  contrato cobre um campo que faltou).

## Gates (iguais aos da esteira /demanda)

- QA verde é pré-requisito de deploy. Se reprovar, o qa mensageia o
  backend-dev ou o frontend-dev (conforme o critério) para corrigir, e
  retesta.
- DPO sinaliza riscos, não bloqueia. Risco novo fora do guideline → recomende
  `/add-guideline`.
- Contrato mudou no meio da implementação → tech-lead avisa backend-dev E
  frontend-dev antes de atualizar `tech-spec.md`. Mudança sem aviso é a
  causa mais comum de retrabalho entre os dois.
- O merge na `main` NÃO acontece aqui. Ao fim, oriente o Felipe a rodar
  `/release` (a trava humana de produção continua valendo).

## Encerramento

Quando as tarefas fecharem, sintetize o resultado (mesmo resumo consolidado da
`/demanda`: stories, telas, roteamento do tech-lead, o que foi construído,
QA, parecer do DPO, status). Depois, desligue os teammates e limpe o time —
teammates ativos consomem tokens.

## Custo (avise o Felipe se a demanda for pequena)

Agent teams são pesados: cada teammate é uma sessão completa. Um time de vários
teammates usa múltiplas vezes os tokens de uma sessão única. Para demanda
simples ou muito sequencial, a esteira `/demanda` (subagents) sai mais barata e
entrega o mesmo. Use `/team` quando o paralelismo e a coordenação direta entre
papéis justificarem o custo — e, se o volume de stories for grande, considere
não spawnar `backend-dev` para stories frontend-only, para não pagar por um
teammate ocioso.
