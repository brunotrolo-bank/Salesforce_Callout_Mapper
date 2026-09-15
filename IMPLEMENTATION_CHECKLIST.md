# ✅ Checklist de Implementação

## Visão Geral

Este documento é um **checklist executável** para rastrear o progresso da implementação da skill `sf-autonomous-mapper` de forma consistente.

**Como usar**: 
- Marque [x] ao completar cada tarefa
- Adicione datas ao lado de tarefas completadas
- Escale bloqueadores imediatamente
- Atualize este documento a cada 2-3 horas durante desenvolvimento

---

## Fase 0: Setup & Validação Inicial

### Documentação

- [ ] ARCHITECTURE.md completo e revisado
- [ ] EXECUTION_PLAN.md completo e revisado
- [ ] PAYLOAD_DETECTION_STRATEGY.md completo e revisado
- [ ] VALIDATION_PROTOCOL.md completo e revisado
- [ ] SCHEMA.md completo e revisado
- [ ] TEST_CASE.md completo e revisado
- [ ] Estrutura de pastas criada: `.claude/skills/sf-autonomous-mapper/`

**Data Conclusão**: ___________

---

## Fase 1: Descoberta Iterativa (Sprint 2.1)

### 1.1 Estrutura Base da Skill

- [ ] Criar arquivo `.claude/skills/sf-autonomous-mapper/SKILL.md` base
- [ ] Definir meta-skill conforme pipeline de 6 fases (Fase 1 da meta-skill)
- [ ] Implementar aceitar caminho local como input
- [ ] Criar estrutura de passos: `phase1_discovery()`
- [ ] Implementar logging & cache system
- [ ] Validar error handling básico

**Critério de Aceitação**:
- Skill executa sem erros
- Logs são claros e informativos
- Cache funciona para retomada

**Data Início**: ___________  
**Data Conclusão**: ___________

---

### 1.2 Scanner de Arquivos

- [ ] Implementar busca recursiva por arquivos `.cls`
- [ ] Implementar busca recursiva por arquivos `.js` (LWC)
- [ ] Implementar busca recursiva por `**/namedCredentials/*.namedCredential-meta.xml`
- [ ] Implementar busca recursiva por `**/externalCredentials/*.externalCredential-meta.xml`
- [ ] Implementar busca recursiva por `**/customMetadata/**`
- [ ] Implementar busca recursiva por `**/labels/*.labels-meta.xml`
- [ ] Criar índice em `discovery.json` com:
  - [ ] Caminho completo
  - [ ] Tipo de arquivo
  - [ ] Tamanho
  - [ ] Hash SHA1

**Testes**:
- [ ] Teste com projeto com 100+ arquivos
- [ ] Valida descoberta 100%
- [ ] Índice está correto

**Data Conclusão**: ___________

---

### 1.3 Cache Temporal

- [ ] Criar `.callout-cache/discovery.json`
- [ ] Implementar salvamento a cada 50 arquivos
- [ ] Implementar retomada a partir do último checkpoint
- [ ] Implementar limpeza de cache (após conclusão)
- [ ] Validar integridade do cache

**Testes**:
- [ ] Interrupção & retomada funciona
- [ ] Cache não duplica dados
- [ ] Limpeza remove arquivo corretamente

**Data Conclusão**: ___________

---

## Fase 2: Análise de Contexto (Sprint 2.2 & 2.3)

### 2.1 Parser de Apex - HttpRequest

- [ ] Implementar detecção de `new HttpRequest()`
- [ ] Extrair `req.setEndpoint(...)`
- [ ] Extrair `req.setMethod(...)`
- [ ] Extrair `req.setHeader(...)`
- [ ] Extrair `req.setTimeout(...)`
- [ ] Extrair `req.setBody(...)`
- [ ] Detectar padrão: `Http.send(req)` ou `Http http = new Http()`
- [ ] Salvar localização (linha, classe)

**Testes Unitários**:
- [ ] 20+ exemplos Apex com HttpRequest
- [ ] Detecção 100% de HttpRequest válidos
- [ ] 0 falsos positivos

**Data Conclusão**: ___________

---

### 2.2 Parser de Apex - Padrões Avançados

- [ ] Detectar `@AuraEnabled` anotação
- [ ] Detectar `@future(callout=true)` anotação
- [ ] Detectar `Queueable` implementação
- [ ] Detectar `Continuation` uso
- [ ] Detectar `Database.AllowsCallouts` interface
- [ ] Mapear método de entrada (qual método invoca HTTP)

**Testes**:
- [ ] Detecta 100% de @AuraEnabled
- [ ] Detecta 100% de @future(callout=true)
- [ ] Detecta Queueable classes corretamente

**Data Conclusão**: ___________

---

### 2.3 Detecção de Payloads - Estratégia 1 (DTO)

- [ ] Implementar detecção de `JSON.serialize(new Tipo())`
- [ ] Backtrack para encontrar instância de classe
- [ ] Mapear campos da classe DTO
- [ ] Extrair tipos de campo
- [ ] Gerar exemplo JSON
- [ ] Marcar confiança: 1.0
- [ ] Marcar inferred: false

**Testes**:
- [ ] 10 exemplos com DTO tipado
- [ ] Detecção 100% correta
- [ ] Schema gerado valida perfeitamente

**Data Conclusão**: ___________

---

### 2.4 Detecção de Payloads - Estratégia 2 (Map)

- [ ] Implementar detecção de `new Map<String, Object>()`
- [ ] Rastrear sequência de `put()` calls
- [ ] Para cada put(), extrair tipo de valor
- [ ] Backtrack variáveis para tipagem
- [ ] Lidar com valores aninhados (Maps/Lists)
- [ ] Gerar schema estruturado
- [ ] Calcular confiança (70-85%)
- [ ] Marcar inferred: true

**Testes**:
- [ ] 15 exemplos com Maps
- [ ] Detecção 75%+ correta
- [ ] Confiança marcada apropriadamente

**Data Conclusão**: ___________

---

### 2.5 Detecção de Payloads - Estratégia 3 (String)

- [ ] Implementar detecção de string literal JSON
- [ ] Implementar detecção de template string (com +)
- [ ] Parsear JSON válido
- [ ] Extrair placeholder variables
- [ ] Calcular confiança (60-75%)
- [ ] Marcar inferred: true

**Testes**:
- [ ] 5 exemplos com string literal
- [ ] Detecção 50%+ correta
- [ ] Confiança marcada apropriadamente

**Data Conclusão**: ___________

---

### 2.6 Detecção de Payloads - Fallback

- [ ] Implementar detecção de builder pattern
- [ ] Implementar detecção de função externa
- [ ] Buscar comentários JSDoc
- [ ] Extrair exemplos de comentários
- [ ] Marcar como UNKNOWN
- [ ] Confiança < 0.4
- [ ] NUNCA bloquear execução

**Testes**:
- [ ] 5 exemplos com fallback
- [ ] Nenhum bloqueia a execução
- [ ] Confiança marcada corretamente

**Data Conclusão**: ___________

---

### 2.7 Parser de XML Metadata

- [ ] Parser para `namedCredential-meta.xml`
  - [ ] Extrair endpoint
  - [ ] Extrair principalType
  - [ ] Extrair OAuth token endpoint
  - [ ] Referenciar externalCredential

- [ ] Parser para `externalCredential-meta.xml`
  - [ ] Extrair authenticationProtocol
  - [ ] Extrair scopes
  - [ ] Extrair token endpoint

- [ ] Parser para `customMetadata` XML
  - [ ] Extrair todos os fields
  - [ ] Mapear tipos
  - [ ] Buscar padrões (URL, endpoint, baseUrl)

- [ ] Parser para `labels-meta.xml`
  - [ ] Extrair valores com http:// ou /api/

**Testes**:
- [ ] Validar parsing de 10+ files
- [ ] Extrair 100% dos dados esperados

**Data Conclusão**: ___________

---

### 2.8 Parser de LWC

- [ ] Implementar detecção de `import ... from '@salesforce/apex/...'`
- [ ] Extrair classe e método Apex
- [ ] Rastrear invocações do método Apex
- [ ] Detectar função JS que dispara (event handler)
- [ ] Detectar tipo de disparo (click, change, load, etc)
- [ ] Marcar contexto de UI

**Testes**:
- [ ] 10 exemplos LWC
- [ ] Detecção 100% de imports Apex
- [ ] Rastreamento de invocações correto

**Data Conclusão**: ___________

---

### 2.9 Análise de Contexto (Jornada, Domínio)

- [ ] Implementar heurísticas de jornada:
  - [ ] Nome da classe contém palavra-chave?
  - [ ] Nome do método contém palavra-chave?
  - [ ] Pasta contém padrão?
  - [ ] Custom Metadata tem `business_domain`?

- [ ] Mapear domínios:
  - [ ] "Payment", "Charge", "Transaction" → Pagamentos
  - [ ] "Auth", "OAuth", "Token" → Autenticação
  - [ ] "Sync", "Reconcil" → Sincronização
  - [ ] Etc.

- [ ] Scoring de consenso multi-fonte
- [ ] Fallback para "UNMAPPED"

**Testes**:
- [ ] 20 exemplos com nomes diferentes
- [ ] Acurácia > 85%

**Data Conclusão**: ___________

---

### 2.10 Salvar Cache de Análise

- [ ] Criar `.callout-cache/analysis.json` com todos os dados extraídos
- [ ] Estrutura bem organizada
- [ ] Validar integridade

**Data Conclusão**: ___________

---

## Fase 3: Mapeamento de Linhagem (Sprint 3.2)

### 3.1 Resolução de Classes Abstratas

- [ ] Implementar busca de classe base (extends)
- [ ] Implementar busca de interfaces (implements)
- [ ] Implementar rastreamento de override methods
- [ ] Seguir cadeia até implementação real

**Testes**:
- [ ] 5 exemplos com herança
- [ ] Rastreamento 100% correto

**Data Conclusão**: ___________

---

### 3.2 Construção do Grafo

- [ ] Criar mapping: LWC → Apex methods
- [ ] Criar mapping: Apex methods → Http.send()
- [ ] Criar mapping: HttpRequest → Named Credential
- [ ] Criar mapping: Named Credential → External Credential
- [ ] Criar mapping: Custom Metadata → Endpoints

**Testes**:
- [ ] Grafo completo sem gaps
- [ ] Sem referências circulares
- [ ] Todas as relações 1:N mapeadas

**Data Conclusão**: ___________

---

### 3.3 Detecção de Dependências

- [ ] Identificar callouts que compartilham Named Credential
- [ ] Identificar callouts que compartilham classe Apex
- [ ] Identificar sequências (A deve completar antes de B)
- [ ] Marcar em `relationships.json`

**Testes**:
- [ ] Dependências mapeadas corretamente
- [ ] Sem duplicação

**Data Conclusão**: ___________

---

### 3.4 Calcular Criticidade

- [ ] Contar referências a cada callout
- [ ] Analisar método HTTP (DELETE/PUT mais crítico)
- [ ] Analisar timeout (> 30s é gargalo)
- [ ] Analisar retry (sem retry = mais crítico)
- [ ] Gerar score de criticidade
- [ ] Mapear para: Baixa, Média, Alta, Crítica

**Testes**:
- [ ] Criticidade faz sentido heuristicamente
- [ ] Sem contradições óbvias

**Data Conclusão**: ___________

---

### 3.5 Salvar Cache de Linhagem

- [ ] Criar `.callout-cache/lineage.json`
- [ ] Criar `.callout-kb/relationships.json`
- [ ] Estrutura bem organizada
- [ ] Validar integridade

**Data Conclusão**: ___________

---

## Fase 4: Compilação da Knowledge Base (Sprint 4.1)

### 4.1 Validação de Schema

- [ ] Implementar validador AJV
- [ ] Carregar schema de `SCHEMA.md`
- [ ] Validar cada record
- [ ] Reportar erros especificamente

**Testes**:
- [ ] Schema validation passa 100%
- [ ] Mensagens de erro são claras

**Data Conclusão**: ___________

---

### 4.2 Consolidação em database.json

- [ ] Mesclar discovery + analysis + lineage
- [ ] Aplicar heurísticas dinâmicas:
  - [ ] Jornada
  - [ ] Criticidade
  - [ ] Domain
  - [ ] Tags

- [ ] Enriquecer com explicações detalhadas (natural language)
- [ ] Calcular métricas de cobertura
- [ ] Gerar `database.json` final

**Validações**:
- [ ] Todos os callouts presentes
- [ ] Schema válido 100%
- [ ] Confiança média > 70%

**Data Conclusão**: ___________

---

### 4.3 Compilação do Dashboard HTML

#### 4.3.1 Estrutura Base

- [ ] Criar `docs/index.html` boilerplate
- [ ] Importar Tailwind CSS via CDN
- [ ] Importar Fuse.js via CDN
- [ ] Importar Lucide Icons
- [ ] Injetar `window.CALLOUT_DATABASE = {...}`
- [ ] Implementar estado global com Alpine.js ou vanilla JS

**Data Conclusão**: ___________

---

#### 4.3.2 Header & Métricas

- [ ] Criar header com logo/title
- [ ] Exibir métricas:
  - [ ] Total de callouts
  - [ ] Total de endpoints
  - [ ] Distribution por gateway
  - [ ] % de cobertura

- [ ] Dark/Light mode toggle
- [ ] Search bar global

**Data Conclusão**: ___________

---

#### 4.3.3 Filtros Sidebar

- [ ] Filtro por Jornada (multiselect)
- [ ] Filtro por Gateway (multiselect)
- [ ] Filtro por Método HTTP (GET/POST/PUT/DELETE)
- [ ] Filtro por Criticidade (slider ou multiselect)
- [ ] Botão "Limpar Filtros"
- [ ] Estado syncronizado com URL (opcional, Fase 2)

**Testes**:
- [ ] Filtros funcionam independentemente
- [ ] Múltiplos filtros combinam corretamente
- [ ] Reset limpa tudo

**Data Conclusão**: ___________

---

#### 4.3.4 Busca Fuzzy

- [ ] Integrar Fuse.js
- [ ] Índice campos: endpoint, classe, jornada, explicação
- [ ] Busca em tempo real (< 100ms)
- [ ] Highlight de matches
- [ ] Case-insensitive
- [ ] Trata typos (fuzzy matching)

**Testes**:
- [ ] Busca por "payment" encontra paymentService ✅
- [ ] Busca fuzzy "paymnt" encontra "payment" ✅
- [ ] Performance < 100ms

**Data Conclusão**: ___________

---

#### 4.3.5 Visualização de Cards

- [ ] Card para cada callout com:
  - [ ] ID do callout
  - [ ] Jornada (badge)
  - [ ] Endpoint formatado em código
  - [ ] Método HTTP (badge com cor)
  - [ ] Gateway (badge)
  - [ ] Criticidade (badge com cor)
  - [ ] Preview de linhagem

- [ ] Hover mostra mais informações
- [ ] Click abre painel de detalhes

**Design**:
- [ ] Sem gradientes genéricos ✅
- [ ] Tipografia clara ✅
- [ ] Espaçamento consistente ✅
- [ ] Cores neutras (Slate/Zinc) ✅

**Data Conclusão**: ___________

---

#### 4.3.6 Painel de Detalhes (Modal/Drawer)

- [ ] Abas: Geral, Payload, Apex, Linhagem, Segurança
- [ ] Aba "Geral":
  - [ ] ID, Jornada, Gateway, Endpoint
  - [ ] Explicação completa
  - [ ] Links para documentação externa

- [ ] Aba "Payload":
  - [ ] JSON Request formatado
  - [ ] JSON Response formatado
  - [ ] Botão "Copiar"
  - [ ] Syntax highlighting

- [ ] Aba "Apex":
  - [ ] Classes envolvidas
  - [ ] Métodos específicos
  - [ ] Código snippet (se possível)

- [ ] Aba "Linhagem":
  - [ ] Diagrama visual LWC → Apex → Endpoint
  - [ ] Click em nó mostra detalhes

- [ ] Aba "Segurança":
  - [ ] Tipo autenticação
  - [ ] SLA de resposta
  - [ ] Conformidade (PCI, LGPD, etc)

**Data Conclusão**: ___________

---

#### 4.3.7 Visualização de Linhagem (Diagrama)

- [ ] Implementar canvas ou SVG para grafo
- [ ] Renderizar nós: LWC, Apex, Named Credential, Endpoint
- [ ] Renderizar setas entre nós
- [ ] Hover mostra informações
- [ ] Click em nó navega ou filtra

**Alternativa simplificada**:
- Renderizar texto em formato:
```
checkoutPaymentForm.js 
  ↓ invoca
CheckoutController.processCheckout()
  ↓ chama
PaymentService.executePaymentCharge()
  ↓ http
POST /v1/payments/charge (Apigee)
```

**Data Conclusão**: ___________

---

#### 4.3.8 Responsividade

- [ ] Testa em desktop (1920x1080)
- [ ] Testa em tablet (768x1024)
- [ ] Testa em mobile (375x667)
- [ ] Menu lateral colapsível em mobile
- [ ] Tabelas convertidas em cards em mobile
- [ ] Scroll funciona suavemente

**Data Conclusão**: ___________

---

#### 4.3.9 Performance & Otimizações

- [ ] Carregamento < 2 segundos
- [ ] Busca < 100ms
- [ ] Scroll suave (60 fps)
- [ ] Minify CSS/JS (opcional)
- [ ] Lazy loading de images (se houver)

**Benchmark**:
- [ ] Chrome DevTools: < 2s First Contentful Paint
- [ ] Lighthouse: > 80 performance score

**Data Conclusão**: ___________

---

#### 4.3.10 Dark/Light Mode

- [ ] Detectar preferência do sistema
- [ ] Toggle button no header
- [ ] Salvar preferência em localStorage
- [ ] Aplicar cores apropriadas:
  - [ ] Light: branco/cinza claro
  - [ ] Dark: cinza escuro/preto

**Data Conclusão**: ___________

---

## Fase 5: Testes & Validação (Sprint 5)

### 5.1 Testes Unitários

- [ ] Suite com 50+ testes de payload detection
- [ ] Testes de parsing Apex
- [ ] Testes de parsing XML
- [ ] Testes de rastreamento LWC
- [ ] Executar todos os testes
- [ ] Coverage > 80%

**Comando**:
```bash
pytest tests/ -v --cov=src
```

**Data Conclusão**: ___________

---

### 5.2 Teste com Projeto Pequeno

- [ ] Preparar projeto de teste (~100 callouts)
- [ ] Executar skill
- [ ] Validar descoberta: 100% esperado?
- [ ] Validar análise: payloads detectados?
- [ ] Validar linhagem: completa?
- [ ] Dashboard carrega sem erros?

**Critérios**:
- [ ] Descoberta: 100%
- [ ] Payloads: confiança > 70%
- [ ] Linhagem: 100% completa
- [ ] Dashboard: 0 erros JS

**Data Conclusão**: ___________

---

### 5.3 Teste com Projeto Médio

- [ ] Usar projeto Salesforce real (100-500 callouts)
- [ ] Executar skill
- [ ] Validar Recall > 90%
- [ ] Validar Precision > 95%
- [ ] Performance < 5 min?
- [ ] Dashboard performático?

**Data Conclusão**: ___________

---

### 5.4 Teste End-to-End (Caso Real)

Ver TEST_CASE.md para detalhes.

- [ ] Setup projeto `salesforce-payment-platform`
- [ ] Executar skill
- [ ] Descobrir 4/4 callouts esperados ✅
- [ ] Validar esquema JSON ✅
- [ ] Abrir dashboard no navegador ✅
- [ ] Testar busca e filtros ✅
- [ ] Validar linhagem visual ✅
- [ ] Zero erros no console ✅

**Checklist de Validação Manual**:
- [ ] CALLOUT-PAY-001 encontrado e correto
- [ ] CALLOUT-AUTH-001 encontrado e correto
- [ ] CALLOUT-SYNC-001 encontrado e correto
- [ ] CALLOUT-NOTIF-001 encontrado e correto
- [ ] Busca "payment" funciona
- [ ] Filtro por Jornada funciona
- [ ] Clique em card abre detalhes
- [ ] Payload JSON mostra corretamente
- [ ] Linhagem visual está completa

**Data Conclusão**: ___________

---

### 5.5 Gerar Relatório de Validação

- [ ] Consolidar métricas de todos os testes
- [ ] Documentar issues encontradas
- [ ] Score final: Ready for Production?
- [ ] Criar VALIDATION_REPORT.json

**Data Conclusão**: ___________

---

## Fase 6: Documentação & Entrega (Sprint 6)

### 6.1 Documentação Final

- [ ] README.md - Como usar a skill
- [ ] TROUBLESHOOTING.md - Problemas comuns
- [ ] EXAMPLES.md - Exemplos de uso
- [ ] FAQ.md - Perguntas frequentes

**Data Conclusão**: ___________

---

### 6.2 CI/CD Setup

- [ ] Criar `.github/workflows/sync-callouts.yml`
- [ ] Gatilho em PR com mudanças em `force-app/`
- [ ] Executar skill em CI
- [ ] Gerar relatório de mudanças
- [ ] Falhar se cobertura < 80%
- [ ] Comentar no PR com resultado

**Data Conclusão**: ___________

---

### 6.3 Instrções de Instalação

- [ ] Documentar como copiar skill para `.claude/skills/`
- [ ] Documentar como executar `/sf-autonomous-mapper`
- [ ] Documentar localização dos outputs
- [ ] Criar script de setup (opcional)

**Data Conclusão**: ___________

---

### 6.4 Revisão Final

- [ ] Revisar código quanto a bugs
- [ ] Revisar documentação quanto à clareza
- [ ] Executar testes finais
- [ ] Validação final com stakeholder

**Data Conclusão**: ___________

---

## Fase 7: Entrega & Suporte

### 7.1 Deploy

- [ ] Commit todos os arquivos para GitHub
- [ ] Tag v1.0.0
- [ ] Criar GitHub Release
- [ ] Notificar usuários

**Data Conclusão**: ___________

---

### 7.2 Suporte Inicial (Semana 1)

- [ ] Monitorar issues reportadas
- [ ] Responder dúvidas
- [ ] Fazer hotfixes se necessário
- [ ] Documentar lições aprendidas

**Data Conclusão**: ___________

---

## Resumo de Status

| Fase | Tarefa | Status | Data | Notas |
|------|--------|--------|------|-------|
| 0 | Documentação | ⏳ | | |
| 1 | Descoberta | ⏳ | | |
| 2a | Apex HTTP | ⏳ | | |
| 2b | Padrões | ⏳ | | |
| 2c | Payloads | ⏳ | | |
| 2d | XML/LWC | ⏳ | | |
| 3 | Linhagem | ⏳ | | |
| 4 | Database | ⏳ | | |
| 4.1 | Dashboard | ⏳ | | |
| 5 | Testes | ⏳ | | |
| 6 | Docs | ⏳ | | |
| 7 | Deploy | ⏳ | | |

---

## Métricas de Sucesso

```
MÉTRICA                    META          STATUS
─────────────────────────────────────────────────
Discovery Recall           > 90%         ⏳
Discovery Precision        > 95%         ⏳
Payload Confiança          > 75%         ⏳
Linhagem Completa          100%          ⏳
Dashboard Performance      < 2s          ⏳
Test Coverage              > 80%         ⏳
E2E Test Cases Passed      4/4           ⏳
```

---

## Bloqueadores & Riscos

### Bloqueadores Atuais
(Liste qualquer coisa que está bloqueando progresso)

- [ ] Nenhum identificado

### Riscos Potenciais

| Risco | Probabilidade | Impacto | Mitigation |
|-------|---------------|--------|-----------|
| Padrão Apex novo descoberto | Média | Médio | Implementar fallback genérico |
| Performance em 1000+ callouts | Baixa | Alto | Implementar cache & lazy loading |
| Schema muda durante dev | Baixa | Médio | Versionar schema, migrations |

---

## Notas & Observações

```
[Espaço para anotações livres durante desenvolvimento]

```

---

## Aprovações

- [ ] Arquiteto: ___________________________  Data: ___________
- [ ] QA Lead: ______________________________  Data: ___________
- [ ] Product Owner: _________________________  Data: ___________

---

**Última Atualização**: ___________  
**Próxima Revisão**: ___________
