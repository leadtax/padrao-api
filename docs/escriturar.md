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
