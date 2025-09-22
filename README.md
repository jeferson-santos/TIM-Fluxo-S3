# CASOS DE TESTE - INTEGRAÇÃO S3 E SMARTID (IDENTITYIQ)

## 1. INTRODUÇÃO

### 1.1. Objetivo
Este documento apresenta os casos de teste para validação da integração entre a aplicação S3 e a plataforma SmartID (IdentityIQ), cobrindo os cenários de concessão e remoção de acessos.

### 1.2. Escopo
- Testes funcionais do workflow TIM-S3
- Validação de integração via API REST
- Testes de validação de dados
- Testes de aprovação e provisionamento
- Testes de tratamento de erros

### 1.3. Ambiente de Teste
- **URL Base**: `http://smartid.internal.timbrasil.com.br/identityiq/rest/workflows/TIM-S3/launch`
- **Ambiente**: Homologação/Desenvolvimento
- **Dados de Teste**: Conjunto específico de identidades e roles para testes

## 2. DADOS DE TESTE

### 2.1. Identidades de Teste
| Matrícula | Nome | Status | Aprovador |
|-----------|------|--------|-----------|
| F8015590 | João Silva | Ativo | F8000036 |
| F8015591 | Maria Santos | Ativo | F8000037 |
| F8015592 | Pedro Costa | Inativo | F8000036 |
| F8015593 | Ana Oliveira | Ativo | F8000038 |

### 2.2. Sistemas e Perfis de Teste
| Sistema | Perfil | Status | Permite Duplicidade |
|---------|--------|--------|-------------------|
| BDOH | AFBANCR1 | Ativo | Não |
| BDOH | AFBANCR2 | Ativo | Não |
| SIEBEL | SIEBELPOS_TIBPCS01 | Ativo | Sim |
| SIEBEL | SIEBELPOS_TIBPCS02 | Ativo | Sim |

### 2.3. Aprovadores de Teste
| Matrícula | Nome | Status | Tipo |
|-----------|------|--------|------|
| F8000036 | Carlos Manager | Ativo | Manager |
| F8000037 | Lucia Supervisor | Ativo | Supervisor |
| F8000038 | Roberto Admin | Ativo | Admin |

## 3. CASOS DE TESTE FUNCIONAIS

### 3.1. CT001 - Concessão de Acesso com Sucesso

**Objetivo**: Validar concessão de acesso com todos os parâmetros válidos

**Pré-condições**:
- Identidade existe e está ativa
- Role existe e está ativa
- Aprovador existe e está ativo
- Sistema está cadastrado na custom

**Dados de Entrada**:
```json
{
  "workflowArgs": {
    "requestId": "CT001-001",
    "user": "F8015590",
    "operation": "add",
    "system": "BDOH",
    "profile": "AFBANCR1",
    "approver": "F8000036",
    "args": {
      "area": "ADMINISTRADOR",
      "filial": "SP"
    }
  }
}
```

**Passos**:
1. Enviar requisição via API REST
2. Verificar validação da requisição
3. Verificar validação dos dados
4. Verificar validação dos parâmetros
5. Verificar criação do plano
6. Verificar aprovação manual
7. Verificar provisionamento

**Resultado Esperado**:
- Status: success
- Workflow executado completamente
- Acesso concedido com sucesso
- Email de notificação enviado

---

### 3.2. CT002 - Remoção de Acesso com Sucesso

**Objetivo**: Validar remoção de acesso com parâmetros válidos

**Pré-condições**:
- Identidade existe e está ativa
- Role existe e está ativa
- Aprovador existe e está ativo
- Sistema está cadastrado na custom

**Dados de Entrada**:
```json
{
  "workflowArgs": {
    "requestId": "CT002-001",
    "user": "F8015590",
    "operation": "remove",
    "system": "BDOH",
    "profile": "AFBANCR1",
    "approver": "F8000036"
  }
}
```

**Passos**:
1. Enviar requisição via API REST
2. Verificar validação da requisição
3. Verificar validação dos dados
4. Verificar criação do plano (sem validação de parâmetros)
5. Verificar aprovação manual
6. Verificar provisionamento

**Resultado Esperado**:
- Status: success
- Workflow executado completamente
- Acesso removido com sucesso
- Email de notificação enviado

---

### 3.3. CT003 - Concessão com App Desconectada

**Objetivo**: Validar fluxo de aprovação para aplicação desconectada

**Pré-condições**:
- Identidade existe e está ativa
- Role existe e está ativa
- Aprovador existe e está ativo
- Sistema está cadastrado na custom
- Aplicação configurada como desconectada

**Dados de Entrada**:
```json
{
  "workflowArgs": {
    "requestId": "CT003-001",
    "user": "F8015590",
    "operation": "add",
    "system": "SIEBEL",
    "profile": "SIEBELPOS_TIBPCS01",
    "approver": "F8000036",
    "args": {
      "position": "XYZ",
      "pin": "XYZ"
    }
  }
}
```

**Passos**:
1. Enviar requisição via API REST
2. Verificar validação da requisição
3. Verificar validação dos dados
4. Verificar validação dos parâmetros
5. Verificar criação do plano
6. Verificar aprovação manual
7. Verificar detecção de app desconectada
8. Verificar aprovação adicional para app desconectada
9. Verificar provisionamento

**Resultado Esperado**:
- Status: success
- Workflow executado completamente
- Duas aprovações manuais realizadas
- Acesso concedido com sucesso

---

## 4. CASOS DE TESTE DE VALIDAÇÃO

### 4.1. CT004 - Request ID Inválido

**Objetivo**: Validar tratamento de Request ID inválido

**Dados de Entrada**:
```json
{
  "workflowArgs": {
    "requestId": "INVALID-REQUEST-ID",
    "user": "F8015590",
    "operation": "add",
    "system": "BDOH",
    "profile": "AFBANCR1",
    "approver": "F8000036"
  }
}
```

**Resultado Esperado**:
- Status: error
- Código: REQUEST_NOT_FOUND
- Mensagem: "Request ID inválido ou não encontrado"

---

### 4.2. CT005 - Identidade Não Encontrada

**Objetivo**: Validar tratamento de identidade inexistente

**Dados de Entrada**:
```json
{
  "workflowArgs": {
    "requestId": "CT005-001",
    "user": "F9999999",
    "operation": "add",
    "system": "BDOH",
    "profile": "AFBANCR1",
    "approver": "F8000036"
  }
}
```

**Resultado Esperado**:
- Status: error
- Código: IDENTITY_NOT_FOUND
- Mensagem: "Identidade não encontrada para a matrícula F9999999"

---

### 4.3. CT006 - Identidade Inativa

**Objetivo**: Validar tratamento de identidade inativa

**Dados de Entrada**:
```json
{
  "workflowArgs": {
    "requestId": "CT006-001",
    "user": "F8015592",
    "operation": "add",
    "system": "BDOH",
    "profile": "AFBANCR1",
    "approver": "F8000036"
  }
}
```

**Resultado Esperado**:
- Status: error
- Código: IDENTITY_INACTIVE
- Mensagem: "Identidade F8015592 está inativa"

---

### 4.4. CT007 - Role Não Encontrada

**Objetivo**: Validar tratamento de role inexistente

**Dados de Entrada**:
```json
{
  "workflowArgs": {
    "requestId": "CT007-001",
    "user": "F8015590",
    "operation": "add",
    "system": "BDOH",
    "profile": "INVALID_PROFILE",
    "approver": "F8000036"
  }
}
```

**Resultado Esperado**:
- Status: error
- Código: ROLE_NOT_FOUND
- Mensagem: "Role BDOH_INVALID_PROFILE não encontrada"

---

### 4.5. CT008 - Role Inativa

**Objetivo**: Validar tratamento de role inativa

**Dados de Entrada**:
```json
{
  "workflowArgs": {
    "requestId": "CT008-001",
    "user": "F8015590",
    "operation": "add",
    "system": "BDOH",
    "profile": "INACTIVE_PROFILE",
    "approver": "F8000036"
  }
}
```

**Resultado Esperado**:
- Status: error
- Código: ROLE_INACTIVE
- Mensagem: "Role BDOH_INACTIVE_PROFILE está desabilitada"

---

### 4.6. CT009 - Aprovador Não Encontrado

**Objetivo**: Validar tratamento de aprovador inexistente

**Dados de Entrada**:
```json
{
  "workflowArgs": {
    "requestId": "CT009-001",
    "user": "F8015590",
    "operation": "add",
    "system": "BDOH",
    "profile": "AFBANCR1",
    "approver": "F9999999"
  }
}
```

**Resultado Esperado**:
- Status: error
- Código: APPROVER_NOT_FOUND
- Mensagem: "Approver não encontrado após todas as tentativas: F9999999"

---

### 4.7. CT010 - Sistema Não Cadastrado

**Objetivo**: Validar tratamento de sistema não cadastrado na custom

**Dados de Entrada**:
```json
{
  "workflowArgs": {
    "requestId": "CT010-001",
    "user": "F8015590",
    "operation": "add",
    "system": "INVALID_SYSTEM",
    "profile": "AFBANCR1",
    "approver": "F8000036"
  }
}
```

**Resultado Esperado**:
- Status: error
- Código: SYSTEM_NOT_REGISTERED
- Mensagem: "Sistema 'INVALID_SYSTEM' não está cadastrado na custom TIM-S3-Applications"

---

### 4.8. CT011 - Parâmetros Obrigatórios Ausentes

**Objetivo**: Validar tratamento de parâmetros obrigatórios não fornecidos

**Dados de Entrada**:
```json
{
  "workflowArgs": {
    "requestId": "CT011-001",
    "user": "F8015590",
    "operation": "add",
    "system": "BDOH",
    "profile": "AFBANCR1",
    "approver": "F8000036",
    "args": {}
  }
}
```

**Resultado Esperado**:
- Status: error
- Código: MISSING_REQUIRED_PARAMETERS
- Mensagem: "Campo obrigatório não encontrado: [campo_obrigatorio]"

---

### 4.9. CT012 - Operação Inválida

**Objetivo**: Validar tratamento de operação inválida

**Dados de Entrada**:
```json
{
  "workflowArgs": {
    "requestId": "CT012-001",
    "user": "F8015590",
    "operation": "invalid",
    "system": "BDOH",
    "profile": "AFBANCR1",
    "approver": "F8000036"
  }
}
```

**Resultado Esperado**:
- Status: error
- Código: INVALID_OPERATION
- Mensagem: "Operação inválida - invalid. Deve ser 'Add' ou 'Remove'"

---

## 5. CASOS DE TESTE DE INTEGRAÇÃO

### 5.1. CT013 - Falha na Autenticação S3

**Objetivo**: Validar tratamento de falha na autenticação com API S3

**Pré-condições**:
- API Key ou Secret incorretos
- API S3 indisponível

**Dados de Entrada**:
```json
{
  "workflowArgs": {
    "requestId": "CT013-001",
    "user": "F8015590",
    "operation": "add",
    "system": "BDOH",
    "profile": "AFBANCR1",
    "approver": "F8000036"
  }
}
```

**Resultado Esperado**:
- Status: error
- Código: AUTHENTICATION_FAILED
- Mensagem: "Falha ao obter token de autenticação"

---

### 5.2. CT014 - Timeout na API S3

**Objetivo**: Validar tratamento de timeout na comunicação com API S3

**Pré-condições**:
- API S3 com resposta lenta (> 30 segundos)

**Dados de Entrada**:
```json
{
  "workflowArgs": {
    "requestId": "CT014-001",
    "user": "F8015590",
    "operation": "add",
    "system": "BDOH",
    "profile": "AFBANCR1",
    "approver": "F8000036"
  }
}
```

**Resultado Esperado**:
- Status: error
- Código: TIMEOUT
- Mensagem: "Timeout na comunicação com API S3"

---

### 5.3. CT015 - Falha no Provisionamento

**Objetivo**: Validar tratamento de falha no provisionamento

**Pré-condições**:
- Sistema de destino indisponível
- Erro na aplicação de destino

**Dados de Entrada**:
```json
{
  "workflowArgs": {
    "requestId": "CT015-001",
    "user": "F8015590",
    "operation": "add",
    "system": "BDOH",
    "profile": "AFBANCR1",
    "approver": "F8000036"
  }
}
```

**Resultado Esperado**:
- Status: error
- Código: PROVISIONING_FAILED
- Mensagem: "Falha no provisionamento do acesso"

---

## 6. CASOS DE TESTE DE PERFORMANCE

### 6.1. CT016 - Carga Normal

**Objetivo**: Validar performance com carga normal

**Pré-condições**:
- 10 requisições simultâneas
- Dados válidos

**Passos**:
1. Enviar 10 requisições simultâneas
2. Medir tempo de resposta
3. Verificar taxa de sucesso

**Resultado Esperado**:
- Tempo médio de resposta: < 30 segundos
- Taxa de sucesso: 100%
- Sem erros de concorrência

---

### 6.2. CT017 - Carga Alta

**Objetivo**: Validar performance com carga alta

**Pré-condições**:
- 50 requisições simultâneas
- Dados válidos

**Passos**:
1. Enviar 50 requisições simultâneas
2. Medir tempo de resposta
3. Verificar taxa de sucesso

**Resultado Esperado**:
- Tempo médio de resposta: < 60 segundos
- Taxa de sucesso: > 95%
- Sem erros de concorrência

---

## 7. CASOS DE TESTE DE SEGURANÇA

### 7.1. CT018 - Injeção SQL

**Objetivo**: Validar proteção contra injeção SQL

**Dados de Entrada**:
```json
{
  "workflowArgs": {
    "requestId": "CT018-001",
    "user": "F8015590'; DROP TABLE users; --",
    "operation": "add",
    "system": "BDOH",
    "profile": "AFBANCR1",
    "approver": "F8000036"
  }
}
```

**Resultado Esperado**:
- Status: error
- Código: INVALID_INPUT
- Mensagem: "Dados de entrada inválidos"
- Sistema não comprometido

---

### 7.2. CT019 - XSS

**Objetivo**: Validar proteção contra XSS

**Dados de Entrada**:
```json
{
  "workflowArgs": {
    "requestId": "CT019-001",
    "user": "F8015590",
    "operation": "add",
    "system": "BDOH",
    "profile": "AFBANCR1",
    "approver": "F8000036",
    "args": {
      "area": "<script>alert('XSS')</script>"
    }
  }
}
```

**Resultado Esperado**:
- Status: success ou error (dependendo da validação)
- Script não executado
- Dados sanitizados

---

## 8. CASOS DE TESTE DE APROVAÇÃO

### 8.1. CT020 - Aprovação Aprovada

**Objetivo**: Validar fluxo de aprovação aprovada

**Pré-condições**:
- Workflow em estado de aprovação
- Aprovador acessa work item

**Passos**:
1. Acessar work item no SmartID
2. Aprovar requisição
3. Verificar continuação do workflow
4. Verificar provisionamento

**Resultado Esperado**:
- Work item aprovado
- Workflow continua para provisionamento
- Acesso concedido/removido

---

### 8.2. CT021 - Aprovação Rejeitada

**Objetivo**: Validar fluxo de aprovação rejeitada

**Pré-condições**:
- Workflow em estado de aprovação
- Aprovador acessa work item

**Passos**:
1. Acessar work item no SmartID
2. Rejeitar requisição
3. Verificar finalização do workflow
4. Verificar não provisionamento

**Resultado Esperado**:
- Work item rejeitado
- Workflow finalizado com erro
- Acesso não concedido/removido

---

### 8.3. CT022 - Timeout de Aprovação

**Objetivo**: Validar tratamento de timeout de aprovação

**Pré-condições**:
- Workflow em estado de aprovação
- Aprovador não responde em 24 horas

**Passos**:
1. Aguardar 24 horas
2. Verificar timeout automático
3. Verificar finalização do workflow

**Resultado Esperado**:
- Timeout automático
- Workflow finalizado com erro
- Acesso não concedido/removido

---

## 9. CASOS DE TESTE DE MONITORAMENTO

### 9.1. CT023 - Logs de Auditoria

**Objetivo**: Validar geração de logs de auditoria

**Pré-condições**:
- Workflow executado com sucesso

**Passos**:
1. Executar workflow completo
2. Verificar logs no IdentityIQ
3. Verificar logs da API S3
4. Verificar logs de auditoria

**Resultado Esperado**:
- Logs detalhados gerados
- Rastreabilidade completa
- Informações de auditoria corretas

---

### 9.2. CT024 - Métricas de Performance

**Objetivo**: Validar coleta de métricas de performance

**Pré-condições**:
- Workflow executado múltiplas vezes

**Passos**:
1. Executar 10 workflows
2. Verificar métricas no PRTG
3. Verificar alertas de performance

**Resultado Esperado**:
- Métricas coletadas corretamente
- Alertas gerados quando necessário
- Performance dentro dos limites

---

## 10. PLANO DE EXECUÇÃO

### 10.1. Ordem de Execução

1. **Fase 1 - Testes Básicos** (CT001-CT003)
2. **Fase 2 - Testes de Validação** (CT004-CT012)
3. **Fase 3 - Testes de Integração** (CT013-CT015)
4. **Fase 4 - Testes de Performance** (CT016-CT017)
5. **Fase 5 - Testes de Segurança** (CT018-CT019)
6. **Fase 6 - Testes de Aprovação** (CT020-CT022)
7. **Fase 7 - Testes de Monitoramento** (CT023-CT024)

### 10.2. Critérios de Aprovação

- **Taxa de Sucesso**: > 95% para testes funcionais
- **Performance**: Tempo de resposta < 30 segundos (carga normal)
- **Segurança**: 100% dos testes de segurança aprovados
- **Auditoria**: 100% dos logs gerados corretamente

### 10.3. Ambiente de Teste

- **Servidor**: smartid-homolog.internal.timbrasil.com.br
- **Banco de Dados**: IdentityIQ Homologação
- **API S3**: Ambiente de homologação
- **Dados**: Conjunto específico para testes

---

## 11. RELATÓRIO DE TESTES

### 11.1. Template de Relatório

| CT | Descrição | Status | Tempo | Observações |
|----|-----------|--------|-------|-------------|
| CT001 | Concessão de Acesso com Sucesso | ✅ | 15s | - |
| CT002 | Remoção de Acesso com Sucesso | ✅ | 12s | - |
| CT003 | Concessão com App Desconectada | ✅ | 25s | - |
| ... | ... | ... | ... | ... |

### 11.2. Métricas de Qualidade

- **Cobertura de Testes**: 100% dos cenários críticos
- **Taxa de Sucesso**: [X]%
- **Tempo Médio de Execução**: [X] segundos
- **Bugs Encontrados**: [X]
- **Bugs Críticos**: [X]

---

## 12. CONCLUSÕES

Este documento de casos de teste cobre todos os cenários críticos da integração S3 e SmartID, garantindo:

- Validação completa da funcionalidade
- Cobertura de cenários de erro
- Testes de performance e segurança
- Validação de aprovação e auditoria
- Monitoramento e métricas

A execução destes testes garante a qualidade e confiabilidade da integração antes da implementação em produção.

---

**Documento**: CASOS_DE_TESTE_S3_SMARTID.md  
**Versão**: 1.0  
**Data**: 15 de Janeiro de 2024  
**Status**: Aprovado  
**Próxima Revisão**: 15 de Julho de 2024
