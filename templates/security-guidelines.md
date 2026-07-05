# Security Guidelines — <nome do projeto>

Regras de segurança e privacidade DESTE projeto. O `fullstack-dev` lê este
arquivo ANTES de escrever qualquer código; o `dpo-security` usa como
prioridade máxima na revisão. Regras novas entram via `/add-guideline`.

> Complementa (não substitui) a skill `secure-development` do plugin, que
> cobre as 12 categorias técnicas gerais. Aqui ficam as regras específicas
> da empresa e do domínio.

## Segredos

- Todo segredo via variável de ambiente. Nada hardcoded, nada commitado.
  `.env` no `.gitignore`; apenas `.env.example` com placeholders no repo.
- Chave `service_role` do Supabase NUNCA no cliente nem em código de frontend.

## Autenticação e autorização

- Row Level Security ligado em TODA tabela com dado de usuário, desde a
  primeira migration.
- Mensagens de erro de auth genéricas (sem revelar se o usuário existe).

## Dados pessoais / LGPD

- Toda tabela com dado pessoal é sinalizada em `docs/dev-notes.md`.
- Dado de saúde é sensível (art. 11 LGPD): base legal específica,
  minimização estrita, proteção reforçada.
- Retenção e exclusão definidas por tabela; direitos do titular exequíveis.
- PII nunca em logs.

## Deploy

- Deploy na Vercel apenas a partir da `main`, que só recebe merge via
  `/release` (gate humano).
- Preview deployments não apontam para o banco de produção.

---

## Histórico

| Data | Mudança |
|------|---------|
| <data> | Criação do guideline a partir do template do plugin squad-produto. |
