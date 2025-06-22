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
