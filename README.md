# Melhoria: Exceção de Aplicações no Fluxo de Férias/Afastamento

## 1. Contexto e Problema Identificado

No IdentityIQ, algumas aplicações apresentavam problemas durante o processo de inativação/ativação no fluxo de afastamento e férias. Esses problemas ocorriam quando o sistema tentava desabilitar ou habilitar aplicações que não deveriam ser processadas nesses cenários específicos, resultando em:

- Falhas no processamento do fluxo de afastamento/férias
- Erros durante operações de disable/enable em aplicações específicas
- Impacto na experiência do usuário e no processo de gestão de identidades

## 2. Solução Implementada

Foi implementada uma solução que permite **excluir aplicações específicas** do processamento durante os fluxos de afastamento e férias através de um atributo configurável na aplicação.

### 2.1. Componentes da Solução

1. **Atributo de Aplicação**: "Exceção em Férias/Afastamento"
   - Tipo: Boolean
   - Valor padrão: `false`
   - Quando configurado como `true`, a aplicação é ignorada durante o fluxo

2. **Modificação na Rule `[sec]common_lib`**
   - Método ajustado: `disableApps`
   - Lógica implementada: verificação do atributo antes de processar a aplicação

## 3. Detalhes Técnicos da Implementação

### 3.1. Atributo Criado

**Nome do Atributo**: `Exceção em Férias/Afastamento`  
**Tipo**: Boolean  
**Localização**: Objeto Application (Aplicação)  
**Propósito**: Indicar se a aplicação deve ser ignorada durante os fluxos de afastamento e férias

### 3.2. Modificação na Rule `[sec]common_lib`

A rule `[sec]common_lib` foi ajustada no método `disableApps` para incluir a verificação do atributo de exceção. A lógica implementada segue o seguinte fluxo:

1. **Iteração sobre as aplicações** a serem processadas
2. **Verificação do atributo** "Exceção em Férias/Afastamento" para cada aplicação
3. **Condição de exclusão**: Se o atributo estiver configurado como `true`, a aplicação é **ignorada** (não processada)
4. **Processamento normal**: Se o atributo estiver como `false` ou não configurado, a aplicação é processada normalmente

#### Pseudocódigo da Implementação

```
Para cada aplicação no fluxo:
    Se aplicação.Exceção_em_Férias_Afastamento == TRUE:
        Ignorar aplicação (não processar)
        Log: "Aplicação [nome] ignorada devido à exceção em Férias/Afastamento"
    Caso contrário:
        Processar aplicação normalmente (disable/enable)
```

### 3.3. Comportamento

- **Atributo = `true`**: Aplicação **NÃO** será processada durante o fluxo de afastamento/férias
- **Atributo = `false` ou não configurado**: Aplicação será processada **NORMALMENTE**

## 4. Configuração do Atributo

### 4.1. Como Configurar

Para configurar uma aplicação como exceção no fluxo de férias/afastamento:

1. Acesse o **IdentityIQ**
2. Navegue até **Admin > Applications**
3. Selecione a aplicação desejada
4. Localize o atributo **"Exceção em Férias/Afastamento"**
5. Configure o valor como **`true`** para ignorar a aplicação, ou **`false`** para processá-la normalmente
6. Salve as alterações

### 4.2. Quando Usar

Configure o atributo como `true` nas seguintes situações:

- Aplicações que apresentam problemas técnicos durante disable/enable
- Aplicações que não devem ser inativadas/ativadas em cenários de férias/afastamento
- Aplicações críticas que precisam manter acesso durante períodos de afastamento
- Sistemas legados ou integrações com limitações técnicas específicas

## 5. Impactos e Considerações

### 5.1. Impactos Positivos

- ✅ **Redução de falhas** nos fluxos de afastamento e férias
- ✅ **Flexibilidade** para excluir aplicações problemáticas do processamento
- ✅ **Continuidade** do fluxo mesmo com aplicações que apresentam problemas
- ✅ **Configuração simples** através de atributo boolean

### 5.2. Considerações Importantes

⚠️ **Atenção**: Aplicações configuradas como exceção (`true`) **NÃO serão inativadas/ativadas** durante o fluxo de afastamento/férias. Isso significa que:

- O usuário pode manter acesso às aplicações mesmo estando em férias/afastamento
- É necessário revisar periodicamente as aplicações configuradas como exceção
- Documentar o motivo de cada aplicação configurada como exceção

### 5.3. Recomendações

1. **Documentação**: Mantenha um registro das aplicações configuradas como exceção e os motivos
2. **Revisão Periódica**: Reavalie periodicamente se as exceções ainda são necessárias
3. **Testes**: Realize testes do fluxo após configurar novas exceções
4. **Comunicação**: Informe aos stakeholders sobre aplicações que não serão processadas

## 6. Logs e Troubleshooting

### 6.1. Logs Gerados

Durante a execução do método `disableApps`, a rule gera logs quando uma aplicação é ignorada:

```
[INFO] Aplicação [Nome da Aplicação] ignorada devido à exceção em Férias/Afastamento
```

### 6.2. Verificação de Configuração

Para verificar se uma aplicação está configurada como exceção:

1. Acesse a aplicação no IdentityIQ
2. Verifique o valor do atributo "Exceção em Férias/Afastamento"
3. Consulte os logs durante a execução do fluxo

## 7. Compatibilidade

- **Versão do IdentityIQ**: Compatível com versões que suportam atributos customizados em Application
- **Backward Compatibility**: Aplicações sem o atributo configurado continuam funcionando normalmente (comportamento padrão: `false`)

## 8. Manutenção Futura

### 8.1. Monitoramento

- Acompanhe os logs do método `disableApps` para identificar aplicações ignoradas
- Monitore falhas nos fluxos de afastamento/férias para identificar necessidade de novas exceções

### 8.2. Melhorias Futuras

Possíveis melhorias que podem ser implementadas:

- Interface administrativa para visualizar todas as aplicações com exceção
- Relatório de aplicações ignoradas durante cada execução do fluxo
- Validação automática de aplicações configuradas como exceção

---

## 9. Contato e Suporte

Para dúvidas ou problemas relacionados a esta funcionalidade, entre em contato com a equipe de IdentityIQ.

**Data de Implementação**: [Data a ser preenchida]  
**Versão do Documento**: 1.0  
**Autor**: Equipe de Desenvolvimento IdentityIQ

