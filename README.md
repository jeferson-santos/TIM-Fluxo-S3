# Análise e Otimização de Alocação de Recursos - IGA Gamma

## Resumo Executivo

Este documento apresenta a análise da alocação atual de recursos e uma proposta de otimização baseada nas necessidades dos projetos.

## Situação Atual

### Time Disponível
- **5 pessoas** totalizando **20 FTE disponíveis** (5 pessoas × 4 FTE cada)
- Todas as pessoas possuem skills em **IIQ**, com variações em **ISC** e **DEV**

### Necessidades dos Projetos
- **Total necessário (ideal): 6.6 FTE**
- **Saldo disponível: 13.4 FTE** (capacidade ociosa)

### Problemas Identificados na Alocação Atual

1. **TIM - Excesso de alocação**
   - Contratado: 3.0 FTE
   - Atual: 11.0 FTE ⚠️
   - Ideal: 4.0 FTE
   - **Problema**: Excesso de 7.0 FTE

2. **Riachuelo - Déficit**
   - Contratado: 1.0 FTE
   - Atual: 0.0 FTE ⚠️
   - Ideal: 1.0 FTE
   - **Problema**: Projeto sem alocação

3. **Itau CL - Alocação acima do ideal**
   - Atual: 5.5 FTE (não consta na lista com requisitos específicos)

4. **Projetos T&M - Alocação acima do necessário**
   - XP: 1.0 FTE atual vs 0.3 FTE ideal
   - B3: 1.5 FTE atual vs 0.3 FTE ideal
   - C6: 1.0 FTE atual (não consta na lista)

## Proposta de Alocação Otimizada

### Estratégia de Alocação

A nova alocação foi otimizada considerando:

1. **Priorização por necessidade**
   - TIM: 4.0 FTE (maior demanda, projeto 100%)
   - Itau CL: 1.0 FTE (projeto 100%)
   - Riachuelo: 1.0 FTE (T&M)
   - XP: 0.3 FTE (T&M)
   - B3: 0.3 FTE (T&M)

2. **Distribuição balanceada**
   - Períodos consecutivos (manhã/tarde do mesmo dia) para projetos dedicados
   - Distribuição de T&M entre várias pessoas para flexibilidade

3. **Consideração de skills e experiência**
   - Priorização de pessoas com mais skills para projetos complexos
   - Squad Lead e Senior para projetos maiores

### Alocação Proposta por Pessoa

#### Fabiel Rodrigues (Senior) - IIQ, ISC, DEV
- **Total: 4.0 FTE**
- TIM: 1.0 FTE (SM, ST)
- Itau CL: 1.0 FTE (TM, TT) - período consecutivo
- Riachuelo (T&M): 1.0 FTE (QM, QT) - período consecutivo
- XP (T&M): 0.5 FTE (QIM)
- B3 (T&M): 0.5 FTE (QIT)

**Justificativa**: Pessoal com mais skills (IIQ, ISC, DEV) e experiência (Senior), ideal para múltiplos projetos e suporte T&M.

#### Jeferson Santos (Squad Lead) - IIQ, ISC, DEV
- **Total: 1.0 FTE**
- TIM: 1.0 FTE (SM, ST)

**Justificativa**: Squad Lead focado em TIM, projeto de maior prioridade e complexidade.

#### Lucas Bueno (Senior) - IIQ, ISC
- **Total: 1.0 FTE**
- TIM: 1.0 FTE (SM, ST)

**Justificativa**: Senior com skills IIQ/ISC, suporte ao projeto TIM.

#### Guilherme Lima (Pleno) - IIQ
- **Total: 0.5 FTE**
- TIM: 0.5 FTE (SM)

**Justificativa**: Nível Pleno, alocação parcial para TIM.

#### Romeu Oggiam (Ongoing Pl) - IIQ, ISC
- **Total: 0.5 FTE**
- TIM: 0.5 FTE (SM)

**Justificativa**: Alocação parcial para TIM.

**Nota**: Na proposta otimizada, várias pessoas têm capacidade ociosa, permitindo flexibilidade para:
- Atividades emergentes
- Suporte adicional quando necessário
- Melhoria contínua e desenvolvimento
- Férias e ausências

## Comparativo: Atual vs Proposta

| Projeto | Contratado | Atual | Proposta | Status |
|---------|------------|-------|----------|--------|
| TIM | 3.0 | 11.0 | 4.0 | ✅ Otimizado |
| Itau CL | 1.0 | 5.5 | 1.0 | ✅ Otimizado |
| Riachuelo | 1.0 | 0.0 | 1.0 | ✅ Corrigido |
| XP (T&M) | 0.3 | 1.0 | 0.5 | ✅ Otimizado |
| B3 (T&M) | 0.3 | 1.5 | 0.5 | ✅ Otimizado |

## Recomendações

### Curto Prazo
1. **Implementar a nova alocação proposta** para corrigir os desequilíbrios
2. **Monitorar TIM** - garantir que 4 FTE sejam suficientes (há comentário sobre muitas atividades)
3. **Distribuir responsabilidade de Riachuelo** - atualmente sem alocação

### Médio Prazo
1. **Avaliar necessidade de contratação**
   - O documento menciona necessidade de 1 GP/Engenheiro (Ongoing/Suporte)
   - Com 13.4 FTE ociosos, pode ser que a contratação não seja urgente do ponto de vista de capacidade

2. **Revisar projetos T&M**
   - Definir se C6 e Embraer devem ser mantidos
   - Revisar alocação de T&M (atualmente 3.0 FTE vs ideal de 0.9 FTE)

3. **Otimização de processos**
   - Com 67% da capacidade ociosa (13.4/20 FTE), há espaço para:
     - Projetos adicionais
     - Melhoria de processos
     - Desenvolvimento de skills

### Observações Importantes

1. **TIM - Alta demanda**
   - O comentário indica: "Atividades de Manday, existe uma lista grande e precisa de uma pessoa dedicada"
   - A proposta aloca 4.0 FTE distribuídos entre várias pessoas
   - **Recomendação**: Considerar alocar 1 pessoa 100% dedicada para TIM se as atividades de Manday exigirem isso

2. **Projetos T&M**
   - São atividades que "surgem às vezes"
   - A proposta mantém alocação mínima (0.3-0.5 FTE) para cada
   - Permite flexibilidade para aumentar quando necessário

## Arquivos Gerados

1. `analise_alocacao_recursos.py` - Script de análise
2. `IGA - Nova Estrutura - GammaFase 1 - OTIMIZADO.csv` - CSV com alocação proposta

## Próximos Passos

1. Revisar a proposta com o time
2. Ajustar conforme feedback e conhecimento específico dos projetos
3. Implementar a nova alocação
4. Monitorar e ajustar conforme necessário

