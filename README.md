# Squad Produto — Plugin para Claude Code

Squad multi-agente com 5 papéis (PO, UX, Full Stack Dev, QA, DPO/Segurança),
esteiras orquestradas com gates de integridade, protocolo de handoff entre
agentes, loop de segurança fechado (skill `secure-development` em modo BUILD
no Dev e modo REVIEW no DPO) e gate humano de release. Funciona no Claude Code
em qualquer interface: terminal, VS Code, JetBrains e app desktop.

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

## Skills (invocação por /)

| Skill | Faz | Auto-invocação |
|-------|-----|----------------|
| `/demanda <texto>` | Esteira completa PO→UX→Dev→QA→DPO (subagents) | Não (só manual) |
| `/team <texto>` | Mesma cadeia como Agent Team (teammates se falam)* | Não (só manual) |
| `/story <texto>` | Esteira leve PO→UX→Dev para prototipagem | Não (só manual) |
| `/fix-seguranca <risco>` | Dev corrige → QA valida regressão → DPO confirma | Sim |
| `/add-guideline <regra>` | Registra risco novo nas guidelines | Sim |
| `/release [alvo]` | Merge na main só com aprovação humana explícita | Não (só manual) |
| `secure-development` | Framework de segurança (BUILD/REVIEW) usado por Dev e DPO | Sim |

*`/team` exige `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` no settings.json e
reinício da sessão.

## Agentes

`po`, `ux`, `fullstack-dev`, `qa`, `dpo-security` — cada um com escopo de
ferramentas próprio e o protocolo de handoff (status, premissas, questões em
aberto, confiança) que garante comunicação sem ruído entre etapas.

## Pós-instalação (por projeto)

1. Crie `docs/security-guidelines.md` com as regras da empresa (o Dev lê antes
   de codar; o DPO usa como prioridade). Um modelo acompanha o projeto de
   referência.
2. Copie o `CLAUDE.md` de orquestração (protocolo de handoff + cadeia) para
   `.claude/CLAUDE.md` do projeto — plugins não carregam CLAUDE.md
   automaticamente.

## Publicando alterações

O `name` do plugin no marketplace é imutável após publicado (renomear quebra
a instalação de quem já tem). Para mudar o rótulo exibido, altere
`displayName`. Versione via `version` no `plugin.json`.
