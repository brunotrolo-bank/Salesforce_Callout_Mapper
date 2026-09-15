# 📊 Schema JSON - Banco de Dados de Callouts

## Visão Geral

Este documento define o **schema JSON completo e expandido** para o arquivo `.callout-kb/database.json`, que é a "fonte da verdade" de toda a Knowledge Base.

**Objetivo**: Permitir:
- ✅ Armazenamento estruturado de callouts
- ✅ Validação automática (via AJV)
- ✅ Injeção no dashboard sem processamento
- ✅ Integração com ferramentas externas (CI/CD, monitoring)
- ✅ Rastreabilidade completa (confiança, método de detecção)

---

## 1. Estrutura de Nível Superior

```json
{
  "metadata": {
    "version": "1.0",
    "generatedAt": "2025-09-15T10:30:00Z",
    "generatedBy": "sf-autonomous-mapper",
    "projectPath": "/path/to/salesforce-dx",
    "totalCallouts": 45,
    "totalLWCComponents": 12,
    "totalApexClasses": 28,
    "coverage": {
      "discoveryRecall": 0.956,
      "discoveryPrecision": 1.0,
      "payloadConfidenceAvg": 0.82,
      "lineageCompletePercentage": 1.0
    }
  },
  "callouts": [ ... ],
  "statistics": { ... },
  "validationResults": { ... }
}
```

---

## 2. Schema Completo da Raiz

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Salesforce Callout Knowledge Base",
  "type": "object",
  "required": ["metadata", "callouts"],
  "properties": {
    "metadata": {
      "type": "object",
      "required": ["version", "generatedAt", "projectPath"],
      "properties": {
        "version": {
          "type": "string",
          "pattern": "^\\d+\\.\\d+(\\.\\d+)?$",
          "description": "Semantic version of database schema"
        },
        "generatedAt": {
          "type": "string",
          "format": "date-time",
          "description": "ISO 8601 timestamp when database was generated"
        },
        "generatedBy": {
          "type": "string",
          "enum": ["sf-autonomous-mapper", "manual"],
          "description": "Source of generation"
        },
        "projectPath": {
          "type": "string",
          "description": "Absolute path to Salesforce DX project"
        },
        "totalCallouts": {
          "type": "integer",
          "minimum": 0,
          "description": "Total number of callouts found"
        },
        "totalLWCComponents": {
          "type": "integer",
          "minimum": 0,
          "description": "Total unique LWC components involved"
        },
        "totalApexClasses": {
          "type": "integer",
          "minimum": 0,
          "description": "Total unique Apex classes involved"
        },
        "coverage": {
          "type": "object",
          "properties": {
            "discoveryRecall": {
              "type": "number",
              "minimum": 0,
              "maximum": 1,
              "description": "Recall metric (found / expected)"
            },
            "discoveryPrecision": {
              "type": "number",
              "minimum": 0,
              "maximum": 1,
              "description": "Precision metric (correct / total found)"
            },
            "payloadConfidenceAvg": {
              "type": "number",
              "minimum": 0,
              "maximum": 1,
              "description": "Average confidence of payload detection"
            },
            "lineageCompletePercentage": {
              "type": "number",
              "minimum": 0,
              "maximum": 1,
              "description": "Percentage of callouts with complete lineage"
            }
          }
        }
      }
    },
    "callouts": {
      "type": "array",
      "items": {
        "$ref": "#/definitions/Callout"
      }
    },
    "statistics": {
      "type": "object",
      "description": "Aggregated statistics"
    },
    "validationResults": {
      "type": "object",
      "description": "Post-collection validation results"
    }
  },
  "definitions": {
    "Callout": { ... }
  }
}
```

---

## 3. Definição Completa de Callout

```json
{
  "Callout": {
    "type": "object",
    "required": [
      "id",
      "jornada",
      "metadadosRede",
      "salesforceArtefatos"
    ],
    "properties": {
      "id": {
        "type": "string",
        "pattern": "^CALLOUT-[A-Z0-9]+-[0-9]{3}$",
        "description": "Unique identifier (ex: CALLOUT-PAY-001)"
      },
      "jornada": {
        "type": "string",
        "enum": [
          "Autenticação",
          "Checkout & Pagamentos",
          "Sincronização & Reconciliação",
          "Notificações & Comunicação",
          "Relatórios & Análise",
          "Suporte & Help Desk",
          "Inventário & Logística",
          "UNMAPPED"
        ],
        "description": "Business journey/domain"
      },
      "dominio": {
        "type": "string",
        "description": "Technical domain (ex: 'Financeiro', 'RH', 'Supply Chain')"
      },
      "descricaoBreve": {
        "type": "string",
        "maxLength": 200,
        "description": "One-liner description"
      },
      "status": {
        "type": "string",
        "enum": ["Production", "Staging", "Development", "Deprecated", "Legacy"],
        "default": "Production",
        "description": "Maturity status"
      },
      "criticidade": {
        "type": "string",
        "enum": ["Baixa", "Média", "Alta", "Crítica"],
        "description": "Risk/importance level (calculated dynamically)"
      },
      "criticidadeJustificativa": {
        "type": "string",
        "description": "Why this criticidade level (ex: 'Used by 15 components')"
      },
      "metadadosRede": {
        "$ref": "#/definitions/MetadadosRede"
      },
      "salesforceArtefatos": {
        "$ref": "#/definitions/SalesforceArtefatos"
      },
      "payloadSchema": {
        "$ref": "#/definitions/PayloadSchema"
      },
      "performance": {
        "$ref": "#/definitions/Performance"
      },
      "resilience": {
        "$ref": "#/definitions/Resilience"
      },
      "seguranca": {
        "$ref": "#/definitions/Seguranca"
      },
      "dependencias": {
        "type": "array",
        "items": {
          "type": "string",
          "pattern": "^CALLOUT-[A-Z0-9]+-[0-9]{3}$"
        },
        "description": "IDs of callouts this depends on"
      },
      "dependentes": {
        "type": "array",
        "items": {
          "type": "string",
          "pattern": "^CALLOUT-[A-Z0-9]+-[0-9]{3}$"
        },
        "description": "IDs of callouts that depend on this"
      },
      "padraoAvancado": {
        "type": "string",
        "enum": [
          "SIMPLES",
          "@future(callout=true)",
          "Queueable",
          "Continuation",
          "FFLIB_Service",
          "FFLIB_Selector",
          "Event-based"
        ],
        "description": "Advanced pattern if applicable"
      },
      "explicacaoDetalhada": {
        "type": "string",
        "maxLength": 1000,
        "description": "Natural language explanation of purpose and flow"
      },
      "documentacaoExterna": {
        "type": "object",
        "properties": {
          "swaggerUrl": {
            "type": "string",
            "format": "uri"
          },
          "readmeUrl": {
            "type": "string",
            "format": "uri"
          },
          "contatoTecnico": {
            "type": "string"
          }
        }
      },
      "rastreabilidade": {
        "$ref": "#/definitions/Rastreabilidade"
      }
    }
  }
}
```

---

## 4. Definição de MetadadosRede

```json
{
  "MetadadosRede": {
    "type": "object",
    "required": ["metodo", "basePath", "endpoint", "tipoAutenticacao"],
    "properties": {
      "metodo": {
        "type": "string",
        "enum": ["GET", "POST", "PUT", "PATCH", "DELETE"],
        "description": "HTTP method"
      },
      "basePath": {
        "type": "string",
        "description": "Base path from Named Credential (ex: /api/v1/payments)"
      },
      "endpoint": {
        "type": "string",
        "description": "Endpoint relative to basePath (ex: /charge)"
      },
      "urlCompleta": {
        "type": "string",
        "description": "Full URL (ex: callout:Apigee_Payment_Gateway/v1/payments/charge)"
      },
      "urlDinamica": {
        "type": "boolean",
        "default": false,
        "description": "True if URL is constructed dynamically"
      },
      "gateway": {
        "type": "string",
        "enum": [
          "Apigee",
          "Kong",
          "AWS API Gateway",
          "Auth0",
          "Salesforce",
          "Customizado",
          "Desconhecido"
        ],
        "description": "Gateway/platform type"
      },
      "tokenEndpoint": {
        "type": "string",
        "format": "uri",
        "description": "OAuth2 token endpoint if applicable"
      },
      "tipoAutenticacao": {
        "type": "string",
        "enum": [
          "OAuth2 Client Credentials",
          "OAuth2 User Password",
          "API Key",
          "mTLS/Certificate",
          "AWS SigV4",
          "BasicAuth",
          "Custom Header",
          "Nenhuma"
        ],
        "description": "Authentication type"
      },
      "statusEsperado": {
        "type": "array",
        "items": {
          "type": "integer",
          "minimum": 100,
          "maximum": 599
        },
        "default": [200],
        "description": "Expected HTTP status codes (ex: [200, 201])"
      },
      "headers": {
        "type": "object",
        "description": "HTTP headers sent",
        "additionalProperties": {
          "type": "object",
          "properties": {
            "valor": {
              "type": "string",
              "description": "Header value or placeholder"
            },
            "dinamico": {
              "type": "boolean",
              "description": "True if value varies at runtime"
            },
            "origem": {
              "type": "string",
              "enum": ["codificado", "custom-metadata", "external-credential"],
              "description": "Where value comes from"
            }
          }
        }
      }
    }
  }
}
```

---

## 5. Definição de SalesforceArtefatos

```json
{
  "SalesforceArtefatos": {
    "type": "object",
    "required": ["namedCredential", "apexClasses"],
    "properties": {
      "namedCredential": {
        "type": "string",
        "description": "Named Credential used (ex: Apigee_Payment_Gateway)"
      },
      "namedCredentialUrl": {
        "type": "string",
        "description": "URL from Named Credential"
      },
      "externalCredential": {
        "type": "string",
        "description": "External Credential used for OAuth"
      },
      "apexClasses": {
        "type": "array",
        "items": {
          "type": "object",
          "required": ["nome", "metodoApex"],
          "properties": {
            "nome": {
              "type": "string",
              "description": "Class name (ex: PaymentService.cls)"
            },
            "metodoApex": {
              "type": "string",
              "description": "Method that contains HttpRequest (ex: executePaymentCharge)"
            },
            "tipoUso": {
              "type": "string",
              "enum": [
                "Executor",
                "Entry point (@AuraEnabled)",
                "Orchestrator",
                "Helper",
                "Abstrato"
              ],
              "description": "Role in the flow"
            },
            "assinatura": {
              "type": "string",
              "description": "Method signature (ex: 'public static ChargeResponse executePaymentCharge(Decimal amount)')"
            },
            "anotacoes": {
              "type": "array",
              "items": {"type": "string"},
              "description": "Annotations (@AuraEnabled, @future, etc)"
            }
          }
        }
      },
      "lwcComponentes": {
        "type": "array",
        "items": {
          "type": "object",
          "required": ["nome", "arquivo"],
          "properties": {
            "nome": {
              "type": "string",
              "description": "Component name (ex: checkoutPaymentForm)"
            },
            "arquivo": {
              "type": "string",
              "description": "JS file (ex: checkoutPaymentForm.js)"
            },
            "funcaoJS": {
              "type": "string",
              "description": "JS function that triggers Apex call"
            },
            "gatilhoInterface": {
              "type": "string",
              "description": "UI trigger (ex: 'Clique no botão Confirmar Pagamento')"
            }
          }
        }
      },
      "customMetadata": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "nome": {
              "type": "string",
              "description": "Custom Metadata Type name"
            },
            "campo": {
              "type": "string",
              "description": "Field name used"
            },
            "valor": {
              "type": "string",
              "description": "Value or placeholder"
            }
          }
        }
      },
      "customLabels": {
        "type": "array",
        "items": {
          "type": "string"
        },
        "description": "Custom labels referenced"
      }
    }
  }
}
```

---

## 6. Definição de PayloadSchema

```json
{
  "PayloadSchema": {
    "type": "object",
    "properties": {
      "request": {
        "type": "object",
        "properties": {
          "dtoClass": {
            "type": "string",
            "description": "DTO class name if applicable (ex: ChargeRequest)"
          },
          "metodo": {
            "type": "string",
            "enum": [
              "DTO_TIPADO",
              "MAP_DINAMICO",
              "STRING_LITERAL",
              "UNKNOWN",
              "BUILDER_PATTERN"
            ],
            "description": "Detection method used"
          },
          "exemplo": {
            "type": "string",
            "description": "Example JSON payload (formatted)"
          },
          "campos": {
            "type": "array",
            "items": {
              "type": "object",
              "properties": {
                "nome": {
                  "type": "string",
                  "description": "Field name"
                },
                "tipo_inferido": {
                  "type": "string",
                  "enum": [
                    "String",
                    "Decimal",
                    "Integer",
                    "Boolean",
                    "Long",
                    "DateTime",
                    "Map",
                    "List",
                    "Unknown"
                  ],
                  "description": "Inferred type"
                },
                "obrigatorio": {
                  "type": "boolean",
                  "description": "Is field required"
                },
                "valor_exemplo": {
                  "description": "Example value for this field"
                },
                "confianca": {
                  "type": "number",
                  "minimum": 0,
                  "maximum": 1,
                  "description": "Confidence of field detection"
                }
              }
            }
          },
          "inferred": {
            "type": "boolean",
            "description": "True if payload was inferred"
          },
          "confianca": {
            "type": "number",
            "minimum": 0,
            "maximum": 1,
            "description": "Overall confidence (0-1)"
          }
        }
      },
      "response": {
        "type": "object",
        "description": "Response schema (same structure as request)"
      }
    }
  }
}
```

---

## 7. Definição de Performance

```json
{
  "Performance": {
    "type": "object",
    "properties": {
      "timeoutMs": {
        "type": "integer",
        "minimum": 1000,
        "maximum": 120000,
        "description": "HTTP request timeout in milliseconds"
      },
      "expectedLatencyMs": {
        "type": "integer",
        "description": "Expected response time (from documentation or observation)"
      },
      "sla": {
        "type": "object",
        "properties": {
          "responseTimeSLA": {
            "type": "string",
            "description": "SLA for response time (ex: '500ms p95')"
          },
          "availabilitySLA": {
            "type": "string",
            "description": "SLA for availability (ex: '99.9%')"
          }
        }
      },
      "rateLimit": {
        "type": "string",
        "description": "Rate limiting info (ex: '1000/hour', '100/minute')"
      },
      "throttling": {
        "type": "object",
        "properties": {
          "maxRequestsPerSecond": {
            "type": "integer"
          },
          "backoffStrategy": {
            "type": "string",
            "enum": ["linear", "exponential", "adaptive"]
          }
        }
      }
    }
  }
}
```

---

## 8. Definição de Resilience

```json
{
  "Resilience": {
    "type": "object",
    "properties": {
      "temRetry": {
        "type": "boolean",
        "description": "Does implementation have retry logic"
      },
      "estrategiaRetry": {
        "type": "string",
        "enum": [
          "nenhuma",
          "conditional-simple",
          "exponential-backoff",
          "queueable-requeue",
          "custom"
        ],
        "description": "Retry strategy used"
      },
      "maxAttempts": {
        "type": "integer",
        "minimum": 1,
        "maximum": 10,
        "description": "Maximum retry attempts"
      },
      "tratamentoErro": {
        "type": "string",
        "description": "How errors are handled (ex: 'Retries on 503, logs to custom object')"
      },
      "fallback": {
        "type": "string",
        "description": "Fallback mechanism if available (ex: 'Uses cached value')"
      },
      "circuitBreaker": {
        "type": "boolean",
        "description": "Has circuit breaker pattern"
      },
      "timeoutEscalation": {
        "type": "boolean",
        "description": "Escalates if timeout"
      }
    }
  }
}
```

---

## 9. Definição de Segurança

```json
{
  "Seguranca": {
    "type": "object",
    "properties": {
      "tipoAutenticacao": {
        "type": "string",
        "description": "Authentication type (repeated from metadadosRede for clarity)"
      },
      "certificatoPinning": {
        "type": "boolean",
        "default": false,
        "description": "Uses certificate pinning"
      },
      "encriptaoEmTransito": {
        "type": "boolean",
        "default": true,
        "description": "HTTPS/TLS used"
      },
      "validacaoSSL": {
        "type": "boolean",
        "default": true,
        "description": "Validates SSL certificates"
      },
      "sensibilidadeDados": {
        "type": "string",
        "enum": ["Publica", "Interna", "Confidencial", "Restrita"],
        "description": "Sensitivity level of data transmitted"
      },
      "conformidade": {
        "type": "array",
        "items": {
          "type": "string",
          "enum": ["PCI-DSS", "LGPD", "GDPR", "SOC2", "ISO27001"]
        },
        "description": "Compliance standards"
      },
      "auditoria": {
        "type": "boolean",
        "default": true,
        "description": "Calls are audited/logged"
      }
    }
  }
}
```

---

## 10. Definição de Rastreabilidade

```json
{
  "Rastreabilidade": {
    "type": "object",
    "properties": {
      "metodoDeteccao": {
        "type": "string",
        "enum": [
          "Discovery_Phase_1",
          "Analysis_Phase_2",
          "Lineage_Phase_3",
          "Manual"
        ],
        "description": "How callout was detected"
      },
      "confiancaGeral": {
        "type": "number",
        "minimum": 0,
        "maximum": 1,
        "description": "Overall confidence score"
      },
      "ultimaAtualizacao": {
        "type": "string",
        "format": "date-time",
        "description": "Last update timestamp"
      },
      "dataDiscoberta": {
        "type": "string",
        "format": "date-time",
        "description": "When first discovered"
      },
      "versaoDoMapper": {
        "type": "string",
        "description": "Version of mapper that found it"
      },
      "observacoes": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "data": {
              "type": "string",
              "format": "date-time"
            },
            "nota": {
              "type": "string",
              "description": "Note from reviewer/update"
            },
            "autor": {
              "type": "string",
              "description": "Who made the note"
            }
          }
        }
      },
      "tags": {
        "type": "array",
        "items": {"type": "string"},
        "description": "Custom tags for organization"
      }
    }
  }
}
```

---

## 11. Exemplo Completo

```json
{
  "metadata": {
    "version": "1.0",
    "generatedAt": "2025-09-15T10:30:00Z",
    "generatedBy": "sf-autonomous-mapper",
    "projectPath": "/home/user/projects/salesforce-payment-platform",
    "totalCallouts": 4,
    "totalLWCComponents": 2,
    "totalApexClasses": 6,
    "coverage": {
      "discoveryRecall": 1.0,
      "discoveryPrecision": 1.0,
      "payloadConfidenceAvg": 0.84,
      "lineageCompletePercentage": 1.0
    }
  },
  "callouts": [
    {
      "id": "CALLOUT-PAY-001",
      "jornada": "Checkout & Pagamentos",
      "dominio": "Financeiro",
      "descricaoBreve": "Executa cobrança no Apigee Payment Gateway",
      "status": "Production",
      "criticidade": "Alta",
      "criticidadeJustificativa": "Usado por 2 componentes LWC, operação DELETE crítica",
      "metadadosRede": {
        "metodo": "POST",
        "basePath": "/v1/payments",
        "endpoint": "/charge",
        "urlCompleta": "callout:Apigee_Payment_Gateway/v1/payments/charge",
        "urlDinamica": false,
        "gateway": "Apigee",
        "tokenEndpoint": "https://auth0.example.com/oauth/v2/token",
        "tipoAutenticacao": "OAuth2 Client Credentials",
        "statusEsperado": [200],
        "headers": {
          "Content-Type": {
            "valor": "application/json",
            "dinamico": false,
            "origem": "codificado"
          },
          "X-Request-ID": {
            "valor": "generateRequestId()",
            "dinamico": true,
            "origem": "codificado"
          }
        }
      },
      "salesforceArtefatos": {
        "namedCredential": "Apigee_Payment_Gateway",
        "namedCredentialUrl": "https://api.apigee.example.com/api/v1/payments",
        "externalCredential": "Apigee_OAuth_Credentials",
        "apexClasses": [
          {
            "nome": "PaymentService.cls",
            "metodoApex": "executePaymentCharge",
            "tipoUso": "Executor",
            "assinatura": "public static ChargeResponse executePaymentCharge(Decimal amount, String currency, String paymentMethod, String orderId)",
            "anotacoes": []
          },
          {
            "nome": "CheckoutController.cls",
            "metodoApex": "processCheckout",
            "tipoUso": "Entry point (@AuraEnabled)",
            "assinatura": "public static void processCheckout(String orderId, Decimal amount, String paymentMethod)",
            "anotacoes": ["@AuraEnabled(cacheable=false)"]
          }
        ],
        "lwcComponentes": [
          {
            "nome": "checkoutPaymentForm",
            "arquivo": "checkoutPaymentForm.js",
            "funcaoJS": "handlePaymentSubmit",
            "gatilhoInterface": "Clique no botão 'Confirmar Pagamento'"
          }
        ],
        "customMetadata": [
          {
            "nome": "Integration_Settings",
            "campo": "Payment_Timeout",
            "valor": "30000"
          },
          {
            "nome": "Integration_Settings",
            "campo": "Retry_Policy",
            "valor": "EXPONENTIAL_BACKOFF"
          }
        ],
        "customLabels": []
      },
      "payloadSchema": {
        "request": {
          "dtoClass": "ChargeRequest",
          "metodo": "DTO_TIPADO",
          "exemplo": "{\n  \"amount\": 150.00,\n  \"currency\": \"BRL\",\n  \"paymentMethod\": \"PIX\",\n  \"orderId\": \"ORD-001\"\n}",
          "campos": [
            {
              "nome": "amount",
              "tipo_inferido": "Decimal",
              "obrigatorio": true,
              "valor_exemplo": 150.00,
              "confianca": 1.0
            },
            {
              "nome": "currency",
              "tipo_inferido": "String",
              "obrigatorio": true,
              "valor_exemplo": "BRL",
              "confianca": 1.0
            },
            {
              "nome": "paymentMethod",
              "tipo_inferido": "String",
              "obrigatorio": true,
              "valor_exemplo": "PIX",
              "confianca": 1.0
            },
            {
              "nome": "orderId",
              "tipo_inferido": "String",
              "obrigatorio": true,
              "valor_exemplo": "ORD-001",
              "confianca": 1.0
            }
          ],
          "inferred": false,
          "confianca": 1.0
        },
        "response": {
          "dtoClass": "ChargeResponse",
          "metodo": "DTO_TIPADO",
          "exemplo": "{\n  \"transactionId\": \"TXN-998123\",\n  \"status\": \"APPROVED\",\n  \"timestamp\": 1695000000000\n}",
          "campos": [
            {
              "nome": "transactionId",
              "tipo_inferido": "String",
              "obrigatorio": true,
              "confianca": 1.0
            },
            {
              "nome": "status",
              "tipo_inferido": "String",
              "obrigatorio": true,
              "confianca": 1.0
            },
            {
              "nome": "timestamp",
              "tipo_inferido": "Long",
              "obrigatorio": false,
              "confianca": 1.0
            }
          ],
          "inferred": false,
          "confianca": 1.0
        }
      },
      "performance": {
        "timeoutMs": 30000,
        "expectedLatencyMs": 500,
        "sla": {
          "responseTimeSLA": "500ms p95",
          "availabilitySLA": "99.9%"
        },
        "rateLimit": "1000/hour",
        "throttling": {
          "maxRequestsPerSecond": 10,
          "backoffStrategy": "exponential"
        }
      },
      "resilience": {
        "temRetry": true,
        "estrategiaRetry": "conditional-simple",
        "maxAttempts": 2,
        "tratamentoErro": "Valida status 200, retry em exceção de conectividade",
        "fallback": "Usa valor cacheado da última transação bem-sucedida",
        "circuitBreaker": false,
        "timeoutEscalation": true
      },
      "seguranca": {
        "tipoAutenticacao": "OAuth2 Client Credentials",
        "certificatoPinning": false,
        "encriptaoEmTransito": true,
        "validacaoSSL": true,
        "sensibilidadeDados": "Restrita",
        "conformidade": ["PCI-DSS", "LGPD"],
        "auditoria": true
      },
      "dependencias": [],
      "dependentes": ["CALLOUT-NOTIF-001"],
      "padraoAvancado": "SIMPLES",
      "explicacaoDetalhada": "Callout acionado quando usuário confirma pagamento no LWC checkoutPaymentForm. O Apex CheckoutController orquestra a chamada com o PaymentService que executa a cobrança no Apigee usando a Named Credential Apigee_Payment_Gateway com autenticação OAuth2 Client Credentials via External Credential Apigee_OAuth_Credentials.",
      "documentacaoExterna": {
        "swaggerUrl": "https://api-docs.apigee.example.com/payments",
        "readmeUrl": "https://wiki.company.com/integration/apigee-payment",
        "contatoTecnico": "payment-team@company.com"
      },
      "rastreabilidade": {
        "metodoDeteccao": "Discovery_Phase_1",
        "confiancaGeral": 0.98,
        "ultimaAtualizacao": "2025-09-15T10:30:00Z",
        "dataDiscoberta": "2025-09-15T10:15:00Z",
        "versaoDoMapper": "1.0.0-beta",
        "observacoes": [
          {
            "data": "2025-09-15T10:30:00Z",
            "nota": "Validado manualmente - Schema 100% correto",
            "autor": "reviewer-1"
          }
        ],
        "tags": ["payment", "critical", "apigee", "oauth2"]
      }
    }
  ],
  "statistics": {
    "jornadas": {
      "Checkout & Pagamentos": 1,
      "Autenticação": 1,
      "Sincronização & Reconciliação": 1,
      "Notificações & Comunicação": 1
    },
    "gateways": {
      "Apigee": 1,
      "Auth0": 1,
      "AWS API Gateway": 1,
      "Notification_Service": 1
    },
    "metodos": {
      "POST": 3,
      "PUT": 1
    },
    "criticidade": {
      "Crítica": 0,
      "Alta": 2,
      "Média": 2,
      "Baixa": 0
    }
  },
  "validationResults": {
    "schemaValidation": {
      "status": "PASSED",
      "errors": [],
      "warnings": []
    },
    "referentialIntegrity": {
      "status": "PASSED",
      "orphans": [],
      "invalidReferences": []
    },
    "payloadQuality": {
      "status": "PASSED",
      "averageConfidence": 0.84,
      "lowConfidenceCallouts": []
    }
  }
}
```

---

## 12. Validação & Conformidade

### Schema Validation

Usar AJV (JSON Schema Validator):

```bash
ajv validate -s schema.json -d database.json
```

### Regras de Integridade

```
✅ Todos os IDs únicos
✅ Todas as dependências resolvem para callouts existentes
✅ Todas as classes Apex referenciadas existem
✅ Todas as LWC referenciadas existem
✅ Confiança entre 0 e 1
✅ Status válido
✅ Jornada não é "UNMAPPED" exceto em casos extremos
```

---

## 13. Próximas Atualizações

Campos reservados para Fase 2 (Multi-ambiente):

```json
{
  "ambientes": {
    "development": { ... },
    "staging": { ... },
    "production": { ... }
  },
  "observabilidade": {
    "metricas": { ... },
    "dashboards": { ... }
  }
}
```
