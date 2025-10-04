# Step: Remove Token - Fluxo Leaver

## Visão Geral

Este step foi desenvolvido para remover tokens de autenticação biométrica (Typing DNA e Hypr) durante o processo de desligamento de funcionários e terceiros no fluxo Leaver.

## Funcionalidade

O step verifica se a identidade possui uma conta no Ping Directory com atributo `onboardingStatus` configurado e remove o token correspondente da API externa.

### Tipos de Token Suportados

- **TYPE**: Token Typing DNA (biometria de digitação)
- **HYPER**: Token Hypr (autenticação biométrica)

## Configuração

### Parâmetros de Entrada

- **identityName**: Nome da identidade a ser processada
- **appName**: Nome da aplicação (fixo: "Ping Directory")

### APIs Integradas

#### Typing DNA API
- **URL**: `https://verify.typingdna.com/oidc/webhook/reset-profile`
- **Método**: POST
- **Content-Type**: `application/x-www-form-urlencoded`
- **Autenticação**: Basic Auth
- **Payload**: `user_id={uid}`

#### Hypr API
- **URL**: `https://tim.hypr.com/cc/api/rpUser/delete`
- **Método**: POST
- **Content-Type**: `application/json`
- **Autenticação**: Custom Header
- **Payload**: `{"appId":"pingFederateProd","username":"{uid}"}`

## Fluxo de Execução

1. **Busca da Identidade**: Localiza a identidade pelo nome fornecido
2. **Verificação de Links**: Verifica se a identidade possui conta no Ping Directory
3. **Análise do onboardingStatus**: Identifica o tipo de token configurado
4. **Remoção do Token**: Chama a API correspondente para remover o token
5. **Limpeza do Atributo**: Remove o atributo `onboardingStatus` do Ping Directory
6. **Logging**: Registra todas as operações para auditoria

## Código Principal

```java
// Verificação do onboardingStatus
if ("TYPE".equalsIgnoreCase(onboardingStatus)) {
    result = removeTokenTyping(uid);
    if (result.contains("sucesso")) {
        clearOnboardingStatus(identityName, link.getNativeIdentity(), onboardingStatus);
    }
} else if ("HYPER".equalsIgnoreCase(onboardingStatus)) {
    result = removeTokenHypr(uid);
    if (result.contains("sucesso")) {
        clearOnboardingStatus(identityName, link.getNativeIdentity(), onboardingStatus);
    }
}
```

## Tratamento de Erros

- **Identidade não encontrada**: Retorna mensagem de erro específica
- **Conta Ping Directory não encontrada**: Continua processamento
- **onboardingStatus vazio/nulo**: Retorna mensagem informativa
- **Falha na API**: Registra erro e retorna mensagem de falha

## Logs Gerados

- `Removendo tokenPing`: Início do processamento
- `Verificando links da identidade`: Busca por contas vinculadas
- `Encontrada aplicação Ping Directory`: Conta encontrada
- `onboardingStatus encontrado`: Tipo de token identificado
- `Removendo token [Tipo] para uid`: Início da remoção
- `Resultado da remoção do token`: Status da operação
- `Limpando atributo onboardingStatus`: Limpeza do atributo

## Dependências Técnicas

- **JAX-RS Client**: Para chamadas HTTP às APIs externas
- **SailPoint Provisioning**: Para modificação de atributos no Ping Directory
- **Base64 Encoding**: Para autenticação Basic Auth

## Segurança

- Credenciais das APIs são configuradas como constantes no código
- Autenticação Basic Auth para Typing DNA
- Custom Header Authorization para Hypr
- Logs não expõem informações sensíveis

## Monitoramento

O step gera logs detalhados que permitem:
- Rastreamento de execução
- Identificação de falhas
- Auditoria de operações
- Troubleshooting de problemas

## Retorno

- **Sucesso**: Mensagem confirmando remoção do token e limpeza do atributo
- **Falha**: Mensagem de erro específica com detalhes da falha
- **Não aplicável**: String vazia quando não há token para remover

## Integração com Fluxo Leaver

Este step é executado automaticamente durante o processo de desligamento, garantindo que:
- Tokens biométricos sejam removidos das APIs externas
- Atributos de onboarding sejam limpos
- Auditoria completa seja mantida
- Processo seja executado de forma transparente
