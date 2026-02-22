# Análise dos Índices Atuais - IdentityIQ

## 📊 Resumo Executivo

**Data da Análise:** 2026-02-21  
**Status Geral:** ⚠️ **PARCIALMENTE OTIMIZADO** - Alguns índices críticos já foram criados, mas ainda faltam otimizações importantes.

---

## ✅ Índices Já Criados (Bom!)

### 1. spt_bundle_profile_relation

#### ✅ `idx_bundle_created` (JÁ CRIADO!)
```
Índice: idx_bundle_created
Colunas: bundle_id, created DESC
Cardinalidade: bundle_id=50842, created=13356175
Status: ✅ PERFEITO para Query #1
```

**Análise:**
- ✅ **EXCELENTE!** Este índice foi criado e está otimizando a Query #1
- ✅ Ordem correta: `bundle_id` primeiro, depois `created DESC`
- ✅ Cardinalidade alta indica boa seletividade
- ✅ Deve melhorar drasticamente o `ORDER BY created DESC`

#### Outros Índices Existentes:
- `PRIMARY` em `id` ✅
- `spt_bpr_bundle_id_hash_code` (bundle_id, hash_code) ✅
- `FKmagmq7lgmic1artsln48lpcaw` em `source_application` ✅
- Vários índices em outras colunas para outras queries ✅

**Recomendação:** ✅ **Nenhuma ação necessária** - índices adequados para esta tabela.

---

### 2. spt_bundle_profile_relation_object

#### ⚠️ `idx_type_modified` (CRIADO COM PREFIXO)
```
Índice: idx_type_modified
Colunas: type, modified_id(100) - PREFIXO DE 100 CARACTERES
Cardinalidade: type=1, modified_id=6256
Status: ⚠️ PODE SER MELHORADO
```

**Análise:**
- ⚠️ **PROBLEMA:** Prefixo de apenas 100 caracteres em `modified_id` (VARCHAR 1024)
- ⚠️ Ordem das colunas: `type` primeiro, depois `modified_id`
- ⚠️ Para a Query #1, o filtro é `WHERE type = 'TYPE_INSERT' AND modified_id = ?`
- ✅ A ordem `type, modified_id` está correta para este filtro
- ⚠️ **MAS:** Prefixo de 100 pode não ser suficiente para garantir unicidade

**Query Problemática:**
```sql
WHERE bundleprof0_.bundle_id = bundleprof1_.modified_id 
  AND bundleprof1_.type = 'TYPE_INSERT'
```

**Problema Identificado:**
- O JOIN usa `bundle_id = modified_id`
- O índice atual tem `type` primeiro, mas o JOIN é por `modified_id`
- **Pode não estar sendo usado eficientemente para o JOIN**

**Recomendações:**

1. **Verificar se o prefixo de 100 é suficiente:**
   ```sql
   SELECT 
       COUNT(DISTINCT modified_id) AS total_unique,
       COUNT(DISTINCT LEFT(modified_id, 100)) AS unique_in_first_100,
       ROUND((COUNT(DISTINCT LEFT(modified_id, 100)) / COUNT(DISTINCT modified_id)) * 100, 2) AS coverage_percent
   FROM spt_bundle_profile_relation_object
   WHERE type = 'TYPE_INSERT';
   ```

2. **Criar índice adicional otimizado para o JOIN:**
   ```sql
   -- Índice específico para o JOIN (modified_id primeiro)
   CREATE INDEX idx_modified_id_for_join 
   ON spt_bundle_profile_relation_object(modified_id(650))
   ALGORITHM=INPLACE, LOCK=NONE;
   ```

3. **OU melhorar o índice existente (se possível):**
   - Aumentar prefixo de 100 para 650 caracteres
   - Mas isso requer recriar o índice

**Status:** ⚠️ **MELHORIA RECOMENDADA**

---

### 3. spt_identity

#### ✅ `PRIMARY KEY` em `id` (JÁ EXISTE)
```
Índice: PRIMARY
Coluna: id
Cardinalidade: 575460
Status: ✅ PERFEITO para Query #2
```

**Análise:**
- ✅ **EXCELENTE!** A Query #2 usa `WHERE id = ?` e já tem PRIMARY KEY
- ✅ O problema da Query #2 **NÃO é falta de índice**, mas sim **locks longos**
- ✅ A query já está otimizada do ponto de vista de índice

**Outros Índices Relevantes:**
- ✅ `spt_identity_isworkgroup` em `workgroup` - útil para Query #147
- ✅ `spt_identity_created` e `spt_identity_modified` - para ordenações
- ✅ Muitos outros índices para diferentes queries

**Recomendação:** ✅ **Nenhuma ação de índice necessária** - o problema é de locks, não de índices.

---

### 4. spt_identity_assigned_roles

#### ⚠️ FALTA ÍNDICE PARA QUERY #147
```
Índices Existentes:
- PRIMARY (identity_id, idx) ✅
- FKheohgr0xuxklx9sfhjde58ig9 em bundle ✅
```

**Query Problemática #147:**
```sql
SELECT COUNT(identity0_.id) 
FROM spt_identity identity0_ 
INNER JOIN spt_identity_assigned_roles assignedro1_ 
    ON identity0_.id = assignedro1_.identity_id 
INNER JOIN spt_bundle bundle2_ 
    ON assignedro1_.bundle = bundle2_.id 
WHERE bundle2_.id = '...' 
  AND identity0_.workgroup <> 1;
```

**Análise:**
- ✅ Tem índice em `bundle` (FKheohgr0xuxklx9sfhjde58ig9)
- ✅ Tem PRIMARY KEY em `identity_id` (primeira coluna)
- ⚠️ **PROBLEMA:** O JOIN é `identity0_.id = assignedro1_.identity_id`
- ✅ Como `identity_id` é primeira coluna da PRIMARY KEY, o índice já existe
- ⚠️ **MAS:** Para otimizar o JOIN + filtro, um índice composto seria melhor

**Recomendações:**

1. **Índice composto para otimizar a query completa:**
   ```sql
   -- Já existe PRIMARY KEY em (identity_id, idx)
   -- Mas podemos criar índice específico para o JOIN com bundle
   CREATE INDEX idx_identity_bundle 
   ON spt_identity_assigned_roles(identity_id, bundle)
   ALGORITHM=INPLACE, LOCK=NONE;
   ```

2. **OU verificar se o índice em bundle é suficiente:**
   - Se a query filtra primeiro por `bundle`, o índice atual pode ser suficiente
   - Mas o JOIN com `identity_id` pode se beneficiar de índice composto

**Status:** ⚠️ **MELHORIA OPCIONAL** (pode já estar funcionando bem)

---

### 5. spt_request

#### ❌ FALTAM ÍNDICES PARA UPDATEs LENTOS
```
Índices Existentes:
- PRIMARY em id ✅
- Vários índices em outras colunas ✅
- ❌ NÃO TEM índices em: modified, created, significant_modified
```

**Problema Identificado:**
- ⚠️ UPDATEs em `spt_request` são lentos (13-14s cada)
- ⚠️ Não há índices nas colunas que são atualizadas: `modified`, `created`, `significant_modified`
- ⚠️ Se houver queries filtrando por essas colunas, podem estar lentas

**Recomendações:**

1. **Criar índices se houver queries filtrando por essas colunas:**
   ```sql
   -- Verificar se há queries usando essas colunas no WHERE
   -- Se sim, criar índices:
   CREATE INDEX idx_request_modified 
   ON spt_request(modified)
   ALGORITHM=INPLACE, LOCK=NONE;
   
   CREATE INDEX idx_request_created 
   ON spt_request(created)
   ALGORITHM=INPLACE, LOCK=NONE;
   
   CREATE INDEX idx_request_significant_modified 
   ON spt_request(significant_modified)
   ALGORITHM=INPLACE, LOCK=NONE;
   ```

2. **NOTA:** Se os UPDATEs são por `id` (PRIMARY KEY), os índices acima não ajudam diretamente
   - O problema pode ser o tamanho do campo `attributes` (12KB XML)
   - Ou contenção de locks

**Status:** ⚠️ **ANÁLISE NECESSÁRIA** - Verificar se há queries filtrando por essas colunas

---

### 6. spt_bundle

#### ✅ Índices Adequados
```
Índices Existentes:
- PRIMARY em id ✅
- UK_smf7ppq8j0o6ijtrhh7ga9ck3 em name ✅
- Vários outros índices ✅
```

**Análise:**
- ✅ PRIMARY KEY em `id` é usado no JOIN da Query #147
- ✅ Índices adequados para as queries

**Status:** ✅ **Nenhuma ação necessária**

---

## 📋 Resumo por Prioridade

### 🔴 CRÍTICO - Ação Imediata

1. **spt_bundle_profile_relation_object:**
   - ⚠️ Verificar se prefixo de 100 caracteres é suficiente
   - ⚠️ Considerar criar índice adicional em `modified_id` para otimizar JOIN

### 🟡 ALTA - Esta Semana

2. **spt_identity_assigned_roles:**
   - ⚠️ Considerar índice composto `(identity_id, bundle)` para Query #147

3. **spt_request:**
   - ⚠️ Verificar se há queries filtrando por `modified`, `created`, `significant_modified`
   - ⚠️ Se sim, criar índices nessas colunas

### 🟢 BAIXA - Monitoramento

4. **Monitorar performance após implementações**
5. **Revisar slow query log periodicamente**

---

## 🎯 Ações Recomendadas

### 1. Verificar Eficiência do Índice Atual

```sql
-- Verificar coverage do prefixo de 100 caracteres
SELECT 
    COUNT(DISTINCT modified_id) AS total_unique,
    COUNT(DISTINCT LEFT(modified_id, 100)) AS unique_in_first_100,
    ROUND((COUNT(DISTINCT LEFT(modified_id, 100)) / COUNT(DISTINCT modified_id)) * 100, 2) AS coverage_percent
FROM spt_bundle_profile_relation_object
WHERE type = 'TYPE_INSERT';
```

**Se coverage < 95%:** Criar índice adicional em `modified_id` com prefixo maior

### 2. Criar Índice Adicional para JOIN

```sql
-- Índice otimizado para o JOIN da Query #1
CREATE INDEX idx_modified_id_for_join 
ON spt_bundle_profile_relation_object(modified_id(650))
ALGORITHM=INPLACE, LOCK=NONE;
```

### 3. Verificar Queries em spt_request

```sql
-- Verificar se há queries filtrando por modified/created
-- (Consultar slow query log ou performance schema)
SELECT sql_text 
FROM performance_schema.events_statements_history_long
WHERE sql_text LIKE '%spt_request%'
  AND (sql_text LIKE '%WHERE%modified%' 
       OR sql_text LIKE '%WHERE%created%'
       OR sql_text LIKE '%WHERE%significant_modified%')
LIMIT 20;
```

### 4. Criar Índices em spt_request (se necessário)

```sql
-- Apenas se houver queries filtrando por essas colunas
CREATE INDEX idx_request_modified 
ON spt_request(modified)
ALGORITHM=INPLACE, LOCK=NONE;

CREATE INDEX idx_request_created 
ON spt_request(created)
ALGORITHM=INPLACE, LOCK=NONE;
```

---

## 📊 Comparação: Recomendado vs. Atual

| Tabela | Índice Recomendado | Status Atual | Ação |
|--------|-------------------|--------------|------|
| `spt_bundle_profile_relation` | `idx_bundle_created` | ✅ **CRIADO** | ✅ OK |
| `spt_bundle_profile_relation_object` | `idx_modified_id_type` | ⚠️ `idx_type_modified` com prefixo 100 | ⚠️ Melhorar |
| `spt_identity` | PRIMARY KEY em `id` | ✅ **EXISTE** | ✅ OK |
| `spt_identity_assigned_roles` | `idx_identity_bundle` | ⚠️ Só tem em `bundle` | ⚠️ Opcional |
| `spt_request` | `idx_modified`, `idx_created` | ❌ **NÃO EXISTE** | ⚠️ Verificar necessidade |

---

## ✅ Conclusão

**Progresso:** ~60% das otimizações críticas já foram implementadas!

**Principais Conquistas:**
- ✅ `idx_bundle_created` criado - Query #1 parcialmente otimizada
- ✅ PRIMARY KEY em `spt_identity.id` - Query #2 tem índice adequado

**Próximos Passos:**
1. ⚠️ Verificar e melhorar índice em `spt_bundle_profile_relation_object`
2. ⚠️ Analisar necessidade de índices em `spt_request`
3. ⚠️ Considerar índice composto em `spt_identity_assigned_roles`

**Impacto Esperado Após Completar:**
- Query #1: Melhoria adicional de 20-30% (já melhorou com `idx_bundle_created`)
- Query #147: Melhoria de 50-70% se criar índice composto
- UPDATEs spt_request: Verificar se índices ajudam (pode ser problema de locks/IO)

---

## 🔍 Queries de Diagnóstico Adicional

```sql
-- 1. Verificar uso dos índices atuais
EXPLAIN
SELECT bundleprof0_.bundle_id, bundleprof0_.hash_code 
FROM spt_bundle_profile_relation bundleprof0_ 
INNER JOIN spt_bundle_profile_relation_object bundleprof1_ 
    ON bundleprof0_.bundle_id = bundleprof1_.modified_id 
WHERE bundleprof1_.type = 'TYPE_INSERT' 
ORDER BY bundleprof0_.created DESC 
LIMIT 1000;

-- 2. Verificar cardinalidade e seletividade
SELECT 
    'spt_bundle_profile_relation_object' AS table_name,
    COUNT(*) AS total_rows,
    COUNT(DISTINCT modified_id) AS distinct_modified_id,
    COUNT(DISTINCT type) AS distinct_type,
    COUNT(DISTINCT LEFT(modified_id, 100)) AS distinct_first_100_chars
FROM spt_bundle_profile_relation_object;
```
