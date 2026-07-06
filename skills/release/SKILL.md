---
name: release
description: Gate de release para produção — consolida QA e DPO, apresenta o veredito e SÓ com aprovação explícita do Felipe faz o merge do branch na main. Use quando o Felipe pedir para subir/liberar/publicar/mandar para produção, fazer deploy, ou liberar o release — em linguagem natural ou via /release.
argument-hint: <branch ou feature a liberar> (opcional)
---

RESPONSÁVEL PELO GATE DE RELEASE: decide, formalmente, se o pacote sobe para
`main`. Decisão final é do Felipe — você consolida evidência, dá veredito
recomendado, só executa o merge com "OK" explícito.

Alvo (se informado): `$ARGUMENTS`

## 1. Evidência (leia os arquivos, não confie na memória)

- `docs/qa-report.md` — critérios de aceite passaram? Teste vermelho?
- `docs/security-review.md` — riscos, severidade, status (aberto/resolvido)?
- `docs/backlog.md` — todas as stories do escopo cobertas?
- `docs/dev-notes.md` — o que foi construído, env, tabelas com dado pessoal.
- Git: branch com o código existe? `main` limpa para receber?

## 2. Classifique o veredito

- **PRONTO** — QA 100% verde E nenhum risco Alto aberto no DPO.
- **PRONTO COM RESSALVAS** — QA verde, riscos Médios/Baixos abertos que o
  Felipe pode aceitar com ciência. Liste cada um.
- **NÃO PRONTO** — critério quebrado (QA vermelho) OU risco Alto aberto.
  Aponte o que falta e o comando que resolve (`/fix-seguranca` para risco,
  devolver ao Dev para teste quebrado).

DPO sinaliza, não bloqueia — risco Alto aberto não impede tecnicamente o
merge, mas o veredito recomendado é NÃO PRONTO; prosseguir mesmo assim é
escolha consciente do Felipe, registrada.

## 3. Apresente e PEÇA a decisão

```
RELEASE — <alvo>
[ ] QA: <verde / X critérios quebrados>
[ ] DPO: <sem risco alto / N riscos altos abertos>
[ ] Escopo: <todas as stories cobertas / faltam quais>
Veredito recomendado: PRONTO / PRONTO COM RESSALVAS / NÃO PRONTO
Ressalvas a aceitar: <lista, se houver>
```

PARE e pergunte: "Autoriza o merge na main? (sim / não / sim aceitando as
ressalvas acima)". NÃO faça merge sem resposta afirmativa — esta é a trava
humana, vale mesmo se o pedido original veio em linguagem natural.

## 4. Merge (só após "sim")

- Merge do branch na `main` (ou PR + merge, conforme o fluxo do repo).
- Registre em `docs/release-log.md` (crie se não existir): data, o que
  subiu, veredito, ressalvas aceitas, quem autorizou.
- Confirme: mergeado, e o que acontece no deploy da Vercel a seguir.

"Não" → registre que o release foi segurado, o motivo, e os próximos passos
para destravar.

## Regras

- Subir pra main é SEMPRE decisão humana. Você recomenda; o Felipe decide.
- Nunca faça merge com QA vermelho sem o Felipe assumir isso explicitamente.
- Todo release (aprovado ou segurado) fica em `release-log.md`.
