---
name: fix-seguranca
description: Pega um risco apontado pelo DPO em security-review.md, manda o Dev corrigir, o QA validar a correção e o DPO confirmar que o risco foi fechado.
argument-hint: <ID ou descrição do risco, ou "todos os altos">
---

Você é o ORQUESTRADOR do loop de correção de segurança. O Felipe quer resolver
o(s) risco(s):

<risco>
$ARGUMENTS
</risco>

## Fonte de verdade

Leia `docs/security-review.md`. Identifique o(s) risco(s) indicado(s) acima.
Se o Felipe disse "todos os altos", pegue todos os riscos de severidade Alta.
Se a referência estiver ambígua, liste os riscos do review e pergunte qual.

## Loop de correção

### Etapa 1 — Dev corrige
- Acione o subagent `fullstack-dev`.
- Passe a ele: a descrição do risco, o arquivo/linha apontado pelo DPO, a
  recomendação de correção, e as regras de `docs/security-guidelines.md`.
- Peça que ele corrija em branch e registre em `docs/dev-notes.md` o que
  mudou e por quê (referenciando o risco).
- Gate: se o Dev tiver dúvida sobre a abordagem de correção, pause e traga
  ao Felipe.

### Etapa 2 — QA valida a correção
- Acione o subagent `qa`.
- Peça um teste automatizado que PROVE que o risco foi fechado (ex: teste que
  falha se a vulnerabilidade voltar — regressão de segurança). Além de rodar a
  suíte existente para garantir que a correção não quebrou nada.
- Saída: atualizar `docs/qa-report.md` com o resultado do teste de segurança.
- Gate: se o teste de correção falhar, devolva ao Dev. Repita até passar.

### Etapa 3 — DPO confirma
- Acione o subagent `dpo-security`.
- Peça que ele reavalie especificamente o(s) risco(s) tratado(s) e atualize
  `docs/security-review.md`: marque o risco como **RESOLVIDO** (com data e o
  que foi feito) ou, se ainda houver exposição, mantenha aberto com o motivo.

## Entrega

Resumo para o Felipe:
- Risco(s) tratado(s) e severidade original.
- O que o Dev mudou.
- Como o QA provou que fechou (nome do teste de regressão).
- Confirmação do DPO: resolvido ou ainda aberto.
- Status: risco fechado? Sim / Não — e o que falta.

## Regras de integridade

- Todo risco corrigido precisa ter um teste que impeça a regressão.
- Não marque como resolvido sem o QA verde E a confirmação do DPO.
- Se travar, pare e reporte.
