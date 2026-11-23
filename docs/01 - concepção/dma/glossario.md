# Glossário — Alume

> Este glossário reúne os principais termos utilizados na documentação do **Alume**.

---

## Termos Gerais

| Termo | Definição |
| :--- | :--- |
| **Alume** | Plataforma web proposta para auxiliar os estudantes do Instituto Federal de Sergipe (IFS) no gerenciamento de faltas nas disciplinas do período letivo atual. |
| **Aplicativo oficial do IFS** | Aplicativo institucional que disponibiliza, em dispositivos móveis, uma visualização simplificada de informações acadêmicas, incluindo dados relacionados às faltas. |
| **Banco de dados próprio** | Banco de dados mantido pelo Alume. O Alume utiliza o Redis como banco de dados próprio **temporário**, exclusivamente para sessões e tokens de acesso. Não há banco de dados permanente (como o PostgreSQL), e as informações acadêmicas dos usuários não são armazenadas. |
| **Campus Aracaju** | Campus do IFS ao qual se restringe o público-alvo inicial do Alume, devido à disponibilidade das informações necessárias no SUAP. |
| **Cookie HttpOnly** | Cookie que não pode ser lido por JavaScript no navegador. É utilizado pelo Alume para guardar o JWT do Alume. |
| **Dashboard** | Tela do Alume que apresenta as informações detalhadas de cada disciplina: total de aulas, faltas, percentual de frequência, limite de faltas, faltas restantes e risco de reprovação por falta. |
| **Diário** | Registro acadêmico do SUAP associado à oferta de uma disciplina. O Alume o utiliza apenas para obter os dias da semana e os horários das aulas. |
| **Disciplina** | Componente curricular no qual o estudante está matriculado durante o período letivo. |
| **Falta** | Ausência do estudante contabilizada em uma aula de determinada disciplina. |
| **Faltas restantes** | Número de aulas em que o estudante ainda pode faltar sem ultrapassar o limite calculado para a disciplina. É obtido pela subtração das faltas atuais do limite de faltas. |
| **Frequência** | Participação do estudante nas aulas de uma disciplina. No contexto do Alume, está relacionada ao acompanhamento das faltas e ao cumprimento da frequência mínima exigida. |
| **Gerenciamento de faltas** | Acompanhamento das faltas atuais, do limite máximo permitido e das faltas restantes em cada disciplina. |
| **Grade semanal de aulas** | Representação dos dias da semana e dos horários em que ocorrem as aulas das disciplinas do período letivo atual. |
| **IFS** | Instituto Federal de Sergipe, instituição cujos estudantes constituem o público-alvo do Alume. |
| **JWT do Alume** | Token emitido pelo backend do Alume após a autenticação bem-sucedida no SUAP. Contém o identificador da sessão e a data de expiração, e é guardado em um cookie HttpOnly. |
| **Limite de faltas** | Quantidade máxima de faltas que o estudante pode acumular em uma disciplina, calculada com base no percentual máximo de 25% de faltas sobre a quantidade total de aulas, com arredondamento para baixo. |
| **Matrícula** | Número que identifica o estudante no SUAP. Corresponde ao `username` utilizado na autenticação. |
| **Percentual de frequência** | Percentual de frequência do estudante em uma disciplina, conforme informado pelo SUAP (campo `frequencia`). O Alume o exibe sem recalcular. |
| **Percentual do limite consumido** | Proporção do limite de faltas que o estudante já utilizou em uma disciplina. É a base para classificar o risco de reprovação por falta. |
| **Período letivo** | Intervalo semestral de aulas, identificado por ano e número do período (por exemplo, 2027.1). Em meses anteriores a agosto, o período é o 1; de agosto em diante, é o 2. Corresponde ao parâmetro `{semestre}` das rotas do SUAP. |
| **Plataforma web** | Sistema acessado por meio de um navegador e de um dispositivo conectado à internet. O Alume será disponibilizado nesse formato. |
| **PWA** | *Progressive Web App*: aplicação web que pode ser instalada e utilizada em dispositivos móveis com experiência próxima à de um aplicativo nativo. |
| **Quantidade atual de faltas** | Número de faltas que o estudante possui, até o momento da consulta, em uma determinada disciplina (campo `qtd_faltas` do SUAP). |
| **Quantidade total de aulas** | Número total de aulas previsto para uma disciplina no período letivo (campo `ch_total_aula` do SUAP). É utilizado como base para calcular o limite de faltas. |
| **Redis** | Banco de dados em memória utilizado pelo Alume, de forma temporária, para armazenar as sessões e o token de acesso do SUAP, com expiração automática (TTL). |
| **Risco de reprovação por falta** | Classificação de uma disciplina (crítico, em risco ou estável) de acordo com o percentual do limite de faltas já consumido. |
| **Sessão** | Período em que o estudante permanece autenticado no Alume. É representada por um registro no Redis, identificado por um `sid`, e pelo JWT do Alume no cookie do navegador. |
| **SUAP** | Sistema Unificado de Administração Pública. No contexto do projeto, é o sistema utilizado pelos estudantes para consultar informações acadêmicas, como a quantidade de aulas de uma disciplina e a quantidade atual de faltas. |
| **Tela inicial** | Tela do Alume exibida após o login, que apresenta a grade semanal de aulas. |
| **Token do SUAP** | Token de acesso (*access token*) retornado pelo SUAP na autenticação. Fica sob responsabilidade do backend do Alume e nunca é enviado ao navegador. O *refresh token* retornado pelo SUAP não é utilizado nem armazenado. |

---

## Termos e Fórmulas Utilizados nos Cálculos

### Símbolos

| Símbolo | Significado |
| :---: | :--- |
| **$L$** | Limite de faltas da disciplina. |
| **$Q$** | Quantidade total de aulas da disciplina. |
| **$F$** | Quantidade atual de faltas do estudante na disciplina. |
| **$F_r$** | Quantidade de faltas restantes, ou seja, quantas faltas ainda podem ser acumuladas sem ultrapassar o limite. |
| **$P_c$** | Percentual do limite de faltas já consumido. |

### Fórmulas

* **Cálculo do limite de faltas:**
  $$L = \lfloor Q \times 0{,}25 \rfloor$$

* **Cálculo das faltas restantes:**
  $$F_r = L - F$$

* **Cálculo do percentual do limite consumido:**
  $$P_c = \frac{F}{L} \times 100$$

> **Observação:** o resultado fracionário do limite de faltas é sempre arredondado para baixo, pois a falta é contada por aula inteira. Por exemplo, para $Q = 54$, tem-se $54 \times 0{,}25 = 13{,}5$ e, portanto, $L = 13$. O estudante é considerado reprovado por falta quando $F > L$. O valor de $Q$ corresponde ao campo `ch_total_aula` do SUAP.
