
# F05 — Encerrar sessão

## Objetivo

Permitir que o estudante encerre sua sessão no Alume por meio de um botão na interface, removendo a sessão armazenada e invalidando o JWT do Alume.

## Atores

- Estudante

## Pré-condições

- O estudante possui uma sessão ativa no Alume.

## Fluxo principal

1. O estudante aciona o botão de encerrar sessão disponível na interface.
2. O Alume envia ao backend uma requisição autenticada de encerramento de sessão.
3. O backend valida o JWT do Alume e extrai o identificador de sessão (`sid`).
4. O backend remove a sessão correspondente do Redis, o que invalida o JWT do Alume.
5. O backend remove o cookie do JWT do navegador e confirma o encerramento da sessão.
6. O estudante é redirecionado para a tela de login.

## Regras

- Após a remoção da sessão no Redis, novas requisições com o JWT correspondente são rejeitadas, mesmo que o JWT ainda não tenha atingido sua expiração.
- O Alume não consegue invalidar o token emitido pelo SUAP. Ele apenas deixa de armazená-lo, e o token continua válido no SUAP até a sua própria expiração.

## Fluxos alternativos

**FA01 — Sessão já inválida ou expirada**

1. O Alume identifica que o JWT é inválido ou expirado, ou que a sessão não existe mais no Redis.
2. O Alume considera a sessão encerrada e remove o cookie do navegador.
3. O estudante é redirecionado para a tela de login.

**FA02 — Serviço de sessão indisponível**

1. O Alume não consegue remover a sessão do Redis.
2. O Alume informa que não foi possível encerrar a sessão no momento e que o estudante pode tentar novamente.
3. O estudante permanece na tela em que estava.

## Pós-condições

- Em caso de sucesso, a sessão não existe mais no Redis, o JWT do Alume não é mais aceito e o estudante precisa se autenticar novamente para acessar o Alume.
- Em caso de falha, a sessão permanece ativa até o seu tempo de expiração.
