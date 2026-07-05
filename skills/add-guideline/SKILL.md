---
name: add-guideline
description: Adiciona uma regra/risco novo ao docs/security-guidelines.md, para que Dev e DPO passem a considerá-lo desde o começo nas próximas demandas.
argument-hint: <descrição da nova regra ou risco de segurança>
---

O Felipe quer registrar uma nova diretriz de segurança/privacidade no guideline
do squad:

<nova_regra>
$ARGUMENTS
</nova_regra>

## O que fazer

1. Leia `docs/security-guidelines.md` (crie se não existir, com um cabeçalho
   e uma seção de histórico no fim).

2. Verifique se a regra já está contemplada. Se for duplicata ou conflitar
   com uma regra existente, avise o Felipe e proponha como reconciliar em vez
   de simplesmente adicionar.

3. Transforme a entrada do Felipe numa diretriz clara e acionável:
   - **Regra:** o que deve (ou não deve) ser feito, em uma frase imperativa.
   - **Por quê:** o risco que ela previne.
   - **Como verificar:** o que o Dev implementa e o que o DPO checa para
     confirmar conformidade. Se der, sugira o tipo de teste automatizado
     (o QA pode usar isso como regressão).
   - **Origem:** de onde veio (ex: "risco identificado no review da US-004"
     ou "guideline do DPO da empresa"), com a data.

4. Adicione a diretriz numa seção categorizada apropriada (ex: Segredos,
   Autenticação, Dados Pessoais/LGPD, Dependências, Deploy). Se a categoria
   não existir, crie.

5. Registre no histórico no fim do arquivo: data + resumo do que foi adicionado.

## Efeito no squad

A partir de agora essa diretriz vale para TODAS as próximas demandas:
- O `fullstack-dev` lê o guideline ANTES de codar e já previne o problema.
- O `dpo-security` inclui a nova regra na revisão.

## Entrega

Confirme ao Felipe: a diretriz adicionada (formatada), a categoria onde entrou,
e se ela substitui/ajusta alguma regra anterior.

Comunique-se em português do Brasil.
