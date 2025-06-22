# 📦 Padrão de Payload – API Municipais

Esta documentação define o formato padrão esperado para o payload de comunicação à API Municipais.

--- 

## 🔁 Rotas Disponíveis

Todas as rotas utilizam o método HTTP **`POST`**.

| Rota                         | Descrição                                     |
| ---------------------------- | --------------------------------------------- |
| [`/consultar`](#consultar)   | Consulta as notas já escrituradas ou status.  |
| [`/escriturar`](#escriturar) | Realiza a escrituração das notas.             |
| [`/guia`](#guia)             | Gera a guia de recolhimento (ISS ou similar). |
| [`/excluir`](#excluir)       | Exclui uma nota.                              |

---

## 🔍 Consultar

A operação de consulta permite recuperar informações sobre notas fiscais previamente escrituradas em determinado período, bem como verificar a integridade e o status das notas processadas via integração.

### 🔼 Requisição
Requisição para a rota `/consultar`

**Metodo**: `POST`

**Formato**: `json`

**Body**: Objeto JSON contendo o campo dados_acesso.

#### 🔐 `dados_acesso` (objeto)

Objeto que contém as credenciais de autenticação e os parâmetros de filtragem por competência,
utilizados para estabelecer a sessão de comunicação com o sistema municipal e delimitar o escopo da consulta.

| Campo                  | Tipo    | Obrigatório | Descrição                                                                     | Observação                                                             |
| ---------------------- | ------- | ----------- | ----------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| `municipio`            | string  | Sim         | Nome do municipio.                                                            |                                                                        |
| `inscricao`            | string  | Sim         | CNPJ/CPF/Inscricao estadual/inscricao municipal da empresa (somente números). |                                                                        |
| `extra`                | string  | Não         | Informações extras. Para caso a prefeitura precise de mais informações.       |                                                                        |
| `login_password`       | string  | Sim         | Senha de login.                                                               |                                                                        |
| `mes`                  | integer | Sim         | Mês de competência (1 a 12).                                                  |                                                                        |
| `ano`                  | integer | Sim         | Ano de competência (ex: `2025`).                                              |                                                                        |
| `identificador`        | string  | Não         | Identificador da prefeitura.                                                  | Algumas prefeituras precisam do numero identificador para fazer login. |
| `nome_filial`          | string  | Não         | Nome da filial.                                                               | Algumas prefeituras precisam para identificar a filial.                |
| `certificado_b64`      | string  | Não         | Certificado digital em base64 no formato `.pfx`.                              |                                                                        |
| `certificado_password` | string  | Não         | Senha do certificado digital.                                                 |                                                                        |

---
#### 📑 Exemplo de Payload

```json
{
  "dados_acesso": {
	"municipio": "prata",
    "inscricao": "27473570",
	"extra": "extra",
    "login_password": "123456",
    "mes": 5,
    "ano": 2025,
	"identificador": "123456",
	"nome_filial": "Filial 1",
    "certificado_b64": "-----BEGIN CERTIFICATE-----\n-----END CERTIFICATE-----",
    "certificado_password": "123456"

  }
}
```

***Obs: Não precisamos enviar na consulta as `notas` pois serão descartadas. Assim conseguimos montar payloads mas fácil de ler.*** 


### 🔽  Resposta

**Formato**: `json`

| Campo             | Tipo    | Obrigatório | Descrição                                                                                | Observação                                                                                          |
| ----------------- | ------- | ----------- | ---------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| `cnpj_filial`     | string  | Sim         | CNPJ do emitente da nota (somente números).                                              |                                                                                                     |
| `nome_municipio`  | string  | Não         | Nome do municipio.                                                                       |                                                                                                     |
| `cnpj_prestador`  | string  | Sim         | CNPJ do prestador.                                                                       | Alguns municipios não tem o CNPJ do prestador, para esses casos precisa ver com o cliente           |
| `numero_nota`     | integer | Sim         | Número da nota fiscal.                                                                   | Sempre que possivel envie como inteiro. Se tiver 0 à esquerda ou for muito longo envie como string. |
| `data_emissao`    | string  | Sim         | Data de emissão no formato `DD/MM/AAAA`.                                                 |                                                                                                     |
| `situacao`        | boolean | Sim         | Indica se houve retenção de ISS (`true` ou `false`).                                     |                                                                                                     |
| `total`           | float   | Sim         | Valor total da nota incluidos os ISS.                                                    |                                                                                                     |
| `valor_principal` | float   | Sim         | Valor somente do ISS.                                                                    |                                                                                                     |
| `chave`           | string  | Sim         | Chave montada juntando `cnpj_filial` + `cnpj_prestador` + `numero_nota` + `data_emissao` | Pode manter o "/" da data de emissão.                                                               |


## 📝 Escriturar

A operação de escrituração permite a inclusão de notas fiscais no sistema municipal.

### 🔼 Requisição

Requisição para a rota `/escriturar`

**Metodo**: `POST`

**Formato**: `json`

**Body**: Objeto JSON contendo o campo dados_acesso e notas.

#### 🔐 `dados_acesso` (objeto)

Objeto que contém as credenciais de autenticação e os parâmetros de filtragem por competência,
utilizados para estabelecer a sessão de comunicação com o sistema municipal e delimitar o escopo da consulta.

| Campo                  | Tipo    | Obrigatório | Descrição                                                                     | Observação                                                             |
| ---------------------- | ------- | ----------- | ----------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| `municipio`            | string  | Sim         | Nome do municipio.                                                            |                                                                        |
| `inscricao`            | string  | Sim         | CNPJ/CPF/Inscricao estadual/inscricao municipal da empresa (somente números). |                                                                        |
| `extra`                | string  | Não         | Informações extras. Para caso a prefeitura precise de mais informações.       |                                                                        |
| `login_password`       | string  | Sim         | Senha de login.                                                               |                                                                        |
| `mes`                  | integer | Sim         | Mês de competência (1 a 12).                                                  |                                                                        |
| `ano`                  | integer | Sim         | Ano de competência (ex: `2025`).                                              |                                                                        |
| `identificador`        | string  | Não         | Identificador da prefeitura.                                                  | Algumas prefeituras precisam do numero identificador para fazer login. |
| `nome_filial`          | string  | Não         | Nome da filial.                                                               | Algumas prefeituras precisam para identificar a filial.                |
| `certificado_b64`      | string  | Não         | Certificado digital em base64 no formato `.pfx`.                              |                                                                        |
| `certificado_password` | string  | Não         | Senha do certificado digital.                                                 |                                                                        |

---

#### 📄 `notas` (array de objetos)

Lista de notas fiscais a serem escrituradas.


| Campo            | Tipo    | Obrigatório | Descrição                                                          | Observação               | Exemplo          |
| ---------------- | ------- | ----------- | ------------------------------------------------------------------ | ------------------------ | ---------------- |
| `id`             | integer | Sim         | Identificador interno da nota.                                     | ID da nota no DB         | 1                |
| `cnpj_prestador` | string  | Sim         | CNPJ do prestador da nota (somente números).                       |                          | "12345678901234" |
| `nome_municipio` | string  | Sim         | Nome do municipio.                                                 |                          | "prata"          |
| `numero_nota`    | string  | Sim         | Número da nota fiscal.                                             |                          | "123456789"      |
| `serie`          | string  | Não         | Série da nota fiscal.                                              |                          | "A"              |
| `data_emissao`   | string  | Sim         | Data de emissão no formato `DD/MM/AAAA`.                           |                          | "01/01/2025"     |
| `retido`         | boolean | Sim         | Indica se houve retenção de ISS (`true` ou `false`).               |                          | false            |
| `valor_nota`     | string  | Sim         | Valor da nota fiscal.                                              | Separado por "." (ponto) | "100.00"         |
| `aliquota`       | string  | Sim         | Valor da alíquota da nota fiscal.                                  | Separado por "." (ponto) | "2.00"           |
| `cod_servico`    | string  | Sim         | Código do serviço.                                                 |                          | "07.01"          |
| `valor_iss`      | string  | Sim         | Valor do ISS.                                                      | Separado por "." (ponto) | "10.00"          |
| `simples`        | boolean | Sim         | Indica se o prestador possui simples nacional (`true` ou `false`). |                          | true             |
| `cod_ibge`       | string  | Sim         | Código do IBGE do municipio do prestador.                          |                          | "123456"         |

---

#### 📑 Exemplo de Payload

```json
{
  "dados_acesso": {
	"municipio": "prata",
    "inscricao": "27473570",
	"extra": "extra",
    "login_password": "123456",
    "mes": 5,
    "ano": 2025,
	"identificador": "123456",
	"nome_filial": "Filial 1",
    "certificado_b64": "-----BEGIN CERTIFICATE-----\n-----END CERTIFICATE-----",
    "certificado_password": "123456"

  },
  "notas": [
	{
	  "id": 1,
	  "cnpj_prestador": "27473570",
	  "nome_municipio": "São Paulo",
	  "numero_nota": "123456789",
	  "serie": "A",
	  "data_emissao": "01/01/2025",
	  "retido": false,
	  "valor_nota": "100.00",
	  "aliquota": "2.00",
	  "cod_servico": "1234",
	  "valor_iss": "10.00",
	  "simples": false
	}
  ]
}
```

### 🔽 Resposta

A maioria das API's rodam dentro de workers(processos assíncronos) utilizando [rq](https://github.com/rq/rq), portanto, a resposta não é retornada diretamente ao cliente.

Como é um processo assíncrono, precisamos retornar um `id` que representa o status da escrituração.

Algumas notas serão escrituradas em batchs, por isso, o `id` retornado não representa uma unica nota, mas sim um batch de notas.

**Exemplo**: São Enviados 50 notas, mas são criados batches de 10 notas, por isso, o `id` retornado representa o status de 5 batches.

**Formato**: `json`

Body: Objeto JSON contendo o campo `id` gerada automaticamente pelo worker.

| Campo    | Tipo    | Obrigatório | Descrição                     |
| -------- | ------- | ----------- | ----------------------------- |
| `id`     | integer | Sim         | ID do status da escrituração. |
| `status` | string  | Sim         | Status da escrituração.       |


### 🔨 Resposta de escriturar no worker

Mesmo as notas escrituradas dentro do worker precisam retornar algo para a gente saber se foram escrituradas ou não.

Atualmente salvamos o status da escrituração no DB, mas isso pode mudar.

Retorne `true` no `sucesso` se foi escriturada, `false` se não foi escriturada.

**Formato**: `json` | `dict`

Body: Array de Objeto JSON.

| Campo            | Tipo    | Obrigatório | Descrição                                 |
| ---------------- | ------- | ----------- | ----------------------------------------- |
| `id`             | integer | Sim         | ID do status da escrituração.             |
| `sucesso`        | boolean | Sim         | Status da escrituração `true` ou `false`. |
| `numero_nota`    | string  | Sim         | Número da nota escriturada.               |
| `cnpj_prestador` | string  | Sim         | CNPJ do prestador.                        |
| `mensagem`       | string  | Sim         | Mensagem com o motivo do erro.            |

***Obs: Deve retornar isso para cada nota escriturada. Independente se você já salvou no DB ou não, pois precisamos dessa informação para criar o log.***

## 📝 Gerar

A Operação para geração de guias de recolhimento para pagamento de ISS.

### 🔼 Requisição

Requisição para a rota `/gerar`

**Metodo**: `POST`

**Formato**: `json`

**Body**: Objeto JSON contendo o campo dados_acesso e notas.

#### 🔐 `dados_acesso` (objeto)

Objeto que contém as credenciais de autenticação e os parâmetros de filtragem por competência,
utilizados para estabelecer a sessão de comunicação com o sistema municipal e delimitar o escopo da consulta.

| Campo                  | Tipo    | Obrigatório | Descrição                                                                     | Observação                                                             |
| ---------------------- | ------- | ----------- | ----------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| `municipio`            | string  | Sim         | Nome do municipio.                                                            |                                                                        |
| `inscricao`            | string  | Sim         | CNPJ/CPF/Inscricao estadual/inscricao municipal da empresa (somente números). |                                                                        |
| `extra`                | string  | Não         | Informações extras. Para caso a prefeitura precise de mais informações.       |                                                                        |
| `login_password`       | string  | Sim         | Senha de login.                                                               |                                                                        |
| `mes`                  | integer | Sim         | Mês de competência (1 a 12).                                                  |                                                                        |
| `ano`                  | integer | Sim         | Ano de competência (ex: `2025`).                                              |                                                                        |
| `identificador`        | string  | Não         | Identificador da prefeitura.                                                  | Algumas prefeituras precisam do numero identificador para fazer login. |
| `nome_filial`          | string  | Não         | Nome da filial.                                                               | Algumas prefeituras precisam para identificar a filial.                |
| `certificado_b64`      | string  | Não         | Certificado digital em base64 no formato `.pfx`.                              |                                                                        |
| `certificado_password` | string  | Não         | Senha do certificado digital.                                                 |                                                                        |

---


***Deixar isso para conversar depois com o pessoal por causa dos municipios que não precisam escriturar, só escolhem a nota para gerar a guia***
### 🔽 Resposta

Resposta da rota `/gerar`

**Formato**: `json`

**Body**: Objeto JSON contendo o campo guias.

#### 📄 guia (Objeto)

Objeto contendo as informações da guia de recolhimento.
O PDF precisa ser convertido para base64.

| Campo             | Tipo   | Obrigatório | Descrição                                               | Observação                                       |
| ----------------- | ------ | ----------- | ------------------------------------------------------- | ------------------------------------------------ |
| `pdf`             | string | Sim         | PDF da guia de recolhimento.                            | O PDF precisa ser convertido para base64.        |
| `vencimento`      | string | Sim         | Data de vencimento da guia de recolhimento(DD/MM/AAAA). | Caso não tenha vencimento, busque dentro do PDF. |
| `valor_principal` | float  | Sim         | Valor somente do ISS.                                   | Caso não tenha vencimento, busque dentro do PDF. |
| `valor_base`      | float  | Sim         | Valor total da nota incluidos os ISS.                   | Caso não tenha vencimento, busque dentro do PDF. |
| `codigo_barra`    | string | Sim         | Codigo de barras.                                       | Caso não tenha vencimento, busque dentro do PDF. |



