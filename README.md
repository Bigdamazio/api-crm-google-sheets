# API de CRM Integrada com Google Sheets

Esta API foi desenvolvida usando Google Apps Script, integrando uma planilha do Google Sheets como backend. Ela oferece funcionalidades para cadastro, consulta e atualização de dados dos clientes via API. A API permite a interação utilizando métodos `GET` e `POST`.

## Funcionalidades

1. **Cadastro de Cliente (POST)**
2. **Consulta de Cliente (GET)**
3. **Atualização de Dados do Cliente (POST)**
4. **Respostas JSON para sucesso e erro**

## Pré-requisitos

* **Google Sheets**: A API utiliza o Google Sheets como banco de dados.
* **Google Apps Script**: O código é implementado no ambiente de script do Google.
* **API Key (opcional)**: Caso queira autenticação adicional, você pode adicionar uma chave de API.

## Como Utilizar

### 1. Função Principal para POST (Cadastro e Verificação de Cliente)

A função `doPost` recebe os dados de um cliente (via JSON) e realiza a verificação para determinar se o cliente já está cadastrado. Caso contrário, o cliente será inserido na planilha.

```javascript
function doPost(e) {
  try {
    if (!e || !e.postData || !e.postData.contents) {
      throw new Error("Nenhum dado foi recebido na requisição.");
    }

    var params = JSON.parse(e.postData.contents);
    Logger.log("🚀 JSON Recebido (POST): " + JSON.stringify(params));

    var numeroCliente = params.NUMERO_CLIENTES ? String(params.NUMERO_CLIENTES).trim() : "";

    if (!numeroCliente) {
      return cadastrarNovoCliente(params);
    }

    var clientRange = findClientRangeByNumber(numeroCliente);

    if (clientRange) {
      Logger.log("⚠️ Cliente já cadastrado na linha: " + clientRange.getRow());
      var data = getSheet().getRange(clientRange.getRow(), 1, 1, 9).getValues()[0];
      
      var cliente = {
        mensagem: "✅ Número já cadastrado!",
        NUMERO_CLIENTES: data[0],
        NOME: data[1] || "",
        EMAIL: data[2] || "",
        CNPJ: data[3] || "",
        COD_EMPRESA: data[4] || "",
        UTILIZA_BOT: data[5] || "",
        FUNCIONALIDADES: data[6] || "",
        EMISSOR_NFSE: data[7] || "",
        AREA_CLIENTE: data[8] || ""
      };

      return createJsonResponse(cliente);
    }

    return cadastrarNovoCliente(params, numeroCliente);

  } catch (error) {
    Logger.log("❌ Erro (POST): " + error.message);
    return createErrorResponse("Erro ao processar a requisição", error.message);
  }
}
```

#### Como Funciona:

1. O script espera um JSON contendo informações do cliente, como `NUMERO_CLIENTES`, `NOME`, `EMAIL`, etc.
2. Se o número do cliente (`NUMERO_CLIENTES`) não for fornecido, o script cadastra um novo cliente.
3. Se o número do cliente for fornecido, ele verifica se o cliente já existe. Caso sim, retorna os dados já cadastrados. Caso contrário, insere um novo cliente.

---

### 2. Função para Consulta (GET)

A função `doGet` consulta os dados de um cliente específico a partir do número de cliente. Os dados são retornados em formato JSON.

```javascript
function doGet(e) {
  try {
    if (!e || !e.parameter || !e.parameter.NUMERO_CLIENTES) {
      throw new Error("Parâmetro NUMERO_CLIENTES é obrigatório.");
    }

    var numeroCliente = e.parameter.NUMERO_CLIENTES.trim();
    Logger.log("🔍 Buscando cliente: " + numeroCliente);

    var clientRange = findClientRangeByNumber(numeroCliente);

    if (clientRange) {
        var data = getSheet().getRange(clientRange.getRow(), 1, 1, 9).getValues()[0];
        var cliente = {
            NUMERO_CLIENTES: data[0],
            NOME: data[1] || "",
            EMAIL: data[2] || "",
            CNPJ: data[3] || "",
            COD_EMPRESA: data[4] || "",
            UTILIZA_BOT: data[5] || "",
            FUNCIONALIDADES: data[6] || "",
            EMISSOR_NFSE: data[7] || "",
            AREA_CLIENTE: data[8] || ""
        };
        Logger.log("✅ Cliente encontrado: " + JSON.stringify(cliente));
        return createJsonResponse(cliente);
    }

    Logger.log("⚠️ Cliente não encontrado.");
    return createErrorResponse("Cliente não encontrado", null, 404);

  } catch (error) {
    Logger.log("❌ Erro (GET): " + error.message);
    return createErrorResponse("Erro ao processar a requisição", error.message);
  }
}
```

#### Como Funciona:

1. A requisição `GET` precisa incluir o número do cliente (`NUMERO_CLIENTES`) como parâmetro na URL.
2. O script procura pelo número do cliente na planilha e, caso encontrado, retorna os dados no formato JSON.

---

### 3. Função para Atualização de Dados (POST)

Esta função permite a atualização de informações de um cliente já cadastrado.

```javascript
function atualizarDados(params) {
  try {
    Logger.log("🔄 Atualizando cliente: " + JSON.stringify(params));

    var numeroCliente = params.NUMERO_CLIENTES ? String(params.NUMERO_CLIENTES).trim() : "";
    if (!numeroCliente) {
      throw new Error("O campo 'NUMERO_CLIENTES' é obrigatório para atualizar os dados.");
    }

    var clientRange = findClientRangeByNumber(numeroCliente);

    if (clientRange) {
      var sheet = getSheet();
      var row = clientRange.getRow();
      
      var dadosParaAtualizar = [
        params.NOME || "",
        params.EMAIL || "",
        params.CNPJ || "",
        params.COD_EMPRESA || "",
        params.UTILIZA_BOT || "",
        params.FUNCIONALIDADES || "",
        params.EMISSOR_NFSE || "",
        params.AREA_CLIENTE || ""
      ];
      
      sheet.getRange(row, 2, 1, 8).setValues([dadosParaAtualizar]);

      Logger.log("✅ Dados atualizados para o cliente: " + numeroCliente);
      return createJsonResponse({ mensagem: "✅ Dados do cliente atualizados com sucesso!" });
    }

    return createErrorResponse("Cliente não encontrado.", null, 404);

  } catch (error) {
    Logger.log("❌ Erro (Atualizar Dados): " + error.message);
    return createErrorResponse("Erro ao atualizar os dados", error.message);
  }
}
```

---

### 4. Funções Auxiliares

* **findClientRangeByNumber**: Encontra o cliente na planilha com base no número do cliente.
* **cadastrarNovoCliente**: Insere um novo cliente na planilha.
* **createJsonResponse**: Retorna uma resposta JSON com os dados fornecidos.
* **createErrorResponse**: Retorna uma resposta JSON com detalhes do erro.

### Exemplo de Request/Response

#### Exemplo de POST:

**Requisição (POST)**:

```json
{
  "NUMERO_CLIENTES": "12345",
  "NOME": "Cliente Exemplo",
  "EMAIL": "cliente@exemplo.com",
  "CNPJ": "12345678000100",
  "COD_EMPRESA": "001",
  "UTILIZA_BOT": "Sim",
  "FUNCIONALIDADES": "Consultoria",
  "EMISSOR_NFSE": "Sim",
  "AREA_CLIENTE": "TI"
}
```

**Resposta (JSON)**:

```json
{
  "mensagem": "✅ Cliente cadastrado! Agora preencha os demais dados."
}
```

#### Exemplo de GET:

**Requisição (GET)**:

```http
GET https://script.google.com/macros/s/EXAMPLE_ID/exec?NUMERO_CLIENTES=12345
```

**Resposta (JSON)**:

```json
{
  "NUMERO_CLIENTES": "12345",
  "NOME": "Cliente Exemplo",
  "EMAIL": "cliente@exemplo.com",
  "CNPJ": "12345678000100",
  "COD_EMPRESA": "001",
  "UTILIZA_BOT": "Sim",
  "FUNCIONALIDADES": "Consultoria",
  "EMISSOR_NFSE": "Sim",
  "AREA_CLIENTE": "TI"
}
```

---

## Contribuindo

Se você quiser contribuir para o desenvolvimento desta API, basta fazer um fork do repositório e enviar um pull request com suas melhorias ou correções.
