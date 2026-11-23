
# F04 — Consultar disciplinas por risco de reprovação por falta

## Objetivo

Permitir que o estudante identifique, no dashboard, as disciplinas que apresentam maior risco de reprovação por falta, considerando o percentual do limite de faltas já consumido em cada disciplina.

## Atores

- Estudante
- SUAP (sistema externo)

## Pré-condições

- O estudante possui uma sessão ativa no Alume.

## Fluxo principal

1. O estudante acessa o dashboard do Alume.
2. O Alume valida a sessão do estudante.
3. O Alume determina o período letivo atual a partir da data da requisição.
4. O Alume consulta a rota de disciplinas da API do SUAP para obter as informações de frequência das disciplinas do período letivo atual.
5. O SUAP retorna os dados de frequência das disciplinas.
6. O Alume processa as informações recebidas, calcula o limite de faltas e o percentual do limite já consumido em cada disciplina, e identifica a situação de cada uma em relação ao limite de faltas permitido.
7. O Alume organiza as disciplinas conforme seus respectivos níveis de risco de reprovação por falta.
8. O Alume apresenta os indicadores ao estudante, destacando as disciplinas que exigem maior atenção.

## Fluxos alternativos

**FA01 — Nenhuma disciplina encontrada**

1. O Alume identifica que não existem disciplinas vinculadas ao estudante no período letivo atual.
2. O Alume informa que não há disciplinas disponíveis para análise.

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

- Em caso de sucesso, as disciplinas são apresentadas conforme seus respectivos níveis de risco.
- Em caso de falha na consulta, os indicadores não são atualizados e o estudante é informado sobre o problema.
