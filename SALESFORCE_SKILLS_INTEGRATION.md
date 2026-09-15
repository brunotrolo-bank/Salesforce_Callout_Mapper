# 🔗 Integração de Salesforce Skills Oficiais

## Visão Geral

Este documento detalha a integração das **3 top skills oficiais da Salesforce** com `sf-autonomous-mapper`, criando uma solução profissional e de classe mundial para análise de codebase Salesforce.

**Objetivo**: Combinar análise autônoma de callouts com análise de qualidade e estrutura da ORG.

---

## 1. Top 3 Skills Salesforce Selecionadas

### 1.1 dx-org-analyze

**Repositório**: `https://github.com/forcedotcom/sf-skills/tree/main/skills/dx-org-analyze`

**O que faz**:
- ✅ Análise profunda da estrutura de ORG
- ✅ Descobre todos os objetos, campos, configurações
- ✅ Identifica apex classes, LWC, triggers
- ✅ Mapeia permissões e segurança
- ✅ Reporta débito técnico

**Como se conecta com sf-autonomous-mapper**:
```
dx-org-analyze (Descoberta de estrutura)
    ↓
sf-autonomous-mapper (Detecção de callouts)
    ↓
dx-apexguru-scan (Análise de qualidade)
    ↓
dx-code-analyzer-run (Análise estática)
    ↓
Dashboard unificado (Resultados)
```

**Inputs**:
- Caminho do projeto local (mesmo input que sf-autonomous-mapper)
- Acesso à ORG (se validação desejada)

**Outputs**:
- Estrutura de objetos/campos
- Listagem de Apex classes
- Listagem de LWC components
- Configurações de segurança

**Integração na Skill**:
```bash
# Executar antes ou em paralelo com sf-autonomous-mapper
npx skills add forcedotcom/sf-skills
sf skills run dx-org-analyze --path ./force-app
```

---

### 1.2 dx-apexguru-scan

**Repositório**: `https://github.com/forcedotcom/sf-skills/tree/main/skills/dx-apexguru-scan`

**O que faz**:
- ✅ Code quality scanning de Apex
- ✅ Detecta anti-patterns e violations
- ✅ Identifica performance issues
- ✅ Reporta security issues
- ✅ Integra-se com Apex Guru

**Como se conecta com sf-autonomous-mapper**:
```
sf-autonomous-mapper (Descoberta de callouts)
    ↓
dx-apexguru-scan (Code quality dos callouts encontrados)
    ↓
Classificar callouts por risco/qualidade
    ↓
Enriquecer schema com issues
```

**Inputs**:
- Diretório `force-app/main/default/classes/`
- Regras customizadas (opcionais)

**Outputs**:
- Issues por classe Apex
- Severidade (Critical, Major, Minor, Info)
- Padrões de código problemáticos
- Recomendações de fixing

**Integração na Skill**:
```bash
# Executar na Fase 2 (Análise)
sf skills run dx-apexguru-scan --path ./force-app/main/default/classes
```

**Exemplo de Enriquecimento no Schema**:
```json
{
  "id": "CALLOUT-PAY-001",
  "salesforceArtefatos": {
    "apexClasses": [
      {
        "nome": "PaymentService.cls",
        "metodoApex": "executePaymentCharge",
        "tipoUso": "Executor",
        "qualityIssues": [
          {
            "tipo": "ApexGuru",
            "severidade": "Major",
            "mensagem": "Long parameter list detected",
            "recomendacao": "Refactor para usar wrapper objects"
          }
        ]
      }
    ]
  }
}
```

---

### 1.3 dx-code-analyzer-run

**Repositório**: `https://github.com/forcedotcom/sf-skills/tree/main/skills/dx-code-analyzer-run`

**O que faz**:
- ✅ Análise estática profunda
- ✅ Detecta issues de arquitetura
- ✅ Valida padrões de design
- ✅ Integra com regras customizadas
- ✅ Gera relatórios detalhados

**Como se conecta com sf-autonomous-mapper**:
```
sf-autonomous-mapper (Descobre padrões Apex/LWC)
    ↓
dx-code-analyzer-run (Valida padrões contra regras)
    ↓
Classificar issues por tipo
    ↓
Adicionar recomendações ao schema
```

**Inputs**:
- Diretório do projeto
- Arquivo `.sfcodecanalyzer.json` (config de regras)

**Outputs**:
- Issues estruturadas por arquivo
- Violações de regras
- Recomendações de arquitetura
- Relatório consolidado

**Integração na Skill**:
```bash
# Configurar regras
sf config set sf-skills.code-analyzer.rules=recommended

# Executar
sf skills run dx-code-analyzer-run --path ./force-app
```

---

## 2. Arquitetura de Integração

### 2.1 Pipeline Completo

```
┌─────────────────────────────────────────────────────────────┐
│              ENTRADA: Caminho Local ORG                      │
└──────────────────────┬──────────────────────────────────────┘
                       │
          ┌────────────┼────────────┐
          │            │            │
    ┌─────▼─────┐ ┌───▼────┐ ┌────▼──────┐
    │ dx-org-   │ │ sf-    │ │ dx-       │
    │ analyze   │ │ auto-  │ │ apexguru- │
    │ (struct)  │ │ mapper │ │ scan      │
    │           │ │ (call) │ │ (quality) │
    └─────┬─────┘ └───┬────┘ └────┬──────┘
          │            │            │
          └────────────┼────────────┘
                       │
            ┌──────────▼──────────┐
            │ dx-code-analyzer-run│
            │ (architecture)      │
            └──────────┬──────────┘
                       │
          ┌────────────▼────────────┐
          │ Consolidação de Dados   │
          │ - Merge estrutura       │
          │ - Merge quality issues  │
          │ - Merge arch violations │
          └──────────┬─────────────┘
                     │
          ┌──────────▼──────────────┐
          │ database.json Enriquecido
          │ + qualityIssues         │
          │ + architectureIssues    │
          └──────────┬──────────────┘
                     │
          ┌──────────▼──────────────┐
          │ Dashboard Unificado     │
          │ - Callouts              │
          │ - Quality Score         │
          │ - Architecture Score    │
          └─────────────────────────┘
```

### 2.2 Sequência Temporal

**Fase 1: Descoberta (Paralelo)**
```
Início
  ├─ dx-org-analyze (2-3 min)
  ├─ sf-autonomous-mapper Fase 1 (1-2 min)
  └─ Aguardar conclusão

Saída:
  ├─ discovery.json (estrutura)
  └─ discovery.json (callouts)
```

**Fase 2: Análise (Sequencial)**
```
sf-autonomous-mapper Fase 2
  ├─ Análise de contexto
  ├─ Payload detection
  └─ Linhagem

Em paralelo:
  ├─ dx-apexguru-scan (todas classes)
  └─ Enriquecer análise.json com quality issues
```

**Fase 3: Arquitetura (Sequencial)**
```
dx-code-analyzer-run
  ├─ Validar padrões
  ├─ Detectar issues
  └─ Gerar relatório

Consolidar resultados
  ├─ Merge quality issues
  ├─ Merge architecture issues
  └─ Calcular scores
```

**Fase 4: Compilação**
```
database.json final
  ├─ Callouts + payloads
  ├─ Quality issues por classe
  ├─ Architecture issues
  └─ Scores calculados
```

---

## 3. Integração no SCHEMA.md

Adicionar novos campos ao schema de Callout:

### 3.1 Campos de Qualidade

```json
{
  "id": "CALLOUT-PAY-001",
  "qualidade": {
    "apexGuruIssues": [
      {
        "severidade": "Major",
        "tipo": "Long parameter list",
        "classe": "PaymentService.cls",
        "linha": 42,
        "mensagem": "Method has 8 parameters",
        "recomendacao": "Refactor para usar wrapper objects"
      }
    ],
    "scoreQualidade": 0.75,
    "scoreArquitetura": 0.82,
    "scoreFinal": 0.78
  }
}
```

### 3.2 Campos de Análise Estática

```json
{
  "id": "CALLOUT-PAY-001",
  "arquitetura": {
    "codeAnalyzerIssues": [
      {
        "regra": "apex-best-practices",
        "severidade": "Warning",
        "mensagem": "Avoid hardcoded values",
        "localizacao": "PaymentService.cls:55",
        "recomendacao": "Use Custom Metadata ou Custom Label"
      }
    ],
    "patternsDetected": [
      "Queueable",
      "ErrorHandling",
      "Retry-Logic"
    ],
    "debitotecnico": "Baixo"
  }
}
```

---

## 4. Implementação em EXECUTION_PLAN.md

### Adicionar Sprint 2.0: Pré-Análise com Salesforce Skills

**Sprint 2.0: Análise com Salesforce Skills Oficiais** (Novo - Dia 0-1)

```
Objetivo: Coletar metadados estruturais e análise de qualidade

Tarefas:
- [ ] Instalar sf-skills CLI
- [ ] Executar dx-org-analyze em paralelo
- [ ] Capturar estrutura de ORG (objetos, campos, classes)
- [ ] Prépara dx-apexguru-scan para Fase 2
- [ ] Validar outputs (discovery.json estrutura)

Entrada: Caminho do projeto
Saída: discovery.json com estrutura (complementar ao discovery de callouts)

Critério de Conclusão:
- dx-org-analyze conclui com sucesso
- Estrutura de ORG capturada
- Sem erros de CLI
```

### Modificar Sprint 2.2: Integrar ApexGuru

**Sprint 2.2: Análise Apex + Quality Scan**

```
Modificações:
- [ ] Executar dx-apexguru-scan paralelo
- [ ] Coletar issues por classe Apex
- [ ] Mapear issues para callouts encontrados
- [ ] Enriquecer analysis.json com qualityIssues

Nova saída:
  - analysis.json com payload + quality issues
```

### Novo Sprint 3.0: Análise Estatática

**Sprint 3.0: Code Analysis & Architecture Validation**

```
Objetivo: Validar arquitetura e detectar issues

Tarefas:
- [ ] Configurar .sfcodecanalyzer.json
- [ ] Executar dx-code-analyzer-run
- [ ] Coletar architecture violations
- [ ] Mapear issues para callouts
- [ ] Gerar relatório de débito técnico

Saída: architecture-issues.json

Critério de Conclusão:
- Análise completa
- Issues mapeadas para callouts
- Scores calculados
```

---

## 5. Modificação no SCHEMA.md

### Adicionar Definição de Qualidade

```json
{
  "Qualidade": {
    "type": "object",
    "properties": {
      "apexGuruIssues": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "severidade": {
              "type": "string",
              "enum": ["Critical", "Major", "Minor", "Info"]
            },
            "tipo": {"type": "string"},
            "classe": {"type": "string"},
            "linha": {"type": "integer"},
            "mensagem": {"type": "string"},
            "recomendacao": {"type": "string"}
          }
        }
      },
      "scoreQualidade": {
        "type": "number",
        "minimum": 0,
        "maximum": 1,
        "description": "Score de qualidade Apex (0-1)"
      },
      "scoreArquitetura": {
        "type": "number",
        "minimum": 0,
        "maximum": 1,
        "description": "Score de arquitetura (0-1)"
      },
      "scoreFinal": {
        "type": "number",
        "minimum": 0,
        "maximum": 1,
        "description": "Score final consolidado (0-1)"
      }
    }
  }
}
```

---

## 6. Modificação no IMPLEMENTATION_CHECKLIST.md

### Adicionar Seção de SF-Skills

**Sprint 2.0: SF-Skills Pre-Analysis** (Novo)

- [ ] Instalar `@salesforce/cli`
- [ ] Instalar `sf-skills`
- [ ] Executar `dx-org-analyze`
- [ ] Validar saída de estrutura
- [ ] Armazenar em `.callout-cache/org-structure.json`

**Sprint 2.2: ApexGuru Integration**

- [ ] Executar `dx-apexguru-scan`
- [ ] Mapear issues para classes Apex
- [ ] Enriquecer `analysis.json`
- [ ] Calcular `scoreQualidade`

**Sprint 3.0: Code Analyzer Integration**

- [ ] Criar `.sfcodecanalyzer.json`
- [ ] Executar `dx-code-analyzer-run`
- [ ] Coletar violations
- [ ] Mapear para callouts
- [ ] Calcular `scoreArquitetura`

---

## 7. Instalação & Uso

### 7.1 Setup Inicial

```bash
# 1. Instalar CLI da Salesforce
npm install --save-dev @salesforce/cli

# 2. Instalar sf-skills
npx skills add forcedotcom/sf-skills

# 3. Verificar instalação
sf skills list
```

### 7.2 Executar Skills

```bash
# Fase 1: Descoberta (Paralelo)
sf skills run dx-org-analyze --path ./force-app &
/sf-autonomous-mapper --phase=1 &

# Fase 2: Análise
sf skills run dx-apexguru-scan --path ./force-app/main/default/classes
/sf-autonomous-mapper --phase=2

# Fase 3: Arquitetura
sf skills run dx-code-analyzer-run --path ./force-app
/sf-autonomous-mapper --phase=3

# Fase 4: Compilação
/sf-autonomous-mapper --phase=4
```

### 7.3 Integração na Skill

Adicionar ao SKILL.md:

```yaml
---
name: sf-autonomous-mapper
description: |
  Agente autônomo que mapeia callouts + integra com Salesforce Skills oficiais
  para análise completa: descoberta (dx-org-analyze), qualidade (dx-apexguru-scan),
  arquitetura (dx-code-analyzer-run).
dependencies:
  - forcedotcom/sf-skills
  - @salesforce/cli >= 2.0.0
---
```

---

## 8. Outputs Consolidados

### 8.1 database.json Enriquecido

```json
{
  "metadata": { /* ... */ },
  "callouts": [
    {
      "id": "CALLOUT-PAY-001",
      "jornada": "Checkout & Pagamentos",
      "explicacaoDetalhada": "...",
      "qualidade": {
        "apexGuruIssues": [ /* ... */ ],
        "scoreQualidade": 0.75,
        "scoreArquitetura": 0.82,
        "scoreFinal": 0.78,
        "recomendacoes": [
          "Refactor PaymentService para usar wrapper objects",
          "Usar Custom Metadata para hardcoded values"
        ]
      },
      "salesforceArtefatos": { /* ... */ }
    }
  ],
  "statistics": {
    "qualidade": {
      "scoreQualidadeMedia": 0.78,
      "scoreArquiteturaMedia": 0.82,
      "apexGuruIssuesTotal": 12,
      "codeAnalyzerIssuesTotal": 5,
      "debitoTecnicoTotal": "Baixo"
    }
  }
}
```

### 8.2 Dashboard Enriquecido

Adicionar ao dashboard:

```html
<!-- Quality Tabs -->
<div class="tabs">
  <div id="tab-general">Geral</div>
  <div id="tab-quality">Qualidade (ApexGuru)</div>
  <div id="tab-architecture">Arquitetura (Code Analyzer)</div>
</div>

<!-- Quality Score Card -->
<div class="quality-card">
  <div class="score-circle" data-score="0.78">78%</div>
  <div>
    <h4>Score de Qualidade</h4>
    <p>ApexGuru Issues: 12</p>
    <p>Arch Issues: 5</p>
    <p>Débito Técnico: Baixo</p>
  </div>
</div>

<!-- Quality Issues List -->
<div id="quality-issues">
  <h4>Issues Detectadas</h4>
  <div class="issue-item">
    <span class="severity-major">MAJOR</span>
    <span class="tipo">Long parameter list</span>
    <span class="recomendacao">Refactor para wrapper</span>
  </div>
</div>
```

---

## 9. Benefícios da Integração

| Benefício | Descrição |
|-----------|-----------|
| **Análise Completa** | Estrutura + Callouts + Qualidade + Arquitetura |
| **Oficial** | Skills desenvolvidas e mantidas pela Salesforce |
| **Escalável** | Funciona em projetos pequenos e grandes |
| **CI/CD Ready** | Todas as skills suportam automação |
| **Rastreável** | Scores quantificáveis e comparáveis |
| **Melhorias** | Recomendações acionáveis para cada issue |
| **Débito Técnico** | Identificar e priorizar refactoring |

---

## 10. Próximos Passos

### Imediato
- [ ] Ler este documento completo
- [ ] Setup inicial (instalar sf-skills)
- [ ] Testar cada skill isoladamente

### Fase 1
- [ ] Modificar EXECUTION_PLAN.md (adicionar Sprint 2.0 e 3.0)
- [ ] Modificar SCHEMA.md (adicionar campos de qualidade)
- [ ] Modificar IMPLEMENTATION_CHECKLIST.md (adicionar tasks)

### Fase 2
- [ ] Implementar integração no código da skill
- [ ] Consolidar outputs de 3 skills
- [ ] Enriquecer database.json

### Fase 3
- [ ] Atualizar dashboard com quality metrics
- [ ] Criar visualizações de scores
- [ ] Adicionar filtros por qualidade/arquitetura

---

## 11. Referências

- **sf-skills Repository**: https://github.com/forcedotcom/sf-skills
- **dx-org-analyze**: https://github.com/forcedotcom/sf-skills/tree/main/skills/dx-org-analyze
- **dx-apexguru-scan**: https://github.com/forcedotcom/sf-skills/tree/main/skills/dx-apexguru-scan
- **dx-code-analyzer-run**: https://github.com/forcedotcom/sf-skills/tree/main/skills/dx-code-analyzer-run
- **Salesforce CLI**: https://developer.salesforce.com/tools/salesforcecli
- **ApexGuru**: https://apexguru.io/

---

**Documento Preparado Por**: OpenCode Agent  
**Data**: 2025-09-15  
**Status**: ✅ Completo e Pronto para Integração  
**Integração Recomendada**: Opção A (Top 3 Skills)
