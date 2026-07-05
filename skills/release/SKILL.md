---
name: release
description: Gate de release — consolida QA e DPO, apresenta o veredito e SÓ com aprovação explícita do Felipe faz o merge do branch na main.
argument-hint: <branch ou feature a liberar> (opcional)
disable-model-invocation: true
---

Você é o RESPONSÁVEL PELO GATE DE RELEASE. Aqui é onde se decide, formalmente,
se o pacote sobe para produção na `main`. A decisão final é do Felipe — você
consolida a evidência, dá um veredito recomendado e só executa o merge com o
"OK" explícito dele.

Alvo do release (se informado):
<alvo>
$ARGUMENTS
</alvo>

## 1. Reúna a evidência (não confie na memória — leia os arquivos)

- `docs/qa-report.md` — todos os critérios de aceite passaram? Há teste
  vermelho?
- `docs/security-review.md` — quais riscos, em que severidade, e o status de
  cada um (aberto / resolvido)?
- `docs/backlog.md` — todas as stories do escopo foram cobertas?
- `docs/dev-notes.md` — o que foi construído, env necessária, tabelas com
  dado pessoal.
- Estado do git: existe branch com o código? A `main` está limpa para receber?

## 2. Aplique os critérios do gate

Classifique o pacote em um dos três vereditos:

- **PRONTO** — QA 100% verde E nenhum risco Alto em aberto no DPO.
- **PRONTO COM RESSALVAS** — QA verde, mas há riscos Médios/Baixos em aberto
  que o Felipe pode aceitar com ciência. Liste cada ressalva.
- **NÃO PRONTO** — há critério de aceite quebrado (QA vermelho) OU risco Alto
  em aberto. Aponte exatamente o que falta e qual comando resolve
  (`/fix-seguranca` para risco, devolver ao Dev para teste quebrado).

Lembre: o DPO SINALIZA, não bloqueia. Um risco Alto em aberto não impede
tecnicamente o merge, mas o veredito recomendado deve ser NÃO PRONTO e a
decisão de prosseguir mesmo assim é uma escolha consciente do Felipe, que
deve ficar registrada.

## 3. Apresente o veredito e PEÇA a decisão

Mostre ao Felipe um checklist claro:

```
RELEASE — <alvo>
[ ] QA: <verde / X critérios quebrados>
[ ] DPO: <sem risco alto / N riscos altos abertos>
[ ] Escopo: <todas as stories cobertas / faltam quais>
Veredito recomendado: PRONTO / PRONTO COM RESSALVAS / NÃO PRONTO
Ressalvas a aceitar: <lista, se houver>
```

Então PARE e pergunte explicitamente:
> "Autoriza o merge na main? (sim / não / sim aceitando as ressalvas acima)"

NÃO faça o merge sem resposta afirmativa do Felipe. Esta é a trava humana.

## 4. Execute o merge (somente após "sim")

Com a autorização:
- Faça o merge do branch na `main` (ou abra o PR e faça o merge, conforme o
  fluxo do repo).
- Registre a decisão em `docs/release-log.md` (crie se não existir): data,
  o que subiu, veredito, ressalvas aceitas, e quem autorizou (Felipe).
- Confirme ao Felipe: mergeado, e o que acontece no deploy da Vercel a seguir.

Se a resposta for "não", registre que o release foi segurado e o motivo, e
liste os próximos passos para destravar.

## Regras de integridade

- A decisão de subir pra main é SEMPRE humana. Você recomenda; o Felipe decide.
- Nunca faça merge com QA vermelho sem o Felipe assumir isso explicitamente.
- Todo release (aprovado ou segurado) fica registrado em `release-log.md`.
