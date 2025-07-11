# VTEX IO Backend - Glean Middleware

Este é um backend VTEX IO que implementa um middleware para integração com a API do Glean, permitindo consultas inteligentes ao conhecimento da empresa.

## 🚀 Funcionalidades

- **Consulta ao Glean**: Envia perguntas para a API do Glean e retorna respostas baseadas no conhecimento da empresa
- **Processamento de Resposta**: Extrai e concatena todos os textos dos fragments da última mensagem
- **Tratamento de Erros**: Implementa tratamento robusto de erros com logs detalhados
- **TypeScript**: Código totalmente tipado para maior segurança e desenvolvimento

## 📋 Pré-requisitos

- VTEX CLI configurado
- Acesso à API do Glean configurado

## 🛠️ Instalação

1. Clone o repositório
2. Instale as dependências:
3. Link do app em um workspace `vtex link`

## 🔧 Configuração

### 1. Configuração do Token no Client

**IMPORTANTE**: É necessário configurar o token de autenticação no arquivo `node/clients/glean.ts`.

Abra o arquivo `node/clients/glean.ts` e substitua `{{TOKEN AQUI}}` pelo seu token real:

```typescript
headers: {
  'X-VTEX-Use-Https': 'true',
  'Content-Type': 'application/json',
  'Authorization': 'Bearer SEU_TOKEN_AQUI', // Substitua pelo token real
},
```

## 📡 Como Usar

### Endpoint

**POST** `/_v/glean`

### Request Body

```json
{
  "pergunta": "Qual API cria um pedido?"
}
```

### Response

**Sucesso (200):**
```json
{
  "text": "Para criar um pedido na VTEX, você deve utilizar a API Place Order do Checkout..."
}
```

**Sem conteúdo (204):**
```json
{
  "text": ""
}
```

**Erro (500):**
```json
{
  "error": "Erro ao consultar a API do Glean",
  "message": "Detalhes do erro"
}
```

## 🏗️ Estrutura do Projeto

```
node/
├── clients/
│   ├── glean.ts          # Cliente para comunicação com API do Glean
│   └── index.ts          # Exportação dos clients
├── middlewares/
│   └── gleanMiddleware.ts # Middleware principal
├── __tests__/
│   └── simple.test.ts    # Testes básicos
├── index.ts              # Configuração principal do serviço
├── service.json          # Configuração das rotas
└── package.json          # Dependências do projeto
```

## 🔍 Como Funciona

### 1. Recebimento da Requisição
O middleware recebe uma requisição POST com uma pergunta no body.

### 2. Chamada para API do Glean
A pergunta é enviada para a API do Glean através do client configurado, seguindo a [documentação oficial da API Chat do Glean](https://developers.glean.com/api/client-api/chat/chat).

### 3. Processamento da Resposta
- Extrai a última mensagem do array `messages`
- Concatena todos os textos dos `fragments` dessa mensagem
- Remove fragments vazios e junta os textos com espaço

### 4. Retorno da Resposta
Retorna o texto concatenado ou uma resposta vazia se não houver conteúdo.

## 📚 Documentação da API Glean

Este middleware utiliza a **API Chat do Glean** para processar consultas. Para mais detalhes sobre como a API funciona, consulte a [documentação oficial](https://developers.glean.com/api/client-api/chat/chat).

### Endpoint Utilizado
- **POST** `/rest/api/v1/chat` - Para ter uma conversa com o Glean AI

### Códigos de Status da API
- **200** - OK (Sucesso)
- **400** - Invalid request (Requisição inválida)
- **401** - Not Authorized (Não autorizado)
- **408** - Request Timeout (Timeout da requisição)
- **429** - Too Many Requests (Muitas requisições)

### Estrutura da Requisição
O middleware envia uma requisição com a seguinte estrutura:

```json
{
  "stream": false,
  "applicationId": "enfclujvunxyd2pk",
  "messages": [
    {
      "author": "USER",
      "fragments": [
        {
          "text": "Sua pergunta aqui"
        }
      ]
    }
  ]
}
```


**Resultado processado:**
```
"Para criar um pedido na VTEX, você deve utilizar a API Place Order do Checkout. A documentação completa está disponível em developers.vtex.com"
```

### Estrutura da Resposta da API
A API do Glean retorna uma resposta com a seguinte estrutura:

```json
{
  "messages": [
    {
      "author": "GLEAN_AI",
      "fragments": [
        {
          "text": "Texto da resposta"
        },
        {
          "citation": {
            "sourceDocument": {
              "id": "DOCUMENT_ID"
            }
          }
        }
      ],
      "messageId": "unique-message-id",
      "messageTrackingToken": "tracking-token",
      "messageType": "UPDATE",
      "stepId": "step-id",
      "workflowId": "workflow-id",
      "workflowRunId": "workflow-run-id"
    }
  ]
}
```

### Campos Importantes da Resposta
- **`messages`**: Array de mensagens da conversa
- **`fragments`**: Array de fragmentos que compõem a mensagem
- **`text`**: Conteúdo textual do fragmento
- **`citation`**: Referência à fonte do conhecimento (quando aplicável)
- **`author`**: Identifica se a mensagem é do usuário ou do Glean AI


## 📊 Logs

O middleware inclui logs detalhados para debug:

- Resposta completa da API do Glean
- Estrutura da resposta processada
- Texto final extraído
- Erros detalhados em caso de falha
