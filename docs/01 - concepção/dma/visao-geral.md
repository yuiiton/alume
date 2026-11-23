# Introdução

O **Alume** é uma plataforma web cujo objetivo é fornecer aos estudantes do Instituto Federal de Sergipe os dados e ferramentas necessárias para gerenciar suas faltas nas disciplinas do período letivo atual.

# Contexto

O Alume nasce da dificuldade que os estudantes enfrentam para gerenciar suas faltas. Atualmente, esse gerenciamento acontece por meio de duas plataformas principais: o **Suap**, site que disponibiliza informações como quantidade de aulas de uma disciplina e quantidade atual de faltas do estudante, e o **Aplicativo oficial do IFS**, que disponibiliza uma visualização simplificada desses dados para os dispositivos celulares.
Contudo, existem informações importantes para o gerenciamento de faltas que não são disponibilizadas por essas plataformas, como:

1. Limite de faltas de uma disciplina.
   Esse limite é calculado através da regra de porcentagem máxima de faltas em uma disciplina (25%). A fórmula para calcular esse limite é:

   $$
   L = \lfloor Q \times 0{,}25 \rfloor
   $$

   Onde:

   $$
   L = \text{Limite de faltas, expresso em quantidade de aulas*}
   $$

   $$
   Q = \text{Quantidade total de aulas da disciplina prevista para o período letivo.}
   $$

   *O limite é expresso em quantidade de aulas porque se utiliza a quantidade total de aulas, ao invés da carga horária da disciplina. Isso facilita compreender quão próximo ou distante o aluno está do limite de faltas.

   Dessa forma, para uma disciplina de 72 aulas no período letivo, teríamos:

   $$
   L = \lfloor 72 \times 0{,}25 \rfloor \\
   L = \text{18 aulas.}
   $$

   Assim, o estudante consegue saber que se tiver mais de 18 faltas nessa disciplina, ele será reprovado por ultrapassar o limite de faltas.

   Quando o resultado do cálculo é fracionário, o limite é **arredondado para baixo**. Por exemplo, para uma disciplina de 54 aulas, teríamos 54 × 0,25 = 13,5, e o limite de faltas seria de 13 aulas. Isso acontece porque a falta é contada por aula inteira: com 13 faltas o estudante ainda está dentro do limite, mas a falta seguinte elevaria o total para 14 e o reprovaria. Não existe a possibilidade de o estudante acumular 13,5 faltas.

   O problema é que nem o Suap nem o aplicativo do IFS disponibilizam essa informação, obrigando o estudante a realizar esse cálculo manualmente.
2. Faltas restantes em uma disciplina.
   Essa quantidade é calculada a partir da subtração do limite de faltas da disciplina pela quantidade de faltas atual que o estudante possui. Dessa forma, para uma disciplina de 72 aulas no período letivo, cujo limite de faltas é 18 aulas (como calculamos anteriormente), um estudante com uma quantidade X de faltas conseguiria descobrir quantas vezes ainda pode faltar a partir deste cálculo:

   $$
   F_r = L - F
   $$

   Onde:

   $$
   F_r = \text{Quantidade de faltas restantes, expressa em aulas.}
   $$

   $$
   F = \text{Quantidade atual de faltas do estudante na disciplina.}
   $$

   Dessa forma, se um estudante possui 8 faltas nessa disciplina, teríamos:

   $$
   F_r = 18 - 8 \\
   F_r = \text{10 aulas.}
   $$

   O problema permanece o mesmo citado no primeiro tópico: nem o Suap, nem o aplicativo do IFS disponibilizam essas informações, obrigando o estudante a realizar esses cálculos manualmente.

Em suma, o Alume visa automatizar esses cálculos e disponibilizar essas informações de maneira simples, para que os estudantes consigam gerenciar suas faltas com segurança e praticidade.


# Objetivos

O Alume pretende permitir que o estudante não precise realizar cálculos manuais para gerenciar suas faltas. Visto que essa prática pode causar diversos problemas, como:
1. Erros de cálculo, tornando o gerenciamento de faltas ineficaz.
2. Consumo de tempo, já que os estudantes precisam buscar manualmente os dados necessários para realizar os cálculos.
3. Dúvidas em relação às faltas, já que os estudantes precisam compreender o sistema de faltas do IFS para gerenciarem suas faltas, o que se torna um problema para calouros.

Dessa forma, o Alume visa apresentar essas informações de maneira automatizada e simples, com o intuito de facilitar o gerenciamento de faltas para todos os estudantes.

# Público-alvo

O Alume tem como público-alvo os estudantes do **Instituto Federal de Sergipe (IFS)**, matriculados no **Campus Aracaju**, independentemente do nível de formação (ensino médio integrado, subsequente ou superior).

Essa delimitação existe porque o Alume consultará informações do Suap que só estão disponíveis para os estudantes do campus de Aracaju, não possuindo garantia de que funcionará adequadamente para estudantes de outros campi.

# Visão geral da solução

O Alume possuirá um site que poderá ser acessado por qualquer dispositivo que esteja conectado à internet. O estudante acessa o site autenticando-se com suas credenciais do Suap e, dentro dele, encontra três telas principais:

- **Tela de login:** permite que o estudante se autentique com sua matrícula e senha do Suap.
- **Tela inicial:** apresenta a grade semanal de aulas, com as disciplinas do período letivo atual organizadas pelos dias da semana e seus horários.
- **Dashboard:** apresenta as informações detalhadas de cada disciplina do período letivo atual:
  - quantidade total de aulas da disciplina
  - quantidade atual de faltas em cada disciplina
  - percentual de frequência em cada disciplina
  - limite máximo de faltas em cada disciplina
  - quantidade de faltas restantes em cada disciplina
  - risco de reprovação por falta de cada disciplina, com base no percentual do limite de faltas já consumido

O estudante também pode encerrar sua sessão a qualquer momento, por meio de um botão disponível na interface.

Dessa forma, o Alume apresenta um panorama completo, de maneira simplificada, para que o estudante possa gerenciar suas faltas.

# Escopo

## Funcionalidades principais

- Autenticar-se no Alume com as credenciais do Suap
- Visualizar a grade semanal de aulas, organizada pelos dias da semana
- Visualizar as disciplinas vigentes do período letivo atual
- Visualizar a quantidade total de aulas de cada disciplina
- Visualizar o percentual de frequência de cada disciplina
- Visualizar a quantidade máxima de faltas permitida em cada disciplina
- Visualizar a quantidade atual de faltas em cada disciplina
- Visualizar a quantidade de faltas restantes em cada disciplina
- Identificar as disciplinas por risco de reprovação por falta
- Encerrar a sessão no Alume

## Limitações

- O Alume não armazenará as informações acadêmicas do usuário em banco de dados próprio, seja ele permanente ou temporário. O único banco de dados próprio do Alume é o **Redis**, de natureza temporária, utilizado exclusivamente para armazenar sessões e tokens de acesso.
- A consulta das informações acadêmicas dependerá da disponibilidade do Suap e de uma conexão ativa com a internet. Caso o Suap esteja indisponível, não será possível consultar os dados acadêmicos por meio do Alume.
- O acesso às funcionalidades autenticadas dependerá da disponibilidade do Redis. Caso o Redis esteja indisponível, não será possível validar as sessões nem acessar essas funcionalidades.
- A sessão no Alume expira junto com o token de acesso emitido pelo Suap. Quando isso ocorrer, o estudante precisará se autenticar novamente.
- O encerramento da sessão no Alume não invalida o token emitido pelo Suap. O Alume apenas deixa de armazená-lo, e o token continua válido até a sua própria expiração.
