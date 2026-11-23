
# Lista de funcionalidades do Alume

## 1. Usuários

- **F01 — Autenticar usuário:** permitir que o estudante acesse o Alume utilizando suas credenciais do SUAP.
- **F05 — Encerrar sessão:** permitir que o estudante encerre sua sessão no Alume por meio de um botão na interface, removendo a sessão armazenada e invalidando o JWT do Alume.

## 2. Disciplinas

- **F02 — Consultar disciplinas por dia da semana:** apresentar, na tela inicial, as disciplinas do estudante organizadas pelos dias da semana, de acordo com os horários registrados no SUAP.
- **F03 — Consultar frequência por disciplina:** apresentar, no dashboard, as informações de frequência de cada disciplina, incluindo percentual de frequência, quantidade de faltas, limite de faltas e faltas restantes.
- **F04 — Consultar disciplinas por risco de reprovação por falta:** organizar e destacar, no dashboard, as disciplinas de acordo com o percentual do limite de faltas já consumido, permitindo identificar aquelas em que o estudante precisa ter maior atenção.

## 3. Distribuição por tela

| Tela | Funcionalidades |
| ---- | --------------- |
| Tela de login | F01 |
| Tela inicial (grade semanal de aulas) | F02 |
| Dashboard (informações detalhadas das disciplinas) | F03, F04 |
| Botão de encerrar sessão (disponível nas telas autenticadas) | F05 |
