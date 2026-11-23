# Modelo de Domínio — Alume

## 1. Objetivo

Este documento apresenta uma representação conceitual do domínio do Alume, identificando os principais conceitos envolvidos na consulta e no acompanhamento da frequência acadêmica de estudantes do Instituto Federal de Sergipe (IFS).

## 2. Contexto do domínio

O Alume é uma plataforma web voltada inicialmente aos estudantes do Campus Aracaju do IFS. Seu propósito é reunir informações de frequência e a grade semanal de aulas das disciplinas do período letivo atual, apresentando indicadores que facilitem o acompanhamento das faltas.

As informações acadêmicas consultadas pelo Alume são obtidas por meio da integração com o SUAP. A partir dos dados disponíveis, o sistema calcula informações que não são apresentadas diretamente pelo SUAP ou pelo aplicativo oficial do IFS, como o limite de faltas, a quantidade de faltas restantes e o risco de reprovação por falta.

O foco deste modelo está no contexto de acompanhamento de frequência. Ele não pretende representar todo o domínio acadêmico do IFS.

## 3. Conceitos do domínio

| Conceito | Descrição |
| -------- | --------- |
| **Estudante** | Pessoa matriculada no IFS que utiliza o Alume para consultar suas informações acadêmicas. |
| **Matrícula** | Registro que identifica o vínculo acadêmico do estudante com a instituição. No SUAP, corresponde ao `username` utilizado na autenticação. |
| **Período letivo** | Intervalo semestral ao qual pertencem as disciplinas consultadas, identificado por ano e número do período (por exemplo, 2027.1). |
| **Disciplina** | Componente curricular no qual o estudante está matriculado durante determinado período letivo. |
| **Diário** | Registro acadêmico associado à oferta de uma disciplina. O Alume o utiliza apenas para obter os dias da semana e os horários das aulas. |
| **Grade semanal de aulas** | Representação dos dias da semana e horários em que ocorrem as aulas dos diários do período letivo atual. |
| **Informações de frequência** | Dados associados à frequência do estudante em uma disciplina: a quantidade total de aulas prevista para o período letivo, as faltas acumuladas e o percentual de frequência. |
| **Limite de faltas** | Quantidade máxima de aulas que o estudante pode perder em uma disciplina, calculada conforme a regra de 25% da quantidade total de aulas, com arredondamento para baixo. |
| **Faltas restantes** | Quantidade de aulas que o estudante ainda pode perder sem ultrapassar o limite calculado. |
| **Percentual do limite consumido** | Proporção do limite de faltas que o estudante já utilizou em uma disciplina. |
| **Risco de reprovação por falta** | Classificação da disciplina (crítico, em risco ou estável) de acordo com o percentual do limite consumido. |
| **Indicadores de frequência** | Informações utilizadas para facilitar o acompanhamento da situação do estudante, incluindo o percentual de frequência fornecido pelo SUAP, o limite de faltas, as faltas restantes, o percentual do limite consumido e o risco de reprovação por falta. |

Os conceitos "limite de faltas", "faltas restantes", "percentual do limite consumido" e "risco de reprovação por falta" representam resultados derivados dos dados acadêmicos. Eles não precisam ser tratados como entidades independentes do domínio ou como registros persistidos.

## 4. Relacionamentos entre os conceitos

- Um estudante possui uma matrícula na instituição.
- Uma matrícula está associada a um período de vínculo acadêmico.
- Durante um período letivo, o estudante pode estar matriculado em várias disciplinas.
- Um diário contém informações sobre os dias da semana e horários das aulas, utilizadas para montar a grade semanal.
- Para cada vínculo entre estudante e disciplina no período letivo, existem informações de frequência correspondentes.
- As informações de frequência permitem ao Alume apresentar o percentual de frequência e calcular indicadores como o limite de faltas, as faltas restantes, o percentual do limite consumido e o risco de reprovação por falta.

## 5. Regras de negócio relacionadas ao domínio

### 5.1. Limite de faltas

O Alume considera a regra segundo a qual o estudante pode faltar a até 25% do total de aulas da disciplina. Quando o resultado é fracionário, o limite é arredondado para baixo.

$$
L = \lfloor Q \times 0{,}25 \rfloor
$$

Onde:

- `L`: limite de faltas;
- `Q`: quantidade total de aulas da disciplina prevista para o período letivo.

O arredondamento para baixo se justifica porque a falta é contada por aula inteira. Em uma disciplina de 54 aulas, o cálculo resulta em 13,5; o estudante com 13 faltas ainda está dentro do limite, mas a falta seguinte eleva o total para 14 e o reprova. Não existe a possibilidade de acumular 13,5 faltas.

### 5.2. Quantidade de faltas restantes

A quantidade de faltas restantes é calculada subtraindo as faltas atuais do limite de faltas.

$$
F_r = L - F
$$

Onde:

- `F_r`: quantidade de faltas restantes;
- `L`: limite de faltas;
- `F`: quantidade atual de faltas.

### 5.3. Reprovação por falta

O estudante é considerado reprovado por falta em uma disciplina quando a quantidade atual de faltas ultrapassa o limite de faltas:

$$
F > L
$$

### 5.4. Percentual do limite consumido e risco de reprovação

O risco de reprovação por falta é determinado pelo percentual do limite de faltas já consumido:

$$
P_c = \frac{F}{L} \times 100
$$

Onde:

- `P_c`: percentual do limite consumido;
- `F`: quantidade atual de faltas;
- `L`: limite de faltas.

A classificação da disciplina em **crítico**, **em risco** ou **estável** é feita a partir de faixas de `P_c`. As faixas ainda precisam ser definidas.

## 6. Origem e transformação das informações

O modelo considera duas origens para as informações apresentadas:

1. **Dados acadêmicos obtidos do SUAP:**
   - da rota de **disciplinas**: disciplinas do período letivo atual, quantidade total de aulas, faltas atuais e percentual de frequência;
   - da rota de **diários**: dias da semana e horários das aulas de cada disciplina, usados para montar a grade semanal.
2. **Informações calculadas pelo Alume:** limite de faltas, faltas restantes, percentual do limite consumido, risco de reprovação por falta e demais indicadores que possam ser derivados dos dados acadêmicos.

O Alume atua como uma camada de consulta e processamento dessas informações. Os dados acadêmicos do estudante não são armazenados pelo Alume, nem no Redis, que guarda somente sessões e tokens de acesso de forma temporária.
