# 🧪 Caso de Teste End-to-End - Plataforma Salesforce de Pagamentos

## Visão Geral

Este documento descreve um **caso de uso realista e completo** que será usado para validar a skill `sf-autonomous-mapper` em um cenário de ORG Salesforce profissional.

**Objetivo**: Testar todos os componentes da skill em um ambiente controlado mas realista, cobrindo:
- ✅ Callouts simples (DTO tipado)
- ✅ Callouts complexos (Maps dinâmicos)
- ✅ Padrões avançados (Continuations, @future, Queueable)
- ✅ LWC com múltiplas interações
- ✅ Named Credentials & External Credentials
- ✅ Custom Metadata configuração
- ✅ Linhagem multi-salto

---

## Parte 1: Descrição da Plataforma de Teste

### Contexto de Negócio

**Nome**: Plataforma de Pagamentos Integrada (PPI)  
**Descrição**: Sistema Salesforce que orquestra fluxo de pagamentos integrando múltiplos gateways (Apigee, Auth0, AWS)

**Jornadas Principais**:
1. **Autenticação** - OAuth2 com Auth0
2. **Pagamento** - Apigee Payment Gateway
3. **Reconciliação** - AWS Lambda via API Gateway
4. **Notificação** - Webhook de terceiros

### Arquitetura Esperada

```
Frontend (LWC)
  ├─ checkoutPaymentForm
  │   ├─ handlePaymentSubmit()
  │   │   └─ invoca CheckoutController.processCheckout()
  │   │
  │   └─ handleAuthLogin()
  │       └─ invoca AuthController.authenticate()
  │
  └─ orderManagement
      └─ handleReconciliation()
          └─ invoca ReconciliationService.sync()

Backend (Apex)
  ├─ Controllers
  │   ├─ CheckoutController.cls
  │   ├─ AuthController.cls
  │   └─ OrderController.cls
  │
  ├─ Services
  │   ├─ PaymentService.cls (executa HttpRequest)
  │   ├─ AuthService.cls (OAuth, @future)
  │   ├─ ReconciliationService.cls (Queueable)
  │   └─ NotificationService.cls (Continuation)
  │
  └─ Utilities
      └─ HttpUtility.cls (base abstrata)

Metadata
  ├─ Named Credentials
  │   ├─ Apigee_Payment_Gateway
  │   ├─ Auth0_OAuth
  │   └─ AWS_Lambda_API
  │
  ├─ External Credentials
  │   ├─ Apigee_OAuth
  │   ├─ Auth0_Client_Credentials
  │   └─ AWS_SigV4
  │
  └─ Custom Metadata
      └─ Integration_Settings (Payment_Timeout, Retry_Policy, etc)
```

---

## Parte 2: Callouts a Serem Encontrados

### Callout 1: Payment Charge (DTO Tipado)

**ID Esperado**: `CALLOUT-PAY-001`

**Arquivo**: `force-app/main/default/classes/PaymentService.cls`

```apex
public class PaymentService {
    
    public class ChargeRequest {
        public Decimal amount;
        public String currency;
        public String paymentMethod;
        public String orderId;
    }
    
    public class ChargeResponse {
        public String transactionId;
        public String status;
        public Long timestamp;
    }
    
    public static ChargeResponse executePaymentCharge(
        Decimal amount, 
        String currency, 
        String paymentMethod,
        String orderId
    ) {
        // Criar request tipado
        ChargeRequest chargeReq = new ChargeRequest();
        chargeReq.amount = amount;
        chargeReq.currency = currency;
        chargeReq.paymentMethod = paymentMethod;
        chargeReq.orderId = orderId;
        
        // Montar HTTP request
        HttpRequest req = new HttpRequest();
        req.setEndpoint('callout:Apigee_Payment_Gateway/v1/payments/charge');
        req.setMethod('POST');
        req.setHeader('Content-Type', 'application/json');
        req.setHeader('X-Request-ID', generateRequestId());
        req.setTimeout(30000);
        
        // Serializar DTO
        String requestBody = JSON.serialize(chargeReq);
        req.setBody(requestBody);
        
        try {
            Http http = new Http();
            HttpResponse resp = http.send(req);
            
            if (resp.getStatusCode() == 200) {
                ChargeResponse chargeResp = (ChargeResponse) JSON.deserialize(
                    resp.getBody(), 
                    ChargeResponse.class
                );
                return chargeResp;
            } else {
                throw new PaymentException('Charge failed: ' + resp.getStatus());
            }
        } catch (Exception ex) {
            // Retry logic
            if (shouldRetry(ex)) {
                return executePaymentCharge(amount, currency, paymentMethod, orderId);
            }
            throw new PaymentException('Fatal charge error', ex);
        }
    }
}
```

**O que a skill deve encontrar**:
- ✅ HttpRequest em `executePaymentCharge()`
- ✅ Endpoint: `callout:Apigee_Payment_Gateway/v1/payments/charge`
- ✅ Método: `POST`
- ✅ Payload: DTO `ChargeRequest` (confiança 100%)
- ✅ Response: DTO `ChargeResponse` (confiança 100%)
- ✅ Headers: `X-Request-ID` customizado
- ✅ Timeout: 30000ms
- ✅ Tratamento de erro: Try-catch com retry
- ✅ Classe relacionada: `CheckoutController` (invoca este método)
- ✅ LWC relacionada: `checkoutPaymentForm` (chama `CheckoutController`)

**Validação Expected**:
```json
{
  "id": "CALLOUT-PAY-001",
  "jornada": "Checkout & Pagamentos",
  "gateway": "Apigee",
  "metadadosRede": {
    "metodo": "POST",
    "basePath": "/v1/payments",
    "endpoint": "/charge",
    "urlCompleta": "callout:Apigee_Payment_Gateway/v1/payments/charge",
    "tokenEndpoint": "https://auth0-domain/oauth/v2/token",
    "tipoAutenticacao": "OAuth2 Client Credentials"
  },
  "salesforceArtefatos": {
    "namedCredential": "Apigee_Payment_Gateway",
    "externalCredential": "Apigee_OAuth_Credentials",
    "apexClasses": [
      {"nome": "PaymentService.cls", "metodoApex": "executePaymentCharge", "tipoUso": "Executor"},
      {"nome": "CheckoutController.cls", "metodoApex": "processCheckout", "tipoUso": "Entry point"}
    ],
    "lwcComponentes": [
      {"nome": "checkoutPaymentForm", "arquivo": "checkoutPaymentForm.js", "funcaoJS": "handlePaymentSubmit"}
    ]
  },
  "payloadSchema": {
    "request": {
      "dtoClass": "ChargeRequest",
      "exemplo": "{\"amount\": 150.00, \"currency\": \"BRL\", \"paymentMethod\": \"PIX\", \"orderId\": \"ORD-001\"}",
      "inferred": false,
      "confianca": 1.0
    },
    "response": {
      "dtoClass": "ChargeResponse",
      "exemplo": "{\"transactionId\": \"TXN-998123\", \"status\": \"APPROVED\", \"timestamp\": 1695000000000}",
      "inferred": false,
      "confianca": 1.0
    }
  },
  "performance": {
    "timeoutMs": 30000,
    "expectedLatencyMs": 500,
    "rateLimit": "1000/hour"
  },
  "resilience": {
    "temRetry": true,
    "estrategiaRetry": "conditional-simple",
    "maxAttempts": 2,
    "tratamentoErro": "Valida status 200, retry em exceção"
  },
  "explicacaoDetalhada": "Callout acionado quando usuário confirma pagamento no LWC checkoutPaymentForm. O Apex CheckoutController orquestra com PaymentService que executa a cobrança no Apigee usando Named Credential Apigee_Payment_Gateway com autenticação OAuth2."
}
```

---

### Callout 2: OAuth Token (Map Dinâmico + @future)

**ID Esperado**: `CALLOUT-AUTH-001`

**Arquivo**: `force-app/main/default/classes/AuthService.cls`

```apex
public class AuthService {
    
    @future(callout=true)
    public static void obtainAccessToken(String clientId, String clientSecret) {
        // Construir payload com Map
        Map<String, Object> payload = new Map<String, Object>();
        payload.put('client_id', clientId);
        payload.put('client_secret', clientSecret);
        payload.put('grant_type', 'client_credentials');
        payload.put('scope', 'api:write');
        
        HttpRequest req = new HttpRequest();
        req.setEndpoint('callout:Auth0_OAuth/oauth/v2/token');
        req.setMethod('POST');
        req.setHeader('Content-Type', 'application/x-www-form-urlencoded');
        req.setTimeout(10000);
        
        String body = JSON.serialize(payload);
        req.setBody(body);
        
        try {
            Http http = new Http();
            HttpResponse resp = http.send(req);
            
            if (resp.getStatusCode() == 200) {
                // Armazenar token em cache
                Map<String, Object> respMap = (Map<String, Object>) JSON.deserializeUntyped(resp.getBody());
                String accessToken = (String) respMap.get('access_token');
                
                // Salvar em Custom Metadata ou cache
                updateTokenCache(accessToken);
            }
        } catch (Exception ex) {
            // Log mas não relança (async)
            logAuthError(ex);
        }
    }
    
    private static void updateTokenCache(String token) {
        // Implement cache update
    }
    
    private static void logAuthError(Exception ex) {
        // Implement logging
    }
}
```

**O que a skill deve encontrar**:
- ✅ Método com anotação `@future(callout=true)` → marcado como async
- ✅ HttpRequest para token OAuth
- ✅ Endpoint: `callout:Auth0_OAuth/oauth/v2/token`
- ✅ Método: `POST`
- ✅ Payload: Map dinâmico (confiança 70-80%)
- ✅ Timeout: 10000ms
- ✅ Padrão: Async callout
- ✅ Tratamento de erro: Try-catch com log (sem retry)

**Validação Expected**:
```json
{
  "id": "CALLOUT-AUTH-001",
  "jornada": "Autenticação",
  "padraoAvancado": "@future(callout=true)",
  "async": true,
  "payloadSchema": {
    "request": {
      "metodo": "MAP_DINAMICO",
      "confianca": 0.78,
      "inferred": true,
      "campos": [
        {"nome": "client_id", "tipo_inferido": "String", "confianca": 0.95},
        {"nome": "client_secret", "tipo_inferido": "String", "confianca": 0.95},
        {"nome": "grant_type", "tipo_inferido": "String", "confianca": 1.0},
        {"nome": "scope", "tipo_inferido": "String", "confianca": 0.9}
      ]
    }
  }
}
```

---

### Callout 3: Reconciliação (Queueable + Custom Metadata)

**ID Esperado**: `CALLOUT-SYNC-001`

**Arquivo**: `force-app/main/default/classes/ReconciliationService.cls`

```apex
public class ReconciliationService implements Queueable, Database.AllowsCallouts {
    
    private String batchId;
    
    public ReconciliationService(String batchId) {
        this.batchId = batchId;
    }
    
    public void execute(QueueableContext context) {
        try {
            String awsApiUrl = Integration_Settings__mdt.getInstance('AWS_Lambda_Sync').API_Endpoint__c;
            String awsApiKey = Integration_Settings__mdt.getInstance('AWS_Lambda_Sync').API_Key__c;
            
            Map<String, Object> syncPayload = new Map<String, Object>();
            syncPayload.put('batchId', batchId);
            syncPayload.put('action', 'reconcile');
            syncPayload.put('timestamp', System.now().getTime());
            
            HttpRequest req = new HttpRequest();
            req.setEndpoint('callout:AWS_Lambda_API' + awsApiUrl);
            req.setMethod('PUT');
            req.setHeader('X-API-Key', awsApiKey);
            req.setHeader('Content-Type', 'application/json');
            req.setTimeout(60000);
            
            req.setBody(JSON.serialize(syncPayload));
            
            Http http = new Http();
            HttpResponse resp = http.send(req);
            
            if (resp.getStatusCode() != 200) {
                // Requeue if failed
                System.enqueueJob(new ReconciliationService(batchId));
            }
        } catch (Exception ex) {
            logReconciliationError(ex);
        }
    }
}
```

**O que a skill deve encontrar**:
- ✅ Classe implementa `Queueable, Database.AllowsCallouts`
- ✅ HttpRequest em `execute()` method
- ✅ Endpoint vem de Custom Metadata: `Integration_Settings__mdt`
- ✅ Dinâmico: `'callout:AWS_Lambda_API' + awsApiUrl`
- ✅ Método: `PUT`
- ✅ Payload: Map dinâmico (confiança 75%)
- ✅ Header dinâmico: `X-API-Key` vem de Custom Metadata
- ✅ Timeout: 60000ms (grande, operação longa)
- ✅ Padrão: Queueable (processamento assíncrono)
- ✅ Retry automático: `System.enqueueJob()`

**Validação Expected**:
```json
{
  "id": "CALLOUT-SYNC-001",
  "jornada": "Sincronização & Reconciliação",
  "gateway": "AWS Lambda",
  "padraoAvancado": "Queueable + Database.AllowsCallouts",
  "queued": true,
  "metadadosRede": {
    "metodo": "PUT",
    "basePath": "/api/v1/reconcile",
    "endpoint": "/{batchId}",
    "urlCompleta": "callout:AWS_Lambda_API/api/v1/reconcile",
    "tipoAutenticacao": "API Key (Custom Metadata)"
  },
  "performance": {
    "timeoutMs": 60000,
    "expectedLatencyMs": 5000,
    "rateLimit": "100/hour"
  },
  "resilience": {
    "temRetry": true,
    "estrategiaRetry": "queueable-requeue",
    "maxAttempts": 3
  }
}
```

---

### Callout 4: Notificação (Continuation - Long Running)

**ID Esperado**: `CALLOUT-NOTIF-001`

**Arquivo**: `force-app/main/default/classes/NotificationService.cls`

```apex
public class NotificationService {
    
    public static String initiateNotification(String orderId, String email) {
        // Criar Continuation para long-running operation
        Continuation con = new Continuation(120);  // 120 segundo timeout
        con.continuationMethod = 'handleNotificationResponse';
        
        HttpRequest req = new HttpRequest();
        req.setMethod('POST');
        req.setEndpoint('callout:Notification_Service/api/v1/notify');
        req.setHeader('Content-Type', 'application/json');
        
        Map<String, Object> notifPayload = new Map<String, Object>();
        notifPayload.put('orderId', orderId);
        notifPayload.put('recipient', email);
        notifPayload.put('type', 'order_confirmation');
        notifPayload.put('metadata', new Map<String, String>{
            'currency': 'BRL',
            'timestamp': String.valueOf(System.now())
        });
        
        req.setBody(JSON.serialize(notifPayload));
        String requestId = con.addHttpRequest(req);
        
        return requestId;
    }
    
    public static Continuation handleNotificationResponse(Map<String, HttpResponse> responses) {
        HttpResponse resp = responses.get('requestId');
        
        if (resp != null && resp.getStatusCode() == 202) {
            // Process async response
            Map<String, Object> respMap = (Map<String, Object>) JSON.deserializeUntyped(resp.getBody());
            String notificationId = (String) respMap.get('notification_id');
            
            // Store notification tracking
            updateNotificationStatus(notificationId, 'SENT');
        }
        
        return null;
    }
    
    private static void updateNotificationStatus(String notifId, String status) {
        // Implement status update
    }
}
```

**O que a skill deve encontrar**:
- ✅ Uso de `Continuation` → padrão long-running
- ✅ HttpRequest adicionado via `con.addHttpRequest(req)`
- ✅ Endpoint: `callout:Notification_Service/api/v1/notify`
- ✅ Método: `POST`
- ✅ Callback method: `handleNotificationResponse`
- ✅ Payload: Map dinâmico com estrutura aninhada (confiança 70%)
- ✅ Status code esperado: 202 (Accepted)
- ✅ Padrão: Continuation (async callback)

**Validação Expected**:
```json
{
  "id": "CALLOUT-NOTIF-001",
  "jornada": "Notificações & Comunicação",
  "padraoAvancado": "Continuation",
  "longRunning": true,
  "continuationTimeout": 120,
  "callbackMethod": "handleNotificationResponse",
  "metadadosRede": {
    "metodo": "POST",
    "basePath": "/api/v1",
    "endpoint": "/notify",
    "statusEsperado": 202
  }
}
```

---

## Parte 3: LWC Components

### LWC 1: checkoutPaymentForm

**Arquivo**: `force-app/main/default/lwc/checkoutPaymentForm/checkoutPaymentForm.js`

```javascript
import { LightningElement, api } from 'lwc';
import { ShowToastEvent } from 'lightning/platformShowToastEvent';
import processCheckout from '@salesforce/apex/CheckoutController.processCheckout';

export default class CheckoutPaymentForm extends LightningElement {
    @api orderId;
    amount = 0;
    paymentMethod = 'PIX';
    
    handlePaymentSubmit(event) {
        event.preventDefault();
        
        // Chama Apex que dispara CALLOUT-PAY-001
        processCheckout({
            orderId: this.orderId,
            amount: this.amount,
            paymentMethod: this.paymentMethod
        })
        .then(result => {
            this.dispatchEvent(
                new ShowToastEvent({
                    title: 'Success',
                    message: 'Payment processed',
                    variant: 'success'
                })
            );
        })
        .catch(error => {
            console.error('Payment error', error);
        });
    }
}
```

**O que a skill deve encontrar**:
- ✅ Import de método Apex: `CheckoutController.processCheckout`
- ✅ Invocação: `processCheckout({...})`
- ✅ Função disparadora: `handlePaymentSubmit`
- ✅ Relacionamento: checkoutPaymentForm → CALLOUT-PAY-001

---

### LWC 2: orderManagement

**Arquivo**: `force-app/main/default/lwc/orderManagement/orderManagement.js`

```javascript
import { LightningElement, api } from 'lwc';
import initiateSync from '@salesforce/apex/ReconciliationController.initiateSync';

export default class OrderManagement extends LightningElement {
    @api batchId;
    
    handleReconciliation(event) {
        // Chama Apex que dispara CALLOUT-SYNC-001
        initiateSync({ batchId: this.batchId })
            .then(result => {
                console.log('Sync initiated', result);
            });
    }
}
```

**O que a skill deve encontrar**:
- ✅ Import de método Apex: `ReconciliationController.initiateSync`
- ✅ Função disparadora: `handleReconciliation`
- ✅ Relacionamento: orderManagement → CALLOUT-SYNC-001

---

## Parte 4: Named Credentials & External Credentials

### Named Credential 1: Apigee_Payment_Gateway

**Arquivo**: `force-app/main/default/namedCredentials/Apigee_Payment_Gateway.namedCredential-meta.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<NamedCredential xmlns="http://soap.sforce.com/2006/04/metadata">
    <allowMergeFieldsInBody>false</allowMergeFieldsInBody>
    <allowMergeFieldsInHeader>false</allowMergeFieldsInHeader>
    <description>Apigee Payment Gateway Integration</description>
    <endpoint>https://api.apigee.example.com/api/v1/payments</endpoint>
    <generateAuthorizationHeader>true</generateAuthorizationHeader>
    <label>Apigee Payment Gateway</label>
    <principalType>NamedPrincipal</principalType>
    <protocol>OAuth</protocol>
    <oauthTokenEndpoint>https://auth0.example.com/oauth/v2/token</oauthTokenEndpoint>
    <externalCredential>Apigee_OAuth_Credentials</externalCredential>
</NamedCredential>
```

**O que a skill deve encontrar**:
- ✅ Endpoint: `https://api.apigee.example.com/api/v1/payments`
- ✅ Tipo autenticação: OAuth
- ✅ Token endpoint: `https://auth0.example.com/oauth/v2/token`
- ✅ External Credential referenciada: `Apigee_OAuth_Credentials`
- ✅ Relacionamento: Usado por CALLOUT-PAY-001

---

### External Credential 1: Apigee_OAuth_Credentials

**Arquivo**: `force-app/main/default/externalCredentials/Apigee_OAuth_Credentials.externalCredential-meta.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ExternalCredential xmlns="http://soap.sforce.com/2006/04/metadata">
    <authenticationProtocol>OAuth2</authenticationProtocol>
    <label>Apigee OAuth Credentials</label>
    <oauthClientCredentialFlow>
        <clientId>***</clientId>
        <clientSecret>***</clientSecret>
        <oauthTokenEndpoint>https://auth0.example.com/oauth/v2/token</oauthTokenEndpoint>
        <scope>api:write payment:execute</scope>
    </oauthClientCredentialFlow>
</ExternalCredential>
```

**O que a skill deve encontrar**:
- ✅ Protocolo: OAuth2 Client Credentials
- ✅ Token endpoint: `https://auth0.example.com/oauth/v2/token`
- ✅ Scopes: `api:write payment:execute`
- ✅ Relacionamento: Usado por Named Credential `Apigee_Payment_Gateway`

---

## Parte 5: Custom Metadata

### Integration_Settings

**Arquivo**: `force-app/main/default/customMetadata/Integration_Settings.Payment_Timeout.md-meta.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomMetadata xmlns="http://soap.sforce.com/2006/04/metadata" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <label>Payment Timeout</label>
    <protected>false</protected>
    <values>
        <name>API_Endpoint__c</name>
        <value xsi:type="xsd:string">/api/v1/payments</value>
    </values>
    <values>
        <name>Timeout_Ms__c</name>
        <value xsi:type="xsd:int">30000</value>
    </values>
    <values>
        <name>Retry_Policy__c</name>
        <value xsi:type="xsd:string">EXPONENTIAL_BACKOFF</value>
    </values>
    <values>
        <name>Business_Domain__c</name>
        <value xsi:type="xsd:string">Pagamentos</value>
    </values>
</CustomMetadata>
```

**O que a skill deve encontrar**:
- ✅ Business_Domain: `Pagamentos` (para inferir jornada)
- ✅ Timeout: 30000ms
- ✅ Retry Policy: EXPONENTIAL_BACKOFF
- ✅ Endpoint template: `/api/v1/payments`

---

## Parte 6: Validação Expected Final

Após executar a skill no projeto de teste, espera-se:

### Descoberta
```
Total de callouts esperados: 4
Total encontrado: 4 ✅
- CALLOUT-PAY-001 ✅
- CALLOUT-AUTH-001 ✅
- CALLOUT-SYNC-001 ✅
- CALLOUT-NOTIF-001 ✅

Recall: 100% ✅
Precision: 100% ✅
```

### Análise de Payload
```
Confiança média: 0.82 ✅
- DTO: 2/4 (100% confiança)
- Map: 2/4 (75% confiança média)

Distribuição esperada: ✅
```

### Linhagem
```
LWC → Apex → Endpoint:
- checkoutPaymentForm → CheckoutController → CALLOUT-PAY-001 ✅
- orderManagement → ReconciliationController → CALLOUT-SYNC-001 ✅

Rastreamento: 100% ✅
```

### Dashboard
```
Cards encontrados: 4 ✅
Jornadas mapeadas: 3 ✅
  - Autenticação
  - Pagamentos
  - Sincronização

Búsca funciona: ✅
Filtros funcionam: ✅
Linhagem visual completa: ✅
Performance < 2s: ✅
```

---

## Parte 7: Instruções de Setup

### Preparar Projeto de Teste

```bash
# Criar pasta
mkdir -p ~/test-projects/sf-payment-platform
cd ~/test-projects/sf-payment-platform

# Inicializar SFDX
sfdx force:project:create -n payment-platform

# Copiar estrutura (criar arquivos conforme Parte 2-5 acima)

# Estrutura final esperada:
force-app/main/default/
├── classes/
│   ├── PaymentService.cls
│   ├── AuthService.cls
│   ├── ReconciliationService.cls
│   ├── NotificationService.cls
│   ├── CheckoutController.cls
│   ├── ReconciliationController.cls
│   └── Integration_Settings__mdt.cls (mock)
├── lwc/
│   ├── checkoutPaymentForm/
│   │   └── checkoutPaymentForm.js
│   └── orderManagement/
│       └── orderManagement.js
├── namedCredentials/
│   ├── Apigee_Payment_Gateway.namedCredential-meta.xml
│   ├── Auth0_OAuth.namedCredential-meta.xml
│   └── AWS_Lambda_API.namedCredential-meta.xml
├── externalCredentials/
│   ├── Apigee_OAuth_Credentials.externalCredential-meta.xml
│   └── Auth0_Client_Credentials.externalCredential-meta.xml
└── customMetadata/
    └── Integration_Settings.Payment_Timeout.md-meta.xml
```

### Executar Skill

```bash
# Navegar para projeto
cd ~/test-projects/sf-payment-platform

# Executar skill
/sf-autonomous-mapper

# Aguardar execução (2-5 minutos)

# Validar outputs
ls -la .callout-kb/database.json
ls -la docs/index.html

# Abrir dashboard
open docs/index.html
```

### Validação Manual

1. Abrir `docs/index.html` no navegador
2. Validar 4 callouts encontrados
3. Testar busca (ex: "payment", "auth", "sync")
4. Testar filtros por jornada
5. Clicar em cada callout para ver detalhes
6. Validar linhagem visual

---

## Critério de Sucesso Final

✅ **TESTE PASSOU SE**:
- Todos os 4 callouts encontrados
- Schema JSON válido 100%
- Dashboard carrega sem erros
- Busca funciona instantânea
- Linhagem completa para 100%
- Confiança média > 80%
- Tempo execução < 5min
- 0 erros no console JS

❌ **TESTE FALHOU SE**:
- Menos de 3 callouts encontrados
- Schema inválido > 5%
- Dashboard com erros JS
- Busca lenta > 500ms
- Linhagem incompleta em qualquer callout
- Confiança média < 70%
