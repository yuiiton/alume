
# F02 — Consultar disciplinas por dia da semana

## Objetivo

Permitir que o estudante visualize, na tela inicial, as disciplinas programadas para cada dia da semana (grade semanal de aulas) no período letivo atual.

## Atores

- Estudante
- SUAP (sistema externo)

## Pré-condições

- O estudante possui uma sessão ativa no Alume.

## Fluxo principal

1. O estudante acessa a tela inicial do Alume.
2. O Alume valida a sessão do estudante.
3. O Alume determina o período letivo atual a partir da data da requisição.
4. O Alume consulta a rota de diários da API do SUAP para obter os dias da semana e os horários das disciplinas do estudante no período letivo atual.
5. O Alume processa as informações recebidas e organiza as disciplinas conforme os dias da semana em que possuem aulas programadas.
6. O Alume apresenta as disciplinas organizadas por dia da semana.

## Fluxos alternativos

**FA01 — Não há disciplinas programadas para determinado dia**

1. O Alume identifica que não existem disciplinas programadas para o dia consultado.
2. O Alume informa que não há disciplinas programadas para aquele dia.

**FA02 — API do SUAP indisponível**

1. O Alume não consegue estabelecer comunicação com a API do SUAP.
2. O Alume informa que o serviço de consulta acadêmica está temporariamente indisponível.
3. O estudante permanece na tela inicial.

**FA03 — Sessão inválida ou expirada**

1. O Alume identifica que o JWT é inválido ou expirado, ou que a sessão não existe mais no Redis.
2. O Alume rejeita a requisição e informa que a sessão expirou.
3. O estudante é redirecionado para a tela de login.

**FA04 — Serviço de sessão indisponível**

1. O Alume não consegue consultar o Redis para validar a sessão.
2. O Alume informa que o serviço de sessão está temporariamente indisponível.
3. O estudante permanece na tela inicial.

## Pós-condições

- Em caso de sucesso, as disciplinas são exibidas conforme os dias da semana.
- Em caso de falha na consulta, as disciplinas não são atualizadas e o estudante é informado sobre o problema.
