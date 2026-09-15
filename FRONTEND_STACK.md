# 🎨 Frontend Stack & Integrações Externas

## Visão Geral

Este documento integra todas as **referências externas** recomendadas pelo Gemini, detalhando como cada tecnologia/skill se conecta à implementação da `sf-autonomous-mapper` skill.

**Objetivo**: Garantir que o dashboard final seja profissional, performático e evite "AI aesthetics" genéricas.

---

## 1. Stack de Tecnologias

### 1.1 Core Technologies

```
Frontend Base
├─ HTML5 (single-file, no build)
├─ CSS3 + Tailwind CSS v4 (via CDN)
├─ Vanilla JavaScript (ou Alpine.js para reatividade leve)
└─ JSON Data Embedding (injetar direto no HTML)

Bibliotecas de UI/UX
├─ Fuse.js - Busca fuzzy
├─ Lucide Icons - Ícones vetoriais
├─ Alpine.js - Reatividade leve (opcional)
└─ (Evitar React/Vue - não precisa de build)

Design System
├─ Tailwind CSS Design Tokens
├─ Tailwind Color Palette (Slate/Zinc neutros)
├─ Sistema de Tipografia (Inter + JetBrains Mono)
└─ Component Library (shadcn/ui + Radix UI primitivos)
```

---

## 2. Tecnologias Detalhadas com Integração

### 2.1 Tailwind CSS v4

**O que é**: Utility-first CSS framework via CDN

**Por que usar**:
- ✅ Sem build necessário (CDN)
- ✅ Consistência visual garantida
- ✅ Dark mode nativo
- ✅ Responsividade automática
- ✅ Performance excelente

**Como integrar**:

```html
<!-- Em docs/index.html -->
<head>
    <script src="https://cdn.tailwindcss.com"></script>
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        // Palette neutra (anti-slop)
                        slate: { /* ... */ },
                        zinc: { /* ... */ }
                    },
                    fontFamily: {
                        sans: ['Inter', 'system-ui'],
                        mono: ['JetBrains Mono', 'monospace']
                    }
                }
            }
        }
    </script>
</head>

<body class="bg-white dark:bg-slate-950 text-slate-900 dark:text-slate-50">
    <!-- Seu conteúdo -->
</body>
```

**Em ARCHITECTURE.md**: Seção 7 (Anti-Slop & Qualidade)
- Mencionar: "Use Tailwind CSS colors: Slate/Zinc/Neutral apenas"
- Mencionar: "Sem gradientes genéricos em azul/roxo"

**Em IMPLEMENTATION_CHECKLIST.md**: Sprint 4.3.2 (Header & Métricas)
- [ ] Tailwind CSS CDN importado
- [ ] Config customizado com cores neutras
- [ ] Dark/Light mode toggle funciona

---

### 2.2 Fuse.js

**O que é**: Biblioteca de busca fuzzy performática

**Por que usar**:
- ✅ Busca em tempo real (< 100ms)
- ✅ Tolerância a typos (fuzzy matching)
- ✅ Sem backend necessário
- ✅ Weightable fields (search em múltiplos campos)

**Como integrar**:

```html
<!-- Em docs/index.html -->
<script src="https://cdn.jsdelivr.net/npm/fuse.js@7.0.0"></script>

<script>
    // Ao carregar página
    const fuse = new Fuse(window.CALLOUT_DATABASE.callouts, {
        keys: [
            { name: 'id', weight: 2.0 },
            { name: 'jornada', weight: 1.5 },
            { name: 'metadadosRede.endpoint', weight: 2.0 },
            { name: 'salesforceArtefatos.apexClasses[0].nome', weight: 1.5 },
            { name: 'explicacaoDetalhada', weight: 1.0 }
        ],
        threshold: 0.3,  // Permite typos (fuzzy)
        minMatchCharLength: 2
    });

    // Ao usuário digitar
    function handleSearch(query) {
        const results = fuse.search(query);
        renderResults(results);
    }
</script>
```

**Campos para indexar**:
- `id` (CALLOUT-PAY-001) - peso 2.0
- `jornada` (Pagamentos) - peso 1.5
- `endpoint` (/v1/payments/charge) - peso 2.0
- `apexClasses[].nome` (PaymentService) - peso 1.5
- `explicacaoDetalhada` (descrição natural) - peso 1.0

**Em ARCHITECTURE.md**: Seção 2 (Arquitetura de Componentes)
- Mencionar: "Fuse.js para busca fuzzy instantânea"

**Em IMPLEMENTATION_CHECKLIST.md**: Sprint 4.3.4 (Busca Fuzzy)
- [ ] Fuse.js CDN importado
- [ ] Índice construído com 5 campos
- [ ] Busca < 100ms validada
- [ ] Typo tolerance (fuzzy matching) funciona

---

### 2.3 Lucide Icons

**O que é**: Biblioteca de ícones SVG vetoriais modular

**Por que usar**:
- ✅ Diferenciação visual clara (GET vs POST vs DELETE)
- ✅ Ícones Salesforce nativos (se disponível)
- ✅ Tamanho mínimo (SVG inline)
- ✅ Dark mode automático

**Como integrar**:

```html
<!-- Em docs/index.html -->
<script src="https://cdn.jsdelivr.net/npm/lucide@latest/dist/umd/lucide.min.js"></script>

<!-- Usar ícones em HTML -->
<div class="flex items-center gap-2">
    <i data-lucide="globe"></i>
    <span>Gateway</span>
</div>

<div class="flex gap-4">
    <span class="inline-flex items-center gap-1">
        <i data-lucide="arrow-up" class="text-blue-500"></i>
        GET
    </span>
    <span class="inline-flex items-center gap-1">
        <i data-lucide="arrow-down" class="text-green-500"></i>
        POST
    </span>
    <span class="inline-flex items-center gap-1">
        <i data-lucide="trash-2" class="text-red-500"></i>
        DELETE
    </span>
</div>

<script>
    lucide.createIcons();  // Renderizar após DOM
</script>
```

**Mapeamento de Ícones por Contexto**:

| Contexto | Ícone Lucide | Cor |
|----------|-------------|-----|
| GET | arrow-down-left | blue-500 |
| POST | arrow-down | green-500 |
| PUT | refresh-cw | amber-500 |
| DELETE | trash-2 | red-500 |
| PATCH | pencil | purple-500 |
| Gateway | globe | slate-500 |
| Named Credential | key | slate-500 |
| Apex Class | code | slate-500 |
| LWC Component | layers | slate-500 |
| Criticidade Alta | alert-circle | red-500 |
| Criticidade Média | alert-triangle | amber-500 |
| Criticidade Baixa | info | blue-500 |

**Em IMPLEMENTATION_CHECKLIST.md**: Sprint 4.3.5 (Cards de Callout)
- [ ] Lucide Icons CDN importado
- [ ] Mapeamento de ícones por método HTTP
- [ ] Cores consistentes com design system

---

### 2.4 Alpine.js (Reatividade Leve)

**O que é**: Framework JavaScript leve para reatividade sem build

**Por que usar**:
- ✅ Alternativa leve ao React/Vue
- ✅ Sintaxe declarativa (data binding)
- ✅ Sem necessidade de build
- ✅ Performance excelente

**Como integrar** (Opcional - apenas se necessário reatividade avançada):

```html
<!-- Em docs/index.html -->
<script defer src="https://cdn.jsdelivr.net/npm/alpinejs@3.x.x/dist/cdn.min.js"></script>

<div x-data="calloutApp()" class="space-y-4">
    <!-- Filtros -->
    <div class="flex gap-4">
        <input 
            x-model="search" 
            type="text" 
            placeholder="Buscar callouts..."
            @input="updateResults()"
        >
        <select x-model="filterJornada" @change="updateResults()">
            <option value="">Todas as jornadas</option>
            <template x-for="jornada in jornadas" :key="jornada">
                <option :value="jornada" x-text="jornada"></option>
            </template>
        </select>
    </div>

    <!-- Resultados -->
    <div class="grid gap-4">
        <template x-for="callout in filteredResults" :key="callout.id">
            <div class="border rounded p-4 cursor-pointer" @click="selectCallout(callout)">
                <h3 x-text="callout.id"></h3>
                <p x-text="callout.descricaoBreve"></p>
            </div>
        </template>
    </div>
</div>

<script>
function calloutApp() {
    return {
        search: '',
        filterJornada: '',
        allCallouts: window.CALLOUT_DATABASE.callouts,
        filteredResults: [],
        
        updateResults() {
            this.filteredResults = this.allCallouts.filter(c => {
                const matchSearch = this.search === '' || 
                    c.id.toLowerCase().includes(this.search.toLowerCase()) ||
                    c.explicacaoDetalhada.toLowerCase().includes(this.search.toLowerCase());
                
                const matchJornada = this.filterJornada === '' || 
                    c.jornada === this.filterJornada;
                
                return matchSearch && matchJornada;
            });
        },
        
        selectCallout(callout) {
            // Abrir modal com detalhes
            console.log('Selected:', callout);
        }
    }
}
</script>
```

**Alternativa**: Usar Vanilla JavaScript puro (sem Alpine.js)
- Alpine.js é opcional se preferir vanilla JS
- Benchmark: Alpine.js vs Vanilla não é crítico para 50-200 callouts

**Em IMPLEMENTATION_CHECKLIST.md**: Sprint 4.3.1 (Estrutura Base)
- [ ] Decidir: Alpine.js vs Vanilla JavaScript
- [ ] Se Alpine: CDN importado e testado

---

### 2.5 JSON Data Embedding

**O que é**: Injetar dados JSON diretamente no HTML para zero latência

**Por que usar**:
- ✅ Sem requests HTTP (tudo carregado)
- ✅ Performance zero latency
- ✅ Offline capable
- ✅ Sem dependência de servidor

**Como integrar**:

```html
<!-- Em docs/index.html -->
<script>
    window.CALLOUT_DATABASE = {
        "metadata": { /* ... */ },
        "callouts": [
            {
                "id": "CALLOUT-PAY-001",
                "jornada": "Checkout & Pagamentos",
                // ... 30+ campos
            },
            // ... mais callouts
        ],
        "statistics": { /* ... */ },
        "validationResults": { /* ... */ }
    };
    
    // Inicializar após injeção
    console.log(`Carregado: ${window.CALLOUT_DATABASE.callouts.length} callouts`);
</script>

<!-- Renderizar com dados injetados -->
<script>
    document.addEventListener('DOMContentLoaded', () => {
        const db = window.CALLOUT_DATABASE;
        renderMetrics(db.metadata);
        renderCallouts(db.callouts);
        initSearch(db.callouts);
    });
</script>
```

**Em ARCHITECTURE.md**: Seção 4 (Estrutura de Dados)
- Mencionar: "Injetar database.json como window.CALLOUT_DATABASE"
- Garantir: Zero latência na busca

**Em IMPLEMENTATION_CHECKLIST.md**: Sprint 4.3.1 (Estrutura Base)
- [ ] window.CALLOUT_DATABASE injetado corretamente
- [ ] Verificar no console: window.CALLOUT_DATABASE.callouts.length

---

## 3. Padrões Anti-Slop (Do Gemini & Impeccable)

### 3.1 Diretrizes de Design (Impeccable)

**Inspiração**: https://github.com/pbakaus/impeccable

Aplicar os 6 princípios:

#### 1. Proibição de 'AI Aesthetics'

```css
/* ❌ AI SLOP - PROIBIDO */
.card {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    box-shadow: 0 20px 60px rgba(0,0,0,0.3);
    border-radius: 20px;
}

/* ✅ ENTERPRISE DENSITY - PERMITIDO */
.card {
    background: #ffffff;
    border: 1px solid #e2e8f0;
    border-radius: 6px;
}

.dark .card {
    background: #0f172a;
    border: 1px solid #334155;
}
```

#### 2. Tipografia Profissional

```css
/* Body */
.text-body {
    font-family: 'Inter', 'system-ui', sans-serif;
    font-size: 14px;
    line-height: 1.5;
    font-weight: 400;
}

/* Monospace para código/payload */
.text-mono {
    font-family: 'JetBrains Mono', 'Fira Code', monospace;
    font-size: 12px;
    line-height: 1.4;
}

/* Heading */
.text-heading {
    font-family: 'Inter', sans-serif;
    font-weight: 600;
    line-height: 1.3;
}
```

#### 3. Paleta de Cores Neutra

```css
/* Primária: Slate (neutro profissional) */
--slate-50: #f8fafc;
--slate-100: #f1f5f9;
--slate-200: #e2e8f0;
--slate-300: #cbd5e1;
--slate-400: #94a3b8;
--slate-500: #64748b;
--slate-600: #475569;
--slate-700: #334155;
--slate-800: #1e293b;
--slate-900: #0f172a;

/* Secundária: Cores semanticamente corretas */
--green-500: #22c55e;  /* POST/sucesso */
--blue-500: #3b82f6;   /* GET/info */
--amber-500: #f59e0b;  /* WARNING/PUT */
--red-500: #ef4444;    /* DELETE/crítico */
--purple-500: #a855f7; /* PATCH */
```

#### 4. Estados Interativos Explícitos

```css
/* Hover */
.button:hover {
    background-color: #e2e8f0;
    cursor: pointer;
    transition: background-color 150ms ease-in-out;
}

/* Active */
.button:active {
    background-color: #cbd5e1;
    transform: scale(0.98);
}

/* Focus (acessibilidade) */
.button:focus-visible {
    outline: 2px solid #3b82f6;
    outline-offset: 2px;
}

/* Disabled */
.button:disabled {
    opacity: 0.5;
    cursor: not-allowed;
}
```

#### 5. Micro-interações Discretas

```css
/* Transição suave */
.element {
    transition: all 150ms ease-in-out;
}

/* Hover card lift (não exagerado) */
.card:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 12px rgba(0,0,0,0.1);
}

/* Copy button feedback */
.copy-btn.copied {
    background-color: #22c55e;
    color: white;
    animation: copyFeedback 2s ease-in-out;
}

@keyframes copyFeedback {
    0%, 100% { opacity: 1; }
    50% { opacity: 0.8; }
}
```

#### 6. Densidade Visual (Enterprise Density)

```css
/* Compacto, sem espaço desnecessário */
.table {
    border-collapse: collapse;
    font-size: 13px;
}

.table td {
    padding: 8px 12px;
    border-bottom: 1px solid #e2e8f0;
}

.table tr:hover {
    background-color: #f8fafc;
}

/* Sidebar compacto */
.sidebar {
    width: 280px;
    font-size: 13px;
}

.sidebar-item {
    padding: 6px 12px;
    border-radius: 4px;
}
```

---

### 3.2 Componentes Reutilizáveis (shadcn/ui + Radix)

**Padrão**: Usar primitivos acessíveis de Radix UI + estilo Tailwind

```html
<!-- Card Component -->
<div class="border border-slate-200 rounded-lg p-4 dark:border-slate-700 dark:bg-slate-800">
    <h3 class="font-semibold text-slate-900 dark:text-slate-50">Título</h3>
    <p class="text-sm text-slate-600 dark:text-slate-400 mt-2">Conteúdo</p>
</div>

<!-- Button Component -->
<button class="
    px-4 py-2 
    rounded-lg 
    font-medium 
    text-sm
    bg-slate-100 text-slate-900
    hover:bg-slate-200
    active:bg-slate-300
    focus-visible:outline-2 focus-visible:outline-blue-500 focus-visible:outline-offset-2
    transition-colors duration-150 ease-in-out
    disabled:opacity-50 disabled:cursor-not-allowed
">
    Clique aqui
</button>

<!-- Badge Component -->
<span class="
    inline-flex items-center gap-1
    px-3 py-1
    rounded-full
    text-xs font-medium
    bg-blue-100 text-blue-700
    dark:bg-blue-900 dark:text-blue-200
">
    <i data-lucide="info" class="w-3 h-3"></i>
    Info
</span>
```

---

## 4. Integrações Opcionais (Fase 2+)

### 4.1 shadcn/ui Para Componentes Avançados

**Quando usar**: Se decidir usar React/Vue em Fase 2+

```bash
# Instalação (Fase 2)
npx shadcn-ui@latest init

# Adicionar componentes específicos
npx shadcn-ui@latest add button
npx shadcn-ui@latest add card
npx shadcn-ui@latest add dialog
npx shadcn-ui@latest add select
npx shadcn-ui@latest add table
```

**Para MVP (Fase 1)**: Implementar com Tailwind puro (sem shadcn)

### 4.2 v0 Skill da Vercel (Fase 2+)

**Quando usar**: Se precisar de geração de componentes avançados

```bash
# Integração (Fase 2)
npx v0-cli@latest
```

**Para MVP (Fase 1)**: Implementar manualmente (Fuse.js + Tailwind suficientes)

---

## 5. Checklist de Implementação (Frontend)

### 5.1 Tech Stack Inicial (MVP - Fase 1)

```
✅ OBRIGATÓRIO (MVP)
├─ HTML5 single-file
├─ Tailwind CSS v4 (CDN)
├─ Vanilla JavaScript (ou Alpine.js leve)
├─ Fuse.js (busca fuzzy)
├─ Lucide Icons (ícones)
└─ JSON data embedding

⏳ OPCIONAL (Fase 1)
├─ Alpine.js (se preferir reatividade)

⏳ FASE 2+
├─ React/Next.js (se necessário)
├─ shadcn/ui (componentes avançados)
├─ v0 Skill (geração)
└─ TypeScript (tipagem)
```

### 5.2 Espaço em ARCHITECTURE.md

Adicionar na **Seção 7 (Anti-Slop & Qualidade)**:

```markdown
## 7. Anti-Slop & Qualidade de Frontend

### Tech Stack Aprovado
- ✅ Tailwind CSS v4 (colors: Slate/Zinc only)
- ✅ Fuse.js (busca fuzzy < 100ms)
- ✅ Lucide Icons (ícones semânticos)
- ✅ Alpine.js OU Vanilla JS (reatividade leve)
- ✅ JSON embedding (zero latência)

### Diretrizes Design (Impeccable Pattern)
1. **Proibição de AI Aesthetics**: Sem gradientes genéricos
2. **Tipografia Profissional**: Inter + JetBrains Mono
3. **Paleta Neutra**: Slate/Zinc only
4. **Estados Explícitos**: Hover, Active, Focus, Disabled
5. **Micro-interações**: 150ms ease-in-out, não exagerado
6. **Enterprise Density**: Compacto, sem espaço desnecessário

### Componentes Reutilizáveis
- Card
- Button
- Badge
- Input
- Select
- Table
- Modal/Drawer

(Ver FRONTEND_STACK.md para detalhes)
```

---

## 6. Integração com SCHEMA.md

No `docs/index.html`, garantir:

```html
<!-- Validação de schema ao carregar -->
<script>
    function validateSchema() {
        const db = window.CALLOUT_DATABASE;
        
        // Validar metadata
        if (!db.metadata || !db.metadata.version) {
            console.warn('Schema inválido: faltam metadados');
            return false;
        }
        
        // Validar callouts
        if (!db.callouts || db.callouts.length === 0) {
            console.warn('Schema inválido: nenhum callout encontrado');
            return false;
        }
        
        // Validar campos obrigatórios por callout
        const requiredFields = ['id', 'jornada', 'metadadosRede', 'salesforceArtefatos'];
        for (const callout of db.callouts) {
            for (const field of requiredFields) {
                if (!callout[field]) {
                    console.warn(`Callout ${callout.id} faltando campo obrigatório: ${field}`);
                    return false;
                }
            }
        }
        
        console.log(`✅ Schema válido: ${db.callouts.length} callouts carregados`);
        return true;
    }
    
    validateSchema();
</script>
```

---

## 7. Referências Externas

### Padrões & Práticas
- **Impeccable** (pbakaus): https://github.com/pbakaus/impeccable
- **Enterprise Design Patterns**: Stripe, GitHub, Vercel dashboards

### Bibliotecas
- **Tailwind CSS**: https://tailwindcss.com/
- **Fuse.js**: https://fusejs.io/
- **Lucide Icons**: https://lucide.dev/
- **Alpine.js**: https://alpinejs.dev/
- **shadcn/ui**: https://shadcn-ui.com/
- **Radix UI**: https://www.radix-ui.com/

### Documentações
- **JSON Schema**: https://json-schema.org/
- **WCAG Accessibility**: https://www.w3.org/WAI/intro/wcag
- **Web Performance**: https://web.dev/performance/

---

## 8. Como Este Documento se Conecta

```
ARCHITECTURE.md
  └─ Seção 7 (Anti-Slop)
      └─ Referencia FRONTEND_STACK.md (este documento)
          ├─ Tech Stack detalhado
          ├─ Guia de integração
          └─ Componentes reutilizáveis

IMPLEMENTATION_CHECKLIST.md
  └─ Sprint 4.3 (Dashboard)
      └─ Referencia FRONTEND_STACK.md (este documento)
          ├─ [ ] Tailwind CSS importado
          ├─ [ ] Fuse.js configurado
          ├─ [ ] Lucide Icons renderizado
          └─ [ ] Padrões Anti-Slop aplicados

SCHEMA.md
  └─ Estrutura de dados (database.json)
      └─ Injetado em docs/index.html (window.CALLOUT_DATABASE)
          └─ Validado conforme FRONTEND_STACK.md
```

---

## 9. Exemplo Prático: Component Completo

### Badge de Método HTTP (Com Lucide + Tailwind)

```html
<!-- Component -->
<div class="flex items-center gap-1 px-3 py-1 rounded-full text-xs font-medium"
     x-show="callout.metadadosRede.metodo === 'GET'"
     :class="'bg-blue-100 text-blue-700 dark:bg-blue-900 dark:text-blue-200'">
    <i data-lucide="arrow-down-left" class="w-3 h-3"></i>
    GET
</div>

<div class="flex items-center gap-1 px-3 py-1 rounded-full text-xs font-medium"
     x-show="callout.metadadosRede.metodo === 'POST'"
     :class="'bg-green-100 text-green-700 dark:bg-green-900 dark:text-green-200'">
    <i data-lucide="arrow-down" class="w-3 h-3"></i>
    POST
</div>

<!-- Reutilizável em qualquer lugar -->
```

---

## 10. Próximos Passos

### Imediato
- [ ] Ler seção 7 de ARCHITECTURE.md (atualizado)
- [ ] Adicionar este documento (FRONTEND_STACK.md) ao índice
- [ ] Revisar IMPLEMENTATION_CHECKLIST.md sprint 4.3

### Durante Sprint 4.1
- [ ] Escolher Alpine.js vs Vanilla JS
- [ ] Importar Tailwind CSS (atualizar CDN com config)
- [ ] Importar Fuse.js
- [ ] Importar Lucide Icons

### Durante Sprint 4.3.4
- [ ] Implementar componentes (Card, Button, Badge)
- [ ] Validar padrões Anti-Slop
- [ ] Testar responsividade

---

**Documento Preparado Por**: OpenCode Agent  
**Data**: 2025-09-15  
**Status**: ✅ Completo e Integrado  
**Referências do Gemini**: ✅ Incorporadas  

**Seções Atualizadas**:
- ARCHITECTURE.md (Seção 7)
- IMPLEMENTATION_CHECKLIST.md (Sprint 4.3)
- README_DOCUMENTACAO.md (Índice)
