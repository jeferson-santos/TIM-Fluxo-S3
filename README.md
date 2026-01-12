# Documentação de Integração - API de Gerenciamento de Tokens RSA

## 1. Visão Geral

Esta API permite adicionar ou remover tokens RSA para usuários através de uma integração REST com o IdentityIQ.

**Endpoint Base:**
```
POST http://smartid.internal.timbrasil.com.br/identityiq/rest/workflows/TIM-RSA/launch
```

**Ambientes Suportados:**
- **OnPremise**: Ambiente on-premise
- **Cloud**: Ambiente cloud

---

## 2. Autenticação

A API utiliza **HTTP Basic Authentication**.

### 2.1 Header de Autenticação

```
Authorization: Basic <base64(user:password)>
Content-Type: application/json
```

### 2.2 Exemplo de Autenticação

**JavaScript:**
```javascript
const credentials = Buffer.from(`${username}:${password}`).toString('base64');
headers: {
  'Authorization': `Basic ${credentials}`,
  'Content-Type': 'application/json'
}
```

**Python:**
```python
import base64
credentials = base64.b64encode(f"{username}:{password}".encode()).decode()
headers = {
    'Authorization': f'Basic {credentials}',
    'Content-Type': 'application/json'
}
```

**cURL:**
```bash
curl -X POST \
  -u username:password \
  -H "Content-Type: application/json" \
  -d @request.json \
  http://smartid.internal.timbrasil.com.br/identityiq/rest/workflows/TIM-RSA/launch
```

---

## 3. Estrutura da Requisição

### 3.1 Formato JSON

Todas as requisições devem seguir o formato:

```json
{
    "workflowArgs": {
        "requestId": "string",
        "type": "string",
        "operation": "string",
        "user": "string",
        ...
    }
}
```

### 3.2 Parâmetros

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| `requestId` | String | ✅ | ID único para rastreamento (ex: "TK001", número de ticket) |
| `type` | String | ✅ | Ambiente: `"onpremise"` ou `"cloud"` |
| `operation` | String | ✅ | Operação: `"addtoken"` ou `"removetoken"` |
| `user` | String | ✅ | Matrícula do usuário (ex: "T3622753") |
| `profile` | String | ⚠️ | Perfil do dispositivo. Obrigatório apenas se `type="onpremise"` E `operation="addtoken"`. Valores: `"IOS"`, `"Android"`, `"Desktop"` |
| `emailToSend` | String | ⚠️ | Email para envio. Obrigatório apenas se `type="cloud"` E `operation="addtoken"` |
| `deviceSerialNumber` | String | ❌ | Número de série do dispositivo (opcional, apenas OnPremise) |

---

## 4. Operações Disponíveis

### 4.1 Adicionar Token OnPremise

Adiciona um novo token RSA no ambiente OnPremise.

**Parâmetros Obrigatórios:**
- `requestId`
- `type: "onpremise"`
- `operation: "addtoken"`
- `user`
- `profile` (IOS, Android ou Desktop)

**Exemplo de Requisição:**

```json
{
    "workflowArgs": {
        "requestId": "TK001",
        "type": "onpremise",
        "operation": "addtoken",
        "user": "T3622753",
        "profile": "IOS"
    }
}
```

**Resposta de Sucesso:**

```json
{
    "status": null,
    "requestID": "0a9736459b6c1b13819b8ff522d51e09",
    "warnings": null,
    "errors": null,
    "retryWait": 0,
    "attributes": {
        "identityName": "09257316920",
        "type": "onpremise",
        "activationUrl": "com.rsa.securid://ctf?ctfData=200193850501325125304724240511562112550013127207153660245152163270154577437524233",
        "user": "T3622753",
        "operation": "addtoken",
        "profile": "IOS"
    },
    "complete": false,
    "success": false
}
```

**Campo Importante:** `attributes.activationUrl` - URL para ativação do token no dispositivo.

---

### 4.2 Remover Token OnPremise

Remove todos os tokens RSA do usuário no ambiente OnPremise.

**Parâmetros Obrigatórios:**
- `requestId`
- `type: "onpremise"`
- `operation: "removetoken"`
- `user`

**Exemplo de Requisição:**

```json
{
    "workflowArgs": {
        "requestId": "TK002",
        "type": "onpremise",
        "operation": "removetoken",
        "user": "T3622753"
    }
}
```

**Resposta de Sucesso:**

```json
{
    "status": null,
    "requestID": "0a9736459b6c1b13819b8ff522d51e09",
    "warnings": null,
    "errors": null,
    "retryWait": 0,
    "attributes": {
        "identityName": "09257316920",
        "type": "onpremise",
        "user": "T3622753",
        "operation": "removetoken"
    },
    "complete": false,
    "success": false
}
```

---

### 4.3 Adicionar Token Cloud

Adiciona um novo token RSA no ambiente Cloud.

**Parâmetros Obrigatórios:**
- `requestId`
- `type: "cloud"`
- `operation: "addtoken"`
- `user`
- `emailToSend`

**Exemplo de Requisição:**

```json
{
    "workflowArgs": {
        "requestId": "TK003",
        "type": "cloud",
        "operation": "addtoken",
        "user": "T3622753",
        "emailToSend": "usuario@timbrasil.com.br"
    }
}
```

**Resposta de Sucesso:**

```json
{
    "status": null,
    "requestID": "0a9736459b6c1b13819b8ff522d51e09",
    "warnings": null,
    "errors": null,
    "retryWait": 0,
    "attributes": {
        "identityName": "09257316920",
        "type": "cloud",
        "user": "T3622753",
        "operation": "addtoken"
    },
    "complete": false,
    "success": false
}
```

---

### 4.4 Remover Token Cloud

Remove todos os tokens RSA do usuário no ambiente Cloud.

**Parâmetros Obrigatórios:**
- `requestId`
- `type: "cloud"`
- `operation: "removetoken"`
- `user`

**Exemplo de Requisição:**

```json
{
    "workflowArgs": {
        "requestId": "TK004",
        "type": "cloud",
        "operation": "removetoken",
        "user": "T3622753"
    }
}
```

**Resposta de Sucesso:**

```json
{
    "status": null,
    "requestID": "0a9736459b6c1b13819b8ff522d51e09",
    "warnings": null,
    "errors": null,
    "retryWait": 0,
    "attributes": {
        "identityName": "09257316920",
        "type": "cloud",
        "user": "T3622753",
        "operation": "removetoken"
    },
    "complete": false,
    "success": false
}
```

---

## 5. Tratamento de Erros

### 5.1 Estrutura de Resposta de Erro

Quando ocorre um erro, a resposta terá a seguinte estrutura:

```json
{
    "status": null,
    "requestID": null,
    "warnings": null,
    "errors": [
        "Status : failed\nAn unexpected error occurred: sailpoint.tools.GeneralException: <mensagem de erro detalhada>\n"
    ],
    "retryWait": 0,
    "attributes": {
        "type": "onpremise",
        "user": "T3622753",
        "operation": "addtoken"
    },
    "complete": false,
    "success": false
}
```

**Importante:** Sempre verificar o campo `errors`. Se `errors` for `null`, a operação foi bem-sucedida.

### 5.2 Códigos HTTP

| Código | Descrição |
|--------|-----------|
| `200` | Requisição processada (sucesso ou erro será indicado no campo `errors`) |
| `401` | Não autorizado (credenciais inválidas) |
| `403` | Proibido (sem permissões) |
| `404` | Endpoint não encontrado |
| `500` | Erro interno do servidor |

### 5.3 Mensagens de Erro Comuns

#### 5.3.1 Erros de Validação de Parâmetros

**Campo Obrigatório Ausente:**
```
Parameter validation error: required parameter 'user' is missing or empty.
```
**Solução:** Verificar se todos os parâmetros obrigatórios foram fornecidos.

**Valor Inválido para Type:**
```
Parameter validation error: invalid value for 'Type'. Expected 'Cloud' or 'OnPremise'. Provided value: {valor}
```
**Solução:** Usar `"onpremise"` ou `"cloud"` (case-insensitive).

**Valor Inválido para Operation:**
```
Parameter validation error: invalid value for 'Operation'. Allowed operations for Type 'OnPremise' are: 'addToken' or 'removeToken'. Provided value: {valor}
```
**Solução:** Usar `"addtoken"` ou `"removetoken"` (case-insensitive).

**Perfil Inválido:**
```
Parameter validation error: invalid value for 'profile'. Valid values: 'IOS', 'Desktop', or 'Android'. Provided value: {valor}
```
**Solução:** Usar `"IOS"`, `"Android"` ou `"Desktop"` (case-insensitive).

**Perfil Obrigatório Ausente:**
```
Parameter validation error: required parameter 'profile' is missing or empty. Valid values: 'IOS', 'Desktop', or 'Android'.
```
**Solução:** Fornecer o parâmetro `profile` quando `type="onpremise"` e `operation="addtoken"`.

#### 5.3.2 Erros de Identidade

**Identidade Não Encontrada:**
```
Identity error: identity not found with matricula: {matricula}
```
**Causa:** A matrícula fornecida não existe no IdentityIQ.

**Solução:**
- Verificar se a matrícula está correta
- Confirmar que o usuário existe no sistema
- Verificar o atributo `att_usuario` da identidade

**Identidade Bloqueada:**
```
Identity error: identity is blocked (blacklist) for matricula: {matricula}
```
**Causa:** O usuário está na lista de bloqueio.

**Solução:**
- Remover o bloqueio no IdentityIQ (atributo `tim_block_list`)
- Verificar com a equipe de administração

**Identidade Inativa (Cloud):**
```
Identity error: identity '{identityName}' is inactive. Cannot enable RSA attributes for inactive identity.
```
**Causa:** A identidade está inativa no IdentityIQ.

**Solução:** Ativar a identidade antes de processar.

#### 5.3.3 Erros de Conta (Cloud)

**Conta Ping Directory Não Encontrada:**
```
Account error: identity '{identityName}' has no Ping Directory account with matricula '{matricula}'.
```
**Causa:** O usuário não possui conta no Ping Directory.

**Solução:**
- Verificar se a aplicação Ping Directory está configurada
- Confirmar que o usuário possui conta no Ping Directory
- Verificar se a matrícula corresponde ao nativeIdentity do link

**Conta Ping Directory Desabilitada:**
```
Account error: Ping Directory account is disabled for link '{nativeIdentity}'. Cannot enable RSA attributes for disabled account.
```
**Causa:** A conta do Ping Directory está desabilitada.

**Solução:** Habilitar a conta do Ping Directory antes de processar.

#### 5.3.4 Erros de Autenticação/API Externa

**Erro de Autenticação RSA OnPremise:**
```
Onpremise API error: failed to obtain authentication token.
```
**Causa:** Problema na autenticação com o servidor RSA OnPremise.

**Solução:**
- Verificar credenciais no Custom Object `TIM-Custom-RSA-Credentials`
- Verificar conectividade com o servidor RSA
- Verificar se o servidor RSA está acessível

**Erro de Autenticação RSA Cloud:**
```
Authentication error: OAuth access token could not be obtained.
```
**Causa:** Problema na autenticação OAuth com RSA Cloud.

**Solução:**
- Verificar configuração de chaves privadas no Custom Object
- Verificar credenciais OAuth
- Verificar conectividade com RSA Cloud

**Erro ao Criar Token:**
```
Onpremise API error: failed to assign next token via assignNext.
```
**Causa:** Erro na API RSA ao criar token.

**Solução:**
- Verificar logs do servidor RSA
- Verificar permissões da conta de serviço RSA
- Verificar se o usuário já possui tokens (serão removidos automaticamente)

**Erro de Provisioning:**
```
Provisioning error: failed to execute provisioning plan.
```
**Causa:** Erro ao executar provisioning no Ping Directory (Cloud).

**Solução:**
- Verificar conectividade com Ping Directory
- Verificar configuração do conector Ping Directory
- Verificar permissões de provisioning

---

## 6. Exemplos de Implementação

### 6.1 JavaScript/Node.js

```javascript
const axios = require('axios');
const https = require('https');

async function addTokenOnPremise(user, profile, requestId) {
    const username = 'seu_usuario';
    const password = 'sua_senha';
    const credentials = Buffer.from(`${username}:${password}`).toString('base64');
    
    const url = 'http://smartid.internal.timbrasil.com.br/identityiq/rest/workflows/TIM-RSA/launch';
    
    const payload = {
        workflowArgs: {
            requestId: requestId,
            type: 'onpremise',
            operation: 'addtoken',
            user: user,
            profile: profile
        }
    };
    
    try {
        const response = await axios.post(url, payload, {
            headers: {
                'Authorization': `Basic ${credentials}`,
                'Content-Type': 'application/json'
            }
        });
        
        if (response.data.errors && response.data.errors.length > 0) {
            throw new Error(response.data.errors[0]);
        }
        
        return {
            success: true,
            requestID: response.data.requestID,
            activationUrl: response.data.attributes.activationUrl
        };
    } catch (error) {
        return {
            success: false,
            error: error.response?.data?.errors?.[0] || error.message
        };
    }
}

// Uso
addTokenOnPremise('T3622753', 'IOS', 'TK001')
    .then(result => {
        if (result.success) {
            console.log('Token criado:', result.activationUrl);
        } else {
            console.error('Erro:', result.error);
        }
    });
```

### 6.2 Python

```python
import requests
import base64
import json

def add_token_on_premise(user, profile, request_id):
    username = 'seu_usuario'
    password = 'sua_senha'
    credentials = base64.b64encode(f"{username}:{password}".encode()).decode()
    
    url = 'http://smartid.internal.timbrasil.com.br/identityiq/rest/workflows/TIM-RSA/launch'
    
    payload = {
        "workflowArgs": {
            "requestId": request_id,
            "type": "onpremise",
            "operation": "addtoken",
            "user": user,
            "profile": profile
        }
    }
    
    headers = {
        'Authorization': f'Basic {credentials}',
        'Content-Type': 'application/json'
    }
    
    try:
        response = requests.post(url, json=payload, headers=headers)
        response.raise_for_status()
        
        data = response.json()
        
        if data.get('errors'):
            return {
                'success': False,
                'error': data['errors'][0]
            }
        
        return {
            'success': True,
            'requestID': data.get('requestID'),
            'activationUrl': data.get('attributes', {}).get('activationUrl')
        }
    except requests.exceptions.RequestException as e:
        return {
            'success': False,
            'error': str(e)
        }

# Uso
result = add_token_on_premise('T3622753', 'IOS', 'TK001')
if result['success']:
    print(f"Token criado: {result['activationUrl']}")
else:
    print(f"Erro: {result['error']}")
```

### 6.3 Java

```java
import java.net.HttpURLConnection;
import java.net.URL;
import java.io.*;
import java.util.Base64;
import org.json.JSONObject;

public class RSATokenManager {
    
    private static final String BASE_URL = "http://smartid.internal.timbrasil.com.br/identityiq/rest/workflows/TIM-RSA/launch";
    private static final String USERNAME = "seu_usuario";
    private static final String PASSWORD = "sua_senha";
    
    public static Response addTokenOnPremise(String user, String profile, String requestId) {
        try {
            URL url = new URL(BASE_URL);
            HttpURLConnection conn = (HttpURLConnection) url.openConnection();
            conn.setRequestMethod("POST");
            conn.setRequestProperty("Content-Type", "application/json");
            
            // Autenticação Basic
            String auth = USERNAME + ":" + PASSWORD;
            String encodedAuth = Base64.getEncoder().encodeToString(auth.getBytes());
            conn.setRequestProperty("Authorization", "Basic " + encodedAuth);
            
            // Body
            JSONObject payload = new JSONObject();
            JSONObject workflowArgs = new JSONObject();
            workflowArgs.put("requestId", requestId);
            workflowArgs.put("type", "onpremise");
            workflowArgs.put("operation", "addtoken");
            workflowArgs.put("user", user);
            workflowArgs.put("profile", profile);
            payload.put("workflowArgs", workflowArgs);
            
            conn.setDoOutput(true);
            try (OutputStream os = conn.getOutputStream()) {
                byte[] input = payload.toString().getBytes("utf-8");
                os.write(input, 0, input.length);
            }
            
            int responseCode = conn.getResponseCode();
            BufferedReader in = new BufferedReader(new InputStreamReader(
                responseCode >= 200 && responseCode < 300 
                    ? conn.getInputStream() 
                    : conn.getErrorStream()
            ));
            
            StringBuilder response = new StringBuilder();
            String line;
            while ((line = in.readLine()) != null) {
                response.append(line);
            }
            in.close();
            
            JSONObject jsonResponse = new JSONObject(response.toString());
            
            if (jsonResponse.has("errors") && jsonResponse.get("errors") != null) {
                return new Response(false, jsonResponse.getJSONArray("errors").getString(0), null, null);
            }
            
            JSONObject attributes = jsonResponse.getJSONObject("attributes");
            return new Response(
                true,
                null,
                jsonResponse.getString("requestID"),
                attributes.optString("activationUrl", null)
            );
            
        } catch (Exception e) {
            return new Response(false, e.getMessage(), null, null);
        }
    }
    
    static class Response {
        boolean success;
        String error;
        String requestID;
        String activationUrl;
        
        Response(boolean success, String error, String requestID, String activationUrl) {
            this.success = success;
            this.error = error;
            this.requestID = requestID;
            this.activationUrl = activationUrl;
        }
    }
    
    // Uso
    public static void main(String[] args) {
        Response result = addTokenOnPremise("T3622753", "IOS", "TK001");
        if (result.success) {
            System.out.println("Token criado: " + result.activationUrl);
        } else {
            System.out.println("Erro: " + result.error);
        }
    }
}
```

---

## 7. Melhores Práticas

### 7.1 Tratamento de Erros

1. **Sempre verificar o campo `errors`** na resposta antes de considerar a operação como bem-sucedida
2. **Logar o `requestID`** para facilitar rastreamento e troubleshooting
3. **Implementar retry logic** para erros temporários (erros de conectividade)
4. **Validar parâmetros** antes de enviar a requisição
5. **Tratar erros específicos** de forma diferenciada (ex: identidade não encontrada vs. erro de conectividade)

### 7.2 Validação de Entrada

```javascript
function validateRequest(type, operation, user, profile, emailToSend) {
    const errors = [];
    
    if (!user || user.trim() === '') {
        errors.push('user é obrigatório');
    }
    
    if (type !== 'onpremise' && type !== 'cloud') {
        errors.push('type deve ser "onpremise" ou "cloud"');
    }
    
    if (operation !== 'addtoken' && operation !== 'removetoken') {
        errors.push('operation deve ser "addtoken" ou "removetoken"');
    }
    
    if (type === 'onpremise' && operation === 'addtoken') {
        if (!profile || !['IOS', 'Android', 'Desktop'].includes(profile)) {
            errors.push('profile é obrigatório e deve ser IOS, Android ou Desktop');
        }
    }
    
    if (type === 'cloud' && operation === 'addtoken') {
        if (!emailToSend || emailToSend.trim() === '') {
            errors.push('emailToSend é obrigatório para cloud addtoken');
        }
    }
    
    return errors;
}
```

### 7.3 Rastreamento

1. **Usar `requestId` significativo**: Utilize IDs que facilitem o rastreamento (ex: número de ticket, UUID, timestamp)
2. **Armazenar `requestID` retornado**: O `requestID` retornado pode ser usado para consultar logs no IdentityIQ
3. **Logar todas as requisições**: Mantenha logs das requisições e respostas para auditoria

### 7.4 Performance

1. **Timeout adequado**: Configure timeout apropriado (recomendado: 60-120 segundos)
2. **Não fazer requisições síncronas em loop**: Use processamento assíncrono para múltiplas requisições
3. **Implementar cache quando apropriado**: Cache informações que não mudam frequentemente

---

## 8. Checklist de Integração

Antes de fazer deploy em produção, verificar:

- [ ] Autenticação configurada corretamente (Basic Auth)
- [ ] Tratamento de erros implementado
- [ ] Validação de parâmetros antes do envio
- [ ] Logging implementado (requestId, erros)
- [ ] Timeout configurado adequadamente
- [ ] Testes para todos os cenários (addtoken/removetoken, onpremise/cloud)
- [ ] Tratamento de casos de erro específicos
- [ ] Documentação interna atualizada
- [ ] Credenciais seguras (não hardcoded)
- [ ] Tratamento de conexão perdida/reconexão

---

## 9. Suporte e Troubleshooting

### 9.1 Informações para Suporte

Ao reportar problemas, forneça:

1. **RequestID** retornado na resposta (se disponível)
2. **RequestId** enviado na requisição
3. **Matrícula do usuário** (`user`)
4. **Tipo e operação** (`type`, `operation`)
5. **Mensagem de erro completa** do campo `errors`
6. **Timestamp** da requisição
7. **Headers** enviados (sem credenciais)

### 9.2 Consulta de Logs

Os logs podem ser consultados no IdentityIQ:
1. Acessar: Monitor → Tasks
2. Buscar pelo `requestID` retornado na resposta
3. Verificar detalhes da execução do workflow

### 9.3 Problemas Comuns

| Problema | Verificar |
|----------|-----------|
| Erro 401 (Não autorizado) | Credenciais corretas? Permissões adequadas? |
| Identidade não encontrada | Matrícula existe? Atributo `att_usuario` correto? |
| Identidade bloqueada | Atributo `tim_block_list` está como "true"? |
| Perfil inválido | Usando IOS, Android ou Desktop? (case-insensitive) |
| Timeout | Servidor RSA acessível? Timeout configurado corretamente? |

---

## 10. Referência Rápida

### 10.1 Matriz de Parâmetros

| Operação | Type | Parâmetros Obrigatórios |
|----------|------|------------------------|
| addtoken | onpremise | requestId, type, operation, user, **profile** |
| removetoken | onpremise | requestId, type, operation, user |
| addtoken | cloud | requestId, type, operation, user, **emailToSend** |
| removetoken | cloud | requestId, type, operation, user |

### 10.2 Valores Aceitos

**type:** `"onpremise"`, `"cloud"` (case-insensitive)

**operation:** `"addtoken"`, `"removetoken"` (case-insensitive)

**profile:** `"IOS"`, `"Android"`, `"Desktop"` (case-insensitive)

---

## 11. Exemplos Completos

### 11.1 Adicionar Token OnPremise - Todos os Perfis

**iOS:**
```json
{
    "workflowArgs": {
        "requestId": "TK-IOS-001",
        "type": "onpremise",
        "operation": "addtoken",
        "user": "T3622753",
        "profile": "IOS"
    }
}
```

**Android:**
```json
{
    "workflowArgs": {
        "requestId": "TK-AND-001",
        "type": "onpremise",
        "operation": "addtoken",
        "user": "T3622753",
        "profile": "Android"
    }
}
```

**Desktop:**
```json
{
    "workflowArgs": {
        "requestId": "TK-DESK-001",
        "type": "onpremise",
        "operation": "addtoken",
        "user": "T3622753",
        "profile": "Desktop",
        "deviceSerialNumber": "DEVICE123456"
    }
}
```

### 11.2 Operações Cloud

**Adicionar Token Cloud:**
```json
{
    "workflowArgs": {
        "requestId": "TK-CLOUD-ADD-001",
        "type": "cloud",
        "operation": "addtoken",
        "user": "T3622753",
        "emailToSend": "usuario@timbrasil.com.br"
    }
}
```

**Remover Token Cloud:**
```json
{
    "workflowArgs": {
        "requestId": "TK-CLOUD-REM-001",
        "type": "cloud",
        "operation": "removetoken",
        "user": "T3622753"
    }
}
```

---

## 12. Contatos

Para questões técnicas ou problemas:
1. Consultar esta documentação
2. Verificar logs do IdentityIQ usando o `requestID`
3. Contatar equipe de suporte com informações detalhadas (ver seção 9.1)

