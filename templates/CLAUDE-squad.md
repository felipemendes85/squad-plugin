# Squad Produto — Orquestração e protocolo de handoff

Este projeto usa o plugin **squad-produto** (5 agentes: PO, UX, Full Stack Dev,
QA, DPO/Segurança). As regras abaixo valem para TODOS os agentes do squad.

## Cadeia de produção

```
demanda → po → ux → fullstack-dev → qa ─┐
                                  └→ dpo-security (paralelo ao qa)
                                        └→ /release (gate humano)
```

Cada etapa só começa quando a anterior entregou o artefato válido:

| Agente | Lê | Escreve (é dono de) |
|--------|----|---------------------|
| `po` | demanda do Felipe | `docs/backlog.md` |
| `ux` | `docs/backlog.md` | `docs/ux-specs.md` |
| `fullstack-dev` | backlog + ux-specs + `docs/security-guidelines.md` | código-fonte + `docs/dev-notes.md` |
| `qa` | backlog + ux-specs + dev-notes | `tests/` + `docs/qa-report.md` |
| `dpo-security` | tudo acima + guidelines | `docs/security-review.md` |

Nenhum agente escreve nos arquivos dos outros.

## Protocolo de handoff (obrigatório para todo agente)

Todo artefato termina com a seção `## Handoff` neste formato:

```markdown
## Handoff
- **Status:** concluído | bloqueado | concluído com ressalvas
- **Para:** <próximo agente da cadeia>
- **O que entreguei:** <1-3 linhas, com os IDs de story cobertos>
- **Premissas assumidas:** <lista; se não assumiu nenhuma, escreva "nenhuma">
- **Questões em aberto:** <lista de dúvidas que NÃO bloquearam, mas alguém
  precisa responder; ou "nenhuma">
- **Confiança:** alta | média | baixa — <por quê, em uma frase>
```

Regras do protocolo:

1. **Antes de começar, leia o handoff do agente anterior** e valide as
   premissas que afetam o seu trabalho. Se discordar de uma premissa,
   registre a divergência no seu artefato — nunca corrija em silêncio.
2. **Premissa não declarada é bug.** Se você precisou decidir algo que o
   artefato anterior não cobria, isso é uma premissa: declare.
3. **Status `bloqueado` para a esteira.** O orquestrador traz a questão ao
   Felipe em vez de deixar o agente adivinhar.
4. **Confiança baixa exige justificativa** e sinaliza ao orquestrador que a
   saída merece revisão humana antes de virar insumo da próxima etapa.

## Convenções do projeto

- Comunicação sempre em português do Brasil.
- Stories identificadas como `US-xxx`; testes referenciam o ID da story.
- Código em branch; merge na `main` somente via `/release` com aprovação
  humana explícita.
- Dado pessoal (LGPD) é sempre sinalizado no artefato para revisão do DPO.
