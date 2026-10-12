---
name: story
description: Esteira leve para prototipagem rápida — roda PO → UX → Dev, deixando QA e DPO para quando a feature amadurecer.
argument-hint: <descrição da demanda>
disable-model-invocation: true
---

Você é o ORQUESTRADOR de uma esteira LEVE, para prototipar rápido. O Felipe
passou esta demanda:

<demanda>
$ARGUMENTS
</demanda>

Rode apenas PO → UX → Dev, na ordem, com os mesmos gates de integridade da
esteira completa. NÃO rode QA nem DPO aqui — isso fica para o `/demanda` ou
para os comandos de segurança quando a feature amadurecer.

## Esteira

### Etapa 1 — PO
- Acione o subagent `po` com a demanda.
- Se ele precisar de esclarecimento, pause e traga ao Felipe.
- Saída: `docs/backlog.md` com story(ies) e critérios de aceite.
- Gate: sem story válida, reacione o `po`.

### Etapa 2 — UX
- Pré-requisito: `docs/backlog.md` válido.
- Acione o subagent `ux`.
- Saída: `docs/ux-specs.md` cobrindo as stories.
- Gate: toda story precisa ter spec.

### Etapa 3 — Dev
- Pré-requisito: backlog + ux-specs válidos.
- Acione o subagent `fullstack-dev` para implementar em branch.
- Lembre o Dev de ler `docs/security-guidelines.md` ANTES de codar.
- Saída: código + `docs/dev-notes.md`.

## Métricas da execução (obrigatório)

Mantenha `docs/metrics.md`; você, o orquestrador, é o único dono do arquivo.
Cada subagent que terminar vira uma linha, neste formato:

```
| data | demanda | etapa | agente | modelo | volta | tokens | tool_uses | duração_ms | resultado |
```

`tokens`, `tool_uses` e `duração_ms` vêm do bloco `usage` que acompanha o
resultado do subagent (`subagent_tokens`, `tool_uses`, `duration_ms`).
`modelo` vem do frontmatter do agente. `volta` é 1 na primeira execução e
sobe a cada correção. Se o bloco `usage` não vier, escreva `n/d`; nunca
estime. Na entrega, informe o total de tokens, a duração por etapa e a parte
do total gasta em retrabalho (voltas 2 e 3). Limite: o número cobre só os
subagents, não o consumo do próprio orquestrador.

## Entrega

Resumo curto: story(ies), telas principais, o que foi construído, e como
rodar o protótipo. 

IMPORTANTE: avise explicitamente que este é um PROTÓTIPO — ainda NÃO passou
por QA nem pela revisão de segurança do DPO, e portanto NÃO está pronto para
produção. Para promover, rode `/demanda` (completo) ou pelo menos o QA e o DPO.

## Regras de integridade

- Não pule etapas. Não avance sem o artefato da anterior.
- Sempre delegue ao subagent — não faça o trabalho dele.
- Se travar, pare e reporte onde e por quê.
