---
name: setup-squad
description: Prepara o projeto atual para usar o squad — cria docs/security-guidelines.md e instala o protocolo de handoff no CLAUDE.md do projeto. Rode UMA vez por projeto, logo depois de instalar o plugin.
disable-model-invocation: true
---

Você vai preparar o projeto ATUAL para trabalhar com o squad de 5 agentes do
plugin `squad-produto`. Os templates estão no diretório do plugin, em
`${CLAUDE_PLUGIN_ROOT}/templates/`.

## Passos

1. **Protocolo de handoff no CLAUDE.md do projeto.**
   - Leia `${CLAUDE_PLUGIN_ROOT}/templates/CLAUDE-squad.md`.
   - Se o projeto NÃO tem `CLAUDE.md` (nem `.claude/CLAUDE.md`), crie
     `CLAUDE.md` na raiz com esse conteúdo.
   - Se JÁ tem, verifique se a seção "Protocolo de handoff" já existe.
     Se não existir, APENDE o conteúdo do template ao final do arquivo
     existente (não sobrescreva nada do que o Felipe já escreveu).
     Se existir, avise que já está instalado e não duplique.

2. **Guidelines de segurança do projeto.**
   - Se `docs/security-guidelines.md` NÃO existe, crie a partir de
     `${CLAUDE_PLUGIN_ROOT}/templates/security-guidelines.md`, preenchendo
     o nome do projeto e a data de hoje no histórico.
   - Se já existe, não toque — apenas confirme que está lá.

3. **Estrutura de docs.**
   - Garanta que o diretório `docs/` existe (os agentes escrevem os
     artefatos nele: backlog.md, ux-specs.md, dev-notes.md, qa-report.md,
     security-review.md).

4. **Pergunte sobre o domínio.** Pergunte ao Felipe em uma frase: o que é
   este projeto e se ele lida com dados pessoais ou dados de saúde. Ajuste
   a seção LGPD do `security-guidelines.md` recém-criado conforme a resposta
   (ex: projeto sem dado de saúde → remova a regra do art. 11 e anote no
   histórico).

## Entrega

Confirme ao Felipe o que foi criado/alterado e mostre os próximos passos:

```
Squad pronto neste projeto. Fluxo de uso:
  /demanda <o que você quer construir>   → esteira completa PO→UX→Dev→QA→DPO
  /story <ideia>                          → protótipo rápido (PO→UX→Dev)
  /fix-seguranca <risco>                  → corrige risco apontado pelo DPO
  /release                                → gate humano para subir à main
```

## Regras

- NUNCA sobrescreva um CLAUDE.md ou security-guidelines.md existente — só
  apenda ou avise.
- Se `${CLAUDE_PLUGIN_ROOT}` não estiver disponível por algum motivo,
  localize o diretório do plugin instalado (procure por
  `**/squad-produto/templates/CLAUDE-squad.md` no diretório de plugins do
  Claude Code) antes de desistir.
