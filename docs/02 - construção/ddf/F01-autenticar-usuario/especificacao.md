
# F01 — Autenticar usuário

## Objetivo

Permitir que o estudante acesse o Alume utilizando suas credenciais institucionais do SUAP.

## Atores

- Estudante
- SUAP (sistema externo)

## Pré-condições

- O estudante possui uma conta ativa no SUAP.

## Fluxo principal

1. O estudante informa seu número de matrícula e senha na tela de login do Alume.
2. O Alume envia as credenciais à API de autenticação do SUAP.
3. O SUAP valida as credenciais e retorna os tokens de autenticação (*access token* e *refresh token*).
4. O Alume descarta o *refresh token* e gera um identificador de sessão (`sid`).
5. O Alume armazena o *access token* do SUAP no Redis, associado ao `sid`, com tempo de expiração igual ao do próprio token.
6. O Alume emite o JWT do Alume, contendo o `sid` e a data de expiração (a mesma do *access token* do SUAP), e o envia ao navegador em um cookie HttpOnly.
7. O estudante é redirecionado para a tela inicial do Alume.

## Regras

- A senha do estudante é utilizada apenas durante a autenticação e não é armazenada em nenhum momento.
- O Alume só emite o JWT após confirmar que a sessão foi armazenada com sucesso no Redis.
- Os tokens do SUAP nunca são enviados ao navegador.

## Fluxos alternativos

**FA01 — Credenciais incorretas**

1. O SUAP rejeita a tentativa de autenticação.
2. O Alume informa ao estudante que as credenciais fornecidas são inválidas.
3. O estudante permanece na tela de login.

**FA02 — API do SUAP indisponível**

1. O Alume não consegue estabelecer comunicação com a API do SUAP.
2. O Alume informa que o serviço de autenticação está temporariamente indisponível.
3. O estudante permanece na tela de login.

**FA03 — Serviço de sessão indisponível**

1. O Alume não consegue armazenar a sessão no Redis.
2. O Alume não emite o JWT e informa que o serviço de sessão está temporariamente indisponível.
3. O estudante permanece na tela de login.

## Pós-condições

- Em caso de sucesso, o estudante possui uma sessão ativa no Alume, representada por um registro no Redis e pelo JWT do Alume no cookie do navegador.
- Em caso de falha, nenhuma sessão autenticada é estabelecida e nenhum cookie é enviado ao navegador.
