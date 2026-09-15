# 📐 Arquitetura - Agente Autônomo SF Callout Mapper

## Visão Geral

O agente `sf-autonomous-mapper` é uma skill de Claude Code/OpenCode que realiza mapeamento profundo, autônomo e inteligente de **todas as integrações HTTP** em uma plataforma Salesforce DX.

**Objetivo**: Transformar um codebase Salesforce complexo em uma Knowledge Base estruturada, rastreável e visualmente interativa.

---

## 1. Escopo & Limites

### ✅ Escopo Incluído

- **Apex** - Classes com HttpRequest, Http.send(), Continuations, @future(callout=true), Queueable
- **LWC** - Componentes que invocam @AuraEnabled e disparam callouts
- **Metadata XML** - Named Credentials, External Credentials, Custom Metadata, Custom Labels
- **Padrões Avançados** - FFLIB (Domain, Selector, Service), Abstract classes, Interfaces
- **Eventos** - LightningMessageService, PubSub (rastreamento de fluxo de dados)
- **Autenticação** - OAuth2, API Key, mTLS, SigV4, BasicAuth
- **Payloads** - DTO-based, Map-based, String-literal, inferência inteligente

### ❌ Fora do Escopo

- Padrão de REST (@RestResource) - apenas leitura, não mapeamento de invocação
- Flow/Process Builder (legado) - foco em Apex/LWC moderno
- Batch jobs assíncronos sem Http.send()
- Visualforce (foco em LWC, mas pode ser adicionado depois)

---

## 2. Arquitetura de Componentes

```
┌─────────────────────────────────────────────────────────────────┐
│                    ENTRADA: Caminho Local ORG                    │
│              (ex: C:\projects\salesforce-dx)                     │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                    ┌──────▼──────┐
                    │  Fase 1     │
                    │ Descoberta  │
                    │ Iterativa   │
                    └──────┬──────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
   ┌────▼────┐         ┌───▼────┐        ┌───▼─────┐
   │ Scan    │         │ Scan   │        │ Scan    │
   │ Apex    │         │  LWC   │        │ XMLs    │
   │ (.cls)  │         │ (.js)  │        │Metadata │
   └────┬────┘         └───┬────┘        └───┬─────┘
        │                  │                  │
        └──────────────────┼──────────────────┘
                           │
                    ┌──────▼──────────┐
                    │  Fase 2         │
                    │ Análise de      │
                    │ Contexto &      │
                    │ Payloads        │
                    └──────┬──────────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
   ┌────▼─────┐        ┌───▼─────┐      ┌────▼────┐
   │Payload   │        │Métodos  │      │Autent.  │
   │Detection │        │& Endpoints      │Parse    │
   │(3 tipos) │        │         │      │         │
   └────┬─────┘        └───┬─────┘      └────┬────┘
        │                  │                  │
        └──────────────────┼──────────────────┘
                           │
                    ┌──────▼──────────┐
                    │  Fase 3         │
                    │ Mapeamento de   │
                    │ Linhagem &      │
                    │ Relacionamentos │
                    └──────┬──────────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
   ┌────▼──────────┐  ┌────▼─────────┐  ┌────▼────────┐
   │LWC -> Apex    │  │Apex Abstrato │  │Custom Meta  │
   │Resolution     │  │Resolução     │  │Parsing      │
   └────┬──────────┘  └────┬─────────┘  └────┬────────┘
        │                  │                  │
        └──────────────────┼──────────────────┘
                           │
                    ┌──────▼──────────────┐
                    │  Fase 4             │
                    │ Compilação da KB &  │
                    │ Dashboard           │
                    └──────┬──────────────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
   ┌────▼────────────┐ ┌───▼──────┐    ┌────▼──────────┐
   │database.json    │ │relations │    │docs/index.html│
   │(tipado)         │ │hip.json  │    │(Dashboard)    │
   └─────────────────┘ └──────────┘    └───────────────┘
                           │
                    ┌──────▼──────────┐
                    │ Validação Pós   │
                    │ -Collection     │
                    │ (Tests)         │
                    └──────┬──────────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
   ┌────▼────┐         ┌───▼────┐        ┌───▼─────┐
   │Schema   │         │Conectiv│        │Cobertura│
   │Valid.   │         │idade   │        │Análise  │
   └─────────┘         └────────┘        └─────────┘
                           │
                    ┌──────▼──────────┐
                    │  SAÍDA          │
                    │ Knowledge Base  │
                    │ + Dashboard     │
                    └─────────────────┘
```

---

## 3. Detalhamento de Fases

### Fase 1: Descoberta Iterativa

**Objetivo**: Mapear estrutura completa do projeto e indexar todos os artefatos relevantes.

**Entrada**: Caminho do projeto local

**Processo**:
1. Listar todos os diretórios em `force-app/main/default/`
2. Buscar arquivos:
   - `**/*.cls` (Apex classes)
   - `**/*.js` (LWC JavaScript)
   - `**/namedCredentials/*.namedCredential-meta.xml`
   - `**/externalCredentials/*.externalCredential-meta.xml`
   - `**/customMetadata/**/*.md-meta.xml`
   - `**/labels/*.labels-meta.xml`
3. Para cada arquivo, armazenar:
   - Caminho completo
   - Tipo (class, lwc, namedCred, etc)
   - Tamanho (para determinar complexidade)
   - Hash SHA1 (para detecção de mudanças em CI/CD)

**Saída**: `.callout-cache/discovery.json` (index temporal)

**Cache Temporal**: Salvar progresso a cada 50 arquivos para permitir retomada

---

### Fase 2: Análise de Contexto & Engenharia Reversa de Payloads

**Objetivo**: Extrair de cada artefato o máximo de contexto técnico.

#### 2a. Análise de Apex Classes

Para cada `.cls` contendo `HttpRequest` ou `Http.send()`:

```
1. Encontrar instâncias de HttpRequest
   ↓
2. Extrair:
   - req.setEndpoint(...)   → URL ou callout:NamedCred
   - req.setMethod(...)     → GET/POST/PUT/DELETE/PATCH
   - req.setHeader(...)     → Autenticação, content-type
   - req.setBody(...)       → Payload (estratégia 1-3)
   - req.setTimeout(...)    → Timeout em ms
   ↓
3. Analisar contexto:
   - Nome da classe (inferir jornada)
   - @AuraEnabled ? → é invocado por LWC
   - @future(callout=true) ? → async
   - Dentro de Queueable ? → queued
   - Dentro de Continuation ? → long-running
   ↓
4. Buscar tratamento de erro:
   - Try-catch ? Qual exceção?
   - Retry logic ? Como?
   - Fallback ? Qual?
```

#### 2b. Detecção de Payloads (3 Estratégias)

**Estratégia 1: DTO Tipado** (confiança: 100%)
```apex
req.setBody(JSON.serialize(new PaymentRequest(amount, currency)));
```
→ Localizar classe `PaymentRequest`, mapear fields

**Estratégia 2: Map Dinâmico** (confiança: 70-80%)
```apex
Map<String, Object> payload = new Map<String, Object>();
payload.put('amount', amount);
payload.put('currency', 'BRL');
req.setBody(JSON.serialize(payload));
```
→ Rastrear puts() em sequência, inferir structure

**Estratégia 3: String Literal** (confiança: 60-70%)
```apex
String body = '{"amount": 150.00, "currency": "BRL"}';
req.setBody(body);
```
→ Parsear JSON diretamente

**Fallback: Desconhecido** (confiança: 20%)
```apex
req.setBody(buildDynamicPayload(data)); // função externa
```
→ Marcar como `inferred: true, confidence: 0.2, pattern: "UNKNOWN_BUILDER"`

#### 2c. Análise de LWC

Para cada `.js` em `lwc/`:

```
1. Buscar importações:
   import myApexMethod from '@salesforce/apex/ControllerName.methodName'
   ↓
2. Encontrar invocações:
   myApexMethod({params}) ou myApexMethod.invoke({params})
   ↓
3. Mapear contexto de disparo:
   - Dentro de qual função JS?
   - Qual evento a ativa? (click, change, etc)
   - Há condicionais?
   ↓
4. Buscar comunicação via eventos:
   - LightningMessageService.publish()
   - import ... from 'c/pubsub'
```

#### 2d. Parsing de XMLs de Metadata

```
namedCredential-meta.xml
  ├── endpoint (URL)
  ├── principalType (OAuth/Credentials)
  └── allowMergeFieldsInHeader / Body

externalCredential-meta.xml
  ├── authenticationProtocol (OAuth2, etc)
  ├── oauthTokenEndpoint
  ├── oauthScopes
  └── customHeaders

customMetadata-meta.xml
  ├── Extrair todos os fields
  └── Buscar por padrões: URL, endpoint, baseUrl, path

labels-meta.xml
  ├── Buscar <short> contendo http://, /api/, etc
```

**Saída**: `.callout-cache/analysis.json`

---

### Fase 3: Mapeamento de Linhagem & Relacionamentos

**Objetivo**: Conectar LWC → Apex → Metadata → Endpoint em uma cadeia única.

#### 3a. Resolução de Abstrações

Se um LWC chama:
```apex
import syncCustomer from '@salesforce/apex/OrgService.syncCustomer'
```

E `OrgService.syncCustomer` é abstrato:
```apex
public virtual void syncCustomer(Customer cust) { }
```

Encontrar a **implementação real**:
1. Buscar classe que estende `OrgService`
2. Encontrar override de `syncCustomer`
3. Naquela classe, buscar `Http.send()` ou `Continuation`

#### 3b. Construção do Grafo de Dependências

Para cada callout encontrado:

```
callout_id: "CALLOUT-SF-001"
  ├─ lwcComponents: [
  │    {nome: "checkoutForm", arquivo: "...", funcao: "handlePayment"}
  │  ]
  ├─ apexClasses: [
  │    {nome: "CheckoutController", metodo: "processCheckout", tipo: "entry"},
  │    {nome: "PaymentService", metodo: "charge", tipo: "executor"}
  │  ]
  ├─ namedCredential: "Apigee_Payment"
  ├─ externalCredential: "Apigee_OAuth"
  ├─ customMetadata: [
  │    {name: "Integration_Settings.Payment_Timeout", value: "30"}
  │  ]
  └─ endpoints: [
       {url: "/v1/payments/charge", method: "POST"}
     ]
```

#### 3c. Detecção de Padrões Avançados

- **Continuations**: Se há `Continuation cont = new Continuation(timeout)`, marcar como `longRunning: true`
- **@future**: Se método tem `@future(callout=true)`, marcar como `async: true`
- **Queueable**: Se implementa `Queueable`, marcar como `queued: true`
- **FFLIB Pattern**: Se classe estende `fflib_SObjectDomain`, `fflib_Selector`, `fflib_Service`, rastrear através da hierarquia

**Saída**: `.callout-cache/lineage.json`

---

### Fase 4: Compilação da Knowledge Base & Dashboard

#### 4a. Consolidação em database.json

Mesclar dados de:
- `discovery.json` (índice)
- `analysis.json` (contexto técnico)
- `lineage.json` (relacionamentos)

Aplicar **cálculos dinâmicos**:
- **Jornada** (baseada em semântica de classe/método)
- **Criticidade** (baseada em frequência, tipo de op, timeout)
- **Domain** (baseada em pasta/metadata)

Gerar `database.json` final com todos os campos do schema.

#### 4b. Compilação do Dashboard HTML

Gerar `docs/index.html` com:
- Data embedding (injetar `window.CALLOUT_DATABASE = {...}`)
- Busca fuzzy via Fuse.js
- Filtros multidimensionais
- Visualizações (Lista, Matriz, Grafo)
- Design Enterprise Density (sem AI slop)

**Saída**: `.callout-kb/database.json` e `docs/index.html`

---

### Fase 5: Validação Pós-Collection

**Objetivo**: Garantir integridade e qualidade dos dados extraídos.

#### 5a. Validação de Schema

Verificar que cada record em `database.json`:
- Possui todos os campos obrigatórios
- Valores estão dentro de tipos esperados
- Confiança de inferred está entre 0 e 1

#### 5b. Validação de Conectividade (Opcional)

Se ORG for acessível:
1. Conectar via SFDX (Force CLI)
2. Para cada Named Credential, fazer query na ORG
3. Verificar se está ativa e configurada
4. Marcar `validated: true/false` em database.json

#### 5c. Análise de Cobertura

Calcular métricas:
```json
{
  "cobertura": {
    "totalCallouts": 45,
    "calloutsMapeados": 43,
    "percentualCobertura": 95.5,
    "endpointsOrfaos": 2,
    "apexNaoDocumentado": 0,
    "lwcComErro": 0
  }
}
```

---

## 4. Estrutura de Dados

### 4.1 Banco de Dados Principal: `.callout-kb/database.json`

Ver `SCHEMA.md` para detalhes completos.

### 4.2 Índice de Relacionamentos: `.callout-kb/relationships.json`

```json
{
  "calloutDependencies": [
    {
      "source": "CALLOUT-SF-001",
      "target": "CALLOUT-SF-002",
      "reason": "PaymentService aguarda AuthService completar"
    }
  ],
  "classToCallout": {
    "PaymentController": ["CALLOUT-SF-001"],
    "AuthService": ["CALLOUT-SF-003", "CALLOUT-SF-004"]
  },
  "namedCredToCallouts": {
    "Apigee_Payment": ["CALLOUT-SF-001", "CALLOUT-SF-002"],
    "AWS_Auth": ["CALLOUT-SF-003"]
  },
  "lwcToCallouts": {
    "checkoutForm": ["CALLOUT-SF-001"],
    "userAuth": ["CALLOUT-SF-003"]
  }
}
```

### 4.3 Métricas & Telemetria: `.callout-kb/metrics.json`

```json
{
  "timestamp": "2025-09-15T10:30:00Z",
  "versao": "1.0",
  "cobertura": { "totalCallouts": 45, "mapeados": 43 },
  "performance": {
    "tempoExecucao": 45000,
    "arquivosScaneados": 1250,
    "payloadsInferidos": 12,
    "abstracoesResolvidas": 8
  }
}
```

---

## 5. Fluxo de Autonomia (Zero-Prompt Mode)

Quando um valor é desconhecido:

```
┌─────────────────────────────┐
│ Parâmetro desconhecido?     │
└──────────────┬──────────────┘
               │
               ├─ Consegue inferir? ──YES──> Calcular confidence
               │                             │
               │                             └─> Marcar inferred: true
               │
               └─ NÃO ──> Usar fallback/padrão
                          │
                          └─> Marcar confidence: 0.2-0.4
                              Continuar sem bloquear
```

**Exemplos**:
- URL dinâmica desconhecida → fallback: "callout:UNKNOWN"
- Payload indeterminado → fallback: pattern "MAP_BUILDER"
- Jornada não mapeável → fallback: "UNMAPPED"

**Nunca bloquear a execução. Sempre marcar confiança.**

---

## 6. Integração em CI/CD

### GitHub Actions Workflow

```yaml
name: Sync Callout Mapper
on:
  pull_request:
    paths:
      - 'force-app/**'

jobs:
  map:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: |
          /sf-autonomous-mapper --path force-app --ci-mode
      - name: Compare Changes
        run: |
          # Detectar novos callouts, removidos, modificados
          # Falhar se há callouts sem documentação
```

---

## 7. Anti-Slop & Qualidade

### Diretrizes de Design do Dashboard

✅ **Obrigatório**:
- Cores neutras (Slate/Zinc)
- Tipografia monoespaçada para código
- Sem gradientes genéricos
- Feedback visual explícito
- Tabelas compactas

❌ **Proibido**:
- Gradientes roxo/azul
- Sombras exageradas
- Cards flutuantes genéricos
- Animações sem propósito
- Layouts que parecem "AI-gerado"

---

## 8. Sucesso & Métricas

### Critérios de Aceitação

✅ Agente executa sem intervenção manual  
✅ Detecta 95%+ dos callouts existentes  
✅ Payloads têm confiança média > 75%  
✅ Linhagem rastreada para 100% dos callouts encontrados  
✅ Dashboard é interativo e performático (< 2s carregamento)  
✅ Schema JSON é válido (passível de validação)  
✅ Sem erros críticos no console JS  

---

## 9. Ciclo de Vida Futuro

**Fase 1** (MVP): Descoberta + Análise + Compilação  
**Fase 2**: Validação + CI/CD  
**Fase 3**: Observabilidade (links para Setup, Apex Logs)  
**Fase 4**: Multi-ambiente (Dev/Staging/Prod)  
**Fase 5**: Integração com Salesforce CLI para telemetria real
