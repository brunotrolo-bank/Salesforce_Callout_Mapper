---
name: meta-agent-skill-creator
description: Meta-skill que cria novos agentes e novas skills para o Claude Code seguindo a metodologia "Maestros da IA" (do protótipo manual supervisionado à automação total). Use quando o usuário pedir para "criar um agente", "criar uma skill", "gerar um sub-agente", "montar um agente autônomo", ou quando quiser transformar um fluxo de trabalho manual repetido em uma skill/agente reutilizável. NÃO pule direto para gerar código: esta skill impõe um pipeline de validação manual antes de compilar a automação final.
---

# Meta-Skill: Criador de Agentes e Skills

Esta skill implementa a metodologia do canal **Maestros da IA**: a transição de
execução manual supervisionada -> template validado -> skill autônoma
("meta-skill"). O objetivo NÃO é gerar um agente/skill de uma vez só a partir
de uma frase vaga — é rodar um pipeline curto de validação humana antes de
compilar a versão automatizada, para que o resultado final não vire um
"Frankenstein" indepurável.

Regra de ouro herdada da metodologia: **leia e explique ao usuário cada
arquivo de configuração gerado** (`SKILL.md`, frontmatter de agente em
`.claude/agents/*.md`, `CLAUDE.md`, manifests de ferramentas). Nunca gere
um agente/skill "caixa-preta".

## Quando usar esta skill

- Pedido para criar um novo **agente** (subagent, worker, orquestrador).
- Pedido para criar uma nova **skill** para o Claude Code.
- Um fluxo manual repetido nesta conversa que vale a pena virar automação.
- Pedido explícito de "meta-skill", "agente que cria agentes", "fábrica de skills".

## Quando NÃO usar esta skill

- Edição pontual de uma skill/agente já existente **sem mudança de escopo**
  (corrigir um typo, ajustar uma frase, trocar um caminho) — é uma edição
  direta de arquivo, não passa pelo pipeline de 6 fases. Se a edição
  adiciona/remove uma ferramenta, muda o que o artefato tem permissão de
  fazer, ou muda o critério de sucesso, isso É mudança de escopo — trate
  como uma nova Fase 1 (rápida) seguida de Fase 4, mesmo que Fases 2–3
  sejam puladas por já existir template aprovado.
- Dúvidas conceituais sobre como o Claude Code funciona (frontmatter,
  slash commands, hooks) sem intenção de criar um artefato novo — nesse
  caso use `claude-code-guide`.
- Quando já existe, em `templates/`, um template aprovado para exatamente
  este tipo de artefato **e** o pedido é simples/bem entendido: vá direto
  para a Fase 4 reutilizando esse template, sem repetir as Fases 2–3 (ver
  nota de escalonamento de esforço na Fase 3).

## O que esta skill NUNCA faz

- Nunca aceita "crie uma skill que cria agentes" como primeiro e único passo
  e pula direto para gerar arquivos quando não há template aprovado
  reaproveitável. Sempre roda a Fase 1 e, na ausência de um template
  aprovado equivalente em `templates/`, também as Fases 2–3, antes da
  Fase 4 — com o esforço da Fase 3 escalado à complexidade do pedido (ver
  regra de escalonamento de esforço na Fase 3).
- Nunca inventa nomes de ferramentas, variáveis de ambiente, chaves de API
  ou caminhos de diretório. Se algo é necessário e desconhecido, pergunta.
- Nunca escala silenciosamente uma falha crítica (permissão de terminal,
  API key ausente, dependência quebrada) — ver Fase 6.

---

## Pipeline (6 fases)

### Fase 1 — Molde do Arquiteto (definir a persona antes do código)

Antes de escrever qualquer arquivo, defina com o usuário (ou com premissas
explícitas registradas, se o pedido já for claro):

1. **Papel/persona do agente ou objetivo da skill** — uma frase de propósito.
2. **Ferramentas de sistema necessárias** — leitura/escrita de arquivo,
   execução de shell, chamadas de rede/API, acesso a MCP servers.
3. **Arquivos de instrução base**:
   - `SKILL.md` (para skills) ou o arquivo de frontmatter do subagente em
     `.claude/agents/<nome>.md` (para agentes) — contrato de comportamento,
     escopo, limites.
   - `CLAUDE.md` de projeto, se o agente/skill precisa de contexto
     persistente do repositório.
4. **Critério de sucesso** — como saberemos que o agente/skill funcionou.

Se qualquer um desses 4 pontos for ambíguo, pare e pergunte ao usuário
(via `AskUserQuestion` quando disponível) em vez de assumir.

**Saída da Fase 1:** um resumo curto (3-6 linhas) do molde, mostrado ao
usuário antes de prosseguir. Exemplo real (pedido: "crie um agente que
resume as principais causas de erro de um arquivo de log"):

> **Molde:** agente `log-error-summarizer` — lê um arquivo de log, agrupa
> ocorrências por causa raiz e devolve um ranking com contagem.
> **Ferramentas:** `Read`, `Grep` (leitura e contagem; sem `Bash`, sem
> escrita — não precisa modificar nada).
> **Arquivos:** `.claude/agents/log-error-summarizer.md`; sem `CLAUDE.md`
> (não depende de contexto do projeto).
> **Sucesso:** dado um caminho de log, devolve uma lista rankeada de causas
> com contagem, sem exigir que o usuário descreva o formato do log antes.

Esse nível de densidade (curto, concreto, sem enrolação) é o que se espera
— não um parágrafo de justificativa nem uma lista genérica.

### Fase 2 — Execução manual de um caso de teste ("prompt a prompt")

Não pule para a meta-skill. Construa **um agente/skill de teste simples**
primeiro, manualmente, passo a passo:

1. Peça/gere a estrutura mínima do agente de teste (ex.: um agente
   "analista" simples, ou a skill mais simples possível dentro do escopo
   pedido).
2. Gere e mostre o código/arquivo gerado — não rode em lote, avalie linha a
   linha o que foi criado.
3. Adicione as ferramentas que esse agente/skill realmente vai usar, uma a
   uma, testando cada uma.
4. Resolva imediatamente qualquer problema encontrado: diretório errado,
   variável de ambiente/API key faltando, dependência ausente, permissão de
   ferramenta negada. **Registre cada correção** — elas viram regras da
   meta-skill na Fase 4.

Mantenha um log informal de decisões e correções durante esta fase (pode
ser apenas no histórico da conversa) — ele é a matéria-prima da Fase 4.

### Fase 3 — Validação e refinamento de templates

Antes de fixar a estrutura definitiva:

1. Gere **5 variações de template** para a base do agente/skill (estrutura
   de arquivos, frontmatter, organização de seções).
2. Apresente as 5 ao usuário (ou avalie você mesmo contra os critérios de
   `references/template-checklist.md`) e escolha a mais limpa/mínima.
3. Gere mais 5 variações **em cima da escolhida**, refinando.
4. Repita até a estrutura estar estável (sem mudanças estruturais entre
   rodadas) — isso normalmente leva 1–2 iterações, não mais.

Critérios de "template limpo" (ver `references/template-checklist.md`):
frontmatter mínimo e correto, descrição de trigger específica (não vaga),
sem ferramentas não usadas, sem seções vazias, sem instruções redundantes
com o comportamento padrão do Claude Code.

**Regra de escalonamento de esforço** (evita desperdício de token, ver
Fase 5): o ciclo completo de 5+5 variações é para artefatos **novos ou
arquiteturalmente diferentes** de tudo que já existe em `templates/`. Para
um pedido simples e bem entendido cujo tipo de artefato já tem um template
aprovado em `templates/skill-template.md` ou `templates/agent-template.md`,
pule direto para a Fase 4 usando esse template — não regenere 10 variações
de algo já validado. Rode o ciclo completo (ou uma versão reduzida, 2–3
variações) apenas quando o template existente não cobrir bem o caso novo.

**Saída da Fase 3:** um template final aprovado, salvo em
`templates/agent-template.md` ou `templates/skill-template.md` (reutilizável
em execuções futuras desta própria meta-skill — não regere do zero toda vez).

### Fase 4 — Geração da meta-skill / do artefato final

Com o caso de teste validado (Fases 1–3), compile o resultado final:

1. Se o pedido original era **criar UM agente/skill específico**: gere o
   artefato final usando o template aprovado na Fase 3, incorporando cada
   correção técnica da Fase 2 como regra explícita (não como comentário
   solto) no `SKILL.md`/frontmatter do agente.
2. Se o pedido original era **criar a própria meta-skill** (uma skill que
   cria outras skills/agentes): gere esta seção do pipeline como o corpo do
   novo `SKILL.md`, transformando cada feedback e correção técnica do
   usuário durante a sessão em "regras inquebráveis" explícitas na nova
   skill — não em texto vago.
3. Estrutura de arquivos padrão gerada:
   ```
   .claude/skills/<nome-da-skill>/SKILL.md
   .claude/skills/<nome-da-skill>/templates/       # templates reutilizáveis
   .claude/skills/<nome-da-skill>/references/       # checklists, regras
   .claude/agents/<nome-do-agente>.md               # se for um agente
   ```
4. Toda skill/agente gerado por esta meta-skill DEVE incluir, seguindo as
   seções já definidas em `templates/skill-template.md` ou
   `templates/agent-template.md` (os nomes exatos das seções podem variar
   entre os dois templates, mas o conteúdo abaixo é obrigatório nos dois):
   - `description`/frontmatter com gatilhos concretos (quando usar / quando
     não usar) — nunca uma frase genérica.
   - Uma seção listando exatamente as ferramentas validadas na Fase 2
     (nunca uma lista genérica "todas as ferramentas").
   - Uma seção com as falhas mapeadas na Fase 2 e como o artefato gerado
     evita repeti-las.

**Nunca gere o artefato final sem antes ter feito a Fase 1 nesta mesma
sessão** (o molde do arquiteto é sempre obrigatório, mesmo que rápido) **e
sem ter concluído as Fases 2–3 nesta sessão ou em uma execução anterior
cujo template aprovado já exista em `templates/`** (ver regra de
escalonamento de esforço na Fase 3).

### Fase 5 — Otimização em contexto limpo

A skill/agente só está "pronta" depois de testada fora do contexto que a
criou:

1. Informe ao usuário: "recomendo testar isto numa sessão/terminal novo,
   sem o histórico desta conversa, para validar de verdade."
2. Ao testar (nesta sessão ou relatado pelo usuário em outra), busque
   ineficiências de token: chamadas redundantes, geração de código do zero
   quando um template em `templates/` já resolveria, passos que poderiam
   ser um script determinístico em vez de raciocínio do modelo.
3. Prefira sempre **reusar templates salvos** (`templates/*.md`) em vez de
   reinventar a estrutura a cada execução — essa é a otimização central da
   metodologia. Se o mesmo passo determinístico se repetir em múltiplas
   gerações (ex.: sempre criar os mesmos 3 diretórios), considere propor um
   script auxiliar em `scripts/` dentro da skill/agente gerado — mas só
   crie esse diretório quando houver de fato um script, nunca como pasta
   vazia "por precaução".
4. Sempre registre em `CHANGELOG.md` o que foi aprendido nesse teste (o quê
   e por quê, uma linha) — mesmo quando a lição é específica de domínio e
   não justifica mudar um template genérico. Quando a lição for genérica o
   suficiente para valer para outros agentes/skills (não específica deste
   caso), atualize também `templates/` e/ou `references/`. Sem o registro
   em `CHANGELOG.md`, a próxima execução desta meta-skill não sabe por que
   o template é como é, e pode revertê-lo sem querer.

### Fase 6 — Escalonamento para humanos (falha crítica nunca é silenciosa)

Esta é uma skill sob demanda (não agendada), mas toda skill/agente que ela
gerar deve tratar falhas críticas assim:

1. Classifique falhas em dois níveis:
   - **Recuperável**: a própria skill/agente tenta resolver (ex.: reler
     um arquivo de config, pedir permissão de ferramenta novamente).
   - **Crítica**: bloqueia a execução por completo (permissão de terminal
     negada, API key ausente/inválida, dependência que não pode ser
     instalada, diretório inexistente que não deveria faltar).
2. Numa falha crítica, a skill/agente gerado **nunca falha em silêncio**:
   deve reportar de forma clara ao usuário (na conversa) o que travou e
   por quê, incluindo o comando/arquivo exato que causou o erro.
3. Se houver um canal de notificação externo configurado no projeto (ex.:
   MCP de WhatsApp/Evolution API já disponível na sessão), a skill/agente
   pode usar esse canal para alertar sobre a falha crítica — mas **nunca
   presuma que esse canal existe**: verifique se há uma ferramenta MCP
   correspondente disponível antes de tentar usá-la, e trate a ausência
   dela como algo normal, não como erro adicional.
4. Nunca insira credenciais, tokens ou números de telefone diretamente no
   `SKILL.md` ou no código gerado — referencie variáveis de ambiente/config
   já existentes no projeto.

---

## Exemplo de execução completa

Pedido: "crie um agente que resume as principais causas de erro de um
arquivo de log". Este é o mesmo caso usado como exemplo na Fase 1 acima;
aqui está o pipeline inteiro:

1. **Fase 1** — molde definido (ver exemplo na Fase 1). Sem ambiguidade,
   segue sem perguntar.
2. **Fase 2** — rascunho inicial listava `Read`, `Grep` e `Bash` (para
   `wc -l` contar ocorrências). Ao testar, a contagem via `Bash` era
   redundante: o modo `count` do próprio `Grep` já resolve, sem abrir uma
   ferramenta de shell genérica para um agente só de leitura. **Correção
   registrada:** nunca inclua `Bash` num agente somente-leitura quando a
   ferramenta especializada (`Grep`/`Glob`) já cobre o caso.
3. **Fase 3** — o pedido é simples e `templates/agent-template.md` já
   cobre bem esse tipo de agente (leitura + análise) → regra de
   escalonamento de esforço aplicada, pula direto para a Fase 4 sem gerar
   variações novas.
4. **Fase 4** — gera `.claude/agents/log-error-summarizer.md` a partir do
   template, com `tools: Read, Grep` (não `Bash`), e a seção de falhas
   conhecidas documentando: "se o formato do log for desconhecido, não
   tente adivinhar o parser — peça uma amostra de 5-10 linhas ao usuário".
5. **Fase 5** — recomendado testar em sessão nova; a correção da Fase 2
   (não usar `Bash` à toa) é registrada em `CHANGELOG.md` como regra
   reutilizável — o template genérico (`agent-template.md`) continua
   agnóstico de domínio, mas a próxima vez que alguém gerar um agente
   parecido (leitura + análise), essa lição já está documentada em vez de
   precisar ser redescoberta.
6. **Fase 6** — se o log não existir no caminho informado, o agente reporta
   isso como falha crítica ao usuário (arquivo não encontrado) em vez de
   inventar um resumo vazio.

## Templates e referências desta skill

- `templates/skill-template.md` — esqueleto aprovado para novas skills.
- `templates/agent-template.md` — esqueleto aprovado para novos subagentes.
- `references/template-checklist.md` — critérios de "template limpo" usados
  na Fase 3.
- `references/escalation-rules.md` — como a Fase 6 deve ser implementada em
  cada artefato gerado.
- `CHANGELOG.md` — histórico de tudo que foi aprendido em execuções desta
  skill, com o motivo de cada entrada (ver Fase 5, item 4); nem toda
  entrada implica mudar `templates/` ou `references/` — lições específicas
  de um domínio ficam só registradas aqui. Quando a seção "Não versionado"
  acumular ~10-15 entradas, consolide-a num release datado (ver a
  convenção de manutenção no topo do próprio `CHANGELOG.md`) em vez de
  deixá-la crescer indefinidamente.

Sempre que uma nova iteração desta meta-skill aprender algo (mude ou não um
template), registre em `CHANGELOG.md`; quando a lição for genérica o
suficiente, atualize também `templates/` e/ou `references/` — não deixe a
melhoria apenas na conversa, o ganho de eficiência da Fase 5 depende disso.
