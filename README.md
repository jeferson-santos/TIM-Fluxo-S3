# Casos de Teste - TimApprovalAssignmentRuleN1

## Objetivo
Documento resumido de casos de teste para validar os principais cenários da regra de aprovação de primeiro nível (N1).

---

## 1. CONFIGURAÇÃO DE APROVADOR

### CT001 - Aprovador não configurado
**Pré-condições:** `tim_approvalN1` = null ou vazio  
**Resultado:** ❌ Rejeitado - "Aprovador não configurado no perfil"

### CT002 - Aprovador "Disabled"
**Pré-condições:** `tim_approvalN1` = "Disabled"  
**Resultado:** ✅ Autoaprovado - "Aprovação desabilitada no perfil"

### CT003 - Aprovador "Manager"
**Pré-condições:** `tim_approvalN1` = "Manager", solicitante possui gestor  
**Resultado:** ✅ `roleApprover` convertido para gestor do solicitante

### CT004 - Manager sem gestor
**Pré-condições:** `tim_approvalN1` = "Manager", solicitante sem gestor  
**Resultado:** ❌ Rejeitado - "solicitante não possui gestor definido"

---

## 2. VALIDAÇÃO DE APROVADOR

### CT005 - Aprovador não encontrado
**Pré-condições:** Aprovador configurado não existe no sistema  
**Resultado:** ❌ Rejeitado - "Aprovador configurado não encontrado no sistema"

### CT006 - Aprovador inativo
**Pré-condições:** Aprovador existe mas `isInactive() == true` ou `att_descr_status != "Ativo"`  
**Resultado:** ❌ Rejeitado - "Aprovador configurado está inativo no sistema"

### CT007 - Aprovador sem att_descr_status
**Pré-condições:** Aprovador existe mas `att_descr_status` = null ou vazio  
**Resultado:** ❌ Rejeitado - "Aprovador não possui atributo obrigatório 'att_descr_status'"

### CT008 - Aprovador válido
**Pré-condições:** Aprovador existe, ativo, com `att_descr_status = "Ativo"`  
**Resultado:** ✅ Enviado para aprovação

---

## 3. WORKGROUP E FORWARD

### CT009 - Workgroup sem membros ativos
**Pré-condições:** Aprovador é workgroup sem membros ativos  
**Resultado:** ❌ Rejeitado - "Grupo de aprovadores não possui membros ativos"

### CT010 - Workgroup com membros ativos
**Pré-condições:** Aprovador é workgroup com membros ativos  
**Resultado:** ✅ Enviado para aprovação

### CT011 - Forward inválido (não encontrado/inativo/sem status)
**Pré-condições:** Forward configurado mas usuário não existe, inativo ou sem `att_descr_status`  
**Resultado:** ❌ Rejeitado - mensagem específica do problema

### CT012 - Forward válido
**Pré-condições:** Forward configurado para usuário ativo  
**Resultado:** ✅ Processamento continua com forward

---

## 4. TRATAMENTO DE TERCEIROS

### CT013 - Terceiro sem aprovador específico
**Pré-condições:** `tipoUsuario = "Terceiro"`, `aprovadorTerceiro` = null  
**Resultado:** ❌ Rejeitado - "Usuário terceiro não possui aprovador específico configurado"

### CT014 - Terceiro com aprovador inválido
**Pré-condições:** `tipoUsuario = "Terceiro"`, `aprovadorTerceiro` não existe  
**Resultado:** ❌ Rejeitado - "Aprovador específico para terceiro não encontrado no sistema"

### CT015 - Terceiro com aprovador válido
**Pré-condições:** `tipoUsuario = "Terceiro"`, `aprovadorTerceiro` válido e ativo  
**Resultado:** ✅ Aprovador substituído pelo aprovador de terceiro

---

## 5. ESCALONAMENTO HIERÁRQUICO

**Nota:** Escalonamento baseado no tipo do **APROVADOR** (não do solicitante). Escala quando aprovador é funcionário e está inativo.

### CT016 - Escalonamento: Sem gestor superior
**Pré-condições:** Aprovador funcionário inativo, sem gestor superior  
**Resultado:** ❌ Rejeitado - "não há gestor superior na hierarquia"

### CT017 - Escalonamento: Atingiu cargo limite
**Pré-condições:** Aprovador funcionário inativo, escalou até cargo limite sem gestor ativo  
**Resultado:** ❌ Rejeitado - "atingiu cargo limite sem encontrar gestor ativo"

### CT018 - Escalonamento: Gestor sem att_descr_status
**Pré-condições:** Durante escalonamento, gestor sem `att_descr_status`  
**Resultado:** ❌ Rejeitado - "Gestor não possui atributo obrigatório 'att_descr_status'"

### CT019 - Escalonamento: Encontrou gestor ativo
**Pré-condições:** Aprovador funcionário inativo, encontrou gestor ativo em algum nível  
**Resultado:** ✅ `roleApprover` = gestor ativo encontrado

### CT020 - Escalonamento: Terceiro com aprovador funcionário inativo
**Pré-condições:** Solicitante terceiro, aprovador funcionário inativo  
**Resultado:** ✅ Escalona acima do aprovador funcionário (mesmo sendo terceiro)

### CT021 - Escalonamento: Não aplicado (forward ativo ou aprovador terceiro)
**Pré-condições:** Forward ativo OU aprovador é terceiro  
**Resultado:** ❌ Não escala, rejeita se aprovador inativo

---

## 6. DUPLICIDADE E BLOCK LIST

### CT022 - Duplicidade não permitida
**Pré-condições:** Múltiplos perfis do mesmo sistema, `tim_permite_duplicidade = false`  
**Resultado:** ❌ Rejeitado - "o sistema não permite a atribuição de múltiplos perfis"

### CT023 - Duplicidade permitida
**Pré-condições:** Múltiplos perfis do mesmo sistema, `tim_permite_duplicidade = true`  
**Resultado:** ✅ Processado normalmente

### CT024 - Block list
**Pré-condições:** `tim_block_list = "true"`  
**Resultado:** ❌ Rejeitado - "usuário em block list"

---

## 7. AUTO-APROVAÇÃO

### CT025 - Auto-aprovação: Operação Remove
**Pré-condições:** `itemOperation = "Remove"`  
**Resultado:** ✅ Autoaprovado - "Operação de remoção de perfil não requer aprovação"

### CT026 - Auto-aprovação: Bypass (spadmin)
**Pré-condições:** Launcher = "spadmin"  
**Resultado:** ✅ Autoaprovado - "Bypass de aprovação ativado"

### CT027 - Auto-aprovação: Solicitante é o próprio aprovador
**Pré-condições:** Launcher = aprovador configurado  
**Resultado:** ✅ Autoaprovado - "Solicitante é o próprio aprovador configurado"

---

## 8. FLUXO COMPLETO

### CT028 - Fluxo completo de aprovação
**Pré-condições:** 
- Perfil válido com aprovador válido e ativo
- Sem block list, duplicidade ou forward
- Operação Add

**Resultado:** ✅ Item adicionado ao `thisApprovalSet`, Approval criado com owner correto

---

## CHECKLIST DE VALIDAÇÃO

### ✅ Cobertura de Cenários
- [ ] CT001-CT028 - Todos os casos executados
- [ ] Validações de aprovador (configurado/existe/ativo/atributos)
- [ ] Validações de workgroup e forward
- [ ] Validações de terceiro
- [ ] Escalonamento hierárquico (baseado no aprovador)
- [ ] Duplicidade e Block list
- [ ] Auto-aprovação (Remove, Bypass, Auto-aprovador)

### ✅ Validação de Mensagens
- [ ] Mensagens específicas e acionáveis
- [ ] Comentários identificam problema exato
- [ ] Logs contêm informações suficientes

### ✅ Validação de Comportamento
- [ ] Rejeições persistidas corretamente
- [ ] Auto-aprovações persistidas corretamente
- [ ] Aprovações manuais enviadas corretamente
- [ ] Argumentos do approval preenchidos

---

## ORDEM RECOMENDADA DE EXECUÇÃO

1. **Configuração de aprovador** (CT001-CT004)
2. **Validações de aprovador** (CT005-CT008)
3. **Workgroup e Forward** (CT009-CT012)
4. **Terceiros** (CT013-CT015)
5. **Escalonamento** (CT016-CT021)
6. **Duplicidade e Block List** (CT022-CT024)
7. **Auto-aprovação** (CT025-CT027)
8. **Fluxo completo** (CT028)

---

**Total de Casos de Teste: 28**

**Nota:** Execute os testes em ambiente de desenvolvimento antes de produção. Mantenha logs detalhados para análise posterior.
