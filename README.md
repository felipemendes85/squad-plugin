# Squad Produto — Plugin para Claude Code

Squad multi-agente com 7 papéis (PO, UX, Tech Lead, Backend Dev, Frontend Dev,
QA, DPO/Segurança), esteiras orquestradas com gates de integridade, protocolo
de handoff entre agentes, governança Spec-Driven (o Tech Lead publica o
contrato técnico antes de Backend e Frontend codarem em paralelo), loop de
segurança fechado (skill `secure-development` em modo BUILD nos devs e modo
REVIEW no DPO) e gate humano de release. Funciona no Claude Code em qualquer
interface: terminal, VS Code, JetBrains e app desktop.

## Instalação

Este repositório é ao mesmo tempo o plugin e o marketplace. No Claude Code:

```
/plugin marketplace add felipemendes85/squad-plugin
/plugin install squad-produto@felipemendes-plugins
```

Para testar localmente sem publicar:

```
claude --plugin-dir /caminho/para/squad-plugin
```

## Pós-instalação (uma vez por projeto)

Dentro do projeto onde você quer usar o squad, rode:

```
/setup-squad
```

Ele instala o protocolo de handoff no `CLAUDE.md` do projeto (plugins não
carregam CLAUDE.md automaticamente), cria `docs/security-guidelines.md` a
partir do template e prepara a pasta `docs/` onde os agentes escrevem os
artefatos. Nada existente é sobrescrito.

## Skills (invocação por /)

| Skill | Faz | Auto-invocação |
|-------|-----|----------------|
| `/setup-squad` | Prepara o projeto atual (handoff + guidelines + docs/) | Não (só manual) |
| `/demanda <texto>` | Esteira completa PO→UX→Tech Lead→Backend/Frontend Dev→QA→DPO (subagents) | Não (só manual) |
| `/team <texto>` | Mesma cadeia como Agent Team (teammates se falam)* | Não (só manual) |
| `/story <texto>` | Esteira leve PO→UX→Dev fullstack (sem split) para prototipagem | Não (só manual) |
| `/fix-seguranca <risco>` | Dev corrige → QA valida regressão → DPO confirma | Sim |
| `/add-guideline <regra>` | Registra risco novo nas guidelines | Sim |
| `/release [alvo]` | Merge na main só com aprovação humana explícita | Não (só manual) |
| `secure-development` | Framework de segurança (BUILD/REVIEW) usado pelos devs e pelo DPO | Sim |

*`/team` exige agent teams habilitado — adicione ao `.claude/settings.json`
do projeto e reinicie a sessão:

```json
{ "env": { "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1" } }
```

Se o nome curto colidir com skill de outro plugin, use a forma qualificada:
`/squad-produto:demanda`.

## Agentes

`po`, `ux`, `tech-lead`, `backend-dev`, `frontend-dev`, `qa`, `dpo-security`
— cada um com escopo de ferramentas próprio, comunicação direta via
`SendMessage`/`TaskList`/`TaskUpdate`/`TaskGet` (usada de verdade no modo
`/team`), e o protocolo de handoff (status, premissas, questões em aberto,
confiança) que garante comunicação sem ruído entre etapas. O `tech-lead`
decide, por story, se ela é frontend-only (Supabase direto) ou cruza para o
`backend-dev` (Python/FastAPI) — nem toda story precisa dos dois. Um agente
`fullstack-dev` (Python/FastAPI + frontend) continua disponível e é o usado
pela esteira leve `/story`, para prototipagem sem o split completo.

O formato do handoff vive em `templates/CLAUDE-squad.md` e é instalado no
projeto pelo `/setup-squad`.

## Fluxo típico

```
/setup-squad                          # uma vez por projeto
/demanda quero um cadastro de metas   # esteira completa
/fix-seguranca todos os altos         # fecha riscos do DPO
/release                              # gate humano → merge na main
```

## Estrutura do repositório

```
.claude-plugin/   plugin.json + marketplace.json
agents/           os 7 papéis do squad (+ fullstack-dev p/ /story)
skills/           esteiras, loop de segurança, release, setup
templates/        CLAUDE-squad.md (handoff) + security-guidelines.md
```

## Publicando alterações

O `name` do plugin no marketplace é imutável após publicado (renomear quebra
a instalação de quem já tem). Para mudar o rótulo exibido, altere
`displayName`. Versione via `version` no `plugin.json`. O
`.claude-plugin/marketplace.json` precisa estar na raiz do repositório no
GitHub — os arquivos deste diretório SÃO a raiz do repo `squad-plugin`.
