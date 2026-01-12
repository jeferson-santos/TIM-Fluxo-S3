# Resumo Executivo - API de Gerenciamento de Tokens RSA

## Visão Geral

API REST para gerenciamento de tokens RSA através do workflow IdentityIQ `TIM-RSA`, suportando operações de adição e remoção de tokens nos ambientes **OnPremise** e **Cloud**.

**Endpoint:** `POST /identityiq/rest/workflows/TIM-RSA/launch`

**Autenticação:** Basic Auth (HTTP Basic Authentication)

---

## Parâmetros Obrigatórios

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| `requestId` | String | ✅ | ID para rastreamento (ex: "TK001") |
| `type` | String | ✅ | Ambiente: `"onpremise"` ou `"cloud"` |
| `operation` | String | ✅ | Operação: `"addtoken"` ou `"removetoken"` |
| `user` | String | ✅ | Matrícula do usuário (ex: "T3622753") |
| `onpremiseProfile` | String | ⚠️ | Perfil: `"IOS"`, `"Android"` ou `"Desktop"` (obrigatório se `type="onpremise"` e `operation="addtoken"`) |
| `emailToSend` | String | ⚠️ | Email para envio (obrigatório se `type="cloud"` e `operation="addtoken"`) |

---

## Validações Principais

### ✅ Validações de Entrada
- Usuário existe no IdentityIQ (atributo `att_usuario`)
- Usuário não está bloqueado (`tim_block_list != "true"`)
- Tipo válido: `"onpremise"` ou `"cloud"`
- Operação válida: `"addtoken"` ou `"removetoken"`
- Perfil válido (OnPremise): `"IOS"`, `"Android"` ou `"Desktop"`

### ⚠️ Validações Específicas por Ambiente

**OnPremise:**
- Perfil obrigatório para `addtoken`
- Tokens antigos são removidos automaticamente antes de adicionar novo

**Cloud:**
- Email obrigatório para `addtoken`
- Conta Ping Directory deve existir e estar ativa
- Tokens antigos são removidos automaticamente antes de adicionar novo

---

## Fluxo Resumido

```
1. Start
   ↓
2. Verify User (valida matrícula e bloqueio)
   ↓
3. Roteamento por Type (Cloud ou OnPremise)
   ↓
4. Validações Específicas (profile/operation)
   ↓
5. Execução da Operação
   ├─ OnPremise: addTokenOnpremise() ou removeTokenOnpremise()
   └─ Cloud: updatePingDirRSA() + removeTokenCloud() + addTokenCloud()
   ↓
6. Write Audit (registro de auditoria)
   ↓
7. Stop
```

---

## Resposta de Sucesso

**addtoken (OnPremise):**
```json
{
    "requestID": "0a9736459b6c1b13819b8ff522d51e09",
    "errors": null,
    "attributes": {
        "activationUrl": "com.rsa.securid://ctf?ctfData=...",
        "user": "T3622753",
        "operation": "addtoken",
        "type": "onpremise"
    }
}
```

**removetoken:**
```json
{
    "requestID": "0a9736459b6c1b13819b8ff522d51e09",
    "errors": null,
    "attributes": {
        "user": "T3622753",
        "operation": "removetoken",
        "type": "onpremise"
    }
}
```

---

## Resposta de Erro

```json
{
    "requestID": null,
    "errors": [
        "Status : failed\nAn unexpected error occurred: sailpoint.tools.GeneralException: Identidade não encontrada com matrícula: T36227533\n"
    ],
    "attributes": {
        "user": "T36227533",
        "operation": "addtoken",
        "type": "onpremise"
    }
}
```

---

## Erros Comuns

| Erro | Causa | Solução |
|------|-------|---------|
| `Identity not found` | Matrícula não existe | Verificar matrícula no IdentityIQ |
| `Identity is blocked` | Usuário na blacklist | Remover bloqueio (`tim_block_list`) |
| `Profile invalid` | Perfil inválido | Usar: IOS, Android ou Desktop |
| `Ping Directory account not found` | Conta não existe | Verificar conta no Ping Directory |
| `Authentication token could not be obtained` | Erro de autenticação RSA | Verificar credenciais no Custom Object |

---

## Exemplos Rápidos

### Adicionar Token OnPremise iOS
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

### Remover Token OnPremise
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

### Adicionar Token Cloud
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

---

## Checklist Pré-Requisição

- [ ] Matrícula existe no IdentityIQ
- [ ] Usuário não está bloqueado
- [ ] Parâmetros obrigatórios fornecidos
- [ ] Tipo e operação são valores válidos
- [ ] Perfil válido (se OnPremise addtoken)
- [ ] Email fornecido (se Cloud addtoken)
- [ ] Custom Object `TIM-Custom-RSA-Credentials` configurado
- [ ] Permissões de autenticação corretas

---

## Dependências

- **IdentityIQ** com workflow TIM-RSA implantado
- **Custom Object:** `TIM-Custom-RSA-Credentials` com credenciais RSA
- **OnPremise:** Servidor RSA acessível com credenciais válidas
- **Cloud:** Ping Directory configurado (apenas para Cloud)
- **Permissões:** Usuário com permissão para executar workflows

---

## Auditoria

Todos os eventos são registrados automaticamente:
- **Action:** RSA
- **Target:** Matrícula do usuário
- **Type:** onpremise/cloud
- **Operation:** addtoken/removetoken
- **RequestID:** ID fornecido na requisição

---

## Suporte

Para problemas:
1. Verificar campo `errors` na resposta
2. Consultar logs do IdentityIQ pelo `requestID`
3. Verificar configuração do Custom Object
4. Validar checklist pré-requisição

**Documentação Completa:** Ver `DOCUMENTACAO_TECNICA_RSA.md`

