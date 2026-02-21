# Análise de Slow Queries - IdentityIQ MySQL Flexible (Azure)

## Resumo Executivo

**Período analisado:** 2026-02-20T07:37:11 a 2026-02-20T11:49:41 (4h 12min)

**Estatísticas Gerais:**
- **Total de queries:** 48.71k
- **Queries únicas:** 165
- **QPS médio:** 3.22
- **Concorrência média:** 48.73x
- **Tempo total de execução:** 738.314s (205 horas acumuladas)
- **Tempo médio por query:** 15s
- **Rows examinadas:** 2.38 bilhões
- **Rows retornadas:** 1.24 milhões

---

## Top 3 Queries Mais Problemáticas

### 1. Query #1 - CROSS JOIN em spt_bundle_profile_relation (CRÍTICA)

**Query:**
```sql
SELECT bundleprof0_.bundle_id as col_0_0_, 
       bundleprof0_.hash_code as col_1_0_ 
FROM spt_bundle_profile_relation bundleprof0_ 
CROSS JOIN spt_bundle_profile_relation_object bundleprof1_ 
WHERE bundleprof0_.bundle_id = bundleprof1_.modified_id 
  AND bundleprof1_.type = 'TYPE_INSERT' 
ORDER BY bundleprof0_.created DESC 
LIMIT 1000;
```

**Estatísticas:**
- **Execuções:** 16
- **Tempo total:** 52.692s (7.1% do tempo total)
- **Tempo médio:** 3.293s (55 minutos por execução!)
- **Tempo máximo:** 4.415s (73 minutos)
- **Rows examinadas:** 2.26 bilhões (144.65M por execução em média)
- **Rows retornadas:** 15.62k (1.000 por execução)

**Problema Identificado:**
- CROSS JOIN sem índice adequado
- Ordenação por `created` sem índice
- Filtro em `type` sem índice composto
- Exame de bilhões de linhas para retornar apenas 1.000

**Recomendações:**

#### Índices Necessários:

```sql
-- Índice composto na tabela spt_bundle_profile_relation_object
-- Prioridade: ALTA
CREATE INDEX idx_modified_id_type 
ON spt_bundle_profile_relation_object(modified_id, type);

-- Índice composto na tabela spt_bundle_profile_relation
-- Prioridade: ALTA
CREATE INDEX idx_bundle_created 
ON spt_bundle_profile_relation(bundle_id, created DESC);

-- Índice adicional para otimizar o JOIN
CREATE INDEX idx_bundle_id 
ON spt_bundle_profile_relation(bundle_id);
```

#### Otimização da Query:

```sql
-- Versão otimizada usando INNER JOIN explícito
SELECT bundleprof0_.bundle_id as col_0_0_, 
       bundleprof0_.hash_code as col_1_0_ 
FROM spt_bundle_profile_relation bundleprof0_ 
INNER JOIN spt_bundle_profile_relation_object bundleprof1_ 
    ON bundleprof0_.bundle_id = bundleprof1_.modified_id 
WHERE bundleprof1_.type = 'TYPE_INSERT' 
ORDER BY bundleprof0_.created DESC 
LIMIT 1000;
```

**Impacto Esperado:** Redução de 99%+ no tempo de execução (de ~55min para <1s)

---

### 2. Query #2 - SELECT FOR UPDATE em spt_identity (CRÍTICA)

**Query:**
```sql
SELECT identity0_.name as col_0_0_, 
       identity0_.iiqlock as col_1_0_ 
FROM spt_identity identity0_ 
WHERE identity0_.id = '0a9736457f7a1e7b817f7ec4d36c198f' 
FOR UPDATE;
```

**Estatísticas:**
- **Execuções:** 989
- **Tempo total:** 47.434s (6.4% do tempo total)
- **Tempo médio:** 48s por execução
- **Tempo máximo:** 52s
- **Lock time:** 47.434s (90% do tempo total - 100% do lock time global!)
- **Rows examinadas:** 121 (0.12 por execução)
- **Rows retornadas:** 121 (0.12 por execução)

**Problema Identificado:**
- **LOCK TIME EXTREMAMENTE ALTO** - 90% do tempo total de lock do sistema
- Queries bloqueando por muito tempo (10-52 segundos)
- Possível contenção de locks ou deadlocks
- Índice em `id` provavelmente existe, mas locks estão sendo mantidos por muito tempo

**Recomendações:**

#### Verificações Necessárias:

```sql
-- 1. Verificar se existe índice na coluna id (deve ser PRIMARY KEY)
SHOW INDEX FROM spt_identity WHERE Key_name = 'PRIMARY';

-- 2. Verificar transações longas
SELECT * FROM information_schema.innodb_trx 
WHERE trx_started < DATE_SUB(NOW(), INTERVAL 10 SECOND)
ORDER BY trx_started;

-- 3. Verificar locks ativos
SELECT * FROM performance_schema.data_locks 
WHERE object_schema = 'identityiq' 
  AND object_name = 'spt_identity';

-- 4. Verificar deadlocks recentes
SHOW ENGINE INNODB STATUS\G
```

#### Otimizações:

1. **Reduzir tempo de transação:**
   - Mover lógica de processamento para fora da transação
   - Usar transações menores e mais granulares
   - Implementar retry logic com backoff exponencial

2. **Otimizar nível de isolamento:**
   ```sql
   -- Verificar nível atual
   SELECT @@transaction_isolation;
   
   -- Considerar READ COMMITTED se aplicável
   SET SESSION transaction_isolation = 'READ COMMITTED';
   ```

3. **Implementar lock timeout:**
   ```sql
   SET innodb_lock_wait_timeout = 5; -- Reduzir de padrão (50s) para 5s
   ```

4. **Adicionar índice se não existir:**
   ```sql
   -- Verificar estrutura da tabela
   SHOW CREATE TABLE spt_identity;
   
   -- Se id não for PRIMARY KEY, criar índice único
   CREATE UNIQUE INDEX idx_identity_id ON spt_identity(id);
   ```

**Impacto Esperado:** Redução de 80-90% no lock time

---

### 3. Queries #3-100+ - UPDATE em spt_request (ALTA FREQUÊNCIA)

**Query Pattern:**
```sql
UPDATE spt_request 
SET created = ?, 
    modified = ?, 
    significant_modified = ?, 
    owner = NULL, 
    assigned_scope = NULL, 
    assigned_scope_path = NULL, 
    stack = NULL, 
    attributes = '<Attributes>...' -- XML muito grande (12KB)
WHERE id = ?;
```

**Estatísticas:**
- **Total de execuções:** ~20.000+ (maioria das queries do top 100)
- **Tempo médio:** 13-14s por UPDATE
- **Query size:** 12KB (muito grande devido ao XML em attributes)
- **Rows examinadas:** 1 por UPDATE (eficiente)
- **Problema:** Tempo de execução alto mesmo com 1 row examinada

**Problema Identificado:**
- UPDATEs muito lentos (13-14s) mesmo com índice em PRIMARY KEY
- Campo `attributes` é XML muito grande (12KB)
- Possível problema de I/O ou contenção de locks
- Muitas atualizações simultâneas na mesma tabela

**Recomendações:**

#### Verificações:

```sql
-- 1. Verificar estrutura da tabela e índices
SHOW CREATE TABLE spt_request;
SHOW INDEX FROM spt_request;

-- 2. Verificar tamanho da tabela e fragmentação
SELECT 
    table_name,
    ROUND(((data_length + index_length) / 1024 / 1024), 2) AS "Size (MB)",
    ROUND((data_free / 1024 / 1024), 2) AS "Free (MB)"
FROM information_schema.TABLES 
WHERE table_schema = 'identityiq' 
  AND table_name = 'spt_request';

-- 3. Verificar I/O wait
SELECT * FROM sys.schema_table_statistics 
WHERE table_schema = 'identityiq' 
  AND table_name = 'spt_request';
```

#### Otimizações:

1. **Otimizar coluna attributes:**
   ```sql
   -- Considerar compressão da coluna
   ALTER TABLE spt_request 
   MODIFY COLUMN attributes LONGTEXT 
   COMPRESSION='zlib';
   
   -- OU mover para tabela separada (normalização)
   CREATE TABLE spt_request_attributes (
       request_id VARCHAR(255) PRIMARY KEY,
       attributes LONGTEXT,
       FOREIGN KEY (request_id) REFERENCES spt_request(id)
   );
   ```

2. **Adicionar índices para queries de busca:**
   ```sql
   -- Se houver queries filtrando por modified ou created
   CREATE INDEX idx_modified ON spt_request(modified);
   CREATE INDEX idx_created ON spt_request(created);
   CREATE INDEX idx_significant_modified ON spt_request(significant_modified);
   ```

3. **Otimizar configurações do InnoDB:**
   ```sql
   -- Aumentar buffer pool se possível
   SET GLOBAL innodb_buffer_pool_size = <valor_adequado>;
   
   -- Otimizar log files
   SET GLOBAL innodb_log_file_size = 512M;
   SET GLOBAL innodb_flush_log_at_trx_commit = 2; -- Cuidado: reduz durabilidade
   ```

4. **Batch Updates:**
   - Considerar agrupar múltiplos UPDATEs em transações maiores
   - Usar prepared statements para reduzir parsing

**Impacto Esperado:** Redução de 50-70% no tempo de UPDATE

---

## Outras Queries Problemáticas

### Query #147 - COUNT com JOINs

**Query:**
```sql
SELECT COUNT(identity0_.id) as col_0_0_ 
FROM spt_identity identity0_ 
INNER JOIN spt_identity_assigned_roles assignedro1_ 
    ON identity0_.id = assignedro1_.identity_id 
INNER JOIN spt_bundle bundle2_ 
    ON assignedro1_.bundle = bundle2_.id 
WHERE bundle2_.id = '0a9736487f6a1ec0817f6af736260909' 
  AND identity0_.workgroup <> 1;
```

**Recomendações:**
```sql
-- Índices necessários
CREATE INDEX idx_identity_assigned_roles_identity_id 
ON spt_identity_assigned_roles(identity_id);

CREATE INDEX idx_identity_assigned_roles_bundle 
ON spt_identity_assigned_roles(bundle);

CREATE INDEX idx_identity_workgroup 
ON spt_identity(workgroup);

-- Índice composto para otimizar a query completa
CREATE INDEX idx_bundle_workgroup 
ON spt_identity_assigned_roles(bundle, identity_id);
```

---

## Queries SQL para Coleta de Informações Adicionais

Execute os seguintes comandos para obter mais informações sobre o banco:

### 1. Estrutura das Tabelas Problemáticas

```sql
-- Estrutura da tabela spt_bundle_profile_relation
SHOW CREATE TABLE spt_bundle_profile_relation\G

-- Estrutura da tabela spt_bundle_profile_relation_object
SHOW CREATE TABLE spt_bundle_profile_relation_object\G

-- Estrutura da tabela spt_identity
SHOW CREATE TABLE spt_identity\G

-- Estrutura da tabela spt_request
SHOW CREATE TABLE spt_request\G
```

### 2. Índices Existentes

```sql
-- Todos os índices das tabelas problemáticas
SHOW INDEX FROM spt_bundle_profile_relation;
SHOW INDEX FROM spt_bundle_profile_relation_object;
SHOW INDEX FROM spt_identity;
SHOW INDEX FROM spt_request;
SHOW INDEX FROM spt_identity_assigned_roles;
SHOW INDEX FROM spt_bundle;
```

### 3. Estatísticas das Tabelas

```sql
-- Tamanho e estatísticas das tabelas
SELECT 
    table_name,
    table_rows,
    ROUND(((data_length + index_length) / 1024 / 1024 / 1024), 2) AS "Size (GB)",
    ROUND((data_length / 1024 / 1024 / 1024), 2) AS "Data (GB)",
    ROUND((index_length / 1024 / 1024 / 1024), 2) AS "Index (GB)",
    ROUND((data_free / 1024 / 1024 / 1024), 2) AS "Free (GB)"
FROM information_schema.TABLES 
WHERE table_schema = 'identityiq' 
  AND table_name IN (
      'spt_bundle_profile_relation',
      'spt_bundle_profile_relation_object',
      'spt_identity',
      'spt_request',
      'spt_identity_assigned_roles',
      'spt_bundle'
  )
ORDER BY (data_length + index_length) DESC;
```

### 4. Análise de Fragmentação

```sql
-- Verificar fragmentação
SELECT 
    table_name,
    ROUND((data_free / (data_length + index_length)) * 100, 2) AS "Fragmentation %"
FROM information_schema.TABLES 
WHERE table_schema = 'identityiq' 
  AND table_name IN (
      'spt_bundle_profile_relation',
      'spt_bundle_profile_relation_object',
      'spt_identity',
      'spt_request'
  )
HAVING "Fragmentation %" > 10;
```

### 5. Transações e Locks Ativos

```sql
-- Transações ativas
SELECT 
    trx_id,
    trx_state,
    trx_started,
    TIMESTAMPDIFF(SECOND, trx_started, NOW()) AS duration_seconds,
    trx_requested_lock_id,
    trx_wait_started,
    trx_mysql_thread_id,
    trx_query
FROM information_schema.innodb_trx
ORDER BY trx_started;

-- Locks esperando
SELECT 
    r.trx_id AS waiting_trx_id,
    r.trx_mysql_thread_id AS waiting_thread,
    r.trx_query AS waiting_query,
    b.trx_id AS blocking_trx_id,
    b.trx_mysql_thread_id AS blocking_thread,
    b.trx_query AS blocking_query
FROM information_schema.innodb_lock_waits w
INNER JOIN information_schema.innodb_trx b ON b.trx_id = w.blocking_trx_id
INNER JOIN information_schema.innodb_trx r ON r.trx_id = w.requesting_trx_id;
```

### 6. Configurações do InnoDB

```sql
-- Configurações importantes do InnoDB
SHOW VARIABLES LIKE 'innodb%';
SHOW VARIABLES LIKE 'max_connections';
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';
SHOW VARIABLES LIKE 'innodb_log_file_size';
SHOW VARIABLES LIKE 'innodb_flush_log_at_trx_commit';
SHOW VARIABLES LIKE 'innodb_lock_wait_timeout';
```

### 7. Estatísticas de Performance Schema

```sql
-- Queries mais lentas (últimas 24h)
SELECT 
    sql_text,
    COUNT(*) AS exec_count,
    AVG(timer_wait) / 1000000000000 AS avg_time_sec,
    SUM(timer_wait) / 1000000000000 AS total_time_sec
FROM performance_schema.events_statements_history_long
WHERE sql_text LIKE '%spt_%'
GROUP BY sql_text
ORDER BY total_time_sec DESC
LIMIT 20;
```

---

## Plano de Ação Recomendado

### Fase 1: Crítico (Implementar Imediatamente)

1. **Criar índices para Query #1:**
   ```sql
   CREATE INDEX idx_modified_id_type 
   ON spt_bundle_profile_relation_object(modified_id, type);
   
   CREATE INDEX idx_bundle_created 
   ON spt_bundle_profile_relation(bundle_id, created DESC);
   ```

2. **Investigar e resolver locks na Query #2:**
   - Executar queries de diagnóstico de locks
   - Revisar código da aplicação para reduzir tempo de transação
   - Implementar lock timeout

### Fase 2: Alta Prioridade (Esta Semana)

3. **Otimizar UPDATEs em spt_request:**
   - Analisar possibilidade de compressão ou normalização do campo attributes
   - Verificar fragmentação e executar OPTIMIZE TABLE se necessário

4. **Criar índices para outras queries problemáticas:**
   - Índices para spt_identity_assigned_roles
   - Índices para spt_bundle

### Fase 3: Melhorias Contínuas (Próximas 2 Semanas)

5. **Monitoramento:**
   - Configurar alertas para queries lentas
   - Revisar slow query log semanalmente
   - Implementar dashboard de performance

6. **Otimizações de Configuração:**
   - Ajustar innodb_buffer_pool_size
   - Revisar configurações de log do InnoDB
   - Considerar particionamento de tabelas grandes

---

## Observações Importantes

1. **Teste em Ambiente de Desenvolvimento/Staging primeiro**
2. **Faça backup antes de criar índices em produção** (pode demorar em tabelas grandes)
3. **Monitore o impacto após cada mudança**
4. **Crie índices durante horário de menor uso** (pode bloquear a tabela)
5. **Considere usar `ALGORITHM=INPLACE, LOCK=NONE` quando possível** (MySQL 5.6+)

---

## Métricas de Sucesso Esperadas

Após implementar as otimizações:

- **Query #1:** Redução de 99%+ (de 55min para <1s)
- **Query #2:** Redução de 80-90% no lock time
- **UPDATEs spt_request:** Redução de 50-70% (de 13s para 4-6s)
- **Tempo total de execução:** Redução de 60-70% no geral
- **Throughput:** Aumento de 2-3x em QPS

---

## Contato para Dúvidas

Execute as queries de diagnóstico acima e compartilhe os resultados para análise mais detalhada.
