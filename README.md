# FORM CHANGE - DEPLOY INTEGRAÇÃO S3 E SMARTID EM PRODUÇÃO

## INFORMAÇÕES GERAIS

**Título**: Deploy da Integração S3 e SmartID (IdentityIQ) em Produção  
**Data**: 15 de Janeiro de 2024  
**Solicitante**: Equipe TIM - Cybersecurity  
**Responsável Técnico**: [Nome do Responsável]  
**Ambiente**: Produção  
**Tipo de Change**: Deploy de Nova Funcionalidade  
**Prioridade**: Alta  
**Janela de Manutenção**: [Data/Hora]  

---

## 1. PASSO A PASSO DA INSTALAÇÃO

### 1.1. Área: IT-OPS - Infraestrutura

#### **Pré-requisitos**
- [ ] Verificar disponibilidade do ambiente SmartID Produção
- [ ] Confirmar backup completo do IdentityIQ
- [ ] Validar conectividade de rede entre S3 e SmartID
- [ ] Verificar certificados SSL válidos
- [ ] Confirmar recursos de CPU/Memória disponíveis

#### **Atividades de Instalação**

**1.1.1. Backup e Preparação**
- [ ] Executar backup completo do banco IdentityIQ
- [ ] Criar snapshot do servidor SmartID
- [ ] Documentar configurações atuais
- [ ] Preparar ambiente de rollback

**1.1.2. Deploy dos Componentes**
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

**1.1.3. Configuração de Variáveis**
- [ ] Configurar variáveis do workflow TIM-S3:
  - `s3_apiKey`: [Chave da API S3 Produção]
  - `s3_apiSecret`: [Secret da API S3 Produção]
  - `s3_baseUrl`: `https://azrwin0145.internal.timbrasil.com.br:8083/api/v1`
  - `customName`: `TIM-S3-Applications`
  - `fallbackApprover`: `spadmin`
- [ ] Configurar permissões de execução do workflow
- [ ] Configurar timeouts e retry policies

**1.1.4. Configuração de Custom**
- [ ] Criar/atualizar custom `TIM-S3-Applications` com sistemas permitidos
- [ ] Configurar mapeamentos de argumentos por sistema
- [ ] Validar configurações de duplicidade de roles

**1.1.5. Configuração de Monitoramento**
- [ ] Configurar alertas no PRTG para o workflow TIM-S3
- [ ] Configurar logs de auditoria
- [ ] Configurar métricas de performance

### 1.2. Área: TIM - Desenvolvimento

#### **Atividades de Configuração**

**1.2.1. Configuração da API S3**
- [ ] Configurar endpoint de produção no S3
- [ ] Validar credenciais de API
- [ ] Configurar timeout e retry policies
- [ ] Testar conectividade com SmartID

**1.2.2. Configuração de Dados**
- [ ] Configurar identidades de teste em produção
- [ ] Configurar roles e perfis necessários
- [ ] Configurar aprovadores padrão
- [ ] Validar mapeamentos de sistemas

**1.2.3. Configuração de Segurança**
- [ ] Configurar autenticação entre S3 e SmartID
- [ ] Validar certificados SSL
- [ ] Configurar firewall e regras de rede
- [ ] Implementar logs de segurança

### 1.3. Área: TIM - Cybersecurity

#### **Atividades de Validação**

**1.3.1. Validação de Segurança**
- [ ] Validar controles de acesso
- [ ] Verificar auditoria e rastreabilidade
- [ ] Validar criptografia de dados
- [ ] Verificar conformidade com políticas

**1.3.2. Validação de Governança**
- [ ] Validar fluxo de aprovação
- [ ] Verificar controles de duplicidade
- [ ] Validar custom de sistemas permitidos
- [ ] Verificar logs de auditoria

---

## 2. PROCEDIMENTO DE VALIDAÇÃO FUNCIONAL E TÉCNICA

### 2.1. Validação Técnica

#### **Área: IT-OPS**

**2.1.1. Validação de Infraestrutura**
- [ ] Verificar status dos serviços IdentityIQ
- [ ] Validar conectividade de rede
- [ ] Verificar logs de sistema
- [ ] Validar performance do servidor
- [ ] Verificar backup automático

**2.1.2. Validação de Configuração**
- [ ] Verificar importação dos workflows
- [ ] Validar configuração das rules
- [ ] Verificar templates de email
- [ ] Validar variáveis do workflow
- [ ] Verificar permissões de execução

**2.1.3. Validação de Monitoramento**
- [ ] Verificar alertas do PRTG
- [ ] Validar logs de auditoria
- [ ] Verificar métricas de performance
- [ ] Validar notificações de erro

#### **Área: TIM - Desenvolvimento**

**2.1.4. Validação de Integração**
- [ ] Testar conectividade S3 → SmartID
- [ ] Validar autenticação da API
- [ ] Verificar timeout e retry
- [ ] Validar tratamento de erros
- [ ] Testar carga normal (10 requisições)

### 2.2. Validação Funcional

#### **Área: TIM - Cybersecurity**

**2.2.1. Testes de Concessão de Acesso**
- [ ] **CT001**: Concessão de acesso com sucesso
- [ ] **CT002**: Remoção de acesso com sucesso
- [ ] **CT003**: Concessão com app desconectada
- [ ] Validar fluxo de aprovação
- [ ] Verificar emails de notificação

**2.2.2. Testes de Validação**
- [ ] **CT004**: Request ID inválido
- [ ] **CT005**: Identidade não encontrada
- [ ] **CT006**: Identidade inativa
- [ ] **CT007**: Role não encontrada
- [ ] **CT008**: Aprovador não encontrado
- [ ] **CT009**: Sistema não cadastrado
- [ ] **CT010**: Parâmetros obrigatórios ausentes
- [ ] **CT011**: Operação inválida

**2.2.3. Testes de Segurança**
- [ ] **CT017**: Proteção contra injeção SQL
- [ ] **CT018**: Proteção contra XSS
- [ ] Validar auditoria e logs
- [ ] Verificar rastreabilidade

**2.2.4. Testes de Performance**
- [ ] **CT015**: Carga normal (10 requisições)
- [ ] **CT016**: Carga alta (50 requisições)
- [ ] Verificar tempo de resposta < 30s
- [ ] Validar taxa de sucesso > 95%

#### **Área: TIM - Negócio**

**2.2.5. Validação de Processo de Negócio**
- [ ] Validar fluxo de aprovação manual
- [ ] Verificar notificações por email
- [ ] Validar controle de duplicidade
- [ ] Verificar auditoria de mudanças
- [ ] Validar conformidade com políticas

---

## 3. PROCEDIMENTOS DE ROLLBACK

### 3.1. Rollback Completo

#### **Área: IT-OPS**

**3.1.1. Rollback de Workflows e Rules**
- [ ] Parar execução de workflows ativos
- [ ] Remover workflows importados:
  - `TIM-S3.xml`
  - `TIM-S3-Identity Request Initialize.xml`
  - `TIM-S3-Identity Request Provision.xml`
  - `TIM-S3-Provision with retries.xml`
- [ ] Remover rules importadas (12 rules)
- [ ] Remover templates de email
- [ ] Restaurar backup do banco IdentityIQ
- [ ] Restaurar snapshot do servidor

**3.1.2. Rollback de Configurações**
- [ ] Remover custom `TIM-S3-Applications`
- [ ] Restaurar variáveis do sistema
- [ ] Remover configurações de monitoramento
- [ ] Restaurar configurações de rede

#### **Área: TIM - Desenvolvimento**

**3.1.3. Rollback da Integração S3**
- [ ] Desabilitar endpoint de integração no S3
- [ ] Restaurar configurações de API
- [ ] Remover credenciais de produção
- [ ] Restaurar configurações de segurança

### 3.2. Plano de Ação para Falhas

**Em caso de falha crítica durante o deploy:**

1. **Imediato (0-15 minutos)**:
   - [ ] Parar execução de workflows ativos
   - [ ] Notificar equipe de incidente
   - [ ] Iniciar procedimento de rollback

2. **Curto Prazo (15-60 minutos)**:
   - [ ] Executar rollback completo
   - [ ] Validar restauração do sistema
   - [ ] Documentar falhas encontradas
   - [ ] Notificar stakeholders

3. **Médio Prazo (1-24 horas)**:
   - [ ] Análise de causa raiz
   - [ ] Correção dos problemas identificados
   - [ ] Novo planejamento de deploy
   - [ ] Atualização de procedimentos

**Justificativa da Ausência de Rollback Parcial**:
Não é possível realizar rollback parcial devido à natureza integrada dos componentes. O workflow TIM-S3 depende de múltiplas rules e configurações que devem ser consistentes. Um rollback parcial poderia deixar o sistema em estado inconsistente, causando falhas adicionais.

---

## 4. AMBIENTE DE DISASTER RECOVERY

### 4.1. Área: IT-OPS - DR

#### **Atividades no Ambiente DR**

**4.1.1. Preparação do Ambiente DR**
- [ ] Verificar disponibilidade do ambiente DR
- [ ] Validar conectividade de rede
- [ ] Confirmar backup recente do ambiente principal
- [ ] Verificar recursos disponíveis

**4.1.2. Deploy no Ambiente DR**
- [ ] Executar backup do ambiente DR atual
- [ ] Importar workflows para IdentityIQ DR
- [ ] Importar rules para IdentityIQ DR
- [ ] Importar templates de email
- [ ] Configurar variáveis do workflow
- [ ] Configurar custom `TIM-S3-Applications`
- [ ] Configurar monitoramento

**4.1.3. Validação no Ambiente DR**
- [ ] Executar testes funcionais básicos
- [ ] Validar conectividade com S3
- [ ] Verificar logs e auditoria
- [ ] Testar procedimento de failover

### 4.2. Área: TIM - Desenvolvimento

#### **Configuração da Integração DR**

**4.2.1. Configuração S3 para DR**
- [ ] Configurar endpoint DR no S3
- [ ] Validar credenciais de API DR
- [ ] Configurar failover automático
- [ ] Testar conectividade DR

**4.2.2. Validação de Dados DR**
- [ ] Sincronizar dados de identidades
- [ ] Validar roles e perfis
- [ ] Configurar aprovadores
- [ ] Verificar mapeamentos de sistemas

### 4.3. Área: TIM - Cybersecurity

#### **Validação de Segurança DR**

**4.3.1. Testes de Segurança DR**
- [ ] Executar testes de segurança básicos
- [ ] Validar auditoria e logs
- [ ] Verificar conformidade
- [ ] Testar procedimento de failover

**4.3.2. Validação de Governança DR**
- [ ] Validar fluxo de aprovação
- [ ] Verificar controles de acesso
- [ ] Validar custom de sistemas
- [ ] Verificar logs de auditoria

---

## 5. CRONOGRAMA DE EXECUÇÃO

### 5.1. Timeline Detalhado

| Horário | Atividade | Responsável | Duração |
|---------|-----------|-------------|---------|
| 20:00 | Backup e preparação | IT-OPS | 30 min |
| 20:30 | Deploy workflows e rules | IT-OPS | 45 min |
| 21:15 | Configuração de variáveis | IT-OPS | 30 min |
| 21:45 | Configuração custom | IT-OPS | 30 min |
| 22:15 | Configuração S3 | TIM-Dev | 30 min |
| 22:45 | Validação técnica | IT-OPS | 30 min |
| 23:15 | Validação funcional | TIM-Cyber | 45 min |
| 00:00 | Validação final | TIM-Cyber | 30 min |
| 00:30 | Liberação para produção | TIM-Cyber | 15 min |

### 5.2. Janela de Manutenção

- **Início**: [Data] às 20:00
- **Fim**: [Data] às 01:00
- **Duração Total**: 5 horas
- **Tempo de Rollback**: 2 horas (se necessário)

---

## 6. CRITÉRIOS DE SUCESSO

### 6.1. Critérios Técnicos
- [ ] Todos os workflows importados com sucesso
- [ ] Todas as rules funcionando corretamente
- [ ] Conectividade S3 ↔ SmartID estabelecida
- [ ] Logs de auditoria funcionando
- [ ] Monitoramento ativo

### 6.2. Critérios Funcionais
- [ ] 100% dos testes de validação aprovados
- [ ] Fluxo de aprovação funcionando
- [ ] Emails de notificação enviados
- [ ] Auditoria completa
- [ ] Performance dentro dos limites

### 6.3. Critérios de Negócio
- [ ] Processo de concessão/remoção funcionando
- [ ] Controles de segurança ativos
- [ ] Conformidade com políticas
- [ ] Usuários treinados e operacionais

---

## 7. RISCOS E MITIGAÇÕES

### 7.1. Riscos Identificados

| Risco | Probabilidade | Impacto | Mitigação |
|-------|---------------|---------|-----------|
| Falha na conectividade S3-SmartID | Média | Alto | Teste prévio de conectividade |
| Problemas de performance | Baixa | Médio | Monitoramento contínuo |
| Falha na autenticação | Baixa | Alto | Validação de credenciais |
| Erro na configuração | Média | Alto | Testes de validação |
| Falha no rollback | Baixa | Crítico | Procedimento testado |

### 7.2. Plano de Contingência

- **Falha Crítica**: Rollback imediato
- **Falha de Performance**: Otimização e monitoramento
- **Falha de Conectividade**: Troubleshooting de rede
- **Falha de Configuração**: Correção e validação

---

## 8. APROVAÇÕES

| Área | Responsável | Assinatura | Data |
|------|-------------|------------|------|
| IT-OPS | [Nome] | [ ] | [Data] |
| TIM-Desenvolvimento | [Nome] | [ ] | [Data] |
| TIM-Cybersecurity | [Nome] | [ ] | [Data] |
| TIM-Gerência | [Nome] | [ ] | [Data] |

---

**Documento**: FORM_CHANGE_DEPLOY_S3_SMARTID.md  
**Versão**: 1.0  
**Data**: 15 de Janeiro de 2024  
**Status**: Aguardando Aprovação
