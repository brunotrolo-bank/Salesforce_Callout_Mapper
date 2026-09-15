# 📋 Plano de Execução - SF Autonomous Mapper

## Visão Geral do Projeto

| Item | Detalhe |
|------|---------|
| **Nome da Skill** | `sf-autonomous-mapper` |
| **Escopo** | Mapeamento completo de callouts em ORG Salesforce |
| **Entrada** | Caminho local do projeto Salesforce DX |
| **Saída** | `.callout-kb/database.json` + `docs/index.html` |
| **Autonomia** | 100% (zero-prompt mode) |
| **Duração Estimada** | 4-6 semanas (MVP) |
| **Complexidade** | Alta (múltiplos padrões de código) |

---

## Fase de Planejamento (Semana 1)

### Sprint 1.1: Definição de Arquitetura & Documentação

**Objetivo**: Estabelecer base sólida para desenvolvimento

**Deliverables**:
- [ ] `ARCHITECTURE.md` - Documentado ✅
- [ ] `SCHEMA.md` - Schema JSON expandido
- [ ] `PAYLOAD_DETECTION_STRATEGY.md` - Estratégias de parsing
- [ ] `VALIDATION_PROTOCOL.md` - Protocolo de testes
- [ ] `TEST_CASE.md` - Caso de uso real

**Responsáveis**: Arquiteto (você) + Designer

**Critério de Conclusão**: Todos os documentos revisados e aprovados

---

## Fase de Desenvolvimento (Semanas 2-4)

### Sprint 2.1: Descoberta Iterativa (Fase 1)

**Objetivo**: Implementar scanner completo de artefatos Salesforce

**Tarefas**:
- [ ] Implementar `phase1_discovery.md` (estrutura lógica)
- [ ] Criar função para listar arquivos `.cls`, `.js`, XMLs
- [ ] Implementar cache temporal (`.callout-cache/discovery.json`)
- [ ] Validar com teste local (estrutura mínima)

**Entrada**: Caminho do projeto  
**Saída**: `discovery.json` com índice de arquivos

**Critério de Conclusão**:
- Detecta 100% de arquivos esperados
- Cache funciona corretamente
- Sem erros em projeto com 500+ arquivos

---

### Sprint 2.2: Análise de Apex Classes

**Objetivo**: Extrair contexto técnico de Apex

**Tarefas**:
- [ ] Implementar parsing de `HttpRequest` instâncias
- [ ] Extrair `setEndpoint()`, `setMethod()`, `setHeader()`, `setBody()`
- [ ] Implementar detecção de padrões (@AuraEnabled, @future, Queueable, Continuation)
- [ ] Detectar tratamento de erro (try-catch, retry logic)
- [ ] Teste com 10 classes Apex reais

**Entrada**: Lista de `.cls` arquivos  
**Saída**: `analysis.json` com contexto de cada classe

**Critério de Conclusão**:
- Extrai todos os HttpRequest
- Detecta 95%+ dos padrões
- Confiança média > 80%

---

### Sprint 2.3: Estratégias de Detecção de Payloads

**Objetivo**: Implementar 3 estratégias + fallback inteligente

**Tarefas**:
- [ ] Estratégia 1: DTO Tipado (confiança: 100%)
- [ ] Estratégia 2: Map Dinâmico (confiança: 70-80%)
- [ ] Estratégia 3: String Literal (confiança: 60-70%)
- [ ] Fallback com Pattern Detection
- [ ] Testes unitários para cada estratégia
- [ ] Teste com 20+ payloads diferentes

**Entrada**: Apex code com `setBody()`  
**Saída**: Payload schema JSON com confiança

**Critério de Conclusão**:
- Detecta 90%+ dos payloads
- Confiança marcada corretamente
- Não bloqueia em casos desconhecidos

---

### Sprint 3.1: Análise de LWC & XML Metadata

**Objetivo**: Extrair LWC e configurações

**Tarefas**:
- [ ] Parser para `@salesforce/apex/...` imports
- [ ] Rastrear invocações de métodos Apex
- [ ] Detectar eventos (click, change, etc)
- [ ] Parser para Named Credentials XML
- [ ] Parser para External Credentials XML
- [ ] Parser para Custom Metadata & Labels
- [ ] Testes com 30+ LWC componentes

**Entrada**: Arquivos LWC e XMLs  
**Saída**: `analysis.json` expandido com LWC e metadata

**Critério de Conclusão**:
- Rastreia 100% das importações Apex
- Detecta contextos de disparo
- XML parsing sem erros

---

### Sprint 3.2: Mapeamento de Linhagem

**Objetivo**: Conectar LWC → Apex → Metadata → Endpoint

**Tarefas**:
- [ ] Implementar resolução de classes abstratas/interfaces
- [ ] Rastrear cadeia LWC → Apex → Http
- [ ] Detectar Continuations e @future
- [ ] Construir grafo de dependências
- [ ] Validar linhagem end-to-end

**Entrada**: Dados de Discovery, Analysis, LWC  
**Saída**: `lineage.json` e `relationships.json`

**Critério de Conclusão**:
- 100% dos callouts rastreados até LWC/trigger
- Grafo completo sem gaps
- Sem referências circulares

---

### Sprint 3.3: Compilação da Knowledge Base

**Objetivo**: Consolidar dados em `database.json`

**Tarefas**:
- [ ] Implementar schema JSON final (ver `SCHEMA.md`)
- [ ] Mesclar discovery + analysis + lineage
- [ ] Calcular Jornada (heurísticas semânticas)
- [ ] Calcular Criticidade (dinâmica)
- [ ] Calcular Domain & Tags
- [ ] Validação de schema JSON
- [ ] Teste com estrutura completa

**Entrada**: Todos os caches da fase 1-3  
**Saída**: `.callout-kb/database.json` final

**Critério de Conclusão**:
- Schema JSON válido 100%
- Todos os callouts têm linhagem completa
- Confiança média > 75%

---

### Sprint 4.1: Dashboard HTML (Fase 4)

**Objetivo**: Criar interface interativa profissional

**Tarefas**:
- [ ] Estrutura base HTML (Tailwind CDN)
- [ ] Data embedding (injetar `window.CALLOUT_DATABASE`)
- [ ] Implementar Fuse.js busca fuzzy
- [ ] Criar filtros multidimensionais
- [ ] Criar visualização em lista (padrão)
- [ ] Criar visualização em matriz
- [ ] Cards com linhagem visual
- [ ] Design "Enterprise Density"
- [ ] Dark/Light mode toggle
- [ ] Responsividade mobile
- [ ] Testes de performance

**Entrada**: `database.json`  
**Saída**: `docs/index.html` (arquivo único)

**Critério de Conclusão**:
- Carrega em < 2s
- Busca instantânea (< 100ms)
- Sem AI slop, design profissional
- 0 erros no console JS

---

## Fase de Testes & Validação (Semana 5)

### Sprint 5.1: Validação Pós-Collection

**Objetivo**: Garantir integridade dos dados

**Tarefas**:
- [ ] Validação de schema JSON (ajv ou similar)
- [ ] Validação de conectividade (if ORG accessible)
- [ ] Análise de cobertura (metrics)
- [ ] Detecção de endpoints órfãos
- [ ] Teste de performance em 1000+ callouts
- [ ] Gerar relatório de validação

**Saída**: Relatório de validação + `metrics.json`

**Critério de Conclusão**:
- 99%+ schema validity
- Cobertura > 90%
- Performance aceitável

---

### Sprint 5.2: Teste End-to-End (Caso Real)

**Objetivo**: Validar em ambiente real Salesforce

**Tarefas**:
- [ ] Preparar projeto Salesforce DX de teste
- [ ] Executar skill completa
- [ ] Validar descoberta de callouts conhecidos
- [ ] Validar precisão de linhagem
- [ ] Validar dashboard interativo
- [ ] Gerar relatório de teste

**Caso de Uso**: Ver `TEST_CASE.md`

**Critério de Conclusão**:
- Detecta 100% dos callouts esperados
- Linhagem precisa em 100%
- Dashboard funciona sem erros

---

### Sprint 5.3: Otimizações & Bug Fixes

**Objetivo**: Polir MVP

**Tarefas**:
- [ ] Fix de bugs encontrados em testes
- [ ] Otimização de performance
- [ ] Melhorias de UX/UI baseadas em feedback
- [ ] Validação final de schema
- [ ] Teste de regressão

**Critério de Conclusão**:
- Sem bugs críticos
- Performance otimizada
- Pronto para produção

---

## Fase de Entrega (Semana 6)

### Sprint 6.1: Documentação Final & CI/CD

**Objetivo**: Pronto para uso em produção

**Tarefas**:
- [ ] Documentação de uso (README)
- [ ] Documentação de troubleshooting
- [ ] GitHub Actions workflow
- [ ] Instruções de instalação
- [ ] Exemplos de uso
- [ ] FAQ

**Deliverables**:
- `README.md` - Como usar
- `TROUBLESHOOTING.md` - Problemas comuns
- `.github/workflows/sync-callouts.yml` - CI/CD

**Critério de Conclusão**:
- Documentação completa
- Qualquer desenvolvedor consegue usar
- CI/CD funciona

---

## Estrutura de Arquivos Esperada

```
.claude/skills/sf-autonomous-mapper/
├── SKILL.md                              # Definição da skill
├── README.md                             # Uso & documentação
├── ARCHITECTURE.md                       # Arquitetura (este documento)
├── SCHEMA.md                             # Schema JSON detalhado
├── PAYLOAD_DETECTION_STRATEGY.md         # Estratégias de parsing
├── VALIDATION_PROTOCOL.md                # Protocolos de teste
├── EXECUTION_PLAN.md                     # Este arquivo
├── TEST_CASE.md                          # Caso de uso real
├── IMPLEMENTATION_CHECKLIST.md           # Checklist de tarefas
├── TROUBLESHOOTING.md                    # Problemas comuns
│
├── templates/                            # Templates reutilizáveis
│   ├── discovery-phase-template.md
│   ├── analysis-phase-template.md
│   ├── lineage-phase-template.md
│   └── dashboard-template.html
│
├── references/                           # Guias & referências
│   ├── salesforce-patterns.md            # Padrões Salesforce
│   ├── payload-examples.md               # Exemplos de payloads
│   ├── jornada-heuristics.md             # Heurísticas de jornada
│   └── criticality-scoring.md            # Cálculo de criticidade
│
├── scripts/                              # Utilitários
│   ├── validate-schema.py
│   ├── compare-databases.py
│   └── generate-report.py
│
├── examples/                             # Exemplos
│   ├── sample-database.json
│   ├── sample-dashboard.html
│   └── sample-metrics.json
│
└── CHANGELOG.md                          # Histórico de versões
```

---

## Marcos Principais (Milestones)

| Data | Marco | Status |
|------|-------|--------|
| **Semana 1 - Dia 5** | Documentação completa | 🔄 Em progresso |
| **Semana 2 - Dia 5** | Fase 1 (Descoberta) completa | ⏳ Pendente |
| **Semana 3 - Dia 5** | Fase 2 (Análise) completa | ⏳ Pendente |
| **Semana 3 - Dia 10** | Fase 3 (Linhagem) completa | ⏳ Pendente |
| **Semana 4 - Dia 3** | Fase 4 (Dashboard) completa | ⏳ Pendente |
| **Semana 5 - Dia 3** | Testes & Validação OK | ⏳ Pendente |
| **Semana 5 - Dia 7** | Teste E2E sucesso | ⏳ Pendente |
| **Semana 6 - Dia 3** | Documentação & CI/CD | ⏳ Pendente |
| **Semana 6 - Dia 5** | MVP Pronto | ⏳ Pendente |

---

## Dependências & Riscos

### Dependências

- ✅ Acesso a projeto Salesforce DX local
- ✅ Ferramentas: Claude Code / OpenCode
- ✅ Conhecimento de Apex & LWC
- ✅ Capacidade de testar em ORG real (opcional mas recomendado)

### Riscos & Mitigações

| Risco | Probabilidade | Impacto | Mitigação |
|-------|---------------|--------|-----------|
| Padrões Apex desconhecidos | Média | Alto | Documentar padrões encontrados, fallback inteligente |
| Performance em projetos grandes | Baixa | Alto | Implementar cache e lazy loading |
| Confiança de inferência baixa | Média | Médio | Marcar claramente, validação pós-collection |
| CI/CD complexo | Média | Médio | Template GitHub Actions base |

---

## Critério de Aceitação Final

### ✅ Requisitos Obrigatórios (MVP)

- [ ] Agente executa 100% sem intervenção manual
- [ ] Detecta 95%+ dos callouts existentes
- [ ] Payloads com confiança média > 75%
- [ ] Linhagem rastreada para 100% dos callouts
- [ ] Dashboard interativo e sem erros
- [ ] Schema JSON válido
- [ ] Documentação completa

### ✅ Requisitos Desejáveis (Fase 2)

- [ ] Validação de conectividade com ORG
- [ ] CI/CD GitHub Actions funcional
- [ ] Multi-ambiente (Dev/Staging/Prod)
- [ ] Links diretos para Salesforce Setup
- [ ] Detecção de endpoints órfãos
- [ ] Integração com Salesforce CLI

---

## Próximos Passos Imediatos

1. ✅ Aprovar ARCHITECTURE.md (feito)
2. ⏳ Criar SCHEMA.md (próximo)
3. ⏳ Criar PAYLOAD_DETECTION_STRATEGY.md
4. ⏳ Criar VALIDATION_PROTOCOL.md
5. ⏳ Criar TEST_CASE.md
6. ⏳ Iniciar desenvolvimento Sprint 2.1

---

## Glossário de Termos

| Termo | Definição |
|-------|-----------|
| **Callout** | Chamada HTTP de Apex para endpoint externo |
| **Named Credential** | Configuração Salesforce com endpoint + autenticação |
| **External Credential** | Config de OAuth/autenticação para Named Credential |
| **Jornada** | Área de negócio (Pagamentos, Autenticação, etc) |
| **Linhagem** | Cadeia LWC → Apex → Endpoint |
| **Confiança** | Score 0-1 indicando certeza da inferência |
| **Zero-Prompt Mode** | Agente funciona sem perguntas ao usuário |
| **DTO** | Data Transfer Object (classe tipada) |
| **FFLIB** | Apex Enterprise Patterns framework |

---

## Contato & Escalação

**Responsável**: OpenCode Agent  
**Revisor**: Você (usuário)  
**Escalação crítica**: Se skill não mapear >80% dos callouts
