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

#### Estrutura da Resposta (Sucesso)

```json
{
    "status": null,
    "requestID": "0a9736459b6c1b13819b8ff522d51e09",
    "warnings": null,
    "errors": null,
    "retryWait": 0,
    "metaData": null,
    "attributes": {
        "identityName": "09257316920",
        "customCredentials": "TIM-Custom-RSA-Cloud-Credentials",
        "type": "onpremise",
        "activationUrl": "com.rsa.securid://ctf?ctfData=200193850501325125304724240511562112550013127207153660245152163270154577437524233",
        "user": "T3622753",
        "operation": "addtoken",
        "onpremiseProfile": "IOS"
    },
    "complete": false,
    "success": false,
    "retry": false,
    "failure": false
}
```

#### Campos Importantes da Resposta

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `requestID` | String | Identificador único da requisição gerado pelo IdentityIQ |
| `attributes.activationUrl` | String | **URL de ativação do token RSA**. Este é o valor principal retornado para operações `addtoken`. Formato: `com.rsa.securid://ctf?ctfData=...` |
| `attributes.user` | String | Matrícula do usuário processado |
| `attributes.operation` | String | Operação executada (`addtoken` ou `removetoken`) |
| `attributes.type` | String | Tipo de ambiente utilizado |
| `attributes.onpremiseProfile` | String | Perfil do dispositivo utilizado |
| `errors` | Array/String | **Campo de erros**. Será `null` em caso de sucesso. Em caso de erro, conterá a mensagem de erro detalhada |
| `warnings` | Array/String | Avisos gerados durante a execução (se houver) |

#### Tratamento de Erros

Em caso de erro na execução, o campo `errors` será preenchido com a mensagem de erro específica. Exemplos de erros comuns:

- **Type inválido**: Quando o valor de `type` não é suportado (ex: valor diferente de `"onpremise"` quando ainda não há suporte para `"cloud"`)
- **Operation não existe**: Quando o valor de `operation` não é `"addtoken"` ou `"removetoken"`
- **User inválido**: Quando a matrícula do usuário não é encontrada ou é inválida
- **onpremiseProfile inválido**: Quando o perfil informado não é `"IOS"`, `"Android"` ou `"Desktop"`
- **Campos obrigatórios ausentes**: Quando campos obrigatórios não são informados

Exemplo de resposta com erro:

```json
{
    "status": null,
    "requestID": "0a9736459b6c1b13819b8ff522d51e09",
    "warnings": null,
    "errors": ["Operation 'invalidoperation' não é suportada. Valores válidos: addtoken, removetoken"],
    "retryWait": 0,
    "metaData": null,
    "attributes": {
        "user": "T3622753",
        "operation": "invalidoperation"
    },
    "complete": false,
    "success": false,
    "retry": false,
    "failure": false
}
```

**Importante**: Sempre verifique o campo `errors` na resposta para identificar problemas na execução da requisição.

### Observações

- O campo `requestId` é utilizado apenas para fins de auditoria e rastreamento. Pode conter qualquer informação relevante (número de ticket, identificador único, etc.)
- O atributo `onpremiseProfile` é obrigatório apenas quando `type` é `"onpremise"`
- O suporte para `type: "cloud"` está em desenvolvimento e será disponibilizado em versões futuras

### Collection Postman

Uma collection do Postman está disponível no arquivo `TIM-RSA.postman_collection.json` com exemplos prontos para uso das operações `addtoken` e `removetoken`.

