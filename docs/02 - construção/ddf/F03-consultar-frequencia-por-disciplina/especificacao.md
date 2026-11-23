
# F03 — Consultar frequência por disciplina

## Objetivo

Permitir que o estudante consulte, no dashboard, as informações de frequência de cada disciplina do período letivo atual, incluindo percentual de frequência, quantidade de faltas, limite de faltas e faltas restantes.

## Atores

- Estudante
- SUAP (sistema externo)

## Pré-condições

- O estudante possui uma sessão ativa no Alume.

## Fluxo principal

1. O estudante acessa o dashboard do Alume.
2. O Alume valida a sessão do estudante.
3. O Alume determina o período letivo atual a partir da data da requisição.
4. O Alume realiza uma requisição à rota de disciplinas da API do SUAP para obter as disciplinas e suas respectivas informações de frequência no período letivo atual.
5. O SUAP retorna os dados de frequência das disciplinas: quantidade total de aulas, quantidade atual de faltas e percentual de frequência.
6. O Alume processa as informações recebidas e calcula, para cada disciplina, o limite de faltas (25% da quantidade total de aulas, arredondado para baixo) e a quantidade de faltas restantes.
7. O Alume apresenta as informações de frequência organizadas por disciplina.

## Fluxos alternativos

**FA01 — Nenhuma disciplina encontrada**

1. O Alume identifica que não existem disciplinas vinculadas ao estudante no período letivo atual.
2. O Alume informa que não foram encontradas disciplinas para consulta.

**FA02 — API do SUAP indisponível**

1. O Alume não consegue estabelecer comunicação com a API do SUAP.
2. O Alume informa que o serviço de consulta acadêmica está temporariamente indisponível.
3. O estudante permanece no dashboard.

**FA03 — Sessão inválida ou expirada**

1. O Alume identifica que o JWT é inválido ou expirado, ou que a sessão não existe mais no Redis.
2. O Alume rejeita a requisição e informa que a sessão expirou.
3. O estudante é redirecionado para a tela de login.

**FA04 — Serviço de sessão indisponível**

1. O Alume não consegue consultar o Redis para validar a sessão.
2. O Alume informa que o serviço de sessão está temporariamente indisponível.
3. O estudante permanece no dashboard.

## Pós-condições

- Em caso de sucesso, as informações de frequência são exibidas para cada disciplina.
- Em caso de falha na consulta, as informações não são atualizadas e o estudante é informado sobre o problema.
