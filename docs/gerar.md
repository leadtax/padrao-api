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
