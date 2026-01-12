# Documentação Técnica - API de Gerenciamento de Tokens RSA

## 1. Visão Geral

A API de Gerenciamento de Tokens RSA permite solicitar a adição ou remoção de tokens RSA para usuários através de um workflow do IdentityIQ. O workflow processa as requisições e executa as operações no sistema RSA, suportando dois ambientes:

- **OnPremise**: Ambiente on-premise (totalmente funcional)
- **Cloud**: Ambiente cloud (em desenvolvimento)

### 1.1 Endpoint

```
POST http://smartid.internal.timbrasil.com.br/identityiq/rest/workflows/TIM-RSA/launch
```

### 1.2 Autenticação

O endpoint requer autenticação **Basic Auth (HTTP Basic Authentication)** no IdentityIQ.

**Header de Autenticação:**
```
Authorization: Basic <base64(user:password)>
```

**Requisitos de Permissão:**
- O usuário deve possuir permissões para executar workflows no IdentityIQ
- Deve ter acesso ao workflow TIM-RSA
- Deve ter permissões para realizar operações de gerenciamento de tokens RSA

---

## 2. Estrutura da Requisição

### 2.1 Body (JSON)

A requisição deve ser enviada com Content-Type: `application/json` e conter um objeto `workflowArgs` com os parâmetros do workflow.

### 2.2 Parâmetros da Requisição

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `workflowArgs` | Object | **Sim** | Objeto contendo os argumentos do workflow |
| `workflowArgs.requestId` | String | **Sim** | Identificador único para rastreamento e auditoria (ex: "TK001", ticket, etc.) |
| `workflowArgs.type` | String | **Sim** | Tipo de ambiente. Valores: `"onpremise"` ou `"cloud"` |
| `workflowArgs.operation` | String | **Sim** | Operação a ser executada. Valores: `"addtoken"` ou `"removetoken"` |
| `workflowArgs.user` | String | **Sim** | Matrícula do usuário (ex: "T3622753") |
| `workflowArgs.onpremiseProfile` | String | **Condicional** | Perfil do dispositivo. Obrigatório quando `type` é `"onpremise"`. Valores: `"IOS"`, `"Android"` ou `"Desktop"` |
| `workflowArgs.emailToSend` | String | **Condicional** | Email para envio do código de verificação. Obrigatório quando `type` é `"cloud"` e `operation` é `"addtoken"` |
| `workflowArgs.deviceSerialNumber` | String | **Opcional** | Número de série do dispositivo (apenas para OnPremise) |

### 2.3 Valores Válidos

#### `type`
- `"onpremise"` - Ambiente on-premise (suportado)
- `"cloud"` - Ambiente cloud (em desenvolvimento)

#### `operation`
- `"addtoken"` - Adiciona um novo token para o usuário
- `"removetoken"` - Remove um token existente do usuário

#### `onpremiseProfile` (quando `type` é `"onpremise"`)
- `"IOS"` - Perfil para dispositivos iOS
- `"Android"` - Perfil para dispositivos Android
- `"Desktop"` - Perfil para aplicações desktop

---

## 3. Fluxo do Workflow - Step by Step

### 3.1 Fluxo Geral

O workflow TIM-RSA executa os seguintes passos:

1. **Start** - Inicialização do workflow
2. **Verify User** - Validação do usuário (matrícula)
3. **Roteamento por Type** - Direcionamento para Cloud ou OnPremise
4. **Validações Específicas** - Validações de perfil/operação
5. **Execução da Operação** - Adição ou remoção de token
6. **Write Audit** - Registro de auditoria
7. **Stop** - Finalização

### 3.2 Step: Verify User

**Objetivo:** Validar se a matrícula fornecida existe no IdentityIQ e se o usuário não está bloqueado.

**Validações Realizadas:**
1. Verifica se o parâmetro `user` foi fornecido (não nulo e não vazio)
2. Busca a identidade no IdentityIQ pelo atributo `att_usuario` (matrícula)
3. Verifica se a identidade existe
4. Verifica se a identidade não está na blacklist (`tim_block_list != "true"`)
5. Armazena o `identityName` no workflow para uso posterior

**Transições:**
- Se `type` é `"cloud"` → vai para step **Cloud**
- Se `type` é `"onpremise"` → vai para step **OnPremise**
- Se `type` é inválido → vai para step **Type Invalid** (erro)

**Possíveis Erros:**
- `"Parameter validation error: required parameter 'user' is missing or empty."`
- `"Identity error: identity not found with matricula: {matricula}"`
- `"Identity error: identity is blocked (blacklist) for matricula: {matricula}"`

### 3.3 Fluxo OnPremise

#### Step: OnPremise

**Objetivo:** Receber o fluxo para operações OnPremise e validar a operação.

**Validações Realizadas:**
- Valida se `operation` é `"addtoken"` ou `"removetoken"`

**Transições:**
- Se `operation` é `"addtoken"` → vai para step **Verify Profile**
- Se `operation` é `"removetoken"` → vai para step **Onpremise - Token Remove**
- Se `operation` é inválido → vai para step **Onpremise Operation Invalid** (erro)

#### Step: Verify Profile

**Objetivo:** Validar o perfil do dispositivo fornecido.

**Validações Realizadas:**
1. Verifica se o parâmetro `profile` foi fornecido (não nulo e não vazio)
2. Valida se o perfil é um ENUM válido: `"IOS"`, `"Desktop"` ou `"Android"` (case-insensitive)
3. Normaliza o perfil para o formato correto

**Transições:**
- Se válido → vai para step **Onpremise - Token Add**
- Se inválido → lança GeneralException

**Possíveis Erros:**
- `"Parameter validation error: required parameter 'profile' is missing or empty. Valid values: 'IOS', 'Desktop', or 'Android'."`
- `"Parameter validation error: invalid value for 'profile'. Valid values: 'IOS', 'Desktop', or 'Android'. Provided value: {profile}"`

#### Step: Onpremise - Token Add

**Objetivo:** Adicionar um token RSA no ambiente OnPremise.

**Método Executado:** `addTokenOnpremise(user, profile, deviceSerialNumber)` da Rule `TIM-Rule-RSA-Library-Onpremise`

**Fluxo Interno (4 Steps):**

**STEP 1 - Autenticação:**
- Obtém token de autenticação através do método `getAuthTokenOnpremise()`
- Faz POST para `/auth/authn` com credenciais do Custom `TIM-Custom-RSA-Credentials`
- Extrai `authenticationToken` da resposta XML

**STEP 2 - Localizar e Remover Tokens Antigos:**
- Chama `getTokenOnpremise(user)` para buscar tokens existentes do usuário
- Para cada token encontrado, chama `removeTokenOnpremise(tokenSN)` para remover
- Faz POST para `/am8/token/unassign/{tokenSN}`

**STEP 3 - Criar Novo Token:**
- Faz GET para `/am8/user/assignNext/{userid}/software`
- Extrai `TokenSerialNumber` da resposta XML

**STEP 4 - Atribuir Perfil ao Token:**
- Faz PUT para `/am8/token/update/{tokenSerialNumber}`
- Envia XML com perfil (IOS/Android/Desktop) e DeviceSerialNumber (se fornecido)
- Para Desktop, inclui configuração de distribuição STDID
- Extrai `activationUrl` da resposta XML

**Retorno:**
- Retorna `activationUrl` (formato: `com.rsa.securid://ctf?ctfData=...`)
- Armazenado na variável `activationUrl` do workflow

**Transições:**
- Sempre → vai para step **Write Audit**

#### Step: Onpremise - Token Remove

**Objetivo:** Remover todos os tokens RSA do usuário no ambiente OnPremise.

**Fluxo:**
1. Chama `getTokenOnpremise(user)` para buscar todos os tokens do usuário
2. Para cada token encontrado, chama `removeTokenOnpremise(tokenSN)`
3. Faz POST para `/am8/token/unassign/{tokenSN}` de cada token

**Validações:**
- Se nenhum token for encontrado, retorna sem erro (não é considerado erro)

**Transições:**
- Sempre → vai para step **Write Audit**

### 3.4 Fluxo Cloud

#### Step: Cloud

**Objetivo:** Receber o fluxo para operações Cloud e validar a operação.

**Validações Realizadas:**
- Valida se `operation` é `"addtoken"` ou `"removetoken"`

**Transições:**
- Se `operation` é `"addtoken"` → vai para step **Cloud - Token Add**
- Se `operation` é `"removetoken"` → vai para step **Cloud - Token Remove**
- Se `operation` é inválido → vai para step **Cloud Operation Invalid** (erro)

#### Step: Cloud - Token Add

**Objetivo:** Adicionar um token RSA no ambiente Cloud.

**Métodos Executados (em sequência):**
1. `updatePingDirRSA(identityName, "enable")` - Habilita atributos RSA no Ping Directory
2. `removeTokenCloud()` - Remove tokens existentes do usuário
3. `addTokenCloud(user, emailToSend)` - Adiciona novo token

**Fluxo Detalhado:**

**updatePingDirRSA (Ping Directory):**
- Busca link do Ping Directory do usuário
- Valida se o link existe e está ativo
- Cria ProvisioningPlan para atualizar atributos:
  - `casSecurIdDN`: DN do usuário
  - `casSecurIdFlag`: "Active"
  - `casSecurIdMail`: Email no formato `{matricula}@timbrasil.com.br`
- Executa o provisioning

**removeTokenCloud:**
- Obtém token OAuth através de `getAuthTokenCloud()` (JWT Client Assertion)
- POST para `/users/lookup` para obter userId
- GET para `/users/{userId}/devices` para listar dispositivos
- DELETE para `/users/{userId}/devices/{deviceId}` de cada dispositivo

**addTokenCloud:**
- Obtém token OAuth (reutiliza ou obtém novo)
- POST para `/users/generateVerifyCode/enroll`
- Body JSON com email e email customizado
- Retorna resposta da API

**Transições:**
- Sempre → vai para step **Write Audit**

#### Step: Cloud - Token Remove

**Objetivo:** Remover todos os tokens RSA do usuário no ambiente Cloud.

**Métodos Executados (em sequência):**
1. `updatePingDirRSA(identityName, "disable")` - Desabilita atributos RSA no Ping Directory
2. `removeTokenCloud()` - Remove todos os tokens do usuário

**Fluxo Detalhado:**

**updatePingDirRSA (disable):**
- Busca link do Ping Directory do usuário
- Cria ProvisioningPlan para atualizar:
  - `casSecurIdFlag`: "Inactive"
- Executa o provisioning

**removeTokenCloud:**
- Mesmo fluxo descrito em "Cloud - Token Add"

**Transições:**
- Sempre → vai para step **Write Audit**

### 3.5 Step: Write Audit

**Objetivo:** Registrar evento de auditoria no IdentityIQ.

**Ações:**
- Normaliza variáveis (user, type, operation) para formato padronizado
- Cria AuditEvent com:
  - `target`: matrícula do usuário
  - `accountName`: matrícula do usuário
  - `action`: "RSA"
  - `attributeName`: "type"
  - `attributeValue`: tipo (onpremise/cloud)
  - `string1`: operação executada
  - `string2`: requestID
- Registra evento através de `Auditor.log(event)`
- Commita transação

**Transições:**
- Sempre → vai para step **Stop**

### 3.6 Steps de Erro

#### Type Invalid
**Erro:** Tipo inválido fornecido
```
"Parameter validation error: invalid value for 'Type'. Expected 'Cloud' or 'OnPremise'. Provided value: {type}"
```

#### Onpremise Operation Invalid
**Erro:** Operação inválida para OnPremise
```
"Parameter validation error: invalid value for 'Operation'. Allowed operations for Type 'OnPremise' are: 'addToken' or 'removeToken'. Provided value: {operation}"
```

#### Cloud Operation Invalid
**Erro:** Operação inválida para Cloud
```
"Parameter validation error: invalid value for 'Operation'. Allowed operations for Type 'Cloud' are: 'addToken' or 'removeToken'. Provided value: {operation}"
```

---

## 4. Validações e Regras de Negócio

### 4.1 Validações Gerais

#### Validação de Usuário
- Matrícula (`user`) é obrigatória
- Matrícula deve existir no IdentityIQ (atributo `att_usuario`)
- Identidade não pode estar bloqueada (`tim_block_list != "true"`)

#### Validação de Type
- `type` deve ser `"onpremise"` ou `"cloud"` (case-insensitive)
- Valores aceitos: `"onpremise"`, `"OnPremise"`, `"cloud"`, `"Cloud"`

#### Validação de Operation
- `operation` deve ser `"addtoken"` ou `"removetoken"` (case-insensitive)
- Valores aceitos: `"addtoken"`, `"addToken"`, `"removetoken"`, `"removeToken"`

### 4.2 Validações Específicas - OnPremise

#### Validação de Profile (para addtoken)
- `onpremiseProfile` é obrigatório quando `type` é `"onpremise"` e `operation` é `"addtoken"`
- Deve ser um ENUM válido: `"IOS"`, `"Android"` ou `"Desktop"` (case-insensitive)

#### Validação de DeviceSerialNumber
- Opcional
- Se fornecido, será utilizado na criação do token

### 4.3 Validações Específicas - Cloud

#### Validação de EmailToSend (para addtoken)
- `emailToSend` é obrigatório quando `type` é `"cloud"` e `operation` é `"addtoken"`
- Deve ser um email válido

#### Validação de Ping Directory
- Identidade deve ter conta no Ping Directory
- Conta do Ping Directory deve estar ativa (para operação enable)

---

## 5. Estrutura da Resposta

### 5.1 Resposta de Sucesso (addtoken - OnPremise)

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

### 5.2 Resposta de Sucesso (removetoken)

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
        "type": "onpremise",
        "user": "T3622753",
        "operation": "removetoken",
        "onpremiseProfile": "Android"
    },
    "complete": false,
    "success": false,
    "retry": false,
    "failure": false
}
```

### 5.3 Resposta de Erro

```json
{
    "status": null,
    "requestID": null,
    "warnings": null,
    "errors": [
        "Status : failed\nAn unexpected error occurred: sailpoint.tools.GeneralException: The application script threw an exception: sailpoint.tools.GeneralException: Identidade não encontrada com matrícula: T36227533 BSF info: script at line: 0 column: columnNo\n"
    ],
    "retryWait": 0,
    "metaData": null,
    "attributes": {
        "type": "onpremise",
        "user": "T36227533",
        "operation": "addtoken",
        "onpremiseProfile": "IOS"
    },
    "complete": false,
    "success": false,
    "retry": false,
    "failure": false
}
```

### 5.4 Campos Importantes da Resposta

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `requestID` | String | Identificador único da requisição gerado pelo IdentityIQ |
| `attributes.activationUrl` | String | URL de ativação do token RSA (apenas para addtoken OnPremise). Formato: `com.rsa.securid://ctf?ctfData=...` |
| `attributes.user` | String | Matrícula do usuário processado |
| `attributes.operation` | String | Operação executada (addtoken ou removetoken) |
| `attributes.type` | String | Tipo de ambiente utilizado (onpremise ou cloud) |
| `attributes.onpremiseProfile` | String | Perfil do dispositivo utilizado (apenas OnPremise) |
| `attributes.identityName` | String | Nome da identidade no IdentityIQ |
| `errors` | Array/String | Campo de erros. Será `null` em caso de sucesso. Em caso de erro, conterá a mensagem de erro detalhada |

---

## 6. Tratamento de Erros

### 6.1 Tipos de Erros

#### Erros de Validação de Parâmetros

**Campos Obrigatórios Ausentes:**
- Erro: `"Parameter validation error: required parameter '{parametro}' is missing or empty."`
- Causa: Parâmetro obrigatório não foi fornecido ou está vazio
- Solução: Verificar se todos os parâmetros obrigatórios estão presentes na requisição

**Valores Inválidos:**
- Erro: `"Parameter validation error: invalid value for '{parametro}'. Valid values: {...}. Provided value: {valor}"`
- Causa: Valor fornecido não está na lista de valores aceitos
- Solução: Verificar valores válidos na documentação e corrigir o parâmetro

#### Erros de Identidade

**Identidade Não Encontrada:**
- Erro: `"Identity error: identity not found with matricula: {matricula}"`
- Causa: Matrícula fornecida não existe no IdentityIQ
- Solução: Verificar se a matrícula está correta e se o usuário existe no sistema

**Identidade Bloqueada:**
- Erro: `"Identity error: identity is blocked (blacklist) for matricula: {matricula}"`
- Causa: Usuário está na lista de bloqueio (`tim_block_list == "true"`)
- Solução: Remover o usuário da lista de bloqueio antes de processar

**Identidade Inativa (Cloud):**
- Erro: `"Identity error: identity '{identityName}' is inactive. Cannot enable RSA attributes for inactive identity."`
- Causa: Identidade está inativa no IdentityIQ
- Solução: Ativar a identidade antes de processar

#### Erros de Conta

**Conta Ping Directory Não Encontrada:**
- Erro: `"Account error: identity '{identityName}' has no Ping Directory account with matricula '{matricula}'."`
- Causa: Identidade não possui conta no Ping Directory
- Solução: Verificar se a conta existe no Ping Directory

**Conta Ping Directory Desabilitada:**
- Erro: `"Account error: Ping Directory account is disabled for link '{nativeIdentity}'. Cannot enable RSA attributes for disabled account."`
- Causa: Conta do Ping Directory está desabilitada
- Solução: Habilitar a conta antes de processar

#### Erros de Configuração

**Custom Object Não Encontrado:**
- Erro: `"Configuration error: custom object 'TIM-Custom-RSA-Credentials' not found."`
- Causa: Objeto Custom não existe no IdentityIQ
- Solução: Criar/configurar o objeto Custom com as credenciais necessárias

**Atributos de Configuração Ausentes:**
- Erro: `"Configuration error: attribute '{atributo}' not found in Custom 'TIM-Custom-RSA-Credentials'."`
- Causa: Atributo necessário não está configurado no Custom
- Solução: Verificar e configurar todos os atributos necessários no Custom

#### Erros de API Externa

**Erro de Autenticação RSA OnPremise:**
- Erro: `"Onpremise API error: failed to obtain authentication token."`
- Causa: Credenciais RSA OnPremise inválidas ou servidor inacessível
- Solução: Verificar credenciais e conectividade com servidor RSA

**Erro de Autenticação RSA Cloud:**
- Erro: `"Authentication error: OAuth access token could not be obtained."`
- Causa: Problema na geração do JWT ou na autenticação OAuth
- Solução: Verificar configuração de chaves privadas e credenciais OAuth

**Erro de Operação RSA:**
- Erro: `"Onpremise API error: failed to assign next token via assignNext."`
- Causa: Erro na API RSA ao criar/remover token
- Solução: Verificar logs do servidor RSA e status da API

**Erro de Provisioning:**
- Erro: `"Provisioning error: failed to execute provisioning plan."`
- Causa: Erro ao executar provisioning no Ping Directory
- Solução: Verificar conectividade e configuração do conector Ping Directory

### 6.2 Como Interpretar Erros

1. **Sempre verificar o campo `errors` na resposta** - Este campo contém a mensagem de erro detalhada
2. **Verificar o `requestID`** - Utilizar para rastreamento e consulta de logs
3. **Verificar os logs do IdentityIQ** - Mensagens de log incluem prefixos como "Tim-RSA-" para facilitar busca
4. **Consultar atributos na resposta** - Os atributos processados são retornados mesmo em caso de erro

---

## 7. Exemplos de Requisição

### 7.1 Adicionar Token OnPremise - iOS

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

### 7.2 Adicionar Token OnPremise - Android

```json
{
    "workflowArgs": {
        "requestId": "TK002",
        "type": "onpremise",
        "operation": "addtoken",
        "user": "T3622753",
        "onpremiseProfile": "Android"
    }
}
```

### 7.3 Adicionar Token OnPremise - Desktop

```json
{
    "workflowArgs": {
        "requestId": "TK003",
        "type": "onpremise",
        "operation": "addtoken",
        "user": "T3622753",
        "onpremiseProfile": "Desktop",
        "deviceSerialNumber": "DEVICE123456"
    }
}
```

### 7.4 Remover Token OnPremise

```json
{
    "workflowArgs": {
        "requestId": "TK004",
        "type": "onpremise",
        "operation": "removetoken",
        "user": "T3622753",
        "onpremiseProfile": "Android"
    }
}
```

### 7.5 Adicionar Token Cloud

```json
{
    "workflowArgs": {
        "requestId": "TK005",
        "type": "cloud",
        "operation": "addtoken",
        "user": "T3622753",
        "emailToSend": "usuario@timbrasil.com.br"
    }
}
```

### 7.6 Remover Token Cloud

```json
{
    "workflowArgs": {
        "requestId": "TK006",
        "type": "cloud",
        "operation": "removetoken",
        "user": "T3622753"
    }
}
```

---

## 8. Configuração e Dependências

### 8.1 Objetos Custom Necessários

#### TIM-Custom-RSA-Credentials

Este objeto Custom deve conter as credenciais e configurações para ambos os ambientes (OnPremise e Cloud).

**Estrutura OnPremise:**
```json
{
  "onpremise": {
    "url": "https://rsa-server.example.com",
    "host": "rsa-server.example.com",
    "user": "usuario_api",
    "pass": "senha_api"
  }
}
```

**Estrutura Cloud:**
```json
{
  "cloud": {
    "url": "https://timbrasil.auth.securid.com",
    "urlAuth": "https://timbrasil.auth.securid.com/oauth/token",
    "clientid": "client_id_oauth",
    "kid": "key_id",
    "privateKey": {
      "kty": "RSA",
      "use": "sig",
      "alg": "RS256",
      "kid": "key_id",
      "n": "modulus_base64url",
      "e": "exponent_base64url",
      "d": "private_exponent_base64url",
      "p": "prime1_base64url",
      "q": "prime2_base64url",
      "dp": "exponent1_base64url",
      "dq": "exponent2_base64url",
      "qi": "coefficient_base64url"
    }
  }
}
```

### 8.2 Permissões Necessárias

**Usuário de Autenticação:**
- Permissão para executar workflows
- Permissão de acesso ao workflow TIM-RSA
- Permissão para operações de gerenciamento de tokens RSA

**Conta de Serviço RSA:**
- OnPremise: Credenciais de API com permissões para criar/remover tokens
- Cloud: Client ID OAuth com permissões para gerenciar tokens de usuários

### 8.3 Aplicações Necessárias

**Ping Directory (apenas Cloud):**
- Aplicação Ping Directory configurada no IdentityIQ
- Contas de usuários devem existir no Ping Directory
- Atributos RSA devem estar mapeados no schema:
  - `casSecurIdDN`
  - `casSecurIdFlag`
  - `casSecurIdMail`

---

## 9. Logs e Auditoria

### 9.1 Logs do Workflow

O workflow gera logs detalhados com os seguintes prefixos:
- `Tim-RSA-Start` - Inicialização
- `Tim-RSA-VerifyUser` - Validação de usuário
- `Tim-RSA-OnPremise` - Operações OnPremise
- `Tim-RSA-Cloud` - Operações Cloud
- `Tim-RSA-Library-*` - Métodos das Rules (Onpremise, Cloud, PingDirectory)

### 9.2 Eventos de Auditoria

Todos os eventos são registrados no IdentityIQ através do step **Write Audit** com as seguintes informações:
- **Target**: Matrícula do usuário
- **Action**: "RSA"
- **Attribute Name**: "type"
- **Attribute Value**: Tipo de ambiente (onpremise/cloud)
- **String1**: Operação executada (addtoken/removetoken)
- **String2**: RequestID fornecido na requisição

### 9.3 Consulta de Logs

Para consultar logs:
1. Acessar IdentityIQ → Monitor → Tasks
2. Buscar pelo `requestID` retornado na resposta
3. Verificar detalhes da execução do workflow

---

## 10. Troubleshooting

### 10.1 Problemas Comuns

#### Erro: "Identity not found"
**Solução:**
- Verificar se a matrícula está correta
- Verificar se o usuário existe no IdentityIQ
- Verificar se o atributo `att_usuario` está populado corretamente

#### Erro: "Identity is blocked"
**Solução:**
- Verificar atributo `tim_block_list` da identidade
- Remover bloqueio se necessário

#### Erro: "Ping Directory account not found" (Cloud)
**Solução:**
- Verificar se a aplicação Ping Directory está configurada
- Verificar se o usuário possui conta no Ping Directory
- Verificar se a matrícula corresponde ao nativeIdentity do link

#### Erro: "Authentication token could not be obtained"
**Solução:**
- Verificar credenciais no Custom `TIM-Custom-RSA-Credentials`
- Verificar conectividade com servidor RSA
- Verificar se as chaves privadas (Cloud) estão configuradas corretamente

#### Erro: "Failed to assign next token"
**Solução:**
- Verificar logs do servidor RSA
- Verificar se o usuário já possui tokens e se a remoção foi bem-sucedida
- Verificar permissões da conta de serviço RSA

### 10.2 Checklist de Verificação

Antes de reportar problemas, verificar:
- [ ] Todos os parâmetros obrigatórios foram fornecidos
- [ ] Valores estão no formato correto (case-insensitive)
- [ ] Usuário existe no IdentityIQ
- [ ] Usuário não está bloqueado
- [ ] Objeto Custom está configurado corretamente
- [ ] Conectividade com servidor RSA está ok
- [ ] Permissões do usuário de autenticação estão corretas
- [ ] Logs do IdentityIQ foram consultados

---

## 11. Collection Postman

A collection do Postman está disponível no arquivo `TIM-RSA.postman_collection.json` e inclui:

1. **Adicionar Token (addtoken)** - Exemplo para OnPremise iOS
2. **Remover Token (removetoken)** - Exemplo para OnPremise Android

**Variáveis da Collection:**
- `base_url`: URL base do IdentityIQ
- `workflow_name`: Nome do workflow (TIM-RSA)
- `username`: Usuário para autenticação Basic Auth
- `password`: Senha para autenticação Basic Auth

**Nota:** Atualizar as variáveis antes de usar a collection.

---

## 12. Referências

### 12.1 Rules Utilizadas

- **TIM-Rule-RSA-Library-PingDirectory**: Métodos para atualização de atributos RSA no Ping Directory
- **TIM-Rule-RSA-Library-Onpremise**: Métodos para operações com RSA OnPremise
- **TIM-Rule-RSA-Library-Cloud**: Métodos para operações com RSA Cloud

### 12.2 Endpoints RSA

**OnPremise:**
- Autenticação: `{urlBase}/auth/authn`
- Criar Token: `{urlBase}/am8/user/assignNext/{userid}/software`
- Atualizar Token: `{urlBase}/am8/token/update/{tokenSerialNumber}`
- Buscar Tokens: `{urlBase}/am8/token/search/UserID/{userid}?searchType=equals`
- Remover Token: `{urlBase}/am8/token/unassign/{tokenSerialNumber}`

**Cloud:**
- OAuth Token: `https://timbrasil.auth.securid.com/oauth/token`
- Lookup User: `{urlBase}/AdminInterface/restapi/v1/users/lookup`
- List Devices: `{urlBase}/AdminInterface/restapi/v2/users/{userId}/devices`
- Remove Device: `{urlBase}/AdminInterface/restapi/v1/users/{userId}/devices/{deviceId}`
- Enroll Token: `{urlBase}/AdminInterface/restapi/v1/users/generateVerifyCode/enroll`

---

## 13. Limitações e Considerações

### 13.1 Ambiente Cloud

- O suporte para ambiente Cloud está em desenvolvimento
- Algumas funcionalidades podem estar sujeitas a alterações

### 13.2 Performance

- Operações OnPremise podem levar alguns segundos devido a múltiplas chamadas de API
- Operações Cloud podem levar mais tempo devido ao processo de OAuth e múltiplas chamadas

### 13.3 Token OnPremise

- Ao adicionar um token, tokens antigos são automaticamente removidos
- O `activationUrl` retornado deve ser usado para ativação do token no dispositivo

### 13.4 Token Cloud

- Ao adicionar um token, tokens antigos são automaticamente removidos
- Um código de verificação é enviado por email
- O processo de ativação é diferente do OnPremise

---

## 14. Changelog e Versões

**Versão Atual:** 1.0

**Data de Criação:** Baseado na análise do código atual

**Última Modificação do Workflow:** 1768247072185 (timestamp)

---

## 15. Contatos e Suporte

Para questões técnicas ou problemas:
1. Consultar logs do IdentityIQ
2. Verificar esta documentação
3. Verificar configurações do Custom Object
4. Contatar equipe de suporte com o `requestID` da requisição

