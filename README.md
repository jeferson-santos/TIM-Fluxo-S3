# TIM - Rules

Este repositório contém as regras e workflows do projeto TIM para IdentityIQ.

## API de Gerenciamento de Tokens RSA

### Endpoint

```
POST http://10.151.54.69:8080/identityiq/rest/workflows/TIM-RSA/launch
```

### Descrição

Este endpoint permite solicitar a adição ou remoção de tokens RSA para usuários. O workflow processa a requisição e executa a operação solicitada no sistema RSA.

### Autenticação

O endpoint requer autenticação adequada no IdentityIQ. Certifique-se de incluir as credenciais necessárias na requisição.

### Parâmetros da Requisição

#### Body (JSON)

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `workflowArgs` | Object | Sim | Objeto contendo os argumentos do workflow |
| `workflowArgs.requestId` | String | Sim | Identificador único para rastreamento e auditoria. Pode ser qualquer informação (ex: "TK001", ticket, etc.) |
| `workflowArgs.type` | String | Sim | Tipo de ambiente. Atualmente apenas `"onpremise"` é suportado. Suporte para `"cloud"` está em desenvolvimento |
| `workflowArgs.operation` | String | Sim | Operação a ser executada. Valores possíveis: `"addtoken"` ou `"removetoken"` |
| `workflowArgs.user` | String | Sim | Matrícula do usuário (ex: "T3622753") |
| `workflowArgs.onpremiseProfile` | String | Condicional | Perfil do dispositivo. Obrigatório quando `type` é `"onpremise"`. Valores possíveis: `"IOS"`, `"Android"` ou `"Desktop"` |

### Exemplos de Requisição

#### Adicionar Token (addtoken)

```json
{
    "workflowArgs": {
        "requestId": "TK001",
        "type": "onpremise",
        "operation": "addtoken",
        "user": "T3622753",
        "onpremiseProfile": "IOS"
    }
}
```

#### Remover Token (removetoken)

```json
{
    "workflowArgs": {
        "requestId": "TK002",
        "type": "onpremise",
        "operation": "removetoken",
        "user": "T3622753",
        "onpremiseProfile": "Android"
    }
}
```

### Valores Válidos

#### `type`
- `"onpremise"` - Ambiente on-premise (atualmente suportado)
- `"cloud"` - Ambiente cloud (em desenvolvimento)

#### `operation`
- `"addtoken"` - Adiciona um novo token para o usuário
- `"removetoken"` - Remove um token existente do usuário

#### `onpremiseProfile` (quando `type` é `"onpremise"`)
- `"IOS"` - Perfil para dispositivos iOS
- `"Android"` - Perfil para dispositivos Android
- `"Desktop"` - Perfil para aplicações desktop

### Resposta

A resposta do endpoint segue o formato padrão do IdentityIQ para workflows. O resultado da operação será retornado no corpo da resposta.

### Observações

- O campo `requestId` é utilizado apenas para fins de auditoria e rastreamento. Pode conter qualquer informação relevante (número de ticket, identificador único, etc.)
- O atributo `onpremiseProfile` é obrigatório apenas quando `type` é `"onpremise"`
- O suporte para `type: "cloud"` está em desenvolvimento e será disponibilizado em versões futuras

### Collection Postman

Uma collection do Postman está disponível no arquivo `TIM-RSA.postman_collection.json` com exemplos prontos para uso das operações `addtoken` e `removetoken`.

