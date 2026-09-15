---
name: <nome-curto-em-kebab-case>
description: <Uma frase objetiva do que a skill faz> + <gatilhos concretos: "use quando o usuário pedir X, Y, Z"> + <quando NÃO usar, se houver ambiguidade comum>.
---

# <Nome legível da skill>

<Um parágrafo curto: propósito e resultado esperado. Sem enrolação.>

## Quando usar

- <gatilho concreto 1>
- <gatilho concreto 2>

## Quando NÃO usar

- <situação onde outra skill/abordagem é melhor>

## Ferramentas necessárias

<Liste exatamente as ferramentas validadas na Fase 2 do pipeline da
meta-skill. Nunca "todas as ferramentas" — seja específico: Read, Edit,
Bash (apenas comandos X), MCP server Y.>

## Passo a passo

1. <passo determinístico>
2. <passo determinístico>
3. <passo que exige julgamento do modelo, se houver>

## Limites conhecidos

<Liste falhas mapeadas durante a validação manual (Fase 2) e como esta
skill evita repeti-las. Exemplo: "não assume variável de ambiente X —
verifica antes de usar e pede ao usuário se ausente".>

## Escalonamento de falhas

<Copie/adapte de references/escalation-rules.md: o que fazer quando algo
crítico trava a execução.>
