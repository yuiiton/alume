# Alume

**Acompanhe suas faltas. Tenha clareza sobre sua frequência.**

O **Alume** é uma plataforma web desenvolvida para auxiliar estudantes do Instituto Federal de Sergipe (IFS) no acompanhamento e gerenciamento de faltas acadêmicas.

Integrado à API do SUAP, o sistema reúne informações de frequência e grade semanal de aulas, automatizando cálculos que normalmente precisam ser realizados manualmente. Dessa forma, o estudante consegue visualizar o limite de faltas de cada disciplina, quantas faltas ainda pode ter e quais disciplinas exigem maior atenção.

> **Escopo inicial:** estudantes do Campus Aracaju do IFS.

---

## Sobre o projeto

O gerenciamento de faltas exige que o estudante acompanhe diferentes informações acadêmicas e realize cálculos para compreender sua situação em cada disciplina.

Embora o SUAP disponibilize informações como a quantidade total de aulas e as faltas acumuladas, alguns indicadores importantes não são apresentados diretamente, como o limite máximo de faltas e a quantidade de faltas restantes.

O Alume surge para simplificar esse processo, consultando os dados acadêmicos diretamente no SUAP e transformando-os em informações mais claras e úteis para o estudante.

### Funcionalidades

- **Autenticação com o SUAP:** acesso utilizando matrícula e senha institucionais.
- **Grade semanal de aulas:** visualização das disciplinas organizadas por dia da semana e horário.
- **Acompanhamento de frequência:** consulta do percentual de frequência e da quantidade de faltas de cada disciplina.
- **Cálculo do limite de faltas:** identificação da quantidade máxima de aulas que o estudante pode perder.
- **Faltas restantes:** visualização de quantas faltas ainda podem ser acumuladas sem ultrapassar o limite.
- **Classificação de risco:** identificação das disciplinas que exigem maior atenção em relação ao limite de faltas.
- **Gerenciamento de sessão:** encerramento da sessão diretamente pela interface.

## Como funciona

O Alume utiliza a API do SUAP como fonte das informações acadêmicas.

A partir dos dados retornados, o sistema calcula indicadores que facilitam o acompanhamento da frequência.

Entre eles, está o limite de faltas, calculado considerando o máximo de 25% da quantidade total de aulas da disciplina, com arredondamento para baixo:

\[
L = \lfloor Q \times 0{,}25 \rfloor
\]

Onde:

- \(L\): limite de faltas, expresso em aulas.
- \(Q\): quantidade total de aulas da disciplina.

A quantidade de faltas restantes é calculada pela diferença entre o limite e as faltas atuais:

\[
F_r = L - F
\]

Onde:

- \(F_r\): quantidade de faltas restantes.
- \(L\): limite de faltas.
- \(F\): quantidade atual de faltas.

O Alume também utiliza o percentual do limite de faltas consumido para identificar a situação de cada disciplina.

---

## Tecnologias

A arquitetura da primeira versão foi planejada com foco em simplicidade, separação de responsabilidades e integração com a API institucional.

| Tecnologia | Responsabilidade |
|---|---|
| Python | Linguagem principal do backend |
| FastAPI | Construção da API do Alume |
| Pydantic | Validação e estruturação dos dados |
| HTTPX | Comunicação HTTP com o SUAP |
| Redis | Armazenamento temporário de sessões e tokens |
| HTML | Estrutura da interface |
| Bootstrap | Estilização e responsividade |
| JavaScript | Interatividade do frontend |

### Arquitetura

A V1 utiliza uma arquitetura monolítica, com frontend e backend separados.

O backend concentra as responsabilidades de autenticação, gerenciamento de sessões, integração com o SUAP e processamento das informações acadêmicas.

O Redis é utilizado exclusivamente como armazenamento temporário de sessões e tokens de acesso. Não existe persistência própria de dados acadêmicos na primeira versão.

A autenticação utiliza dois elementos distintos:

- **JWT do Alume:** identifica a sessão do estudante nas requisições ao backend e é armazenado em cookie HttpOnly.
- **Access token do SUAP:** permanece sob responsabilidade do backend e é armazenado temporariamente no Redis.

O token de atualização (*refresh token*) do SUAP é descartado. A sessão do Alume expira junto com o token de acesso institucional.

Para conhecer os detalhes dessa estrutura, consulte a documentação de arquitetura.

---

# Documentação

A documentação do Alume está organizada em duas etapas principais: **Concepção** e **Construção**.

A primeira apresenta o problema, o domínio e os requisitos do sistema. A segunda descreve como a solução será estruturada e especifica o comportamento das funcionalidades.

## Roadmap de leitura

Se você está conhecendo o projeto agora, recomendamos seguir a ordem abaixo. Ela foi pensada para apresentar primeiro o contexto do Alume e, gradualmente, aprofundar os conceitos de negócio e as decisões técnicas.

### 01. Concepção

Esta etapa apresenta o que é o Alume, qual problema pretende resolver e quais comportamentos fazem parte da solução.

| Ordem | Documento | O que você encontrará |
|---|---|---|
| 01 | [Visão geral](docs/01%20-%20concep%C3%A7%C3%A3o/dma/visao-geral.md) | Contexto, motivação, objetivos, público-alvo, escopo e limitações do projeto. |
| 02 | [Glossário](docs/01%20-%20concep%C3%A7%C3%A3o/dma/glossario.md) | Definição dos principais termos e conceitos utilizados na documentação. |
| 03 | [Modelo de domínio](docs/01%20-%20concep%C3%A7%C3%A3o/dma/modelo-de-dominio.md) | Conceitos do domínio acadêmico representados pelo Alume e seus relacionamentos. |
| 04 | [Requisitos funcionais](docs/01%20-%20concep%C3%A7%C3%A3o/clf/requisitos-funcionais.md) | Comportamentos e funcionalidades que o sistema deve oferecer. |
| 05 | [Requisitos não funcionais](docs/01%20-%20concep%C3%A7%C3%A3o/clf/requisitos-nao-funcionais.md) | Requisitos relacionados à qualidade, segurança, disponibilidade e manutenção. |
| 06 | [Lista de funcionalidades](docs/01%20-%20concep%C3%A7%C3%A3o/clf/lista-de-funcionalidades.md) | Relação das funcionalidades identificadas e sua distribuição pelas telas da aplicação. |

### 02. Construção

Esta etapa apresenta como o Alume será desenvolvido, incluindo sua arquitetura técnica e as especificações de comportamento de cada funcionalidade.

| Ordem | Documento | O que você encontrará |
|---|---|---|
| 07 | [Arquitetura](docs/02%20-%20constru%C3%A7%C3%A3o/cpf/arquitetura.md) | Organização dos componentes, tecnologias, integração com o SUAP, autenticação, sessões, segurança, infraestrutura e decisões arquiteturais. |
| 08 | [F01 — Autenticar usuário](docs/02%20-%20constru%C3%A7%C3%A3o/ddf/F01-autenticar-usuario/especificacao.md) | Fluxo de autenticação utilizando as credenciais institucionais. |
| 09 | [F02 — Consultar disciplinas por dia da semana](docs/02%20-%20constru%C3%A7%C3%A3o/ddf/F02-consultar-disciplinas-por-dia-da-semana/especificacao.md) | Consulta e organização da grade semanal de aulas. |
| 10 | [F03 — Consultar frequência por disciplina](docs/02%20-%20constru%C3%A7%C3%A3o/ddf/F03-consultar-frequencia-por-disciplina/especificacao.md) | Consulta dos dados de frequência e cálculo dos indicadores. |
| 11 | [F04 — Consultar disciplinas por risco de reprovação por falta](docs/02%20-%20constru%C3%A7%C3%A3o/ddf/F04-consultar-disciplinas-por-risco-de-reprovacao-por-falta/especificacao.md) | Classificação e apresentação das disciplinas conforme o risco de reprovação por falta. |
| 12 | [F05 — Encerrar sessão](docs/02%20-%20constru%C3%A7%C3%A3o/ddf/F05-encerrar-sessao/especificacao.md) | Fluxo de encerramento e invalidação da sessão no Alume. |

> **Sugestão:** se você deseja compreender o projeto antes de analisar o código, leia primeiro toda a etapa de Concepção. Depois, avance para a Arquitetura e finalize com as especificações individuais das funcionalidades.

---

## Escopo da primeira versão

A primeira versão do Alume prioriza o acompanhamento de frequência e a visualização da grade semanal de aulas.

Algumas decisões importantes delimitam seu funcionamento:

- O público-alvo inicial é composto por estudantes do Campus Aracaju do IFS.
- As informações acadêmicas são consultadas diretamente na API do SUAP.
- O Alume não armazena dados acadêmicos dos estudantes em banco de dados próprio.
- O Redis é utilizado exclusivamente para armazenamento temporário de sessões e tokens de acesso.
- O sistema depende da disponibilidade da API do SUAP e do Redis para oferecer suas funcionalidades autenticadas.
- O encerramento da sessão no Alume não revoga o token emitido pelo SUAP.

Essas limitações fazem parte das decisões de escopo e arquitetura da primeira versão.

## Organização da documentação

```text
docs/
├── 01 - concepção/
│   ├── clf/
│   │   ├── lista-de-funcionalidades.md
│   │   ├── requisitos-funcionais.md
│   │   └── requisitos-nao-funcionais.md
│   │
│   └── dma/
│       ├── glossario.md
│       ├── modelo-de-dominio.md
│       └── visao-geral.md
│
└── 02 - construção/
    ├── cpf/
    │   └── arquitetura.md
    │
    └── ddf/
        ├── F01-autenticar-usuario/
        │   └── especificacao.md
        ├── F02-consultar-disciplinas-por-dia-da-semana/
        │   └── especificacao.md
        ├── F03-consultar-frequencia-por-disciplina/
        │   └── especificacao.md
        ├── F04-consultar-disciplinas-por-risco-de-reprovacao-por-falta/
        │   └── especificacao.md
        └── F05-encerrar-sessao/
            └── especificacao.md
```

---

## Status do projeto

O Alume encontra-se na etapa de desenvolvimento, tendo sua concepção, requisitos, modelagem de domínio e arquitetura documentados.

O desenvolvimento está organizado para seguir as funcionalidades e decisões estabelecidas na documentação, mantendo a primeira versão dentro do escopo definido.

## Licença

A licença do projeto será definida posteriormente.# alume
