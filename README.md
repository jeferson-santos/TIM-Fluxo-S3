# Casos de Teste - TimApprovalAssignmentRuleN1

## Objetivo
Documento completo de casos de teste para validar todos os cenários da regra de aprovação de primeiro nível (N1).

---

## 1. VALIDAÇÃO DE PERFIL

### CT001 - Perfil não encontrado no sistema
**Pré-condições:**
- Perfil `PERFIL_INEXISTENTE` não existe no sistema
- Solicitante válido configurado

**Ações:**
- Criar solicitação para perfil `PERFIL_INEXISTENTE`

**Resultado Esperado:**
- ✅ Solicitação rejeitada automaticamente
- ✅ Comentário: "Rejeitado automaticamente no Nível 1: Perfil não encontrado no sistema: PERFIL_INEXISTENTE"
- ✅ Log: `N1 - Perfil não encontrado no sistema: PERFIL_INEXISTENTE`

---

### CT002 - Perfil encontrado com sucesso
**Pré-condições:**
- Perfil `SAP_VENDAS` existe no sistema
- Perfil possui atributos: `tim_nome_sistema`, `tim_approvalN1`

**Ações:**
- Criar solicitação para perfil `SAP_VENDAS`

**Resultado Esperado:**
- ✅ Processamento continua normalmente
- ✅ Log: `N1 - Processando item: SAP_VENDAS (Add)`

---

## 2. CONFIGURAÇÃO DE APROVADOR NO PERFIL

### CT003 - Aprovador não configurado no perfil
**Pré-condições:**
- Perfil `SAP_VENDAS` existe
- Atributo `tim_approvalN1` = null ou vazio

**Ações:**
- Criar solicitação para perfil `SAP_VENDAS`

**Resultado Esperado:**
- ✅ Solicitação rejeitada automaticamente
- ✅ Comentário: "Rejeitado automaticamente no Nível 1: Aprovador não configurado no perfil: SAP_VENDAS"
- ✅ Log: `N1 - Aprovador não configurado no perfil: SAP_VENDAS`

---

### CT004 - Aprovador configurado como "Disabled"
**Pré-condições:**
- Perfil `SAP_VENDAS` existe
- Atributo `tim_approvalN1` = "Disabled"

**Ações:**
- Criar solicitação para perfil `SAP_VENDAS` (Add)

**Resultado Esperado:**
- ✅ Solicitação autoaprovada automaticamente
- ✅ Comentário ApprovalSummary: "Solicitação autoaprovada no Nível 1. Regra aplicada: Aprovação desabilitada no perfil (tim_approvalN1 = Disabled)"
- ✅ Log: `N1 - Item aprovado automaticamente: SAP_VENDAS - Regra: Aprovação desabilitada no perfil (tim_approvalN1 = Disabled)`

---

### CT005 - Aprovador configurado como "Manager"
**Pré-condições:**
- Perfil `SAP_VENDAS` existe
- Atributo `tim_approvalN1` = "Manager"
- Solicitante possui gestor definido (`identity.getManager() != null`)

**Ações:**
- Criar solicitação para perfil `SAP_VENDAS`
- Solicitante: `joao.silva` com gestor `maria.santos`

**Resultado Esperado:**
- ✅ `roleApprover` convertido para `maria.santos`
- ✅ Log: `N1 - Aprovador 'Manager' convertido para: maria.santos`
- ✅ Processamento continua validando o gestor

---

### CT006 - Aprovador "Manager" mas solicitante sem gestor
**Pré-condições:**
- Perfil `SAP_VENDAS` existe
- Atributo `tim_approvalN1` = "Manager"
- Solicitante não possui gestor (`identity.getManager() == null`)

**Ações:**
- Criar solicitação para perfil `SAP_VENDAS`
- Solicitante: `usuario.sem.gestor`

**Resultado Esperado:**
- ✅ Solicitação rejeitada automaticamente
- ✅ Comentário: "Rejeitado automaticamente no Nível 1: Perfil configurado para usar gestor, mas solicitante não possui gestor definido"
- ✅ Log: `N1 - Perfil configurado para usar gestor, mas solicitante não possui gestor definido`

---

### CT007 - Aprovador configurado com nome específico
**Pré-condições:**
- Perfil `SAP_VENDAS` existe
- Atributo `tim_approvalN1` = "aprovador.especifico"
- Usuário `aprovador.especifico` existe e está ativo

**Ações:**
- Criar solicitação para perfil `SAP_VENDAS`

**Resultado Esperado:**
- ✅ `roleApprover` = "aprovador.especifico"
- ✅ Processamento continua validando o aprovador

---

## 3. VALIDAÇÃO DE APROVADOR NO SISTEMA

### CT008 - Aprovador configurado não encontrado no sistema
**Pré-condições:**
- Perfil `SAP_VENDAS` existe
- Atributo `tim_approvalN1` = "aprovador.inexistente"
- Usuário `aprovador.inexistente` NÃO existe no sistema

**Ações:**
- Criar solicitação para perfil `SAP_VENDAS`

**Resultado Esperado:**
- ✅ Solicitação rejeitada automaticamente
- ✅ Comentário: "Rejeitado automaticamente no Nível 1: Aprovador configurado não encontrado no sistema: aprovador.inexistente"
- ✅ Log: `N1 - Aprovador configurado não encontrado no sistema: aprovador.inexistente`

---

### CT009 - Aprovador existe mas está inativo
**Pré-condições:**
- Perfil `SAP_VENDAS` existe
- Atributo `tim_approvalN1` = "aprovador.inativo"
- Usuário `aprovador.inativo` existe mas `isInactive() == true`

**Ações:**
- Criar solicitação para perfil `SAP_VENDAS`

**Resultado Esperado:**
- ✅ Solicitação rejeitada automaticamente
- ✅ Comentário: "Rejeitado automaticamente no Nível 1: Aprovador configurado está inativo no sistema: aprovador.inativo"
- ✅ Log: `N1 - Aprovador configurado está inativo no sistema: aprovador.inativo`

---

### CT010 - Aprovador existe mas sem atributo att_descr_status
**Pré-condições:**
- Perfil `SAP_VENDAS` existe
- Atributo `tim_approvalN1` = "aprovador.sem.status"
- Usuário `aprovador.sem.status` existe mas `att_descr_status` = null ou vazio

**Ações:**
- Criar solicitação para perfil `SAP_VENDAS`

**Resultado Esperado:**
- ✅ Solicitação rejeitada automaticamente
- ✅ Comentário: "Rejeitado automaticamente no Nível 1: Aprovador configurado não possui atributo obrigatório 'att_descr_status' preenchido: aprovador.sem.status"
- ✅ Log: `N1 - Aprovador configurado não possui atributo obrigatório 'att_descr_status' preenchido: aprovador.sem.status`

---

### CT011 - Aprovador válido e ativo
**Pré-condições:**
- Perfil `SAP_VENDAS` existe
- Atributo `tim_approvalN1` = "aprovador.valido"
- Usuário `aprovador.valido` existe, está ativo (`isInactive() == false`)
- `att_descr_status` = "Ativo"

**Ações:**
- Criar solicitação para perfil `SAP_VENDAS`

**Resultado Esperado:**
- ✅ Solicitação enviada para aprovação
- ✅ Aprovador definido: `aprovador.valido`
- ✅ Log: `N1 - Item enviado para aprovação: SAP_VENDAS - Aprovador: aprovador.valido`

---

## 4. VALIDAÇÃO DE WORKGROUP

### CT012 - Workgroup sem membros ativos
**Pré-condições:**
- Perfil `SAP_VENDAS` existe
- Atributo `tim_approvalN1` = "GRP_APROVADORES"
- `GRP_APROVADORES` é um workgroup
- Workgroup não possui membros ativos ou todos estão inativos

**Ações:**
- Criar solicitação para perfil `SAP_VENDAS`

**Resultado Esperado:**
- ✅ Solicitação rejeitada automaticamente
- ✅ Comentário: "Rejeitado automaticamente no Nível 1: Grupo de aprovadores não possui membros ativos: GRP_APROVADORES"
- ✅ Log: `N1 - Grupo de aprovadores não possui membros ativos: GRP_APROVADORES`

---

### CT013 - Workgroup com membros ativos
**Pré-condições:**
- Perfil `SAP_VENDAS` existe
- Atributo `tim_approvalN1` = "GRP_APROVADORES"
- `GRP_APROVADORES` é um workgroup
- Workgroup possui pelo menos um membro ativo

**Ações:**
- Criar solicitação para perfil `SAP_VENDAS`

**Resultado Esperado:**
- ✅ Solicitação enviada para aprovação
- ✅ Aprovador definido: `GRP_APROVADORES`
- ✅ Log: `N1 - Workgroup GRP_APROVADORES possui membros ativos: true`
- ✅ Log: `N1 - Item enviado para aprovação: SAP_VENDAS - Aprovador: GRP_APROVADORES`

---

## 5. VALIDAÇÃO DE FORWARD (DELEGAÇÃO)

### CT014 - Forward configurado mas usuário não encontrado
**Pré-condições:**
- Perfil `SAP_VENDAS` existe
- Atributo `tim_approvalN1` = "aprovador.principal"
- `aprovador.principal` possui preferência `forward` = "usuario.inexistente"
- Usuário `usuario.inexistente` NÃO existe no sistema

**Ações:**
- Criar solicitação para perfil `SAP_VENDAS`

**Resultado Esperado:**
- ✅ Solicitação rejeitada automaticamente
- ✅ Comentário: "Rejeitado automaticamente no Nível 1: Aprovador possui delegação (forward) configurada, mas usuário não encontrado no sistema: usuario.inexistente"
- ✅ Log: `N1 - Aprovador possui delegação (forward) configurada, mas usuário não encontrado no sistema: usuario.inexistente`

---

### CT015 - Forward configurado mas usuário sem att_descr_status
**Pré-condições:**
- Perfil `SAP_VENDAS` existe
- Atributo `tim_approvalN1` = "aprovador.principal"
- `aprovador.principal` possui preferência `forward` = "usuario.sem.status"
- Usuário `usuario.sem.status` existe mas `att_descr_status` = null

**Ações:**
- Criar solicitação para perfil `SAP_VENDAS`

**Resultado Esperado:**
- ✅ Solicitação rejeitada automaticamente
- ✅ Comentário: "Rejeitado automaticamente no Nível 1: Aprovador possui delegação (forward) configurada, mas usuário não possui atributo 'att_descr_status' preenchido: usuario.sem.status"
- ✅ Log: `N1 - Aprovador possui delegação (forward) configurada, mas usuário não possui atributo 'att_descr_status' preenchido: usuario.sem.status`

---

### CT016 - Forward configurado e usuário inativo
**Pré-condições:**
- Perfil `SAP_VENDAS` existe
- Atributo `tim_approvalN1` = "aprovador.principal"
- `aprovador.principal` possui preferência `forward` = "usuario.inativo"
- Usuário `usuario.inativo` existe mas `att_descr_status` = "Inativo"

**Ações:**
- Criar solicitação para perfil `SAP_VENDAS`

**Resultado Esperado:**
- ✅ Solicitação rejeitada automaticamente
- ✅ Comentário: "Rejeitado automaticamente no Nível 1: Aprovador configurado possui delegação (forward) para usuário inativo: usuario.inativo"
- ✅ Log: `N1 - Aprovador configurado possui delegação (forward) para usuário inativo: usuario.inativo`

---

### CT017 - Forward configurado e usuário ativo
**Pré-condições:**
- Perfil `SAP_VENDAS` existe
- Atributo `tim_approvalN1` = "aprovador.principal"
- `aprovador.principal` possui preferência `forward` = "usuario.ativo"
- Usuário `usuario.ativo` existe e `att_descr_status` = "Ativo"

**Ações:**
- Criar solicitação para perfil `SAP_VENDAS`

**Resultado Esperado:**
- ✅ Forward aceito
- ✅ `hasFoward = true`
- ✅ Log: `N1 - Aprovador aprovador.principal possui forward ativo: usuario.ativo`
- ✅ Processamento continua (forward válido não bloqueia)

---

### CT018 - Forward não configurado
**Pré-condições:**
- Perfil `SAP_VENDAS` existe
- Atributo `tim_approvalN1` = "aprovador.principal"
- `aprovador.principal` não possui preferência `forward`

**Ações:**
- Criar solicitação para perfil `SAP_VENDAS`

**Resultado Esperado:**
- ✅ Processamento continua normalmente
- ✅ Não há validação de forward
- ✅ Validação direta do aprovador

---

## 6. TRATAMENTO DE TERCEIROS

### CT019 - Terceiro com aprovador configurado
**Pré-condições:**
- Perfil `SAP_VENDAS` existe
- Solicitante: `tipoUsuario` = "Terceiro"
- Variável `aprovadorTerceiro` configurada e aponta para usuário válido
- Usuário apontado por `aprovadorTerceiro` existe e está ativo

**Ações:**
- Criar solicitação para perfil `SAP_VENDAS`
- Solicitante: terceiro com `aprovadorTerceiro` configurado

**Resultado Esperado:**
- ✅ Aprovador substituído pelo aprovador de terceiro
- ✅ Log: `N1 - Aprovador para terceiro: {nomeAprovadorTerceiro}`
- ✅ Processamento continua com o aprovador de terceiro

---

### CT020 - Terceiro sem aprovador configurado
**Pré-condições:**
- Perfil `SAP_VENDAS` existe
- Solicitante: `tipoUsuario` = "Terceiro"
- Variável `aprovadorTerceiro` = null ou não configurada

**Ações:**
- Criar solicitação para perfil `SAP_VENDAS`
- Solicitante: terceiro sem `aprovadorTerceiro`

**Resultado Esperado:**
- ✅ Solicitação rejeitada automaticamente
- ✅ Comentário: "Rejeitado automaticamente no Nível 1: Usuário terceiro não possui aprovador específico configurado na solicitação"
- ✅ Log: `N1 - Usuário terceiro não possui aprovador específico configurado na solicitação`

---

### CT021 - Terceiro com aprovador configurado mas usuário não encontrado
**Pré-condições:**
- Perfil `SAP_VENDAS` existe
- Solicitante: `tipoUsuario` = "Terceiro"
- Variável `aprovadorTerceiro` configurada mas usuário não existe

**Ações:**
- Criar solicitação para perfil `SAP_VENDAS`
- Solicitante: terceiro com `aprovadorTerceiro` inválido

**Resultado Esperado:**
- ✅ Solicitação rejeitada automaticamente
- ✅ Comentário: "Rejeitado automaticamente no Nível 1: Aprovador específico para terceiro configurado na solicitação não foi encontrado no sistema"
- ✅ Log: `N1 - Aprovador específico para terceiro configurado na solicitação não foi encontrado no sistema`

---

## 7. ESCALONAMENTO HIERÁRQUICO (apenas Funcionário)

### CT022 - Escalonamento: Sem gestor superior na hierarquia
**Pré-condições:**
- Perfil `SAP_VENDAS` existe
- Solicitante: `tipoUsuario` = "Funcionario"
- Aprovador inicial não está ativo (`att_descr_status` != "Ativo")
- Aprovador inicial não possui forward
- Aprovador inicial não possui gestor superior (`getManager() == null`)

**Ações:**
- Criar solicitação para perfil `SAP_VENDAS`

**Resultado Esperado:**
- ✅ Solicitação rejeitada automaticamente
- ✅ Comentário: "Rejeitado automaticamente no Nível 1: Escalonamento hierárquico interrompido: não há gestor superior na hierarquia"
- ✅ Log: `N1 - Escalonamento hierárquico interrompido: não há gestor superior na hierarquia`

---

### CT023 - Escalonamento: Atingiu cargo limite sem gestor ativo
**Pré-condições:**
- Perfil `SAP_VENDAS` existe
- Solicitante: `tipoUsuario` = "Funcionario"
- Aprovador inicial não está ativo
- Escalonamento sobe na hierarquia
- Gestor em algum nível tem `att_cargo` = "DIRETOR" (ou outro cargo limite)
- Esse gestor não está ativo

**Ações:**
- Criar solicitação para perfil `SAP_VENDAS`

**Resultado Esperado:**
- ✅ Solicitação rejeitada automaticamente
- ✅ Comentário: "Rejeitado automaticamente no Nível 1: Escalonamento hierárquico atingiu cargo limite (DIRETOR) sem encontrar gestor ativo"
- ✅ Log: `N1 - Escalonamento hierárquico atingiu cargo limite (DIRETOR) sem encontrar gestor ativo`

---

### CT024 - Escalonamento: Gestor sem att_descr_status
**Pré-condições:**
- Perfil `SAP_VENDAS` existe
- Solicitante: `tipoUsuario` = "Funcionario"
- Aprovador inicial não está ativo
- Durante escalonamento, gestor encontrado não possui `att_descr_status`

**Ações:**
- Criar solicitação para perfil `SAP_VENDAS`

**Resultado Esperado:**
- ✅ Solicitação rejeitada automaticamente
- ✅ Comentário: "Rejeitado automaticamente no Nível 1: Gestor na hierarquia não possui atributo obrigatório 'att_descr_status' preenchido: {nomeGestor}"
- ✅ Log: `N1 - Gestor na hierarquia não possui atributo obrigatório 'att_descr_status' preenchido: {nomeGestor}`

---

### CT025 - Escalonamento: Gestor sem att_cargo (continua escalonamento)
**Pré-condições:**
- Perfil `SAP_VENDAS` existe
- Solicitante: `tipoUsuario` = "Funcionario"
- Aprovador inicial não está ativo
- Durante escalonamento, gestor encontrado não possui `att_cargo` ou está vazio

**Ações:**
- Criar solicitação para perfil `SAP_VENDAS`

**Resultado Esperado:**
- ✅ Escalonamento continua (cargo não é obrigatório)
- ✅ Log: `N1 - Gestor {nome} não possui cargo (att_cargo) preenchido, continuando escalonamento`
- ✅ Processamento continua para próximo nível

---

### CT026 - Escalonamento: Encontrou gestor ativo
**Pré-condições:**
- Perfil `SAP_VENDAS` existe
- Solicitante: `tipoUsuario` = "Funcionario"
- Aprovador inicial não está ativo
- Escalonamento sobe 3 níveis
- No 3º nível encontra gestor com `att_descr_status` = "Ativo"

**Ações:**
- Criar solicitação para perfil `SAP_VENDAS`

**Resultado Esperado:**
- ✅ `managerActive` = nome do gestor encontrado
- ✅ `roleApprover` = gestor ativo
- ✅ Log: `N1 - Gestor ativo encontrado: {nomeGestor} (nível 3)`
- ✅ Solicitação enviada para aprovação com o gestor escalado

---

### CT027 - Escalonamento: Não encontrou gestor ativo após N níveis
**Pré-condições:**
- Perfil `SAP_VENDAS` existe
- Solicitante: `tipoUsuario` = "Funcionario"
- Aprovador inicial não está ativo
- Escalonamento sobe vários níveis mas nenhum está ativo
- Não atingiu cargo limite e não chegou ao topo

**Ações:**
- Criar solicitação para perfil `SAP_VENDAS`

**Resultado Esperado:**
- ✅ Solicitação rejeitada automaticamente
- ✅ Comentário: "Rejeitado automaticamente no Nível 1: Escalonamento hierárquico não encontrou gestor ativo após {X} níveis"
- ✅ Log: `N1 - Escalonamento hierárquico não encontrou gestor ativo após {X} níveis`

---

### CT028 - Escalonamento: Não aplicado para terceiro
**Pré-condições:**
- Perfil `SAP_VENDAS` existe
- Solicitante: `tipoUsuario` = "Terceiro"
- Aprovador inicial não está ativo

**Ações:**
- Criar solicitação para perfil `SAP_VENDAS`

**Resultado Esperado:**
- ✅ Escalonamento NÃO é executado
- ✅ Processamento segue com aprovador original ou rejeita se inválido

---

### CT029 - Escalonamento: Não aplicado quando tem forward ativo
**Pré-condições:**
- Perfil `SAP_VENDAS` existe
- Solicitante: `tipoUsuario` = "Funcionario"
- Aprovador inicial não está ativo mas possui forward ativo

**Ações:**
- Criar solicitação para perfil `SAP_VENDAS`

**Resultado Esperado:**
- ✅ Escalonamento NÃO é executado (forward ativo impede)
- ✅ Processamento continua com forward

---

## 8. VALIDAÇÃO DE DUPLICIDADE

### CT030 - Duplicidade: Múltiplos perfis do mesmo sistema (não permitido)
**Pré-condições:**
- Dois perfis: `SAP_VENDAS` e `SAP_VENDAS_AVANCADO`
- Ambos têm `tim_nome_sistema` = "SAP"
- Perfil `SAP_VENDAS` não permite duplicidade (`tim_permite_duplicidade` = false)
- Operação: Add

**Ações:**
- Criar solicitação com ambos os perfis na mesma requisição

**Resultado Esperado:**
- ✅ Solicitação rejeitada automaticamente por duplicidade
- ✅ Comentário: "Rejeitado automaticamente no Nível 1: o sistema não permite a atribuição de múltiplos perfis"
- ✅ Log: `N1 - AUTO-REJECT: Perfil duplicado não permitido - {nomePerfil}`
- ✅ Log: `N1 - Sistema duplicado detectado: SAP`

---

### CT031 - Duplicidade: Múltiplos perfis do mesmo sistema (permitido)
**Pré-condições:**
- Dois perfis: `SAP_VENDAS` e `SAP_VENDAS_AVANCADO`
- Ambos têm `tim_nome_sistema` = "SAP"
- Perfil permite duplicidade (`tim_permite_duplicidade` = true)
- Operação: Add

**Ações:**
- Criar solicitação com ambos os perfis na mesma requisição

**Resultado Esperado:**
- ✅ Solicitação processada normalmente
- ✅ Cada perfil avaliado independentemente
- ✅ Não há rejeição por duplicidade

---

### CT032 - Duplicidade: Perfis de sistemas diferentes
**Pré-condições:**
- Dois perfis: `SAP_VENDAS` (sistema: SAP) e `ORACLE_ERP` (sistema: ORACLE)

**Ações:**
- Criar solicitação com ambos os perfis

**Resultado Esperado:**
- ✅ Solicitação processada normalmente
- ✅ Não há duplicidade (sistemas diferentes)
- ✅ Log: `N1 - Total de sistemas: 2`
- ✅ Log: `N1 - Sistemas duplicados: []`

---

## 9. BLOCK LIST

### CT033 - Usuário em block list
**Pré-condições:**
- Solicitante existe
- Atributo `tim_block_list` = "true"

**Ações:**
- Criar qualquer solicitação para o usuário

**Resultado Esperado:**
- ✅ Solicitação rejeitada automaticamente
- ✅ Comentário: "Rejeitado automaticamente no Nível 1: usuário em block list. Para maiores informações entrar em contato com: dl_cfo_antifraudecontroledelogins@timbrasil.com.br"
- ✅ Log: `N1 - BLOCK LIST: Usuário {identityName} está na lista de bloqueio`
- ✅ Log: `N1 - Rejeição por block list: {itemValue}`

---

### CT034 - Usuário não está em block list
**Pré-condições:**
- Solicitante existe
- Atributo `tim_block_list` != "true" ou null

**Ações:**
- Criar solicitação para o usuário

**Resultado Esperado:**
- ✅ Processamento continua normalmente
- ✅ Não há rejeição por block list

---

## 10. AUTO-APROVAÇÃO

### CT035 - Auto-aprovação: Operação Remove
**Pré-condições:**
- Perfil `SAP_VENDAS` existe
- Operação: Remove (não Add)

**Ações:**
- Criar solicitação para remover perfil `SAP_VENDAS`

**Resultado Esperado:**
- ✅ Solicitação autoaprovada automaticamente
- ✅ Comentário ApprovalSummary: "Solicitação autoaprovada no Nível 1. Regra aplicada: Operação de remoção de perfil não requer aprovação"
- ✅ Log: `N1 - Item aprovado automaticamente: SAP_VENDAS - Regra: Operação de remoção de perfil não requer aprovação`

---

### CT036 - Auto-aprovação: Bypass (spadmin)
**Pré-condições:**
- Perfil `SAP_VENDAS` existe
- Launcher = "spadmin"

**Ações:**
- Criar solicitação iniciada por spadmin

**Resultado Esperado:**
- ✅ Solicitação autoaprovada automaticamente
- ✅ `byPassApprovals = true`
- ✅ Comentário ApprovalSummary: "Solicitação autoaprovada no Nível 1. Regra aplicada: Bypass de aprovação ativado (solicitação iniciada por administrador)"
- ✅ Log: `N1 - Bypass ativado: solicitação iniciada por spadmin`
- ✅ Log: `N1 - Item aprovado automaticamente: SAP_VENDAS - Regra: Bypass de aprovação ativado (solicitação iniciada por administrador)`

---

### CT037 - Auto-aprovação: Solicitante é o próprio aprovador
**Pré-condições:**
- Perfil `SAP_VENDAS` existe
- Atributo `tim_approvalN1` = "joao.silva"
- Launcher = "joao.silva" (mesmo usuário)

**Ações:**
- Usuário `joao.silva` solicita perfil onde ele é o aprovador

**Resultado Esperado:**
- ✅ Solicitação autoaprovada automaticamente
- ✅ Comentário ApprovalSummary: "Solicitação autoaprovada no Nível 1. Regra aplicada: Solicitante é o próprio aprovador configurado"
- ✅ Log: `N1 - Item aprovado automaticamente: SAP_VENDAS - Regra: Solicitante é o próprio aprovador configurado`

---

### CT038 - Auto-aprovação: Múltiplas regras (consolidação)
**Pré-condições:**
- Dois perfis: `SAP_VENDAS` (Disabled) e `ORACLE_ERP` (Remove)

**Ações:**
- Criar solicitação com ambos os perfis

**Resultado Esperado:**
- ✅ Comentário ApprovalSummary: "Solicitação autoaprovada no Nível 1. Regras aplicadas: Aprovação desabilitada no perfil (tim_approvalN1 = Disabled); Operação de remoção de perfil não requer aprovação"
- ✅ Ambos autoaprovados

---

## 11. FLUXO COMPLETO - APROVAÇÃO NORMAL

### CT039 - Fluxo completo: Aprovação enviada com sucesso
**Pré-condições:**
- Perfil `SAP_VENDAS` existe
- Atributo `tim_approvalN1` = "aprovador.valido"
- Aprovador existe, está ativo, não tem forward
- Solicitante não está em block list
- Não há duplicidade
- Operação: Add

**Ações:**
- Criar solicitação para perfil `SAP_VENDAS`

**Resultado Esperado:**
- ✅ Item adicionado ao `thisApprovalSet`
- ✅ Approval criado com owner = "aprovador.valido"
- ✅ Approval contém argumentos: `gestorFuncional`, `userLogin`, `managerLogin`, `managerDescUnidOrg`
- ✅ Log: `N1 - Item enviado para aprovação: SAP_VENDAS - Aprovador: aprovador.valido`
- ✅ Solicitação aguarda aprovação manual

---

## 12. LOGS E RASTREABILIDADE

### CT040 - Logs de início e fim
**Ações:**
- Qualquer solicitação

**Resultado Esperado:**
- ✅ Log inicial: `========== INÍCIO: TimApprovalAssignmentRuleN1 ==========`
- ✅ Log final: `========== FIM: TimApprovalAssignmentRuleN1 ==========`

---

### CT041 - Logs de processamento
**Pré-condições:**
- Solicitante válido com gestor
- Perfil válido

**Ações:**
- Criar solicitação

**Resultado Esperado:**
- ✅ Log: `N1 - Solicitante: {displayName} ({userLogin}) - Tipo: {tipoUsuario}`
- ✅ Log: `N1 - Gestor: {managerDisplayName} ({managerName})`
- ✅ Log: `N1 - Processando item: {nomePerfil} ({itemOperation})`
- ✅ Logs apropriados para cada validação

---

## 13. VALIDAÇÕES DE ATRIBUTOS OBRIGATÓRIOS

### CT042 - att_tipoFunc ausente (se usado)
**Pré-condições:**
- Solicitante existe mas `att_tipoFunc` = null
- Validação pode afetar tratamento de terceiro ou escalonamento

**Ações:**
- Criar solicitação

**Resultado Esperado:**
- ✅ Processamento continua (tipoFunc não é bloqueador, apenas afeta lógica)
- ✅ Log mostra `tipoUsuario` = null

---

## CHECKLIST DE VALIDAÇÃO

### ✅ Cobertura de Cenários
- [ ] CT001 até CT042 - Todos os casos executados
- [ ] Validações de perfil (existente/inexistente)
- [ ] Validações de aprovador (configurado/não configurado/existe/não existe/ativo/inativo)
- [ ] Validações de workgroup (membros ativos/inativos)
- [ ] Validações de forward (configurado/não configurado/ativo/inativo/sem atributo)
- [ ] Validações de terceiro (com/sem aprovador)
- [ ] Escalonamento hierárquico (todos os cenários)
- [ ] Duplicidade de sistemas
- [ ] Block list
- [ ] Auto-aprovação (todas as regras)
- [ ] Atributos obrigatórios (att_descr_status, att_cargo)

### ✅ Validação de Mensagens
- [ ] Todas as mensagens são específicas (não genéricas)
- [ ] Mensagens identificam o problema exato
- [ ] Comentários são claros e acionáveis
- [ ] Logs contêm informações suficientes para troubleshooting

### ✅ Validação de Comportamento
- [ ] Rejeições são persistidas corretamente
- [ ] Auto-aprovações são persistidas corretamente
- [ ] Aprovações manuais são enviadas corretamente
- [ ] Argumentos do approval estão preenchidos

---

## ORDEM RECOMENDADA DE EXECUÇÃO

1. **Cenários básicos de perfil** (CT001-CT007)
2. **Validações de aprovador** (CT008-CT011)
3. **Workgroup e Forward** (CT012-CT018)
4. **Terceiros** (CT019-CT021)
5. **Escalonamento** (CT022-CT029)
6. **Duplicidade e Block List** (CT030-CT034)
7. **Auto-aprovação** (CT035-CT038)
8. **Fluxo completo** (CT039)
9. **Validações complementares** (CT040-CT042)

---

**Nota:** Execute os testes em ambiente de desenvolvimento antes de produção. Mantenha logs detalhados para análise posterior.

