# FORM CHANGE - DEPLOY INTEGRAÇÃO S3 E SMARTID EM PRODUÇÃO

## INFORMAÇÕES GERAIS

**Título**: Deploy da Integração S3 e SmartID (IdentityIQ) em Produção  
**Data**: 15 de Janeiro de 2024  
**Solicitante**: Equipe TIM - Cybersecurity  
**Responsável Técnico**: [Nome do Responsável]  
**Ambiente**: Produção  
**Tipo de Change**: Deploy de Nova Funcionalidade  
**Prioridade**: Média  
**Janela de Manutenção**: [Data/Hora]  

---

## 1. PASSO A PASSO DA INSTALAÇÃO

### 1.1. Área: IT-OPS - Infraestrutura

#### **Atividades de Instalação**

**1.1.1. Deploy dos Componentes**
- [ ] Fazer upload dos workflows XML para o IdentityIQ
  - `TIM-S3.xml`
  - `TIM-S3-Identity Request Initialize.xml`
  - `TIM-S3-Identity Request Provision.xml`
  - `TIM-S3-Provision with retries.xml`
- [ ] Fazer upload das rules para o IdentityIQ
  - `TIM-Rule-Workflow-S3-GetToken`
  - `TIM-Rule-Workflow-S3-CheckRequest`
  - `TIM-Rule-Workflow-S3-CheckData`
  - `TIM-Rule-Workflow-S3-CheckParameters`
  - `TIM-Rule-Workflow-S3-BuildPlan`
  - `TIM-Rule-Workflow-S3-CheckApprover`
  - `TIM-Rule-Workflow-S3-ApprovalSet`
  - `TIM-Rule-Workflow-S3-CheckApprovalResult`
  - `TIM-Rule-Workflow-S3-CheckAppDesconectada`
  - `TIM-Rule-Workflow-S3-ApprovalSet-AppDesconectada`
  - `TIM-Rule-Workflow-S3-ModifyTask`
  - `TIM-Rule-Workflow-S3-InjectProjectArgs`
- [ ] Fazer upload dos templates de email
  - `TIM-EmailTemplate-S3-Approval`
  - `TIM-EmailTemplate-S3-AppDesconectada`

**1.1.2. Configuração de Variáveis**
- [ ] Configurar variáveis do workflow TIM-S3:
  - `s3_apiKey`: [Chave da API S3 Produção]
  - `s3_apiSecret`: [Secret da API S3 Produção]
  - `s3_baseUrl`: `https://azrwin0145.internal.timbrasil.com.br:8083/api/v1`
  - `customName`: `TIM-S3-Applications`
  - `fallbackApprover`: `spadmin`

**1.1.3. Configuração de Custom**
- [ ] Criar/atualizar custom `TIM-S3-Applications` com sistemas permitidos
- [ ] Configurar mapeamentos de argumentos por sistema

### 1.2. Área: TIM - Desenvolvimento

#### **Atividades de Configuração**

**1.2.1. Configuração da API S3**
- [ ] Configurar endpoint de produção no S3
- [ ] Validar credenciais de API
- [ ] Testar conectividade com SmartID

**1.2.2. Configuração de Dados**
- [ ] Configurar identidades de teste em produção
- [ ] Configurar roles e perfis necessários
- [ ] Configurar aprovadores padrão

---

## 2. PROCEDIMENTO DE VALIDAÇÃO FUNCIONAL E TÉCNICA

### 2.1. Validação Técnica

#### **Área: IT-OPS**

**2.1.1. Validação de Configuração**
- [ ] Verificar importação dos workflows
- [ ] Validar configuração das rules
- [ ] Verificar templates de email
- [ ] Validar variáveis do workflow

#### **Área: TIM - Desenvolvimento**

**2.1.2. Validação de Integração**
- [ ] Testar conectividade S3 → SmartID
- [ ] Validar autenticação da API
- [ ] Testar requisição básica

### 2.2. Validação Funcional

#### **Área: TIM - Cybersecurity**

**2.2.1. Testes Básicos**
- [ ] Concessão de acesso com sucesso
- [ ] Remoção de acesso com sucesso
- [ ] Validação de identidade inexistente
- [ ] Validação de role inexistente
- [ ] Fluxo de aprovação manual

---

## 3. PROCEDIMENTOS DE ROLLBACK

### 3.1. Rollback Simples

#### **Área: IT-OPS**

**3.1.1. Remoção de Componentes**
- [ ] Parar execução de workflows ativos
- [ ] Remover workflows importados:
  - `TIM-S3.xml`
  - `TIM-S3-Identity Request Initialize.xml`
  - `TIM-S3-Identity Request Provision.xml`
  - `TIM-S3-Provision with retries.xml`
- [ ] Remover rules importadas (12 rules)
- [ ] Remover templates de email
- [ ] Remover custom `TIM-S3-Applications`

#### **Área: TIM - Desenvolvimento**

**3.1.2. Rollback da Integração S3**
- [ ] Desabilitar endpoint de integração no S3
- [ ] Remover credenciais de produção

### 3.2. Plano de Ação para Falhas

**Em caso de erro durante o deploy:**
- [ ] Remover todas as rules e workflows
- [ ] Tratar erro específico
- [ ] Re-deploy após correção

---

## 4. CRONOGRAMA DE EXECUÇÃO

### 4.1. Timeline Simplificado

| Horário | Atividade | Responsável | Duração |
|---------|-----------|-------------|---------|
| 20:00 | Deploy workflows e rules | IT-OPS | 30 min |
| 20:30 | Configuração de variáveis | IT-OPS | 15 min |
| 20:45 | Configuração custom | IT-OPS | 15 min |
| 21:00 | Configuração S3 | TIM-Dev | 15 min |
| 21:15 | Validação básica | TIM-Cyber | 15 min |
| 21:30 | Liberação para produção | TIM-Cyber | 15 min |

### 4.2. Janela de Manutenção

- **Início**: [Data] às 20:00
- **Fim**: [Data] às 22:00
- **Duração Total**: 2 horas
- **Tempo de Rollback**: 30 minutos (se necessário)

---

## 5. CRITÉRIOS DE SUCESSO

### 5.1. Critérios Técnicos
- [ ] Todos os workflows importados com sucesso
- [ ] Todas as rules funcionando corretamente
- [ ] Conectividade S3 ↔ SmartID estabelecida

### 5.2. Critérios Funcionais
- [ ] Testes básicos aprovados
- [ ] Fluxo de aprovação funcionando
- [ ] Emails de notificação enviados

---

## 6. APROVAÇÕES

| Área | Responsável | Assinatura | Data |
|------|-------------|------------|------|
| IT-OPS | [Nome] | [ ] | [Data] |
| TIM-Desenvolvimento | [Nome] | [ ] | [Data] |
| TIM-Cybersecurity | [Nome] | [ ] | [Data] |

---

**Documento**: FORM_CHANGE_DEPLOY_S3_SMARTID.md  
**Versão**: 1.0  
**Data**: 15 de Janeiro de 2024  
**Status**: Aguardando Aprovação
