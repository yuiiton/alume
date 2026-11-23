# Arquitetura do Alume

## 1. Visão geral

O Alume é uma aplicação web voltada para estudantes do Instituto Federal de Sergipe, com foco em facilitar o acompanhamento e a gestão de faltas acadêmicas. O sistema funciona como uma camada de consulta, processamento e apresentação de informações acadêmicas obtidas do SUAP.

A arquitetura proposta deve priorizar:

- simplicidade operacional;
- integração segura com a API do SUAP;
- baixo acoplamento entre autenticação, integração e apresentação;
- experiência responsiva em navegadores e dispositivos móveis;
- uso do Redis como banco de dados próprio temporário, destinado a sessões e tokens, com possibilidade de evolução para cache acadêmico;
- autenticação por JWT próprio do Alume, guardado em cookie HttpOnly;
- organização em monólito, com separação entre frontend e backend.

---

## 2. Objetivo da arquitetura

A arquitetura do Alume deve permitir que o estudante:

- autentique-se utilizando as credenciais do SUAP;
- consulte as disciplinas do período letivo atual;
- visualize a grade semanal de aulas;
- consulte a frequência atual e os dados de cada disciplina;
- identifique o limite de faltas;
- consulte a quantidade de faltas restantes;
- identifique o risco de reprovação por falta;
- encerre sua sessão.

A solução deve tratar o SUAP como sistema externo de origem dos dados acadêmicos e o Alume como camada de autenticação, processamento e apresentação dessas informações.

---

## 3. Contexto do sistema

### 3.1. Sistema de origem

A API do SUAP fornece os dados acadêmicos reais do estudante, incluindo:

- autenticação via tokens JWT;
- disciplinas do período letivo, com dados de frequência e quantidade de faltas;
- diários do período letivo, com os dias da semana e os horários das aulas.

Os endpoints de interesse são:

| Endpoint | Finalidade |
|---|---|
| `POST /api/token/pair` | Autenticação inicial com `username` e `password` |
| `GET /api/ensino/disciplinas/{semestre}/` | Fonte principal dos dados de frequência: total de aulas, faltas e percentual de frequência |
| `GET /api/ensino/diarios/{semestre}/` | Fonte dos dias da semana e horários das aulas, usados para montar a grade semanal |

O SUAP também oferece o endpoint `POST /api/token/refresh`, que **não é utilizado** pelo Alume (ver seção 6.3).

### 3.2. Papel do Alume

O Alume não deve atuar como repositório oficial dos dados do estudante. Sua função é:

1. autenticar o estudante no SUAP;
2. gerenciar a sessão de acesso ao Alume;
3. consultar os dados necessários no SUAP;
4. transformar os dados em indicadores úteis;
5. apresentar as informações de forma clara e acessível.

---

## 4. Requisitos arquiteturais

### 4.1. Requisitos funcionais relevantes

- autenticação do estudante com as credenciais do SUAP;
- consulta das disciplinas do período letivo atual;
- leitura dos dias da semana e horários das aulas para a grade semanal;
- cálculo do limite de faltas;
- cálculo das faltas restantes;
- apresentação da situação de frequência, incluindo o percentual de frequência;
- classificação do risco de reprovação por falta;
- encerramento de sessão;
- apresentação em interface responsiva.

### 4.2. Requisitos não funcionais relevantes

- segurança na autenticação e no tratamento de tokens;
- gerenciamento de sessão autenticada e expiração;
- compatibilidade com navegadores modernos;
- suporte a dispositivos móveis e uso como PWA;
- desempenho adequado para consultas em tempo real;
- manutenibilidade por meio da separação de responsabilidades;
- disponibilidade do armazenamento de sessões no Redis.

---

## 5. Arquitetura de alto nível

A arquitetura do projeto é um monólito organizado em duas grandes áreas: frontend e backend.

O backend será implementado com FastAPI, Pydantic e HTTPX para validação de dados e comunicação HTTP. O frontend será iniciado com HTML, Bootstrap e JavaScript, permitindo uma possível evolução para React no futuro.

O backend será responsável pela autenticação, gerenciamento das sessões, integração com o SUAP e processamento dos dados. O Redis será utilizado como armazenamento temporário compartilhado entre as instâncias do backend.

### 5.1. Organização arquitetural

#### Frontend

Responsável pela interface do usuário e pela apresentação dos dados.

Principais responsabilidades:

- tela de login;
- tela inicial, com a grade semanal de aulas;
- dashboard, com a frequência, o limite de faltas, as faltas restantes e o risco de cada disciplina;
- botão de encerrar sessão nas telas autenticadas;
- componentes responsivos;
- redirecionamento para a tela de login quando o backend indicar sessão inválida ou expirada;
- consumo do backend por requisições HTTP, com envio automático do cookie da sessão (`credentials: 'include'`).

O frontend **não tem acesso ao JWT do Alume**, que fica em um cookie HttpOnly gerenciado pelo navegador e pelo backend.

Tecnologias iniciais:

- HTML;
- Bootstrap;
- JavaScript.

Organização por feature:

- `features/auth/`;
- `features/home/`;
- `features/dashboard/`;
- `shared/`.

#### Backend

Responsável por receber as requisições do cliente, validar a autenticação, orquestrar os fluxos e coordenar a integração com o SUAP.

Tecnologias:

- FastAPI;
- Pydantic;
- HTTPX.

Principais responsabilidades:

- autenticação e gerenciamento de sessão (ver Serviço de autenticação);
- determinação do período letivo atual;
- consulta de disciplinas e diários no SUAP;
- validação das respostas externas;
- composição das informações para a interface;
- cálculo de indicadores derivados;
- exposição de endpoints organizados por feature.

#### Serviço de autenticação

Responsável por receber e utilizar temporariamente as credenciais do usuário no processo de autenticação com o SUAP, sem armazená-las.

Principais responsabilidades:

- recebimento de `username` e `password` do cliente;
- chamada ao SUAP para autenticação inicial;
- obtenção do `access token` do SUAP (o `refresh token` retornado é descartado);
- geração do identificador da sessão (`sid`);
- armazenamento do `access token` do SUAP no Redis, com TTL;
- emissão do JWT próprio do Alume e envio em cookie HttpOnly;
- validação da sessão;
- encerramento da sessão (remoção no Redis e do cookie).

**Obs: Todo `username` de um estudante cadastrado no SUAP é o seu número de matrícula.**

#### Integrador SUAP

Responsável por encapsular a comunicação com a API externa.

Principais responsabilidades:

- abstração da URL e dos endpoints do SUAP;
- tratamento de erros HTTP;
- tratamento da paginação das rotas de disciplinas e de diários;
- serialização e desserialização dos payloads;
- normalização dos dados de resposta;
- isolamento do payload bruto do SUAP por meio de DTOs e mappers.

#### Serviço de cálculo de faltas

Responsável por transformar os dados do SUAP em indicadores úteis para o estudante.

Cálculos:

- limite de faltas = ⌊total de aulas × 0,25⌋, ou seja, 25% do total de aulas arredondado para baixo, em que o total de aulas é o campo `ch_total_aula` do SUAP;
- faltas restantes = limite de faltas − faltas atuais;
- percentual do limite consumido = faltas atuais ÷ limite de faltas × 100;
- estado da disciplina (risco de reprovação por falta): crítico, em risco ou estável, de acordo com o percentual do limite consumido.

O percentual de frequência exibido é o valor `frequencia` informado pelo SUAP, sem recálculo pelo Alume.

As faixas do percentual do limite consumido que definem cada estado ainda precisam ser definidas antes da implementação dessa classificação (ver seção 13.3).

#### Infraestrutura transversal

Inclui as responsabilidades técnicas compartilhadas por toda a aplicação.

Exemplos:

- configuração de ambiente e variáveis;
- tratamento de exceções e erros globais;
- integração com o Redis;
- dependências de autenticação do FastAPI (obtenção da sessão atual a partir do cookie);
- configuração de CORS (origens autorizadas e envio de credenciais);
- padrões de timeout e retry para chamadas externas.

#### Testes

A estrutura de testes acompanha as features do backend, mantendo o mesmo padrão organizacional da aplicação.

- `auth/`;
- `disciplines/`;
- `integrations/`.

O objetivo é validar os fluxos principais, os cálculos de domínio e os adaptadores de integração com o SUAP.

---

## 6. Fluxos principais

### 6.1. Fluxo de autenticação

O Alume utilizará dois tipos de token, com responsabilidades distintas:

**JWT do Alume**

Emitido pelo backend após a autenticação bem-sucedida no SUAP. É utilizado para autenticar as requisições do navegador ao Alume.

O token conterá inicialmente os seguintes claims:

```json
{
  "sid": "abc123",
  "exp": 1794945600
}
```

- `sid`: identificador único da sessão;
- `exp`: instante de expiração do JWT, igual ao instante de expiração do `access token` do SUAP.

O `sid` será utilizado para localizar a sessão correspondente no Redis.

O JWT do Alume será entregue ao navegador **exclusivamente em um cookie HttpOnly**, com os atributos `HttpOnly`, `Secure`, `SameSite=Lax` e `Path=/`, e com expiração igual à do JWT. O JavaScript do frontend não consegue ler o cookie.

Como o cookie é mantido pelo navegador, recarregar a página ou fechar e reabrir a aplicação **não** encerra a sessão: o estudante permanece autenticado até a expiração do token ou até o encerramento da sessão (seção 6.4).

**Token do SUAP**

É obtido durante a autenticação e permanece exclusivamente sob responsabilidade do backend. O `access token` será utilizado nas consultas à API do SUAP. O `refresh token` retornado pelo SUAP não é utilizado nem armazenado.

Fluxo esperado:

1. o estudante envia `username` e `password` ao backend (`POST /auth/login`);
2. o backend encaminha as credenciais ao endpoint `/api/token/pair` do SUAP;
3. o SUAP valida as credenciais e retorna o `access token` e o `refresh token`;
4. o backend descarta o `refresh token`;
5. o backend gera um identificador aleatório para a sessão (`sid`);
6. o backend lê a data de expiração do `access token` do SUAP e armazena o token no Redis, associado ao `sid`, com TTL igual ao tempo restante até essa expiração;
7. o backend gera o JWT do Alume, contendo o `sid` e o `exp`;
8. o backend responde ao frontend enviando o JWT em um cookie HttpOnly;
9. o frontend redireciona o estudante para a tela inicial, e o navegador passa a enviar o cookie automaticamente nas requisições ao backend.

O backend somente deve emitir o JWT do Alume após confirmar que a sessão foi armazenada corretamente no Redis.

### 6.2. Fluxo de validação de sessão

As requisições autenticadas enviam o cookie com o JWT do Alume automaticamente, desde que o frontend use `credentials: 'include'`.

Fluxo esperado:

1. o frontend envia uma requisição ao backend, acompanhada do cookie com o JWT do Alume;
2. o backend valida a assinatura e a expiração do JWT;
3. o backend extrai o `sid` dos claims;
4. o backend consulta o Redis utilizando o identificador da sessão;
5. caso a sessão exista e esteja válida, o backend recupera o `access token` do SUAP;
6. o backend executa a operação solicitada;
7. o backend retorna a resposta ao frontend.

Caso o JWT seja inválido, esteja expirado ou a sessão não exista mais no Redis, a requisição deverá ser rejeitada com resposta de não autorizado, e o frontend redireciona o estudante para a tela de login.

A existência de um JWT válido, isoladamente, não garante que a sessão continue ativa. A consulta ao Redis permite invalidar sessões imediatamente, inclusive antes da expiração do JWT.

### 6.3. Fluxo de expiração da sessão

O Alume **não utiliza o refresh token do SUAP**. O JWT do Alume, o `access token` do SUAP e a sessão no Redis possuem o mesmo prazo de expiração: quando um deles expira, os demais também expiram.

Quando o prazo de expiração é atingido:

1. a sessão é removida automaticamente do Redis (TTL);
2. o navegador deixa de enviar o cookie expirado ou o backend rejeita o JWT expirado;
3. o próximo acesso a uma funcionalidade autenticada é rejeitado e o estudante é redirecionado para a tela de login;
4. o estudante deve se autenticar novamente.

Se o SUAP rejeitar o `access token` armazenado antes do prazo esperado, o backend remove a sessão do Redis e responde como sessão inválida, exigindo novo login.

### 6.4. Fluxo de encerramento da sessão

O encerramento da sessão é uma funcionalidade própria (F05), acionada por um botão na interface, que chama um endpoint de logout (`POST /auth/logout`).

Fluxo esperado:

1. o frontend envia uma requisição autenticada para encerrar a sessão;
2. o backend valida o JWT e extrai o `sid`;
3. o backend remove do Redis a chave correspondente à sessão;
4. o backend remove o cookie do JWT do navegador e confirma o encerramento da sessão;
5. o frontend redireciona o estudante para a tela de login.

Após a exclusão da sessão no Redis, novas requisições utilizando o JWT correspondente deverão ser rejeitadas, mesmo que o token ainda não tenha atingido sua expiração.

**Limitação:** o Alume não consegue invalidar o token criado pelo SUAP. Ao encerrar a sessão, o Alume apenas deixa de armazenar o `access token` do SUAP, que continua válido no SUAP até a sua própria expiração.

### 6.5. Determinação do período letivo

O período letivo atual é determinado pelo backend a partir da data da requisição:

1. o backend obtém a data atual (por exemplo, `2027/03/03`) e guarda o ano e o mês;
2. se o mês for anterior a agosto, o período é o **1**; se for agosto ou posterior, o período é o **2**.

O resultado é utilizado em dois formatos, conforme a exigência de cada rota do SUAP:

| Rota | Formato do parâmetro `{semestre}` | Requisição em 03/03/2027 | Requisição em 15/09/2027 |
|---|---|---|---|
| `GET /api/ensino/disciplinas/{semestre}/` | texto no formato `AAAA.P` | `2027.1` | `2027.2` |
| `GET /api/ensino/diarios/{semestre}/` | inteiro com o número do período | `1` | `2` |

A diferença de formato entre as rotas é uma característica da API do SUAP, que o Alume apenas deve respeitar.

### 6.6. Fluxo de consulta de disciplinas, horários e frequência

Cada rota do SUAP tem um papel específico:

- `GET /api/ensino/disciplinas/{semestre}/` é a fonte principal dos dados de frequência. Dela são utilizados `ch_total_aula` (quantidade total de aulas prevista para o período letivo), `qtd_faltas` (quantidade atual de faltas) e `frequencia` (percentual de frequência atual). O campo `ch_total_relogio` é apenas a carga horária da disciplina em horas ao longo do período, de caráter informativo.
- `GET /api/ensino/diarios/{semestre}/` é utilizada apenas para obter os dias da semana e os horários das disciplinas do período letivo atual, para montar a grade semanal. Os campos de frequência que essa rota devolve dentro de `disciplina` **não são utilizados**.

Quando for necessário relacionar dados das duas rotas, a ligação é feita pela `sigla` da disciplina.

Fluxo da tela inicial (grade semanal):

1. o backend valida a sessão (seção 6.2);
2. o backend determina o período letivo atual (seção 6.5);
3. o backend consulta a rota de diários, tratando a paginação;
4. o backend normaliza os dados e organiza as disciplinas pelos dias da semana e horários;
5. o backend retorna os dados organizados ao frontend;
6. o frontend apresenta a grade semanal ao estudante.

Fluxo do dashboard (frequência e risco):

1. o backend valida a sessão (seção 6.2);
2. o backend determina o período letivo atual (seção 6.5);
3. o backend consulta a rota de disciplinas, tratando a paginação;
4. o backend normaliza os dados recebidos;
5. o serviço de cálculo produz o limite de faltas, as faltas restantes, o percentual do limite consumido e o estado de risco;
6. o backend retorna os dados organizados ao frontend;
7. o frontend apresenta a situação acadêmica ao estudante.

---

## 7. Integração com o SUAP

### 7.1. Endpoints de integração

A integração com o SUAP deve ser encapsulada no módulo de integrador externo.

| Endpoint | Finalidade | Observações |
|---|---|---|
| `POST /api/token/pair` | Autenticação inicial | Recebe `username` e `password` |
| `GET /api/ensino/disciplinas/{semestre}/` | Dados de frequência das disciplinas | `{semestre}` no formato `AAAA.P` (ex.: `2027.1`); paginado |
| `GET /api/ensino/diarios/{semestre}/` | Dias da semana e horários das disciplinas | `{semestre}` como inteiro com o número do período (ex.: `1`); paginado |

O endpoint `POST /api/token/refresh` não é utilizado na V1.

### 7.2. Requests e responses das rotas

Os exemplos abaixo usam **dados fictícios**, mantendo a estrutura real das respostas do SUAP. A URL base é `https://suap.ifs.edu.br`. Os valores de tokens são representados por marcadores.

#### 7.2.1. `POST /api/token/pair`

Autentica o estudante e retorna os tokens do SUAP.

**Request:**

```json
{
  "username": "202700000001",
  "password": "senha-do-estudante"
}
```

**Response:**

```json
{
  "username": "202700000001",
  "refresh": "<refresh_token_ficticio>",
  "access": "<access_token_ficticio>"
}
```

Observações:

- `username` é o número de matrícula do estudante.
- `access` é um JWT cujo claim `exp` indica o instante de expiração. O Alume utiliza essa data para definir o `exp` do JWT do Alume e o TTL da sessão no Redis.
- `refresh` é descartado pelo Alume e não é armazenado.

#### 7.2.2. `GET /api/ensino/disciplinas/{semestre}/`

Retorna as disciplinas do período letivo com os dados de frequência. É a **fonte principal** de total de aulas, faltas e frequência.

**Request:**

```bash
curl -X 'GET' \
  'https://suap.ifs.edu.br/api/ensino/disciplinas/2027.1/?page=1' \
  -H 'accept: */*' \
  -H 'Authorization: Bearer <access_token_do_suap>'
```

O parâmetro `{semestre}` segue o formato `AAAA.P` (ano e número do período). O parâmetro de query `page` indica o número da página.

**Response:**

```json
{
  "results": [
    {
      "id": 100001,
      "descricao": "DISCIPLINA EXEMPLO A",
      "sigla": "GRAD.0001",
      "situacao": {
        "rotulo": "Aprovado",
        "status": "success"
      },
      "ch_total_aula": 72,
      "ch_total_relogio": 60,
      "ch_cumprida_aula": 36,
      "qtd_faltas": 4,
      "qtd_avaliacoes": 1,
      "frequencia": 89,
      "notas": [
        { "tipo": "N1", "nota": "7,0" }
      ],
      "medias": [
        { "tipo": "MD", "nota": null },
        { "tipo": "MFD", "nota": null },
        { "tipo": "NAF", "nota": null }
      ]
    },
    {
      "id": 100002,
      "descricao": "DISCIPLINA EXEMPLO B",
      "sigla": "GRAD.0002",
      "situacao": {
        "rotulo": "Aprovado",
        "status": "success"
      },
      "ch_total_aula": 54,
      "ch_total_relogio": 45,
      "ch_cumprida_aula": 30,
      "qtd_faltas": 5,
      "qtd_avaliacoes": 1,
      "frequencia": 83,
      "notas": [
        { "tipo": "N1", "nota": "8,5" }
      ],
      "medias": [
        { "tipo": "MD", "nota": null },
        { "tipo": "MFD", "nota": null },
        { "tipo": "NAF", "nota": null }
      ]
    }
  ],
  "count": 2,
  "next": null,
  "previous": null
}
```

Campos utilizados pelo Alume:

| Campo | Significado | Uso no Alume |
|---|---|---|
| `descricao` | Nome da disciplina | Exibição |
| `sigla` | Código da disciplina | Exibição e ligação com os dados da rota de diários |
| `ch_total_aula` | Quantidade total de aulas prevista para o período letivo | Valor de `Q` no cálculo do limite de faltas |
| `qtd_faltas` | Quantidade atual de faltas do estudante | Valor de `F` nos cálculos |
| `frequencia` | Percentual de frequência atual do estudante | Exibição, sem recálculo |
| `ch_total_relogio` | Carga horária da disciplina em horas ao longo do período | Informativo |

Os demais campos (`id`, `situacao`, `ch_cumprida_aula`, `qtd_avaliacoes`, `notas` e `medias`) não são utilizados na V1.

Exemplo de cálculo com a disciplina B: `L = ⌊54 × 0,25⌋ = ⌊13,5⌋ = 13`; `F_r = 13 − 5 = 8`; `P_c = 5 ÷ 13 × 100 ≈ 38%`.

#### 7.2.3. `GET /api/ensino/diarios/{semestre}/`

Retorna os diários do período letivo. É utilizada **apenas** para obter os dias da semana e os horários das aulas.

**Request:**

```bash
curl -X 'GET' \
  'https://suap.ifs.edu.br/api/ensino/diarios/1/?page=1' \
  -H 'accept: */*' \
  -H 'Authorization: Bearer <access_token_do_suap>'
```

O parâmetro `{semestre}` é o inteiro com o número do período (`1` ou `2`). O parâmetro de query `page` indica o número da página.

**Response:**

```json
{
  "results": [
    {
      "id": 200001,
      "disciplina": {
        "id": 300001,
        "descricao": "DISCIPLINA EXEMPLO A",
        "sigla": "GRAD.0001",
        "situacao": null,
        "ch_total_aula": 0,
        "ch_total_relogio": 0,
        "ch_cumprida_aula": 0,
        "qtd_faltas": 0,
        "qtd_avaliacoes": 0,
        "frequencia": 0,
        "notas": [],
        "medias": []
      },
      "professores": [
        {
          "id": 1,
          "nome": "Professor Exemplo A",
          "matricula": "0000001",
          "email": "professor.a@exemplo.edu.br"
        }
      ],
      "horarios": [
        { "dia": "Terça", "horario": "18:40 - 19:30" },
        { "dia": "Terça", "horario": "19:30 - 20:20" }
      ],
      "local": {
        "id": 10,
        "sala": "Sala Exemplo 01 - Bloco X (AJU)"
      },
      "laboratorio": null,
      "ambiente_virtual": ""
    },
    {
      "id": 200002,
      "disciplina": {
        "id": 300002,
        "descricao": "DISCIPLINA EXEMPLO B",
        "sigla": "GRAD.0002",
        "situacao": null,
        "ch_total_aula": 0,
        "ch_total_relogio": 0,
        "ch_cumprida_aula": 0,
        "qtd_faltas": 0,
        "qtd_avaliacoes": 0,
        "frequencia": 0,
        "notas": [],
        "medias": []
      },
      "professores": [
        {
          "id": 2,
          "nome": "Professor Exemplo B",
          "matricula": "0000002",
          "email": "professor.b@exemplo.edu.br"
        }
      ],
      "horarios": [
        { "dia": "Terça", "horario": "15:50 - 16:40" },
        { "dia": "Quinta", "horario": "18:40 - 19:30" }
      ],
      "local": {
        "id": 11,
        "sala": "Sala Exemplo 02 - Bloco X (AJU)"
      },
      "laboratorio": null,
      "ambiente_virtual": ""
    }
  ],
  "count": 2,
  "next": null,
  "previous": null
}
```

Campos utilizados pelo Alume:

| Campo | Significado | Uso no Alume |
|---|---|---|
| `disciplina.descricao` e `disciplina.sigla` | Nome e código da disciplina | Exibição na grade semanal e ligação com os dados da rota de disciplinas (pela `sigla`) |
| `horarios[].dia` | Dia da semana (nome em português, como `Segunda` ou `Terça`) | Organização da grade semanal |
| `horarios[].horario` | Faixa de horário da aula | Organização da grade semanal |
| `local.sala` | Sala da aula | Exibição, quando houver |
| `professores` | Docentes da disciplina | Exibição, quando houver |

Os campos de frequência dentro de `disciplina` (`ch_total_aula`, `qtd_faltas`, `frequencia` e demais) **não são utilizados**, pois essa rota não é a fonte confiável desses dados.

#### 7.2.4. Paginação

As rotas de disciplinas e de diários são paginadas. A resposta traz `count` (total de itens), `next` e `previous` (referências das páginas vizinhas). O integrador deve percorrer todas as páginas até que `next` seja nulo.

### 7.3. Modelo de uso dos tokens

O token retornado pela API do SUAP deve ser tratado com cuidado:

- o `access token` será utilizado nas requisições de consulta;
- o `refresh token` não será utilizado nem armazenado;
- o `access token` será armazenado temporariamente no Redis;
- o token permanecerá associado ao identificador da sessão;
- o frontend não receberá os tokens do SUAP;
- a expiração do `access token` do SUAP define a expiração da sessão no Alume (JWT e Redis).

### 7.4. Normalização dos dados

O SUAP devolve dados em formatos específicos, como listas de `results` e objetos aninhados, por exemplo:

- disciplina;
- professores;
- horários;
- local;
- frequência.

O Alume deverá normalizar esses dados em modelos internos, por exemplo:

- `Disciplina`;
- `HorarioAula`;
- `DiarioDisciplina`;
- `IndicadorFalta`;
- `SessaoUsuario`.

Esse processo reduz o acoplamento com o formato da API externa e facilita a evolução do sistema.

---

## 8. Modelo de dados

### 8.1. Dados externos

Os dados obtidos do SUAP devem ser considerados dados externos de origem e não serão armazenados pelo Alume na V1, nem mesmo no Redis. As informações acadêmicas serão consultadas diretamente no SUAP conforme a necessidade das funcionalidades.

Exemplos:

- código da disciplina;
- nome da disciplina;
- professores;
- dias e horários das aulas;
- sala ou local;
- frequência atual;
- quantidade total de aulas.

### 8.2. Dados calculados

Os dados calculados pelo Alume serão derivados das informações do SUAP e representados internamente em objetos de domínio:

- limite de faltas;
- faltas restantes;
- percentual do limite consumido;
- situação da disciplina (risco de reprovação por falta).

Esses dados não precisam ser armazenados na V1, pois podem ser calculados durante o processamento das consultas.

### 8.3. Dados temporários

O Redis é o único banco de dados próprio do Alume na V1. Sua natureza é temporária, diferentemente de bancos permanentes como o PostgreSQL. Ele armazena apenas os dados de autenticação, associados ao identificador da sessão.

A estrutura lógica de armazenamento será:

**Chave:**

```text
alume:session:{sid}
```

**Valor:**

```json
{
  "suap_access_token": "TOKEN_DO_SUAP"
}
```

O `refresh token` do SUAP não é armazenado.

O TTL da chave é igual ao tempo restante até a expiração do `access token` do SUAP, que é também o instante de expiração do JWT do Alume. Assim, os três prazos coincidem, e a sessão é removida do Redis automaticamente quando o token expira.

Outros metadados poderão ser adicionados conforme a necessidade de gerenciamento da sessão, sem armazenar informações acadêmicas.

O `sid` não precisa ser armazenado novamente dentro do valor, pois já faz parte da chave utilizada para localizar a sessão.

---

## 9. Segurança

A segurança é um ponto central da arquitetura.

### 9.1. Regras

- não armazenar a senha do SUAP em nenhum banco de dados do Alume;
- não persistir a senha do estudante no Alume;
- armazenar o `access token` do SUAP somente no Redis, sob controle de expiração (TTL);
- não armazenar o `refresh token` do SUAP;
- não retornar os tokens do SUAP ao frontend;
- utilizar JWT assinado pelo backend para autenticar as requisições ao Alume;
- entregar o JWT do Alume somente em cookie com os atributos `HttpOnly`, `Secure` e `SameSite`;
- validar assinatura e expiração do JWT em todas as requisições protegidas;
- verificar a existência da sessão no Redis antes de permitir o acesso aos recursos autenticados;
- invalidar a sessão (remover do Redis e do cookie) no logout e quando o SUAP rejeitar o `access token` armazenado;
- validar o cabeçalho `Origin` nas requisições que alteram estado (login e logout), como proteção contra CSRF;
- utilizar HTTPS na comunicação entre o navegador e o backend;
- exigir autenticação e TLS na comunicação com o Redis;
- nunca registrar tokens, senhas ou credenciais nos logs da aplicação.

### 9.2. Autenticação por cookie HttpOnly

O JWT do Alume será armazenado **somente em cookie HttpOnly**. Ele não deve ser guardado em `localStorage`, `sessionStorage` nem em variáveis acessíveis ao JavaScript do frontend.

Consequências dessa decisão:

- scripts executados na página não conseguem ler o JWT, o que reduz o risco de roubo do token em caso de vulnerabilidade XSS;
- o navegador envia o cookie automaticamente, então o frontend deve usar `credentials: 'include'` nas requisições ao backend;
- como o envio é automático, é necessário proteger as requisições que alteram estado contra CSRF (atributo `SameSite` e validação do cabeçalho `Origin`);
- recarregar a página não encerra a sessão, pois o cookie permanece no navegador até a expiração ou o logout;
- o frontend não sabe se há sessão ativa até receber a resposta do backend, e por isso trata respostas de não autorizado redirecionando para a tela de login;
- para que o cookie seja enviado de forma confiável, frontend e backend devem estar no mesmo site (ver seção 11.7).

Apesar dessa estratégia, o frontend ainda deverá ter medidas para reduzir vulnerabilidades XSS, como evitar a execução de scripts não confiáveis e utilizar políticas de segurança de conteúdo (CSP) adequadas, pois um script malicioso ainda conseguiria disparar requisições em nome do usuário enquanto a sessão estiver ativa.

### 9.3. Limitações

- O Alume não consegue invalidar o token emitido pelo SUAP. No logout, o token deixa de ser armazenado pelo Alume, mas continua válido no SUAP até a sua própria expiração.

---

## 10. Estrutura de módulos sugerida

A organização do projeto deve seguir o modelo monolítico com separação clara entre frontend e backend.

```text
alume/
│
├── backend/
│   ├── app/
│   │   ├── auth/                       # Autenticação e gerenciamento de sessão
│   │   │   ├── router.py               # Endpoints de login e logout
│   │   │   ├── schemas.py              # Contratos de entrada e saída
│   │   │   ├── service.py              # Orquestração dos fluxos de autenticação
│   │   │   ├── jwt.py                  # Emissão e validação do JWT do Alume
│   │   │   └── session.py              # Operações de gerenciamento de sessão
│   │   │
│   │   ├── disciplines/                # Disciplinas, horários e frequência
│   │   │   ├── router.py               # Endpoints de consulta
│   │   │   ├── schemas.py              # Modelos de saída do Alume
│   │   │   ├── service.py              # Orquestração das consultas
│   │   │   ├── period.py               # Determinação do período letivo atual a partir da data
│   │   │   └── calculator.py           # Cálculo de limite, faltas restantes e risco
│   │   │
│   │   ├── integrations/               # Comunicação com sistemas externos
│   │   │   └── suap/
│   │   │       ├── client.py           # Cliente HTTP assíncrono com timeout, retry e paginação
│   │   │       ├── schemas.py          # DTOs dos payloads do SUAP
│   │   │       └── mapper.py           # Conversão dos DTOs para modelos internos
│   │   │
│   │   ├── core/                       # Infraestrutura transversal
│   │   │   ├── config.py               # Configurações e variáveis de ambiente
│   │   │   ├── redis.py                # Conexão e configuração do Redis
│   │   │   ├── security.py             # Dependências do FastAPI para autenticação (sessão atual via cookie)
│   │   │   └── exceptions.py           # Exceções e handlers globais
│   │   │
│   │   └── main.py                     # Inicialização do FastAPI, CORS e registro dos routers
│   │
│   ├── tests/
│   │   ├── conftest.py                 # Fixtures globais e mocks
│   │   ├── auth/
│   │   │   ├── test_auth_service.py
│   │   │   ├── test_jwt.py
│   │   │   └── test_session.py
│   │   ├── disciplines/
│   │   │   ├── test_calculator.py
│   │   │   ├── test_period.py
│   │   │   └── test_router.py
│   │   └── integrations/
│   │       └── test_suap_mapper.py
│   │
│   ├── pyproject.toml
│   └── .env.example
│
├── frontend/
│   ├── src/
│   │   ├── features/
│   │   │   ├── auth/                   # Tela de login
│   │   │   │   ├── login.html
│   │   │   │   ├── login.js
│   │   │   │   └── auth-api.js         # Chamadas de login e logout
│   │   │   │
│   │   │   ├── home/                   # Tela inicial (grade semanal de aulas)
│   │   │   │   ├── home.html
│   │   │   │   ├── home.js
│   │   │   │   └── schedule-api.js
│   │   │   │
│   │   │   └── dashboard/              # Dashboard (frequência, faltas e risco)
│   │   │       ├── dashboard.html
│   │   │       ├── dashboard.js
│   │   │       ├── discipline-card.js
│   │   │       └── disciplines-api.js
│   │   │
│   │   └── shared/
│   │       ├── http-client.js          # Cliente HTTP com credentials: 'include' e tratamento de 401
│   │       ├── auth-state.js           # Redirecionamento por estado de autenticação (sem acesso ao JWT)
│   │       ├── navbar.js               # Navegação e botão de encerrar sessão
│   │       └── styles.css
│   │
│   ├── public/
│   │   ├── icons/
│   │   ├── manifest.json
│   │   └── sw.js
│   │
│   └── index.html
│
├── docs/
├── .gitignore
└── README.md
```

### 10.1. Justificativa

Essa organização separa claramente:

- regras de negócio por funcionalidade;
- comunicação com sistemas externos;
- apresentação do frontend;
- autenticação e gerenciamento de sessão;
- infraestrutura compartilhada;
- testes das funcionalidades e integrações.

O padrão *folder by feature* é adequado ao projeto porque o sistema é pequeno, mas já possui responsabilidades bem definidas: autenticação, disciplinas e frequência.

A separação entre `jwt.py`, `session.py` e `redis.py` também evita concentrar responsabilidades distintas em um único módulo:

- `jwt.py`: emissão e validação do JWT do Alume;
- `session.py`: criação, consulta e encerramento das sessões;
- `redis.py`: conexão e configuração do Redis.

O módulo `core/security.py` contém apenas as dependências do FastAPI que obtêm a sessão atual a partir do cookie, utilizando `jwt.py` e `session.py`, sem duplicar a lógica de emissão ou validação do JWT.

Essa organização facilita a manutenção e permite o crescimento incremental sem introduzir complexidade desnecessária.

---

## 11. Decisões de arquitetura

### 11.1. Autenticação com JWT próprio e sessões no Redis

O Alume utilizará um JWT próprio para autenticar as requisições do navegador ao backend.

Esse token será emitido após a autenticação bem-sucedida no SUAP e conterá inicialmente:

- `sid`: identificador único da sessão;
- `exp`: instante de expiração do token, igual ao do `access token` do SUAP.

O identificador `sid` será utilizado para relacionar o JWT à sessão armazenada no Redis.

A chave de armazenamento seguirá o padrão:

```text
alume:session:{sid}
```

O valor armazenará o `access token` retornado pelo SUAP. O `refresh token` é descartado: o Alume não renova tokens do SUAP.

O Redis também será responsável por controlar o TTL da sessão, igual ao tempo restante até a expiração do `access token` do SUAP. Dessa forma, o JWT do Alume, o token do SUAP e a sessão no Redis expiram no mesmo instante. Quando isso ocorre, a sessão é apagada do Redis e o estudante precisa fazer login novamente.

O navegador receberá somente o JWT do Alume. O token do SUAP permanecerá sob responsabilidade do backend, não sendo exposto ao cliente.

Essa abordagem separa duas responsabilidades:

- o JWT do Alume identifica e autentica a sessão nas requisições ao backend;
- o Redis mantém os dados necessários para validar a existência da sessão e acessar a API do SUAP.

A validação de uma requisição protegida dependerá tanto da validade do JWT quanto da existência da sessão correspondente no Redis. Isso permite revogar sessões imediatamente, sem aguardar a expiração do JWT.

O Redis será uma dependência necessária para as operações autenticadas. Caso esteja indisponível, o backend não conseguirá confirmar a existência das sessões e deverá rejeitar temporariamente as requisições protegidas.

Como o cookie do JWT permanece no navegador até a expiração, uma sessão só fica abandonada no Redis se o estudante apagar os cookies sem fazer logout. Nesse caso, o TTL funciona como mecanismo de limpeza.

### 11.2. JWT em cookie HttpOnly

O JWT do Alume será guardado em cookie HttpOnly, e não em memória do frontend nem em `localStorage` ou `sessionStorage`.

Motivos:

- o JWT fica inacessível ao JavaScript, o que reduz o impacto de uma vulnerabilidade XSS;
- recarregar a página ou reabrir o PWA não obriga o estudante a se autenticar de novo;
- o frontend com várias páginas HTML consegue navegar entre as telas sem perder a autenticação.

Implicações:

- o cookie deve ter os atributos `HttpOnly`, `Secure` e `SameSite=Lax`;
- o frontend deve enviar as requisições com `credentials: 'include'`;
- as requisições que alteram estado devem ser protegidas contra CSRF, com validação do cabeçalho `Origin`;

### 11.3. Frontend responsivo

O Alume deve ser implementado como uma aplicação web responsiva, com suporte a PWA.

O frontend inicial será desenvolvido em HTML, Bootstrap e JavaScript, com possibilidade de migração para React em versões futuras.

O frontend será organizado por feature (login, tela inicial e dashboard), para facilitar a evolução para um modelo mais modular, sem perder a simplicidade inicial da V1.

### 11.4. Backend em FastAPI

O backend será implementado em FastAPI, com Pydantic para validação de dados e HTTPX para requisições HTTP ao SUAP.

Essa escolha facilita o desenvolvimento, a validação de contratos e a integração com operações assíncronas.

Além disso, o backend utilizará Redis para armazenar temporariamente as sessões e o token do SUAP, garantindo controle centralizado de autenticação sem armazenar dados acadêmicos.

### 11.5. Backend como camada de orquestração

Mesmo sendo um sistema simples, o backend deve existir para:

- esconder detalhes da API externa;
- controlar a autenticação e as sessões;
- emitir e validar os JWTs do Alume;
- acessar o Redis para recuperar o token do SUAP;
- centralizar a lógica de cálculo dos indicadores;
- disponibilizar endpoints claros para o frontend.

### 11.6. Cálculo derivado em serviço dedicado

A lógica de limite de faltas, faltas restantes e risco deve estar em um serviço independente, facilitando manutenção e testes.

O serviço deverá receber os dados necessários e retornar os indicadores calculados, sem depender diretamente do cliente HTTP ou do Redis.

### 11.7. Deploy inicial

A arquitetura de deploy inicial será:

- frontend: Vercel;
- backend: Vercel;
- Redis: serviço externo gerenciado, compatível com a execução serverless.

Essa estratégia contempla a simplicidade operacional e o baixo custo inicial.

O frontend estático será responsável pela interface do usuário, enquanto o backend atenderá às rotas de autenticação, consulta e cálculo.

O Redis será mantido como infraestrutura externa, permitindo que diferentes instâncias do backend acessem o mesmo armazenamento de sessões. Essa continuidade depende da disponibilidade e conectividade do serviço Redis.

---

## 12. Considerações de desempenho e disponibilidade

### 12.1. Desempenho

- dados de disciplinas e frequência devem ser carregados em lote sempre que possível;
- a paginação das rotas de disciplinas e de diários deve ser tratada corretamente;
- o backend deve utilizar operações assíncronas para comunicação HTTP;
- chamadas externas devem possuir timeout e política de retry controlada.

Em versões posteriores, o Redis poderá ser utilizado também como cache temporário de informações acadêmicas, como disciplinas, frequência, faltas e indicadores calculados.

O objetivo será reduzir requisições repetidas à API do SUAP e melhorar o desempenho da aplicação.

Essa evolução deverá considerar que os dados acadêmicos podem mudar ao longo do tempo, exigindo uma política adequada de expiração e invalidação.

Na V1, o sistema não utilizará cache acadêmico e deverá consultar diretamente o SUAP conforme a necessidade das funcionalidades.

### 12.2. Disponibilidade

O sistema depende da disponibilidade do SUAP e do Redis.

Portanto, a arquitetura precisa contar com:

- tratamento de falhas em chamadas externas;
- mensagens claras para o usuário;
- identificação de indisponibilidade do SUAP;
- timeouts e tentativas de repetição controladas;
- tratamento de falhas de conexão com o Redis.

Como a V1 não possui cache acadêmico nem uma fonte alternativa de dados, a aplicação não deve prometer recuperação automática de informações acadêmicas em cenários de indisponibilidade do SUAP.

A indisponibilidade do Redis também impede a validação das sessões, afetando temporariamente o acesso às funcionalidades autenticadas.

---

## 13. Riscos e pendências

### 13.1. Riscos técnicos

- instabilidade da API do SUAP;
- variação dos payloads da API;
- necessidade de controle de paginação e filtros;
- indisponibilidade ou falha de conexão com o Redis;
- ataques CSRF, por causa do envio automático do cookie;
- vulnerabilidades XSS, que permitiriam disparar requisições em nome do usuário enquanto a sessão estiver ativa;
- limitações ou incompatibilidades entre o ambiente serverless e o serviço Redis escolhido;
- o token do SUAP não pode ser invalidado pelo Alume e permanece válido até expirar, mesmo após o logout.

### 13.2. Pendências técnicas

- escolher o provedor do Redis e validar sua compatibilidade com o ambiente de deploy;

### 13.3. Pendências de negócio

- definir as faixas do percentual do limite consumido que classificam cada disciplina como crítica, em risco ou estável.

---

## 14. Conclusão

A arquitetura do Alume deve ser simples, orientada à integração com o SUAP e centrada na separação de responsabilidades.

O projeto utilizará um backend em FastAPI, responsável pela autenticação, gerenciamento das sessões, integração externa e processamento dos indicadores. O Redis será utilizado como banco de dados temporário para armazenar as sessões e o token do SUAP, enquanto o navegador usará um JWT próprio do Alume, guardado em cookie HttpOnly, para autenticar suas requisições. A sessão expira junto com o token do SUAP, sem uso de refresh.

A solução prioriza:

- autenticação segura e controle de sessão;
- isolamento do token do SUAP no backend;
- consulta de dados acadêmicos externos;
- processamento dos indicadores de frequência;
- ausência de persistência acadêmica própria na V1;
- apresentação intuitiva e responsiva ao estudante.
