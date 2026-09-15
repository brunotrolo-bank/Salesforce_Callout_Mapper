# ✅ Protocolo de Validação & Testes End-to-End

## Visão Geral

Este documento define o protocolo completo de validação da skill `sf-autonomous-mapper`, incluindo testes unitários, de integração e end-to-end com um caso real de ORG Salesforce.

---

## Fase 1: Validação de Schema

### 1.1 Validação de JSON Schema

**Objetivo**: Garantir que `database.json` obedece ao schema definido em `SCHEMA.md`

**Ferramenta**: AJV (JSON Schema Validator)

**Validações Obrigatórias**:
- [ ] Todos os campos obrigatórios presentes
- [ ] Tipos de dados corretos (string, number, object, array)
- [ ] Ranges de valores válidos (confiança 0-1, criticidade alta/média/baixa)
- [ ] Sem campos extras inesperados
- [ ] Todas as referências (IDs de callouts, classes) existem

**Comando**:
```bash
ajv validate -s SCHEMA.json -d .callout-kb/database.json
```

**Critério de Aceitação**:
- ✅ 100% de valididade
- ✅ 0 erros schema
- ✅ 0 warnings

---

### 1.2 Validação de Integridade Referencial

**Objetivo**: Verificar links entre callouts, classes, LWC

**Checklist**:
- [ ] Cada `apexClass` em callout existe no projeto
- [ ] Cada `namedCredential` referenciado existe em metadados
- [ ] Cada `lwcComponent` existe em `lwc/` folder
- [ ] Não há IDs de callout duplicados
- [ ] Todas as dependências em `relationships.json` referem callouts existentes

**Implementação**:

```python
def validate_referential_integrity(database, project_path):
    errors = []
    
    for callout in database['callouts']:
        # Validar Apex Classes
        for apex_class in callout['salesforceArtefatos']['apexClasses']:
            class_file = find_file(f"{apex_class['nome']}", project_path)
            if not class_file:
                errors.append(f"Classe não encontrada: {apex_class['nome']}")
        
        # Validar Named Credentials
        named_cred = callout['salesforceArtefatos']['namedCredential']
        if not file_exists(f"force-app/main/default/namedCredentials/{named_cred}.namedCredential-meta.xml"):
            errors.append(f"Named Credential não encontrada: {named_cred}")
        
        # Validar LWC Components
        for lwc in callout['salesforceArtefatos']['lwcComponentes']:
            lwc_path = find_file(f"{lwc['nome']}.js", project_path)
            if not lwc_path:
                errors.append(f"LWC não encontrada: {lwc['nome']}")
    
    return errors
```

**Critério de Aceitação**:
- ✅ 100% das referências válidas
- ✅ 0 orfãos (itens sem referência)

---

## Fase 2: Validação de Detecção

### 2.1 Precisão de Descoberta

**Objetivo**: Comparar callouts encontrados vs. esperados

**Método**:

```
1. Listar todos os HttpRequest/Http.send() no código manualmente (baseline)
2. Executar skill
3. Comparar descoberta com baseline
4. Calcular:
   - Recall = encontrados / total esperado
   - Precision = encontrados corretos / total encontrado
```

**Exemplo**:

```
Baseline manual: 45 callouts
Skill encontrou: 43 callouts
Falsos positivos: 0
Falsos negativos: 2

Recall = 43/45 = 95.5% ✅
Precision = 43/43 = 100% ✅
```

**Critério de Aceitação**:
- ✅ Recall > 90% (detecta 90%+ dos callouts)
- ✅ Precision > 95% (>95% sem falsos positivos)

---

### 2.2 Validação de Payload Detection

**Objetivo**: Verificar qualidade de detecção de payloads

**Método**:

```
Para cada callout encontrado:
1. Executar skill (obtém payload detectado)
2. Revisar manualmente (payload real)
3. Comparar (estrutura + confiança)
4. Calcular confiança média
```

**Métricas**:

```json
{
  "payloadValidation": {
    "estrategia_1_dto": {
      "testados": 10,
      "corretos": 10,
      "precisao": 1.0,
      "confiancaMedia": 1.0
    },
    "estrategia_2_map": {
      "testados": 15,
      "corretos": 12,
      "precisao": 0.8,
      "confiancaMedia": 0.76
    },
    "estrategia_3_string": {
      "testados": 2,
      "corretos": 1,
      "precisao": 0.5,
      "confiancaMedia": 0.65
    },
    "confiancaMediaGeral": 0.84
  }
}
```

**Critério de Aceitação**:
- ✅ Confiança média geral > 75%
- ✅ Estratégia 1: precisão = 100%
- ✅ Estratégia 2: precisão > 75%
- ✅ Estratégia 3: precisão > 50%

---

### 2.3 Validação de Linhagem

**Objetivo**: Garantir rastreamento preciso LWC → Apex → Endpoint

**Método**:

```
Para cada callout com LWC:
1. Verificar que LWC → Apex está correto
2. Verificar que Apex → Endpoint está correto
3. Validar sem gaps na cadeia
4. Contar quantidade de "saltos" na linhagem
```

**Exemplo de Validação**:

```
Callout: CALLOUT-SF-001
LWC esperada: checkoutForm.js ✅ encontrada
Apex esperada: CheckoutController.cls ✅ encontrada
Método Apex: processCheckout ✅ encontrado
Http.send() esperado: ✅ encontrado
Endpoint esperado: /v1/payments/charge ✅ encontrado

Linhagem completa: ✅ SIM (sem gaps)
Confiança: 100%
```

**Critério de Aceitação**:
- ✅ 100% dos callouts com linhagem completa
- ✅ 0 gaps não resolvidos
- ✅ 0 referências circulares

---

## Fase 3: Validação de Dados

### 3.1 Validação de Criticidade

**Objetivo**: Verificar se cálculo de criticidade é sensato

**Método**:

```python
def validate_criticality(database):
    """
    Heurísticas de validação:
    - Se 10+ componentes usam → deve ser "Alta"
    - Se DELETE/PUT em recurso crítico → deve ser "Alta"
    - Se timeout > 30s → deve ser "Média+" (gargalo)
    - Se tem retry automático → deve ser "Média" no máximo
    """
    
    issues = []
    
    for callout in database['callouts']:
        refs = count_references(callout['id'])
        method = callout['metadadosRede']['metodo']
        timeout = callout['performance'].get('timeoutMs', 30000)
        has_retry = callout['resilience']['temRetry']
        
        # Heurística 1: Alta referência = Alta criticidade
        if refs > 10 and callout['criticidade'] != 'Alta':
            issues.append(
                f"{callout['id']}: Usado por {refs} componentes mas marcado como "
                f"{callout['criticidade']} (deveria ser Alta)"
            )
        
        # Heurística 2: Operação destrutiva = Alta criticidade
        if method in ['DELETE', 'PUT'] and 'customer' in callout['jornada'].lower():
            if callout['criticidade'] not in ['Alta', 'Crítica']:
                issues.append(
                    f"{callout['id']}: {method} em recurso crítico mas marcado "
                    f"como {callout['criticidade']}"
                )
        
        # Heurística 3: Sem retry = risco
        if not has_retry and callout['criticidade'] == 'Baixa':
            issues.append(
                f"{callout['id']}: Sem retry automático. Deveria ser Média?"
            )
    
    return issues
```

**Critério de Aceitação**:
- ✅ Criticidade heuristically consistent (max 5% anomalias aceitáveis)
- ✅ Sem contradições óbvias

---

### 3.2 Validação de Jornada

**Objetivo**: Verificar se categorização de jornada é sensata

**Método**:

```
Para cada callout, verificar:
1. Nome da classe contém palavra-chave de jornada?
2. Método contém palavra-chave?
3. Pasta contém palavra-chave?
4. Custom Metadata business_domain está preenchido?

Se múltiplas heurísticas concordam → jornada está correta ✅
Se contraditórias → manualmente revisar
```

**Exemplo de Validação**:

```
Callout: CALLOUT-SF-042
Classe: PaymentProcessingService
├─ Contém "Payment" ✅
├─ Método: processPayment
   ├─ Contém "Payment" ✅
├─ Pasta: modules/payments/
   ├─ Contém "payments" ✅
└─ Custom Metadata: business_domain = "Payments"
   ├─ Matches ✅

Consenso: Jornada = "Pagamentos" ✅ CORRETO
Confiança: 100%
```

**Critério de Aceitação**:
- ✅ Jornadas coincidem entre múltiplas fontes > 85% do tempo
- ✅ Sem "UNMAPPED" exceto em casos extremos

---

## Fase 4: Testes Unitários (Por Estratégia)

### 4.1 Testes de Detecção de Payload (50 casos)

```python
@pytest.mark.parametrize("apex_code,expected_method,min_confidence", [
    # DTO Tipado (confiança 100%)
    ("""
    public class PaymentRequest { Decimal amount; String currency; }
    String body = JSON.serialize(new PaymentRequest());
    req.setBody(body);
    """, "DTO_TIPADO", 0.95),
    
    # Map Dinâmico (confiança 70-85%)
    ("""
    Map<String, Object> payload = new Map<String, Object>();
    payload.put('amount', 150.00);
    payload.put('currency', 'BRL');
    req.setBody(JSON.serialize(payload));
    """, "MAP_DINAMICO", 0.7),
    
    # String Literal (confiança 60-75%)
    ("""
    String body = '{"status": "pending", "amount": 100}';
    req.setBody(body);
    """, "STRING_LITERAL", 0.6),
    
    # Fallback (confiança < 40%)
    ("""
    String body = buildPayload(data);
    req.setBody(body);
    """, "UNKNOWN", 0.2),
])
def test_payload_detection(apex_code, expected_method, min_confidence):
    result = detect_payload(apex_code)
    assert result['metodo'] == expected_method
    assert result['confianca'] >= min_confidence
```

**Critério de Aceitação**:
- ✅ 50/50 testes passam
- ✅ Confiança média > 75%

---

### 4.2 Testes de Rastreamento LWC → Apex

```python
@pytest.mark.parametrize("lwc_code,expected_apex_method", [
    ("import sync from '@salesforce/apex/SyncController.sync'", "SyncController.sync"),
    ("import { doCallout } from 'c/baseComponent'", None),  # Event-based
])
def test_lwc_to_apex_tracing(lwc_code, expected_apex_method):
    result = trace_lwc_imports(lwc_code)
    if expected_apex_method:
        assert expected_apex_method in result['apexMethods']
```

**Critério de Aceitação**:
- ✅ 100% dos imports detectados
- ✅ 100% das invocações rastreadas

---

## Fase 5: Teste de Integração

### 5.1 Teste com Projeto Pequeno (< 100 callouts)

**Objetivo**: Validar pipeline completo em ambiente controlado

**Projeto de Teste**: Criar um projeto Salesforce DX mínimo com:
- 5 Apex classes (com 8 callouts)
- 3 LWC components
- 3 Named Credentials
- 2 External Credentials

**Execução**:
```bash
/sf-autonomous-mapper --path test-project --ci-mode
```

**Validações**:
- [ ] Descoberta encontra 8/8 callouts (100%)
- [ ] Análise extrai payload schema (confiança > 70%)
- [ ] Linhagem LWC → Endpoint completa
- [ ] database.json gerado válido (schema OK)
- [ ] dashboard.html carrega sem erros
- [ ] Busca no dashboard funciona
- [ ] Tempo total execução < 30s

---

### 5.2 Teste com Projeto Médio (100-500 callouts)

**Objetivo**: Validar escalabilidade e performance

**Projeto de Teste**: Projeto Salesforce real (produção simulada)

**Validações**:
- [ ] Descoberta encontra > 90% dos callouts
- [ ] Tempo total execução < 5min
- [ ] database.json < 50MB
- [ ] Dashboard carrega em < 2s
- [ ] Busca performática (< 100ms para resultado)

---

## Fase 6: Teste End-to-End (Caso Real)

### 6.1 Setup do Caso Real

Ver `TEST_CASE.md` para detalhes completos.

**Projeto**: Plataforma Salesforce de Pagamentos (fictícia mas realista)

**Componentes**:
- 12 Apex classes com callouts
- 8 LWC components
- 6 Named Credentials
- 4 Custom Metadata integrations
- 3 padrões avançados (Continuation, @future, Queueable)

---

### 6.2 Execução do Teste E2E

```bash
# Diretório de teste
cd ~/test-projects/salesforce-payment-platform

# Executar skill
/sf-autonomous-mapper

# Aguardar conclusão
# (deve levar 2-5 minutos)

# Validar outputs
ls -la .callout-kb/
ls -la docs/

# Abrir dashboard no navegador
open docs/index.html
# ou
firefox docs/index.html
```

---

### 6.3 Checklist de Validação Manual

Ao abrir `docs/index.html`, validar:

#### Interface & UX
- [ ] Dashboard carrega corretamente
- [ ] Layout é responsivo (testa em mobile)
- [ ] Sem erros no console do navegador
- [ ] Dark/light mode funciona
- [ ] Nenhuma piscagem ou delay visual

#### Funcionalidade de Busca
- [ ] Busca por "payment" encontra callouts de pagamento
- [ ] Busca por endpoint "/v1/charge" funciona
- [ ] Busca por classe "PaymentController" funciona
- [ ] Busca fuzzy "paymnt" encontra "payment"
- [ ] Busca case-insensitive funciona

#### Filtros
- [ ] Filtro por Jornada funciona
- [ ] Filtro por Gateway funciona
- [ ] Filtro por Método HTTP (GET/POST) funciona
- [ ] Múltiplos filtros combinados funcionam
- [ ] Reset de filtros funciona

#### Detalhes do Callout
- [ ] Click em callout abre painel de detalhes
- [ ] Explicação detalhada está presente
- [ ] Payload JSON tem syntax highlighting
- [ ] Botão "Copiar" funciona
- [ ] Links de linhagem são navegáveis
- [ ] Endpoints e métodos corretos

#### Visualização de Linhagem
- [ ] Diagrama visual LWC → Apex → Endpoint
- [ ] Todos os nós estão conectados
- [ ] Sem setas "quebradas" ou incompletas
- [ ] Hover mostra informações adicionais

#### Performance
- [ ] Primeira busca < 100ms
- [ ] Filtros aplicam < 50ms
- [ ] Scroll smooth (60 fps)
- [ ] Nenhuma lentidão perceptível

---

### 6.4 Validação de Dados Específicos

**Callout Esperado 1: Payment Charge**

```
✅ Encontrado com ID: CALLOUT-PAY-001
✅ Jornada: Checkout & Pagamentos
✅ Gateway: Apigee
✅ Método: POST
✅ Endpoint: /v1/payments/charge
✅ LWC: checkoutPaymentForm
✅ Apex: PaymentController.processCheckout + PaymentCalloutService.charge
✅ Payload: 100% confiança (DTO)
✅ Método HTTP: POST
✅ Timeout: 30000ms
```

**Callout Esperado 2: Auth Token**

```
✅ Encontrado com ID: CALLOUT-AUTH-001
✅ Jornada: Autenticação
✅ Gateway: Auth0
✅ Método: POST
✅ Endpoint: /oauth/v2/token
✅ Classe Apex: AuthService.getAccessToken
✅ Padrão: @future(callout=true)
✅ Retry: Exponential backoff (3 tentativas)
✅ Criticidade: Alta
```

---

## Fase 7: Métricas Finais

### 7.1 Relatório de Validação

Após todos os testes, gerar relatório:

```json
{
  "validationReport": {
    "timestamp": "2025-09-20T15:30:00Z",
    "skillVersion": "1.0.0-beta",
    "projectTested": "salesforce-payment-platform",
    "resultados": {
      "schemaValidation": {
        "status": "PASSED ✅",
        "errorsFound": 0,
        "warningsFound": 0
      },
      "discoveryAccuracy": {
        "status": "PASSED ✅",
        "calloutsExpected": 45,
        "calloutsFound": 43,
        "recall": 0.956,
        "precision": 1.0
      },
      "payloadDetection": {
        "status": "PASSED ✅",
        "confiancaMedia": 0.84,
        "estrategia1Precisao": 1.0,
        "estrategia2Precisao": 0.8,
        "estrategia3Precisao": 0.5
      },
      "lineageTracing": {
        "status": "PASSED ✅",
        "calloutsCom100PercentLinhagem": 43,
        "percentual": 100
      },
      "dashboardTesting": {
        "status": "PASSED ✅",
        "loadTimeMs": 1850,
        "searchPerformanceMs": 85,
        "consoleErrors": 0
      },
      "e2eTesting": {
        "status": "PASSED ✅",
        "manualValidationChecks": 28,
        "checksPassed": 28,
        "issuesFound": 0
      }
    },
    "recommendation": "READY FOR PRODUCTION ✅",
    "nextSteps": [
      "Deploy skill para OpenCode",
      "Documentar lessons learned",
      "Planejar Fase 2 (multi-ambiente)"
    ]
  }
}
```

---

## Fase 8: Critérios de Aceitação Final

### ✅ MVP Pronto se:

```
✅ Schema Validation: 100% válido
✅ Discovery Accuracy: Recall > 90%, Precision > 95%
✅ Payload Detection: Confiança média > 75%
✅ Lineage Tracing: 100% dos callouts com linhagem completa
✅ Dashboard: Sem erros, < 2s carregamento, design profissional
✅ E2E Tests: Todos os casos passam
✅ Performance: Execução < 5min para 500 callouts
✅ Documentação: Completa e clara
```

### ❌ MVP Bloqueado se:

```
❌ Discovery < 85% (muitos callouts perdidos)
❌ Schema invalidity > 5% (muitos dados ruins)
❌ Dashboard com erros JS em console
❌ Performance > 15min para 500 callouts
❌ Qualquer teste E2E falha
```

---

## Próximos Passos

1. [ ] Preparar projeto de teste pequeno
2. [ ] Implementar suite de testes unitários
3. [ ] Executar Fase 1 (Schema Validation)
4. [ ] Executar Fase 2 (Discovery)
5. [ ] Executar Fase 3 (Payload Detection)
6. [ ] Executar Fase 4-5 (Integration tests)
7. [ ] Executar Fase 6 (E2E com caso real)
8. [ ] Gerar relatório final
9. [ ] Deploy para produção se tudo passa
