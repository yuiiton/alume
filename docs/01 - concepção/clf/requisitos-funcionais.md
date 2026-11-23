# Requisitos funcionais do Alume

## RF001 - Visualização da grade semanal de aulas

- Descrição: O sistema deve permitir visualizar a grade semanal de aulas com base nos dias da semana, na tela inicial.

## RF002 - Visualização do percentual de frequência da disciplina

- Descrição: O sistema deve exibir ao aluno o percentual de frequência da disciplina, conforme informado pelo SUAP.

## RF003 - Visualização do limite de faltas da disciplina

- Descrição: O sistema deve mostrar o número máximo de faltas permitido para a disciplina, calculado como 25% da quantidade total de aulas, com arredondamento para baixo.

## RF004 - Visualização da quantidade de faltas atuais

- Descrição: O sistema deve apresentar a quantidade de faltas já registradas na disciplina.

## RF005 - Visualização da quantidade de faltas restantes

- Descrição: O sistema deve indicar quantas faltas o aluno ainda pode ter antes de ultrapassar o limite da disciplina.

## RF006 - Login com as credenciais do SUAP

- Descrição: O sistema deve permitir que o usuário autentique-se com suas credenciais do SUAP (matrícula e senha), obtendo uma sessão autenticada por meio de um JWT próprio do Alume.

## RF007 - Consulta de disciplinas do período letivo atual

- Descrição: O sistema deve consultar as disciplinas do aluno para o período letivo atual, determinado a partir da data da requisição: em meses anteriores a agosto, o 1º período do ano; de agosto em diante, o 2º período.

## RF008 - Visualização em interface responsiva

- Descrição: O sistema deve funcionar em navegador em desktop e em dispositivos móveis, com experiência apropriada para PWA.

## RF009 - Exibição de insights de faltas

- Descrição: O sistema deve apresentar indicadores e avisos sobre a situação da disciplina, como "faltas restantes baixas" ou "aluno próximo do limite", classificando o risco de reprovação por falta com base no percentual do limite de faltas já consumido.

## RF010 - Encerramento de sessão

- Descrição: O sistema deve permitir que o usuário encerre sua sessão por meio de um botão na interface, removendo a sessão armazenada e invalidando o JWT do Alume.
